# DNS基础
**域名系统**（英文：**D**omain **N**ame **S**ystem，缩写：**DNS**）是因特网的一项服务。它作为将域名和IP地址相互映射的一个分布式数据库，能够使人更方便地访问互联网。DNS使用TCP和UDP端口53。当前，对于每一级域名长度的限制是63个字符，域名总长度则不能超过253个字符。

在现代化的大规模基础设施中，DNS几乎已经成了一种标配。

## DNS发展历史
- 早期的IP网络通信时：当时网络中的主机还不是很多，人们事先在网络中的每一台主机上，构造一张主机的IP地址表：然后将网络中的所有主机的IP地址写到这个地址表中，通信时，先去查询这个IP地址表，就可以得到对方主机的IP地址，最后再和对方实现IP通信(这个表直到今天依旧存在，想想看是什么？)
- 到了20世纪80年代，网络中互联的主机数，不断的增加，人们发现，再采用构造IP地址表的办法已经行不通了，因为：
    - 第一，这个IP地址表太大了，主机没有过多的存储空间了
    - 第二，在查询这个IP地址表时花费了太多的时间
- 1983年保罗·莫卡派乔斯（Paul Mockapetris）发明了DNS；原始的技术规范在882号因特网标准草案（RFC 882）中发布。1987年发布的第1034和1035号草案修正了DNS技术规范，并废除了之前的第882和883号草案。在此之后对因特网标准草案的修改基本上没有涉及到DNS技术规范部分的改动。

DNS主要提供的功能就是域名和IP的双向解析(尽管域名解析为IP用的更多)

## DNS域名的结构
DNS的域名是层级结构，如下图所示:

![](https://upload.wikimedia.org/wikipedia/commons/b/b1/Domain_name_space.svg)

整个DNS域名从右至左，以`.`分割形成层级关系。所有域名都是根域`.`下的子域。现代浏览器中早已兼容忽略根域的写法，浏览器会自动补上根域再请求(如`www.taobao.com`，浏览器内部会补全成`www.taobao.com.`)。

如`www.baidu.com`，层级关系是这样的
```
. ----------- 根域
` com ------- 顶级域名
   ` baidu -- 一级域名
      ` www - 二级域名
```

顶级域名通常的含义列表如下:

顶级域名 | 分配给
--------|-------
com     | 商业组织
edu     | 教育机构
gov     | 政府部门
mil     | 军事部门
net     | 主要网络支持中心
org     | 其他组织
int     | 国际组织
地理模式 | 各个国家

DNS通过`递归`(最常见)与`迭代`两种算法将域名解析为对应的IP。

> 关于递归与迭代算法的描述: http://blog.csdn.net/lycb_gz/article/details/11720247

## DNS的记录类型
DNS常见的记录类型如下:

代码    |   描述                 |              功能
-------|------------------------|-----------------------------
 A     | IP 地址记录             | 传回一个 32 比特的 IPv4 地址，最常用于映射主机名称到 IP地址，但也用于DNSBL（RFC 1101）等。
 AAAA  | IPV6地址记录            | 传回一个 128 比特的 IPv6 地址，最常用于映射主机名称到 IP 地址。
 CNAME | 规范名称记录(常被称为别名) | 一个主机名字的别名：域名系统将会继续尝试查找新的名字。
 MX    | 电邮交互记录             | 引导域名到该域名的邮件传输代理（MTA, Message Transfer Agents）列表。
 NS    | 名称服务器记录           | 委托DNS区域（DNS zone）使用已提供的权威域名服务器。
 PTR   | 指针记录                | 引导至一个规范名称（Canonical Name）。与 CNAME 记录不同，DNS“不会”进行进程，只会传回名称。最常用来运行反向 DNS 查找，其他用途包括引作 DNS-SD。
 SRV   | 服务定位器              | 广义为服务定位记录，被新式协议使用而避免产生特定协议的记录，例如：MX 记录。

* `A`记录是最常见的解析记录类型，直接存储着域名->IP的映射关系。注意A记录可以同时存储多条IP记录，如果存储多条记录，则轮询。
* `CNAME`优先级比`A`低。`CNAME`存储的不是IP记录，而是另一个域名。因此，CNAME需要经过多次解析才能得到真正的目标IP地址。
* `SRV`记录定义于RFC 2782标准。用于定义提供特定服务的服务器的位置，如主机（hostname），端口（port number）等。
* 泛解析: 将`*.domain.com`的`A记录`解析到某个IP上。则`domain.com`下的二级域名都将解析到这个IP上。

## DNS查询演示
使用命令行工具`dig`做演示，其他dns lookup工具(如: nslookup, host等)同样可以查到类似的结果。

比如查看`www.baidu.com`的解析记录，使用`dig www.baidu.com`即可。dig默认请求的是`A`记录解析，如果想看到域名的实际解析记录，可以使用`dig www.baidu.com any`。已经省略无关紧要的输出:

```bash
$ dig www.baidu.com
;; QUESTION SECTION:
;www.baidu.com.         IN  A

;; ANSWER SECTION:
www.baidu.com.      262 IN  A   180.97.33.108
www.baidu.com.      262 IN  A   180.97.33.107
```

可以看到`www.baidu.com.`这个域名返回的是一个`A记录`，有两个解析结果。再次运行同样的命令，会发现被轮询了，返回的A记录的IP顺序变了。

```bash
;; QUESTION SECTION:
;www.baidu.com.         IN  A

;; ANSWER SECTION:
www.baidu.com.      65  IN  A   180.97.33.107
www.baidu.com.      65  IN  A   180.97.33.108
```

反复运行可以发现，每次返回的IP都是轮询顺序的。

如果要查询`CNAME`解析，用`dig`的话需要跟上解析记录参数，`any`代表任何类型，否则就是`a`。

```bash
$ dig img.alicdn.com any
;; QUESTION SECTION:
;img.alicdn.com.            IN  ANY

;; ANSWER SECTION:
img.alicdn.com.     15752   IN  CNAME   img.alicdn.com.danuoyi.alicdn.com.
```

用`host`命令可以看到访问这个域名的详细请求过程:

```bash
$ host img.alicdn.com
img.alicdn.com has address 180.97.245.110
img.alicdn.com has address 180.97.159.242
img.alicdn.com has address 180.97.245.109
img.alicdn.com has address 180.97.159.243
img.alicdn.com has address 58.220.27.86
img.alicdn.com has address 58.218.215.120
img.alicdn.com has address 58.218.215.110
img.alicdn.com has address 58.220.27.85
img.alicdn.com is an alias for img.alicdn.com.danuoyi.alicdn.com.
img.alicdn.com is an alias for img.alicdn.com.danuoyi.alicdn.com.
```

```bash
$ dig 163.com any
;; QUESTION SECTION:
;163.com.           IN  ANY

;; ANSWER SECTION:
163.com.        163 IN  A   123.58.180.8
163.com.        163 IN  A   123.58.180.7
163.com.        163 IN  TXT "v=spf1 include:spf.163.com -all"
163.com.        163 IN  MX  10 163mx02.mxmail.netease.com.
163.com.        163 IN  MX  10 163mx03.mxmail.netease.com.
163.com.        163 IN  MX  50 163mx00.mxmail.netease.com.
163.com.        163 IN  MX  10 163mx01.mxmail.netease.com.
163.com.        163 IN  NS  ns8.nease.net.
163.com.        163 IN  NS  ns2.nease.net.
163.com.        163 IN  NS  ns1.nease.net.
163.com.        163 IN  NS  ns4.nease.net.
163.com.        163 IN  NS  ns3.nease.net.
163.com.        163 IN  NS  ns5.nease.net.
163.com.        163 IN  NS  ns6.nease.net.
```

```bash
$ host 163.com
163.com has address 123.58.180.8
163.com has address 123.58.180.7
163.com mail is handled by 10 163mx01.mxmail.netease.com.
163.com mail is handled by 50 163mx00.mxmail.netease.com.
163.com mail is handled by 10 163mx02.mxmail.netease.com.
163.com mail is handled by 10 163mx03.mxmail.netease.com.
```

只要要有`NS`记录才能使用域名邮箱(`@163.com`这样的邮箱)

暴露在互联网的域名解析服务中，`SRV`记录较为少见，因为这样会暴露服务的端口，带来安全隐患。但`SRV`记录在微服务架构中极为便利。

```bash
$ dig @172.16.250.10 -p 8600 consul.service.consul SRV
;consul.service.consul.     IN  SRV

;; ANSWER SECTION:
consul.service.consul.  0   IN  SRV 1 1 8300 rtdstest.node.sinoiot-test-dc1.consul.
consul.service.consul.  0   IN  SRV 1 1 8300 rtdstest3.node.sinoiot-test-dc1.consul.
consul.service.consul.  0   IN  SRV 1 1 8300 rtdstest2.node.sinoiot-test-dc1.consul.

;; ADDITIONAL SECTION:
rtdstest.node.sinoiot-test-dc1.consul. 0 IN A   172.16.250.10
rtdstest3.node.sinoiot-test-dc1.consul. 0 IN A  172.16.250.14
rtdstest2.node.sinoiot-test-dc1.consul. 0 IN A  172.16.250.13
```

可以看到`SRV`记录不但会返回像`A记录`那样的IP地址，还会返回其他记录中均不会出现的端口号。可以说，通过`SRV`解析记录，可以得知哪些服务器上提供哪些服务，这些服务暴露的端口号是什么。大大减轻了维护成本。
