# Arthas

`Arthas` 是Alibaba开源的Java诊断工具，深受开发者喜爱。

当你遇到以下类似问题而束手无策时，`Arthas`可以帮助你解决：

1. 这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
2. 我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？
3. 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？
4. 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
5. 是否有一个全局视角来查看系统的运行状况？
6. 有什么办法可以监控到JVM的实时运行状态？

`Arthas`采用命令行交互模式，同时提供丰富的 `Tab` 自动补全功能，进一步方便进行问题的定位和诊断。

## Ⅰ Arthas Install

```shell
curl -L https://alibaba.github.io/arthas/install.sh | sh
```

卸载

```
rm -rf ~/.arthas/ ~/.arthas_history
```

## Ⅱ  快速入门

### 1. 启动Demo

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
public class Demo {
    static class Counter {
        private static AtomicInteger count = new AtomicInteger(0);
        public static void increment() {
            count.incrementAndGet();
        }
        public static int value() {
            return count.get();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        while (true) {
            Counter.increment();
            System.out.println("counter: " + Counter.value());
            TimeUnit.SECONDS.sleep(1);
        }
    }
}
```

### 2. 启动arthas

```bash
./as.sh
```

选择应用java进程：

```bash
$ ./as.sh
Arthas script version: 3.0.2
Found existing java process, please choose one and hit RETURN.
* [1]: 95428 
  [2]: 22647 org.jetbrains.jps.cmdline.Launcher
  [3]: 21736
  [4]: 13560 Demo
```

### 3. 查看dashboard

输入[dashboard](https://alibaba.github.io/arthas/dashboard.html)，Ctrl + c断开。

### 4. watch

watch

### 5.退出

quit 

shutdown

## Ⅲ  进阶使用

### 基础命令

- help——查看命令帮助信息
- cls——清空当前屏幕区域
- session——查看当前会话的信息
- [reset](https://alibaba.github.io/arthas/reset.html)——重置增强类，将被 Arthas 增强过的类全部还原，Arthas 服务端关闭时会重置所有增强过的类
- version——输出当前目标 Java 进程所加载的 Arthas 版本号
- quit——退出当前 Arthas 客户端，其他 Arthas 客户端不受影响
- shutdown——关闭 Arthas 服务端，所有 Arthas 客户端全部退出
- [keymap](https://alibaba.github.io/arthas/keymap.html)——Arthas快捷键列表及自定义快捷键

### jvm相关

- [dashboard](https://alibaba.github.io/arthas/dashboard.html)——当前系统的实时数据面板
- [thread](https://alibaba.github.io/arthas/thread.html)——查看当前 JVM 的线程堆栈信息
- [jvm](https://alibaba.github.io/arthas/jvm.html)——查看当前 JVM 的信息
- [sysprop](https://alibaba.github.io/arthas/sysprop.html)——查看和修改JVM的系统属性
- **New!** [getstatic](https://alibaba.github.io/arthas/getstatic.html)——查看类的静态属性

### class/classloader相关

- [sc](https://alibaba.github.io/arthas/sc.html)——查看JVM已加载的类信息
- [sm](https://alibaba.github.io/arthas/sm.html)——查看已加载类的方法信息
- [dump](https://alibaba.github.io/arthas/dump.html)——dump 已加载类的 byte code 到特定目录
- [redefine](https://alibaba.github.io/arthas/redefine.html)——加载外部的`.class`文件，redefine到JVM里
- [jad](https://alibaba.github.io/arthas/jad.html)——反编译指定已加载类的源码
- [classloader](https://alibaba.github.io/arthas/classloader.html)——查看classloader的继承树，urls，类加载信息，使用classloader去getResource

### monitor/watch/trace相关

> 请注意，这些命令，都通过字节码增强技术来实现的，会在指定类的方法中插入一些切面来实现数据统计和观测，因此在线上、预发使用时，请尽量明确需要观测的类、方法以及条件，诊断结束要执行 `shutdown` 或将增强过的类执行 `reset` 命令。

- [monitor](https://alibaba.github.io/arthas/monitor.html)——方法执行监控
- [watch](https://alibaba.github.io/arthas/watch.html)——方法执行数据观测
- [trace](https://alibaba.github.io/arthas/trace.html)——方法内部调用路径，并输出方法路径上的每个节点上耗时
- [stack](https://alibaba.github.io/arthas/stack.html)——输出当前方法被调用的调用路径
- [tt](https://alibaba.github.io/arthas/tt.html)——方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测

### options

- [options](https://alibaba.github.io/arthas/options.html)——查看或设置Arthas全局开关

### 管道

Arthas支持使用管道对上述命令的结果进行进一步的处理，如`sm org.apache.log4j.Logger | grep <init>`

- grep——搜索满足条件的结果
- plaintext——将命令的结果去除颜色
- wc——按行统计输出结果

### 后台异步任务

当线上出现偶发的问题，比如需要watch某个条件，而这个条件一天可能才会出现一次时，异步后台任务就派上用场了，详情请参考[这里](https://alibaba.github.io/arthas/async.html)

- 使用 > 将结果重写向到日志文件，使用 & 指定命令是后台运行，session断开不影响任务执行（生命周期默认为1天）
- jobs——列出所有job
- kill——强制终止任务
- fg——将暂停的任务拉到前台执行
- bg——将暂停的任务放到后台执行

### Web Console

通过websocket连接Arthas。

Arthas目前支持Web Console，用户在attach成功之后，可以直接访问：<http://localhost:8563/>。

可以填入IP，远程连接其它机器上的arthas。

![web console](https://alibaba.github.io/arthas/_images/web-console-local.png)

后续更多Web Console功能支持，请到issue下留言：<https://github.com/alibaba/arthas/issues/15>

### 其他特性

#### 异步命令支持

arthas中的异步调用，使用了仿linux系统任务相关的命令。[linux任务相关介绍](https://ehlxr.me/2017/01/18/Linux-%E4%B8%AD-fg%E3%80%81bg%E3%80%81jobs%E3%80%81-%E6%8C%87%E4%BB%A4/)。

##### 1. 使用&在后台执行任务

比如希望执行后台执行trace命令，那么调用下面命令

```
trace Test t &  
```

这时命令在后台执行，可以在console中继续执行其他命令。

##### 2. 通过jobs查看任务

如果希望查看当前有哪些arthas任务在执行，可以执行jobs命令，执行结果如下

```
$ jobs
[10]*
       Stopped           watch com.taobao.container.Test test "params[0].{? #this.name == null }" -x 2
       execution count : 19
       start time      : Fri Sep 22 09:59:55 CST 2017
       timeout date    : Sat Sep 23 09:59:55 CST 2017
       session         : 3648e874-5e69-473f-9eed-7f89660b079b (current)
```

可以看到目前有一个后台任务在执行。

- job id是10, `*` 表示此job是当前session创建
- 状态是Stopped
- execution count是执行次数，从启动开始已经执行了19次
- timeout date是超时的时间，到这个时间，任务将会自动超时退出

##### 3. 任务暂停和取消

当任务正在前台执行，比如直接调用命令`trace Test t`或者调用后台执行命令`trace Test t &`后又通过`fg`命令将任务转到前台。这时console中无法继续执行命令，但是可以接收并处理以下事件：

- ‘ctrl + z’：将任务暂停。通过`jbos`查看任务状态将会变为Stopped，通过`bg <job-id>`或者`fg <job-id>`可让任务重新开始执行
- ‘ctrl + c’：停止任务
- ‘ctrl + d’：按照linux语义应当是退出终端，目前arthas中是空实现，不处理

##### 4. fg、bg命令，将命令转到前台、后台继续执行

- 任务在后台执行或者暂停状态（‘ctrl + z’暂停任务）时，执行fg <job-id>将可以把对应的任务转到前台继续执行。在前台执行时，无法在console中执行其他命令
- 当任务处于暂停状态时（‘ctrl + z’暂停任务），执行bg <job-id>将可以把对应的任务在后台继续执行
- 非当前session创建的job，只能由当前session fg到前台执行

##### 5. 任务输出重定向

可通过`>`或者`>>`将任务输出结果输出到指定的文件中，可以和`&`一起使用，实现arthas命令的异步调用。比如：

```
$ trace Test t >> test.out &
```

这时trace命令会在后台执行，并且把结果输出到~/logs/arthas-cache/test.out。可继续执行其他命令。并可查看文件中的命令执行结果。

当远程调用时，可能无法确定机器文件路径，arthas同时支持了自动重定向到缓存路径。使用方法如下：

```
$ trace Test t >>  &
job id  : 2
cache location  : /Users/gehui/logs/arthas-cache/28198/2
```

可以看到并没有指定重定向文件位置，arthas自动重定向到缓存中了，执行命令后会输出job id和cache location。cache location就是重定向文件的路径，在系统logs目录下，路径包括pid和job id，避免和其他任务冲突。命令输出结果到`/Users/gehui/logs/arthas-cache/28198/2`中，job id为2。

##### 6. 停止命令

异步执行的命令，如果希望停止，可执行kill <job-id>

##### 7. 其他

- 最多同时支持8个命令使用重定向将结果写日志
- 请勿同时开启过多的后台异步命令，以免对目标JVM性能造成影响

#### 执行结果存日志

> 将命令的结果完整保存在日志文件中，便于后续进行分析

- 默认情况下，该功能是关闭的，如果需要开启，请执行以下命令：

```
$ options save-result true
 NAME         BEFORE-VALUE  AFTER-VALUE
----------------------------------------
 save-result  false         true
Affect(row-cnt:1) cost in 3 ms.
```

看到上面的输出，即表示成功开启该功能；

- 日志文件路径

结果会异步保存在：`{user.home}/logs/arthas-cache/result.log`，请定期进行清理，以免占据磁盘空间

##### 使用新版本Arthas的异步后台任务将结果存日志文件

```
$ trace Test t >>  &
job id  : 2
cache location  : /Users/zhuyong/logs/arthas-cache/28198/2
```

此时命令会在后台异步执行，并将结果异步保存在文件（~/logs/arthas-cache/${PID}/${JobId}）中；

- 此时任务的执行不受session断开的影响；任务默认超时时间是1天，可以通过全局 `options` 命令修改默认超时时间；
- 此命令的结果将异步输出到文件中；此时不管 `save-result` 是否为true，都不会再往~/logs/arthas-cache/result.log 中异步写结果

#### 批处理功能

通过批处理功能，arthas支持一次性批量运行多个命令，并取得命令执行的结果。

##### 第一步： 创建你的批处理脚本

这里我们新建了一个`test.as`脚本，为了规范，我们采用了.as后缀名，但事实上任意的本文文件都ok。

> 注意事项
>
> - 目前需要每个命令占一行
> - dashboard务必开启批处理模式(`-b`)，指定执行次数(`-n`)，否则会导致批处理脚本无法终止
> - watch/tt/trace/monitor/stack等命令务必指定执行次数(`-n`)，否则会导致批处理脚本无法终止
> - 可以使用异步后台任务，如 `watch c.t.X test returnObj > &`，让命令一直在后台运行，通过日志获取结果，[获取更多异步任务的信息](https://alibaba.github.io/arthas/async.html)

```
➜  arthas git:(develop) cat /var/tmp/test.as
help
dashboard -b -n 1
session
thread
sc -d org.apache.commons.lang.StringUtils
```



##### 第二步： 运行你的批处理脚本

通过`-b`开启批处理模式， `-f`执行脚本文件， 批处理脚本默认会输出到标准输出中，可以将结果重定向到文件中。

```
./as.sh -b -f /var/tmp/test.as 56328 > test.out
```



##### 第三步： 查看运行结果

```
cat test.out
```

#### OGNL表达式

## Ⅳ  命令列表

### 1. dashborad

> 当前系统的实时数据面板，按 ctrl+c 退出。

当运行在Ali-tomcat时，会显示当前tomcat的实时信息，如HTTP请求的qps, rt, 错误数, 线程池信息等等。

#### 使用参考

```
$ dashboard
ID         NAME                              GROUP                  PRIORITY   STATE       %CPU       TIME       INTERRUPTED DAEMON
889        RMI TCP Connection(15)-30.10.166. RMI Runtime            9          RUNNABLE    48         0:5        false       true
1077       Timer-for-arthas-dashboard-0      system                 9          RUNNABLE    24         0:0        false       true
1074       as-selector-daemon                system                 9          RUNNABLE    12         0:0        false       true
284        JMX server connection timeout 284 RMI Runtime            9          TIMED_WAITI 8          0:3        false       true
16         Timer-1                           main                   5          TIMED_WAITI 5          0:9        false       true
47         Pandora pandora-qos-reporter Pool main                   5          TIMED_WAITI 0          0:0        false       true
48         JmonitorClient-CheckThread Pool [ main                   5          TIMED_WAITI 0          0:0        false       true
49         JmonitorClient-HeartBeat Pool [Th main                   5          TIMED_WAITI 0          0:0        false       true
50         JmonitorClient-ReaderThread Pool  main                   5          TIMED_WAITI 0          0:0        false       true
957        RMI TCP Connection(16)-30.10.166. RMI Runtime            9          RUNNABLE    0          0:2        false       true
51         JmonitorClient-WriterThread Pool  main                   5          TIMED_WAITI 0          0:0        false       true
52         ContainerBackgroundProcessor[Stan main                   5          TIMED_WAITI 0          0:0        false       true
53         http-bio-8080-Acceptor-0          main                   5          RUNNABLE    0          0:2        false       true
54         http-bio-8080-AsyncTimeout        main                   5          TIMED_WAITI 0          0:0        false       true
11         GC Daemon                         system                 2          TIMED_WAITI 0          0:0        false       true
Memory                       used      total     max      usage     GC
heap                         59M       223M      1820M    3.26%     gc.ps_scavenge.count              118
ps_eden_space                14M       114M      668M     2.11%     gc.ps_scavenge.time(ms)           1890
ps_survivor_space            6M        6M        6M       96.08%    gc.ps_marksweep.count             5
ps_old_gen                   39M       103M      1365M    2.86%     gc.ps_marksweep.time(ms)          1140
nonheap                      234M      240M      0M       97.46%
code_cache                   46M       47M       240M     19.49%
metaspace                    167M      172M      0M       97.36%
Runtime                                                             Tomcat
os.name                           Mac OS X                          connector                         http-bio-8080
os.version                        10.10.5                           QPS                               0.00
java.version                      1.8.0_60                          RT(ms)                            1.13
java.home                                                           error/s                           0.00
                                                                    received/s                        0B
systemload.average                3.44                              sent/s                            0B
processors                        4                                 threadpool                        http-bio-8080
uptime                            16020s                            busy    
```

#### 数据说明

- ID: Java级别的线程ID，注意这个ID不能跟jstack中的nativeID一一对应
- NAME: 线程名
- GROUP: 线程组名
- PRIORITY: 线程优先级, 1~10之间的数字，越大表示优先级越高
- STATE: 线程的状态
- CPU%: 线程消耗的cpu占比，采样100ms，将所有线程在这100ms内的cpu使用量求和，再算出每个线程的cpu使用占比。
- TIME: 线程运行总时间，数据格式为`分：秒`
- INTERRUPTED: 线程当前的中断位状态
- DAEMON: 是否是daemon线程

#### 截图展示

![alt text](https://alibaba.github.io/arthas/_images/dashboard.png)

### 2. thread

> 查看当前线程信息，查看线程的堆栈

#### 参数说明

| 参数名称      | 参数说明                              |
| ------------- | ------------------------------------- |
| *id*          | 线程id                                |
| [n:]          | 指定最忙的前N个线程并打印堆栈         |
| [b]           | 找出当前阻塞其他线程的线程            |
| [i `<value>`] | 指定cpu占比统计的采样间隔，单位为毫秒 |

​	cpu占比是如何统计出来的？

​	这里的cpu统计的是，一段采样间隔内，当前JVM里各个线程所占用的cpu时间占总cpu时间的百分比。其计算方法为： 首先进行一次采样，获得所有线程的cpu的使用时间(调用的是`java.lang.management.ThreadMXBean#getThreadCpuTime`这个接口)，然后睡眠一段时间，默认100ms，可以通过`-i`参数指定，然后再采样一次，最后得出这段时间内各个线程消耗的cpu时间情况，最后算出百分比。

​	注意： 这个统计也会产生一定的开销（JDK这个接口本身开销比较大），因此会看到as的线程占用一定的百分比，为了降低统计自身的开销带来的影响，可以把采样间隔拉长一些，比如5000毫秒。

​	如果想看从Java进程启动开始到现在的cpu占比情况：可以使用[show-busy-java-threads](https://github.com/oldratlee/useful-scripts/blob/master/docs/java.md#-show-busy-java-threads)这个脚本

### 使用参考

#### 支持一键展示当前最忙的前N个线程并打印堆栈：

```bash
$ thread -n 3
```

#### 当没有参数时，显示所有线程的信息。

```
$ thread
```

#### thread id， 显示指定线程的运行堆栈

```
$ thread 1
```

#### thread -b, 找出当前阻塞其他线程的线程

有时候我们发现应用卡住了， 通常是由于某个线程拿住了某个锁， 并且其他线程都在等待这把锁造成的。 为了排查这类问题， arthas提供了`thread -b`， 一键找出那个罪魁祸首。

```
$ thread -b
```

#### thread -i, 指定采样时间间隔

```
$ thread -n 3 -i 1000
```

### 3. jvm

> 查看当前JVM信息

#### 使用参考

```
$ jvm
```

#### THREAD相关

- COUNT: JVM当前活跃的线程数
- DAEMON-COUNT: JVM当前活跃的守护线程数
- LIVE-COUNT: 从JVM启动开始曾经活着的最大线程数
- STARTED-COUNT: 从jvm启动开始总共启动过的线程次数

### 4. sysprop

#### 使用参考

```
 USAGE:
   sysprop [-h] [property-name] [property-value]

 SUMMARY:
   Display, and change all the system properties.

 EXAMPLES:
 sysprop
 sysprop file.encoding
 sysprop production.mode true

 WIKI:
   https://alibaba.github.io/arthas/sysprop

 OPTIONS:
 -h, --help                                  this help
 <property-name>                             property name
 <property-value>                            property value
```

#### 查看所有属性

```
$ sysprop
```

#### 查看单个属性

> 支持通过`TAB`键自动补全

```
$ sysprop java.version
java.version=1.8.0_51
```

#### 修改单个属性

```bash
$ sysprop user.country
user.country=US
$ sysprop user.country CN
Successfully changed the system property.
user.country=CN
```

### 5. getstatic

通过getstatic命令可以方便的查看类的静态属性。使用方法为`getstatic class_name field_name`

如果该静态属性是一个复杂对象，还可以支持在该属性上通过ognl表示进行遍历，过滤，访问对象的内部属性等操作。

例如，假设n是一个Map，Map的Key是一个Enum，我们想过滤出Map中Key为某个Enum的值，可以写如下命令

```bash
$ getstatic com.alibaba.arthas.Test n 'entrySet().iterator.{? #this.key.name()=="STOP"}'
field: n
@ArrayList[
    @Node[STOP=bbb],
]
Affect(row-cnt:1) cost in 68 ms.

$ getstatic com.alibaba.arthas.Test m 'entrySet().iterator.{? #this.key=="a"}'
field: m
@ArrayList[
    @Node[a=aaa],
]
```

### 6. sc

> 查看JVM已加载的类信息

“Search-Class” 的简写，这个命令能搜索出所有已经加载到 JVM 中的 Class 信息，这个命令支持的参数有 `[d]`、`[E]`、`[f]` 和 `[x:]`。

#### 参数说明

| 参数名称         | 参数说明                                                     |
| ---------------- | ------------------------------------------------------------ |
| *class-pattern*  | 类名表达式匹配                                               |
| *method-pattern* | 方法名表达式匹配                                             |
| [d]              | 输出当前类的详细信息，包括这个类所加载的原始文件来源、类的声明、加载的ClassLoader等详细信息。 如果一个类被多个ClassLoader所加载，则会出现多次 |
| [E]              | 开启正则表达式匹配，默认为通配符匹配                         |
| [f]              | 输出当前类的成员变量信息（需要配合参数-d一起使用）           |
| [x:]             | 指定输出静态变量时属性的遍历深度，默认为 0，即直接使用 `toString` 输出 |

> class-pattern支持全限定名，如com.taobao.test.AAA，也支持com/taobao/test/AAA这样的格式，这样，我们从异常堆栈里面把类名拷贝过来的时候，不需要在手动把`/`替换为`.`啦。

> sc 默认开启了子类匹配功能，也就是说所有当前类的子类也会被搜索出来，想要精确的匹配，请打开`options disable-sub-class true`开关

#### 使用参考

例如， 查看类的静态变量信息， 可以用`sc -df class-name`

```
$ sc -df org.apache.commons.lang.StringUtils
```

### 7. sm

> 查看已加载类的方法信息

“Search-Method” 的简写，这个命令能搜索出所有已经加载了 Class 信息的方法信息。

`sm` 命令只能看到由当前类所声明 (declaring) 的方法，父类则无法看到。

#### 参数说明

| 参数名称         | 参数说明                             |
| ---------------- | ------------------------------------ |
| *class-pattern*  | 类名表达式匹配                       |
| *method-pattern* | 方法名表达式匹配                     |
| [d]              | 展示每个方法的详细信息               |
| [E]              | 开启正则表达式匹配，默认为通配符匹配 |

#### 使用参考

```
$ sm org.apache.catalina.connector.Connector
```

```
# sm org.apache.catalina.connector.Connector -d
```

### 8. dump

> dump 已加载类的 bytecode 到特定目录

#### 参数说明

| *class-pattern* | 类名表达式匹配                       |
| --------------- | ------------------------------------ |
| `[c:]`          | 类所属 ClassLoader 的 hashcode       |
| [E]             | 开启正则表达式匹配，默认为通配符匹配 |

#### 使用参考

```
dump -E org\.apache\.commons\.lang\.StringUtils
```

### 9. jad

> 反编译指定已加载类的源码

`jad` 命令将 JVM 中实际运行的 class 的 byte code 反编译成 java 代码，便于你理解业务逻辑；

- 在 Arthas Console 上，反编译出来的源码是带语法高亮的，阅读更方便
- 当然，反编译出来的 java 代码可能会存在语法错误，但不影响你进行阅读理解



#### 参数说明

| 参数名称        | 参数说明                             |
| --------------- | ------------------------------------ |
| *class-pattern* | 类名表达式匹配                       |
| `[c:]`          | 类所属 ClassLoader 的 hashcode       |
| [E]             | 开启正则表达式匹配，默认为通配符匹配 |

#### 使用参考

> 当有多个 `ClassLoader` 都加载了这个类时，`jad` 命令会输出对应 `ClassLoader` 实例的 `hashcode`，然后你只需要重新执行 `jad` 命令，并使用参数 `-c <hashcode>` 就可以反编译指定 ClassLoader 加载的那个类了；

```
$ jad org.apache.log4j.Logger
```

反编译指定的方法：

```
jad com.taobao.container.web.arthas.rest.MetricsController directMetrics
```

### 10. classloader

> 查看classloader的继承树，urls，类加载信息

`classloader` 命令将 JVM 中所有的classloader的信息统计出来，并可以展示继承树，urls等。

可以让指定的classloader去getResources，打印出所有查找到的resources的url。对于`ResourceNotFoundException`比较有用。



#### 参数说明

| 参数名称  | 参数说明                                |
| --------- | --------------------------------------- |
| [l]       | 按类加载实例进行统计                    |
| [t]       | 打印所有ClassLoader的继承树             |
| [a]       | 列出所有ClassLoader加载的类，请谨慎使用 |
| `[c:]`    | ClassLoader的hashcode                   |
| `[c: r:]` | 用ClassLoader去查找resource             |

#### 使用参考

- 按类加载类型查看统计信息

  ```
  $ classloader
  ```

- 按类加载实例查看统计信息

  ```
  $ classloader -l
  ```

- 查看ClassLoader的继承树

  ```
  $ classloader -t
  ```

- 查看URLClassLoader实际的urls

  ```
  $ classloader -c 5ffe9775
  ```

- 使用ClassLoader去查找resource

  ```
  $ classloader -c 226b143b -r META-INF/MANIFEST.MF
  ```

- 也可以尝试查找类的class文件：

  ```
  $ classloader -c 1b6d3586 -r java/lang/String.class
  ```

### 11. redefine

> 加载外部的`.class`文件，redefine jvm已加载的类。

参考：[Instrumentation#redefineClasses](https://docs.oracle.com/javase/8/docs/api/java/lang/instrument/Instrumentation.html#redefineClasses-java.lang.instrument.ClassDefinition...-)

> 注意， redefine后的原来的类不能恢复，redefine有可能失败（比如增加了新的field），参考jdk本身的文档。



#### 参数说明

| 参数名称 | 参数说明                               |
| -------- | -------------------------------------- |
| [c:]     | ClassLoader的hashcode                  |
| [p:]     | 外部的`.class`文件的完整路径，支持多个 |

#### 使用参考

```
   redefine -p /tmp/Test.class
   redefine -c 327a647b -p /tmp/Test.class /tmp/Test\$Inner.class
```

### 12. monitor

> 方法执行监控

对匹配 `class-pattern`／`method-pattern`的类、方法的调用进行监控。

`monitor` 命令是一个非实时返回命令，实时返回命令是输入之后立即返回，而非实时返回的命令，则是不断的等待目标 Java 进程返回信息，直到用户输入 `Ctrl+C` 为止。服务端是以任务的形式在后台跑任务，植入的代码随着任务的中止而被不会被执行，所以任务关闭后，不会对原有性能产生太大影响，而且原则上，任何 Arthas 的命令也不会引起任何原有业务逻辑的改变。

#### 监控的维度说明

| 监控项    | 说明                       |
| --------- | -------------------------- |
| timestamp | 时间戳                     |
| class     | Java类                     |
| method    | 方法（构造方法、普通方法） |
| total     | 调用次数                   |
| success   | 成功次数                   |
| fail      | 失败次数                   |
| rt        | 平均RT                     |
| fail-rate | 失败率                     |

#### 参数说明

方法拥有一个命名参数 `[c:]`，意思是统计周期（cycle of output），拥有一个整形的参数值

| 参数名称         | 参数说明                             |
| ---------------- | ------------------------------------ |
| *class-pattern*  | 类名表达式匹配                       |
| *method-pattern* | 方法名表达式匹配                     |
| [E]              | 开启正则表达式匹配，默认为通配符匹配 |
| `[c:]`           | 统计周期，默认值为120秒              |

#### 使用参考

```
$ monitor -c 5 com.alibaba.sample.petstore.web.store.module.screen.ItemList execute
```

### 13. watch

> 方法执行数据观测

让你能方便的观察到指定方法的调用情况。能观察到的范围为：`返回值`、`抛出异常`、`入参`，通过编写 OGNL 表达式进行对应变量的查看。



#### 参数说明

watch 的参数比较多，主要是因为它能在 4 个不同的场景观察对象

| 参数名称            | 参数说明                                   |
| ------------------- | ------------------------------------------ |
| *class-pattern*     | 类名表达式匹配                             |
| *method-pattern*    | 方法名表达式匹配                           |
| *express*           | 观察表达式                                 |
| *condition-express* | 条件表达式                                 |
| [b]                 | 在**方法调用之前**观察                     |
| [e]                 | 在**方法异常之后**观察                     |
| [s]                 | 在**方法返回之后**观察                     |
| [f]                 | 在**方法结束之后**(正常返回和异常返回)观察 |
| [E]                 | 开启正则表达式匹配，默认为通配符匹配       |
| [x:]                | 指定输出结果的属性遍历深度，默认为 1       |

这里重点要说明的是观察表达式，观察表达式的构成主要由 ognl 表达式组成，所以你可以这样写`"{params,returnObj}"`，只要是一个合法的 ognl 表达式，都能被正常支持。

观察的维度也比较多，主要体现在参数 `advice` 的数据结构上。`Advice` 参数最主要是封装了通知节点的所有信息。请参考[表达式核心变量](https://alibaba.github.io/arthas/advice-class.html)中关于该节点的描述。

- 特殊用法请参考：<https://github.com/alibaba/arthas/issues/71>
- OGNL表达式官网：<https://commons.apache.org/proper/commons-ognl/language-guide.html>

**特别说明**：

- watch 命令定义了4个观察事件点，即 `-b` 方法调用前，`-e` 方法异常后，`-s` 方法返回后和 `-f` 方法结束后
- 4个观察事件点 `-b`、`-e`、`-s` 默认关闭，`-f` 默认打开，当指定观察点被打开后，在相应事件点会对观察表达式进行求值并输出
- 这里要注意`方法入参`和`方法出参`的区别，有可能在中间被修改导致前后不一致，除了 `-b` 事件点 `params` 代表方法入参外，其余事件都代表方法出参
- 当使用 `-b` 时，由于观察事件点是在方法调用前，此时返回值或异常均不存在

#### 使用参考

代码示例：

```
    public void execute() {
        List<String> list = new ArrayList<String>();
        list.add("a");
        list.add("b");

        List<String> list2 = new ArrayList<String>();
        list2.add("c");
        list2.add("d");

        int len = add(list, list2);
    }

    private static int add(List<String> list, List<String> list2) {
        list.addAll(list2);
        return list.size();
    }
```

##### 观察方法出参和返回值

```
$ watch com.alibaba.sample.petstore.web.store.module.screen.ItemList add "{params,returnObj}" -x 2
```

##### 观察方法入参

```
$ watch com.alibaba.sample.petstore.web.store.module.screen.ItemList add "{params,returnObj}" -x 2 -b
```

> 对比前一个例子，params[0] 其size为2（入参），返回值为空（事件点为方法执行前，因此获取不到返回值）

##### 同时观察方法调用前和方法返回后

```
$ watch com.alibaba.sample.petstore.web.store.module.screen.ItemList add "{params,returnObj}" -x 2 -b -s 
```

> 这里输出结果中，第一次输出的是方法调用前的观察表达式的结果，第二次输出的是方法返回后的表达式的结果

> 结果的顺序和命令中 `-s -b` 的顺序没有关系，只与事件本身的先后顺序有关

##### 调整`-x`的值，观察具体的方法参数值

```
$ watch com.alibaba.sample.petstore.web.store.module.screen.ItemList add "{params,returnObj}" -x 3 
```

`-x`表示遍历深度，可以调整来打印具体的参数和结果内容。

##### 条件表达式的例子

```
$ watch com.alibaba.sample.petstore.biz.impl.UserManagerImpl testAdd "{params, returnObj}" "params[0].equals('aaa')" -x 2
```

> 只有满足条件的调用，才会有响应。

##### 观察异常信息的例子

```
$ watch com.alibaba.sample.petstore.biz.impl.UserManagerImpl testAdd "{params, throwExp}"  -e -x 2 
```

> express中，表示异常信息的变量是`throwExp`

##### 按照耗时进行过滤

```
$ watch com.alibaba.sample.petstore.web.store.module.screen.ItemList add "{params,returnObj}" #cost>200 -x 3 
```

> #cost>200(单位是`ms`)表示只有当耗时大于200ms时才会输出，过滤掉执行时间小于200ms的调用

##### 观察当前对象中的全局属性

如果想查看方法运行前后，当前对象中的全局属性，可以使用`target`关键字，代表当前对象

```
$ watch com.taobao.container.web.arthas.rest.MyAppsController myFavoriteApps 'target'
```

然后使用`target.field_name`访问当前对象的某个全局属性

```
$ watch com.taobao.container.web.arthas.rest.MyAppsController myFavoriteApps 'target.myFavAppsMapper'
```

### 14. trace

> 方法内部调用路径，并输出方法路径上的每个节点上耗时

`trace` 命令能主动搜索 `class-pattern`／`method-pattern` 对应的方法调用路径，渲染和统计整个调用链路上的所有性能开销和追踪调用链路。

#### 参数说明

| 参数名称            | 参数说明                             |
| ------------------- | ------------------------------------ |
| *class-pattern*     | 类名表达式匹配                       |
| *method-pattern*    | 方法名表达式匹配                     |
| *condition-express* | 条件表达式                           |
| [E]                 | 开启正则表达式匹配，默认为通配符匹配 |
| `[n:]`              | 命令执行次数                         |
| `#cost`             | 方法执行耗时                         |

这里重点要说明的是观察表达式，观察表达式的构成主要由 ognl 表达式组成，所以你可以这样写`"{params,returnObj}"`，只要是一个合法的 ognl 表达式，都能被正常支持。

观察的维度也比较多，主要体现在参数 `advice` 的数据结构上。`Advice` 参数最主要是封装了通知节点的所有信息。

请参考[表达式核心变量](https://alibaba.github.io/arthas/advice-class.html)中关于该节点的描述。

- 特殊用法请参考：<https://github.com/alibaba/arthas/issues/71>
- OGNL表达式官网：<https://commons.apache.org/proper/commons-ognl/language-guide.html>

很多时候我们只想看到某个方法的rt大于某个时间之后的trace结果，现在Arthas可以按照方法执行的耗时来进行过滤了，例如`trace *StringUtils isBlank '$cost>100'`表示当执行时间超过100ms的时候，才会输出trace的结果。

注意：

1. watch/stack/trace这个三个命令都支持$cost
2. 如果是Arthas 3.0，请把`$cost`改为`#cost`

#### 注意事项

`trace` 能方便的帮助你定位和发现因 RT 高而导致的性能问题缺陷，但其每次只能跟踪一级方法的调用链路。

#### 使用参考

代码示例：

```java
    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        list.add("a");
        list.add("b");

        List<String> list2 = new ArrayList<String>();
        list2.add("c");
        list2.add("d");

        int len = add(list, list2);
    }

    private static int add(List<String> list, List<String> list2) {
        int i = 10;
        while (i >= 0) {
            try {
                hehe(i);
            } catch (Throwable t) {
                t.printStackTrace();
            }
            i--;
        }

        list.addAll(list2);
        return list.size();
    }

    private static void hehe(int i) {
        if (i == 0) {
            throw new RuntimeException("ZERO");
        }
    }
```

监测 `add` 方法：

```
$ trace com.alibaba.sample.petstore.web.store.module.screen.ItemList add params.length==2
```

按照耗时过滤：

```
$ trace com.alibaba.sample.petstore.web.store.module.screen.ItemList execute #cost>4
```

> 只会展示耗时大于4ms的调用路径，有助于在排查问题的时候，只关注异常情况

- 是不是很眼熟，没错，在 JProfiler 等收费软件中你曾经见识类似的功能，这里你将可以通过命令就能打印出指定调用路径。 友情提醒下，`trace` 在执行的过程中本身是会有一定的性能开销，在统计的报告中并未像 JProfiler 一样预先减去其自身的统计开销。所以这统计出来有些许的不准，渲染路径上调用的类、方法越多，性能偏差越大。但还是能让你看清一些事情的。
- [2ms] 的含义，`2` 的含义是：当前节点在当前步骤的耗时，单位为毫秒
- [0,0,0ms,11]xxx:yyy() [throws Exception]，对该方法中相同的方法调用进行了合并，`0,0,0ms,11` 表示方法调用耗时，`min,max,total,count`；`throws Exception` 表明该方法调用中存在异常返回
- 这里存在一个统计不准确的问题，就是所有方法耗时加起来可能会小于该监测方法的总耗时，这个是由于 Arthas 本身的逻辑会有一定的耗时

### 15.stack

> 输出当前方法被调用的调用路径

很多时候我们都知道一个方法被执行，但这个方法被执行的路径非常多，或者你根本就不知道这个方法是从那里被执行了，此时你需要的是 stack 命令。

#### 参数说明

| 参数名称            | 参数说明                             |
| ------------------- | ------------------------------------ |
| *class-pattern*     | 类名表达式匹配                       |
| *method-pattern*    | 方法名表达式匹配                     |
| *condition-express* | 条件表达式                           |
| [E]                 | 开启正则表达式匹配，默认为通配符匹配 |
| `[n:]`              | 执行次数限制                         |

这里重点要说明的是观察表达式，观察表达式的构成主要由 ognl 表达式组成，所以你可以这样写`"{params,returnObj}"`，只要是一个合法的 ognl 表达式，都能被正常支持。

观察的维度也比较多，主要体现在参数 `advice` 的数据结构上。`Advice` 参数最主要是封装了通知节点的所有信息。

请参考[表达式核心变量](https://alibaba.github.io/arthas/advice-class.html)中关于该节点的描述。

- 特殊用法请参考：<https://github.com/alibaba/arthas/issues/71>
- OGNL表达式官网：<https://commons.apache.org/proper/commons-ognl/language-guide.html>



#### 使用例子

> 注意：如果表达式里面包含了引号，那么需要把整个表达式用引号括起来，如果表达式中没有包含引号，那么可以不用引号。当然，一个好的习惯是，不管表达式中有没有引号，都使用引号括起来。

```
$ stack com.alibaba.sample.petstore.dal.dao.ProductDao getProductById 'params[0]=="K9-BD-01"'
```

按照耗时查询:

```
$ stack com.alibaba.sample.petstore.web.store.module.screen.ItemList execute #cost>30
```

### 16. tt

> 方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测

`watch` 虽然很方便和灵活，但需要提前想清楚观察表达式的拼写，这对排查问题而言要求太高，因为很多时候我们并不清楚问题出自于何方，只能靠蛛丝马迹进行猜测。

这个时候如果能记录下当时方法调用的所有入参和返回值、抛出的异常会对整个问题的思考与判断非常有帮助。

于是乎，TimeTunnel 命令就诞生了。

#### 记录方法的调用

- 基本用法

  对于一个最基本的使用来说，就是记录下当前方法的每次调用环境现场。

  ```
  $ tt -t -n 3 *Test print
  ```

- 命令参数解析

  - ```
    -t
    ```

    tt 命令有很多个主参数，`-t` 就是其中之一。这个参数的表明希望记录下类 `*Test` 的 `print` 方法的每次执行情况。

  - ```
    -n 3
    ```

    当你执行一个调用量不高的方法时可能你还能有足够的时间用 `CTRL+C` 中断 tt 命令记录的过程，但如果遇到调用量非常大的方法，瞬间就能将你的 JVM 内存撑爆。

    此时你可以通过 `-n` 参数指定你需要记录的次数，当达到记录次数时 Arthas 会主动中断tt命令的记录过程，避免人工操作无法停止的情况。

- 表格字段说明

  | 表格字段  | 字段解释                                                     |
  | --------- | ------------------------------------------------------------ |
  | INDEX     | 时间片段记录编号，每一个编号代表着一次调用，后续tt还有很多命令都是基于此编号指定记录操作，非常重要。 |
  | TIMESTAMP | 方法执行的本机时间，记录了这个时间片段所发生的本机时间       |
  | COST(ms)  | 方法执行的耗时                                               |
  | IS-RET    | 方法是否以正常返回的形式结束                                 |
  | IS-EXP    | 方法是否以抛异常的形式结束                                   |
  | OBJECT    | 执行对象的`hashCode()`，注意，曾经有人误认为是对象在JVM中的内存地址，但很遗憾他不是。但他能帮助你简单的标记当前执行方法的类实体 |
  | CLASS     | 执行的类名                                                   |
  | METHOD    | 执行的方法名                                                 |

- 条件表达式

  - 不知道大家是否有在使用过程中遇到以下困惑

    - Arthas 似乎很难区分出重载的方法
    - 我只需要观察特定参数，但是 tt 却全部都给我记录了下来

    条件表达式也是用 `OGNL` 来编写，核心的判断对象依然是 `Advice` 对象。除了 `tt` 命令之外，`watch`、`trace`、`stack` 命令也都支持条件表达式。

  - 解决方法重载

    `tt -t *Test print params[0].length==1`

    通过制定参数个数的形式解决不同的方法签名，如果参数个数一样，你还可以这样写

    `tt -t *Test print 'params[1] instanceof Integer'`

  - 解决指定参数

    `tt -t *Test print params[0].mobile=="13989838402"`

  - 构成条件表达式的 `Advice` 对象

    前边看到了很多条件表达式中，都适用了 `params[0]`，有关这个变量的介绍，请参考[表达式核心变量](https://alibaba.github.io/arthas/advice-class.html)

#### 检索调用记录

当你用 `tt` 记录了一大片的时间片段之后，你希望能从中筛选出自己需要的时间片段，这个时候你就需要对现有记录进行检索。

假设我们有这些记录

```
$ tt -l
+----------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
|    INDEX |            TIMESTAMP |   COST(ms) |   IS-RET |   IS-EXP |          OBJECT |                          CLASS |                         METHOD |
+----------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
|     1000 |  2015-07-26 01:16:27 |        130 |     true |    false |      0x42cc13a0 |                GaOgnlUtilsTest |                          print |
+----------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
|     1001 |  2015-07-26 01:16:27 |          0 |    false |     true |      0x42cc13a0 |                GaOgnlUtilsTest |                   printAddress |
+----------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
|     1002 |  2015-07-26 01:16:28 |        119 |     true |    false |      0x42cc13a0 |                GaOgnlUtilsTest |                          print |
+----------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
|     1003 |  2015-07-26 01:16:28 |          0 |    false |     true |      0x42cc13a0 |                GaOgnlUtilsTest |                   printAddress |
+----------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
|     1004 |  2015-07-26 12:21:56 |        130 |     true |    false |      0x42cc13a0 |                GaOgnlUtilsTest |                          print |
+----------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
|     1005 |  2015-07-26 12:21:57 |        138 |     true |    false |      0x42cc13a0 |                GaOgnlUtilsTest |                          print |
+----------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
|     1006 |  2015-07-26 12:21:58 |        130 |     true |    false |      0x42cc13a0 |                GaOgnlUtilsTest |                          print |
+----------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
Affect(row-cnt:7) cost in 2 ms.
$ 
```

我需要筛选出 `printAddress` 方法的调用信息

```
$ tt -s method.name=="printAddress"
+----------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
|    INDEX |            TIMESTAMP |   COST(ms) |   IS-RET |   IS-EXP |          OBJECT |                          CLASS |                         METHOD |
+----------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
|     1001 |  2015-07-26 01:16:27 |          0 |    false |     true |      0x42cc13a0 |                GaOgnlUtilsTest |                   printAddress |
+----------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
|     1003 |  2015-07-26 01:16:28 |          0 |    false |     true |      0x42cc13a0 |                GaOgnlUtilsTest |                   printAddress |
+----------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
Affect(row-cnt:2) cost in 55 ms.
$ 
```

你需要一个 `-s` 参数。同样的，搜索表达式的核心对象依旧是 `Advice` 对象。

#### 查看调用信息

对于具体一个时间片的信息而言，你可以通过 `-i` 参数后边跟着对应的 `INDEX` 编号查看到他的详细信息。

```
$ 
$ tt -i 1003
+-----------------+------------------------------------------------------------------------------------------------------+
|           INDEX | 1003                                                                                                 |
+-----------------+------------------------------------------------------------------------------------------------------+
|      GMT-CREATE | 2015-07-26 01:16:28                                                                                  |
+-----------------+------------------------------------------------------------------------------------------------------+
|        COST(ms) | 0                                                                                                    |
+-----------------+------------------------------------------------------------------------------------------------------+
|          OBJECT | 0x42cc13a0                                                                                           |
+-----------------+------------------------------------------------------------------------------------------------------+
|           CLASS | GaOgnlUtilsTest                                                                                      |
+-----------------+------------------------------------------------------------------------------------------------------+
|          METHOD | printAddress                                                                                         |
+-----------------+------------------------------------------------------------------------------------------------------+
|       IS-RETURN | false                                                                                                |
+-----------------+------------------------------------------------------------------------------------------------------+
|    IS-EXCEPTION | true                                                                                                 |
+-----------------+------------------------------------------------------------------------------------------------------+
|   PARAMETERS[0] | Address@53448f87                                                                                     |
+-----------------+------------------------------------------------------------------------------------------------------+
| THROW-EXCEPTION | java.lang.RuntimeException: test                                                                     |
|                 |     at GaOgnlUtilsTest.printAddress(Unknown Source)                                                  |
|                 |     at GaOgnlUtilsTest.<init>(Unknown Source)                                                        |
|                 |     at GaOgnlUtilsTest.main(Unknown Source)                                                          |
+-----------------+------------------------------------------------------------------------------------------------------+
Affect(row-cnt:1) cost in 1 ms.
$ 
```

####　重做一次调用

当你稍稍做了一些调整之后，你可能需要前端系统重新触发一次你的调用，此时得求爷爷告奶奶的需要前端配合联调的同学再次发起一次调用。而有些场景下，这个调用不是这么好触发的。

`tt` 命令由于保存了当时调用的所有现场信息，所以我们可以自己主动对一个 `INDEX` 编号的时间片自主发起一次调用，从而解放你的沟通成本。此时你需要 `-p` 参数。

```
$ tt -i 1003 -p
+-----------------+---------------------------------------------------------------------------------------------------------+
|        RE-INDEX | 1003                                                                                                    |
+-----------------+---------------------------------------------------------------------------------------------------------+
|      GMT-REPLAY | 2015-07-26 17:29:51                                                                                     |
+-----------------+---------------------------------------------------------------------------------------------------------+
|          OBJECT | 0x42cc13a0                                                                                              |
+-----------------+---------------------------------------------------------------------------------------------------------+
|           CLASS | GaOgnlUtilsTest                                                                                         |
+-----------------+---------------------------------------------------------------------------------------------------------+
|          METHOD | printAddress                                                                                            |
+-----------------+---------------------------------------------------------------------------------------------------------+
|   PARAMETERS[0] | Address@53448f87                                                                                        |
+-----------------+---------------------------------------------------------------------------------------------------------+
|       IS-RETURN | false                                                                                                   |
+-----------------+---------------------------------------------------------------------------------------------------------+
|    IS-EXCEPTION | true                                                                                                    |
+-----------------+---------------------------------------------------------------------------------------------------------+
| THROW-EXCEPTION | java.lang.RuntimeException: test                                                                        |
|                 |     at GaOgnlUtilsTest.printAddress(GaOgnlUtilsTest.java:78)                                            |
|                 |     at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)                                      |
|                 |     at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)                    |
|                 |     at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)            |
|                 |     at java.lang.reflect.Method.invoke(Method.java:483)                                                 |
|                 |     at com.github.ompc.Arthas.util.GaMethod.invoke(GaMethod.java:81)                                     |
|                 |     at com.github.ompc.Arthas.command.TimeTunnelCommand$6.action(TimeTunnelCommand.java:592)             |
|                 |     at com.github.ompc.Arthas.server.DefaultCommandHandler.execute(DefaultCommandHandler.java:175)       |
|                 |     at com.github.ompc.Arthas.server.DefaultCommandHandler.executeCommand(DefaultCommandHandler.java:83) |
|                 |     at com.github.ompc.Arthas.server.GaServer$4.run(GaServer.java:329)                                   |
|                 |     at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)                  |
|                 |     at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)                  |
|                 |     at java.lang.Thread.run(Thread.java:745)                                                            |
+-----------------+---------------------------------------------------------------------------------------------------------+
replay time fragment[1003] success.
Affect(row-cnt:1) cost in 3 ms.
$ 
```

你会发现结果虽然一样，但调用的路径发生了变化，有原来的程序发起变成了 Arthas 自己的内部线程发起的调用了。

- 需要强调的点

  1. **ThreadLocal 信息丢失**

     很多框架偷偷的将一些环境变量信息塞到了发起调用线程的 ThreadLocal 中，由于调用线程发生了变化，这些 ThreadLocal 线程信息无法通过 Arthas 保存，所以这些信息将会丢失。

     一些常见的 CASE 比如：鹰眼的 TraceId 等。

  2. **引用的对象**

     需要强调的是，`tt` 命令是将当前环境的对象引用保存起来，但仅仅也只能保存一个引用而已。如果方法内部对入参进行了变更，或者返回的对象经过了后续的处理，那么在 `tt` 查看的时候将无法看到当时最准确的值。这也是为什么 `watch` 命令存在的意义。

### 17. options

> 全局开关

| 名称               | 默认值 | 描述                                                         |
| ------------------ | ------ | ------------------------------------------------------------ |
| unsafe             | false  | 是否支持对系统级别的类进行增强，打开该开关可能导致把JVM搞挂，请慎重选择！ |
| dump               | false  | 是否支持被增强了的类dump到外部文件中，如果打开开关，class文件会被dump到`/${application dir}/arthas-class-dump/`目录下，具体位置详见控制台输出 |
| batch-re-transform | true   | 是否支持批量对匹配到的类执行retransform操作                  |
| json-format        | false  | 是否支持json化的输出                                         |
| disable-sub-class  | false  | 是否禁用子类匹配，默认在匹配目标类的时候会默认匹配到其子类，如果想精确匹配，可以关闭此开关 |
| debug-for-asm      | false  | 打印ASM相关的调试信息                                        |
| save-result        | false  | 是否打开执行结果存日志功能，打开之后所有命令的运行结果都将保存到`/home/admin/logs/arthas/arthas.log`中 |
| job-timeout        | 1d     | 异步后台任务的默认超时时间，超过这个时间，任务自动停止；比如设置 1d, 2h, 3m, 25s，分别代表天、小时、分、秒 |



#### 使用说明

例如，想打开执行结果存日志功能，输入如下命令即可：

```
$ options save-result true                                                                                         
 NAME         BEFORE-VALUE  AFTER-VALUE                                                                            
----------------------------------------                                                                           
 save-result  false         true
```