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

## 两阶段提交协议 
但是目前随着互联网技术的飞速发展，单一系统明显不能满足业务的需求，如何在分布式系统中去保证上述的问题就成了一个难题。

两阶段提交协议（Two-phase Commit，2PC）被广泛的应用于分布式事务中来。这种分布式事务的实现方式包括一个调度器和若干个事务执行者组成。如下图所示：
![](/images/transation1.png)

	在这种协议中，当一个请求到达调度器TC中来之后，TC会像2个事务执行者各发一条prepare消息去通知他们执行各自的操作（扣款和加款），同时将在发送消息前需要写一条日志去记录下这一次请求，目的是为了系统崩溃后可以不丢失这一次行为。

	2个SI执行完各自的操作后，并不会进行commit的操作，而是返回TC各自的执行结果，同时记录下这次操作的记录。
	
	TC收集完2个SI的操作返回后，如果2个SI都执行成功的话会再向2个事务执行者发送commit信息，否则则发送rollback操作。SI收到信息后执行各自的操作。
	
	这种协议解决了分布式事务的问题，也被大量使用。但是这种协议同时也暴露出一定的问题：
	
	（1）各系统间通信次数较多，一定程度上拖慢了系统的速度。
	（2）一次操作耗时时间较长，这对目前的互联网企业的分布式系统来说是不能容忍的。
	（3）会留下大量的log。