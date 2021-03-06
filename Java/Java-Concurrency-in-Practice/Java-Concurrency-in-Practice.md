# Java 并发编程实战 读书笔记

## 第1章 简介

### 1.1 并发简史

> 计算机中加入操作系统来实现多个程序的同事执行, 主要基于以下原因: 
1. 资源利用率: 例如在等待IO操作时CPU资源将闲置, 引入并发将提高CPU利用率

2. 公平性: 不同用户和程序对于计算机的上资源有着同等的使用权.

3. 便利性: 在计算多个任务时, 应该编写多个程序, 每个程序执行一个任务在必要时进行通信, 这比编写一个程序来计算所有任务更容易实现.

### 1.2 线程的优势

1. 发挥多处理器的强大能力

2. 建模的简单性

3. 异步事件的简化处理

4. 响应更灵敏的用户界面

### 1.3 线程带来的风险

1. 安全性问题: 永远不发生糟糕的事件

2. 活跃性问题: 某个正确的事件最终会发生(具有随机性)

3. 性能问题: 多线程将增加线程上下文切换, 线程调度, 共享内存总线的同步流量等开销, 尽量保证这些开销总是小于引入多线程所带来的性能的提升

### 1.4 线程无处不在

1. Timer

2. Servlet和JSP

3. 远程方法调用(RMI)

4. Swing和AWT

---

## 第2章 线程安全性

> 构建稳健的并发程序必须正确使用线程和锁, 其核心在于对状态访问操作进行管理, 特别是共享的和可变的状态的访问.
> 共享意味着可以有多个线程同时访问(通常是对象实例域, 静态域), 可变意味着变量的值再起生命周期内可以发生变化.

1. Java中同步机制包括关键字synchronized, volatile, 显示锁(java.util.concurrent.lock包内), 原子变量(java.util.concurrent.atomic包内).

2. 如果某个并发程序缺少必要的同步机制并且看上去似乎能正确执行, 但程序仍可能在某个时刻发生错误(活跃性风险).

3. 如果多个程序访问可变状态变量时没有使用合适的同步, 那么程序就会出现错误, 有三种方式可以修复: 
不在程序中共享该变量(例如将其变为局部变量)
将状态变量修改为不可变得变量(多线程只能读取而不能修改)
在访问变量时使用同步机制

4. 程序的封装性越好越容易实现程序的线程安全性, 在设计线程安全的类时, 良好的面向对象技术, 不可修改性, 以及明晰的不可变性规范都能起到一定的帮助作用.

5. 在编写并发应用程序时, 一种正确的编程方法是: 首先使代码正确运行, 然后在提高代码的速度, 即便如此, 最好也只是当性能测试结果和应用需求告诉你必须提高性能, 以及测试结果表明这种优化在实际环境中确实能带来性能提升时, 才进行优化.

6. 完全由线程安全的类构成的程序并不一定就是线程安全的, 而在线程安全的类中也可以包含非线程安全的类.

### 2.1 什么是线程安全性

> 当多个线程访问某个类时, 不管运行时环境采用何种调度方式或者这些线程将如何交替执行, 并且在主调代码中不需要任何额外的同步或协同, 这个累都能表现出正确的行为(也就是与其规范一致的行为), 那么就称这个类是线程安全的.

1. 在线程安全的类中封装了必要的同步机制, 因此客户端无须进一步采取同步措施, 但是当客户端想要保证调用线程安全类的两个方法的原子性时, 需要使用外部同步.

2. 无状态对象一定是线程安全的.

3. 局部变量不在线程之间共享, 因此不需要同步.

### 2.2 原子性

#### 2.2.1 竞态条件

> 竞态条件: 由于不恰当的执行时序出现不正确的执行结果.

1. 最常见的竞态条件就是先检查后执行操作, 即通过一个可能失效的观测结果来决定下一步操作, 也就说说正确的结果取决于运气.

2. **竞态条件**很容易与**数据竞争**相混淆, 数据竞争是指多线程访问共享的非final域时, 没有采用同步进行协同, 会出现数据竞争. 当一个线程写入一个变量而另一个线程读取这个变量, 或者读取一个之前由另一个线程写入的变量时, 并且线程之间没有同步就会出现数据竞争.

3. 并非所有竞态条件都是数据竞争, 并发所有数据竞争都是竞态条件.

#### 2.2.2 延迟初始化中的竞态条件
```java
@NotThreadSafe
public class LazyInitRace {
    private ExpensiveObject instance = null;

    public ExpensiveObject getInstance() {
        if (instance == null)
            instance = new ExpensiveObject();
        return instance;
    }
    private static class ExpensiveObject {}
}
```

#### 2.2.3 复合操作

> 假设有两个操作A和B, 如果从执行A的线程来看, 当另一个线程执行B时, 要么将B全部执行完, 要么完全不执行B, 那么A和B对彼此来说是原子的. 原子操作是指, 对于访问同一个状态的所有操作(包括该操作本身)来说, 这个操作是一个以原子方式执行的操作.

1. 在实际情况中, 应尽可能地使用现有的线程安全对象来管理类的状态. 与非线程安全的对象相比, 判断线程安全对象的可能状态及其状态转换情况要更为容易, 从而也更容易维护和验证线程安全性.

### 2.3 加锁机制

> 要保持状态一致性, 就需要在单个原子操作中更新所有相关的状态碧昂量.

#### 2.3.1 内置锁

1. Java提供了synchronized关键字来支持原子性(第3章介绍synchronized保证的内存可见性).

```java
// 以下两种方法等效
public synchronizedd void method() {
}
public void method() {
	synchronizedd (this) {
	}
}

// 以下两种方法等效
public static synchronizedd void method() {
}
public static void method() {
	// 以当前对象的Class对象作为锁
	synchronizedd (SyncObject.class) {
	}
}
```

2. 每个Java对象都可以用做一个实现同步的锁, 这些锁被称为内置锁或监视器锁, 线程进入同步代码块之前获取锁, 退出同步代码块时释放锁, 较显式锁不同, **内置锁无论是正常退出还是异常退出都会释放锁**.

```java
private final Object lock = new Object();
public void method() {
	synchronized (lock) {
	}
}
```

#### 2.3.2 重入

1. 线程可以获取一个已经由它持有的锁, 内置锁是可重入的, 如果内置锁不可重入, 那么以下这段代码将发生死锁

```java
class Widget {
    public synchronizedd void doSomething() {}
}

class LoggingWidget extends Widget {
    public synchronizedd void doSomething() {
        System.out.println(toString() + ": calling doSomething");
        super.doSomething();
    }
}
```

2. 重入的实现方法是为锁关联一个获取计数值和锁当前的持有者线程, 当计数值为0时, 这个锁被认为没有被任何线程持有, 当线程请求一个未被持有的锁时, JVM将记录锁的持有者线程, 计数值将递增, 而当线程退出同步代码块时, 计数值将递减, 当计数值为0时, 这个锁将释放, JVM将取消记录锁的持有者线程.

### 2.4 用锁来保护状态

> 锁能使其保护的代码以串行的形式来访问.

1. 在读写共享变量是都需要使用同步, 且是使用同一个锁来同步, 并非在线程写入共享变量时才使用.

2. 对于可能被多个线程同时访问的可变状态变量, 在访问它时都需要持有同一个锁, 在这种情况下我们称这个状态是由同一个锁来保护的.

3. 对象的内置锁与其状态之间没有内在关联, 对象的与不一定要通过内置锁来保护.

4. 当共享的可变变量相互之间独立时, 每个共享的可变的变量都应该值由一个锁来保护(锁分解技术, 增加程序可伸缩性, 第11章介绍), 从而使维护人员知道是哪一种锁.

5. 对于每个包含多个变量的不变性条件, 其中涉及的所有变量都需要由同一个锁来保护.

### 2.5 活跃性与性能

1. 再简单性与性能之间存在着相互制约因素, 当实现某个同步策略时, 一定不要盲目的为了性能而牺牲简单性(这可能会破坏安全性).

2. 锁的持有时间过长会带来活跃性或性能问题, 当执行较长时间的计算或者可能无法快速完成的操作时(例如, 网路IO或控制台IO), 一定不要持有锁.

---

## 第3章 对象的共享

> 一种常见的误解是, 认为关键字synchronized只能用于实现原子性或者确定临界区, 同步还有一个重要的方面: 内存可见性.

### 3.1 可见性

> 为了确保多个线程之间对内存写入操作的可见性, 必须使用同步机制.

1. 请考虑以下程序:

```java
public class NoVisibility {
    private static boolean ready;

    private static class ReaderThread extends Thread {
        private int i;

        @Override
        public void run() {
            while (!ready)
                i++;
            System.out.println(i);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new ReaderThread().start();

        TimeUnit.SECONDS.sleep(1);
        ready = true;
    }
}
```

1. 以上程序看似最终输出一个i值, 实际运行发现这段程序大多数时候根本无法终止, 也就是说主线程将ready置为true, 而ReaderThread根本就没看到.

2. 如果你认为以上程序错误的输出只是因为原子性的问题, 那也不对. 其实ready的读写原本就是原子性的, 如果说synchronized只保证了原子性, 那么将ready的读写用synchronized保护起来的话也是多此一举, 程序可能还是无法终止, 修改后代码如下:

```java
public class NoVisibility {
    private static boolean ready;

    private synchronizedd static boolean getReady() {
        return ready;
    }

    private synchronizedd static void setReady(boolean ready) {
        NoVisibility.ready = ready;
    }

    private static class ReaderThread extends Thread {
        private int i;

        @Override
        public void run() {
            while (!getReady())
                i++;
            System.out.println(i);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new ReaderThread().start();

        TimeUnit.SECONDS.sleep(1);
        setReady(true);
    }
}
```

1. 按照之前的推测修改后的程序也不会终止, 实际运行1s左右之后就输出i值了, 也就是说之前的推测是不对的, **synchronized不仅保证了操作的原子性, 又保证了它所保护的变量的可见性**.

```java
public class NoVisibility {
    private static boolean ready;

    private static class ReaderThread extends Thread {
        private int i;

        @Override
        public void run() {
            while (!ready)
                i++;
            System.out.println(i);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new ReaderThread().start();

        TimeUnit.SECONDS.sleep(1);
        ready = true;
    }
}
```

1. 看以上这段代码, 较第一段代码相比, 只修改了读线程while循环体, 将i++改为Thread.yield();, 运行发现它又能正常工作了, 这又印证一件事情: 如果不采用同步机制, 线程之间操作共享变量的可见性是随机的, 之前的i++使读线程所在CPU过于忙碌, 没时间去主存中获取最新的ready值, 而Thread.yield();则不是计算密集型任务, CPU可能会抽时间去主内存重新获取最新的ready值, 最终导致程序终止了, 但是这种重新获取值的操作带有随机性, 还是会有安全性风险. 如果强制让CPU每次访问ready值都从主内存中读取, 那么就不会带来这种问题, synchronized可以解决这个问题(volatile也能解决, 后序介绍). 当然每次都从主内存读取变量, 获取锁和释放锁, 锁的独占性等都会带来性能问题, 但是程序首先应以安全性为主, 其次才是性能.

```java
public class NoVisibility {
    private static boolean ready;
    private static int number;

    private static class ReaderThread extends Thread {
        @Override
        public void run() {
            while (!ready)
                Thread.yield();
            System.out.println(number);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new ReaderThread().start();
        number = 42;
        ready = true;
    }
}
```

1. 这段程序可能会一直循环下去, 这是由于内存可见性的问题. 更奇怪的有时程序会终止输出的number为0, 也就是说读线程看到了写入的ready值, 但是没有看到写入的number值, 这种线程称为重排序(详见第16章).

在没有同步的情况下, 编译器, 处理器, 运行时等可能对操作的执行顺序进行一些意想不到的调整, 在缺乏足够同步的多线程程序中, 要想对内存操作的执行顺序进行判断, 几乎无法得出正确的结论.

#### 3.1.1 失效值

1. 3.1节中的几个程序展示了在缺乏同步的程序中可能产生错误结果的一种情况: 失效数据. 更糟糕的是, 失效值不会同时出现: 一个线程可能获得某个变量的最新值, 而获得另一个变量的失效值.

#### 3.1.2 非原子的64位操作

1. Java内存模型要求, 变量的读取和写入操作都必须是原子操作, 但对于非volatile类型的long和double变量, JVM允许将64位的读操作或写操作, 分解为两个32位的操作. 当读取一个非volatile类型的long变量时, 如果对该变量的读操作和写操作在不同的线程中执行, 那么很可能会读取到每个值的高32位和另一个值的第32位, 这被称为字撕裂, 通过volatile修饰或者用锁保护起来的long或double可以保证读写原子性.

#### 3.1.3 加锁与可见性

1. 加锁可以用于确保某个线程以一种可预测的方式来查看另一个线程执行结果.

2. 加锁的含义不仅仅局限于互斥行为, 还包括内存可见性. 为了确保所有线程都能看到共享变量的最新值, 所有执行读操作和写操作的线程都必须在同一个锁上进行同步.

#### 3.1.4 volatile变量

1. 编译器和运行时不会对volatile变量上的操作与其他内存操作一起重排序, volatile变量也不会被缓存在寄存器或者对其它处理器不可见的地方, 读取volatile变量时总是返回最新写入的值.

2. volatile变量是一种比synchronized关键字更轻量级的同步机制.(在当前大多数处理器中, 读取volatile变量比读取非volatile变量的开销略高一些)

3. 加锁机制既可以确保可见性又可以确保原子性, 而volatile变量只能确保可见性.

当且仅当满足以下条件时, 才可以使用volatile变量:
* 对变量的写入操作不依赖于变量当前值, 或者你能确保只有单个线程更新变量的值.
* 该变量不会与其它状态变量一起纳入不变性条件中.
* 在访问变量时不需要加锁.

(个人理解: 只要能够保证变量的操作原子性, 那么就可以使用volatile, 例如3.1节中的ready变量值, 它的读写操作都是原子操作, 就可以用volatile来修正3.1.1节中程序存在的问题.)

### 3.2 发布与逸出

1. 发布一个对象是指该对象能够在当前作用于之外的代码中使用(例如对象的初始化)

2. 当发布某个对象时, 如果在发布过程中确保线程安全性, 则可能需要使用同步.(在发布一个没有状态的对象或者所有状态均由final修饰的对象时不需要使用同步)

3. 发布内部状态可能会破坏封装性, 并使得程序难以维持不变性条件.

4. 当某个不应该被发布的对象被发布时, 这种情况被称为逸出.

5. 发布某个对象时会间接发布其它对象, 例如发布一个Map时, 会间接将Map中存储的键值也发布.

6. 不要再构造过程中使this引用逸出.

```java
// this引用逸出
public class ThisEscape {
    public ThisEscape(EventSource source) {
        source.registerListener(new EventListener() {
            public void onEvent(Event e) {
                doSomething(e);
            }
        });
    }

    void doSomething(Event e) {}

    interface EventSource {
        void registerListener(EventListener e);
    }

    interface EventListener {
        void onEvent(Event e);
    }

    interface Event {}
}
```

7. 可以使用私有构造函数和工厂方法来避免this引用逸出问题.

```java
public class SafeListener {
    private final EventListener listener;

    private SafeListener() {
        listener = new EventListener() {
            public void onEvent(Event e) {
                doSomething(e);
            }
        };
    }

    public static SafeListener newInstance(EventSource source) {
        SafeListener safe = new SafeListener();
        source.registerListener(safe.listener);
        return safe;
    }

    void doSomething(Event e) {}

    interface EventSource {
        void registerListener(EventListener e);
    }

    interface EventListener {
        void onEvent(Event e);
    }

    interface Event {}
}
```

### 3.3 线程封闭

> 当某个对象封闭在一个线程中时, 这中用法将自动实现线程安全性, 即使被封闭的对象本事不是线程安全的, 同样在现成封闭的环境中使用线程不安全的对象也免去了线程同步和协调的开销, 比如在方法中肯定是优先使用HashMap, 而不是HashTable.

#### 3.3.1 Ad-hoc线程封闭

1. Ad-hoc线程封闭是指, 维护线程封闭性的职责完全由程序实现来承担.

2. Ad-hoc线程封闭程序较脆弱, 因此在程序中尽量少用, 在可能的情况下使用更强的线程封闭技术, 例如栈封闭或者ThreadLocal类.

#### 3.3.2 栈封闭

1. 栈封闭是将变量封闭在线程本地栈中, 其它线程无法访问这个栈.

2. 一种常用的栈封闭手段就是使用局部变量

```java
public int loadTheArk(Collection<Animal> candidates) {
	SortedSet<Animal> animals;
	int numPairs = 0;
	Animal candidate = null;
	// animals confined to method, don’t let them escape!
	animals = new TreeSet<Animal>(new SpeciesGenderComparator());
	animals.addAll(candidates);
	for (Animal a : animals) {
		if (candidate == null || !candidate.isPotentialMate(a))
			candidate = a;
		else {
			ark.load(new AnimalPair(candidate, a));
			++numPairs;
			candidate = null;
		}
	}
	return numPairs;
}
```

#### 3.3.3 ThreadLocl类

1. 使用ThreadLocal类也可以实现线程封闭.

2. ThreadLocal通常用作全局变量, 降低代码可重用性, 在类之间引入隐含的耦合性, 使用时要注意.

### 3.4 不变性

1. 不可变的对象一定是线程安全的.

2. 不可变对象满足以下条件: 
对象创建以后状态不能修改.
对象的所有域都是final类型(如果保证域在业务上是不可变的, 也可以不用final类型(如: String的hashCode域), 尽量避免这么做).
对象是正确创建的(创建期间, this引用没有逸出)

### 3.4.1 final域

1. 在Java内存模型中, final域能确保初始化过程的安全性, 共享这些对象时无需同步.

2. 尽量将于设置成final类型.

### 3.4.2 使用volatile类型来发布不可变对象

1. 对于在访问和更新多个相关变量时出现的竞争条件问题, 可以通过将这些变量全部保存在一个不可变对象中消除.

```java
@ThreadSafe
public class VolatileCachedFactorizer extends GenericServlet implements
        Servlet {
    private volatile OneValueCache cache = new OneValueCache(null, null);

    public void service(ServletRequest req, ServletResponse resp) {
        BigInteger i = extractFromRequest(req);
        BigInteger[] factors = cache.getFactors(i);
        if (factors == null) {
            factors = factor(i);
            cache = new OneValueCache(i, factors);
        }
        encodeIntoResponse(resp, factors);
    }

    void encodeIntoResponse(ServletResponse resp, BigInteger[] factors) {}

    BigInteger extractFromRequest(ServletRequest req) {
        return new BigInteger("7");
    }

    BigInteger[] factor(BigInteger i) {
        // Doesn't really factor
        return new BigInteger[] { i };
    }
}

@Immutable
public class OneValueCache {
    private final BigInteger lastNumber;
    private final BigInteger[] lastFactors;

    public OneValueCache(BigInteger i, BigInteger[] factors) {
        lastNumber = i;
        lastFactors = Arrays.copyOf(factors, factors.length);
    }

    public BigInteger[] getFactors(BigInteger i) {
        if (lastNumber == null || !lastNumber.equals(i))
            return null;
        else
            return Arrays.copyOf(lastFactors, lastFactors.length);
    }
}
```

### 3.5 安全发布

1. 由于内存可见性的问题, 导致对象不安全发布.

#### 3.5.1 不正确的发布: 正确的对象被坏

1. 由于没有使用同步来确保内存可见性, Holder类的assertSanity方法会抛异常.

```java
public class Holder {
    private int n;

    public Holder(int n) {
        this.n = n;
    }

    public void assertSanity() {
        if (n != n)
            throw new AssertionError("This statement is false.");
    }
}
```

2. 一个线程初始化字段n, 对另一个线程看到的可能是一个null或者旧值(默认初始值), 导致if(n != n)两次读取到不一致的值.

#### 3.5.1 不可变对象与初始化安全性

1. 任何线程都可以在不需要同步的情况下安全地访问不可变对象, 即使发布对象时没有使用同步.

2. 如果final类型的域指向的是可变对象, 那么在访问这些域指向的对象的状态时仍然需要同步.

#### 3.5.3 安全发布的常用模式

安全发布方式:

* 在静态初始化函数中初始化一个对象引用.

* 将对象的引用保存到volatile类型的域或者AtomicReferance对象中.

* 将对象的引用保存到某个被正确构造对象的final类型域中.

* 将对象的引用保存到一个由锁保护的域中.

#### 3.5.4 事实不可变对象

1. 如果对象从技术上看是可变的, 但是状态在发布之后不会再改变, 这种对象称之为事实不可变对象.

2. 在没有额外同步的情况下, 任何线程都可以安全地使用被安全发布的事实不可变对象.

#### 3.5.5 可变对象

对象的发布需求取决于它的可变性

* 不可变对象可以通过任意方式来发布

* 事实不可变对象必须通过安全发布的方式来发布

* 可变对象必须通过安全发布的方式来发布, 并且必须是线程安全的类或者由某个锁保护起来

#### 3.5.6 安全地共享对象

在并发程序中使用和共享对象时, 可以使用一些实用的策略

* 线程封闭: 线程封闭的对象只能由一个线程访问.

* 只读共享: 不可变对象和事实不可变对象.

* 线程安全共享: 线程安全的对象内部实现同步机制, 如ConcurrentHashMap.

* 保护对象: 通过持有特定的锁来访问.

---

## 第10章 避免活跃性风险

> 在安全性和活跃性之间通常存在着某种制衡.

### 10.1 死锁

死锁的四个必要条件:

1. 互斥条件: 一个资源每次只能被一个线程使用.
2. 占有且等待: 一个进程因请求资源而阻塞时, 对已获得的资源保持不放.
3. 不可强行占有: 线程已获得的资源, 在未使用完之前, 不能强行被剥夺.
4. 循环等待条件: 若干进程之间形成一种头尾相接的循环等待资源关系.

对于Java中的synchronized来说, 前三个条件是天然满足且无法打破的, 所以只要等防止循环等待条件发生, 就可以避免synchronized造成的死锁.

#### 10.1.1 锁顺序死锁

```java
// 锁顺序死锁
public class LeftRightDeadlock {
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
                doSomethingElse();
            }
        }
    }

    void doSomething() {}

    void doSomethingElse() {}
}
```

1. 如果所有的线程以固定的顺序来获得锁, 那么在程序中就不会出现所顺序死锁的问题.

#### 10.1.2 动态锁顺序死锁

```java
public class DynamicOrderDeadlock {
    public static void transferMoney(Account fromAccount, Account toAccount, DollarAmount amount) throws InsufficientFundsException {
        synchronized (fromAccount) {
            synchronized (toAccount) {
                if (fromAccount.getBalance().compareTo(amount) < 0)
                    throw new InsufficientFundsException();
                else {
                    fromAccount.debit(amount);
                    toAccount.credit(amount);
                }
            }
        }
    }
}

Account myAccount = new Account();
Account yourAccount = new Account();
DynamicOrderDeadlock.transferMoney(myAccount, yourAccount,
		new DollarAmount(10));
DynamicOrderDeadlock.transferMoney(yourAccount, myAccount,
		new DollarAmount(20));
```

1. 当获取锁的顺序是由外部程序调用来决定时, 可能会发生动态所顺序死锁.

```java
private static final Object tieLock = new Object();

public void transferMoney(final Account fromAcct, final Account toAcct,
						  final DollarAmount amount) throws InsufficientFundsException {
	class Helper {
		public void transfer() throws InsufficientFundsException {
			if (fromAcct.getBalance().compareTo(amount) < 0)
				throw new InsufficientFundsException();
			else {
				fromAcct.debit(amount);
				toAcct.credit(amount);
			}
		}
	}
	int fromHash = System.identityHashCode(fromAcct);
	int toHash = System.identityHashCode(toAcct);

	if (fromHash < toHash) {
		synchronized (fromAcct) {
			synchronized (toAcct) {
				new Helper().transfer();
			}
		}
	} else if (fromHash > toHash) {
		synchronized (toAcct) {
			synchronized (fromAcct) {
				new Helper().transfer();
			}
		}
	} else {
		synchronized (tieLock) {
			synchronized (fromAcct) {
				synchronized (toAcct) {
					new Helper().transfer();
				}
			}
		}
	}
}
```

2. 可以在方法内部为fromAcct对象和toAcct对象生成一个hash值, 根据hash值来决定获取锁顺序, 若hash相同则现获取加时赛锁tieLock来避免死锁, 如果频繁发生hash值相同的情况, 这可能会成为程序性能瓶颈.

3. 如果Account对象内存在唯一的, 不可变的, 并且具有可比性的键值, 可以通过比较键值来决定获取锁的顺序, 也避免了引入加时赛锁的复杂.

#### 10.1.3 在协作对象之间发生的死锁

```java
public class CooperatingDeadlock {
    // Warning: deadlock-prone!
    class Taxi {
        private final Dispatcher dispatcher;
        @GuardedBy("this")
        private Point location, destination;

        public Taxi(Dispatcher dispatcher) {
            this.dispatcher = dispatcher;
        }

        public synchronized Point getLocation() {
            return location;
        }

        public synchronized void setLocation(Point location) {
            this.location = location;
            if (location.equals(destination))
                dispatcher.notifyAvailable(this);
        }

        public synchronized Point getDestination() {
            return destination;
        }

        public synchronized void setDestination(Point destination) {
            this.destination = destination;
        }
    }

    class Dispatcher {
        @GuardedBy("this")
        private final Set<Taxi> taxis;
        @GuardedBy("this")
        private final Set<Taxi> availableTaxis;

        public Dispatcher() {
            taxis = new HashSet<Taxi>();
            availableTaxis = new HashSet<Taxi>();
        }

        public synchronized void notifyAvailable(Taxi taxi) {
            availableTaxis.add(taxi);
        }

        public synchronized Image getImage() {
            Image image = new Image();
            for (Taxi t : taxis)
                image.drawMarker(t.getLocation());
            return image;
        }
    }

    class Image {
        public void drawMarker(Point p) {}
    }
}
```

1. 如果持有锁时调用某个外部方法, 那么将出现活跃性问题. 在这个外部方法中可能会获得其它锁(这可能会产生死锁), 或者阻塞时间过长, 导致其它线程无法及时获得当前被持有的锁.

#### 10.1.4 开放调用

```java
class CooperatingNoDeadlock {
    @ThreadSafe
    class Taxi {
        private final Dispatcher dispatcher;
        @GuardedBy("this")
        private Point location, destination;

        public Taxi(Dispatcher dispatcher) {
            this.dispatcher = dispatcher;
        }

        public synchronized Point getLocation() {
            return location;
        }

        public synchronized void setLocation(Point location) {
            boolean reachedDestination;
            synchronized (this) {
                this.location = location;
                reachedDestination = location.equals(destination);
            }
            if (reachedDestination)
                dispatcher.notifyAvailable(this);
        }

        public synchronized Point getDestination() {
            return destination;
        }

        public synchronized void setDestination(Point destination) {
            this.destination = destination;
        }
    }

    @ThreadSafe
    class Dispatcher {
        @GuardedBy("this")
        private final Set<Taxi> taxis;
        @GuardedBy("this")
        private final Set<Taxi> availableTaxis;

        public Dispatcher() {
            taxis = new HashSet<Taxi>();
            availableTaxis = new HashSet<Taxi>();
        }

        public synchronized void notifyAvailable(Taxi taxi) {
            availableTaxis.add(taxi);
        }

        public Image getImage() {
            Set<Taxi> copy;
            synchronized (this) {
                copy = new HashSet<Taxi>(taxis);
            }
            Image image = new Image();
            for (Taxi t : copy)
                image.drawMarker(t.getLocation());
            return image;
        }
    }

    class Image {
        public void drawMarker(Point p) {}
    }
}
```

1. 如果在调用某个方法是不需要持有锁, 那么这种调用被称为开方调用.

2. 在程序中尽量使用开放调用. 与那些在持有锁时调用外部方法的程序相比, 更易于对依赖于开发调用的程序进行死锁分析.

3. 开放调用会使得原子操作变成非原子操作.

#### 10.1.5 资源死锁

1. 假如线程在获取不同数据库连接的时候, 不同的获取顺序会导致资源死锁. 资源池越大, 死锁的可能性就越小.

2. 线程饥饿死锁, 查阅8.1.1节.

### 10.2 死锁的避免与诊断

1. 尽量减少潜在的加锁交互数量, 将获取锁时需要遵循的协议写入正式文档并始终遵循这些协议.

#### 10.2.1 支持定时的锁

1. 使用java.util.concurrent.locks包下的显式锁中提供的定时锁(tryLock方法)可有效避免死锁.

#### 10.2.2 通过线程转储信息来分析死锁

1. 分析线程转储文件可以发现死锁问题.

### 10.3 其它活跃性危险

#### 10.3.1 饥饿

1. 当线程无法访问它所需要的资源而不能继续执行时, 就发生了饥饿, java应用程序中线程优先级使用不当会导致线程获取不到CPU时钟周期引发饥饿.

2. 避免使用线程优先级, 因为这会增加平台依赖性, 并可能导致活跃性问题. 在大多数应用程序中, 都可以使用默认的优先级.

#### 10.3.2 糟糕的响应性

1. 不良的锁管理可能会导致糟糕的响应性. 如果某个线程长时间持有一个锁, 其它获取这个锁的线程就必须等待很长时间.

#### 10.3.3 活锁

1. 活锁: 线程不断重复执行相同的操作, 而且总会失败, 两个过于礼貌的人在半路面对面相遇, 他们都彼此让出对方的路, 然而又在另一条路上相遇, 因此就这样反复的避让下去.

2. 在重试机制中引入随机性可解决活锁问题.

---

## 第11章 性能与可伸缩性

### 11.1 对性能的思考

引入多线程会增加以下开销: 

* 线程之间协调(加锁, 触发信号, 内存同步等).
 
* 线程的上下文切换.

* 线程的创建和销毁.

* 线程的调度.

如果过度地使用线程, 那么这些开销甚至会超过提高吞吐量, 响应性, 计算能力锁带来的性能提升.

一个并发设计糟糕的程序, 其性能比实现相同功能的串行程序的性能还要差.

通过并发来获得更好的性能, 需要努力做好两件事:
 
* 更有效的利用现有处理资源.

* 在出现新的处理资源时使程序尽可能地利用这些新资源(程序伸缩性).

#### 11.1.1 性能和可伸缩性

1. 可伸缩性指的是: 当增加计算资源时(例如CPU, 内存, 存储容量或I/O带宽), 程序的吞吐量或者处理能力能相应地增加.

2. 性能的两个方面: **多快**和**多少**是完全独立的, 有时候甚至是相互矛盾的.

多快指的是性能调优: 用更小的代价完成相同的工作.

多少指的是可伸缩性, 吞吐量: 用更多的资源来完成更多的工作.

#### 11.1.2 评估各种性能权衡因素

1. 避免不成熟的优化. 首先使程序正确, 然后在提高运行速度----如果它还运行得不够快.

在使某个方案比其他方案更快之前, 首先问自己一些问题:

* 更快的含义是什么?
* 该方法在什么条件下运行的更快? 在低负载还是高负载的情况下? 大数据集还是小数据集? 能否通过测试结果来验证你的答案?
* 这些条件在运行环境中的发生频率? 能否通过测试结果来验证你的答案?
* 在其它不同条件的环境中能否使用这些代码?
* 在实现这种性能提升时需要付出哪些隐含的代价, 例如增加开发风险或维护开销? 这种权衡是否合适?

对性能的提升可能是并发错误的最大来源.

**以测试为基准, 不要猜测.**

### 11.2 Amdahl定律

1. 在增加计算资源的情况下, 程序在理论上能够实现最高加速比, 这个值取决于程序中并行组件与串行组件所占的比重. 假定F是必须被串行执行的部分, 那么根据Amdahl定律, 在包含N个处理器的机器中, 最高的加速比:

![11-1](https://github.com/ossaw/notes/blob/master/Pictures/jcip/java-concurrency-in-practice-11-1.jpg)

2. 在所有并发程序中都包含一些串行部分. 如果你认为在你的程序中不存在串行部分, 那么可以在检查一遍.

### 11.2.1 示例: 在各种框架中隐藏的串行部分

1. 要想知道串行部分是如何隐藏在应用程序的架构中, 可以比较线程增加时吞吐量的变化, 根据观察到的可伸缩性变化来推断串行部分中的差异.

#### 11.2.2 Amdahl定律的应用

锁分解技术: 将一个锁分解为两个锁.

锁分段技术: 将一个锁分解为多个锁.

### 11.3 线程引入的开销

对于提升性能而引入的线程来说, 并行带来的性能提升必须超过并发导致的开销.

#### 11.3.1 上下文切换

1. 如果可运行的线程数大于CPU的数量, 那么操作系统最终会将某个正在运行的线程调度出来, 从而使其它线程能够使用CPU, 这将导致一次上下文切换, 这个过程将保存当前线程的上下文, 并将新调度进来的线程执行上下文设置为当前上下文.

上下文切换开销如下:
* 分摊CPU资源: 线程上下文切换需要操作系统和JVM介入, 它们会分摊CPU资源.
* 缓存缺失: 新调度进来的线程需要的数据不在本地缓存中.

2. 线程阻塞(竞争锁, 线程等待, IO阻塞)会导致上下文切换, 无阻塞算法有助于线程上下文切换.

#### 11.3.2 内存同步

1. 不要过度担心非竞争同步带来的开销, 这个基本的机制已经非常快了, 并且JVM能够进行额外的优化以进一步降低或消除开销. 因此, 我们应该将重点放在那些发生锁竞争的地方.

#### 11.3.3 阻塞

1. 阻塞时间过短, 适合使用线程自旋等待方式, 阻塞时间过长, 适合使用线程挂起方式, 这由JVM分析历史等待时间来选择.

2. 被阻塞的线程在其执行时间片未被用完之前就被交换出去, 而在随后当要获取的锁或者资源可用时, 又再次被切换回来.

3. 由于锁竞争而导致阻塞时, 线程在持有锁时将存在一定的开销, 当它释放锁时, 必须告诉操作系统恢复运行阻塞的线程.

---

## 第15章 原子变量与非阻塞同步机制

### 15.1 锁的劣势

1. 多线程竞争锁, 需要挂起和恢复未获取到锁的线程, 需要借助操作系统介入, 存在很大的开销, 现代JVM的锁优化技术并不能全部优化掉.

2. 与锁相比, volatile并不会引起线程上下文切换或线程调度.

3. 如果被阻塞的线程优先级高, 持有锁的线程优先级低, 会发生优先级翻转问题.(不要依赖线程优先级, 它具有不稳定性和平台相关性, 考虑使用优先级队列来实现同样的问题)

4. 如果锁一直被持有, 其它线程无法继续, 线程饥饿问题.

5. 独占锁是一种悲观技术.

### 15.2 硬件对并发的支持

1. 几乎所有的现代处理器中都包含了某种形式的原子读-改-写指令, 例如比较并交换(Compare-and-Swap, CAS)或者关联加载/条件存储(Load-Linked/Store-Conditional), Java1.5之前, Java类中不能直接使用这些指令.

#### 15.2.1 比较并交换(Compare-and-Swap, CAS)

1. CAS需要三个参数: 内存地址V, 进行比较的变量预期值A, 即将写入的变量更新值B.

2. CAS的含义是: 我认为V的值是A, 如果是, 那么将V的值更新为B, 否则不修改并告诉V的值实际是多少, 这一系列操作时原子性的, 并且如果CAS操作失败, 线程并不会挂起阻塞.

3. CAS能检测来自其它线程的干扰

4. CAS通常配合volatile使用, 已达到非阻塞锁的作用.

```java
// 模拟CAS操作
@ThreadSafe
public class SimulatedCAS {
    @GuardedBy("this")
    private int value;

    public synchronized int get() {
        return value;
    }

    public synchronized int compareAndSwap(int expectedValue, int newValue) {
        int oldValue = value;
        if (oldValue == expectedValue)
            value = newValue;
        return oldValue;
    }

    public synchronized boolean compareAndSet(int expectedValue, int newValue) {
        return (expectedValue == compareAndSwap(expectedValue, newValue));
    }
}
```

#### 15.2.2 非阻塞的计数器

```java
@ThreadSafe
public class CasCounter {
    private SimulatedCAS value;

    public int getValue() {
        return value.get();
    }

    public int increment() {
        int v;
        do {
            v = value.get();
        } while (v != value.compareAndSwap(v, v + 1));
        return v + 1;
    }
}
```

1. CAS主要缺点: 它使调用者处理竞争问题(重试, 回退, 放弃), 而在锁中能自动处理竞争问题(线程获取锁之前将一直阻塞), CAS最大的缺陷是难以围绕CAS正确构建外部算法.

2. 虽然和锁相比CAS代码更复杂, 但是CAS不需要执行JVM代码, 而锁需要JVM介入, 总体开销小于锁开销.

3. 经验法则: 在大多数处理器上, 在无竞争的锁获取和释放的**快速代码路径**上的开销, 大约是CAS开销的两倍.

#### 15.2.3 JVM对CAS的支持

1. 引入sun.misc.Unsafe类的本地方法来支持CAS技术.

### 15.3 原子变量类

1. 位于java.util.concurrent.atomic包中, 原子变量类未重写equals和hashCode方法, 不宜用作基于散列容器中的键值.

#### 15.3.1 原子变量是一种更好的volatile

```java
@ThreadSafe
public class CasNumberRange {
    @Immutable
    private static class IntPair {
        // INVARIANT: lower <= upper
        final int lower;
        final int upper;

        public IntPair(int lower, int upper) {
            this.lower = lower;
            this.upper = upper;
        }
    }

    private final AtomicReference<IntPair> values =
            new AtomicReference<>(new IntPair(0, 0));

    public int getLower() {
        return values.get().lower;
    }

    public int getUpper() {
        return values.get().upper;
    }

    public void setLower(int i) {
        while (true) {
            IntPair oldv = values.get();
            if (i > oldv.upper)
                throw new IllegalArgumentException("Can't set lower to " + i + " > upper");
            IntPair newv = new IntPair(i, oldv.upper);
            if (values.compareAndSet(oldv, newv))
                return;
        }
    }

    public void setUpper(int i) {
        while (true) {
            IntPair oldv = values.get();
            if (i < oldv.lower)
                throw new IllegalArgumentException("Can't set upper to " + i + " < lower");
            IntPair newv = new IntPair(oldv.lower, i);
            if (values.compareAndSet(oldv, newv))
                return;
        }
    }
}
```

#### 15.3.2 性能比较: 锁与原子变量

1. 在高强度竞争下, 锁的性能超过原子变量的性能, 且具有更高的可伸缩性, 在中低强度的竞争下, 锁的性能低于原子变量的性能, 且伸缩性也不如原子变量. 大多数程序的锁竞争程度不会太高.

2. 只有完全消除竞争, 才能实现真正的可伸缩性.

### 15.4 非阻塞算法

1. 一个线程的失败或挂起不会导致其它线程也失败或挂起, 这种算法称为非阻塞算法.

2. 构建非阻塞算法的技巧: 将执行原子修改的范围缩小到单个变量上.

#### 15.4.1 非阻塞的栈

```java
@ThreadSafe
public class ConcurrentStack<E> {
    private final AtomicReference<Node<E>> top = new AtomicReference<Node<E>>();

    public void push(E item) {
        Node<E> newHead = new Node<E>(item);
        Node<E> oldHead;
        do {
            oldHead = top.get();
            newHead.next = oldHead;
        } while (!top.compareAndSet(oldHead, newHead));
    }

    public E pop() {
        Node<E> oldHead;
        Node<E> newHead;
        do {
            oldHead = top.get();
            if (oldHead == null)
                return null;
            newHead = oldHead.next;
        } while (!top.compareAndSet(oldHead, newHead));
        return oldHead.item;
    }

    private static class Node<E> {
        public final E item;
        public Node<E> next;

        public Node(E item) {
            this.item = item;
        }
    }
}
```

#### 15.4.2 非阻塞的链表

```java
@ThreadSafe
public class LinkedQueue<E> {

    private static class Node<E> {
        final E item;
        final AtomicReference<Node<E>> next;

        public Node(E item, Node<E> next) {
            this.item = item;
            this.next = new AtomicReference<>(next);
        }
    }

    private final Node<E> dummy = new Node<>(null, null);
    private final AtomicReference<Node<E>> head = new AtomicReference<>(dummy);
    private final AtomicReference<Node<E>> tail = new AtomicReference<>(dummy);

    public boolean put(E item) {
        final Node<E> newNode = new Node<>(item, null);
        while (true) {
            Node<E> curTail = tail.get();
            Node<E> tailNext = curTail.next.get();
            if (curTail == tail.get()) {
                if (tailNext != null)
                    // Queue in intermediate state, advance tail
                    tail.compareAndSet(curTail, tailNext);
                else {
                    // In quiescent state, try inserting new node
                    if (curTail.next.compareAndSet(null, newNode)) {
                        // Insertion succeeded, try advancing tail
                        tail.compareAndSet(curTail, newNode);
                        return true;
                    }
                }
            }
        }
    }
}
```

![15-1](https://github.com/ossaw/notes/blob/master/Pictures/jcip/java-concurrency-in-practice-15-1.jpg)

![15-2](https://github.com/ossaw/notes/blob/master/Pictures/jcip/java-concurrency-in-practice-15-2.jpg)

![15-3](https://github.com/ossaw/notes/blob/master/Pictures/jcip/java-concurrency-in-practice-15-3.jpg)

#### 15.4.3 原子的域更新器

# 待补充

```java
// 待补充
```

#### 15.4.4 ABA问题

1. ABA问题可由java.util.concurrent.atomic.AtomicStampedReference解决.

### 小结

1. 并发性能的主要提升来自于对阻塞算法的使用, JDK中java.util.concurrent包中的大多数类都采用非阻塞算法实现.

---

## 第16章 Java内存模型

> Java 线程中的一些高层设计问题例如安全发布, 同步策略, 一致性等, 这些问题的保证都来自于Java内存模型

### 16.1 什么是内存模型, 为什么需要它

线程为**共享**变量aVailable赋值: aVailable = 3; 在什么条件下此变量值可以被其它线程读取, 而这组条件就是有Java内存模型来保证.

> JMM规定了JVM必须遵循一组保证, 这组保证规定了对变量的写入操作在何时将对于其它线程可见.

#### 16.1.1 平台的内存模型

1. 在共享内存的多处理器体系架构中, 每个处理器都拥有自己的缓存, 并且定期的与主内存进行协调(这种定期协调并不存在一定的规律性, 它是由CPU的忙碌状态来决定, 程序中不要依赖这种随机性), 在不同的处理器架构中提供了不同级别的缓存一致性, 其中一部分只提供最小的保证, 及允许不同的处理器在任意时刻从同一个存储位置上看到不同的值. 操作系统, 编译器以及运行时需要弥合这种在硬件能力与线程安全需求之间的差异.

2. 要想知道每个处理器在任意时刻知道其它处理器正在进行的工作, 需要非常大的开销, 在大多数的情况下, 这个开销通常是不必要的, 因此处理器通常会放宽这种保证, 以换取性能的提升. 但是在一些特殊情况下, 假如两个处理器在对内存中同一变量进行访问时, 其中至少有一个处理器在对其执行写入操作时, 我们就需要这样的存储协调保证, 即使它开销很大. 为此Java定义了一些特殊的指令用来满足这种存储协调保证, 叫做内存栅栏或内存屏障, 当需要共享数据时, 这组指令能够实现额外的存储协调保证, 为了使Java开发人员无需关心在不同物理机器和操作系统中内存模型的差异, Java在此之上提供了自己的内存模型JMM, JVM在适当的位置插入内存屏障来屏蔽在JMM与底层平台内存模型差异, 此类内存屏障指令在Java中以关键字的形式提供给使用者, 如synchronized, volatile, final等.

#### 16.1.2 重排序

> 在单线程的环境中, 无法看到这种底层技术. Java语言规范规定JVM在线程中维护一种串行一致的协议: 只要程序运行的最终结果与严格串行环境中的执行结果相同, 也就是最终一致性, 那么重排序是允许的.

> 当先后两条指令之间不存在依赖时, 会发生重排序, 重排序主要包括以下三方面:

1. 编译器中程序的指令顺序, 可以与源代码中的顺序不同

2. 处理器中可以采用乱序或并行等方式来执行指令

3. 处理器本地缓存可能会改变将写入变量提交到主内存的次序, 而且保存在处理器本地缓存中的值对于其他处理器是不可见的

```java
// 程序16-1
public class PossibleReordering {
    private static int x = 0, y = 0;
    private static int a = 0, b = 0;

    public static void main(String[] args) throws InterruptedException {
        final Thread one = new Thread(() -> {
            a = 1;
            x = b;
        });
        final Thread other = new Thread(() -> {
            b = 1;
            y = a;
        });
        
        one.start();
        other.start();
        one.join();
        other.join();
        System.out.println("( " + x + ", " + y + ")");
    }
}
```

经验证上述程序会打印出(0, 0), 如果不出现重排序的情况, 是不会出现这种结果, 程序运行时序可能如下:

![16-1](https://github.com/ossaw/notes/blob/master/Pictures/jcip/java-concurrency-in-practice-16-1.jpg)

#### 16.1.3 Java内存模型简介

> Java内存模型是通过各种来做定义的, 包括对变量的读写操作, 监视器的获取锁和释放锁操作, 以及线程的启动和合并操作. JMM为程序中所有的操作定义一个偏序关系, 称之为Happen-Before. 要想保证执行操作B的线程看到操作A的结果(无论A和B是否在同一线程中执行), 那么A和B之间必须满足Happen-Before关系. 如果不满足这种关系, 那么JVM可以对它们任意的进行重排序.

The rules for happens-before are:

1. Program order rule. Each action in a thread happens-before every action in that thread that comes later in the program order

2. Monitor lock rule. An unlock on a monitor lock happens-before every subsequent lock on that same monitor lock

3. Volatile variable rule. A write to a volatile field happens-before every subsequent read of that same field

4. Thread start rule. A call to Thread.start on a thread happens-before every action in the started thread

5. Thread termination rule. Any action in a thread happens-before any other thread detects that thread has terminated, either by successfully return from Thread.join or by Thread.isAlive returning false

6. Interruption rule. A thread calling interrupt on another thread happens-before the interrupted thread detects the interrupt (either by having InterruptedException thrown, or invoking isInterrupted or interrupted)

7. Finalizer rule. The end of a constructor for an object happens-before the start of the finalizer for that object

8. Transitivity. If A happens-before B, and B happens-before C, then A happens-before C

> 在多线程访问共享数据时, 至少有一条线程执行写入操作时, 如果读操作和写操作之间没有Happen-Before关系, 那么就会存在数据竞争问题.

> 个人理解: Happen-Before关系时程序安全的保证, 如果程序不满足Happen-Before关系, 根据存在问题的类规范来修正, 以使其满足Happen-Before关系, 在并发程序中可以使用synchronized, volatile, final, 显示锁. 在单线程环境中, 可以调整程序代码顺序.

#### 16.1.4 借助同步

1. 现实中当某个类在其规范中规定它的各个方法之间必须遵循一种Happen-Before关系, 可以借助这种关系, 来实现安全发布. 例如: ReentrantLock规范规定, 线程A释放锁操作Happen-Before线程B获取同一个锁操作.

```java
/**
 * 这段程序是如何保证共享变量sharedVariable的内存可见性?
 */
public class ReentrantLockMemoryVisibilityTest implements Runnable {
    private final ReentrantLock lock = new ReentrantLock();
    private int sharedVariable = 0;

    public int getSharedVariable() {
        return sharedVariable;
    }

    @Override
    public void run() {
        updateSharedVariable();
    }

    public void updateSharedVariable() {
        // 引入局部变量lock, 可以提升性能, 具体参见effective java第三版83条目
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            System.out.println(++sharedVariable);
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        final ExecutorService exec = Executors.newCachedThreadPool();
        final ReentrantLockMemoryVisibilityTest reentrantLockMemoryVisibilityTest = 
				new ReentrantLockMemoryVisibilityTest();
        for (int i = 0; i < 10; i++)
            exec.execute(reentrantLockMemoryVisibilityTest);
        exec.shutdown();

        SECONDS.sleep(1);
        System.out.println("final: " + reentrantLockMemoryVisibilityTest.getSharedVariable());
    }
}
```

jdk1.5推出ReentrantLock显式锁, 可以实现内置锁synchronized相同的内存语义和独占访问. 也就是说以上例子sharedVariable的可见性是由ReentrantLock保证, 可通过ReentrantLock源码分析可以知道是如何保证的内存可见性.

这点根据Happen-Before原则就可推断:

```java
// ReetrantLock.lock()部分代码
protected final boolean tryAcquire(int acquires) {
	// 1. 局部变量, 线程之间不共享
	final Thread current = Thread.currentThread();
	// 2. 读取volatile state值, 此值代表线程持有当前锁的数量(由于可重入的原因)
	int c = getState();
	// 3. 后序其它操作, 包括对共享变量的操作
}

// ReetrantLock.unlock()部分代码
protected final boolean tryRelease(int releases) {
	// 4. setState之前的操作, 包括对共享变量的操作
	// 5. 写入volatile state值
	setState(c);
	return free;
}
```

程序中对sharedVariable的修改在lock与unlock之间, 也就是步骤3到步骤4之间

目前的问题是在线程1在unlock之后, 在线程2执行lock之前是如何保证读取到的sharedVariable是最新值.

1. 线程1修改完共享变量执行unlock操作, 根据程序次序规则, 步骤4Happen-Before步骤5, 步骤5**执行setState(对volatile state的写入操作)**, 此时sharedVariable的可见性可以保持.

2. 线程1释放完全释放锁之后, 线程2执行了lock操作, **先执行getState(对volatile state的读取操作)**, 此处存在一个Happen-Before关系, **线程2对volatile state的写操作是Happen-Before线程1对volatile state读操作的**, 根据volatile变量原则, 此处的内存可见性也可以保持. (这种Happen-Before关系是由ReentrantLock的类规范提供的, 即线程执行unlock操作Happen-Before其它线程执行lock操作, 锁被初次获取时除外, 此时也不会存在数据竞争问题)

3. 综上所述: 线程1执行步骤4Happen-Before线程1执行步骤5, 线程1执行步骤5Happen-Before线程2执行步骤2, 线程2执行步骤2Happen-Before线程执行步骤3, 根据传递性线程1执行步骤4Happen-Before线程2执行步骤3, 所以线程1在步骤4中对共享变量的修改对线程2执行步骤3时是可见的.

4. 整个流程是在单线程内通过**程序次序规则**保证Happen-Before关系, 线程之间采用**volatile变量规则**来保证Happen-Before关系, 最后根据**传递性**来保证线程之间的Happen-Before关系, 所以最终内存可见性得以保持.

5. 这是借助类的规范实现的内存可见性, 这种程序及其脆弱, 它严重依赖于代码中的执行顺序和类的规范中天然存在的Happen-Before关系, 试想一下, 上述程序中步骤4和步骤5代码位置交换或者步骤2和步骤3的代码位置交换, 内存可见性都不会得以维持, 因为他破坏了程序次序规则, 此时再根据第3条推断就会失败. 再或者ReentrantLock的类规范没有保证线程执行unlock操作Happen-Before其它线程执行lock之前, 内存可见性也无法维持, 因为它破坏了volatile变量规则.

6. 借助同步的小技巧: tryAcquire中读取volatile变量的操作在第一行, 方法tryRelease中写入同一volatile变量的操作在最后一行, 且必须保证tryRelease Happen-Before tryAcquire, 其中方法内局部变量可随意安放位置, 因为它们不是线程之间共享的, 比如tryAcquire中的步骤1在getState之前声明, tryRelease中的return free;在setState之后返回, 这都不影响, 当然这种情况下如果还需要原子性也需要你自己来实现了或是加锁(如果加锁的话本身就保证内存可见性了也就没必要借助同步多此一举了)或是使用CAS技术(采用原子类).

### 16.2 发布

> 造成不正确发布的真正原因, 就是在**发布一个对象**与**另一个线程访问该对象**之间缺少一种Happen-Before关系.

#### 16.2.1 不安全的发布

1. 当缺少Happen-Before关系时, 就可能出现重排序问题, 这就解释了为什么在没有充分同步的情况下发布一个对象会导致另一个线程看到一个只被部分构造的对象.

2. 在初始化一个新对象时通常需要写入多个变量, 即对象中的各个域, 同样, 在发布一个引用是也需要写入一个变量, 即新对象的引用, 如果无法确保发布共享引用的操作Happen-Before另一个线程加载该引用, 那么新对象引用的写入操作将与对象中各个域的写入操作重排序, 在这种情况下, 另一个线程可能看到对象引用的最新值, 但同时也将看到对象的某些或全部状态中包含的是无效值, 即一个被部分构造的对象.

3. 错误的延迟初始化将导致不正确的发布.

4. 除不可变对象以外, 使用被另一个线程初始化的对象通常都是不安全的, 除非对象的发布操作是在使用该对象的线程开始使用之前执行.

```java
@NotThreadSafe
public class UnsafeLazyInitialization {
    private static Resource resource;

    public static Resource getInstance() {
        if (resource == null)
            // unsafe publication
            resource = new Resource();
        return resource;
    }

    private static class Resource {}
}
```

5. 上述程序中, 线程A执行new Resource()操作(并非原子操作), 线程B执行resource == null操作, 这两个操作之间不存在Happen-Before关系.

6. 当新分配一个Resource时, 由于在两个线程之间操作存在重排序, 线程B看到线程A中的操作顺序, 可能与线程A执行这些操作时的顺序并不相同, resource = new Resource();这段代码在堆中申请一份空间, 为对象实例字段赋默认值(1), 然后执行构造函数中的属性赋值, 然后将内存地址赋值给resource引用(2), 在(1)和(2)之间可能存在重排序, 导致线程B在执行resource == null操作时可能看到的是一个未完全初始化的对象.

7. **上述问题可能觉得引入volatile变了可以解决, 实则不然, 程序中还存在一个new Resource()非原子操作的问题, 如果说这个操作是原子操作, 那么就没问题, 也就是说要想安全发布原子性和内存可见性必须同时保证**

#### 16.2.2 安全的发布

1. 第三章介绍的安全发布常用模式可以确保被发布对象对于其它线程是可见的.

2. Happen-Before是内存可见性的保证, 要想保证多线程内存可见性, 可依靠**volatile变量规则**和**监视器锁规则**来实现.

#### 16.2.3 安全初始化模式

```java
@ThreadSafe
public class SafeLazyInitialization {
    private static Resource resource;

    public synchronizedd static Resource getInstance() {
        if (resource == null)
            resource = new Resource();
        return resource;
    }

    private static class Resource {}
}
```

1. 16.2.2节中最后一点存在的问题可以用synchronized关键字来保证, 但是引入synchronized关键字增加程序中串行执行部分, 会限制程序并行执行能力和吞吐量, 降低可伸缩性.

```java
@ThreadSafe
public class EagerInitialization {
    private static Resource resource = new Resource();

    public static Resource getResource() {
        return resource;
    }

    private static class Resource {}
}
```

2. 通过提前初始化可以避免调用getResource时产生的同步开销.

```java
@ThreadSafe
public class ResourceFactory {
    private static class ResourceHolder {
        public static Resource resource = new Resource();
    }

    public static Resource getResource() {
        return ResourceFactory.ResourceHolder.resource;
    }

    private static class Resource {}
}
```

3. 使用延长初始化占位类模式即可以达到延迟初始化, 又避免同步开销. JVM将推迟ResourceHolder的初始化操作, 直到开始使用时才初始化, 通过一个静态初始化来初始化Resource, 因此不需要同步. (建议采用此种方式)

#### 16.2.4 双重检查加锁

```java
@NotThreadSafe
public class DoubleCheckedLocking {
    private static Resource resource;

    public static Resource getInstance() {
        if (resource == null)
            synchronized (DoubleCheckedLocking.class) {
                if (resource == null)
                    resource = new Resource();
            }
        return resource;
    }

    private static class Resource {}
}
```

1. 双重检查由于重排序的问题, 线程可能看到一个仅被部分构造的resource.

2. 在Java1.5及更高版本中可通过将resource声明为volatile来修正上述问题, Java1.5之后的版本中增强了volatile的语义, 这样做也印证了16.2.1节中最后一点, DCL中的synchronized保证块内new Resource的原子性.

3. 不建议使用此种方式, 较延迟初始化占位类模式相比, DCL模式代码过于复杂且更脆弱.

### 16.3 初始化过程中的安全性

> 如果能确保初始化过程中的安全性, 那么就可以使得被正确构造的不可变对象在没有同步的情况下也能安全的在多个线程之间共享, 而不管它们是如何发布的, 甚至是通过竞争发布. (如果Resource是不可变的, 那么UnsafeLazyInitialization实际上是安全的)

1. 初始化安全性将确保, 对于被正确构造的对象, 所有线程都能看到由构造函数为对象各个**final**域设置的正确值, 而不管采用何种方式来发布对象, 而且对于可以通过被正确构造对象中某个final域到达的任意变量(例如某个final数组中的元素或者由一个final域引用的HashMap的键值)将同样对于其它线程是可见的.

2. **对于含有final域的对象, 初始化安全性可防止对该对象的初始引用被重排序到构造过程之前**, 当构造函数完成时, 构造函数对final域的所有写入操作, 以及通过这些域可达的所有变量的写入操作, 都将被冻结, 并且任何获得该对象引用的线程都至少能确保看到被冻结的值. 对于通过final域可到达的初始变量的写入操作, 将不会与构造过程后的操作一起被重排序.

```java
@ThreadSafe
public class SafeStates {
    private final Map<String, String> states;

    public SafeStates() {
        states = new HashMap<String, String>();
        states.put("alaska", "AK");
        states.put("alabama", "AL");
        states.put("wyoming", "WY");
    }

    public String getAbbreviation(String s) {
        return states.get(s);
    }
}
```

3. 许多对SaeState的修改都可能破坏它的线程安全性, 例如states不是final类型或者在构造函数之外的方法能修改states, 再或者在SafeStates初始化过程中还有其它的非final域.

4. 初始化安全性只能保证final域可达的值从构造过程完成时开始的可见性. 对于通过非final域可达的值, 或者在构造过程完成后, 可能改变的值, 必须采用同步来确保可见性.

### 小结

**Java内存模型说明了某个线程的内存操作在哪些情况下对于其它线程是可见的. 其中包括这些操作是按照一种Happen-Before的偏序关系进行排序, 而这种关系时基于内存操作和同步操作等级别来定义的. 如果缺少充足的同步, 那么当线程访问共享数据时, 会发生一些奇怪的问题. 然而, 如果使用第2章和第三章介绍的更高级规则, 例如@GuardedBy和安全发布, 那么即使不考虑Happen-Before的底层细节, 也能确保线程安全性.**
