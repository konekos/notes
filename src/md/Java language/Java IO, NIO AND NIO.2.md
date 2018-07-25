# Java I/O, NIO and NIO.2

## Chapter 1 I/O Basics and APIs

NIO作为JDK 1.4的一部分被引入来支持操作系统的新的I/O规范。由于时间不够，没能把一些计划的特性加到这个release，被推迟到了JDK 5 and JDK 7。

这章介绍classic I/O, NIO, and more NIO (NIO.2)。接下来的章节深入研究它们的api。

### Classic I/O

JDK1.0引进初步的I/O设施，用于访问文件系统（创建文件夹，删除文件等操作），随机访问文件目录（而不是按顺序），和源和目标之间以顺序方式的面向字节数据流。

### File System Access and the File Class

*file system*是操作系统组件管理数据储存和后续检索。运行JVM的操作系统支持至少一个文件系统。例如，Unix或Linux结合所有安装（attached and prepared)）的disks到一个虚拟文件系统。与此相反，Windows把一个分割的文件系统和每个活动的磁盘驱动器联系起来。

Windows和类似的操作系统可以管理多个文件系统。 每个文件系统都用一个驱动器说明符来标识，比如`C:`。 指定一条没有驱动器说明符的路径，路径是相对于当前文件系统。

一个 `java.io.File  `类实例抽象一个文件或者文件路径。这个实例提供文件系统访问来在这个path上执行任务比如移除下面的文件和文件夹。比如：

```
new File("temp").mkdir();
```

### Accessing File Content via RandomAccessFile 

文件内容可以按顺序或随机访问。随机访问可以加快搜索和排序功能。在 `java. io.RandomAccessFile `类提供随机访问文件。例如：

```java
RandomAccessFile raf = new RandomAccessFile("employees.dat", "r");
int empIndex = 10;
raf.seek(empIndex * EMP_REC_LEN);
// Read contents of employee record.

```

` employees.dat`文件被分割成固定长度的employees记录，每个记录 EMP_REC_LEN bytes长，被访问。第10个索引的employee被查找（第一个index 0）。这个任务通过seeking（设置file pointer）这个记录的第一个字节的字节位置，它处于记录长度乘索引。记录然后被访问。

### Streaming Data via Stream Classes 

Classic I/O 包含streams用于执行I/O操作。流是任意长度的有顺序的bytes sequence。bytes从应用的*output stream*流出到目的地，和从一个source的input stream流出到应用。

![1532340528079](https://github.com/konekos/notes/blob/master/src/pic/1532340528079.png)

Java在 `java.io `包提供类用于识别用于writing的stream destinations；例如byte arrays和files。也提供类识别各种stream sources用于reading。例子包括files和 thread pipes。

例如，你会用 `FileInputStream`打开一个存在的文件，并且用一个 input stream连接它。你会用各种` read() `方法通过input stream从file读取字节。最后，调用`close() `关闭stream和文件。例如：

```java
FileInputStream fis = null;
try
{
 fis = new FileInputStream("image.jpg");
 // Read bytes from file.
 int _byte;
 while ((_byte = fis.read()) != -1) // -1 signifies EOF
 ; // Process _byte in some way.
}
catch (IOException ioe)
{
 // Handle exception.
}
finally
{
 if (fis != null)
 try
 {
 fis.close();
 }
}

```

这个例子展示了打开文件一个文件的传统方法且创建一个输入流从文件中读取字节。然后它继续读取文件的内容。异常处理程序负责处理抛出的异常，由`java.io.IOException`表示。

不管有没有抛出异常，输入流和之下的文件必须关闭。这个动作发生在try声明的finally块。因为关闭文件的冗长，你可以选择用JDK 7的try-with-resources陈述来自动关闭，如下：

```
try (FileInputStream fis = new FileInputStream("image.jpg"))
{
 // Read bytes from file.
 int _byte;
 while ((_byte = fis.read()) != -1) // -1 signifies EOF
 ; // Process _byte in some way.
}
catch (IOException ioe)
{
 // Handle exception.
}

```

一些stream类用于过滤其他stream。例如，为了提升性能，` BufferedInputStream `从其他stream读取一块bytes，从它的buffer返回字节直到buffer空为止，然后读取另一块。例如：

```java
try (FileInputStream fis = new FileInputStream("image.jpg");
 BufferedInputStream bis = new BufferedInputStream(fis))
{
 // Read bytes from file.
 int _byte;
 while ((_byte = bis.read()) != -1) // -1 signifies EOF
 ; // Process _byte in some way.
}
catch (IOException ioe)
{
 // Handle exception.
}

```

从`image.jpg `读取的 file input stream被创建。这个stream被转换成buffered输入流构造器。随后的读在buffered input stream上执行，它在合适时候调用file input stream的`read()`。

### Stream Classes and Standard I/O 

很多操作系统支持标准I/O， preconnected streams被称为 standard input, standard output, and standard error。

标准输入默认从键盘输入。然而，也可以重定向从不同的源读输出到不同的目的地，比如文件。

JDK 1.0引入标准I/O支持通过添加InputStream and PrintStream类型的in, out, and err o对象到`java.lang.System `类。你指定调用这些对象的方法来连接标准输入输出和error，如下：

```
int ch = System.in.read(); // Read single character from standard input.
System.out.println("Hello"); // Write string to standard output.
System.err.println("I/O error: " +
 ioe.getMessage()); // Write string to standard error.
```

### JDK 1.1 and the Writer/Reader Classes 

JDK 1.0的I/O适合字节流，但不适合字符流因为不考虑字符编码。JDK 1.1引入了 writer/reader类克服这个问题，把字符编码考虑了在内。例如`java.io `包含有`s FileWriter and FileReader `类用于读写字符流。

### NIO 

现代操作系统提供精致的I/O服务（比如readiness selection）用于提升I/O性能与简化I/O。 Java Specification Request (JSR) 51  (www.jcp.org/en/jsr/detail?id=51) 被创建用于落地这个特性。

JSR 51的描述表明提供了APIs用于可伸缩I/O， fast buffered binary and character I/O ，正则表达式和字符集转换。这些APIs就是NIO。JDK1.4在以下APIs方面实现NIO：

- Buffers
- Channels
- Selectors
- Regular expression
- Charsets

regular expression and charset APIs被提供给简化常规与I/O有联系的tasks。

### Buffers 

Buffers（缓冲区）是NIO操作的基础。本质上，NIO全部都是关于把数据从Buffers移入或移出。

一个过程比如JVM执行I/O通过请求操作系统去排干buffer的内容来通过写操作存储。类似地，它请求操作系统用从存储设备读取的数据填充buffer。

考虑一个涉及磁盘驱动的读操作。操作系统发出一个指令到磁盘controller来读取磁盘的一块字节到一个操作系统的buffer。一旦操作完成，操作系统复制这个buffer contents到发生`read()`操作的指定的buffer。

![1532487035762](https://github.com/konekos/notes/blob/master/src/pic/1532487035762.png)

图1-2中一个程序向操作系统发起一个 read() 调用。依次地，操作系统请求磁盘controller从磁盘读取一块bytes。磁盘controller（也叫DMA controller）通过*Direct Memory Access (DMA)*直接把这些字节读到操作系统buffer，DMA是计算机系统的一个特性，允许某个硬件子系统不依赖 [central processing unit](https://en.wikipedia.org/wiki/Central_processing_unit#Central%20processing%20unit) (CPU) 直接访问 系统[内存](https://en.wikipedia.org/wiki/Computer_storage#Computer%20storage)  (RAM) 。

从操作系统buffer复制 bytes到程序buffer是不那么高效的。让DMA controller直接复制到程序buffer会更有效，这个方法有2个问题：

- DMA controller特别是不能和JVM运行其上的user space（用户空间）连接，而是和 operating system’s kernel space（操作系统内核空间）连接。
- Block-oriented设备比如DMA controller使用固定大小的数据块工作。不同的是，JVM程序可能会请求一个不是block size倍数大小的数据。

因为这些问题，操作系统作为一个中介角色，当它在JVM程序和DMA controller切换时扯碎和重组数据。

装配/拆卸数据的任务可以更有效，通过让JVM程序在一个系统调用传递一个buffer addresses给操作系统。操作系统按顺序填充或者排干这些buffer，在一个读操作分散数据到多个buffer或者在写操作时从多个buffer聚集数据。这个聚集/分散的活动减少了JVM程序必须要做的系统调用（潜在地很昂贵）并且使操作系统优化数据处理因为系统知道buffer空间的总数。此外，当多个处理器或内核可用，操作系统可能允许buffers同时填充或者排干。

JDK 1.4的 java.nio.Buffer类抽象了JVM程序buffer的概念。作为`java.nio.ByteBuffer`和其他类的父类。因为I/O根本上是面向字节的，只有` ByteBuffer `实例可以用channels。大多数其他Buffer的子类方便于处理multibyte数据（例如characters or integers）。

### Channels 

强制CPU执行I/O任务并等待I/O完成（这样的CPU被叫做 I/O bound）是资源的浪费。将这些任务卸载到DMA controller可以改善性能，处理器可以做其他工作。

一个channel充当一个管道用于连接DMA controller来从硬盘有效排干或填充字节数据。JDK 1.4的`java.nio.channels.Channel `接口，它的子接口和众多类实现了channel架构。

其中一个类是` java.nio.channels.FileChannel`，抽象了一个channel用于读写映射操作文件。 `FileChannel `的一个有趣的特性是支持file locking，在这些复杂的应用程序中，如数据库管理系统。

*File locking*让程序在访问文件的时候阻止或限制对文件的访问。尽管文件锁定可以应用到整个文件中，但它通常会缩小到一个更小的区域。一个锁的范围从一个起始字节偏移量开始到持续指定数量的字节。

` FileChannel`另一个有趣的特性是 *memory-mapped file I/O*，通过`map() `方法，返回一个` java.nio.MappedByteBuffer `它的内容是文件的memory-mapped region。文件内容通过内存访问被访问。缓冲区复制和读-写系统调用被消除。

你可以调用`java.nio.channels.Channels`类或者经典I/O类比如` RandomAccessFile `的方法来获得一个channel。

### Selectors 

I/O被分类为block-oriented 或 stream-oriented。从一个文件读或写入一个文件是block-oriented I/O的例子。相比之下，从键盘读或写入到网络连接是stream-oriented I/O的例子。

Stream I/O 通常比block I/O慢。另外，输入往往是间歇的。例如，用户可能会输入字符流然后暂停，或者网络连接的短暂延迟导致视频断续播放。

很多操作系统允许 streams 被配置成`nonblocking ` mode，一个线程不断地检查可用的输入，如果没有输入，就不阻塞。线程可以处理传入的数据或在数据来之前处理其他工作。

“polling for available input”（轮询可用输入）活动是浪费的，特别是线程需要监控许多输入流（比如在 web server context）。现代操作系统可以高效地做这个检查，被称为 *readiness selection* ，通常建立在非阻塞模式上。操作系统监控一个streams的集合并且返回给线程指示哪个streams准备好了做I/O操作。因此，一个单线程可以 multiplex（多路复用）许多活跃的streams，使通过普通代码在web server context管理巨量网络连接成为可能。

JDK 1.4通过提供*selectors*提供readiness selection，是`java.nio.channels.Selector`类的实例，可以检查一个或者多个channels并决定哪个channels准备好读写。通过这种方式，单个线程可以有效管理多个channels（因此，多个网络连接）。能使用更少的线程是有利的，因为线程的创建和线程上下文的切换对性能或内存使用来说都很昂贵。

![1532501340607](https://github.com/konekos/notes/blob/master/src/pic/1532501340607.png)

### Regular Expressions 

正则表达式作为NIO一部分引入。虽然你想知道做这个的原理（正则表达式对I/O做了什么），正则表达式通常用于扫描从文件或其他来源读取的文本数据。需要尽可能快地执行这些扫描，要求将其包含在内。JDK 1.4提供了正则表达式通过` java.util.regex  `包和它的`Pattern` and `Matcher  `。

### Charsets 

