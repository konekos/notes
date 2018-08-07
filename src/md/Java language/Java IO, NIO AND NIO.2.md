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

假设你想写一个，可以有临时文件去保存快照的程序。使用`createTempFile()`创建临时文件。如果你没有指定储存路径，会保存到系统属性` java.io.tmpdir  `。你可能想移除临时文件，`deleteOnExit() `方法，在JVM没有崩溃或断电退出时删除临时文件。

***Listing 2-6. Experimenting with Temporary Files*** 

```java
import java.io.File;
import java.io.IOException;
public class TempFileDemo
{
 public static void main(String[] args) throws IOException
 {
 System.out.println(System.getProperty("java.io.tmpdir"));
 File temp = File.createTempFile("text", ".txt");
 System.out.println(temp);
 temp.deleteOnExit();
 }
}
```

#### Setting and Getting Permissions 

Java 1.2添加` boolean setReadOnly()  `方法到File，标记一个文件或文件夹为只读。然而恢复可写状态的方法没有。更重要的是，在Java 6之前， File没有办法管理抽象路径的读、写和执行权限。

Java 6添加之下方法：

- boolean setExecutable(boolean executable, boolean ownerOnly)  false为全部用户
- boolean setExecutable(boolean executable)  设置拥有者的执行权限
- boolean setReadable(boolean readable, boolean ownerOnly)  
- boolean setReadable(boolean readable) 
- boolean setWritable(boolean writable, boolean ownerOnly) 
- boolean setWritable(boolean writable) 

除了这些方法，Java 6改造了` boolean canRead()  `和`boolean canWrite() `方法，引入` boolean canExecute() `方法返回xxx权限。

***Listing 2-7. Checking a File’s or Directory’s Permissions*** 

```java
import java.io.File;
public class Permissions
{
 public static void main(String[] args)
 {
 if (args.length != 1)
 {
 System.err.println("usage: java Permissions filespec");
 return;
 }
 File file = new File(args[0]);
 System.out.println("Checking permissions for " + args[0]);
 System.out.println(" Execute = " + file.canExecute());
 System.out.println(" Read = " + file.canRead());
 System.out.println(" Write = " + file.canWrite());
 }
}
```

#### Exploring Miscellaneous Capabilities 

最后，File实现`java.lang.Comparable `接口的` compareTo()`方法，重写了` equals()  `和` hashCode() `。

***Table 2-5. File’s Miscellaneous Methods*** 

| Method                      | Description                                                  |
| --------------------------- | ------------------------------------------------------------ |
| int compareTo(File path)    | 字典比较两个path。方法的排序定义取决于操作系统。对  Unix/Linux，字母排序是有意义的。windows无意义。抽象路径相同返回0。调用 getCanonicalFile() 再比较。 |
| boolean equals(Object obj)) | 取决于操作系统。path一样。                                   |
| int hashCode()              | 返回path 的哈希。计算取决于底层操作系统。在  Unix/Linux 等于path 字符串的哈希值异或 1234321。windows，是小写的path字符串。小写path不考虑  current locale 。 |

***Listing 2-8. Comparing Files*** 

```java
import java.io.File;
import java.io.IOException;
public class Compare
{
 public static void main(String[] args) throws IOException
 {
 if (args.length != 2)
 {
 System.err.println("usage: java Compare filespec1 filespec2");
 return;
 }

 File file1 = new File(args[0]);
 File file2 = new File(args[1]);
 System.out.println(file1.compareTo(file2));
 System.out.println(file1.getCanonicalFile()
 .compareTo(file2.getCanonicalFile()));
 }
}

```

#### exercise

```
1. What is the purpose of the File class?
2. What do instances of the File class contain?
3. What is a path?
4. What is the difference between an absolute path and a relative path?
5. How do you obtain the current user (also known as working) directory?
6. Define parent path.
7. File’s constructors normalize their path arguments. What does
normalize mean?
8. How do you obtain the default name-separator character?
9. What is a canonical path?
10. What is the difference between File’s getParent() and
getName() methods?
11. True or false: File’s exists() method only determines whether or
not a file exists.
12. What is a normal file?
13. What does File’s lastModified() method return?
14. What does File’s listRoots() method accomplish?
15. True or false: File’s list() method returns an array of Strings
where each entry is a file name rather than a complete path.
16. What is the difference between the FilenameFilter and
FileFilter interfaces?
17. True or false: File’s createNewFile() method doesn’t check for
file existence and create the file when it doesn’t exist in a single
operation that’s atomic with respect to all other file system activities
that might affect the file.
18. File’s createTempFile(String, String) method creates a
temporary file in the default temporary directory. How can you locate
this directory?
19. Temporary files should be removed when no longer needed after an
application exits (to avoid cluttering the file system). How do you
ensure that a temporary file is removed when the JVM ends normally
(it doesn’t crash and the power isn’t lost)?
20. Which one of the boolean canRead(), boolean canWrite(), and
boolean canExecute() methods was introduced by Java 6?
21. How would you accurately compare two File objects?
22. Create a Java application named Touch for setting a file’s or
directory’s timestamp to the current time. This application has the
following usage syntax: java Touch pathname.
```

#### Summary 

### Chapter 3 RandomAccessFile 

File可以被创建和/或打开用于*random access* ，在文件关闭前，在多个位置可以发生混合读写操作。Java提供` java.io.RandomAccessFile `类支持这种随机访问。

#### Exploring RandomAccessFile 

构造器：

- RandomAccessFile(File file, String mode) ：创建和打开一个不存在的文件或打开存在的文件。由abstract path确定，根据mode创建和/或打开文件。
- RandomAccessFile(String path, String mode)：创建和打开一个不存在的文件或打开存在的文件。由path确定，根据mode创建和/或打开文件。

 constructor’s mode参数必须是 `r`,`rw`,`rws`,`rwd`其中之一，否则抛出`java.lang. IllegalArgumentException `，参数意义如下：

- `r`，通知构造器打开一个存在的文件只用于读。任何写的尝试抛出`java.io.IOException`。
- `rw`，如果不存在，通知构造器创建并打开一个新的file的时候用于读写或者打开一个存在的文件用于读写。
- `rwd`，如果不存在，通知构造器创建并打开一个新的file的时候用于读写或者打开一个存在的文件用于读写。另外，每个对于文件内容的更新必须同步写入底层储存设备。
- `rws`如果不存在，通知构造器创建并打开一个新的file的时候用于读写或者打开一个存在的文件用于读写。另外，每个对于文件内容或者 metadata的更新必须同步写入底层储存设备。

**注意**：文件的metadata是关于文件的但不是实际文件内容。metadata例如有：file的length，上次更改的时间。

`rws`和`rwd`同步写入，保证了关键数据在操作系统崩溃不会丢失。当文件不在本地设备上时是不能保证的。

**注意**：`rws`和`rwd`打开的文件要比`rw`模式打开的文件慢。

当`r`模式path打不开的时候（不存在或是文件夹），`rw`模式的path是只读权限的时候或者是文件夹，抛出` java.io.FileNotFoundException `。

```java
RandomAccessFile raf = new RandomAccessFile("employee.dat", "r");
```

一个 random access file和一个file指针关联，一个游标指示下个要读/写的字节的位置。当一个存在文件被打开，file指针被设置为第一个字节，在offset 0 。当文件被创建时，file指针也被设为0。

读写操作在文件pointer开始，并把他推进已经读或写过的字节数。写超出了文件当前末尾的操作会导致文件被扩展。操作会继续直到File关闭。

`RandomAccessFile`的代表方法：

***Table 3-1. RandomAccessFile Methods*** 



| Method                         | Description                                                  |
| ------------------------------ | ------------------------------------------------------------ |
| int readInt()                  | 从文件读返回  32-bit integer。从当前file指针开始读取4个字节。 如果字节读取依次是b1 b2 b3和b4 ，`0 <= b1, b2, b3, b4 <= 255` ，结果等于 `(b1 << 24) | (b2 << 16) | (b3 << 8) | b4`. 这个方法在4个字节读完、检测到文件末尾、或抛出异常之前会阻塞。在读4个字节前到达文件末尾抛出  `EOFException `，I/O错误抛出  `IOException `。 |
| void seek(long pos)            | 设置文件指针的当前offset（从文件开头开始用字节测量）。如果 offset 设置超出了文件尾，file的length不会变。只有通过写才行。pos是负值或者I/O错误抛出`  IOException `。 |
| void setLength(long newLength) | 设置文件length。如果` length() `返回的比newLength大，file被 truncated。在这种情况下，如果` getFilePointer() `返回的offset比newLength大，在` setLength()  `返回后，offset会等于 newLength。如果当前长度比newLength小，文件被扩展。这种情况下，扩展部分的文件内容没有被定义。I/O错误抛出` IOException `。 |
| int skipBytes(int n)           | 尝试跳过n个字节。 This method skips over a smaller number of bytes (possibly zero) when the end of file is reached before n bytes have been skipped。在这种情况不抛出` EOFException `。n是负数，不跳。返回实际的跳跃数。I/O错误抛出` IOException `。 |
| void write(byte[] b)           | 从当前文件指针开始写   byte array b  的  b.length bytes。I/O错误抛出` IOException `。 |
| void write(int b)              | 写一个更低的8 bits的 b作为一个32 bits的integer到文件从当前文件指针开始。I/O错误抛出` IOException `。 |
| void writeChars(String s)      | 写 s 作为 characters 的序列到文件从当前文件指针开始。I/O错误抛出` IOException ` |
| void writeInt(int i)           | 写32 bits的integer到file从当前文件指针开始。 这四个字节是先用高字节写的。I/O错误抛出` IOException ` |

上面方法是不言而喻的。然而，`getFD() ` 方法需要进一步启示。

**注意**：`RandomAccessFile `的read开头的方法和`skipBytes() `起源于` java.io.DataInput `接口，`RandomAccessFile`实现了它。另外，writer开头的方法源于`java.io.DataOutput `接口，也实现了这个接口。

当文件被打开，底层操作系统创建一个基于操作系统的结构来代表文件。对这个结构的handle保存在`java.io.FileDescriptor `类的一个实例，` getFD()`返回这个实例。

**注意**：一个handle是一个Java传递给底层操作系统来识别的标识符，在这种情况，是一个特定的打开的文件，当需要底层操作系统执行文件操作时候。

`FileDescriptor `是一个小的类，声明了3个`FileDescriptor `常量叫做`in`,`out`,`error`。这三个常量使`System.in`, `System. out`, and `System.err `提供到标准输入输出错误流的访问。

`FileDescriptor` 也声明以下2个方法：

- void sync() 告诉底层操作系统去flush(empty)打开文件输出buffers的内容到关联的本地磁盘设备。 `sync() `返回所有修改了的数据和属性。当buffers不能被flushed或者因为操作系统不能保证所有buffers和物理媒介同步时， 抛出`java. io.SyncFailedException `。
- boolean valid() 决定file descriptor object是不是有效的。当 file descriptor object对象表示打开的文件或活动的I/O连接返回true；否则返回false。

写入到打开文件的数据被保存在底层操作系统的输出buffers。当buffers充满到了capacity（容量），操作系统清空它们到磁盘。buffers改善了性能因为磁盘访问比内存慢。

然而，当你写数据到`rwd`或`rws`模式打开的 random access file，每个写操作的数据都是直接写到磁盘。结果是，写操作比`rw`模式要慢。

假设一个场景，要结合通过output buffers写数据和直接写数据到磁盘。下面的例子落地了这个混合场景，用`rw`模式打开，选择性调用 `FileDescriptor`的`sync()  `方法。

```java
RandomAccessFile raf = new RandomAccessFile("employee.dat", "rw");
FileDescriptor fd = raf.getFD();
// Perform a critical write operation.
raf.write(...);
// Synchronize with the underlying disk by flushing the operating system
// output buffers to the disk.
fd.sync();
// Perform a non-critical write operation where synchronization isn't
// necessary.
raf.write(...);
// Do other work.
// Close the file, emptying output buffers to the disk.
raf.close();
```

#### Using RandomAccessFile 

`RandomAccessFile `对建一个*flat file database* 很有用，一个单个文件被组织成records and fields。一个record保存单个entry（就像一个parts database的part），一个field保存entry的单个属性（就像a part number）。

**注意**：term field也用于引用类中声明的变量 。为了避免和重载术语混淆，把一个field变量想象成类似于一个record field属性。

 flat file database 典型地组织内容为固定长度的records序列。每个record被进一步组织成一个或多个固定长度的records。

![1532677457611](https://github.com/konekos/notes/blob/master/src/pic/1532677457611.png?raw=true)

演示用`RandomAccessFile `实现一个 flat file database：

***Listing 3-1. Implementing the Parts Flat File Database*** 

```java
import java.io.IOException;
import java.io.RandomAccessFile;

public class PartsDB {
    public final static int PNUMLEN = 20;
    public final static int DESCLEN = 30;
    public final static int QUANLEN = 4;
    public final static int COSTLEN = 4;
    private final static int RECLEN = 2 * PNUMLEN + 2 * DESCLEN + QUANLEN +
            COSTLEN;
    private RandomAccessFile raf;

    public PartsDB(String path) throws IOException {
        raf = new RandomAccessFile(path, "rw");
    }

    public void append(String partnum, String partdesc, int qty, int ucost)
            throws IOException {
        raf.seek(raf.length());
        write(partnum, partdesc, qty, ucost);
    }

    public void close() {
        try {
            raf.close();
        } catch (IOException ioe) {
            System.err.println(ioe);
        }
    }

    public int numRecs() throws IOException {
        return (int) raf.length() / RECLEN;
    }

    public Part select(int recno) throws IOException {
        if (recno < 0 || recno >= numRecs())
            throw new IllegalArgumentException(recno + " out of range");
        raf.seek(recno * RECLEN);
        return read();
    }

    public void update(int recno, String partnum, String partdesc, int qty,
                       int ucost) throws IOException {
        if (recno < 0 || recno >= numRecs())
            throw new IllegalArgumentException(recno + " out of range");
        raf.seek(recno * RECLEN);
        write(partnum, partdesc, qty, ucost);
    }

    private Part read() throws IOException {
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < PNUMLEN; i++)
            sb.append(raf.readChar());
        String partnum = sb.toString().trim();
        sb.setLength(0);
        for (int i = 0; i < DESCLEN; i++)
            sb.append(raf.readChar());
        String partdesc = sb.toString().trim();
        int qty = raf.readInt();
        int ucost = raf.readInt();
        return new Part(partnum, partdesc, qty, ucost);
    }

    private void write(String partnum, String partdesc, int qty, int ucost)
            throws IOException {
        StringBuffer sb = new StringBuffer(partnum);
        if (sb.length() > PNUMLEN)
            sb.setLength(PNUMLEN);
        else if (sb.length() < PNUMLEN) {
            int len = PNUMLEN - sb.length();
            for (int i = 0; i < len; i++)
                sb.append(" ");
        }
        raf.writeChars(sb.toString());
        sb = new StringBuffer(partdesc);
        if (sb.length() > DESCLEN)
            sb.setLength(DESCLEN);
        else if (sb.length() < DESCLEN) {
            int len = DESCLEN - sb.length();
            for (int i = 0; i < len; i++)
                sb.append(" ");
        }
        raf.writeChars(sb.toString());
        raf.writeInt(qty);
        raf.writeInt(ucost);
    }

    public static class Part {
        private String partnum;
        private String desc;
        private int qty;
        private int ucost;

        public Part(String partnum, String desc, int qty, int ucost) {
            this.partnum = partnum;
            this.desc = desc;
            this.qty = qty;
            this.ucost = ucost;
        }

        String getDesc() {
            return desc;
        }

        String getPartnum() {
            return partnum;
        }

        int getQty() {
            return qty;
        }

        int getUnitCost() {
            return ucost;
        }
    }
}
```

`PartsDB` 先声明了常量确定string和32-bits integer field的length。然后声明常量用字节计算的record length。计算考虑到了字符在文件中占两个字节。

然后是field `raf `，是`RandomAccessFile `类型。这个field在随后的构造函数被分配为一个`RandomAccessFile `类实例，他会创建/打开一个新的文件或者打开一个存在文件，因为是`rw`mode。

`PartsDB `然后声明`append(), close(), numRecs(), select(), and update() `。这个方法添加一个record到file，关闭file，返回file的record数，选择和返回一个特定的record，更新一个特定的record：

- append() 方法先调用`length() and seek() `。确保在调用private `write()`方法把record写入前，文件指针在文件末尾。
- `RandomAccessFile’s close()`方法可以抛出`IOException` 。因为异常很少见，选择在`close()`方法里处理掉异常，让方法签名简单。然而，错误发生会打印异常信息。
-  `numRecs()`方法返回records数。这些记录从0编号到`numRecs() -1`。`select() `和`update()`方法的`recno`参数都在这个范围。
-  `select() `方法调用private `read()`返回` recno`决定的record实例（`Part`）。
- `update() `方法同样简单。和`select()`一样，把file指针定位到`recno`决定的record。和`append()`一样，调用`write()`写入参数替换record而不是添加。

使用 private `write() `写入records。因为fields必须有精确的size，`write()`拉长比一个field size短的基于String的值，在右边加空格。必要时将这些值裁剪成field大小 。

使用private` read() `读record。` read()  `在给Part赋值时移除基于String值填充。

就其本身而言，`PartsDB`是无用的。做个实验体验一下：

***Listing 3-2. Experimenting with the Parts Flat File Database*** 

```java
import java.io.IOException;

public class UsePartsDB {
    public static void main(String[] args) {
        PartsDB pdb = null;
        try {
            pdb = new PartsDB("parts.db");
            if (pdb.numRecs() == 0) {
                // Populate the database with records.
                pdb.append("1-9009-3323-4x", "Wiper Blade Micro Edge", 30,
                        2468);
                pdb.append("1-3233-44923-7j", "Parking Brake Cable", 5, 1439);
                pdb.append("2-3399-6693-2m", "Halogen Bulb H4 55/60W", 22, 813);
                pdb.append("2-599-2029-6k", "Turbo Oil Line O-Ring ", 26, 155);
                pdb.append("3-1299-3299-9u", "Air Pump Electric", 9, 20200);
            }
            dumpRecords(pdb);
            pdb.update(1, "1-3233-44923-7j", "Parking Brake Cable", 5, 1995);
            dumpRecords(pdb);
        } catch (IOException ioe) {
            System.err.println(ioe);
        } finally {
            if (pdb != null)
                pdb.close();
        }
    }

    static void dumpRecords(PartsDB pdb) throws IOException {
        for (int i = 0; i < pdb.numRecs(); i++) {
            PartsDB.Part part = pdb.select(i);
            System.out.print(format(part.getPartnum(), PartsDB.PNUMLEN, true));
            System.out.print(" | ");
            System.out.print(format(part.getDesc(), PartsDB.DESCLEN, true));
            System.out.print(" | ");
            System.out.print(format("" + part.getQty(), 10, false));
            System.out.print(" | ");
            String s = part.getUnitCost() / 100 + "." + part.getUnitCost() %
                    100;
            if (s.charAt(s.length() - 2) == '.') s += "0";
            System.out.println(format(s, 10, false));
        }
        System.out.println("Number of records = " + pdb.numRecs());
        System.out.println();
    }

    static String format(String value, int maxWidth, boolean leftAlign) {
        StringBuffer sb = new StringBuffer();
        int len = value.length();
        if (len > maxWidth) {
            len = maxWidth;
            value = value.substring(0, len);
        }
        if (leftAlign) {
            sb.append(value);
            for (int i = 0; i < maxWidth - len; i++)
                sb.append(" ");
        } else {
            for (int i = 0; i < maxWidth - len; i++)
                sb.append(" ");
            sb.append(value);
        }
        return sb.toString();
    }
}
```

`main()`方法先实例化`PartsDB`，使用` parts.db `作为database file的name。当没记录,` numRecs()  `返回0。然后通过`append()  `添加了几个record。

`main() `方法然后dump这5个记录到标准输出流，更新 number是1的record的unit cost，再次将这些记录dump到标准输出流中以显示，然后关闭。

**注意**：使用基于integer的美分数量存储cost值。如果要用`java.math.BigDecimal `对象来保存货币值，必须重构`PartsDB`以利用对象序列化的优势，还没做这个（discuss object serialization in Chapter 4 ）。

`main()`依赖`dumpRecords() ` helper方法来dump records，`dumpRecords()`依赖`format()`helper方法format field的值，因此能在正确对齐的列中显示。我可以用`java.util.Formatter `代替 (see Chapter 11 )。

编译运行，输出：

```
1-9009-3323-4x | Wiper Blade Micro Edge | 30 | 24.68
1-3233-44923-7j | Parking Brake Cable | 5 | 19.95
2-3399-6693-2m | Halogen Bulb H4 55/60W | 22 | 8.13
2-599-2029-6k | Turbo Oil Line O-Ring | 26 | 1.55
3-1299-3299-9u | Air Pump Electric | 9 | 202.00
Number of records = 5
1-9009-3323-4x | Wiper Blade Micro Edge | 30 | 24.68
1-3233-44923-7j | Parking Brake Cable | 5 | 19.95
2-3399-6693-2m | Halogen Bulb H4 55/60W | 22 | 8.13
2-599-2029-6k | Turbo Oil Line O-Ring | 26 | 1.55
3-1299-3299-9u | Air Pump Electric | 9 | 202.00
Number of records = 5
```

**Note** ：Check out Wikipedia’s “Flat file database” entry (https://en.wikipedia.org/wiki/Flat_file_database) to learn more about flat file databases. 

#### exercise

```
The following exercises are designed to test your understanding of Chapter 3’s content:
1. What is the purpose of the RandomAccessFile class?
2. What is a file’s metadata?
3. What is the purpose of the "rwd" and "rws" mode arguments?
4. What is a file pointer?
5. What happens when you write past the end of the file?
6. True or false: When you call RandomAccessFile’s seek(long)
method to set the file pointer’s value, and when this value is greater
than the length of the file, the file’s length changes.
7. What does method void write(int b) accomplish?
8. What does FileDescriptor’s sync() method accomplish?
9. Define flat file database.
10. Write a small Java application named RAFDemo that opens file data in
read/write mode, uses void write(int b) to write byte value 127
followed by void writeChars(String s) to write string "Test"
(minus the quotes) to this file, resets the file pointer to the start of the
file, and read/outputs these values.
```

#### Summary 

### Chapter 4 Streams 

连同`java.io.File and java.io.RandomAccessFile`，Java’s classic I/O也提供了 streams用于文件操作。stream是一个有顺序的任意长度的字节的序列。 字节从应用到目的地流过输出流，从源到应用流过输入流。

#### Stream Classes Overview 

output stream类结构层次图：

![1532685952107](https://github.com/konekos/notes/blob/master/src/pic/1532685952107.png?raw=true)

input stream类结构层次图：

![1532686025994](https://github.com/konekos/notes/blob/master/src/pic/1532686025994.png?raw=true)

`LineNumberInputStream and StringBufferInputStream `已被弃用，因为不支持不同的字符编码， 在Chapter 5 讨论代替它们的`java.io.LineNumberReader and java.io.StringReader `。

**注意**：`PrintStream`是另一个因为不支持不同编码而应该被弃用的类；`java.io.PrintWriter `是替代。然而，Oracle弃用这个类是存疑的，因为`PrintStream `是`java.lang.System class’s out and err class fields `类型，而且太多的遗留代码依赖于它。

其他Java包提供另外的 output stream and input stream类。例如，` java.util.zip `提供4个输出流类将未压缩的数据压缩成各种格式，以及4个对应的输入流从相同的数据格式中解压压缩数据。

- CheckedOutputStream 
- CheckedInputStream 
- DeflaterOutputStream 
- GZIPOutputStream 
- GZIPInputStream 
- InflaterInputStream 
- ZipOutputStream 
- ZipInputStream 

同时，`java.util.jar `提供一对流类用于写内容到JAR文件和读取JAR文件的内容：

- JarOutputStream 
- JarInputStream 

#### Touring the Stream Classes 

下面开始 java.io’s output stream and input stream classes 的tour，从OutputStream and InputStream 开始。

##### OutputStream and InputStream 

Java提供抽象的` OutputStream and InputStream `类描述I/O流。`OutputStream `是所有output stream的超类。

***Table 4-1. OutputStream Methods*** 

| Method                                 | Description                                                  |
| -------------------------------------- | ------------------------------------------------------------ |
| void close()                           | 关流和释放任何和流有关的操作系统资源。I/O错误发生抛出` java.io.IOException `。 |
| void flush()                           | 把任何缓冲的输出字节写到目的地来刷新输出流。如果输出流的目的地是底层操作系统提供的一个抽象（例如file），刷新这个流只保证先前写入流的字节被传递给底层操作系统用于write；不保证确实被写入物理设备比如一个磁盘驱动器。I/O错误发生抛出` java.io.IOException `。 |
| void write(byte[] b)                   | 从字节数组b写b个length字节到输入流。通常  write(b) 表现得像你指定了 write(b, 0, b.length)。b是null抛出`  java.lang .NullPointerException `，I/O错误发生抛出` java.io.IOException ` |
| void write(byte[] b, int off, int len) | 从byte数组b的off offset开始，写len个字节到输入流。b是null抛出`  java.lang .NullPointerException `，` IndexOutOfBoundsException  `当off是负，len是负，或者off + bean>b.length；I/O错误发生抛出` java.io.IOException ` |
| void write(int b)                      | 写字节b到输入流。只有8个 ow-order bits被写；24个 high-order bits 被忽略。I/O错误发生抛出` java.io.IOException ` |

 flush()  在长期运行的应用需要经常保持更改的时候很有用。记住flush() 只刷新字节到操作系统；不一定导致操作系统刷新字节到磁盘。

**注意**： close() 自动刷新输出流，如果应用在close被调用前结束，输入流自动关系，数据被刷新。

`InputStream `是所有输入流的超类。

***Table 4-2. InputStream Methods*** 

| Method                               | Description                                                  |
| ------------------------------------ | ------------------------------------------------------------ |
| int available()                      | 返回从输入流读取的字节数量的估计，通过  read() 方法（或 skipped over via skip() ）调用而不阻塞调用线程。I/O错误发生抛出` java.io.IOException `。用它的返回值来分配一个缓冲区来存放所有流的数据是不正确的，因为子类的实现可能不会返回流的总大小。 |
| void close()                         | 关闭输入流并释放任何和流有关的操作系统资源。I/O错误发生抛出` java.io.IOException `。 |
| void mark(int readlimit)             | 在输入流中标记当前位置。随后调用  reset()  ，定位流到最后一次标记的位置，然后随后的读操作读取相同的字节。 `readlimit `参数告诉输入流在失效标记前，允许读取多少字节（因此这个流不能被重置到标记的位置 ）。 |
| boolean markSupported()              | 当这个输入流支持  mark() and reset() 返回true；否则，返回false。 |
| int read()                           | 读取返回（作为0-255的一个int值）输入流的下一个字节，到达流终点返回-1；这个方法会阻塞，直到input可用，流end被检测，或抛出异常。I/O错误发生抛出` java.io.IOException `。 |
| int read(byte[] b)                   | 从这个输入流读一定数目的字节，把他们存到字节数组b。返回实际读取的字节数（可能比b的length小但是永远不会超过），或者返回-1当到达流end（无字节可读）。这个方法会阻塞，直到input可用，流end被检测，或抛出异常。b是null抛出空指针，I/O错误发生抛出` java.io.IOException `。 |
| int read(byte[] b, int off, int len) | 从输入流读不超过len个字节，存在array b，从off开始。返回实际读取的字节数（可能比b的length小但是永远不会超过），或者返回-1当到达流end（无字节可读）。这个方法会阻塞，直到input可用，流end被检测，或抛出异常。。b是null抛出空指针；off是负，len是负，len比b.length-off大抛出` IndexOutOfBoundsException `I/O错误发生抛出` java.io.IOException `。 |
| void reset()                         | 重定位输入流到最后一次 mark()的位置。输入流没被标记或标记失效抛出`IOException`  。 |
| long skip(long n)                    | 跳过并丢弃n字节的数据。这个方法可能跳过的数量会小一点（可0），例如，file end。返回实际的跳过数。n是负，不跳过。输入流不支持skip或者I/O错误抛出` IOException ` |

子类例如`ByteArrayInputStream`支持mark()标记当前位置，然后reset返回。

**注意**：不要忘了调用`markSupported() `来看是不是支持mark() and reset()。

##### ByteArrayOutputStream and ByteArrayInputStream 

字节数组在作为流目的地和源时很有用。`ByteArrayOutputStream `类让你写一个字节流到字节数组；`ByteArrayInputStream `让你从字节数组读取字节流。

`ByteArrayOutputStream `声明2个构造器。每个构造器使用内部的字节数组创建 byte array output stream；这个array的copy可以通过调用` byte[] toByteArray() `返回：

- ByteArrayOutputStream() 使用内部的字节数组初始容量是32字节 创建一个byte array output stream。这个数组必要时增长。
- ByteArrayOutputStream(int size) 使用指定的长度创建byte array output stream。 <0抛出`java.lang.IllegalArgumentException `。

下面的例子用`ByteArrayOutputStream()  `创建一个byte array output stream，使用默认size。

```java
ByteArrayOutputStream baos = new ByteArrayOutputStream();
```

`ByteArrayInputStream `也有两个构造器。每个构造器基于指定字节数组创建a byte array input stream 

- ByteArrayInputStream(byte[] ba) 使用ba数组创建（直接使用；不创建copy）。position为0，要读的字节数为ba.length。
- ByteArrayInputStream(byte[] ba, int offset, int count)  position是offset，读取count个字节。

例子，

```
ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
```

这两个流在你需要转换图片为字节数组，用某种方法处理字节，然后转换字节成图片。

一个例子：

```java
String path = ... ; // Assume a legitimate path to an image.
Bitmap bm = BitmapFactory.decodeFile(path);
ByteArrayOutputStream baos = new ByteArrayOutputStream();
if (bm.compress(Bitmap.CompressFormat.PNG, 100, baos))
{
 byte[] imageBytes = baos.toByteArray();
 // Do something with imageBytes.
 bm = BitMapFactory.decodeStream(new ByteArrayInputStream(imageBytes));
}
```

##### FileOutputStream and FileInputStream 

文件是常见的流目的地和源。 

下面用`FileOutputStream(String path)`创建文件输出流。

```java
FileOutputStream fos = new FileOutputStream("employee.dat");
```

**Tip**：FileOutputStream(String name) 重新存在的文件。要添加数据而不是覆盖存在的内容，调用带有 boolean append 的构造器，设置参数为true。

下面创建一个 FileInputStream(String name) 的文件输入流，employee.dat为source。

```java
FileInputStream fis = new FileInputStream("employee.dat");
```

FileOutputStream and FileInputStream 在文件复制有用。

***Listing 4-1. Copying a Source File to a Destination File***

```java
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

public class Copy {
    public static void main(String[] args) {
        if (args.length != 2) {
            System.err.println("usage: java Copy srcfile dstfile");
            return;
        }
        FileInputStream fis = null;
        FileOutputStream fos = null;
        try {
            fis = new FileInputStream(args[0]);
            fos = new FileOutputStream(args[1]);
            int b; // I chose b instead of byte because byte is a reserved
            // word.
            while ((b = fis.read()) != -1)
                fos.write(b);
        } catch (FileNotFoundException fnfe) {
            System.err.println(args[0] + " could not be opened for input, or "
                    + args[1] + " could not be created for output");
        } catch (IOException ioe) {
            System.err.println("I/O error: " + ioe.getMessage());
        } finally {
            if (fis != null)
                try {
                    fis.close();
                } catch (IOException ioe) {
                    assert false; // shouldn't happen in this context
                }
            if (fos != null)
                try {
                    fos.close();
                } catch (IOException ioe) {
                    assert false; // shouldn't happen in this context
                }
        }
    }
}
```

使用try-resource

```java
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

public class Copy1 {
    public static void main(String[] args) {
        if (args.length != 2) {
            System.err.println("usage: java Copy srcfile dstfile");
            return;
        }
        try (FileInputStream fis = new FileInputStream(args[0]);
             FileOutputStream fos = new FileOutputStream(args[1])) {
            int b; // I chose b instead of byte because byte is a reserved
            // word.
            while ((b = fis.read()) != -1)
                fos.write(b);
        } catch (FileNotFoundException fnfe) {
            System.err.println(args[0] + " could not be opened for input, or "
                    + args[1] + " could not be created for output");
        } catch (IOException ioe) {
            System.err.println("I/O error: " + ioe.getMessage());
        }
    }
}
```

##### PipedOutputStream and PipedInputStream 

线程必须经常交流。一种方法是引入共享变量。另一种是通过`PipedOutputStream and PipedInputStream `类使用piped streams（管道流）。`PipedOutputStream `类让一个sending线程写一个字节流到一个`PipedInputStream `类的实例，然后一个receiving线程读取这些字节。

**警告**:尝试使用来自一个single thread的 a`PipedOutputStream` object and a `PipedInputStream` object 是不推荐的，可能会死锁。

`PipedOutputStream `声明2个构造器：

- PipedOutputStream() 创建一个 piped output stream ，还没有连接到 a piped input stream 。在使用前，它必须被连接到一个 piped input stream ，无论使用receiver or the sender 。
- PipedOutputStream(PipedInputStream dest) 创建piped output stream 连接到piped input stream dest 。写到piped output stream的字节可以从dest读出来。I/O错误抛出`IOException`。

 connect(PipedInputStream dest)方法，用于连接。

`PipedInputStream `2个构造器：

- PipedInputStream()  用前必须连接。
- PipedInputStream(int pipeSize) 使用pipeSize指定管道大小。用前必须连接。
- PipedInputStream(PipedOutputStream src) 
- PipedInputStream(PipedOutputStream src, int pipeSize)  

 connect(PipedOutputStream src) 方法用于连接。

创建一对管道流的最简单的方法是在同一个线程中任意顺序。例如：

```java
PipedOutputStream pos = new PipedOutputStream();
PipedInputStream pis = new PipedInputStream(pos);
```

或者：

```java
PipedInputStream pis = new PipedInputStream();
PipedOutputStream pos = new PipedOutputStream(pis);
```

也可以:

```java
PipedOutputStream pos = new PipedOutputStream();
PipedInputStream pis = new PipedInputStream();
// ...
pos.connect(pis);
```

实例：

***Listing 4-3. Piping Randomly Generated Bytes from a Sender Thread to a Receiver Thread*** 

```java
import java.io.IOException;
import java.io.PipedInputStream;
import java.io.PipedOutputStream;

public class PipedStreamsDemo {
    final static int LIMIT = 10;

    public static void main(String[] args) throws IOException {
        final PipedOutputStream pos = new PipedOutputStream();
        final PipedInputStream pis = new PipedInputStream(pos);
        Runnable senderTask = () -> {
            try {
                for (int i = 0; i < LIMIT; i++)
                    pos.write((byte)
                            (Math.random() * 256));
            } catch (IOException ioe) {
                ioe.printStackTrace();
            } finally {
                try {
                    pos.close();
                } catch (IOException ioe) {
                    ioe.printStackTrace();
                }
            }
        };
        Runnable receiverTask = () -> {
            try {
                int b;
                while ((b = pis.read()) != -1)
                    System.out.println(b);
            } catch (IOException ioe) {
                ioe.printStackTrace();
            } finally {
                try {
                    pis.close();
                } catch (IOException ioe) {
                    ioe.printStackTrace();
                }
            }
        };
        Thread sender = new Thread(senderTask);
        Thread receiver = new Thread(receiverTask);
        sender.start();
        receiver.start();
    }
}
```

##### FilterOutputStream and FilterInputStream 

Byte array, file, and piped streams传递没有变动过的字节给目标。也支持 filter streams来buffer, compress/ uncompress, encrypt/decrypt, or otherwise manipulate a stream’s byte sequence (that is input to the filter) 。

一个filter output stream拿到传递给 write()方法的数据（(the input stream ），过滤，写过滤过的数据到下面的output stream，可能是 another filter output stream or a destination output stream such as a file output stream。

Filter output streams用`OutputStream `子类`FilterOutputStream `创建，单个构造器`FilterOutputStream(OutputStream out) `。示例：

***Listing 4-4. Scrambling a Stream of Bytes*** 

```java
import java.io.FilterOutputStream;
import java.io.IOException;
import java.io.OutputStream;

public class ScrambledOutputStream extends FilterOutputStream {
    private int[] map;

    public ScrambledOutputStream(OutputStream out, int[] map) {
        super(out);
        if (map == null)
            throw new NullPointerException("map is null");
        if (map.length != 256)
            throw new IllegalArgumentException("map.length != 256");
        this.map = map;
    }

    @Override
    public void write(int b) throws IOException {
        out.write(map[b]);
    }
}
```

4-5打乱文件字节：

***Listing 4-5. Scrambling a File’s Bytes*** 

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Random;

public class Scramble {
    public static void main(String[] args) {
        if (args.length != 2) {
            System.err.println("usage: java Scramble srcpath destpath");
            return;
        }
        FileInputStream fis = null;
        ScrambledOutputStream sos = null;
        try {
            fis = new FileInputStream(args[0]);
            FileOutputStream fos = new FileOutputStream(args[1]);
            sos = new ScrambledOutputStream(fos, makeMap());
            int b;
            while ((b = fis.read()) != -1)
                sos.write(b);
        } catch (IOException ioe) {
            ioe.printStackTrace();
        } finally {
            if (fis != null)
                try {
                    fis.close();
                } catch (IOException ioe) {
                    ioe.printStackTrace();
                }
            if (sos != null)
                try {
                    sos.close();
                } catch (IOException ioe) {
                    ioe.printStackTrace();
                }
        }
    }

    static int[] makeMap() {
        int[] map = new int[256];
        for (int i = 0; i < map.length; i++)
            map[i] = i;
        // Shuffle map.
        Random r = new Random(0);
        for (int i = 0; i < map.length; i++) {
            int n = r.nextInt(map.length);
            int temp = map[i];
            map[i] = map[n];
            map[n] = temp;
        }
        return map;
    }
}
```

Filter input streams，示例：

***Listing 4-6. Unscrambling a Stream of Bytes*** 

使用ScrambledInputStream 解密：

***Listing 4-7. Unscrambling a File’s Bytes*** 

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Random;

public class Unscramble {
    public static void main(String[] args) {
        if (args.length != 2) {
            System.err.println("usage: java Unscramble srcpath destpath");
            return;
        }
        ScrambledInputStream sis = null;
        FileOutputStream fos = null;
        try {
            FileInputStream fis = new FileInputStream(args[0]);
            sis = new ScrambledInputStream(fis, makeMap());
            fos = new FileOutputStream(args[1]);
            int b;
            while ((b = sis.read()) != -1)
                fos.write(b);
        } catch (IOException ioe) {
            ioe.printStackTrace();
        } finally {
            if (sis != null)
                try {
                    sis.close();
                } catch (IOException ioe) {
                    ioe.printStackTrace();
                }
            if (fos != null)
                try {
                    fos.close();
                } catch (IOException ioe) {
                    ioe.printStackTrace();
                }
        }
    }

    static int[] makeMap() {
        int[] map = new int[256];
        for (int i = 0; i < map.length; i++)
            map[i] = i;
        // Shuffle map.
        Random r = new Random(0);
        for (int i = 0; i < map.length; i++) {
            int n = r.nextInt(map.length);
            int temp = map[i];
            map[i] = map[n];
            map[n] = temp;
        }
        int[] temp = new int[256];
        for (int i = 0; i < temp.length; i++)
            temp[map[i]] = i;
        return temp;
    }
}
```

##### BufferedOutputStream and BufferedInputStream 

FileOutputStream和FileInputStream有一个性能问题。 每个 file output stream write() 方法调用和 file input stream read() 方法调用导致 a native method调用操作系统程式，这些native methods减缓了I/O。

 `BufferedOutputStream` and `BufferedInputStream` filter stream类提高了性能，通过最小化底层的output stream write() 调用和 input stream read() 方法调用。使用`BufferedOutputStream`’s write() and `BufferedInputStream`’s read() 代替，将Java缓冲区考虑进去：

- 当write buffer满了，write() 调用底层的output stream write()方法来排空buffer。随后调用`BufferedOutputStream`的write()方法将字节保存在这个缓冲区中，直到它再次满为止。
- 当 read buffer是空的， read() 调用底层的 input stream read() 方法来充满buffer。随后调用 `BufferedInputStream`’s read() 从该buffer返回字节直到它再次为空。

`BufferedOutputStream`以下构造器

- BufferedOutputStream(OutputStream out) 创建一个buffered output stream把输出流到out。一个内部的buffer被创建用来保存写出的字节。
- BufferedOutputStream(OutputStream out, int size) 指定了长度。

```java
FileOutputStream fos = new FileOutputStream("employee.dat");
BufferedOutputStream bos = new BufferedOutputStream(fos); // Chain bos
// to fos.
bos.write(0); // Write to employee.dat through the buffer.
// Additional write() method calls.
bos.close(); // This method call internally calls fos's close() method.
```

`BufferedInputStream `有两个构造器：

- BufferedInputStream(InputStream in)  
- BufferedInputStream(InputStream in, int size) 

```java
FileInputStream fis = new FileInputStream("employee.dat");
BufferedInputStream bis = new BufferedInputStream(fis); // Chain bis to fis.
int ch = bis.read(); // Read employee.dat through the buffer.
// Additional read() method calls.
bis.close(); // This method call internally calls fis's close() method.
```

##### DataOutputStream and DataInputStream 

`FileOutputStream and FileInputStream `对读取字节和字节数组很有用。然而，不支持读写primitive-type的值（比如integer）和strings。

于是Java提供了 DataOutputStream and DataInputStream filter stream 类。克服了这个限制，提供方法读写primitive-type和strings，用依赖于操作系统的方式：

- Integer values是用 big-endian 形式读写。
- Floating-point and double precision floating-point  根据 IEEE 754 标准读写，每个floatingpoint value 4 字节，每个double precision floating-point value8字节。
- Strings根据修改的UTF-8 版本读写，一个可变长度编码标准有效保存2字节 Unicode characters 。

`DataOutputStream `有一个构造器，DataOutputStream(OutputStream out) 。因为实现了`java.io.DataOutput `，`DataOutputStream `也提供了到` java.io.RandomAccessFile `提供的相同命名的写方法的访问。

`DataInputStream `一个构造器，DataInputStream(InputStream in) 。因为实现了` java.io.DataInput `，`DataInputStream `也提供` java.io.RandomAccessFile `的相同命名的读方法的访问。

例子：

***Listing 4-8. Outputting and then Inputting a Stream of Multibyte Values*** 

```java
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

public class DataStreamsDemo {
    final static String FILENAME = "values.dat";

    public static void main(String[] args) {
        try (FileOutputStream fos = new FileOutputStream(FILENAME);
             DataOutputStream dos = new DataOutputStream(fos)) {
            dos.writeInt(1995);
            dos.writeUTF("Saving this String in modified UTF-8 format!");
            dos.writeFloat(1.0F);
        } catch (IOException ioe) {
            System.err.println("I/O error: " + ioe.getMessage());
        }
        try (FileInputStream fis = new FileInputStream(FILENAME);
             DataInputStream dis = new DataInputStream(fis)) {
            System.out.println(dis.readInt());
            System.out.println(dis.readUTF());
            System.out.println(dis.readFloat());
        } catch (IOException ioe) {
            System.err.println("I/O error: " + ioe.getMessage());
        }
    }
}
```

**警告**：当读一个序列`DataOutputStream`的方法写入的值，确保使用相同的方法调用序列。否则得到错误的数据。

#### Object Serialization and Deserialization 

Java提供`DataOutputStream and DataInputStream classes `来流化 primitive-type values and String objects。但你不能stream非string对象。你必须用`object serialization and deserialization `来stream任意类型对象。

对象*serialization*是一个JVM机制，序列化对象状态变成一个字节流。对应的 *deserialization* 是相反的。

**注意**：对象state由实例fields组成，保存了原始类型值和其他对象的引用。当一个对象序列化，是这个state一部分的对象也会实例化（除非你阻止）。另外，这些对象的部分是对象的话也一样实例化。

Java支持默认 serialization and deserialization，自定义 serialization and deserialization 和externalization 。

##### Default Serialization and Deserialization 

默认的是最简单的方式但是提供很少的control。虽然Java代你做了大部分事情，但是你也要2个任务必须做。

第一个任务要被序列化的类要实现`java.io.Serializable `接口，实现的原因是避免无限序列化。

**注意**：`Serializable `是一个空的标记接口（没有方法来实现），告诉JVM序列化是okay的。当序列化机制遭遇没有实现这个接口的对象，抛出`java.io.NotSerializableException `（一个`IOException`的非直接子类）。

*Unlimited serialization*是序列化整个object graph的过程。不支持它的原因：

- Security: 
- Performance: 序列化利用了反射API，会降低应用程序的性能。无限序列化可能会损害应用程序的性能。
- Objects not amenable to serialization: 

下面一个例子：

***Listing 4-9. Implementing Serializable*** 

```java
import java.io.Serializable;
public class Employee implements Serializable
{
 private String name;
 private int age;
 public Employee(String name, int age)
 {
 this.name = name;
 this.age = age;
 }
 public String getName() { return name; }
 public int getAge() { return age; }
}

```

`java.lang.String`也实现了Serializable。

第二个任务是用`ObjectOutputStream `类，和它的`writeObject() `方法来序列化，`ObjectInputStream `的`readObject() `方法反序列化。

**注意**：虽然没继承 FilterOutputStream 和 FilterInputStream ，上面那2个序列化的类也表现如同filter streams 。

 `ObjectOutputStream`序列化对象state，构造器` ObjectOutputStream(OutputStream out)， `当传递output stream reference到out，构造器尝试写一个 serialization header 到输出流。out是null抛异常阻止写入header。

`writeObject(Object obj) `方法尝试写关于对象类的信息，紧接着写对象实例field的值。不会序列化static field，和 transient 标注的field（忽略）。

**注意**：`ObjectOutputStream` implements `DataOutput `，提供了写primitive-type values and strings到对象输出流的方法。

`ObjectInputStream  `类反序列化。`ObjectInputStream(InputStream in) `构造器。当传递 input stream reference 到in，尝试从输入流读取serialization header ，抛出空指针 IO异常，如果header不对就抛出`java.io.StreamCorruptedException `。

` readObject() `方法反序列化对象。

**注意**：类似上一个。

例子：

***Listing 4-10. Serializing and Deserializing an Employee Object***

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;

public class SerializationDemo {
    final static String FILENAME = "employee.dat";

    public static void main(String[] args) {
        ObjectOutputStream oos = null;
        ObjectInputStream ois = null;
        try {
            FileOutputStream fos = new FileOutputStream(FILENAME);
            oos = new ObjectOutputStream(fos);
            Employee emp = new Employee("John Doe", 36);
            oos.writeObject(emp);
            oos.close();
            oos = null;
            FileInputStream fis = new FileInputStream(FILENAME);
            ois = new ObjectInputStream(fis);
            emp = (Employee) ois.readObject(); // (Employee) cast is necessary.
            ois.close();
            System.out.println(emp.getName());
            System.out.println(emp.getAge());
        } catch (ClassNotFoundException cnfe) {
            System.err.println(cnfe.getMessage());
        } catch (IOException ioe) {
            System.err.println(ioe.getMessage());
        } finally {
            if (oos != null)
                try {
                    oos.close();
                } catch (IOException ioe) {
                    assert false; // shouldn't happen in this context
                }
            if (ois != null)
                try {
                    ois.close();
                } catch (IOException ioe) {
                    assert false; // shouldn't happen in this context
                }
        }
    }
}

```

 反序列化后不能不保证是同一个类（可能一个实例field被删除了）。当类不同抛出`java.io.InvalidClassException `。

每个序列化的对象有一个标识符。标识符不匹配抛出`InvalidClassException `。

也许您已经在类中添加了一个实例字段，你想让这个field有默认值而不是抛出` InvalidClassException `，通过添加static final long serialVersionUID = long integer value到class。long integer value 必须是唯一的作为一个stream unique identifier (SUID)。

反序列化时，会比较SUID，如果匹配，当遭遇兼容的class  change不会抛出`InvalidClassException `（比如添加实例field）。但是不兼容的class变更还是抛出异常（比如修改field name）。

**注意**：当你以某种方式改变一个类的时候，必须重新计算SUID。

JDK提供serialver tool 来生成 SUID。例如：

```java
serialver Employee
```

生成：

```java
Employee: static final long serialVersionUID = 1517331364702470316L;
```

windows可视化界面：

```java
serialver -show
```

##### Custom Serialization and Deserialization 

如果你要序列化没实现`Serializable `的类，一种解决方案是，继承这个类，让子类实现` Serializable `，子类构造器呼叫父类。

当父类没声明无参构造是不行的。例子：

***Listing 4-11. Problematic Deserialization*** 

```java
public class Employee1 {
    private String name;

    Employee1(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

```java
public class SerEmployee extends Employee1 implements Serializable {
    SerEmployee(String name)
    {
        super(name);
    }
}
```

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;


public class SerializationDemo {

    private final static String PATH = "E:\\SpringSourceCode\\src\\main\\resources\\image.txt";

    public static void main(String[] args) {
        ObjectOutputStream oos = null;
        ObjectInputStream ois = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream(PATH));
            SerEmployee se = new SerEmployee("John Doe");
            System.out.println(se);
            oos.writeObject(se);
            oos.close();
            oos = null;
            System.out.println("se object written to file");
            ois = new ObjectInputStream(new FileInputStream(PATH));
            se = (SerEmployee) ois.readObject();
            System.out.println("se object read from file");
            System.out.println(se);
        } catch (ClassNotFoundException cnfe) {
            cnfe.printStackTrace();
        } catch (IOException ioe) {
            ioe.printStackTrace();
        } finally {
            if (oos != null)
                try {
                    oos.close();
                } catch (IOException ioe) {
                    assert false; // shouldn't happen in this context
                }
            if (ois != null)
                try {
                    ois.close();
                } catch (IOException ioe) {
                    assert false; // shouldn't happen in this context
                }
        }
    }

}
```

输出

```java
John Doe
java.io.InvalidClassException: com.jasu.nio._04_Streams.SerEmployee; no valid constructor
	at java.io.ObjectStreamClass$ExceptionInfo.newInvalidClassException(ObjectStreamClass.java:157)
	at java.io.ObjectStreamClass.checkDeserialize(ObjectStreamClass.java:862)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2034)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1567)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:427)
	at com.jasu.nio._04_Streams.SerializationDemo.main(SerializationDemo.java:31)
```

因为Employee1 没有声明无参构造器。你可以用adapter pattern解决这个问题（(https://en.wikipedia.org/wiki/Adapter_pattern ）。

此外的例子：

***Listing 4-12. Solving Problematic Deserialization*** 

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;
class Employee
{
 private String name;
 Employee(String name)
 {
 this.name = name;
 }
 @Override
 public String toString()
 {
 return name;
 }
}
class SerEmployee implements Serializable
{
 private Employee emp;
 private String name;
 SerEmployee(String name)
 {
 this.name = name;
 emp = new Employee(name);
 }
 private void writeObject(ObjectOutputStream oos) throws IOException
 {
 oos.writeUTF(name);
 }
 private void readObject(ObjectInputStream ois)
 throws ClassNotFoundException, IOException
 {
 name = ois.readUTF();
 emp = new Employee(name);
 }
 @Override
 public String toString()
 {
 return name;
 }
}public class SerializationDemo
{
 public static void main(String[] args)
 {
 ObjectOutputStream oos = null;
 ObjectInputStream ois = null;
 try
 {
 oos = new ObjectOutputStream(new FileOutputStream("employee.dat"));
 SerEmployee se = new SerEmployee("John Doe");
 System.out.println(se);
 oos.writeObject(se);
 oos.close();
 oos = null;
 System.out.println("se object written to file");
 ois = new ObjectInputStream(new FileInputStream("employee.dat"));
 se = (SerEmployee) ois.readObject();
 System.out.println("se object read from file");
 System.out.println(se);
 }
 catch (ClassNotFoundException cnfe)
 {
 cnfe.printStackTrace();
 }
 catch (IOException ioe)
 {
 ioe.printStackTrace();
 }
 finally
 {
 if (oos != null)
 try
 {
 oos.close();
 }
 catch (IOException ioe)
 {
 assert false; // shouldn't happen in this context
 }
 if (ois != null)
 try
 {
 ois.close();
 }public class SerializationDemo
{
 public static void main(String[] args)
 {
 ObjectOutputStream oos = null;
 ObjectInputStream ois = null;
 try
 {
 oos = new ObjectOutputStream(new FileOutputStream("employee.dat"));
 SerEmployee se = new SerEmployee("John Doe");
 System.out.println(se);
 oos.writeObject(se);
 oos.close();
 oos = null;
 System.out.println("se object written to file");
 ois = new ObjectInputStream(new FileInputStream("employee.dat"));
 se = (SerEmployee) ois.readObject();
 System.out.println("se object read from file");
 System.out.println(se);
 }
 catch (ClassNotFoundException cnfe)
 {
 cnfe.printStackTrace();
 }
 catch (IOException ioe)
 {
 ioe.printStackTrace();
 }
 finally
 {
 if (oos != null)
 try
 {
 oos.close();
 }
 catch (IOException ioe)
 {
 assert false; // shouldn't happen in this context
 }
 if (ois != null)
 try
 {
 ois.close();
 }catch (IOException ioe)
 {
 assert false; // shouldn't happen in this context
 }
 }
 }
}

```

输出：

```
John Doe
se object written to file
se object read from file
John Doe
```

##### Externalization 

Java支持 externalization 。不像默认/自定义serialization/deserialization , externalization提供序列化反序列化的完整控制。

**注意**：Externalization 帮你改善基于反射的序列化反序列化性能，给你完全控制那些field参与序列化。

通过`java.io.Externalizable`支持Externalization。接口有以下方法：

- void writeExternal(ObjectOutput out)  
- void readExternal(ObjectInput in)  

如果类实现` Externalizable `，例子：

***Listing 4-13. Refactoring Listing 4-9’s Employee Class to Support Externalization*** 

```java
import java.io.Externalizable;
import java.io.IOException;
import java.io.ObjectInput;
import java.io.ObjectOutput;
public class Employee implements Externalizable
{
 private String name;
 private int age;
 public Employee()
 {
 System.out.println("Employee() called");
 } public Employee(String name, int age)
 {
 this.name = name;
 this.age = age;
 }
 public String getName() { return name; }
 public int getAge() { return age; }
 @Override
 public void writeExternal(ObjectOutput out) throws IOException
 {
 System.out.println("writeExternal() called");
 out.writeUTF(name);
 out.writeInt(age);
 }
 @Override
 public void readExternal(ObjectInput in)
 throws IOException, ClassNotFoundException
 {
 System.out.println("readExternal() called");
 name = in.readUTF();
 age = in.readInt();
 }
}


```

输出：

```java
writeExternal() called
Employee() called
readExternal() called
John Doe
36
```

#### PrintStream 

在全部的stream类里，`PrintStream `是古怪的：按命名约定应该叫` PrintOutputStream `。这个filter output stream 类写输入数据项的字符串形式到底层输出流。

**注意**：`PrintStream `会使用默认字符编码转化string为字节（5章讨论）。`PrintStream `不支持不同的编码，你该用等价的` PrintWriter `代替。然而因为standard I/O，你需要了解`PrintStream `。

`PrintStream `实例是print streams ，它的方法print() and println() ，打印integers, floating-point values, and other data items 的字符串形式到下面的输出流。

**注意**： line terminator （也叫 line separator ），不一定newline（也通常被称为换行 ），line separator 是系统属性`line.separator. `windows是回车码（13）也就是` \r `，紧接着实际的newline/line feed code(10)，`\n`，linux只返回newline/line feed code 。

**警告**：不要在你用 print() or println() 方法输出的字符串上硬编码` \n escape sequence `。这样是不稳定的。

有3个有用的特点：

- 不像其他stream，不会从底层抛出`IOException`。而是有checkError() 方法
- 可以被创建来自动刷新输出到底层输出流。字节写入自动flush()  ，println()被调用，或者一个newline被写。
- 声明了一个`format(String format, Object... args)` 形式用于得到格式化输出。是用的Formatter类，11章介绍。也有便利的`printf(String format, Object... args) `方法。

#### Revisiting Standard I/O 

System.in, System.out, and System.err are formally described by the following class fields in the System class: 

- public static final InputStream in 
- public static final PrintStream out 
- public static final PrintStream err

对应standard input, standard output, and standard error streams 。

当你调用 System.in.read(), 来自于绑定到in的source。当标准输入时，Java初始化以引用键盘或文件流被重定向到文件。` out/err `设计屏幕或文件。你可以编程式指定input source, output destination, and error destination 通过`System`的方法：

- void setIn(InputStream in) 
- void setOut(PrintStream out) 
- void setErr(PrintStream err) 

RedirectIO application例子：

***Listing 4-14. Programmatically Specifying the Standard Input Source and Standard Output/Error Destinations*** 

```
import java.io.FileInputStream;
import java.io.IOException;
import java.io.PrintStream;
public class RedirectIO
{
 public static void main(String[] args) throws IOException
 {
 if (args.length != 3)
 {
 System.err.println("usage: java RedirectIO stdinfile " +
 "stdoutfile stderrfile");
 return;
 }
 System.setIn(new FileInputStream(args[0]));
 System.setOut(new PrintStream(args[1]));
 System.setErr(new PrintStream(args[2]));
 int ch;
 while ((ch = System.in.read()) != -1)
 System.out.print((char) ch);
 System.err.println("Redirected error output");
 }
}

```

run `java RedirectIO RedirectIO.java out.txt err.txt `。

#### Exercise

```
The following exercises are designed to test your understanding of Chapter 4’s content:
1. What is a stream?
2. What is the purpose of OutputStream’s flush() method?
3. True or false: OutputStream’s close() method automatically
flushes the output stream.
4. What is the purpose of InputStream’s mark(int) and reset()
methods?
5. How would you access a copy of a ByteArrayOutputStream
instance’s internal byte array?
6. True or false: FileOutputStream and FileInputStream provide
internal buffers to improve the performance of write and read
operations.
7. Why would you use PipedOutputStream and PipedInputStream?
8. Define filter stream.
9. What does it mean for two streams to be chained together?
10. How do you improve the performance of a file output stream or a file
input stream?
11. How do DataOutputStream and DataInputStream support
FileOutputStream and FileInputStream?
12. What is object serialization and deserialization?
13. What three forms of serialization and deserialization does Java
support?
14. What is the purpose of the Serializable interface?
15. What does the serialization mechanism do when it encounters an
object whose class doesn’t implement Serializable?
16. Identify the three stated reasons for Java not supporting unlimited
serialization.
17. How do you initiate serialization? How do you initiate deserialization?
18. True or false: Class fields are automatically serialized.
19. What is the purpose of the transient reserved word?
20. What does the deserialization mechanism do when it attempts to
deserialize an object whose class has changed?
21. How does the deserialization mechanism detect that a serialized
object’s class has changed?
22. How can you add an instance field to a class and avoid trouble when
deserializing an object that was serialized before the instance field
was added? What JDK tool can you use to help with this task?
23. How do you customize the default serialization and deserialization
mechanisms without using externalization?
24. How do you tell the serialization and deserialization mechanisms to
serialize or deserialize the object’s normal state before serializing or
deserializing additional data items?
25. How does externalization differ from default and custom serialization
and deserialization?
26. How does a class indicate that it supports externalization?
27. True or false: During externalization, the deserialization mechanism
throws InvalidClassException with a “no valid constructor”
message when it doesn’t detect a public noargument constructor.
28. What is the difference between PrintStream’s print() and
println() methods?
29. What does PrintStream’s noargument void println() method
accomplish?
30. How do you redirect the standard input, standard output, and standard
error streams?
31. Improve Listing 4-1’s Copy application (performance wise) by using
BufferedInputStream and BufferedOutputStream. Copy
should read the bytes to be copied from the buffered input stream and
write these bytes to the buffered output stream.
32. Create a Java application named Split for splitting a large file into
a number of smaller partx files (where x starts at 0 and increments;
for example, part0, part1, part2, and so on). Each partx file
(except possibly the last partx file, which holds the remaining bytes)
will have the same size. This application has the following usage
syntax: java Split path. Furthermore, your implementation must
use the BufferedInputStream, BufferedOutputStream, File,
FileInputStream, and FileOutputStream classes.
```

#### Summary 

Java uses streams to perform I/O operations. A stream is an ordered sequence of bytes of an arbitrary length. Bytes flow over an output stream from an application to a destination and flow over an input stream from a source to an application. 

The java.io package provides several classes that identify various stream destinations and sources. These classes are descendants of the abstract OutputStream and InputStream classes. FileOutputStream and BufferedInputStream are examples. 

This chapter explored OutputStream and InputStream, followed by the byte array, file, piped, filter, buffered, data, object, and print streams. While covering object streams, it introduced the topics of serialization and externalization. The chapter concluded by revisiting standard I/O. 

### Chapter 5 Writers and Readers 

Java的streams对字节序列流很有用，但是对字符流不好，因为字节流和字符流是两种东西：一个字节是8-bit data item ，一个字符是16-bit data item 。`char and java.lang. String `天生的处理字符而不是字节。

更重要的是byte streams 不了解character sets （字符集，整型值之间的映射，称为代码点，和符号，如Unicode ）和 character encodings （字符集合的成员之间的映射，以及为效率编码这些字符的字节序列，比如UTF-8 ）

#### A BRIEF HISTORY OF CHARACTER SETS AND CHARACTER ENCODINGS 

如果你需要 stream characters ，你需要用Java’s writer and reader 类，它们支持character I/O （用chart工作而不是byte）。此外， writer and reader 也考虑了字符编码。

#### Writer and Reader Classes Overview 

层次结构：

![1533093724082](https://github.com/konekos/notes/blob/master/src/pic/1533093724082.png?raw=true)

![1533093741720](https://github.com/konekos/notes/blob/master/src/pic/1533093741720.png?raw=true)

 FilterWriter and FilterReader是抽象的。BufferedWriter and BufferedReader 不继承 FilterWriter and FilterReader ，这是和stream层次不一样的地方。

 output stream and input stream 是1.0引入的，FilterOutputStream and FilterInputStream 本该是抽象的，但是已经不能变了已经被使用了。1.1’s writer and reader 就有时间改变这些错误。

**注意**：关于 BufferedWriter and BufferedReader 直接继承Writer and Reader 而不是 FilterWriter and FilterReader ，这种变化与性能有关 。调用BufferedOutputStream’s write() methods and BufferedInputStream’s read() 方法导致调用FilterOutputStream’s write() methods and FilterInputStream’s read() 方法。BufferedWriter and BufferedReader 不继承Filexxxxer会有更好的性能。

#### Writer and Reader 

 Writer and java.io.OutputStream 的不同点：

- Writer声明了几个append() 方法添加字符到writer。因为Writer implements the java.lang.Appendable接口，和`java.util.Formatter `类协作，来输出格式化的string。
- writer声明额外的 write() 方法，和便利的`write(String str) `方法

 Reader and java.io.InputStream 的不同点：

- Reader 声明read(char[]) and read(char[], int, int)  
- Reader 没有声明available() 方法
- 声明了一个 boolean ready() 方法，当下一个read() 调用被保证不被阻塞输出就返回true。
- 声明了一个int read(CharBuffer target) 方法，从一个 character buffer 读取字符（ CharBuffer in Chapter 6 ）。

##### OutputStreamWriter and InputStreamReader 

`OutputStreamWriter `是一个输入的字符序列和要流出的字节流的桥梁。被写到这个writer的字符根据默认或指定编码被编码成字节。

**注意**： default character encoding 可以从`file.encoding `系统属性拿到。

每一个 OutputStreamWriter’s write() 方法的调用都会导致一个encoder 在给出的字符上被调用。生成的字节在buffer累积，在被写入下面的输出流之前。传递给write的方法没有被buffered。

OutputStreamWriter有4个构造器，包括下面2个：

- OutputStreamWriter(OutputStream out)  
- OutputStreamWriter(OutputStream out, String charsetName) 指定编码

**注意**：OutputStreamWriter 依赖` java.nio.charset. Charset and java.nio.charset.CharsetEncoder `抽象类执行字符编码。

例子：

```java
FileOutputStream fos = new FileOutputStream("polish.txt");
OutputStreamWriter osw = new OutputStreamWriter(fos, "8859_2");
char ch = '\u0323'; // Accented N.
osw.write(ch);
```

 InputStreamReader类是一个进来的字节流和出去的字符序列的桥梁。每个InputStreamReader’s read() 的调用导致一个或多个字节从底层输入流被读。为了效率可能会读更多的字节。

4个构造器，其中2个：

- InputStreamReader(InputStream in) 
- InputStreamReader(InputStream in, String charsetName)  

**注意**：OutputStreamWriter and InputStreamReader 声明了getEncoding() 方法，返回使用的编码

##### FileWriter and FileReader 

FileWriter 是一个方便类，写字符到文件。它继承OutputStreamWriter ，一个这个类的实例等同于下面

```java
FileOutputStream fos = new FileOutputStream(path);
OutputStreamWriter osw;
osw = new OutputStreamWriter(fos, System.getProperty("file.encoding"));
```

FileReader是一个方便类，从文件读取字符。继承InputStreamReader ，一个这个类的实例等同于：

```java
FileInputStream fis = new FileInputStream(path);
InputStreamReader isr;
isr = new InputStreamReader(fis, System.getProperty("file.encoding"));
```

FileWriter nor FileReader 都没提供它们自己的方法，你可以调用继承方法。

例子：

***Listing 5-1. Demonstrating the FileWriter and FileReader Classes*** 

```java
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
public class FWFRDemo
{
 final static String MSG = "Test message";
 public static void main(String[] args) throws IOException
 {
 try (FileWriter fw = new FileWriter("temp"))
 {
 fw.write(MSG, 0, MSG.length());
 }
 char[] buf = new char[MSG.length()];
 try (FileReader fr = new FileReader("temp"))
 {
 fr.read(buf, 0, MSG.length());
 System.out.println(buf);
 }
 }
}
```

##### BufferedWriter and BufferedReader 

BufferedWriter 写文本到字符输出流（ Writer 实例），缓冲字符，以提供高效的写单个字符、数组和字符串。构造函数：

- BufferedWriter(Writer out) 
- BufferedWriter(Writer out, int size) 

size可以指定，或者默认的8,192 bytes 。

BufferedWriter包含一个便利的void newLine() 方法，写一个行分隔符字符串，来打断这一行。

BufferedReader 从字符输入流（Reader 实例）读取文本，缓冲字符以便有效读取字符、数组和行。构造函数：

- BufferedReader(Reader in) 
- BufferedReader(Reader in, int size) 

size可以指定，或者默认8,192 bytes 。

有个方便的 String readLine() 方法，读取行文本，不包括任何换行符。

例子：

***Listing 5-2. Demonstrating the BufferedWriter and BufferedReader Classes*** 

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
public class BWBRDemo
{
 static String[] lines =
 {
 "It was the best of times, it was the worst of times,",
 "it was the age of wisdom, it was the age of foolishness,",
 "it was the epoch of belief, it was the epoch of incredulity,",
 "it was the season of Light, it was the season of Darkness,",
 "it was the spring of hope, it was the winter of despair."
 };
 public static void main(String[] args) throws IOException
 {
 try (BufferedWriter bw = new BufferedWriter(new FileWriter("temp")))
 {
 for (String line: lines)
 {
 bw.write(line, 0, line.length());
 bw.newLine();
 }
 }
 try (BufferedReader br = new BufferedReader(new FileReader("temp")))
 {
 String line;
 while ((line = br.readLine()) != null)
 System.out.println(line);
 }
 }
}
```

#### Exercise

The following exercises are designed to test your understanding of Chapter 5’s content:

```java
1. Why are Java’s stream classes not good at streaming characters?
2. What does Java provide as the preferred alternative to stream classes
when it comes to character I/O?
3. True or false: Reader declares an available() method.
4. What is the purpose of the OutputStreamWriter class? What is the
purpose of the InputStreamReader class?
5. How do you identify the default character encoding?
6. What is the purpose of the FileWriter class? What is the purpose of
the FileReader class?
7. What method does BufferedWriter provide for writing a line
separator?
8. It’s often convenient to read lines of text from standard input, and the
InputStreamReader and BufferedReader classes make this task
possible. Create a Java application named CircleInfo that, after
obtaining a BufferedReader instance that is chained to standard
input, presents a loop that prompts the user to enter a radius, parses
the entered radius into a double value, and outputs a pair of messages
that report the circle’s circumference and area based on this radius.
```

#### Summary 

Java’s stream classes are good for streaming sequences of bytes, but they’re not good for streaming sequences of characters because bytes and characters are two different things. A byte represents an 8-bit data item and a character represents a 16-bit data item. Also, Java’s char and String types naturally handle characters instead of bytes. More importantly, byte streams have no knowledge of character sets and their encodings. 

Java provides writer and reader classes to stream characters. They support character I/O (they work with char instead of byte) and take character encodings into account. The abstract Writer and Reader classes describe what it means to be a writer and a reader. 

Writer and Reader are subclassed by OutputStreamWriter and InputStreamReader, which bridge the gap between character and byte streams. These classes are subclassed by the FileWriter and FileReader convenience classes, which facilitate writing/reading characters to/from files. Writer and Reader are also subclassed by BufferedWriter and BufferedReader, which buffer characters for efficiency. 

## PART Ⅲ New I/O APIs 

### Chapter 6 Buffers 

NIO是基于buffer的，它的内容通过channels被发送到 I/O services 或者从 I/O services 接收。

#### Introducing Buffers 

一个buffer是一个对象，存储固定数量的发送到或从 I/O service（操作系统执行输入输出的组件）接收的数据。它处于应用和channel之间，channel写缓冲数据到service，或者从service读取数据存到buffer。

缓冲区具有四个属性：

- Capacity: 可以存在buffer的data item的总数。创建时指定，并且之后不能更改。
- Limit: 第一个不能被读写的元素的以0为基准的索引。换句话说，它表明了buffer里的 “live” data items 。
- Position:  下一个可以被读的data item或者data item 可以被写位置的以0为基准的索引。
- Mark:  当buffer的 reset() 方法被调用时设置位置的以0为基准的索引。Mark开始未被定义。

这4个属性联系如下：0 <= mark <= position <= limit <= capacity 

![1533108252365](https://github.com/konekos/notes/blob/master/src/pic/1533108252365.png?raw=true)

buffer最多可以存储7个元素。Mark初始未被定义，position被设置为0，limit被设置为容量（7），你只可以访问0-6 positions。

#### Buffer and its Children 

 Buffer’s 的方法：

***Table 6-1. Buffer Methods*** 

| Method                            | Description                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| Object array()                    | 返回支持这个buffer的array。这个方法用于允许  array-backed buffers 更效率地传递给 native code 。具体的子类重写这个方法提供更强大的类型返回值。当buffer由一个只读数组支撑返回` java.nio.ReadOnlyBufferException `，不可访问数组返回` java.lang.UnsupportedOperationException `。 |
| int arrayOffset()                 | 返回buffer支持数组内第一个buffer元素的offset。当buffer由一个数组支撑， buffer position p 对应array index p +  arrayOffset() 。在调用此方法前调用  hasArray() 确保buffer有可访问的支撑数组。只读抛出  ReadOnlyBufferException ，不可访问抛出 UnsupportedOperationException 。 |
| int capacity()                    | 返回数组的容量                                               |
| Buffer clear()                    | 清空buffer。position设置为0，limit被设置为容量，mark被丢弃。这个方法不删除buffer的数据，但是方法名就像删除了似得，因为大部分要使用这个方法的情况都是删除。 |
| Buffer flip()                     | 翻转这个buffer。limit被设置为当前position，然后position被设置为0。Mark被定义的话会被丢弃。 |
| boolean hasArray()                | 如果  buffer 由array支撑且不是只读返回true；否则false。返回true时， array() and arrayOffset() 都可以安全调用。 |
| boolean hasRemaining()            | 如果至少一个元素残留在buffer返回true（就是，在当前position和limit之间）。否则，返回false。 |
| boolean isDirect()                | 当buffer是  direct byte buffer 返回true。否则false           |
| boolean isReadOnly()              | buffer read-only返回true。                                   |
| int limit()                       | 返回limit                                                    |
| Buffer limit(int newLimit)        | 设置limit为newLimit。当position大于newLimit，position被设置为newLimit。当mark被定义为大于newLimit，mark被丢弃。newLimit是负的或者大于容量抛出` java.lang.IllegalArgumentException `；否则返回这个buffer。 |
| Buffer mark()                     | 设置Mark到它的position，返回buffer。                         |
| int position()                    | 返回position                                                 |
| Buffer position (int newPosition) | 设置position为newPosition。当Mark被定义大于newPosition，Mark被丢弃。为负或大于当前limit抛出`  IllegalArgumentException `；否则返回buffer。 |
| int remaining()                   | 返回当前position和limit之间元素的数量                        |
| Buffer reset()                    | 重置buffer的position到先前的Mark位置。调用这个方法不会改变和丢弃mark的值。mark没设置抛出` java.nio.InvalidMarkException `；否则返回buffer。 |
| Buffer rewind()                   | 倒回去，然后返回这个buffer。position被设为0，mark被丢弃。    |

上面方法很多都返回buffer实例，所以可以链式编程。例如：

```java
buf.mark();
buf.position(2);
buf.reset();
```

可以用：

```java
buf.mark().position(2).reset();
```

6-1也表明所有buffer都可以读，但是不是所有都能被写——例如，一个由memory-mapped file支撑的buffer是只读的。一定不能写只读的buffer；否则抛出`ReadOnlyBufferException `。试图写的时候调用isReadOnly() 看是否可以写。

**警告**：Buffers不是线程安全的。你想从多线程访问buffer必须采取同步。

`java.nio `包有几个抽象类拓展了Buffer，除了Boolean之外，每个基本类型都有一个 ： ByteBuffer, CharBuffer, DoubleBuffer, FloatBuffer, IntBuffer, LongBuffer, and ShortBuffer。此外，也有MappedByteBuffer 。

**注意**：操作系统执行面向字节的I/O，你使用 ByteBuffer来创建byte-oriented buffers 。其他primitive-type buffer 类让你创建multibyte view buffers （discussed later ），然后你可以概念性地对字符，双精度浮点，32-bits integer等等执行I/O。然而，I/O操作是字节的流运行的。

演示例子：

***Listing 6-1. Demonstrating a Byte-Oriented Buffer*** 

```java
import java.nio.Buffer;
import java.nio.ByteBuffer;
public class BufferDemo
{
 public static void main(String[] args)
 {
 Buffer buffer = ByteBuffer.allocate(7);
 System.out.println("Capacity: " + buffer.capacity());
 System.out.println("Limit: " + buffer.limit());
 System.out.println("Position: " + buffer.position());
 System.out.println("Remaining: " + buffer.remaining());
 System.out.println("Changing buffer limit to 5");
 buffer.limit(5);
 System.out.println("Limit: " + buffer.limit());
 System.out.println("Position: " + buffer.position());
 System.out.println("Remaining: " + buffer.remaining());
 System.out.println("Changing buffer position to 3");
 buffer.position(3);
 System.out.println("Position: " + buffer.position());
 System.out.println("Remaining: " + buffer.remaining());
 System.out.println(buffer);
 }
}

```

不能实例化buffer因为是抽象的，而是用ByteBuffer 类的 allocate() 方法分配一个7字节的buffer。

输出：

```java
Capacity: 7
Limit: 7
Position: 0
Remaining: 7
Changing buffer limit to 5
Limit: 5
Position: 0
Remaining: 5
Changing buffer position to 3
Position: 3
Remaining: 2
java.nio.HeapByteBuffer[pos=3 lim=5 cap=7]
```

最后的buffer是一个` java.nio.HeapByteBuffer `实例。

#### Buffers in Depth 

这节深入探索buffer创建，读写，flipping，marking， Buffer subclass operations ，byte ordering和direct buffers 。

**注意**：虽然 primitive-type buffer类有相似的能力，ByteBuffer 是最大最全能通用的。毕竟，字节是操作系统用来传递data items的最小单元。

##### Buffer Creation 

ByteBuffer and the other primitive-type buffer类声明了很多方法创建一个该类型的buffer。例如ByteBuffer：

- ByteBuffer allocate(int capacity): 分配一个新的指定容量的byte buffer。position是0，limit是容量，mark未定义，每个元素被初始化为0。有一个支撑数组，offset是0。容量为负抛出`IllegalArgumentException `。

- ByteBuffer allocateDirect(int capacity):  分配一个新的指定容量的direct byte buffer (discussed later) 。position是0，limit是容量，mark未定义，每个元素被初始化为0。它是否有一个支撑数组是未知的。容量为负抛出`IllegalArgumentException `。  

  在JDK 7之前，这个方法分配的direct buffers 和page边界对齐。在JDK 7，实现改变了，不再是页面对齐的。减少了创建大量buffers应用的内存需求。（To learn about an operating system’s paging memory-management mechanism, which is based on pages, check out Wikipedia’s “Paging” topic at https://en.wikipedia.org/wiki/Paging ）

- ByteBuffer wrap(byte[] array):  将一个字节数组封装到buffer。新buffer是数组支撑的；就是，改变buffer就会改变数组，反之亦然。容量和limit是array.length ，position为0，Mark未定义。array的offset是0。

- ByteBuffer wrap(byte[] array, int offset, int length)   将一个字节数组封装到buffer。新buffer是数组支撑的。新buffer的容量为array.length，position被设置为offset，limit被设置为offset+length，mark未定义。数组offset是0。是offset是负，或大于array.length或者当length是负或大于array.length - offset 抛出` java.lang. IndexOutOfBoundsException `。

下面两种方式创建buffer：

```java
ByteBuffer buffer = ByteBuffer.allocate(10);
byte[] bytes = new byte[200];
ByteBuffer buffer2 = ByteBuffer.wrap(bytes);
```

```java
buffer = ByteBuffer.wrap(bytes, 10, 50);
```

上面这个创建了一个buffer，position=10，limit=50，capacity=200。虽然看起来buffer只能访问这个数组的子串，但你可以通过 array()  方法访问这些支撑数组（就是byte[] array()  ）。并且hasArray()  返回true。（当 hasArray() 返回true，你需要调用arrayOffset() 得到数组里第一个data item的offset）

例子：

***Listing 6-2. Creating Byte-Oriented Buffers via Allocation and Wrapping*** 

```java
import java.nio.ByteBuffer;
public class BufferDemo
{
 public static void main(String[] args)
 {
 ByteBuffer buffer1 = ByteBuffer.allocate(10);
 if (buffer1.hasArray())
 {
 System.out.println("buffer1 array: " + buffer1.array());
 System.out.println("Buffer1 array offset: " +
 buffer1.arrayOffset());
 System.out.println("Capacity: " + buffer1.capacity());
 System.out.println("Limit: " + buffer1.limit());
 System.out.println("Position: " + buffer1.position());
 System.out.println("Remaining: " + buffer1.remaining());
 System.out.println();
 }
 byte[] bytes = new byte[200];
 ByteBuffer buffer2 = ByteBuffer.wrap(bytes);
 buffer2 = ByteBuffer.wrap(bytes, 10, 50);
 if (buffer2.hasArray())
 {
 System.out.println("buffer2 array: " + buffer2.array());
 System.out.println("Buffer2 array offset: " +
 buffer2.arrayOffset());
 System.out.println("Capacity: " + buffer2.capacity());
 System.out.println("Limit: " + buffer2.limit());
 System.out.println("Position: " + buffer2.position());
 System.out.println("Remaining: " + buffer2.remaining());
 }
 }
}

```

编译运行，输出：

```
buffer1 array: [B@659e0bfd
Buffer1 array offset: 0
Capacity: 10
Limit: 10
Position: 0
Remaining: 10
buffer2 array: [B@2a139a55
Buffer2 array offset: 0
Capacity: 200
Limit: 60
Position: 10
Remaining: 50
```

除了管理保存在外部数组的数据元素外（via the wrap() 方法），buffer也能管理保存在其他buffers的数据。当你创建一个buffer管理另一个buffer的data，被创建的buffer被认为是*view buffer*。在两个buffer中所做的更改都反映在另一个buffer中。

View buffers 用Buffer子类的duplicate()方法创建。产生的view buffer相当于原始buffer; 两个buffer共享相同的data items且容量相同。然而，每个buffer有它自己的 position, limit, and mark 。当被duplicated 的buffer是read-only或者direct，view buffer 也是read-only或者direct。

考虑以下：

```java
ByteBuffer buffer = ByteBuffer.allocate(10);
ByteBuffer bufferView = buffer.duplicate();
```

两个buffer共享array。此时，position, limit, and (undefined) mark 是一样的。然而，这些属性可以彼此独立于另一个buffer。

也可以调用ByteBuffer’s asxBuffer() 创建view buffers。例如，LongBuffer asLongBuffer() 返回一个view buffer将字节buffer概念化为长整型的buffer。 

**注意**：Read-only view buffers可以调用比如ByteBuffer asReadOnlyBuffer() 的方法创建。任何view buffer改变的尝试导致` ReadOnlyBufferException `。然而，原始buffer内容（假设不是只读）可以变，只读的view buffer将反映这些变化 。

##### Buffer Writing and Reading 

ByteBuffer and the other primitive-type buffer类声明了几个重载的 put() and get() 方法用于写 data items 和从buffer读 data items  。这些方法在需要索引参数的时候是绝对的不需要索引时是相对的。

例如，ByteBuffer声明绝对的ByteBuffer put(int index, byte b) 方法在索引index存储byte b，以及byte get(int index) 方法获取index位置的字节。也声明了相对的 ByteBuffer put(byte b)  方法在当前buffer的position存b然后增加position，以及相对的byte get() 获取buffer当前position的字节并且增加当前position。

绝对的put() and get()方法当索引是负或超出或者等于limit抛出IndexOutOfBoundsException 。相对的 put()  在当前position大于或等于limit时抛出`java.nio.BufferOverflowException `，相对的get() 方法在当前position大于或等于limit时抛出`java.nio.BufferUnderflowException `。另外，两种put()方法都在只读时抛出` ReadOnlyBufferException `。

例子：

***Listing 6-3. Writing Bytes to and Reading Them from a Buffer*** 

```java
import java.nio.ByteBuffer;
public class BufferDemo
{
 public static void main(String[] args)
 {
 ByteBuffer buffer = ByteBuffer.allocate(7);
 System.out.println("Capacity = " + buffer.capacity());
 System.out.println("Limit = " + buffer.limit());
 System.out.println("Position = " + buffer.position());
 System.out.println("Remaining = " + buffer.remaining());
 buffer.put((byte) 10).put((byte) 20).put((byte) 30);
 System.out.println("Capacity = " + buffer.capacity());
 System.out.println("Limit = " + buffer.limit());
 System.out.println("Position = " + buffer.position());
 System.out.println("Remaining = " + buffer.remaining());
 for (int i = 0; i < buffer.position(); i++)
 System.out.println(buffer.get(i));
 }
}
```

编译运行输出：

```
Capacity = 7
Limit = 7
Position = 0
Remaining = 7
Capacity = 7
Limit = 7
Position = 3
Remaining = 4
10
20
30

```

如图：

![1533118585640](https://github.com/konekos/notes/blob/master/src/pic/1533118585640.png?raw=true)

随后的调用get() 方法没有改变position，还是3。

**技巧**：为获得最高效率，你可以执行批量数据传输，使用ByteBuffer put(byte[] src), ByteBuffer put(byte[] src, int offset, int length), ByteBuffer get(byte[] dst), and ByteBuffer get(byte[] dst, int offset, int length)  来读写字节数组。

##### Flipping Buffers 

在填充buffer后，你必须准备它用于通过channel排干。您通过缓冲区时，通道将访问未定义的数据，超出当前位置。 

要解决这个问题，你可以reset position为0，但是channel怎么知道什么时候到达插入的数据的末尾呢？解决办法是使用limit属性，它表明了buffer活动区间的end。基本上，设置limit到当前position，然后重置current position为0。

你可以执行以下代码完成这个任务，也会清除所有定义的mark：

```java
buffer.limit(buffer.position()).position(0);
```

然而，有一个更简便的做法：

```java
buffer.flip();
```

两种情况，buffer都准备好去排干了。

假设`buffer.flip() `在listing 6-3最后执行，figure 6-3 表明了 flip() 之后的buffer状态：

![1533175926471](https://github.com/konekos/notes/blob/master/src/pic/1533175926471.png?raw=true)

调用 buffer.remaining() 返回3。这个值表明了能用于排干的字节数。

Listing 6-4 给出另一个buffer-flipping示范，使用了character buffer。

***Listing 6-4. Writing Characters to and Reading Them from a Character Buffer*** 

```java
import java.nio.CharBuffer;
public class BufferDemo
{
 public static void main(String[] args)
 {
 String[] poem =
 {
 "Roses are red",
 "Violets are blue",
 "Sugar is sweet",
 "And so are you."
 };
 CharBuffer buffer = CharBuffer.allocate(50);
 for (int i = 0; i < poem.length; i++)
 {
 // Fill the buffer.
 for (int j = 0; j < poem[i].length(); j++)
 buffer.put(poem[i].charAt(j));
 // Flip the buffer so that its contents can be read.
 buffer.flip();
 // Drain the buffer.
 while (buffer.hasRemaining())
 System.out.print(buffer.get());
 // Empty the buffer to prevent BufferOverflowException.
 buffer.clear();
 System.out.println();
 }
 }
}

```

输出：

```
Roses are red
Violets are blue
Sugar is sweet
And so are you.
```

**注意**：rewind()和 flip() 类似但是忽略limit。并且，调用flip()2次也不会得到buffer的初始状态，而是size为0。调用put() 抛出`BufferOverflowException `，调用get()抛出`BufferUnderflowException `，get(int i)抛出` IndexOutOfBoundsException `。

##### Marking Buffers 

使用mark() 标记buffer，随后调用reset()返回标记位置。例如，假设你执行了`ByteBuffer buffer = ByteBuffer.allocate(7);, `然后`buffer.put((byte) 10).put((byte) 20).put((byte) 30).put((byte) 40);, `然后`buffer.limit(4);. `。当前position和limit都为4。

继续，假设你执行了` buffer.position(1).mark(). position(3) `。6-4显示了buffer的状态：

![1533177587773](https://github.com/konekos/notes/blob/master/src/pic/1533177587773.png?raw=true)

如果你把buffer发到一个channel，字节40会被发送（因为position(3) 现在的position是3），然后position会提到4。如果你接着执行buffer.reset() ；然后把buffer发送给channel，position会被设置到mark (1) ，bytes 20, 30, and 40  （所有从当前位置到limit的字节），会被发到channel（以这个顺序）。

代码：

***Listing 6-5. Marking the Current Buffer Position and Resetting the Current Position to the Marked Position*** 

```java
import java.nio.ByteBuffer;
public class BufferDemo
{
 public static void main(String[] args)
 {
 ByteBuffer buffer = ByteBuffer.allocate(7);
 buffer.put((byte) 10).put((byte) 20).put((byte) 30).put((byte) 40);
 buffer.limit(4);
 buffer.position(1).mark().position(3);
 System.out.println(buffer.get());
 System.out.println();
 buffer.reset();
 while (buffer.hasRemaining())
 System.out.println(buffer.get());
 }
}

```

输出：

```
40
20
30
40
```

**警告**：不要混淆reset() 和clear() 。clear() 方法标记buffer为空而 reset() 改变buffer的当前position到先前mark的位置，或者没标记就抛出` InvalidMarkException `。

##### Buffer Subclass Operations 

ByteBuffer and the other primitive-type buffer类声明了compact() 方法，对压缩buffer很有用，通过复制当前position到limit的所有bytes到buffer的开头。index p = position() 的字节被复制到index 0， index p + 1 的字节复制到index 1，...直到 index limit() - 1 被复制到index n = limit() - 1 - p 。buffer的当前position被设置为n+1，limit被设置为容量。mark被定义就丢弃。

你从一个buffer写数据后调用compact，来解决不是所有buffer内容都被写入的场景。考虑下面：

```
buf.clear(); // Prepare buffer for use
while (in.read(buf) != -1)
{
 buf.flip(); // Prepare buffer for draining.
 out.write(buf); // Write the buffer.
 buf.compact(); // Do this in case of a partial write.
}

```

compact() 方法调用移动没有被写的buffer数据到buffer开头，然后下次read()可以添加读数据到buffer的数据，而不是重写。

你偶尔可能需要比较buffer的等同或顺序。所有的buffer除了` MappedByteBuffer `都重写了equals() and compareTo() 来执行比较———MappedByteBuffer继承ByteBuffer没重写。

```
System.out.println(bytBuf1.equals(bytBuf2));
System.out.println(bytBuf1.compareTo(bytBuf2));
```

equals()， same element type ， same number of remaining elements ，two sequences of remaining elements ，不取决于starting positions。

compareTo() ，比较顺序，比较sequences of remaining elements lexicographically ，不考虑starting position 。

##### Byte Ordering 

Nonbyte primitive types except for Boolean 由几个字节组成：一个字符或short integer占2个字节，32-bit integer或者一个浮点值占4字节，long integer或双精度浮点值占8字节。这些多字节类型的值被存在连续的内存位置的序列。然而，这些字节的排序可能因操作系统而异。

例如，32-bit long integer 0x10203040 。This value’s four bytes could be stored in memory (from low address to high address) as 10, 20, 30, 40 ；这种编排方式叫 big endian order (the most-significant byte, the “big” end, is stored at the lowest address) 。也可以用40, 30, 20, 10 存；叫little endian order (the least-significant byte, the “little” end, is stored at the lowest address)。

Java提供` java.nio.ByteOrder ` 类帮你处理字节排序在多字节值互写的时候。声明了`ByteOrder nativeOrder() `返回操作系统的ByteOrder 实例。这个实例是 ByteOrder’s BIG_ENDIAN and LITTLE_ENDIAN constants 的其中之一，没有其他的，你可以比较 nativeOrder()的返回值和常数。

多字节类也有提供。

order() 返回的 ByteOrder 可能考虑buffer的创建方式。view buffer的order不能被变。

ByteBuffer 不同与多字节类在字节排序上。默认字节排序总是 big endian ，即使操作系统是little endian 。因为Java’s default byte order 是 big endian ，让类文件和序列化的对象存储数据一致。

声明了 ByteBuffer order(ByteOrder bo) 来改变字节排序。

改变的字节排序对例如 LongBuffer asLongBuffer() 形成的view buffer是可见的。

##### Direct Byte Buffers 

不像多字节buffers， byte buffers 可以作为channel-based I/O 的sources和targets。这不应该是一个惊喜因为操作系统在内存区域执行IO是连续的8字节的序列（不是浮点和32-bit integer）。

操作系统可以直接访问进程的地址空间。例如，操作系统可以直接访问JVM进程地址空间来执行基于字节数组的数据传输操作。然而，JVM可能不会连续地存储字节数组或者它的垃圾收集器把字节数组移动到另一个位置。由于这些限制， direct byte buffers 被创建。

***direct byte buffer***，是一个byte buffer ，与 channels 和native code 交互来执行I/O。direct byte buffer 尝试在内存区域储存字节元素，channel通过native code用字节元素来执行 direct (raw) access ，告诉操作系统直接排干/填充内存区域。

Direct byte buffers是JVM上最有效的执行I/O的方法。尽管你也可以传递nondirect byte buffers给通道，可能产生性能问题，因为nondirect byte buffers不总是能够充当native I/O operations的target。

当传递了一个 nondirect byte buffer ，channel可能需要创建一个临时的direct byte buffer ，复制nondirect byte buffer的内容到direct byte buffer，在临时的direct byte buffer上执行I/O操作，并且复制临时 direct byte buffer的内容到nondirect byte buffer 。临时direct byte buffer会被垃圾收集。

虽然优化了I/O，创建 direct byte buffer可能很昂贵因为在JVM堆之外的内存需要由操作系统分配，建立/拆除这个内存会比这个buffer在堆内的时间更长。

通过`ByteBuffer’s allocateDirect() `得到direct byte buffer。

#### Exercise

```
The following exercises are designed to test your understanding of Chapter 6’s content:
1. What is a buffer?
2. Identify a buffer’s four properties.
3. What happens when you invoke Buffer’s array() method on a
buffer backed by a read-only array?
4. What happens when you invoke Buffer’s flip() method on a buffer?
5. What happens when you invoke Buffer’s reset() method on a
buffer where a mark has not been set?
6. True or false: Buffers are thread-safe.
7. Identify the classes that extend the abstract Buffer class.
8. How do you create a byte buffer?
9. Define view buffer.
10. How is a view buffer created?
11. How do you create a read-only view buffer?
12. Identify ByteBuffer’s methods for storing a single byte in a byte
buffer and fetching a single byte from a byte buffer.
13. What causes BufferOverflowException or
BufferUnderflowException to occur?
14. What is the equivalent of executing buffer.flip();?
15. True or false: Calling flip() twice returns you to the original state.
16. What is the difference between Buffer’s clear() and reset()
methods?
17. What does ByteBuffer’s compact() method accomplish?
18. What is the purpose of the ByteOrder class?
19. Define direct byte buffer.
20. How do you obtain a direct byte buffer?
21. Why could it be expensive to create a direct byte buffer?
22. Create a ViewBufferDemo application that populates a byte buffer
with the values 0, 0x6e, 0, 0x69, 0, 0x6f; creates a character view
buffer; and iterates over the view buffer, outputting each character.
```

#### Summary 

A buffer is an NIO object that stores a fixed amount of data to be sent to or received from an I/O service. It sits between an application and a channel that writes the buffered data to the service or reads the data from the service and deposits it into the buffer. 

Buffers possess capacity, limit, position, and mark properties. These four properties are related as follows: 0 <= mark <= position <= limit <= capacity. 

Buffers are implemented by abstract classes that derive from the abstract Buffer class. These classes include ByteBuffer, CharBuffer, DoubleBuffer, FloatBuffer, IntBuffer, LongBuffer, and ShortBuffer. Furthermore, ByteBuffer is subclassed by the abstract MappedByteBuffer class. 

In this chapter, you learned how to create buffers (including view buffers), write and read buffer contents, flip buffers, mark buffers, and perform additional operations on buffers such as compaction. You also learned about byte ordering and direct byte buffers. 

### Chapter 7 Channels 

Channel协同buffer执行高性能I/O。这章介绍channel类型。

#### Introducing Channels 

*channel*是一个对象代表一个到 hardware device, a file, a network socket, an application component, or another entity(能读写和执行其他I/O操作)的一个 open connection 。channels效率地在byte  buffers 和操作系统 I/O service sources or destinations之间传递数据。

**注意**：Channels是访问 I/O services 的gateways 。Channels使用byte buffers作为端点传递和接收数据。

在操作系统文件句柄（ file handle ，Windows）/文件描述符（file descriptor ） 与 Channel之间总存在一一对应。当你在file context使用channel，channel总是被连接到一个打开的文件描述符。尽管channel比 file descriptors更抽象，他们仍然能够对操作系统的I/O设施进行建模。

#### Channel and Its Children 

通过`java.nio.channels` and `java.nio.channels.spi `提供channel支持。应用程序与位于前包中的类型进行交互 ；定义新的selector providers的开发人员使用后包。

所有的channels都是最终实现了`java.nio.channels.Channel `接口的类的实例。Channel声明下面的方法：

- void close():  关闭channel。当channel已经关闭，调用close()无效。如果另一个线程已经调用了close()，一个新的close()调用会在第一个调用结束之前阻塞住，之后 close() 没有效果地返回。I/O错误抛出` java.io.IOException`。在channel关闭后，任何在它上面尝试I/O操作都会抛出`java.nio. channels.ClosedChannelException`。
- boolean isOpen():  返回channel的开闭状态。

这些方法表明只有两个操作是对所有channel通用：关闭channel和确定打开或关闭状态。为了支持I/O，channel被扩展为`java.nio.channels. WritableByteChannel `和`java.nio.channels.ReadableByteChannel `接口：

- WritableByteChannel 声明了一个抽象的int write(ByteBuffer buffer) 方法，从buffer到一个序列的字节到当前channel。返回实际被写的字节数量。当channel没有开放写抛出`java.nio.channels.NonWritableChannelException`，channel关闭了抛出` java.nio. channels.ClosedChannelException`，写的时候另一个线程关闭了channel抛出`java.nio.channels.AsynchronousCloseException`，当写操作执行的时候当前线程被另一个线程打断了（从而关闭通道并设置当前线程的中断状态）抛出`java.nio.channels.ClosedByInterruptException`，I/O error 抛出`IOException`。
- ReadableByteChannel 声明了抽象read(ByteBuffer buffer) 方法，从当前channel读取字节到buffer。返回实际读取的字节数（没有字节可读返回-1）。当channel没开放读抛出`java.nio.channels. NonReadableChannelException`，channel关闭了抛出`ClosedChannelException`，读的时候另一个线程关闭了channel抛出`java.nio.channels.AsynchronousCloseException`，当读操作执行的时候当前线程被另一个线程打断了抛出`java.nio.channels.ClosedByInterruptException`,从而关闭通道并设置当前线程的中断状态;I/O error 抛出`IOException`。

**注意**：一个只实现了WritableByteChannel or ReadableByteChannel的channel类是单向的。从可写的byte channel读或者从可读的byte channel写都导致异常抛出。

你可以使用 instanceof  操作符确定一个channel实例实现了那个接口。因为测试这两个接口很尴尬，Java提供` java.nio.channels.ByteChannel `接口，是一个空的标记接口继承了这两个接口。当你要了解一个channel是不是双向的，使用`channel instanceof ByteChannel `会很方便。

Channel也被扩展为`java.nio.channels.InterruptibleChannel `接口。 InterruptibleChannel接口描述了一个可以被异步closed and interrupted 的channel。接口重写了父接口close() 方法的header，表示了这个方法的channel约束的附加规定：任何在这个channel线程的I/O操作被阻塞都会收到`AsynchronousCloseException `（IOException 的派生）。

实现这个接口的channel可异步*closeable*：当一个线程带一个可中断channel上在I/O操作阻塞了，另一个线程可以调用channel的close()方法。这导致被阻塞的线程收到一个被抛出的` AsynchronousCloseException `实例。

实现这个接口的channel也是*interruptible*：当一个线程带一个可中断channel上在I/O操作阻塞了，另一个线程可以调用channel的interrupt()方法。这样会使channel关闭，被阻塞的线程收到`ClosedByInterruptException `实例，并有它的nterrupt status set 。（当一个线程的 interrupt status 被设置，并且在channel上调用阻塞的I/O操作 ，channel被关闭，线程马上收到`ClosedByInterruptException `实例；它的中断状态将保持设置 ）

NIO的设计者当阻塞线程被打断选择关闭channel，因为他们不能找到一个方式去可靠处理打断的I/O操作，通过操作系统用相同的方式。保证确定性行为的唯一方式就是关闭channel。

**技巧**：你可以用类似` channel instanceof InterruptibleChannel `来确定 channel 是不是支持`asynchronous closing and interruption`。

除了channels，有2种方式获取channel：

- `java.nio.channels` 包提供了一个Channels utility类，它有2个方法从streams得到channel。下面每个方法，当channel是关闭的，stream也是关闭的，并且channel没有被 buffered。
  - WritableByteChannel newChannel(OutputStream outputStream) 对给定输出流返回一个writable byte channel 
  - ReadableByteChannel newChannel(InputStream inputStream)  对给定输入流返回一个 readable byte channel
- 很多 classic I/O 类被改造支持channel创建。例如，`java.io.RandomAccessFile `声明了FileChannel getChannel()  方法得到一个 file channel ，`java.net.Socket `声明了 SocketChannel getChannel() 方法返回一个socket channel。

例子：

***Listing 7-1. Copying Bytes from an Input Channel to an Output Channel*** 

```java
@Test
    public void testUseChannelCopyFile() throws IOException {
        try (ReadableByteChannel src = Channels.newChannel(System.in);
             WritableByteChannel dest = Channels.newChannel(System.out)) {
            copyAlt(src, dest);
        }
    }

    static void copy(ReadableByteChannel src, WritableByteChannel dest) throws IOException {
        ByteBuffer buffer = ByteBuffer.allocateDirect(2048);
        while (src.read(buffer) != -1) {
            buffer.flip();
            dest.write(buffer);
            buffer.compact();
        }
        buffer.flip();
        while (buffer.hasRemaining()) {
            dest.write(buffer);
        }
    }

    static void copyAlt(ReadableByteChannel src, WritableByteChannel dest) throws IOException {
        ByteBuffer buffer = ByteBuffer.allocateDirect(2048);
        while (src.read(buffer) != -1) {
            buffer.flip();
            while (buffer.hasRemaining()) {
                dest.write(buffer);
            }
            buffer.clear();
        }
    }
```

7-1展示了2种方法从标准输入流复制字节到标准输出流。在copy() 方法，目的是最小化操作系统I/O调用（通过write() 调用），尽管更多的数据最终作为compact() 方法调用的结果被复制。第二种方法，copyAlt() ，目标是消除数据复制，尽管可能会出现更多的操作系统I/O调用。 

 copy() and copyAlt() 方法首先分配direct byte buffer （上章最后一节），然后一个while循环持续从source channel读字节，直到end-of-input （read() returns -1 ）。随着读取，buffer被 flipped 然后可以被排干。下面是方法的分歧点：

- copy() 方法的while循环只调用了write()一次。因为 write() 可能没完全排干buffer， compact()  被调用在下次读之前compact buffer。Compaction 确保没被写的buffer内容在下次读操作不会被覆盖。循环之后，flip buffer，准备排干任何剩下的内容，然后用 hasRemaining() and write() 完成buffer的排干。
- copyAlt() 方法while循环里还有个while循环，使用hasRemaining() and write() 持续排干buffer，直到buffer清空。然后clear()  ，清空buffer然后在下次read调用可以被充满。

**注意**：认识到单个write() 方法调用不能输出buffer的全部内容是很重要的。类似地，单个的 read()  也可能不会完全充满buffer。

#### Channels in Depth 

本节深入了解exploring scatter/gather I/O, file channels, socket channels, and pipes 。

##### Scatter/Gather I/O 

Channel提供了跨多个buffer执行单一I/O操作的功能。也就是*scatter/gather I/O*（也叫*vectored I/O*）。

在写操作的context，几个buffer的内容按顺序被集中（排干），然后通过channel送到目的地。这些buffers不需要有等同的容量。在读操作的context，channel的内容按顺序被分散（填充）到多个buffer；每个buffer被填充到limit，直到channel空了或buffer全部空间都用了。

**注意**：现代操作系统提供了API支持vectored I/O 消除系统调用（或至少减少），因此提高性能。例如，Win32/Win64 APIs 提供了ReadFileScatter() and WriteFileGather() 。

Java提供`java.nio.channels.ScatteringByteChannel `接口支持 scattering，以及`java.nio.channels.GatheringByteChannel `接口来支持gathering 。

ScatteringByteChannel 提供以下方法：

- long read(ByteBuffer[] buffers, int offset, int length) 
- long read(ByteBuffer[] buffers) 

GatheringByteChannel 方法：

- long write(ByteBuffer[] buffers, int offset, int length) 
- long write(ByteBuffer[] buffers) 

例子：

***Listing 7-2. Demonstrating Scatter/Gather*** 

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.Channels;
import java.nio.channels.GatheringByteChannel;
import java.nio.channels.ScatteringByteChannel;

/**
 * @author @Jasu
 * @date 2018-08-03 10:47
 */
public class ChannelDemo1 {
    public static void main(String[] args) throws IOException {
        ScatteringByteChannel src;
        FileInputStream fis = new FileInputStream("E:\\SpringSourceCode\\src\\main\\resources\\image.txt");
        src = (ScatteringByteChannel) Channels.newChannel(fis);
        ByteBuffer buffer1 = ByteBuffer.allocateDirect(5);
        ByteBuffer buffer2 = ByteBuffer.allocateDirect(3);
        ByteBuffer[] buffers = {buffer1, buffer2};
        src.read(buffers);

        buffer1.flip();
        while (buffer1.hasRemaining()) {
            System.out.println(buffer1.get());
        }

        System.out.println();

        buffer2.flip();
        while (buffer2.hasRemaining()) {
            System.out.println(buffer2.get());
        }

        buffer1.rewind();
        buffer2.rewind();
        GatheringByteChannel dest;
        FileOutputStream fos = new FileOutputStream("E:\\SpringSourceCode\\src\\main\\resources\\imageCopy.txt");
        dest = (GatheringByteChannel) Channels.newChannel(fos);
        buffers[0] = buffer2;
        buffers[1] = buffer1;
        dest.write(buffers);
    }
}
```

通过实例化`java.io.FileInputStream `并把这个实例传递给` ReadableByteChannel newChannel(InputStream inputStream) `获取了一个scattering byte channel 。返回的` ReadableByteChannel `被转换为`ScatteringByteChannel `因为这个实例确实是 a file channel （discussed later ），它实现了`ScatteringByteChannel `。

然后，创建了一对direct byte buffers，容量不同。然后这两个buffer被存在了数组，数组被传递到`read(ByteBuffer[]) `来填充。

填充之后， flip buffer然后输出内容到标准输出。输出后，rewound  buffer准备通过一个聚集操作排干buffer。

实例化`java.io.FileOutputStream `传递给Channels的`WritableByteChannel newChannel(OutputStream outputStream) `方法。返回的` WritableByteChannel `被强转为`GatheringByteChannel `。

最后，分配这些buffers到buffer数组，和它们最开始的分配顺序相反，传递数组到`write(ByteBuffer[]) `来排干。

```
原文件
12345abcdefg
输出
49
50
51
52
53

97
98
99
输出文件
abc12345

```



##### File Channels 

之前提到，` RandomAccessFile `声明了`FileChannel getChannel() `返回一个file channel实例，描述了一个到文件的 open connection。其实FileInputStream and FileOutputStream也提供了相同的方法。与之不同，`java.io.FileReader` and `java.io.FileWriter` 没有提供活动file channel的方式。

**警告**：从FileInputStream’s getChannel() 返回的file channel是只读的，FileOutputStream’s getChannel() 返回的是只写的。.... ，否则异常。

抽象`java.nio.channels.FileChannel `类描述了一个file channel。这个类实现了` InterruptibleChannel `接口，file channels是interruptible 。还实现了`ByteChannel, GatheringByteChannel, and ScatteringByteChannel `接口，你可以写到它，从它读，在下面的文件执行scatter/gather I/O。然而，还有更多。

**注意**：不像buffers线程不安全，file channels是线程安全的。

一个file channel维护一个当前position到文件，`FileChannel `让你得到和改变这个position。它还允许您请求将缓存的数据强制发送到磁盘，读写文件内容，得到channel下的文件的大小， truncate文件，尝试锁住全部文件或者仅一个文件区域，执行memory-mapped file I/O ，使用操作系统可能优化的方式直接传输数据到另一个channel。

***Table 7-1. FileChannel Methods*** 

| Method                                   | Description                                                  |
| ---------------------------------------- | ------------------------------------------------------------ |
| void force(boolean metadata)             | 请求把所有对这个file channel的更新提交到存储设备。当这个方法返回，当文件处于本地存储设备的时候，所有基于channel对文件的修改都被提交。然而，当文件不在本地（例如 网络文件系统 ），应用不能确定修改是不是被提交了。（不保证其他地方定义的方法做出的更改会被提交。比如，通过  mapped byte buffer 的改变可能不会被提交）                                                                                   metadata值表明是否更新要包含文件的metadata（比如 modification time and last access time）。true可能会调用底层的write到操作系统，（如果操作系统在维护metadata，比如  last access time ）,即使channel是以只读打开的。       抛异常  ClosedChannelException  IOException |
| long position()                          | 返回file channel维护的以0为基准的当前file position。抛异常  ClosedChannelException  IOException |
| FileChannel position(long newPosition)   | 设置这个file channel的当前file position到newPosition。参数是从文件开头开始的字节计数。不能为负。但是可以超出当前文件size，如果超过了，read会返回end of file。写操作会成功，会在当前文件末尾和new position之间使用需求数量的（未定义）字节值填充字节。offset为负抛出` java.lang.IllegalArgumentException `。  ClosedChannelException ， IOException |
| int read(ByteBuffer buffer)              | 从file channel读取字节到给定buffer。读取字节的最大数量是调用方法时buffer剩余的字节数量。字节会从buffer的当前position开始复制到buffer。调用会阻塞如果其他线程也尝试从这个channel读。读完后，buffer的position会指到读取字节的末尾。buffer的limit没有变。返回实际读取的字节数，抛出和` ReadableByteChannel `一样的异常。 |
| int read(ByteBuffer dst, long position)  | 和上面一样，但是从文件的position位置读，position是负抛出` IllegalArgumentException `。 |
| long size()                              | 返回file channel下的文件的（in bytes）size。 ClosedChannelException ，  IOException |
| FileChannel truncate(long size)          | Truncate 这个file channel下的文件到size。任何超出  size 的字节都被从文件移除。当没有字节超出给定size，文件内容没被改变。当 current file position 超过了size，它被设置为size。 |
| int write(ByteBuffer buffer)             | 从给定buffer写一个序列的字节到file channel。字节从channel的  current file position 开始写，除非是append模式，这个模式position会先提到文件末尾。file会增长（必要时）来容纳写入的字节，然后文件position随着被实际写入的字节更新。另外的表现得和` WritableByteChannel `的方法一样。方法返回实际写的字节数，抛出和` WritableByteChannel `一样的异常。 |
| int write(ByteBuffer src, long position) | 等同上个方法，从position开始写。                             |

 force(boolean)  方法确保所有更改写入本地文件系统上的文件。这个能力对例如事务的critical任务很关键，你要维护数据完整性和可靠恢复。然而，对远程文件系统没有保证。

传true到 force(boolean) 导致metadata也被同步到磁盘。因为metadata对文件恢复通常不关键，你可以传false，它是个额外的I/O操作。

`FileChannel `对象支持current file position的概念，决定了下个要读或写入的data item的位置。 position() 返回当前position，position(long newPosition) 设置当前position到newPosition。

read() and write() 方法有两种形式。相对的形式，不带position参数，确保在方法调用后当前文件的position的更新。绝对的形式带有position参数，不会更新position。绝对形式的读写更效率因为不需要更新channel的state。

如果你尝试执行一个绝对的读文件末尾，参数是size()的返回值，会返回-1表明到达文件末尾。如果你尝试执行一个绝对的读文件末尾，参数是size()的返回值导致文件增长来容纳写入的字节。在先前文件末尾和第一个新写入字节之间的字节的值是系统特定的，可能形成一个hole。

一个*hole*发生在文件，当分配给文件的磁盘空间数量比文件size要小的时候。现代操作系统通常只对写入文件的数据分配空间。当数据被写入不连续的区域，holes出现。当文件被读，holes通常表现为zero-filled（补0）但是不占磁盘空间。

truncate(long size) 方法对减小文件size很有用。截断所有超出size的数据。当size比文件size大或者相等，文件不变。

***Listing 7-3. Demonstrating a File Channel*** 

```java
public class ChannelDemo2 {
    public static void main(String[] args) throws IOException {
        RandomAccessFile raf = new RandomAccessFile("temp", "rw");
        FileChannel fc = raf.getChannel();
        long pos;
        System.out.println(pos = fc.position());
        System.out.println(fc.size());

        String msg = "er34 we3e 34awe";
        ByteBuffer buffer = ByteBuffer.allocateDirect(msg.length()*2);
        buffer.asCharBuffer().put(msg);

        fc.write(buffer);
        fc.force(true);
        System.out.println("————————————————————————————————————");
        System.out.println(fc.position());
        System.out.println(fc.size());
        buffer.clear();
        fc.position(pos);
        fc.read(buffer);
        buffer.flip();
        while (buffer.hasRemaining()) {
            System.out.print(buffer.getChar());
        }
    }
}
```

先创建了randomly-accessible File temp，然后从file得到一个file channel，输出position和size，都是0。

然后分配一个direct buffer，存放messgae，把buffer作为character buffer，调用put()保存buffer里的message，随后输出到文件。

调用force(true)，托付底层操作系统把数据存到存储设备。

然后输出当前position和size，然后clear buffer，重置file position到message被写入前的位置（0），读取先前写的内容到buffer。然后flip buffer输出内容。

```
输出
Position = 0
size: 0
position: 46
size: 46
This is a test message.
再次运行
Position = 0
size: 46
position: 46
size: 46
This is a test message.
```

###### Locking Files 

lock全部或部分文件的能力是重要的，但这个特性Java 1.4才有。这个能力让JVM进程阻止其他进程访问整个或者部分文件直到它结束了整个或者部分文件的操作。

虽然可以锁整个文件，但通常希望锁更小的一部分。例如，数据库管理系统可以锁更新中的单个行而不是锁整个表，因此读取请求可以被授予，从而提高吞吐量 。

和files有联系的locks也叫 *file locks* 。每个文件锁从文件中一个确切的字节位置开始，和一个指定的从这个位置开始的length（byte）。它们一起定义了由锁控制的区域。 文件锁让进程协调对文件中各个区域的访问。

有2种文件锁：exclusive （排它锁）和shared （共享锁）。一个*exclusive lock* 给单个writer进程访问一个文件区域；它阻止其他文件锁同时适用与该区域。一个*shared lock* 给出多个reader进程的其中之一访问相同的文件区域；它不阻止其他共享锁但是阻止排它锁同时适用此区域。

Exclusive and shared locks 普遍使用于一个文件主要是读偶尔更新的场景。一个需要读取文件的进程，需要一把shared lock，锁到整个文件或者期望的子区域。第二个需要读的进程也需要一把共享锁，锁到期望区域。两个进程都能读文件而互不干扰。

假设第三个进程想执行更新。要这么做，需要一个排它锁。这个进程会阻塞住，直到所有的和它的区域重叠的排他/共享锁被释放。一旦更新进程抢到排它锁（Once the exclusive lock was granted to the updater process ），任何请求共享锁的reader进程都会阻塞住，直到排它锁被释放。更新进程更新了文件，读进程也不会观测到不一致的数据。

对 file locking 还有几件事要注意 ：

- 当操作系统不支持共享锁，一个共享锁的请求默默地提升到了排它锁。虽然正确性可以肯定，性能可能会受到影响。
- Locks是适用于per-file的基础上的。不适用于基于per-thread或者 per-channel。JVM上的2个线程，通过不同的channel，一个到相同区域的排它锁都可以授权访问。然而，如果是不同JVM的线程，第二个线程会阻塞。Locks最终由操作系统的文件系统进行公断，而且几乎总是在进程层面。不在线程层面公断。锁和file关联的，不和文件句柄或channel关联。

FileChannel声明4个方法得到共享/排它锁：

- FileLock lock(): 得到这个file channel底下文件的排它锁。这个方便方法等同于执行`fileChannel.lock(0L, Long. MAX_VALUE, false); `；                                                                                                                                                    

  返回一个`java.nio.channels.FileLock `对象代表被锁区域。可抛出ClosedChannelException，NonWritableChannelException ，java.nio.channels.OverlappingFileLockException当任意一个和请求区域重叠锁已经被JVM持有，或另一个在这个方法被阻塞了并尝试锁住同文件的重叠区域；java.nio.channels.FileLockInterruptionException 当等待获取锁时被打断；AsynchronousCloseException  当等待获取锁时channel被关闭；IOException 等。

- FileLock lock(long position, long size, boolean shared): 在这个channel的文件的区域加锁。

- FileLock tryLock(): 尝试不阻塞地获得排他锁。等同于 `fileChannel.tryLock(0L, Long.MAX_VALUE, false)`; 返回一个 FileLock ，代表被锁的区域或者null，当锁和其他操作系统进程重叠的时候。抛出 ClosedChannelException ，OverlappingFileLockException 当重叠区域的锁被JVM持有，或另一个在这个方法被阻塞了并尝试锁住同文件的重叠区域。 IOException

- FileLock tryLock(long position, long size, boolean shared): 

lock()当要锁的区域已经被锁会block（除非都是共享锁）。对比之下，tryLock() 会迅速返回一个null值（当要和其他操作系统的进程的独占锁重叠）

都返回 FileLock 实例，封装了一个file的被锁的区域。 FileLock’s methods ：

- FileChannel channel(): 当得到锁返回file channel，当file channel没得到锁返回null
- void close(): 调用release() 方法释放锁
- boolean isShared():  是不是共享锁
- boolean isValid(): 是valid lock返回true，否则false。锁是valid的直到它被释放或者关联的file channel被关闭，无论哪个在前
- boolean overlaps(long position, long size): 返回是不是和被锁区域重叠
- long position(): 返回被锁区域的第一个字节的位置。返回值可能比文件size要大。
- void release():  释放锁。如果锁是valid的，调用这个方法释放锁，把对象变成invalid。如果锁是，invalid ，调用这个方法无效。
- long size(): 返回文件锁的的length（字节）
- String toString(): 返回string描述range，type和validity 。

FileLock实例和一个FileChannel 实例关联，但是FileLock实例代表的file lock，和underlying file 关联而不是和file channel关联。不小心的话，当你用完后没释放锁可能会遇到冲突（甚至可能死锁）。要避免这些问题，你要采用一种模式比如下面的，确保lock总是被释放：

```java
FileLock lock = fileChannel.lock();
try
{
 // interact with the file channel
}
catch (IOException ioe)
{
 // handle the exception
}
finally
{
 lock.release();
}

```

例子：

***Listing 7-4. Demonstrating File Locking*** 

```java
public class ChannelDemo3 {
    final static int MAXQUERIES = 150000;
    final static int MAXUPDATES = 150000;

    final static int RECLEN = 16;
    static ByteBuffer buffer = ByteBuffer.allocate(RECLEN);
    static IntBuffer intBuffer = buffer.asIntBuffer();

    static int counter = 1;

    public static void main(String[] args) throws IOException {
        boolean writer = false;
        if (args.length != 0) {
            writer = true;

            RandomAccessFile raf = new RandomAccessFile("E:\\SpringSourceCode\\src\\main\\resources\\image.txt", (writer) ? "rw" : "r");
            FileChannel fc = raf.getChannel();
            if (writer) {
                update(fc);
            } else {
                query(fc);
            }
        }
    }

    private static void query(FileChannel fc) throws IOException {
        for (int i = 0; i < MAXQUERIES; i++) {
            System.out.println("acquiring shared lock");
            FileLock lock = fc.lock(0, RECLEN, true);

            try {
                buffer.clear();
                fc.read(buffer, 0);
                int a = intBuffer.get(0);
                int b = intBuffer.get(1);
                int c = intBuffer.get(2);
                int d = intBuffer.get(3);
                System.out.println("Reading" + a + ", " + b + ", " + c + ", " + d);
                if (a * 2 != b || a * 3 != c || a * 4 != d) {
                    System.out.println("error");
                    return;
                }
            } finally {
                lock.release();
            }
        }
    }

    private static void update(FileChannel fc) throws IOException {
        for (int i = 0; i < MAXUPDATES; i++) {
            System.out.println("acquiring exclusive lock");
            FileLock lock = fc.lock(0, RECLEN, false);
            try {
                intBuffer.clear();
                int a = counter;
                int b = counter * 2;
                int c = counter * 3;
                int d = counter * 4;

                System.out.println("Writing" + a + ", " + b + ", " + c + ", " + d);
                intBuffer.put(a);
                intBuffer.put(b);
                intBuffer.put(c);
                intBuffer.put(d);
                counter++;
                buffer.clear();
                fc.write(buffer, 0);
            } finally {
                lock.release();
            }
        }
    }
}
```

7-4描述了应用要么更新要么查询文件。因为file lock应用于进程级别而不是线程级别。你需要启动2个application。一个writer，一个reader。

ChannelDemo3 类先定义常数和变量。然后分配了一个byte buffer，得到int-based view buffer 。

main() 决定了是writer还是reader。

**注意：** ChannelDemo reader 进程越多，ChannelDemo writer 进程运行越慢。

###### Mapping Files into Memory 

FileChannel声明了map() 方法可以在打开的文件的一个region和一个包装了这个region的java.nio.MappedByteBuffer实例之间创建一个 virtual memory mapping 。mapping机制提供了一个效率的访问文件的方式，因为没有执行I/O调用的时间消耗。

**注意**：*Virtual memory* 是一种memory，virtual addresses（也叫artificial addresses ）代替（physical (RAM memory) addresses 。Check out Wikipedia’s “Virtual Memory” topic (http://en.wikipedia.org/wiki/ Virtual_memory) to learn more about virtual memory. 

 map() 方法有如下签名：

MappedByteBuffer map(FileChannel.MapMode mode, long position, long size) 

mode参数定义映射类型，是 FileChannel.MapMode 的枚举类型：

- READ_ONLY:  任何更改的尝试抛出java.nio.ReadOnlyBufferException 
- READ_WRITE:  对resulting buffer 的更改最终传播到文件；Changes 可能不对映射了同一文件的程序可见。
- PRIVATE:  对resulting buffer 的更改不会传播到文件，对其他映射同一文件的程序不可见。而是，更改会造成buffer修改部分的 private copies 会被创建。当buffer被垃圾回收，更改就丢失。

指定的映射模式受到调用着FileChannel对象访问权限的限制。例如，如果 file channel是只读打开的，请求READ_WRITE mode， map()抛出NonWritableChannelException 。类似地 NonReadableChannelException 。

**技巧**：调用MappedByteBuffer’s isReadOnly() 方法看你能不能更改 mapped file 。

position and size 参数定义开始位置和映射区域范围。不是负的。不能超过Integer.MAX_VALUE 。

指定的range不能超过文件size，因为文件会被变得更大来接收range。例如，如果你size为Integer.MAX_VALUE ，文件会超过2G。同时，对于只读映射， map() 可能抛出IOException 。

返回的MappedByteBuffer 对象表现得像一个memory-mapped buffer。但是内容保存在文件。在这个对象调用get() ，得到当前文件内容，甚至这些内如被外部程序更改的时候。类似地，如果你有写权限， 调用 put()  更新文件，更改对外部程序可用。

**注意**：因为mapped byte buffers 是direct byte buffers ，分配给它们的内存空间存在于JVM堆之外。

```java
MappedByteBuffer buffer = fileChannel.map(FileChannel.MapMode.READ_ONLY, 50, 100);
```

这个例子映射一个子范围，从50到149。下面是全部文件

```java
MappedByteBuffer buffer =
fileChannel.map(FileChannel.MapMode.READ_ONLY, 0, fileChannel.size());
```

没有unmap() 方法。一旦映射建立了，它会在垃圾收集前一直存在（或者程序关闭）。因为a mapped byte buffer 没有连接到建立它的 file channel ，当file channel关闭mapping 不会销毁。

MappedByteBuffer 方法继承于java.nio.ByteBuffer 。有以下方法：

- MappedByteBuffer load():  尝试加载全部映射文件到内存。这会使大文件访问更快因为 virtual memory manager  不需要在文件的部分被通过mapped buffer请求的时候加载文件的部分到内存（by reading from/writing to their locations ）。load() 做了最大努力，可能不成功因为额外程序会使 virtual memory manager 移除文件部分的内容来为加载内容到物理空间的请求留出空间。另外，load() 是expensive time-wise ，他会使virtual memory manager 执行I/O；完成这个方法要花费时间
- boolean isLoaded(): 当全部映射文件内容被加载到内存返回true ；否则false。如果返回true，你可以用很少或没有的I/O操作来访问文件。如果返回false，buffer access 仍然可能很快，mapped content完全居于内存。考虑isLoaded() 为暗示映射字节缓冲区的状态。
- MappedByteBuffer force(): 让对mapped byte buffer 的更改写入储存。当使用mapped byte buffers 的时候，你应该用这个方法而不是file channel’s force() 方法，因为channel可能没意识到通过mapped byte buffer 做出的更改。在 READ_ONLY and PRIVATE mappings 这个方法没用。