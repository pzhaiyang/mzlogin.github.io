---
layout: post
title: kafka数据可靠性的深入研究
categories: 分布式
description: kafka
keywords: Kafka
---

## Kafka简介

**前世今生：**Kakfa起初是由LinkedIn公司开发的一个分布式的消息系统，后成为Apache的一部分，它使用Scala编写，以可水平扩展和高吞吐率而被广泛使用。目前越来越多的开源分布式处理系统如Cloudera、Apache Storm、Spark等都支持与Kafka集成。

Kafka凭借着自身的优势，越来越受到互联网企业的青睐，唯品会也采用Kafka作为其内部核心消息引擎之一。Kafka作为一个商业级消息中间件，消息可靠性的重要性可想而知。如何确保消息的精确传输？如何确保消息的准确存储？如何确保消息的正确消费？这些都是需要考虑的问题。本文首先从Kafka的架构着手，先了解下Kafka的基本原理，然后通过对kakfa的存储机制、复制原理、同步原理、可靠性和持久性保证等等一步步对其可靠性进行分析，最后通过benchmark来增强对Kafka高可靠性的认知。。

![](/images/kafka.png)

如上图所示，Kafka体系由N个Producer，N个borker，N个Consumer，以及一个Zookeeper集群组成，Zookeeper在这里主要管理集群leader选举，负载均衡等工作，Producer将消息push到broker中，Consumer则相应在broker的订阅并pull消息。

## Topic&Partition&segment
熟悉MQ的朋友们应该都知道，topic是一类消息的集合，在kafka中，topic又可以分为若干个Partition，partition在存储层面是append log文件。新发布到此partition的消息都会被追加到log文件的尾部，同时以offset去唯一标记这条消息。这种写消息的方式是顺序写磁盘，这样保证了kafka的高吞吐量。

在topic被创建时，可以通过修改配置文件（$KAFKA_HOME/config/server.properties）来设置partition的数量，当一条消息被推送到broker中时，会根基partition的规则来选择具体的partition。如果这种规则足够合理，那么消息的分布将会非常均匀。
![](/images/kafka1.png)

## Kafka文件存储机制
在Kafka的文件系统里，topic下包含多个partition，每个partition作为一个目录，命名规则为topic名称+有序序号，有序序号从0开始到partition的总数量-1.

而在Kafka的设计中，每个partition又可以细分为多个segment段数据文件中，这些数据文件只需要支持顺序读写，这样极大的提高了磁盘的利用率。每个segment文件由.index和.log两个文件组成，分别为segment的索引文件和数据文件。partition的segment文件从0开始，后面的每个segment文件名都是该文件最后一条消息的offset值（偏移量）。


```java
00000000000000000000.index
00000000000000000000.log
00000000000000170410.index
00000000000000170410.log
00000000000000239430.index
00000000000000239430.log
```


## Kafka复制原理及同步方式
![](/images/kafka2.png)
在partition中有2个概念要介绍一下，一个是LEO(Long End Offset)，及这个partition最后一条消息的偏移量，还有一个是HW（High Water Mark），是consumer能看到此partition的位置。

为了提高消息的可靠性，每个topic的partition有N个副本（replicas），N被称为是topic的复制因子（replica fator）的个数。Kafka通过多副本机制实现故障自动转移，这样保证在Kafka集群中一个broker失效，仍然保证集群服务的可用性。其中在这N个replicas中，有一个replica为leader，其他的都是follower，leader处理partition的所有的读写请求，并定期去同步follower上的数据。
如下图所示：
![](/images/kafka3.png)

