---
layout: post
title: 简说分布式事务
categories: 分布式
description: 分布式事务
keywords: 分布式事务
---

## 传统系统事务实现机制

**举个栗子：**这里我们拿一个很常规的转账操作来模拟一下，在规模较小的系统中，表都在一个数据库实例中，我们可以轻松的实现事务的控制。比如A账户给B账户转账100元钱，我们需要做的操作是A账户扣100元，同时给B账户加100元。
SQL语句为:
update table set amount=amount-100 where user='A'；
update table set amount=amount+100 where user='B'；
用事务解决的话就是：
```java
Begin transaction
         update A set amount=amount-10000 where userId=1;
         update B set amount=amount+10000 where userId=1;
End transaction
commit;
```
另外我们也可以用spring很轻松的实现，加个注解就可以了。

![](/images/kafka.png)

如上图所示，Kafka体系由N个Producer，N个borker，N个Consumer，以及一个Zookeeper集群组成，Zookeeper在这里主要管理集群leader选举，负载均衡等工作，Producer将消息push到broker中，Consumer则相应在broker的订阅并pull消息。

## Topic&Partition&segment
熟悉MQ的朋友们应该都知道，topic是一类消息的集合，在kafka中，topic又可以分为若干个Partition，partition在存储层面是append log文件。新发布到此partition的消息都会被追加到log文件的尾部，同时以offset去唯一标记这条消息。这种写消息的方式是顺序写磁盘，这样保证了kafka的高吞吐量。

在topic被创建时，可以通过修改配置文件（$KAFKA_HOME/config/server.properties）来设置partition的数量，当一条消息被推送到broker中时，会根基partition的规则来选择具体的partition。如果这种规则足够合理，那么消息的分布将会非常均匀。
![](/images/kafka1.png)

