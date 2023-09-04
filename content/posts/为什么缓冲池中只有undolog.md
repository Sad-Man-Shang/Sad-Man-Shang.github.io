---
title: "为什么缓冲池中只有undolog"
date: 2023-09-04T20:16:33+08:00
draft: false
authors: [Sad_man]
tags: [mysql, innodb]
categories: [innodb, log]
series: [看看时间都浪费在哪里了]
series_weight: 1
---

为什么mysql的innodb存储引擎中，只有undo log，redo log缓冲却不在缓冲池中呢？

个人理解，undolog可以在mvcc的时候可以通过缓存快速读取之前的版本，再出现脏页后可以刷回磁盘与数据页的使用方式类似，所以放在缓冲池中；而redo log 缓冲则是负责保存redo log，正常情况下是不需要从redo log缓冲中读取数据的，而且为了保证数据的持久性，**或许**更高频率的保存到磁盘才是更安全的方法，不太可能使用和缓冲池一样的落盘策略，比如redo log缓冲的写盘使用了两次写的策略。



1. **Undo Logs**:
   - **主要用途**: 提供数据的旧版本，从而支持MVCC以及事务回滚功能。
   - **在Buffer Pool中的角色**:
     - **读**: 当需要基于MVCC为某个事务提供数据的旧版本时，Undo Logs中缓存的数据页可以加速此读取过程。或者当事务需要回滚的时候应用undo logs到数据页。
     - **写**: 当事务修改数据时，这些更改首先在内存中的Undo Logs里记录（WAL，写undo、写redo、commit）。随后，这些内存中的更改会随着脏页的刷新被写回到磁盘上的Undo Logs中。

2. **Redo Logs**:
   - **主要用途**: 确保数据的持久性，记录数据的变化以便在系统崩溃时重新应用这些变化。
   - **Redo Log Buffer的角色**:
     - **写**: 当事务在内存中进行修改时，相关的更改首先被写入到Redo Log Buffer中。之后，为了确保数据持久性，这些缓存在Redo Log Buffer中的更改会被定期或在事务提交时批量写入到磁盘上的Redo Logs中。

所以：
- Undo logs 的缓存（在Buffer Pool中）主要是为了支持MVCC的读取操作和事物的回滚以及将脏页写回磁盘。
- Redo logs 的缓冲（Redo Log Buffer）主要用来优化数据更改的磁盘写入操作。

这种设计确保了InnoDB在提供高并发性和数据持久性之间取得了很好的平衡。
