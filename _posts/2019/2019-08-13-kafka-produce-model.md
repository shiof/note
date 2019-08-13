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

### 参数调优

|参数|默认值|推荐值|说明|
|--|--|--|--|
|buffer.memory| 33554432|536870912|producer可以用来缓存数据的内存大小。如果数据产生速度大于向broker发送的速度，producer会阻塞或者抛出界常，以“block.on.buffer.full”来表明。这项设置将和producer能够使用的总内存相关，但并不是一个硬性的限制，因为不是producer使用的所有内存都是用于缓冲。一些额外的内存会用于压缩(如果引入压缩机制），同样还有一些用于维护请求。|
|linger.ms|0|0|producer组将会汇总任何在访求与发送之间到达的消息记录一个单独批量的请求。通常来说，这只有在记策产生速度大于发送速度的时候才能发生。然而，在某些条件下，客户端将希望降低请求的数量，其至降低到屮等负栽一下。这项设置将通过增加小的延迟来完成。即，立即发送一条记录，producer将会等待给定的延迟时间以允许其他消息记录发送，这些消息记录可以批最处理。这可以认为是TCP中Nagle的算法类似。这项设置设定了批量处理的更高的延迟边界：一但我们获得某个partition的batch.size，他将会立即发送而不顾这项设置，然而如果我们获得消息字节数比这项设置要小的多，我们需要“linger”特定的时间以获取更多的消息。这个设置默认为0,即没存延迟，设定linger.ms=5，例如，将会减少请求数目，但是同时会增加5ms的延迟。|
|receive.buffer.bytes|32768|默认值或者略大于带宽*时延|TCP receive缓存大小，当发送数据时使用|
|send.buffer.bytes|131072|默认值或者略大于带宽*时延|TCP send缓存大小，当发送数据时使用|
|acks|1|1|producer需要server接收到数据之后发出的确认接收的信号，此项配置就是指producer需耍多少个这样的确认信号。此配置实际上代表了数据备份的可用性。以下设置为常用选项：<br> (1) acks=0:设置为0表示producer不需要等待任何确认收到的信息。 副本将立即加到socket buffer并认为己经发送。没有任何保障可以保证此情况下server己经成功接收数据， 同时重试配置不会发生作用（因为客户端不知道足否失败）回馈的offset会总足设置为-1; <br> (2) acks=l：这意味着至少要等待leader己经成功将数据写入本地log， 但是并没有等待所有follower是否成功写入。这种情况下，如果follower 没有成功备份数据，而此时leader又挂掉，则消息会丢失。<br> (3) acks =all：这意味着leader需要等待所有备份都成功写入日志，这 种策略会保证只要有一个备份存活就不会丢失数据。这是最强的保证。 目前Broker端限制只支持这3个值。|
|batch.size|16384|262144|producer将试图批处理消息记录，以减少请求次数。这将改善client与server之间的性能。这项配置控制默认的批量处理消息字节数。不会试图处理大于这个字节数的消息字节数。<br>发送到brokers的请求将包含多个批量处理，其屮会包含对每个partition的一个请求。<br>较小的批量处理数值比较少用，并且可能降低吞吐量（0则会仅用批量处理）。较大的批量处理数值将会浪费更多内存空间，这样就需要分配特定批量处理数值的内存大小。||

### 参考资料

> [1] [https://education.huaweicloud.com:8443/courses/course-v1:HuaweiX+CBUCNXP017+Self-paced/courseware/993bf22863da445b95ac46fc141717a6/bae0b75672e041e49f4bbb36d08a179c/](https://education.huaweicloud.com:8443/courses/course-v1:HuaweiX+CBUCNXP017+Self-paced/courseware/993bf22863da445b95ac46fc141717a6/bae0b75672e041e49f4bbb36d08a179c/)