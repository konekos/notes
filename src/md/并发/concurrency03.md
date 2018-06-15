# concurrency包

## 一、原子类型

### 1. AtomicInteger

volitale: 可见性，顺序性sequence

AtomicInteger：保证原子性，可见性，有序性
```
public class JITTest {

    private static boolean init = false;


    public static void main(String[] args) throws InterruptedException {

        new Thread(){
            @Override
            public void run() {
                while (!init) {

                }
                //JIT优化如果什么都没写,等同于while(true)
                //如果写了 判断条件是!init
            }
        }.start();

        Thread.sleep(1000);

        new Thread(){

            @Override
            public void run() {
                init = true;

                System.out.println("Set init true");
            }
        }.start();
    }

}
```

lock free 无锁
CAS ：check and set。最快失败

### 2. AtomicBoolean
可以替代volitale

### 3. AtomicLong
32
long 64 原子性不能保证 VMSupportCS8()
high 32
low 32


## 二、Java并发包工具

### 1.CountDownLatch

一个线程等多个线程执行完毕。

实现了 AQS AbstractQueuedSynchronizer

串行过程中，如果其中一块可以并行，采用并行提高效率，然后再串行。采用latch

需要添加裁判员角色
### 2.CyclicBarrier

可循环栅栏。

多个线程同时等待，到达一个栅栏。

可以传入Runnable回调

不需要裁判员。

BrokenBarrierException：在等的时候，reSet。isBroken。

### 3. CountDownLatch和CyclicBarrier区别
1. cyclic可以reset，
2. Cyclic线程之间互不关心，共同到某一个点执行动作。
### 4.Lock&ReentrantLock

显示锁。

### 5.Executor

Executor和ExecutorService

参数

corePoolSize 一直是x个，空闲也是x个
maximumPoolSize 最大是y个
keepAliveTime 线程数量超过core，超过时间清掉多余线程
timeunit
blockingqueue submit都，交给queue，hold住，
threadfactory 通过什么方式创建线程
handler

queue满了，core--max。submit > queue+max 拒绝策略

停止线程池

**shutDown**
20个线程：不再submit，10个在工作，10个空闲，20个interrupt，工作的等待完成，空闲的退出。

**等待所有任务结束，关闭线程池。或超出时间**
shutdown();
awaitTermination(1, unit);

**等待一段时间关闭线程池**
awaitTermination(1, unit);
ThreadFactory 设置守护线程。

**shutDownNow**  打断所有 排干 返回queue
核心线程10个 队列10个 10个在running 10个stored in queue
return List<Runnable> remain unhandle runnable
运行的处理掉


##Executors
newCache  0,Max_integer SynchronousQueue
newFixed  core=max=n linkedBlockingQueue (n, factory)
newSingleThreadExecutor core=max=1,
newWorkStealingPool ForkJoinPool
Schedule

##ExecutorService API
isShutdown
isTermited
executeError factory setUncaughtExceptionHandler
executeRunnableTask  catch Throwable

#### 四大拒绝策略
AbortPolicy();  抛出RejectedExecutionException
DiscardPoicy(); 不理你，直接丢了
CallerRunsPolicy(); 谁提交的谁去run，main线程
DiscardOldestPolicy(); 把queue的等的最久的删掉，执行提交的。


## JUC

### ConcurrentHashMap
HashMap 线程不安全
Hashtable 线程安全  this锁

1.8的ConcurrentHashMap 和 HashMap性能差不多。
