---
layout: article
title:	kafka入门介绍
date:	2019-08-12 21:44:56
categories:
    - article
tags:
    - Java
    - Kafka
---

### 为什么需要消息服务

解决分布式场景中交易通讯问题，提供高可靠，高性能，高拓展的消息管道.

基于消息中间件构建消息服务，解决云分布式场景的消息通信问题，提供高可靠、高性能（高并发、高吞吐）、高可扩展的消息管道。

1.系统解耦: 基于发布订阅模型，分布式应用异步解耦，可以增加应用的水平扩展性，增加前端应用快速客户反应能力

2.削峰填谷: 大促等流量洪流突然来袭时，消息服务可以缓冲突发流量，避免整个系崩溃。

3.数据交换: 解决跨DC传输，通道安全可靠、服务可用的问题。

4.异步通知: 海量终端接入、高吞吐、低时延实时并发消费。

5.日志通道: 做为重要日志的监控通信管道，将应用日志监控对系统性能影响降到最.

### Kafka 一种分布式、基于发布/订阅的消息中间件

1.高性能、高吞吐 顺序写盘、消息压缩•

2.在线扩容 分区机制，在线扩容副本在线扩容节点在线扩容•

3.高可靠：集群部署、多副本机制、多副本机制、支持在线扩容、支持分区在线迁移。

### 以kafka 为中心的解决方案

![image](https://user-images.githubusercontent.com/29170657/62908725-0d7f9380-bdac-11e9-8141-d80feaad4ca0.png)

### kafka基本概念

1.Broker: Kafka集群包含一个或多个服务实例，这种服务实例被称为broker（节点）。

2.Topic: 每条发布到kafka集群的消息有一个类型，这个类型被称为Topic。

3.Partition: Partition是物理上的概念，每个Topic包含一个或多个Partition。

> ![image](https://user-images.githubusercontent.com/29170657/62872857-e8116c00-bd50-11e9-9cc2-c5d0f03c3d30.png)
>
>每个分区消息只能通过追加消息方式增加消息，消息都有一个偏移量(offset)，顺序写入确保性能高效。
>
>消费者通过分区中offset定位消息和记录消费的位置

4.Producer: 负责发布消息到kafka Broker。

5.Consumer: 消息消费者，向kafka broker读取消息的客户端。

6.Consumer Group: 每个consumer属于一个特定的Consumer Group，没有就默认一个组。

![image](https://user-images.githubusercontent.com/29170657/62871684-85b76c00-bd4e-11e9-8c43-1e1e81669b85.png)

### Kafka消费者组与分区

1.每个消费者都属于一个消费组内，通过消费组概念可以实现Topic消息的广播（发给所有消费组）或者单播（组内消息均衡分担）。

2.消费者采用Pull模式进行消费，方便消费进度记录在客户端，服务端无状态。

3.组内的消费者以Topic分区个数进行均衡分配，所以组内消费者最多只能有分区个数的消费者

![image](https://user-images.githubusercontent.com/29170657/62873092-59511f00-bd51-11e9-8afa-8b0bc09b6010.png)

### Kafka分区副本

1.Kafka通过副本方式达到高可用的目标，每个分区可以有1个或者多个副本，分别分配在不同的节点上。

2.多个副本之间只有一个Leader，其他副本通过Pull模式同步Leader消息，处于同步状态的副本集合称为ISR

3.生产者和消费者都只能从Leader写入或者读取数据，Leader故障后，会优先从ISR集合中选择副本作为Leader

4.ISR同步副本的集合

![image](https://user-images.githubusercontent.com/29170657/62873198-8bfb1780-bd51-11e9-8ad5-2ed550fbc1fb.png)

### Kafka批量生产机制

1.在客户端通过异步接口发送消息，消息首先在客户端根据分区打包

2.发送线程根据分区Leader所在broker，把多个batch组成一个请求，打包发送。

3.2个参数控制打包速度：当batch.size达到或者linger.ms时间达到。批量发送包的大小越大吞吐量越高，但是时延也响应增

![image](https://user-images.githubusercontent.com/29170657/62873342-d086b300-bd51-11e9-9544-5b34e6212a38.png)

### Kafka消费机制

1.每个消费组里面的消费者都需要先查找一个协调者

2.消费者加入到这个组内，主要目的是为了对分区进行分区

3.分配分区完毕后，再查找各自分区的Leader，进行消费

![image](https://user-images.githubusercontent.com/29170657/62873455-01ff7e80-bd52-11e9-8a57-286ebe5e2e95.png)

![image](https://user-images.githubusercontent.com/29170657/62873493-15aae500-bd52-11e9-9377-708004dffc75.png)


### Kafka消费进度管理机制

![image](https://user-images.githubusercontent.com/29170657/62874156-47707b80-bd53-11e9-9da3-bf2dabbf54b4.png)


### Kafka高可靠机制

分布式系统下，单点故障不可避免，kafka如何管理节点故障?

1.从Kafka的broker中选择一个节点作为分区管理与副本状态变更的控制，称为controller。

2.统一侦听zk元数据变化，通知各节点状态信息。

3.管理broker节点的故障恢复，对故障节点所在分区进行重新Leader选举，帮助业务故障切换到新的Leader

4.如果Controller节点本身故障？各个Broker节点通过watch zk的controller节点，如果controller故障，会触发节点进行争夺创建controller节点，创建上的节点成为新controller 

![image](https://user-images.githubusercontent.com/29170657/62874325-9dddba00-bd53-11e9-85ef-dfae867431a3.png)









