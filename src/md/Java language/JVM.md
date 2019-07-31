 -Duser.timezone=Asia/Shanghai 

-Xss256K 

 设置每个线程的运行时栈的大小为 256K。

-server 

server模式

-XX:+DisableExplicitGC 

忽略手动调用GC的代码

-XX:+PrintFlagsFinal 

显示所有可设置参数及默认值，可结合-XX:+PrintFlagsInitial与-XX:+PrintFlagsFinal对比设置前、设置后的差异，方便知道对那些参数做了调整。

-XX:+UseConcMarkSweepGC 

使用的并发标记清除（CMS）收集器

-Xmx6500m 

置JVM最大可用内存

-Xms6500m 

设置JVM促使内存为3550m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。

-XX:NewSize=2000m 

设置年轻代大小

-XX:PermSize=128m 

设置持久代大小

-XX:MaxPermSize=128m 

设置最大持久代大小

-Xloggc:/data/clicker/api/bootstrap/gc.log 

垃圾回收统计信息

-XX:+PrintGCDetails 

-XX:+PrintGCTimeStamps 

-XX:+HeapDumpOnOutOfMemoryError 

JVM会在遇到OutOfMemoryError时拍摄一个“堆转储快照”,并将其保存在一个文件中

-XX:HeapDumpPath=/data/clicker/api/bootstrap/dump/heap.dump 