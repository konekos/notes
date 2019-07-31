

# Java性能权威指南

## 第一章 导论

### 1.3 全面的性能调优

本书关注于如何最佳使用JVM和Java API。这只是高性能主题的一小部分。

#### 1.3.1 编写更好的算法

#### 1.3.2 编写更少的代码

#### 1.3.3 老调重弹的过早优化

![1542008018092](E:\studydyup\notes\src\pic\1542008018092.png)

#### 1.3.4 其他：数据库很可能就是瓶颈

不只JVM有bug和性能问题。

#### 1.3.5 常见的优化

- 借助性能分析优化代码，重点关注性能分析最耗时的操作。
- 奥卡姆剃刀原则诊断性能问题。
- 为应用最常用操作采取简单算法。



很多情况下，JVM只占整体性能的一小部分。

## 第二章 性能测试方法

### 2.1 原则1：测试真实应用

应该在产品实际使用的环境中进行性能测试。

#### 2.1.1 微基准测试

测量微小单元代码的性能。计算耗时。

![1542017208502](E:\studydyup\notes\src\pic\1542017208502.png)

存在的问题。

1. 必须使用被测的结果 d必须要被使用，不然就被编译器忽略代码。执行结果是![1542017313794](E:\studydyup\notes\src\pic\1542017313794.png)

   解决的办法是，确保读取被测结果。将局部变量l定位为实例变量（并用volitile声明）就能测了。![1542017417653](E:\studydyup\notes\src\pic\1542017417653.png)

2. 不要包含无关的操作

   ![1542017787845](E:\studydyup\notes\src\pic\1542017787845.png)

3. 必须输入合理的参数

   ![1542017830300](E:\studydyup\notes\src\pic\1542017830300.png)

综上正确的代码：

```java
package com.zhanghui.clicker;


import java.util.Random;

public class FibonacciTest {
    private volatile double l;
    private int nLoops;
    private int[] input;
    public static void main(String[] args) {

    }

    private FibonacciTest(int n) {
        nLoops = n;
        input = new int[nLoops];
        Random r = new Random();
        for (int i = 0; i < nLoops; i++) {
            input[i] = r.nextInt(100);
        }
    }

    private void doTest(boolean isWarmUp) {
        long then = System.currentTimeMillis();
        for (int i = 0; i < nLoops; i++) {
            l = fibImpl1(input[i]);
        }
        if (!isWarmUp) {
            long now = System.currentTimeMillis();
            System.out.println("Elapsed Time: " + (then - now));
        }
    }

    private double fibImpl1(int n) {
        if (n < 0) {
            throw new IllegalArgumentException("<0");
        }
        if (n == 0) {
            return 0d;
        }
        if (n == 1) {
            return 1d;
        }
        double d = fibImpl1(n - 2) + fibImpl1(n - 1);
        if (Double.isInfinite(d)) {
            throw new ArithmeticException("Overflow");
        }
        return d;
    }
}
```

微基准测试难编写。

#### 2.1.2 宏基准测试

#### 2.1.3 介基准测试

![1542103454901](E:\studydyup\notes\src\pic\1542103454901.png)

#### 2.1.4 代码实例

示例应用。计算股票某段时间内的史低和史高。

略。

### 2.2 原则2：理解批处理流逝时间，吞吐量和响应时间

#### 2.2.1 批处理流逝时间

测试应用性能最简单的方法，看它完成任务花的时间。

![1542182874663](E:\studydyup\notes\src\pic\1542182874663.png)

#### 2.2.2 吞吐量测试

吞吐量测试是基于一段时间内完成的工作量。

客户端—服务器吞吐量测试，不考虑客户端的思考时间，收到响应立即请求。测试结束，客户端报告操作总量。通常为每秒完成的操作量，不是测试期间的总量。指标通常为每秒事务数（TPS），每秒请求数（RPS），每秒操作次数（OPS）。

吞吐量测试也会报告请求平均响应时间，它的变化不表示性能有问题。

吞吐量测试通常适合热身期之后运行。

#### 2.2.3 响应时间测试

最后一个常用指标是响应时间：从客户端发送请求到收到响应之间的流逝时间。

响应时间测试和吞吐量测试的区别：响应时间测试客户端线程会在操作之间休眠一段时间。思考时间。模拟用户行为。

### 2.3 原则3：用统计方法应对性能的变化

性能测试结果随时间改变。多次测试，求平均。

因代码更改而进行的测试为回归测试。原先的代码为基线（baseline），新的代码为试样（specimen）

### 2.4 原则4：尽早频繁测试

遵循以下原则，使尽早频繁测试更有用。

- 自动化一切

  所有性能测试都应脚本化（或程序化）。全部环境必须通过脚本安装和配置新代码，脚本运行测试集。另外：脚本运行多次测试，结果t检验分析，生成置信度报告。

  测试运行前，自动化技术确保机器处于已知状态：检查是否有不希望的进程，操作系统配置是否正确等等。每轮运行保持相同环境，性能测试才可重复。

- 测试一切

  自动收集每一点数据，用于后续分析。数据包括整个运行过程采集的系统信息：CPU使用率，磁盘使用率，网络使用率，内存使用率等。还包括应用的日志——应用产生的，和垃圾收集器的。理想情况下，还应包括

  ![1542186172835](E:\studydyup\notes\src\pic\1542186172835.png)

- 真实系统上运行

  ![1542186237723](E:\studydyup\notes\src\pic\1542186237723.png)

### 2.5 小结

![1542186266754](E:\studydyup\notes\src\pic\1542186266754.png)

## 第三章 Java性能调优工具箱

性能分析过程一切都要能可视化。

### 3.1 操作系统的工具和分析

基于Unix的，有sar，如vmstat、iostat、prstat。Windows：typeperf。

至少收集CPU、磁盘、内存、网络（如需要）使用率。自动化测试，要依赖命令行工具。

#### 3.1.1 CPU使用率

CPU使用率可以分为2类：用户态时间和系统态时间（Windows上被称作为privileged time）。用户态时间是CPU执行应用代码所占时间的百分比，系统态时间是CPU执行内核代码所占时间的百分比。系统态时间与应用有关，比如I/O操作。

性能调优目的是，尽可能短时间内让Cpu使用率尽可能高。

首先注意，CPU使用率是一段时间内的平均数——5、30s或1s（最短时间）。

![1542190796603](E:\studydyup\notes\src\pic\1542190796603.png)

![1542190816460](E:\studydyup\notes\src\pic\1542190816460.png)

1. **Java和单CPU的使用率**
2. **Java和多CPU的使用率**

#### 3.1.2 CPU运行队列

![1544772920051](E:\studydyup\notes\src\pic\1544772920051.png)

