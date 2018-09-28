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

