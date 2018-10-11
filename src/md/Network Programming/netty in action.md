---
typora-copy-images-to: ..\..\pic
---

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

​	Netty 使用不同的事件来通知我们状态的改变或者是操作的状态。这使得我们能够基于已经发生的事件来触发适当的动作。这些动作可能是：

- 记录日志；
- 数据转换；
- 流控制；
- 应用程序逻辑。

​	Netty 是一个网络编程框架，所以事件是按照它们与入站或出站数据流的相关性进行分类的。可能由入站数据或者相关的状态更改而触发的事件包括：

- 连接已被激活或者连接失活；

- 数据读取；

- 用户事件；

- 错误事件。

  出站事件是未来将会触发的某个动作的操作结果，这些动作包括：

- 打开或者关闭到远程节点的连接；
- 将数据写到或者冲刷到套接字。

​       每个事件都可以被分发给 ChannelHandler 类中的某个用户实现的方法。这是一个很好的将事件驱动范式直接转换为应用程序构件块的例子。图 1-3 展示了一个事件是如何被一个这样的ChannelHandler 链处理的。

![1538102930925](E:\studydyup\notes\src\pic\%5CUsers%5CJudy%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1538102930925.png)

​	Netty 的 ChannelHandler 为处理器提供了基本的抽象，如图 1-3 所示的那些。我们会在适当的时候对 ChannelHandler 进行更多的说明，但是目前你可以认为每个 ChannelHandler的实例都类似于一种为了响应特定事件而被执行的回调。

​	Netty 提供了大量预定义的可以开箱即用的 ChannelHandler 实现，包括用于各种协议（如 HTTP 和 SSL/TLS）的 ChannelHandler。在内部，ChannelHandler 自己也使用了事件和 Future，使得它们也成为了你的应用程序将使用的相同抽象的消费者。	

#### 1.3.5 把它们放在一起

​	在本章中，我们介绍了 Netty 实现高性能网络编程的方式，以及它的实现中的一些主要的组
件。让我们大体回顾一下我们讨论过的内容吧。

##### 1．Future、回调和 ChannelHandler

​	Netty的异步编程模型是建立在Future和回调的概念之上的，而将事件派发到ChannelHandler的方法则发生在更深的层次上结合在一起，这些元素就提供了一个处理环境，使你的应用程序逻辑可以独立于任何网络操作相关的顾虑而独立地演变。这也是 Netty 的设计方式的一个关键目标。

​	拦截操作以及高速地转换入站数据和出站数据，都只需要你提供回调或者利用操作所返回的Future。这使得链接操作变得既简单又高效，并且促进了可重用的通用代码的编写。	

##### 2．选择器、事件和 EventLoop

​	Netty 通过触发事件将 Selector 从应用程序中抽象出来，消除了所有本来将需要手动编写的派发代码。在内部，将会为每个 Channel 分配一个 EventLoop，用以处理所有事件，包括：	

- 注册感兴趣的事件；
- 将事件派发给 ChannelHandler；
- 安排进一步的动作。

​       EventLoop 本身只由一个线程驱动，其处理了一个 Channel 的所有 I/O 事件，并且在该EventLoop 的整个生命周期内都不会改变。这个简单而强大的设计消除了你可能有的在ChannelHandler 实现中需要进行同步的任何顾虑，因此，你可以专注于提供正确的逻辑，用来在有感兴趣的数据要处理的时候执行。如同我们在详细探讨 Netty 的线程模型时将会看到的，该 API 是简单而紧凑的。

### 1.4 小结

​	...

​	在下一章中，我们将要深入地探讨 Netty 的 API 以及编程模型的基础知识，而你则将编写你的第一款客户端和服务器应用程序。

## 第 2 章 你的第一款 Netty 应用程序

### 2.1 设置开发环境

略

### 2.2 Netty 客户端/服务器概览

​	图 2-1 从高层次上展示了一个你将要编写的 Echo 客户端和服务器应用程序。虽然你的主要关注点可能是编写基于 Web 的用于被浏览器访问的应用程序，但是通过同时实现客户端和服务器，你一定能更加全面地理解 Netty 的 API。

![1538103449758](E:\studydyup\notes\src\pic\%5CUsers%5CJudy%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1538103449758.png)

​	虽然我们已经谈及到了客户端，但是该图展示的是多个客户端同时连接到一台服务器。所能够支持的客户端数量，在理论上，仅受限于系统的可用资源（以及所使用的 JDK 版本可能会施加的限制）。

​	Echo 客户端和服务器之间的交互是非常简单的；在客户端建立一个连接之后，它会向服务器发送一个或多个消息，反过来，服务器又会将每个消息回送给客户端。虽然它本身看起来好像用处不大，但它充分地体现了客户端/服务器系统中典型的请求-响应交互模式。

​	我们将从考察服务器端代码开始这个项目。

### 2.3 编写 Echo 服务器

所有的 Netty 服务器都需要以下两部分。

- 至少一个 ChannelHandler—该组件实现了服务器对从客户端接收的数据的处理，即它的业务逻辑。
- 引导—这是配置服务器的启动代码。至少，它会将服务器绑定到它要监听连接请求的端口上。

在本小节的剩下部分，我们将描述 Echo 服务器的业务逻辑以及引导代码。

#### 2.3.1 ChannelHandler 和业务逻辑

​	在第 1 章中，我们介绍了 Future 和回调，并且阐述了它们在事件驱动设计中的应用。我们还讨论了 ChannelHandler，它是一个接口族的父接口，它的实现负责接收并响应事件通知。在 Netty 应用程序中，所有的数据处理逻辑都包含在这些核心抽象的实现中。

​	因为你的 Echo 服务器会响应传入的消息，所以它需要实现 ChannelInboundHandler 接口，用来定义响应入站事件的方法。这个简单的应用程序只需要用到少量的这些方法，所以继承ChannelInboundHandlerAdapter类也就足够了，它提供了 ChannelInboundHandler 的默认实现。	

我们感兴趣的方法是：

- channelRead()—对于每个传入的消息都要调用；
- channelReadComplete()—通知ChannelInboundHandler最后一次对channelRead()的调用是当前批量读取中的最后一条消息；
- exceptionCaught()—在读取操作期间，有异常抛出时会调用。

该 Echo 服务器的 ChannelHandler 实现是 EchoServerHandler，如代码清单 2-1 所示。

***代码清单 2-1 EchoServerHandler***

```java
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelHandler;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.util.CharsetUtil;

/**
 * @author @Jasu
 * @date 2018-09-28 11:07
 */
@ChannelHandler.Sharable//标示一个ChannelHandler可以被多个Channel安全地共享
public class EchoServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf in = (ByteBuf) msg;
        System.out.println("Server received: " + in.toString(CharsetUtil.UTF_8));//将消息记录到控制台
        ctx.write(in);//将接收到的消息写给发送者，而不冲刷出站消息
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) {
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER)//将未决消息冲刷到远程节点，并且关闭该Channel
                .addListener(ChannelFutureListener.CLOSE);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx,
                                Throwable cause) {
        cause.printStackTrace();//打印异常栈追踪
        ctx.close();//关闭该channel
    }
}
```

​	ChannelInboundHandlerAdapter 有一个直观的 API，并且它的每个方法都可以被重写以挂钩到事件生命周期的恰当点上。因为需要处理所有接收到的数据，所以你重写了 channelRead()方法。在这个服务器应用程序中，你将数据简单地回送给了远程节点。

​	重写 exceptionCaught()方法允许你对 Throwable 的任何子类型做出反应，在这里你记录了异常并关闭了连接。虽然一个更加完善的应用程序也许会尝试从异常中恢复，但在这个场景下，只是通过简单地关闭连接来通知远程节点发生了错误。

> **如果不捕获异常，会发生什么呢**
>
> 每个 Channel 都拥有一个与之相关联的 ChannelPipeline，其持有一个 ChannelHandler 的实例链。在默认的情况下，ChannelHandler 会把对它的方法的调用转发给链中的下一个ChannelHandler。因此，如果exceptionCaught()方法没有被该链中的某处实现，那么所接收的异常将会被传递到 ChannelPipeline 的尾端并被记录。为此，你的应用程序应该提供至少有一个实现了exceptionCaught()方法的 ChannelHandler。（6.4 节详细地讨论了异常处理）。

​	除了 ChannelInboundHandlerAdapter 之外，还有很多需要学习的 ChannelHandler 的子类型和实现，我们将在第 6 章和第 7 章中对它们进行详细的阐述。目前，请记住下面这些关键点：

- 针对不同类型的事件来调用 ChannelHandler；
- 应用程序通过实现或者扩展 ChannelHandler 来挂钩到事件的生命周期，并且提供自定义的应用程序逻辑；
- 在架构上，ChannelHandler 有助于保持业务逻辑与网络处理代码的分离。这简化了开发过程，因为代码必须不断地演化以响应不断变化的需求。

#### 2.3.2 引导服务器

​	在讨论过由 EchoServerHandler 实现的核心业务逻辑之后，我们现在可以探讨引导服务器本身的过程了，具体涉及以下内容：

- 绑定到服务器将在其上监听并接受传入连接请求的端口；
- 配置 Channel，以将有关的入站消息通知给 EchoServerHandler 实例。

> **传输**
>
> ​	在这一节中，你将遇到术语传输。在网络协议的标准多层视图中，传输层提供了端到端的或者主机到主机的通信服务。
>
> ​	因特网通信是建立在 TCP 传输之上的。除了一些由 Java NIO 实现提供的服务器端性能增强之外，NIO 传输大多数时候指的就是 TCP 传输。	
>
> ​	我们将在第 4 章对传输进行详细的讨论。

代码清单 2-2 展示了 EchoServer 类的完整代码。

***代码清单 2-2 EchoServer 类***

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;

import java.net.InetSocketAddress;

/**
 * @author @Jasu
 * @date 2018-09-28 11:51
 */
public class EchoServer {
    private final int port;

    public EchoServer(int port) {
        this.port = port;
    }

    public static void main(String[] args) throws Exception {
        if (args.length != 1) {
            System.err.println(
                    "Usage: " + EchoServer.class.getSimpleName() +
                            " <port>");
        }
        int port = Integer.parseInt(args[0]);
        new EchoServer(port).start();
    }

    private void start() throws Exception {
        final EchoServerHandler serverHandler = new EchoServerHandler();
        //1.创建EventLoopGroup
        EventLoopGroup group = new NioEventLoopGroup();

        try {
            //2.创建ServerBootstrap
            ServerBootstrap b = new ServerBootstrap();
            b.group(group)
                    //3.指定所使用的NIO传输Channel
                    .channel(NioServerSocketChannel.class)
                    //4.使用指定的端口设置套接字地址
                    .localAddress(new InetSocketAddress(port))
                    //5.添加一个EchoServerHandler到子Channel的ChannelPipeline
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            //EchoServerHandler 被标注为@Shareable，所以我们可以总是使用同样的实例
                            socketChannel.pipeline().addLast(serverHandler);
                        }
                    });
            //6.异步地绑定服务器；调用sync()方法阻塞等待直到绑定完成
            ChannelFuture f = b.bind().sync();
            //7.获取Channel的CloseFuture，并且阻塞当前线程直到它完成
            f.channel().closeFuture().sync();
        } finally {
            //8.关闭EventLoopGroup，释放所有的资源
            group.shutdownGracefully().sync();
        }
    }
}
```

​	在2处创建ServerBootstrap实例，使用NIO传输，指定 NioEventLoopGroup 来接受和处理新的连接，并把channel类型指定为NioServerSocketChannel。然后将本地地址设置为一个具有选定端口的 InetSocketAddress。服务器将绑定到这个地址以监听新的连接请求。

​	在5处，你使用了一个特殊的类——ChannelInitializer这是关键。当一个新的连接被接受时，一个新的子 Channel 将会被创建，而 ChannelInitializer 将会把一个你的EchoServerHandler 的实例添加到该 Channel 的 ChannelPipeline 中。正如我们之前所解释的，这个 ChannelHandler 将会收到有关入站消息的通知。

​	虽然 NIO 是可伸缩的，但是其适当的尤其是关于多线程处理的配置并不简单。Netty 的设计封装了大部分的复杂性，而且我们将在第 3 章中对相关的抽象（EventLoopGroup、SocketChannel和 ChannelInitializer）进行详细的讨论。	

​	然后在6绑定服务器，并等待绑定完成。（对 sync()方法的调用将导致当前 Thread阻塞，一直到绑定操作完成为止）在7处，该应用程序将会阻塞等待直到服务器的 Channel关闭（因为你在 Channel 的 CloseFuture 上调用了 sync()方法）。然后，你将可以关闭EventLoopGroup，并释放所有的资源，包括所有被创建的线程 。

​	这个示例使用了 NIO，因为得益于它的可扩展性和彻底的异步性，它是目前使用最广泛的传输。但是也可以使用一个不同的传输实现。如果你想要在自己的服务器中使用 OIO 传输，将需要指定 OioServerSocketChannel 和 OioEventLoopGroup。我们将在第 4 章中对传输进行更加详细的探讨。	

​	与此同时，让我们回顾一下你刚完成的服务器实现中的重要步骤。下面这些是服务器的主要代码组件：

- EchoServerHandler 实现了业务逻辑；

- main()方法引导了服务器；

  引导过程中所需要的步骤如下：

- 创建一个 ServerBootstrap 的实例以引导和绑定服务器；

- 创建并分配一个 NioEventLoopGroup 实例以进行事件的处理，如接受新连接以及读/写数据；

- 指定服务器绑定的本地的 InetSocketAddress；

- 使用一个 EchoServerHandler 的实例初始化每一个新的 Channel；

- 调用 ServerBootstrap.bind()方法以绑定服务器。

在这个时候，服务器已经初始化，并且已经就绪能被使用了。在下一节中，我们将探讨对应的客户端应用程序的代码。

### 2.4 编写 Echo 客户端

Echo 客户端将会：
（1）连接到服务器；
（2）发送一个或者多个消息；
（3）对于每个消息，等待并接收从服务器发回的相同的消息；
（4）关闭连接。
编写客户端所涉及的两个主要代码部分也是业务逻辑和引导，和你在服务器中看到的一样。

#### 2.4.1 通过 ChannelHandler 实现客户端逻辑

​	如同服务器，客户端将拥有一个用来处理数据的 ChannelInboundHandler。在这个场景下，你将扩展 SimpleChannelInboundHandler 类以处理所有必须的任务，如代码清单 2-3所示。这要求重写下面的方法：

- channelActive()——在到服务器的连接已经建立之后将被调用；
- channelRead0()——当从服务器接收到一条消息时被调用；
- exceptionCaught()——在处理过程中引发异常时被调用。

***代码清单 2-3 客户端的 ChannelHandler***

```java
import com.google.common.base.Charsets;
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandler;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

/**
 * @author @Jasu
 * @date 2018-09-28 15:21
 */
@ChannelHandler.Sharable//标记该类的实例可以被多个Channel共享
public class EchoClientHandler extends SimpleChannelInboundHandler<ByteBuf> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, ByteBuf msg) throws Exception {
        //记录已接收消息的转储
        System.out.println("Client received: " + msg.toString(Charsets.UTF_8));
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        //当被通知 Channel是活跃的时候，发送一条消息
        ctx.writeAndFlush(Unpooled.copiedBuffer("Netty rocks", Charsets.UTF_8));
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //在发生异常时，记录错误并关闭Channel
        cause.printStackTrace();
        ctx.close();
    }
}
```

​	channelActive()方法，其将在一个连接建立时被调用。这确保了数据将会被尽可能快地写入服务器，其在这个场景下是一个编码了字符串"Netty rocks!"的字节缓冲区。

​	重写了 channelRead0()方法。每当接收数据时，都会调用这个方法。需要注意的是，由服务器发送的消息可能会被分块接收。也就是说，如果服务器发送了 5 字节，那么不能保证这 5 字节会被一次性接收。即使是对于这么少量的数据，channelRead0()方法也可能会被调用两次，第一次使用一个持有 3 字节的 ByteBuf（Netty 的字节容器），第二次使用一个持有 2 字节的 ByteBuf。作为一个面向流的协议，TCP 保证了字节数组将会按照服务器发送它们的顺序被接收。

​	重写的第三个方法是 exceptionCaught()。在这个场景下，终止到服务器的连接。

> **SimpleChannelInboundHandler 与 ChannelInboundHandler**
>
> ​	你可能会想：为什么我们在客户端使用的是 SimpleChannelInboundHandler，而不是在EchoServerHandler中所使用的 ChannelInboundHandlerAdapter 呢？这和两个因素的相互作用有关：业务逻辑如何处理消息以及 Netty 如何管理资源。
>
> ​	在客户端，当 channelRead0()方法完成时，你已经有了传入消息，并且已经处理完它了。当该方法返回时，SimpleChannelInboundHandler 负责释放指向保存该消息的 ByteBuf 的内存引用。	
>
> ​	在 EchoServerHandler 中，你仍然需要将传入消息回送给发送者，而 write()操作是异步的，直到 channelRead()方法返回后可能仍然没有完成（如代码清单 2-1 所示）。为此，EchoServerHandler扩展了 ChannelInboundHandlerAdapter，其在这个时间点上不会释放消息。	
>
> ​	消息在 EchoServerHandler 的 channelReadComplete()方法中，当 writeAndFlush()方法被调用时被释放（见代码清单 2-1）。
> ​	第 5 章和第 6 章将对消息的资源管理进行详细的介绍。
>
> ​	

#### 2.4.2 引导客户端

​	客户端是使用主机和端口参数来连接远程地址，也就是这里的 Echo 服务器的地址，而不是绑定到一个一直被监听的端口。

***代码清单 2-4 客户端的主类***

```java
import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;

import java.net.InetSocketAddress;

/**
 * @author @Jasu
 * @date 2018-09-28 16:27
 */
public class EchoClient {
    private final String host;
    private final int port;

    public EchoClient(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public static void main(String[] args) throws Exception {
        if (args.length != 2) {
            System.err.println(
                    "Usage: " + EchoClient.class.getSimpleName() +
                            " <host> <port>");
            return;
        }
        String host = args[0];
        int port = Integer.parseInt(args[1]);
        new EchoClient(host, port).start();
    }

    private void start() throws Exception {
        EventLoopGroup group = new NioEventLoopGroup();

        try {
            //创建 Bootstrap
            Bootstrap b = new Bootstrap();
            //指定 EventLoopGroup 以处理客户端事件；需要适用于 NIO 的实现
            b.group(group)
                    //适用于 NIO 传输的Channel 类型
                    .channel(NioSocketChannel.class)
                    //设置服务器的InetSocketAddress
                    .remoteAddress(new InetSocketAddress(host, port))
                    //在创建Channel时，向 ChannelPipeline中添加一个 EchoClientHandler实例
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new EchoClientHandler());
                        }
                    });
            //连接到远程节点，阻塞等待直到连接完成
            ChannelFuture f = b.connect().sync();
            //阻塞，直到Channel 关闭
            f.channel().closeFuture().sync();
        } finally {
            //关闭线程池并且释放所有的资源
            group.shutdownGracefully().sync();
        }
    }
}
```

​	和之前一样，使用了 NIO 传输。注意，你可以在客户端和服务器上分别使用不同的传输。例如，在服务器端使用 NIO 传输，而在客户端使用 OIO 传输。在第 4 章，我们将探讨影响你选择适用于特定用例的特定传输的各种因素和场景。

​	让我们回顾一下这一节中所介绍的要点：

- 为初始化客户端，创建了一个 Bootstrap 实例；
- 为进行事件处理分配了一个 NioEventLoopGroup 实例，其中事件处理包括创建新的连接以及处理入站和出站数据；
- 为服务器连接创建了一个 InetSocketAddress 实例；
- 当连接被建立时，一个 EchoClientHandler 实例会被安装到（该 Channel 的）ChannelPipeline 中；
- 在一切都设置完成后，调用 Bootstrap.connect()方法连接到远程节点；完成了客户端，你便可以着手构建并测试该系统了。

### 2.5 构建和运行 Echo 服务器和客户端

略

### 2.6 小结

​	在本章中，你设置好了开发环境，并且构建和运行了你的第一款 Netty 客户端和服务器。虽然这只是一个简单的应用程序，但是它可以伸缩到支持数千个并发连接——每秒可以比普通的基于套接字的 Java 应用程序处理多得多的消息。
​	在接下来的几章中，你将看到更多关于 Netty 如何简化可伸缩性和并发性的例子。我们也将更加深入地了解 Netty 对于关注点分离的架构原则的支持。通过提供正确的抽象来解耦业务逻辑和网络编程逻辑，Netty 使得可以很容易地跟上快速演化的需求，而又不危及系统的稳定性。
​	在下一章中，我们将提供对 Netty 体系架构的概述。这将为你在后续的章节中对 Netty 的内部进行深入而全面的学习提供上下文。

## 第 3 章 Netty 的组件和设计

​	从高层次的角度来看，Netty 解决了两个相应的关注领域，我们可将其大致标记为技术的和体系结构的。首先，它的基于 Java NIO 的异步的和事件驱动的实现，保证了高负载下应用程序性能的最大化和可伸缩性。其次，Netty 也包含了一组设计模式，将应用程序逻辑从网络层解耦，简化了开发过程，同时也最大限度地提高了可测试性、模块化以及代码的可重用性。

### 3.1 Channel、EventLoop 和 ChannelFuture

​	接下来的各节将会为我们对于 Channel、EventLoop 和 ChannelFuture 类进行的讨论增添更多的细节，这些类合在一起，可以被认为是 Netty 网络抽象的代表：

- Channel—Socket；
- EventLoop—控制流、多线程处理、并发；
- ChannelFuture—异步通知；

#### 3.1.1 Channel 接口

​	基本的 I/O 操作（bind()、connect()、read()和 write()）依赖于底层网络传输所提供的原语。在基于 Java 的网络编程中，其基本的构造是 class Socket。Netty 的 Channel 接口所提供的 API，大大地降低了直接使用 Socket 类的复杂性。此外，Channel 也是拥有许多预定义的、专门化实现的广泛类层次结构的根，下面是一个简短的部分清单：

- EmbeddedChannel；
- LocalServerChannel；
- NioDatagramChannel；
- NioSctpChannel；
- NioSocketChannel。

#### 3.1.2 EventLoop 接口

​	EventLoop 定义了 Netty 的核心抽象，用于处理连接的生命周期中所发生的事件。我们将在第 7 章中结合 Netty 的线程处理模型的上下文对 EventLoop 进行详细的讨论。目前，图 3-1在高层次上说明了 Channel、EventLoop、Thread 以及 EventLoopGroup 之间的关系。

​	![1538126890303](E:\studydyup\notes\src\pic\%5CUsers%5CJudy%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5C1538126890303.png)

这些关系是：

- 一个 EventLoopGroup 包含一个或者多个 EventLoop；
- 一个 EventLoop 在它的生命周期内只和一个 Thread 绑定；
- 所有由 EventLoop 处理的 I/O 事件都将在它专有的 Thread 上被处理；
- 一个 Channel 在它的生命周期内只注册于一个 EventLoop；
- 一个 EventLoop 可能会被分配给一个或多个 Channel。

注意，在这种设计中，一个给定 Channel 的 I/O 操作都是由相同的 Thread 执行的，实际上消除了对于同步的需要。

#### 3.1.3 ChannelFuture 接口

​	正如我们已经解释过的那样，Netty 中所有的 I/O 操作都是异步的。因为一个操作可能不会立即返回，所以我们需要一种用于在之后的某个时间点确定其结果的方法。为此，Netty 提供了ChannelFuture 接口，其 addListener()方法注册了一个 ChannelFutureListener，以便在某个操作完成时（无论是否成功）得到通知。

> **关于 ChannelFuture 的更多讨论** 可以将 ChannelFuture 看作是将来要执行的操作的结果的
> 占位符。它究竟什么时候被执行则可能取决于若干的因素，因此不可能准确地预测，但是可以肯
> 定的是它将会被执行。此外，所有属于同一个 Channel 的操作都被保证其将以它们被调用的顺序
> 被执行。
> 我们将在第 7 章中深入地讨论 EventLoop 和 EventLoopGroup。

### 3.2 ChannelHandler 和 ChannelPipeline

现在，我们将更加细致地看一看那些管理数据流以及执行应用程序处理逻辑的组件。

#### 3.2.1 ChannelHandler 接口

​	从应用程序开发人员的角度来看，Netty 的主要组件是 ChannelHandler，它充当了所有处理入站和出站数据的应用程序逻辑的容器这是可行的，因为 ChannelHandler 的方法是由网络事件（其中术语“事件”的使用非常广泛）触发的。事实上，ChannelHandler 可专门用于几乎任何类型的动作，例如将数据从一种格式转换为另外一种格式，或者处理转换过程中所抛出的异常。

​	举例来说，ChannelInboundHandler 是一个你将会经常实现的子接口。这种类型的ChannelHandler 接收入站事件和数据，这些数据随后将会被你的应用程序业务逻辑所处理。当你要给连接的客户端发送响应时，也可以从 ChannelInboundHandler 冲刷数据。你的应用程序业务逻辑通常驻留在一个或者多个 ChannelInboundHandler 中。

#### 3.2.2 ChannelPipeline 接口

​	ChannelPipeline 提供了 ChannelHandler 链的容器，并定义了用于在该链上传播入站和出站事件流的 API。当 Channel 被创建时，它会被自动地分配到它专属的 ChannelPipeline。

ChannelHandler 安装到 ChannelPipeline 中的过程如下所示：

- 一个ChannelInitializer的实现被注册到了ServerBootstrap（或客户端）中 ；
- 当 ChannelInitializer.initChannel()方法被调用时，ChannelInitializer将在 ChannelPipeline 中安装一组自定义的 ChannelHandler；
- ChannelInitializer 将它自己从 ChannelPipeline 中移除。

​        为了审查发送或者接收数据时将会发生什么，让我们来更加深入地研究 ChannelPipeline和 ChannelHandler 之间的共生关系吧。

​	ChannelHandler 是专为支持广泛的用途而设计的，可以将它看作是处理往来 ChannelPipeline事件（包括数据）的任何代码的通用容器。图 3-2 说明了这一点，其展示了从 ChannelHandler派生的 ChannelInboundHandler 和 ChannelOutboundHandler 接口。

​	![1538129402604](E:\studydyup\notes\src\pic\1538129402604.png)

​	使得事件流经 ChannelPipeline 是 ChannelHandler 的工作，它们是在应用程序的初始化或者引导阶段被安装的。这些对象接收事件、执行它们所实现的处理逻辑，并将数据传递给链中的下一个 ChannelHandler。它们的执行顺序是由它们被添加的顺序所决定的。实际上，被我们称为 ChannelPipeline 的是这些 ChannelHandler 的编排顺序。
​	图 3-3 说明了一个 Netty 应用程序中入站和出站数据流之间的区别。从一个客户端应用程序的角度来看，如果事件的运动方向是从客户端到服务器端，那么我们称这些事件为出站的，反之则称为入站的。

![1538129749356](E:\studydyup\notes\src\pic\1538129749356.png)

图 3-3 也显示了入站和出站 ChannelHandler 可以被安装到同一个 ChannelPipeline
中。如果一个消息或者任何其他的入站事件被读取，那么它会从 ChannelPipeline 的头部
开始流动，并被传递给第一个 ChannelInboundHandler。这个 ChannelHandler 不一定
会实际地修改数据，具体取决于它的具体功能，在这之后，数据将会被传递给链中的下一个
ChannelInboundHandler。最终，数据将会到达 ChannelPipeline 的尾端，届时，所有
处理就都结束了。

数据的出站运动（即正在被写的数据）在概念上也是一样的。在这种情况下，数据将从
ChannelOutboundHandler 链的尾端开始流动，直到它到达链的头部为止。在这之后，出站
数据将会到达网络传输层，这里显示为 Socket。通常情况下，这将触发一个写操作。

>**关于入站和出站 ChannelHandler 的更多讨论**
>通过使用作为参数传递到每个方法的ChannelHandlerContext，事件可以被传递给当前ChannelHandler链中的下一个 ChannelHandler。因为你有时会忽略那些不感兴趣的事件，所以Netty提供了抽象基类 ChannelInboundHandlerAdapter和ChannelOutboundHandlerAdapter。通过调用 ChannelHandlerContext 上的对应方法，每个都提供了简单地将事件传递给下一个ChannelHandler的方法的实现。随后，你可以通过重写你所感兴趣的那些方法来扩展这些类。

鉴于出站操作和入站操作是不同的，你可能会想知道如果将两个类别的 ChannelHandler
都混合添加到同一个 ChannelPipeline 中会发生什么。虽然 ChannelInboundHandle 和
ChannelOutboundHandle 都扩展自 ChannelHandler，但是 Netty 能区分 ChannelInboundHandler
实现和 ChannelOutboundHandler 实现，并确保数据只会在具有相同定
向类型的两个 ChannelHandler 之间传递。

当ChannelHandler 被添加到ChannelPipeline 时，它将会被分配一个ChannelHandlerContext，其代表了ChannelHandler 和 ChannelPipeline 之间的绑定。虽然这个对象可以被用于获取底层的 Channel，但是它主要还是被用于写出站数据。
在 Netty 中，有两种发送消息的方式。你可以直接写到 Channel 中，也可以 写到和 ChannelHandler相关联的ChannelHandlerContext对象中。前一种方式将会导致消息从ChannelPipeline
的尾端开始流动，而后者将导致消息从 ChannelPipeline 中的下一个 ChannelHandler
开始流动。

#### 3.2.3 更加深入地了解 ChannelHandler

正如我们之前所说的，有许多不同类型的 ChannelHandler，它们各自的功能主要取决于
它们的超类。Netty 以适配器类的形式提供了大量默认的 ChannelHandler 实现，其旨在简化
应用程序处理逻辑的开发过程。你已经看到了，ChannelPipeline中的每个ChannelHandler
将负责把事件转发到链中的下一个 ChannelHandler。这些适配器类（及它们的子类）将自动
执行这个操作，所以你可以只重写那些你想要特殊处理的方法和事件。

>为什么需要适配器类
>有一些适配器类可以将编写自定义的 ChannelHandler 所需要的努力降到最低限度，因为它们提
>供了定义在对应接口中的所有方法的默认实现。
>下面这些是编写自定义 ChannelHandler 时经常会用到的适配器类：
> ChannelHandlerAdapter
> ChannelInboundHandlerAdapter
> ChannelOutboundHandlerAdapter
> ChannelDuplexHandler

​	接下来我们将研究 3 个 ChannelHandler 的子类型：编码器、解码器和SimpleChannelInboundHandler—— ChannelInboundHandlerAdapter 的一个子类。

#### 3.2.4 编码器和解码器

当你通过 Netty 发送或者接收一个消息的时候，就将会发生一次数据转换。入站消息会被解
码；也就是说，从字节转换为另一种格式，通常是一个 Java 对象。如果是出站消息，则会发生
相反方向的转换：它将从它的当前格式被编码为字节。这两种方向的转换的原因很简单：网络数
据总是一系列的字节。

对应于特定的需要，Netty 为编码器和解码器提供了不同类型的抽象类。例如，你的应用程
序可能使用了一种中间格式，而不需要立即将消息转换成字节。你将仍然需要一个编码器，但是
它将派生自一个不同的超类。为了确定合适的编码器类型，你可以应用一个简单的命名约定。

通常来说，这些基类的名称将类似于 ByteToMessageDecoder 或 MessageToByteEncoder。对于特殊的类型，你可能会发现类似于
ProtobufEncoder 和 ProtobufDecoder
这样的名称——预置的用来支持 Google 的 Protocol Buffers。

严格地说，其他的处理器也可以完成编码器和解码器的功能。但是，正如有用来简化
ChannelHandler 的创建的适配器类一样，所有由 Netty 提供的编码器/解码器适配器类都实现
了 ChannelOutboundHandler 或者 ChannelInboundHandler 接口。

你将会发现对于入站数据来说，channelRead 方法/事件已经被重写了。对于每个从入站
Channel 读取的消息，这个方法都将会被调用。随后，它将调用由预置解码器所提供的 decode()
方法，并将已解码的字节转发给 ChannelPipeline 中的下一个 ChannelInboundHandler。

出站消息的模式是相反方向的：编码器将消息转换为字节，并将它们转发给下一个
ChannelOutboundHandler。

#### 3.2.5 抽象类 SimpleChannelInboundHandler

​	最常见的情况是，你的应用程序会利用一个 ChannelHandler 来接收解码消息，并对该数据应用业务逻辑。要创建一个这样的 ChannelHandler，你只需要扩展基类SimpleChannelInboundHandler\<T\>，其中T 是你要处理的消息的 Java 类型 。在这个 ChannelHandler 中，你将需要重写基类的一个或者多个方法，并且获取一个到 ChannelHandlerContext 的引用，这个引用将作为输入参数传递给 ChannelHandler 的所有方法。

​	在这种类型的 ChannelHandler 中，最重要的方法是 channelRead0(ChannelHandlerContext,T)。除了要求不要阻塞当前的I/O 线程之外，其具体实现完全取决于你。我们稍后将对这一主题进行更多的说明。

### 3.3 引导

Netty 的引导类为应用程序的网络层配置提供了容器，这涉及将一个进程绑定到某个指定的
端口，或者将一个进程连接到另一个运行在某个指定主机的指定端口上的进程。

通常来说，我们把前面的用例称作引导一个服务器，后面的用例称作引导一个客户端。虽然
这个术语简单方便，但是它略微掩盖了一个重要的事实，即“服务器”和“客户端”实际上表示
了不同的网络行为；换句话说，是监听传入的连接还是建立到一个或者多个进程的连接。

>面向连接的协议 请记住，严格来说，“连接”这个术语仅适用于面向连接的协议，如 TCP，其
>保证了两个连接端点之间消息的有序传递。

因此，有两种类型的引导：一种用于客户端（简单地称为 Bootstrap），而另一种
（ServerBootstrap）用于服务器。无论你的应用程序使用哪种协议或者处理哪种类型的数据，
唯一决定它使用哪种引导类的是它是作为一个客户端还是作为一个服务器。表 3-1 比较了这两种
类型的引导类。

***表 3-1 比较 Bootstrap 类***

| 类 别                 | Bootstrap            | ServerBootstrap    |
| --------------------- | -------------------- | ------------------ |
| 网络编程中的作用      | 连接到远程主机和端口 | 绑定到一个本地端口 |
| EventLoopGroup 的数目 | 1                    | 2                  |

这两种类型的引导类之间的第一个区别已经讨论过了：ServerBootstrap 将绑定到一个
端口，因为服务器必须要监听连接，而 Bootstrap 则是由想要连接到远程节点的客户端应用程
序所使用的。

第二个区别可能更加明显。引导一个客户端只需要一个 EventLoopGroup，但是一个
ServerBootstrap 则需要两个（也可以是同一个实例）。为什么呢？

因为服务器需要两组不同的 Channel。第一组将只包含一个 ServerChannel，代表服务
器自身的已绑定到某个本地端口的正在监听的套接字。而第二组将包含所有已创建的用来处理传
入客户端连接（对于每个服务器已经接受的连接都有一个）的 Channel。图 3-4 说明了这个模
型，并且展示了为何需要两个不同的 EventLoopGroup。

![1538205662276](E:\studydyup\notes\src\pic\1538205662276.png)

与 ServerChannel 相关联的 EventLoopGroup 将分配一个负责为传入连接请求创建
Channel 的 EventLoop。一旦连接被接受，第二个 EventLoopGroup 就会给它的 Channel
分配一个 EventLoop。

### 3.4 小结

在本章中，我们从技术和体系结构这两个角度探讨了理解 Netty 的重要性。我们也更加详
细地重新审视了之前引入的一些概念和组件，特别是 ChannelHandler、ChannelPipeline
和引导。
特别地，我们讨论了 ChannelHandler 类的层次结构，并介绍了编码器和解码器，描述了
它们在数据和网络字节格式之间来回转换的互补功能。
下面的许多章节都将致力于深入研究这些组件，而这里所呈现的概览应该有助于你对整体
的把控。
下一章将探索 Netty 所提供的不同类型的传输，以及如何选择一个最适合于你的应用程序的
传输。

## 第 4 章 传输

>本章主要内容
> OIO——阻塞传输
> NIO——异步传输
> Local——JVM 内部的异步通信
> Embedded——测试你的 ChannelHandler

流经网络的数据总是具有相同的类型：字节。这些字节是如何流动的主要取决于我们所说的
网络传输—一个帮助我们抽象底层数据传输机制的概念。用户并不关心这些细节；他们只想确
保他们的字节被可靠地发送和接收。

如果你有 Java 网络编程的经验，那么你可能已经发现，在某些时候，你需要支撑比预期多
很多的并发连接。如果你随后尝试从阻塞传输切换到非阻塞传输，那么你可能会因为这两种网络
API 的截然不同而遇到问题。

然而，Netty 为它所有的传输实现提供了一个通用 API，这使得这种转换比你直接使用 JDK
所能够达到的简单得多。所产生的代码不会被实现的细节所污染，而你也不需要在你的整个代码
库上进行广泛的重构。简而言之，你可以将时间花在其他更有成效的事情上。

在本章中，我们将学习这个通用 API，并通过和 JDK 的对比来证明它极其简单易用。我们
将阐述 Netty 自带的不同传输实现，以及它们各自适用的场景。有了这些信息，你会发现选择最
适合于你的应用程序的选项将是直截了当的

本章的唯一前提是 Java 编程语言的相关知识。有网络框架或者网络编程相关的经验更好，
但不是必需的。

我们先来看一看传输在现实世界中是如何工作的。

### 4.1 案例研究：传输迁移

我们将从一个应用程序开始我们对传输的学习，这个应用程序只简单地接受连接，向客户端
写“Hi!”，然后关闭连接。

#### 4.1.1 不通过 Netty 使用 OIO 和 NIO

我们将介绍仅使用了 JDK API 的应用程序的阻塞（OIO）版本和异步（NIO）版本。代码清
单 4-1 展示了其阻塞版本的实现。如果你曾享受过使用 JDK 进行网络编程的乐趣，那么这段代码
将唤起你美好的回忆。

***代码清单 4-1 未使用 Netty 的阻塞网络编程***

```java
import java.io.IOException;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.charset.Charset;

/**
 * @author @Jasu
 * @date 2018-09-29 15:43
 */
public class PlainOioServer {
    public void serve(int port) throws IOException {
        //绑定
        final ServerSocket socket = new ServerSocket(port);

        try {
            //noinspection InfiniteLoopStatement
            for (; ; ) {
                //接收连接
                final Socket clientSocket = socket.accept();
                System.out.println("Accepted connection from " + clientSocket);

                new Thread(() -> {
                    OutputStream out;
                    try {
                        out = clientSocket.getOutputStream();
                        out.write("Hi\r\n".getBytes(Charset.forName("UTF-8")));
                        out.flush();
                        clientSocket.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }finally {
                        try {
                            clientSocket.close();
                        } catch (IOException e) {
                            //ignore on close
                        }
                    }
                }).start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

其非阻塞版本如代码清单 4-2 所示。

***代码清单 4-2 未使用 Netty 的异步网络编程***

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.ServerSocket;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

/**
 * @author @Jasu
 * @date 2018-09-29 16:28
 */
public class PlainNioServer {
    public void serve(int port) throws IOException {
        ServerSocketChannel serverChannel = ServerSocketChannel.open();
        serverChannel.configureBlocking(false);
        ServerSocket serverSocket = serverChannel.socket();
        //绑定
        serverSocket.bind(new InetSocketAddress(port));
        //打开Selector处理channel
        Selector selector = Selector.open();
        //将ServerSocketChannel注册到selector以接受连接
        serverChannel.register(selector, SelectionKey.OP_ACCEPT);
        final ByteBuffer msg = ByteBuffer.wrap("Hi\r\n".getBytes());

        for (; ; ) {
            try {
                //等待需要处理的新事件；在事件传入前阻塞
                selector.select();
            } catch (IOException e) {
                e.printStackTrace();
                //handle exception
                break;
            }
            //获取接收事件的所有selectionKey实例
            Set<SelectionKey> readyKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = readyKeys.iterator();
            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                iterator.remove();

                try {
                    //检查事件是否是一个新的就绪的可接受的连接
                    if (key.isAcceptable()) {
                        ServerSocketChannel server = (ServerSocketChannel) key.channel();
                        SocketChannel client = server.accept();
                        client.configureBlocking(false);
                        //接受客户端并注册到selector
                        client.register(selector, SelectionKey.OP_WRITE | SelectionKey.OP_READ, msg.duplicate());
                        System.out.println("Accepted connection from " + client);
                    }
                    //检查套接字是否准备好写数据
                    if (key.isWritable()) {
                        SocketChannel client = (SocketChannel) key.channel();
                        ByteBuffer buffer = (ByteBuffer) key.attachment();
                        while (buffer.hasRemaining()) {
                            //数据写到已连接的客户端
                            if (client.write(buffer) == 0) {
                                break;
                            }
                        }
                        //关闭连接
                        client.close();
                    }
                } catch (IOException e) {
                    key.cancel();
                    try {
                        key.channel().close();
                    } catch (IOException e1) {
                        //ignore on close
                    }
                }
            }
        }
    }
}
```

如同你所看到的，虽然这段代码所做的事情与之前的版本完全相同，但是代码却截然不同。
如果为了用于非阻塞 I/O 而重新实现这个简单的应用程序，都需要一次完全的重写的话，那么不
难想象，移植真正复杂的应用程序需要付出什么样的努力。
鉴于此，让我们来看看使用 Netty 实现该应用程序将会是什么样子吧。

#### 4.1.2 通过 Netty 使用 OIO 和 NIO

我们将先编写这个应用程序的另一个阻塞版本，这次我们将使用 Netty 框架，如代码清单 4-3
所示。

***代码清单 4-3 使用 Netty 的阻塞网络处理***

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.*;
import io.netty.channel.oio.OioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.oio.OioServerSocketChannel;
import io.netty.util.CharsetUtil;

import java.net.InetSocketAddress;

/**
 * @author @Jasu
 * @date 2018-09-29 17:59
 */
public class NettyOioServer {
    public void server(int port) throws Exception {
        final ByteBuf buf = Unpooled.unreleasableBuffer(Unpooled.copiedBuffer("Hi\r\n", CharsetUtil.UTF_8));
        EventLoopGroup group = new OioEventLoopGroup();

        try {
            //创建ServerBootstrap
            ServerBootstrap b = new ServerBootstrap();
            b.group(group)
                    //使用旧的OioEventLoopGroup以允许阻塞模式
                    .channel(OioServerSocketChannel.class)
                    .localAddress(new InetSocketAddress(port))
                    //指定ChannelInitializer，每个被接受的连接都调用它
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            //添加一个ChannelInboundHandlerAdapter以处理事件
                            ch.pipeline().addLast(new ChannelInboundHandlerAdapter() {
                                @Override
                                public void channelActive(ChannelHandlerContext ctx) throws Exception {
                                    //消息写到客户端，添加ChannelFutureListener，消息写完就关闭连接
                                    ctx.writeAndFlush(buf.duplicate()).addListener(ChannelFutureListener.CLOSE);
                                }
                            });
                        }
                    });
            //绑定服务器以接受连接
            ChannelFuture f = b.bind().sync();
            f.channel().closeFuture().sync();
        } finally {
            //释放资源
            group.shutdownGracefully().sync();
        }
    }
}
```

接下来，我们使用 Netty 和非阻塞 I/O 来实现同样的逻辑。

#### 4.1.3 非阻塞的 Netty 版本

代码清单 4-4 和代码清单 4-3 几乎一模一样，除了高亮显示的那两行。这就是从阻塞（OIO）
传输切换到非阻塞（NIO）传输需要做的所有变更。

***代码清单 4-4 使用 Netty 的异步网络处理***

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.util.CharsetUtil;

import java.net.InetSocketAddress;

/**
 * @author @Jasu
 * @date 2018-10-08 15:26
 */
public class NettyNioServer {
    public void server(int port) throws InterruptedException {
        final ByteBuf buf = Unpooled.copiedBuffer("Hi\r\n", CharsetUtil.UTF_8);

        EventLoopGroup group = new NioEventLoopGroup();

        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(group).channel(NioServerSocketChannel.class)
                    .localAddress(new InetSocketAddress(port))
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new ChannelInboundHandlerAdapter() {
                                @Override
                                public void channelActive(ChannelHandlerContext ctx) throws Exception {
                                    ctx.writeAndFlush(buf.duplicate())
                                            .addListener(ChannelFutureListener.CLOSE);
                                }
                            });
                        }
                    });
            ChannelFuture f = b.bind().sync();
            f.channel().closeFuture().sync();
        } finally {
            group.shutdownGracefully().sync();
        }
    }
}
```

因为 Netty 为每种传输的实现都暴露了相同的 API，所以无论选用哪一种传输的实现，你的代码都仍然几乎不受影响。在所有的情况下，传输的实现都依赖于 interface Channel、ChannelPipeline 和 ChannelHandler。

在看过一些使用基于 Netty 的传输的这些优点之后，让我们仔细看看传输 API 本身。

### 4.2 传输 API

传输 API 的核心是 interface Channel，它被用于所有的 I/O 操作。Channel 类的层次结构如图 4-1 所示。

![1539055320389](E:\studydyup\notes\src\pic\1539055320389.png)

如图所示，每个 Channel 都将会被分配一个 ChannelPipeline 和 ChannelConfig。ChannelConfig 包含了该 Channel 的所有配置设置，并且支持热更新。由于特定的传输可能具有独特的设置，所以它可能会实现一个 ChannelConfig 的子类型。（请参考 ChannelConfig实现对应的 Javadoc。）

由于 Channel 是独一无二的，所以为了保证顺序将 Channel 声明为 java.lang.Comparable 的一个子接口。因此，如果两个不同的 Channel 实例都返回了相同的散列码，那么 AbstractChannel 中的 compareTo()方法的实现将会抛出一个 Error。

ChannelPipeline 持有所有将应用于入站和出站数据以及事件的 ChannelHandler 实例，这些 ChannelHandler 实现了应用程序用于处理状态变化以及数据处理的逻辑。

ChannelHandler 的典型用途包括：

- 将数据从一种格式转换为另一种格式；

- 提供异常的通知；
- 提供 Channel 变为活动的或者非活动的通知；
- 提供当 Channel 注册到 EventLoop 或者从 EventLoop 注销时的通知；
- 提供有关用户自定义事件的通知。

>**拦截过滤器** ChannelPipeline 实现了一种常见的设计模式—拦截过滤器（Intercepting
>Filter）。UNIX 管道是另外一个熟悉的例子：多个命令被链接在一起，其中一个命令的输出端将连
>接到命令行中下一个命令的输入端。

你也可以根据需要通过添加或者移除ChannelHandler实例来修改ChannelPipeline。

通过利用Netty的这项能力可以构建出高度灵活的应用程序。例如，每当STARTTLS协议被请求时，你可以简单地通过 向 ChannelPipeline 添 加 一个适当的 ChannelHandler（SslHandler）来按需地支持STARTTLS协议。

除了访问所分配的 ChannelPipeline 和 ChannelConfig 之外，也可以利用 Channel的其他方法，其中最重要的列举在表 4-1 中。

***表 4-1 Channel 的方法***

| 方 法 名      | 描 述                                                        |
| ------------- | ------------------------------------------------------------ |
| eventLoop     | 返回分配给 Channel 的 EventLoop                              |
| pipeline      | 返回分配给 Channel 的 ChannelPipeline                        |
| isActive      | 如果 Channel 是活动的，则返回 true。活动的意义可能依赖于底层的传输。例如，<br/>一个 Socket 传输一旦连接到了远程节点便是活动的，而一个 Datagram 传输一旦
被打开便是活动的 |
| localAddress  | 返回本地的 SokcetAddress                                     |
| remoteAddress | 返回远程的 SocketAddress                                     |
| write         | 将数据写到远程节点。这个数据将被传递给 ChannelPipeline，并且排队直到它被<br/>冲刷 |
| flush         | 将之前已写的数据冲刷到底层传输，如一个 Socket                |
| writeAndFlush | 一个简便的方法，等同于调用 write()并接着调用 flush()         |

稍后我们将进一步深入地讨论所有这些特性的应用。目前，请记住，Netty 所提供的广泛功能只依赖于少量的接口。这意味着，你可以对你的应用程序逻辑进行重大的修改，而又无需大规模地重构你的代码库。

考虑一下写数据并将其冲刷到远程节点这样的常规任务。代码清单 4-5 演示了使用Channel.writeAndFlush()来实现这一目的。

***代码清单 4-5 写出到 Channel***

```java
Channel channel = ...;
        ByteBuf buf = Unpooled.copiedBuffer("Hi\r\n", CharsetUtil.UTF_8);
        ChannelFuture cf = channel.writeAndFlush(buf);
        cf.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) throws Exception {
                if (future.isSuccess()) {
                    System.out.println("Write successful");
                } else {
                    System.err.println("Write error");
                    future.cause().printStackTrace();
                }
            }
        });
```

Netty 的 Channel 实现是线程安全的，因此你可以存储一个到 Channel 的引用，并且每当你需要向远程节点写数据时，都可以使用它，即使当时许多线程都在使用它。代码清单 4-6 展示了一个多线程写数据的简单例子。需要注意的是，消息将会被保证按顺序发送。

***代码清单 4-6 从多个线程使用同一个 Channel***

```java
final Channel channel = ...
        final ByteBuf buf = Unpooled.copiedBuffer("Hi\r\n", CharsetUtil.UTF_8).retain();
        Runnable writer = new Runnable() {
            @Override
            public void run() {
                channel.writeAndFlush(buf.duplicate());
            }
        };
        ExecutorService service = Executors.newCachedThreadPool();
        //write in one thread
        service.execute(writer);
        // write in another thread
        service.execute(writer);
```

### 4.3 内置的传输

Netty 内置了一些可开箱即用的传输。因为并不是它们所有的传输都支持每一种协议，所以你必须选择一个和你的应用程序所使用的协议相容的传输。在本节中我们将讨论这些关系。

表 4-2 显示了所有 Netty 提供的传输。

 **4-2 Netty 所提供的传输**

| 名称     | 包                          | 描述                                                         |
| -------- | --------------------------- | ------------------------------------------------------------ |
| NIO      | io.netty.channel.socket.nio | 使用 java.nio.channels 包作为基础——基于选择器的方式          |
| Epoll①   | io.netty.channel.epoll      | 由 JNI 驱动的 epoll()和非阻塞 IO。这个传输支持只有在Linux上可用的多种特性，如SO_REUSEPORT，比 NIO 传输更快，而且是完全非阻塞的 |
| OIO      | io.netty.channel.socket.oio | 使用 java.net 包作为基础——使用阻塞流                         |
| Local    | io.netty.channel.local      | 可以在 VM 内部通过管道进行通信的本地传输                     |
| Embedded | io.netty.channel.embedded   | Embedded 传输，允许使用 ChannelHandler 而又不需要一个真正的基于网络的传输。这在测试你的ChannelHandler 实现时非常有用 |

#### 4.3.1 NIO——非阻塞 I/O

NIO 提供了一个所有 I/O 操作的全异步的实现。它利用了自 NIO 子系统被引入 JDK 1.4 时便可用的基于选择器的 API。

Selector背后的基本概念是充当一个注册表，在那里你将可以请求在 Channel 的状态发生变化时得到通知。可能的状态变化有：

- 新的 Channel 已被接受并且就绪；
- Channel 连接已经完成；
- Channel 有已经就绪的可供读取的数据；
- Channel 可用于写数据。

Selector运行在一个检查状态变化并对其做出相应响应的线程上，在应用程序对状态的改变做出响应之后，Selector将会被重置，并将重复这个过程。

表4-3中的常量值代表了由class java.nio.channels.SelectionKey定义的位模式。这些位模式可以组合起来定义一组应用程序正在请求通知的状态变化集。

***表 4-3 选择操作的位模式***

| 名 称      | 描 述                                                        |
| ---------- | ------------------------------------------------------------ |
| OP_ACCEPT  | 请求在接受新连接并创建 Channel 时获得通知                    |
| OP_CONNECT | 请求在建立一个连接时获得通知                                 |
| OP_READ    | 请求当数据已经就绪，可以从 Channel 中读取时获得通知          |
| OP_WRITE   | 请求当可以向 Channel 中写更多的数据时获得通知。这处理了套接字缓冲区被完全填满时的情况，这种情况通常发生在数据的发送速度比远程节点可处理的速度更快的时候 |

对于所有 Netty 的传输实现都共有的用户级别 API 完全地隐藏了这些 NIO 的内部细节。图 4-2 展示了该处理流程。

​									*4.3 内置的传输*

![1539160462529](E:\studydyup\notes\src\pic\1539160462529.png)

> 零拷贝
> 零拷贝（zero-copy）是一种目前只有在使用 NIO 和 Epoll 传输时才可使用的特性。它使你可以快速高效地将数据从文件系统移动到网络接口，而不需要将其从内核空间复制到用户空间，其在像 FTP 或者HTTP 这样的协议中可以显著地提升性能。但是，并不是所有的操作系统都支持这一特性。特别地，它对于实现了数据加密或者压缩的文件系统是不可用的——只能传输文件的原始内容。反过来说，传输已被加密的文件则不是问题。

#### 4.3.2 Epoll—用于 Linux 的本地非阻塞传输

正如我们之前所说的，Netty 的 NIO 传输基于 Java 提供的异步/非阻塞网络编程的通用抽象。虽然这保证了 Netty 的非阻塞 API 可以在任何平台上使用，但它也包含了相应的限制，因为 JDK为了在所有系统上提供相同的功能，必须做出妥协。

Linux作为高性能网络编程的平台，其重要性与日俱增，这催生了大量先进特性的开发，其中包括epoll——一个高度可扩展的I/O事件通知特性。这个API自Linux内核版本 2.5.44（2002）被引入，提供了比旧的POSIX select和poll系统调用 ①更好的性能，同时现在也是Linux上非阻塞网络编程的事实标准。Linux JDK NIO API使用了这些epoll调用。

① 参见 Linux 手册页中的 epoll(4)：http://linux.die.net/man/4/epoll。

Netty为Linux提供了一组NIO API，其以一种和它本身的设计更加一致的方式使用epoll，并且以一种更加轻量的方式使用中断。①如果你的应用程序旨在运行于Linux系统，那么请考虑利用这个版本的传输；你将发现在高负载下它的性能要优于JDK的NIO实现。

这个传输的语义与在图 4-2 所示的完全相同，而且它的用法也是简单直接的。相关示例参照代码清单 4-4。如果要在那个代码清单中使用 epoll 替代 NIO，只需要将 NioEventLoopGroup替换为 EpollEventLoopGroup ，并且将 NioServerSocketChannel.class 替换为EpollServerSocketChannel.class 即可。

#### 4.3.3 OIO—旧的阻塞 I/O

Netty 的 OIO 传输实现代表了一种折中：它可以通过常规的传输 API 使用，但是由于它是建立在 java.net 包的阻塞实现之上的，所以它不是异步的。但是，它仍然非常适合于某些用途。

例如，你可能需要移植使用了一些进行阻塞调用的库（如JDBC②）的遗留代码，而将逻辑转换为非阻塞的可能也是不切实际的。相反，你可以在短期内使用Netty的OIO传输，然后再将你的代码移植到纯粹的异步传输上。让我们来看一看怎么做。

在 java.net API 中，你通常会有一个用来接受到达正在监听的 ServerSocket 的新连接的线程。会创建一个新的和远程节点进行交互的套接字，并且会分配一个新的用于处理相应通信流量的线程。这是必需的，因为某个指定套接字上的任何 I/O 操作在任意的时间点上都可能会阻塞。使用单个线程来处理多个套接字，很容易导致一个套接字上的阻塞操作也捆绑了所有其他的套接字。

有了这个背景，你可能会想，Netty是如何能够使用和用于异步传输相同的API来支持OIO的呢。答案就是，Netty利用了SO_TIMEOUT这个Socket标志，它指定了等待一个I/O操作完成的最大毫秒数。如果操作在指定的时间间隔内没有完成，则将会抛出一个SocketTimeout Exception。Netty将捕获这个异常并继续处理循环。在EventLoop下一次运行时，它将再次尝试。这实际上也是类似于Netty这样的异步框架能够支持OIO的唯一方式③。图 4-3 说明了这个逻辑。

① JDK 的实现是水平触发，而 Netty 的（默认的）是边沿触发。有关的详细信息参见 epoll 在维基百科上的解释：http://en.wikipedia.org/wiki/Epoll - Triggering_modes。
② JDBC 的文档可以在 www.oracle.com/technetwork/java/javase/jdbc/index.html 获取。
③ 这种方式的一个问题是，当一个 SocketTimeoutException 被抛出时填充栈跟踪所需要的时间，其
对于性能来说代价很大。

![1539163138476](E:\studydyup\notes\src\pic\1539163138476.png)

​							*图 4-3 OIO 的处理逻辑*

#### 4.3.4 用于 JVM 内部通信的 Local 传输

Netty 提供了一个 Local 传输，用于在同一个 JVM 中运行的客户端和服务器程序之间的异步通信。同样，这个传输也支持对于所有 Netty 传输实现都共同的 API。

在这个传输中，和服务器 Channel 相关联的 SocketAddress 并没有绑定物理网络地址；相反，只要服务器还在运行，它就会被存储在注册表里，并在 Channel 关闭时注销。因为这个传输并不接受真正的网络流量，所以它并不能够和其他传输实现进行互操作。因此，客户端希望连接到（在同一个 JVM 中）使用了这个传输的服务器端时也必须使用它。除了这个限制，它的使用方式和其他的传输一模一样。

#### 4.3.5 Embedded 传输

Netty 提供了一种额外的传输，使得你可以将一组 ChannelHandler 作为帮助器类嵌入到其他的 ChannelHandler 内部。通过这种方式，你将可以扩展一个 ChannelHandler 的功能，而又不需要修改其内部代码。
不足为奇的是，Embedded 传输的关键是一个被称为 EmbeddedChannel 的具体的 Channel实现。在第 9 章中，我们将详细地讨论如何使用这个类来为 ChannelHandler 的实现创建单元测试用例。

### 4.4 传输的用例

既然我们已经详细地了解了所有的传输，那么让我们考虑一下选用一个适用于特定用途的协议的因素吧。正如前面所提到的，并不是所有的传输都支持所有的核心协议，其可能会限制你的选择。表 4-4 展示了截止出版时的传输和其所支持的协议。

​							***表 4-4 支持的传输和网络协议***

| 传输              | TCP  | UDP  | SCTP | UDT  |
| ----------------- | ---- | ---- | ---- | ---- |
| NIO               | ×    | ×    | ×    | ×    |
| Epoll（仅 Linux） | ×    | ×    | —    | —    |
| OIO               | ×    | ×    | ×    | ×    |

> 在 Linux 上启用 SCTP
> SCTP 需要内核的支持，并且需要安装用户库。
> 例如，对于 Ubuntu，可以使用下面的命令：
>
> `# sudo apt-get install libsctp1`
>
> 对于 Fedora，可以使用 yum：
> `#sudo yum install kernel-modules-extra.x86_64 lksctp-tools.x86_64`
> 有关如何启用 SCTP 的详细信息，请参考你的 Linux 发行版的文档。

虽然只有SCTP传输有这些特殊要求，但是其他传输可能也有它们自己的配置选项需要考虑。此外，如果只是为了支持更高的并发连接数，服务器平台可能需要配置得和客户端不一样。

这里是一些你很可能会遇到的用例。

- *非阻塞代码库*——如果你的代码库中没有阻塞调用（或者你能够限制它们的范围），那么在 Linux 上使用 NIO 或者 epoll 始终是个好主意。虽然 NIO/epoll 旨在处理大量的并发连接，但是在处理较小数目的并发连接时，它也能很好地工作，尤其是考虑到它在连接之间共享线程的方式。
- *阻塞代码库*——正如我们已经指出的，如果你的代码库严重地依赖于阻塞 I/O，而且你的应用程序也有一个相应的设计，那么在你尝试将其直接转换为 Netty 的 NIO 传输时，你将可能会遇到和阻塞操作相关的问题。不要为此而重写你的代码，可以考虑分阶段迁移：先从OIO 开始，等你的代码修改好之后，再迁移到 NIO（或者使用 epoll，如果你在使用 Linux）
- *在同一个 JVM 内部的通信*——在同一个 JVM 内部的通信，不需要通过网络暴露服务，是Local 传输的完美用例。这将消除所有真实网络操作的开销，同时仍然使用你的 Netty 代码库。如果随后需要通过网络暴露服务，那么你将只需要把传输改为 NIO 或者 OIO 即可。
- *测试你的 ChannelHandler 实现*——如果你想要为自己的 ChannelHandler 实现编写单元测试，那么请考虑使用 Embedded 传输。这既便于测试你的代码，而又不需要创建大量的模拟（mock）对象。你的类将仍然符合常规的 API 事件流，保证该 ChannelHandler在和真实的传输一起使用时能够正确地工作。

表 4-5 总结了我们探讨过的用例。

​							***表 4-5 应用程序的最佳传输***

| 应用程序的需求                 | 推荐的传输                       |
| ------------------------------ | -------------------------------- |
| 非阻塞代码库或者一个常规的起点 | NIO（或者在 Linux 上使用 epoll） |
| 阻塞代码库                     | OIO                              |
| 在同一个 JVM 内部的通信        | Local                            |
| 测试 ChannelHandler 的实现     | Embedded                         |

### 4.5 小结

在本章中，我们研究了传输、它们的实现和使用，以及 Netty 是如何将它们呈现给开发者的。

我们深入探讨了 Netty 预置的传输，并且解释了它们的行为。因为不是所有的传输都可以在相同的 Java 版本下工作，并且其中一些可能只在特定的操作系统下可用，所以我们也描述了它们的最低需求。最后，我们讨论了你可以如何匹配不同的传输和特定用例的需求。

在下一章中，我们将关注于 ByteBuf 和 ByteBufHolder——Netty 的数据容器。我们将展示如何使用它们以及如何通过它们获得最佳性能。

## 第 5 章 ByteBuf

>本章主要内容
> ByteBuf——Netty 的数据容器
> API 的详细信息
> 用例
> 内存分配

正如前面所提到的，网络数据的基本单位总是字节。Java NIO 提供了 ByteBuffer 作为它的字节容器，但是这个类使用起来过于复杂，而且也有些繁琐。

Netty 的 ByteBuffer 替代品是 ByteBuf，一个强大的实现，既解决了 JDK API 的局限性，又为网络应用程序的开发者提供了更好的 API。

在本章中我们将会说明和 JDK 的 ByteBuffer 相比，ByteBuf 的卓越功能性和灵活性。这也将有助于更好地理解 Netty 数据处理的一般方式，并为将在第 6 章中针对 ChannelPipeline和 ChannelHandler 的讨论做好准备。

### 5.1 ByteBuf 的 API

Netty 的数据处理 API 通过两个组件暴露——abstract class ByteBuf 和 interfaceByteBufHolder。

下面是一些 ByteBuf API 的优点：

- 它可以被用户自定义的缓冲区类型扩展；
- 通过内置的复合缓冲区类型实现了透明的零拷贝；
- 容量可以按需增长（类似于 JDK 的 StringBuilder）；
- 在读和写这两种模式之间切换不需要调用 ByteBuffer 的 flip()方法；
- 读和写使用了不同的索引；
- 支持方法的链式调用；
- 支持引用计数；
- 支持池化。

其他类可用于管理 ByteBuf 实例的分配，以及执行各种针对于数据容器本身和它所持有的数据的操作。我们将在仔细研究 ByteBuf 和 ByteBufHolder 时探讨这些特性。

### 5.2 ByteBuf 类——Netty 的数据容器

因为所有的网络通信都涉及字节序列的移动，所以高效易用的数据结构明显是必不可少的。Netty 的 ByteBuf 实现满足并超越了这些需求。让我们首先来看看它是如何通过使用不同的索引来简化对它所包含的数据的访问的吧。

#### 5.2.1 它是如何工作的

ByteBuf 维护了两个不同的索引：一个用于读取，一个用于写入。当你从 ByteBuf 读取时，它的 readerIndex 将会被递增已经被读取的字节数。同样地，当你写入 ByteBuf 时，它的writerIndex 也会被递增。图 5-1 展示了一个空 ByteBuf 的布局结构和状态。

![1539229312124](E:\studydyup\notes\src\pic\1539229312124.png)

​				***图 5-1 一个读索引和写索引都设置为 0 的 16 字节 ByteBuf***

要了解这些索引两两之间的关系，请考虑一下，如果打算读取字节直到 readerIndex 达到和 writerIndex 同样的值时会发生什么。在那时，你将会到达“可以读取的”数据的末尾。就如同试图读取超出数组末尾的数据一样，试图读取超出该点的数据将会触发一个IndexOutOfBoundsException。

名称以 read 或者 write 开头的 ByteBuf 方法，将会推进其对应的索引，而名称以 set 或者 get 开头的操作则不会。后面的这些方法将在作为一个参数传入的一个相对索引上执行操作。

可以指定 ByteBuf 的最大容量。试图移动写索引（即 writerIndex）超过这个值将会触发一个异常①。（默认的限制是 Integer.MAX_VALUE。）

① 也就是说用户直接或者间接使 capacity(int)或者 ensureWritable(int)方法来增加超过该最大
容量时抛出异常。—译者注

#### 5.2.2 ByteBuf 的使用模式

在使用 Netty 时，你将遇到几种常见的围绕 ByteBuf 而构建的使用模式。在研究它们时，我们心里想着图 5-1 会有所裨益—一个由不同的索引分别控制读访问和写访问的字节数组。

##### 1．堆缓冲区

最常用的 ByteBuf 模式是将数据存储在 JVM 的堆空间中。这种模式被称为支撑数组（backing array），