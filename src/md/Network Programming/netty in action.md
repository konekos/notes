# 第一部分 Netty 的概念及体系结构

## 第 1 章 Netty——异步和事件驱动

### 1.1 Java 网络编程

最早期的 Java API（java.net）

![1538041077927](E:\studydyup\notes\src\pic\%5CUsers%5CJudy%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1538041077927.png)

代码清单 1-1 实现了 Socket API 的基本模式之一。以下是最重要的几点。

- ServerSocket 上的 accept()方法将会一直阻塞到一个连接建立 ，随后返回一个
  新的 Socket 用于客户端和服务器之间的通信。该 ServerSocket 将继续监听传入的
  连接。

- BufferedReader 和 PrintWriter 都衍生自 Socket 的输入输出流 。前者从一个
  字符输入流中读取文本，后者打印对象的格式化的表示到文本输出流。

- readLine()方法将会阻塞，直到在 处一个由换行符或者回车符结尾的字符串被
  读取

- 客户端的请求已经被处理 。



  这段代码片段将只能同时处理一个连接，要管理多个并发客户端，需要为每个新的客户端
  Socket 创建一个新的 Thread。

​	让我们考虑一下这种方案的影响。第一，在任何时候都可能有大量的线程处于休眠状态，只是等待输入或者输出数据就绪，这可能算是一种资源浪费。第二，需要为每个线程的调用栈都分配内存，其默认值大小区间为 64 KB 到 1 MB，具体取决于操作系统。第三，即使 Java 虚拟机（JVM）在物理上可以支持非常大数量的线程，但是远在到达该极限之前，上下文切换所带来的开销就会带来麻烦，例如，在达到 10 000 个连接的时候。

​	虽然这种并发方案对于支撑中小数量的客户端来说还算可以接受，但是为了支撑 100 000 或
者更多的并发连接所需要的资源使得它很不理想。幸运的是，还有一种方案。

#### 1.1.1 Java NIO

​	Java 对于非阻塞 I/O 的支持是在 2002 年引入的，位于 JDK 1.4 的 java.nio 包中。

>新的还是非阻塞的
>NIO 最开始是新的输入/输出（New Input/Output）的英文缩写，但是，该Java API 已经出现足够长的时间
>了，不再是“新的”了，因此，如今大多数的用户认为NIO 代表非阻塞 I/O（Non-blocking I/O），而阻塞I/O（blocking
>I/O）是旧的输入/输出（old input/output，OIO）。你也可能遇到它被称为普通I/O（plain I/O）的时候。

#### 1.1.2 选择器

​	class java.nio.channels.Selector 是Java 的非阻塞 I/O 实现的关键。

​	它使用了事件通知 API以确定在一组非阻塞套接字中有哪些已经就绪能够进行 I/O 相关的操作。因为可以在任何的时间检查任意的读操作或者写操作的完成状态，所以如图 1-2 所示，一个单一的线程便可以处理多个并发的连接。

总体来看，与阻塞 I/O 模型相比，这种模型提供了更好的资源管理：

- 使用较少的线程便可以处理许多连接，因此也减少了内存管理和上下文切换所带来开销；
- 当没有 I/O 操作需要处理的时候，线程也可以被用于其他任务

尽管已经有许多直接使用 Java NIO API 的应用程序被构建了，但是要做到如此正确和安全并不容易。特别是，在高负载下可靠和高效地处理和调度 I/O 操作是一项繁琐而且容易出错的任务，最好留给高性能的网络编程专家——Netty。

### 1.2 Netty 简介

​	在网络编程领域，Netty是Java的卓越框架。它驾驭了Java高级API的能力，并将其隐藏在一个易于使用的API之后。Netty使你可以专注于自己真正感兴趣的——你的应用程序的独一无二的价值。

​	在我们开始首次深入地了解 Netty 之前，请仔细审视表 1-1 中所总结的关键特性。有些是技术性的，而其他的更多的则是关于架构或设计哲学的。在本书的学习过程中，我们将不止一次地重新审视它们。

***表 1-1 Netty 的特性总结***

| 分 类    | Netty 的特性                                                 |
| -------- | ------------------------------------------------------------ |
| 设计     | 统一的 API，支持多种传输类型，阻塞的和非阻塞的<br/>简单而强大的线程模型
真正的无连接数据报套接字支持
链接逻辑组件以支持复用 |
| 易于使用 | 详实的Javadoc和大量的示例集<br/>不需要超过JDK 1.6+③的依赖。（一些可选的特性可能需要Java 1.7+和/或额外的依赖） |
| 性能     | 拥有比 Java 的核心 API 更高的吞吐量以及更低的延迟<br/>得益于池化和复用，拥有更低的资源消耗
最少的内存复制 |
| 健壮性   | 不会因为慢速、快速或者超载的连接而导致 OutOfMemoryError<br/>消除在高速网络中 NIO 应用程序常见的不公平读/写比率 |
| 安全性   | 完整的 SSL/TLS 以及 StartTLS 支持<br/>可用于受限环境下，如 Applet 和 OSGI |
| 社区驱动 | 发布快速而且频繁                                             |

#### 1.2.1 谁在使用 Netty

很多..略

#### 1.2.2 异步和事件驱动

​	因为我们要大量地使用“异步”这个词，所以现在是一个澄清上下文的好时机。异步（也就是非同步）事件肯定大家都熟悉。考虑一下电子邮件：你可能会也可能不会收到你已经发出去的电子邮件对应的回复，或者你也可能会在正在发送一封电子邮件的时候收到一个意外的消息。异步事件也可以具有某种有序的关系。通常，你只有在已经问了一个问题之后才会得到一个和它对应的答案，而在你等待它的同时你也可以做点别的事情。

​	在日常的生活中，异步自然而然地就发生了，所以你可能没有对它考虑过多少。但是让一个
计算机程序以相同的方式工作就会产生一些非常特殊的问题。本质上，一个既是异步的又是事件
驱动的系统会表现出一种特殊的、对我们来说极具价值的行为：它可以以任意的顺序响应在任意
的时间点产生的事件。

​	这种能力对于实现最高级别的可伸缩性至关重要，定义为：“一种系统、网络或者进程在
需要处理的工作不断增长时，可以通过某种可行的方式或者扩大它的处理能力来适应这种增长
的能力。

​	异步和可伸缩性之间的联系又是什么呢？

- 非阻塞网络调用使得我们可以不必等待一个操作的完成。完全异步的 I/O 正是基于这个
  特性构建的，并且更进一步：异步方法会立即返回，并且在它完成时，会直接或者在稍
  后的某个时间点通知用户。
- Selector使得我们能够通过较少的线程便可监视许多连接上的事件。

将这些元素结合在一起，与使用阻塞 I/O 来处理大量事件相比，使用非阻塞 I/O 来处理更快速、更经济。从网络编程的角度来看，这是构建我们理想系统的关键，而且你会看到，这也是Netty 的设计底蕴的关键。

​	在 1.3 节中，我们将首先看一看 Netty 的核心组件。现在，只需要将它们看作是域对象，而不是具体的 Java 类。随着时间的推移，我们将看到它们是如何协作，来为在网络上发生的事件提供通知，并使得它们可以被处理的。

### 1.3 Netty 的核心组件

​	在本节中我将要讨论 Netty 的主要构件块：

- Channel；
- 回调；
- Future；
- 事件和 ChannelHandler。

​	这些构建块代表了不同类型的构造：资源、逻辑以及通知。你的应用程序将使用它们来访问网络以及流经网络的数据。

#### 1.3.1 Channel

​	Channel 是 Java NIO 的一个基本构造。

​	**它代表一个到实体（如一个硬件设备、一个文件、一个网络套接字或者一个能够执行一个或者多个不同的I/O操作的程序组件）的开放连接，如读操作和写操作 ①*	。

​	目前，可以把 Channel 看作是传入（入站）或者传出（出站）数据的载体。因此，它可以
被打开或者被关闭，连接或者断开连接。	

#### 1.3.2 回调

​	一个回调其实就是一个方法，一个指向已经被提供给另外一个方法的方法的引用。这使得后者可以在适当的时候调用前者。回调在广泛的编程场景中都有应用，而且也是在操作完成后通知相关方最常见的方式之一。

​	Netty 在内部使用了回调来处理事件；当一个回调被触发时，相关的事件可以被一个interfaceChannelHandler的实现处理。代码清单 1-2 展示了一个例子：当一个新的连接已经被建立时，ChannelHandler 的 channelActive()回调方法将会被调用，并将打印出一条信息。	

***代码清单 1-2 被回调触发的 ChannelHandler***

```java
public class ConnectHandler extends ChannelInboundHandlerAdapter {
        @Override
        public void channelActive(ChannelHandlerContext ctx)
                throws Exception {
            System.out.println(
                    "Client " + ctx.channel().remoteAddress() + " connected");
        }
    }
```

#### 1.3.3 Future

​	Future 提供了另一种在操作完成时通知应用程序的方式。这个对象可以看作是一个异步操作的结果的占位符；它将在未来的某个时刻完成，并提供对其结果的访问。

​	JDK 预置了 interface java.util.concurrent.Future，但是其所提供的实现，只允许手动检查对应的操作是否已经完成，或者一直阻塞直到它完成。这是非常繁琐的，所以 Netty提供了它自己的实现——ChannelFuture，用于在执行异步操作的时候使用。

​	ChannelFuture提供了几种额外的方法，这些方法使得我们能够注册一个或者多个ChannelFutureListener实例。监听器的回调方法operationComplete()，将会在对应的操作完成时被调用 。然后监听器可以判断该操作是成功地完成了还是出错了。如果是后者，我们可以检索产生的Throwable。简而 言之 ，由ChannelFutureListener提供的通知机制消除了手动检查对应的操作是否完成的必要。

​	每个 Netty 的出站 I/O 操作都将返回一个 ChannelFuture；也就是说，它们都不会阻塞。
正如我们前面所提到过的一样，Netty 完全是异步和事件驱动的。	

​	代码清单 1-3 展示了一个 ChannelFuture 作为一个 I/O 操作的一部分返回的例子。这里，connect()方法将会直接返回，而不会阻塞，该调用将会在后台完成。这究竟什么时候会发生则取决于若干的因素，但这个关注点已经从代码中抽象出来了。因为线程不用阻塞以等待对应的操作完成，所以它可以同时做其他的工作，从而更加有效地利用资源。	

***代码清单 1-3 异步地建立连接***

```java
Channel channel = ...;
    //does not block
    ChannelFuture future = channel.connect(
            new InetSocketAddress("192.168.0.1", 25));
```

​	代码清单 1-4 显示了如何利用 ChannelFutureListener。首先，要连接到远程节点上。然后，要注册一个新的 ChannelFutureListener 到对 connect()方法的调用所返回的 ChannelFuture 上。当该监听器被通知连接已经建立的时候，要检查对应的状态 。如果该操作是成功的，那么将数据写到该 Channel。否则，要从 ChannelFuture 中检索对应的 Throwable。

***代码清单 1-4 回调实战***

```java
Channel channel = ...;
    // Does not block
    ChannelFuture future = channel.connect(//异步地连接到远程节点
            new InetSocketAddress("192.168.0.1", 25));
future.addListener(new ChannelFutureListener() {//注册一ChannelFutureListener，以便在操作完成时获得通知
        @Override
        public void operationComplete(ChannelFuture future) {
            if (future.isSuccess()){//如果操作是成功的，则创建一个 ByteBuf 以持有数据
                ByteBuf buffer = Unpooled.copiedBuffer(
                        "Hello",Charset.defaultCharset());
                ChannelFuture wf = future.channel()
                        .writeAndFlush(buffer);//将数据异步地发送到远程节点。返回一个 ChannelFuture
....
            } else {//如果发生错误，则访问描述原因的 Throwable
                Throwable cause = future.cause();
                cause.printStackTrace();
            }
        }
    });
```

​	需要注意的是，对错误的处理完全取决于你、目标，当然也包括目前任何对于特定类型的错误加以的限制。例如，如果连接失败，你可以尝试重新连接或者建立一个到另一个远程节点的连接。
​	如果你把 ChannelFutureListener 看作是回调的一个更加精细的版本，那么你是对的。事实上，回调和 Future 是相互补充的机制；它们相互结合，构成了 Netty 本身的关键构件块之一。

#### 1.3.4 事件和 ChannelHandler

