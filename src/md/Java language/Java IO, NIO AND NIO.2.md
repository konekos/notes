# Java I/O, NIO and NIO.2

## Chapter 1 I/O Basics and APIs

NIO作为JDK 1.4的一部分被引入来支持操作系统的新的I/O规范。由于时间不够，没能把一些计划的特性加到这个release，被推迟到了JDK 5 and JDK 7。

这章介绍classic I/O, NIO, and more NIO (NIO.2)。接下来的章节深入研究它们的api。

### Classic I/O

JDK1.0引进初步的I/O设施，用于访问文件系统（创建文件夹，删除文件等操作），随机访问文件目录（而不是按顺序），和源和目标之间以顺序方式的面向字节数据流。

### File System Access and the File Class

*file system*是操作系统组件管理数据储存和后续检索。运行JVM的操作系统支持至少一个文件系统。例如，Unix或Linux结合所有安装（attached and prepared)）的disks到一个虚拟文件系统。与此相反，Windows把一个分割的文件系统和每个活动的磁盘驱动器联系起来。



