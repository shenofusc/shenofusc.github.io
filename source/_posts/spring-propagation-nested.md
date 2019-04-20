---
title: "Spring嵌套事务机制"
date: 2019-04-20
tags: Spring
---

Spring的几种事务传播机制如下:

PROPAGATION_REQUIRED:Support a current transaction, create a new one if none exists.  
PROPAGATION_SUPPORTS:Support a current transaction, execute non-transactionally if none exists.  
PROPAGATION_MANDATORY:Support a current transaction, throw an exception if none exists.  
PROPAGATION_REQUIRES_NEW:Create a new transaction, and suspend the current transaction if one exists.  
PROPAGATION_NOT_SUPPORTED:Execute non-transactionally, suspend the current transaction if one exists.  
PROPAGATION_NEVER:Execute non-transactionally, throw an exception if a transaction exists.
PROPAGATION_NESTED:Execute within a nested transaction if a current transaction exists, behave like PROPAGATION_REQUIRED else. There is no analogous feature in EJB.

这里的英文注释是直接从源码里贴过来的，我感觉已经描述的非常简洁明了了。这里主要记录下最后一种: PROPAGATION_NESTED 。主要通过两个问题来说明下它的机制，问题如下：  

1.如果子事务回滚，会发生什么？

我自己做了下实验，发现如果子事务回滚的话，不仅子事务本身会回滚，父事务同样会回滚。而且实验过程中还发现了一个有意思的东西，如果数据库的主键设置为自增的话，那些没有提交的事务依然会占用自增ID，导致ID跳号。例如测试前的最后一条数据ID为 1 ，一次插入操作回滚后再执行另一次成功的插入，最新的ID会是 3 。

2.如果父事务回滚，会发生什么？

父事务回滚的情况和 PROPAGATION_REQUIRED 模式下没什么区别，子事务和父事务都会回滚。
