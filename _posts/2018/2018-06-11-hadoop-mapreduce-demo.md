---
layout: article
title:	Hadoop MapReduce Demo
date:	2018-06-11 14:01:33
categories:
    - article
tags:
    - hadoop
    - bigdata
---


本章节开始介绍hadoop三大组件之**MapReduce**。
MapReduce是一种分布式计算模型，开始是由Google提出，主要用于搜索领域，解决海量数据的计算问题。简而言之，就是把一个很庞大的任务，分解成很多个小任务，分发集群里不同机器执行，然后汇总成一个最终的结果。通这章节我们用MapReduce统计 **π** 后一百万位小数，0到9出现的频率,通过这样的小demo，来了解MapReduce，(～￣▽￣)～ 这样好像有点无聊。

##### 1.准备

准备一份 π 后一百万位小数的文件，，通过上一章节学习的`hdfs dfs`命令，将[pi.txt](./pi.txt)文件放入HDFS的`/test`目录下。

~~~shell
[root@hadoop-master ~]# hdfs dfs -put pi.txt /test/pi.test
[root@hadoop-master ~]# hdfs dfs -ls /test/
Found 1 items
-rw-r--r--   2 root supergroup    1048578 2018-08-08 13:55 /test/pi.test
~~~

##### 2.分析MapReduce中数据流向图

MapReduce分为6个阶段：Input、Split、Map、Shuffle、Reduce、output。

- Input（输入）：读取统计文件。
- Split（任务拆分）：将文件按每一行拆分成KV键值对的形式，K为偏移量（即每行第一个字符的所在位置，比如，第一行的字符串长度为3，那么第二行的KEY值为4，具体可参考`org.apache.hadoop.mapreduce.lib.input.FileInputFormat<K, V>#getSplits`实现方式），V为文件每行的内容。splits的个数即为map tasks的个数。重写`org.apache.hadoop.mapreduce.InputFormat<K, V>#getSplits`方法可自定义任务拆分力度。
- Map：将拆分之后的内容转换成新的KV键值对。
- Shuffle（派发）：将Key值相同的扔到一个Reduce。
- Reduce ：将Map阶段处理的结果整合到一起。
- Output：输出最终的结果。

###### π 0-9统计数据流向图
![./img/3.jpg](./img/3.jpg)

##### 3. 程序实现

[源码](./mapreduce)

添加依赖包

~~~xml
...
<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>2.5.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-mapreduce-client-core</artifactId>
        <version>2.5.1</version>
    </dependency>
</dependencies>
...
~~~

工程目录

![./img/4.jpg](./img/4.jpg)

App.java

~~~java

public class App extends Configured implements Tool {
    private static final Logger LOGGER = Logger.getLogger(App.class);

    @Override
    public int run(String[] strings) throws Exception {
        Configuration conf = getConf();

        conf.set("fs.defaultFS", "hdfs://hadoop-master:9000");

        Job job = Job.getInstance(conf, "MapReduce_Dome");

        job.setJarByClass(App.class);

        job.setMapperClass(MyMapper.class);
        job.setReducerClass(MyReduce.class);

        // 指定Mapper的输出类型,对应Reduce的输入类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        // 指定reducetask的输出类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        //读取要统计的文件，可以为多个路径
        FileInputFormat.setInputPaths(job, new Path(strings[0]));
        //统计结果保存
        FileOutputFormat.setOutputPath(job, new Path(strings[1]));
        //执行MapReduce
        job.waitForCompletion(true);

        return 0;
    }

    /**
     * @param args args[0] 统计文件的路径，args[1]统计结果输出路径
     * @throws Exception 执行异常
     */
    public static void main(String[] args) throws Exception {
        if (args.length < 2){
            LOGGER.error("请指定统计文件和输出路径");
            System.exit(0);
        }
        LOGGER.info(Arrays.toString(args));
        //MapReduce执行入口
        ToolRunner.run(new Configuration(), new App(), args);
    }
}

~~~

MyMapper.java

mapper实现，必须继承`Mapper<KEYIN, VALUEIN, KEYOUT, VALUEOUT>`,且重写其map方法

 * 第1个泛型为： 读取文件的key类型，
 * 第2个泛型为： 读取文件某一行值的类型
 * 第3个泛型为： mapper 输出结果分组类型，对应Reduce的输入key类型
 * 第4个泛型为： mapper 输出结果值类型，对应Reduce的输入value类型

map阶段将出现的字符作为一个key值,value为1

~~~java
public class MyMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    private static final Logger LOGGER = Logger.getLogger(MyMapper.class);
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        LOGGER.info("mapper key:" + key);
        LOGGER.info("mapper value:" + value);
        for (char c : value.toString().toCharArray()) {
            context.write(new Text(String.valueOf(c)), new IntWritable(1));
        }
    }
}
~~~

MyReduce.java

Reducer实现，必须继承`Reducer<KEYIN, VALUEIN, KEYOUT, VALUEOUT>`,且重写其reduce方法

 * 第1个泛型为： mapper 输出key类型
 * 第2个泛型为： mapper 输出value类型
 * 第3个泛型为： Reducer 输出key类型
 * 第4个泛型为： Reducer 输出结果值类型

Reducer阶段计算同一分组集合的大小，即为字符出现的次数。

~~~java
public class MyReduce extends Reducer<Text, IntWritable, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        context.write(key, new IntWritable(IteratorUtils.toList(values.iterator()).size()));
    }
}
~~~



