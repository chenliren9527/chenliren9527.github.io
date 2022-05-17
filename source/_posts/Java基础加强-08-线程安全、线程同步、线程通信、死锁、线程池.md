---
title: Java基础加强(08)-线程安全、线程同步、线程通信、死锁、线程池
tags:
  - 笔记
  - Java基础加强
  - 线程安全
  - 线程同步
  - 线程通信
  - 死锁
  - 线程池
  - 多线程
categories:
  - Java基础加强
date: 2020-09-06 23:09:14
---



## 今日内容

* 线程安全
* 线程同步
* 线程通信
* 死锁
* 线程池

## 教学目标  

- [ ] 能够解释安全问题的出现的原因
- [ ] 能够使用同步代码块解决线程安全问题
- [ ] 能够使用同步方法解决线程安全问题
- [ ] 能够描述Java中线程池运行原理
- [ ] 能够描述死锁产生的原因
- [ ] 能够理解等待唤醒案例
- [ ] 能够说出线程6个状态的名称



# 第一章 线程安全

## 1.1 线程安全

如果有多个线程在同时运行，而这些线程可能会同时运行这段代码。程序每次运行结果和单线程运行的结果是一样的，而且其他的变量的值也和预期的是一样的，就是线程安全的。

我们通过一个案例，演示线程的安全问题：

电影院要卖票，我们模拟电影院的卖票过程。假设要播放的电影是 “葫芦娃大战奥特曼”，本次电影的座位共100个(本场电影只能卖100张票)。

我们来模拟电影院的售票窗口，实现多个窗口同时卖 “葫芦娃大战奥特曼”这场电影票(多个窗口一起卖这100张票)

需要窗口，采用线程对象来模拟；需要票，Runnable接口子类来模拟。

模拟票：

```java
public class Ticket implements Runnable {
    private int ticket = 100;
    /*
     * 执行卖票操作
     */
    @Override
    public void run() {
        //每个窗口卖票的操作 
        //窗口 永远开启 
        while (true) {
            if (ticket > 0) {//有票 可以卖
                //出票操作
                //使用sleep模拟一下出票时间 
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                //获取当前线程对象的名字 
                String name = Thread.currentThread().getName();
                System.out.println(name + "正在卖:" + ticket--);
            }
        }
    }
}
```

测试类：

```java
public class Demo {
	public static void main(String[] args) {
		//创建线程任务对象
		Ticket ticket = new Ticket();
		//创建三个窗口对象
		Thread t1 = new Thread(ticket, "窗口1");
		Thread t2 = new Thread(ticket, "窗口2");
		Thread t3 = new Thread(ticket, "窗口3");
		
		//同时卖票
		t1.start();
		t2.start();
		t3.start();
	}
}
```

结果中有一部分这样现象：

![](线程安全、线程同步、线程通信、死锁、线程池.assets/线程安全问题.png)

发现程序出现了两个问题：

1. 相同的票数,比如5这张票被卖了两回。
2. 不存在的票，比如0票与-1票，是不存在的。

这种问题，几个窗口(线程)票数不同步了，这种问题称为线程不安全。

> 线程安全问题都是由全局变量及静态变量引起的。若每个线程中对全局变量、静态变量只有读操作，而无写操作，一般来说，这个全局变量是线程安全的；若有多个线程同时执行写操作，一般都需要考虑线程同步，否则的话就可能影响线程安全。

## 1.2 线程同步

线程同步是为了解决线程安全问题。

当我们使用多个线程访问同一资源的时候，且多个线程中对资源有写的操作，就容易出现线程安全问题。

要解决上述多线程并发访问一个资源的安全性问题:也就是解决重复票与不存在票问题，Java中提供了同步机制(**synchronized**)来解决。

根据案例简述：

```
窗口1线程进入操作的时候，窗口2和窗口3线程只能在外等着，窗口1操作结束，窗口1和窗口2和窗口3才有机会进入代码去执行。也就是说在某个线程修改共享资源的时候，其他线程不能修改该资源，等待修改完毕同步之后，才能去抢夺CPU资源，完成对应的操作，保证了数据的同步性，解决了线程不安全的现象。
```

为了保证每个线程都能正常执行原子操作,Java引入了线程同步机制。

那么怎么去使用呢？有三种方式完成同步操作：

1. 同步代码块。
2. 同步方法。
3. 锁机制。

## 1.3 同步代码块

- **同步代码块**：`synchronized`关键字可以用于方法中的某个区块中，表示只对这个区块的资源实行互斥访问。

格式: 

```java
synchronized(同步锁){
     需要同步操作的代码
}
```

**同步锁**:

对象的同步锁只是一个概念,可以想象为在对象上标记了一个锁.

1. 锁对象 可以是任意类型。
2. 多个线程对象  要使用同一把锁。

> 注意:在任何时候,最多允许一个线程拥有同步锁,谁拿到锁就进入代码块,其他的线程只能在外等着(BLOCKED)。

使用同步代码块解决代码：

```java
public class Ticket implements Runnable{
	private int ticket = 100;
	
	Object lock = new Object();
	/*
	 * 执行卖票操作
	 */
	@Override
	public void run() {
		//每个窗口卖票的操作 
		//窗口 永远开启 
		while(true){
			synchronized (lock) {
				if(ticket>0){//有票 可以卖
					//出票操作
					//使用sleep模拟一下出票时间 
					try {
						Thread.sleep(50);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					//获取当前线程对象的名字 
					String name = Thread.currentThread().getName();
					System.out.println(name+"正在卖:"+ticket--);
				}
			}
		}
	}
}
```

当使用了同步代码块后，上述的线程的安全问题，解决了。

## 1.4 同步方法

- **同步方法**:使用synchronized修饰的方法,就叫做同步方法,保证A线程执行该方法的时候,其他线程只能在方法外等着。

格式：

```java
public synchronized void method(){
   	可能会产生线程安全问题的代码
}
```

> 同步锁是谁?
>
> ​      对于非static方法,同步锁就是this。  
>
> ​      对于static方法,我们使用当前方法所在类的字节码对象(类名.class)。

使用同步方法代码如下：

```java
public class Ticket implements Runnable{
	private int ticket = 100;
	/*
	 * 执行卖票操作
	 */
	@Override
	public void run() {
		//每个窗口卖票的操作 
		//窗口 永远开启 
		while(true){
			sellTicket();
		}
	}
	
	/*
	 * 锁对象 是 谁调用这个方法 就是谁 
	 *   隐含 锁对象 就是  this
	 *    
	 */
	public synchronized void sellTicket(){
        if(ticket>0){//有票 可以卖	
            //出票操作
            //使用sleep模拟一下出票时间 
            try {
              	Thread.sleep(100);
            } catch (InterruptedException e) {
              	// TODO Auto-generated catch block
              	e.printStackTrace();
            }
            //获取当前线程对象的名字 
            String name = Thread.currentThread().getName();
            System.out.println(name+"正在卖:"+ticket--);
        }
	}
}
```

## 1.5 Lock锁

`java.util.concurrent.locks.Lock`机制提供了比**synchronized**代码块和**synchronized**方法更广泛的锁定操作,同步代码块/同步方法具有的功能Lock都有,除此之外更强大

Lock锁也称同步锁，加锁与释放锁方法化了，如下：

- `public void lock() `:加同步锁。
- `public void unlock()`:释放同步锁。

使用如下：

```java
public class Ticket implements Runnable{
	private int ticket = 100;
	
	Lock lock = new ReentrantLock();
	/*
	 * 执行卖票操作
	 */
	@Override
	public void run() {
		//每个窗口卖票的操作 
		//窗口 永远开启 
		while(true){
			lock.lock();
			if(ticket>0){//有票 可以卖
				//出票操作 
				//使用sleep模拟一下出票时间 
				try {
					Thread.sleep(50);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				//获取当前线程对象的名字 
				String name = Thread.currentThread().getName();
				System.out.println(name+"正在卖:"+ticket--);
			}
			lock.unlock();
		}
	}
}
```





# 第二章 线程状态

## 2.1 线程状态概述

线程由生到死的完整过程：技术素养和面试的要求。

当线程被创建并启动以后，它既不是一启动就进入了执行状态，也不是一直处于执行状态。在线程的生命周期中，有几种状态呢？在API中`java.lang.Thread.State`这个枚举中给出了六种线程状态：

这里先列出各个线程状态发生的条件，下面将会对每种状态进行详细解析

| 线程状态                | 导致状态发生条件                                             |
| ----------------------- | ------------------------------------------------------------ |
| NEW(新建)               | 线程刚被创建，但是并未启动。还没调用start方法。MyThread t = new MyThread只有线程对象，没有线程特征。 |
| Runnable(可运行)        | 线程可以在java虚拟机中运行的状态，可能正在运行自己代码，也可能没有，这取决于操作系统处理器。调用了t.start()方法   ：就绪（经典教法） |
| Blocked(锁阻塞)         | 当一个线程试图获取一个对象锁，而该对象锁被其他的线程持有，则该线程进入Blocked状态；当该线程持有锁时，该线程将变成Runnable状态。 |
| Waiting(无限等待)       | 一个线程在等待另一个线程执行一个（唤醒）动作时，该线程进入Waiting状态。进入这个状态后是不能自动唤醒的，必须等待另一个线程调用notify或者notifyAll方法才能够唤醒。 |
| Timed Waiting(计时等待) | 同waiting状态，有几个方法有超时参数，调用他们将进入Timed Waiting状态。这一状态将一直保持到超时期满或者接收到唤醒通知。带有超时参数的常用方法有Thread.sleep 、Object.wait。 |
| Teminated(被终止)       | 因为run方法正常退出而死亡，或者因为没有捕获的异常终止了run方法而死亡。 |

![1573352376549](线程安全、线程同步、线程通信、死锁、线程池.assets/1573352376549.png)

​	我们不需要去研究这几种状态的实现原理，我们只需知道在做线程操作中存在这样的状态。那我们怎么去理解这几个状态呢，新建与被终止还是很容易理解的，我们就研究一下线程从Runnable（可运行）状态与非运行状态之间的转换问题。

## 2.2 睡眠sleep方法

我们看到状态中有一个状态叫做计时等待，可以通过Thread类的方法来进行演示.

`public static void  sleep(long time)`  让当前线程进入到睡眠状态，到毫秒后自动醒来继续执行

```java
public class Test{
  public static void main(String[] args){
    for(int i = 1;i<=5;i++){
      	Thread.sleep(1000);
        System.out.println(i)   
    } 
  }
}
```

这时我们发现主线程执行到sleep方法会休眠1秒后再继续执行。

## 2.3 等待和唤醒

Object类的方法

`public void wait()` : 让当前线程进入到等待状态 此方法必须锁对象调用.

```java
public class Demo1_wait {
    public static void main(String[] args) throws InterruptedException {
	   // 步骤1 : 子线程开启,进入无限等待状态, 没有被唤醒,无法继续运行.
        new Thread(() -> {
            try {

                System.out.println("begin wait ....");
                synchronized ("") {
                    "".wait();
                }
                System.out.println("over");
            } catch (Exception e) {
            }
        }).start();
    }
```

`public void notify()` : 唤醒当前锁对象上等待状态的线程  此方法必须锁对象调用.

```java
public class Demo2_notify {
    public static void main(String[] args) throws InterruptedException {
	   // 步骤1 : 子线程开启,进入无限等待状态, 没有被唤醒,无法继续运行.
        new Thread(() -> {
            try {

                System.out.println("begin wait ....");
                synchronized ("") {
                    "".wait();
                }
                System.out.println("over");
            } catch (Exception e) {
            }
        }).start();

        //步骤2:  加入如下代码后, 3秒后,会执行notify方法, 唤醒wait中线程.
        Thread.sleep(3000);
        new Thread(() -> {
            try {
                synchronized ("") {
                    System.out.println("唤醒");
                    "".notify();
                }
            } catch (Exception e) {
            }
        }).start();
    }
}
```

## 2.4 等待唤醒案例（包子铺卖包子）

```java
定义一个集合，包子铺线程完成生产包子，包子添加到集合中；吃货线程完成购买包子，包子从集合中移除。
1. 当包子没有时（包子状态为false），吃货线程等待.
2. 包子铺线程生产包子（即包子状态为true），并通知吃货线程（解除吃货的等待状态）
```

**代码示例：**

生成包子类：

```java
public class BaoZiPu extends Thread {
    private List<String> list;
    public BaoZiPu(ArrayList<String> list) {
        this.list = list;
    }
    
    @Override
    public void run() {
        int i = 0; 
        while(true) {
            //list作为锁对象
            synchronized (list) {
                if(list.size() > 0) {
                    //存元素的线程进入到等待状态
                    try {
                        list.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                //如果线程没进入到等待状态 说明集合中没有元素
                //向集合中添加元素
                list.add("包子"+i++);
                System.out.println(list);
                //集合中已经有元素了 唤醒获取元素的线程
                list.notify();
            }
        }
    }
}
```

消费包子类：

```java
public class ChiHuo extends Thread {
    private List<String> list;
    public ChiHuo(ArrayList<String> list) {
        this.list = list;
    }

    @Override
    public void run() {
        //为了能看到效果 写个死循环
        while(true) {
            //由于使用的同一个集合 list作为锁对象
            synchronized (list) {
                //如果集合中没有元素 获取元素的线程进入到等待状态
                if(list.size()==0) {
                    try {
                        list.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                //如果集合中有元素 则获取元素的线程获取元素(删除)
                list.remove(0);
                //打印集合 集合中没有元素了
                System.out.println(list);
                //集合中已经没有元素 则唤醒添加元素的线程 向集合中添加元素
                list.notify();
            }
        }
    }
}
```

测试类：

```java
public class Demo {
    public static void main(String[] args) {
        //等待唤醒案例
        List<String> list = new ArrayList<>();
        // 创建线程对象        
         BaoZiPu bzp = new BaoZiPu(list);
        ChiHuo ch = new ChiHuo(list);
        // 开启线程
        bzp.start();
        ch.start();
    }
}
```



# 第三章 线程池方式

## 3.1 线程池的思想 



![](线程安全、线程同步、线程通信、死锁、线程池.assets/游泳池.jpg)

我们使用线程的时候就去创建一个线程，这样实现起来非常简便，但是就会有一个问题：

如果并发的线程数量很多，并且每个线程都是执行一个时间很短的任务就结束了，这样频繁创建线程就会大大降低系统的效率，因为频繁创建线程和销毁线程需要时间，线程也属于宝贵的系统资源。

那么有没有一种办法使得线程可以复用，就是执行完一个任务，并不被销毁，而是可以继续执行其他的任务？

在Java中可以通过线程池来达到这样的效果。今天我们就来详细讲解一下Java的线程池。

## 3.2 线程池概念

- **线程池：**其实就是一个容纳多个线程的容器，其中的线程可以反复使用，省去了频繁创建线程对象的操作，无需反复创建线程而消耗过多资源。

由于线程池中有很多操作都是与优化资源相关的，我们在这里就不多赘述。我们通过一张图来了解线程池的工作原理：

![](线程安全、线程同步、线程通信、死锁、线程池.assets/线程池原理.bmp)

合理利用线程池能够带来三个好处：

1. 降低资源消耗。减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。
2. 提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
3. 提高线程的可管理性。可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存，而把服务器累趴下(每个线程需要大约1MB内存，线程开的越多，消耗的内存也就越大，最后死机)。

> 当线程池中线程已经被任务分配关后，多余的任务就会等待，直到有新的线程空闲

## 3.3 线程池的使用

Java里面线程池的顶级接口是`java.util.concurrent.Executor`，但是严格意义上讲`Executor`并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是`java.util.concurrent.ExecutorService`。

要配置一个线程池是比较复杂的，尤其是对于线程池的原理不是很清楚的情况下，很有可能配置的线程池不是较优的，因此在`java.util.concurrent.Executors`线程工厂类里面提供了一些静态工厂，生成一些常用的线程池。官方建议使用Executors工程类来创建线程池对象。

Executors类中有个创建线程池的方法如下：

- `public static ExecutorService newFixedThreadPool(int nThreads)`：返回线程池对象。(创建的是有界线程池,也就是池中的线程个数可以指定最大数量)

获取到了一个线程池ExecutorService 对象，那么怎么使用呢，在这里定义了一个使用线程池对象的方法如下：

- `public Future<?> submit(Runnable task)`:获取线程池中的某一个线程对象，并执行

  > Future接口：用来记录线程任务执行完毕后产生的结果。

使用线程池中线程对象的步骤：

1. 创建线程池对象。
2. 创建Runnable接口子类对象。(task)
3. 提交Runnable接口子类对象。(take task)
4. 关闭线程池(一般不做)。

**Runnable实现类代码：**

```java
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
```

线程池测试类：

```java
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
```

**Callable测试代码:**

- `<T> Future<T> submit(Callable<T> task)` : 获取线程池中的某一个线程对象，并执行.

  Future : 表示计算的结果.

- `V get()  ` : 获取计算完成的结果。

```java
public class ThreadPoolDemo2 {
    public static void main(String[] args) throws Exception {
        // 创建线程池对象
      ExecutorService service = Executors.newFixedThreadPool(2);//包含2个线程对象

        // 创建Runnable实例对象
        Callable<Double> c = new Callable<Double>() {
            @Override
            public Double call() throws Exception {
                return Math.random();
            }
        };

        // 从线程池中获取线程对象,然后调用Callable中的call()
        Future<Double> f1 = service.submit(c);
        // Futur 调用get() 获取运算结果
        System.out.println(f1.get());

        Future<Double> f2 = service.submit(c);
        System.out.println(f2.get());

        Future<Double> f3 = service.submit(c);
        System.out.println(f3.get());
    }
}
```

## 3.4 线程池的练习

**需求:** 使用线程池方式执行任务,返回1-n的和

**分析:** 因为需要返回求和结果,所以使用Callable方式的任务

**代码:**

```java
public class Demo04 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(3);

        SumCallable sc = new SumCallable(100);
        Future<Integer> fu = pool.submit(sc);
        Integer integer = fu.get();
        System.out.println("结果: " + integer);
        
        SumCallable sc2 = new SumCallable(200);
        Future<Integer> fu2 = pool.submit(sc2);
        Integer integer2 = fu2.get();
        System.out.println("结果: " + integer2);

        pool.shutdown();
    }
}
```



**SumCallable.java**

```java
public class SumCallable implements Callable<Integer> {
    private int n;

    public SumCallable(int n) {
        this.n = n;
    }

    @Override
    public Integer call() throws Exception {
        // 求1-n的和?
        int sum = 0;
        for (int i = 1; i <= n; i++) {
            sum += i;
        }
        return sum;
    }
}
```



# 第四章 死锁

## 4.1 什么是死锁

在多线程程序中,使用了多把锁,造成线程之间相互等待.程序不往下走了。

## 4.2 产生死锁的条件

1.有多把锁
2.有多个线程
3.有同步代码块嵌套

## 4.3 死锁代码

```java
public class Demo05 {
    public static void main(String[] args) {
        MyRunnable mr = new MyRunnable();

        new Thread(mr).start();
        new Thread(mr).start();
    }
}

class MyRunnable implements Runnable {
    Object objA = new Object();
    Object objB = new Object();

    /*
    嵌套1 objA
    嵌套1 objB
    嵌套2 objB
    嵌套1 objA
     */
    @Override
    public void run() {
        synchronized (objA) {
            System.out.println("嵌套1 objA");
            synchronized (objB) {// t2, objA, 拿不到B锁,等待
                System.out.println("嵌套1 objB");
            }
        }

        synchronized (objB) {
            System.out.println("嵌套2 objB");
            synchronized (objA) {// t1 , objB, 拿不到A锁,等待
                System.out.println("嵌套2 objA");
            }
        }
    }
}
```

注意:**我们应该尽量避免死锁**

