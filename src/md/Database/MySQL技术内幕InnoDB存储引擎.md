# 第1章　MySQL体系结构和存储引擎

数据库（database）和实例（instance）。

需要特别注意的是，存储引擎是基于表的，而不是数据库。

MySQL组件：

![image-20191212165914908](E:\studydyup\notes\src\pic\image-20191212165914908.png)

❑连接池组件
❑管理服务和工具组件
❑SQL接口组件 

❑查询分析器组件
❑优化器组件
❑缓冲（Cache）组件
❑插件式存储引擎
❑物理文件

## InnoDB

InnoDB存储引擎支持事务，其设计目标主要面向在线事务处理（OLTP）的应用。

InnoDB通过使用多版本并发控制（MVCC）来获得高并发性，并且实现了SQL标准的4种隔离级别，默认为REPEATABLE级别。

同时，使用一种被称为next-key locking的策略来避免幻读（phantom）现象的产生。

除此之外，InnoDB储存引擎还提供了插入缓冲（insert buffer）、二次写（double write）、自适应哈希索引（adaptive hash index）、预读（read ahead）等高性能和高可用的功能。

对于表中数据的存储，InnoDB存储引擎采用了聚集（clustered）的方式，因此每张表的存储都是按主键的顺序进行存放。如果没有显式地在表定义时指定主键，InnoDB存储引擎会为每一行生成一个6字节的ROWID，并以此作为主键。

## 连接MySQL

### TCP/IP

```
> mysql-h192.168.0.101-u david-p
```



# 第2章　InnoDB存储引擎

InnoDB是事务安全的MySQL存储引擎，InnoDB存储引擎是OLTP应用中核心表的首选存储引擎。

## 2.1　InnoDB存储引擎概述

是第一个完整支持ACID事务的MySQL存储引擎，其特点是行锁设计、支持MVCC、支持外键、提供一致性非锁定读，同时被设计用来最有效地利用以及使用内存和CPU。

## 2.2　InnoDB存储引擎的版本

## 2.3　InnoDB体系架构

![image-20191212172004130](E:\studydyup\notes\src\pic\image-20191212172004130.png)

上图简单显示了InnoDB的存储引擎的体系架构，从图可见，InnoDB存储引擎有多个内存块，可以认为这些内存块组成了一个大的内存池，负责如下工作：

❑维护所有进程/线程需要访问的多个内部数据结构。
❑缓存磁盘上的数据，方便快速地读取，同时在对磁盘文件的数据修改之前在这里缓存。
❑重做日志（redo log）缓冲。

后台线程的主要作用是负责刷新内存池中的数据，保证缓冲池中的内存缓存的是最近的数据。此外将已修改的数据文件刷新到磁盘文件，同时保证在数据库发生异常的情况下InnoDB能恢复到正常运行状态。

### 2.3.1　后台线程

InnoDB存储引擎是多线程的模型，因此其后台有多个不同的后台线程，负责处理不同的任务。

1. Master Thread

Master Thread是一个非常核心的后台线程，主要负责将缓冲池中的数据异步刷新到磁盘，保证数据的一致性，包括脏页的刷新、合并插入缓冲（INSERT BUFFER）、UNDO页的回收等。

2. IO Thread

在InnoDB存储引擎中大量使用了AIO（Async IO）来处理写IO请求，这样可以极大提高数据库的性能。

3. Purge Thread

事务被提交后，其所使用的undo log可能不再需要，因此需要PurgeThread来回收已经使用并分配的undo页。

```
SHOW VARIABLES LIKE'innodb_purge_threads';
```

### 2.3.2 内存

#### 1. 缓冲池

InnoDB存储引擎是基于磁盘存储的，并将其中的记录按照页的方式进行管理。因此可将其视为基于磁盘的数据库系统（Disk-base Database）。在数据库系统中，由于CPU速度与磁盘速度之间的鸿沟，基于磁盘的数据库系统通常使用缓冲池技术来提高数据库的整体性能。

缓冲池简单来说就是一块内存区域，通过内存的速度来弥补磁盘速度较慢对数据库性能的影响。在数据库中进行读取页的操作，首先将从磁盘读到的页存放在缓冲池中，这个过程称为将页“FIX”在缓冲池中。下一次再读相同的页时，首先判断该页是否在缓冲池中。若在缓冲池中，称该页在缓冲池中被命中，直接读取该页。否则，读取磁盘上的页。

对于数据库中页的修改操作，则首先修改在缓冲池中的页，然后再以一定的频率刷新到磁盘上。这里需要注意的是，页从缓冲池刷新回磁盘的操作并不是在每次页发生更新时触发，而是通过一种称为Checkpoint的机制刷新回磁盘。同样，这也是为了提高数据库的整体性能。

综上所述，缓冲池的大小直接影响着数据库的整体性能。

对于InnoDB存储引擎而言，其缓冲池的配置通过参数innodb_buffer_pool_size来设置。

```mysql
SHOW VARIABLES LIKE'innodb_buffer_pool_size'
```

具体来看，缓冲池中缓存的数据页类型有：索引页、数据页、undo页、插入缓冲（insert buffer）、自适应哈希索引（adaptive hash index）、InnoDB存储的锁信息（lock info）、数据字典信息（data dictionary）等。

不能简单地认为，缓冲池只是缓存索引页和数据页，它们只是占缓冲池很大的一部分而已。

![image-20191212173048101](E:\studydyup\notes\src\pic\image-20191212173048101.png)

从InnoDB 1.0.x版本开始，允许有多个缓冲池实例。每个页根据哈希值平均分配到不同缓冲池实例中。这样做的好处是减少数据库内部的资源竞争，增加数据库的并发处理能力。可以通过参数innodb_buffer_pool_instances来进行配置，该值默认为1。

```mysql
SHOW VARIABLES LIKE'innodb_buffer_pool_instances'
```

```mysql
SHOW ENGINE INNODB STATUS
```

```mysql
SELECT POOL_ID,POOL_SIZE,FREE_BUFFERS,DATABASE_PAGES FROM INNODB_BUFFER_POOL_STATS;
```

#### 2. LRU List、Free List和Flush List

通常来说，数据库中的缓冲池是通过LRU（Latest Recent Used，最近最少使用）算法来进行管理的。最频繁使用的页在LRU列表的前端，而最少使用的页在LRU列表的尾端。当缓冲池不能存放新读取到的页时，将首先释放LRU列表中尾端的页。

在InnoDB存储引擎中，缓冲池中页的大小默认为16KB，同样使用LRU算法对缓冲池进行管理。稍有不同的是InnoDB存储引擎对传统的LRU算法做了一些优化。

在InnoDB的存储引擎中，LRU列表中还加入了midpoint位置。新读取到的页，虽然是最新访问的页，但并不是直接放入到LRU列表的首部，而是放入到LRU列表的midpoint位置。这个算法在InnoDB存储引擎下称为midpoint insertion strategy。



在LRU列表中的页被修改后，称该页为脏页（dirty page），即缓冲池中的页和磁盘上的页的数据产生了不一致。这时数据库会通过CHECKPOINT机制将脏页刷新回磁盘，而Flush列表中的页即为脏页列表。需要注意的是，脏页既存在于LRU列表中，也存在于Flush列表中。LRU列表用来管理缓冲池中页的可用性，Flush列表用来管理将页刷新回磁盘，二者互不影响。

#### 3. 3.重做日志缓冲

InnoDB存储引擎的内存区域除了有缓冲池外，还有重做日志缓冲（redo log buffer）。

InnoDB存储引擎首先将重做日志信息先放入到这个缓冲区，然后按一定频率将其刷新到重做日志文件。

重做日志缓冲一般不需要设置得很大，因为一般情况下每一秒钟会将重做日志缓冲刷新到日志文件，因此用户只需要保证每秒产生的事务量在这个缓冲大小之内即可。

该值可由配置参数innodb_log_buffer_size控制，默认为8MB：

```mysql
SHOW VARIABLES LIKE'innodb_log_buffer_size'
```

