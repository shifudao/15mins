# Bottled Water简介
Bottled Water是[Confluent](http://www.confluent.io/)公司开发的一款可以将postgresql数据库转换为kafka events的工具。

Bottled Water可以实时地将postgresql的变化推送至kafka中。有以下几个主要特性:

- 支持postgresql 9.4及以后版本。不限数据库的schema。
- 无需schema changes，无需触发器及额外表。(但是需要添加一个扩展)
- 几乎不影响数据库本身性能。
- 事务一致性的输出。即: 只有写入操作成功commit数据库之后才会将变化写入kafka，顺序与commit顺序一致。
- 容错。在进程崩溃、机器故障、网络中断等情况下不会丢失数据。

## 为什么使用Bottled Water
在某些场景下，可能需要将数据存储在不止一个数据存储中。比如，为了搜索方便快速，可能会考虑冗余存储一份数据到elasticsearch或某些缓存数据库中；为了迁移到不同的数据库，在迁移过程中也可能会同时存储多种不同的数据库。

这种需求下，常见的一种实现方式是“dual writes”，即应用程序内部做这样的处理 —— 将变更的数据同时并行写入不同的数据库中。如下图所示:

![](http://cdn2.hubspot.net/hub/540072/file-3062873213-png/blog-files/slide-37-4-3.png)

"dual writes"的[做法是有隐患的](http://martin.kleppmann.com/2014/10/28/staying-agile-at-span.html)，会受到[竞态条件](http://baike.baidu.com/view/6952316.htm)(race conditions)和可靠性问题的影响，造成不同的数据库之间的差异越来越大，最终变得难以修复。

从数据库快照中重建缓存或索引有利于消除从dump重建的不一致性。但是，在一个大型数据库这个过程将是十分缓慢低效的，需要找到一种方案使其更快。

如果能模仿数据库主从之间复制的过程，将数据库的修改变成流，携带每一次修改的消息，并且完全按照写入顺序重放这些修改，那么将得到一份一模一样的数据库，并且这种做法比dual writes效率好，可靠性更高。

一个更加理想的工作流程如下图所示:

![](http://cdn2.hubspot.net/hub/540072/file-3062873223-png/blog-files/slide-42b-4-3.png)

Bottled Water就是按照上述工作流程设计的工具，将postgresql的变更存入kafka中。

## Bottled Water工作原理
Bottled Water就是模拟数据库主从复制的过程，将数据库的变化写入kafka中。

其中，这个过程使用了postgresql 9.4版本引入的新特性[logical decoding](https://www.postgresql.org/docs/9.5/static/logicaldecoding.html)，可以从一个数据库中提取出consistent snapshot和连续的change events流，提取出的数据按照行级别推送至kafka中。默认情况下，会在kafka中对每个表创建一个同名topic，以存储change events。

> 有关逻辑解码(logical decoding)的概念和过程描述，可以参考postgresql的文档: http://www.postgres.cn/docs/9.4/logicaldecoding-explanation.html

Bottled Water项目在使用中包含了两个部分:
- postgresql的扩展bottledwater。
- 一个可执行程序`bottledwater`，负责从postgresql中将变更内容推送至kafka中。

`bottledwater`这个可执行程序在启动时会创建一个名为`bottledwater`的复制槽(通过命令行参数可修改复制槽的名字，复制槽如果已存在就不会新建)，以存储数据库的更改，并且追踪已消耗的`diff`。由于复制槽会保留最后被消耗掉的`diff`位置，因此，即使bottledwater正常或异常退出，也不会丢失数据库的`diff`追踪，下次启动会接着上次的break point继续执行。由于复制槽本身不保留消费者的状态，每个消费者只能得到最后一个消费者停止消费之后的修改，所以特别注意不要启动多个bottledwater连接同一个复制槽。

## Bottled Water使用方法

### 安装
按照[官方文档](https://github.com/confluentinc/bottledwater-pg/blob/master/README.md#quickstart)的说明，有三种方法可以上手使用: docker、源码编译、ubuntu ppa。本文将使用第三种方案，使用的环境为Ubuntu 14.04，postgresql 9.4讲解。

添加Bottled Water的ppa源，然后安装两个软件包即可:

```bash
$ sudo apt-add-repository -y ppa:stub/bottledwater
$ sudo apt-get update
$ sudo apt-get install postgresql-9.4-bottledwater bottledwater
```

使用上述命令安装`postgresql-9.4-bottledwater`扩展和`bottledwater`可执行程序。

### 使用前配置
Bottled Water会连接到postgresql获取相关数据，连接的账户需要有`replication`权限，pg中数据库的变化存储在`WAL`中(类似于mysql的`binlog`)，至少需要replication权限才能读取WAL。

需要修改WAL的日志级别为`logical`。本例中，我们将创建一个`repl`账户，具有`REPLICATION`权限。

打开`/etc/postgresql/9.4/main/postgresql.conf`文件，修改以下属性:

```
wal_level = logical             # 默认WAL级别minimal，不符合要求，需要调整为logical
max_wal_senders = 8
wal_keep_segments = 4
max_replication_slots = 4
```

打开`/etc/postgresql/9.4/main/pg_hba.conf`，增加以下内容:

```
local   replication     repl                 trust
host    replication     repl  127.0.0.1/32   trust
host    replication     repl  ::1/128        trust
```

重启`postgresql`服务，让配置生效:

    sudo service postgresql restart

以管理员身份(postgres)登录psql，进行创建角色操作:

```bash
$ sudo -u postgres psql
psql (9.4.8)
Type "help" for help.

postgres=#
```

```sql
CREATE ROLE repl WITH REPLICATION PASSWORD 'password' LOGIN;
```

至此使用前的配置已全部完成。

### Bottled Water使用演示
我们创建一个数据库`mybw`，包含一个表`test`，让Bottled Water追踪`mybw.test`这张表的变化。

```sql
CREATE DATABASE mybw;
\c mybw
CREATE EXTENSION bottledwater; -- 创建bottledwater扩展
CREATE TABLE test (id serial primary key, value text);
```

在另一个终端，启动`bottledwater`可执行程序:

    $ bottledwater -d postgres://repl:password@127.0.0.1/mybw -b 172.16.250.10:9092 -f json

解释一下上述命令行:

- `-d`参数指定postgresql的URL，这里使用`repl`账户，密码`password`连接至`mybw`数据库
- `-b`参数指定kafka broker，以逗号分割多个broker
- `-f`参数指定输出格式为`json`，默认格式为`avro`。为了演示简单这里选择了`json`

执行成功后，终端输出如下，应该无任何报错:

```
Writing messages to Kafka in JSON format
Created replication slot "bottledwater", capturing consistent snapshot "0049D495-1".
INFO:  bottledwater_export: Table public.test is keyed by index test_pkey
Snapshot complete, streaming changes from 2/44DAB970.
```

现在向`mybw.test`这张表插入一条记录:

```sql
INSERT INTO test (value) values('hello world!');
```

此时从kafka中可以看到如下的消息:

```bash
$ kafka-console-consumer --zookeeper localhost --topic test --from-beginning --property print.key=true
{"id": {"int": 1}}	{"id": {"int": 1}, "value": {"string": "hello world!"}}
```

继续执行以下会修改数据库的sql:

```sql
INSERT INTO test (value) values('hello');
UPDATE test SET value = 'world' WHERE id = 2;
DELETE FROM test WHERE id = 2;
```

那么可以观察到kafka中多了以下几条消息:

```
{"id": {"int": 2}}	{"id": {"int": 2}, "value": {"string": "hello"}}
{"id": {"int": 2}}	{"id": {"int": 2}, "value": {"string": "world"}}
{"id": {"int": 2}}	null
```

官方文档中，对kafka中消息格式的描述如下:

- Bottled Water为每个数据库表在kafka中创建一个同名的topic(可以通过`-p`参数设置topic前缀)
- 每条消息使用表的主键作为kafka message的key，整行记录作为message value
- insert和update操作，message value将会是行的新值
- delete操作，message value将会是null
- 如果表没有主键，Bottled Water将不会处理此表。可以通过`--allow-unkeyed`参数强行使用

最后，只需要使用一个kafka consumer连接kafka就可以实时获取到数据库表的变更了。

## 总结
Bottled Water总体来说使用还是很方便的，并且响应速度十分灵敏。即使停掉bottledwater进程，下次启动时也会继续上次的断点，不会丢失数据。需要使用postgresql与kafka结合方案的可以考虑。

## 参考资料
- [Bottled Water: Real-time integration of PostgreSQL and Kafka](http://www.confluent.io/blog/bottled-water-real-time-integration-of-postgresql-and-kafka/)
- [bottledwater-pg README](https://github.com/confluentinc/bottledwater-pg/blob/master/README.md)
- [Streaming Replication - PostgreSQL wiki](https://wiki.postgresql.org/wiki/Streaming_Replication)
- [postgresql文档: 章 46. 逻辑解码](http://www.postgres.cn/docs/9.4/logicaldecoding.html)
