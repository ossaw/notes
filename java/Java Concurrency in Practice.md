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

** Happen-Before规则如下: **
&nbsp;
** 程序顺序规则: 如果程序中操作A在操作B之前, 那么在线程中A操作将在B操作之前发生 **
&nbsp;
** 监视器规则: 在监视器锁上的解锁操作必须在同一件监视器上的加锁操作之前发生 **
&nbsp;
** 3. volatile变量规则: 对volatile变量的写操作必须在对此变量的读操作之前执行 **
&nbsp;
** 4. 线程启动规则: 在线程上对Thread.start()的调用必须在该线程中执行任何操作之前发生 **
&nbsp;
** 5: 线程结束规则: 线程终结规则: 线程中所有的操作都先行发生于线程的终止检测, 我们可以通过Thread.join()方法结束, Thread.isAlive()的返回值手段检测到线程已经终止执行 **
&nbsp;
** 6. 中断规则: 当一个线程在另一个线程上调用interrupt时, 必须在被中断线程监测到interrupt调用之前执行(通过抛出InterruptedException, 或者调用isInterrupted和interrupted) **
&nbsp;
** 7. 终结器规则: 对象的构造函数必须在启动该对象的终结器之前执行 **
&nbsp;
** 8. 传递性: 如果操作A在操作B之前执行, 操作B在操作C之前执行, 那么操作A在操作C之前执行 **
&nbsp;
** The rules for happens-before are: **
&nbsp;
** 1. Program order rule. Each action in a thread happens-before every action in that thread that comes later in the program order **
&nbsp;
** 2. Monitor lock rule. An unlock on a monitor lock happens-before every subsequent lock on that same monitor lock **
&nbsp;
** 3. Volatile variable rule. A write to a volatile field happens-before every subsequent read of that same field **
&nbsp;
** 4. Thread start rule. A call to Thread.start on a thread happens-before every action in the started thread **
&nbsp;
** 5. Thread termination rule. Any action in a thread happens-before any other thread detects that thread has terminated, either by successfully return from Thread.join or by Thread.isAlive returning false **
&nbsp;
** 6. Interruption rule. A thread calling interrupt on another thread happens-before the interrupted thread detects the interrupt (either by having InterruptedException thrown, or invoking isInterrupted or interrupted) **
&nbsp;
** 7. Finalizer rule. The end of a constructor for an object happens-before the start of the finalizer for that object **
&nbsp;
** 8. Transitivity. If A happens-before B, and B happens-before C, then A happens-before C **

> 在多线程访问共享数据时, 至少有一条线程执行写入操作时, 如果读操作和写操作之间没有Happen-Before关系, 那么就会存在数据竞争问题.

> 程序不满足Happen-Before的情况下, 要对程序进行修改以使它满足此规则, 在多线程环境中或是使用synchronize, 或是使用volatile, 在单线程环境中, 调整程序代码顺序满足, 总之Happen-Before原则不必全部满足, 但是也不可以一条原则都不满足

#### 16.1.4 借助同步
