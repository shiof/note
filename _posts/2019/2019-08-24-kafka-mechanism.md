---
layout: article
title:	kafka机制
date:	2019-08-24 22:33:34
categories:
    - article
tags:
    - Java
    - Kafka
---

### 总体架构

![image](https://user-images.githubusercontent.com/29170657/63638483-2eed5300-c6bb-11e9-88ad-443ac087a5b8.png)

![image](https://user-images.githubusercontent.com/29170657/63638517-d1a5d180-c6bb-11e9-8f44-cc51beb5b3d9.png)

>Zookeeper：存储Kafka元数据
>
>Broker互为主备
>
>topic按分区存储
>
>副本分部在不同节点

### 数据流图

![image](https://user-images.githubusercontent.com/29170657/63638598-fcdcf080-c6bc-11e9-8fea-a3d875e06b1d.png)

### 节点角色

#### Controller

主要负责Partition管理和副本状态管理，也会执行类似于重分配Partition之类的管理任务。如果当前的Controller失败，会从其他正常的Broker中重新选举Controller。

![image](https://user-images.githubusercontent.com/29170657/63638607-25fd8100-c6bd-11e9-8b22-46ebda664b0f.png)

zookeeper相关节点

![image](https://user-images.githubusercontent.com/29170657/63638621-4af1f400-c6bd-11e9-87a2-82b38918f7ae.png)

>Broker节点状态管理：新增节点、节点下线、节点故障、节点元数据更新等
>
>Topic分区状态管理：topic创建删除、分区扩容、分区迁移、leader切换等

#### Leader

负责分区消息的读写请求，故障会自动进行切换

选举

![image](https://user-images.githubusercontent.com/29170657/63638674-0fa3f500-c6be-11e9-8a74-3e0672ccdbf5.png)

#### Follower

负责同步leader数据，形成副本，leader故障时，可变成leader，进行故障切换

#### Coordinator

负责consumer group的管理，内部消费进度队列的维护等

### Topic流程

创建

![image](https://user-images.githubusercontent.com/29170657/63638648-b4720280-c6bd-11e9-97bc-2f902abea390.png)

删除

![image](https://user-images.githubusercontent.com/29170657/63638659-d4a1c180-c6bd-11e9-9b6c-66941d5dfa37.png)

### 分区扩容流程

![image](https://user-images.githubusercontent.com/29170657/63638690-4548de00-c6be-11e9-8865-4d39870101db.png)

### 副本迁移流程

![image](https://user-images.githubusercontent.com/29170657/63638710-71645f00-c6be-11e9-86ce-d45e5213cb04.png)

### 生产请求流程

![image](https://user-images.githubusercontent.com/29170657/63638723-93f67800-c6be-11e9-926e-6f250336defe.png)