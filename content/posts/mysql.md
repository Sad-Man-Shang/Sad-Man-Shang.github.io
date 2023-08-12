---
title: "Mysql"
date: 2023-07-21T16:40:24+10:00
draft: false
authors: [Sad_man]
tags: [面试]
categories: [数据库]
series: [24届从零开始到找到工作]
series_weight: 1
---

关于**innodb 存储引擎**以及**mysql45讲**，[CMU](https://15445.courses.cs.cmu.edu/fall2022/schedule.html)的笔记

# 零、基本用法

判断等`=` 

使用`''`

desc 降序（大的在前）

`Limit N offset M` 第N-1开始M个即`Limit N, M`

解决空的问题，`select () as xx`对原语句进行包裹，作为临时表，或者使用 `select ifnull ( (sentence), null) as xx`







# 一、数据库整体框架（基本没写，别看）

WALogging

查询优化：优化树（拆分条件，条件下拉，使用join，排序最严格的条件，下拉列选择）选择索引（有索引用索引，没索引用有序，无序遍历），loop算法

h

引擎层与服务层的交界：引擎层向服务层提供读写数据的接口



##A. 数据库存储

![image-20221123202452860](./mysql/image-20221123202452860.png)

Disk manage goals：

使用超过内存的空间

有序存储在稳定的工具上

减少IO



必要性：为什么不直接交给OS来做？因为需要针对内存进行优化。

使用OS的缺点：

https://blog.toadworld.com/author/juan-carlos-olamendy

https://blog.toadworld.com/2017/10/19/data-flushing-mechanisms-in-innodb

事务安全：OS可以随时刷新脏页，被别的书屋读取，而BPM可以把握什么时候写入

I/O 停顿：不是随时可以获得的,要等数据被加载，而优化可以预读，

错误处理：每次访问page都要进行checksum，而BPM只需要读如内存的时候进行一次checksum就好了

性能：OS 的page淘汰机制不具备很好的扩展性，尤其是SSD这种bandwidth很高的情况下，

1. page table上的多线程竞争
2. page cache的单线程淘汰机制

3. TLB shootdown



BPM可以管理：
正确的刷新脏页

预读取

缓冲置换方法

线程/进程 安排



### 1. File storage

**Storage manager**

用于维护文件，制定scheduling来读写，以提定位页的时间和空间。文件以页的集合的形式存在。要管理数据读写，以及可用空间。

**Pages**

一定大小的block，基本不会混存，meta-data，log records，index，tuples。

**Page ID** ->**indirection layer** ->  **physical loaction**



DB page 16KBMySQL

OS page 4KB

Hardware page 4KB



**页的存储结构**

堆：无序，CRUD，支持迭代，offset=page size* page #，需要meta-data来维持page在哪些页中以及哪些有剩余空间。

directory pages来记录data page位置（因此要求同步），同时记录free space以及free page



树：

有序文件

hash文件





### 2. Page Layout

**page header**

记录meta-data，页大小，checksum，DBMS版本，事务能见度，鸭锁信息



如何存储tuple？

面向tuple——slotted pages

header 维护了# used slots以及最后一个slot的起始位置

![image-20221124014353146](./mysql_2.png)



Record ID page_id + offset/slot

log-structured

### 3. Tuple Layout

**Tuple header**： Visibility info (concurrency control)  & Bit Map for **NULL** values.



physically **denormalize**, 将tuple和相关的tuple（外键联系的）存在一个page上，可以减少IO，

### 4. disk- oriented 架构

主要存储位置在磁盘上。

组件管理非易失性和易失性存储之间的数据移动。



### 5. 面向页的架构

**插入数据**（mysql插入索引，插入叶子结点为目的。）

1. 查看page directory 找到页中的free slot（）
1. 读取到内存中
1. 从slot array中找到empty space来放置（mysql，1/16的xx空间）





**更新存在的数据**

1. 根据page directory找到页
2. 取回到内存
3. 使用slot array找到偏移量
4. 覆盖





### 6. LOG-STRUCTURED STORAGE

面向页存储的选择

存储日志记录来包含改变。

记录包括了：

record id（=page_id + offset/slot）

操作：删除则mark删除，添加则记录加入数据。

当日志页满了的时候，写入磁盘。日志有序且不可变。

读取：track，record id的最近更新位置。



日志压缩：保留最新纪录，维持顺序



存储日志的方式的问题：写频率较高，压缩困难



### 7. 数据表示

DATA REPRESENTATION

**INTEGER**/**BIGINT**/**SMALLINT**/**TINYINT** → C/C++ Representation

**float/real vs. numeric/decimal**

IEEE-754 Standard / Fixed-point Decimals

**VARCHAR**/**VARBINARY**/**TEXT**/**BLOB

** → Header with length, followed by data bytes. → Need to worry about collations / sorting.

**TIME**/**DATE**/**TIMESTAMP

** → 32/64-bit integer of (micro)seconds since Unix epoch



[MySQL源码](https://github.com/mysql/mysql-server/blob/8.0/strings/decimal.cc#L1828)

![image-20221125005326870](./mysql/image-20221125005326870.png)



**Large Value**

不允许存超过一个页的tuple，这个时候引入 overflow storage pages（BLOB page）虽然不保证持久化和事务。

### 8. 系统目录

在内部目录中，存了表，列，索引，视图

用户与权限

内部统计

可以通过SELECT *
       FROM information_schema.tables;查看







## B. 存储模型与压缩

OLTP，一次少量read/update

OLAP，大量复杂数据计算聚合

HTAP，混合OLTP和OLTP在一个数据库实例上

## C. 内存管理





##D. Hash Table





## 5. Tree index



## 6. Index Concurrency Control；



## 7. Sorting & Aggreation Algorithms



## 8. Joins Algorithms



## 9. Query Execution





## 10. Query Planning & Optimization





## 11. Concurrency Control Theory





## 12. Two-phase locking Concurrency Control



## 13. Timestamp Ordering Concurrency COntrol



## 14. Multi-Version Concurrency Control



## 15. Database Logging



## 16. Database Recovery



## 17. Distributed Database





















# 二、innodb存储引擎体系架构



## A. MySQL整体结构

连接池、管理服务和工具、SQL接口、解析器、优化器、执行器和缓冲池、可插拔引擎、文件系统、文件日志索引等

![image-20221117171612627](./mysql/image-20221117171612627.png)

## B. InnoDB体系架构

5.5后作为默认的表存储引擎

支持ACID事务，行锁，MVCC，外键

有效使用内存（缓存）和CPU（多线程）

| 版本         | 功能                          |
| ------------ | ----------------------------- |
| 老版本InnoDB | 支持ACID、行锁设计、MVCC      |
| InnoDB 1.0.x | 增加了compress和dynamic页格式 |
| InnoDB 1.1.x | 增加了Linux AIO、多回滚段     |
| InnoDB 1.2.x | 增加了全文索引、在线索引      |



**内存结构**

缓冲池、重做日志缓冲

 ![image-20221118004046432](./mysql/image-20221118004046432.png)



### 内存-缓冲池

#### Why？

innoDB基于磁盘存储的，且按照页的方式进行管理。

**磁盘IO慢**，为了提速，引入**缓冲池**，提高效率

#### 如何保证数据不丢失？

使用了checkpoint机制刷新回磁盘。	commit与checkpoint的关系

设置参数：innodb_buffer_pool_size,默认128MB

***高并发访问下的性能问题***

***可以缓存热点数据（不可控）也可以放在第三方介质（可控）***



**缓冲池**包括：数据页，undo页，索引页，插入缓冲，自适应哈斯索引，lock信息，数据字典信息，锁信息。

redo页在缓冲池外面，***WHY？***。

- 可以有多个缓冲池实例，每个page根据hash value平均分配到不同的缓冲池里，减少数据库内部的数据竞争。

innodb_buffer_pool_instances

pool size小于1G的时候，pool_instances不生效

 5.6版本开始，还可以通过information_schema架构下的表INNODB_BUFFER_POOL_STATS查看





#### 缓冲池管理

**LRU算法**

缓冲池中页的大小默认为16KB

midpoint insertion strategy，读取后放在LRU列表的midpoint，innodb_old_blocks_pct默认值为37，即从尾短开始37%的位置

innodb_old_blocks_time，value默认是1000，即1000后放置到热端。

midpoint之前是new，之后是old

**free 列表**

数据库刚启动时，lru列表是空的。页存放在free list中，当需要从缓冲池中分页的时候，首先从free list查找是否有空闲页，如果有责将其从free list中删除，并加入到lru list中。否则，根据LRU算法，淘汰末尾的页，分配给新的page。

从old到new的过程是page made young，没有进入new的过程是page not made young

free buffers是free列表的page数量，即空闲的数量。

database pages就是LRU列表中页的数量。

free buffers+database pages 不等于buffer pool size，因为缓冲池中的页还可以被分配给锁信息，自适应哈希索引，插入缓存，数据字典信息等，不需要LRU维护。





buffer pool hit rate，越高越好，不应当小于95%，如果发生全表扫描，可能会污染LRU



LRU包含了unzip_LRU，用于存放非16kb内存页，即1，2，4，8kb内存页

LRU数量包含了unzip_LRU数量

**unzip_LRU**：对于不同大小的页分别管理，如对需要从缓冲池中申请页为4KB的大小，其过程如下：

1）检查4KB的unzip_LRU列表，检查是否有可用的空闲页；

2）若有，则直接使用；

3）否则，检查8KB的unzip_LRU列表；

4）若能够得到空闲页，将页分成2个4KB页，存放到4KB的unzip_LRU列表；

5）若不能得到空闲页，从LRU列表中申请一个16KB的页，将页分为1个8KB的页、2个4KB的页，分别存放到对应的unzip_LRU列表中。





当LRU list中数据被修改后，page变成了dirty page，即缓冲池和磁盘的数据不一致。通过checkpoint机制来刷新回磁盘。Flush list里面都是dirty page。

LRU用来管理缓冲池中page的可用性，而flush list用来管理刷新到磁盘，不影响。

### redo log 缓冲

innodb_log_buffer_size 默认8MB

每秒都会从buffer到redo log，所以只要保证大小够1秒的就好

何时刷出到外部磁盘？

1.Master Thread每一秒将重做日志缓冲刷新到重做日志文件；

2.每个事务提交时前，会将重做日志缓冲刷新到重做日志文件；

3.当重做日志缓冲池剩余空间小于1/2时，重做日志缓冲刷新到重做日志文件。



### 额外的内存池

用于存放帧缓冲，缓冲控制对象，这些对象记录了如LRU，锁，等待等信息，





### 后台线程-Innodb的多线程模型

master thread

核心线程，主要负责将缓冲池中的数据异步刷新到磁盘，保证数据的一致性，包括脏页的刷新、合并插入缓冲（INSERT BUFFER）、UNDO页的回收等



#### Master thread

#### loop(1.0.x)

Master Thread具有最高的线程优先级别。

由多个循环（loop）组成

​	1. 主循环（loop)

​	2. 后台循环（backgroup loop）

​	3. 刷新循环（flush loop）

​	4. 暂停循环（suspend loop）。

Master Thread会根据数据库运行的状态在loop、background loop、flush loop和suspendloop中进行切换。Loop被称为主循环，因为大多数的操作是在这个循环中，其中有两大部分的操作——每秒钟的操作和每10秒的操作。

![image-20221120232524042](./mysql/image-20221120232524042.png)

#### 每秒操作

❑日志缓冲刷新到磁盘，即使这个事务还没有提交（总是）；

❑合并插入缓冲 if IO 次数小于5；

❑至多刷新100个InnoDB的缓冲池中的脏页到磁盘 if innodb_max_dirty_pages_pct > 90；

❑如果当前没有用户活动，则切换到background loop（可能）。



#### 每10秒的操作

❑合并至多5个插入缓冲（总是）

❑将日志缓冲刷新到磁盘（总是）；

❑删除无用的Undo页（总是）至多20条；1.2.x后，交给page cleaner thread

❑刷新100个或者10个脏页到磁盘（总是）。if buf_get_modified_ratio_pct > 70, 刷新100哥 else if IO times < 200 刷新100, else 刷新10



#### background loop，flush loop，suspend loop

若当前没有用户活动（数据库空闲时）或者数据库关闭（shutdown），就会切换到这个循环。

执行以下操作：

❑删除无用的Undo页（总是）；

❑合并20个插入缓冲（总是）；

❑跳回到主循环（总是）；

不断刷新100个页直到符合条件（可能，跳转到flush loop中完成）。

若flush loop中也没有什么事情可以做了，InnoDB存储引擎会切换到suspend__loop，将Master Thread挂起，等待事件的发生。

**InnoDB1.2.x** **版本之前**的Master ThreadInnoDB Plugin（从InnoDB1.0.x版本开始）提供了参数innodb_io_capacity，用来表示磁盘IO的吞吐量，默认值为200。对于刷新到磁盘页的数量，会按照innodb_io_capacity的百分比来进行控制。规则如下：❑在合并插入缓冲时，合并插入缓冲的数量为innodb_io_capacity值的5%；❑在从缓冲区刷新脏页时，刷新脏页的数量为innodb_io_capacity。若用户使用了SSD类的磁盘，或者将几块磁盘做了RAID，当存储设备拥有更高的IO速度时，完全可以将innodb_io_capacity的值调得再高点，直到符合磁盘IO的吞吐量为止。



还有一个改变是：之前每次进行full purge操作时，最多回收20个Undo页，从InnoDB 1.0.x版本开始引入了参数innodb_purge_batch_size，该参数可以控制每次full purge回收的Undo页的数量。该参数的默认值为20，并可以动态地对其进行修改InnoDB 1.0.x版本在性能方面取得了极大的提高，其实这和前面提到的Master Thread的改动是密不可分的，因为InnoDB存储引擎的核心操作大部分都集中在Master Thread后台线程中。

3. InnoDB 1.2.x版本的Master Thread在InnoDB 1.2.x版本中再次对Master Thread进行了优化，由此也可以看出Master Thread对性能所起到的关键作用。在InnoDB 1.2.x版本中，Master Thread的伪代码如下：if InnoDB is idlesrv_master_do_idle_tasks();elsesrv_master_do_active_tasks();其中srv_master_do_idle_tasks()就是之前版本中每10秒的操作，srv_master_do_active_tasks()处理的是之前每秒中的操作。同时对于刷新脏页的操作，从Master Thread线程分离到一个单独的Page Cleaner Thread，从而减轻了Master Thread的工作，同时进一步提高了系统的并发性。



#### IO Thread

在InnoDB存储引擎中大量使用了AIO（Async IO）来处理写IO请求，这样可以极大提高数据库的性能。而IO Thread的工作主要是负责这些IO请求的回调（call back）处理。InnoDB 1.0版本之前共有4个IO Thread，分别是write、read、insert buffer和log IO thread。在Linux平台下，IO Thread的数量不能进行调整，但是在Windows平台下可以通过参数innodb_file_io_threads来增大IO Thread。从InnoDB 1.0.x版本开始，read thread和write thread分别增大到了4个，并且不再使用innodb_file_io_threads参数，而是分别使用innodb_read_io_threads和innodb_write_io_threads参数进行设置。  可以看到IO Thread 0为insert buffer thread。IO Thread 1为log thread。之后就是根据参数innodb_read_io_threads及innodb_write_io_threads来设置的读写线程，并且读线程的ID总是小于写线程。

	SHOW VARIABLES LIKE'innodb_%io_threads';
	SHOW ENGINE INNODB STATUS;
	SHOW VARIABLES LIKE'innodb_purge_threads';

` 

Purge Thread

事务被提交后不需要undolog，那么需要PurgeThread来回收undo页

1.0 1.1 1.2

在master中，单独的，多个



Page cleaner 1.2.x

作用是将之前版本中脏页的刷新操作都放入到单独的线程中来完成。而其目的是为了减轻原Master Thread的工作及对于用户查询线程的阻塞，进一步提高InnoDB存储引擎的性能。





## C. checkpoint

**Write Ahead Log**策略，即先写日志，再改数据。先写BFIM再写AFIM。先AFIM，再提交。

**作用**：缩短恢复时间；缓冲池不够用时，将脏页刷新到磁盘；重做日志不可用时，刷新脏页，以减少redo log。

**LSN**，log sequence number，8btyes，页，checkpoint和redo log都有LSN



innodb支持两种checkpoint：

sharp checkpoint，发生在数据库关闭时；

fuzzy checkpoint：上述中作用的三种情况，以及

![image-20221118010424074](./mysql/image-20221118010424074.png)

**master thread checkpoint**：每秒或每10秒

**FLUSH_LRU_LIST checkpoint**：释放LRU空间，1.12 版本开始， page cleaner thread负责，innodb_lru_sacn_depth，默认保证有1024个page可用

 page cleaner thread

**Async/Sync Flush Checkpoint**:用于减少redo log。将写入redo log的LSN记为redo_LSN, 已经刷新回磁盘最新页的LSN记为checkpoint_LSN

于是可以知道从redo_lsn 到checkpoint_lsn的范围是没有写入到redo log的，如果其小于日志总文件大小的75%，则无所谓，75-90，触发异步刷新，90以上同步刷新。刷新到小于75

 page cleaner thread

**Dirty page too much checkpoint**： 保证缓冲池可用页，innodb_dirty_pages_pct 默认75







## D. innodb五个关键特性

❑插入缓冲（Insert Buffer）（减少IO次数，只对不唯一插入生效）

❑两次写（Double Write）（保证写入成功）

❑自适应哈希索引（Adaptive Hash Index）

❑异步IO（Async IO）（减少IO次数，连续多页读）

❑刷新邻接页（Flush Neighbor Page）（减少IO次数，连续多页写）

### 插入缓冲

对于非唯一非聚集索引的插入或更新操作



查看索引页是否在缓冲池中，如果在则直接插入，如果不在，先放在insert buffer对象中。然后在merge



缺点：如果发生宕机，不在缓冲池且至少在10s内发生了高并发，根据WAL，则需要根据log需要大量的时间回复。占用大量缓冲池内存，最多可以到50%



innodb1.0.x开始引入change buffer，可以对插入，删除，修改都进行缓冲，即insert buffer，delete buffer，purge buffer。



#### 实现细节

insert buffer 是b+树，

**非叶子结点：**![image-20221123150933088](./mysql/image-20221123150933088.png)

space 表空间ID 4byte

marker 兼容性位置 1byte

offset 偏移量 4byte



**叶子结点：**

![image-20221123151141201](./mysql/image-20221123151141201.png)

metadata：4 bytes

IBUF_REC_OFFSET_COUNT：进入Insert Buffer的顺序

IBUF_REC_OFFSET_TYPE：

IBUF_REC_OFFSET_FLAGS：



为了保证每次Merge Insert Buffer页必须成功，还需要有一个特殊的页用来标记每个辅助索引页（space，page_no）的可用空间。这个页的类型为Insert Buffer Bitmap。

![image-20221123151803075](./mysql/image-20221123151803075.png)







什么时候Merge？

辅助索引页被读取到缓冲池时

Insert Buffer Bitmap页追踪到该辅助索引页已无可用空间时；Insert Buffer Bitmap页用来追踪每个辅助索引页的可用空间，并至少有1/32页的空间

Master Thread。之前在分析Master Thread时曾讲到，在Master Thread线程中每秒或每10秒会进行一次Merge Insert Buffer的操作，不同之处在于每次进行merge操作的页的数量不同。



merge 页选择：随机选择，更公平，如果页已经被删除，则直接丢弃。



### 两次写

先通过memcopy函数复制到内存的doublewrite buffer 2MB, 然后 分两次复制到共享表空间的共享物理磁盘上，然后调用fsync函数同步磁盘，避免缓冲写带来的问题。



双写其实是备份了一个完整的page。如果直接写入数据文件，则可能写到一半宕机，则页被破坏。无法通过redolog恢复，因为redolog记录的是在页的某一个片段的数据，而如果整个页的部份发生损毁，那么即使记录了部份页的数据也不能恢复整个页



### 自适应哈希索引

速度快

统计访问频率

访问了N次，访问了N=页中记录数量的/16



### 异步IO

1.速度快

2.判断访问页是否可以连续读取， 比如连续的三个页，一次读取48kb，减少IO次数



### 刷新邻接页

刷新脏页的时候，这个页所在区的所有page，如果有脏页，一块刷新，好处是合并IO次数





## E. Innodb关闭启动与恢复

#### 关闭：

innodb_fast_shutdown:0,1,2	默认1

❑0表示在MySQL数据库关闭时，InnoDB需要完成所有的full purge和merge insert buffer，并且将所有的脏页刷新回磁盘。这需要一些时间，有时甚至需要几个小时来完成。如果在进行InnoDB升级时，必须将这个参数调为0，然后再关闭数据库。

❑1是参数innodb_fast_shutdown的默认值，表示不需要完成上述的full purge和merge insert buffer操作，但是在缓冲池中的一些数据脏页还是会刷新回磁盘。

❑2表示不完成full purge和merge insert buffer操作，也不将缓冲池中的数据脏页写回磁盘，而是将日志都写入日志文件。这样不会有任何事务的丢失，但是下次MySQL数据库启动时，会进行恢复操作（recovery）。







# 三、文件



## 参数文件

启动时先读取参数文件，可以看成key - value pair

有的是只读，有的是可以改

## 日志文件-错误日志（实用性

辅助定位错误，log_error





## 日志文件-慢查询日志（调优重要

slow_query_log 默认不开启

**按时间划分**

long_query_time精确到微秒，大于该时间的会被记录

**按是否使用索引划分**

log_queries_not_using_indexes

这种情况下可能造成频繁的记录，导致slow log不断增大，可通过log_throttle_queries_not_using_indexes来进行限制每分钟可记录条数



## 日志文件-查询日志（了解

所有请求信息。

默认名为主机名.log

general.log

## 日志文件-二进制文件（最重要

记录了所有修改操作（不包括show 和 select

### 作用

恢复（recovery）：某些数据的恢复需要二进制日志，例如，在一个数据库全备文件恢复后，用户可以通过二进制日志进行point-in-time的恢复。

复制（replication）：其原理与恢复类似，通过复制和执行二进制日志使一台远程的MySQL数据库（一般称为slave或standby）与一台MySQL数据库（一般称为master或primary）进行实时同步。

审计（audit）：用户可以通过二进制日志中的信息来进行审计，判断是否有对数据库进行注入的攻击。



默认关闭：since 恢复可以用redo，如果是单机，不需要审计

log_bin = off OR log_bin = 文件名



分为log_name.index 和log_name.000001



**参数**

max_binlog_size 默认1GB

binlog_cache_size 默认32KB，基于会话。存储uncommitted log record，大于该值时，写入临时文件，

sync_binlog 每写多少次缓冲就同步到磁盘，如果为1则不使用缓冲，0则没有限制。

binlog_do_db 哪些需要二进制文件

binlog_ignore_db 忽略哪些库的日志

log_slave_update 从主服务器获取二进制文件并且入自己的二进制文件中去，

默认off，依然可以同步，但是只用不存，ON则会存入自己的二进制文件。

所以，当a->b->c的时候，b off，则c无法读取到a

binlog_format 存储形式。statement，row，mixed。如果设置为ROW可以重复读，获得更高的并发性。



MIXED：默认采用STATEMENT格式进行二进制日志文件的记录，但是在一些情况下会使用ROW格式：	1）表的存储引擎为NDB，这时对表的DML操作都会以ROW格式记录。	2）使用了UUID()、USER()、CURRENT_USER()、FOUND_ROWS()、ROW_COUNT()等不确定函数。	3）使用了INSERT DELAY语句。	4）使用了用户定义函数（UDF）。	5）使用了临时表（temporary table）。



## Socket文件（了解

UNIX系统下，用UNIX域套接字方式。需要一个套接字文件，一般在/tmp目录下，名为mysql.sock。







## pid文件（了解

当MySQL实例启动时，会将自己的进程ID写入一个文件中——该文件即为pid文件。该文件可由参数pid_file控制，默认位于数据库目录下，文件名为主机名.pid



## MySQL表结构文件

每个表都会有与之对应的表结构文件。frm为后缀。不需要修改，了解即可

![image-20221124144520427](./mysql/image-20221124144520427.png)



## 存储引擎文件-表空间文件

默认12MB，自动扩展，ibdata1文件，默认表空间

innodb_data_file_path，设置了之后，所有表都存在这里，可以用两个文件组成表空间，通过放在不同的磁盘上，磁盘的负载被平均，提高性能。



innodb_file_per_table ，默认开启每个表都有独立表空间，.ibd，这里仅存data， index，插入缓冲BITMAP，其余信息依旧存在于ibdata1



## 存储引擎文件-重做日志文件

in_logfile0 和in_logfile1，innodb至少有一个group，每个group至少有2个重做日志文件。

innodb_log_file_size，1.2之前4G，1.2扩大到512G

innodb_log_files_in_group

innodb_mirrored_log_groups

innodb_log_group_home_dir

大小设置很关键，太大则恢复时间长，太小可能导致一个事物的日志需要多次切换重做日志文件，或者进而导致async checkpoint频繁发生，导致性能抖动

**为什么不用redolog进行主从复制？**

二进制文件会记录所有日志记录，各种引擎的都会记录。而重做日志只记录innodb引擎的。

二进制文件关于事务的具体操作内容。而重做日志是记录了每个页的物理情况。

写入时间不同，二进制文件只在commit前写一次，重做日志不断写入。

![image-20221124150740852](./mysql/image-20221124150740852.png)

先写入redo log  buffer，再写入redo log文件。写入磁盘时，512bytes一组，一个扇区，原子性，因此不需要有double write

# 四、InnoDB存储引擎的表

## A. innodb 逻辑存储结构

根据主键顺序，称之为index organized table。如果没有主键，则选择非空唯一**索引**，不然，则自动创建6bytes pointer，_rowid

### 表空间结构

![image-20221124152721303](./mysql/image-20221124152721303.png)



如果启用了innodb_file_per_table的参数，需要注意的是每张表的表空间内存放的只是数据、索引和插入缓冲Bitmap页，其他类的数据，如回滚（undo）信息，插入缓冲索引页、系统事务信息，二次写缓冲（Double write buffer）等还是存放在原来的共享表空间ibdata1内。

共享表空间的undo不会被回收，但是会被标记为可用。



### 段

数据段（叶子结点）、索引段（非叶子结点）、回滚段。

### 区

1MB，连续页组成，每次申请4～5个区保证页的连续。

16KB一个页，64页/区



1.0.x	key_block_size，压缩页，设置为2，4，8KB

1.2.x	innodb_page_size，修改默认大小4，8KB



开启inndob_file_per_table 后，表默认大小96KB，为什么不是1MB？不会在一开始就使用连续的页，而是先使用32个页大小的碎片页，即数据来到0.5MB后，第二次申请区才会连续。



## 页

innodb磁盘管理最小单位。

页的类型

❑数据页（B-tree Node）

❑undo页（undo Log Page）

❑系统页（System Page）

❑事务数据页（Transaction system Page）

❑插入缓冲位图页（Insert Buffer Bitmap）

❑插入缓冲空闲列表页（Insert Buffer Free List）

❑未压缩的二进制大对象页（Uncompressed BLOB Page）

❑压缩的二进制大对象页（compressed BLOB Page）

## 行-compact格式（默认

innodb按照行存储，与列存储区别？

每页最多存放16KB/2 - 200，即2～7992行

![image-20221124155147215](./mysql/image-20221124155147215.png)



Compact行记录格式的首部是一个非NULL变长字段长度列表，并且其是按照列的顺序逆序放置的，其长度为：

❑若列的长度小于255字节，用1字节表示；

❑若大于255个字节，用2字节表示。变长字段的长度最大不可以超过2字节，这是因在MySQL数据库中VARCHAR类型的最大长度限制为65535bytes,实际为65532，且是一列中所有变长字符串之和。对于大对象或者长字符串，*如果一个page只能存放一个对象，为了使用B+树*，则把大对象放在BLOB PAGE

NULL标志位，行中有NULL值，则1。

记录头信息（record header），固定占用5字节（40位），每位的含义见表

![image-20221124160026756](./mysql/image-20221124160026756.png)



最后的部分就是实际存储每个列的数据。需要特别注意的是，NULL不占该部分任何空间，即NULL除了占有NULL标志位，实际存储不占有任何空间。另外有一点需要注意的是，每行数据除了用户定义的列外，还有两个隐藏列，事务ID列和回滚指针列，分别为6字节和7字节的大小。若InnoDB表没有定义主键，每行还会增加一个6字节的rowid列。



行compact格式下，char要用20补齐



## 行- redundant（5.0之前的默认，被淘汰了。

字段长度偏移列表，同样是按照列的顺序逆序放置的。若列的长度小于255字节，用1字节表示；若大于255字节，用2字节表示。

记录头信息（record header），不同于Compact行记录格式，Redundant行记录格式的记录头占用6字节（48位）



![image-20221124161235388](./mysql/image-20221124161235388.png)

![image-20221124161346603](./mysql/image-20221124161346603.png)





## 数据页结构（了解



![image-20221124162257772](./mysql/image-20221124162257772.png)

InnoDB数据页由以下7个部分组成，如图：

File Header（文件头）

Page Header（页头）

Infimun和Supremum Records

User Records（用户记录，即行记录）

Free Space（空闲空间）

Page Directory（页目录）

File Trailer（文件结尾信息）

其中File Header、Page Header、File Trailer的大小是固定的，分别为38、56、8字节，这些空间用来标记该页的一些信息，如Checksum，数据页所在B+树索引的层数等。User Records、Free Space、Page Directory这些部分为实际的行记录存储空间，因此大小是动态的。

**File Header**用来记录页的一些头信息，由表中8个部分组成，共占用38字节。

![image-20221124162530496](./mysql/image-20221124162530496.png)



![image-20221124162507691](./mysql/image-20221124162507691.png)



**Page Header**，该部分用来记录数据页的状态信息，由14个部分组成，共占用56字节，

![image-20221124162553841](./mysql/image-20221124162553841.png)

**Infimum和Supremum Record**

在InnoDB存储引擎中，每个数据页中有两个虚拟的行记录，用来限定记录的边界。Infimum记录是比该页中任何主键值都要小的值，Supremum指比任何可能大的值还要大的值。这两个值在页创建时被建立，并且在任何情况下不会被删除。

![image-20221124162709352](./mysql/image-20221124162709352.png)





**User Record,Free Space,Page Directory和File Trailer**

User Record即实际存储行记录的内容。

Free Space很明显指的就是空闲空间，同样也是个链表数据结构。在一条记录被删除后，该空间会被加入到空闲链表中。

Page Directory（页目录）中存放了记录的相对位置（注意，这里存放的是页相对位置，而不是偏移量），有些时候这些记录指针称为Slots（槽）或目录槽（Directory Slots）。与其他数据库系统不同的是，在InnoDB中并不是每个记录拥有一个槽，而是一个槽中可能包含多个记录。B+树索引本身并不能找到具体的一条记录，能找到只是该记录所在的页。数据库把页载入到内存，然后通过Page Directory再进行二叉查找。只不过二叉查找的时间复杂度很低，同时在内存中的查找很快，因此通常忽略这部分查找所用的时间。

File Trailer只有一个FIL_PAGE_END_LSN部分，占用8字节。前4字节代表该页的checksum值，最后4字节和File Header中的FIL_PAGE_LSN相同。将这两个值与File Header中的FIL_PAGE_SPACE_OR_CHKSUM和FIL_PAGE_LSN值进行比较，看是否一致（checksum的比较需要通过InnoDB的checksum函数来进行比较，不是简单的等值比较），以此来保证页的完整性（not corrupted）。为了检测页是否已经完整地写入磁盘（如可能发生的写入过程中磁盘损坏、机器关机等）



**Named File Formates 机制，保证兼容性**

InnoDB存储引擎将1.0.x版本之前的文件格式（file format）定义为Antelope，将之后版本支持的文件格式定义为Barracuda。新的文件格式总是包含于之前的版本的页格式。下图显示了Barracuda文件格式和Antelope文件格式之间的关系，Antelope文件格式有Compact和Redudant的行格式，Barracuda文件格式既包括了Antelope所有的文件格式，另外新加入了之前已经提到过的Compressed和Dynamic行格式。

compressed可以使用zlib算法压缩，对于BLOB，text，varchar进行有效存储

![image-20221124164830330](./mysql/image-20221124164830330.png)



**行溢出**

一个页要至少存储两条记录（不然链表了），不然大数据存到BLOB Page。

varchar一行总和理论存储65535bytes，实际上65532. 



## 视图

一条SQL记录。

可以通过view来修改数据。对于一个10列的表，可以选中想展示的3条，来进行数据封装。

不支持物化视图，但是可以提前定时查询，插入一张**聚合表**。



## 分区

表容量较大的时候，分区后**查询**速度较快。插入效率可能影响。不在引擎层完成。

如MyISAM，INNODB

5.1版本后可以分区！！！

**水平分区**：一台服务器上（同行不同物理文件）

**垂直分区**：（同列不同物理文件）不太好用，如果选择了两个列在不同分区。



mysql调优的水平拆分

一个表变两个表，10条数据，5个在表1，5个在表2

垂直拆分：字段拆分，ID，name，age编程ID，name和ID，age。如果查询是分开查询，则拆分。如果需要join，考虑水平拆分。





**分区类型**：

假删除，删除标记，不真删。

需要整形或者转化为整形



range分区，主键范围，指定列的保存分区。null小于任何非null

list分区，奇数一部分，偶数一部分。指定列的保存分区。

hash分区，均匀分布，指定hash列和hash表达式。null->0

key分区，不用自己指定表达式

5.5之后可以不转成整形。

常见：

```sql
CREATE TABLE t columns_range(
a INT,
b DATETIME
)ENGINE=INNODB
PARTITION BY RANGE COLUMNS(B)
PARTITION p0 VALUES LESS THAN('2009-01-01'),
PARTITION p1 VALUES LESS THAN('2010-01-01');
```

子分区维护成本很大。复合分区。可以使用拆分+分区。

```sql
mysql> CREATE TABLE ts(a INT,b DATE)engine=innodb
-> PARTITION BY RANGE(YEAR(b))
-> SUBPARTITION BY HASHTO DAYS(b))
-> SUBPARTITIONS 2(
-> PARTITION PO VALUES LESS THAN(1990)
-> PARTITION P1 VALUES LESS THAN(2000)
-> PARTITION P2 VALUES LESS THAN MAXVALUE
-> );
```







## 约束



表建立时就进行约束定义，如：

利用ALTER TABLE命令来进行创建约束，对Unique Key（唯一索引）的约束

通过命令CREATE UNIQUE INDEX来建立。

对于主键约束而言，其默认约束名为PRIMARY。

而对于Unique Key约束而言，默认约束名和列名一样，当然也可以人为指定Unique Key约束的名字。

Foreign Key约束也是如此。

NOT NULL 字段的约束。

ENUM 和 SET 字段的约束。

触发器约束。 创建触发器的命令是CREATE TRIGGER，只有具备Super权限的MySQL数据库用户才可以执行这条命令。





# 五、索引｜重点

数据结构：B+树，hash table，Auxiliary table，R-tree index





**Oracle**：

![image-20221124182518552](./mysql/image-20221124182518552.png)



## A. B+树

索引太多，需要维护，创建，时间空间，修改，

太少则查询效率低。



不能直接找到记录，可以找到记录所在页，读入内存后，通过page_directory进行二分查找。

高扇出性。一般2～4层。机械磁盘100 I/O per second

聚集索引、辅助索引都是B+树

### 使用经验

已经知道数据库中存在两种类型的应用，OLTP和OLAP应用。在OLTP应用中，查询操作只从数据库中取得一小部分数据，一般可能都在10条记录以下，甚至在很多时候只取1条记录，如根据主键值来取得用户信息，根据订单号取得订单的详细信息，这都是典型OLTP应用的查询语句。在这种情况下，B+树索引建立后，对该索引的使用应该只是通过该索引取得表中少部分的数据。这时建立B+树索引才是有意义的，否则即使建立了，优化器也可能选择不使用索引。

对于OLAP应用，情况可能就稍显复杂了。不过概括来说，在OLAP应用中，都需要访问表中大量的数据，根据这些数据来产生查询的结果，这些查询多是面向分析的查询，目的是为决策者提供支持。如这个月每个用户的消费情况，销售额同比、环比增长的情况。因此在OLAP中索引的添加根据的应该是宏观的信息，而不是微观，因为最终要得到的结果是提供给决策者的。例如不需要在OLAP中对姓名字段进行索引，因为很少需要对单个用户进行查询。但是对于OLAP中的复杂查询，要涉及多张表之间的联接操作，因此索引的添加依然是有意义的。但是，如果联接操作使用的是Hash Join，那么索引可能又变得不是非常重要了，所以这需要DBA或开发人员认真并仔细地研究自己的应用。不过在OLAP应用中，通常会需要对时间字段进行索引，这是因为大多数统计需要根据时间维度来进行数据的筛选。





## B. 聚集索引

主键构造b+树

叶子结点是整行数据。

查询倾向采用聚集索引。因为可以在叶子结点直接找到数据。

对于范围查找a～d，先找到a，依次找到b，c，d





通过Recorder Header中最后的两个字节来判断下一条记录的位置，读取键值可得80 00 00 01，这就是主键为1的键值，80 00 00 01后的值00 00 00 04代表指向数据页的页号。同样的方式可以找到80 00 00 02和80 00 00 04这两个键值以及它们指向的数据页。通过以上对非数据页节点的分析，可以发现数据页上存放的是完整的每行的记录，而在非数据页的索引页中，存放的仅仅是键值及指向数据页的偏移量，而不是一个完整的行记录。因此这棵聚集索引树的构造大致如图：聚集索引的存储并不是物理上连续的，而是逻辑上连续的。这其中有两点：一是前面说过的页通过双向链表链接，页按照主键的顺序排序；另一点是每个页中的记录也是通过双向链表进行维护的，物理存储上可以同样不按照主键存储。



![image-20221124175051715](./mysql/image-20221124175051715.png)



## C. 辅助索引 

key index_name(cloum_name)

叶子结点：索引列value，和主键（为什么不直接存储位置？数据移动不需要修改辅助索引）。通过主键再去primary index查找。



那么重复的是什么数据结构？？？







## D. 索引管理

通过SHOW INDEX FROM table_name

![image-20221124191018694](./mysql/image-20221124191018694.png)

❑Table：索引所在的表名。

❑Non_unique：非唯一的索引，可以看到primary key是0，因为必须是唯一的

❑Key_name：索引的名字，用户可以通过这个名字来执行DROP INDEX

❑Seq_in_index：索引中该列的位置，如果看联合索引idx_a_c就比较直观了。

❑Column_name：索引列的名称。

❑Collation：列以什么方式存储在索引中。可以是NULL。B+树索引总是A，即排序的。如果使用了Heap存储引擎，并且建立了Hash索引，这里就会显示NULL了。因为Hash根据Hash桶存放索引数据，而不是对数据进行排序。

❑Cardinality：非常关键的值，表示索引中唯一值（即多少个不同的值）的数目的估计值。Cardinality表的行数应尽可能接近1，如果非常小，那么用户需要考虑是否可以删除此索引。

❑Sub_part：是否是列的部分被索引。如果看idx_b这个索引，这里显示100，表示只对b列的前100字符进行索引。如果索引整个列，则该字段为NULL。

❑Packed：关键字如何被压缩。如果没有被压缩，则为NULL。

❑Null：是否索引的列含有NULL值。可以看到idx_b这里为Yes，因为定义的列b允许NULL值。

❑Index_type：索引的类型。InnoDB存储引擎只支持B+树索引，所以这里显示的都是BTREE。Comment：注释。





### 1. FIC 快速索引创建



MySQL 5.5版本之前（不包括5.5）存在的一个普遍被人诟病的问题是MySQL数据库对于索引的添加或者删除的这类DDL操作，MySQL数据库的操作过程为：

❑首先创建一张新的临时表，表结构为通过命令ALTER TABLE新定义的结构（创建索引）。

❑然后把原表中数据导入到临时表。

❑接着删除原表。

❑最后把临时表重名为原来的表名。

可以发现，若用户对于一张大表进行索引的添加和删除操作，那么这会需要很长的时间。更关键的是，若有大量事务需要访问正在被修改的表，这意味着数据库服务不可用。

FIC优化了辅助索引的创建、删除

创建：表加上s锁，

删除：辅助索引操作就更简单了，InnoDB存储引擎只需更新内部视图，并将辅助索引的空间（内存）标记为可用（你随时使用这块空间，我现在这块空间没人使用了），同时删除MySQL数据库内部视图上对该表的索引定义即可。



### 2. OSC（online schema change）了解

Online Schema Change（在线架构改变，简称OSC）最早是由Facebook实现的一种在线执行DDL的方式，并广泛地应用于Facebook的MySQL数据库。所谓“在线”是指在事务的创建过程中，可以有读写事务对表进行操作，这提高了原有MySQL数据库在DDL操作时的并发性。Facebook采用PHP脚本来现实OSC，而并不是通过修改InnoDB存储引擎源码的方式。

实现OSC步骤如下：

❑init，即初始化阶段，会对创建的表做一些验证工作，如检查表是否有主键，是否存在触发器或者外键等。

❑createCopyTable，创建和原始表结构一样的新表。

❑alterCopyTable：对创建的新表进行ALTER TABLE操作，如添加索引或列等。

❑createDeltasTable，创建deltas表，该表的作用是为下一步创建的触发器所使用。之后对原表的所有DML操作会被记录到createDeltasTable中。

❑createTriggers，对原表创建INSERT、UPDATE、DELETE操作的触发器。触发操作产生的记录被写入到deltas表。

❑startSnpshotXact，开始OSC操作的事务。

❑selectTableIntoOutfile，将原表中的数据写入到新表。为了减少对原表的锁定时间，这里通过分片（chunked）将数据输出到多个外部文件，然后将外部文件的数据导入到copy表中。分片的大小可以指定，默认值是500 000。

❑dropNCIndexs，在导入到新表前，删除新表中所有的辅助索引。

❑loadCopyTable，将导出的分片文件导入到新表。

❑replayChanges，将OSC过程中原表DML操作的记录应用到新表中，这些记录被保存在deltas表中。

❑recreateNCIndexes，重新创建辅助索引。

❑replayChanges，再次进行DML日志的回放操作，这些日志是在上述创建辅助索引中过程中新产生的日志。

❑swapTables，将原表和新表交换名字，整个操作需要锁定2张表，不允许新的数据产生。由于改名是一个很快的操作，因此阻塞的时间非常短。上述只是简单介绍了OSC的实现过程，实际脚本非常复杂。由于OSC只是一个PHP脚本，因此其有一定的局限性。





### 3. online ddl

5.6开始

**实现原理**

InnoDB存储引擎实现Online DDL的原理是在执行创建或者删除操作的同时，将INSERT、UPDATE、DELETE这类DML操作日志写入到一个缓存中。待完成索引创建后再将重做应用到表上，以此达到数据的一致性。这个缓存的大小由参数innodb_online_alter_log_max_size控制，默认的大小为128MB。



允许辅助索引创建的同时，还允许其他诸如INSERT、UPDATE、DELETE这类DML操作

以下这几类DDL操作都可以通过“在线”的方式进行操作：

❑辅助索引的创建与删除

❑改变自增长值

❑添加或删除外键约束

❑列的重命名通过新的ALTER TABLE语法，

用户可以选择索引的创建方式：

![image-20221124203803356](./mysql/image-20221124203803356.png)

算法:

COPY表示按照MySQL 5.1版本之前的工作模式，即创建临时表的方式。

INPLACE表示索引创建或删除操作不需要创建临时表。

DEFAULT表示根据参数old_alter_table来判断是通过INPLACE还是COPY的算法，该参数的默认值为OFF，表示采用INPLACE的方式



LOCK

（1）NONE  执行索引创建或者删除操作时，对目标表不添加任何的锁，即事务仍然可以进行读写操作，不会收到阻塞。因此这种模式可以获得最大的并发度。

（2）SHARE  这和之前的FIC类似，执行索引创建或删除操作时，对目标表加上一个S锁。对于并发地读事务，依然可以执行，但是遇到写事务，就会发生等待操作。如果存储引擎不支持SHARE模式，会返回一个错误信息。

（3）EXCLUSIVE  在EXCLUSIVE模式下，执行索引创建或删除操作时，对目标表加上一个X锁。读写事务都不能进行，因此会阻塞所有的线程，这和COPY方式运行得到的状态类似，但是不需要像COPY方式那样创建一张临时表。

（4）DEFAULT  DEFAULT模式首先会判断当前操作是否可以使用NONE模式，若不能，则判断是否可以使用SHARE模式，最后判断是否可以使用EXCLUSIVE模式。也就是说DEFAULT会通过判断事务的最大并发性来判断执行DDL的模式。



操作日志写入缓存，待完成索引创建后再将重做应用到表上。这个缓存的大小由参数innodb_online_alter_log_max_size控制，默认的大小为128MB。

用户更新的表比较大，并且在创建过程中伴有大量的写事务，如遇到innodb_online_alter_log_max_size的空间不能存放日志时，会抛出类似如下的错误：Error:1799SQLSTATE:HY000(ER_INNODB_ONLINE_LOG_TOO_BIG)Message:Creating index'idx_aaa'required more than'innodb_online_alter_log_max_size'bytes of modification log.Please try again.对于这个错误，用户可以调大参数innodb_online_alter_log_max_size，以此获得更大的日志缓存空间。



此外，还可以设置ALTER TABLE的模式为SHARE，这样在执行过程中不会有写事务发生，因此不需要进行DML日志的记录。需要特别注意的是，由于Online DDL在创建索引完成后再通过重做日志达到数据库的最终一致性，这意味着在索引创建过程中，**SQL优化器不会选择正在创建中的索引**。





### Cardnality

抽样估计列中多少种不同的value

Cardnality/row # 尽可能等于1



更新：表中有1/16的数据发生变化 **OR** 修改次数stat_modified_counter

对8个叶子结点采样。

取得叶子结点总数量 A，随机取8个叶子结点，记录8个页中不同value 的个数 P

Cardnality～P*A/8





当执行SQL语句ANALYZE TABLE、SHOW TABLE STATUS、SHOW INDEX以及访问INFORMATION_SCHEMA架 构下的表TABLES和STATISTICS时会导致InnoDB存储引擎去重新计算索引的Cardinality值。若表中的数据量非 常大，并且表中存在多个辅助索引时，执行上述这些操作可能会非常慢。虽然用户可能并不希望去更新 Cardinality值。



些情况下可能会发生索引建立了却没有用到的情况，可能会出现Cardinality为NULL，致使优化器不选择 使用索引。这时最好的解决办法就是做一次ANALYZE TABLE的操作。因此在场景允许的情况下，建议在一个 非高峰时间，对应用程序下的几张核心表做ANALYZE TABLE操作，这能使优化器和索引更好地为你工作。





![image-20221125135429402](./mysql/image-20221125135429402.png)



## E. 联合索引

多列索引

对于(a, b)的联合索引，支持a，支持ab，不支持b

对于b，正逆排序都无需重新排序。



## F. 覆盖索引

其实是覆盖了所有projection的列。



## G. Multi-Range Read (MRR)

5.6开始

**目的**：减少磁盘随机访问，将随机访问转化成顺序访问

对于range，ref，eq_ref的查询类型，则可以优化。



**优势**：

顺序访问。使用辅助索引时候，得到查询结果set，对主键排序，回到聚簇索引搜索，减少IO。

减少缓冲页被替换的次数

批量处理



**工作原理**

对于select和join操作

辅助索引键值放到缓存中，此时是按照辅助索引键值排序

按照主键排序

根据主键访问。



**参数**

启用通过参数optimizer_switch中的标记（flag）来控制。当mrr为on时，表示启用Multi-Range Read优化。

mrr_cost_based标记表示是否通过cost based的方式来选择是否启用mrr。若将mrr设为on，mrr_cost_based设为off，则总是启用Multi-Range Read优化。

例如，下述语句可以将Multi-Range Read优化总是设为开启状态：mysql＞SET@@optimizer_switch='mrr=on,mrr_cost_based=off';

## H. ICP 优化 index condition pushdown

5.6开始

作用于condition上。将where操作放在了存储引擎层。

减少了上层sql对于记录的索取



支持range、ref、eq_ref、ref_or_null类型的查询，当前支持MyISAM和InnoDB存储引擎。当优化器选择Index Condition Pushdown优化时，可在执行计划的列Extra看到Using index condition提示

**example**

![image-20221125142936791](./mysql/image-20221125142936791.png)



## I. Hash算法

通过时间复杂度为O(1)来找到指定内存页。

对字典进行查找。

缓冲池的hash 表来说，池中每个page都有一个chain指针指向相同hash value的页。m应略大于2* pages # in buffer pool的质数



K = 表空间space_id << 20 + offset+space_id



k%m





自适应哈希索引是数据库自身创建并且使用的，dba不能干预。





## J. 全文检索 1.2.x

倒排索引（inverted index）

它在辅助表（auxiliary table）中存储了单词与单词自身在一个或多个文档中所在位置之间的映射。这通常利用关联数组实现，其拥有两种表现形式：

❑inverted file index，其表现形式为{单词，单词所在文档的ID}

❑full inverted index，**MySQL**，其表现形式为{单词，(单词所在文档的ID，在具体文档中的位置)}

![image-20221125144602913](./mysql/image-20221125144602913.png)



事务提交->分词写入FTS Index Cache（红黑树）->类似插入缓冲原理，到Auxiliary Table

如果宕机，下次全文检索的时候重新读取文档，分词



删除时候，只删除chach记录，不删除磁盘记录。将删除记录的FTS Document ID存在DELETED auxiliary table



OPTIMIZE TABLE 会优化很多，比如cardinality统计

“mysql＞SET GLOBAL innodb_optimize_fulltext_only=1;




mysql＞OPTIMIZE TABLEfts_a;”

![WX20230811-195546](./mysql/WX20230811-195546.png)





# 六、锁｜重点

表锁行锁

LRU锁->线程锁｜主键ID自增->共享资源，上锁

latch，针对线程，时间短，分为mutex（互斥量）和rwlock（读写锁），无死锁检测

lock，对表，行，数据，有死锁检测



![image-20221125153623406](./mysql/image-20221125153623406.png)





## Innodb存储引擎中的锁

行级锁：

❑共享锁（S Lock），允许事务读一行数据。

❑排他锁（X Lock），允许事务删除或更新一行数据。



意向锁，表级锁，先意向锁，再行级锁。不会阻塞除全表扫描意外的任何请求

意义：当select * from table, 确保所有的行都没有被加x锁，但是为了快速确定，是否存在行级锁，可以直接查看IX lock

多粒度锁定，支持行级锁和表级锁同时存在。



在INFORMATION_SCHEMA架构下添加了表INNODB_TRX、INNODB_LOCKS、INNODB_LOCK_WAITS



## MVCC

为什么？并发

非锁定读：读快照，即从undo日志完成。



在事务隔离级别READ COMMITTED和REPEATABLE READ（InnoDB存储引擎的默认事务隔离级别）下，InnoDB存储引擎使用非锁定的一致性读。然而，对于快照数据的定义却不相同。在READ COMMITTED事务隔离级别下，对于快照数据，非一致性读总是读取被锁定行的最新一份快照数据。而在REPEATABLE READ事务隔离级别下，对于快照数据，非一致性读总是读取事务开始时的行数据版本。

锁定读

❑SELECT…FOR UPDATE

❑SELECT…LOCK IN SHARE MODE



## 自增长与锁

https://www.cnblogs.com/mengxinJ/p/14352038.html#_label1

AUTO-INC Locking，表锁机制，完成自增长value插入释放。

5.1.22提供latch，提供轻量级互斥量mutex，提高性能

innodb_autoinc_lock_mode：

0-> AUTO-INC Locking

1-> simple insert 则0， bulk insert（知道插入数量） 则用mutex，对内存中的计数器做累加操作，使用statement-based 复制。

2-> 只用互斥量mutex，但是可能产生不连续的情况，，主从复制过程中出现id不一致，所以要使用row-based 复制

## 锁算法

InnoDB存储引擎有3种行锁的算法，其分别是：

❑Record Lock：单个行记录上的锁

❑Gap Lock：间隙锁，锁定一个范围，但不包含记录本身

❑Next-Key Lock∶Gap Lock+Record Lock，锁定一个范围，并且锁定记录本身。其设计的目的是为了支持一些不需要使用幻读和不可重复读的场景。

Next-Key Lock->record lock,索引value唯一的时候进行降级



**example**

![image-20221125163928225](./mysql/image-20221125163928225.png)



## 四种读

### 脏读

只在读未提交下，读到了未提交的数据。

### 幻读

只在读提交，读到了新的tuple，结果数量不一致。

可以接受

Innodb 中通过next key lock来解决

### 不可重复读

只在读提交，读到了新的colum，结果value不一致。

可以接受

Innodb 中通过next key lock来解决



## 死锁

两种机制检测：

超时检测

时间过长则导致死锁存在时间较长，但是判断精准

时间短则死锁存在时间短，但是可能错杀。

timeout本身也是有等待的

且要考虑

The cost of aborting a transaction o All updates must be undone (recall the “A” in ACID)

How long a transaction has been running?

How long it will take a transaction to finish?

How many deadlocks will be resolved if a particular transaction is aborted o Is the transaction in more than one cycle?

How many times this transaction was already aborted due to deadlocks?

innodb_lock_wait_timeout用来控制等待的时间（默认是50秒），innodb_rollback_on_timeout用来设定是否在等待超时时对进行中的事务进行回滚操作（默认是OFF





死锁检测：

WFG

锁信息链表

事务等待链表

检测到后，选择回滚undo量最小的事务。







# 七、事务｜重点

事务特性ACID

redo log称为重做日志，用来保证事务的原子性和持久性。

undo log用来保证事务的一致性。（还有mvcc和回滚）

锁用来保证事物的隔离性。



## A. redo log与bin log

redo重做

innodb，redo log buffer + redo log file

通过force log at commit，

事务提交前保证刷入磁盘（顺序写，无序读，fsync操作）->事务提交

innodb_flush_log_at_trx_commit，默认1，fsync per commit

0->master thread 的每秒操作

2->仅写入文件系统的缓存，不调用fsync。dbs宕机，os不宕机，则可以恢复。



可以批量提交减少fsync次数



redo innodb|bin log MySQL上层

redo log 物理页信息，

binlog，POINT-IN-TIME恢复，主从复制



redo log 物理页变化，对页的修改｜事务进行中不断的被写入

binlog 逻辑日志，sql|提交后写入



## B. redo log 结构

log block 512bytes per block

磁盘扇区大小 512 bytes 原子写，不需要double write



log block header  12bytes

log body 				492bytes

log block tailer 	 8bytes



log buffer管理log block，

![image-20221125175912175](./mysql/image-20221125175912175.png)



InnoDB 1.2版本之前，重做日志文件的总大小要小于4GB（不能等于4GB）

InnoDB 1.2版本开始重做日志文件总大小的限制提高为了512GB。



重做日志格式

![image-20221125180431518](./mysql/image-20221125180431518.png)

**LSN**

LSN，写入日志的字节数，标记了重做日志写入总量，checkpoint位置，页的版本。

**恢复**

无论是否正常，都尝试进行恢复。因为是物理日志，所以比binlog快。以及顺序读和并行重做。从checkpoint后开始恢复





## C. undo log

回滚/MVCC

请求回滚 OR 事务失败

purge thread

redo存放在重做日志文件中，与redo不同，undo存放在数据库内部的一个特殊段（segment）中，这个段称为undo段（undo segment）。undo段位于共享表空间内。



逻辑日志。没办法物理日志，因为页的其他内容发生了变化。



增删改：undo log与redo log同时产生，回滚了之后，rodo log 也要记录回滚





## D. undo log 结构

InnoDB存储引擎有rollback segment，每个回滚段种记录了1024个undo log segment，而在每个undo log segment段中进行undo页的申请。





undo log结构在InnoDB存储引擎中，undo log分为：

❑insert undo log 指在insert操作中产生的undo log。

❑update undo log 记录的是对delete和update操作产生的undo log。该undo log可能需要提供MVCC机制，因此不能在事务提交时就进行删除。



## E. Purge

用于最终完成update 和delete操作



因为支持mvcc，所以如果删除了一个数据，则对于快照版本不复存在。

至于是否可以删除则通过Purge删除，如果没有被其他事物引用，则删除。



InnoDB存储引擎有一个history列表，它根据事务提交的顺序，将undo log进行链接

![image-20221125183622836](./mysql/image-20221125183622836.png)



通过检索需要删除的undo log 的所在页的其他内容，可以减少IO次数



## F. Group commit

不同于批量提交，一次提交100个插入记录，通过用户语言。

而group commit是指一次fsync刷新确保多个事务被刷新



InnoDB存储引擎来说，事务提交时会进行两个阶段的操作：

1）修改内存中事务对应的信息，并且将日志写入重做日志缓冲。

2）调用fsync将确保日志都从重做日志缓冲写入磁盘。



1.2之前，开启binlog后group commit失效

1）当事务提交时InnoDB存储引擎进行prepare操作。

2）MySQL数据库上层写入二进制日志。（当前可以确保事务提交）

3）InnoDB存储引擎层将日志写入重做日志文件。	

​	a）修改内存中事务对应的信息，并且将日志写入重做日志缓冲。	

​	b）调用fsync将确保日志都从重做日志缓冲写入磁盘。

MySQL数据库内部使用了prepare_commit_mutex这个锁。但是在启用这个锁之后，步骤3）a）步不可以在其他事务执行步骤b）时进行，从而导致了group commit失效。



换而言之，Group commit只针对redo log，而binlog没有group commit

5.6 BLGC（bin log GC）

![image-20221125184808846](./mysql/image-20221125184808846.png)



在MySQL数据库上层进行提交时首先按顺序将其放入一个队列中，队列中的第一个事务称为leader，其他事务称为follower，leader控制着follower的行为。

❑Flush阶段，将每个事务的二进制日志写入内存中。

❑Sync阶段，将内存中的二进制日志刷新到磁盘，若队列中有多个事务，那么仅一次fsync操作就完成了二进制日志的写入，这就是BLGC。

❑Commit阶段，leader根据顺序调用存储引擎层事务的提交，InnoDB存储引擎本就支持group commit，因此修复了原先由于锁prepare_commit_mutex导致group commit失效的问题。



## G. 事务的隔离级别

SQL标准定义的四个隔离级别为：

❑READ UNCOMMITTED

❑READ COMMITTED

❑REPEATABLE READ

❑SERIALIZABLE

隔离级别越低，事务请求的锁越少或保持锁的时间就越短。

❑在SERIALIABLE的事务隔离级别，InnoDB存储引擎会对每个SELECT语句后自动加上LOCK IN SHARE MODE，即为每个读取操作加一个共享锁。因此在这个事务隔离级别下，读占用了锁，对一致性的非锁定读不再予以支持。

❑InnoDB存储引擎默认支持的隔离级别是REPEATABLE READ，但是与标准SQL不同的是，InnoDB存储引擎在REPEATABLE READ事务隔离级别下，使用Next-Key Lock锁的算法，读事务开始前的版本

❑在READ COMMITTED的事务隔离级别下，除了唯一性的约束检查及外键约束的检查需要gap lock，InnoDB存储引擎不会使用gap lock的锁算法。MVCC 控制读取最新版本快照



## H. 分布式事务

分布式事就是要在分布式系统中实现事务，InnoDB存储引擎提供了对XA事务的支持，并通过XA事务来支持分布式事务的实现。

分布式事务指的是允许多个独立的事务资源（transactional resources）参与到一个全局的事务中。全局事务要求在其中的所有参与的事务要么都提交，要么都回滚，这对于事务原有的ACID要求又有了提高。



XA事务

一个或多个资源管理器（Resource Managers）

一个事务管理器（Transaction Manager）

一个应用程序（Application Program）组成。

❑资源管理器：提供访问事务资源的方法。通常一个数据库就是一个资源管理器。

❑事务管理器：协调参与全局事务中的各个事务。需要和参与全局事务的所有资源管理器进行通信。

❑应用程序：定义事务的边界，指定全局事务中的操作。在MySQL数据库的分布式事务中，资源管理器就是MySQL数据库，事务管理器为连接MySQL服务器的客户端。







2PC使用两段式提交（two-phase commit）的方式。

在第一阶段，所有参与全局事务的节点都开始准备（PREPARE），告诉事务管理器它们准备好提交了。

在第二阶段，事务管理器告诉资源管理器执行ROLLBACK还是COMMIT。如果任何一个节点显示不能提交，则所有的节点都被告知需要回滚。



TCC
TCC 是 Try、Confirm、Cancel 三个词语的缩写，TCC 要求每个分支事务实现三个操作：预处理 Try、确认 Confirm、撤销 Cancel。Try 操作做业务检查及资源预留，Confirm 做业务确认操作，Cancel 实现一个与 Try 相反的操作即回滚操作。



## I. 内部事务

最为常见的内部XA事务存在于binlog与InnoDB存储引擎之间。由于复制的需要，因此目前绝大多数的数据库都开启了binlog功能。在事务提交时，先写二进制日志，再写InnoDB存储引擎的重做日志。对上述两个操作的要求也是原子的，即二进制日志和重做日志必须同时写入。若二进制日志先写了，而在写入InnoDB存储引擎时发生了宕机，那么slave可能会接收到master传过去的二进制日志并执行，最终导致了主从不一致的情况。



为了解决这个问题，MySQL数据库在binlog与InnoDB存储引擎之间采用XA事务。当事务提交时，InnoDB存储引擎会先做一个PREPARE操作，将事务的xid写入，接着进行二进制日志的写入。如果在InnoDB存储引擎提交前，MySQL数据库宕机了，那么MySQL数据库在重启后会先检查准备的UXID事务是否已经提交，若没有，则在存储引擎层再进行一次提交操作。





# 八、数据备份

**备份方法划分**

热备份

数据库运行中直接备份，

ibbackup，官方提供。

1）记录备份开始时，InnoDB存储引擎重做日志文件检查点的LSN。

2）复制共享表空间文件以及独立表空间文件。

3）记录复制完表空间文件后，InnoDB存储引擎重做日志文件检查点的LSN。

4）复制在备份时产生的重做日志。



优点：

❑在线备份，不阻塞任何的SQL语句。

❑备份性能好，备份的实质是复制数据库文件和重做日志文件。

❑支持压缩备份，通过选项，可以支持不同级别的压缩。

❑跨平台支持，ibbackup可以运行在Linux、Windows以及主流的UNIX系统平台上。



ibbackup对InnoDB存储引擎表的恢复步骤为：

❑恢复表空间文件。

❑应用重做日志文件。

ibbackup提供了一种高性能的热备方式，是InnoDB存储引擎备份的首选方式。不过它是收费软件，并非免费的软件。



xtrabackup

MySQL数据库本身提供的工具并不支持真正的增量备份，更准确地说，二进制日志的恢复应该是point-in-time的恢复而不是增量备份。而XtraBackup工具支持对于InnoDB存储引擎的增量备份，其工作原理如下：1）首选完成一个全备，并记录下此时检查点的LSN。2）在进行增量备份时，比较表空间中每个页的LSN是否大于上次备份时的LSN，如果是，则备份该页，同时记录当前检查点的LSN。



冷备份

数据库停止的情况下，复制物理文件即可

温备份

加读锁进行备份



备份MySQL数据库的frm文件，共享表空间文件，独立表空间文件（*.ibd），重做日志文件。另外建议定期备份MySQL数据库的配置文件my.cnf，这样有利于恢复的操作。

冷备的优点是：

❑备份简单，只要复制相关文件即可。

❑备份文件易于在不同操作系统，不同MySQL版本上进行恢复。

❑恢复相当简单，只需要把文件恢复到指定位置即可。

❑恢复速度快，不需要执行任何SQL语句，也不需要重建索引。

冷备的缺点是：

❑InnoDB存储引擎冷备的文件通常比逻辑文件大很多，因为表空间中存放着很多其他的数据，如undo段，插入缓冲等信息。

❑冷备也不总是可以轻易地跨平台。操作系统、MySQL的版本、文件大小写敏感和浮点数格式都会成为问题。



**备份内容**

逻辑备份

备份文件可读，一般由sql语句组成或者是表内实际数据组成

裸文件备份（热备份）直接复制物理文件，即可在运行中复制，也可以在停止时复制（用ibbackup、xtrabackup）。恢复时间比逻辑备份短



mysqldumpmysqldump的语法如下：

```sql
shell＞mysqldump[arguments]＞fle_name如果想要备份所有的数据库，可以使用--all-databases选项：shell＞mysqldump--all-databases＞dump.sql如果想要备份指定的数据库，可以使用--databases选项:shell＞mysqldump--databases db1 db2 db3＞dump.sql2. 　SELECT...INTO OUTFILESELECT...INTO语句也是一种逻辑备份的方法，更准确地说是导出一张表中的数据。SELECT...INTO的语法如下：SELECT[column 1],[column 2]...INTOOUTFILE'file_name'[{FIELDS|COLUMNS}[TERMINATED BY'string'][[OPTIONALLY]ENCLOSED BY'char'][ESCAPED BY'char']][LINES[STARTING BY'string'][TERMINATED BY'string']]FROM TABLE WHERE......
```









数据库内容

完全备份

增量备份

日志备份

## MySQL主从同步



## 复制过程

复制（replication）是MySQL数据库提供的一种高可用高性能的解决方案，一般用来建立大型的应用。总体来说，replication的工作原理分为以下3个步骤：

1）主服务器（master）把数据更改记录到二进制日志（binlog）中。

2）从服务器（slave）把主服务器的二进制日志复制到自己的中继日志（relay log）中。

3）从服务器重做中继日志中的日志，把更改应用到自己的数据库上，以达到数据的最终一致性。

复制的工作原理并不复杂，其实就是一个完全备份加上二进制日志备份的还原。从服务器有2个线程，一个是I/O线程，负责读取主服务器的二进制日志，并将其保存为中继日志；另一个是SQL线程，复制执行中继日志。

![image-20221128153639534](./mysql/image-20221128153639534.png)



![image-20221128153702815](/Users/shengquan/Library/Application Support/typora-user-images/image-20221128153702815.png)





# 九、性能调优｜重点

上游调优：添加代理层，数据库连接池，如hikari druid c3p0 dbcp jdbc

**单表调优。** 

索引特性，存储结构，什么情况下使用，cardinility，使用explain，（extre可以展示MRR）

一个页至少放2条，高度，

多插入的调优。（批量插入；id自增； group commit）

多查询调优。索引，explain（），B+树（覆盖索引特性多理解，回表），严格约束字段的长度，能用数字就用数字，尤其是tinyint。（男女，0,1）（like，join）尽量用timestamp不要用data

不用*，无法避免走primary index，一定要用的话，用5.6，提供离散读和icp

覆盖索引，

查看执行计划



分区 ，更快的**查询速度**。

拆分表（水平拆分 ，需要带id进行查询、 垂直拆分=减少单表的字段量，单独业务查询）



**版本调优**

MRR（5.5->5.6，需要开启）

ICP



**服务器层面调优**

测试（非压测环境，非预生产环境）环境开启慢查询日志。进行日常 、 上线前的测试。分析慢查询sql。

CPU / 磁盘 / 内存 / RAID





**分布式调优**



**通过外置中间件调优**

缓存（减少访问） 、 queue（削峰）、分布式、读写分离





表设计->内存页大小相关->b+树

索引设计

分区1024个分区和拆分。

拆分到不同的服务器，一致性hash。一个查询要多个mysql。用哪一个key来避免查询多个数据库。







时间判断：set profiling = 1;

show profiles

show profile for query #；



show processlist











### **SQL调优（单机）**



这是最常用、每一个技术人员都应该掌握基本的SQL调优手段（包括方法、工具、辅助系统等）。这里以MySQL为例，最常见的方式是，由自带的慢查询日志或者开源的慢查询系统定位到具体的出问题的SQL，然后使用explain、profile等工具来逐步调优，最后经过测试达到效果后上线。这方面的细节，可以参考[MySQL索引原理及慢查询优化](http://tech.meituan.com/mysql-index.html)。

###  

### **架构层面的调优（分布式）**



这一类调优包括读写分离、多从库负载均衡、水平和垂直分库分表等方面，一般需要的改动较大，但是频率没有SQL调优高，而且一般需要DBA来配合参与。那么什么时候需要做这些事情？我们可以通过内部监控报警系统（比如Zabbix），定期跟踪一些指标数据是否达到瓶颈，一旦达到瓶颈或者警戒值，就需要考虑这些事情。通常，DBA也会定期监控这些指标值。

###  

### **连接池调优**



我们的应用为了实现数据库连接的高效获取、对数据库连接的限流等目的，通常会采用连接池类的方案，即每一个应用节点都管理了一个到各个数据库的连接池。随着业务访问量或者数据量的增长，原有的连接池参数可能不能很好地满足需求，这个时候就需要结合当前使用连接池的原理、具体的连接池监控数据和当前的业务量作一个综合的判断，通过反复的几次调试得到最终的调优参数。



如何保证事务结束后，对数据的修改永久的保存？

方案1.事务提交前页面写盘

方案2. wal





![IMG_A57C25321366](./mysql/IMG_A57C25321366.jpeg)









如果存在内存上是可以用btree的，PMEM，断电保存







druid





# 十、链接资源

[基于代价的慢查询优化建议](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5NjQ5MTI5OA==&action=getalbum&album_id=2364684582387892225&scene=173&from_msgid=2651767904&from_itemidx=1&count=3&nolastread=1#wechat_redirect)





Disadvantages of MMAP:

- Not all operating systems support it (*ahem* Windows)
- Coarse locking. It's difficult to allow many clients to make concurrent access to the file.
- Relying on the OS to buffer I/O writes leads to increased risk of data loss if the RDBMS engine crashes. Need to use a journaling filesystem, which may not be supported on all operating systems.
- Can only map a file size up to the size of the virtual memory address space, so on 32-bit OS, the database files are limited to 4GB (per comment from Roger Lipscombe above).

Early versions of MongoDB tried to use MMAP in the primary storage engine (the only storage engine in the earliest MongoDB). Since then, they have introduced other storage engines, notably WiredTiger. This has greater support for tuning, better performance on multicore systems, support for encryption and compression, multi-document transactions, and so on.









# 十一、SQL题目

`group by class having count(student) >= 5`

