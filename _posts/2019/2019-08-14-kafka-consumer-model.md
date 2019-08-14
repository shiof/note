---
layout: article
title:	kafka消费机制
date:	2019-08-14 10:07:34
categories:
    - article
tags:
    - Java
    - Kafka
---

### 消费者基本概念

1.Consumer：kafka消费者负责拉取消息和确认消息，分new consumer和old consumer

2.Group：每个消费者都属于一个消费组内，通过消费组概念可以实现Topic消息的广播（发给所有消费组）或者单播（组内消息均衡分担）。

3.Rebalance：组内的消费者以Topic分区个数进行均衡分配，所以组内消费者最多只能有分区个数的消费者。

4.assign模式：手工分配消费分区

5.Subscribe模式：自动分配消费分区

### 消费模型

![image](https://user-images.githubusercontent.com/29170657/62990340-2bb5c400-be7e-11e9-97ed-c173b14e3d13.png)

### 消费进度管理机制

![image](https://user-images.githubusercontent.com/29170657/62990361-4b4cec80-be7e-11e9-91f6-7247819e3f28.png)

### 消费机制

1.每个消费组里面的消费者都需要先查找一个协调者

2.消费者加入到这个组内，主要目的是为了对分区进行分配

3.分配分区完毕后，再查找各自分区的Leader，进行消费

![image](https://user-images.githubusercontent.com/29170657/62990418-8c450100-be7e-11e9-8bb6-dd94259a5938.png)

![image](https://user-images.githubusercontent.com/29170657/62990413-7f281200-be7e-11e9-83ea-594fec8ed50a.png)

### GroupCoordinator：职责

1.处理JoinGroupRequest、SyncGroupRequest完成partition分配

2.维护_consumer_offset，管理消费进度

3.通过心跳检查消费者状态

### Group Rebalance：触发条件

1.新consumer加入group

2.有consumer退出：主动leave、宕机、网络故障等

3.Topic分区数变化

4.Consumer调用unsubscrible


### Group Rebalance：rebalance流程

1.Consumer 发送JoinGroupReuest请求，服务端Group进入PreparingRebalance状态

2.Consumer 发送SyncGroupRequest请求，服务端Group进入Stable状态

	* 如果是group leader，计算分配分区，通过SyncGroupRequest发送至Coordinator
	* 非group leader在SyncGroupResponse中获取分配的分区
	
![image](https://user-images.githubusercontent.com/29170657/62990607-29079e80-be7f-11e9-9a3e-e40ae3ceb0a4.png)

### Group Coordinator计算

1.计算__consumers_offsetstopic分区，计算公式：

~~~text
__consumers_offsetspartition# = Math.abs(groupId.hashCode() % groupMetadataTopicPartitionCount) 

注意：groupMetadataTopicPartitionCount由offsets.topic.num.partitions指定，默认是50个分区
~~~

2.该分区leader所在的broker就是被选定的coordinator

### 参数调优

|参数|默认值|推荐值|说明|
|----|------|------|----|
|max.partition.fetch.bytes|1048576|默认值|每次 fetch请求中,针对每次 fetch消息的最大字节数。这些字节将会督导用于每个 partition的内存中,因此,此设置将会控制 consumer所使用的 memory大小。 这个 fetch请求尺寸必须至少和 server允许的最大消息尺寸相等,否则, producer可能发送的消息尺寸大于 consumer所能消耗的尺寸。|
|fetch.min.bytes|1|默认值|每次 fetch请求时, server应该返回的最小字节数。如果没有足够的数据返回, 请求会等待,直到足够的数据才会返回。|
|fetch.wait.max.ms|500|默认值|如果没有足够的数据能够满足 fetch.min.bytes,则此项配置是指在应答 fetch 请求之前, server会阻塞的最大时间。|


### 使用规范

1.consumer owner线程需确保不会异常退出，避免客户端没有发起消费请求，阻塞消费。

2.确保处理完消息后再做消息commit，避免业务消息处理失败，无法重新拉取处理失败的消息。

3.consumer 不能频繁加入和退出group，会导致consumer 频繁做rebalance，阻塞消费。

4.consumer 数量不能超过topic分区数，否则会有consumer拉取不到消息。

5.consumer 需周期poll，维持和server的心跳，避免心跳超时，导致consumer频繁加入和退出，阻塞消费。

6.consumer 拉取的消息本地缓存应有大小限制，避免OOM7.Kafka不能保证消费重复的消息，业务侧需保证消息处理的幂等性

8.消费线程退出要调用consumer的close方法，避免同一个组的其他消费者阻塞sesstion.timeout.ms的时间
