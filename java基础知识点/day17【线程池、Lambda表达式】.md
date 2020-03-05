1.![1572266324873](C:\Users\HC\AppData\Roaming\Typora\typora-user-images\1572266324873.png)

2.![1572267776114](C:\Users\HC\AppData\Roaming\Typora\typora-user-images\1572267776114.png)

2.

![1572493731572](C:\Users\HC\AppData\Roaming\Typora\typora-user-images\1572493731572.png)

## 主要内容

# day07【线程池、Lambda表达式】

## 主要内容

*   等待唤醒机制
*   线程池
*   Lambda表达式

## 教学目标

-[ ] 能够理解线程通信概念

​	多个线程在处理同一个资源，但是处理的动作（线程的任务）却不相同。

-[ ] 能够理解等待唤醒机制

```
 等待唤醒机制:是多个线程间的一种协作机制
                    在一个线程进行了规定操作后，就进入等待状态（wait()），
                    等待其他线程执行完他们的指定代码过后 再将其唤醒（notify()）;
                    在有多个线程进行等待时， 如果需要，可以使用 notifyAll()来唤醒所有的等待线程。

 使用wait()和notify()以及notifyAll()方法来实现等待唤醒机制:
                - 调用wait()方法会使得线程进入无线等待,这个时候无限等待线程不会去争夺锁,不会争夺cpu
                - 调用notify()以及notifyAll()方法唤醒无限等待线程
                - 调用wait()和notify()以及notifyAll()方法必须使用同一个锁对象

                - 线程进入无限等待之后,被唤醒了,并且获取到了锁对象,那么就会从唤醒的位置继续往下执行...
                - 线程间的调度是抢占式调度
```



-[ ] 能够描述Java中线程池运行原理

​	1.创建线程池的时候初始化几条线程在线程池中

​	2.添加任务到线程池中,线程池就会分配空闲的线程来执行任务

​	3.当任务执行完了之后,这个线程需要回到线程池中

-[ ] 能够理解函数式编程相对于面向对象的优点

​	无须创建接口的实现类,无须创建接口的对象,只需要传入Lambda表达式("方法")

-[ ] 能够掌握Lambda表达式的标准格式

​	(形参) -> { 方法体 }

-[ ] 能够使用Lambda标准格式使用Runnable与Comparator接口

​	lue 

-[ ] 能够掌握Lambda表达式的省略格式与规则

​	2.1 小括号中的形参的类型可以省略

​	2.2 如果小括号中的只有一个形参,那么这对小括号,还有形参类型可以省略

​	2.3 如果大括号中只有一句代码,那么可以同时省略: 这对大括号,分号,return关键字

-[ ] 能够使用Lambda省略格式使用Runnable与Comparator接口

​	略

-[ ] 能够通过Lambda的标准格式使用自定义的接口（有且仅有一个抽象方法）

-[ ] 能够通过Lambda的省略格式使用自定义的接口（有且仅有一个抽象方法）
-[ ] 能够明确Lambda的两项使用前提

​	接口是一个函数式接口,也就是有且仅有一个抽象方法

# 第一章 等待唤醒机制

## 1.1 线程间通信

### 目标:

- 线程间通信

### 步骤:

- 线程间通信的概述
- 为什么要处理线程间通信
- 如何保证线程间通信有效利用资源

### 讲解:

**概念：**多个线程在处理同一个资源，但是处理的动作（线程的任务）却不相同。

比如：线程A用来生成包子的，线程B用来吃包子的，包子可以理解为同一资源，线程A与线程B处理的动作，一个是生产，一个是消费，那么线程A与线程B之间就存在线程通信问题。

![](img\线程间通信.bmp)

**为什么要处理线程间通信：**

多个线程并发执行时, 在默认情况下CPU是随机切换线程的，当我们需要多个线程来共同完成一件任务，并且我们希望他们有规律的执行, 那么多线程之间需要一些协调通信，以此来帮我们达到多线程共同操作一份数据。

**如何保证线程间通信有效利用资源：**

多个线程在处理同一个资源，并且任务不同时，需要线程通信来帮助解决线程之间对同一个变量的使用或操作。 就是多个线程在操作同一份数据时， 避免对同一共享变量的争夺。也就是我们需要通过一定的手段使各个线程能有效的利用资源。而这种手段即—— **等待唤醒机制。**

### 小结:

- **概念：**多个线程在处理同一个资源，但是处理的动作（线程的任务）却不相同。
- 因为我们需要使得线程按照规律的执行,所以需要让线程间有效通信
- 使用等待唤醒机制来实现线程间有规律的执行

## 1.2 等待唤醒机制

### 目标:

- 等待唤醒机制的概述

### 步骤:

- 什么是等待唤醒机制
- 等待唤醒中的方法
- 调用wait和notify方法需要注意的细节

### 讲解:

**什么是等待唤醒机制**

这是多个线程间的一种**协作**机制。谈到线程我们经常想到的是线程间的**竞争（race）**，比如去争夺锁，但这并不是故事的全部，线程间也会有协作机制。就好比在公司里你和你的同事们，你们可能存在在晋升时的竞争，但更多时候你们更多是一起合作以完成某些任务。

就是在一个线程进行了规定操作后，就进入等待状态（**wait()**）， 等待其他线程执行完他们的指定代码过后 再将其唤醒（**notify()**）;在有多个线程进行等待时， 如果需要，可以使用 notifyAll()来唤醒所有的等待线程。

wait/notify 就是线程间的一种协作机制。

**等待唤醒中的方法**

等待唤醒机制就是用于解决线程间通信的问题的，使用到的3个方法的含义如下：

1. wait：线程不再活动，不再参与调度，**进入 wait set 中，因此不会浪费 CPU 资源，也不会去竞争锁了**，这时的线程状态即是 WAITING。它还要执行一个**特别的动作**，也即是“**通知（notify）**”在这个对象上等待的线程从wait set 中释放出来，重新进入到调度队列（ready queue）中
2. notify：则选取所通知对象的 wait set 中的一个线程释放；例如，餐馆有空位置后，等候就餐最久的顾客最先入座。
3. notifyAll：则释放所通知对象的 wait set 上的全部线程。

>注意：
>
>哪怕只通知了一个等待的线程，被通知线程也不能立即恢复执行，因为它当初中断的地方是在同步块内，而此刻它已经不持有锁，所以她需要再次尝试去获取锁（很可能面临其它线程的竞争），成功后才能在当初调用 wait 方法之后的地方恢复执行。
>
>总结如下：
>
>- 如果能获取锁，线程就从 WAITING 状态变成 RUNNABLE 状态；
>- 否则，从 wait set 出来，又进入 entry set，线程就从 WAITING 状态又变成 BLOCKED 状态

**调用wait和notify方法需要注意的细节**

1. wait方法与notify方法必须要由同一个锁对象调用。因为：对应的锁对象可以通过notify唤醒使用同一个锁对象调用的wait方法后的线程。
2. wait方法与notify方法是属于Object类的方法的。因为：锁对象可以是任意对象，而任意对象的所属类都是继承了Object类的。
3. wait方法与notify方法必须要在同步代码块或者是同步函数中使用。因为：必须要通过锁对象调用这2个方法。

### 小结:

- 等待唤醒机制其实就是一种线程间协作机制

- 使用wait()和notify()以及notifyAll()方法来实现等待唤醒机制:

  - 调用wati()方法会使得线程进入无线等待,这个时候不会去争夺锁,不会争夺cpu

  - 调用notify()以及notifyAll()方法唤醒无限等待线程

  - 调用wait()和notify()以及notifyAll()方法必须使用同一个锁对象

    

## 1.3 生产者与消费者问题

### 目标:

- 生产者与消费者问题

### 步骤:

- 生产者与消费者问题

### 讲解:

等待唤醒机制其实就是经典的“生产者与消费者”的问题。

就拿生产包子消费包子来说等待唤醒机制如何有效利用资源：

![](img\线程间通信.bmp)

~~~java
包子铺线程生产包子，吃货线程消费包子。当包子没有时（包子状态为false），吃货线程等待，包子铺线程生产包子（即包子状态为true），并通知吃货线程（解除吃货的等待状态）,因为已经有包子了，那么包子铺线程进入等待状态。接下来，吃货线程能否进一步执行则取决于锁的获取情况。如果吃货获取到锁，那么就执行吃包子动作，包子吃完（包子状态为false），并通知包子铺线程（解除包子铺的等待状态）,吃货线程进入等待。包子铺线程能否进一步执行则取决于锁的获取情况。
~~~

**代码演示：**

包子资源类：

~~~java
public class BaoZi {
     String  pier ;
     String  xianer ;
     boolean  flag = false ;//包子资源 是否存在  包子资源状态
}
~~~

吃货线程类：

~~~java
public class ChiHuo extends Thread{
    private BaoZi bz;

    public ChiHuo(String name,BaoZi bz){
        super(name);
        this.bz = bz;
    }
    @Override
    public void run() {
        while(true){
            synchronized (bz){
                if(bz.flag == false){//没包子
                    try {
                        bz.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("吃货正在吃"+bz.pier+bz.xianer+"包子");
                bz.flag = false;
                bz.notify();
            }
        }
    }
}
~~~

包子铺线程类：

~~~java
public class BaoZiPu extends Thread {

    private BaoZi bz;

    public BaoZiPu(String name,BaoZi bz){
        super(name);
        this.bz = bz;
    }

    @Override
    public void run() {
        int count = 0;
        //造包子
        while(true){
            //同步
            synchronized (bz){
                if(bz.flag == true){//包子资源  存在
                    try {

                        bz.wait();

                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                // 没有包子  造包子
                System.out.println("包子铺开始做包子");
                if(count%2 == 0){
                    // 冰皮  五仁
                    bz.pier = "冰皮";
                    bz.xianer = "五仁";
                }else{
                    // 薄皮  牛肉大葱
                    bz.pier = "薄皮";
                    bz.xianer = "牛肉大葱";
                }
                count++;

                bz.flag=true;
                System.out.println("包子造好了："+bz.pier+bz.xianer);
                System.out.println("吃货来吃吧");
                //唤醒等待线程 （吃货）
                bz.notify();
            }
        }
    }
}
~~~

测试类：

~~~java
public class Demo {
    public static void main(String[] args) {
        //等待唤醒案例
        BaoZi bz = new BaoZi();

        ChiHuo ch = new ChiHuo("吃货",bz);
        BaoZiPu bzp = new BaoZiPu("包子铺",bz);

        ch.start();
        bzp.start();
    }
}
~~~

执行效果：

~~~java
包子铺开始做包子
包子造好了：冰皮五仁
吃货来吃吧
吃货正在吃冰皮五仁包子
包子铺开始做包子
包子造好了：薄皮牛肉大葱
吃货来吃吧
吃货正在吃薄皮牛肉大葱包子
包子铺开始做包子
包子造好了：冰皮五仁
吃货来吃吧
吃货正在吃冰皮五仁包子
~~~

### 小结:

略

# 第二章 线程池

## 2.1 线程池思想概述

### 目标:

- 了解线程池的概述 

### 步骤:

- 了解线程池的概述 

### 讲解:

![](img\游泳池.jpg)

我们使用线程的时候就去创建一个线程，这样实现起来非常简便，但是就会有一个问题：

如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，这样频繁创建线程就会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间。

那么有没有一种办法使得线程可以复用，就是执行完一个任务，并不被销毁，而是可以继续执行其他的任务？

在Java中可以通过线程池来达到这样的效果。今天我们就来详细讲解一下Java的线程池。

### 小结:

略

## 2.2 线程池概念

### 目标:

- 线程池的概念

### 步骤:

- 线程池的概念

### 讲解:

* **线程池：**其实就是一个容纳多个线程的容器，其中的线程可以反复使用，省去了频繁创建线程对象的操作，无需反复创建线程而消耗过多资源。

由于线程池中有很多操作都是与优化资源相关的，我们在这里就不多赘述。我们通过一张图来了解**线程池的工作原理：**

![](img\线程池原理.bmp)

合理利用线程池能够带来三个好处：

1. 降低资源消耗。减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。
2. 提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
3. 提高线程的可管理性。可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。

### 小结:

略

## 2.3 线程池的使用

### 目标:

- 线程池的使用

### 步骤:

- 线程池的使用

### 讲解:

Java里面线程池的顶级接口是`java.util.concurrent.Executor`，但是严格意义上讲`Executor`并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是`java.util.concurrent.ExecutorService`。

要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，很有可能配置的线程池不是较优的，因此在`java.util.concurrent.Executors`线程工厂类里面提供了一些静态工厂，生成一些常用的线程池。官方建议使用Executors工厂类来创建线程池对象。

Executors类中有个创建线程池的方法如下：

* `public static ExecutorService newFixedThreadPool(int nThreads)`：返回线程池对象。(创建的是有界线程池,也就是池中的线程个数可以指定最大数量)

获取到了一个线程池ExecutorService 对象，那么怎么使用呢，在这里定义了一个使用线程池对象的方法如下：

* `public Future<?> submit(Runnable task)`:获取线程池中的某一个线程对象，并执行

  > Future接口：用来记录线程任务执行完毕后产生的结果。线程池创建与使用。

使用线程池中线程对象的步骤：

1. 创建线程池对象。
2. 创建Runnable接口子类对象。(task)
3. 提交Runnable接口子类对象。(take task)
4. 关闭线程池(一般不做)。

Runnable实现类代码：

~~~java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("我要一个教练");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("教练来了： " + Thread.currentThread().getName());
        System.out.println("教我游泳,交完后，教练回到了游泳池");
    }
}
~~~

线程池测试类：

~~~java
public class ThreadPoolDemo {
    public static void main(String[] args) {
        // 创建线程池对象
        ExecutorService service = Executors.newFixedThreadPool(2);//包含2个线程对象
        // 创建Runnable实例对象
        MyRunnable r = new MyRunnable();

        //自己创建线程对象的方式
        // Thread t = new Thread(r);
        // t.start(); ---> 调用MyRunnable中的run()

        // 从线程池中获取线程对象,然后调用MyRunnable中的run()
        service.submit(r);
        // 再获取个线程对象，调用MyRunnable中的run()
        service.submit(r);
        service.submit(r);
        // 注意：submit方法调用结束后，程序并不终止，是因为线程池控制了线程的关闭。
        // 将使用完的线程又归还到了线程池中
        // 关闭线程池
        //service.shutdown();
    }
}
~~~

### 小结:

略

# 第三章 Lambda表达式

## 3.1 函数式编程相对于面向对象的对比

### 目标:

- 函数式编程相对于面向对象的优点

### 路径:

- 面向对象方式创建并启动线程
- 函数式编程创建并启动线程

### 讲解:

理解

```java
public class MyRunnable implements Runnable{
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("i = " + i);

        }
}
}
public class Demo {
    public static void main(String[] args) {
        //1.使用子类
    //    new Thread(new MyRunnable()).start();

        //2.使用匿名内部类
        /*new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 100; i++) {
                    System.out.println("i = " + i);

                }
            }
        }).start();*/

        //3.使用Lambda表达式
        new Thread(()-> {
            for (int i = 0; i < 100; i++) {
                System.out.println("i = " + i);
            }
        }).start();
        
        
        for (int k = 0; k < 100; k++) {
            System.out.println("k = " + k);

        }
    }
}

```

- 面向对象编程思想:注重的是对象,所以如果想要完成某个功能,只需要找到能完成该功能的对象,让其完成即可.
- 函数式编程思想:**强调做什么，而不是以什么形式做**。特点:直接给解决方案,不需要依靠对象

### 小结:

- Lambda表达式：它是一个JDK8开始一个新语法。代替我们之前编写的 "面向某种接口”编程的情况 
- 函数式编程思想：  不需要定义子类；不需要创建子类对象；只需要传入一个“方法”即可。 
- 面向对象编程思想:需要.先定义Runnable子类；创建子类对象；传入子类对象

## 3.2 Lambda标准格式

### 目标:

- Lambda表达式的标准格式

### 步骤:

- Lambda表达式的标准格式

### 讲解:

Lambda表达式的**标准格式**为：

```
(参数类型 参数名称) -> { 代码语句 }
```

格式说明：

* 小括号内的语法与传统方法参数列表一致：无参数则留空；多个参数则用逗号分隔。
* `->`是新引入的语法格式，代表指向动作。
* 大括号内的语法与传统方法体要求基本一致。

### 小结:

略

## 3.3 使用Lambda标准格式使用Runnable与Comparator接口

### 目标:

- 使用Lambda标准格式使用Runnable与Comparator接口

### 路径:

- 使用Lambda标准格式使用Runnable
- 使用Lambda标准格式使用Comparator接口

### 讲解:

- 案例一:

  ```java
  new Thread(()->{
      for (int i = 0; i < 100; i++) {
          System.out.println("i = " + i);
      }
  
  }).start();
  ```

  

- 案例二:

  ```java
  ArrayList<String> list = new ArrayList<>();
  list.add("jfdksl");
  list.add("jfdskljfs");
  list.add("afds");
  list.add("jfdowjieoj");
  
          
  Collections.sort(list,(String s1,String s2) -> {//直接重写compare()方法
      return s1.length() - s2.length();
      //由于compare()方法声明了返回一个int值，所以这里重写时必须返回一个int值。
  });
  
  ```

### 小结:

略

## 3.4 Lambda表达式的省略格式与规则

### 目标:

- 掌握Lambda表达式的省略格式与规则

### 路径:

- Lambda表达式的省略格式与规则

### 讲解:

在Lambda标准格式的基础上，使用省略写法的规则为：

1. 形参：形参的数据类型都可以省略。

2. 形参：如果形参只有一个，可以同时省略：一对小括号，和数据类型。

   ​                如果省略“小括号”，必须同时省略“数据类型”。

   ​                如果省略“数据类型”，可以不省略“小括号”。

3. 方法体：如果方法体中只有一句话，可以同时省略：一对大括号，语句后的分号，return关键字；

### 小结:

略

## 3.5 使用Lambda省略格式使用Runnable与Comparator接口

### 目标:

- 使用Lambda省略格式使用Runnable与Comparator接口

### 路径:

- 使用Lambda省略格式使用Runnable
- 使用Lambda省略格式使用Comparator接口

### 讲解:

- 案例一:

  ```java
  // 标准格式
  new Thread(()->{
      System.out.println("多线程需要执行的代码");
  }).start();
  
  // 省略格式:
  new Thread(() ->
          System.out.println("多线程需要执行的代码")
   ).start();
  ```

  

- 案例二:

  ```java
  ArrayList<String> list = new ArrayList<>();
  list.add("jfdksl");
  list.add("jfdskljfs");
  list.add("afds");
  list.add("jfdowjieoj");
  
   // 标准格式       
  Collections.sort(list,(String s1,String s2) -> {//直接重写compare()方法
      return s1.length() - s2.length();
      //由于compare()方法声明了返回一个int值，所以这里重写时必须返回一个int值。
  });
  
  // 省略格式     
  Collections.sort(list,( s1,  s2) -> 
         s1.length() - s2.length()
  );
  
  ```

### 小结:

略

## 3.6 使用Lambda表达式的前提

### 目标:

- 使用Lambda表达式的前提

### 路径:

- 使用Lambda表达式的前提

### 讲解:

- 面向“接口”；
- “接口”中，有、且只有一个抽象方法的时候——这种接口叫：函数式接口。
  - 已经接触过的几个函数式接口：
    - Runnable接口
    - Comparator接口
  - 函数式接口，都会使用一个注解：@FunctionalInterface修饰，用这个注解修饰的，都是“函数式接口”。
- 只有当我们面向“函数式接口"编程时，才可以使用Lambda表达式。

### 小结:

- 只有是函数式接口的才可以使用Lambda表达式
- 函数式接口:有、且只有一个抽象方法的接口
- 如果要标识该接口是函数式接口,就使用@FunctionalInterface注解修饰

## 3.7 练习：通过Lambda的标准格式使用自定义的接口

### 题目

给定一个厨子`Cook`接口，内含唯一的抽象方法`makeFood`，且无参数、无返回值。如下：

```java
public interface Cook {
    void makeFood();
}
```

在下面的代码中，请使用Lambda的**标准格式**调用`invokeCook`方法，打印输出“吃饭啦！”字样：

```java
public class Demo05InvokeCook {
    public static void main(String[] args) {
        // TODO 请在此使用Lambda【标准格式】调用invokeCook方法
    }

    private static void invokeCook(Cook cook) {
        cook.makeFood();
    }
}
```

### 解答

```java
public static void main(String[] args) {
    invokeCook(() -> {
      	System.out.println("吃饭啦！");
    });
}
```

> 备注：小括号代表`Cook`接口`makeFood`抽象方法的参数为空，大括号代表`makeFood`的方法体。



## 3.8 练习：使用Lambda标准格式使用自定义的接口

### 题目

给定一个计算器`Calculator`接口，内含抽象方法`calc`可以将两个int数字相加得到和值：

```java
public interface Calculator {
    int calc(int a, int b);
}
```

在下面的代码中，请使用Lambda的**标准格式**调用`invokeCalc`方法，完成120和130的相加计算：

```java
public class Demo08InvokeCalc {
    public static void main(String[] args) {
        // TODO 请在此使用Lambda【标准格式】调用invokeCalc方法来计算120+130的结果ß
    }

    private static void invokeCalc(int a, int b, Calculator calculator) {
        int result = calculator.calc(a, b);
        System.out.println("结果是：" + result);
    }
}
```

### 解答

```java
public static void main(String[] args) {
    invokeCalc(120, 130, (int a, int b) -> {
      	return a + b;
    });
}
```

> 备注：小括号代表`Calculator`接口`calc`抽象方法的参数，大括号代表`calc`的方法体。



## 3.9 练习：使用Lambda省略格式使用自定义的接口

### 题目

仍然使用前文含有唯一`makeFood`抽象方法的厨子`Cook`接口，在下面的代码中，请使用Lambda的**省略格式**调用`invokeCook`方法，打印输出“吃饭啦！”字样：

```java
public class Demo09InvokeCook {
    public static void main(String[] args) {
        // TODO 请在此使用Lambda【省略格式】调用invokeCook方法
    }

    private static void invokeCook(Cook cook) {
        cook.makeFood();
    }
}
```

### 解答

```java
public static void main(String[] args) {
  	invokeCook(() -> System.out.println("吃饭啦！"));
}
```

## 3.10 练习：使用Lambda省略格式使用自定义的接口

### 题目

给定一个计算器`Calculator`接口，内含抽象方法`calc`可以将两个int数字相加得到和值：

```java
public interface Calculator {
    int calc(int a, int b);
}
```

在下面的代码中，请使用Lambda的**标准格式**调用`invokeCalc`方法，完成120和130的相加计算：

```java
public class Demo08InvokeCalc {
    public static void main(String[] args) {
        // TODO 请在此使用Lambda【标准格式】调用invokeCalc方法来计算120+130的结果ß
    }

    private static void invokeCalc(int a, int b, Calculator calculator) {
        int result = calculator.calc(a, b);
        System.out.println("结果是：" + result);
    }
}
```

### 解答

```java
public static void main(String[] args) {
    invokeCalc(120, 130, ( a,  b) -> 
      	 a + b
    );
}
```

> 备注：小括号代表`Calculator`接口`calc`抽象方法的参数，大括号代表`calc`的方法体。

## 3.11 扩展---Lambda的几种使用形式

- .使用变量的形式：

          ```java
     //**使用子类：**
         class MyThread implements Runnable{
            public void run(){
        
             }
        }
        
        Runnable run = new MyThread();
        Thread t = new Thread(run);
        
    //2.使用匿名内部类
    	Runnable run = new Runnable(){
    		public void run(){
    		}
    	};
    	Thread t = new Thread(run);
    	t.start();
    	
    //3.使用Lambda
    Runnable run = () -> {System.out.println("线程启动…");};


          ```

- 在调用方法时，作为“实参”

                  ```java
 new Thread(()->{System.out.println(“线程启动…”);}).start();
                  ```

- 作为方法的“返回值”：

        ```java
	    public Runnable get(){
        
            //return new 子类**();// 以前
            return ()-> { System.out.println("线程启动…");};  // 返回Lambda表达式
        
         }
        
        ```

