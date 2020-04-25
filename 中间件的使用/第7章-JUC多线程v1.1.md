# JUC（java.util.current）多线程 (一)

## 1. 多线程基础

### 1.1 目标

​	理解多线程技术的一些基本概念。

### 1.2 路径

1. 多线程的概念及作用
2. 线程和进程的介绍
3. 多线程的创建
4. 用户线程和守护线程

### 1.3 讲解

#### 	1.3.1 多线程的概念及作用

​	    一个采用了多线程技术的应用程序可以更好地利用系统资源。其主要优势在于充分利用了CPU的空闲时间片，可以用尽可能少的时间来对用户的要求做出响应，使得进程的整体运行效率得到较大提高，同时增强了应用程序的灵活性。

​	   更为重要的是，由于同一进程的所有线程是共享同一内存，所以不需要特殊的数据传送机制，不需要建立共享存储区或共享文件，从而使得不同任务之间的协调操作与运行、数据的交互、资源的分配等问题更加易于解决。

​	   **简单说，多线程就是以空间（cpu的占用）换时间(响应时间)，期望以更多的资源消耗（cpu）来减少响应时间，提高执行速度的技术。**

#### 	1.3.2 线程和进程 

- **进程：**

​	是指一个内存中运行的应用程序，每个进程都有一个**独立的内存空间**，一个应用程序可以同时运行多个进程；进程也是程序的一次执行过程，是系统运行程序的基本单位；系统运行一个程序即是一个进程从创建、运行到消亡的过程。



- **线程：**

​	进程内部的一个独立执行单元；一个进程可以同时并发的运行多个线程，可以理解为一个进程便相当于一个单 CPU 操作系统，而线程便是这个系统中运行的多个任务。



- **进程与线程的区别：**

​	进程：有独立的内存空间，进程中的数据存放空间（堆空间和栈空间）是独立的，至少有一个线程。

​	线程：堆空间是共享的，栈空间是独立的，线程消耗的资源比进程小的多。



- **注意：**

1. 因为一个进程中的多个线程是并发运行的，那么从微观角度看也是有先后顺序的，哪个线程执行完全取决于
    CPU 的调度，程序员是不能完全控制的（可以设置线程优先级）。而这也就造成的多线程的随机性。
2. Java 程序的进程里面至少包含两个线程，主线程也就是 main()方法线程，另外一个是垃圾回收机制线程。每
    当使用 java 命令执行一个类时，实际上都会启动一个 JVM，每一个 JVM 实际上就是在操作系统中启动了一个
    线程，java 本身具备了垃圾的收集机制，所以在 Java 运行时至少会启动两个线程。
3. 由于创建一个线程的开销比创建一个进程的开销小的多，那么我们在开发多任务运行的时候，通常考虑创建
    多线程，而不是创建多进程。



#### 1.3.3 多线程的创建

创建Maven工程，编写测试类



##### 1.继承Thread类

​	第一种继承Thread类 重写run方法

```java
public class Demo1CreateThread extends Thread {

    public static void main(String[] args) throws InterruptedException {

        System.out.println("-----多线程创建开始-----");
        // 1.创建一个线程
        CreateThread createThread1 = new CreateThread();
        CreateThread createThread2 = new CreateThread();
        // 2.开始执行线程 注意 开启线程不是调用run方法，而是start方法
        System.out.println("-----多线程创建启动-----");
        createThread1.start();
        createThread2.start();
        System.out.println("-----多线程创建结束-----");
    }

    static class CreateThread extends Thread {
        public void run() {
            String name = Thread.currentThread().getName();
            for (int i = 0; i < 5; i++) {
                System.out.println(name + "打印内容是:" + i);
            }
        }
    }
}
```



##### 2.实现Runnable接口

​	实现Runnable接口,重写run方法

​	实际上所有的多线程代码都是通过运行Thread的start()方法来运行的。因此，不管是继承Thread类还是实现Runnable接口来实现多线程，最终还是通过Thread的对象的API来控制线程的。

```java
public class Demo2CreateRunnable {

    public static void main(String[] args) {
        System.out.println("-----多线程创建开始-----");
        // 1.创建线程
        CreateRunnable createRunnable = new CreateRunnable();
        Thread thread1 = new Thread(createRunnable);
        Thread thread2 = new Thread(createRunnable);
        // 2.开始执行线程 注意 开启线程不是调用run方法，而是start方法
        System.out.println("-----多线程创建启动-----");
        thread1.start();
        thread2.start();
        System.out.println("-----多线程创建结束-----");
    }

    static class CreateRunnable implements Runnable {

        public void run() {
            String name = Thread.currentThread().getName();
            for (int i = 0; i < 5; i++) {
                System.out.println(name + "的内容:" + i);
            }
        }
    }
}
```



>  实现Runnable接口比继承Thread类所具有的优势：
>  1. 适合多个相同的程序代码的线程去共享同一个资源。
>  2. 可以避免java中的单继承的局限性。
>  3. 增加程序的健壮性，实现解耦操作，代码可以被多个线程共享，代码和数据独立。



##### 3.匿名内部类方式

​	使用线程的内匿名内部类方式，可以方便的实现每个线程执行不同的线程任务操作

```java
public class Demo3Runnable {
    public static boolean exit = true;

    public static void main(String[] args) throws InterruptedException {
        new Thread(new Runnable() {
            public void run() {
                String name = Thread.currentThread().getName();
                for (int i = 0; i < 5; i++) {
                    System.out.println(name + "执行内容：" + i);
                }
            }
        }).start();

        new Thread(new Runnable() {
            public void run() {
                String name = Thread.currentThread().getName();
                for (int i = 0; i < 5; i++) {
                    System.out.println(name + "执行内容：" + i);
                }
            }
        }).start();

        Thread.sleep(1000l);
    }
}}
```



#### 1.3.4 用户线程和守护线程

Java中有两种线程，一种是用户线程，另一种是守护线程。

用户线程是指用户自定义创建的线程，主线程停止，用户线程不会停止。

守护线程当进程不存在或主线程停止，守护线程也会被停止。

```java
public class Demo4Daemon {
    public static void main(String[] args) {
        Thread thread = new Thread(new Runnable() {
            public void run() {
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(10);
                    } catch (Exception e) {
                    }
                    System.out.println("子线程..." + i);
                }
            }
        });
		
        // 设置线程为守护线程
        //thread.setDaemon(true);
        thread.start();

        for (int i = 0; i < 5; i++) {
            try {
                Thread.sleep(10);
                System.out.println("主线程" + i);
            } catch (Exception e) {

            }
        }

        System.out.println("主线程执行完毕!");
    }
}
```

### 1.4 小结

多线程：通过空间（cpu的时间片的利用）换取响应时间

线程和进程的概念

- 进程：程序的一次执行，进程之间内存是独立的，无法共享内存空间，至少有一个线程。
- 线程：进程内部的最小执行单元，线程之间是共享堆内存，栈内存是独立的。

创建线程的方式

- 继承Thread类，重写run方法
- 实现Runnable接口，实现里面的run方法
- 匿名内部类实现Runnable接口，New Thread(()->执行内容)

线程的执行：一定要用Start方法执行线程，如果run方法执行是直接执行类的方法，不会以线程的方式执行

用户线程和守护线程：

- 用户线程：一般是用户创建的，不会随着主线程的终止而终止

- 守护线程：一般是系统创建的，会随着主线的终止而终止，垃圾回收线程就是守护线程，可以使用Thread::setDaemon方法将用户线程转化为守护线程

  ​	

## 2 线程安全

### 1.1 目标

​	理解多线程技术的一些基本概念。

### 1.2 路径

1. 通过案例演示理解线程安全的重要性
2. 保证线程安全的方案
3. 死锁

### 1.3 讲解

线程安全：当多个线程执行一段程序的时候，如果发生了和预期结果不一致的情况，就是线程不安全的。如果一致就是线程安全

#### 1.3.1.卖票案例

​	如果有多个线程在同时运行，而这些线程可能会同时运行这段代码。程序每次运行结果和单线程运行的结果是一样的，而且其他的变量的值也和预期的是一样的，就是线程安全的，反之则是线程不安全的。

```java
public class Demo5Ticket {

    public static void main(String[] args) {
        //创建线程任务对象
        Ticket ticket = new Ticket();
        //创建三个窗口对象
        Thread t1 = new Thread(ticket, "窗口1");
        Thread t2 = new Thread(ticket, "窗口2");
        Thread t3 = new Thread(ticket, "窗口3");

        //卖票
        t1.start();
        t2.start();
        t3.start();
    }

    static class Ticket implements Runnable {

        //Object lock = new Object();
        ReentrantLock lock = new ReentrantLock();
        private int ticket = 10;

        public void run() {
            String name = Thread.currentThread().getName();
            while (true) {
                sell(name);
                if (ticket <= 0) {
                    break;
                }
            }
        }

        private void sell(String name) {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if (ticket > 0) {
                System.out.println(name + "卖票：" + ticket);
                ticket--;
            }
        }
    }
}
```



​	线程安全问题都是由全局变量及静态变量引起的。若每个线程中对全局变量、静态变量只有读操作，而无写
操作，一般来说，这个全局变量是线程安全的；若有多个线程同时执行写操作，一般都需要考虑线程同步，
否则的话就可能影响线程安全。



#### 1.3.2.线程同步

当我们使用多个线程访问同一资源的时候，且多个线程中对资源有写的操作，就容易出现线程安全问题。
要解决上述多线程并发访问一个资源的安全问题，Java中提供了同步机制(synchronized)来解决。

synchronized都是对象锁

**同步代码块**

```java
Object lock = new Object(); //创建锁
synchronized(lock){
     //可能会产生线程安全问题的代码
}
```



**同步方法**

```java
//同步方法 this对象
public synchronized void method(){
   //可能会产生线程安全问题的代码 
}
```

同步方法使用的是this锁

证明方式: 一个线程使用同步代码块(this明锁),另一个线程使用同步方法。如果两个线程抢票不能实现同步，那么会出现数据错误。

```java
//使用this锁的同步代码块
synchronized(this){
     //需要同步操作的代码
}


public class Demo5Ticket2 {

    public static void main(String[] args) {
        //创建线程任务对象
        Ticket ticket = new Ticket();
        //创建三个窗口对象
        Thread t1 = new Thread(ticket, "窗口1");
        Thread t2 = new Thread(ticket, "窗口2");
        Thread t3 = new Thread(ticket, "窗口3");

        //卖票
        t1.start();
        t2.start();
        t3.start();
    }

    static class Ticket implements Runnable {

        Object lock1 = new Object();
        ReentrantLock lock = new ReentrantLock();
        private static int ticket = 10;

        public void run() {
            String name = Thread.currentThread().getName();
            int i = 0;
            while (true) {
                i++;
                if(i%2 == 0){
                    sell(name);
                }else{
                    sell2(name);
                }
                if (ticket <= 0) {
                    break;
                }
            }
        }

        private void sell(String name) {
            synchronized (Ticket.class){
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                if (ticket > 0) {
                    System.out.println(name + "卖票：" + ticket);
                    ticket--;
                }
            }
        }

        private static synchronized void sell2(String name) {
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if (ticket > 0) {
                System.out.println(name + "卖票：" + ticket);
                ticket--;
            }
        }
    }
}
```

1.synchronized(lock)，对lock加锁

2.同步方法：this锁

3.静态同步方法：当前类的class对象

**Lock锁**

```java
Lock lock = new ReentrantLock();
lock.lock();
	//需要同步操作的代码
lock.unlock();
```



#### 1.3.3.死锁

多线程死锁：同步中嵌套同步,导致锁无法释放。

死锁解决办法：不要在同步中嵌套同步

```java
public class Demo6DeadLock {

    public static void main(String[] args) {
        //创建线程任务对象
        Ticket ticket = new Ticket();
        //创建三个窗口对象
        Thread t1 = new Thread(ticket, "窗口1");
        Thread t2 = new Thread(ticket, "窗口2");
        Thread t3 = new Thread(ticket, "窗口3");

        //卖票
        t1.start();
        t2.start();
        t3.start();
    }

    static class Ticket implements Runnable {

        Object lock = new Object();
        private int ticket = 100;

        public void run() {
            String name = Thread.currentThread().getName();
            while (true) {
                if ("窗口1".equals(name)) {
                    synchronized (lock) {
                        sell(name);
                    }
                } else {
                    sell(name);
                }
                if (ticket <= 0) {
                    break;
                }
            }
        }
        //this锁  如果非窗口1，取得了this锁，然后还需要lock，等待
        //窗口1，取得了lock锁，需要this锁，等待
        private synchronized void sell(String name) {
            synchronized (lock) {
                if (ticket > 0) {
                    System.out.println(name + "卖票：" + ticket);
                    ticket--;
                }
            }
        }
    }
```

死锁案例二

````java
public class DeadLockDemo {
    static Object obj1 = new Object();
    static Object obj2 = new Object();

    public static void fun01(){
        System.out.println(Thread.currentThread().getId()+"尝试获取obj1的锁");
        synchronized (obj1){
            System.out.println(Thread.currentThread().getId()+"获取到了ob1的锁");
            try {
                Thread.sleep(30);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            fun02();
        }
    }

    public static void fun02(){
        System.out.println(Thread.currentThread().getId()+"尝试获取obj2的锁");
        synchronized (obj2){
            System.out.println(Thread.currentThread().getId()+"获取到了ob2的锁");
            try {
                Thread.sleep(30);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            fun01();
        }
    }

    public static void main(String[] args) {
        Thread thread1 = new Thread(() -> fun01());
        Thread thread2 = new Thread(() -> fun02());
        thread1.start();
        thread2.start();
    }
}
````

### 1.4 小结

线程安全：当多线程执行同一段程序的时候，如果发生了和预期结果不一致的情况，就是线程不安全的，如果和预期结果一致就是线程安全的，可以加锁解决(把并行运行的线程变成串行化执行)。

同步锁的几种方式(锁对象):

- 同步代码块加锁：sync....(obj)
- 同步方法加锁：等价于sync....(this)
- 静态同步方法加锁:等价于sync....(this.getClass())

死锁：线程之间互相等待对方释放锁，就产生了死锁，尽量不要同步中嵌套同步。



## 3 线程状态

### 1.1 目标

​	理解多线程技术的一些基本概念。

### 1.2 路径

1. 线程状态介绍
2. wait()和notify()
3. wait()和sleep的区别
4. 线程停止

### 1.3 讲解

#### 	1.3.1.线程状态介绍

查看Thread源码，能够看到java的线程有六种状态：

```java
public enum State {
        NEW,
        RUNNABLE,
        BLOCKED,
        WAITING,
        TIMED_WAITING,
        TERMINATED;
    }
```

**NEW(新建) **
``线程刚被创建，但是并未启动。``
**RUNNABLE(可运行)**
``线程可以在java虚拟机中运行的状态，可能正在运行自己代码，也可能没有，这取决于操作系统处理器。``
**BLOCKED(锁阻塞)**
``当一个线程试图获取一个对象锁，而该对象锁被其他的线程持有，则该线程进入Blocked状态；当该线程持有锁时，该线程将变成Runnable状态。``
**WAITING(无限等待)**
``一个线程在等待另一个线程执行一个（唤醒）动作时，该线程进入Waiting状态。进入这个状态后是不能自动唤醒的，必须等待另一个线程调用notify或者notifyAll方法才能够唤醒。``
**TIMED_WAITING(计时等待)**
``同waiting状态，有几个方法有超时参数，调用他们将进入Timed Waiting状态。这一状态将一直保持到超时期满或者接收到唤醒通知。带有超时参数的常用方法有Thread.sleep 、Object.wait。``
**TERMINATED(被终止)**
``因为run方法正常退出而死亡，或者因为没有捕获的异常终止了run方法而死亡。``



##### 线程状态图

![img](assets\v2-326a2be9b86b1446d75b6f52f54c98fb_hd.jpg)

#### 1.3.2.wait()、notify()

wait()、notify()、notifyAll()是三个定义在Object类里的方法，可以用来控制线程的状态。

wait		方法会使持有该对象的线程把该对象的控制权交出去，然后处于等待状态。
notify	方法会通知**某个**正在等待这个对象的控制权的线程继续运行。
notifyAll	方法会通知**所有**正在等待这个对象的控制权的线程继续运行。

**注意：一定要在线程同步中使用,并且是同一个锁的资源**

wait和notify方法例子，打开关闭开关：

```java
public class DemoSwitch {

    static class Switch{
        //开关状态
        Boolean state = false;

        public Boolean getState() {
            return state;
        }

        public void setState(Boolean state) {
            String s = state?"打开":"关闭";
            System.out.println(s);
            if(this.state==state){
                System.out.println("开关不能连续"+s);
            }
            this.state = state;
        }
    }

    static abstract class ActionThread implements Runnable{
        //开关
        Switch aSwitch;

        public ActionThread(Switch aSwitch) {
            this.aSwitch = aSwitch;
        }

        @Override
        public void run() {
            //循环
            while (true){
                //同步
                synchronized (aSwitch){
                    action();
                }
            }
        }

        protected abstract void action();
    }

    public static void main(String[] args) {
        //1.创建开关
        Switch aSwitch = new Switch();
        //2.关闭线程
        Thread thread1 = new Thread(new ActionThread(aSwitch){
            @Override
            protected void action() {
                //RUNNABLE
                System.out.println("thread1获取到aSwitch对象的锁后:"+Thread.currentThread().getState().name());
                //如果开关是关闭的
                if(!this.aSwitch.getState()){
                    try {
                        //让当前线程释放锁，进入等待
                        this.aSwitch.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                //关闭开关
                this.aSwitch.setState(false);
                //唤醒等待中的一个线程
                this.aSwitch.notify();
            }
        });
        //new
        System.out.println("创建thread1未运行时的状态是:"+thread1.getState().name());
        //2.打开线程
        Thread thread2 = new Thread(new ActionThread(aSwitch){
            @Override
            protected void action() {
                //BLOCKED
                System.out.println("thread2获取到aSwitch对象的锁后:"+thread1.getState().name());
                //如果开关是打开的
                if(this.aSwitch.getState()){
                    try {
                        //让当前线程释放锁，进入等待
                        this.aSwitch.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                //打开开关
                this.aSwitch.setState(true);
                //wait
                System.out.println("thread2线程在调用notify之前:"+thread1.getState().name());
                //唤醒等待中的一个线程
                this.aSwitch.notify();

            }
        });

        thread1.start();
        thread2.start();
    }
}
```



#### 1.3.3.wait与sleep区别

* 对于sleep()方法，首先要知道该方法是属于Thread类中的。而wait()方法，则是属于Object类中的。

* sleep()方法导致了程序暂停执行指定的时间，让出cpu调度其他线程，但是他的监控状态依然保持者，当指定的时间到了又会自动恢复运行状态。

  wait()是把控制权交出去，然后进入等待此对象的等待锁定池处于等待状态，只有针对此对象调用notify()方法后本线程才进入对象锁定池准备获取对象锁进入运行状态。

* 在调用sleep()方法的过程中，线程不会释放对象锁。而当调用wait()方法的时候，线程会放弃对象锁。





#### 1.3.4.线程停止

结束线程有以下三种方法：
（1）设置退出标志，使线程正常退出。
（2）使用interrupt()方法中断线程。
（3）使用stop方法强行终止线程（不推荐使用Thread.stop, 这种终止线程运行的方法已经被废弃，使用它们是极端不安全的！）



#####1.5.1 使用退出标志 

       一般run()方法执行完，线程就会正常结束，然而，常常有些线程是伺服线程。它们需要长时间的运行，只有在外部某些条件满足的情况下，才能关闭这些线程。使用一个变量来控制循环，例如：最直接的方法就是设一个boolean类型的标志，并通过设置这个标志为true或false来控制while循环是否退出，代码示例：

```java
public class Demo8Exit {

    public static boolean exit = true;

    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(new Runnable() {
            public void run() {
                while (exit) {
                    try {
                        System.out.println("线程执行！");
                        Thread.sleep(100l);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
        t.start();

        Thread.sleep(1000l);
        exit = false;//线程再执行，执行中while（exit),main线程也在执行，会改变exit
        System.out.println("退出标识位设置成功");
    }
}
```



#####1.5.2 使用interrupt()方法

使用interrupt()方法来中断线程有两种情况：

**1)线程处于阻塞状态**

​	如使用了sleep,同步锁的wait,socket中的receiver,accept等方法时，会使线程处于阻塞状态。当调用线程的interrupt()方法时，会抛出InterruptException异常。阻塞中的那个方法抛出这个异常，通过代码捕获该异常，然后break跳出循环状态，从而让我们有机会结束这个线程的执行。



**2)线程未处于阻塞状态**

​	使用isInterrupted()判断线程的中断标志来退出循环。当使用interrupt()方法时，中断标志就会置true，和使用自定义的标志来控制循环是一样的道理。

```java
public class Demo9Interrupt {

    public static boolean exit = true;

    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(new Runnable() {
            public void run() {
                while (exit) {
                    try {
                        System.out.println("线程执行！");

                        //判断线程的中断标志来退出循环--监视是否使用t.interrupt如果线程正在运行，进入if，使用breake
                        if (Thread.currentThread().isInterrupted()) {
                            break;
                        }

                        Thread.sleep(100l);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                        //线程处于阻塞状态,当调用线程的interrupt()方法时，
                        //会抛出InterruptException异常,跳出循环
                        break;
                    }
                }
            }
        });
        t.start();

        Thread.sleep(1000l);
        //中断线程
        t.interrupt();
        System.out.println("线程中断了");
    }
}
```

### 1.4 小结

线程状态：

- NEW：线程被创建但是没有start运行
- RUNABLE:线程可以运行，是否运行取决于cpu是否调度该线程，如果没有调度就是ready，如调度到就是running
- WAITING：当锁对象调用wait方法，会让持有该锁对象的线程进入无限等待状态，这个状态只有被同一个锁对象的notify才能解除，解除后进入RUNABLE状态。
- TIMED_WAITING：sleep(time),wait(time)的时候进入计时等待，当时间到了，继续运行
- BLOCKED：在线程获取不到锁对象的时候，就会进入阻塞状态，当其他线程释放锁，本线程获取到锁才能够继续运行。
- TERMINATED：run方法执行完毕之后，进入终止状态。

wait和sleep的区别：

- wait是属于object对象的方法，sleep是属于Thread类的方法
- wait会让当前持有该对象锁的线程停止运行，进入waiting状态，并且会释放锁，只有同一个对象的notify/notifyAll方法才能够唤醒,sleep不会释放锁，让线程等待一段时间继续运行。

线程终止：

- 自定义退出标识：缺点是线程在sleep的时候是无法立刻退出，必须等线程执行到了判断标识的地方才能够停止

- 使用线程自己的退出标识：

  - 如果线程正常运行，可以通过Thread.currentThread().isInterrupted()判断退出

  - 如果当前线程正在sleep，通过‘t.interrupt();’会抛出异常，可以通过捕获异常立刻线程的运行。

    

## 4 线程优先级

### 1.1 目标

​	理解多线程技术的一些基本概念。

### 1.2 路径

1. 设置并理解线程的优先级priority
2. 理解并掌握join()方法的使用
3. 理解yield()方法的概念

### 1.3 讲解

##### 	1.优先级priority

​	现今操作系统基本采用分时的形式调度运行的线程，线程分配得到时间片的多少决定了线程使用处理器资源的多少，也对应了线程优先级这个概念。

​	在JAVA线程中，通过一个int priority来控制优先级，范围为1-10，其中10最高，默认值为5。

```java
public class Demo10Priorityt {

    public static void main(String[] args) {
        PrioritytThread prioritytThread = new PrioritytThread();

        // 如果8核CPU处理3线程，无论优先级高低，每个线程都是单独一个CPU执行，就无法体现优先级
        // 开启10个线程，让8个CPU处理，这里线程就需要竞争CPU资源，优先级高的能分配更多的CPU资源
        for (int i = 0; i < 10; i++) {
            Thread t = new Thread(prioritytThread, "线程" + i); //开启十个线程，每个线程使用同一个prioritytThread，共同操作其中的run
            if (i == 1) {  //当线程为1时，设置为最高级
                t.setPriority(10);
            }
            if (i == 2) {  //当线程为2时，设置为最低级
                t.setPriority(1);
            }
            t.setDaemon(true);//设置为守护线程，主线程停止，守护线程也停止
            t.start();
        }

        try {
            Thread.sleep(1000l);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("线程1总计：" + PrioritytThread.count1);
        System.out.println("线程2总计：" + PrioritytThread.count2);
    }

    static class PrioritytThread implements Runnable {
        public static Integer count1 = 0;
        public static Integer count2 = 0;

        public void run() {
            while (true) {  //当前是线程1运行prioritytThread的run方法时，count1++，就是通过计时器，来判断当前是线程1运行run，还是线程2运行run
                if ("线程1".equals(Thread.currentThread().getName())) {
                    count1++;
                }
                if ("线程2".equals(Thread.currentThread().getName())) {
                    count2++;
                }

                if (Thread.currentThread().isInterrupted()) {
                    break;
                }
            }
        }
    }
}
```



##### 	2.join()方法

join作用是让其他线程变为等待。thread.Join把指定的线程加入到当前线程，可以将两个交替执行的线程合并为顺序执行的线程。比如在线程B中调用了线程A的Join()方法，直到线程A执行完毕后，才会继续执行线程B。

```java
public class Demo11Join {
    public static void main(String[] args) {
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    String name = Thread.currentThread().getName();
                    System.out.println(name+"执行"+i);
                }
            }
        }, "线程1");
        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    thread1.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                for (int i = 0; i < 10; i++) {
                    String name = Thread.currentThread().getName();
                    System.out.println(name+"执行"+i);
                }
            }
        }, "线程2");
        Thread thread3 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    thread2.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                for (int i = 0; i < 10; i++) {
                    String name = Thread.currentThread().getName();
                    System.out.println(name+"执行"+i);
                }
            }
        }, "线程3");
        thread1.start();
        thread2.start();
        thread3.start();

       /* try {
            thread3.join();
        } catch (Exception e) {

        }*/
        for (int i = 0; i < 5; i++) {
            System.out.println("main ---i:" + i);
        }
    }

}
```





##### 	3.yield方法（running->ready）

Thread.yield()方法的作用：暂停当前正在执行的线程，并执行其他线程。（可能没有效果）
yield()让当前正在运行的线程回到可运行状态，以允许具有相同优先级的其他线程获得运行的机会。因此，使用yield()的目的是让具有相同优先级的线程之间能够适当的轮换执行。但是，实际中无法保证yield()达到让步的目的，因为，让步的线程可能被线程调度程序再次选中。

查看源码介绍：

![1558669569583](assets\1558669569583.png)



结论：大多数情况下，yield()将导致线程从运行状态转到可运行状态，但有可能没有效果。

### 1.4 小结

优先级：从1-10，默认值是5，表示线程被cpu调度的概率，值越大优先级越高就越有可能被cpu调度到去执行，从宏观角度来说，不同优先级的线程在运行一段时间后，它们的优先级表示的就是线程占用cpu时间片的多少。

join方法：线程a里面调用线程b.join方法，就是让线程b先执行，b执行完了在执行线程a

yield方法：就是让当前线程让出cpu资源（running->ready切换）,可能没有效果，因为cpu有可能再度调度到该线程执行



## 5. 多线程并发的3个特性

### 1.1 目标

​	理解多线程并发的3个特性。

### 1.2 路径

1. 原子性
2. 可见性
3. 有序性

### 1.3 讲解

​	多线程并发开发中，要知道什么是多线程的原子性，可见性和有序性，以避免相关的问题产生。

#### 1.3.1 原子性（不可分割的最小单位）

原子性：``即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行``

一个很经典的例子就是银行账户转账问题：

比如从账户A向账户B转1000元，那么必然包括2个操作：从账户A减去1000元，往账户B加上1000元。

试想一下，如果这2个操作不具备原子性，会造成什么样的后果。假如从账户A减去1000元之后，操作突然中止。这样就会导致账户A虽然减去了1000元，但是账户B没有收到这个转过来的1000元。

所以这2个操作**必须要具备原子性**才能保证不出现一些意外的问题。

**回顾之前的买票案例**。

#### 1.3.2 可见性

可见性：``当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值``

前面讲过多线程的内存可见性，现在我们写一个内存不可见的问题。

案例如下：

```java
public class DemoVisible {

    public static void main(String[] args) throws InterruptedException {

        JmmDemo demo = new JmmDemo();
        Thread t = new Thread(demo);
        t.start();
        Thread.sleep(100);
        demo.flag = false;
        System.out.println("已经修改为false");
        System.out.println(demo.flag);
    }

    static class JmmDemo implements Runnable {
        public boolean flag = true;

        public void run() {
            System.out.println("子线程执行。。。");
            while (flag) {
            }
            System.out.println("子线程结束。。。");
        }
    }
}
```

执行结果

![1558602652109](assets/1558602652109.png)

​	按照main方法的逻辑，我们已经把flag设置为false，那么从逻辑上讲，子线程就应该跳出while死循环，因为这个时候条件不成立，但是我们可以看到，程序仍旧执行中，并没有停止。

​	原因:

​		1.和**java内存模型**有关，多线程访问共享变量时会创建副本变量，修改后再写回主变量。

​		2.本案例中线程一直读取副本变量没有读取主变量也和**jvm编译器**有关，由于flag变量没有加任何的同步措施，jdk认为该变量只会由单线程访问。所以导致一直读取的是副本，没有读取到主内存。

```java
//线程1执行的代码
int i = 0;
i = 10;

//线程2执行的代码
j = i;
```

当线程1执行`int i = 0`这句时，`i`的初始值0加载到内存中，然后再执行`i = 10`，那么在内存中`i`的值变为10了。

如果当线程1执行到`int i = 0`这句时，此时线程2执行 j = i，它读取`i`的值并加载到内存中，注意此时内存当中i的值是0，那么就会使得`j`的值也为0，而不是10。



这就是可见性问题，线程1对变量`i`修改了之后，线程2没有立即看到线程1修改的值。

​	**解决方案：一般使用volatile或同步锁保证其内存的可见性**

#### 1.3.3 有序性

有序性：``程序执行的顺序按照代码的先后顺序执行``

```java
boolean flag = false;
int count = 0;
count = 1; //语句1
flag = true; //语句2
```

​	以上代码定义了一个int型变量，定义了一个boolean类型变量，然后分别对两个变量进行赋值操作。从代码顺序上看，语句1是在语句2前面的，那么JVM在真正执行这段代码的时候会保证语句1一定会在语句2前面执行吗？不一定，为什么呢？这里可能会发生**指令重排序**（Instruction Reorder）。

​	什么是重排序？一般来说，处理器为了**提高程序运行效率**，可能会对输入代码进行优化，它不保证程序中各个语句的执行先后顺序同代码中的顺序一致。

​	as-if-serial:无论如何重排序，程序最终执行结果和代码顺序执行的结果是一致的。Java编译器、运行时和处理器都会保证Java在**单线程**下遵循as-if-serial语意）



​	 上面的代码中，语句1和语句2谁先执行对最终的程序结果并没有影响，那么就有可能在执行过程中，语句2先执行而语句1后执行。但是要注意，虽然处理器会对指令进行重排序，但是它会保证程序最终结果会和代码顺序执行结果相同，那么它靠什么保证的呢？



再看下面一个例子：

```java
int a = 10; //语句1
int b = 2; //语句2
a = a + 3; //语句3
b = a*a; //语句4
```

这段代码有4个语句，那么可能的一个执行顺序是：  语句2   语句1    语句3   语句4



不可能是这个执行顺序： 语句2   语句1    语句4   语句3

因为处理器在进行重排序时是会考虑指令之间的数据依赖性，如果一个指令Instruction 2必须用到Instruction 1的结果，那么处理器会保证Instruction 1会在Instruction 2之前执行。虽然重排序不会影响单个线程内程序执行的结果，但是多线程会有影响



下面看一个例子：

```java
//线程1:
init = false
context = loadContext(); //语句1
init = true; //语句2

//线程2:
while(!init){//如果初始化未完成，等待
  sleep();
}
execute(context);//初始化完成，执行逻辑
```

上面代码中，由于语句1和语句2没有数据依赖性，因此可能会被重排序。假如发生了重排序，在线程1执行过程中先执行语句2，而此是线程2会以为初始化工作已经完成，那么就会跳出while循环，去执行execute(context)方法，而此时context并没有被初始化，就会导致程序出错。

从上面可以看出，**重排序不会影响单个线程的执行，但是会影响到线程并发执行的正确性**。

### 1.4 小结

​		要想并发程序正确地执行，必须要保证原子性、可见性以及有序性。只要有一个没有被保证，就有可能会导致程序运行不正确。

​		原子性：不可分割，说明这一段程序要么都执行，要么不执行，并且不能被其他线程影响，可以使用同步方法,同步块，锁解决。

​		可见性：多线程对于共享变量的访问，必须每次都获取到主变量的值。可以使用volatile或者同步方法，同步块，锁解决。

​		原因1：和内存模型有关，每次去读取内存中的值会加载到cpu缓存。

​		原因2：jvm有时候会认为程序是单线程执行，或者执行的逻辑对变量没有影响，就会重复读取cpu缓存的变量，导致可见性的问题。

​		有序性:由jvm指令重排导致的，指令重排的结果对于单线程来说是一致性，没有什么影响（在指令重排的过程中会考虑数据依赖的问题），多线程情况下避免指令重排带来程序执行错误的危害。有些情况可以使用volatile，或者使用synchronized同步块或者同步方法或者同步锁。



## 6. Java内存可见性

### 1.1 目标

- 了解java内存模型

- 理解java内存可见性

### 1.2 路径

1. java内存模型介绍
2. java内存可见性的介绍

### 1.3 讲解

### 3.1 了解Java内存模型

​	JVM内存结构、Java对象模型和Java内存模型，这就是三个截然不同的概念，而这三个概念很容易混淆。这里详细区别一下



#### 3.1.1 JVM内存结构

​	我们都知道，Java代码是要运行在虚拟机上的，而虚拟机在执行Java程序的过程中会把所管理的内存划分为若干个不同的数据区域，这些区域都有各自的用途。其中有些区域随着虚拟机进程的启动而存在，而有些区域则依赖用户线程的启动和结束而建立和销毁。

在《Java虚拟机规范（Java SE 8）》中描述了JVM运行时内存区域结构如下：

![QQ20180624-150918](assets\QQ20180624-150918.png)



​	**JVM内存结构，由Java虚拟机规范定义。描述的是Java程序执行过程中，由JVM管理的不同数据区域。各个区域有其特定的功能。**



#### 3.1.2 Java对象模型

​	Java是一种面向对象的语言，而**Java对象**在JVM中的**存储**也是有一定的**结构**的。而这个关于Java对象自身的存储模型称之为Java对象模型。

​	HotSpot虚拟机中（Sun JDK和OpenJDK中所带的虚拟机，也是目前使用范围最广的Java虚拟机），设计了一个OOP-Klass Model。OOP（Ordinary Object Pointer）指的是普通对象指针，而Klass用来描述对象实例的具体类型。

​	每一个Java类，在被JVM加载的时候，JVM会给这个类创建一个`instanceKlass`对象，保存在方法区，用来在JVM层表示该Java类。当我们在Java代码中，使用new创建一个对象的时候，JVM会创建一个`instanceOopDesc`对象，这个对象中包含了对象头以及实例数据。

![20170615230126453](assets\20170615230126453.jpeg)

这就是一个简单的Java对象的OOP-Klass模型，即Java对象模型。



#### 3.1.3 内存模型

​	**Java内存模型就是一种符合内存模型规范的，屏蔽了各种硬件和操作系统的访问差异的，保证了Java程序在各种平台下对内存的访问都能保证效果一致的机制及规范。**

有兴趣详细了解Java内存模型是什么，为什么要有Java内存模型，Java内存模型解决了什么问题的学员，参考：<https://www.hollischuang.com/archives/2550>。

​	Java内存模型是根据英文Java Memory Model（JMM）翻译过来的。其实**JMM并不像JVM内存结构一样是真实存在的。他只是一个抽象的概念**。[JSR-133: Java Memory Model and Thread Specification](http://www.cs.umd.edu/~pugh/java/memoryModel/jsr133.pdf)中描述了，JMM是和多线程相关的，他描述了一组规则或规范，这个规范定义了一个线程对共享变量的写入时对另一个线程是可见的。

​	简单总结下，Java的多线程之间是通过共享内存进行通信的，而由于采用共享内存进行通信，在通信过程中会存在一系列如可见性、原子性、顺序性等问题，而JMM就是围绕着多线程通信以及与其相关的一系列特性而建立的模型。JMM定义了一些语法集，这些语法集映射到Java语言中就是volatile、synchronized等关键字。

![11](assets\11.png)

JMM线程操作内存的基本的规则：

第一条关于线程与主内存：线程对共享变量的所有操作都必须在自己的工作内存（本地内存）中进行，不能直接从主内存中读写

第二条关于线程间本地内存：不同线程之间无法直接访问其他线程本地内存中的变量，线程间变量值的传递需要经过主内存来完成。

* **主内存**

  主要存储的是Java实例对象，所有线程创建的实例对象都存放在主内存中，不管该实例对象是成员变量还是方法中的本地变量(也称局部变量)，当然也包括了共享的类信息、常量、静态变量。由于是共享数据区域，多条线程对同一个变量进行访问可能会发现线程安全问题。

* **本地内存**

  主要存储当前方法的所有本地变量信息(本地内存中存储着主内存中的变量副本拷贝)，每个线程只能访问自己的本地内存，即**线程中的本地变量对其它线程是不可见的**，就算是两个线程执行的是同一段代码，它们也会各自在自己的工作内存中创建属于当前线程的本地变量，当然也包括了字节码行号指示器、相关Native方法的信息。注意由于工作内存是每个线程的私有数据，线程间无法相互访问工作内存，因此存储在工作内存的数据不存在线程安全问题。



####3.1.4 小结

​	JVM内存结构，和Java虚拟机的运行时区域有关。 

​	Java对象模型，和Java对象如何在jvm中存储的描述。 

​    **Java内存模型，和Java的并发编程有关**。



### 3.2 内存可见性

#### 3.2.1 内存可见性介绍

`可见性：`一个线程对共享变量值的修改，能够及时的被其他线程看到

`共享变量：`如果一个变量在多个线程的工作内存中都存在副本，那么这个变量就是这几个线程的共享变量



线程 A 与线程 B 之间如要通信的话，必须要经历下面 2 个步骤：

  本地内存（类似于A的缓存区域，缓存存在于cpu上）  主内存：类似于类似于我们使用的物理内存

1. 首先，线程 A 把本地内存 A 中更新过的共享变量刷新到主内存中去。
2. 然后，线程 B 到主内存中去读取线程 A 之前已更新过的共享变量。



![img](assets\2c452d147bf0d09b14b770d3990740cb.png)

​	如上图所示，本地内存 A 和 B 有主内存中共享变量 x 的副本。假设初始时，这三个内存中的 x 值都为 0。线程 A 在执行时，把更新后的 x 值（假设值为 1）临时存放在自己的本地内存 A 中。当线程 A 和线程 B 需要通信时，线程 A 首先会把自己本地内存中修改后的 x 值刷新到主内存中，此时主内存中的 x 值变为了 1。随后，线程 B 到主内存中去读取线程 A 更新后的 x 值，此时线程 B 的本地内存的 x 值也变为了 1。

​	从整体来看，这两个步骤实质上是线程 A 在向线程 B 发送消息，而且这个通信过程必须要经过主内存。JMM 通过控制主内存与每个线程的本地内存之间的交互，来为 java 程序员提供内存可见性保证。

​	**回顾上面多线程并发的可见性问题的案例**



## 7. synchronized

### 1.1 目标

- 理解synchronized实现可见性的过程
- 理解同步原理
- 理解jvm是如何进行锁优化的

### 1.2 路径

1. 解决可见性问题
2. 同步原理
3. 锁优化

### 1.3 讲解

​	synchronized可以保证方法或者代码块在运行时，同一时刻只有一个线程执行synchronized声明的代码块。还可以保证共享变量的内存可见性。同一时刻只有一个线程执行，这部分代码块的重排序也不会影响其执行结果。也就是说使用了synchronized可以保证并发的原子性，可见性，有序性。

#### 1.3.1 解决可见性问题 

JMM关于synchronized的两条规定：

线程加锁时（进入同步代码块时）：将清空本地内存中共享变量的值，从而使用共享变量时需要从主内存中重新读取最新的值（加锁与解锁是同一把锁）

线程解锁前（退出同步代码块时）：必须把自己工作内存中共享变量的最新值刷新到主内存中



做如下修改，在死循环中添加同步代码块

```java
        while (flag) {
            synchronized (this) {
            }
        }
```



synchronized实现可见性的过程

1. 获得互斥锁（同步获取锁）
2. 清空本地内存
3. 从主内存拷贝变量的最新副本到本地内存
4. 执行代码
5. 将更改后的共享变量的值刷新到主内存
6. 释放互斥锁





#### 1.3.2 同步原理 

synchronized的同步可以解决原子性、可见性和有序性的问题，那是如何实现同步的呢？

Java中每一个对象都可以作为锁，这是synchronized实现同步的基础：

1. 普通同步方法，锁是当前实例对象this
2. 静态同步方法，锁是当前类的class对象
3. 同步方法块，锁是括号里面的对象


当一个线程访问同步代码块时，它首先是需要得到锁才能执行同步代码，当退出或者抛出异常时必须要释放锁。



synchronized的同步操作主要是monitorenter和monitorexit这两个jvm指令实现的，先写一段简单的代码：

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

![50061740](assets/1558750061740.png)




#### 1.3.3 锁优化

​	synchronized是重量级锁，效率不高。但在jdk 1.6中对synchronize的实现进行了各种优化，使得它显得不是那么重了。jdk1.6对锁的实现引入了大量的优化，如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销。

​	锁主要存在四中状态，依次是：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态，他们会随着竞争的激烈而逐渐升级。

**注意锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率。**



##### 1 偏向锁

​	轻量级锁的加锁解锁操作是需要依赖多次CAS原子指令的。而偏向锁只需要检查是否为偏向锁、锁标识为以及ThreadID即可，可以减少不必要的CAS操作。



##### 2 轻量级锁

​	synchronized会在对象的头部打标记，这个加锁的动作是必须要做的，悲观锁通常还会做许多其他的指令动作，轻量级锁希望通过CAS实现，它认为通过CAS尝试修改对象头部的mark区域的内容就可以达到目的，由于mark区域的宽度通常是4~8字节，也就是相当于一个int或者long的宽度，适合于CAS操作。

轻量级锁通常会做一下4个步骤：

- 在栈中分配一块空间用来做一份对象头部mark word的拷贝，在mark word中将对象锁的二进制位设置为“未锁定”(在32位的JVM中通常有2位用于存储锁标记，未锁定的标记为01)，这个动作是方便等到释放锁的时候将这份数据拷贝到对象头部。
- 通过CAS尝试将头部的二进制位修改为“线程私有栈中对mark区域拷贝存放的地址”，如果成功，则会将最后2位设置为00，代表已经被轻量级锁锁住了。
- 如果没有成功，则判定对象头部是否已经指向了当前线程所在的栈当中，如果成立则代表当前线程已经是拥有着，可以继续执行。
- 如果不是拥有着，则说明有多个线程在争用，那么此时会将锁升级为悲观锁，线程进入BLOCKED状态。

​    JVM发现在轻量级锁里面多次“重入”和“释放”时，需要做的判断和拷贝动作还是很多，而在某些应用程序中，锁就是被某一个线程一直使用，为了进一步减小锁的开销，JVM中出现了偏向锁，偏向锁希望记录的是一个线程ID，它比轻量级锁更加轻量，当再次重入判定时，首先判定对象头部的线程ID是不是当前线程，若是则表示当前线程已经是对象锁的OWNER，无须做其他任何动作。



##### 3 自旋锁

​	 轻量级锁失败后，虚拟机为了避免线程真实地在操作系统层面挂起，还会进行一项称为自旋锁的优化手段。这是基于 在大多数情况下 对象锁的锁状态只会**持续很短一段时间**，为了这一段很短的时间频繁地阻塞和唤醒线程是非常不值得的。所以引入自旋锁。

​	所谓自旋锁，就是让该线程等待一段时间，不会被立即挂起，看持有锁的线程是否会很快释放锁。怎么等待呢？执行一段无意义的循环即可（自旋）。

​	自旋等待不能替代阻塞，虽然它可以避免线程切换带来的开销，但是它占用了处理器的时间。如果持有锁的线程很快就释放了锁，那么自旋的效率就非常好，反之，自旋的线程就会白白消耗掉处理的资源，它不会做任何有意义的工作，典型的占着茅坑不拉屎，这样反而会带来性能上的浪费。所以说，自旋等待的时间（自旋的次数）必须要有一个限度，如果自旋超过了定义的时间仍然没有获取到锁，则应该被挂起。

​	自旋锁在JDK 1.4.2中引入，默认关闭，但是可以使用-XX:+UseSpinning开开启，在JDK1.6中默认开启。同时自旋的默认次数为10次，可以通过参数-XX:PreBlockSpin来调整；

​	如果通过参数-XX:preBlockSpin来调整自旋锁的自旋次数，会带来诸多不便。假如我将参数调整为10，但是系统很多线程都是等你刚刚退出的时候就释放了锁（假如你多自旋一两次就可以获取锁），你是不是很尴尬。于是JDK1.6引入自适应的自旋锁，让虚拟机会变得越来越聪明。



##### 4 适应自旋锁

​	JDK 1.6引入了更加聪明的自旋锁，即自适应自旋锁。所谓自适应就意味着自旋的次数不再是固定的，它是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。它怎么做呢？线程如果自旋成功了，那么下次自旋的次数会更加多，因为虚拟机认为既然上次成功了，那么此次自旋也很有可能会再次成功，那么它就会允许自旋等待持续的次数更多。反之，如果对于某个锁，很少有自旋能够成功的，那么在以后要或者这个锁的时候自旋的次数会减少甚至省略掉自旋过程，以免浪费处理器资源。

有了自适应自旋锁，随着程序运行和性能监控信息的不断完善，虚拟机对程序锁的状况预测会越来越准确，虚拟机会变得越来越聪明。



##### 5 锁消除

​	为了保证数据的完整性，我们在进行操作时需要对这部分操作进行同步控制，但是在有些情况下，JVM检测到不可能存在共享数据竞争，这是JVM会对这些同步锁进行锁消除。锁消除的依据是逃逸分析的数据支持。

​	如果不存在竞争，为什么还需要加锁呢？所以锁消除可以节省毫无意义的请求锁的时间。变量是否逃逸，对于虚拟机来说需要使用数据流分析来确定，但是对于我们程序员来说这还不清楚么？我们会在明明知道不存在数据竞争的代码块前加上同步吗？但是有时候程序并不是我们所想的那样？我们虽然没有显示使用锁，但是我们在使用一些JDK的内置API时，如StringBuffer、Vector、HashTable等，这个时候会存在隐形的加锁操作。比如StringBuffer的append()方法，Vector的add()方法：

```java
public void test(){
    Vector<Integer> vector = new Vector<Integer>();
    for(int i = 0 ; i < 10 ; i++){
        vector.add(i);
    }
    System.out.println(vector);
}
```

​	在运行这段代码时，JVM可以明显检测到变量vector没有逃逸出方法vectorTest()之外，所以JVM可以大胆地将vector内部的加锁操作消除。



##### 6 锁粗化

​	在使用同步锁的时候，需要让同步块的作用范围尽可能小，仅在共享数据的实际作用域中才进行同步，这样做的目的是为了使需要同步的操作量尽可能缩小，如果存在锁竞争，那么等待锁的线程也能尽快拿到锁。

​	在大多数的情况下，上述观点是正确的。但是如果一系列的连续加锁解锁操作，可能会导致不必要的性能损耗，所以引入锁粗化的概念。

​	锁粗话概念比较好理解，就是将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁。如上面实例：vector每次add的时候都需要加锁操作，JVM检测到对同一个对象（vector）连续加锁、解锁操作，会合并一个更大范围的加锁、解锁操作，即加锁解锁操作会移到for循环之外。

````java
Vector<Integer> vector = new Vector<Integer>();
public void test(){
    for(int i = 0 ; i < 10 ; i++){
        vector.add(i);
    }
    System.out.println(vector);
}
````






##### 7 重量锁

​	重量级锁通过对象内部的监视器（monitor）实现，其中monitor的本质是依赖于底层操作系统的Mutex Lock（互斥锁）实现，操作系统实现线程之间的切换需要从用户态到内核态的切换，切换成本非常高。因为用户态与内核态都有各自专用的内存空间，专用的寄存器等，用户态切换至内核态需要传递给许多变量、参数给内核，内核也需要保护好用户态在切换时的一些寄存器值、变量等，以便内核态调用结束后切换回用户态继续工作。

​    因为需要限制不同的程序之间的访问能力, 防止他们获取别的程序的内存数据, 或者获取外围设备的数据, 并发送到网络, CPU划分出两个权限等级 :用户态 和 内核态

- 内核态：CPU可以访问内存所有数据, 包括外围设备, 例如硬盘, 网卡. CPU也可以将自己从一个程序切换到另一个程序
- 用户态：只能受限的访问内存, 且不允许访问外围设备. 占用CPU的能力被剥夺, CPU资源可以被其他程序获取

​    所有用户程序都是运行在用户态的, 但是有时候程序确实需要做一些内核态的事情, 例如从硬盘读取数据, 或者从键盘获取输入等.，而唯一可以做这些事情的就是操作系统, 所以此时程序就需要先操作系统请求以程序的名义来执行这些操作（比如java的I/O操作底层都是通过native方法来调用操作系统）。这时需要一个这样的机制: 用户态程序切换到内核态, 但是不能控制在内核态中执行的指令。这种机制叫系统调用, 在CPU中的实现称之为陷阱指令(Trap Instruction)。



### 总结

1. 线程和进程区别

   1. 进程独享内存空间，线程之间是共享堆内存空间，独享栈内存空间
   2. 进程是程序的一次运行，一个进程可能包含很多线程，线程是一次任务执行的最小单元。

2. **多线程创建**

   1. 继承Thread类，重写run方法
   2. 实现runable接口，实现run方法
   3. 匿名内部类实现runable接口

3. **多线程创建方式注意的点**

   1. 实现runable接口可以让多个线程同时执行一个任务，继承Thread类方式不行
   2. 线程创建后要通过start方法运行，通过本地native方法调用底层c同名方法创建线程执行，不要通过run方法，因为run方法相当于在主线程运行类的run方法而已，不会创建线程执行

4. 线程安全（多线程执行同一段代码，如果发生和预期结果不一致的情况就是线程不安全的）

   1. 由于不满足原子性，有序性，可见性导致线程安全问题
   2. 解决方案：加锁解决
   3. 死锁：线程之间互相等待对方释放锁产生死锁。解决方案：不要在同步中嵌套同步

5. 线程同步方式Synchorized（Synchorized都是对对象加锁）

   1. 同步代码块加锁 sync...(obj)->对obj对象加锁
   2. 同步方法 sync fun01->对this对象加锁
   3. 静态同步方法 static sync fun02->对当前类的class加锁

6. 线程状态

   1. new（新建线程未执行）
   2. runable（可运行）：由ready（就绪）和running（运行中）两个状态组成,状态的切换是由cpu调度切换的，cpu调度到该线程，就是running，没调度到就是ready
   3. waiting：由当前获取到锁的线程，锁对象.wait()方法引起的，只能由同一个对象的notify（）/nodifyAll()方法唤醒
   4. timed_waiting: sleep（时间），wait（时间），到达时间后自动唤醒
   5. blocked：线程获取不到锁就进入阻塞状态
   6. TERMINATED：线程终止

7. **wait和sleep区别**

   1. wait是属于Object类的，sleep是属于Thread类的
   2. wait会让持有锁的线程释放锁，并进入waiting状态，sleep是不会释放锁的，必须有等待时间，时间过后继续执行

8. 线程停止

   1. 通过自定义标识：缺点是线程sleep情况下不能被立刻停止
   2. 通过线程自带标识：
      1. 当前线程正常执行：Thread.currentThread().isInterrupt()判断为true退出
      2. 当前线程sleep：会抛出异常，捕获后退出

9. 线程优先级

   1. 从1-10，表示的是cpu调度到该线程的概率，值越大优先级越高，越有可能被调度到，默认值是5，在宏观角度来看就是占用cpu时间片的多少
   2. join：在线程b运行的时候执行线程a.join,a先执行执行完了执行b
   3. yield：将当前运行线程running->ready，但是可能没效果，该线程可能被再次调度

10. 保证线程安全的三个特性

    1. 原子性：线程执行一段代码的过程中不能被其他的线程干扰对结果产生影响。
    2. 可见性：线程修改了变量之后能够立刻被其他线程看见
    3. 有序性：由jvm指令重排序导致的，jvm指令重排能够保证在单线程执行下结果一致，多线程不能保证。

11. jvm内存结构：jvm运行时的区域划分

12. java对象模型：对象在jvm内存中的存储

13. 内存模型：jmm不是真实存在的是虚拟的，它的语法集映射在java语言中就是volidate，synchronized

14. synchronized

    1. 保证原子性：因为被synchronized包裹的代码，只能被一个线程执行，所以不会被其他线程干扰
    2. 保证可见性：在加锁时，会清空本地内存变量，解锁的时候会把修改的本地内存变量刷到主内存里面去
    3. 保证有序性：因为被synchronized包裹的代码，只能被一个线程执行,jvm指令重排的结果对单线程没有影响，就算发生了指令重排对synchronized包裹的代码也不会有影响
    4. 加锁：对象的moniterEnter和moniterExit加锁和解锁

15. 锁优化

    1. 自旋锁：循环
    2. 适应自旋锁：循环，成功，给锁多循环几次的机会，失败则减少机会，如果多次失败就直接进入重量级锁
    3. 锁消除：如果jvm检测到锁没有竞争就把锁消除了
    4. 锁粗化：把一个个小的锁换成一个大锁
    5. 偏向锁：在markWord中保存threadid，发生竞争就升级
    6. 轻量级锁：通过cas实现
    7. 重量级锁：加锁解锁会导致线程从用户态到核心态的切换，消耗比较大的资源。

16. **i++为什么不是线程安全的**

    1. 把变量读到cpu缓存
    2. 操作缓存中的值++
    3. 写回主内存
    4. 由于上述三个步骤不是原子性的，所以会导致线程安全问题

    

    

    





