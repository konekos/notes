# 一、 数据结构与对象

总结：

- Redis 数据库中每个键值对的键和值都是一个对象
- String，List，Hash，Set，Sorted Set 5种类型的对象，每种对象有至少2种的编码方式。
- 服务器执行命令前，先check键能否执行指定的命令，通过检查键对象的类型。
- Redis 对象系统带有引用计数实现的内存回收机制，一个对象不再被使用时，释放对象占用的内存。
- Redis 共享值为 0-9999 的字符串对象。
- 对象会记录自己最后一次被访问的时间，用于计算对象的空转时间。



# 二、单机数据库的实现

## 第9章 数据库

0-15 总16个DB。切换数据库，客户端通过 SELECT 命令切换。

**键空间**：数据库的所有键值对。

- 键空间的键是数据库的键，是字符串对象。
- 键空间的值是数据库的值，任意 Redis 对象。

键：添加、删除、更新、取值、其他键空间的操作（FLUSHDB、RANDONKEY、DBSIZE、EXISTS、RENAME、KEYS等）

**设置键的过期时间**：EXPIRE。SETEX（仅字符串），移除过期时间 PRESIST、返回剩余时间 TTL。



**过期键的删除策略**：

- 定时删除	创建key的同时创建一个 Timer，对内存友好，对CPU不友好
- 惰性删除     获取时检查是否过期。对CPU友好，对内存不友好。
- 定期删除    每隔一段时间检查数据库，删除过期键。删除多少，检查多少数据库由算法决定。



**Redis 的过期键删除策略**
使用的惰性删除和定期删除两种策略。



**RDB和AOF和复制功能对过期键的处理**

- RDB 
  - 生成RDB： 过期键不会存入RDB文件。
  - 载入RDB： Master服务器，删除过期键。Slave服务器，都会载入。
- AOF
  - 写入AOF： 如果已过期，还没被惰性删除或者定期删除，没有影响一样写入。被惰性删除或定期删除时，向 AOF 文件追加 DEL 指令。
  - AOF 重写：类似RDB生成，过期的key不会存入。
- 复制：由Master控制。
  - 删除过期键时，向Slave发送DEL指令。
  - Slave执行读时，遇到过期key不会做删除操作。
  - Slave只会接收Master的DEL指令才会删除key。



## 第10章 RDB持久化

**创建和载入：** 

- 生成RDB文件。SAVE（阻塞）、BGSAVE（子进程、非阻塞）命令。

- 载入RDB，在服务器启动时自动检测RDB文件，有的话阻塞直到写完。

**自动间隔保存：**

默认save条件：

save 900 1

save 300 10

save 60 10000 

满足3个条件之一就会触发BGSAVE。



不同类型的键值对：RDB文件用不同的方式保存。



RDB 文件检查工具 ：redis-check-dump。



## 第11章 AOF持久化

Append Only File 持久化。保存Redis服务器执行的写命令记录数据库状态。以Redis命令请求协议的格式保存。



**实现：** 

- 命令追加（append）：
- 文件写入，先写入缓冲区，再落盘。
  -  appendfsync 值为always，写入文件
  -  everysecond  每隔一秒对子线程的aof文件进行同步。
  -  no 由操作系统完成缓冲区刷新到磁盘的操作。
- 文件同步（sync）



**载入与还原**



**AOF重写**

AOF 文件体积会越来越大。提供了rewrite的功能。创建一个新的AOF文件代替当前的AOF文件。

- 重写的实现：不会对当前AOF文件读取分析和写入，通过读取当前数据库状态实现。
- AOF 后台重写。 aof_rewrite会造成阻塞。BGREWRITEAOF命令。 会先创建一个缓冲区，保存新的指令，然后追加到新的AOF文件，使一致。



## 第12章 事件

- Redis服务器是事件驱动程序，包括文件时间和时间事件。
- 文件事件处理器是基于 Reactor 模式实现的网络通信程序。
- 文件事件是对套接字操作的抽象：每次套接字变为可应答（acceptable），可写入（writable）或者可读（readable）时，就产生相应的文件事件。
- 文件事件包括 AE_READABLE 事件（读事件）和 AE_WRITABLE （写事件）两类。



## 第13章 客户端

Redis 服务器以单线程单进程方式处理命令请求，与多个客户端通信。

Redis 服务器会保存客户端的信息。

**客户端属性：**

- 套接字描述符
- name
- 标志
- 标志
- 输入缓冲区
- 命名和命令参数
- 命令实现函数
- 输出缓冲区
- 身份验证
- 时间



**客户端的创建与关闭**

略



## 第14章 服务器

**命令的执行过程**

- 客户端发送命令
- server读取命令
- 命令执行器：
  - 查找命令实现
  - 执行预备操作
  - 调用命令实现函数
  - 执行后续工作
- 回复客户端



**serverCron 函数**

- 更新服务器时间缓存
- 更新LRU时钟
- 更新服务器每秒执行命令次数
- 更新服务器内存峰值记录
- 处理 SIGTERM信号。
- 管理客户端资源
- 管理数据库资源
- 执行被延迟的BGREWRITEAOF
- 检查持久化操作运行状态。
- 将AOF缓冲区的内容写入AOF。
- 关闭异步客户端
- 增加 cronloops 计数器的值



**初始化服务器**

- 初始化服务器的结构状态
- 载入配置选项
- 初始化服务器数据结构
- 还原数据库状态
- 执行事件循环



# 三、多机数据库的实现

## 第15章 复制

SLAVEOF命令或配置，复制另一台服务器（Master）。

**旧版复制的实现**

- 同步：向master发送SYNC指令
- 命令传播：

缺陷：初次复制和断线后重复制。旧版断线后重复制效率低。



**新版复制**

从Redis 2.8开始，使用PSYNC代替SYNC执行复制的同步操作。解决了断线后重复制效率低的问题。



**复制的实现**

略

**心跳检测**
REPLCONF ACK命令心跳。

配置项 ：min-slaves-to-write 和 min-slaves-max-lag  。 都不满足时拒绝写入。



## 第16章 Sentinel



**启动和初始化Sentinel**

- 初始化：Sentinel 本质是运行在特殊模式下的 Redis服务器。
- 使用 Sentinel 专用代码
- 初始化 Sentinel 状态
- 初始化Sentinel 状态的 masters 属性
- 创建连向主服务器的网络连接



**获取主服务器信息**
Sentinel 默认每 10s 向被监视的master发送 INFO 命令。



**获取从服务器信息**
 Sentinel 发现master有新的salve时，



**向主服务器和从服务器发送信息**
默认情况下 2s 一次向master和Slave发送信息。



**接受来自主服务器和从服务器的频道信息**
略



**检测主观下线状态**
默认下，Sentinel 1s 一次的频率向所有master，slave 和 其他的 Sentinel 发送PING 命令，判断是否在线。



**检查客观下线状态**
Sentinel 将一个 master 判断为主观下线后，为了确认 master 是否真的挂掉了， 会向监视这个master的其他Sentinel 进行询问。

- 发送 Sentinel is-master-down-by-addr 命令
- 接收 Sentinel is-master-down-by-addr 命令  其他的Sentinel接收命令，根据命令的master信息，检查master是否下线。然后返回回复。
- 接收 Sentinel 的回复。

*当 认为master下线的Sentinel 超过了Sentinel配的quorum的值，就认为master挂了。*

*不同的Sentinel 判断客观下线的条件可能不同*



**选举 Leader 的 Sentinel**
当 master 被认为下线。Sentinel 集群会选举一个 leader Sentinel，由leader 对挂掉的master进行故障转移。



**故障转移**
选出的Sentinel 的Leader会进行故障转移操作：

- 在所有slave节点中，挑出一个转换为master
  - 选择一个状态好，数据完整的slave，向这个slave发出 SLAVEOF no one 命令，转为Master
- 让所有slave 复制新的master 
  - 向slave 发送 SLAVEOF 命令实现
- 将挂掉的master 设置为 新master的slave，等旧master上线的时候，会成为新的master的slave。
  - 当旧master 重新上线，Sentinel 向它发出 SLAVEOF 命令，复制新的master。



## 第17章 集群（redis-cluster)

#### 节点



#### 槽指派

集群整个数据库被分为 16384个槽，每个key 都在其中一个槽。分配槽。

计算key属于哪个槽，slot_number(key): return CRC16(key) & 16383
使用 CLUSTER KEYSLOT  key命令 查看属于哪个槽。



**重新分片**
将已经分配给某个节点的槽改为分配给另外一个节点。 重分片时，集群不需要下线。



**ASK错误和MOVED错误**

m代表负责权已转移

a只是迁移中的一种临时措施



#### 复制与故障转移



