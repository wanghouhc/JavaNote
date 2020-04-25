# JUC多线程 (二)

## 8 Volatile

### 8.1 目标

- 掌握volatile的使用方法和场景
- 理解volatile的作用原理

### 8.2 路径

1. 解决内存可见性的问题
2. 原子性的问题
3. volatile的适用场景
4. synchronized和volatile的对比

### 8.3 讲解

​	通过前面内容我们了解了synchronized，虽然JVM对它做了很多优化，但是它还是一个重量级的锁。而接下来要介绍的volatile则是轻量级的synchronized。如果一个变量使用volatile，则它比使用synchronized的成本更加低，因为它不会引起线程上下文的切换和调度。

Java语言规范对volatile的定义如下：

``Java允许线程访问共享变量，为了确保共享变量能被准确和一致地更新，线程应该确保通过排他锁单独获得这个变量。``

​	通俗点讲就是说一个变量如果用volatile修饰了，则Java可以确保所有线程看到这个变量的值是一致的，如果某个线程对volatile修饰的共享变量进行更新，那么其他线程可以立马看到这个更新，这就是内存可见性。

volatile虽然看起来比较简单，使用起来无非就是在一个变量前面加上volatile即可，但是要用好并不容易。



#### 1 解决内存可见性问题

##### 1.1 缓存一致协议保证读到最新值

设置-XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly 可以查看执行过程中的汇编码

````
汇编代码：

0x01a3de1d: movb $0x0,0x1104800(%esi);

0x01a3de24: lock  addl $0x0,(%esp);
````

有volatile变量修饰的共享变量进行**写操作**的时候会多第二行汇编代码，通过查IA-32架构软件开发者手册可知，lock前缀的指令在多核处理器下会引发了两件事情。

- 将当前处理器缓存行的数据会写回到系统内存。
- 这个写回内存的操作会引起在其他CPU里缓存了该内存地址的数据无效。

处理器为了提高处理速度，不直接和内存进行通讯，而是先将系统内存的数据读到内部缓存（L1,L2或其他）后再进行操作，但操作完之后不知道何时会写到内存，如果对**声明了Volatile变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存**。但是就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题，所以在多处理器下，为了保证各个处理器的缓存是一致的，就会实现**缓存一致性协议**，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是**过期**了，当处理器发现自己缓存行对应的**内存地址**被修改，就会将当前处理器的**缓存行设置**成**无效状态**，当处理器要对这个数据进行修改操作的时候，会**强制**重新从**系统内存**里把数据读到**处理器缓存**里。

这两件事情在IA-32软件开发者架构手册的第三册的多处理器管理章节（第八章）中有详细阐述。

Lock前缀指令会引起处理器缓存回写到内存 。Lock前缀指令导致在执行指令期间，声言处理器的 LOCK# 信号。在多处理器环境中，LOCK# 信号确保在声言该信号期间，处理器可以独占使用任何共享内存。（因为它会锁住总线，导致其他CPU不能访问总线，不能访问总线就意味着不能访问系统内存），但是在最近的处理器里，LOCK＃信号一般不锁总线，而是锁缓存，毕竟锁总线开销比较大。在8.1.4章节有详细说明锁定操作对处理器缓存的影响，对于Intel486和Pentium处理器，在锁操作时，总是在总线上声言LOCK#信号。但在P6和最近的处理器中，如果访问的内存区域已经缓存在处理器内部，则不会声言LOCK#信号。相反地，它会锁定这块内存区域的缓存并回写到内存，并使用缓存一致性机制来确保修改的原子性，此操作被称为“缓存锁定”，缓存一致性机制会阻止同时修改被两个以上处理器缓存的内存区域数据 。

一个处理器的缓存回写到内存会导致其他处理器的缓存无效 。IA-32处理器和Intel 64处理器使用MESI（修改，独占，共享，无效）控制协议去维护内部缓存和其他处理器缓存的一致性。在多核处理器系统中进行操作的时候，IA-32 和Intel 64处理器能嗅探其他处理器访问系统内存和它们的内部缓存。它们使用嗅探技术保证它的内部缓存，系统内存和其他处理器的缓存的数据在总线上保持一致。例如在Pentium和P6 family处理器中，如果通过嗅探一个处理器来检测其他处理器打算写内存地址，而这个地址当前处理共享状态，那么正在嗅探的处理器将无视它的缓存行，在下次访问相同内存地址时，强制执行缓存行填充。

##### 1.2 内存屏障防止指令重排

写操作时，通过在写操作指令后加入一条store屏障指令，让本地内存中变量的值能够刷新到主内存中

读操作时，通过在读操作前加入一条load屏障指令，及时读取到变量在主内存的值

``PS: 内存屏障（Memory Barrier）是一种CPU指令，用于控制特定条件下的重排序和内存可见性问题。Java编译器也会根据内存屏障的规则禁止重排序``



volatile的底层实现是通过插入内存屏障，但是对于编译器来说，发现一个最优布置来最小化插入内存屏障的总数几乎是不可能的，所以，JMM采用了保守策略。如下：

- StoreStore屏障可以保证在volatile写之前，其前面的所有普通写操作都已经刷新到主内存中。

- StoreLoad屏障的作用是避免volatile写与后面可能有的volatile读/写操作重排序。

- LoadLoad屏障用来禁止处理器把上面的volatile读与下面的普通读重排序。

- LoadStore屏障用来禁止处理器把上面的volatile读与下面的普通写重排序。

  

#### 2 原子性的问题

虽然Volatile 关键字可以让变量在多个线程之间可见，但是Volatile不能保证对修饰变量的操作是原子性的。

```java
public class Demo3Volatile {

    public static void main(String[] args) throws InterruptedException {
        VolatileDemo demo = new VolatileDemo();

        for (int i = 0; i < 5; i++) {
            Thread t = new Thread(demo);
            t.start();
        }

        Thread.sleep(1000);
        System.out.println(demo.count);
    }

    static class VolatileDemo implements Runnable {
        public volatile int count;
        //public volatile AtomicInteger count = new AtomicInteger(0);

        public void run() {
            addCount();
        }

        public void addCount() {
            for (int i = 0; i < 10000; i++) {
                count++;
            }
        }
    }
}
```



以上出现原子性问题的原因是count++并不是原子性操作。（ 线程通过JMM规范volatile，都会去内存读取最新值，但是不会再去读取第二次，而由于count++的读写分离，两个线程呢个都+1，都是6），一步能操作完成的一般具有原子性

count = 5 开始，流程分析：

1. 线程1读取count的值为5 
2. 线程2读取count的值为5
3. 线程2加1操作
4. 线程2最新count的值为6
5. 线程2写入值到主内存的最新值为6

这个时候，线程1的count为5，线程2的count为6

如果切换到线程1执行，那么线程1得到的结果是6，写入到主内存的值还是6

现在的情况是对count进行了两次加1操作，但是主内存实际上只是加1一次



解决方案：

1. 使用synchronized
2. 使用ReentrantLock（可重入锁）
3. 使用AtomicInteger（原子操作）

使用synchronized

```java
public synchronized void addCount() {
    for (int i = 0; i < 10000; i++) {
        count++;
    }
}
```



使用ReentrantLock（可重入锁）

```java
//可重入锁
private Lock lock = new ReentrantLock();

public void addCount() {
    for (int i = 0; i < 10000; i++) {
        lock.lock();
        count++;
        lock.unlock();
    }
}
```



使用AtomicInteger（原子操作）

```java
public static AtomicInteger count = new AtomicInteger(0);
public void addCount() {
    for (int i = 0; i < 10000; i++) {
        //count++;
        count.incrementAndGet();
    }
}
```



#### 3 Volatile 适合使用场景（能够解决可见性，和一定程度的有序性，但是不能保证原子性）

a）对变量的写入操作不依赖其当前值

​    	不满足：number++、count=count*5等

​    	满足：boolean变量、直接赋值的变量等

b）该变量没有包含在具有其他变量的不变式中

​	不满足：不变式 low<up

总结：对变量的操作本身是原子性的操作，就可以使用volatile 

扩展：

````txt
Java中的原子操作包括：
1）除long和double之外的基本类型的赋值操作
2）所有引用reference的赋值操作
3）java.util.concurrent.Atomic.* 包中所有类的一切操作。

但是java对long和double的赋值操作是非原子操作！！long和double占用的字节数都是8，也就是64bits。在32位操作系统上对64位的数据的读写要分两步完成，每一步取32位数据。这样对double和long的赋值操作就会有问题：如果有两个线程同时写一个变量内存，一个进程写低32位，而另一个写高32位，这样将导致获取的64位数据是失效的数据。因此需要使用volatile关键字来防止此类现象。volatile本身不保证获取和设置操作的原子性，仅仅保持修改的可见性。但是java的内存模型保证声明为volatile的long和double变量的get和set操作是原子的。
````



#### 4 synchronized和volatile比较

a）volatile不需要加锁，比synchronized更轻便，不会阻塞线程

b）synchronized既能保证可见性，又能保证原子性，而volatile只能保证可见性，无法保证原子性



​	与锁相比，Volatile 变量是一种非常简单但同时又非常脆弱的同步机制，它在某些情况下将提供优于锁的性能和伸缩性。如果严格遵循 volatile 的使用条件（**对变量的操作是原子性的** ） 在某些情况下可以使用 volatile 代替 synchronized 来优化代码提升效率。



### 8.4 小结



​	



## 9 J.U.C之CAS

### 9.1 目标

- 理解cas的原理以及存在的缺陷

### 9.2 路径

1. cas介绍
2. cas原理剖析
3. 多cpu的cas处理
4. cas的缺陷

### 9.3 讲解

J.U.C 即 java.util.concurrent，是 JSR 166 标准规范的一个实现； JSR 166 以及 J.U.C 包的作者是 Doug Lea 。



J.U.C 框架是 Java 5 中引入的，而我们最熟悉的线程池机制就在这个包，J.U.C 框架包含的内容有：

- AbstractQueuedSynchronizer（AQS框架），J.U.C 中实现锁和同步机制的基础；
- Locks & Condition（锁和条件变量），比 synchronized、wait、notify 更细粒度的锁机制；
- Executor 框架（线程池、Callable、Future），任务的执行和调度框架；
- Synchronizers（同步器），主要用于协助线程同步，有 CountDownLatch、CyclicBarrier、Semaphore、Exchanger；
- Atomic Variables（原子变量），方便程序员在多线程环境下，无锁的进行原子操作，核心操作是 CAS 原子操作，所谓的 CAS 操作，即 compare and swap，指的是将预期值与当前变量的值比较(compare)，如果相等则使用新值替换(swap)当前变量，否则不作操作；
- BlockingQueue（阻塞队列），阻塞队列提供了可阻塞的入队和出对操作，如果队列满了，入队操作将阻塞直到有空间可用，如果队列空了，出队操作将阻塞直到有元素可用；
- Concurrent Collections（并发容器），说到并发容器，不得不提同步容器。在 JDK1.5 之前，为了线程安全，我们一般都是使用同步容器，同步容器主要的缺点是：对所有容器状态的访问都串行化，严重降低了并发性；某些复合操作，仍然需要加锁来保护；迭代期间，若其它线程并发修改该容器，会抛出 ConcurrentModificationException 异常，即快速失败机制；
- Fork/Join 并行计算框架，这块内容是在 JDK1.7 中引入的，可以方便利用多核平台的计算能力，简化并行程序的编写，开发人员仅需关注如何划分任务和组合中间结果；
- TimeUnit 枚举，TimeUnit 是 java.util.concurrent 包下面的一个枚举类，TimeUnit 提供了可读性更好的线程暂停操作，以及方便的时间单位转换方法；





#### 1 CAS介绍

​	CAS，Compare And Swap，即比较并交换。同步组件中大量使用CAS技术实现了Java多线程的并发操作。整个AQS同步组件、Atomic原子类操作等等都是以CAS实现的，甚至ConcurrentHashMap在1.8的版本中也调整为了CAS+Synchronized。可以说CAS是整个JUC的基石。

![img](assets\2018120816001.png)



#### 2 CAS原理剖析

​	再次测试之前Volatile的例子，把循环的次数调整为一亿（保证在一秒之内不能遍历完成，从而测试三种原子操作的性能），我们发现，AtomicInteger原子操作性能最高，他是用的就是CAS。



##### 2.1 synchronized同步分析

​	注意，本小节是解释synchronized性能低效的原因，只要能理解synchronized同步过程其实还需要做很多事，这些逻辑的执行都需要占用资源，从而导致性能较低，是为了对比CAS的高效。这部分分析过于深入JMM底层原理，不适合初级甚至中级程序员学习。

​	我们之前讲过，synchronized的同步操作主要是monitorenter和monitorexit这两个jvm指令实现的，我们先写一段简单的代码：

```java
public class Demo2Synchronized {
    public void test2() {
        synchronized (this) {
        }
    }
}
```

在cmd命令行执行javac编译和javap -c Java 字节码的指令

```
javac Demo2Synchronized.java
javap -c Demo2Synchronized.class
```



从结果可以看出，同步代码块是使用monitorenter和monitorexit这两个jvm指令实现的：

![50061740](assets\1558750061740.png)



​	monitorenter和monitorexit这两个jvm指令实现锁的使用，主要是基于 Mark Word和、monitor。



  **Mark Word**

  	Hotspot虚拟机的对象头主要包括两部分数据：Mark Word（标记字段）、Klass Pointer（类型指针）。其中Klass Point是是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例，Mark Word用于存储对象自身的运行时数据，它是synchronized实现轻量级锁和偏向锁的关键。

  	Mark Word用于存储对象自身的运行时数据，如**哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳**等等。Java对象头一般占有两个机器码（在32位虚拟机中，1个机器码等于4字节，也就是32bit），但是如果对象是数组类型，则需要三个机器码，因为JVM虚拟机可以通过Java对象的元数据信息确定Java对象的大小，但是无法从数组的元数据来确认数组的大小，所以用一块来记录数组长度。下图是Java对象头的存储结构（32位虚拟机）：

![img](assets/201812081002.png)



  	对象头信息是与对象自身定义的数据无关的额外存储成本，但是考虑到虚拟机的空间效率，Mark Word被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的数据，它会根据对象的状态复用自己的存储空间，也就是说，Mark Word会随着程序的运行发生变化，变化状态如下（32位虚拟机）：

![img](assets/201812081003.png)



* **monitor**

  	什么是Monitor？我们可以把它理解为一个同步工具，也可以描述为一种同步机制，它通常被描述为一个对象。与一切皆对象一样，所有的Java对象是天生的Monitor，每一个Java对象都有成为Monitor的潜质，因为在Java的设计中 ，每一个Java对象都带了一把看不见的锁，它叫做内部锁或者Monitor锁。

    	Monitor 是线程私有的数据结构，每一个线程都有一个可用monitor record列表，同时还有一个全局的可用列表。每一个被锁住的对象都会和一个monitor关联（对象头的MarkWord中的LockWord指向monitor的起始地址），同时monitor中有一个Owner字段存放拥有该锁的线程的唯一标识，表示该锁被这个线程占用。其结构如下：

![img](assets/201812081004.png)

- **Owner**：初始时为NULL表示当前没有任何线程拥有该monitor record，当线程成功拥有该锁后保存线程唯一标识，当锁被释放时又设置为NULL；
- **EntryQ**:关联一个系统互斥锁（semaphore），阻塞所有试图锁住monitor record失败的线程。
- **RcThis**:表示blocked或waiting在该monitor record上的所有线程的个数。
- **Nest**:用来实现重入锁的计数。
- **HashCode**:保存从对象头拷贝过来的HashCode值（可能还包含GC age）。
- **Candidate**:用来避免不必要的阻塞或等待线程唤醒，因为每一次只有一个线程能够成功拥有锁，如果每次前一个释放锁的线程唤醒所有正在阻塞或等待的线程，会引起不必要的上下文切换（从阻塞到就绪然后因为竞争锁失败又被阻塞）从而导致性能严重下降。Candidate只有两种可能的值0表示没有需要唤醒的线程1表示要唤醒一个继任线程来竞争锁。



##### 2.2 CAS原理

​	在上一部分，我们介绍了synchronized底层做了大量的工作，才实现同步，而同步保证了原子操作。但是不可避免的是性能较低。CAS是如何提高性能的呢？

​	CAS的思想很简单：三个参数，一个当前内存值V、旧的预期值A、即将更新的值B，当且仅当旧的预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做，并返回false。如果CAS操作失败，通过自旋的方式等待并再次尝试，直到成功。

​	CAS在  **先比较后修改**  这个CAS过程中，根本没有获取锁，释放锁的操作，是**硬件层面的原子操作**，跟JMM内存模型没有关系。大家可以理解为直接使用其他的语言，在JVM虚拟机之外直接操作计算机硬件，正因为如此，对比synchronized的同步，少了很多的逻辑步骤，使得性能大为提高。

​	JUC下的atomic类都是通过CAS来实现的，下面就是一个AtomicInteger原子操作类的例子，在其中使用了Unsafe unsafe = Unsafe.getUnsafe()。Unsafe 是CAS的核心类，它提供了硬件级别的原子操作。

```java
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;

static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
//操作的值也进行了volatile修饰，保证内存可见性
private volatile int value;
```



继续查看AtomicInteger的addAndGet()方法：

```java
public final int addAndGet(int delta) {
    return unsafe.getAndAddInt(this, valueOffset, delta) + delta;
}

public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;//预期值
    do {
        var5 = this.getIntVolatile(var1, var2);//获取value的当前值，val2，代表value对于对象的偏移量，他是实时改变的，这个var5记录的是采用这个方法时的value的内存位置---》通过位置可以找到value值
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));//比较和交换，原子类中的value属性现在的内存地址和当初的var5的值进行比较，如果相同就改变，while条件为！true，如果不相同就while（！false），从新do（自旋）获取当前value状态，也就是再次来过

    return var5;
}
```



​	其内部调用unsafe的getAndAddInt方法，查看看compareAndSwapInt方法，该方法为native方法，有四个参数，分别代表：对象、对象中的变量的内存地址偏移量、预期值、修改值。：

```java
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```



​	Unsafe 是一个比较危险的类，主要是用于执行低级别、不安全的方法集合。尽管这个类和所有的方法都是公开的（public），但是这个类的使用仍然受限，你无法在自己的java程序中直接使用该类，因为只有授信的代码才能获得该类的实例。可是为什么Unsafe的native方法就可以保证是原子操作呢？



##### 2.3 native关键词

​	前面提到了sun.misc.Unsafe这个类，里面的方法使用native关键词声明本地方法，为什么要用native？

​	Java无法直接访问底层操作系统，但有能力调用其他语言编写的函数or方法，是通过JNI(Java Native Interfface)实现。使用时，通过native关键字告诉JVM这个方法是在外部定义的。但JVM也不知道去哪找这个原生方法，此时需要通过javah命令生成.h文件。

示例步骤(c语言为例)：

1. javac生成.class文件，比如javac NativePeer.java
2. javah生成.h文件，比如javah NativePeer
3. 编写c语言文件，在其中include进上一步生成的.h文件，然后实现其中声明而未实现的函数
4. 生成dll共享库，然后Java程序load库，调用即可

		native可以和任何除abstract外的关键字连用，这也说明了这些方法是有实体的，并且能够和其他Java方法一样，拥有各种Java的特性。

		native方法有效地扩充了jvm，实际上我们所用的很多代码已经涉及到这种方法了，通过非常简洁的接口帮我们实现Java以外的工作。



native优势：

1. 很多层次上用Java去实现是很麻烦的，而且Java解释执行的效率也差了c语言啥的很多，纯Java实现可能会导致效率不达标，或者可读性奇差。
2. Java毕竟不是一个完整的系统，它经常需要一些底层的支持，通过JNI和native method我们就可以实现jre与底层的交互，得到强大的底层操作系统的支持，使用一些Java本身没有封装的操作系统的特性。





#### 3 多CPU的CAS处理

​	CAS可以保证一次的读-改-写操作是原子操作，在单处理器上该操作容易实现，但是在多处理器上实现就有点儿复杂了。CPU提供了两种方法来实现多处理器的原子操作：总线加锁或者缓存加锁。

- **总线加锁**：总线加锁就是就是使用处理器提供的一个LOCK#信号，当一个处理器在总线上输出此信号时，其他处理器的请求将被阻塞住,那么该处理器可以独占使用共享内存。但是这种处理方式显得有点儿霸道，不厚道，他把CPU和内存之间的通信锁住了，在锁定期间，其他处理器都不能其他内存地址的数据，其开销有点儿大。
- **缓存加锁**：其实针对于上面那种情况我们只需要保证在同一时刻对某个内存地址的操作是原子性的即可。缓存加锁就是缓存在内存区域的数据如果在加锁期间，当它执行锁操作写回内存时，处理器不在输出LOCK#信号，而是**修改内部的内存地址**，利用**缓存一致性协议**来保证原子性。缓存一致性机制可以保证**同一个内存区域的数据仅能被一个处理器修改**，也就是说当CPU1修改缓存行中的i时使用缓存锁定，那么CPU2就不能同时缓存了i的缓存行。





#### 4 CAS缺陷

​	CAS虽然高效地解决了原子操作，但是还是存在一些缺陷的，主要表现在三个方法：循环时间太长、只能保证一个共享变量原子操作、ABA问题。

- **循环时间太长**

  如果CAS一直不成功呢？这种情况绝对有可能发生，如果自旋CAS长时间地不成功，则会给CPU带来非常大的开销。在JUC中有些地方就限制了CAS自旋的次数，例如BlockingQueue的SynchronousQueue。

- **只能保证一个共享变量原子操作**

  看了CAS的实现就知道这只能针对一个共享变量，如果是多个共享变量就只能使用锁了。

- **ABA问题**

  CAS需要检查操作值有没有发生改变，如果没有发生改变则更新。但是存在这样一种情况：如果一个值原来是A，变成了B，然后又变成了A，那么在CAS检查的时候会发现没有改变，但是实质上它已经发生了改变，这就是所谓的ABA问题。对于ABA问题其解决方案是加上版本号，即在每个变量都加上一个版本号，每次改变时加1，即A —> B —> A，变成1A —> 2B —> 3A。

		CAS的ABA隐患问题，Java提供了AtomicStampedReference来解决。AtomicStampedReference通过包装[E,Integer]的元组来对对象标记版本戳stamp，从而避免ABA问题。对于上面的案例应该线程1会失败。



​	下面我们将通过一个例子可以可以看到AtomicStampedReference和AtomicInteger的区别。我们定义两个线程，线程1负责将100 —> 110 —> 100，线程2执行 100 —>120，看两者之间的区别。

```java
public class Demo4ABA {

    private static AtomicInteger ai = new AtomicInteger(100);
    private static AtomicStampedReference air = new AtomicStampedReference(100, 1);

    //ABA问题演示：
    //1. 线程1先对数据进行修改 A-B-A过程
    //2. 线程2也对数据进行修改 A-C的过程
    public static void main(String[] args) throws InterruptedException {

        // AtomicInteger可以看到不会有任何限制随便改
        // 线程2修改的时候也不可能知道要A-C 的时候，A是原来的A还是修改之后的A
        Thread at1 = new Thread(new Runnable() {
            public void run() {
                ai.compareAndSet(100, 110);
                ai.compareAndSet(110, 100);
            }
        });

        Thread at2 = new Thread(new Runnable() {
            public void run() {
                try {
                    //为了让线程1先执行完，等一会
                    TimeUnit.MILLISECONDS.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("AtomicInteger:" + ai.compareAndSet(100, 120));
                System.out.println("执行结果：" + ai.get());
            }
        });

        at1.start();
        at2.start();

        //顺序执行，AtomicInteger案例先执行
        at1.join();
        at2.join();

        //AtomicStampedReference可以看到每次修改都需要设置标识Stamp，相当于进行了1A-2B-3A的操作
        //线程2进行操作的时候，虽然数值都一样，但是可以根据标识很容易的知道A是以前的1A，还是现在的3A
        Thread tsf1 = new Thread(new Runnable() {
            public void run() {
                try {
                    TimeUnit.MILLISECONDS.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 预期引用：100，更新后的引用：110，预期标识getStamp() 更新后的标识getStamp() + 1
                air.compareAndSet(100, 110, air.getStamp(), air.getStamp() + 1);
                air.compareAndSet(110, 100, air.getStamp(), air.getStamp() + 1);
            }
        });

        Thread tsf2 = new Thread(new Runnable() {
            public void run() {

                //tsf2先获取stamp，导致预期时间戳不一致
                int stamp = air.getStamp();

                try {
                    TimeUnit.MILLISECONDS.sleep(100);     //线程tsf1执行完
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("AtomicStampedReference:" + air.compareAndSet(100, 120, stamp, stamp + 1));
                int[] stampArr = {stamp + 1};
                System.out.println("执行结果：" + air.get(stampArr));
            }
        });

        tsf1.start();
        tsf2.start();
    }
}
```

运行结果充分展示了AtomicInteger的ABA问题和AtomicStampedReference解决ABA问题。

### 9.4 小结

1. cas介绍

   - 就是比较并交换，通过三个参数 当前值v 旧的预期值o 要更新的值n，只有v==o的时候才能更新值o

2. cas缺陷

   1. cas每次只能更新一个变量，如果想对多个变量的更新保证线程安全，就不能使用cas了
   2. cas就是自旋，通过循环不断尝试直到成功为止，如果并发量特别大，比如100个线程同时更新一个变量。最长自旋的线程要等到99次，因为每次只会有一个线程操作成功，这样会给系统带来很大开销

3. cas aba问题

   1. 一个值在经历了A-B-A的过程，cas如果只比较值，值没有发生变化，就错误的操作成功了
   2. 通过版本号的方式解决AtomicStampedReference

4. native

   1. 通过底层c的同名方法调用操作系统：创建线程，cas

   

   





## 10 J.U.C之atomic包

### 10.1 目标

- 掌握atomic包下原子类的使用方式

### 10.2 路径

1. atomic包介绍
2. 基本类型
3. 引用类型
4. 数组类型
5. 对象的属性修改类型
6. jdk1.8新增类（LongAddr...）

### 10.3 讲解

#### 1 atomic包介绍

​	通过前面CAS的学习，我们了解到AtomicInteger的工作原理，它们的内部都维护者一个对应的基本类型的成员变量value，这个变量是被volatile关键字修饰的，保证多线程环境下看见的是同一个（可见性）。

​	AtomicInteger在进行一些原子操作的时候，依赖Unsafe类里面的CAS方法，原子操作就是通过自旋方式，不断地使用CAS函数进行尝试直到达到自己的目的。

​	除了AtomicInteger类以外还有很多其他的类也有类似的功能，在JUC中有一个包java.util.concurrent.atomic存放原子操作的类，atomic里的类主要包括：

- 基本类型
  使用原子的方式更新基本类型

  > AtomicInteger：整形原子类
  > AtomicLong：长整型原子类
  > AtomicBoolean ：布尔型原子类



- 引用类型

  > AtomicReference：引用类型原子类
  > AtomicStampedReference：原子更新引用类型里的字段原子类
  > AtomicMarkableReference ：原子更新带有标记位的引用类型



- 数组类型
  使用原子的方式更新数组里的某个元素

  > AtomicIntegerArray：整形数组原子类
  > AtomicLongArray：长整形数组原子类
  > AtomicReferenceArray ：引用类型数组原子类


- 对象的属性修改类型

  > AtomicIntegerFieldUpdater:原子更新整形字段的更新器
  > AtomicLongFieldUpdater：原子更新长整形字段的更新器
  > AtomicReferenceFieldUpdater ：原子更新引用类形字段的更新器



- JDK1.8新增类

  > DoubleAdder：双浮点型原子类
  > LongAdder：长整型原子类
  > DoubleAccumulator：类似DoubleAdder，但要更加灵活(要传入一个函数式接口)
  > LongAccumulator：类似LongAdder，但要更加灵活(要传入一个函数式接口)



​	虽然涉及到的类很多，但是原理和AtomicInteger都是一样，使用CAS进行的原子操作，其方法和使用都是大同小异的。



#### 2 基本类型

使用原子的方式更新基本类型

> AtomicInteger：整形原子类
> AtomicLong：长整型原子类
> AtomicBoolean ：布尔型原子类



**AtomicInteger主要API如下：**

```java
get()				//直接返回值
getAndAdd(int)		//增加指定的数据，返回变化前的数据
getAndDecrement()	//减少1，返回减少前的数据
getAndIncrement()	//增加1，返回增加前的数据
getAndSet(int)		//设置指定的数据，返回设置前的数据

addAndGet(int)		//增加指定的数据后返回增加后的数据
decrementAndGet()	//减少1，返回减少后的值
incrementAndGet()	//增加1，返回增加后的值
lazySet(int)		//当修改的值不需要立即被看到的时候，使用该api，优化volitale的内存屏障

compareAndSet(int, int)//尝试对比后赋值，若对比成功再赋值并返回true否则返回false
```

**AtomicLong主要API和AtomicInteger，只是类型不是int，而是long**



**AtomicBoolean主要API如下：**

```java
compareAndSet(boolean, boolean) //尝试对比后赋值，若对比成功再赋值并返回true否则返回false
getAndSet(boolean)// 尝试设置新的boolean值，直到成功为止，返回设置前的数据
```



#### 3 引用类型

> AtomicReference：引用类型原子类
> AtomicStampedRefrence：原子更新引用类型里的字段原子类
> AtomicMarkableReference ：原子更新带有标记位的引用类型

​	AtomicReference引用类型和基本类型的作用基本一样，例子如下：

```java
public class Demo5AtomicReference {

    public static void main(String[] args) throws InterruptedException {
        User u1 = new User("张三", 22);
        User u2 = new User("李四", 33);

        AtomicReference ar = new AtomicReference(u1);
        ar.compareAndSet(u1, u2);

        System.out.println(ar.get());

    }

    static class User {
        private String name;
        public volatile int age;

        public User(String name, int age) {
            super();
            this.name = name;
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }

        @Override
        public String toString() {
            return "User{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
}
```



​	AtomicStampedReference其实它仅仅是在AtomicReference类的再一次包装，里面增加了一层引用和计数器，其实是否为计数器完全由自己控制，大多数我们是让他自增的，你也可以按照自己的方式来标示版本号。案例参考前面的ABA例子



​	AtomicMarkableReference和AtomicStampedReference功能差不多，区别的是：它描述更加简单的是与否的关系。通常ABA问题只有两种状态，而AtomicStampedReference是多种状态。

```java
public class Demo6AtomicMrkableReference {

    public static void main(String[] args) throws InterruptedException {
        User u1 = new User("张三", 22);
        User u2 = new User("李四", 33);

        //和AtomicStampedReference效果一样，用于解决ABA的
        //区别是表示不是用的版本号，而只有true和false两种状态。相当于未修改和已修改
        AtomicMarkableReference<User> amr = new AtomicMarkableReference(u1, true);
        amr.compareAndSet(u1, u2, false, true);

        System.out.println(amr.getReference());

    }

    static class User {
        private String name;
        public volatile int age;

        public User(String name, int age) {
            super();
            this.name = name;
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }

        @Override
        public String toString() {
            return "User{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
}
```





#### 4 数组类型

使用原子的方式更新数组里的某个元素

> AtomicIntegerArray：整形数组原子类
> AtomicLongArray：长整形数组原子类
> AtomicReferenceArray ：引用类型数组原子类

**AtomicIntegerArray主要API如下：**

```java
addAndGet(int, int)//执行加法，第一个参数为数组的下标，第二个参数为增加的数量，返回增加后的结果
compareAndSet(int, int, int)// 对比修改，参1数组下标，参2原始值，参3修改目标值，成功返回true否则false
decrementAndGet(int)// 参数为数组下标，将数组对应数字减少1，返回减少后的数据
incrementAndGet(int)// 参数为数组下标，将数组对应数字增加1，返回增加后的数据

getAndAdd(int, int)// 和addAndGet类似，区别是返回值是变化前的数据
getAndDecrement(int)// 和decrementAndGet类似，区别是返回变化前的数据
getAndIncrement(int)// 和incrementAndGet类似，区别是返回变化前的数据
getAndSet(int, int)// 将对应下标的数字设置为指定值，第二个参数为设置的值，返回是变化前的数据
```

**AtomicIntegerArray主要API和AtomicLongArray，只是类型不是int，而是long**

AtomicIntegerArray案例：

```java
public class Demo7AtomicIntegerArray {

    public static void main(String[] args) throws InterruptedException {
        int[] arr = {1, 2, 3, 4, 5};
        AtomicIntegerArray aia = new AtomicIntegerArray(arr);

        aia.compareAndSet(1, 2, 200);

        System.out.println(aia.toString());
    }
}
```



**AtomicReferenceArray 主要API：**

```java
//参数1：数组下标；
//参数2：修改原始值对比；
//参数3：修改目标值 
//修改成功返回true，否则返回false
compareAndSet(int, Object, Object) 

//参数1：数组下标
//参数2：修改的目标
//修改成功为止，返回修改前的数据
getAndSet(int, Object) 

```

AtomicReferenceArray 案例：

```java
public class Demo8AtomicReferenceArray {

    public static void main(String[] args) throws InterruptedException {
        User u1 = new User("张三", 22);
        User u2 = new User("李四", 33);
        User[] arr = {u1, u2};

        AtomicReferenceArray<User> ara = new AtomicReferenceArray<User>(arr);
        System.out.println(ara.toString());

        User u3 = new User("王五", 44);
        ara.compareAndSet(0, u1, u3);
        System.out.println(ara.toString());
    }

    static class User {
        private String name;
        public volatile int age;

        public User(String name, int age) {
            super();
            this.name = name;
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }

        @Override
        public String toString() {
            return "User{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
}
```



#### 5 对象的属性修改类型

如果需要原子更新某个类里的某个字段时，需要用到对象的属性修改类型原子类。

> AtomicIntegerFieldUpdater:原子更新整形字段的更新器
> AtomicLongFieldUpdater：原子更新长整形字段的更新器
> AtomicReferenceFieldUpdater ：原子更新引用类形字段的更新器

但是他们的使用通常有以下几个限制：

- 限制1：操作的目标不能是static类型，前面说到的unsafe提取的是非static类型的属性偏移量，如果是static类型在获取时如果没有使用对应的方法是会报错的，而这个Updater并没有使用对应的方法。
- 限制2：操作的目标不能是final类型的，因为final根本没法修改。
- 限制3：必须是volatile类型的数据，也就是数据本身是读一致的。
- 限制4：属性必须对当前的Updater所在的区域是可见的，也就是private如果不是当前类肯定是不可见的，protected如果不存在父子关系也是不可见的，default如果不是在同一个package下也是不可见的。

实现方式：通过反射找到属性，对属性进行操作。

例子：

```java
public class AtomicIntegerFieldUpdaterTest {
    public static void main(String[] args) {
        AtomicIntegerFieldUpdater<User> a = AtomicIntegerFieldUpdater.newUpdater(User.class, "age");

        User user = new User("Java", 22);
        System.out.println(a.get(user));
        System.out.println(a.getAndAdd(user,10));
        System.out.println(a.get(user));
    }
}

class User {
    private String name;
    public volatile int age;

    public User(String name, int age) {
        super();
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

}
```





#### 6 JDK1.8新增类

> LongAdder：长整型原子类
> DoubleAdder：双浮点型原子类
> LongAccumulator：类似LongAdder，但要更加灵活(要传入一个函数式接口)
> DoubleAccumulator：类似DoubleAdder，但要更加灵活(要传入一个函数式接口)

​	LongAdder是jdk1.8提供的累加器，基于Striped64实现，所提供的API基本上可以替换原先的AtomicLong。

​	LongAdder类似于AtomicLong是原子性递增或者递减类，AtomicLong已经通过CAS提供了非阻塞的原子性操作，相比使用阻塞算法的同步器来说性能已经很好了，但是JDK开发组并不满足，因为在非常高的并发请求下AtomicLong的性能不能让他们接受，虽然AtomicLong使用CAS但是CAS失败后还是通过无限循环的自旋锁不断尝试。

```java
public final long incrementAndGet() {
    for (;;) {
        long current = get();
        long next = current + 1;
        if (compareAndSet(current, next))
            return next;
    }
}
```



​	在高并发下N多线程同时去操作一个变量会造成大量线程CAS失败然后处于自旋状态，这大大浪费了cpu资源，降低了并发性。那么既然AtomicLong性能由于过多线程同时去竞争一个变量的更新而降低的，那么如果把一个变量分解为多个变量，让同样多的线程去竞争多个资源那么性能问题不就解决了？是的，JDK8提供的LongAdder就是这个思路。下面通过图形来标示两者不同。



AtomicLong和LongAdder对比：

![img](assets\20170903201230654)



​	一段LongAdder和Atomic的对比测试代码：

```java
public class Demo9Compare {

    public static void main(String[] args) {
        AtomicLong atomicLong = new AtomicLong(0L);
        LongAdder longAdder = new LongAdder();

        long start = System.currentTimeMillis();
        for (int i = 0; i < 50; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    for (int j = 0; j < 1000000; j++) {
                        //atomicLong.incrementAndGet();
                        longAdder.increment();
                    }
                }
            }).start();
        }

        while (Thread.activeCount() > 2) { //测试存活的线程，一般mian线程和gc线程是默认存在的，当其他线程还存在时，便会一直while循环，当其他线程完成时，只存活两个，那么就会执行while循环后面的语句
        }

        System.out.println(atomicLong.get());
        System.out.println(longAdder.longValue());

        System.out.println("耗时：" + (System.currentTimeMillis() - start));

    }
}
```



不同计算机因为CPU、内存等硬件不一样，所以测试的数值也不一样，但是得到的结论都是一样的

测试结果：

![img](assets\20170903201405380)

​	从上结果图可以看出，在并发比较低的时候，LongAdder和AtomicLong的效果非常接近。但是当并发较高时，两者的差距会越来越大。上图中在线程数为1000，每个线程循环数为100000时，LongAdder的效率是AtomicLong的6倍左右。

### 10.4 小结

1. atomic包介绍

   里面放了一些原子操作类，我们可以使用这些api对共享变量进行并发操作，底层都是使用的cas，性能非常高

2. 基本类型

   ATInteger，ATLong,ATBoolean

   get在前就是获取修改前的值，get在后就是获取修改后的值

3. 引用类型

   ATRef...：对Object的原子操作

   ATS..Ref:给Object加版本号，每次修改都比较版本号和Object，可以有多个版本（int）

   ATmak...Ref:给Object加版本号，每次修改都比较版本号和Object，只能有两个版本号（boolean）

4. 数组类型

   ATIntegerArr...:Integer类型的数组原子类

   ATLongArr..:Long类型的数组原子类

   前两个和基本类型使用方式差不多，多了个参数（下标）

   ATRef...Arr:Object类型的数组

   和引用类型使用方式差不多，多了个参数（下标）

5. 对象的属性修改类型（修改volidate修饰的属性）

   AtomicIntegerFieldUpdater:原子更新整形字段的更新器
   AtomicLongFieldUpdater：原子更新长整形字段的更新器
   AtomicReferenceFieldUpdater ：原子更新引用类形字段的更新器

   通过这些类对要修改的javabean进行包装，能够原子性的修改javabean的属性

   使用修改器要注意的点：

   - 要修改的类和属性不能static，负责拿不到内存地址
   - 属性必须是外部可直接访问的，public修饰
   - 属性不能是final类型，因为final不能被修改
   - 属性必须被volidate修饰，因为cas只能保证原子性，要通过volidate保证有序性和可见性，才能保证线程安全。

   

6. jdk1.8新增类（LongAddr...）

   对比ATLong性能更高，底层是使用多个变量对同一个变量进行分割，解决了同一个变量操作并发大时，等待较长时间消耗系统cpu资源较高的问题
   
   

## 11 J.U.C之AQS

### 11.1 目标

- 知道什么是AQS
- 理解AQS工作原理

### 11.2 路径

1. AQS简介
2. CLH同步队列
3. 入列
4. 出列

### 11.3 讲解

#### 1 AQS简介

​	AQS(AbstractQueuedSynchronizer），即队列同步器。它是构建锁或者其他同步组件的基础框架（如ReentrantLock、ReentrantReadWriteLock、Semaphore等），JUC并发包的作者（Doug Lea）期望它能够成为实现大部分同步需求的基础。它是JUC并发包中的核心基础组件。

​	在这里我们只是对AQS进行了解，它只是一个抽象类，但是JUC中的很多组件都是基于这个抽象类，也可以说这个AQS是多数JUC组件的基础。



##### 1.1 AQS的作用

​	Java的内置锁一直都是备受争议的，在JDK 1.6之前，synchronized这个重量级锁其性能一直都是较为低下，虽然在1.6后，进行大量的锁优化策略，但是与Lock相比synchronized还是存在一些缺陷的：它缺少了获取锁与释放锁的可操作性，可中断、超时获取锁，而且独占式在高并发场景下性能大打折扣。

​	AQS解决了实现同步器时涉及到的大量细节问题，例如获取同步状态、FIFO同步队列。基于AQS来构建同步器可以带来很多好处。它不仅能够极大地减少实现工作，而且也不必处理在多个位置上发生的竞争问题。

​	

##### 1.2 state状态

​	AQS维护了一个volatile int类型的变量state表示当前同步状态。当state>0时表示已经获取了锁，当state = 0时表示释放了锁。

它提供了三个方法来对同步状态state进行操作：

> getState()：返回同步状态的当前值
> setState()：设置当前同步状态
> compareAndSetState()：使用CAS设置当前状态，该方法能够保证状态设置的原子性

这三种操作均是原子操作，第三种是基于cas，其中compareAndSetState的实现依赖于Unsafe的compareAndSwapInt()方法



##### 1.3 资源共享方式

​	AQS定义两种资源共享方式：

* Exclusive（独占，只有一个线程能执行，如ReentrantLock）
* Share（共享，多个线程可同时执行，如Semaphore/CountDownLatch）



​	不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源state的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。自定义同步器实现时主要实现以下几种方法：

> - isHeldExclusively()：当前同步器是否在独占式模式下被线程占用，一般该方法表示是否被当前线程所独占。只有用到condition才需要去实现它。
> - tryAcquire(int)：独占方式。尝试获取同步状态，成功则返回true，失败则返回false。其他线程需要等待该线程释放同步状态才能获取同步状态。
> - tryRelease(int)：独占方式。尝试释放同步状态，成功则返回true，失败则返回false。
> - tryAcquireShared(int)：共享方式。尝试获取同步状态。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
> - tryReleaseShared(int)：共享方式。尝试释放同步状态，如果释放后允许唤醒后续等待结点，返回true，否则返回false。

#### 

#### 2 CLH同步队列

​	AQS内部维护着一个FIFO队列，该队列就是CLH同步队列，遵循FIFO原则（ First Input First Output先进先出）。CLH同步队列是一个FIFO双向队列，AQS依赖它来完成同步状态的管理。

![img](assets\721070-20170504110246211-10684485.png)

​	当前线程如果获取同步状态失败时，AQS则会将当前线程已经等待状态等信息构造成一个节点（Node）并将其加入到CLH同步队列，同时会阻塞当前线程，当同步状态释放时，会把首节点唤醒（公平锁），使其再次尝试获取同步状态。





#### 3 入列

​	CLH队列入列非常简单，就是tail指向新节点、新节点的prev指向当前最后的节点，当前最后一个节点的next指向当前节点。

![img](assets\2018120810002.png)



代码我们可以看看addWaiter(Node node)方法：

```java
private Node addWaiter(Node mode) {
    //新建Node
    Node node = new Node(Thread.currentThread(), mode);
    //快速尝试添加尾节点
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        //CAS设置尾节点
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    //多次尝试，1.当前的tail为空，说明队列没有初始化需要初始化；2.有一个线程比当前线程先入队了
    enq(node);
    return node;
}

 private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            //尾节点为空，需要初始化
            if (t == null) { // Must initialize
                //cas设置头部节点为空节点，如果失败说明有线程先初始化了队列，说明队列已经初始化好了
                if (compareAndSetHead(new Node()))
                    //如果成功就首尾设置一致
                    tail = head;
            //如果队列初始化完毕    
            } else {
                //将当前节点的前一个设置成尾部
                node.prev = t;
                //cas设置尾节点为当前节点
                if (compareAndSetTail(t, node)) {
                    //之前尾节点的后一个设置成当前节点
                    t.next = node;
                    return t;
                }
            }
        }
    }
```

​	在上面代码中，两个方法都是通过一个CAS方法compareAndSetTail(Node expect, Node update)来设置尾节点，该方法可以确保节点是线程安全添加的。在enq(Node node)方法中，AQS通过“死循环”的方式来保证节点可以正确添加，只有成功添加后，当前线程才会从该方法返回，否则会一直执行下去。



#### 4 出列

​	CLH同步队列遵循FIFO，首节点的线程释放同步状态后，将会唤醒它的后继节点（next），而后继节点将会在获取同步状态成功时将自己设置为首节点。head执行该节点并断开原首节点的next和当前节点的prev即可，注意在这个过程是不需要使用CAS来保证的，因为只有一个线程能够成功获取到同步状态。过程图如下：

![img](assets\2018120810003.png)

### 11.4 小结

- 知道什么是AQS

  AQS是juc的locks包下组件的基础框架，框架中制定了一个基本流程，里面的state具体的获取方式需要它的实现类去实现，这些实现类都是不同的锁里面都有一个内部类对应，state表示资源，如果获取不到资源就将当前线程加入队列，通过自旋的方式重复尝试获取资源。

- 理解AQS工作原理

  FIFO双向队列，先进先出，node对应的是被阻塞的线程，head，tail，这两个变量的操作包括入列操作都是cas原子操作。但是出列并不是cas，因为独享方式下只有一个线程获取到了state状态。
  
  

## 12 J.U.C之锁

### 12.1 目标

- 理解几种锁的概念
- 掌握J.U.C.locks 包下几种基本锁的使用

### 12.2 路径

1. 锁的基本概念
2. 重入锁（ReentrantLock）
3. 读写锁（ReentrantReadWriteLock）

### 12.3 讲解

#### 1 锁的基本概念

虽然在前面锁优化的部分已经提到过一些锁的概念，但不完全，这里是对锁的概念补充。

##### 1.1 互斥锁

​	在编程中，引入了对象互斥锁的概念，来保证共享数据操作的完整性。每个对象都对应于一个可称为" 互斥锁" 的标记，这个标记用来保证在任一时刻，只能有**一个线程**访问该对象。



##### 1.2 阻塞锁

​	阻塞锁，可以说是让线程进入阻塞状态进行等待，当获得相应的信号（唤醒，时间） 时，才可以进入线程的准备就绪状态，准备就绪状态的所有线程，通过竞争，进入运行状态。



##### 1.3 自旋锁

自旋锁是采用让当前线程不停地的在循环体内执行实现的，当循环的条件被其他线程改变时，才能进入临界区。

​	由于自旋锁只是将当前线程不停地执行循环体，不进行线程状态的改变，所以响应速度更快。但当线程数不停增加时，性能下降明显，因为每个线程都需要执行，占用CPU时间。如果线程竞争不激烈，并且保持锁的时间段很短。适合使用自旋锁。



##### 1.4 读写锁 

​	读写锁实际是一种特殊的自旋锁，它把对共享资源的访问者划分成读者和写者，读者只对共享资源进行读访问，写者则需要对共享资源进行写操作。

​	读写锁相对于自旋锁而言，能提高并发性，因为在多处理器系统中，它允许同时有多个读者来访问共享资源，最大可能的读者数为实际的逻辑CPU数。写者是排他性的，一个读写锁同时只能有一个写者或多个读者（与CPU数相关），但不能同时既有读者又有写者。



##### 1.5 公平锁

公平锁（Fair）：加锁前检查是否有排队等待的线程，优先排队等待的线程，先来先得 

非公平锁（Nonfair）：加锁时不考虑排队等待问题，直接尝试获取锁，获取不到自动到队尾等待

非公平锁性能比公平锁高，因为公平锁需要在多核的情况下维护一个队列。



#### 2 ReentrantLock

​	ReentrantLock，可重入锁，是一种递归无阻塞的同步机制。它可以等同于synchronized的使用，但是ReentrantLock提供了比synchronized更强大、灵活的锁机制，可以减少死锁发生的概率。

​	ReentrantLock还提供了公平锁和非公平锁的选择，构造方法接受一个可选的公平参数（默认非公平锁），当设置为true时，表示公平锁，否则为非公平锁。公平锁的效率往往没有非公平锁的效率高，在许多线程访问的情况下，公平锁表现出较低的吞吐量。



查看ReentrantLock源码中的构造方法：

```java
public ReentrantLock() {
    //非公平锁
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
    //公平锁
    sync = fair ? new FairSync() : new NonfairSync();
}
```

​	Sync为ReentrantLock里面的一个内部类，它继承AQS（AbstractQueuedSynchronizer），它有两个子类：公平锁FairSync和非公平锁NonfairSync。



##### 2.1 获取锁

一般都是这么使用ReentrantLock获取锁的：（默认非公平锁）

```java
//非公平锁
ReentrantLock lock = new ReentrantLock();
lock.lock();
```



lock方法：

```java
public void lock() {
    sync.lock();
}

final boolean nonfairTryAcquire(int acquires) {
    		//获取当前线程
            final Thread current = Thread.currentThread();
    		//获取资源状态
            int c = getState();
    		//如果状态为0，没有线程获取到锁
            if (c == 0) {
                //当前线程cas操作给state赋值
                if (compareAndSetState(0, acquires)) {
                    //赋值成功，说明当前线程拿到锁了，设置aqs里面的独占线程是本线程
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
    		//说明已经有一个线程获取到锁了，如果当前线程==aqs里面的独占线程，说明重入了
            else if (current == getExclusiveOwnerThread()) {
                //这部分代码只有一个线程能访问就是重入的线程，所以不需要考虑线程安全的问题
                //state+1
                int nextc = c + acquires;
                //小于0抛异常
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                //设置state为+1后的值
                setState(nextc);
                return true;
            }
            return false;
        }

//判断队列中是否有等待时间更久的线程，返回false代表没有，返回true代表有
//如果要当前线程继续执行，返回false，以下几种情况
//1.h==t,队列是空的
//2.h!=t,队列不是空的,h.next不为空且h.next.thread就是当前线程
//其他情况
//如果h.next==null说明有线程正在入队
public final boolean hasQueuedPredecessors() {
        // The correctness of this depends on head being initialized
        // before tail and on head.next being accurate if the current
        // thread is first in queue.
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
```



加锁最终可以看到会调用方法：

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

其实底层就是使用AQS同步队列。

总结：获取锁的过程就是尝试对state从0到1的修改，重入就是累加state的值，公平模式获取锁比非公平多了队列的判断。

##### 2.2 释放锁

获取同步锁后，使用完毕则需要释放锁，ReentrantLock提供了unlock释放锁：

```java
public void unlock() {
    sync.release(1);
}
```

unlock内部使用Sync的release()释放锁，release()是在AQS中定义的：

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```



释放同步状态的tryRelease()是同步组件自己实现：

```java
protected final boolean tryRelease(int releases) {
    		//不需要考虑线程安全的问题，因为独占式释放锁的过程只有一个线程能执行就是获取到锁的线程
    		//拿到state-1
            int c = getState() - releases;
   			//只有持有锁的线程能够释放资源
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;//表示锁被释放了，别的线程可以获取到锁
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;//返回true就唤醒后继节点继续执行
        }
```

​	只有当同步状态彻底释放后该方法才会返回true。当同步队列的状态state == 0 时，则将锁持有线程设置为null，free= true，表示释放成功。



##### 2.3 公平锁与非公平锁原理

​	公平锁与非公平锁的区别在于获取锁的时候是否按照FIFO的顺序来。释放锁不存在公平性和非公平性，比较非公平锁和公平锁获取同步状态的过程，会发现两者唯一的区别就在于公平锁在获取同步状态时多了一个限制条件：hasQueuedPredecessors()，定义如下：

```java
public final boolean hasQueuedPredecessors() {
    Node t = tail;  //尾节点
    Node h = head;  //头节点
    Node s;

    //头节点 != 尾节点
    //同步队列第一个节点不为null
    //当前线程是同步队列第一个节点
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

​	该方法主要做一件事情：主要是判断队列中是否有线程在当前线程之前。如果是则返回true，否则返回false。

这个判断返回false的两种情况：

​	1.h==t说明队列为空

​	2.队列不为空时，只有头节点的下一个节点为当前节点才可以。



##### 2.4 ReentrantLock与synchronized的区别

​	前面提到ReentrantLock提供了比synchronized更加灵活和强大的锁机制，那么它的灵活和强大之处在哪里呢？他们之间又有什么相异之处呢？

1. 与synchronized相比，ReentrantLock提供了更多，更加全面的功能，具备更强的扩展性。例如：时间锁等候，可中断锁等候，锁投票。
2. ReentrantLock还提供了条件Condition，对线程的等待、唤醒操作更加详细和灵活，所以在多个条件变量和高度竞争锁的地方，ReentrantLock更加适合（以后会阐述Condition）。
3. ReentrantLock提供了可轮询的锁请求。它会尝试着去获取锁，如果成功则继续，否则可以等到下次运行时处理，而synchronized则一旦进入锁请求要么成功要么阻塞，所以相比synchronized而言，ReentrantLock会不容易产生死锁些。
4. ReentrantLock支持更加灵活的同步代码块，但是使用synchronized时，只能在同一个synchronized块结构中获取和释放。注：ReentrantLock的锁释放一定要在finally中处理，否则可能会产生严重的后果。
5. ReentrantLock支持中断处理，且性能较synchronized会好些。





#### 3 读写锁ReentrantReadWriteLock

​	可重入锁ReentrantLock是互斥锁，互斥锁在同一时刻仅有一个线程可以进行访问，但是在大多数场景下，大部分时间都是提供读服务，而写服务占有的时间较少。然而读服务不存在数据竞争问题，如果一个线程在读时禁止其他线程读势必会导致性能降低。所以就提供了读写锁。

​	读写锁维护着一对锁，一个读锁和一个写锁。通过分离读锁和写锁，使得并发性比一般的互斥锁有了较大的提升：在同一时间可以允许多个读线程同时访问，但是在写线程访问时，所有读线程和写线程都会被阻塞。

​	**读操作不互斥，写操作互斥，读和写互斥**

读写锁的主要特性：

1. 公平性：支持公平性和非公平性。
2. 重入性：支持重入。读写锁最多支持65535个递归写入锁和65535个递归读取锁。
3. 锁降级：写锁能够降级成为读锁，遵循获取写锁、获取读锁在释放写锁的次序。读锁不能升级为写锁。



​	读写锁ReentrantReadWriteLock实现接口ReadWriteLock，该接口维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 writer，读取锁可以由多个 reader 线程同时保持。写入锁是独占的。

```java
public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}
```

ReadWriteLock定义了两个方法。readLock()返回用于读操作的锁，writeLock()返回用于写操作的锁。ReentrantReadWriteLock定义如下：

```java
/** 内部类  读锁 */
private final ReentrantReadWriteLock.ReadLock readerLock;
/** 内部类  写锁 */
private final ReentrantReadWriteLock.WriteLock writerLock;

final Sync sync;

/** 使用默认（非公平）的排序属性创建一个新的 ReentrantReadWriteLock */
public ReentrantReadWriteLock() {
    this(false);
}

/** 使用给定的公平策略创建一个新的 ReentrantReadWriteLock */
public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}

/** 返回用于写入操作的锁 */
public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
/** 返回用于读取操作的锁 */
public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }

abstract static class Sync extends AbstractQueuedSynchronizer {
    //省略其余源代码
}
public static class WriteLock implements Lock, java.io.Serializable{
    //省略其余源代码
}

public static class ReadLock implements Lock, java.io.Serializable {
    //省略其余源代码
}
```



​	ReentrantReadWriteLock与ReentrantLock一样，其锁主体依然是Sync，它的读锁、写锁都是依靠Sync来实现的。所以ReentrantReadWriteLock实际上只有一个锁，只是在获取读取锁和写入锁的方式上不一样而已，它的读写锁其实就是两个类：ReadLock、writeLock，这两个类都是lock实现。



​	在ReentrantLock中使用一个int类型的state来表示同步状态，该值表示锁被一个线程重复获取的次数。但是读写锁ReentrantReadWriteLock内部维护着一对锁，需要用一个变量维护多种状态。所以读写锁采用“按位切割使用”的方式来维护这个变量，将其**切分为两部分**，高16为表示读，低16为表示写。分割之后，读写锁是如何迅速确定读锁和写锁的状态呢？通过位运算。假如当前同步状态为S，那么写状态等于 S & 0x0000FFFF（将高16位全部抹去），读状态等于S >>> 16(无符号补0右移16位)。代码如下：

```java
static final int SHARED_SHIFT   = 16;
static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
```

##### 3.1 读写锁例子

**ex1:读不互斥，写互斥，读写互斥**

```java
public class Demo10ReentrantReadWriteLock {
    private static volatile int count = 0;

    public static void main(String[] args) {
        ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
        WriteDemo writeDemo = new WriteDemo(lock);
        ReadDemo readDemo = new ReadDemo(lock);
        //运行多个写线程，不会重复，证明写互斥
        //运行多个读线程，可能重复，证明读不互斥
        //同时运行，读锁和写锁后面不会出现重复的数字，证明读写互斥
        for (int i = 0; i < 3; i++) {
            new Thread(writeDemo).start();
        }
        for (int i = 0; i < 5; i++) {
            new Thread(readDemo).start();
        }
    }

    static class WriteDemo implements Runnable {
        ReentrantReadWriteLock lock;
        public WriteDemo(ReentrantReadWriteLock lock) {
            this.lock = lock;
        }

        @Override
        public void run() {
            for (int i = 0; i < 5; i++) {
                try {
                    TimeUnit.MILLISECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                lock.writeLock().lock();
                count++;
                System.out.println("-写锁："+count);//不会重复，证明写互斥
                lock.writeLock().unlock();
            }
        }
    }

    static class ReadDemo implements Runnable {
        ReentrantReadWriteLock lock;
        public ReadDemo(ReentrantReadWriteLock lock) {
            this.lock = lock;
        }

        @Override
        public void run() {
            for (int i = 0; i < 5; i++) {
                try {
                    TimeUnit.MILLISECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                lock.readLock().lock();
                count++;
                System.out.println("读锁："+count);//可能重复，证明读不互斥
                lock.readLock().unlock();
            }
        }
    }
}
```

**ex2:写锁的降级**（获取到了写锁的线程能够再次获取到通一把锁的读锁）

​	**demo1**:**写锁可以降级为读锁，读锁无法升级到写锁**

````java
public class LockDegradationDemo1 {

    public static void main(String[] args) {
        new demo1().fun1();
    }

    static class demo1{
        ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();
        public void fun1(){
            reentrantReadWriteLock.writeLock().lock();
            System.out.println("fun1");
            fun2();
            reentrantReadWriteLock.writeLock().unlock();
        }

        public void fun2(){
            reentrantReadWriteLock.readLock().lock();
            System.out.println("fun2");
            reentrantReadWriteLock.readLock().unlock();
        }
    }
}
````

​	**demo2**:

````java
public class LockDegradationDemo2 {

    public static void main(String[] args) {
        demo1 demo1 = new demo1();
        new Thread(new Runnable() {
            @Override
            public void run() {
                demo1.fun1();
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                demo1.fun3();
            }
        }).start();
    }

    static class demo1{

        ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();

        public void fun1(){
            reentrantReadWriteLock.writeLock().lock();
            System.out.println("fun1");
            fun2();
            reentrantReadWriteLock.writeLock().unlock();
            System.out.println("释放写锁");
        }

        public void fun2(){
            reentrantReadWriteLock.readLock().lock();
            try {
                Thread.currentThread().sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("fun2");
            reentrantReadWriteLock.readLock().unlock();
            System.out.println(reentrantReadWriteLock.getWriteHoldCount());
        }

        public void fun3(){
            reentrantReadWriteLock.readLock().lock();
            System.out.println("fun3");
            reentrantReadWriteLock.readLock().unlock();
        }
    }
}
````



​	

##### 3.2 写锁的获取

写锁就是一个支持可重入的互斥锁。

写锁的获取最终会调用tryAcquire(int arg)，该方法在内部类Sync中实现：

```java
 protected final boolean tryAcquire(int acquires) {
            /*
             * Walkthrough:
             * 1. If read count nonzero or write count nonzero
             *    and owner is a different thread, fail.
             * 2. If count would saturate, fail. (This can only
             *    happen if count is already nonzero.)
             * 3. Otherwise, this thread is eligible for lock if
             *    it is either a reentrant acquire or
             *    queue policy allows it. If so, update state
             *    and set owner.
             */
     		//拿到当前线程
            Thread current = Thread.currentThread();
     		//获取state
            int c = getState();
     		//获取低16位的值表示写的状态
            int w = exclusiveCount(c);
     		//当前有线程获取到了写锁或者读锁
            if (c != 0) {
                // (Note: if c != 0 and w == 0 then shared count != 0)
                //w==0有线程获取到了读锁，就不能获取写锁
                //w!=0当前有线程获取到了写锁,且current==getExclusiveOwnerThread()说明重入写锁
                if (w == 0 || current != getExclusiveOwnerThread())
                    return false;
                //如果写的次数超过65535（2^16-1）说明超过了state所能表示的最大重入次数，就抛出异常
                if (w + exclusiveCount(acquires) > MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                // Reentrant acquire
                //增加state，累计重入次数
                setState(c + acquires);
                return true;
            }
     		//c==0说明当前没有线程获取到锁
            if (writerShouldBlock() ||
                !compareAndSetState(c, c + acquires))
                return false;
     		//把当前先设置为独占
            setExclusiveOwnerThread(current);
            return true;
        }
```

​	该方法和ReentrantLock的tryAcquire(int arg)大致一样，在判断重入时增加了一项条件：读锁是否存在。因为要确保写锁的操作对读锁是可见的，如果在存在读锁的情况下允许获取写锁，那么那些已经获取读锁的其他线程可能就无法感知当前写线程的操作。因此只有等读锁完全释放后，写锁才能够被当前线程所获取，一旦写锁开始获取了，所有其他读、写线程均会被阻塞。



##### 3.3 写锁的释放

获取了写锁用完了则需要释放，WriteLock提供了unlock()方法释放写锁：

```java
    public void unlock() {
        sync.release(1);
    }

  protected final boolean tryRelease(int releases) {
      		//写锁的释放只有一个线程执行，所以是线程安全的。
      		//判断当前现场是否是写锁的独占线程，如果不是就抛异常
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
      		//修改状态
            int nextc = getState() - releases;
      		//free标识当前释放已经释放了所有重入次数，计数为0表示锁释放完毕了
            boolean free = exclusiveCount(nextc) == 0;
            if (free)
                setExclusiveOwnerThread(null);
            setState(nextc);
            return free;
        }
```

​	写锁的释放最终还是会调用AQS的模板方法release(int arg)方法，该方法首先调用tryRelease(int arg)方法尝试释放锁，tryRelease(int arg)方法为读写锁内部类Sync中定义了，如下：

```java
    protected final boolean tryRelease(int releases) {
        //释放的线程不为锁的持有者
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        int nextc = getState() - releases;
        //若写锁的新线程数为0，则将锁的持有者设置为null
        boolean free = exclusiveCount(nextc) == 0;
        if (free)
            setExclusiveOwnerThread(null);
        setState(nextc);
        return free;
    }
```

​	写锁释放锁的整个过程和互斥锁ReentrantLock相似，每次释放均是减少写状态，当写状态为0时表示 写锁已经完全释放了，从而等待的其他线程可以继续访问读写锁，获取同步状态，同时此次写线程的修改对后续的线程可见。



##### 3.4 读锁的获取

读锁为一个可重入的共享锁，它能够被多个线程同时持有，在没有其他写线程访问时，读锁总是获取成功。

读锁的获取可以通过ReadLock的lock()方法：

```java
public void lock() {
    sync.acquireShared(1);
}
```

​	Sync的acquireShared(int arg)定义在AQS中：

```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
protected final int tryAcquireShared(int unused) {
            /*
             * Walkthrough:
             * 1. If write lock held by another thread, fail.
             * 2. Otherwise, this thread is eligible for
             *    lock wrt state, so ask if it should block
             *    because of queue policy. If not, try
             *    to grant by CASing state and updating count.
             *    Note that step does not check for reentrant
             *    acquires, which is postponed to full version
             *    to avoid having to check hold count in
             *    the more typical non-reentrant case.
             * 3. If step 2 fails either because thread
             *    apparently not eligible or CAS fails or count
             *    saturated, chain to version with full retry loop.
             */
    		//获取当前线程
            Thread current = Thread.currentThread();
    		//获取state
            int c = getState();
    		//如果当前有线程获取到了写锁，只有持有写锁的线程是当前线程，才能够继续获取读锁
            if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
                return -1;
    		//读锁的状态，读锁的获取的总数
            int r = sharedCount(c);
    		//判断公平性
            if (!readerShouldBlock() &&
                r < MAX_COUNT &&
                compareAndSetState(c, c + SHARED_UNIT)) {
                if (r == 0) {
                    firstReader = current;//当前的线程设置为firstReader
                    firstReaderHoldCount = 1;//重入次数
                } else if (firstReader == current) {
                    //累计重入次数
                    firstReaderHoldCount++;
                } else {
                    //其他线程把重入次数保存在ThreadLocal里面
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;
                }
                return 1;
            }
            return fullTryAcquireShared(current);
        }
```



##### 3.5 读锁的释放

与写锁相同，读锁也提供了unlock()释放读锁：

```java
public void unlock() {
    sync.releaseShared(1);
}
```

unlcok()方法内部使用Sync的releaseShared(int arg)方法，该方法也定义AQS中：

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}

protected final boolean tryReleaseShared(int unused) {
    		//获取当前线程
            Thread current = Thread.currentThread();
    		//第一个获取读锁的线程
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
                if (firstReaderHoldCount == 1)
                    firstReader = null;
                else
                    firstReaderHoldCount--;
            } else {
                //readHolds保存除了第一个获取读锁的线程以外的其他线程重入次数
                //HoldCounter缓存的读的线程重入次数
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();//在threadLocal中获取当前线程的重入计数
                int count = rh.count;
                if (count <= 1) {
                    readHolds.remove();
                    if (count <= 0)
                        throw unmatchedUnlockException();
                }
                --rh.count;
            }
    		//由于可能发生多线程同时释放读锁，所以通过自旋的方式释放读锁
            for (;;) {
                int c = getState();
                int nextc = c - SHARED_UNIT;
                if (compareAndSetState(c, nextc))
                    // Releasing the read lock has no effect on readers,
                    // but it may allow waiting writers to proceed if
                    // both read and write locks are now free.
                    return nextc == 0;
            }
        }
```



##### 3.6 锁降级

读写锁有一个特性就是锁降级，锁降级就意味着写锁是可以降级为读锁的。锁降级需要遵循以下顺序：

`获取写锁=>获取读锁=>释放写锁`





### 12.4 小结

1. ReentrantLock

   1. 默认是非公平的，如果想要公平构造方法传入true，就是公平的
   2. 内部实现原理：通过尝试修改state从0-1获取锁，修改成功表示当前线程获取到锁了，修改失败表示没有获取到，同一个线程获取到了锁可以重入，重入的过程就是累计state的过程，释放锁就是state减到0的过程，减到0了就说明锁释放完成了，唤醒后继节点继续执行。

2. ReetrantReadWriteLock

   1. 内部维护了一对锁，写锁和读锁

      - 写锁之间是互斥的，读锁之间不互斥，读锁和写锁之间是互斥的
      - 锁降级：一个线程获取到了同一把锁的写锁之后还能够获取这把锁的读锁。（锁并不是真的降级了）
      - 锁不能够升级的

   2. 内部原理：

      - 使用state按位分割，高16位表示读，低16位表示写。
      - 获取写锁的过程就是修改低16位的过程，也表示写的重入次数
      - 获取读锁的过程就是修改高16位的过程，也表示所有获取到读锁的线程的重入总数
      - 每个线程自己的读锁的重入次数保存在ThreadLocal<>里面
- 不管是写锁还是读锁，重入次数不能超过2^16-1
      
      





## 13 J.U.C之Condition

如果把juc下的lock锁和synchronized比较，Condition就相当于Object，都是用来控制线程状态

### 13.1 目标

- 理解什么是Condition
- 掌握 Condition的使用方式

### 13.2 路径

1.  Condition介绍
2.  Condition的实现
3. 通知

### 13.3 讲解

#### 1 Condition介绍

​	在没有Lock之前，我们使用synchronized来控制同步，配合Object的wait()、notify()系列方法可以实现等待/通知模式。在JDK5后，Java提供了Lock接口，相对于Synchronized而言，Lock提供了条件Condition，对线程的等待、唤醒操作更加详细和灵活。

下图是Condition与Object的监视器方法的对比：

![2018120815001](assets\2018120815001.png)

Condition提供了一系列的方法来阻塞和唤醒线程：

1. **await()** ：造成当前线程在接到信号或被中断之前一直处于等待状态。

2. **await(long time, TimeUnit unit) **：造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。

3. **awaitNanos(long nanosTimeout) **：造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。返回值表示剩余时间，如果在nanosTimesout之前唤醒，那么返回值 = nanosTimeout – 消耗时间，如果返回值 <= 0 ,则可以认定它已经超时了。

4. **awaitUninterruptibly() **：造成当前线程在接到信号之前一直处于等待状态。【注意：该方法对中断不敏感】。

5. **awaitUntil(Date deadline) **：造成当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态。如果没有到指定时间就被通知，则返回true，否则表示到了指定时间，返回返回false。

6. **signal()**：唤醒一个等待线程。该线程从等待方法返回前必须获得与Condition相关的锁。

7. **signalAll()**：唤醒所有等待线程。能够从等待方法返回的线程必须获得与Condition相关的锁。

   Condition是一种广义上的条件队列（等待队列）。他为线程提供了一种更为灵活的等待/通知模式，线程在调用await方法后执行挂起操作，直到线程等待的某个条件为真时才会被唤醒。Condition必须要配合锁一起使用，因为对共享状态变量的访问发生在多线程环境下。一个Condition的实例必须与一个Lock绑定，因此Condition一般都是作为Lock的内部实现。




案例：

```java
public class Demo11Condition {

    private Lock reentrantLock = new ReentrantLock();
    private Condition condition1 = reentrantLock.newCondition();
    private Condition condition2 = reentrantLock.newCondition();

    public void m1() {
        reentrantLock.lock();
        try {
            System.out.println("线程 " + Thread.currentThread().getName() + " 已经进入执行等待。。。");
            condition1.await();
            System.out.println("线程 " + Thread.currentThread().getName() + " 已被唤醒，继续执行。。。");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            reentrantLock.unlock();
        }
    }

    public void m2() {
        reentrantLock.lock();
        try {
            System.out.println("线程 " + Thread.currentThread().getName() + " 已经进入执行等待。。。");
            condition1.await();
            System.out.println("线程 " + Thread.currentThread().getName() + " 已被唤醒，继续执行。。。");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            reentrantLock.unlock();
        }
    }

    public void m3() {
        reentrantLock.lock();
        try {
            System.out.println("线程 " + Thread.currentThread().getName() + " 已经进入执行等待。。。");
            condition2.await();
            System.out.println("线程 " + Thread.currentThread().getName() + " 已被唤醒，继续执行。。。");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            reentrantLock.unlock();
        }
    }

    public void m4() {
        reentrantLock.lock();
        try {
            System.out.println("线程 " + Thread.currentThread().getName() + " 已经进入发出condition1唤醒信号。。。");
            condition1.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            reentrantLock.unlock();
        }
    }

    public void m5() {
        reentrantLock.lock();
        try {
            System.out.println("线程 " + Thread.currentThread().getName() + " 已经进入发出condition2唤醒信号。。。");
            condition2.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            reentrantLock.unlock();
        }
    }

    public static void main(String[] args) throws Exception {
        final Demo11Condition useCondition = new Demo11Condition();
        Thread t1 = new Thread(new Runnable() {
            public void run() {
                useCondition.m1();
            }
        }, "t1");
        Thread t2 = new Thread(new Runnable() {
            public void run() {
                useCondition.m2();
            }
        }, "t2");
        Thread t3 = new Thread(new Runnable() {
            public void run() {
                useCondition.m3();
            }
        }, "t3");
        Thread t4 = new Thread(new Runnable() {
            public void run() {
                useCondition.m4();
            }
        }, "t4");
        Thread t5 = new Thread(new Runnable() {
            public void run() {
                useCondition.m5();
            }
        }, "t5");

        t1.start();
        t2.start();
        t3.start();

        Thread.sleep(2000);
        t4.start();


        Thread.sleep(2000);
        t5.start();
    }
}
```




#### 2 Condition的实现

​	获取一个Condition必须通过Lock的newCondition()方法。该方法定义在接口Lock下面，返回的结果是绑定到此 Lock 实例的新 Condition 实例。Condition为一个接口，其下仅有一个实现类ConditionObject，由于Condition的操作需要获取相关的锁，而AQS则是同步锁的实现基础，所以ConditionObject则定义为AQS的内部类。定义如下：

```java
public class ConditionObject implements Condition, java.io.Serializable {
}
```

##### 2.1 等待队列

​	每个Condition对象都包含着一个FIFO队列，该队列是Condition对象通知/等待功能的关键。在队列中每一个节点都包含着一个线程引用，该线程就是在该Condition对象上等待的线程。源码如下：

```java
public class ConditionObject implements Condition, java.io.Serializable {
    private static final long serialVersionUID = 1173984872572414699L;

    //头节点
    private transient Node firstWaiter;
    //尾节点
    private transient Node lastWaiter;

    public ConditionObject() {
    }

    /** 省略方法 **/
}
```

​	从上面代码可以看出Condition拥有首节点（firstWaiter），尾节点（lastWaiter）。当前线程调用await()方法，将会以当前线程构造成一个节点（Node），并将节点加入到该队列的尾部。结构如下：

![img](assets\2018120815002.png)

Node里面包含了当前线程的引用。Node定义与AQS的CLH同步队列的节点使用的都是同一个类（AbstractQueuedSynchronized.Node静态内部类）。

Condition的队列结构比CLH同步队列的结构简单些，新增过程较为简单只需要将原尾节点的nextWaiter指向新增节点，然后更新lastWaiter即可。



##### 2.2 等待状态

​	调用Condition的await()方法会使当前线程进入等待状态，同时会加入到Condition等待队列同时释放锁。当从await()方法返回时，当前线程一定是获取了Condition相关连的锁。

```java
public final void await() throws InterruptedException {
    // 当前线程中断
    if (Thread.interrupted())
        throw new InterruptedException();
    //当前线程加入等待队列
    Node node = addConditionWaiter();
    //释放锁
    long savedState = fullyRelease(node);
    int interruptMode = 0;
    /**
         * 检测此节点的线程是否在同步队上，如果不在，则说明该线程还不具备竞争锁的资格，则继续等待
         * 直到检测到此节点在同步队列上
         */
    while (!isOnSyncQueue(node)) {
        //线程挂起
        LockSupport.park(this);
        //如果已经中断了，则退出
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    //竞争同步状态
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    //清理下条件队列中的不是在等待条件的节点
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```

​	此段代码的逻辑是：首先将当前线程新建一个节点同时加入到条件队列中，然后释放当前线程持有的同步状态。然后则是不断检测该节点代表的线程释放出现在CLH同步队列中（收到signal信号之后就会在AQS队列中检测到），如果不存在则一直挂起，否则参与竞争同步状态。



#### 3 通知

​	调用Condition的signal()方法，将会唤醒在等待队列中等待最长时间的节点（条件队列里的首节点），在唤醒节点前，会将节点移到CLH同步队列中。

```java
public final void signal() {
    //检测当前线程是否为拥有锁的独
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    //头节点，唤醒条件队列中的第一个节点
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);    //唤醒
}
```

该方法首先会判断当前线程是否已经获得了锁，这是前置条件。然后唤醒等待队列中的头节点。



doSignal(Node first)：唤醒头节点

```java
private void doSignal(Node first) {
    do {
        //修改头结点，完成旧头结点的移出工作
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
        first.nextWaiter = null;
    } while (!transferForSignal(first) &&
             (first = firstWaiter) != null);
}
```

doSignal(Node first)主要是做两件事：

1.修改头节点，

2.调用transferForSignal(Node first) 方法将节点移动到CLH同步队列中。



### 13.4 小结

condition比synchronize能够更加灵活易用。可以通过一系列api完成对线程的调度。



# 总结 #

1. volatile

   1. 实现可见性：缓存一致性协议
   2. 实现有序性：同步屏障
      1. 让对volatile读写之前和之后的命令不能发生指令重排
      2. 同步屏障之前会让写入的命令立刻刷到主内存里面
      3. volatile读写操作不会发生指令重排
   3. 适用场景:对变量的操作本身是原子性的，才能够保证线程安全
      1. 除了long和double以外的基本类型的读写
      2. 引用类型的读写

2. cas

   1. compareAndSwap->比较并交换的意思
   2. 原理：当前值 v ，旧的预期值o，要更新的值n，当且精当o==v的时候才去把值更新n，通过native接口方法调用底层c的同名方法，通过操作系统在硬件层面上实现比较和赋值两个操作的原子性
   3. 通过自旋的方式，不断的尝试直到成功为止
   4. aba问题：一个变量的值从A-B-A的过程，如果只对比值本身，值没有发生变化，大多数情况下ABA问题不会对操作结果产生影响，加版本号的方式解决，AtomicStampedReference->1A->2B->3C

3. atomic包

   1. 有各种各样的原子类->都是通过cas实现的
   2. 属性修改类型原子类使用的前提
      1. 类和属性都不能是static修饰的，拿不到内存地址
      2. 属性不能final类型，final类型不能被修改
      3. 必须是volatile修饰的，因为cas只能保证原子性，不能保证可见性和有序性
      4. 属性必须是可以被访问到的，一般是public
   3. LongAddr效率比AtomicLong高：
      - 通过将一个变量分割成多个，每一个都是cas操作，通过这种方式提高提升并发访问时的效率。

4. aqs（AbstractQueuedSynchronizer：就是队列同步器）

   1. 队列同步器它是抽象类，里面定义了一系列的模板方法，一些方法交给子类实现，是juc锁，同步组件的基础。
   2. 内部维护了一个队列，一个先进先出双向队列，通过state状态表示资源状态。
   3. 当一个线程尝试获取state资源，失败了就做为一个节点保存队列的尾部
   4. 线程执行完了唤醒下一个节点继续执行

5. juc锁

   1. ReentrantLock
      1. 默认是非公平的，如果想要公平构造方法传入true，就是公平的
      2. 内部实现原理：通过尝试修改state从0-1获取锁，修改成功表示当前线程获取到锁了，修改失败表示没有获取到，同一个线程获取到了锁可以重入，重入的过程就是累计state的过程，释放锁就是state减到0的过程，减到0了就说明锁释放完成了，唤醒后继节点继续执行。
   2. ReetrantReadWriteLock
      1. 内部维护了一对锁，写锁和读锁
         - 写锁之间是互斥的，读锁之间不互斥，读锁和写锁之间是互斥的
         - 锁降级：一个线程获取到了同一把锁的写锁之后还能够获取这把锁的读锁。（锁并不是真的降级了）
         - 锁不能够升级的
      2. 内部原理：
         - 使用state按位分割，高16位表示读，低16位表示写。
         - 获取写锁的过程就是修改低16位的过程，也表示写的重入次数
         - 获取读锁的过程就是修改高16位的过程，也表示所有获取到读锁的线程的重入总数
         - 不管是写锁还是读锁，重入次数不能超过2^16-1

6. condition

   1.等价于synchronized中的Object，通过condition可以灵活的控制线程的调度。

   

 

