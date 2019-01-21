# Java 并发编程实战 读书笔记

## 第1章 简介

### 1.1 并发简史

> 计算机中加入操作系统来实现多个程序的同事执行, 主要基于以下原因: 
1. 资源利用率: 例如在等待IO操作时CPU资源将闲置, 引入并发将提高CPU利用率
&nbsp;
2. 公平性: 不同用户和程序对于计算机的上资源有着同等的使用权.
&nbsp;
3. 便利性: 在计算多个任务时, 应该编写多个程序, 每个程序执行一个任务在必要时进行通信, 这比编写一个程序来计算所有任务更容易实现.

### 1.2 线程的优势

1. 发挥多处理器的强大能力
&nbsp;
2. 建模的简单性
&nbsp;
3. 异步事件的简化处理
&nbsp;
4. 响应更灵敏的用户界面

### 1.3 线程带来的风险

1. 安全性问题: 永远不发生糟糕的事件
&nbsp;
2. 活跃性问题: 某个正确的事件最终会发生(具有随机性)
&nbsp;
3. 性能问题: 多线程将增加线程上下文切换, 线程调度, 共享内存总线的同步流量等开销, 尽量保证这些开销总是小于引入多线程所带来的性能的提升

## 第16章 Java内存模型

> Java 线程中的一些高层设计问题例如安全发布, 同步策略, 一致性等, 这些问题的保证都来自于Java内存模型

### 16.1 什么是内存模型, 为什么需要它

线程为**共享**变量aVailable赋值: aVailable = 3; 在什么条件下此变量值可以被其它线程读取, 而这组条件就是有Java内存模型来保证.

> JMM规定了JVM必须遵循一组保证, 这组保证规定了对变量的写入操作在何时将对于其它线程可见.

#### 16.1.1 平台的内存模型

1. 在共享内存的多处理器体系架构中, 每个处理器都拥有自己的缓存, 并且定期的与主内存进行协调(这种定期协调并不存在一定的规律性, 它是由CPU的忙碌状态来决定, 程序中不要依赖这种规律), 在不同的处理器架构中提供了不同级别的缓存一致性, 其中一部分只提供最小的保证, 及允许不同的处理器在任意时刻从同一个存储位置上看到不同的值. 操作系统, 编译器以及运行时需要你弥合这种在硬件能力与线程安全需求之间的差异.
&nbsp;
2. 要想知道每个处理器在任意时刻知道其它处理器正在进行的工作, 需要非常大的开销, 在大多数的情况下这个开销通常是不必要的, 因此处理器通常会放宽这种保证, 以换取性能的提升. 但是在一些特殊情况下, 假如两个处理器在对内存中同一变量进行访问时, 其中至少有一个处理器在对其执行写入操作时, 我们就需要这样的存储协调保证, 即使它开销很大. 为此Java定义了一些特殊的指令用来满足这种存储协调保证, 叫做内存栅栏或内存屏障, 当需要共享数据时, 这组指令能够实现额外的存储协调保证, 为了使Java开发人员无需关心在不同物理机器和操作系统中内存模型的差异, Java在此之上提供了自己的内存模型JMM, JVM在适当的位置插入内存屏障来屏蔽在JMM与底层平台内存模型差异, 此类内存屏障指令在Java中以关键字的形式提供给使用者, 如synchronize, volatile, final等.

#### 16.1.2 重排序

> 在单线程的环境中, 无法看到这种底层技术. Java语言规范规定JVM在线程中维护一种串行一致的协议: 只要程序运行的最终结果与严格串行环境中的执行结果相同, 也就是最终一致性, 那么重排序是允许的.

> 当先后两条指令之间不存在依赖时, 会发生重排序, 重排序主要包括以下三方面:

1. 编译器中程序的指令顺序, 可以与源代码中的顺序不同
&nbsp;
2. 处理器中可以采用乱序或并行等方式来执行指令
&nbsp;
3. 处理器本地缓存可能会改变将写入变量提交到主内存的次序, 而且保存在处理器本地缓存中的值对于其他处理器是不可见的

```java
// 程序清单16-1
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
![Alt text](https://github.com/ossaw/notes/blob/master/pictures/java%20concurrency%20in%20practice%2016-1.jpg?raw=true)

#### 16.1.3 Java内存模型简介

> Java内存模型是通过各种来做定义的, 包括对变脸的读/写操作, 监视器的加锁和释放操作, 以及线程的启动和合并操作. JMM为程序中所有的操作定义一个偏序关系, 叫做Happen-Before. 要想保证执行操作B的线程看到操作A的结果(无论A和B是否在同一线程中执行), , 那么A和B之间必须满足Happen-Before关系. 如果不满足这种关系, 那么JVM可以对它们任意的进行重排序.

Happen-Before规则如下:
&nbsp;
1. 程序顺序规则: 如果程序中操作A在操作B之前, 那么在线程中A操作将在B操作之前发生
&nbsp;
2. 监视器规则: 在监视器锁上的解锁操作必须在同一件监视器上的加锁操作之前发生
&nbsp;
3. volatile变量规则: 对volatile变量的写操作必须在对此变量的读操作之前执行
&nbsp;
4. 线程启动规则: 在线程上对Thread.start()的调用必须在该线程中执行任何操作之前发生
&nbsp;
5. 线程结束规则: 线程终结规则: 线程中所有的操作都先行发生于线程的终止检测, 我们可以通过Thread.join()方法结束, Thread.isAlive()的返回值手段检测到线程已经终止执行
&nbsp;
6. 中断规则: 当一个线程在另一个线程上调用interrupt时, 必须在被中断线程监测到interrupt调用之前执行(通过抛出InterruptedException, 或者调用isInterrupted和interrupted)
&nbsp;
7. 终结器规则: 对象的构造函数必须在启动该对象的终结器之前执行
&nbsp;
8. 传递性: 如果操作A在操作B之前执行, 操作B在操作C之前执行, 那么操作A在操作C之前执行

***

The rules for happens-before are:
&nbsp;
1. Program order rule. Each action in a thread happens-before every action in that thread that comes later in the program order
&nbsp;
2. Monitor lock rule. An unlock on a monitor lock happens-before every subsequent lock on that same monitor lock
&nbsp;
3. Volatile variable rule. A write to a volatile field happens-before every subsequent read of that same field
&nbsp;
4. Thread start rule. A call to Thread.start on a thread happens-before every action in the started thread
&nbsp;
5. Thread termination rule. Any action in a thread happens-before any other thread detects that thread has terminated, either by successfully return from Thread.join or by Thread.isAlive returning false
&nbsp;
6. Interruption rule. A thread calling interrupt on another thread happens-before the interrupted thread detects the interrupt (either by having InterruptedException thrown, or invoking isInterrupted or interrupted)
&nbsp;
7. Finalizer rule. The end of a constructor for an object happens-before the start of the finalizer for that object
&nbsp;
8. Transitivity. If A happens-before B, and B happens-before C, then A happens-before C

> 在多线程访问共享数据时, 至少有一条线程执行写入操作时, 如果读操作和写操作之间没有Happen-Before关系, 那么就会存在数据竞争问题.

> 程序不满足Happen-Before的情况下, 要对程序进行修改以使它满足此规则, 在多线程环境中或是使用synchronize, 或是使用volatile, 在单线程环境中, 调整程序代码顺序满足, 总之Happen-Before原则不必全部满足, 但是也不可以一条原则都不满足

#### 16.1.4 借助同步

1. 通过现有Happen-Before规则, 可以借助现有同步机制来实现可见性属性. (个人理解: 在无锁的场景下实现内存可见性)

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

jdk1.5推出ReentrantLock显式锁, 可以达到内置锁synchronize相同的内存语义和独占访问. 也就是说以上例子sharedVariable的可见性是由ReentrantLock保证, 可通过ReentrantLock源码分析得知是如何保证的内存可见性.
&nbsp;
这点根据Happen-Before原则就可推断 :

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

程序中对sharedVariable的修改修改在lock与unlock之间, 也就是步骤3到步骤4之间
&nbsp;
目前的问题是在线程1在unlock之后, 在线程2执行lock之前是如何保证读取到的sharedVariable是最新值.
&nbsp;
1. 线程1修改完共享变量执行unlock操作, 根据程序次序规则, 步骤4Happen-Before步骤5, 步骤5**执行setState(对volatile值的写入操作)**, 此时sharedVariable的可见性可以保持.
&nbsp;
2. 线程1释放完全释放锁之后, 线程2执行了lock操作, **先执行getState(对volatile值的读取操作)**, 此处存在一个Happen-Before关系, **线程2对volatile state的写操作是Happen-Before线程1对volatile state读操作的**, 根据volatile变量原则, 此处的内存可见性也是可以维持的. (这种Happen-Before关系是由ReentrantLock的类规范提供的, 即线程执行unlock操作Happen-Before其它线程执行lock操作, 在线程lock时getState=0的情况下除外)
&nbsp;
3. 综上所述: 线程1执行步骤4Happen-Before线程1执行步骤5, 线程1执行步骤5Happen-Before线程2执行步骤2, 线程2执行步骤2Happen-Before线程执行步骤3, 根据传递性线程1执行步骤4Happen-Before线程2执行步骤3, 所以线程1在步骤4中对共享变量的修改对线程2执行步骤3时是可见的.
&nbsp;
4. 整个流程是在单线程内通过**程序次序规则**保证Happen-Before关系, 线程之间采用**volatile变量规则**来保证Happen-Before关系, 最后采用**传递性**来保证线程之间的Happen-Before关系, 所以最终内存可见性得以维持.
&nbsp;
5. 这是一种在无锁定的情况下实现的内存可见性, 这种程序及其脆弱, 它严重依赖于代码中的执行顺序和类的规范中天然存在的一种Happen-Before关系, 试想一下, 上述程序中步骤4和步骤5代码位置交换或者步骤2和步骤3的代码位置交换, 内存可见性都不会得以维持, 因为他破坏了程序次序规则, 此时再根据第3条推断就会失败. 再或者ReentrantLock的类规范没有保证线程执行unlock操作Happen-Before其它线程执行lock之前, 内存可见性也无法维持, 因为它破坏了volatile变量规则.
&nbsp;
6. 借助同步的小技巧: tryAcquire中读取volatile变量的操作在第一行, 方法tryRelease中写入同一volatile变量的操作在最后一行, 且必须保证tryRelease在tryAcquire之前执行, 其中方法内局部变量可随意安放位置, 因为它们不是线程之间共享的, 比如tryAcquire中的步骤1在getState之前声明, tryRelease中的return free;在setState之后返回, 这都不影响, 当然这种情况下如果还需要原子性也需要你自己来实现了或是加锁(如果加锁的话本身就保证内存可见性了也就没必要借助同步多此一举了)或是使用CAS技术.

### 16.2 发布

> 造成不正确发布的真正原因, 就是在**发布一个对象**与**另一个线程访问该对象**之间缺少一种Happen-Before关系.

#### 16.2.1 不安全的发布

1. 当缺少Happen-Before关系时, 就可能出现重排序问题, 这就解释了为什么在没有充分同步的情况下发布一个对象会导致另一个线程看到一个只被部分构造的对象.
&nbsp;
2. 在初始化一个新对象时通常需要写入多个变量, 即对象中的各个域, 同样, 在发布一个引用是也需要写入一个变量, 即新对象的引用, 如果无法确保发布共享引用的操作Happen-Before另一个线程加载该引用, 那么新对象引用的写入操作将与对象中各个域的写入操作重排序, 在这种情况下, 另一个线程可能看到对象引用的最新值, 但同时也将看到对象的某些或全部状态中包含的是无效值, 即一个被部分构造的对象.
&nbsp;
3. 错误的延迟初始化将导致不正确的发布.
&nbsp;
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

5. 上述程序中, 线程A执行new Resource()操作(其并非原子操作), 线程B执行resource == null操作, 这两个操作之间不存在Happen-Before关系.
&nbsp;
6. 当新分配一个Resource时, 由于在两个线程之间操作存在重排序, 线程B看到线程A中的操作顺序, 可能与线程A执行这些操作时的顺序并不相同, resource = new Resource();这段代码在堆中申请一份空间, 为对象实例字段赋默认值(1), 然后执行构造函数中的属性赋值, 然后将内存地址赋值给resource引用(2), 在(1)和(2)之间可能存在重排序, 导致线程B在执行resource == null操作时可能看到的是一个未完全初始化的对象.
&nbsp;
7. **上述问题可能觉得引入volatile变了可以解决, 实则不然, 程序中还存在一个new Resource()非原子操作的问题, 如果说这个操作是原子操作, 那么就没问题, 也就是说要想安全发布原子性和内存可见性必须同时保证**

#### 16.2.2 安全的发布

1. 第三章介绍的安全发布常用模式可以确保被发布对象对于其它线程是可见的.
&nbsp;
2. Happen-Before是内存可见性的保证, 要想保证多线程内存可见性, 可依靠**volatile变量规则**和**监视器锁规则**来实现.

#### 16.2.3 安全初始化模式

```java
@ThreadSafe
public class SafeLazyInitialization {
    private static Resource resource;

    public synchronized static Resource getInstance() {
        if (resource == null)
            resource = new Resource();
        return resource;
    }

    private static class Resource {}
}
```

1. 16.2.2节中最后一点存在的问题恰好可以用synchronize关键字来保证, 但是引入synchronize关键字增加程序中串行执行部分, 会限制程序并行执行能力和吞吐量, 降低可伸缩性.

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

3. 使用延长初始化占位类模式可达到即延迟初始化, 有避免同步开销. JVM将推迟ResourceHolder的初始化操作, 直到开始使用时才初始化, 通过一个静态初始化来初始化Resource, 因此不需要同步. (建议采用此种方式)

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

1. 双重检查存在的问题在16.2.1中介绍过, 线程可能看到一个仅被部分构造的resource.
&nbsp;
2. 在Java1.5及更高版本中可通过将resource声明为volatile来修正上述问题, Java1.5之后的版本中增强了volatile的语义, 这样做也正好印证了16.2.1节中最后一点, DCL中的synchronize正好保证了new Resource的原子性.
&nbsp;
3. 实际环境中不建议使用此种方式, 较延迟初始化占位类模式相比, DCL模式代码过于复杂且更脆弱.

### 16.3 初始化过程中的安全性

> 如果能确保初始化过程中的安全性, 那么就可以使得被正确构造的不可变对象在没有同步的情况下也能安全的在多个线程之间共享, 而不管它们是如何发布的, 甚至是通过竞争发布. (如果Resource是不可变得, 那么UnsafeLazyInitialization实际上是安全的)

1. 初始化安全性将确保, 对于被正确构造的对象, 所有线程都能看到由构造函数为对象各个**final**域设置的正确值, 而不管采用何种方式来发布对象, 而且对于可以通过被正确构造对象中某个final域到达的任意变量(例如某个final数组中的元素, 或者由一个final域引用的HashMap的内容)将同样对于其他线程是可见的.
&nbsp;
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

3. 许多对SaeState的修改都可能破坏它的线程安全性, 例如states不是final类型, 或者在构造函数之外的方法能修改states, 再或者在SafeStates初始化过程中还有其它的非final域.
&nbsp;
4. 初始化安全性只能保证final域可达的值从构造过程完成时开始的可见性. 对于通过非final域可达的值, 或者在构造过程完成后, 可能改变的值, 必须采用同步来确保可见性.

### 小结

**Java内存模型说明了某个线程的内存操作在哪些情况下对于其它线程是可见的. 其中包括这些操作是按照一种Happen-Before的偏序关系进行排序, 而这种关系时基于内存操作和同步操作等级别来定义的. 如果缺少充足的同步, 那么当线程访问共享数据时, 会发生一些奇怪的问题. 然而, 如果使用第2章和第三章介绍的更高级规则, 例如@GuardedBy和安全发布, 那么即使不考虑Happen-Before的底层细节, 也能确保线程安全性.**
