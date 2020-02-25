# 第二章 线程安全性

Java 中的主要同步机制是 synchronized，它提供了一种独占的加锁方式，但“同步”这个术语还包括 volitile 类型的变量，显式锁以及原子变量。

>如果多个线程访问同一个可变的状态变量时没有使用合适的同步，有三种方式修复这个问题：
>
>- 不在线程之间共享该状态变量
>- 将该状态变量修改为不可变的变量。
>- 在访问状态变量时使用同步。

当设计线程安全的类时，良好的面向对象技术、不可修改性，以及清晰的不变性规范都能起到一定的帮助作用。

## 2.1 什么是线程安全性

当多个线程访问某个类时，这个类始终表现正确的行为，那么这个类就是线程安全的。

>无状态对象一定是线程安全的

## 2.2 原子性

```java
@NotThreadSafe
public class UnsafeCounting implement Servlet {
    public long count = 0;
    
    public long getCount() { return count; }
    
    public void service (ServletReqeust req, ServletResponse resp) {
        //...
        ++count;
    }
}
```

该类为统计 Servlet 执行次数，是非线程安全的类。

++count 不是原子性操作。包含三个独立的操作：读取 count，将其 +1，将计算结果写入 count。是一个“读取—修改—写入的操作序列”，结果状态依赖于之前的状态。

在并发编程中，这种由于不恰当的执行时序而出现不正确的结果是一种非常重要的情况，有一个正式的名字：**竞态条件**（Race Condition）。

### 2.2.1 竞态条件

`UnsafeCounting` 类存在多个竞态条件，结果不可靠。当某个计算的正确性取决于多个线程的交替执行时序时，就会发生竞态条件。就是结果取决于运气。最常见的静态条件是“Check-Then-Act 操作”。

### 2.2.2 延迟初始化中的竞态条件

```java
public class SingletonObject2 {
    /**
     * 懒汉式
     * 懒加载。
     * 多线程遇到的问题 可能不是单例 ，会有多个实例。
     * 解决方式: 加个class锁， 串行化 性能差
     */
    private  static SingletonObject2 instance;

    private SingletonObject2(){
        //empty
    }

    public  static SingletonObject2 getInstance() {
        if (null == instance) {
            instance = new SingletonObject2();
        }
        return SingletonObject2.instance;
    }
}
```

线程不安全的懒汉式。

使用带 volatile 的 Double-Check 保证线程安全。

```java
public class SingletonObject4 {
    /**
     * Double-check  单例 懒加载 解决性能问题。
     * 可能会引起空指针。 编译重排序， 一个线程初始化时，另一个线程判断不为null，去调用，会出现空指针。
     */
    private static volatile  SingletonObject4 instance;

    private SingletonObject4(){
        //empty
    }

    public  static SingletonObject4 getInstance() {
        if (null == instance) {
            synchronized (SingletonObject4.class) {
                if (null == instance)
                     instance = new SingletonObject4();
            }
        }
        return SingletonObject4.instance;
    }
}
```

### 2.2.3 复合操作

上面两个例子都需要一组以原子方式执行的操作（或说不可分割）。

`UnsafeCounting` 使用现有的  `AtomicLong` 原子类即可确保线程安全性。

## 2.3 加锁机制

```java
@NotThreadSafe
public class UnsafeCounting implement Servlet {
    public AtomicLong a = 0;
    public AtomicLong b = 0;
    
    public long getCount() { return count; }
    
    public void service (ServletReqeust req, ServletResponse resp) {
        int i = getFromReq(req);
        if(i==a.get){
        	b.set(i * 10);
        }
    }
}
```

a b两个原子变量，但先判断 a 再操作 b 并不是原子性操作，存在竞态条件。

> 要保证状态一致性，需要在单个原子操作中更新所有相关的状态变量。

### 2.3.1 内置锁

Java 提供内置的锁机制支持原子性：同步代码块。

每个 Java 对象都可以用做一个实现同步的锁，这些锁被称为内置锁或者监视器锁（Monitor Lock）。

### 2.3.2 重入

某个线程可以获得一个已经由他持有的锁，这个请求就会成功。重入意味着获取锁的操作的粒度是“线程”，而不是“调用”。重入的一种实现方法为，每一个锁关联一个所有者线程和获取计数器，再次获取值递增，退出时候递减。计数器为0时，释放锁。

## 2.4 用锁来保护状态

一种常见的错误是认为，只有在写入共享变量时才需要同步，然而事实并非如此（3.1）。

> 对于可能被多个线程同时访问的可变状态变量，在访问它时都需要持有同一个锁，在这种情况下，我们称状态变量是由这个锁保护的。

> 每一个共享和可变的变量都应用只由一个锁来保护，从而使维护人员知道是哪个锁。

## 2.5 活跃性与性能

同步可能使你的代码执行性能变得糟糕。

缩小同步块的范围可以对性能进行一定调节。

> 通常在简单性和性能之间存在相互制约因素。当实现某个同步策略时，一定不要盲目为了性能而牺牲简单性（可能破快安全性）。

> 当执行时间较长计算或者可能无法快速完成的操作时（如网络 I/O 或控制台 I/O），一定不要持有锁。



# 第三章 对象的共享

本章介绍如何共享和发布对象，从而使它们能安全地由多个线程同时访问。这两章合一起就形成了构建线程安全类以及通过 `java.util.concurrent` 类库来构建并发应用程序的重要基础。

我们已经知道了同步代码块可以确保以原子的方式执行操作，但一种常见的理解是，认为 synchronized 执行用于实现原子性或者确定“临界区（Critical Section）”。同步还有另外一个重要的方面：内存可见性（Memory Visibility）。

我们不仅防止某个线程正在使用对象状态时另外一个线程修改该状态，而且希望确保当一个线程修改对象状态后，其他线程能看到状态变化。

## 3.1 可见性

可见性是一种复杂的属性，可见性中的错误总是违背我们的直觉。为了确保多个线程之间对内存写入操作的可见性，必须使用同步机制。

```java
public class NoVisibility {
    private static boolean ready;
    private static int num;

    public static void main(String[] args) throws InterruptedException {
        new Thread(()->{
            while (!ready) {
                Thread.yield();
            }
            System.out.println(num);
        }).start();

        num = 42;
        ready = true;
    }
}
```

> 在没有同步的情况下，编译器、处理器以及运行时都可能对操作的执行顺序进行一些意想不到的调整。在缺乏足够同步的多线程程序中，要相对内存操作的执行顺序进行判断，几乎无法得出正确结论。

有一个简单的方法避免这些复杂的问题：只要有数据在多个线程之间共享，就使用正确的同步。

### 3.1.1 失效数据

`NoVisibility` 展示的在缺乏同步情况下产生的错误结果的一种情况：失效数据。当读线程查看ready变量时，可能会得到一个已经失效的值。

```java
@ThreadSafe
public class SynchronizedInteger {
    @GuardedBy("this") private  int value;

    public synchronized int getValue() {
        return value;
    }

    public synchronized void setValue(int value) {
        this.value = value;
    }
}
```

`SynchronizedInteger` 是一个线程安全的类。仅仅对 set 方法同步是不够的，调用 get 的线程仍然会看到失效值。

### 3.1.2 非原子的 64 位操作

当线程在没同步的情况下读取变量时，可能会得到一个失效值，但这个值至少是之前某个线程设置的值，而不是随机值。这种安全性保证也被成为最低安全性（out-of-thin-air-safety）。

最低安全性适用于绝大多数变量，但存在一个例外：非 volatile 类型的 64 位数值变量（double 和 long，@see 3.1.4）。Java 内存模型要求，内存的读取操作和写入操作都必须是原子操作，但对于非 volatile 类型的 long 和 double，JVM 允许将 64 位的读操作或者写操作分解为两个 32 位的操作。

因此，即使不计较数据失效的问题（读到曾经的值也可以），这种变量也是不安全的，除非用 volitile 声明或者加锁保护。

### 3.1.3 加锁和可见性

> 加锁的含义不仅仅局限于互斥行为，还包括内存可见性。为了确保所有线程都能看到共享变量的最新值，所有执行读操作或者写操作的线程都必须在同一个锁上同步。

### 3.1.4 Volatile 变量

Java 语言提供一种稍弱的同步机制，即 volatile 变量，用来确保将变量的更新操作通知到其他线程。当把变量生命为 volatile 类型后，编译器和运行时都会注意到这个变量是共享的，因此不会将该变量上的操作和其他内存操作一起重排序。volatile 变量不会被缓存在寄存器或者对其他处理器不可见的地方，因此读取 volatile 类型的变量时总会返回最新写入的值。

从内存可见性的角度看，写入 volatile 变量相当于退出同步代码块，读取 volatile 变量相当于进入同步代码块。

> volatile 变量能简化代码的实现以及对同步策略的验证时，才应该使用它们。

一个典型的例子：

```java
volatile boolean asleep;
...
 while(!asleep)
 	countSomeSheep();
```

> tip: 对于服务器应用程序，无论开发测试阶段，启动 JVM 时一定指定 -server 命令。

虽然 volatile 变量很方便，但也有局限性。volatile 语义不足以保证递增操作的原子性（count++），除非你可以确保只有一个线程对变量执行写操作。

> 加锁机制既可以确保可见性又能确保原子性，而 volatile 只能确保可见性。

当且晋档满足以下所有条件，才应该用 volatile。

- 对变量的写入操作不依赖变量的当前值（不能check-then-act），或者你能确保只有单个线程更新变量的值。
- 该变量不会和其他状态变量一起纳入不变性条件中。
- 在访问变量时不需要加锁。

## 3.2 发布和溢出

“发布（publish）”一个对象是指，使对象能够在当前作用域之外的代码中使用。例如，将一个该对象的引用保存到其他代码可以访问的地方，或者在没某一个非私有的方法中返回该引用，或者将该引用传递到其他类的方法中。

在许多情况下，我们要确保对象以及内部状态不被发布。某些情况下，又需要发布，则可能需要同步。

当某个不给发布的对象被发布时，这种情况就叫**逸出**（Escape）。

逸出的例子

```java
public static Set<Secret> knownSecrets;

public void init() {
	knownSecrets = new Hashset<Secret>;
}
```

`knownSecrets` 任何代码都可以访问。

```java
class UnsafeStates {
	private String[] states = new String[] {"AK","AL"};
	
	public String[] getStates() {return states;}
}
```

这种方式发布states，就会有问题。任何调用者都能修改这个数组的内容。states已经逸出了它所在的作用域，本应是私有的变量被发布了。

```java
public class ThisEscape {
	public ThisEscape(EventSource source) {
		source.registerListener(
			new EventListener() {
				public void onEvent(Event e){
					doSomething();
                    System.out.print(ThisEscape.this)
				}
			}
		);
	}
}
```

隐式的 this 逸出。在构造函数中创建内部类（内部类有外部类的this引用），并且在构造函数把 this 发布了出去（`source.registerListener()`）。

当且仅当构造器返回的时候，对象才是处于可预测的和一致的状态。因此当对象的构造函数中发布了该对象时，只发布了一个尚未构造完成的对象。即使发布对象的语句是构造函数最后一行也是一样的。

> 不要在构造过程中使 this 引用逸出。

如果想安全的发布，可以在私有的构造器内可以只初始化 Listener，实际的发布使用一个工厂方法。

```
public class SafeListener {
	private final EventListener listener;

	private SafeListener() {
			listener = new EventListener() {
				public void onEvent(Event e){
					doSomething();
                    System.out.print(ThisEscape.this)
				}
			}
	}
	
	public static SafeListener newInstance(EventSource source) {
		SafeListener safe = new SafeListener();
		source.registerListener(safe.listener);
		return safe;
	}
}
```

this 引用逸出的另一个常见错误，在构造函数启动一个线程。this 会被新创建的线程共享。可以在构造函数内不要启动线程。构造完用另外一个方法启动。

## 3.3 线程封闭

当访问共享的可变数据时，通常要同步。一种避免同步的方式就是不共享数据。如果仅仅在单线程内访问数据，就不需要同步。这种技术被称为线程封闭（Thread Confinement），是实现线程安全性的最简单方式之一。

一种常见应用 Swing，略。

另一种常见应用是 JDBC 的 Connection 对象。JDBC 规范并不要求 Connection 对象必须线程安全。应用程序提供的连接池是线程安全的。

Java 语言和核心库提供一些机制帮助维持线程封闭性，例如局部变量和 `ThreadLocal` 类。

### 3.3.1 Ad-hoc 线程封闭

Ad-hoc 线程封闭是指，维护线程封闭性的职责完全由程序实现来承担。

在 volatile 变量上存在一种特殊的线程封闭。只要你能确保只有单个线程对共享的 volatile 变量执行写入操作，在这种情况下，相当于将修改操作封闭在单个线程中防止发生竞态条件，并且 volatile 变量的可见性保证确保了其他线程看到最新的值。

是脆弱的，在程序中尽量少使用它，可能的情况下，应使用更强的线程封闭技术（例如栈封闭或 `ThreadLocal` ）。

### 3.3.2 栈封闭

栈封闭是线程封闭的一种特例，在栈封闭中，只能通过局部变量才能访问对象。

对于基本类型的局部变量，任何方法都无法获得对其引用，Java 语言的语义确保基本类型的局部变量始终封闭在线程内。

引用类型的变量，要确保不会逸出。

### 3.3.3 ThreadLocal 类

略。

## 3.4 不变性

满足同步需求的另一种方法是使用不可变对象（Immutable Object）。

对象在被创建后状态就不能修改，那么就是不可变对象。

> 不可变对象一定是线程安全的。

满足以下条件的对象是不可变对象。

> 对象创建后其状态就不能修改。
>
> 对象所有的 Field 都是 final 类型。
>
> 对象是正确创建的（对象创建期间，this 引用没有逸出）。

### 3.4.1 Final 域

Java 内存模型中，final 还有特殊的语义。final 可以确保初始化过程中的安全性，从而可以不受限制地访问不可变对象，并在共享这些对象时无需同步。

> 除非需要更高可见性，否则所有域都应该声明为 private。
>
> 除非某个域是可变的，否则应声明为 final。

### 3.4.2 示例：使用 Volatile 类型来发布不可变对象

不可变类。

```java
@Immutable
public class OneValueCache {
    private final BigInteger lastNumber;
    private final BigInteger[] lastFactors;

    public OneValueCache(BigInteger lastNumber, BigInteger[] lastFactors) {
        this.lastNumber = lastNumber;
        this.lastFactors = Arrays.copyOf(lastFactors,lastFactors.length);
    }

    public BigInteger[] getLastFactors(BigInteger i) {
        if (lastNumber == null || !lastNumber.equals(i)) {
            return null;
        }else {
            return Arrays.copyOf(lastFactors, lastFactors.length);
        }
    }

    @Override
    public String toString() {
        return "OneValueCache{" +
                "lastNumber=" + lastNumber +
                ", lastFactors=" + Arrays.toString(lastFactors) +
                '}';
    }
}
```

`getLastFactors` 方法，返回的是 copy 的对象。

```java
public class VolatileCachedFactorizer {
    // 保证可见性
    private volatile OneValueCache cache = new OneValueCache(null, new BigInteger[0]);

    public void service(long num) {
        BigInteger i = new BigInteger(String.valueOf(num));
        BigInteger[] factors = cache.getLastFactors(i);
        if (factors == null) {
            factors = factor(i);
            cache = new OneValueCache(i, factors);
        }
    }
```

方法里的 factors 是 cache 里 factors 在这时刻的复制，是局部变量，没有逸出。

对于在访问和更新多个相关变量时出现的竞态条件问题，可以通过将这些变量全部保存在一个不可变对象里来消除。

## 3.5 安全发布

到目前为止，我们重点讨论的是如何确保对象不被发布，例如让对象封闭在线程或另一个对象内部。当然如果要多个线程之间共享，必须确保安全地共享。

### 3.5.1 不正确的发布：正确的对象被破坏

你不能指望一个尚未完全创建的对象拥有完整性。

```java
public class Holder {
	private int n;
	public Holder(int n) {this.n = n;}
	
	public void assertSanity() {
		if (n!=n)
			throw new AssertionError("This statement is false");
	}
}
```

### 3.5.2 不可变对象与初始化安全性

不可变对象是一种非常重要的对象，Java 内存模型为不可变对象的共享提供了一种特殊的初始化安全保证。

即使在发布不可变对象引用时没有使用同步，也仍然可以安全地访问该对象。为了维持这种初始化安全的保证：必须满足不可变性所有要求：状态不可修改，所有域都是 final，以及正确的构造过程。

> 任何线程都可以在不需要额外同步的情况下安全地访问不可变对象，即使在发布这些对象时没有使用同步。

这种保证还延伸到正确创建对象中所有 final 类型的域。在没有额外同步的情况下，也可以安全地访问 final 的域。然而，如果 final 类型的域指向的是可变对象，那么访问这些域所指向的对象的状态时仍然需要同步。

### 3.5.3 安全发布的常用模式

> 要安全地发布一个对象，对象的引用以及对象的状态必须同时对线程可见，一个正确构造的对象可以通过以下方式来安全的发布：
>
> - 在静态初始化函数中初始化一个对象引用。
> - 将对象的引用保存到 volatile 类型的域 或者 `AtomicReference` 对象中。
> - 将对象的引用保存到某个正确构造对象的 final 类型域中。
> - 将对象的引用保存到一个由锁保护的域中。

```java
public static Holder holder = new Holder();
```

静态初始化器由 JVM 类在初始化阶段执行。由于在 JVM 内部存在同步机制，因此通过这种方式初始化的任何对象都可以被安全地发布[JLS 12.4.2]。

### 3.5.4 事实的不可变对象

如果对象从技术上看是可变的，但其状态在发布后不会改变，这种对象叫“事实不可变对象（Effectively Immutable Object）”。

> 在没有额外同步的情况下，任何线程都可以安全地使用**被安全发布**的事实不可变对象。

### 3.5.5 可变对象

如果对象在构造后可以修改，那么安全发布只能确保“发布当时状态的可见性”。对于可变对象，不仅发布对象时需要使用同步，在每次访问对象的时候也需要使用同步确保后继修改操作的可见性。

> 对象的发布需求取决于它的可变性：
>
> - 不可变对象可以通过任意机制发布。
> - 事实不可变对象必须通过安全方式来发布。
> - 可变对象必须通过安全方式来发布，并且必须是线程安全的或者由某个锁保护起来。

### 3.5.6 安全地共享对象

> 并发程序使用和共享对象时，可以使用一些使用的策略，包括：
>
> **线程封闭**。线程封闭的对象只能一个线程拥有。
>
> **只读共享**。共享的只读对象包括不可变对象和事实不可变对象。
>
> **线程安全共享**。线程安全的对象在其内部实现同步。
>
> **保护对象**。通过特定的锁来访问对象。

# 第四章 对象的组合

## 4.1 设计线程安全的类

> 设计线程安全类的过程中，需要包含以下3个基本要素：
>
> - 找出构成对象状态的所有变量。
> - 找出约束状态变量的不变性条件。
> - 建立对象状态的并发访问管理策略。

## 4.1.1 收集同步需求

> 如果不了解对象的不变性条件和后验条件，就不能确保线程安全性。要满足在状态变量的有效值或状态转换上的各种约束条件，就需要借助于原子性和封装性。

### 4.1.2 依赖状态的操作

如果在某个操作中包含基于状态的先驱条件，这个操作就成为依赖状态的操作。并发程序中，一定要等到先验条件为真才能执行该操作。

### 4.1.3 状态的所有权

垃圾回收机制使我们避免了如何处理对象所有权的问题。

许多情况下，所有权和封装性总是相互关联的：对象封装它拥有的状态，反正也成立，对它封装的状态拥有所有权。

## 4.2 实例封闭

> 将数据封装在对象内部，可以将数据的访问限制在对象的方法上，从而更容易确保线程在访问数据时总能持有正确的锁。

```java
@ThreadSafe
public class PersonSet {
    @GuardedBy("this")
    private final Set<Person> mySet = new HashSet<>();

    public synchronized void addPerson(Person p) {
        mySet.add(p);
    }

    public synchronized boolean containsPerson(Person p) {
        return mySet.contains(p);
    }
    
    private class Person{
        
    }
}
```

mySet 的操作使用 PersonSet 的对象锁，是线程安全的。如果 Person 对象不是线程安全的，那么还需要额外的同步。

> 封闭机制更易于构造线程安全的类，因为当封闭类的状态时，在分析类的线程安全性时就无须检查整个程序。

### 4.2.1 Java 监视器模式

Java 监视器模式主要优势在于简单性。

```java
@ThreadSafe
public class PersonSet {
    @GuardedBy("this")
    private final Set<Person> mySet = new HashSet<>();

    public synchronized void addPerson(Person p) {
        mySet.add(p);
    }

    public synchronized boolean containsPerson(Person p) {
        return mySet.contains(p);
    }

    private class Person{

    }
}
```

### 4.2.2 实例：车辆追踪

略

## 4.3 线程安全性的委托

大多数对象都是组合对象。从头构建一个类，或者将很多非线程安全的类组合为一个类时，Java 监视器模式是非常有用的。

### 4.3.1 示例：基于委托的车辆追踪器

Point

```java
@Immutable
public class Point {
    private final int x,y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}
```

Tracker

```java
@ThreadSafe
public class DelegatingVehicleTracker {
    private final ConcurrentMap<String, Point> locations;
    private final Map<String, Point> unmodifiableMap;

    public DelegatingVehicleTracker(ConcurrentMap<String, Point> points) {
        this.locations = new ConcurrentHashMap<>(points);
        this.unmodifiableMap = Collections.unmodifiableMap(locations);
    }

    public Map<String, Point> getLocations(){
        return unmodifiableMap;
    }

    public Point getLocation(String id) {
        return locations.get(id);
    }

    public void setLocations(String id, int x, int y) {
        if (locations.replace(id, new Point(x, y)) == null) {
            throw new IllegalArgumentException("invalid vehicle name: " + id);
        }
    }
}
```

这种实现方式是通过

### 4.3.2 独立的状态变量

我们还可以将线程安全性委托给多个状态变量，只要变量彼此独立。

```java
public class VisualComponent{
	private final List<KeyListener> keyListener = new CopyOnWriteArrarList<>();
	private final List<KeyListener> mouseListener = new CopyOnWriteArrarList<>();
}
```

键盘鼠标互不相干，用两个线程安全的列表管理监听器。

### 4.3.3 当委托失效时

大多数组合对象不像 `VisualComponent`一样简单：在它们的状态变量之间存在着某些不变性条件。

```java
public class NumberRange {

    // 不变性条件：lower <= upper
    private final AtomicInteger lower = new AtomicInteger();
    private final AtomicInteger upper = new AtomicInteger();

    public void setLower(int i) {
        // 注意————不安全的“先检查后执行”
        if (i > upper.get()) {
            throw new IllegalArgumentException("...");
        }
        lower.set(i);
    }

    public void setUpper(int i) {
        // 注意————不安全的先检查后执行
        if (i < lower.get()) {
            throw new IllegalArgumentException("...");
        }
        upper.set(i);
    }

    public boolean isInRange(int i) {
        return (i >= lower.get() && i <= upper.get());
    }
    
}
```

`NumberRange` 不是线程安全的，没有维持对下界和上界进行约束的不变性条件。没有使用加锁机制保证这些操作的原子性。

状态变量 lower 和 upper 不是彼此独立的，不能将线程安全委托为它的线程安全的状态变量。

> 如果一个类由多个独立且线程安全的状态变量组成，并且在所有的操作中都不包含无效状态转换，那么可以将线程安全性委托给底层的状态变量。

这个问题类似于 volatile 的变量规则：仅当一个变量参与到包含其它状态变量的不变性条件时，才能声明为 volatile 类型。

### 4.3.4 发布底层的状态变量

> 如果一个状态变量是线程安全的，并且没有任何不变性条件约束它的值，在变量的操作上也不存在任何不允许的状态转换，那么就可以安全地发布这个变量。

### 4.3.5 发布状态的车辆追踪器

`SafePoint`

```java
@ThreadSafe
public class SafePoint {
    @GuardedBy("this")private int x, y;

    public SafePoint(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    private SafePoint(int[] a) {
        this(a[0], a[1]);
    }
    
    public SafePoint(SafePoint p) {
        this(p.get());
    }
    
    public synchronized int[] get() {
        return new int[]{x, y};
    }

    public synchronized void set(int x, int y) {
        this.x = x;
        this.y = y;
    }
}
```

`Tracker`

```java
@ThreadSafe
public class PublishingVehicleTracker {
    private final Map<String, SafePoint> locations;
    private final Map<String, SafePoint> unmodifiableMap;

    public PublishingVehicleTracker(Map<String, SafePoint> safePoints) {
        this.locations = new ConcurrentHashMap<>(safePoints);
        this.unmodifiableMap = Collections.unmodifiableMap(locations);
    }

    public Map<String, SafePoint> getLocations(){
        return unmodifiableMap;
    }

    public SafePoint getLocation(String id) {
        return locations.get(id);
    }

    public void setLocations(String id, int x, int y) {
        if (locations.replace(id, new SafePoint(x, y)) == null) {
            throw new IllegalArgumentException("invalid vehicle name: " + id);
        }
    }
}
```

`PublishingVehicleTracker` 将线程安全性委托给底层的 ConcurrentHashMap，只是Map中元素是线程安全的，且是可变的（加了同步机制）。

`getLocation` 方法返回底层 Map 对象的一个不可变的副本。调用者不能增加或者删除车辆，但是可以拿到 Map 中的元素，即 `SafePoint` 来修改。

Map 这种“实时”特性究竟是好是坏，取决于实际需求。

## 4.4 在现有的线程安全类中添加功能

为 Vector 添加一个“若没有则添加“的功能。

```java
@ThreadSafe
public class BetterVector<T> extends Vector<T> {
    public synchronized boolean putIfAbsent(T e) {
        boolean absent = !contains(e);
        if (absent) {
            add(e);
        }
        return absent;
    }
}
// 因为 Vector 是 this 锁，所以 synchronized 通用使用 this 锁加在方法上
```

### 4.4.1 客户端加锁机制

另一个 putIfAbsent

```java
@ThreadSafe
public class ListHelper<E> {
    public List<E> list = Collections.synchronizedList(new ArrayList<>());

    public boolean putIfAbsent(E e) {
        // list 的锁是加在其对象自身上的，所以这个方法也要使用相同的锁。
        synchronized (list) {
            boolean absent = !list.contains(e);
            if (absent) {
                list.add(e);
            }
            return absent;
        }
    }
}
```

### 4.4.2 组合

当为现有的类添加一个原子操作时，有一种更好的方法: 组合。

```java
public class ImprovedList<T> implements List<T> {

    private final List<T> list;

    public ImprovedList(List<T> list) {
        this.list = list;
    }

    public synchronized boolean putIfAbsent(T t) {
        boolean absent = !list.contains(t);
        if (absent) {
            list.add(t);
        }
        return absent;
    }

    @Override
    public synchronized int size() {
        return list.size();
    }
    // 实现其他方法...
}
```

## 4.5 将同步策略文档化

> 在文档中说明客户代码需要了解的线程安全性保证，以及代码维护人员需要了解的同步策略。

略。

# 第五章 基础构建模块

委托是创建线程安全类的一个最有效的策略：只需让现有的线程安全类管理所有的状态即可。

本章将介绍一些最有用的并发构建模块，以及使用这些时的一些常用模式。

## 5.1 同步容器类

包括 Vector 和 Hashtable ，以及相似的用 Collections.synchronizedXxx 等工程方法创建的集合类。这些类实现线程安全的方式是：将状态封装起来，并对每个公有方法进行同步，使得每次只有一个线程能访问容器的状态。

### 5.1.1 同步容器的问题

同步容器类都是线程安全的，但某些情况仍需要额外的客户端加锁来保护复合操作。常见的复合操作：迭代（反复访问元素，直到遍历完容器所有元素）、跳转（根据指定顺序找到当前元素的下一个元素）、条件运算（如若没有则添加）。在同步容器类中，这些复合操作在没有客户端加锁的情况下**仍然是线程安全**的，但当其他线程并发地修改容器时，它们可能会出现意料之外的行为。

```java
public static Object getLast(Vector list){
	int loastIndex = list.size() - 1;
	return list.get(lastIndex);
}

public static Object deleteLast(Vector list){
	int loastIndex = list.size() - 1;
	return list.remove(lastIndex);
}
```

无论多少线程调用这两个方法，也不会破坏 Vector。但从调用者角度看，就不同了。如果 get 和 delete 线程交替执行，将可能会抛出 ArrayIndexOutOfBoundsException 异常。

同步容器类遵守同步策略，即客户端加锁的方式，如果要加新的方法的话，我们首先要知道它用的哪个锁，并给我们的新方法使用同一把锁就可以了。

```java
public static Object getLast(Vector list){
	synchronized (list) {
		int loastIndex = list.size() - 1;
		return list.get(lastIndex);
	}	
}

public static Object deleteLast(Vector list){
	synchronized (list) {
		int loastIndex = list.size() - 1;
		return list.remove(lastIndex);
	}
}

```

同理，迭代操作也可能抛出这个异常。同样可以通过客户端加锁的方式处理。

### 5.1.2 迭代器与 ConcurrentModificationException

无论在直接迭代还是在 for-each 语法中，对容器类的标准迭代方式都是使用 Iterator。如果有其他线程并发修改容器，即使是使用迭代器也无可避免在迭代期间加锁。

在设计同步容器类的迭代器时并没有考虑并发修改的问题，表现出的行为为 “及时失败”（fail-fast）。意味着在迭代过程被修改，就会抛出 ConcurrentModificationException。

 “及时失败”的迭代器并不是一种完备的处理机制，而只是“善意地”捕获并发错误，只能作为并发问题的预警器。实现方式是，将计数器的变化与容器关联起来：如果在迭代期间计数器被修改了，hasNext 或者 next 会抛出异常。（单线程中，当对象迭代时直接从容器删除而不是通过 Iterator.remove 删除时，也会抛出这个异常）。

如果要避免异常，就要迭代过程使用和容器相同的一把锁。

然而有时候并不想迭代期间对容器加锁，因为加锁意味着迭代时其他线程不能访问。那么另外一种替代方法就是“克隆”容器，在副本上迭代（克隆过程仍然需要加锁）。

### 5.1.3 隐藏迭代器

虽然加速可以防止迭代器抛出 ConcurrentModificationException，但要记住在所有对共享容器进行迭代的地方都需要加锁。

实际情况会更复杂，迭代器可能会隐藏起来。例如：

```java
public class HiddenIterator {
    @GuardedBy("this")
    private final Set<Integer> set = new HashSet<>();
    
    public synchronized void add(Integer i) {
        set.add(i);
    }

    public synchronized void remove(Integer i) {
        set.remove(i);
    }
    
    public void addTenString(){
        Random r = new Random();
        for (int i = 0; i < 10; i++) {
            add(r.nextInt());
        }
        System.out.println("DEBUG: added ten elements to " + set);
    }
}
```

在输出 DEBUG 信息的时候，set 的 toString 方法会进行迭代。当然，真正问题在于 `HiddenIterator` 不是线程安全的。在 print set 前必须获取锁，在打印日志的时候忽视了这个要求。

> 正如封装对象的状态有助于维持不变性条件一样，封装对象的同步机制同样有助于确保实施同步策略。

容器的 hashCode 和 equals 也会间接执行迭代。同样，containsAll、removeAll、retainAll 等容器作为参数的方法，也会进行迭代。都有可能抛出异常。

## 5.2 并发容器

Java 5.0 提供了多种并发容器来改进同步容器的性能。

如 ConcurrentHashMap。CopyOnWriteArrayList，用于遍历为主要操作的情况下代替同步的 List。

新的 ConcurrentMap 接口添加了一些复合操作的方法，如 putIfAbsent，replace，有条件删除等。

Java 5.0 新增两种新的容器类型：Queue 和 BlockingQueue。

Java 6 引入了 ConcurrentSkipListMap 和 ConcurrentSkipListSet ，作为同步的 SortedMap 和 SortedSet 的替代品（如 Collections 包装的 TreeMap 或者 TreeSet）。

### 5.2.1 ConcurrentHashMap

分段锁机制，在并发访问环境有更高吞吐量，而在单线程环境只损失非常小的性能。

ConcurrentHashMap 和其他并发容器一起增强了同步容器类：它们提供的迭代器不会抛出 ConcurrentModificationException，因此不需要迭代过程加锁。返回的迭代器具有弱一致性（Weakly Consistent），并非“即时失败（fast-fail）”。

尽管有这些改进，但仍然有一些需要权衡的因素。对于需要在整个 Map 上计算的操作，如 size 和 isEmpty，这些方法的语义被略微减弱了。返回的是一个近似值，

只有当应用程序需要加锁 Map 以进行独占访问时，才应该放弃使用 ConcurrentHashMap。（意思是 ConcurrentHashMap 的锁不是 Map 本身）

### 5.2.2 额外的原子 Map 操作

如 put-if-absent、remove-if-equal、replace-if-equal，ConcurrentMap 接口都已经有这些方法。

### 5.2.3 CopyOnWriteArrayList

用于替代同步List，某些情况有更好的并发性能，迭代期间不需要容器加锁或复制。

“写入时复制（Copy-On-Write）”容器的线程安全性在于，只要正确发布一个事实的不可变对象，访问该对象时就不需要进一步的同步。每次修改，都创建并重新发布一个新的容器副本，从而实现可变性。“写入时复制”容器返回的迭代器不会抛出 ConcurrentModificationException。

仅当迭代操作远多于修改操作时，才应该使用“写入时复制”容器。这个规则很好地描述了许多事件通知系统：在分发通知时需要迭代已注册监听器链表，并调用每一个监听器，在大多数情况下，注销和注册事件的监听器操作远少于接收事件通知的操作。

## 5.3 阻塞队列和生产者-消费者模式

阻塞队列提供了可阻塞的 put 和 take 方法，以及支持定时的 offer 和 poll 方法。队列满了，put 会阻塞到有空间可用；如果队列空，take 会阻塞到直到有元素可用。队列可以有界也可以无界。

阻塞队列支持生产者-消费者模式。常见组合是线程池与工作队列的组合。

> 构建高可靠的应用程序时，有界队列是更强大的资源管理工具：能防止产生更多的任务项，使应用程序在高负载的时候更健壮。

阻塞队列在许多情况下能使生产者-消费者模式更简单，如果阻塞队列不符合设计需求，还可以通过信号量（Semaphore）来创建其他的阻塞的数据结构。

LinkedBlockingQueue、ArrayBlockingQueue是 FIFO 队列，分别和 LinkedList 和 ArrayList 类似。PriorityBlockingQueue，有优先级的阻塞队列。SynchronousQueue，实际不是一个真正的队列，不会为队列中元素维护存储空间。它维护一组线程，等待把元素加入或者移除队列。仅当有足够多的消费者，且总是有一个消费者准备好获取任务时，才适合使用同步队列。

### 5.3.1 示例：桌面搜索

略

### 5.3.2 串行线程封闭

`java.util.concurrent` 中实现的各种阻塞队列都包含了足够的内部同步机制，从而安全地将对象从生产者线程发布到消费者线程。

线程封闭对象只能由单个线程拥有，但可以安全地发布该对象来“转移”所有权。

### 5.3.3 双端队列与工作密取

Java 6 新增两种容器类型，Deque 和 BlockingDeque，对 Queue 和 BlockingQueue 进行了扩展。Deque 是双端队列，实现了在队头和队列尾的高效插入和移除。实现有 ArrayDeque 和 LinkedBlockingDeque。

双端队列适用于另一种相关模式——工作窃取（Work Stealing）。工作窃取模式中，每个消费者都有各自的双端队列。如果一个消费者完成了自己双端队列里的工作，它可以从其他消费者的双端队列末尾窃取任务。降低了队列上的竞争程度。

工作窃取适用于既是消费者又是生产者问题——当执行某个工作时可能导致出现更多的工作。例如，爬虫、搜索图算法，垃圾回收阶段对堆标记，都可以用这种机制高效并行。当一个工作线程找到新的任务单元，它会将其放到自己队列的末尾（或者在工作共享设计模式中，放入其他工作线程的队列）。当双端队列为空，它会在另一个线程的队列队尾查找新的任务，确保每个线程都处于忙碌状态。

## 5.4 阻塞方法与中断方法

线程可能会阻塞或暂停执行，原因有：等待 I/O 操作结束，等待获取一个锁，等待从 sleep 醒来，或等待另一个线程的计算结果。线程阻塞时，通常被挂起，处于某种阻塞状态（BLOCKED、WAITING、TIMED_WAITING）。当某个事件发生时，线程被置回 RUNNABLE，并可再次被调度执行。

BlockingQueue 的 put 和 take 等方法会抛出受检异常——中断异常，这和其他类库的做法相同，比如 Thread.sleep。当方法抛出中断异常，表示该方法是一个阻塞方法，如果这个方法被中断，那么它会努力提前结束阻塞状态。

中断是一种协作机制。

处理中断异常。

- 传递。避开异常，包括不捕获抛出去或者捕获后再次抛出。
- 恢复中断。有时不能抛出中断异常，必须捕获，比如 Runnable 的 run 方法。捕获后调用线程的 interrupt 方法，恢复中断状态以避免屏蔽中断。

不可以捕获后什么也不做，这样会导致线程中断的证据丢失。see Chapter 7。

## 5.5 同步工具类

阻塞队列可以作为同步工具类，还包括信号量（Semaphore）、栅栏（Barrier）、以及闭锁（Latch）。

### 5.5.1 闭锁

闭锁是一种同步工具类，可以延迟线程的进度知道其到达终止状态。

闭锁的作用相当于一扇门：在闭锁到达结束状态之前，门一直关着的，没有任何线程通过，到达结束状态时，这扇门会打开并允许所有线程通过。

当闭锁到达结束状态后，将**不会再改变状态**，这扇门会永远保持打开的状态。闭锁用于确保某些活动直到其他活动都完成后才继续执行。

- 确保某个计算在所需要的所有资源都初始化后才开始执行。
- 确保某个服务在其依赖的所有服务都已经启动后才启动。
- 等待直到某个操作的所有参与者都就绪再继续执行。

CountDownLatch 是一种灵活的闭锁实现，可以使一个或者多个线程等待一组事件发生。闭锁状态包括一个计数器，countDown 方法递减计数器，表明一个事件发生了，await 方法等待计数器达到 0，表示所有需要等待的事件都要发生。如果计数器非零，await 会一直阻塞到计数器为 0，或者等待中的线程中断，或等待超时。

例子：TestHadness

```java
public class TestHarness {

    public long timeTasks(int nThreads, final Runnable task) throws InterruptedException{
        final CountDownLatch startGate = new CountDownLatch(1);
        final CountDownLatch endGate = new CountDownLatch(nThreads);

        for (int i = 0; i < nThreads; i++) {
            new Thread(()->{
                try {
                    startGate.await();
                    try {
                        task.run();
                    } finally {
                        endGate.countDown();
                    }
                } catch (InterruptedException e) {
                    // ignored
                }

            }).start();
        }

        long start = System.nanoTime();
        startGate.countDown();
        endGate.await();
        long end = System.nanoTime();
        return end - start;
    }
}
```

"起始门"和“结束门”

起始门用于确保所有线程都准备好执行 task，并同时开始执行任务。结束门用于在所有任务完成时计时。

### 5.5.2 FutureTask

FutrueTask 也可以用作闭锁，表示一种抽象的可生成结果的计算。

示例：使用 FutureTask 提前加载稍后需要的数据

```java
public class PreLoader {
    private final FutureTask<ProductInfo> future =
            new FutureTask<ProductInfo>(new Callable<ProductInfo>() {
                @Override
                public ProductInfo call() throws Exception {
                    return loadProductInfo();
                }
            });

    private final Thread thread = new Thread(future);
    // 不要在构造器里启动线程
    public void start(){
        thread.start();
    }
    
    public ProductInfo get() throws InterruptedException {
        try {
            return future.get();
        } catch (ExecutionException e) {
            Throwable cause = e.getCause();
            if (cause instanceof DataLoadException) {
                throw (DataLoadException) cause;
            } else {
                throw launderThrowable(cause);
            } 
        }
    }
    // 如果 Throwable 是 Error，那么抛出；如果是 RunTime 返回它；否则抛出 IllegalStateException
    private RuntimeException launderThrowable(Throwable t) {
        if (t instanceof RuntimeException) {
            return (RuntimeException) t;
        } else if (t instanceof Error) {
            throw (Error) t;
        } else {
            throw new IllegalStateException("Not checked", t);
        } 
    }


    private ProductInfo loadProductInfo() {
        if (1 == 1) {
            throw new DataLoadException();
        }
        return null;
    }
    
    public static class ProductInfo{
        
    }
    
    public static class DataLoadException extends RuntimeException{
        
    }
}
```

### 5.5.3 信号量

计数信号器（Counting Semaphore）用来控制同时访问某个特定资源的操作数量，或者同时执行某个指定操作的数量。计数信号量还可以用来实现某种资源池，或者对容器施加边界。

Semaphore 中管理一组虚拟的许可（permit），许可通过构造函数指定，执行操作时可以首先获取许可（只要有剩余），并在使用后释放许可。如果没许可，acquire 会阻塞到直到有许可（或被中断或超时）。release 返回一个许可给 Semaphore。

初始值为 1 的 Semaphore 可以用作互斥体（mutex），且不可重入：谁有这个许可，谁就持有这个互斥锁。

Semaphore 用于实现资源池，如数据库连接池。（12 章有界缓冲类使用这个技术）

同样，你可以用 Semaphore 将任何一种容器变成有界阻塞容器，例子：

```java
public class BoundedHashSet<T> {
    private final Set<T> set;
    private final Semaphore semaphore;

    public BoundedHashSet(int bound) {
        this.set = Collections.synchronizedSet(new HashSet<>());
        this.semaphore = new Semaphore(bound);
    }
    
    public boolean add(T o) throws InterruptedException {
        semaphore.acquire();
        boolean wasAdded = false;
        try {
            wasAdded = set.add(o);
            return wasAdded;
        }finally {
            if (!wasAdded) {
                semaphore.release();
            }
        }
    }

    public boolean remove(Object o) {
        boolean wasRemoved = set.remove(o);
        if (wasRemoved) {
            semaphore.release();
        }
        return wasRemoved;
    }
}
```

### 5.5.4 栅栏

我们通过闭锁启动一组相关的操作，或者等待一组相关的操作结束。闭锁是一次性对象，一旦进入终止状态，就不能被重置。

栅栏（Barrier）类似闭锁，它能阻塞一组线程直到某个事件发生。栅栏和闭锁的区别是：所有线程必须同时到达栅栏位置，才能继续执行。闭锁用于等待事件，而栅栏用于等待其他线程。栅栏用于实现一些协议，例如几个家庭决定在某个地方集合：“所有人 6:00 在麦当劳碰头，到了后要等其他人，之后讨论其他事情”。

CyclicBarrier 可以使一定数量的参与方反复地在栅栏处聚集，它在并行迭代算法中非常有用：这种算法通常将一个问题拆分成一系列相互独立的子问题。当线程到达栅栏位置时将调用 await，这个方法将阻塞直到所有线程都到达栅栏位置。所有线程都达到栅栏位置，栅栏打开线程释放，栅栏被重置以便下次用。await 超时或者被中断，则被认为打破了栅栏，所有的 await 都会终止并抛出 BrokenBarrierException。

示例：通过 CyclicBarrier 协调细胞自动衍生系统中的计算

```java
public class CellularAutomata {
    // 略
}
```

另一种形式的栅栏是 Exchanger，它是一种两方（Two-Party）栅栏，各方在栅栏位置上交换数据。当两方执行不对称的操作，Exchanger 会很有用，例如一个线程向缓冲区写入数据，另一个线程从缓冲区读取数据。

## 5.6 构建高效且可伸缩的结果缓存

几乎所有服务器都会使用某种形式的缓存。

例1：使用 HashMap 和同步机制来初始化缓存

缓存接口

```java
public interface Computable<A, V> {
    V compute(A arg) throws InterruptedException;
}
```

实现示例：

```java
public class ExpensiveFunction implements Computable<String, BigInteger> {
    @Override
    public BigInteger compute(String arg) throws InterruptedException {
        // 长时间计算
        return new BigInteger(arg);
    }
}
```

缓存器

```java
public class Memoizer1<A, V> implements Computable<A, V> {

    @GuardedBy("this")
    private final Map<A, V> cache = new HashMap<>();
    private final Computable<A,V> c;

    public Memoizer1(Computable<A,V> c) {
        this.c = c;
    }


    @Override
    public V compute(A arg) throws InterruptedException {
        V result = cache.get(arg);
        if (result == null) {
            result = c.compute(arg);
            cache.put(arg, result);
        }
        return result;
    }
}
```

能确保线程安全性，但带来明显的可伸缩性问题：每次只有一个线程能 compute，其他调用 compute 的线程可能被阻塞很久。

使用 ConcurrentHashMap 替代 HashMap

```java
public class Memoizer2<A, V> implements Computable<A, V> {

    private final Map<A, V> cache = new ConcurrentHashMap<>();
    private final Computable<A,V> c;

    public Memoizer2(Computable<A,V> c) {
        this.c = c;
    }


    @Override
    public V compute(A arg) throws InterruptedException {
        V result = cache.get(arg);
        if (result == null) {
            result = c.compute(arg);
            cache.put(arg, result);
        }
        return result;
    }
}
```

`Memoizer2` 的问题在于，如果某个线程启动了开销很大的计算，而其他线程并不知道这个计算正在运行，那么很可能重复这个运算，比如 X 线程在执行 f(27) 这个运算，这样当另一个线程查找 f(27) 时，他能知道最有效的办法是等待 X 结束，然后从缓存获取。

有一个类能基本实现这个功能：FutureTask。FutureTask 表示一个计算的过程，这个过程可能已经计算完成，也可能正在进行。如果有结果可用，那么 FutureTask.get 将立即返回结果，否则它会一直阻塞，直到结果计算出来再将其返回。

```java
public class Memoizer3<A, V> implements Computable<A, V> {

    private final Map<A, Future<V>> cache = new ConcurrentHashMap<>();
    private final Computable<A,V> c;

    public Memoizer3(Computable<A,V> c) {
        this.c = c;
    }


    @Override
    public V compute(A arg) throws InterruptedException {
        Future<V> f = cache.get(arg);
        if (f == null) {
            Callable<V> callable = new Callable<V>() {
                @Override
                public V call() throws Exception {
                    return c.compute(arg);
                }
            };

            FutureTask<V> ft = new FutureTask<>(callable);
            f = ft;
            cache.put(arg, ft);
            ft.run();
        }
        
        try {
            return f.get();
        } catch (ExecutionException e) {
            throw launderThrowable(e.getCause());
        }
    }
    
    private RuntimeException launderThrowable(Throwable t) {
        if (t instanceof RuntimeException) {
            return (RuntimeException) t;
        } else if (t instanceof Error) {
            throw (Error) t;
        } else {
            throw new IllegalStateException("Not checked", t);
        }
    }
}
```

Memoizer3 实现几乎完美：非常好的并发性，并且如果其他线程在计算，也会等待计算出来。只有一个缺陷，compute 的 if 属于“先检查后执行”, 是非原子性操作，但发生同时计算的概率要小得多。

使用 put-if-absent改进：

```java
public class Memoizer4<A, V> implements Computable<A, V> {

    private final Map<A, Future<V>> cache = new ConcurrentHashMap<>();
    private final Computable<A, V> c;

    public Memoizer4(Computable<A, V> c) {
        this.c = c;
    }


    @Override
    public V compute(A arg) throws InterruptedException {
        for (; ; ) {
            Future<V> f = cache.get(arg);
            if (f == null) {
                Callable<V> callable = new Callable<V>() {
                    @Override
                    public V call() throws Exception {
                        return c.compute(arg);
                    }
                };
                FutureTask<V> ft = new FutureTask<>(callable);
                f = cache.putIfAbsent(arg, ft);
                if (f == null) {
                    f = ft;
                    ft.run();
                }
            }

            try {
                return f.get();
            } catch (CancellationException e) {
                cache.remove(arg, f);
            } catch (ExecutionException e) {
                throw launderThrowable(e.getCause());
            }
        }

    }

    private RuntimeException launderThrowable(Throwable t) {
        if (t instanceof RuntimeException) {
            return (RuntimeException) t;
        } else if (t instanceof Error) {
            throw (Error) t;
        } else {
            throw new IllegalStateException("Not checked", t);
        }
    }
}
```

当缓存的是 Future  而不是值时，会导致缓存污染问题（Cache Pollution）：如果某个计算被取消或者失败，那么在计算这个结果时将指明计算过程被取消或者失败。

同样没有解决缓存过期的问题，以及缓存的清理问题。

解决第二章的缓存因式分解问题：

```java
@ThreadSafe
public class Factorizer {
    private final Computable<BigInteger, BigInteger[]> c =
            new Computable<BigInteger, BigInteger[]>() {
                @Override
                public BigInteger[] compute(BigInteger arg) throws InterruptedException {
                    return factor(arg);
                }
            };
    private final Computable<BigInteger, BigInteger[]> cache = new Memoizer4<>(c);

    public void service(ServletRequest req, ServletResponse resp) {
        try {
            BigInteger i = getFromReq(req);
            encodeIntoResponse(resp, cache.compute(i));
        } catch (InterruptedException e) {
            encodeError(resp, "interrupted");
        }
    }

    private void encodeError(ServletResponse resp, String interrupted) {
        // log
    }

    private void encodeIntoResponse(ServletResponse resp, BigInteger[] compute) {
        // resp
    }

    private BigInteger getFromReq(ServletRequest req) {
        return new BigInteger(req.getAttribute("num").toString());
    }


    private BigInteger[] factor(BigInteger arg) {
        // 计算
        return new BigInteger[0];
    }
}
```



## 第一部分小结

并发技巧清单

> - 可变状态是至关重要的
>
>   ​             所有的并发问题都可以归结为如何协调对并发状态的访问。可变状态越少，越容易确保线程安全性。
>
> - 尽量将域声明为 final 类型，除非需要它们可变
>
> - 不可变对象一定是线程安全的
>
>   ​             不可变对象能极大降低并发编程的复杂性，更简单安全，无需加锁和保护性复制。
>
> - 封装有助于管理复杂性
>
> - 用锁来保护每个可变变量
>
> - 当保护同一个不变性条件的所有变量时，要使用通一把锁
>
> - 在执行复合操作期间，要持有锁。
>
> - 多个线程访问同一个可变变量而且没有同步机制，程序会出问题。
>
> - 不要故作聪明地推断出不需要使用同步
>
> - 设计过程中考虑线程安全，或在文档中明确指出不是线程安全的。
>
> - 将同步策略文档化。

# 第六章 任务执行

## 6.1 在线程中执行任务

当围绕“任务执行”来设计应用程序结构时，第一步就是要找出清晰的任务边界。

### 6.1.1 串行地执行任务

最简单的策略就是在单个线程中串行地执行各项任务。

如：

```java
public class SingleThreadWebServer {
    public static void main(String[] args) throws IOException {
        ServerSocket socket = new ServerSocket(80);
        for (; ; ) {
            Socket connection = socket.accept();
            handleRequest(connection);
        }
    }

    private static void handleRequest(Socket connection) {
        // handle
    }
}
```

串行模型性能太差。

### 6.1.2 显式地为任务创建线程

为每个请求创建一个新的线程，实现更高响应性。

如：

```java
public class ThreadPerTaskWebServer {
    public static void main(String[] args) throws IOException {
        ServerSocket socket = new ServerSocket(80);
        for (; ; ) {
            final Socket connection = socket.accept();
            new Thread(()->handleRequest(connection)).start();
        }
    }

    private static void handleRequest(Socket connection) {
        // handle
    }
}
```

任务处理从主线程中分离了出来，任务可以并行处理，任务处理代码必须是线程安全的。

只要请求速率不超过服务器的负载能力，就能响应迅速。

但是不要这么做。

### 6.1.3 无限制创建线程的不足

生产环境“为每个任务创建一个线程”是有缺陷的，需要创建大量线程时：

- **线程生命周期的开销非常高**。线程的创建和销毁是有代价的。根据平台不同，实际开销也不同，但线程的创建过程都需要时间，延迟处理的请求，需要 JVM 和操作系统提供辅助操作。
- **资源消耗**。活跃的线程会消耗系统资源，尤其是内存。如果你有足够多的线程使 CPU 处于忙碌状态，创建更多线程反而降低性能。
- **稳定性**。在可创建线程的数量上存在一个限制。限制依平台而不同，且受多个因素制约，包括 JVM 启动参数、Thread 构造函数中请求的栈大小、操作系统的限制等。

## 6.2 Executor 框架

Executor 基于生产者-消费者模式。

### 实例：基于 Executor 的 Web 服务器

代码：

```java
public class TaskExecutionWebServer {

    private final static Executor exec = Executors.newFixedThreadPool(100);
    
    public static void main(String[] args) throws IOException {
        ServerSocket socket = new ServerSocket(80);
        for (; ; ) {
            final Socket connection = socket.accept();
            exec.execute(()->handleRequest(connection));
        }
    }

    private static void handleRequest(Socket connection) {
        // handle
    }
}
```

### 6.2.2 执行策略

当你看到 显示创建线程的代码时，并且你希望获得一种更灵活的执行策略时，考虑使用 Executor。

### 6.2.3 线程池

- newFixdTheadPool 长度固定
- newCachedThreadPool 线程池规模无限制
- newSingleThreadExecutor 单个线程，异常结束会创建新的 worker 替代。
- newScheduledThreadPool 类似 Timer

### 6.2.4 Executor 生命周期

ExecutorService 生命周期有 3 种状态：RUNNING、SHUNTDOWN 和 TERMINTED。

### 6.2.5 延迟任务和周期任务

Timer 有缺陷，要用 ScheduledThreadPoolExecutor 代替。

## 6.3 找出可利用的并行性

### 6.3.1 示例：串行的页面渲染器

略

### 6.3.2 携带结果的 Callable 和 Future

Callable 是一种更好的抽象：call 返回一个值，并且能抛出异常。

Future 表示一个任务的生命周期。Future 规范包含的隐含含义：任务的生命周期只能前进，不能后退。

get 方法的行为取决于任务的状态（未开始，在进行，已完成）。

ExecutorService 所有 submit 方法都返回 Future。

### 6.3.3 示例：使用 Future 的页面渲染器

渲染过程分为两个任务，一个是渲染所有文本（CPU 密集型），另一个是下载所有图像（I/O 密集型）。

Callable 和 Future 有助于表示这些协同任务之间的交互。

```java
public class FutureRenderer {
    private final ExecutorService executor = Executors.newFixedThreadPool(100);

    void renderPage(CharSequence source) {
        final List<ImageInfo> imageInfos = scanForImageInfo(source);

        Callable<List<ImageData>> callable = new Callable<List<ImageData>>() {

            @Override
            public List<ImageData> call() throws Exception {
                List<ImageData> imageData = new ArrayList<>();
                for (ImageInfo imageInfo : imageInfos) {
                    imageData.add(imageInfo.downloadImage());
                }
                return imageData;
            }
        };

        Future<List<ImageData>> future = executor.submit(callable);
        
        renderText(source);
        
        try {
            List<ImageData> imageData = future.get();
            for (ImageData imageDatum : imageData) {
                renderImage(imageDatum);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            future.cancel(true);
        } catch (ExecutionException e) {
            throw launderThrowable(e.getCause());
        }

    }
    private void renderText(CharSequence source) {

    }
    private RuntimeException launderThrowable(Throwable t) {
        if (t instanceof RuntimeException) {
            return (RuntimeException) t;
        } else if (t instanceof Error) {
            throw (Error) t;
        } else {
            throw new IllegalStateException("Not checked", t);
        }
    }

    private void renderImage(ImageData imageDatum) {

    }

    private List<ImageInfo> scanForImageInfo(CharSequence source) {
        return null;
    }

    private class ImageInfo {
        public ImageData downloadImage() {
            return null;
        }
    }

    private class ImageData {
    }
}
```

渲染文本和下载图像数据任务并发执行，所有图像下载完，显示到页面。

但这样还不够好，用户不必等到所有图像都下载完成，而是希望每下载完一张图就显示。

### 6.3.4 在异构任务并行化中存在的缺陷

虽然做了很多工作来并发执行异构任务提高并发度，但从中获得的并发性却十分有限（同 11.4.2 、11.4.3）

### 6.3.5 CompletionService: Executor 与 BlockingQueue

CompletionService 把 Executor 和 BlockingQueue 的功能融合在了一起。

### 6.3.6 示例： CompletionService 实现页面渲染器

代码：

```java
public class Renderer {
    private final ExecutorService executor;


    public Renderer(ExecutorService executor) {
        this.executor = executor;
    }

    void renderPage(CharSequence source) {
        final List<Renderer.ImageInfo> imageInfos = scanForImageInfo(source);
        CompletionService<ImageData> completionService = new ExecutorCompletionService<>(executor);
        for (ImageInfo imageInfo : imageInfos) {
            completionService.submit(new Callable<ImageData>() {
                @Override
                public ImageData call() throws Exception {
                    return imageInfo.downloadImage();
                }
            });
        }

        renderText(source);

        try {
            for (ImageInfo imageInfo : imageInfos) {
                Future<ImageData> future = completionService.take();
                ImageData imageData = future.get();
                renderImage(imageData);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } catch (ExecutionException e) {
            throw launderThrowable(e.getCause());
        }

    }

    private void renderText(CharSequence source) {

    }

    private RuntimeException launderThrowable(Throwable t) {
        if (t instanceof RuntimeException) {
            return (RuntimeException) t;
        } else if (t instanceof Error) {
            throw (Error) t;
        } else {
            throw new IllegalStateException("Not checked", t);
        }
    }

    private void renderImage(Renderer.ImageData imageDatum) {

    }

    private List<Renderer.ImageInfo> scanForImageInfo(CharSequence source) {
        return null;
    }

    private class ImageInfo {
        public Renderer.ImageData downloadImage() {
            return null;
        }
    }

    private class ImageData {
    }
}
```

### 6.3.7 为任务设置时限

略。

### 6.3.8 示例：旅行预订门户网站

略。

# 第七章 取消与关闭

任务和线程启动很容易。大多时候，我们希望它们运行到结束，或者自行停止。然而有时候又希望提前结束任务或线程，或许是因为用户取消了操作，也可能是应用程序需要被快速关闭。

要使任务和线程能安全、快速、可靠地停止下来，不是一件容易的事情。Java 没有提供任何机制来安全地终止线程。但是提供了中断（Interruption），这是一种协作机制，能够使一个线程终止另一个线程的当前工作。

这种协作式的方法是必要的，我们很少希望某个任务、线程或服务立即停止，因为这样会使共享的数据结构处于不一致的状态。

本章将给出各种实现取消和中断的机制，一级如何编写任务和服务。

## 7.1 任务取消

如果外部代码能在某个操作正常完成之前将其置入“完成”状态，那么这个操作就可以成为可取消的（Cancellable）。取消的原因有很多：

- 用户请求取消
- 有时间限制的操作
- 应用程序事件
- 错误
- 关闭

Java 没有安全的抢占式方法来停止线程，只有协作式的机制，使请求取消的任务和代码都遵循一种协商好的协议。

例：

```java
@ThreadSafe
public class PrimeGenerator implements Runnable{
    @GuardedBy("this")
    private final List<BigInteger> primes = new ArrayList<>();
    
    private volatile boolean cancelled;
    
    @Override
    public void run() {
        BigInteger p = BigInteger.ONE;
        while (!cancelled) {
            p = p.nextProbablePrime();
            synchronized (this) {
                primes.add(p);
            }
        }
    }
    
    public void cancel(){
        cancelled = true;
    }
    
    public synchronized List<BigInteger> get(){
        return new ArrayList<>(primes);
    }
}
```

通过 volatile 标记，跳出 while 循环。

一个可取消的任务必须拥有取消策略（Cancellation Policy），策略详细定义取消操作的 How、When、What。

### 7.1.1 中断

PrimeGenerator 的退出需要花费一定时间（一次循环）。如果在方法里调用了一个阻塞方法，很可能检查不到标志位而无法退出。

例如下面一种错误的代码示例：

```java
public class BrokenPrimeGenerator extends Thread {
    private final BlockingQueue<BigInteger> queue;
    private volatile boolean cancelled;

    public BrokenPrimeGenerator(BlockingQueue<BigInteger> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        try {
            BigInteger p = BigInteger.ONE;
            while (!cancelled) {
                queue.put(p = p.nextProbablePrime());
            }
        } catch (InterruptedException e) {
            // consumed
        }
    }
    
    public void cancel(){
        this.cancelled = true;
    }

    public static void main(String[] args) throws InterruptedException {
        ArrayBlockingQueue<BigInteger> queue = new ArrayBlockingQueue<>(10);
        BrokenPrimeGenerator generator = new BrokenPrimeGenerator(queue);
        generator.start();
        boolean needMorePrimes = true;

        try {
            while (needMorePrimes) {
                consume(queue.take());
            }
        } finally {
            generator.cancel();
        }
    }

    private static void consume(BigInteger take) {
        // consumer
    }
}
```

代码会导致生产者检测不到。

第五章提到过，一些特殊的阻塞库的方法支持中断。**线程中断是一种协作机制，线程可以通过这种机制通知另外一个线程，告诉它在合适的或者可能的情况下停止当前工作**，并转而执行其他工作。

> 在Java 的 API 或语言规范中，**并没有将中断和任何取消语义关联起来，但实际上，如果在取消之外的其他操作中使用中断，那么都是不合适的**，并且很难支撑更大的应用。

每个线程都有一个 boolean 类型的中断状态。中断线程时，线程的中断状态被设置 true。Thread API 中，interrupt 能中断线程，isInterrupted 返回中断状态。而静态的 interrupted 方法能清除当前线程的中断状态，并返回它之前的值，这也是清除中断状态的唯一方法。

阻塞库方法，例如 Thread.sleep 和 Object.wait 等，都会检查线程何时中断，并且在发现中断时提前返回。它们在响应中断时执行的操作包括：清除中断状态，抛出 InterruptedException，表示阻塞操作由于中断而提前结束。

线程在非阻塞状态下中断时，它的中断状态将被设置，然后根据将被取消的操作来检查中断状态来判断发生了中断。通过这样的方法，中断操作将变得“有粘性”——如果不触发 InterruptedException，那么中断状态会一直保持，直到明确地清楚中断状态。

> 调用 interrupt 方法并不意味着立即停止目标线程正在进行的工作，而只是传递了请求中断的消息。

中断正确理解：并不会真正中断一个正在运行的线程，只是发出中断请求，然后由线程在下一个合适的时刻（也叫取消点）中断自己。有些方法，如 wait、sleep、join 等，会严格处理这种请求，收到中断请求或者在开始执行时发现某个已经被设置好的中断状态时，将会抛出一个异常。而 LockSupport.park，就没有处理中断，只会看到线程停止阻塞。糟糕的方法可能会屏蔽中断请求。

使用静态的 interrupted 时应该小心，因为它会清除当前线程的中断状态。如果调用 interrupted 返回了 true，那么除非你想屏蔽这个中断，否则必须对它进行处理——可以抛出 InterruptedException，或者通过再次调用 interrupt 来恢复中断状态，如 5-10 所示。

> 通常，中断是实现取消的最合理的方式。

BrokenPrimeGenerator 的问题很容易解决（和简化）：使用中断而不是 boolean 标志位来取消。

```java
public class PrimeProducer extends Thread {

    private final BlockingQueue<BigInteger> queue;

    public PrimeProducer(BlockingQueue<BigInteger> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        try {
            BigInteger p = BigInteger.ONE;
            while (!Thread.currentThread().isInterrupted()) {
                queue.put(p = p.nextProbablePrime());
            }
        } catch (InterruptedException e) {
            // 允许线程退出
        }
    }

}
```

put 方法 以及 循环条件都会检查中断状态。

### 7.1.2 中断策略

正如任务应该包含取消策略一样，线程同样应该包含中断策略。中断策略规定线程如何解释某个中断请求——当发现中断请求时，应该做哪些工作（if necessary），哪些工作单元对于中断来说是原子操作，以及以多快的速度来响应中断。

最合理的中断策略是某种形式的线程级（Thread-Level）取消操作或服务级（Service-Level）取消操作：尽快退出，必要时清理，通知某个所有者该线程已经退出。

区分任何和线程对中断的反应是很重要的。

任务不会在其自己拥有的线程中执行，而是在某个服务（如线程池）拥有的线程中执行的。

这就是大部分可阻塞的库函数都只是抛出 InterruptedException 作为中断响应的原因。

当检查到中断请求，任务不需要放弃所有操作——可以推迟处理中断请求，直到某个更合适的时刻。

如果除了将 InterruptedException 传递给调用者外还需要执行其他操作，那么应该在捕获 InterruptedException 后恢复中断状态：`Thread.currentThread.interrupt();`

> 由于每个线程拥有各自的中断策略，因此除非你知道中断该线程的含义，否则就不要中断这个线程。

通过延迟中断请求的处理，开发人员能制定更灵活的中断策略，使程序合理权衡响应性和健壮性。

### 7.1.3 响应中断

在调用可中断的阻塞函数，如 Thread.sleep BlockingQueue 的 put 等，有两种实用策略可以处理 InterruptedException：

- 传递异常（可能在执行某个特定于任务的清除操作之后），从而使你自己的方法也成为可中断的阻塞方法。
- 恢复中断状态，从而使调用栈的上层代码能够对其进行处理

传递示例：

```java
BlockingQueue queue;
...
public Task getNextTask() throws InterruptedException {
	return queue.take();
}
```

如果不想或者无法传递 InterruptedException （如 Runnable），需要寻找另一种方式保存中断请求。

一种标准的方法是通过再次调用 interrupt 来恢复中断状态。你不能屏蔽掉 InterruptedException，例如catch 捕获了异常却不处理，除非你的代码实现了线程的中断策略。

> 只有实现了线程中断策略的代码才可以屏蔽中断请求。在常规的任务和库代码都不应该屏蔽中断请求。

对于一些不支持取消但仍可以调用可中断阻塞方法的操作，必须在循环里调用这些方法，在发现中断后重试。这种情况下，应该在本地保存中断状态，在返回前恢复中断状态而不是在捕获 InterruptedException 的时候，例如：

```java
public class UncancelableTask {

    private Task getNextTask(BlockingQueue<Task> queue) {

        boolean interrupted = false;

        try {
            while (true) {
                try {
                    return queue.take();
                } catch (InterruptedException e) {
                    interrupted = true;
                }
            }
        } finally {
            if (interrupted) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

如果过早设置中断状态，有无限循环的可能性，因为大多数可中断的阻塞方法都会在入口处检查中断状态，发现该状态被设置时立即抛出异常。

在取消过程中可能涉及中断状态之外的其他状态。

例如：当一个由 ThreadPoolExecutor 拥有的 Worker 线程检测到中断时，它会检查线程池是否关闭。如果是，它会在结束之前执行一些线程池清理工作，否则它可能创建一个新线程将线程池恢复到合理的规模。

### 7.1.4 示例：计时运行

许多问题永远无法解决，比如：枚举所有的素数，这样可以指定“最多花10分钟搜索答案”或者“枚举出10分钟内找到的答案”会很有用。

一个错误的示例：

```java
public class WrongInterrupt {
    private static final ScheduledExecutorService cancelExec = new ScheduledThreadPoolExecutor(1);

    public static void timedRun(Runnable r, long timeout, TimeUnit unit) {
        final Thread taskThread = Thread.currentThread();
        cancelExec.schedule(() -> taskThread.interrupt(), timeout, unit);
    }
    r.run();
}
```

这是最简单的方法，但是破坏了以下规则：**在中断线程前，应该了解它的中断策略**。timedRun 方法

在任意线程被调用，它无法得知该线程的中断策略。如果 r 在中断前就返回了，那么取消任务就会在返回后中断。这样的结果肯定是不好的。

而且，如果 r 不响应中断，timedRun 会在任务结束后才返回，可能已经超时了。

解决之前的问题：

```java
public class RightInterrupt {

    private static final ScheduledExecutorService cancelExec = new ScheduledThreadPoolExecutor(1);
    
    public static void timedRun(Runnable r, long timeout, TimeUnit unit) throws InterruptedException {
        class RethrowableTask implements Runnable {
            
            private volatile Throwable t;
            
            @Override
            public void run() {
                try {
                    r.run();
                } catch (Throwable e) {
                    this.t = e;
                }
            }
            
            void rethrow() {
                if (t != null) {
                    throw launderThrowable(t);
                }
            }
        }

        RethrowableTask task = new RethrowableTask();
        Thread taskThread = new Thread(task);
        taskThread.start();
        cancelExec.schedule(() -> taskThread.interrupt(), timeout, unit);
        taskThread.join(unit.toMillis(timeout));
        task.rethrow();
    }

    private static RuntimeException launderThrowable(Throwable t) {
        if (t instanceof RuntimeException) {
            return (RuntimeException) t;
        } else if (t instanceof Error) {
            throw (Error) t;
        } else {
            throw new IllegalStateException("Not checked", t);
        }
    }
}
```

### 7.1.5 通过 Future 来实现取消

另一个版本的 timedRun

```java
public class FutureTimeRun {

    private static final ExecutorService taskExec = Executors.newFixedThreadPool(1);
    
    public static void timedRun(Runnable r, long timeout, TimeUnit unit) throws InterruptedException {
        Future<?> future = taskExec.submit(r);

        try {
            future.get(timeout, unit);
        } catch (ExecutionException e) {
            // 任务取消
        } catch (TimeoutException e) {
            throw launderThrowable(e.getCause());
        }finally {
            future.cancel(true);
        }
        
        
    }

    private static RuntimeException launderThrowable(Throwable t) {
        if (t instanceof RuntimeException) {
            return (RuntimeException) t;
        } else if (t instanceof Error) {
            throw (Error) t;
        } else {
            throw new IllegalStateException("Not checked", t);
        }
    }
}
```

> 当 Future.get 抛出 InterruptedException 或 TimeoutException 时，如果你知道不再需要结果，那么可以直接调用 Future.cancel 来取消任务。

### 7.1.6 处理不可中断的阻塞

Java 库中，许多可阻塞的方法都是通过提前返回或抛出 InterruptedException 来响应中断请求的，从而使开发人员更容易构建出能响应取消请求的任务。然而，并非所有的可阻塞方法或阻塞机制都能响应中断：

- Java.io 包里的同步 Socket I/O。
- Java.io 包里的同步 I/O。
- Selector 的异步 I/O。
- 获取某个锁。如果一个线程由于等待获取某个内置锁而阻塞，那么无法响应中断。Lock 类可以使用 lockInterruptibly 方法在等待获取一个锁时仍然响应中断。

上面这些都是不会响应中断的。

示例程序，如何封装非标准的取消操作。既能处理标准的中断，也能关闭底层 Socket。

```java
public class ReaderThread extends Thread {
    private final Socket socket;
    private final InputStream in;

    public ReaderThread(Socket socket) throws IOException {
        this.socket = socket;
        this.in = socket.getInputStream();
    }
    
    @Override
    public void interrupt() {
        try {
            socket.close();
        } catch (IOException ignored) {
            
        }finally {
            super.interrupt();
        }
    }

    @Override
    public void run() {
        try {
            byte[] buf = new byte[1024];
            while (true) {
                int count = in.read(buf);
                if (count < 0) {
                    break;
                } else if (count > 0) {
                    processBuffer(buf, count);
                }
            }
        } catch (IOException e) {
            // 允许线程退出
        }
    }

    private void processBuffer(byte[] buf, int count) {
        
    }
}
```

### 7.1.7 使用 newTaskFor 来封装非标准的取消

我们可以通过 newTaskFor 方法来进一步优化上个例子中封装非标准取消的技术，这是 Java 6 在 ThreadPoolExecutor 中的新增功能。把 Callable submit 到 Executor，返回 Future， 通过 Future 来取消。newTaskFor 是一个工厂方法，它将创建 Future 来代表任务，返回一个 RunnableFuture 接口。

示例：

```java
public interface CancellableTask<T> extends Callable<T> {
    void cancel();
    RunnableFuture<T> newTask();
}

public class CancellingExecutor extends ThreadPoolExecutor {
    public CancellingExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    }

    @Override
    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        if (callable instanceof CancellableTask) {
            return ((CancellableTask<T>) callable).newTask();
        } else {
            return super.newTaskFor(callable);
        }
    }
}

abstract public class SocketUsingTask<T> implements CancellableTask<T> {
    @GuardedBy("this")
    private Socket socket;

    public synchronized void setSocket(Socket socket) {
        this.socket = socket;
    }

    @Override
    public synchronized void cancel(){
        try {
            if (socket != null) {
                socket.close();
            }
        } catch (IOException ignored) {

        }
    }

    @Override
    public RunnableFuture<T> newTask() {
        return new FutureTask<T>(this) {
            @Override
            public boolean cancel(boolean mayInterruptIfRunning) {
                try {
                    SocketUsingTask.this.cancel();
                }finally {
                    return super.cancel(mayInterruptIfRunning);
                }
            }
        };
    }
}
```

## 7.2 停止基于线程的服务

应用程序通常创建有多个线程的服务，如线程池，这些服务的生命周期通常比创建它们的方法的生命周期更长。如果应用程序准备退出，这些服务拥有的线程也需要结束。由于无法抢占式的方法停止线程，因此它们需要自行结束。

正确的封装原则是：除非拥有某个线程，否则不能对该线程进行操控。线程 API 中，没有线程所有权的正式定义：线程由 Thread 对象表示，像其他对象一样可以被自由共享。然而线程有一个相应的所有者，即创建该线程的类。因此线程池是其工作者线程的所有者，如果要中断这些线程，要用线程池中断。

与其他封装对象一样，线程的所有权不可传递：应用程序可以拥有服务，服务也可以拥有工作线程，但应用程序并不能拥有工作线程，所有应用程序不能直接停止工作者线程。

> 对于持有线程的服务，只要服务的存在时间大于创建线程的方法的存在时间，那么就应该提供生命周期方法。

### 7.2.1 示例：日志服务

不支持关闭的生产者-消费者日志服务：

```java
public class LogWriter {
    private final BlockingQueue<String> queue;
    private final LoggerThread logger;

    public LogWriter(PrintWriter writer) {
        this.queue = new LinkedBlockingQueue<>();
        this.logger = new LoggerThread(writer);
    }
    
    public void start() {
        logger.start();
    }
    
    public void log(String msg) throws InterruptedException {
        queue.put(msg);
    }

    private class LoggerThread extends Thread{
        private final PrintWriter writer;

        public LoggerThread(PrintWriter writer) {
            this.writer = writer;
        }

        @Override
        public void run() {
            try {
                while (true) {
                    writer.println(queue.take());
                }
            } catch (InterruptedException ignored) {
                
            } finally {
                writer.close();
            }
        }
    }
}
```

还需要实现一个终止日志线程的方法，从而避免无法正常关闭 JVM。要停止很容易，take 是可响应中断的。

然而仅仅使日志线程退出也是不完备的，会丢失等待写入的消息，不仅如此，其他线程将在调用 log 时被阻塞。

在取消一个生产者-消费者的模式时，需要同时取消生产者和消费者。在中断日志线程时会处理消费者，但此例中，生产者不是专门的线程，取消很困难。

另一种关闭 LogWriter 的做法是：设置一个“已请求关闭的标志”，避免进一步提交消息。

例如：

```java
public void log(String msg) throws InterruptedException {
		if(!shutdownRequested)
        	queue.put(msg);
        else
        	throw new IllegalStateException("...");
    }
```

然而这个方法存在竞态条件，是不可靠的。生产者此时发现该服务还没关闭，在 put 之前，服务却关闭了，一样有几率会导致生产者阻塞。

解决竞态条件的问题，要使日志提交为原子操作。由于 put 方法本身就可以阻塞，我们也不希望使用内置锁。我们采用原子方式检查关闭请求，并有条件地递增一个计数器来“保持”提交消息的权利。

例如： 

```java
public class LogService {
    private final BlockingQueue<String> queue;
    private final LoggerThread loggerThread;
    private final PrintWriter writer;

    public LogService(BlockingQueue<String> queue, LoggerThread loggerThread, PrintWriter writer) {
        this.queue = queue;
        this.loggerThread = loggerThread;
        this.writer = writer;
    }

    @GuardedBy("this")
    private boolean isShutdown;
    @GuardedBy("this")
    private int reservations;

    public void start() {
        loggerThread.start();
    }

    public void stop(){
        synchronized (this) {
            isShutdown = true;
            loggerThread.interrupt();
        }
    }

    public void log(String msg) throws InterruptedException {
        synchronized (this) {
            if (isShutdown) {
                throw new IllegalStateException("...");
            }
            ++reservations;
        }
        queue.put(msg);
    }

    private class LoggerThread extends Thread {
        @Override
        public void run() {
            try{
                while (true) {
                    try {
                        synchronized (this) {
                            if (isShutdown && reservations == 0) {
                                break;
                            }
                            String msg = queue.take();
                            synchronized (this) {
                                --reservations;
                            }
                            writer.println(msg);
                        }
                    } catch (InterruptedException e) {
                        // retry
                    }
                }
            }finally {
                writer.close();
            }
        }
    }
}
```

### 7.2.2 关闭 ExecutorService

shutdown 和 shutdownNow。

使用线程池的 LogService

```java
public class ExecutorLogService {
    private final ExecutorService exec;
    private static final long TIMEOUT = 10;
    private TimeUnit UNIT = TimeUnit.SECONDS;
    private final PrintWriter printWriter;
    private final BlockingQueue<Runnable> queue;
    private final WriterTask writerTask;

    public ExecutorLogService(PrintWriter printWriter, WriterTask writerTask) {
        this.queue = new ArrayBlockingQueue<Runnable>(10);
        this.exec = new ThreadPoolExecutor(1,1,0,TimeUnit.SECONDS, queue);
        this.printWriter = printWriter;
        this.writerTask = writerTask;
    }

    public void start(){}
    
    public void stop() throws InterruptedException {
        try{
            exec.shutdown();
            exec.awaitTermination(TIMEOUT, UNIT);
        }finally {
            printWriter.close();
        }
    }
    

    private class WriterTask implements Runnable{
        private final String msg;

        public WriterTask(String msg) {
            this.msg = msg;
        }

        @Override
        public void run() {
            printWriter.println(msg);
        }
    }
}
```

### 7.2.3 "毒丸"对象

另一种关闭生产者-消费者服务的方式就是使用“毒丸对象”：毒丸是指一个放在队列上的对象，含义是当得到这个对象，立即停止。

略

### 7.2.4 示例：只执行一次的服务

如果某个方法需要处理一批任务，当所有任务处理完成才返回，可以通过一个私有的 Executor 来简化服务的生命管理周期（invokeAll 和 invokeAny 通常有更大作用）。

例：

```java
public class CheckMail {
    public boolean checkMail(Set<String> hosts, long timeout, TimeUnit unit) throws InterruptedException {
        ExecutorService exec = Executors.newCachedThreadPool();
        final AtomicBoolean hasNewMail = new AtomicBoolean(false);
        
        try{
            for (String host : hosts) {
                exec.execute(()->{
                    if (checkMail(host)) {
                        hasNewMail.set(true);
                    }
                });
            }
        }finally {
            exec.shutdown();
            exec.awaitTermination(timeout, unit);
        }
        return hasNewMail.get();
    }

    private boolean checkMail(String host) {
        // check
        return false;
    }
}
```

### 7.2.5 shutdownNow 的局限性

取消正在执行的任务，返回所有已提交但未开始的任务。

但是我们无法通过常规方法找出哪些任务已经开始但尚未结束。这意味着我们无法在关闭过程中得知正在执行的任务的状态，除非任务自己做某些检查。

例子：在 ExecutorService 中跟踪在关闭后被取消的任务

```java
public class TrackingExecutor extends AbstractExecutorService {

    private final ExecutorService exec;
    private final Set<Runnable> tasksCancelledAtShutDown = Collections.synchronizedSet(new HashSet<>());

    public TrackingExecutor(ExecutorService exec) {
        this.exec = exec;
    }

    public List<Runnable> getTasksCancelledAtShutDown() {
        if (!exec.isTerminated()) {
            throw new IllegalStateException("...");
        }
        return new ArrayList<Runnable>(tasksCancelledAtShutDown);
    }

    @Override
    public void execute( Runnable runnable) {
        exec.execute(()->{
            try {
                runnable.run();
            } finally {
                if (isShutdown() && Thread.currentThread().isInterrupted()) {
                    tasksCancelledAtShutDown.add(runnable);
                }
            }
        });
    }
    
    @Override
    public void shutdown() {
        exec.shutdown();
    }

    @Override
    public List<Runnable> shutdownNow() {
        return exec.shutdownNow();
    }

    @Override
    public boolean isShutdown() {
        return exec.isShutdown();
    }

    @Override
    public boolean isTerminated() {
        return exec.isTerminated();
    }

    @Override
    public boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException {
        return exec.awaitTermination(timeout,unit);
    }
}
```

例2：使用 TrackingExecutorService 保存未完成的任务以备后续执行

```java
abstract public class TrackingExecutorService {
    private volatile TrackingExecutor exec;
    private static final long TIMEOUT = 10;
    private TimeUnit UNIT = TimeUnit.SECONDS;

    @GuardedBy("this")
    private final Set<URL> urlsToCrawl = new HashSet<>();

    public synchronized void start() {
        exec = new TrackingExecutor(Executors.newCachedThreadPool());
        for (URL url : urlsToCrawl) {
            submitCarwlTask(url);
        }
        urlsToCrawl.clear();
    }

    public synchronized void stop() throws InterruptedException {
        try {
            saveUncrawled(exec.shutdownNow());
            if (exec.awaitTermination(TIMEOUT, UNIT)) {
                saveUncrawled(exec.getTasksCancelledAtShutDown())
            }
        } finally {
            exec = null;
        }
    }

    protected abstract List<URL> processPage(URL url);

    private void saveUncrawled(List<Runnable> uncrawled) {
        for (Runnable runnable : uncrawled) {
            urlsToCrawl.add((((CrawlTask) runnable)).getUrl());
        }
    }

    private void submitCarwlTask(URL url) {
        exec.execute(new CrawlTask(url));
    }


    private class CrawlTask implements Runnable {

        private final URL url;

        public CrawlTask(URL url) {
            this.url = url;
        }

        @Override
        public void run() {
            for (URL link : processPage(url)) {
                if (Thread.currentThread().isInterrupted()) {
                    return;
                }
                submitCarwlTask(link);
            }
        }

        public URL getUrl() {
            return url;
        }
    }
}
```

## 7.3 处理非正常的线程终止

单线程控制台程序发生未捕获异常而终止时，程序会停止运行，并打印堆栈。

当线程故障时，应用程序可能看起来还在工作，这个线程的失败很可能会被忽略。幸运的是，我们有可以监测并防止在程序中“遗漏”线程的方法。

导致线程提前死亡的原因主要是 RuntimeException。通常不会被捕获，不会在调用栈中逐层传递，而是默认打印栈跟踪信息，并终止线程。

线程池的做法。

```java
public void run(){
	Throwable thrown = null;
	try{
		while(!isInterrupted)
			runTask(getTaskFromQueue);
	}catch (Throwable e){
		throw = e;
	}finally {
		processWorkerExit(this, thrown);
	}
}
```

**未捕获异常的处理**

除了主动解决 Unchecked 异常。Thread API 提供了 UncaughtExceptionHandler，能检测出某个线程由于未捕获异常而终结的情况。

一个线程未捕获异常退出时，JVM 会把这个事件报告给应用程序提供的 UncaughtExceptionHandler 异常处理器。如果没提供，默认将栈跟踪信息输出到 System.err。

> 在运行时间较长的应用程序中，通常会为所有线程的未捕获异常指定同一个异常处理器，并且该处理器至少会将异常信息记录到日志。

如果要为线程池的所有线程设置 UncaughtExceptionHandler，需要写在 ThreadFactory。如果你希望任务发生异常而失败获得通知，并执行一些操作，可以继承 ThreadPoolExecutor 实现 afterExecute 方法。

只有通过 execute 提交的任务，才能将抛出的异常交给异常处理器，如果是 submit 提交的，无论是受检异常还是 Unchecked 异常，都是任务返回状态的一部分，被封装在了 Future.get。

## 7.4 JVM 关闭

JVM 可以正常，也可以强行关闭。正常的方式有：最后一个非守护线程退出；或 System.exit。

### 7.4.1 关闭钩子（Shutdown Hook）

正常关闭时，JVM 首先调用所有注册的关闭钩子。钩子通过 Runtime.addShutdownHook 添加的。JVM 不保证钩子调用顺序。

关闭钩子应该是线程安全的：访问共享数据要使用同步机制，并避免死锁。

### 7.4.2 守护线程

你希望创建一个线程来执行一些辅助工作，又不希望它阻碍 JVM 关闭，这时就需要守护线程。

JVM 停止时，所有仍然存在的守护线程都被抛弃——不会执行 finally 和执行回卷栈，JVM 直接退出。

> 守护线程往往不能用来替代应用程序管理程序中各个服务的生命周期

### 7.4.3 终结器 finalize

不需要内存资源时，可以通过 GC 回收，对于其他资源，比如文件句柄或套接字句柄，必须显式地交还给操作系统。为实现这个功能，GC 会对实现了finalize 方法的对象特殊处理。

> 避免使用 finalize

## 小结

任务、线程、服务以及应用程序等模块的生命周期结束问题，可能会增加它们在设计和实现时的复杂性。Java 并没有提供某种抢占式的机制来取消操作或者终结线程。而是提供了一种协作式的中断机制来实现取消操作，但这要依赖于如何构建取消操作的协议，以及能否始终遵循这些协议。使用 FutureTask 和 Executor 框架，可以帮我们构建可取消的任务和服务。

# 第八章 线程池的使用

本章介绍对线程池进行配置和调优的高级选项，并分析在使用任务执行框架时需要注意的各种危险，以及一些使用 Executor 的高级示例。

## 8.1 在任务和执行策略之间的隐性耦合

虽然 Executor 框架为制定和修改执行策略都提供了很大的便利性，但并非所有的任务都能适用所有的执行策略。有些类型的任务需要需要明确指定执行策略，包括：

- **依赖性任务**。
- **使用线程封闭机制的任务**。单线程的 Executor 能够对并发性做出更强的承诺。它们能确保任务不会并发地执行，是你能放宽代码对线程安全的要求。这种情形将在任务和执行策略之间形成隐式耦合——任务执行必须使用单线程的 Executor。
- **对响应时间敏感的任务**。
- **使用 ThreadLocal 的任务**。只有当线程本地值的生命周期受限于任务的生命周期时，在线程池的线程中使用 ThreadLocal 才有意义，在线程池的线程中不应该使用 ThreadLocal 在任务间传递值。

只有任务是同类型的并相互独立时，线程池的性能才能达到最佳。

> 在一些任务中，需要拥有或排除某种特定的执行策略。如果某些任务依赖于其他的任务，那么会要求线程池足够大，确保不会被放入阻塞队列或被拒绝，如果是线程封闭机制的任务需要串行执行。

### 8.1.1 线程饥饿死锁

线程池中的任务如果依赖其他任务，那么可能产生死锁。单线程 Executor，如果一个任务将另一个任务提交到同一个 Executor，并且等待这个被提交任务的结果，那么通常会引发死锁。这种现象叫作线程饥饿死锁（Thread Starvation DeadLock）。

例：

```java
public class ThreadDeadLock {
    ExecutorService exec = Executors.newSingleThreadExecutor();
    
    public class RenderPageTask implements Callable<String> {

        @Override
        public String call() throws Exception {
            Future<String> header, footer;
            header = exec.submit(ThreadDeadLock.this::headerTask);
            footer = exec.submit(ThreadDeadLock.this::footerTask);
            String page = readBody();
            // 将发生死锁 —— 由于任务在等待子任务的结果
            return header + page + footer;
        }
    }

    private String readBody() {
        return null;
    }

    private String headerTask() {
        return null;
    }
    private String footerTask() {
        return null;
    }
}
```

> 每当提交了一个有依赖性的 Executor 任务时，要清楚地知道可能会出现线程“饥饿”死锁，因此需要在代码或配置 Executor 的配置文件中记录线程池的大小限制或配置限制。

### 8.1.2 运行时间较长的任务

如果任务阻塞的时间过长，即使不出现死锁，线程池的响应性也会变得糟糕。

有一项技术可以缓解这种影响，限定任务等待资源的时间，而不要无限制地等待。

## 8.2 设置线程池的大小

线程池的理想大小取决于被提交任务的类型以及所部署系统的特性。代码中通常不会固定线程池的大小，而应该通过某种配置机制来提供，或者根据 Runtime.availableProcessors 来动态计算。

幸运的是，设置线程池大小并不困难，只需要避免 “过大”和“过小”两种极端情况。

要想正确设置线程池大小，必须分析计算环境、资源预算和任务的特性。部署的系统有几个 CPU？内存多大？任务是计算密集型、I/O 密集型还是两者都有？它们是否需要像 JDBC 连接这样的稀缺资源？如果需要执行不同类别的任务，并且它们之间的行为相差很大，那么应该考虑使用多个线程池，从而使每个线程池可以根据各自的工作负载来调整。

对于计算密集型任务，在拥有 N 个 CPU 的系统上，线程池大小为 N + 1 时，通常实现最优的利用率。（即使当计算密集型线程偶尔由于页缺失故障或其他原因暂停时，这个 +1 的线程也可以确保 CPU 的时钟周期不被浪费。）对于包含 I/O操作或者其他阻塞操作的任务，由于线程并不会一直执行，因此线程池规模应该更大。

要正确设置线程池大小，必须估算出任务的等待时间与计算时间的比值。估算不需要很精确，并且可以通过一些分析或监控工具来获得。你还可以通过另一种方法调节线程池大小：在某个基准负载下，分别设置不同大小的线程池来运行应用程序，并观察 CPU 利用率水平。

给出以下定义：

N = 系统 CPU 数量

U = 目标 CPU 利用率，0 ≤ U ≤ 1

W/C = I/O 或阻塞等待的时间/CPU 执行时间

要使处理器达到期望使用率，线程池的最优大小等于：

Nt = N x U x (1 + W÷C)

CPU 周期并不是唯一影响线程池大小的资源，还包括内存、文件句柄、套接字句柄和数据库连接等等。计算这些资源对线程池的约束条件是更容易的：计算每个任务对该资源的需求量，然后用该资源的可用总量除以每个任务的需求量，结果就是线程池大小上限。

当任务需要某种通过资源池来管理的资源时，例如数据库连接，那么线程池和资源池大小会相互影响。如果每个任务都需要一个数据库连接，那么连接池的大小就限制了线程池的大小。同样，当线程池中的任务是数据库连接的唯一使用者时，线程池大小又会限制连接池的大小。

## 8.3 配置 ThreadPoolExecutor 

可以自定义配置线程池。

### 8.3.1 线程的创建和销毁

Core Pool Size，基本大小。Maximum Pool Size，最大大小。只有工作队列满时，才会超出基本大小。

### 8.3.2 管理队列任务

有限的线程池中会限制可并发执行的任务数量。（单线程的 Executor 是特例：可以通过线程封闭实现线程安全）。

基本的任务队列有 3 种：无界队列、有界队列和同步移交（Synchronous Handoff）。队列的选择与其他的配置参数有关，如线程池大小。

newFixed 和 newSingle 默认都是用无界队列。

一种稳妥的资源管理策略是使用有界队列，如 ArrayBlockingQueue、有界的 LinkedBlockingQueue、PriorityBlockingQueue。有界队列能避免无终止地往队列里加，导致 OOM。这样又有新的问题，队列被填满时，新的任务要怎么办（see 8.3.3）？使用有界队列，队列大小必须和线程池大小一起调节。

对于非常大的或者无界的线程池，可以使用 SynchronousQueue 来避免任务排队，直接将任务从生产者移交给工作者线程。newCached 就是使用的移交队列。

ArrayBlockingQueue 和 LinkedBlockingQueue 都是 FIFO 的，如果要控制执行顺序，可以使用 PriorityBlockingQueue，这个队列根据优先级安排任务。

> 对于 Executor，newCachedThreadPool 是一种很好的默认选择，可以提供比固定大小的线程池更好的排队性能。

只有任务相互独立，线程池或工作队列有界才是合理的。否则可能导致“线程饥饿死锁”问题。

### 8.3.3 饱和（拒绝）策略

有界队列满了后，饱和策略发挥作用（提交到已被关闭的 Executor 也会执行拒绝策略）。JDK 提供的实现有：AbortPolicy、CallerRunsPolicy、Discard Policy 和 DiscardOldestPolicy。

- Abort 默认的策略。抛出 Unchecked RejectedExecutionException。调用者可以捕获异常处理。
- Discard 悄悄地丢了，什么也不做。
- Discard-Oldest 抛弃下一个将被执行的任务，然后尝试重新提交新的任务。
- Caller-Runs 调用者执行实现了一种调节机制，不抛弃任务也不抛异常，而是把某些任务回退到调用者，降低新任务的流量。

使用 Semaphore 控制任务提交速率。

```java
@ThreadSafe
public class BoundedExecutor {
    private final Executor exec;
    private final Semaphore semaphore;

    public BoundedExecutor(Executor exec, Semaphore semaphore) {
        this.exec = exec;
        this.semaphore = semaphore;
    }

    public void submitTask(final Runnable task) throws InterruptedException {
        semaphore.acquire();

        try {
            exec.execute(()->{
                try{
                    task.run();
                }finally {
                    semaphore.release();
                }
            });
        } catch (RejectedExecutionException e) {
            semaphore.release();
        }
    }
}
```

### 8.3.4 ThreadFactory（线程工厂）

线程池每创建一个线程（Worker 里组合的 Thread 本尊），都是工厂方法来完成的。默认的 ThreadFactory 创建的新的非守护线程。

然而，很多情况下都需要使用定制的线程工厂方法。例如，你希望为线程池中的线程指定一个 UncaughtExceptionHandler，或守护线程，或者命名线程等。

ThreadFactory 接口

```java
public interface ThreadFactory {
	Thread newThread(Runnable r);
}
```

例：定制 Thread 基类

```java
public class MyAppThread extends Thread{
    private static final String DEFAULT_NAME = "MyAppThread";
    private static volatile boolean debugLifecycle = false;
    private static final AtomicInteger created = new AtomicInteger();
    private static final AtomicInteger alive = new AtomicInteger();
    private static final Logger log = LoggerFactory.getLogger(MyAppThread.class);

    public MyAppThread(Runnable r, String name) {
        super(r, name + "-" + created.incrementAndGet());
        setUncaughtExceptionHandler((thread,t)->log.error(thread.getName() + " " + t));
    }
    
    @Override
    public void run(){
        boolean debug = debugLifecycle;
        if (debug) {
            log.debug("Created " + getName());
        }
        try {
            alive.incrementAndGet();
            super.run();
        }finally {
            alive.decrementAndGet();
            if (debug) {
                log.debug("Exiting " + getName());
            }
        }
    }

    public static boolean isDebugLifecycle() {
        return debugLifecycle;
    }

    public static AtomicInteger getCreated() {
        return created;
    }

    public static AtomicInteger getAlive() {
        return alive;
    }

    public static void setDebugLifecycle(boolean debugLifecycle) {
        MyAppThread.debugLifecycle = debugLifecycle;
    }
}
```

### 8.3.5 在调用构造函数后再定制 ThreadPoolExecutor

ThreadPoolExecutor 类的 core，maximum，alive time，ThreadFactory，和 拒绝策略都是可以通过 Set 方法更改的 。

## 8.4 扩展 ThreadPoolExecutor

ThreadPoolExecutor 是可扩展的，提供了子类可以实现的空方法：beforeExecute、afterExecute 和 terminated。

可以通过这些方法添加日志、计时、监视或统计信息收集的功能。

示例：线程池添加统计信息。

```java
public class TimingThreadPool extends ThreadPoolExecutor {
    public TimingThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    }

    private final ThreadLocal<Long> startTime = new ThreadLocal<>();
    private final Logger log = LoggerFactory.getLogger(this.getClass());
    private final AtomicLong numTasks = new AtomicLong();
    private final AtomicLong totalTime = new AtomicLong();

    @Override
    protected void beforeExecute(Thread t, Runnable r) {
        super.beforeExecute(t, r);
        log.info("Thread {}: start {}",t,r);
        startTime.set(System.nanoTime());
    }

    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        try{
            long endTime = System.nanoTime();
            long taskTime = endTime - startTime.get();
            numTasks.incrementAndGet();
            totalTime.addAndGet(taskTime);
            log.info("Thread: {} end: {} time: {}", t, endTime, taskTime);
        }finally {
            super.afterExecute(r, t);
        }
    }

    @Override
    protected void terminated() {
        try {
            log.info("Terminated: avg time={}", totalTime.get()/numTasks.get());
        }finally {
            super.terminated();
        }
    }
}
```

## 8.5 递归算法的并行化

如果循环中的迭代操作都是独立的，并且不需要等待所有的迭代操作都完成再继续执行，那么就可以使用 Executor 将串行循环转换为并行循环，如：

```java
void processSequentially(List<Element> elements) {
	for (Element e: elements) {
		process(e);
	}
}

void processInParallel(Executor exec, List<Element> elements) {
	for (final Element e : elements) {
		exec.execute(()->process(e));
	}
}
```

如果需要一个任务集并等待它们完成，可以使用 ExecutorService.invokeAll，在任务都完成后获取结果。

当串行循环中的各个迭代操作之间彼此独立，每个迭代工作量操作执行的工作量比管理一个新任务时带来的开销更多，这个串行循环就适合并行化。

在一些递归设计中同样可以采用循环串行化的方法。在递归算法中通常会存在串行循环，这些循环可以如上面代码一样并行化。一种简单的情况是：每个迭代操作中都不需要来自后续递归迭代的结果。如下的 sequentialRecursive 用深度优先算法遍历一棵树，在每个节点上执行计算并将结果放入一个集合。

```java
public void sequentialRecursive(List<Node> nodes, Collection result) {
        for (Node node : nodes) {
            result.add(node.compute());
            sequentialRecursive(node.getChildren(),result);
        }
    }

    public void parallelRecursive(Executor exec, List<Node> nodes, Collection result) {
        for (Node node : nodes) {
            exec.execute(()->result.add(node.compute()));
            parallelRecursive(exec, node.getChildren(), result);
        }
    }
```

### 示例：谜题框架

略

# 第九章 图形用户界面应用程序

略

# 第十章 避免活跃性危险

在安全性和活跃性直接通常存在某种制衡。我们使用加锁机制来确保线程安全，但过度加锁，则可能导致锁顺序死锁（Lock-Ordering Deadlock）。同样，我们使用线程池和信号量来限制对资源的使用，这些限制行为可能导致资源死锁（Resource Deadlock）。Java 应用程序无法从死锁中恢复过来，因此设计时一定要排除可能导致死锁出现的条件。

本章介绍一些导致活跃性故障的问题，以及如何避免。

## 10.1 死锁

当线程 A 持有 L 锁的同时想获取锁 M，而线程 B 拥有 M 锁的同时想获取锁 L，这样两个线程就会永远等待下去。这就是最简单的死锁形式（或者叫抱死 Deadly Embrace）。多个线程可能存在成环的锁依赖导致死锁。

数据库系统的设计中考虑了检测死锁以及从死锁中恢复。数据库服务器不能让两个发生死锁的事务永远等待下去，当它检测到死锁（通过在表示等待关系的有向图中搜索循环）时，选择一个事务回滚。

JVM 在解决死锁上没有数据库服务那么强大。一组 Java 线程死锁的话，就会无限等待下去了。恢复的唯一方式是重启。

与其他的并发危险一样，死锁造成的影响很少会立即显现出来。一个类只是有可能造成死锁，不是每次都会发生。往往是高负载的时候发生。

### 10.1.1 锁顺序死锁

例：LeftRightDeadlock

```java
public class LeftRightDeadLock {
    private final Object left = new Object();
    private final Object right = new Object();

    public void leftRight() {
        synchronized (left) {
            synchronized (right) {
                doSomething();
            }
        }
    }

    public void rightLeft() {
        synchronized (right) {
            synchronized (left) {
                doSomething();
            }
        }
    }

    private void doSomething() {
        
        
    }
}
```

LeftRightDeadlock 存在死锁风险。发生死锁的原因是：两个线程试图以不同的顺序来获取相同的锁。

如果按照相同的顺序来请求锁，就不会出现循环的加锁依赖性，也就不会死锁。

> 如果所有线程都是以固定的顺序来获得锁，那么在程序中就不会出现锁顺序死锁的问题。

### 10.1.2 动态的锁顺序死锁

有时候，并不能清楚地知道是否在锁顺序上有足够的控制权来避免死锁的发生。

例：

```java
public class DynamicOrderingDeadLock {
    public void transferMoney(Account from, Account to, DollarAmount amount) throws InsufficientFundsException {
        synchronized (from) {
            synchronized (to) {
                if (from.getBalance().compareTo(amount) < 0) {
                    throw new InsufficientFundsException();
                } else {
                    from.debit(amount);
                    to.credit(amount);
                } 
            }
        }
    }

    private class Account {
        public DollarAmount getBalance() {
            return new DollarAmount();
        }

        public void debit(DollarAmount amount) {
            
        }

        public void credit(DollarAmount amount) {
            
        }
    }

    private class DollarAmount{

        public int compareTo(DollarAmount amount) {
            return 0;
        }
    }

    private class InsufficientFundsException extends Throwable {
    }
}
```

如果一个线程从 X 向 Y 转账，另一个从 Y 向 X 转账，就会死锁。要解决这个问题，必须定义锁的顺序，并在整个应用程序都按照这个顺序来获取锁。

制定锁的顺序时，可以使用 System.identityHashCode 方法，该方法将返回由Object.hashCode 返回的值。使用此值定义锁的顺序。

```java
private static final Object tieLock = new Object();
    public void transferMoney1(Account from, Account to, DollarAmount amount) throws InsufficientFundsException {
        class Helper{
            public void transfer() throws InsufficientFundsException {
                if (from.getBalance().compareTo(amount) < 0) {
                    throw new InsufficientFundsException();
                } else {
                    from.debit(amount);
                    to.credit(amount);
                }
            }
        }

        int fromHash = System.identityHashCode(from);
        int toHash = System.identityHashCode(to);

        if (fromHash < toHash) {
            synchronized (from) {
                synchronized (to) {
                    new Helper().transfer();
                }
            }
        } else if (fromHash > toHash) {
            synchronized (to) {
                synchronized (from) {
                    new Helper().transfer();
                }
            }
        } else {
            synchronized (tieLock) {
                synchronized (from) {
                    synchronized (to) {
                        new Helper().transfer();
                    }
                }
            }
        } 
    }
```

极少数情况下，两个对象的散列值会相同，此时也必须解决顺序问题，否则会发生死锁。这种情况下，可以加一个 tieLock（加时赛锁），消除死锁的可能性。

### 10.1.3 在协作对象之间发生的死锁

某些获取多个锁的操作并不像 LeftRightDeadlock 和 transferMoney 那么明显，这两个锁不一定在同个方法获取。

例：

```java
public class Taxi {
    @GuardedBy("this")
    private Point location, destination;
    private final Dispatcher dispatcher;

    public Taxi(Dispatcher dispatcher) {
        this.dispatcher = dispatcher;
    }
    
    
    public synchronized Point getLocation(){
        return location;
    }

    public synchronized void setLocation(Point location) {
        this.location = location;
        if (location.equals(destination)) {
            dispatcher.notifyAvailable(this);
        }
    }

    private class Point{
        
    }

    private class Dispatcher {
        @GuardedBy("this")
        private final Set<Taxi> taxis;
        @GuardedBy("this")
        private final Set<Taxi> availableTaxis;

        public Dispatcher() {
            this.taxis = new HashSet<>();
            this.availableTaxis = new HashSet<>();
        }

        private synchronized void notifyAvailable(Taxi taxi) {
            availableTaxis.add(taxi);
        }
        
        public synchronized Image getImage() {
            Image image = new Image();
            for (Taxi taxi : taxis) {
                image.drawMarker(taxi.getLocation());
            }
            return image;
        }
    }

    private class Image {
        public void drawMarker(Point location) {
        }
    }
}
```

虽然没有任何方法的代码获取了两个锁，但 setLocation 和 getLocation 等方法的调用者都会获得两个锁。有死锁的可能性。

这种情况下查找死锁是比较困难的：如果在持有锁的情况下调用某个外部方法，就需要警惕死锁。

> 如果在持有锁时调用某个外部方法，那么将会出现活跃性问题。在这个外部方法中可能会获取其他锁（这样会有死锁的可能），或者阻塞时间过长，导致其他线程无法及时获得当前被持有的锁。

### 10.1.4 开放调用

Taxi 和 Dispatcher 并不知道它们要陷入死锁。

如果在调用某个方法时不需要持有锁，那么这种调用被称为开放调用（Open Call）。依赖开放调用的类通常表现更好的行为，也比那些持有锁的类要好写。通过开放调用避免死锁的方法，类似于采用封装机制来提供线程安全的方法：对一个使用了封装的程序进行线程安全分析，比没有使用封装的要简单得多。同理，采用开发调用的形式比分析那些不依赖开放调用的要简单很多。

可以很容易将 Taxi 和 Dispatcher 改成开放调用，消除死锁的风险。

```java
@ThreadSafe
public class ThreadSafeTaxi {
    @GuardedBy("this")
    private Point location, destination;
    private final Dispatcher dispatcher;

    public ThreadSafeTaxi(Dispatcher dispatcher) {
        this.dispatcher = dispatcher;
    }

    public synchronized Point getLocation() {
        return location;
    }

    public void setLocation(Point location) {
        boolean reachedDestination;
        synchronized (this) {
            this.location = location;
            reachedDestination = location.equals(destination);
        }
        if (reachedDestination) {
            dispatcher.notifyAvailable(this);
        }
    }
    @ThreadSafe
    private class Dispatcher {

        @GuardedBy("this")
        private final Set<ThreadSafeTaxi> taxis;
        private final Set<ThreadSafeTaxi> availableTaxis;

        public Dispatcher(Set<ThreadSafeTaxi> taxis, Set<ThreadSafeTaxi> availableTaxis) {
            this.taxis = taxis;
            this.availableTaxis = availableTaxis;
        }

        public synchronized void notifyAvailable(ThreadSafeTaxi threadSafeTaxi) {
            availableTaxis.add(threadSafeTaxi);
        }
        
        public Image getImage() {
            Set<ThreadSafeTaxi> copy;
            synchronized (this) {
                copy = new HashSet<>(taxis);
            }
            Image image = new Image();
            for (ThreadSafeTaxi taxi : copy) {
                image.drawMarker(taxi.getLocation());
            }
            return image;
        }
    }


    private class Point{

    }

    private class Image {
        public void drawMarker(Point location) {
            
        }
    }
}
```

> 在程序中应尽量使用开放调用。与那些在持有锁时调用外部方法的程序相比，更易于对依赖于开放调用的程序进行死锁分析。

有时候重写编写同步代码块以使用开放调用的时候，可能会破坏操作的原子性。许多情况下，使某个操作失去原子性是可以接受的。

然而在某些情况下，丢失原子性可能会引发错误，这时候需要另一种技术实现原子性。例如，使的每次只有单个线程执行使用了开放调用的代码路径。

### 10.1.5 资源死锁

正如当多个线程相互持有彼此正在等待的锁而又不释放自己已持有的锁时会发生死锁，当它们在相同的资源集合上等待时，也会发生死锁。

另一种基于资源的死锁形式是线程饥饿死锁（see 8.1.1）。

## 10.2 死锁的避免与诊断

如果一个程序每次至多只能获得一个锁，那么就不会产生锁顺序死锁，当然这可能并不现实。如果需要获取多个锁，那就必须考虑锁的顺序：尽量减少潜在的加锁交互数量，将获取锁时需要遵循的协议写入正式文档并始终遵循这些协议。
在使用细粒度锁的程序中，可以通过一种两阶段策略（Two-Part Strategy）来检查代码中的死锁：首先找出在什么地方将获取多个锁，然后对这些实例进行全局分析，确保它们在整个程序中获取锁的顺序都保持一致。尽可能使用开放调用，这能极大简化分析过程。如果所有的调用都是开放调用，那么要发现获取多个锁的实例是非常简单的，可以通过代码审查、或者借助自动化的源代码分析工具实现。

### 10.2.1 支持定时的锁

使用 Lock 接口的定时 tryLock 功能来代替内置锁机制。指定一个超时时限（timeout），超过该时间后 tryLock 会返回一个失败信息。

定时锁失败时，你并不需要知道失败的原因。或许是发生了死锁，或许错误的进入无限循环，也可能执行时间远远超出你的预期。但是至少可以记录下此次超时的有用信息，而不会发生死锁这种必须重启解决的致命错误。

### 10.2.2 通过线程转储信息来分析死锁

虽然防止死锁是你自己的职责，但是 JVM 还是可以通过线程转储（Thread Dump）来帮助识别发生的死锁的。

线程转储包括各个运行中的线程的栈追踪信息，类似于异常时的栈追踪信息。还包含加锁信息，例如每个线程持有了哪些锁，在哪些栈帧中获得这些锁，以及被阻塞的线程正在等待获取哪一个锁。

在生成线程转储之前，JVM 将在等待关系图中通过搜索循环来找出死锁。如果发现了一个死锁，则获取相应的死锁信息，例如死锁涉及哪些锁和线程，以及锁获取操作位于程序的哪个位置。

Java 6 包含了对显示锁 Lock 相关转储信息和死锁检测的支持，但是比内置锁的信息精确度低。内置锁和获得它们所在线程的栈帧是关联的，Lock 只与获得它的线程相关联。

jstack 略

## 10.3 其他活跃性危险

尽管死锁是最常见的活跃性危险，并发程序中还存在一些其他的活跃性危险，包括：饥饿、丢失信号和活锁等。

### 10.3.1 饥饿

当线程由于无法访问它需要的资源而不能继续执行时，就发生了“饥饿（Starvation）”。引发饥饿最常见资源就是 CPU 时钟周期。如果 Java 程序中对优先级使用不当，就有可能发生饥饿。

Thread API 中线程优先级只是作为线程调度的参考。

操作系统的线程调度器会尽力提供公平的、活跃性良好的调度，甚至超出 Java 语言规范的需求范围。

> 要避免使用线程优先级，因为这会增加平台依赖性，并可能导致活跃性问题。在大多数并发应用程序中，都可以使用默认的线程优先级。

### 10.3.2 糟糕的响应性

除饥饿外的另一个问题是糟糕的响应性，如果在 GUI 程序中使用了后台线程，那么这种问题是很常见的。CPU 密集型的后台任务会与事件线程竞争 CPU 时钟周期，对响应性造成影响。

### 10.3.3 活锁

活锁（LiveLock）是另一种形式的活跃性问题，该问题尽管不会阻塞线程，但也不能继续执行，因为程序将不断重复执行相同的操作，而且总会失败。

活锁常发生在处理事务消息的应用程序中：如果不能成功处理某个消息，那么消息机制将回滚整个事务，并将它重新放到队列的开头。因为这次消息一直失败，所以还会一直被反复执行。虽然处理消息的线程没有阻塞，但也无法继续执行下去。这种形式的活锁通常是由于过度的错误恢复代码造成的，因为它错误地将不可修复的错误作为可修复的错误。

当多个相互协作的线程都对彼此进行响应从而修改各自的状态，并使得任何一个线程都无法继续执行时，就发生了活锁。

要解决这种活锁问题，需要在重试机制中引入随机性。

## 小结

活跃性故障是一个非常严重的问题，因为发生这种故障时，除了中止应用程序外没有任何机制可以帮助从这种故障中恢复过来。最常见的活跃性故障是锁顺序死锁。在设计的时候应该避免产生锁顺序死锁：确保线程在获取多个锁时采用一致的顺序。最好的解决方法是程序中始终使用开放调用。这将大大减少需要多个同时持有多个锁的地方，也更容易发现这些地方。

# 第十一章 性能与可伸缩性

线程的最主要目的是提高程序的运行性能。线程可以使程序更充分发挥系统的可用处理能力，从而提高系统的资源利用率。此外，线程还能在程序运行现有任务时立即开始处理新的任务，提高系统响应性。

本章将介绍各种分析、监测以及提升并发程序性能的技术。然而，许多提升性能的技术同样会增加复杂性，因此也就增加了在安全性和活跃性上发生失败的风险。虽然我们希望获取更好的性能——但始终要把安全性放在第一位。设计并发应用程序时，最重要的考虑因素通常并不是将程序的性能提升至极限。

## 11.1 对性能的思考

提升性能意味着用更少的资源做更多的事情。“资源”含义很广。对于一个给定操作，通常会缺乏某种特定的资源，如 CPU 时钟周期、内存、网络带宽、I/O 带宽、数据库请求、磁盘空间以及其他资源。当操作性能由于某种特定的资源而受到限制时，通常将这个操作称为资源密集型操作，如 CPU 密集型，数据库密集型。

尽管使用多个线程的目标是提升整体性能，但与单线程相比，使用多个线程总会引入一些额外的性能开销。包括：线程之间的协调（例如加锁、触发信号以及内存同步等），增加的上下文切换，线程的销毁与创建，以及线程的调度等。过度使用线程，这些开销甚至会超过由于提高吞吐量、响应性或计算能力带来的性能提升。如果并发设计太糟糕，性能甚至比不上功能相同的串行程序。

要通过并发来获得更好的性能，需要努力做好两件事情：更有效地利用现有处理资源，以及在出现新的处理资源时使程序尽可能地利用这些新资源。从性能监视的视角看，CPU 需要尽可能保持忙碌状态。如果是计算密集型，可以通过增加处理器来提高性能。如果程序不能使 CPU 处于忙碌状态，那么增加再多处理器也没用。

### 11.1.1 性能和可伸缩性

应用程序性能可以采用多个指标来衡量，例如服务时间、延迟时间、吞吐率、效率、可伸缩性以及容量等。其中一些指标（服务时间、等待时间）用于衡量程序的“运行速度”，即某个指定的任务单元需要“多快”才能处理完成。另一些指标（生产量、吞吐量）用于程序的“处理能力”，即在计算资源一定的情况下，多完成“多少”任务。

> 可伸缩性是指：当增加计算资源时（例如 CPU、内存、存储容量或 I/O 带宽），程序的吞吐量或者处理能力能相应地增加。

并发应用程序中针对可伸缩性进行设计和调整时所采用的方法与传统的性能调优方法截然不同。进行传统性能调优时，目地是用更小的代价完成相同的工作，如使用缓存，优化算法复杂度等。而进行伸缩性调优时，目的是设法将问题的计算并行化，从而能利用更多的计算资源来完成更多的工作。

性能的两个方面——“多快”和“多少”完全是独立的，有时甚至是相互矛盾的。要实现高伸缩性或硬件利用率，通常会增加各个任务要处理的工作量，例如把任务分解为多个“流水线”子任务时。讽刺的是，大多数提高单线程程序性能的技术，往往都会破快可伸缩性（see 11.4.4）。

我们熟悉的三层模型，是彼此独立的，就说明了提高可伸缩性造成性能损失的原因。

对于服务器应该程序来说，“多少”这个方面——可伸缩性、吞吐量和生产量，往往比“多快”这个方面更受重视。（在交互式程序，延迟或许更叫重要，这样用户就不用等待）。本章将重点介绍可伸缩性而不是单线程程序的性能。

### 11.1.2 评估各种性能权衡因素

几乎所有的工程决策中都会涉及某种形式的权衡，包括软件工程。例如，“快排”在大数据集执行效率高，但是对于小数据集，“冒泡排序”实际更高效。编写库排序算法的开放人员不会知道使用这个库的人的需求信息，这是大多数优化措施不成熟的原因之一：它们通常无法获取一组明确的需求。

> 避免不成熟的优化。首先使程序正确，然后再提高运行速度——如果它还运行地不够快。

进行决策时，有时候会通过增加某种形式的成本来降低另一种形式的开销（例如，增加内存使用量以降低服务时间），也会增加开销来换取安全性。安全性并不一定指对人身安全的威胁。很多性能优化措施通常是牺牲可读性或可维护性为代价——代码越“聪明”或越“晦涩”，就越难理解和维护。有时候，优化措施会破坏面向对象的设计原则，例如需要打破封装，有时候它们又会带来更高的错误风险，因为通常越快的算法就越复杂。（如果你无法找出其中的代价或风险，那么或许还没有对这些优化措施进行彻底的思考和分析）。

大多数性能决策中都包含多个变量，并且非常依赖于运行环境。在使某个方案比其他方案“更快”之前，首先问自己一些问题：

- “更快”的含义是什么？
- 该方法在什么条件下运行得更快？低负载还是高负载？大数据集还是小数据集？能否通过测试结果验证你的结论？
- 这些条件在运行环境中的发生频率？能否通过测试结果验证你的结论？
- 在其他不同条件的环境中能否使用这里的代码？
- 在实现这种性能提升时需要付出那些隐含的代价，例如增加开发风险或维护开销？这种权衡是否合适？

在进行任何与性能相关的决策时，都应该考虑这些问题，本书只介绍并发性方面的内容。我们为什么推荐这种保守的优化方法？对性能的提示可能是并发错误的最大来源。有人认为同步机制“太慢”，因而采用一些看似聪明实则危险的方法来减少同步的使用（see 16.2.4 double check），这也通常作为不遵循同步规则的一个常见借口。然而并发错误是最难追踪和消除的错误，所有对引入这类错误的任何可能，都需要谨慎实施。

更糟的是，你的初衷可能是用安全性来换取性能，但最终可能什么都得不到。特别是提到并发时，许多开发人员对哪些地方存在性能问题，哪种方法的运行速度更快，以及哪种方法的可伸缩性更改，往往存在错误的直觉。因此，性能调优时，一定要有明确的性能需求（这样才知道什么时候需要调优，什么时候应该停止），此外还需要一个测试程序以及真实的配置和负载等环境。调优后，再次测试以验证预期。

> 以测试为基准，不要猜测

## 11.2 Amdahl（阿姆达尔） 定律

在有些问题中，如果可用资源越多，那么问题的解决速度就越快。例如，如果参与收割庄稼的工人越多，那么就能越快完成收割工作。而有些任务本质是串行的，例如作物生长速度。如果使用线程主要是为了发挥多个处理器的处理能力，那么就必须对问题就行合理的并行分解，使程序能有效地使用这种潜在的并行能力。

大多数并发程序都和农业耕作有很多相似之处，都是由一系列的并行工作和串行工作组成的。Amdahl 定律描述的是：在增加计算资源的情况下，程序在理论上能够实现最高加速比，这个值取决于程序中可并行组件和串行组件所占的比重。假定 F 是必须被串行执行的部分，那么根据 Amdahl 定律，在包含 N 个处理器的及其中，最高的加速比为：

> Speedup <= 1/ ( F + ( 1 - F ) / N )

N 趋近无穷大时，最大加速比趋近于 1/F。因此，程序中如果有 50%的计算需要串行，最大加速比是 2。Amdahl 定律还量化了串行化的效率开销。如果有 10%需要串行，那么加速比永远不会超过10。

> 所有并发程序中都包含一些串行部分。如果你认为你的程序中不存在串行部分，那么可以再仔细检查一遍。

### 11.2.1 示例：在各种框架中隐藏的串行部分

略

### 11.2.2 Amdahl 定律的应用

如果能准确估计出执行过程中串行部分所占的比例，那么 Amdahl 定律就能量化有更多资源时的加速比。虽然直接测量串行部分的比例非常困难，但即使在不进行测试的情况下，Amdahl 定律仍然是有用的。

当评估一个算法时，要考虑算法在数百个或数千个处理器的情况下的性能表现，从而对可能出现的可伸缩性局限有一定程度的认识。例如，锁分解（分成 2 个锁）、锁分段（分成多个）。通过 Amdahl 分析锁分解，似乎并不能提高多处理器能力。但是锁分段可以根据处理器数量决定分段数，似乎更有前途。

## 11.3 线程引入的开销

单线程程序既不存在线程调度，也不存在同步开销，而且不需要锁来保证数据结构的一致性。在多个线程的调度和协调过程中都需要一定的性能开销：对于为了提升性能而引入的线程来说，并行带来的性能提升必须超过并发导致的开销。

### 11.3.1 上下文切换

如果主线程是唯一的线程，那么它基本上不会被调度出去。另一方面，如果可运行的线程数大于 CPU 数量，那么操作系统最终会将某个正在运行的线程调度出来，从而使其他线程能够使用 CPU。这将导致一次上下文切换，在这个过程中将保存当前运行线程的执行上下文，并将新调度进来的线程的执行上下文设置为当前上下文。

切换上下文需要一定的开销，而在线程调度过程中需要访问由操作系统和 JVM 共享的数据结构。应用程序、操作系统以及 JVM 都使用的是一组相同的 CPU。JVM 和 操作系统代码中消耗越多的 CPU 时钟周期，应用程序的可用 CPU 时钟周期就越少。但上下文切换的开销并不只是包含 JVM 和操作系统的开销。当一个新的线程被切换进来时，他所需要的数据可能不在当前处理器的本地缓存中，因此上下文切换将导致一些缓存丢失，因为线程在首次调度运行时会更加缓慢。这就是为什么调度器会为每个可运行的线程分配一个最小的执行时间，即使有许多其他的线程正在等待执行：它将上下文切换的开销分摊到更多不会中断的执行时间上，从而提升整体吞吐量（以损失响应性为代价）。

当线程由于等待某个发生竞争的锁而阻塞时，JVM 通常将这个线程挂起，并允许它被交换出去。如果线程频繁地发生阻塞，那么它们将无法使用完整的调度时间片。程序中发生越多的阻塞（包括阻塞 I/O，等待获取发生竞争的锁，或者在条件变量上等待），与 CPU 密集型的程序就会发生越多的上下文切换，从而增加调度开销，并因此降低吞吐量。（无阻塞算法同样有助于减少上下文切换，see chapter 15）。

上下文切换实际开销会随平台的不同而变化，按照经验来看：大多数通用的处理器中，上下文切换的开销相当于  5000~10000 个时钟周期，也就是几微秒。

UNIX 系统的 vmstat 命令和 Windows 的 perfmon 工具，都能报告上下文的切换次数以及在内核中执行时间所占比例等信息。如果内核占有率高（超出 10%），那么通常表示调度活动发生得很频繁，很可能是由 I/O 或竞争锁导致的阻塞引起的。

### 11.3.2 内存同步

同步操作的性能开销包括多个方面。在 synchronized 和 volatile 提供的可见性保证中可能会使用有一些特殊指令，即内存栅栏（Memory Barrier）。内存栅栏可以刷新缓存，让缓存无效，刷新硬件的写缓冲，以及通知执行管道。内存栅栏同样会对性能带来间接的影响，因为它将抑制一些编译器优化操作。在内存栅栏中，大多数操作都是不能被重排序的。

评估同步操作带来的性能影响时，区分有竞争的同步和无竞争的同步非常重要。synchronized 对无竞争的同步进行了优化（volatile 通常是非竞争的），编写本书时，一个“Fast-Path”的非竞争同步将消耗 20~250 个时钟周期。虽然无竞争同步的开销不为 0，但对整体性能影响微乎其微，而另一种方法不仅会破坏线程安全性，而且会让你经历很痛苦的排错过程。

现代 JVM 会优化掉一些不会发生竞争的锁，从而减少不必要的同步开销。如果一个锁对象只能由当前线程访问，那么 JVM 就可以优化掉这个锁获取的操作，因为另一个县城无法与当前线程在这个锁上同步。

一些更完备的 JVM 能通过逸出分析（Escape Analysis）来找出不会发布到堆的本地对象引用。例如：

```java
public String getStoogeNames() {
	List<String> s = new Vector<>();
	s.add("A");
	s.add("B");
	s.add("C");
	return stooges.toString();
}
```

s 变量是局部变量，封闭在栈上的变量自动变成线程本地变量。整个方法操作有很多次同步操作，一个智能的运行时编译器通常会分析这个调用，使 s 及其内部状态不会逸出，从而消除 4 次锁获取操作（锁消除 Lock Elision）。

即使不进行逸出分析，编译器也能执行锁粒度粗化（Lock Coarsening），将临近的同步代码块用同一个锁合并起来。

> 不要过度担心非竞争同步带来的开销。这个基本的机制已经非常快了，并且 JVM 还能进行额外的优化以进一步降低或消除开销。隐藏，我们应该将优化重点放在那些发生锁竞争的地方。

某个线程中的同步可能影响其他线程的性能。同步会增加共享内存总线上的通信量，总线的带宽是有限的，并且所有的处理器都将共享这条总线。如果多个线程竞争同步带宽，那么所有使用了同步的线程都会受影响。

### 11.3.3 阻塞

非竞争的同步可以完全在 JVM 中处理，而竞争的同步可能需要操作系统的介入，从而增加开销。锁上发生竞争时，竞争失败的线程肯定会阻塞。JVM 在实现阻塞行为时，可以采用自旋等待（Spin-Waiting，通过循环不断地尝试获取锁，直到成功）或者通过操作系统挂起被阻塞的线程。这两种方式的效率高低，取决于上下文切换的开销和成功获取锁之前需要等待的时间。如果等待时间较短，可以采用自旋方式；如果等待时间长，则采用线程挂起方式。有些 JVM 将根据对历史等待事件的分析数据在这两者之间选择。

当线程无法获取某个锁或者由于在某个条件等待或者在 I/O 操作阻塞时，需要被挂起，在这个过程将包含两次额外的上下文切换，以及所有必要的操作系统操作和缓存操作：被阻塞的线程在其执行时间片还没用完前就被交换出去，而在随后当要获取的锁或其他资源可用时，又再次被切换回来。

## 11.4 减少锁的竞争

我们已经看到，串行操作会降低可伸缩性，并且上下文切换也会降低性能。在锁上发生竞争的同时会导致这两种问题，因此减少锁的竞争能够提高性能和可伸缩性。

> 在并发程序中，对可伸缩性的最主要威胁就是独占方式的资源锁。

有两个因素影响锁上发生竞争的可能性：锁的请求频率，以及每次持有锁的时间（Little 定律的必然结论，也是排队理论的一个推论）。如果两者乘积很小，那么大多数获取锁的操作都不会发生竞争，因此在该锁上的竞争不会对可伸缩性造成严重影响。

> 有 3 种方式降低锁的竞争程度：
>
> - 减少锁的持有时间。
> - 降低锁的请求频率。
> - 使用带有协调机制的独占锁，这些机制允许更高的并发性。

### 11.4.1 缩小锁的范围（“快进快出”）

缩小范围可以减少锁的持有时间。例如，将一些与锁无关的代码移出同步代码块，尤其是开销大的操作，以及可能被阻塞的操作，例如 I/O 操作。

尽管缩小同步代码块能提高伸缩性，但同步代码块不能过小——一些需要原子操作的必须包含在一个同步块。同步需要一定的开销，如果一个同步块拆成多个，反而会对性能产生负面影响（如果 JVM 进行锁粗化，又合并了）。

### 11.4.2 减小锁的粒度

减小锁粒度可以降低线程请求锁的频率。这可以通过锁分解和锁分段来实现。

如果一个锁保护多个相互独立的状态变量，可以将这个锁分解为多个锁，并且每个锁只保护一个变量，从而提高可伸缩性，最终降低每个锁被请求的频率。

例：

```java
@ThreadSafe
public class ServerStatus {
    @GuardedBy("users")
    public final Set<String> users;
    @GuardedBy("queries")
    public final Set<String> queries;

    public ServerStatus(Set<String> users, Set<String> queries) {
        this.users = users;
        this.queries = queries;
    }

    public void addUser(String u) {
        synchronized (users) {
            users.add(u);
        }
    }

    public void addQuery(String s) {
        synchronized (queries) {
            queries.add(s);
        }
    }
}
```

对竞争适中的锁进行分解时，实际将这些锁转变为了非竞争的锁，有效提高了性能和伸缩性。

### 11.4.3 锁分段

某些情况下，可以将锁分解技术进一步扩展为对一组独立对象上的锁进行分解，这叫锁分段。例如，在 ConcurrentHashMap （Java 7）的实现中使用了一个包含16个锁的数组，每个锁保护所有散列桶的 1/16，其中第 N 个散列桶由 N mod 16 个锁来保护。

锁分段的一个劣势在于：与采用单个锁来独占访问相比，要获取多个锁来实现独占访问将更加困难而且开销更高。通常，执行一个操作时只需要一个锁，但在某些情况下需要加锁整个容器，例如当 ConcurrentHashMap 扩容，以及 rehash 时，就要获取分段锁集合中的所有锁（要获取内置锁的一个集合，能采用的唯一方式是递归）。

例：

```java
@ThreadSafe
public class StripedMap {
    // 同步策略：buckets[n] 由 locks[n % N_LOCKS] 保护
    private static final int N_LOCKS = 16;
    private final Node[] buckets;
    private final Object[] locks;

    public StripedMap(int numberBuckets) {
        this.buckets = new Node[numberBuckets];
        locks = new Object[N_LOCKS];
        for (int i = 0; i < N_LOCKS; i++) {
            locks[i] = new Object();
        }
    }

    private static class Node {
        Object key;
        Node next;
        Object value;
    }

    private final int hash(Object key) {
        return Math.abs(key.hashCode() % buckets.length);
    }

    public Object get(Object key) {
        int hash = hash(key);
        synchronized (locks[hash % N_LOCKS]) {
            for (Node m = buckets[hash]; m != null; m = m.next) {
                if (m.key.equals(key)) {
                    return m.value;
                }
            }
        }
        return null;
    }
    
    public void clear(){
        for (int i = 0; i < buckets.length; i++) {
            synchronized (locks[i % N_LOCKS]) {
                buckets[i] = null;
            }
        }
    }
    
}
```

clear 方法并不是原子性方法，如果要保证原子性，那么要首先获取全部的锁。size empty 可能会无效，通常是可以接受的。

### 11.4.4 避免热点域

锁分解和分段都能提高可伸缩性，因为它们都能使不同的线程在不同的数据（或同数据不同部分）上操作，而不会相互干扰。

热点域（Hot Field）往往会限制可伸缩性。

当实现 HashMap 时，有一个 size 变量，在每次调用 put remove 等方法时维护该变量的值。虽然稍微增加了一些开销，但是 size 都是最新值，把开销从 O(n) 降低到 O(1)。

在单线程或者完全同步的程序，这样是十分可行的。但对于分段锁，要对计数器的访问进行同步，这时候计数器就是热点域，造成可伸缩性问题。

为避免这个问题，ConcurrentHashMap size 方法对每个分段进行枚举并将每个分段中的元素数量相加，而不是维护一个全局计数，通过每个分段的锁维护这个值。1.8 使用 CounterCell （Adapted from LongAdder and Striped64）。

### 11.4.5 一些替代独占锁的方法

第三种降低竞争锁的影响的技术就是放弃使用独占锁，使用更友好并发的方式管理共享状态。例如，使用并发容器、读写锁、不可变对象和原子变量。

原子变量，使用了现代处理器中提供的底层并发原语（例如 compare and swap）。如果类中只包含少量热点域，并且这些域不会与其他变量一起参与到不变性条件，那么就可以使用原子变量提高伸缩性。（能降低热点域的更新开销，但不能完全消除）。

### 11.4.6 监测 CPU 的使用率

当测试可伸缩时，通常要保证处理器得到充分利用。UNIX 上如 vmstat 和 mpstat，Windows 上如 perfmon，都能做到。

如果 CPU 没有得到充分利用，通常有以下几种原因：

- 负载不充足。
- I/O 密集。可以通过 iostat 或 perfmon 判断
- 外部限制。应用程序依赖外部服务，例如数据库或 Web 服务，性能瓶颈可能不在自己的代码中。
- 锁竞争。

### 11.4.7 向对象池说“不”

目前 Java 的分配操作已经非常快了。并发应用程序中，对象池表现更加糟糕。这些线程从对象池请求对象，就需要通过某种同步来协调对象池数据结构的访问。如果某个线程由于锁竞争被阻塞，阻塞的开销会是内存分配操作开销的数百倍，也可能会造成可伸缩性瓶颈。

对象池有特定的用途，但对于性能优化来说，用途是有限的。

## 11.5 比较 Map 的性能

略。

## 11.6 减少上下文切换的开销

许多任务都包含一些被阻塞的操作。任务在运行和阻塞两个状态转换时，就相当于一次上下文切换。

请求服务的时间不应该过长。服务时间越长，锁竞争的概率越大，竞争越多，上下文切换次数越多，吞吐量越低。

## 小结

线程通常是为了充分利用多核处理器的计算能力，因此并发程序性能讨论中，侧重点放在吞吐量和可伸缩性上，而不是服务时间。Amdahl 定律告诉我们，可伸缩性取决于所有代码中必须被执行的串行部分。Java 程序中串行操作的主要来源是独占方式的资源锁，因此可以通过以下方式提升可伸缩性：减少锁的持有时间，降低锁的粒度，以及采用非独占的锁或阻塞锁来代替独占锁。

# 第十二章 并发程序的测试

