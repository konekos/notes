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

#### 3. 重做日志缓冲

InnoDB存储引擎的内存区域除了有缓冲池外，还有重做日志缓冲（redo log buffer）。

InnoDB存储引擎首先将重做日志信息先放入到这个缓冲区，然后按一定频率将其刷新到重做日志文件。

重做日志缓冲一般不需要设置得很大，因为一般情况下每一秒钟会将重做日志缓冲刷新到日志文件，因此用户只需要保证每秒产生的事务量在这个缓冲大小之内即可。

该值可由配置参数innodb_log_buffer_size控制，默认为8MB：

```mysql
SHOW VARIABLES LIKE'innodb_log_buffer_size'
```

#### 4. 额外的内存池

在InnoDB存储引擎中，对内存的管理是通过一种称为内存堆（heap）的方式进行的。在对一些数据结构本身的内存进行分配时，需要从额外的内存池中进行申请，当该区域的内存不够时，会从缓冲池中进行申请。

## 2.4 CheckPoint技术

​	前面已经讲到了，缓冲池的设计目的为了协调CPU速度与磁盘速度的鸿沟。因此页的操作首先都是在缓冲池中完成的。如果一条DML语句，如Update或Delete改变了页中的记录，那么此时页是脏的，即缓冲池中的页的版本要比磁盘的新。数据库需要将新版本的页从缓冲池刷新到磁盘。

​	倘若每次一个页发生变化，就将新页的版本刷新到磁盘，那么这个开销是非常大的。若热点数据集中在某几个页中，那么数据库的性能将变得非常差。**同时，如果在从缓冲池将页的新版本刷新到磁盘时发生了宕机，那么数据就不能恢复了。为了避免发生数据丢失的问题，当前事务数据库系统普遍都采用了Write Ahead Log策略，即当事务提交时，先写重做日志，再修改页。当由于发生宕机而导致数据丢失时，通过重做日志来完成数据的恢复**。这也是事务ACID中D（Durability持久性）的要求。

如果重做日志可以无限地增大，同时缓冲池也足够大，能够缓冲所有数据库的数据，那么是不需要将缓冲池中页的新版本刷新回磁盘。因为当发生宕机时，完全可以通过重做日志来恢复整个数据库系统中的数据到宕机发生的时刻。但是这需要两个前提条件：
❑缓冲池可以缓存数据库中所有的数据；
❑重做日志可以无限增大。

还有一个情况需要考虑：**宕机后数据库的恢复时间。**当数据库运行了几个月甚至几年时，这时发生宕机，重新应用重做日志的时间会非常久，此时恢复的代价也会非常大。

因此Checkpoint（检查点）技术的目的是解决以下几个问题：

缩短数据库的恢复时间；
❑缓冲池不够用时，将脏页刷新到磁盘；
❑重做日志不可用时，刷新脏页。

当数据库发生宕机时，数据库不需要重做所有的日志，因为Checkpoint之前的页都已经刷新回磁盘。故数据库只需对Checkpoint后的重做日志进行恢复。这样就大大缩短了恢复的时间。

当缓冲池不够用时，根据LRU算法会溢出最近最少使用的页，若此页为脏页，那么需要强制执行Checkpoint，将脏页也就是页的新版本刷回磁盘。

对于InnoDB存储引擎而言，其是通过LSN（Log Sequence Number）来标记版本的。而LSN是8字节的数字，其单位是字节。每个页有LSN，重做日志中也有LSN，Checkpoint也有LSN。



## 2.5 Master Thread工作方式

## 2.6 InnoDB关键特性

InnoDB存储引擎的关键特性包括：
❑插入缓冲（Insert Buffer）
❑两次写（Double Write）
❑自适应哈希索引（Adaptive Hash Index）
❑异步IO（Async IO）
❑刷新邻接页（Flush Neighbor Page）
上述这些特性为InnoDB存储引擎带来更好的性能以及更高的可靠性。

### 2.6.1 插入缓冲

#### 1. Insert Buffer

这个名字可能会让人认为插入缓冲是缓冲池中的一个组成部分。其实不然，InnoDB缓冲池中有Insert Buffer信息固然不错，但是Insert Buffer和数据页一样，也是物理页的一个组成部分。

在InnoDB存储引擎中，主键是行唯一的标识符。通常应用程序中行记录的插入顺序是按照主键递增的顺序进行插入的。因此，插入聚集索引（Primary Key）一般是顺序的，不需要磁盘的随机读取。

通常应用程序中行记录的插入顺序是按照主键递增的顺序进行插入的。因此，**插入聚集索引（Primary Key）一般是顺序的，不需要磁盘的随机读取**。

并不是所有的主键插入都是顺序的。若主键类是UUID这样的类，那么插入和辅助索引一样，同样是随机的。即使主键是自增类型，但是插入的是指定的值，而不是NULL值，那么同样可能导致插入并非连续的情况。

但是不可能每张表上只有一个聚集索引，更多情况下，一张表上有多个非聚集的辅助索引（secondary index）。在进行插入操作时，数据页的存放还是按主键a进行顺序存放的，但是对于非聚集索引叶子节点的插入不再是顺序的了，这时就需要离散地访问非聚集索引页，由于随机读取的存在而导致了插入操作性能下降。因为B+树的特性决定了非聚集索引插入的离散性。

在某些情况下，辅助索引的插入依然是顺序的，或者说是比较顺序的，比如用户购买表中的时间字段。

InnoDB存储引擎开创性地设计了Insert Buffer，对于非聚集索引的插入或更新操作，不是每一次直接插入到索引页中，而是先判断插入的非聚集索引页是否在缓冲池中，若在，则直接插入；若不在，则先放入到一个Insert Buffer对象中，好似欺骗。

数据库这个非聚集的索引已经插到叶子节点，而实际并没有，只是存放在另一个位置。然后再以一定的频率和情况进行Insert Buffer和辅助索引页子节点的merge（合并）操作，这时通常能将多个插入合并到一个操作中（因为在一个索引页中），这就大大提高了对于非聚集索引插入的性能。

Insert Buffer的使用需要同时满足以下两个条件：

❑索引是辅助索引（secondary index）；
❑索引不是唯一（unique）的。

辅助索引不能是唯一的，因为在插入缓冲时，数据库并不去查找索引页来判断插入的记录的唯一性。如果去查找肯定又会有离散读取的情况发生，从而导致Insert Buffer失去了意义。

目前Insert Buffer存在一个问题是：在写密集的情况下，插入缓冲会占用过多的缓冲池内存（innodb_buffer_pool），默认最大可以占用到1/2的缓冲池内存。这对于其他的操作可能会带来一定的影响。

#### 2. Change Buffer

可将其视为Insert Buffer的升级。从这个版本开始，InnoDB存储引擎可以对DML操作——INSERT、DELETE、UPDATE都进行缓冲，他们分别是：Insert Buffer、Delete Buffer、Purge buffer。

当然和之前Insert Buffer一样，Change Buffer适用的对象依然是非唯一的辅助索引。

对一条记录进行UPDATE操作可能分为两个过程：

#### 3. Insert Buffer 内部实现

可能令绝大部分用户感到吃惊的是，Insert Buffer的数据结构是一棵B+树。在MySQL 4.1之前的版本中每张表有一棵Insert Buffer B+树。而在现在的版本中，全局只有一棵Insert Buffer B+树，负责对所有的表的辅助索引进行Insert Buffer。

###  2.6.2 两次写

如果说Insert Buffer带给InnoDB存储引擎的是性能上的提升，那么doublewrite（两次写）带给InnoDB存储引擎的是数据页的可靠性。

当发生数据库宕机时，可能InnoDB存储引擎正在写入某个页到表中，而这个页只写了一部分，比如16KB的页，只写了前4KB，之后就发生了宕机，这种情况被称为部分**写失效**（partial page write）。在InnoDB存储引擎未使用doublewrite技术前，曾经出现过因为部分写失效而导致数据丢失的情况。

有经验的DBA也许会想，如果发生写失效，可以通过重做日志进行恢复。这是一个办法。但是必须清楚地认识到，重做日志中记录的是对页的物理操作，如偏移量800，写'aaaa'记录。如果这个页本身已经发生了损坏，再对其进行重做是没有意义的。这就是说，在应用（apply）重做日志前，用户需要一个页的副本，当写入失效发生时，先通过页的副本来还原该页，再进行重做，这就是doublewrite。

InnoDB存储引擎中doublewrite的体系架构如图2-5所示。

![image-20191225175146457](E:\studydyup\notes\src\pic\image-20191225175146457.png)

doublewrite由两部分组成，一部分是内存中的doublewrite buffer，大小为2MB，另一部分是物理磁盘上共享表空间中连续的128个页，即2个区（extent），大小同样为2MB。在对缓冲池的脏页进行刷新时，并不直接写磁盘，而是会通过memcpy函数将脏页先复制到内存中的doublewrite buffer，之后通过doublewrite buffer再分两次，每次1MB顺序地写入共享表空间的物理磁盘上，然后马上调用fsync函数，同步磁盘，避免缓冲写带来的问题。因为doublewrite页是连续的，因此这个过程是顺序写的，开销并不是很大。在完成doublewrite页的写入后，再将doublewrite buffer中的页写入各个表空间文件中，此时的写入则是离散的。

如果操作系统在将页写入磁盘的过程中发生了崩溃，在恢复过程中，InnoDB存储引擎可以从共享表空间中的doublewrite中找到该页的一个副本，将其复制到表空间文件，再应用重做日志。

参数skip_innodb_doublewrite可以禁止使用doublewrite功能，这时可能会发生前面提及的写失效问题。不过如果用户有多个从服务器（slave server），需要提供较快的性能（如在slaves erver上做的是RAID0），也许启用这个参数是一个办法。不过对于**需要提供数据高可靠性的主服务器（master server），任何时候用户都应确保开启doublewrite功能**。

### 2.6.3 自适应哈希索引

哈希（hash）是一种非常快的查找方法，在一般情况下这种查找的时间复杂度为O(1)，即一般仅需要一次查找就能定位数据。而B+树的查找次数，取决于B+树的高度，在生产环境中，B+树的高度一般为3～4层，故需要3～4次的查询。

InnoDB存储引擎会监控对表上各索引页的查询。如果观察到建立哈希索引可以带来速度提升，则建立哈希索引，称之为自适应哈希索引（Adaptive Hash Index，AHI）。AHI是通过缓冲池的B+树页构造而来，因此建立的速度很快，而且不需要对整张表构建哈希索引。InnoDB存储引擎会自动根据访问的频率和模式来自动地为某些热点页建立哈希索引。

### 2.6.4 异步IO

为了提高磁盘操作性能，当前的数据库系统都采用异步IO（Asynchronous IO，AIO）的方式来处理磁盘操作。InnoDB存储引擎亦是如此。

与AIO对应的是Sync IO，即每进行一次IO操作，需要等待此次操作结束才能继续接下来的操作。但是如果用户发出的是一条索引扫描的查询，那么这条SQL查询语句可能需要扫描多个索引页，也就是需要进行多次的IO操作。在每扫描一个页并等待其完成后再进行下一次的扫描，这是没有必要的。用户可以在发出一个IO请求后立即再发出另一个IO请求，当全部IO请求发送完毕后，等待所有IO操作的完成，这就是AIO。
AIO的另一个优势是可以进行IO Merge操作，也就是将多个IO合并为1个IO，这样可以提高IOPS的性能。

例如用户需要访问页的（space，page_no）为：
（8，6）、（8，7），（8，8）
每个页的大小为16KB，那么同步IO需要进行3次IO操作。而AIO会判断到这三个页是连续的（显然可以通过（space，page_no）得知）。因此AIO底层会发送一个IO请求，从（8，6）开始，读取48KB的页。

在InnoDB1.1.x之前，AIO的实现通过InnoDB存储引擎中的代码来模拟实现。而从InnoDB 1.1.x开始（InnoDB Plugin不支持），提供了内核级别AIO的支持，称为Native AIO。因此在编译或者运行该版本MySQL时，需要libaio库的支持。

需要注意的是，Native AIO需要操作系统提供支持。Windows系统和Linux系统都提供Native AIO支持，而Mac OSX系统则未提供。

在InnoDB存储引擎中，read ahead方式的读取都是通过AIO完成，脏页的刷新，即磁盘的写入操作则全部由AIO完成。

### 2.6.5　刷新邻接页

InnoDB存储引擎还提供了Flush Neighbor Page（刷新邻接页）的特性。其工作原理为：当刷新一个脏页时，InnoDB存储引擎会检测该页所在区（extent）的所有页，如果是脏页，那么一起进行刷新。这样做的好处显而易见，通过AIO可以将多个IO写入操作合并为一个IO操作，故该工作机制在传统机械磁盘下有着显著的优势。但是需要考虑到下面两个问题：
❑是不是可能将不怎么脏的页进行了写入，而该页之后又会很快变成脏页？
❑固态硬盘有着较高的IOPS，是否还需要这个特性？
为此，InnoDB存储引擎从1.2.x版本开始提供了参数innodb_flush_neighbors，用来控制是否启用该特性。对于传统机械硬盘建议启用该特性，而对于固态硬盘有着超高IOPS性能的磁盘，则建议将该参数设置为0，即关闭此特性。

## 2.7　启动、关闭与恢复

InnoDB是MySQL数据库的存储引擎之一，因此InnoDB存储引擎的启动和关闭，更准确的是指在MySQL实例的启动过程中对InnoDB存储引擎的处理过程。

在关闭时，参数innodb_fast_shutdown影响着表的存储引擎为InnoDB的行为。该参数可取值为0、1、2，默认值为1。

❑0表示在MySQL数据库关闭时，InnoDB需要完成所有的full purge和merge insert buffer，并且将所有的脏页刷新回磁盘。这需要一些时间，有时甚至需要几个小时来完成。如果在进行InnoDB升级时，必须将这个参数调为0，然后再关闭数据库。

❑1是参数innodb_fast_shutdown的默认值，表示不需要完成上述的full purge和merge insert buffer操作，但是在缓冲池中的一些数据脏页还是会刷新回磁盘。

❑2表示不完成full purge和merge insert buffer操作，也不将缓冲池中的数据脏页写回磁盘，而是将日志都写入日志文件。这样不会有任何事务的丢失，但是下次MySQL数据库启动时，会进行恢复操作（recovery）。

当正常关闭MySQL数据库时，下次的启动应该会非常“正常”。但是如果没有正常地关闭数据库，如用kill命令关闭数据库，在MySQL数据库运行中重启了服务器，或者在关闭数据库时，将参数innodb_fast_shutdown设为了2时，下次MySQL数据库启动时都会对InnoDB存储引擎的表进行恢复操作。

参数innodb_force_recovery影响了整个InnoDB存储引擎恢复的状况。该参数值默认为0，代表当发生需要恢复时，进行所有的恢复操作，当不能进行有效恢复时，如数据页发生了corruption，MySQL数据库可能发生宕机（crash），并把错误写入错误日志中去。

现在来做一个实验，模拟故障的发生。在第一个会话中（session），对一张接近1 000万行的InnoDB存储引擎表进行更新操作，但是完成后不要马上提交：

```bash
mysql＞START TRANSACTION;
Query OK,0 rows affected(0.00 sec)
mysql＞UPDATE Profile SET password='';
Query OK,9587770 rows affected(7 min 55.73 sec)
Rows matched:9999248 Changed:9587770 Warnings:0
```

START TRANSACTION语句开启了事务，同时防止了自动提交（auto commit）的发生，UPDATE操作则会产生大量的UNDO日志（undo log）。这时，人为通过kill命令杀掉MySQL数据库服务器：

通过kill命令可以模拟数据库的宕机操作。下次MySQL数据库启动时会对之前的UPDATE事务进行回滚操作，而这些信息都会记录在错误日志文件（默认后缀名为err）中。如果查看错误日志文件，可得如下结果：

```
090922 13:40:20 InnoDB:Started;log sequence number 6 2530474615
InnoDB:Starting in background the rollback of uncommitted transactions
090922 13:40:20 InnoDB:Rolling back trx with id 0 5281035,8867280 rows to undo
InnoDB:Progress in percents:1090922 13:40:20
090922 13:40:20[Note]/usr/local/mysql/bin/mysqld:ready for connections.
Version:'5.0.45-log'socket:'/tmp/mysql.sock'port:3306 MySQL Community Server(GPL)
2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100
InnoDB:Rolling back of trx id 0 5281035 completed
090922 13:49:21 InnoDB:Rollback of non-prepared transactions completed
```

可以看到，采用默认的策略，即将innodb_force_recovery设为0，InnoDB会在每次启动后对发生问题的表进行恢复操作。通过错误日志文件，可知这次回滚操作需要回滚8867280行记录，差不多总共进行了9分钟。

再做一次同样的测试，只不过这次在启动MySQL数据库前，将参数innodb_force_recovery设为3，然后观察InnoDB存储引擎是否还会进行回滚操作。查看错误日志文件，可得：

```
090922 14:26:23 InnoDB:Started;log sequence number 7 2253251193
InnoDB:!!!innodb_force_recovery is set to 3!!!
090922 14:26:23[Note]/usr/local/mysql/bin/mysqld:ready for connections.
Version:'5.0.45-log'socket:'/tmp/mysql.sock'port:3306 MySQL Community Server(GPL)
```

这里出现了“!!!”，InnoDB警告已经将innodb_force_recovery设置为3，不会进行回滚操作了，因此数据库很快启动完成了。但是用户应该小心当前数据库的状态，并仔细确认是否不需要回滚事务的操作。

## 2.8 小结

# 第3章 文件

参数文件：告诉MySQL实例启动时在哪里可以找到数据库文件，并且指定某些初始化参数，这些参数定义了某种内存结构的大小等设置，还会介绍各种参数的类型。

日志文件：用来记录MySQL实例对某种条件做出响应时写入的文件，如错误日志文件、二进制日志文件、慢查询日志文件、查询日志文件等。

socket文件：当用UNIX域套接字方式进行连接时需要的文件。

pid文件：MySQL实例的进程ID文件。

MySQL表结构文件：用来存放MySQL表结构定义文件。

存储引擎文件：因为MySQL表存储引擎的关系，每个存储引擎都会有自己的文件来保存各种数据。这些存储引擎真正存储了记录和索引等数据。本章主要介绍与InnoDB有关的存储引擎文件。

## 3.1 参数文件

当MySQL实例启动时，数据库会先去读一个配置参数文件，用来寻找数据库的各种文件所在位置以及指定某些初始化参数，这些参数通常定义了某种内存结构有多大等。在默认情况下，MySQL实例会按照一定的顺序在指定的位置进行读取，用户只需通过命令mysql --help|grep my.cnf来寻找即可。

### 3.1.1　什么是参数

简单地说，可以把数据库参数看成一个键/值（key/value）对。

可以通过命令SHOW VARIABLES查看数据库中的所有参数，也可以通过LIKE来过滤参数名。

### 3.1.2　参数类型

MySQL数据库中的参数可以分为两类：
❑动态（dynamic）参数
❑静态（static）参数

动态参数意味着可以在MySQL实例运行中进行更改，静态参数说明在整个实例生命周期内都不得进行更改，就好像是只读（read only）的。可以通过SET命令对动态的参数值进行修改，SET的语法如下：

```sql
SET
|[global|session]system_var_name=expr
|[@@global.|@@session.|@@]system_var_name=expr
```

这里可以看到global和session关键字，它们表明该参数的修改是基于当前会话还是整个实例的生命周期。有些动态参数只能在会话中进行修改，如autocommit；而有些参数修改完后，在整个实例生命周期中都会生效，如binlog_cache_size；而有些参数既可以在会话中又可以在整个实例的生命周期内生效，如read_buffer_size。举例如下：

```
mysql＞SET read_buffer_size=524288;
Query OK,0 rows affected(0.00 sec)
mysql＞SELECT@@session.read_buffer_size\G;
***************************1.row***************************
@@session.read_buffer_size:524288
1 row in set(0.00 sec)
mysql＞SELECT@@global.read_buffer_size\G;
***************************1.row***************************
@@global.read_buffer_size:2093056
1 row in set(0.00 sec)
```

上述示例中将当前会话的参数read_buffer_size从2MB调整为了512KB，而用户可以看到全局的read_buffer_size的值仍然是2MB，也就是说如果有另一个会话登录到MySQL实例，它的read_buffer_size的值是2MB，而不是512KB。这里使用了set global|session来改变动态变量的值。用户同样可以直接使用SET@@globl|@@session来更改，如下所示：

```
mysql＞SET@@global.read_buffer_size=1048576;
Query OK,0 rows affected(0.00 sec)
mysql＞SELECT@@session.read_buffer_size\G;
***************************1.row***************************
@@session.read_buffer_size:524288
1 row in set(0.00 sec)
mysql＞SELECT@@global.read_buffer_size\G;
***************************1.row***************************
@@global.read_buffer_size:1048576
1 row in set(0.00 sec)
```

这次把read_buffer_size全局值更改为1MB，而当前会话的read_buffer_size的值还是512KB。这里需要注意的是，对变量的全局值进行了修改，在这次的实例生命周期内都有效，但MySQL实例本身并不会对参数文件中的该值进行修改。也就是说，在下次启动时MySQL实例还是会读取参数文件。若想在数据库实例下一次启动时该参数还是保留为当前修改的值，那么用户必须去修改参数文件。要想知道MySQL所有动态变量的可修改范围，可以参考MySQL官方手册的Dynamic System Variables的相关内容。
对于静态变量，若对其进行修改，会得到类似如下错误：

```
mysql＞SET GLOBAL datadir='/db/mysql';
ERROR 1238(HY000):Variable'datadir'is a read only variable
```

## 3.2　日志文件

日志文件记录了影响MySQL数据库的各种类型活动。MySQL数据库中常见的日志文件有：
❑错误日志（error log）
❑二进制日志（binlog）
❑慢查询日志（slow query log）
❑查询日志（log）
这些日志文件可以帮助DBA对MySQL数据库的运行状态进行诊断，从而更好地进行数据库层面的优化。

### 3.2.1　错误日志

错误日志文件对MySQL的启动、运行、关闭过程进行了记录。MySQL DBA在遇到问题时应该首先查看该文件以便定位问题。该文件不仅记录了所有的错误信息，也记录一些警告信息或正确的信息。用户可以通过命令```SHOW VARIABLES LIKE'log_error'```来定位该文件，

### 3.2.2　慢查询日志

3.2.1小节提到可以通过错误日志得到一些关于数据库优化的信息，而慢查询日志（slow log）可帮助DBA定位可能存在问题的SQL语句，从而进行SQL语句层面的优化。例如，可以在MySQL启动时设一个阈值，将运行时间超过该值的所有SQL语句都记录到慢查询日志文件中。DBA每天或每过一段时间对其进行检查，确认是否有SQL语句需要进行优化。该阈值可以通过参数long_query_time来设置，默认值为10，代表10秒。
在默认情况下，MySQL数据库并不启动慢查询日志，用户需要手工将这个参数设为ON：

```
show variables like 'slow_query%';

set global slow_query_log='ON'
```



```
mysql＞SHOW VARIABLES LIKE'long_query_time';
```

另一个和慢查询日志有关的参数是log_queries_not_using_indexes，如果运行的SQL语句没有使用索引，则MySQL数据库同样会将这条SQL语句记录到慢查询日志文件。首先确认打开了log_queries_not_using_indexes：

```
mysql＞SHOW VARIABLES LIKE'log_queries_not_using_indexes';
```

DBA可以通过慢查询日志来找出有问题的SQL语句，对其进行优化。然而随着MySQL数据库服务器运行时间的增加，可能会有越来越多的SQL查询被记录到了慢查询日志文件中，此时要分析该文件就显得不是那么简单和直观的了。而这时MySQL数据库提供的mysqldumpslow命令，可以很好地帮助DBA解决该问题：

```
mysqldumpslow nh122-190-slow.log
```

如果用户希望得到执行时间最长的10条SQL语句，可以运行如下命令：

```
mysqldumpslow-s al-n 10 david.log
```

MySQL 5.1开始可以将慢查询的日志记录放入一张表中，这使得用户的查询更加方便和直观。慢查询表在mysql架构下，名为slow_log，其表结构定义如下：

```mysql＞SHOW CREATE TABLE mysql.slow_log;```

参数log_output指定了慢查询输出的格式，默认为FILE，可以将它设为TABLE，然后就可以查询mysql架构下的slow_log表了，如：

```
mysql＞SHOW VARIABLES LIKE'log_output';
+---------------+---------+
|Variable_name|Value|
+---------------+---------+
|log_output|FILE|
+---------------+---------+
1 row in set(0.00 sec)
mysql＞SET GLOBAL log_output='TABLE';
Query OK,0 rows affected(0.00 sec)
mysql＞SHOW VARIABLES LIKE'log_output'\G;
+---------------+---------+
|Variable_name|Value|
+---------------+---------+
|log_output|TABLE|
```

```
mysql＞SELECT*FROM mysql.slow_log;
```

例子：

```sql
show variables like 'slow_query%';

set global slow_query_log='ON'


SET GLOBAL log_output='TABLE';


select sleep(11)

SELECT*FROM mysql.slow_log
```

### 3.2.3　查询日志

查询日志记录了所有对MySQL数据库请求的信息，无论这些请求是否得到了正确的执行。默认文件名为：主机名.log。如查看一个查询日志：

```mysql
[root@nineyou0-43 data]#tail nineyou0-43.log
090925 11:00:24 44 Connect zlm@192.168.0.100 on
44 Query SET AUTOCOMMIT=0
44 Query set autocommit=0
44 Quit
090925 11:02:37 45 Connect Access denied for user'root'@'localhost'(using password:NO)
090925 11:03:51 46 Connect Access denied for user'root'@'localhost'(using password:NO)
090925 11:04:38 23 Query rollback
```

通过上述查询日志会发现，查询日志甚至记录了对Access denied的请求，即对于未能正确执行的SQL语句，查询日志也会进行记录。同样地，从MySQL 5.1开始，可以将查询日志的记录放入mysql架构下的general_log表中，该表的使用方法和前面小节提到的slow_log基本一样，这里不再赘述。

### 3.2.4　二进制日志

二进制日志（binary log）记录了对MySQL数据库执行更改的所有操作，但是不包括SELECT和SHOW这类操作，因为这类操作对数据本身并没有修改。然而，若操作本身并没有导致数据库发生变化，那么该操作可能也会写入二进制日志。

MySQL数据库首先进行UPDATE操作，从返回的结果看到Changed为0，这意味着该操作并没有导致数据库的变化。但是通过命令SHOW BINLOG EVENT可以看出在二进制日志中的确进行了记录。

总的来说，二进制日志主要有以下几种作用。
❑恢复（recovery）：某些数据的恢复需要二进制日志，例如，在一个数据库全备文件恢复后，用户可以通过二进制日志进行point-in-time的恢复。
❑复制（replication）：其原理与恢复类似，通过复制和执行二进制日志使一台远程的MySQL数据库（一般称为slave或standby）与一台MySQL数据库（一般称为master或primary）进行实时同步。
❑审计（audit）：用户可以通过二进制日志中的信息来进行审计，判断是否有对数据库进行注入的攻击。

通过配置参数log-bin[=name]可以启动二进制日志。如果不指定name，则默认二进制日志文件名为主机名，后缀名为二进制日志的序列号，所在路径为数据库所在目录（datadir）

```
mysql＞show variables like'datadir';
```

二进制日志文件在默认情况下并没有启动，需要手动指定参数来启动。可能有人会质疑，开启这个选项是否会对数据库整体性能有所影响。不错，开启这个选项的确会影响性能，但是性能的损失十分有限。根据MySQL官方手册中的测试表明，开启二进制日志会使性能下降1%。但考虑到可以使用复制（replication）和point-in-time的恢复，这些性能损失绝对是可以且应该被接受的。

以下配置文件的参数影响着二进制日志记录的信息和行为：
❑max_binlog_size
❑binlog_cache_size
❑sync_binlog
❑binlog-do-db
❑binlog-ignore-db
❑log-slave-update
❑binlog_format

参数max_binlog_size指定了单个二进制日志文件的最大值，如果超过该值，则产生新的二进制日志文件，后缀名+1，并记录到.index文件。

当使用事务的表存储引擎（如InnoDB存储引擎）时，所有未提交（uncommitted）的二进制日志会被记录到一个缓存中去，等该事务提交（committed）时直接将缓冲中的二进制日志写入二进制日志文件，而该缓冲的大小由binlog_cache_size决定，默认大小为32K。此外，binlog_cache_size是基于会话（session）的，也就是说，当一个线程开始一个事务时，MySQL会自动分配一个大小为binlog_cache_size的缓存，因此该值的设置需要相当小心，不能设置过大。当一个事务的记录大于设定的binlog_cache_size时，MySQL会把缓冲中的日志写入一个临时文件中，因此该值又不能设得太小。

MySQL 5.1开始引入了binlog_format参数，该参数可设的值有STATEMENT、ROW和MIXED。



## 3.3　套接字文件

前面提到过，在UNIX系统下本地连接MySQL可以采用UNIX域套接字方式，这种方式需要一个套接字（socket）文件。套接字文件可由参数socket控制。一般在/tmp目录下，名为mysql.sock：

```
mysql＞SHOW VARIABLES LIKE'socket';
```



## 3.4　pid文件

当MySQL实例启动时，会将自己的进程ID写入一个文件中——该文件即为pid文件。该文件可由参数pid_file控制，默认位于数据库目录下，文件名为主机名.pid：

```
mysql＞show variables like'pid_file';
```



## 3.5　表结构定义文件

因为MySQL插件式存储引擎的体系结构的关系，MySQL数据的存储是根据表进行的，每个表都会有与之对应的文件。但不论表采用何种存储引擎，MySQL都有一个以frm为后缀名的文件，这个文件记录了该表的表结构定义。
frm还用来存放视图的定义，如用户创建了一个v_a视图，那么对应地会产生一个v_a.frm文件，用来记录视图的定义，该文件是文本文件，可以直接使用cat命令进行查看：

## 3.6　InnoDB存储引擎文件

之前介绍的文件都是MySQL数据库本身的文件，和存储引擎无关。除了这些文件外，每个表存储引擎还有其自己独有的文件。本节将具体介绍与InnoDB存储引擎密切相关的文件，这些文件包括重做日志文件、表空间文件。

### 3.6.1　表空间文件

InnoDB采用将存储的数据按表空间（tablespace）进行存放的设计。

在默认配置下会有一个初始大小为10MB，名为ibdata1的文件。该文件就是默认的表空间文件（tablespace file），用户可以通过参数innodb_data_file_path对其进行设置，格式如下：

```
innodb_data_file_path=datafle_spec1[;datafle_spec2]...
```

用户可以通过多个文件组成一个表空间，同时制定文件的属性

```
innodb_data_file_path=/db/ibdata1:2000M;/dr2/db/ibdata2:2000M:autoextend
```

设置innodb_data_file_path参数后，所有基于InnoDB存储引擎的表的数据都会记录到该共享表空间中。若设置了参数innodb_file_per_table，则用户可以将每个基于InnoDB存储引擎的表产生一个独立表空间。独立表空间的命名规则为：表名.ibd。通过这样的方式，用户不用将所有数据都存放于默认的表空间中。

图3-1显示了InnoDB存储引擎对于文件的存储方式：

![image-20191226174201983](E:\studydyup\notes\src\pic\image-20191226174201983.png)

### 3.6.2　重做日志文件

在默认情况下，在InnoDB存储引擎的数据目录下会有两个名为ib_logfile0和ib_logfile1的文件。在MySQL官方手册中将其称为InnoDB存储引擎的日志文件，不过更准确的定义应该是重做日志文件（redo log file）。为什么强调是重做日志文件呢？因为重做日志文件对于InnoDB存储引擎至关重要，它们记录了对于InnoDB存储引擎的事务日志。

当实例或介质失败（media failure）时，重做日志文件就能派上用场。例如，数据库由于所在主机掉电导致实例失败，InnoDB存储引擎会使用重做日志恢复到掉电前的时刻，以此来保证数据的完整性。

写入重做日志文件的操作不是直接写，而是先写入一个重做日志缓冲（redo log buffer）中，然后按照一定的条件顺序地写入日志文件。图3-3很好地诠释了重做日志的写入过程。

![image-20191226174458795](E:\studydyup\notes\src\pic\image-20191226174458795.png)

从重做日志缓冲往磁盘写入时，是按512个字节，也就是一个扇区的大小进行写入。因为扇区是写入的最小单位，因此可以保证写入必定是成功的。因此在重做日志的写入过程中不需要有doublewrite。

触发写磁盘的过程是由参数innodb_flush_log_at_trx_commit控制，表示在提交（commit）操作时，处理重做日志的方式。

参数innodb_flush_log_at_trx_commit的有效值有0、1、2。0代表当提交事务时，并不将事务的重做日志写入磁盘上的日志文件，而是等待主线程每秒的刷新。1和2不同的地方在于：1表示在执行commit时将重做日志缓冲同步写到磁盘，即伴有fsync的调用。2表示将重做日志异步写到磁盘，即写到文件系统的缓存中。因此不能完全保证在执行commit时肯定会写入重做日志文件，只是有这个动作发生。

因此为了保证事务的ACID中的持久性，**必须将innodb_flush_log_at_trx_commit设置为1，也就是每当有事务提交时，就必须确保事务都已经写入重做日志文件。**那么当数据库因为意外发生宕机时，可以通过重做日志文件恢复，并保证可以恢复已经提交的事务。而将重做日志文件设置为0或2，都有可能发生恢复时部分事务的丢失。不同之处在于，设置为2时，当MySQL数据库发生宕机而操作系统及服务器并没有发生宕机时，由于此时未写入磁盘的事务日志保存在文件系统缓存中，当恢复时同样能保证数据不丢失。

## 3.7　小结

重做日志非常的重要，用来记录InnoDB存储引擎的事务日志，也因为重做日志的存在，才使得InnoDB存储引擎可以提供可靠的事务。

# 第4章　表

本章将从InnoDB存储引擎表的逻辑存储及实现开始进行介绍，然后将重点分析表的物理存储特征，即数据在表中是如何组织和存放的。简单来说，表就是关于特定实体的数据集合，这也是关系型数据库模型的核心。

## 4.1　索引组织表

