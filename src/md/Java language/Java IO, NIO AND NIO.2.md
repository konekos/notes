# Java I/O, NIO and NIO.2

## PART Ⅰ Getting Started with I/O 

### Chapter 1 I/O Basics and APIs

NIO作为JDK 1.4的一部分被引入来支持操作系统的新的I/O规范。由于时间不够，没能把一些计划的特性加到这个release，被推迟到了JDK 5 and JDK 7。

这章介绍classic I/O, NIO, and more NIO (NIO.2)。接下来的章节深入研究它们的api。

#### Classic I/O

JDK1.0引进初步的I/O设施，用于访问文件系统（创建文件夹，删除文件等操作），随机访问文件目录（而不是按顺序），和源和目标之间以顺序方式的面向字节数据流。

#### File System Access and the File Class

*file system*是操作系统组件管理数据储存和后续检索。运行JVM的操作系统支持至少一个文件系统。例如，Unix或Linux结合所有安装（attached and prepared)）的disks到一个虚拟文件系统。与此相反，Windows把一个分割的文件系统和每个活动的磁盘驱动器联系起来。

Windows和类似的操作系统可以管理多个文件系统。 每个文件系统都用一个驱动器说明符来标识，比如`C:`。 指定一条没有驱动器说明符的路径，路径是相对于当前文件系统。

一个 `java.io.File  `类实例抽象一个文件或者文件路径。这个实例提供文件系统访问来在这个path上执行任务比如移除下面的文件和文件夹。比如：

```
new File("temp").mkdir();
```

#### Accessing File Content via RandomAccessFile 

文件内容可以按顺序或随机访问。随机访问可以加快搜索和排序功能。在 `java. io.RandomAccessFile `类提供随机访问文件。例如：

```java
RandomAccessFile raf = new RandomAccessFile("employees.dat", "r");
int empIndex = 10;
raf.seek(empIndex * EMP_REC_LEN);
// Read contents of employee record.

```

` employees.dat`文件被分割成固定长度的employees记录，每个记录 EMP_REC_LEN bytes长，被访问。第10个索引的employee被查找（第一个index 0）。这个任务通过seeking（设置file pointer）这个记录的第一个字节的字节位置，它处于记录长度乘索引。记录然后被访问。

#### Streaming Data via Stream Classes 

Classic I/O 包含streams用于执行I/O操作。流是任意长度的有顺序的bytes sequence。bytes从应用的*output stream*流出到目的地，和从一个source的input stream流出到应用。

![1532340528079](https://github.com/konekos/notes/blob/master/src/pic/1532340528079.png?raw=true)

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

#### Stream Classes and Standard I/O 

很多操作系统支持标准I/O， preconnected streams被称为 standard input, standard output, and standard error。

标准输入默认从键盘输入。然而，也可以重定向从不同的源读输出到不同的目的地，比如文件。

JDK 1.0引入标准I/O支持通过添加InputStream and PrintStream类型的in, out, and err o对象到`java.lang.System `类。你指定调用这些对象的方法来连接标准输入输出和error，如下：

```
int ch = System.in.read(); // Read single character from standard input.
System.out.println("Hello"); // Write string to standard output.
System.err.println("I/O error: " +
 ioe.getMessage()); // Write string to standard error.
```

#### JDK 1.1 and the Writer/Reader Classes 

JDK 1.0的I/O适合字节流，但不适合字符流因为不考虑字符编码。JDK 1.1引入了 writer/reader类克服这个问题，把字符编码考虑了在内。例如`java.io `包含有`s FileWriter and FileReader `类用于读写字符流。

#### NIO 

现代操作系统提供精致的I/O服务（比如readiness selection）用于提升I/O性能与简化I/O。 Java Specification Request (JSR) 51  (www.jcp.org/en/jsr/detail?id=51) 被创建用于落地这个特性。

JSR 51的描述表明提供了APIs用于可伸缩I/O， fast buffered binary and character I/O ，正则表达式和字符集转换。这些APIs就是NIO。JDK1.4在以下APIs方面实现NIO：

- Buffers
- Channels
- Selectors
- Regular expression
- Charsets

regular expression and charset APIs被提供给简化常规与I/O有联系的tasks。

#### Buffers 

Buffers（缓冲区）是NIO操作的基础。本质上，NIO全部都是关于把数据从Buffers移入或移出。

一个过程比如JVM执行I/O通过请求操作系统去排干buffer的内容来通过写操作存储。类似地，它请求操作系统用从存储设备读取的数据填充buffer。

考虑一个涉及磁盘驱动的读操作。操作系统发出一个指令到磁盘controller来读取磁盘的一块字节到一个操作系统的buffer。一旦操作完成，操作系统复制这个buffer contents到发生`read()`操作的指定的buffer。

![1532487035762](https://github.com/konekos/notes/blob/master/src/pic/1532487035762.png?raw=true)

图1-2中一个程序向操作系统发起一个 read() 调用。依次地，操作系统请求磁盘controller从磁盘读取一块bytes。磁盘controller（也叫DMA controller）通过*Direct Memory Access (DMA)*直接把这些字节读到操作系统buffer，DMA是计算机系统的一个特性，允许某个硬件子系统不依赖 [central processing unit](https://en.wikipedia.org/wiki/Central_processing_unit#Central%20processing%20unit) (CPU) 直接访问 系统[内存](https://en.wikipedia.org/wiki/Computer_storage#Computer%20storage)  (RAM) 。

从操作系统buffer复制 bytes到程序buffer是不那么高效的。让DMA controller直接复制到程序buffer会更有效，这个方法有2个问题：

- DMA controller特别是不能和JVM运行其上的user space（用户空间）连接，而是和 operating system’s kernel space（操作系统内核空间）连接。
- Block-oriented设备比如DMA controller使用固定大小的数据块工作。不同的是，JVM程序可能会请求一个不是block size倍数大小的数据。

因为这些问题，操作系统作为一个中介角色，当它在JVM程序和DMA controller切换时扯碎和重组数据。

装配/拆卸数据的任务可以更有效，通过让JVM程序在一个系统调用传递一个buffer addresses给操作系统。操作系统按顺序填充或者排干这些buffer，在一个读操作分散数据到多个buffer或者在写操作时从多个buffer聚集数据。这个聚集/分散的活动减少了JVM程序必须要做的系统调用（潜在地很昂贵）并且使操作系统优化数据处理因为系统知道buffer空间的总数。此外，当多个处理器或内核可用，操作系统可能允许buffers同时填充或者排干。

JDK 1.4的 java.nio.Buffer类抽象了JVM程序buffer的概念。作为`java.nio.ByteBuffer`和其他类的父类。因为I/O根本上是面向字节的，只有` ByteBuffer `实例可以用channels。大多数其他Buffer的子类方便于处理multibyte数据（例如characters or integers）。

#### Channels 

强制CPU执行I/O任务并等待I/O完成（这样的CPU被叫做 I/O bound）是资源的浪费。将这些任务卸载到DMA controller可以改善性能，处理器可以做其他工作。

一个channel充当一个管道用于连接DMA controller来从硬盘有效排干或填充字节数据。JDK 1.4的`java.nio.channels.Channel `接口，它的子接口和众多类实现了channel架构。

其中一个类是` java.nio.channels.FileChannel`，抽象了一个channel用于读写映射操作文件。 `FileChannel `的一个有趣的特性是支持file locking，在这些复杂的应用程序中，如数据库管理系统。

*File locking*让程序在访问文件的时候阻止或限制对文件的访问。尽管文件锁定可以应用到整个文件中，但它通常会缩小到一个更小的区域。一个锁的范围从一个起始字节偏移量开始到持续指定数量的字节。

` FileChannel`另一个有趣的特性是 *memory-mapped file I/O*，通过`map() `方法，返回一个` java.nio.MappedByteBuffer `它的内容是文件的memory-mapped region。文件内容通过内存访问被访问。缓冲区复制和读-写系统调用被消除。

你可以调用`java.nio.channels.Channels`类或者经典I/O类比如` RandomAccessFile `的方法来获得一个channel。

#### Selectors 

I/O被分类为block-oriented 或 stream-oriented。从一个文件读或写入一个文件是block-oriented I/O的例子。相比之下，从键盘读或写入到网络连接是stream-oriented I/O的例子。

Stream I/O 通常比block I/O慢。另外，输入往往是间歇的。例如，用户可能会输入字符流然后暂停，或者网络连接的短暂延迟导致视频断续播放。

很多操作系统允许 streams 被配置成`nonblocking ` mode，一个线程不断地检查可用的输入，如果没有输入，就不阻塞。线程可以处理传入的数据或在数据来之前处理其他工作。

“polling for available input”（轮询可用输入）活动是浪费的，特别是线程需要监控许多输入流（比如在 web server context）。现代操作系统可以高效地做这个检查，被称为 *readiness selection* ，通常建立在非阻塞模式上。操作系统监控一个streams的集合并且返回给线程指示哪个streams准备好了做I/O操作。因此，一个单线程可以 multiplex（多路复用）许多活跃的streams，使通过普通代码在web server context管理巨量网络连接成为可能。

JDK 1.4通过提供*selectors*提供readiness selection，是`java.nio.channels.Selector`类的实例，可以检查一个或者多个channels并决定哪个channels准备好读写。通过这种方式，单个线程可以有效管理多个channels（因此，多个网络连接）。能使用更少的线程是有利的，因为线程的创建和线程上下文的切换对性能或内存使用来说都很昂贵。

![1532501340607](https://github.com/konekos/notes/blob/master/src/pic/1532501340607.png?raw=true)

#### Regular Expressions 

正则表达式作为NIO一部分引入。虽然你想知道做这个的原理（正则表达式对I/O做了什么），正则表达式通常用于扫描从文件或其他来源读取的文本数据。需要尽可能快地执行这些扫描，要求将其包含在内。JDK 1.4提供了正则表达式通过` java.util.regex  `包和它的`Pattern` and `Matcher  `。

#### Charsets 

之前提到JDK 1.1引入writer/reader类把字符编码考虑在内。最初，比如`java.io.InputStreamReader  `的类协同` java.io.ByteToCharConverter`类工作来执行基于编码的转换。` ByteToCharConverter `从JDK 6开始最终弃用了。在这个位置，更能干的包` java.nio.charset  `和它的`Charset`, `CharsetEncoder`, `CharsetDecoder`, and related types被引入。

#### Formatter 

JSR 51提到一个简单的 printf-style格式化工具。这样一个设施在准备数据去呈现的时候提供有效值。然而，JDK 1.4没有包含这个能力，因为它依赖许多参数列表，一个JDK 5前还没引入的语言特性。幸运的是，JDK 5 包含了`java.util.Formatter `，他有丰富的格式化能力，和支持自定义格式化的相关类型，以及添加 `printf()` (and related `format()`) 方法到 `PrintStream` 类。

#### NIO.2 

JSR 51指出NIO将会引入改善的文件系统接口克服遗留`File`类的众多问题。然而，缺乏时间导致这个特性没包含。同时，也不可能支持 asynchronous I/O和完成 socket channel功能。JSR 203 (www.jcp.org/en/jsr/detail?id=203)  随后落实了这些遗漏，在JDK 7登场。

注意：被官方JDK 7之前， big buffers (buffers with 64-bit addressability) 被认为用于 NIO.2。比如 `BigByteBuffer` and `MappedBigByteBuffer`类计划被包含在` java.nio `或其他的包。然而，如同在e “BigByteBuffer/Mapped BigByteBuffer” OpenJDK 讨论话题中解释的，这个能力被遗弃了取而代之的是追求“64-bit arrays or collections”。

#### Improved File System Interface 

遗留的` File类`有很多问题。例如`renameTo() `方法跨操作系统不一致。同时，许多` File `的方法没有扩展；从服务器请求大目录列表导致 hang住。JSR 203的新操作系统接口修补了这些和其他的一些问题。例如，它支持对文件属性的批量访问，提供变更通知功能，提供了逃离到 file system-specific APIs的能力，和一个service provider接口用于可插拔文件系统实现。

#### Asynchronous I/O 

Nonblocking mode通过阻止执行读写操作的线程在输入可用或输出被全部写入之前阻塞住，从而提高性能。然而，它不让应用决定它是否可以在不实际执行操作的情况下执行操作。例如，当一个非阻塞读操作成功了，应用程序不仅了解读取操作是可能的还已经阅读了一些必须管理的数据。这个 duality 是你不用把检查 stream readiness 的代码从数据执行代码分离，不用让你的代码明显复杂。

*Asynchronous I/O*  改善了这个问题，通过让线程开始操作并马上执行其他工作。线程指定一些 callback函数在操作完成了被调用。

#### Completion of Socket Channel Functionality 

JDK 1.4添加了 `DatagramChannel`, `ServerSocketChannel`, and `SocketChannel`类到 `java.nio.channels `包。然而，缺乏时间这些类不能支持绑定和选项配置。同时，也不支持o, channel-based multicast datagrams。JDK 7添加了绑定支持和选项配制到提到的类。同时，引入新的`java.nio.channels .MulticastChannel`接口。

#### Summary

I/O is fundamental to operating systems, computer languages, and language libraries. Java supports I/O through its classic I/O, NIO, and NIO.2 API categories. 

Classic I/O provides APIs to access the file system, access file content randomly (as opposed to sequentially), stream byte-oriented data between sources and destinations, and support character streams. 

NIO provides APIs to manage buffers, communicate buffered data over channels, leverage readiness selection via selectors, scan textual data quickly via regular expressions, specify character encodings via charsets, and support printf-style formatting. 

NIO.2 provides APIs to improve the file system interface; support asynchronous I/O; and complete socket channel functionality by upgrading DatagramChannel, ServerSocketChannel, and SocketChannel, and by introducing a new MulticastChannel interface. 

## PART Ⅱ Classic I/O APIs 

### Chapter 2 File 

Java使用` java.io.File  `访问文件系统。

#### Constructing File Instances 

一个 File 类实例包含一个文件或者目录路径的抽象表征。创建File实例，

```java
File file1 = new File("/x/y");
File file2 = new File("C:\\temp\\x.dat");
```

第一行假设是 Unix/Linux 操作系统。window也可以，假设是当前驱动器。

**注意**：一个操作系统相关的分隔符（比如windows的`\`出现在路径的连续name之间 ）

第二行假设是Windows系统，你也可以在windows用`/`。

都是绝对路径，从root开始；不需要其他信息定位文件。相比之下，相对路径不从root开始；需要另一个path的信息。

**注意**：`java.io`包的类默认解析相对路径在当前用户目录（working directory），由系统属性`user.dir `指定，特别它是JVM被启动的路径。（调用`java.lang.System`类的` getProperty()`方法获取系统属性）。

路径字符串到/从绝对路径的转换内部是依赖操作系统的。

**注意**：default name-separator character由系统属性 `file.separator `定义，在File的public static separator and separatorChar fields 可用——第一个是` java.lang.String` 第二个是`char`。

File提供额外构造器。例如：

- File(String parent, String child) creates a new

File instance from a parent path string and a child
path string.

- File(File parent, String child) creates a new File

instance from a parent path File instance and a child
path string

parent参数都被转化为parent path：

```
File file3 = new File("prj/books/", "io");
```

路径被合并为`prj/books/io`，如果parent是`prj/books`，会自动在最后加上`/`。

**注意**：因为File(String path), File(String parent, String child), and File(File parent, String child) 不检查无效路径参数（除了path或者child是null抛出` java.lang.NullPointerException `），你必须小心指定path。要力求所有操作系统都有效。例如，不使用硬编码（例如 C:），使用 `listRoots()`返回的root。最好，让你的path和user/working directory 相关（从` user.dir `返回的）。

#### Learning About Stored Abstract Paths 

得到File对象后，你想询问他获取抽象存储路径，可以通过表2-1方法：

***Table 2-1. File Methods for Learning About a Stored Abstract Path*** 

| Method                    | Description                                                  |
| :------------------------ | ------------------------------------------------------------ |
| File getAbsoluteFile()    | 返回文件对象 abstract path的 absolute form。这个方法等同于 `new File(this.getAbsolutePath())` |
| String getAbsolutePath()  | 返回绝对路径。当已经是绝对路径，返回和`getPath()`一样，当是空的 abstract path，返回当前用户目录字符串（` user.dir `）。否则，用操作系统方式解决。在  Unix/ Linux 操作系统，相对路径是在用户目录，window是在当前path命名的drive，如果没有drive是在用户目录。 |
| File getCanonicalFile()   | 返回  canonical （ (simplest possible, absolute and unique）form。I/O错误发生时抛出` java.io.IOException `（创建 canonical path需要文件系统查询）；等同于` File(this.getCanonicalPath()). `。 |
| String getCanonicalPath() | 返回 canonical path string 。首先必要时转换为绝对路径，就像调用` getAbsolutePath() `，然后使用取决于操作系统的方式把它映射到 它的 unique form 。它特别地移除冗余 ，比如`..` `.`符号，解析文件链接（ Unix/Linux ），转换  drive letters 为标准形式（windows）。 |
| String getName()          | 返回文件或文件夹名。是path name sequence最后的name。path name sequence为空返回空。 |
| String getParent()        | 返回  parent path string ，如果没有parent，返回“null”（可以打印）。 |
| File getParentFile()      | 返回parent文件夹；不是文件夹返回"null"。                     |
| String getPath()          | 返回  File’s separator field 分隔的字符串。                  |
| boolean isAbsolute()      | 绝对路径返回true；相对返回false。根据系统的。windows带有driver，后面有`\`，或者`\\`开头。Linux/Unix是`/`。 |
| String toString()         | 同 getPath()                                                 |

Table 2-1 引用 IOException 。

Listing 2-1实例化File。

***Listing 2-1. Obtaining Abstract Path Information*** 

```java
import java.io.File;
import java.io.IOException;
public class PathInfo
{
 public static void main(final String[] args) throws IOException
 {
 if (args.length != 1)
 {
 System.err.println("usage: java PathInfo path");
 return;
 }
 File file = new File(args[0]);
 System.out.println("Absolute path = " + file.getAbsolutePath());
 System.out.println("Canonical path = " + file.getCanonicalPath());
 System.out.println("Name = " + file.getName());
 System.out.println("Parent = " + file.getParent());
 System.out.println("Path = " + file.getPath());
 System.out.println("Is absolute = " + file.isAbsolute());
 }
}

```

编译`javac PathInfo.java`，run `java PathInfo . `，输出：

```
Absolute path = C:\prj\books\io\ch02\code\PathInfo\.
Canonical path = C:\prj\books\io\ch02\code\PathInfo
Name = .
Parent = null
Path = .
Is absolute = false
```

Canonical path没有`.`，没有parent。

继续` java PathInfo C:\reports\2015\..\2014\February `，输出：

```
Absolute path = C:\reports\2015\..\2014\February
Canonical path = C:\reports\2014\February
Name = February
Parent = C:\reports\2015\..\2014
Path = C:\reports\2015\..\2014\February
Is absolute = true

```

最后`java PathInfo "" `，输出:

```
Absolute path = C:\prj\books\io\ch02\code\PathInfo
Canonical path = C:\prj\books\io\ch02\code\PathInfo
Name =
Parent = null
Path =
Is absolute = false
```

 `getName()` and `getPath() `返回空“”，因为empty path is empty 。

#### Learning About a Path’s File or Directory 

你可以询问文件系统了解文件。

***Table 2-2. File Methods for Learning About a File or Directory*** 

| Method                | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| boolean exists()      | 存在为true。                                                 |
| boolean isDirectory() | 是存在的文件夹返回true。                                     |
| boolean isFile()      | 一个normal的存在的文件返回true。normal指的是不是文件夹，满足其他操作系统的约束。例如不能是 symbolic link （符号链接）或 named pipe （命名管道）。Java创建的非文件夹文件都保证是一个normal文件。 |
| boolean isHidden()    | 被隐藏返回true。隐藏的确切定义是依赖于操作系统的。  Unix/Linux是以`.`开头。windows是在文件系统被标记。 |
| long lastModified()   | 上次更改的时间，文件不存在或者I/O错误是0。 返回的值是以毫秒为单位的，自从  Unix epoch (00:00:00 GMT, January 1, 1970)。 |
| long length()         | file的length。是文件夹的返回值是 unspecified ，不存在是0。   |

示例如下。

***Listing 2-2. Obtaining File/Directory Information*** 

```java
import java.io.File;
import java.io.IOException;
import java.util.Date;
public class FileDirectoryInfo
{
 public static void main(final String[] args) throws IOException
 {
 if (args.length != 1)
 {
 System.err.println("usage: java FileDirectoryInfo pathname");
 return;
 }
 File file = new File(args[0]);
 System.out.println("About " + file + ":");
 System.out.println("Exists = " + file.exists());
 System.out.println("Is directory = " + file.isDirectory());
 System.out.println("Is file = " + file.isFile());
 System.out.println("Is hidden = " + file.isHidden());
 System.out.println("Last modified = " +
 new Date(file.lastModified()));
 System.out.println("Length = " + file.length());
 }
}

```

编译`javac FileDirectoryInfo.java `

run `java FileDirectoryInfo x.dat `

假设` x.dat ` 3个字节大。

输出：

```
About x.dat:
Exists = true
Is directory = false
Is file = true
Is hidden = false
Last modified = Sat Jul 25 15:49:41 CDT 2015
Length = 3
```

#### Listing File System Root Directories 

File的方法`File[] listRoots()  `返回root文件夹。

**注意**： available file system roots的集合受系统级别操作影响，比如插入或拔出媒体，断开或卸载物理或虚拟磁盘驱动器。

***Listing 2-3. Dumping Available File System Roots to Standard Output*** 

```java
public class DumpRoots
{
 public static void main(String[] args)
 {
 File[] roots = File.listRoots();
 for (File root: roots)
 System.out.println(root);
 }
}

```

编译&运行，输出：

Win 7 

```
C:\
D:\
E:\
F:\
```

Linux 

```
/
```

#### Obtaining Disk Space Information 

一个 *partition* (分区)对于文件系统是系统特定的储存的一部分。获得分区空闲的空间大小很重要。在Java 6之前，唯一简便的方式来完成这个事情是通过创建不同大小的文件来猜。

Java 6添加了File类的` long getFreeSpace(), long getTotalSpace()`和 `long getUsableSpace()  `方法，返回File分区空间信息，由File实例的abstract path描述：

- long getFreeSpace()  返回File对象abstract path鉴定的分区的未分配字节数。
- long getTotalSpace() 返回File对象abstract path鉴定的分区的size；当abstract path没有命名分区返回0。
- long getUsableSpace() 返回File对象abstract path鉴定的分区对当前JVM可用的字节数；当abstract path没有命名分区返回0。

虽然 `getFreeSpace() `和` getUsableSpace() `似乎相等，他们在以下方面不同：不像`getFreeSpace()`， `getUsableSpace() `检查写权限和其他的操作系统限制，得出一个更准确的估计。

**注意**： `getFreeSpace()` and` getUsableSpace()` 方法返回一个Java应用可以使用可用或未分配空间的提示（不是保证）。因为JVM外面的程序也能用分区空间，导致实际未分配或可用空间比返回值要小。

***Listing 2-4. Outputting the Free, Usable, and Total Space on All Partitions*** 

```java
import java.io.File;
public class PartitionSpace
{
 public static void main(String[] args)
 {
 File[] roots = File.listRoots();
 for (File root: roots)
 {
 System.out.println("Partition: " + root);
 System.out.println("Free space on this partition = " +
 root.getFreeSpace());
 System.out.println("Usable space on this partition = " +
 root.getUsableSpace());
 System.out.println("Total space on this partition = " +
 root.getTotalSpace());
 System.out.println("***");
 }
 }
}

```

编译运行，输出（Win 7）：

```
Partition: C:\
Free space on this partition = 143271129088
Usable space on this partition = 143271129088
Total space on this partition = 499808989184
***
Partition: D:\
Free space on this partition = 0
Usable space on this partition = 0
Total space on this partition = 0
***
Partition: E:\
Free space on this partition = 733418569728
Usable space on this partition = 733418569728
Total space on this partition = 1000169533440
***
Partition: F:\
Free space on this partition = 33728192512
Usable space on this partition = 33728192512
Total space on this partition = 64021835776
***
```

#### Listing Directories 

File声明了5个方法返回位于文件夹里的file和文件夹。

***Table 2-3. File Methods for Obtaining Directory Content*** 

| Method                                  | Description                                                  |
| --------------------------------------- | ------------------------------------------------------------ |
| String[] list()                         | 路径不表示目录或者发生I/O错误，返回null。否则返回strings数组 |
| String[]list(FilenameFilter   filter)   | 只返回符合filter的Strings                                    |
| File[] listFiles()                      | 把Strings数组转换为Files数组返回                             |
| File[] listFiles(FileFilter filter)     | 返回符合filter的File数组                                     |
| File[] listFiles(FilenameFilter filter) | 返回符合filter的File数组                                     |

`FilenameFilter`接口声明一个` boolean accept(File dir, String name) `方法。`dir`是path的parent部分（文件夹path），`name`是最后的文件夹的name或者这个path最后的文件名。

`accept()`使用参数决定是否符合规则，返回true就会在数组里包含。

***Listing 2-5. Listing Specific Names*** 

```java
import java.io.File;
import java.io.FilenameFilter;
public class Dir
{
 public static void main(final String[] args)
 {
 if (args.length != 2)
 {
 System.err.println("usage: java Dir dirpath ext");
 return;
 }
 File file = new File(args[0]);
 FilenameFilter fnf = new FilenameFilter()
 {
 @Override
 public boolean accept(File dir, String name)
 {
 return name.endsWith(args[1]);
 }
 };
 String[] names = file.list(fnf);
 for (String name: names)
 System.out.println(name);
 }
}

```

编译运行：`java Dir C:\windows exe `输出：

```
bfsvc.exe
explorer.exe
fveupdate.exe
HelpPane.exe
hh.exe
IsUninst.exe
kindlegen.exe
notepad.exe
regedit.exe
splwow64.exe
twunk_16.exe
twunk_32.exe
winhlp32.exe
write.exe

```

`listFiles(FileFilter) `引入不对称。

`java.io.FileFilter `声明一个`accept(String path)`方法，参数是完整的path。

**注意**：如果想有目录和文件夹分隔的path，用` FilenameFilter `；要使用完整的路径，用`getParent() `and `getName() `。

#### Creating/Modifying Files and Directories 

File也提供了创建文件，文件夹，和修改存在的文件文件夹。

***Table 2-4. File Methods for Creating New and Manipulating Existing Files and Directories*** 

| Method                             | Description                                                  |
| ---------------------------------- | ------------------------------------------------------------ |
| boolean mkdir()                    | 创建文件夹；成功返回true。                                   |
| boolean mkdirs()                   | 层级创建。成功返回true。                                     |
| boolean renameTo(File dest)        | dest是null抛出空指针异常。成功为true。方法行为很多取决于操作系统。例如， rename 方法可能不能够把文件从一个文件系统移到另一个，可能不是原子操作，目标存在时，可能不成功。返回值一定要检查，看是否成功。 |
| boolean setLastModified(long time) | 设置上次修改的时间。成功为true。time是负数抛出`  IllegalArgumentException `。时间可能会被 truncated。 |

假设你想写一个，可以有临时文件去保存快照的程序。使用`createTempFile()`创建临时文件。如果你没有指定储存路径，会保存到系统属性` java.io.tmpdir  `。