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

![1532340528079](E:\studydyup\notes\src\pic\1532340528079.png)



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

JDK 1.0引入标准I/O支持通过添加in, out, and err objects of type InputStream and PrintStream到`java.lang.System `类。你指定调用这些对象的方法