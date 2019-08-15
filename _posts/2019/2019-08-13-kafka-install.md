---
layout: article
title:	Kafka安装
date:	2019-08-13 08:26:02
categories:
    - article
tags:
    - Java
    - Kafka
---

### 安装地址

~~~shell
wget https://archive.apache.org/dist/kafka/2.3.0/kafka_2.12-2.3.0.tgz
tar -zxvf kafka_2.12-2.3.0.tgz
cd kafka_2.12-2.3.0
~~~

### 修改配置文件 `server.properties`

该`broker.id`属性是群集中每个节点的唯一且永久的名称

~~~
#config/server-1.properties:
broker.id=1
listeners=PLAINTEXT://localhost:9093
log.dirs=/tmp/kafka-logs-1

#config/server-2.properties:
broker.id=2
listeners=PLAINTEXT://localhost:9094
log.dirs=/tmp/kafka-logs-2
~~~


### 启动zookeeper

~~~shell
bin/zookeeper-server-start.sh config/zookeeper.properties
~~~

### 启动kafka

~~~shell
bin/kafka-server-start.sh config/server.properties
~~~

启动错误情况

~~~shell
[root@iZwz9j80yesn9eqdugq563Z kafka_2.12-2.3.0]# bin/kafka-server-start.sh config/server.properties
OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
OpenJDK 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000c0000000, 1073741824, 0) failed; error='Cannot allocate memory' (errno=12)
#
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (mmap) failed to map 1073741824 bytes for committing reserved memory.
# An error report file with more information is saved as:
# /opt/kafka/kafka_2.12-2.3.0/hs_err_pid12440.log
~~~

修改kafka JVM内存配置

> export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M"

### 创建一个只包含一个分区，只有一个副本的Topic

~~~shell
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
~~~

### 查看Topic列表

~~~shell
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
~~~

### 测试Kafka

生产者发送消息

~~~shell
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
~~~

消费者读取消息

~~~
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
~~~



### 参考文档
> [1] [http://kafka.apache.org/documentation/](http://kafka.apache.org/documentation/)