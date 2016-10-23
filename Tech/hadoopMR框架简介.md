# Hadoop MapReduce框架简介

Hadoop是基于Google的若干大数据处理的论文(如GFS，MapReduce等)，实现的一套开源的大数据处理框架。本文主要探讨`MapReduce`这个组件。

## MapReduce的基本原理
`MapReduce`是Hadoop中的一个编程框架，用于将大数据进行`Mapping`和`Reducing`操作，汇总计算出最终想要的结果。整个设计思想的逻辑图如下:

![](http://images.cnitblog.com/blog/617081/201412/031211194206880.jpg)

MapReduce将一个大数据的计算任务拆分成若干个`互不影响`的子任务，进行`并行计算`，这个过程称为`Mapping`。

然后将每个子任务计算得到的结果进行收集汇总，最终得出需要的结果，这个过程称为`Reducing`。

整个MapReduce的过程可以用类似于Linux管道流式处理的过程描述:

![](https://louishust.gitbooks.io/study-hadoop-definitive-guide/content/images/mr_logical_data_flow.png)

为了适应大数据处理，在Hadoop的MapReduce框架中，`input`通常来自于`HDFS`上的文件(包括`HBase`的`Scans`)，`map`和`reduce`这两个函数通常需要开发者自己根据逻辑编写，`map`用于处理每个被分拆的子任务。`shuffle`的过程可以见[这篇文章](http://langyu.iteye.com/blog/992916)的描述，简单来说这个是一个神奇的地方，是`map`函数将结果汇总输出到`reduce`函数的中间处理环节，将相同的key进行合并，value合并为一个List。`shuffle`通常不需要自己编写，内部默认处理算法就能处理的很好，当然也可以手工编写`shuffle`算法。`reduce`接受`map`的输出，通常为遍历每个key的value list，进行排序/求和/筛选等运算操作。`output`为输出结果，通常表现形式为以文本文件形式将`reduce`计算结果存储于`HDFS`上。

> 自Hadoop 2.0版本起，引入了一个新的MapReduce框架，称为**Yarn**。

## MapReduce用例
`MapReduce`能够解决的问题有一个共同特点: 任务可以被分解为多个子问题，且这些子问题相对独立，彼此之间不会有牵制，待并行处理完这些子问题后，任务便被解决。包括分布式grep、URL访问频率统计、Web连接图反转、倒排索引构建、分布式排序等。

除此之外的某些任务，无法用MapReduce解决或难以解决。如Fibonacci数值计算(下一个结果依赖于前一个计算结果)，层次聚类法等。

## MapReduce代码编写
对于简单的MapReduce任务来说，只需要编写`map`和`reduce`两个函数即可。以下内容来源于官方示例[`WordCount`](https://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html#Example:_WordCount_v1.0):

```java
import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {

  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }

  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```

### Mapper类解读
我们由上至下依次解读这个这个程序。首先是`TokenizerMapper`这个class，继承自`org.apache.hadoop.mapreduce.Mapper<KEYIN,VALUEIN,KEYOUT,VALUEOUT>`这个泛型类。需要对`Mapper`类的处理过程做以说明。Mapper接受到的输入是来自于`InputFormat`这个类的，这个类会将输入拆分成一个个的kv对，比如下面的文本文件:
```
This is the first line.
This is the second line.
```

经过`FileInputFormat`处理过后，会变成一个个kv对，按行拆分，key值对应行号:
```
(1, "This is the first line.")
(2, "This is the second line.")
```

`Mapper`类接收到的就是这样的KV对，输出给`Reducer`类的也是KV对。`Mapper<KEYIN, VALUEIN, KEYOUT, VALUEOUT>`就要与这样的kv对对应。

特别注意这里的kv对有些特殊，并不是常见的`String`, `Integer`这些kv值。由于MapReduce服务于大数据，是分布式计算架构，因此`map`和`reduce`可能工作于不同的节点上，为了减少节点间的数据传输量，就要求输出的kv能够**序列化**，以紧缩数据，减少传输带宽，所以要求kv都必须实现[`Writable`](https://hadoop.apache.org/docs/r2.6.1/api/org/apache/hadoop/io/Writable.html)这个接口。同时，对于key通常还有排序的需求，所以key还必须实现[`WritableComparable`](https://hadoop.apache.org/docs/r2.6.1/api/org/apache/hadoop/io/WritableComparable.html)接口。常见的数据类型已经都封装成对应的类，如:
- int: IntWritable
- String: Text
- double: DoubleWritable

如果有自定义的kv类型，需要注意必须实现这个接口。

通常情况下`Mapper`类下面需要自己覆写的方法只有一个`map()`，参数有3个，分别对应`KEYIN`, `VALUEIN`, `Context`。这个`Context`其实就是输出的上下文对象，调用`Context.write(KEY, VALUE)`方法就会输出kv对到下一个环节，即`shuffle`。

按照上述文档作为范例，在这里`map()`方法执行过后，会输出这样的kv对:
```
("This", 1)
("is", 1)
("the", 1)
("first", 1)
...
```

### shuffle
如无特殊需求，通常不用编写`shuffle`相关的代码。经过这个神奇的`shuffle`处理，会将相同的key值进行合并，value被合并成一个List，于是乎`Reducer`得到的输入将会像这样:
```
("This", [1, 1])
("is", [1, 1])
("the", [1, 1])
("first", [1])
...
```

### Reducer类解读
同样继承了`org.apache.hadoop.mapreduce.Reducer<KEYIN,VALUEIN,KEYOUT,VALUEOUT>`这个泛型类。`KEYIN`, `VALUEIN`通常情况下要与`Mapper`类的`KEYOUT`,`VALUEOUT`相同。

需要覆写的函数通常也只有一个`reduce()`，参数和`map()`一样也有3个，前两个参数和`Mapper`类的`KEYOUT, VALUEOUT`相对应，第三个参数`Context`同样作用一样，向输出流写入kv对的，最终用于输出。这个`reduce()`函数的代码也相当好理解，就是把输入过来的list value做个叠加，就得到了对应的单词的统计次数。于是经过`reduce()`函数之后，kv对变成了下面这样:
```
("This", 2)
("is", 2)
("the", 2)
("first", 1)
```

### main函数解读
在`main()`函数中初始化了一个`MapReduce` job。`setJarByClass()`设置`main()`函数所在的class，`setMapperClass`，`setReducerClass`, `setOutputKeyClass`, `setOutputValueClass`分别设置了`Mapper`类, `Reducer`类, `KEYOUT`类, `VALUEOUT`类，最后两个静态方法`FileInputFormat.addInputPath()`和`FileOutputFormat.setOutputPath()`为这个job设置输入和输出路径。

最后调用`job.waitForCompletion(true)`这个函数进行执行job，这个函数会等待job执行完成之后退出，并返回`true/false`表明是否job执行成功，执行过程中还会打印整个map/reduce的进度。`Job`类还有一个`submit()`方法，这个方法没有返回值，法执行的job也不会等待整个job执行完毕才退出，而是成功递交到集群之后就立刻返回了，通常也不会打印执行的进度。

经过`FileOutputFormat`这个类后，会直接把kv对按行打印到文件中，存储于`HDFS`上。文件中看起来就像这样:
```
This 2
is 2
the 2
first 1
...
```

## MapReduce任务执行
上述代码可以直接在hadoop集群上编译成jar包:
```bash
$ bin/hadoop com.sun.tools.javac.Main WordCount.java
$ jar cf wc.jar WordCount*.class
```

也可以在本地使用`gradle`, `maven`等构建工具直接构建成jar包后推送到hadoop集群上。

调用`hadoop jar`命令执行这个任务:
```bash
$ bin/hadoop fs -ls /user/joe/wordcount/input/ /user/joe/wordcount/input/file01 /user/joe/wordcount/input/file02

$ bin/hadoop fs -cat /user/joe/wordcount/input/file01
Hello World Bye World

$ bin/hadoop fs -cat /user/joe/wordcount/input/file02
Hello Hadoop Goodbye Hadoop

$ bin/hadoop jar wc.jar WordCount /user/joe/wordcount/input /user/joe/wordcount/output
```

`hadoop jar`的命令行参数如下:

    hadoop jar jarFile [mainClass] args...

## 几个常见的优化点
1. 启用map的输出压缩
2. 控制map过程处理的数据量，不宜太小，也不宜太大
3. 合理使用combiner(相当于map级别的reduce)

## 小结
Hadoop MapReduce框架总体上来说使用比较简单，但需要理解这个处理过程，明白整个数据流经过了怎样的转换过程。

另外就是，使用这套框架之前一定要先想清楚自己的任务怎么才能拆分成互不影响的map过程，以及reduce过程如何汇总这些map产生的数据。
