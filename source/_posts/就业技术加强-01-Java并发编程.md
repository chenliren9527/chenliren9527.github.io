---
title: 就业技术加强(01)-Java并发编程
tags:
  - 笔记
  - Java并发编程
  - 就业技术加强
  - JMM内存模型
  - 并发编程三大特性
  - 锁
  - 并发工具原子类
  - 倒计数锁
  - 同步屏障
  - 信号量
  - 线程间交换数据
  - 常见面试题总结
  - 多线程
  - jcstress
  - 高并发测试
categories:
  - 就业技术加强
abbrlink: 7219
date: 2021-01-22 19:18:37
---

## 课程目标

目标1: 了解JMM内存模型

目标2: 理解并发编程三大特性

目标3: 理解Java中的锁

目标4: 了解并发工具原子类

目标5: 理解倒计数锁、同步屏障、信号量、线程间交换数据





## 01、JUC并发编程概述

J.U.C并发包，即`java.util.concurrent`包，是JDK的核心工具包，是JDK1.5之后，由Doug Lea实现并引入。 

整个`java.util.concurrent`包，按照功能可以大致划分如下：

- juc-atomic 原子类框架
- juc-locks 锁框架
- juc-sync 同步器框架、工具类
- juc-collections 集合框架

 课程J.U.C，分析所有基于的源码为Oracle JDK1.8





## 02、多线程基础：进程、线程

**多线程概念介绍:**

- **进程**：我们把运行中的程序叫做进程(概念)。每个进程都会占用内存与CPU资源（动态性）。进程与进程之间各自占用各自的内存资源，互相独立（独立性）。

  ![image-20210103200404401](就业技术加强-01-Java并发编程.assets/image-20210103200404401.png)

- **线程**：线程就是进程中的一个执行单元，负责当前进程中程序的执行。一个进程可以包含多个线程。一个进程包含了多个线程就是多线程。

  > 线程简述: 线程是进程的执行单元，用来执行代码。

![image-20210103200928419](就业技术加强-01-Java并发编程.assets/image-20210103200928419.png)



**为什么使用多线程**？

多线程有什么用？这里举例说明:

> 比如使用Excle导出大量数据，这个时候占用的是tomcat的线程，并且占用一个线程很长时间，tomcat只有200个。会导致别的用户无法访问，这时候需要开启新的线程来进行导出任务
>
> 例如图片上传、大批量视频上传转码加水印等等

比如看学习视频时候： 我们在看视频的同时，还可以听到声音，还可以看到广告以及弹幕，这里至少用到四个线程，当其中一个线程卡死如放不了弹幕不影响播放广告。

![image-20210103202214065](就业技术加强-01-Java并发编程.assets/image-20210103202214065.png)

&nbsp;![1566613916316](就业技术加强-01-Java并发编程.assets/1566613916316.png) 

如上图，要达到并行执行的效果，这里就要用到多线程。



> 并行: 两个或两个以上的事件在同一时刻发生(同时发生)
>
> 并发: 两个或两个以上的事件在一个时间段内发生(交替执行)

多线程可以提高资源的利用率，让程序可以并发处理多个任务。

![image-20210103200737593](就业技术加强-01-Java并发编程.assets/image-20210103200737593.png)



**线程调度:**

在单核CPU中，同一个时刻只有一个线程执行，根据CPU时间片算法依次为每个线程服务，这就叫线程调度。JVM采用的是**抢占式调度**，没有采用分时调度，因此可能造成多线程执行结果的的随机性。

![image-20210103200848772](就业技术加强-01-Java并发编程.assets/image-20210103200848772.png)



### 回顾多线程的创建





## 03、多线程编程：wait()、notify()、notifyAll()

> 目标: 线程等待和唤醒使用。

### 介绍

- 等待和唤醒：通常是两个线程之间的事情，一个线程等待，另外一个线程负责唤醒
- 等待和唤醒
  - wait() 等待
  - notity() 唤醒等待的单个线程
  - notityAll() 唤醒等待的全部线程

Object类:

```java
public final void wait();             // 导致当前线程等待
public final native void wait(long timeout) throws InterruptedException;
public final native void notify();    // 唤醒正在等待的单个线程
public final native void notifyAll(); // 唤醒正在等待的全部线程
```

> 注意: **wait和notify必须是在同步代码块中,使用锁对象调用**

+ wait()方法的作用？

  > 使当前线程阻塞，进行等待状态

+ notify()方法的作用？ 

  > 唤醒正在等待的单个线程

+ notifyAll()方法的作用？

  > 唤醒所有等待(对象的)线程，哪一个线程将会第一个处理取决于操作系统的实现。

+ 为什么wait和notify方法放在Object？

  > 因为wait和notify需要使用锁对象来调用,而任何对象都可以作为锁,所以放在Object类中。

+ 详细介绍

  ```txt
  1、wait()、notify()、notifyAll() 方法是Object的本地final方法，子类无法被重写。
  
  2、wait() 使当前线程阻塞，前提是必须先获得锁，一般配合synchronized 关键字使用。
     即，一般在 synchronized 同步代码块里使用 wait()、notify、notifyAll() 方法。
  
  3、由于 wait()、notify()、notifyAll() 方法在 synchronized 代码块执行，说明当前线程一定是获取了锁的。当线程执行wait()方法时候，会释放当前的锁，然后让出CPU，进入等待状态。只有当 notify()/notifyAll() 被执行时候，才会唤醒一个或多个正处于等待状态的线程，然后继续往下执行，直到执行完 synchronized 代码块的代码或是中途遇到wait()，再次释放锁。也就是说，notify()/notifyAll() 的执行只是唤醒沉睡的线程，而不会立即释放锁，锁的释放要看代码块的具体执行情况。所以在编程中，尽量在使用了 notify()/notifyAll() 后立即退出临界区，以唤醒其他线程让其获得锁。
  
  4、notify 和 wait 的顺序不能错，如果A线程先执行notify方法，B线程在执行wait方法，那么B线程是无法被唤醒的。
  
  5、notify 和 notifyAll的区别:
     notify: 只唤醒一个等待（对象的）线程并使该线程开始执行。所以如果有多个线程等待一个
     对象，这个方法只会唤醒其中一个线程，选择哪个线程取决于操作系统对多线程管理的实现。
     
     notifyAll: 会唤醒所有等待(对象的)线程，尽管哪一个线程将会第一个处理取决于操作系统
     的实现。如果当前情况下有多个线程需要被唤醒，推荐使用notifyAll方法。
  ```

  

### 代码

```java
package cn.itcast.thread;

import java.util.concurrent.TimeUnit;

/**
 * 测试 wait()、notify()、notifyAll()
 */
public class Test1 {

    public static void main(String[] args) {
//        testWait();
        testWait_Notify_NotifyAll();
    }

    private static void testWait() {
        // 创建锁对象
        Object obj = new Object();
        // 线程t1
        new Thread(() -> {
            synchronized (obj) {
                try {
                    System.out.println(Thread.currentThread().getName() + "wait 前");
                    obj.wait(); // 等待、线程阻塞(释放锁)
                    System.out.println(Thread.currentThread().getName() + "wait 后");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "t1:").start();
    }

    private static void testWait_Notify_NotifyAll() {
        // 创建锁对象
        Object obj = new Object();
        // 线程t1
        new Thread(() -> {
            synchronized (obj) {
                try {
                    System.out.println(Thread.currentThread().getName() + "wait 前");
                    obj.wait(); // 等待、线程阻塞(释放锁)
                    System.out.println(Thread.currentThread().getName() + "wait 后");
                } catch (InterruptedException e) {}
            }
        }, "t1:").start();

        // 线程t2
        new Thread(() -> {
            synchronized (obj) {
                try {
                    System.out.println(Thread.currentThread().getName() + "wait 前");
                    obj.wait(); // 等待、线程阻塞(释放锁)
                    System.out.println(Thread.currentThread().getName() + "wait 后");
                } catch (InterruptedException e) {}
            }
        }, "t2:").start();

        // 线程t3
        new Thread(() -> {
            synchronized (obj) {
                try {
                    // 休眠2秒
                    TimeUnit.SECONDS.sleep(2);  //目的让前2个线程进入等待状态
                } catch (InterruptedException e) {}
                System.out.println("notifyAll 前");
                obj.notifyAll(); // 唤醒全部等待的线程，不会释放锁
                System.out.println("notifyAll 后");
            }
        }, "t3:").start();
    }
}
```

### 小结

> wait()方法的作用? 导致当前线程等待。
> notify()方法的作用? 唤醒正在等待的单个线程。





## 04、多线程编程：Java线程状态及状态转换

> 目标: 理解线程6种状态以及状态转换。

### 线程状态

- 线程状态。线程可以处于以下状态之一:

  + NEW  **尚未启动**的线程处于此状态。

  + RUNNABLE 在Java虚拟机中**执行的线程**处于此状态。

  + BLOCKED 被**阻塞**等待的线程处于此状态。

  + WAITING **无限等待**另一个线程执行特定动作的线程处于此状态。

  + TIMED_WAITING 一个正在**限时等待**另一个线程执行一个动作的线程处于这一状态。 

  + TERMINATED 已**退出**的线程处于此状态。

    > Thread.State 枚举类中进行了定义。

### 状态转换

![image-20210104094941976](就业技术加强-01-Java并发编程.assets/image-20210104094941976.png)

### 代码

```java
package cn.itcast.thread;

/**
 * 测试: 线程6种状态
 */
public class Test2 {

    // 方法1 (NEW: 新建状态)
    public static void test1() {
        new Thread(() -> {
            synchronized (Test2.class) {
                while (true) {
                }
            }
        }, "t1-runnable");  // 新建状态(VisualVM查看不到)

        while (true) {
        }
    }

    // 方法2 (RUNNABLE: 可运行状态)
    public static void test2() {
        new Thread(() -> {
            synchronized (Test2.class) {
                while (true) {
                }
            }
        }, "t1-runnable").start();  // 线程运行中
    }

    // 方法3 (BLOCKED: 阻塞状态)
    public static void test3() {
        new Thread(() -> {
            synchronized (Test2.class) {
                while (true) {
                }
            }
        }, "t31-runnable").start();

        new Thread(() -> {
            synchronized (Test2.class) {
                //由于上面没有释放锁，被阻塞中
            }
        }, "t32-BLOCKED").start();
    }

    // 方法4 (TIMED_WAITING : 计时等待状态)
    public static void test4() {
        new Thread(() -> {
            synchronized (Test2.class) {
                while (true) {
                    try {
                        Thread.sleep(100000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }, "t4-timed_waiting").start(); // 线程超时等待中
    }

    // 方法5 (WAITING : 无限等待状态)
    public static void test5() {
        new Thread(() -> {
            synchronized (Test2.class) {
                try {
                    Test2.class.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "t5-waiting").start(); // 线程等待中
    }

    // 方法6 (TERMINATED : 终止状态)
    public static void test6() {
        new Thread(() -> {
            int sum = 0;
            for (int i = 0; i < 1000000; i++) {
                sum += i;
            }
            System.out.println("计算完成终止");
        }, "t6-terminated").start(); // 线程等待中

        while (true) {
        }
    }

    public static void main(String[] args) {
        // test1(); // 新建状态
        // test2(); // 可运行状态
        // test3(); // 阻塞状态
        // test4(); // 计时等待中状态
        // test5(); // 无限等待中状态
        // test6(); // 终止状态
    }
}
```



### 查看进程堆栈

使用VisualVM查看

![image-20210104095345566](就业技术加强-01-Java并发编程.assets/image-20210104095345566.png)



![image-20210104095531902](就业技术加强-01-Java并发编程.assets/image-20210104095531902.png)





使用jstack可查看指定进程（pid）的堆栈信息，用以分析线程执行状态:

+ 进入cmd: Win + R

  ```
  jps -l
  ```

  ![1597998092229](就业技术加强-01-Java并发编程.assets/1597998092229.png) 

+ 输入jstack 进程号

  ```
  jstack 14696
  ```

  ![1597998184064](就业技术加强-01-Java并发编程.assets/1597998184064.png) 





## 05、多线程编程：join、yield、sleep区别？

> 目标: 学习如何控制线程执行顺序、线程让步、优先级。

### join 作用

+ join()方法【**加入线程**】，把指定的线程加入到当前线程，可以将两个交替执行的线程合并为顺序执行的线程。线程调用了join方法，那么就要一直运行到该线程结束，才会运行join后的代码。这样可以控制线程执行顺序。
+ join(long millis)
  + 如果为0表示永远等待，其实是等到线程结束后。
  + 传入指定的时间会调用wait(millis)，不再等待。

> ```java
> t1.join();//让主线程等待t1执行完毕，同时等t1完全执行完才会执行t2.join();
> t2.join();让主线程等待t2执行完毕
> ```

### yield 作用

+ thread.yield()【**线程让步**】 让出CPU的时间片尽量切换到其它线程去执行。

+ 使正在运行中的线程重新变成就绪状态，并重新竞争 CPU 的调度权。它可能会获取到，也有可能被其它线程获取到。

  

### yield 和 sleep 的异同

+ 状态: sleep当前线程由运行态进入计时等待状态；yield当前线程还是处于可运行状态。
+ 异常: sleep方法在声明时抛出InterruptedException异常，所以在使用时要么try捕获要么throws抛出，而yield没有声明异常。



### 线程优先级

线程的优先级说明该线程在程序中的重要性。系统会根据优先级决定首先使用哪个线程，但这并不意味着优先级低的线程得不到运行，只是它运行的机率小一点而已，比如垃圾回收机制。

优先级范围1-10，默认为:5  比如，设置最高优先级为10:

t1.setPriority(Thread.MAX_PRIORITY);

### 代码

```java
package cn.itcast.thread;

/**
 * @author zhangping
 * @version 1.0
 * <p>File Created at 2020-04-16<p>
    join() : 加入线程
    yield() : 线程让步
    sleep() : 线程休眠
 */
public class Test3 {

    public static void main(String[] args) throws InterruptedException {
         testJoin();
//        testYield();
    }

    private static void testJoin() throws InterruptedException {
        // 线程1
        Thread t1 = new Thread(() -> {
            for (int i = 1; i <= 20; i++) {
                try {
                    Thread.sleep(300);
                    System.out.println(Thread.currentThread().getName() + i);
                } catch (Exception e) {}
            }
        },"t1:");

        // 启动线程
        t1.start();

        Thread t2 = new Thread(() -> {
            for (int i = 1; i <= 20; i++) {
                try {
                    Thread.sleep(300);
                    System.out.println(Thread.currentThread().getName() + i);
                } catch (Exception e) {}
            }
        },"t2:");

        // 启动线程
        t2.start();

        // 等前面的线程走完,才往下面走 join的作用,等调用者走完,才往后面执行

        t1.join();//让主线程等待t1执行完毕
        t2.join();//让主线程等待t2完毕
        //join的作用就是主线程等待子线程执行完毕了，把活干完了，主线程才能拿着子线程执行完后的完整数据去继续执行（干活）

        // 主线程执行20次循环
        for (int i = 1; i <= 20; i++) {
            Thread.sleep(300);
            System.out.println(Thread.currentThread().getName() + i);
        }
    }

    private static void testYield() throws InterruptedException {

        Runnable runnable = () -> {
            String name = Thread.currentThread().getName();
            for (int i = 1; i <= 20; i++) {
                System.out.println(name + i);
                // Thread.yield(); // 让步,告诉CPU我这个线程不太想执行啦,尽量切换到其他线程,此时线程还是RUNNABLE
                // Thread.yield();

                // try {
                //     Thread.sleep(1); // 进入计时等待状态,线程的状态会切换
                // } catch (InterruptedException e) {
                //     e.printStackTrace();
                // }
            }
        };

        // 线程1
        Thread t1 = new Thread(runnable, "t1:");
        // 设置线程的优先级 最低1, 最高10, 默认是5
        t1.setPriority(1);
        // 线程2
        Thread t2 = new Thread(runnable, "   t2:");
        t2.setPriority(10);

        t1.start();
        t2.start();
    }
}
```

### 小结

+ join作用？

  ```txt
  加入线程，线程调用了join方法，那么就要一直运行到该线程结束，才会运行join后的代码. 这样可以控制线程执行顺序。
  ```

+ yield作用？

  ```txt
  线程让步，让出CPU的时间片尽量切换其他线程去执行。
  ```

+ 线程优先级？

  ```txt
  设置线程优先级，优先级可以设置1-10，数字越大代表优先级越高。
  
  在java语言中，每个线程都有一个优先级，线程的优先级越高越有可能先被优先选择执行。
  ```





## 06、并发编程需要处理的问题：死锁问题

并发编程的目的是为了让程序运行得更快，但是，并不是启动更多的线程就能让程序最大限度地并发执行。在进行并发编程时，如果希望通过多线程执行任务让程序运行得更快，会面临非常多的挑战，比如死锁的问题、上下文切换的问题。

### 描述

锁是个非常有用的工具，运用场景非常多，因为它使用起来非常简单，而且易于理解。但同时它也会带来一些困扰，那就是可能会引起死锁，一旦产生死锁，就会造成系统功能不可用。让我们先来看一段代码，这段代码会引起死锁，使线程t1和线程t2互相等待对方释放锁。

### 演示

什么是死锁？**多线程竞争共享资源,导致线程相互等待,程序无法向下执行**



![image-20210104104901396](就业技术加强-01-Java并发编程.assets/image-20210104104901396.png)



![image-20210104104912765](就业技术加强-01-Java并发编程.assets/image-20210104104912765.png)

![image-20210104103650346](就业技术加强-01-Java并发编程.assets/image-20210104103650346.png)

执行sleep的时候会让出CPU，但不是释放锁。 

```java
package cn.itcast.thread;

/** 学习死锁的概念和解决死锁 */
public class DeadLock {
    // 定义两个对象作为锁
    private static Object objA = new Object();
    private static Object objB = new Object();

    public static void main(String[] args) {
        // 线程1
        Thread t1 = new Thread(() -> {
            synchronized (objA){
                System.out.println(Thread.currentThread().getName() + ": 获取objA锁");

                synchronized (objB){
                    System.out.println(Thread.currentThread().getName() + ": 获取objB锁");
                }
            }
        });

        // 线程2
        Thread t2 = new Thread(() -> {
            // 同步锁
            synchronized (objB){
                System.out.println(Thread.currentThread().getName() + ": 获取objB锁");
                // 同步锁
                synchronized (objA){
                    System.out.println(Thread.currentThread().getName() + ": 获取objA锁");
                }
            }
        });

        t1.start();
        t2.start();
    }
}
```

上面的代码只是演示死锁的场景，在现实中你可能不会写出这样的代码。但是，在一些更为复杂的场景中，你可能会遇到这样的问题，比如t1拿到锁之后，因为一些异常情况没有释放锁（死循环）。又或者是t1拿到一个数据库锁，释放锁的时候抛出了异常，没释放掉。一旦出现死锁，业务是可感知的，因为不能继续提供服务了。

### 查看线程执行情况

打开cmd命令dos窗口输入如下命令

> jps命令: 查看java程序进程id信息
>
> jstack命令: 查看指定进程堆栈信息

![1598003847797](就业技术加强-01-Java并发编程.assets/1598003847797.png) 

![1598003964673](就业技术加强-01-Java并发编程.assets/1598003964673.png) 

### 如何避免死锁？

现在我们介绍避免死锁的几个常见方法:

+ 避免一个线程同时获取多个锁。
+ 避免一个线程在锁内同时占用多个资源，尽量保证每个锁只占用一个资源。
+ 尝试使用定时锁，使用lock.tryLock(timeout)来替代使用内部锁机制。

### 小结

```java
1. 什么是死锁: 多线程竞争共享资源,导致线程相互等待,程序无法向下执行。

2. 死锁产生的条件:
   A.有多个线程
   B.有多把锁
   C.有同步代码块嵌套

3. 如何避免死锁: 干掉其死锁产生的条件中一个条件即可。
```





## 07、并发编程需要处理的问题：上下文过度切换

### 多线程一定快吗？

![image-20200531092523975](就业技术加强-01-Java并发编程.assets/image-20200531092523975.png) 



**测试代码**

代码演示串行和并发执行并累加操作的时间，请分析: 下面的代码并发执行一定比串行执行快吗？

```java
package cn.itcast.thread;

public class Test4 {
    // 定义变量
    private static final long count = 1000000000;

    public static void main(String[] args) throws InterruptedException {
        concurrency();
        serial();
    }

    // 定义方法1(使用线程)
    private static void concurrency() throws InterruptedException {
        long start = System.currentTimeMillis();
        // 创建线程 循环累加
        Thread thread = new Thread(() -> {
            int a = 0;
            for (long i = 0; i < count; i++) {
                a += i;
            }
        });
        // 开启线程
        thread.start();

        // 循环累减
        int b = 0;
        for (long i = 0; i < count; i++) {
            b--;
        }
        long time = System.currentTimeMillis() - start;

        System.out.println("concurrency :" + time + "ms,b=" + b);
    }


    // 定义方法2 (不用线程)
    private static void serial() {
        long start = System.currentTimeMillis();
        // 循环累加
        int a = 0;
        for (long i = 0; i < count; i++) {
            a += 5;
        }
        // 循环累减
        int b = 0;
        for (long i = 0; i < count; i++) {
            b--;
        }
        long time = System.currentTimeMillis() - start;
        System.out.println("serial:" + time + "ms,b=" + b );
    }
}
```



**测试结果**

上述问题的答案是“不一定”，测试结果如表所示:

![1571742917351](就业技术加强-01-Java并发编程.assets/1571742917351.png)

当并发执行累加操作不超过百万次时，速度会比串行执行累加操作要慢。那么，为什么并发执行的速度会比串行慢呢？这是因为线程有创建和上下文切换的开销。



### 上下文切换

- 即使是单核处理器也支持多线程执行代码，CPU 通过给每个线程分配CPU 时间片来实现这个机制。时间片是CPU分配给各个线程的时间，因为时间片非常短，所以CPU 通过不停地切换线程执行，让我们感觉多个线程是同时执行的，时间片一般是几十毫秒（ms）。

- CPU 通过时间片分配算法来循环执行任务，当前任务执行一个时间片后会切换到下一个任务。但是，在切换前会保存上一个任务的状态，以便下次切换回这个任务时，可以再加载这个任务的状态。所以任务从保存到再加载的过程就是一次上下文切换。

  例如: 这就像我们同时读两本书，当我们在读一本英文的技术书时，发现某个单词不认识，于是便打开中英文字典，但是在放下英文技术书之前，大脑必须先记住这本书读到了多少页的第多少行，等查完单词之后，能够继续读这本书。这样的切换是会影响读书效率的，同样上下文切换也会影响多线程的执行速度。





## 08、Java内存模型（Java Memory Model）

> **什么是JMM内存模型?**

Java内存模型(即Java Memory Model，简称JMM)。Java内存模型跟CPU缓存模型类似，是基于CPU缓存模型来建立的，Java内存模型是标准化的，屏蔽了底层不同计算机的区别。



JMM本身是一种抽象的概念，并不真实存在，它描述的是一组规则或规范，通过这组规范定义了程序中各个变量（包括实例字段，静态字段和构成数组对象的元素）的访问方式。由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存(有些地方称为栈空间)，用于存储线程私有的数据。而Java内存模型中规定所有变量都存储在主内存，主内存是共享内存区域，所有线程都可以访问。



线程对变量的操作(读取赋值等)必须在工作内存中进行，首先要将变量从主内存拷贝的自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存，不能直接操作主内存中的变量，工作内存中存储着主内存中的变量副本拷贝，前面说过，工作内存是每个线程的私有数据区域，因此不同的线程间无法访问对方的工作内存，线程间的通信(传值)必须通过主内存来完成，其简要访问过程如下图：

![image-20210111184059792](就业技术加强-01-Java并发编程.assets/image-20210111184059792.png)

关于JMM中的主内存和工作内存说明如下:

- 主内存(Java堆与方法区)

  主要存储的是Java实例对象，所有线程创建的实例对象都存放在主内存中，不管该**实例对象是成员变量还是方法中的本地变量(也称局部变量)**，当然也包括了共享的类信息、常量、静态变量。由于是共享数据区域，多条线程对同一个变量进行访问可能会发现线程安全问题。

- 工作内存(Java虚拟机栈)

  主要存储当前方法的所有本地变量信息(工作内存中存储着主内存中的变量副本拷贝)，每个线程只能访问自己的工作内存，即线程中的本地变量对其它线程是不可见的，就算是两个线程执行的是同一段代码，它们也会各自在自己的工作内存中创建属于当前线程的本地变量，当然也包括了字节码行号指示器、相关Native方法的信息。注意由于工作内存是每个线程的私有数据，线程间无法相互访问工作内存，因此存储在工作内存的数据不存在线程安全问题。





## 09、Java并发编程三大特性

Java并发编程三个特性: 可见性、原子性、有序性。

- 可见性: 一个线程修改了某个共享变量，其状态能够立即被其他线程知晓，通常被解释为将线程本地状态反映到主内存上，volatile就是负责保证可见性的。

- 原子性: 相关操作不会中途被其他线程干扰，一般通过同步机制实现。

- 有序性: 保证线程内串行语义，避免指令重排。

  

### 9.1 可见性

可见性表示的是，如果有线程更新了某一个共享变量的值，则其它线程要能够立即感知到最新的内容。如果不能保证可见性，则可能出现类似于数据库中的脏读情况。

前文介绍JMM的时候也提到了，如果要保证可见性，那么变量被一个线程修改后，需要将其修改后的最新值同步回主存，然后其它线程要读取该变量时，需要从主存刷新最新的值到本地内存，就这样通过主存实现可见性。但是将最新值同步回主存的时机是没有强制要求的，也不知道其它线程什么时候可能会去从主存刷新最新值，所以**普通变量在多线程操作时是保证不了可见性的**。

这时有一个比较好使的关键字: volatile。JMM对它定义了一些特殊的访问规则，它能保证修改后的最新值能立即同步到主存，同时，每次使用都从主存刷新。所以**volatile能够保证多线程场景下的可见性**。



### volatile 介绍

在多线程并发编程中synchronized和volatile都扮演着重要的角色，volatile是轻量级的synchronized，它在多处理器开发中保证了共享变量的“可见性”。可见性的意思是当一个线程修改一个共享变量时，另外一个线程能读到这个修改的值。如果volatile变量修饰符使用恰当的话，它比synchronized的使用和执行成本更低，因为它不会引起线程上下文的切换和调度。



### volatile 使用

```java
package cn.itcast.thread;

/**
 * volatile 实现多线程访问共享成员时的可见性
 */
public class Test6 {
    // volatile 实现多线程访问共享成员时的可见性
    private static volatile boolean flag = true;

    public static void main(String[] args) throws InterruptedException {
        // 创建线程1
        new Thread(() -> {
            long num = 0;
            while (flag) {
                num++;
            }
            // 如果没有打印，说明当前线程无法感知flag的修改
            System.out.println("循环结束 num = " + num);
        }).start();

        // 休眠1000毫秒
        Thread.sleep(1000);

        // 创建线程2
        new Thread(() -> {
            // 修改flag
            flag = false;
            System.out.println("修改 flag = " + flag);
        }).start();
    }
}
```





### 9.2 原子性

在计算机中，它**表示的是一个操作，可能包含一个或多个步骤，这些步骤要么全部执行成功要么全部执行失败，**并且执行的过程中不能被其它操作打断，原子性是不可拆分的最小单位，这类似于数据库中事务的原子性概念。

比如：**i = i + 1，就是一个非原子操作**，它涉及到获取i，相加，赋值等3个操作，所以在多线程情况下可能会出现并发问题。

**比如 i = i+1，我们要保证它的原子性 **该怎么做呢？**synchronized关键字保证原子性**。



**代码:** 

```java
package cn.itcast.thread;

/*
    多线程操作的原子性问题
 */
public class Test7 {
    private static int count = 0;

    public static void main(String[] args) throws InterruptedException {
        // 循环创建线程
        Runnable runnable = () -> {
            long start = System.currentTimeMillis();
            for (int i = 0; i < 100000; i++) {
                synchronized (Test7.class) {
                    count++;
                    // 读取变量到工作内存
                    // 对count+1
                    // 同步回主内存
                }
            }
            long end = System.currentTimeMillis();
            System.out.println("运行结果：" + count + ", 耗时: " + (end - start));
        };

        new Thread(runnable).start(); // t1
        new Thread(runnable).start(); // t2
    }
}
```



### 9.3 有序性

### 为什么要重排序

为了提高程序的执行效率，编译器和CPU会对程序中代码进行重排序。

![image-20210110221701174](就业技术加强-01-Java并发编程.assets/image-20210110221701174.png)



```java
int a = 10;
int b = 20;

最终执行:
int a = 10;
int b = 20;

int b = 20;
int a = 10;
```



### as-if-serial语义

as-if-serial语义的意思是：不管编译器和CPU如何重排序，必须保证在单线程情况下程序的结果是正确的。

以下数据有依赖关系，不能重排序。

写后读：

```java
int a = 1;
int b = a;
```

写后写：

```java
int a = 1;
int a = 2;
```

读后写：

```java
int a = 1;
int b = a;
int a = 2;
```

编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果。但是，如果操作之间不存在数据依赖关系，这些操作就可能被编译器和处理器重排序。

Java内存模型中，允许编辑器和处理器对指令进行重排序，但是**重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性**。指令重排的后果在并发的情况下有时会是严重的。



### 有序性演示

jcstress是java并发压测工具。https://blog.csdn.net/qq_35371031/article/details/100824886

修改pom文件，添加依赖：

```xml
<dependency>
    <groupId>org.openjdk.jcstress</groupId>
    <artifactId>jcstress-core</artifactId>
    <version>0.5</version>
</dependency>

<build>
    <!--配置jcstress编译插件-->
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>2.2</version>
        <executions>
            <execution>
                <id>main</id>
                <phase>package</phase>
                <goals>
                    <goal>shade</goal>
                </goals>
                <configuration>
                    <finalName>${uberjar.name}</finalName>
                    <transformers>
                        <transformer
                                     implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                            <mainClass>org.openjdk.jcstress.Main</mainClass>
                        </transformer>
                        <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                            <resource>META-INF/TestList</resource>
                        </transformer>
                    </transformers>
                </configuration>
            </execution>
        </executions>
    </plugin>
    </plugins>
</build>
```

代码

Test8.java

```java
package cn.itcast.thread;

import org.openjdk.jcstress.annotations.*;
import org.openjdk.jcstress.infra.results.I_Result;

// 标注需要测试的类
@State
// 指定使用并发测试
@JCStressTest
// // 预测的结果与类型，附加描述信息
@Outcome(id = {"1", "4"}, expect = Expect.ACCEPTABLE, desc = "ok")
@Outcome(id = "0", expect = Expect.ACCEPTABLE_INTERESTING, desc = "danger")
public class Test8 {
    int num = 0;
    boolean ready = false;
    // 线程1执行的代码
    @Actor
    public void actor1(I_Result r) {
        if(ready) {
            r.r1 = num + num;
        } else {
            r.r1 = 1;
        }
    }

    // 线程2执行的代码
    @Actor
    public void actor2(I_Result r) {
        num = 2;
        ready = true;
    }
}
```

`I_Result`是一个对象，有一个属性`r1`用来保存结果，在多线程情况下可能出现几种结果？
情况1：线程1先执行actor1，这时ready = false，所以进入else分支结果为1。

情况2：线程2执行到actor2，执行了num = 2;和ready = true，线程1执行，这回进入 if 分支，结果为4。

情况3：线程2先执行actor2，只执行num = 2；但没来得及执行 ready = true，线程1执行，还是进入else分支，结果为1。

<font color='red'>**还有一种结果0。**</font>线程2先执行 ready = true，线程1执行，进入else分支，结果为0。



运行测试：

```java
mvn clean package
java -jar target/jcstress.jar
```



**小结**

程序代码在执行过程中的先后顺序，由于Java在编译期以及运行期的优化，导致了代码的执行顺序未必就是开发者编写代码时的顺序。



## 10、Java中的锁：锁的种类

在面试过程时，经常会被问到各种各样的锁，如乐观锁、读写锁等等，非常繁多，在此做一个总结。介绍的内容如下:

- 乐观锁/悲观锁
- 独享锁/共享锁
- 互斥锁/读写锁
- 可重入锁
- 公平锁/非公平锁
- 分段锁
- 偏向锁/轻量级锁/重量级锁Synchronized
- 自旋锁

以上这些锁并不是我们开发中的某个具体的对象或类，而是指锁的特性，是一些概念。



### 乐观锁/悲观锁

乐观锁: 顾名思义，就是很乐观，每次去读数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量，在Java中java.util.concurrent.atomic包下面的原子变量类就是使用了乐观锁的一种实现方式CAS(Compare and Swap 比较并交换)实现的。

悲观锁: 总是假设最坏的情况，每次去读数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。比如Java里面的synchronized关键字的实现就是悲观锁。

悲观锁适合写操作非常多的场景，乐观锁适合读操作非常多的场景，不加锁会带来大量的性能提升。

乐观锁在Java中的使用，是无锁编程，常常采用的是CAS算法，典型的例子就是原子类，通过CAS自旋实现原子操作的更新。



### 独享锁/共享锁

独享锁是指该锁一次只能被一个线程所持有。

共享锁是指该锁可被多个线程所持有。

对于Java ReentrantLock而言，其是独享锁。但是对于Lock的另一个实现类ReadWriteLock，其读锁是共享锁，其写锁是独享锁。

读锁的共享锁可保证并发读是非常高效的，写的过程是互斥的。

对于synchronized而言，是独享锁。



### 互斥锁/读写锁

独享锁/共享锁就是一种广义的说法，互斥锁/读写锁就是具体的实现。

互斥锁在Java中的具体实现就是ReentrantLock。

读写锁在Java中的具体实现就是ReadWriteLock。



### 可重入锁

可重入锁又名递归锁，是指在同一个线程在已经获得某个锁后，可以再次获取该锁。说的有点抽象，下面会有一个代码的示例。

```java
Thread t1 = new Thread(() -> {
    synchronized (objA) {
        System.out.println(Thread.currentThread().getName() + ": 获取objA锁");

        synchronized (objA) {
            System.out.println(Thread.currentThread().getName() + ": 获取objA锁");
        }
    }
});
```

上面的代码就是一个可重入锁的一个特点。如果是不可重入锁的话，执行里面的synchronized会造成死锁。

对于Java ReetrantLock而言，从名字就可以看出是一个重入锁，其名字是Reentrant Lock重新进入锁。

对于synchronized而言，也是一个可重入锁。可重入锁的一个好处是可一定程度避免死锁。



### 公平锁/非公平锁

公平锁是指多个线程按照申请锁的顺序来获取锁。

![image-20210104165814012](就业技术加强-01-Java并发编程.assets/image-20210104165814012.png)

非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。

对于Java ReetrantLock而言，通过构造函数指定该锁是否是公平锁，默认是非公平锁。非公平锁的优点在于吞吐量比公平锁大。

对于synchronized而言，也是一种非公平锁。



### 分段锁

　　分段锁其实是一种锁的设计，并不是具体的一种锁，对于ConcurrentHashMap而言，其并发的实现就是通过分段锁的形式来实现高效的并发操作。

![image-20210104170302463](就业技术加强-01-Java并发编程.assets/image-20210104170302463.png)



![image-20210104170320324](就业技术加强-01-Java并发编程.assets/image-20210104170320324.png)

　　我们以ConcurrentHashMap来说一下分段锁的含义以及设计思想，ConcurrentHashMap中的分段锁称为Segment，它即类似于HashMap（JDK7和JDK8中HashMap的实现）的结构，即内部拥有一个Entry数组，数组中的每个元素又是一个链表；同时又是一个ReentrantLock（Segment继承了ReentrantLock）。

　　分段锁的设计目的是细化锁的粒度，当操作不需要更新整个数组的时候，就仅仅针对数组中的一项进行加锁操作。



### 自旋锁

　　在Java中，自旋锁是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU。



### 偏向锁/轻量级锁/重量级锁    synchronized

　　**偏向锁** 是JDK 6中的重要引进，因为HotSpot作者经过研究实践发现，在大多数情况下，锁是由同一线程多次获得，为了让线程获得锁的代价更低，引进了偏向锁。偏向锁是在锁对象中做了一个标记，记录获取这个锁的线程。降低获取锁的代价。

　　**轻量级锁**是指当锁是偏向锁的时候，被另一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，提高性能。

　　**重量级锁**是指当锁为轻量级锁的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁。重量级锁会让他申请的线程进入阻塞，性能降低。





## 11、Java中的锁：synchronized

synchronized是并发编程中接触的最基本的同步关键字，是一种重量级锁，也是java内置的同步机制，首先我们知道synchronized提供了互斥性和可见性，那么我们可以通过使用它来保证并发的安全。

synchronized三种用法:

- **同步代码块**

  - 当使用synchronized修饰代码块时，那么当前加锁的级别就是synchronized（X）中配置的x对象实例，当多个线程并发访问该对象的同步方法、同步代码块以及当前的代码块时，会进行同步。

  - 使用同步代码块时要注意的是不要使用String类型对象，因为String常量池的存在，所以很容易导致出问题。

    ```java
    Object obj = new Object();
    synchronized (obj) {
    	// 代码
    }
    
    synchronized (类名.class) {
        // 代码
    }
    ```

    

    

- **对象锁**

  - 当使用synchronized修饰类普通方法时，那么当前加锁的级别就是实例对象，当多个线程并发访问该对象的同步方法、同步代码块时，会进行同步。

    ```java
    private synchronized void 方法名() {
    
    }
    ```

- **类锁**

  - 当使用synchronized修饰类静态方法时，那么当前加锁的级别就是类，当多个线程并发访问该类（所有实例对象）的同步方法以及同步代码块时，会进行同步。

    ```java
    private static synchronized void 方法名() {
    
    }
    ```

    

### synchronized 同步代码块

> synchronized 同步代码块解决线程安全问题
>
> 什么是线程安全问题: 多线程操作共享数据，导致共享数据出现错乱

出现线程安全问题的条件:

1. 有多个线程
2. 有共享数据
3. 有多句代码

**代码**

```java
package cn.itcast.thread;

/**
 * 学习使用同步代码块解决线程安全问题
 */
public class Test8 {
    // 定义票的总数量
    private static int ticket = 100;

    public static void main(String[] args) {

        Runnable runnable = () -> {
            // 循环买票
            while (true) {
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 同步代码块(类锁)
                synchronized (Test8.class) {
                    if (ticket > 0) {
                        ticket--;
                        System.out.println(Thread.currentThread().getName() + "卖了一张票，剩余：" + ticket);
                    } else {
                        // 票没了
                        break;
                    }
                }
            }
        };

        // 创建3个线程
        Thread t1 = new Thread(runnable, "窗口1");
        Thread t2 = new Thread(runnable, "窗口2");
        Thread t3 = new Thread(runnable, "窗口3");
        t1.start();
        t2.start();
        t3.start();
    }
}
```

**小结**

+ 哪些对象可以作为锁？

  ```txt
  任意对象。保证是同一个对象
  ```

+ synchronized锁对象时候要注意什么？

  ```txt
  多线程并发方法同步代码块，需要锁同一个对象。
  ```

+ synchronized中的锁的作用是什么？

  ```txt
  同步代码块中有锁的线程进入，无锁的线程需要等待。
  ```

  



### synchronized 同步方法

> synchronized 同步方法解决线程安全问题

**语法**

```java
// 普通同步方法，对当前一个实例对象加锁，多线程操作同一个对象实例进行同步操作
public synchronized void 方法名() {
    ...
}
// 静态同步方法，对类加锁，多线程操作当前类所有实例对象进行同步操作
public static synchronized void 方法名() {
    ...
}
```

**代码**

```java
package cn.itcast.thread;

/**
 * 学习使用同步代码块解决线程安全问题
 */
public class Test9 {
    // 定义票的总数量
    private static int ticket = 100;

    public static void main(String[] args) {
        Runnable runnable = () -> {
            // 循环买票
            while (true) {
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                sell();
            }
        };

        // 创建3个线程
        Thread t1 = new Thread(runnable, "窗口1");
        Thread t2 = new Thread(runnable, "窗口2");
        Thread t3 = new Thread(runnable, "窗口3");
        t1.start();
        t2.start();
        t3.start();
    }

    // 同步方法
    private static synchronized void sell() {
        // 同步代码块(类锁)
        if (ticket > 0) {
            ticket--;
            System.out.println(Thread.currentThread().getName() + "卖了一张票，剩余：" + ticket);
        }
    }
}
```

**小结**

synchronized是通过对象内部的一个叫==监视器锁==来实现的，但是监视器锁本质又是依赖底层的操作系统的Mutex（互斥） Lock来实现的，而操系统实现线程之间的切换回造成带量得CPU资源浪费，这个成本非常的高，状态之间的转换需要相对比较长的时间，这就是为什么Synchronized效率低的原因，因此这种依赖于操作系统Mutex Lock所实现的锁我们称之为：==重量级锁==。JDK中对Synchronized做的种种优化，其核心都是为了减少这种重量级锁的使用，JDK1.5以后，为来减少获得锁和释放锁所带来的性能消耗，jdk引入了：”轻量级锁“和”偏向锁“进行优化，这个优化自动的无需开发人员介入。

> synchronized 属于最基本的线程通信机制，基于对象监视器实现的。Java中的每个对象都与一个监视器相关联，一个线程可以锁定或解锁。一次只有一个线程可以锁定监视器。试图锁定该监视器的任何其他线程都会被阻塞，直到它们可以获得该监视器上的锁定为止。



## 12、Java中的锁：ReentrantLock

> 目标: 学习使用ReentrantLock可重入锁解决线程安全问题

**介绍**

+ synchronized是内部锁，自动化的上锁与释放锁，而lock是手动的，需要人为的上锁和释放锁。lock比较灵活，但是代码相对较多。

+ lock接口异常的时候不会自动的释放锁，同样需要手动的释放锁，所以一般写在finally语句块中，而synchronized则会在异常的时候自动的释放锁。

  

**API方法**

- lock()

  用来获取锁，如果锁被其他线程获取，处于等待状态。如果采用Lock，必须主动去释放锁，并且在发生异常的时候，不会自动释放锁。因此一般来说，使用Lock必须早try{}catch{}块中进行，并且将释放锁的操作放在finally块中进行，以保证锁一定被释放，防止死锁发生。

- lockInterruptibly()

  通过这个这个方法去获取锁时，如果线程正在等待获取锁，则这个线程能够响应中断，即中断线程的等待状态。

- tryLock()

  tryLock方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true，如果获取失败(即锁已经别其他线程获取)，则返回false，也就是说这个方法无论如何都会立即返回。在获取不到锁的时候，不会再那一直等待。

- tryLock(long time, TimeUnit unit)

  与tryLock类似，只不过是有等待时间，在等待时间内获取到锁返回true，超时返回false。

- unlock()

  释放锁，一定要在finally块中释放。

  

**Lock锁使用语法**

```java
Lock介绍:
    比synchronized更灵活,可以自己调用方法
    void lock() 获得锁
    void unlock() 释放锁

Lock实现类:
    ReentrantLock

Lock使用标准方式
    l.lock(); // 获得锁
    try {
        操作共享资源的代码
    } finally {
        l.unlock(); // 释放锁
    }
```



**代码**

```java
package cn.itcast.thread;

import java.util.concurrent.locks.ReentrantLock;

/**
 * 学习使用Lock解决线程安全问题
 */
public class Test10 {
    // 定义票的总数量
    private static int ticket = 100;

    public static void main(String[] args) {
        // 创建可重入锁对象
        ReentrantLock lock = new ReentrantLock();

        Runnable runnable = () -> {
            // 循环卖票
            while (true) {
                try {
                    Thread.sleep(10);
                    // 获得锁
                    lock.lock();
                    if (ticket > 0) {
                        ticket--;
                        System.out.println(Thread.currentThread().getName() + "卖了一张票，剩余：" + ticket);
                    } else {
                        break;
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    // 释放锁
                    lock.unlock();
                }
            }
        };

        // 创建3个线程
        Thread t1 = new Thread(runnable, "窗口1");
        Thread t2 = new Thread(runnable, "窗口2");
        Thread t3 = new Thread(runnable, "窗口3");
        t1.start();
        t2.start();
        t3.start();
    }
}
```





## 13、Java中的锁：ReentrantReadWriteLock

> 目标: 掌握readwriterlock读写锁分析和场景



**ReadWriteLock接口介绍**

 ReadWriteLock也是一个接口，在它里面只定义了两个方法:

```java
package java.util.concurrent.locks;

public interface ReadWriteLock {

    Lock readLock(); // 获取读锁

    Lock writeLock(); // 获取写锁
}
```

 一个用来获取读锁，一个用来获取写锁。也就是说将文件的读写操作分开，分成2个锁来分配给线程，从而使得多个线程可以同时进行读操作。下面的ReentrantReadWriteLock实现了ReadWriteLock接口。 



**ReentrantReadWriteLock介绍**

ReentrantReadWriteLock里面提供了很多丰富的方法，不过最主要的有两个方法: readLock() 和 writeLock()用来获取读锁和写锁。 



**ReentrantReadWriteLock可重入读写锁**

```java
package cn.itcast.thread;

import java.util.concurrent.locks.ReentrantReadWriteLock;

// ReentrantReadWriteLock: 可重入读写锁 (读锁是共享锁，写锁是独享锁)
public class Test11 {
    // 定义读写锁
    public static ReentrantReadWriteLock rw = new ReentrantReadWriteLock();

    public static void main(String[] args) {
        // testReadLock();
        // testWriteLock();
        testReadWriteLock();
    }

    private static void testReadLock() {
        Runnable readRunnable = () -> {
            // 读锁是共享锁
            rw.readLock().lock();
            try {
                for (int i = 0; i < 50; i++) {
                    System.out.println(Thread.currentThread().getName() + "正在进行读操作");
                    Thread.sleep(5);
                }
                System.out.println(Thread.currentThread().getName() + "读操作完毕");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                rw.readLock().unlock();
            }
        };
        new Thread(readRunnable).start();
        new Thread(readRunnable).start();
    }

    private static void testWriteLock() {
        Runnable writeRunnable = () -> {
            // 写锁是独享锁
            rw.writeLock().lock();
            try {
                for (int i = 0; i < 50; i++) {
                    System.out.println(Thread.currentThread().getName() + "正在进行写操作");
                    Thread.sleep(5);
                }
                System.out.println(Thread.currentThread().getName() + "写操作完毕");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                rw.writeLock().unlock();
            }
        };
        new Thread(writeRunnable).start();
        new Thread(writeRunnable).start();
    }

    private static void testReadWriteLock() {
        Runnable readWriteRunnable = () -> {
            // 读锁是共享锁
            rw.readLock().lock();
            try {
                for (int i = 0; i < 50; i++) {
                    System.out.println(Thread.currentThread().getName() + "正在进行读操作");
                    Thread.sleep(5);
                }
                System.out.println(Thread.currentThread().getName() + "读操作完毕");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                rw.readLock().unlock();
            }

            // 写锁是独享锁
            rw.writeLock().lock();
            try {
                for (int i = 0; i < 50; i++) {
                    System.out.println(Thread.currentThread().getName() + "正在进行写操作");
                    Thread.sleep(5);
                }
                System.out.println(Thread.currentThread().getName() + "写操作完毕");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                rw.writeLock().unlock();
            }
        };
        new Thread(readWriteRunnable).start();
        new Thread(readWriteRunnable).start();
    }
}
```



**小结**

**ReentrantReadWriteLock的优势与应用场景**

```txt
1. 大大提升了读操作的效率。

2. 不过要注意的是，如果有一个线程已经占用了读锁，则此时其他线程如果要申请写锁，则申请写锁的线程会一直等待释放读锁。

3. 如果有一个线程已经占用了写锁，则此时其他线程如果申请写锁或者读锁，则申请的线程会一直等待释放写锁。

4. ReentrantReadWriteLock适合读多写少的应用场景
```





## 14、原子更新基本类型

> 目标: 掌握java.util.concurrent.atomic包下原子更新基本类型

**介绍**

Java从JDK 1.5开始提供了java.util.concurrent.atomic包（以下简称Atomic包），这个包中的原子操作类提供了一种用法简单、性能高效、线程安全地更新一个变量的方式。atomic实现原子性操作采用的是CAS算法保证原子性操作，性能高效。

CAS原理分析:

> 使用CAS（Compare-And-Swap）比较相同并交换，操作保证数据原子性
> CAS算法是jdk对并发操作共享数据的支持，包含了3个操作数
> 	第一个操作数: 内存值value(V)
> 	第二个操作数: 预估值expect(E)
> 	第三个操作数: 更新值new(N)
>
> 含义: 当多线程每个线程执行写的操作时，每个线程都会读取主存最新内存值value，并设置预估的值，只有最新内存值与预估值一致的线程，就会将需要更新的值更新到主存中，其他线程就会失败保证原子性操作；这样就解决了synchronized排队导致性能低下的问题。

![1572058334596](就业技术加强-01-Java并发编程.assets/1572058334596.png) 

java.util.concurrent.atomic包的原子类:

![1598253329423](就业技术加强-01-Java并发编程.assets/1598253329423.png) 



**原子更新基本类型**

| 类            | 含义             |
| ------------- | ---------------- |
| AtomicBoolean | 原子更新布尔类型 |
| AtomicInteger | 原子更新整型     |
| AtomicLong    | 原子更新长整型   |

上面3个类提供方法完全一样，所以我们以AtomicInteger为例进行讲解api方法:

| 方法                                          | 含义                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| int addAndGet(int delta)                      | 以原子方式将输入的数值与实例中的值(AtomicInteger里的value)相加，并返回结果 |
| boolean compareAndSet(int expect，int update) | 如果输入的数值等于预期值，则以原子方式将该值设置为输入的值   |
| int getAndIncrement()                         | 以原子方式将当前值加1，注意，这里返回的是自增前的值          |
| void lazySet(int newValue)                    | 最终会设置成newValue，使用lazySet设置值后，可能导致其他线程在之后的一小段时间内还是可以读到旧的值 |
| int getAndSet(int newValue)                   | 以原子方式设置为newValue的值，并返回旧值。                   |
| int incrementAndGet()                         | 以原子方式将当前值加1，注意，这里返回的是自增后的值          |

**代码**

```java
package cn.itcast.thread;

import java.util.concurrent.atomic.AtomicInteger;

public class Test12 {
    private static AtomicInteger atomicInteger = new AtomicInteger(0);
    public static void main(String[] args) {
        Runnable runnable = () -> {
            long start = System.currentTimeMillis();
            for (int i = 0; i < 10000; i++) {
                // 进行原子性操作+1
                atomicInteger.getAndIncrement();
            }
            long end = System.currentTimeMillis();

            System.out.println("运行结果：" + atomicInteger + ", 耗时: " + (end - start));
        };

        // 开启多线程进行操作共享变量
        Thread t1 = new Thread(runnable);
        Thread t2 = new Thread(runnable);
        t1.start();
        t2.start();
    }
}
```

**运行效果**

![image-20210106102630203](就业技术加强-01-Java并发编程.assets/image-20210106102630203.png) 



## 15、原子更新数组类型

> 目标: 掌握java.util.concurrent.atomic包下原子更新数组类型

**介绍**

通过原子的方式更新数组里的某个元素，Atomic包提供了以下3个类。这几个类提供的方法几乎一样，所以本节仅以AtomicIntegerArray为例进行讲解。

| 类                   | 含义                         |
| -------------------- | ---------------------------- |
| AtomicIntegerArray   | 原子更新整型数组里的元素     |
| AtomicLongArray      | 原子更新长整型数组里的元素   |
| AtomicReferenceArray | 原子更新引用类型数组里的元素 |

**AtomicIntegerArray类:**

| 方法                                                 |                                                              |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| int addAndGet(int i，int delta)                      | 以原子方式将输入值与数组中索引i的元素相加                    |
| boolean compareAndSet(int i，int expect，int update) | 如果当前值等于预期值，则以原子方式将数组位置i的元素设置成update值 |

**代码**

```java
package cn.itcast.thread;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicIntegerArray;

public class Test13 {
    // 定义数组
    private static int[] ints = {0,2,3};

    // 定义原子整型数组对象
    private static AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(new int[] {0,2,3});
    
    public static void main(String[] args) throws InterruptedException {
        Runnable runnable = () -> {
            try {
                // 线程休眠
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            // 操作普通数组中的第一个元素 +1
            ints[0] = ints[0] + 1;

            // 操作原子数组中的第一个元素 +1
            atomicIntegerArray.incrementAndGet(0);
        };

        List<Thread> list = new ArrayList<>();
        // 开启多线程进行操作共享变量
        for (int i = 0; i <10 ; i++) {
            Thread thread = new Thread(runnable);
            list.add(thread);
            thread.start();
        }

        for (Thread thread : list) {
            thread.join(); // 确保所有thread全部运行
        }

        System.out.println("操作普通数组ints[0]结果：" + ints[0]); // 不一定
        System.out.println("操作原子数组[0]结果：" + atomicIntegerArray.get(0)); // 10
    }
}
```

**运行效果**

![1598257116316](就业技术加强-01-Java并发编程.assets/1598257116316.png) 

注意:

> 数组value通过构造方法传递进去，然后AtomicIntegerArray会将当前数组复制一份，所以当AtomicIntegerArray对内部的数组元素进行修改时，不会影响传入的数组。





## 16、原子更新引用类型

> 目标: 使用原子操作更新引用类型数据（也就是原子更新多个变量）

**介绍**

原子更新基本类型的AtomicInteger，只能更新一个变量，如果要原子更新多个变量，就需
要使用这个原子更新引用类型提供的类。Atomic包提供了以下3个类:

| 类                          | 介绍                                                         |
| --------------------------- | ------------------------------------------------------------ |
| AtomicReference             | 原子更新引用类型                                             |
| AtomicReferenceFieldUpdater | 原子更新引用类型里的字段                                     |
| AtomicMarkableReference     | 原子更新带有标记位的引用类型。可以原子更新一个布尔类型的标记位和引用类型。构造方法AtomicMarkableReference（V initialRef，boolean initialMark） |

**AtomicReference类:**

| 方法                                      | 介绍                                             |
| ----------------------------------------- | ------------------------------------------------ |
| boolean compareAndSet(V expect, V update) | 如果是期望值expect与当前内存值一样，更新为update |

**代码**

```java
public class Test14 {
    public static void main(String[] args) {
        // 1. 创建一个User对象封装数据
        User user = new User("张三", 18);

        // 2. 创建一个原子引用类型AtomicReference操作User类型数据
        AtomicReference<User> atomicReference = new AtomicReference<>();

        // 3. 将user对象的数据存入原子引用类型对象中
        atomicReference.set(user);
        // 4. 更新原子引用类型存储的数据
        atomicReference.compareAndSet(user, new User("李四", 20));

        // 5. 打印普通user对象数据与原子引用类型对象数据
        System.out.println("普通对象数据："+ user +",对象hashcode: " + user.hashCode());
        System.out.println("原子引用类型对象数据：" + atomicReference.get()
                + ",对象hashcode: " + atomicReference.get().hashCode());
    }
}
```

**运行效果**

![image-20210106143230628](就业技术加强-01-Java并发编程.assets/image-20210106143230628.png) 



不想使用synchronized，怎么保证线程安全

CAS和synchronized

CAS只能保证修改数据库的原子性,没办法保证一段代码的原子性

synchronized保证一段代码的原子性



## 17、并发工具类：CountDownLatch(倒计数闭锁)

> 目标: 掌握CountDownLatch使用(**实现等待其他线程处理完才继续运行当前线程**)

**介绍**

CountDownLatch是一个同步辅助类，也叫倒计数闭锁，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。用给定的计数初始化 CountDownLatch。这个辅助类可以进行计算递减，所以在当前计数到达零之前，可以让现场一直受阻塞。到达0之后，会释放所有等待的线程，执行后续操作。

| CountDownLatch类          | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| CountDownLatch(int count) | 创建CountDownLatch 实例并设置预定计数次数。                  |
| void countDown()          | 递减锁存器的计数，如果计数到达零，则释放所有等待的线程。如果当前计数大于零，则将计数减少1。 |
| void await()              | 使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断。如果当前的计数为零，则此方法立即返回。 |

CountDownLatch 是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减 1。当计数器值到达 0 时，表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。

![1572094947830](就业技术加强-01-Java并发编程.assets/1572094947830.png) 



使用CountDownLatch实现阻塞代码优化:

```java
package cn.itcast.thread;

import java.util.concurrent.CountDownLatch;

public class Test16 {
    // 定义倒计数闭锁对象
    private static CountDownLatch countDownLatch = new CountDownLatch(3);

    public static void main(String[] args) throws InterruptedException {
        System.out.println("将文件分成三部分处理");

        // 创建线程1
        Thread t1 = new Thread(() -> {
            System.out.println("处理第1部分数据");
            countDownLatch.countDown(); // 计算递减1
        });

        // 创建线程2
        Thread t2 = new Thread(() -> {
            System.out.println("处理第2部分数据");
            countDownLatch.countDown(); // 计算递减1
        });

        // 创建线程2
        Thread t3 = new Thread(() -> {
            System.out.println("处理第3部分数据");
            countDownLatch.countDown(); // 计算递减1
        });

        t1.start();
        t2.start();
        t3.start();

        countDownLatch.await();  // 阻塞，计算为0释放阻塞，运行后面的代码
        System.out.println("合并处理结果");
    }
}
```

使用CountDownLatch实现阻塞效果:

![image-20210106145425186](就业技术加强-01-Java并发编程.assets/image-20210106145425186.png)







## 18、并发工具类：CyclicBarrier(同步屏障)

> 目标: 掌握CyclicBarrier的使用

**介绍**

CyclicBarrier是JDK 1.5的 java.util.concurrent 并发包中提供的一个并发工具类。 

- 所谓 Cyclic 即**循环**的意思，所谓 Barrier 即**屏障**的意思。
- CyclicBarrier (可重用屏障/栅栏)类似于 CountDownLatch功能一样，都有让多个线程等待同步然后再开始下一步动作。
- CyclicBarrier 可以使一定数量的线程反复地在屏障位置处汇集。当线程到达屏障位置时将调用 `await()` 方法，这个方法将阻塞直到所有线程都到达屏障位置。如果所有线程都到达屏障位置，那么屏障将打开，此时所有的线程都将被释放，而屏障将被重置以便下次使用。

![1598327082618](就业技术加强-01-Java并发编程.assets/1598327082618.png) 


| CyclicBarrier类                                    | 说明                                                         |
| -------------------------------------------------- | ------------------------------------------------------------ |
| CyclicBarrier(int parties)                         | 创建对象，参数表示屏障拦截的线程数量，初始化相互等待的线程数量 |
| int await()                                        | 告诉CyclicBarrier自己已经到达了屏障，然后当前线程被阻塞返回值int为达到屏障器的索引: 索引未达到屏障线程数量-1，0表示最后一个达到屏障 |
| int getParties()                                   | 获取 CyclicBarrier 打开屏障的线程数量                        |
| void reset()                                       | 使CyclicBarrier回归初始状态，它做了两件事。如果有正在等待的线程，则会抛出 BrokenBarrierException 异常，且这些线程停止等待，继续执行。将是否破损标志位broken置为 false。 |
| boolean isBroken()                                 | 获取是否破损标志位broken的值，此值有以下几种情况。1.CyclicBarrier初始化时，broken=false表示屏障未破损。 2.如果正在等待的线程被中断,则broken=true,表示屏障破损。 3.如果正在等待的线程超时, 则broken=true,表示屏障破损。 4.如果有线程调用 CyclicBarrier.reset() 方法，则broken=false，表示屏障回到未破损状态。 |
| int getNumberWaiting()                             | 获取达到屏障阻塞等待的线程数                                 |
| CyclicBarrier(int parties，Runnable barrierAction) | 用于所有线程到达屏障时，优先执行barrierAction的线程          |

**代码**

```java
package cn.itcast.thread;

import java.util.Random;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.TimeUnit;

public class Test17 {
    // 定义同步屏障对象
    private static CyclicBarrier barrier = new CyclicBarrier(5);

    public static void main(String[] args) {
        Runnable runnable = () -> {
            try {
                // 使用随机数表示人到了
                Thread.sleep(new Random().nextInt(3000)); // 0~2999
                System.out.println(Thread.currentThread().getName() + " 进入房间,选择角色! 已经来了");
                barrier.await();
                System.out.println(Thread.currentThread().getName() + " 进行游戏");
            } catch (Exception e) {
                e.printStackTrace();
            }
        };

        new Thread(runnable).start();
        new Thread(runnable).start();
        new Thread(runnable).start();
        new Thread(runnable).start();
        new Thread(runnable).start();
    }
}
```

**运行效果**

![image-20210106151131990](就业技术加强-01-Java并发编程.assets/image-20210106151131990.png) 



如果只启动4个线程。线程数量不够5个，这些线程会一直处于等待状态

![image-20210106151244926](就业技术加强-01-Java并发编程.assets/image-20210106151244926.png)





## 19、并发工具类：Semaphore(信号量)

> 目标: 掌握Semaphore的使用

**介绍**

**Semaphore（信号量）限制着访问某些资源的线程数量**，在到达限制的线程数量之前，线程能够继续进行资源的访问，一旦访问资源的数量到达限制的线程数量，这个时候线程就不能够再去获取资源，只有等待有线程退出资源的获取。

**应用场景**

比如模拟一个停车场停车信号，假设停车场只有两个车位，一开始两个车位都是空的。这时同时来了两辆车，看门人允许它们进入停车场，然后放下车拦。以后来的车必须在入口等待，直到停车场中有车辆离开。这时，如果有一辆车离开停车场，看门人得知后，打开车拦，放入一辆，如果又离开一辆，则又可以放入一辆，如此往复。

![1598327854330](就业技术加强-01-Java并发编程.assets/1598327854330.png) 

**api方法**

信号量维护了一个许可集。如有必要，在许可可用前会阻塞每一个 **acquire()**，然后再获取该许可。每个 **release()** 添加一个许可，从而可能释放一个正在阻塞的获取者让其运行。但是，不使用实际的许可对象，Semaphore 只对可用许可的号码进行计数，并采取相应的行动。 

所以一个Semaphore 信号量有且仅有 3 种操作，且它们全部是原子的。 

- 初始化许可集、增加许可、获取许可。
- 增加许可，**release()**方法释放一个阻塞，增加一个许可。
- 获取许可，**acquire()**方法获取许可，再获取许可前处于阻塞等待。

| 方法                                 | 说明                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| Semaphore(int permits)               | permits是允许同时运行的线程数目,创建指定数据线程的信号量     |
| Semaphore(int permits, boolean fair) | permits是允许同时运行的线程数目,创建指定数据线程的信号量；fair指定是公平模式还是非公平模式，默认非公平模式 |
| void acquire()                       | 方法阻塞，直到申请获取到许可证才可以运行当前线程             |
| void release()                       | 释放当前线程一个阻塞的 `acquire()` 方法，方法增加一个许可证  |
| intavailablePermits()                | 返回此信号量中当前可用的许可证数                             |
| intgetQueueLength()                  | 返回正在等待获取许可证的线程数                               |
| booleanhasQueuedThreads()            | 是否有线程正在等待获取许可证                                 |
| void reducePermits（int reduction）  | 减少reduction个许可证，是个protected方法。                   |
| Collection getQueuedThreads()        | 返回所有等待获取许可证的线程集合，是个protected方法          |

**代码**

```java
package cn.itcast.thread;

import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

public class Test18 {

    public static void main(String[] args) {
        // 1. 创建信号量对象控制并发线程数量，设置许可数2个（同时运行2个线程）
        Semaphore semaphore = new Semaphore(2, true);

        // 2. 循环运行10个线程（会看到每次只允许2个线程）
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try {
                    // 2.1 申请获取许可
                    semaphore.acquire();

                    // 2.2 运行业务
                    System.out.println(Thread.currentThread().getName() + "车，进入停车场");
                    TimeUnit.SECONDS.sleep(1);// 让当前线程休眠（让线程多运行一会，方便观察效果）
                    System.out.println(Thread.currentThread().getName() + "车，离开停车场>>>>>>>>>");

                    // 2.3 释放阻塞，增加一个许可（让下一个阻塞的线程运行）
                    semaphore.release();

                } catch (Exception e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```

**运行效果**

![image-20210106151635787](就业技术加强-01-Java并发编程.assets/image-20210106151635787.png)

**特点**

Semaphore 在计数器不为 0 的时候对线程就放行，一旦达到 0，那么所有请求资源的新线程都会被阻塞，包括增加请求到许可的线程，Semaphore 是不可重入的。 

- 每一次请求一个许可都会导致计数器减少 1，同样每次释放一个许可都会导致计数器增加 1，一旦达到 0，新的许可请求线程将被挂起。

Semaphore 有两种模式，**公平模式** 和 **非公平模式(默认使用)**

- 公平模式就是调用 acquire 的顺序就是**获取许可证的顺序**，遵循 FIFO。

  ```java
  Semaphore semaphore = new Semaphore(许可数, true); // 公平模式
  ```

- 非公平模式是抢占式的，也就是有可能一个新的获取线程恰好在一个许可证释放时得到了这个许可证，而前面还有等待的线程。

  ```java
  Semaphore semaphore = new Semaphore(许可数, false); // 非公平模式，(默认)
  ```


**小结**

+ Semaphore 的使用(3个操作)

  + 初始化许可集
  + 增加许可，**release()**方法释放一个阻塞，增加一个许可。
  + 获取许可，**acquire()**方法获取许可，再获取许可前处于阻塞等待。

  

  

  

## 20、并发工具类：Exchanger(交换数据)

> 目标: 掌握Exchanger的使用

**介绍**

Exchanger（交换者）是一个用于线程间协作的工具类，可以用于进行线程间的数据交换。

**交换数据原理**

+ Exchanger提供一个同步点，在这个同步点两个线程可以交换彼此的数据。

+ 两个线程通过 exchange() 方法交换数据。

+ 第一个线程先执行 exchange() 方法，会一直等待第二个线程也执行exchange()到达同步点时，两个线程交换数据，将本线程生产出来的数据传递给对方。

+ 使用 Exchanger 的重点是用对的线程使用 exchange() 方法。

  ![image-20191104101201808](就业技术加强-01-Java并发编程.assets/image-20191104101201808.png) 

**api方法**

| 方法                                         | 说明                       |
| -------------------------------------------- | -------------------------- |
| V exchange(V x)                              | 用于进行线程间的数据交换   |
| V exchange(V x, long timeout, TimeUnit unit) | 设置交换数据并等待超时时间 |

别人给你的数据 exchange(你给别人的数据)



**代码**

```java
 package cn.itcast.thread;

import java.util.concurrent.Exchanger;

public class Test19 {

    public static void main(String[] args) {

        // 1. 创建交换数据对象，并设置传输数据的类型
        Exchanger<String> exchanger = new Exchanger<>();

        // 2. 启动2个线程进行交换数据
        // 创建线程1
        new Thread(() -> {
            // 2.1 定义交换的数据
            String girl1 = "【柳岩】";
            System.out.println(Thread.currentThread().getName() + "说：我的女友 " + girl1);
            System.out.println(Thread.currentThread().getName() + "说：等待线程2交换数据");
            // 2.2 将数据交换给线程2,并拿到线程2的数据
            try {
                String b = exchanger.exchange(girl1);//注意：如果线程2没有到达同步点，当前线程会被阻塞一直等到
                //成功获取线程2的数据后
                System.out.println(Thread.currentThread().getName() + "说：我拿到了 " + b);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        // 创建线程2
        new Thread(()->{
            // 2.3 定义交换的数据
            String girl2 = "【杨幂】";
            System.out.println(Thread.currentThread().getName()+"说：我的女友 " + girl2);
            System.out.println(Thread.currentThread().getName()+"说：等待线程1交换数据");
            // 2.4 将数据交换给线程1,并拿到线程1的数据
            try {
                String a = exchanger.exchange(girl2);
                // 成功获取线程1的数据后
                System.out.println(Thread.currentThread().getName()+"说：我拿到了 " + a);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

**运行效果**

![1598322147550](就业技术加强-01-Java并发编程.assets/1598322147550.png) 

> 注意: 如果两个线程有一个没有执行exchange()方法，则会一直等待，如果担心有特殊情况发
> 生，避免一直等待，可以使用exchange（V x，longtimeout，TimeUnit unit）设置最大等待时长。







## 21、Java并发编程：常见面试题总结

+ 面试题1: 简述线程、进程、程序的基本概念？

  ```txt
  1.进程 我们把运行中的程序叫做进程,每个进程都会占用内存与CPU资源,进程与进程之间互相独立.
  2.线程 线程就是进程中的一个执行单元，负责当前进程中程序的执行。一个进程可以包含多个线程。一个进程包含了多个线程就是多线程。多线程可以提高程序的并行运行效率。是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。
  3.程序 是含有指令和数据的文件，被存储在磁盘或其他的数据存储设备中，也就是说程序是静态的代码。
  ```

+ 面试题2: 线程有什么优缺点？

  ```txt
  优点:
  1. 在多核CPU中，通过并行计算提高程序性能. 比如一个方法的执行比较耗时，现在把这个方法逻辑拆分，分为若干个线程并发执行，提高程序效率。
  2. 可以解决网络等待、io响应导致的耗时问题。
  3. 可以随时停止任务
  4. 可以分别设置各个任务的优先级以优化性能
  5. 提高CPU的使用率.提高网络资源的利用率
  
  缺点:
  1. 线程也是程序，所以线程需要占用内存，线程越多占用内存也越多；
  2. 线程之间对共享资源的访问会相互影响，必须解决竞用共享资源的问题；
  3. 多线程存在上下文切换问题
  CPU 通过时间片分配算法来循环执行任务，当前任务执行一个时间片后会切换到下一个任务。但是，在切换前会保存上一个任务的状态，以便下次切换回这个任务时，可以再加载这个任务的状态。所以任务从保存到再加载的过程就需要进行上下文切换，本身就会占用cpu资源。
  ```

+ 面试题3: 为什么wait和notify方法放在Object类?

  ```txt
   因为wait和notify需要使用锁对象来调用,而任何对象都可以作为锁,所以放在Object类中。
  ```

+ 面试题4: sleep和wait的区别?

  ```txt
  1. sleep 睡眠的时候不会释放锁 Thread.sleep(100)
  2. wait 等待的时候会释放锁
  ```

+ 面试题5: synchronized和lock的区别?

  ```txt
  1. lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现；
  
  2. synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unlock()去释放锁，则很有可能造成死锁现象，因此使用lock时需要在finally块中释放锁；
  
  3. lock可以通过tryLock(timeout)让等待锁的线程超时响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断；
  
  4. 通过lock可以知道有没有成功获取锁，而synchronized却无法办到；
  
  5. lock可以提高多个线程进行读操作的效率。(可以通过readwritelock实现读写分离)
  
  6. 性能上来说，在资源竞争不激烈的情形下，Lock性能稍微比synchronized差点(编译程序通常尽可能的进行优化synchronized)。但是当同步非常激烈的时候，synchronized的性能就会下降几十倍。而ReentrantLock确还能维持常态。
  ```



