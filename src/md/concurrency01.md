# Java并发编程


### 写一个线程
### 线程的生命周期
1、Main函数是一个线程，JVM启动调用，名字为main。
2、实现一个线程必须创建Thread实例，override run方法，并start。
3、JVM启动时，有多个线程，但至少有一个不是守护线程。
4、当调用start方法，此时有两个线程，一个是调用你的线程，另一个是执行run方法的线程。
5、线程的生命周期为new，runnable，running,block,terminate。

![线程的生命周期](https://images0.cnblogs.com/blog2015/778067/201506/282029132836279.jpg)

### Runnable
逻辑和线程控制分离。
策略模式的使用。

### Thread对象的构造
1. 创建Thread，默认有一个线程名，Thread- 开头，从0计数。
Thread(): Thread-0，Thread1，Thread2……
2. Thread(Runnable target),如果构造时没有传入Runnable或者没有重写run方法，将不会调用任何东西。如果传递了Runnable的实例，或复写了run方法，将会执行该方法的逻辑单元。
3. 如果构造线程对象未传入ThreadGroup，会默认获取父线程的ThreadGroup作为ThreadGroup，此时子线程和父线程在同一个ThreadGroup。
4. 构造Thread时候，传入stacksize代表该线程占用的stack大小，如果没有指定stacksize大小，默认为0，0为忽略该参数，会被JNI函数使用。需要注意：该参数在一些平台有效，在有些平台（操作系统）无效。-Xss 10M

### 守护线程
在start之前设置
主线程死亡，守护线程也死亡。
如果被守护的线程打断，则守护线程生命周期结束。

### 线程优先级
并不易控制

### Thread join
join的线程执行完，再执行当前线程(main)。若有多个join，则多个并行执行完再执行当前线程。可以控制join的执行时间，默认为执行完。

### Interrupt
使thread.isInterrupted变为true，如果此时有thread.sleep(),Object.wait()，join()等使当前线程处于block时， 会抛出InterruptedException，并使打断标志重置false.

方法join(),join住的是main线程，打断main线程（调用该线程的线程）才会抛出InterruptedException

终止线程的两种方式。
定义一个volatile变量。 或 循环判断Thread.currentThread().isInterrupted()

案例：可控制时间打断线程，并可使线程正常提前结束。

```
public class ThreadService {

    private Thread executeThread;

    private boolean finished = false;

    public void execute(Runnable task) {
        executeThread = new Thread(){
            @Override
            public void run() {
                Thread runner = new Thread(task);
                runner.setDaemon(true);

                runner.start();

                try {
                    //使runner执行完再执行executeThread
                    runner.join();
                    //当runner死亡(executeThread被打断，守护线程runner死亡,或runner线程执行完毕。)，改变flag；
                    finished = true;
                } catch (InterruptedException e) {
                }
            }

        };

        executeThread.start();
    }

    public void shutdown(long mills) {
        long currentTime = System.currentTimeMillis();
        //循环检查flag
        while (!finished) {
            //如果超时了，则打断executeThread
            if (System.currentTimeMillis() - currentTime >= mills) {
                System.out.println("超时，需要结束");
                executeThread.interrupt();
                break;
            }
            //如果task没有完成，且没有超时，如果executeThread被打断，则因block而抛出异常跳出循环。
            try {
                executeThread.sleep(1);
            } catch (InterruptedException e) {
                System.out.println("执行线程被打断");
                break;
            }

        }
        //还原变量
        finished = false;
    }

    public boolean isFinished() {
        return finished;
    }
}
```
### 数据同步与synchronized

synchronized所包围的代码块变成了单线程，串行化。锁为传入的MONITER对象

synchronized同步方法，锁为this。若同步run方法只有一个线程在做，其他在等，对代码重构。synchronized加锁的范围要尽可能小。

```
public class SynchronizedRunnable1 implements Runnable{

    //readOnly
    private static final int MAX = 500;

    private int index = 1;

    private final Object MONITOR = new Object();

    @Override
    public  void run() {
        while (true) {
            if (ticket()) {
                break;
            }
        }
    }

//    private synchronized boolean ticket() {
//        //getField
//        if (index > MAX) {
//            return true;
//        }
//
//        try {
//            Thread.sleep(5);
//        } catch (InterruptedException e) {
//            e.printStackTrace();
//        }
//        //index++  getField ==>> index=index+，==>>  putField
//        System.out.println("current num is " + index++ + "，柜台为" + Thread.currentThread().getName());
//        return false;
//    }

    //与上面一样
    private  boolean ticket() {
        synchronized (this) {
            //getField
            if (index > MAX) {
                return true;
            }

            try {
                Thread.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //index++  getField ==>> index=index+，==>>  putField
            System.out.println("current num is " + index++ + "，柜台为" + Thread.currentThread().getName());
            return false;
        }
    }
}
```

t1和t2为同一个锁

```
public class SynchronizedThis {

    public static void main(String[] args) {
        ThisLock lock = new ThisLock();

        new Thread("t1"){
            @Override
            public void run() {
                lock.m1();
            }
        }.start();


        new Thread("t2"){
            @Override
            public void run() {
                lock.m2();
            }
        }.start();
    }


}

class ThisLock{


    public synchronized void m1(){
        try {
            System.out.println(Thread.currentThread().getName());
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public synchronized void m2(){
        try {
            System.out.println(Thread.currentThread().getName());
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
static代码块或方法的锁为该类.class

### 死锁

两个线程互相block住。
![死锁](https://images0.cnblogs.com/blog2015/759428/201508/252015315476760.png)


jps查看编号，jstack 编号进入。

### 线程间的通讯

wait(), notify();

##### 单线程的生产者消费者

```
public class ProducerConsumerVersion2 {

    private int i = 1;

    final private Object LOCK = new Object();

    private volatile boolean isProduced = true;

    private void produce() {
        synchronized(LOCK) {
            if (!isProduced) {
                System.out.println("p->" + (i++));
                //notify
                LOCK.notify();
                isProduced = true;

            }else {
                try {
                    LOCK.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }
    }

    private void consume() {
        synchronized (LOCK) {
            if (isProduced) {
                System.out.println("c->" + i);
                LOCK.notify();
                isProduced = false;
            }else {
                try {
                    LOCK.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }
    }
}
```

##### 多生产者消费者

notifyAll()，

```
public class ProducerConsumerVersion3 {

    private int i = 1;

    final private Object LOCK = new Object();

    private volatile boolean isProduced = true;

    private void produce() {
        synchronized (LOCK) {
            //使用while不用if：当notifyAll时，会导致多个生产者wait结束生产，会导致重复生产。
            while (isProduced) {
                try {
                    LOCK.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + "->" + (i++));
            //notify
            LOCK.notifyAll();
            isProduced = true;

        }
    }

    private void consume() {
        synchronized (LOCK) {

            while (!isProduced) {
                try {
                    LOCK.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + "->" + i);
            LOCK.notifyAll();
            isProduced = false;
        }
    }
  }
```
##### wait和sleep的区别

1. sleep是Thread的方法,wait是所有Object的方法。
2. sleep不会释放Object的monitor(LOCK), wait会释放锁并且把锁加入Object monitor waiting queue.
3. 使用sleep不需要锁（synchronized）,wait需要定义synchronized才能用，否则IlleagleMonitorException
4. sleep方法不需要被唤醒，但是wait需要被唤醒。


#### 案例CaptureService

```
public class CaptureService {

    private final static int MAX_WORKER = 5;

    final private static LinkedList CONTROLS = new LinkedList();

    public static void main(String[] args) {


        List<Thread> worker = new ArrayList<>();

        Arrays.asList("M1", "M2", "M3", "M4", "M5", "M6", "M7", "M8", "M9", "M10").stream()
                .map(CaptureService::createCaptureThread)
                .forEach(thread -> {
                    thread.start();
                    worker.add(thread);
                });

        worker.stream().forEach(thread -> {
            try {
                thread.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });


        Optional.of("All work finished").ifPresent(System.out::println);

    }

    private static Thread createCaptureThread(String name) {
        return new Thread(()->{

            Optional.of("The worker [" + Thread.currentThread().getName() + "] begin capture data.")
                    .ifPresent(System.out::println);

            synchronized (CONTROLS) {
                while (CONTROLS.size() > MAX_WORKER) {
                    try {
                        CONTROLS.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                CONTROLS.addLast(new Control());
            }

            Optional.of("The worker [" + Thread.currentThread().getName() + "] is working.")
                    .ifPresent(System.out::println);

            try {
                Thread.sleep(5_000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            synchronized (CONTROLS) {
                Optional.of("The worker [" + Thread.currentThread().getName() + "] capture done.")
                        .ifPresent(System.out::println);

                CONTROLS.removeFirst();
                CONTROLS.notifyAll();

            }

        }, name);
    }


    private static class Control {

    }
}
```

### 自定义显示LOCK

```
public interface Lock {

    class TimeOutException extends Exception {
        public TimeOutException(String message) {
            super(message);
        }
    }

    void lock() throws InterruptedException;

    void lock(long mills) throws InterruptedException,TimeOutException;

    void unlock();

    Collection<Thread> getBlockedThread();

    int getBlockedSize();

}
```

```
public class BooleanLock implements Lock {

    //init value is false, The status is free, while is true, it has been get;
    private boolean initValue;

    private Collection<Thread> blockedThreadCollection = new ArrayList<>();

    private Thread currentThread;

    public BooleanLock() {
        this.initValue = false;
    }

    @Override
    public synchronized void  lock() throws InterruptedException {

        while (initValue) {
            //观察
            blockedThreadCollection.add(Thread.currentThread());
            this.wait();
        }
        blockedThreadCollection.remove(Thread.currentThread());
        //表示已经被用了
        this.initValue = true;
        this.currentThread = Thread.currentThread();
    }

    @Override
    public synchronized void  lock(long mills) throws InterruptedException, TimeOutException {
        if (mills <= 0) {
            lock();
        }

        long hasRemaining = mills;
        long endTime = System.currentTimeMillis() + mills;

        while (initValue) {
            if (hasRemaining <= 0) {
                throw new TimeOutException("Time out");
            }
            blockedThreadCollection.add(Thread.currentThread());
            this.wait(mills);
            //
            hasRemaining = endTime - System.currentTimeMillis();

        }

        //表示已经被用了
        this.initValue = true;
        this.currentThread = Thread.currentThread();
    }

    @Override
    public synchronized void unlock() {

        if (Thread.currentThread() == currentThread){
            //释放
            this.initValue = false;
            System.out.println(Thread.currentThread().getName() + "release the lock monitor");
            this.notifyAll();
        }
    }

    @Override
    public Collection<Thread> getBlockedThread() {
        return Collections.unmodifiableCollection(blockedThreadCollection);
    }

    @Override
    public int getBlockedSize() {
        return blockedThreadCollection.size();
    }
}
```

```
public class LockTest {

    public static void main(String[] args) throws InterruptedException {

        final BooleanLock booleanLock = new BooleanLock();

        Stream.of("T1", "T2", "T3", "T4").forEach(
                name -> new Thread( () -> {
                    try {
//                        booleanLock.lock();
                        booleanLock.lock(1_000);
                        Optional.of(Thread.currentThread().getName() + "have the lock").ifPresent(System.out::println);
                        work();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }catch (Lock.TimeOutException e) {
                        Optional.of(Thread.currentThread().getName() + " Time Out").ifPresent(System.out::println);
                    }finally {
                        booleanLock.unlock();
                    }
                }, name).start()
        );

        Thread.sleep(1000);
        //有问题、哪个线程加的锁，谁释放。
        booleanLock.unlock();
    }

    private static void work() throws InterruptedException {

        Optional.of(Thread.currentThread().getName() + " is working").ifPresent(System.out::println);
        Thread.sleep(2_000);


    }
}
```

### ExitCapture

线程退出时，主动发信息。添加hook，RunTime.getRunTime().addShutdownHook(new Thread())。kill，结束线程还会完成其他任务。 kill -9强杀，不会完成其他任务。


```

```

### 获取线程运行时的Exception

线程不允许抛出异常,方法签名导致的。线程出了异常，只能在控制台或者log看。

thread.getStackTrace();

### ThreadGroup


### 线程池

1. 任务队列。
2. 拒绝策略（抛出异常，直接丢弃,阻塞，临时队列）
3. 初始化值init（min）
4. active
5. max

min <= active <= max

实现一个自己的线程池。
