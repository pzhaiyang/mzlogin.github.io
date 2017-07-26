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

