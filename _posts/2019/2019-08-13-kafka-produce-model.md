---
layout: article
title:	kafka生产机制
date:	2019-08-13 10:16:00
categories:
    - article
tags:
    - Java
    - Kafka
---

### 生产模型

![image](https://user-images.githubusercontent.com/29170657/62910708-29873300-bdb4-11e9-9b35-8fe46c4d8747.png)

Record Accumulator: 消息发送缓冲区，把消息打包成Batch。

>BatchSize 控制打包大小
>
>Linger.ms 控制发送等待时延

Send Thread：从发送缓冲区取数据，封装成Request，发送到Broker。

> 如果Batch都是发往同一个节点，就会封装在同一个Request里，减少请求server端的请求数量，提升效率。

### 消息生产流程

![image](https://user-images.githubusercontent.com/29170657/62911089-964efd00-bdb5-11e9-8ff1-96631e40c77b.png)

#### 主线程

KafkaProducer：消息生产者

Interrceptors：过滤器，发送之前对消息进行加工处理

waitOnMetadata：元数据等待更新，元数据包含：Broker：节点ip和端口消息，Topic：分区，副本。

Key/Value Serializer：对数据进行序列化，判断消息校验大小。

partitioner：计算当前消息发往哪个分区，两种（指点partitioner，指定key的hashcode，或者轮询分区）

RecordAccumulator：把消息放在发送缓冲区里。

![image](https://user-images.githubusercontent.com/29170657/62911603-89cba400-bdb7-11e9-8c3a-7d0f8f968ef2.png)

#### Sender线程

drain：取消息（满足发送条件的消息）

sendProduceRequest：封装成Request

NetworkClient，KSelector：发送消息到kafka server端


![image](https://user-images.githubusercontent.com/29170657/62911623-99e38380-bdb7-11e9-83c3-4026a709e52a.png)

### Batch

1.batch.size默认16384，消息batch大小，当batch达到batch.size大小时，唤醒sender线程发送消息。批量发送消息，减少request数量，提升发送效率，减轻服务端压力

2.ling.ms默认0，sender线程检查batch是否ready，满足batch.size和ling.ms其中一个，即发送消息

3.buffer.memory默认33554432，producer可以用来缓存数据的内存大小。如果数据产生速度大于向broker发送的速度，producer会阻塞或者抛出异常，通过参数“block.on.buffer.full”控制

### 参考资料

> [1] [https://education.huaweicloud.com:8443/courses/course-v1:HuaweiX+CBUCNXP017+Self-paced/courseware/993bf22863da445b95ac46fc141717a6/bae0b75672e041e49f4bbb36d08a179c/](https://education.huaweicloud.com:8443/courses/course-v1:HuaweiX+CBUCNXP017+Self-paced/courseware/993bf22863da445b95ac46fc141717a6/bae0b75672e041e49f4bbb36d08a179c/)