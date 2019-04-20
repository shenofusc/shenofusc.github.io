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
PROPAGATION_NESTED:Execute within a nested transaction if a current transaction exists, behave like PROPAGATION_REQUIRED.

这里的英文注释是直接从源码里贴过来的，我感觉已经描述的非常简洁明了了。这里主要记录下最后一种: PROPAGATION_NESTED 。主要通过以下几个问题来说明下它的机制，问题如下：  

1.如果子事务回滚，会发生什么？

我自己做了下实验，发现如果子事务回滚的话，不仅子事务本身会回滚，父事务同样会回滚。

> 而且实验过程中还发现了一个有意思的东西，如果数据库的主键设置为自增的话，那些没有提交的事务依然会占用自增ID，导致ID跳号。例如测试前的最后一条数据ID为 1 ，一次插入操作回滚后再执行另一次成功的插入，最新的ID会是 3 。

2.如果父事务回滚，会发生什么？

父事务回滚的情况和 PROPAGATION_REQUIRED 模式下没什么区别，子事务和父事务都会回滚。

3.出现异常的话会发生什么？

假设现在是 ServiceA 方法调用 ServiceB 方法，ServiceA 的传播级别为 PROPAGATION_REQUIRED，ServiceB 的传播级别为 PROPAGATION_NESTED，在出现异常的情况下，只要能够吃掉异常，不管是哪个方法的异常，事务就能够正常提交。如果不处理异常的话，不管是 ServiceA 出现异常还是 ServiceB 出现异常，内外层事务都会回滚。很多文章都提到了一个关键字：savepoint，这是一个很特别的东西，在进入 ServiceB 之前，会保存 ServiceA 当前的执行状态到 savepoint，如果 ServiceB 执行过程中出现异常的话，会回滚到 savepoint 所在的状态。但是这里有一个坑，首先，你必须得在 ServiceA 中捕获到异常并且吃掉它，否则 ServiceA 照样回滚，savepoint？不存在的。其次，如果你是在同一个对象中调用不同的方法的话，savepoint 依然是没有的。只有在不同的对象之间进行调用，才会保存 ServiceA 的执行状态到 savepoint ，ServiceB 的事务回滚时才能恢复到 savepoint 所在的状态。

总结一下的话就是这个地方的状态组合超多，稍有不慎就是个大坑，不是很必要的情况下还是老老实实用默认的 PROPAGATION_REQUIRED 比较好。

#### 参考链接

[Spring 事务机制详解](https://juejin.im/post/5a3b1dc4f265da43333e9049)

[Spring事务传播机制-REQUIRED嵌套NESTED](https://blog.csdn.net/ID19870510/article/details/78884130)
