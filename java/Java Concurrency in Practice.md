# Java 并发编程实战 读书笔记

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

目前的问题是在线程1在unlock之后, 在线程2执行lock之前是如何保证读取到的sharedVariable是最新值.

1. 线程1修改完共享变量执行unlock操作, 根据程序次序规则, 步骤4Happen-Before步骤5, 步骤5**执行setState(对volatile值的写入操作)**, 此时sharedVariable的可见性可以保持.

2. 线程1释放完全释放锁之后, 线程2执行了lock操作, **先执行getState(对volatile值的读取操作)**, 此处存在一个Happen-Before关系, **线程2对volatile state的写操作是Happen-Before线程1对volatile state读操作的**, 根据volatile变量原则, 此处的内存可见性也是可以维持的. (这种Happen-Before关系是由ReentrantLock的类规范提供的, 即线程执行unlock操作Happen-Before其它线程执行lock操作, 在线程lock时getState=0的情况下除外)

3. 综上所述: 线程1执行步骤4Happen-Before线程1执行步骤5, 线程1执行步骤5Happen-Before线程2执行步骤2, 线程2执行步骤2Happen-Before线程执行步骤3, 根据传递性线程1执行步骤4Happen-Before线程2执行步骤3, 所以线程1在步骤4中对共享变量的修改对线程2执行步骤3时是可见的.

4. 整个流程是在单线程内通过**程序次序规则**保证Happen-Before关系, 线程之间采用**volatile变量规则**来保证Happen-Before关系, 最后采用**传递性**来保证线程之间的Happen-Before关系, 所以最终内存可见性得以维持.

5. 这是一种在无锁定的情况下实现的内存可见性, 这种程序及其脆弱, 它严重依赖于代码中的执行顺序和类的规范中天然存在的一种Happen-Before关系, 试想一下, 上述程序中步骤4和步骤5代码位置交换或者步骤2和步骤3的代码位置交换, 内存可见性都不会得以维持, 因为他破坏了程序次序规则, 此时再根据第3条推断就会失败. 再或者ReentrantLock的类规范没有保证线程执行unlock操作Happen-Before其它线程执行lock之前, 内存可见性也无法维持, 因为它破坏了volatile变量规则.

6. 借助同步的小技巧: tryAcquire中读取volatile变量的操作在第一行, 方法tryRelease中写入同一volatile变量的操作在最后一行, 且必须保证tryRelease在tryAcquire之前执行, 其中方法内局部变量可随意安放位置, 因为它们不是线程之间共享的, 比如tryAcquire中的步骤1在getState之前声明, tryRelease中的return free;在setState之后返回, 这都不影响.
