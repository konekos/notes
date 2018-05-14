#Java并发编程

### 单例模式

单例的7种写法。

### WaitSet

1. 所有的对象都会有一个WaitSet，用来存放调用了该对象wait方法后进入block状态的线程。
2. 线程被notify后，不一定立即得到执行
3. 线程从WaitSet被唤醒，唤醒顺序不一定是FIFO，
4. 线程被唤醒后，必须重新抢锁

### volatile

不能保证原子性，但能保证内存可见性，有序性。happens-before原则。

##### CPU,内存模型，
CPU  和其他硬件通讯通过总线：数据总线，地址总线，控制总线。


缓存不一致。![JVM内存模型](https://raw.githubusercontent.com/konekos/mds/master/picture/Snipaste_2018-02-25_20-58-39.png)

解决缓存不一致：
- 对总线加锁。串行的方式。性能下降。
- CPU高速缓存一致性协议MESI。intel提出的。核心思想：cpu写入数据的时候，若发现该变量被共享(在其他CPU存在该变量的副本),会发出一个信号，通知其他CPU该变量的缓存无效。

![MESI](https://raw.githubusercontent.com/konekos/mds/master/picture/Snipaste_2018-02-25_21-53-26.png)

##### 并发编程的3个重要概念
1. 原子性。保证一个或多个操作，要么都成功，要么都失败，中间不能由于任何因素失败。a=a+1 不满足。 所有基本类型赋值和读取都是原子性的。int x= 1;  Object o1 = o2;也是原子性。宽字word->4byte->int 赋地址值
2. 可见性。一个线程的修改，另一个线程要看到。
3. 有序性。代码有序执行。

*java的重排序：重排序保证最终一致性，中间的是不关心的（单线程）*

java是如何对上述保证实现的。
1. 原子性
对基本数据类型读取和赋值是保证原子性的，要么都成功，要么都失败，不可中断。```i=10,  cache=10, memory=10 ``` ``` b=a; 1.read a 2.assign b ``` ``` a++; 1.read a 2.add 3.assign ```

2. 可见性
使用volatile保证可见性。

3. 有序性
使用happens-before relationship。
- 代码执行顺序：编写在前面的发生在编写在后面的。a  a+1 ，先执行a
- 锁的原则：unlock必须发生在lock之后。
- volatile修饰的变量，对一个变量的写操作，先于对该变量的读操作。
- 传递规则：操作A先于B,B先于C，A肯定先于C
- 线程启动规则：start方法肯定先于线程run
- 线程的中断规则：interrupt必须发生在捕获该动作之前。
- 对象销毁规则：初始化必须发生在finalize之前。
- 线程的终结规则：所有的操作必须发生在线程死亡之前。

##### volatile关键字

一个共享变量被volitale修饰的意义
- 保证了不同进程的可见性
- 禁止重排序，保证有序性
- 并没有保证原子性

1. 保证重排序的时候不会把后面的指令放到屏障前面，也不会把前面的指令放到屏障后面。
2. 强制对缓存的修改理解写入内存
3. 如果是写操作，会导致其他cpu的缓存失效。

#### volatile使用场景
1.状态量标记。
```
//结束线程
volatile boolean start = true;
while(start){

}
void close(){
  start = false;
}
```
2. 屏障前后的一致性 double-check
```
volatile boolean init;
-------Thread1-------
//obj = createObj;
init = true;
-------Thread2-------
while(!init){
  sleep
}
```
总结；
1. cpu大致结构
2. JMM大致结构
3. 缓存一致性协议
4. 指令重排序
5. happens-before
6. 并发编程三大要素
7. volatile的作用和使用场景

### listener设计模式

事件源，主题。

### Single Threaded Execution design pattern

```
/**
 * SharedResource
 */
public class Gate {

    private int counter = 0;
    private String name = "nobody";
    private String address = "nowhere";

    /**
     * 临界值
     * @param name
     * @param address
     */
    public synchronized void pass(String name, String address) {
        this.counter++;
        //竞争 race
        this.name = name;
        this.address = address;
        verify();
    }

    private void verify() {

            if (this.name.charAt(0) != this.address.charAt(0)) {
                System.out.println("**********BROKEN***********" + toString());
            }
    }

    @Override
    public synchronized String toString() {
        return "No:" + counter + "  name:" + name + "  address:" + address;
    }
}
```

### Read-Write Lock

1. read read  并行化
2. read write 不允许
3. wirte write 不允许

| Lock| | |
|-|:-:|-:|
| |read|Write|
|read|no|yes|
|write|yes|yes|

### 不可变对象设计模式

1. 不可变对象一定是线程安全的。任何属性和引用类型的属性都不能修改。
2. 可变对象不一定线程不安全。如StringBuffer

servlet不是线程安全的。
struts1 Action不是线程安全的。
struts2 Action是线程安全的。每次请求创建1个action

条件
1. 没有setter
2. 所有值为private final
3. 不可继承 final修饰类

String对象，典型的不可变对象。String的所有操作，返回的值是一个新的值。String没有synchronized，传入的方法加锁。StringBuffer是一个线程安全的类，是可变的。StringBuilder是不加锁的，线程不安全的。

### Future

Future代表未来的一个凭据。
FutureTask 将你的调用逻辑进行了隔离。
FutureService 桥接future和futuretask

callback：加入回调。

### Guarded Suspension

Guarded Suspension意为保护暂停，其核心思想是仅当服务进程准备好时，才提供服务。设想一种场景，服务器可能会在很短时间内承受大量的客户端请求，客户端请求的数量可能超过服务器本身的即时处理能力，而服务端程序又不能丢弃任何一个客户请求。此时，最佳的处理方案莫过于让客户端要求进行排队，由服务端程序一个接一个处理。这样，既保证了所有的客户端请求均不丢失，同时也避免了服务器由于同时处理太多的请求而崩溃。

### The Thread-Specific Storage

线程保险箱。

##### ThreadLocal
设置相同的初始值，始终以当前线程作为key值，不同线程操作自身的threadlocal变量，而不会影响其他线程的变量。

### 上下文设计模式

线程安全的，且不用考虑锁。

使用ThreadLocal。

线程的上下文切换：存储和恢复cpu的状态的过程，使得线程执行可以从中断点恢复。

###### invoke一个方法（耗一定时间）
1. Future 提货单。
2. Suspension 挂起来。放入queue
以上两种借助了线程间的通讯。
不借助线程的通讯
↓↓↓
### Balking设计模式

当不适合这个操作，或者没有必要进行这个操作时，就直接放弃这个操作回去。

### 生产者消费者


### CountDown

CountDownLatch  使一个线程等待其他线程完成后再执行。
countDown await

### Thread-per-Message

一个请求一个线程来处理。可以使用线程池代替。

### Two-Phase Termination

telnet Localhost port

### Master-Worker Thread

流水线工人。

### Objects设计模式

## ClassLoader
