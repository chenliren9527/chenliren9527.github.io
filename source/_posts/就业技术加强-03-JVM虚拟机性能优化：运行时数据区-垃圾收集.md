---
title: 就业技术加强(03)-JVM虚拟机性能优化：运行时数据区&垃圾收集
tags:
  - 笔记
  - 就业技术加强
  - JVM调优
  - GC
categories:
  - 就业技术加强
date: 2021-01-25 19:55:10
---

 

## 课程目标

目标1：能够描述运行时数据区

目标2：能够说出垃圾回收机制

目标3：能够说出JVM的类加载机制

目标4：能够说出双亲委派模型

目标5：能够使用JVM性能监控工具

目标6：能够说出JVM性能调优





## JVM的命令行参数参考

https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html



HotSpot参数分类

```shell
标准: - 开头，所有的HotSpot都支持
非标准：-X 开头，特定版本HotSpot支持特定命令
不稳定：-XX 开头，不保证将来可以使用
````

举例:

````shell
java -help
java -Xms20m
java -XX:+PrintInlining
```



```shell
java -XX:+PrintFlagsInitial 打印默认参数值
java -XX:+PrintFlagsFinal 打印最终参数值
```



## 01、Java：背景回顾

### 1.1 Java概述

TIOBE编程语言排行榜是一个比较权威的编程语言使用情况统计机构。今年9月份，TIOBE更新了最新的全球编程语言排行榜。数据显示(Java与C轮询上阵，要么C第一，要么Java第一):

![1599748851312](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1599748851312.png) 

Java能获得广泛的认可，除了它拥有一门结构严谨、面向对象的编程语言之外，还有许多不可忽视的优点:

+ 摆脱了硬件平台的束缚，实现一次编写 到处运行的理想
+ 提供了相对安全的内存管理和访问机制，避免了绝大部分的内存泄漏
+ 实现了热点代码检测和运行时编译及优化，使得java应用能随着运行时间的增加而获得更高的性能
+ 有着良好的生态环境，还有这无数商业机构和开源社区的第三方类库来帮助它实现各种各样的功能

> 作为一名Java程序员，我们是很幸福的，在编写程序时可以尽情发挥Java的各种优势，但与此同时我们也应该去了解和思考下Java技术体系中这些技术特性是如何实现的。认识这些技术运作的本质，是自己思考程序该怎么写更高一些的基础和前提，而且在市场竞争日益激烈的今天，你对基础和底层了解的多少，也是面试官衡量一个程序员是否合格的一个标准！



### 1.2 Java技术体系

Sun公司定义的传统Java技术体系主要有以下几部分组成:

+ Java程序设计语言
+ Class文件
+ JavaAPI类库
+ Java虚拟机
+ 第三方类库

我们可以把Java程序设计语言、Java虚拟机、JavaAPI类库 统称为JDK (Java Development Kit), JDK是用于支持Java程序开发的最小环境。  

![image-20200416213941618](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200416213941618.png)

以上是按照各个组成部分的功能来进行划分的，如果按照技术所服务的领域来划分，Java的技术体系可以分为4个平台:

+ **Java Card:** 支持一些Java小程序,运行在小型内存设备上的平台(如:智能卡 sim卡 提款卡)。
+ **Java ME:** 支持Java程序运行在移动终端(机顶盒、移动电话和PDA)上的平台。
+ **Java SE:** 支持面向桌面级应用的Java平台，提供了完整的Java核心API，以前叫做J2SE。
+ **Java EE:** 支持使用多层架构的企业级应用的Java平台，除了提供Java SE API之外还对其做了大量的扩充。以前叫做J2EE。



### 1.3 Java发展史

~~~
# JDK Beta 1995-05-23

# JAVA 1.0 ,代号Oak（橡树）
  于1996-01-23发行

# JAVA 1.1
  1997-02-19发行,主要更新内容:
  1.引入JDBC
  2.添加内部类支持
  3.引入JAVA BEAN
  4.引入RMI
  5.引入反射

# JAVA 1.2, 代号Playground（操场）
  1998-12-8发行，主要更新内容：
  1.引入集合框架
  2.对字符串常量做内存映射
  3.引入JIT（Just In Time）编译器
  4.引入打包文件数字签名
  5.引入控制授权访问系统资源策略工具
  6.引入JFC（Java Foundation Classes），包括Swing1.0，拖放和Java2D类库
  7.引入Java插件
  8.JDBC中引入可滚动结果集，BLOB,CLOB,批量更新和用户自定义类型
  9.Applet中添加声音支持

# JAVA1.3，代号Kestrel（美洲红隼）
  2000-5-8发布，主要更新内容：
  1.引入Java Sound API
  2.引入jar文件索引
  3.对Java各方面多了大量优化和增强
  4.Java Platform Debugger Architecture用于 Java 调式的平台。

# JAVA 1.4，代号Merlin（灰背隼）
  2004-2-6发布（首次在JCP下发行），主要更新内容：
  1.添加XML处理
  2.添加Java打印服务（Java Print Service API）
  3.引入Logging API
  4.引入Java Web Start
  5.引入JDBC 3.0 API
  6.引入断言
  7.引入Preferences API
  8.引入链式异常处理
  9.支持IPV6
  10.支持正则表达式
  11.引入Image I/O API
  12.NIO，非阻塞的 IO，优化 Java 的 IO 读取。

# JDK 1.5，代号Tiger（老虎），有重大改动
  2004-9-30发布，主要更新内容：
  1.引入泛型
  2.For-Each循环 增强循环，可使用迭代方式
  3.自动装箱与自动拆箱
  4.引入类型安全的枚举
  5.引入可变参数
  6.添加静态引入
  7.引入注解
  8.引入Instrumentation
  9.提供了 java.util.concurrent 并发包。

# JDK 1.6，代号Mustang（野马）
  2006-12-11发布，主要更新内容：
  1.引入了一个支持脚本引擎的新框架（基于 Mozilla Rhino 的 JavaScript 脚本引擎）
  2.UI的增强
  3.对WebService支持的增强（JAX-WS2.0 和 JAXB2.0）
  4.引入JDBC4.0API
  5.引入Java Compiler API
  6.通用的Annotations支持

# JDK 1.7，代号Dolphin（海豚）
  2011-07-28发布，这是sun被oracle收购（2009年4月）后的第一个版本，主要更新内容：
  1.switch语句块中允许以字符串作为分支条件
  2.在创建泛型对象时应用类型推断,比如你之前版本使用泛型类型时这样写 
    ArrayList<User> userList= new ArrayList<User>();
    这个版本只需要这样写 ArrayList<User> userList= new ArrayList<>();，也即
    是后面一个尖括号内的类型，JVM 帮我们自动类型判断补全了。
  3.在一个语句块中捕获多种异常
  4.添加try-with-resources语法支持，使用文件操作后不用再显示执行close了。
  5.支持动态语言
  6.JSR203, NIO.2,AIO,新I/O文件系统，增加多重文件的支持、文件原始数据和符号链接,支持ZIP文件操作
  7.JDBC规范版本升级为JDBC4.1
  8.引入Fork/Join框架，用于并行执行任务
  9.支持带下划线的数值，如 int a = 100000000;，0 太多不便于人阅读，这个版本支持这样写 
    int a = 100_000_000，这样就对数值一目了然了。
  10.Swing组件增强（JLayer,Nimbus Look Feel…）参考

# JDK 1.8  代号Spider（蜘蛛）【长期支持版本】 LTS
  2014-3-19发布，oracle原计划2013年发布，由于安全性问题两次跳票，是自JAVA5以来最具革命性的
  版本，主要更新内容：
  1.接口改进，接口居然可以定义默认方法实现和静态方法了。
  2.引入函数式接口
  3.引入Lambda表达式
  4.引入全新的Stream API，提供了对值流进行函数式操作。
  5.引入新的Date-Time API
  6.引入新的JavaScrpit引擎Nashorn
  7.引入Base64类库
  8.引入并发数组（parallel）
  9.添加新的Java工具：jjs、jdeps
  10.JavaFX，一种用在桌面开发领域的技术
  11.静态链接 JNI 程序库

# JDK 9  2017-9-21发布
  1.模块化（jiqsaw）
  2.交互式命令行（JShell）
  3.默认垃圾回收期切换为G1
  4.进程操作改进
  5.竞争锁性能优化
  6.分段代码缓存
  7.优化字符串占用空间

# JDK 10  2018-3-21发布
  1.JEP286，var 局部变量类型推断。
  2.JEP296，将原来用 Mercurial 管理的众多 JDK 仓库代码，合并到一个仓库中，简化开发和管理过程。
  3.JEP304，统一的垃圾回收接口。
  4.JEP307，G1 垃圾回收器的并行完整垃圾回收，实现并行性来改善最坏情况下的延迟。
  5.JEP310，应用程序类数据 (AppCDS) 共享，通过跨进程共享通用类元数据来减少内存占用空间，
    和减少启动时间。
  6.JEP312，ThreadLocal握手交互。在不进入到全局 JVM 安全点(Safepoint) 的情况下，对线程执行
    回调。优化可以只停止单个线程，而不是停全部线程或一个都不停。
  7.JEP313，移除 JDK 中附带的 javah 工具。可以使用 javac -h 代替。
  8.JEP314，使用附加的 Unicode 语言标记扩展。
  9.JEP317，能将堆内存占用分配给用户指定的备用内存设备。
  10.JEP317，使用 Graal 基于 Java 的编译器，可以预先把 Java 代码编译成本地代码来提升效能。
  11.JEP318，在 OpenJDK 中提供一组默认的根证书颁发机构证书。开源目前 Oracle 提供的的 Java SE 
     的根证书，这样 OpenJDK 对开发人员使用起来更方便。
  12.JEP322，基于时间定义的发布版本，即上述提到的发布周期。版本号为
     \$FEATURE.\$INTERIM.\$UPDATE.\$PATCH，分别是大版本，中间版本，升级包和补丁版本。

# JDK 11  2018-9-25发布【长期支持版本】
  官网公开的 17 个 JEP（JDK Enhancement Proposal 特性增强提议）：
  1.JEP181: Nest-Based Access Control（基于嵌套的访问控制）
  2.JEP309: Dynamic Class-File Constants（动态的类文件常量）
  3.JEP315: Improve Aarch64 Intrinsics（改进 Aarch64 Intrinsics）
  4.JEP318: Epsilon: A No-Op Garbage Collector（Epsilon 垃圾回收器）
  5.JEP320: Remove the Java EE and CORBA Modules（移除 Java EE 和 CORBA 模块，JavaFX 
    也已被移除）
  6.JEP321: HTTP Client (Standard)
  7.JEP323: Local-Variable Syntax for Lambda Parameters（用于 Lambda 参数的局部变量语法）
  8.JEP324: Key Agreement with Curve25519 and Curve448（采用 Curve25519 和 Curve448 算法
    实现的密钥协议）
  9.JEP327: Unicode 10
  10.JEP328: Flight Recorder（飞行记录仪）
  11.JEP329: ChaCha20 and Poly1305 Cryptographic Algorithms（实现 ChaCha20 和 Poly1305 
     加密算法）
  12.JEP330: Launch Single-File Source-Code Programs（启动单个 Java 源代码文件的程序）
  13.JEP331: Low-Overhead Heap Profiling（低开销的堆分配采样方法）
  14.JEP332: Transport Layer Security (TLS) 1.3（对 TLS 1.3 的支持）
  15.JEP333: ZGC: A Scalable Low-Latency Garbage Collector (Experimental)（ZGC：可伸缩的
     低延迟垃圾回收器，处于实验性阶段）
  16.JEP335: Deprecate the Nashorn JavaScript Engine（弃用 Nashorn JavaScript 引擎）
  17.JEP336: Deprecate the Pack200 Tools and API（弃用 Pack200 工具及其 API）

# JDK 12  2019-3-19发布
  1.JEP189:Shenandoah: A Low-Pause-Time Garbage Collector (Experimental)
  2.JEP230:Microbenchmark Suite
  3.JEP325:Switch Expressions (Preview)
  4.JEP334:JVM Constants API
  5.JEP340:One AArch64 Port, Not Two
  6.JEP341:Default CDS Archives
  7.JEP344:Abortable Mixed Collections for G1
  8.JEP346:Promptly Return Unused Committed Memory from G1

# JDK 13  2019-9-17 发布
  1.JEP350:Dynamic CDS Archives
  2.JEP351:ZGC: Uncommit Unused Memory
  3.JEP353:Reimplement the Legacy Socket API
  4.JEP354:Switch Expressions
  5.JEP355:Text Blocks

# JDK 14 2020-3-17 发布【长期支持版本】
  1.JEP305: instanceof的模式匹配 (预览)
  2.JEP361: Switch表达式 (标准)
  3.JEP368: Text Blocks（二次预览）  
  4.JEP343: Java打包工具（孵化项目）  
  5.JEP358: 友好的空指针异常
  6.JEP359: Records记录类型 (预览
  7.JEP352: 非易失性映射字节缓冲区
  8.JEP345: G1的NUMA内存分配优化
  9.JEP349: JFR事件流
  10.JEP370: 外部存储器API（孵化）
  11.JDK14的其他新特性
  
# JDK 15 将在2020-09-17日出release
~~~



### 1.4 Java虚拟机发展史

JVM官方文档:https://docs.oracle.com/javase/specs/jvms/se8/html/index.html

上面我们从整个Java技术的角度观察了Java技术的发展，接下来我们在看看Java虚拟机的发展史，提到Java虚拟机很多人都会想到 Sun公司的HotSpot虚拟机，但实际上JVM的实现还有很多:

+ **Sun classic/Exact VM** 

  ```txt
  Sun Classic VM的技术可能很原始，而且这款虚拟机的使命也早已终结。但仅凭它"世界上第一款商用Java虚拟机"的头衔就应该被我们所记住。
  
  Exact VM在第一代虚拟机的基础上做了大量优化，在JDK1.2中使用，但只存在了很短的时间就被更优秀的HotSpot VM所取代。
  ```

+ **Sun HotSpot VM (重点)** 

  ```txt
  它是Sun JDK和Open JDK中所带的虚拟机，也是目前使用范围最广的Java虚拟机。这款虚拟机是在1997年收购获得，并经过不断优化继承了Java前两任虚拟机的优点也有自己的优势。 在2006年 JavaOne大会上，Sun公司宣布最终会把Java开源，在接下来的一年java在GPL协议下公开源码，在此基础上建立了OpenJDK。
  ```

+ **Sun Mobile-Embedded VM/ Meta-Circular VM**

  ```txt
  Sun公司所研发的虚拟机可不仅前面介绍的服务器、桌面领域的虚拟机，面对移动端和嵌入式市场 也创建了对应的虚拟机。
  ```

+ **BEA JRockit / IBM J9 VM**

  ```txt
  前面介绍了Sun公司的各种虚拟机，除了Sun公司外，其他组织、公司也研发过不少虚拟机实现，其中规模最大的就属BEA和IBM公司
  
  JRockit和J9都是号称世界上最快的虚拟机，J9主要用于运行IBM旗下的java产品，功能和产品方向与HotSpot虚拟机类似。
  
  而JRockit 主要专注于服务器端应用，JRockit的垃圾收集器在众多Java虚拟机实现中也一直处于领先水平
  不过2008~2009 BEA 和 Sun相继被Oracle公司收购，Oracle一下就掌握了世界上最先进的两款虚拟机，并在后续对两款虚拟机进行整合，将JRockit的优秀特质移植到HotSpot虚拟机中。
  ```



**国产JVM**

**Alibaba Dragonwell8 JDK 开源地址：** https://github.com/alibaba/dragonwell8 

**TencentKona-8 开源地址：**https://github.com/Tencent/TencentKona-8



## 02、JVM：组成部分

+ 第一部分: 类加载器（ClassLoader）
+ 第二部分: 运行时数据区（Runtime Data Area）
+ 第三部分: 执行引擎（Execution Engine）
+ 第四部分: 本地库接口（Native Interface）

![image-20200416214837483](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200416214837483.png) 

> 首先通过类加载器（ClassLoader）把 Java 代码转换成字节码，运行时数据区（Runtime Data Area）再把字节码加载到内存中，而字节码文件只是JVM的一套指令集规范，并不能直接交给底层操作系统去执行，因此需要特定的命令解析器执行引擎（Execution Engine），将字节码翻译成底层系统指令，再交由CPU去执行，而这个过程中需要调用其他语言的本地库接口（Native Interface）来实现整个程序的功能。





## 03、JVM：类加载器介绍

### 3.1 认识字节码文件

javap 反汇编，可以看到class文件中的字节码指令



### 3.2 类加载器的作用

将class文件加载进JVM的方法区，并创建一个Class对象。



### 3.3 类加载器的分类

![image-20200417210511437](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417210511437.png) 

**启动类加载器(Bootstrap ClassLoader, 又称根加载器)**  

```
每次执行 java 命令时都会使用该加载器为虚拟机加载核心类。
该加载器是由 native code 实现，而不是 Java 代码，加载类的路径为 <JAVA_HOME>/jre/lib。
特别的 <JAVA_HOME>/jre/lib/rt.jar 中包含了 sun.misc.Launcher 类，
而 sun.misc.Launcher$ExtClassLoader 和 sun.misc.Launcher$AppClassLoader 都是
sun.misc.Launcher的内部类，所以拓展类加载器和系统类加载器都是由启动类加载器加载的。
```

**扩展类加载器(Extension ClassLoader)**  

```
用于加载拓展库中的类。拓展库路径为<JAVA_HOME>/jre/lib/ext/。
实现类为: sun.misc.Launcher$ExtClassLoader。
```

**应用程序类加载器(System ClassLoader)**  

```
负责加载用户classpath下的class文件，又叫系统加载器，其父类是Extension。
它是应用最广泛的类加载器。它从环境变量classpath或者系统属性java.class.path所指定的目录中记载类，是用户自定义加载器的默认父加载器。
实现类为: sun.misc.Launcher$AppClassLoader
```



### 3.4 双亲委派模型

如果一个类加载器收到了加载类的请求，它首先将请求交由父类加载器加载；若父类加载器加载失败，当前类加载器才会自己加载类。

![image-20200417210719026](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417210719026.png) 

> 作用: 像java.lang.Object这些存放在rt.jar中的类，无论使用哪个类加载器加载，最终都会委派给最顶端的启动类加载器加载，从而使得不同加载器加载的Object类都是同一个。



**双亲委派模型的代码在java.lang.ClassLoader类中的loadClass()方法中实现：**  

```java
protected Class<?> loadClass(String name, boolean resolve) 
    throws ClassNotFoundException {
    synchronized (getClassLoadingLock(name)) {
        // 首先，检查类是否被加载
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                // 若未加载，则调用父类加载器的loadClass方法
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    // 没有父类则调用 BootstrapClassLoader
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // 若该方法抛出ClassNotFoundException异常，表示父类加载器无法加载
            } 
            if (c == null) {
                long t1 = System.nanoTime();
                // 则当前类加载器调用findClass加载类
                c = findClass(name);
                // 定义类加载，记录数据
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        } 
        if (resolve) {
            resolveClass(c);
        } 
        return c;
    }
}
```



**小结**  

```
1.首先检查类是否被加载。
2.若未加载，则调用父类加载器的loadClass方法。
3.如果没有父类，则调用BootstrapClassLoader类加载。
4.若父类加载器BootstrapClassLoader和ExtClassLoader也无法加载，调用AppClassLoader的ClassLoader类的findClass方法加载类。
5.若父类加载器可以加载，则直接返回Class对象。
```





### 3.5 自定义类加载器(Custom ClassLoader)

```java
package cn.itcast.jvm;

import java.io.IOException;
import java.io.InputStream;

/**
 * 自定义类加载器
 */
public class ClassLoaderTest {

    public static void main(String[] args) throws Exception {
        // 自定义类加载器
        ClassLoader myLoader = new MyClassLoader();

        // 使用自定义类加载器加载出来的Class对象
        Class c1 = myLoader.loadClass("cn.itcast.jvm.ClassLoaderTest");
        // 使用系统默认的类加载器加载出来的Class对象
        Class c2 = ClassLoaderTest.class;
        // 结果为false
        System.out.println(c1 == c2);
        System.out.println("c1.hashCode(): " + c1.hashCode());
        System.out.println("c2.hashCode(): " + c2.hashCode());
    }
}

class MyClassLoader extends ClassLoader {
    @Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        try {
            // 获取类的class名称: ClassLoaderTest.class
            String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
            // 获取ClassLoaderTest.class文件对应的输入流
            InputStream in = getClass().getResourceAsStream(fileName);
            // 如果不是空
            if(in == null){
                return super.loadClass(name);
            }
            // 定义该class文件对应的字节数组大小(这个数组刚好可以容纳整个字节码文件的数据)
            byte[] b = new byte[in.available()];
            // 读入字节数组中
            in.read(b);
            // 定义class对象
            return defineClass(name, b, 0, b.length);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return super.loadClass(name);
    }
}
```





## 04、JVM：类加载的生命周期

- 负责将 class 加载到 JVM 中

- 审查每个类由谁加载（父优先的等级加载机制）

- 将 Class 字节码重新解析成 JVM 统一要求的对象格式  

  ![image-20200417205706450](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417205706450.png) 

### 4.1 类加载的生命周期

类从被加载到虚拟机内存中开始，直到卸载出内存为止，它的整个生命周期包括了：加载、验证、准备、解析、初始化、使用和卸载这7个阶段。其中，验证、准备和解析这三个部分统称为连接（linking）。

![image-20200417205746471](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417205746471.png) 



### 4.2 类加载的过程详解

#### 加载

```
基本概念: 该过程完成查找并加载类的class文件。该class文件可以来自本地磁盘或者网络等。

虚拟机类加载的第一个阶段，在这个阶段中，虚拟机要完成3件事情:
    1.获取类的二进制字节流。
    2.将字节流的静态数据结构转化为方法区的运行时数据结构。
    3.在内存中生成一个代表这个类的java.lang.Class对象。

而获取类的二进制流又有很多种方式:
    从压缩包中获取，比如 JAR包、EAR、WAR包等
    从网络中获取，比如红极一时的Applet技术
    从运行过程中动态生成，最出名的便是动态代理技术，在java.lang.reflect.Proxy中，
    就是用了ProxyGenerator.generateProxyClass 来为特定接口生成形式为“$Proxy”的代理类的二进制流
    从其它文件生成，如JSP文件生成Class类
```

#### 验证

```
基本概念: 确保类型的正确性，比如class文件的格式是否正确、是否符合语法规定、字节码是否可以被JVM安全执行等

验证总体上分为4个阶段: 文件格式验证、元数据验证、字节码验证、符号引用验证。
```

#### 准备

基本概念: 为类的静态变量分配内存，并赋初值。基本类型的变量赋值为初始值，比如int类型的赋值为0，引用类型赋值为null。

初始值一般为 0 值，例如下面的类变量 value 被初始化为 0 而不是 123。

```java
public static int value = 123;
```

如果类变量是常量，那么它将初始化为表达式所定义的值而不是 0。例如下面的常量 value 被初始化为 123 而不是 0。

```java
public static final int value = 123;
```

#### 解析

```
基本概念: 将符号引用转为直接引用。比如方法中调用了其他方法，方法名可以理解为符号引用，而直接引用就是使用指针直接引用方法。

”解析“阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，主要针对 类或接口、字段、类方法、接口方法、方法类型、方法句柄 和 调用限定符 7类符号引用过程。
```

#### 初始化

```
基本概念: 初始化，则是为标记为常量值的字段赋值的过程。换句话说，只对static修饰的变量或静态代码块进行初始化。如果初始化一个类的时候，其父类尚未初始化，则优先初始化其父类。如果同时包含多个静态变量和静态代码块，则按照自上而下的顺序依次执行。
```

```java
public class Test {
    private static final int a;
    
    static {
        a = 20;
    }
}
```



查看程序加载的类:

```
-verbose:class
```







## Visual GC：插件安装

参见《18、JVM性能监控：JDK可视化工具》





## 05、Java运行时数据区：介绍

对于Java程序员来说，在虚拟机 **自动内存管理机制** 的帮助下，不再需要为每一个new的操作去写对应的内存管理操作，不容易出现内存泄漏和内存溢出问题，由虚拟机管理内存一切都看起来很美好。不过，也正是因为我们把内存控制的权利交给了虚拟机，一旦出现内存泄漏和溢出方面的问题，**如果不了解虚拟机是怎么样使用内存的，那么排查错误将会变得特别的困难**。 所以，接下来我们要一层一层了解JVM内存管理机制它究竟是怎样实现的。



![image-20210118182603464](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20210118182603464.png)

**运行时数据区 (Runtime Data Area)【重点】**:

Java虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域。这些区域都有各自的用途，以及创建和销毁的时间。根据1.7 java虚拟机规范，JVM运行时数据区主要划分为以下几个部分:

![image-20200416214955376](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200416214955376.png)  

**注意：方法区与 Java堆 是所有线程共享的。**

​     **Java虚拟机栈、本地方法栈、程序计数器是线程私有的。**

> 说明: jdk1.8+ 方法区改名叫元数据区(Metaspace)。





## 06、Java运行时数据区：方法区

+ 方法区(Method Area) 

+ 【jdk1.7-】 永久代(Prem)

+ 【jdk1.8+】 元空间(Metaspace)

  ![1597309805162](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597309805162.png) 

  **方法区是线程共享的区域**。存储已被虚拟机加载的类元信息、方法元信息、常量池。方法区无法满足内存分配需求时，抛出OutOfMemoryError。JVM规范并没对这个区域实现垃圾收集，因为这个区域回收主要针对的是类信息、方法信息、常量池的信息回收，回收结果往往难以令人满意。

  

  **常量池: **是方法区的一部分。Java语言不要求常量只能在编译期产生，换言之，在运行期间也能将新的常量放入。

  > jdk1.7 方法区内存大小设置:   -XX:PermSize -XX:MaxPermSize
  > jdk1.8 元数据区内存大小设置: -XX:MetaspaceSize -XX:MaxMetaspaceSize

  代码演示(元空间内存溢出):

  ```java
  package cn.itcast.jvm;
  
  import net.sf.cglib.proxy.Enhancer;
  import net.sf.cglib.proxy.MethodInterceptor;
  import net.sf.cglib.proxy.MethodProxy;
  
  import java.lang.reflect.Method;
  
  /**
   * 方法区|元数据区(内存控制)
   * VM Args: -XX:PermSize=50m -XX:MaxPermSize=50m 【永久代】
   * VM Args: -XX:MetaspaceSize=50m -XX:MaxMetaspaceSize=50m 【元空间】
   **/
  
  public class MetaspaceTest {
  
      public static void main(final String[] args) throws Exception {
          while (true) {
              Thread.sleep(5);
              // 创建字节码增强器对象
              Enhancer enhancer = new Enhancer();
              // 设置父类
              enhancer.setSuperclass(Object.class);
              // 设置是否使用缓存
              enhancer.setUseCache(false);
              // 设置回调对象(方法拦截器)
              enhancer.setCallback(new MethodInterceptor() {
                  public Object intercept(Object o, Method method, Object[] objects,
                                          MethodProxy methodProxy) throws Throwable {
                      return methodProxy.invoke(o, args);
                  }
              });
              enhancer.create();
          }
    }
  }
  ```

  ```java
  Exception in thread "main" java.lang.OutOfMemoryError: Metaspace
  	at net.sf.cglib.core.AbstractClassGenerator.create(AbstractClassGenerator.java:237)
  	at net.sf.cglib.proxy.Enhancer.createHelper(Enhancer.java:377)
  	at net.sf.cglib.proxy.Enhancer.create(Enhancer.java:285)
  at cn.itcast.jvm.MetaspaceTest.main(MetaspaceTest.java:26)
  
  ```

Process finished with exit code 1

![1597627556672](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597627556672.png) 

  > cglib动态代理可以动态创建代理类，这些代理类的Class会动态的加载入内存中，存入到方法区。所以当我们把方法区内存调小后便可能会产生方法区内存溢出,1.8之前的JDK我们可以称方法区为永久代 :PermSpace  1.8之后方法区改为 MetaSpace 元空间。





## 07、Java运行时数据区：Java堆

![image-20210118205846529](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20210118205846529.png)



**Java堆(Heap):**

![1597311848427](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597311848427.png) 

Java 中的堆是 JVM 所管理的最大的一块内存空间，主要用于存放各种类的实例对象。在 Java 中，堆被划分成两个不同的区域：新生代(Young)、老年代 (Old)。新生代 (Young) 又被划分为三个区域：`Eden`、`From Survivor`、`To Survivor`。 这样划分的目的是为了使JVM 能够更好的管理堆内存中的对象，包括内存的分配以及回收。 

![1597803655488](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597803655488.png)



> Java Heap(堆大小) = 新生代 + 老年代 【新生代 与 老年代 的比例的值为 1:2】
>
> 新生代 = Eden + Survivor(S0) + Survivor(S1) 【Eden:S0:S1 = 8:1:1】
>
> ​        Eden:伊甸园  S0(Survivor 0):幸存者0  S1(Survivor 1):幸存者1

  

  ```
常说的“栈内存”、“堆内存”，其中前者指的是虚拟机栈，后者说的就是Java堆了。Java堆是被线程共享的。在虚拟机启动时被创建。Java堆是Java虚拟机所管理的内存中最大的一块。Java堆的作用是存放对象实例。

堆空间内存大小设置: 
-Xms300m                    Java堆初始化内存
-Xmx300m                    Java堆最大内存 xlage

-Xmn15m                    新生代内存大小
-XX:SurvivorRatio=8        新生代Eden/Survivor空间的初始比例
-XX:NewRatio=2             Old区 和 Yong区 的内存比例
  ```

> 查看默认新生代与老年代内存比例：java -XX:+PrintFlagsFinal -version  | grep NewRatio
>
> 由于Windows没有grep命令，可以将数据写入文件在查看：java -XX:+PrintFlagsFinal -version > xxx.txt



![1597827299369](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597827299369.png) 

代码演示(堆内存溢出):

```java
package cn.itcast.jvm;


import java.util.ArrayList;
import java.util.List;

/**
* 堆内存溢出演示
* VM Args: -Xms20m -Xmx20m -XX:NewRatio=1 -XX:SurvivorRatio=1
**/
public class HeapTest {

    public static void main(String[] args) throws Exception {
        // 创建List集合
        List<byte[]> list = new ArrayList<byte[]>();
        // 循环往集合中添加元素
        while (true){
            Thread.sleep(1);
        	list.add(new byte[1024]);
        }
    }
}
```

```shell
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at cn.itcast.jvm.HeapTest.main(HeapTest.java:16)

Process finished with exit code 1
```


![1597827636119](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597827636119.png) 





## 08、Java运行时数据区：Java虚拟机栈

**Java虚拟机栈(JVM Stacks) 或 Java栈:**

![1597309639325](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597309639325.png)  

+ Java虚拟机栈是**线程私有**的，它的生命周期与线程相同（随线程而生，随线程而灭）。

  > **栈帧存储**: 局部变量表、操作数栈、动态链接、方法返回地址。每一个方法从调用到结束，就对应这一个栈帧在虚拟机栈中的进栈和出栈过程。
  >
  > **局部变量表**: 8种基本数据类型、对象引用（reference类型）、returnAddress类型（一条字节码地址）。

+ 如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常。

+ 每个新创建的线程都要分配一个虚拟机栈，如果新创建的线程无法申请到足够的栈内存，就会抛出OutOfMemoryError异常。

  > 栈的内存大小设置: -Xss128k 为jvm启动的每个线程分配的内存大小。

  ![1597822404030](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597822404030.png) 



**线程请求的栈深度大于虚拟机所允许的深度，抛出StackOverflowError:**

```java
package cn.itcast.jvm;

/**
* 栈超出最大深度：StackOverflowError
* VM args: -Xss128k
**/
public class StackOverflowTest {
    // 定义变量，记录栈的深度
    private static int stackLength = 1;
    // 定义方法
    public static void stackLeak(){
        stackLength++;
        stackLeak();
    }

    public static void main(String[] args) {
        try {
            // 调用方法
            stackLeak();
        } catch (Throwable e) {
            System.out.println("当前栈深度:" + stackLength);
            e.printStackTrace();
        }
    }
}
```

```java
当前栈深度:995
java.lang.StackOverflowError
	at cn.itcast.jvm.StackOverflowTest.stackLeak(StackOverflowTest.java:10)
	at cn.itcast.jvm.StackOverflowTest.stackLeak(StackOverflowTest.java:11)
	at cn.itcast.jvm.StackOverflowTest.stackLeak(StackOverflowTest.java:11)
    
方法递归调用，造成栈内存不足，产生栈溢出错误异常
```



**虚拟机栈扩展时无法申请到足够的内存，抛出OutOfMemoryError:**（代码谨慎使用，会引起电脑卡死）

```java
package cn.itcast.jvm;

/**
* 栈内存溢出 (不测试)
* VM Args: -Xss128k
**/
public class StackOutOfMemoryTest {
    private void doStop(){
        while (true){
        }
    }
    public void stackLeakByThread(){
        while(true){
            new Thread(new Runnable() {
                public void run() {
                    doStop();
                }
            }).start();
        }
    }
    public static void main(String[] args) {
        StackOutOfMemoryTest stack = new StackOutOfMemoryTest();
        stack.stackLeakByThread();
    }
}
```

```java
Exception in thread "main" java.lang.OutOfMemoryError:unable to create new native thread
```

单线程下栈的内存出现问题了，都是报StackOverflow的异常，只有在多线程的情况下，当新创建线程无法在分配到新的栈内存资源时，会报内存溢出。





## 09、Java运行时数据区：本地方法栈

**本地方法：**是指C/C++写的代码。



**本地方法栈(Native Method Stacks):**

![1597309563107](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597309563107.png) 

> 本地方法栈（Native Method Stacks）与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的Native方法服务。本地方法栈也会抛出 StackOverflowError、OutOfMemoryError。

![1597819798492](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597819798492.png) 





## 10、Java运行时数据区：程序计数器

**程序计数器(Program Counter Register):**

![1597312213944](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597312213944.png) 

> 1. 线程私有。可看作是**当前线程所执行的字节码的行号记录器**，字节码解释器的工作是通过改变这个计数值来读取下一条要执行的字节码指令。
> 2. Java虚拟机的多线程是通过**线程轮流切换**并分配处理器执行时间的方式来实现，也就是说，在同一时刻一个处理器内核只会执行一条线程。**为了线程切换后能恢复到正确的执行位置，每条线程都需要一个独立的程序计数器**。这就是一开始说的“线程私有”。如果线程正在执行的方法是Java方法，计数器记录的是虚拟机字节码的指令地址；如果是Native方法，计数器值为空。

> **程序计数器是唯一一个在Java虚拟机规范中没有规定OOM(OutOfMemoryError)情况的区域。**



#### Java运行时数据区小结

![image-20200416214955376](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200416214955376.png)





## 11、JVM：对象探秘

介绍完JVM的运行时数据区之后，我们大致知道了虚拟机内存的概况，接下来我们以对象的创建为例，了解下虚拟机内存中的数据的一些细节。 我们以HotSpot虚拟机和Java堆为例，深入了解虚拟机在Java堆中对象分配、布局和访问的全过程。

  

![image-20210118205846529](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20210118205846529.png)



### 11.1 对象的创建

从我们写代码的角度上来看，一个new 关键字就可以把对象创建出来，而从虚拟机的角度上来看，对象的创建是怎样一个过程呢？  

1. JVM遇到new的指令时，首先会去检查这个指令的参数是否能在常量池中定位到类的信息
2. 如果定位到会检查对应的类是否已经加载、解析和初始化
3. JVM为新生对象分配内存（在Java堆中划分一片区域），内存的大小在类加载完毕后便可确定
4. 分配好后,JVM将会对分配到的内存空间都初始化为零值(不包括对象头)
5. JVM对对象进行设置，如：对象所属类、哈希码、GC分代年龄 这些信息都存放到对象头中
6. 从JVM的视角看，一个新的对象诞生了，但从我们的角度看 创建对象刚刚开始，因为对象的所有值都是零值，按我们代码所设计的进行初始化。



### 11.2 对象的内存布局

在HotSpot虚拟机中，对象在内存中存储的布局可以分为3块区域：对象头(Header)、实例数据(Instance Data)、和对齐填充(Padding) 

![1597287419807](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20201117121306779.png) 

+ **对象头**（Header）

  > **存储对象自身的运行时数据**，比如**哈希码**、**GC分代年龄**、**锁状态标志**、**线程持有的锁**、**偏向线程ID**、**数组长度** 等。另外还有一部分是**类型指针**，即对象指向它的类元数据的指针，虚拟机通过该指针来确定这个对象属于哪个类的实例。
  >
  > ![image-20210112172402024](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20210112172402024.png)

+ **实例数据**（Instance Data）

  > **实例数据部分是对象真正存储的有效信息，也是在程序代码中定义的各种的属性值**。无论是从父类继承下来的，还是在子类中定义的，都需要记录起来。这部分的存储顺序会受到虚拟机分配策略参数(FieldsAllocationStyle)和字段在Java源码中定义顺序的影响。HostSpot虚拟机默认的分配策略为longs/doubles、ints、shorts/chars、bytes/booleans、oops(Ordinary Object Pointers)，从分配策略中可以看出，相同宽度的字段总是被分配到一起。在满足这个前提条件的情况下，在父类中定义的变量会出现在子类之前。如果CompactFields参数值为true(默认为true)，那么子类之中较窄的变量也可能会插入到父类变量之中。

+ **对齐填充**（Padding）

  > 对齐填充：**对齐填充并不是必然存在的，也没有特殊的含义，它仅仅起着占位符的作用。** 

>由于HotSpot虚拟机的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说就是**任何对象的大小都必须是8字节的整数倍。** 因此，如果对象实例数据部分没有对齐的话，就需要通过对齐填充来补全。 

  ![1597742802432](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597742802432.png)  

  


### 11.3 使用JOL查看对象布局

添加pom依赖

```xml
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.9</version>
</dependency>
```

示例: 

User.java

```java
package cn.itcast.jvm;

/**
 Instance size: 32 bytes
 Space losses: 2 bytes internal + 0 bytes external = 2 bytes total
 */
public class User {
    // 占 8字节
    private long id = 1L;
    // 占 4字节
    private String name = "李明";
    // 占 2字节
    private char sex = '男';
    // 占 4字节
    private int age = 20;
}
```



ObjectLayoutTest.java

```java
public class ObjectLayoutTest {
    public static void main(String[] args) throws IOException {
        User p1 = new User();
        System.out.println(ClassLayout.parseInstance(p1).toPrintable());

        System.in.read();
        System.out.println("结束!");
    }
}
```



对于64位HotSpot VM引用类型占用8字节，从JDK 1.6 update14开始，64位的JVM正式支持了指针压缩，可以压缩指针，引用类型占用4字节，起到节约内存占用。JDK 1.8中默认该参数就是开启的。关闭指针压缩，在JVM启动参数中添加-XX:-UseCompressedOops。



运行效果

![image-20210112173152128](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20210112173152128.png)



### 11.4 对象的访问定位

Java程序通过栈上的reference数据来操作堆上的具体对象。由于reference类型在Java虚拟机规范中只规定了一个指向对象的引用，并没有定义这个引用应该通过何种方式去定位、访问堆中的对象的具体位置，所以对象访问方式也是取决于虚拟机实现而定的。目前主流的访问方式有使用**句柄**和**直接指针**两种。  



**句柄的方式：**

Java堆中会划分出一块内存来作为句柄池，reference中存储的就是对象的句柄地址，而句柄中包含了对象实例数据和类型数据各自的具体地址信息。使用句柄方式最大的好处就是reference中存储的是**稳定**的句柄地址，在对象被移动（垃圾收集时移动对象是非常普遍的行为）时只会改变句柄中的实例数据指针，而reference本身不需要被修改。 

![image-20200417194052052](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417194052052.png)



**直接指针的方式：**

Java堆对象的布局就必须考虑如何放置访问类型数据的相关信息，reference中直接存储的就是对象地址。使用直接指针方式最大的好处就是**速度更快**，他**节省了一次指针定位的时间开销**。 

![image-20200417194111647](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417194111647.png)

**句方式柄与直接指针方式的对比：** 使用句柄来访问的最大好处就是reference中存储的是稳定的句柄地址，在对象被移动(垃圾收集时移动对象是非常普遍的行为)时只会改变句柄中的实例数据指针，而reference本身不需要修改;使用直接指针访问方式的最大好处就是速度更快，它节省了一次指针定位的时间开销，由于对象的访问在Java中非常频繁，因此这类开销积少成多后也是一项非常可观的执行成本。

**注：就HotSpot而言，它是使用的直接指针的方式进行对象访问的。**





## 12、GC：垃圾收集介绍

+ 说起垃圾收集（Garbage Collection, GC），大家肯定都不陌生，目前内存的动态分配与内存回收技术已经非常成熟，那么我们为什么还要去了解GC和内存分配呢?原因很简单：当需要排查各种内存溢出、内存泄漏问题时，当垃圾收集成为系统达到更高并发量的瓶颈时。 我们就需要对这些自动化的技术实施必要的监控和条件。
+ 在Java运行时内存当中，程序计数器、Java虚拟机栈、本地方法栈3个区域都是随线程生而生，随线程灭而灭，因此这个区域的内存分配和回收都具备了确定性，所以在这几个区域不需要太多的考虑垃圾回收问题，因为方法结束了，内存自然就回收了。但Java堆不一样，它是所有线程共享的区域，我们只有在程序处于运行期间时才能知道会创建哪些对象，这个区域的内存分配和内存回收都是动态的，所以垃圾收集器主要关注的就是这部分的内存。
+ 关于垃圾回收的知识点，我们会从以下几方面去讲解:
  + 什么样的对象需要回收
  + 垃圾收集的算法（如何回收）
  + 垃圾收集器（谁来回收）



在Java堆中存放着Java世界中几乎所有的对象实例，垃圾收集器在对堆进行回收前，第一件事情就是要确定这些对象之中哪些还存活着，哪些已经死去（即不可能在被任何途径使用的对象）如何判断对象是否存活，主要有两种算法: **引用计数法** 和 **可达性分析算法**。

> 主流的商业虚拟机基本使用的是: **可达性分析算法**



### 12.1 引用计数法

![image-20210118220812166](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20210118220812166.png)



循环引用问题

![image-20210118220829194](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20210118220829194.png)



### 12.2 可达性分析算法

~~~txt
基本思路: 通过以 GC Roots 的对象作为起始点，从这些节点开始向下搜索，搜索走过的路径成为引用连，当一个对象与GC Roots没有任何引用链时，则证明此对象是不可达的，会被判断为可回收的对象。

如图: 对象5,6,7 虽然互相有关联，但是他们到GC Roots是不可达的，所以它们会被判定为可回收的对象。
~~~

![image-20200417202851071](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417202851071.png) 

**GC Roots节点的对象**

+ 虚拟机栈中 (栈帧中的本地变量表) 引用的对象
+ 方法区中类静态成员变量引用的对象
+ 方法区中常量的引用对象
+ 本地方法栈中引用的对象



### 12.3 四种对象的引用

无论是通过引用计数算法判断对象的引用数量，还是通过可达性分析判断对象的引用链是否可达，判断对象的存活都与引用有关，在JDK1.2之后，java对引用进行了细分，用于描述一些特殊的对象，如: 当内存空间还足够时，就保留在内存之中，如果内存空间在进行垃圾收集后还是很紧张则抛弃这些对象，所以java又对引用进行了一系列的划分: **强引用**、**软引用**、**弱引用**、**虚引用**  

#### 12.3.1 强引用

~~~
Java中默认声明的就是强引用，比如:

List<Object> list = new ArrayList<Object>();
list.add(new Object());

Object obj = new Object(); // 只要obj还指向Object对象，Object对象就不会被回收
obj = null; // 手动置null
只要强引用存在，垃圾回收器将永远不会回收被引用的对象，哪怕内存不足时，JVM也会直接抛出OutOfMemoryError，不会去回收。如果想中断强引用与对象之间的联系，可以显示的将强引用赋值为null，这样一来，JVM就可以适时的回收对象了
~~~

#### 12.3.2 软引用

~~~
在内存足够的时候，软引用对象不会被回收，只有在内存不足时，系统则会回收软引用对象。
如果回收了软引用对象之后仍然没有足够的内存，才会抛出内存溢出异常。这种特性常常被用来实现缓存技术，比如网页缓存，图片缓存等。
~~~

~~~java
package cn.itcast.jvm;

import java.lang.ref.SoftReference;
import java.util.ArrayList;
import java.util.List;

/**
 * 对象引用
 * VM Args: -Xms10m -Xmx10m 堆内存
 */
public class ReferenceTest {
    // 定义List集合
    private static List<Object> list = new ArrayList<Object>()
    // 软引用测试
    private static void testSoftReference() {
        // 循环创建软引用对象
        for (int i = 0; i < 10; i++) {
            byte[] buff = new byte[1024 * 1024];
            SoftReference<byte[]> sr = new SoftReference<byte[]>(buff);
            list.add(sr);
        }
        System.gc(); //主动通知垃圾回收
        for(int i=0; i < list.size(); i++){
            Object obj = ((SoftReference) list.get(i)).get();
            System.out.println(obj);
        }
    }
    public static void main(String[] args) {
        testSoftReference();
    }
}
~~~

![1597636050861](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597636050861.png) 

#### 12.3.3 弱引用

~~~
弱引用的引用强度比软引用要更弱一些，无论内存是否足够，只要 JVM 开始进行垃圾回收，那些被弱引用关联的对象都会被回收。在 JDK1.2 之后，用 java.lang.ref.WeakReference 来表示弱引用。

我们以与软引用同样的方式来测试一下弱引用:
~~~

~~~java
// 弱引用测试
private static void testWeakReference() {
    // 循环创建弱引用对象
    for (int i = 0; i < 10; i++) {
        byte[] buff = new byte[1024 * 1024];
        WeakReference<byte[]> sr = new WeakReference<byte[]>(buff);
        list.add(sr);
    }

    System.gc(); // 主动通知垃圾回收

    for(int i=0; i < list.size(); i++){
        Object obj = ((WeakReference) list.get(i)).get();
        System.out.println(obj);
    }
}
~~~

![1597644600920](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597644600920.png) 

#### 12.3.4 虚引用

~~~
虚引用是最弱的一种引用关系，如果一个对象仅持有虚引用，那么它就和没有任何引用一样，它随时可能会被回收，在JDK1.2 之后，用 PhantomReference 类来表示，通过查看这个类的源码，发现它只有一个构造函数和一个 get()方法，而且它的 get() 方法仅仅是返回一个null，也就是说将永远无法通过虚引用来获取对象，虚引用必须要和ReferenceQueue 引用队列一起使用。
~~~

```java
// 虚引用测试
private static void testPhantomReference() {
    // 循环创建虚引用对象
    for (int i = 0; i < 10; i++) {
        byte[] buff = new byte[1024 * 1024];
        PhantomReference<byte[]> sr = new PhantomReference<byte[]>
            (buff, new ReferenceQueue());
        list.add(sr);
    }

    System.gc(); // 主动通知垃圾回收

    for(int i=0; i < list.size(); i++){
        Object obj = ((PhantomReference) list.get(i)).get();
        System.out.println(obj);
    }
}
```





## 13、GC：垃圾收集算法

### 13.1 标记-清除算法

标记—清除算法是最基础的收集算法，过程分为标记和清除两个阶段，首先标记出需要回收的对象，之后由虚拟机统一回收已标记的对象。



![image-20200417203403151](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417203403151.png)  

优点：效率相对高

缺点: 产生内存碎片

对象被回收之后会产生大量不连续的内存碎片，当需要分配较大对象时，由于找不到合适的空闲内存而不得不再次触发垃圾回收动作。





### 13.2 标记-整理算法

**标记：**它的第一个阶段与标记**/**清除算法是一模一样的，均是遍历GC Roots，然后将存活的对象标记。

**整理：**移动所有存活的对象，且按照内存地址次序依次排列，然后将末端内存地址以后的内存全部回收。因此，第二阶段才称为整理阶段。

![image-20200417203654958](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417203654958.png) 

可以看到，标记的存活对象将会被整理，按照内存地址依次排列，而未被标记的内存会被清理掉。如此一来，当我们需要给新对象分配内存时，JVM只需要持有一个内存的起始地址即可，这比维护一个空闲列表显然少了许多开销。不难看出，**标记/整理算法不仅可以弥补标记/清除算法当中，内存区域分散的缺点，也消除了复制算法当中，内存减半的高额代价，可谓是一举两得。**  

优点：没有内存碎片

缺点:  效率相对标记清除算法较低一点



### 13.2 复制算法

算法的基本思路是：**将内存划分为大小相等的两部分，每次只使用其中一半，当第一块内存用完了，就把存活的对象复制到另一块内存上，然后清除剩余可回收的对象，这样就解决了内存碎片问题。**

![image-20200417203428722](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417203428722.png)  

优点：没有内存碎片

缺点:  效率相对较低，只有一半内存被使用

合适垃圾较多，存活对象较少的情况。

  

### 13.4 分代收集算法

前面介绍了多种回收算法，每一种算法都有自己的优点也有缺点，谁都不能替代谁，所以根据垃圾回收对象的特点进行选择，才是明智的选择。分代算法其实就是这样的，根据回收对象的特点进行选择。

新生代中的对象绝大部分都是朝生夕死，现代商业虚拟机垃圾收集大多采用分代收集算法。主要思路是根据对象存活生命周期的不同将内存划分为几块。一般是把Java堆分为新生代和老年代，然后根据各个年代的特点采用最合适的收集算法。**新生代中，对象的存活率比较低，所以选用复制算法，老年代中对象存活率高所以使用“标记-清除”或“标记-整理”算法进行回收**。

![1597803655488](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597803655488.png)

因为新生代中的对象 绝大部分都是 朝生夕死，所以并不需要按照1:1的比例来划分内存空间，而是将内存分为一块较大的Eden空间和两块较小的Survivor空间，每次使用Eden和其中一块Survivor。当回收时，将Eden和Survivor中还存活着的对象一次性的复制到另外一块Survivor空间上，最后清理Eden和刚才用过的Survivor空间，HotSpot默认的空间比例是 8:1:1



## 14、GC：垃圾收集器

如果说收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现。下图是HotSpot虚拟机所包含的所有垃圾收集器: 

![image-20200417204357586](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417204357586.png) 

STW: Stop The World 是指由于虚拟机在后台发起垃圾收集时，会暂停所有其他的用户工作线程，造成用户应用暂时性停止。从JDK1.3开始，HotSpot虚拟机开发团队一直为消除或者减少工作线程因为内存回收而导致停顿而努力着。用户线程的停顿时间不断缩短，但仍无法完全消除。



从JDK3(1.3)开始，HotSpot团队一直努力朝着高效收集、减少停顿(STW: Stop The World)的方向努力，也贡献了从串行到CMS乃至最新的G1在内的一系列优秀的垃圾收集器。上图展示了JDK的垃圾回收大家庭，以及相互之间的组合关系。

HotSpot虚拟机:

+ Serial: 最早的单线程串行垃圾回收器，针对新生代，采用**复制**算法。
+ Serial Old: 是 Serial 垃圾收集器的老年代版本，同样也是单线程的，采用"**标记-整理**"算法。
+ ParNew: 是Serial的多线程版本，针对新生代，采用**复制**算法。
+ Parallel Scavenge: 吞吐量优先的收集器，和ParNew收集器类似，是新生代收集器，使用**复制**算法的并行多线程收集器，可以牺牲等待时间换取系统的吞吐量。
+ Parallel Old: 是 Parallel 老年代版本，Parallel Old 使用的是"**标记-整理**"算法。
+ CMS(Concurrent Mark Sweep): 一种以获得最短停顿时间为目标的收集器，采用“**标记-清除**”算法。
+ G1(Garbage First): 为解决CMS算法产生空间碎片和其它一系列的问题缺陷，HotSpot提供了另外一种垃圾回收策略，G1算法，一种兼顾吞吐量和停顿时间的 GC 实现，是 JDK 9 以后的默认 GC 选项。



查看JVM默认使用的垃圾收集器(Windows可以，Linux中没找到默认GC的查看方法)

```shell
java -XX:+PrintCommandLineFlags -version
```

1. 吞吐量：操作数量/单位时间
2. 响应时间：STW越短，响应时间越快



### 14.1 串行收集器

![image-20200417204528977](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417204528977.png) 

**串行收集器组合 Serial + Serial Old**  

串行收集器是最基本、发展时间最长、久经考验的垃圾收集器，也是client模式下的默认收集器配置。
串行收集器采用单线程stop-the-world的方式进行收集。当内存不足时，串行GC设置停顿标识，待所有线程都进入安全点(Safepoint)时，应用线程暂停，串行GC开始工作，采用单线程方式回收空间并整理内存。单线程也意味着复杂度更低、占用内存更少，但同时也意味着不能有效利用多核优势。事实上，**串行收集器特别适合堆内存不高、单核甚至双核CPU的场合。**

> 开启选项: -XX:+UseSerialGC



### 14.2 并行收集器

![image-20200417204642790](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417204642790.png) 

**并行收集器组合 Parallel Scavenge + Parallel Old**

+ **并行收集器是以关注吞吐量为目标的垃圾收集器**，也是server模式下的默认收集器配置，对吞吐量的关注主要体现在年轻代Parallel Scavenge收集器上。
+ 并行收集器与串行收集器工作模式相似，都是stop-the-world方式，只是暂停时并行地进行垃圾收集。年轻代采用复制算法，老年代采用标记-整理，在回收的同时还会对内存进行压缩。关注吞吐量主要指年轻代的Parallel Scavenge收集器，通过两个目标参数-XX:MaxGCPauseMills和-XX:GCTimeRatio，调整新生代空间大小，来降低GC触发的频率。并行收集器适合对吞吐量要求远远高于延迟要求的场景，并且在满足最差延时的情况下，并行收集器将提供最佳的吞吐量。

> 开启选项：-XX:+UseParallelGC 或 -XX:+UseParallelOldGC



### 14.3 并发清除收集器

![image-20200417204749926](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417204749926.png) 

**并发标记清除收集器组合 ParNew + CMS**  

+ 并发标记清除(CMS)是以关注延迟为目标、十分优秀的垃圾回收算法，开启后，年轻代使用并行收集，老年代回收采用CMS进行垃圾回收，对延迟的关注也主要体现在老年代CMS上。
+ 年轻代ParNew与并行收集器类似，而老年代CMS每个收集周期都要经历：**初始标记、并发标记、重新标记、并发清除**。其中，初始标记以STW的方式标记所有的根对象；并发标记则同应用线程一起并行，标记出根对象的可达路径；在进行垃圾回收前，CMS再以一个STW进行重新标记，标记那些由mutator线程(指引起数据变化的线程，即应用线程)修改而可能错过的可达对象；最后得到的不可达对象将在并发清除阶段进行回收。值得注意的是，初始标记和重新标记都已优化为多线程执行。CMS非常适合堆内存大、CPU核数多的服务器端应用，也是G1出现之前大型应用的首选收集器。

**但是CMS并不完美，它有以下缺点:**

+ 由于并发进行，CMS在收集与应用线程会同时会增加对堆内存的占用，也就是说，CMS必须要在老年代堆内存用尽之前完成垃圾回收，否则CMS回收失败时，将触发担保机制，串行老年代收集器将会以STW的方式进行一次GC，从而造成较大停顿时间；
+ 标记清除算法无法整理空间碎片，老年代空间会随着应用时长被逐步耗尽，最后将不得不通过担保机制对堆内存进行压缩。CMS也提供了参数-XX:CMSFullGCsBeForeCompaction(默认0，即每次都进行内存整理)来指定多少次CMS收集之后，进行一次压缩的Full GC。

> 开启选项：-XX:+UseConcMarkSweepGC



### 14.4 Garbage First（G1）

G1(Garbage First)垃圾收集器是当今垃圾回收技术最前沿的成果之一。早在JDK7就已加入JVM的收集器大家庭中，成为HotSpot重点发展的垃圾回收技术。同优秀的CMS垃圾回收器一样，G1也是关注最小时延的垃圾回收器，也同样适合大尺寸堆内存的垃圾收集，官方也推荐使用G1来代替选择CMS。G1最大的特点是引入分区的思路，弱化了分代的概念，合理利用垃圾收集各个周期的资源，解决了其他收集器甚至CMS的众多缺陷。

![image-20200417204953834](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417204953834.png) 

+ G1垃圾收集器也是以关注延迟为目标、服务器端应用的垃圾收集器，被HotSpot团队寄予取代CMS的使命，也是一个非常具有调优潜力的垃圾收集器。虽然G1也有类似CMS的收集动作**：初始标记、并发标记、重新标记、清除、转移回收**，G1收集与以上三组收集器有很大不同：
+ + G1的设计原则是"**首先收集尽可能多的垃圾(Garbage First)**"。因此，G1并不会等内存耗尽或者快耗尽(CMS)的时候开始垃圾收集，而是在内部采用了启发式算法，在老年代找出具有高收集收益的分区进行收集。
+ + G1采用内存分区(Region)的思路，将内存划分为一个个相等大小的内存分区，回收时则以分区为单位进行回收，存活的对象复制到另一个空闲分区中。
+ + G1虽然也是分代收集器，但整个内存分区不存在物理上的年轻代与老年代的区别，也不需要完全独立的survivor做复制准备。G1只有逻辑上的分代概念，或者说每个分区都可能随G1的运行在不同代之间前后切换；
+ G1年轻代和老年代的收集界限比较模糊，采用了混合(mixed)收集的方式。即每次收集既可能只收集年轻代分区(年轻代收集)，也可能在收集年轻代的同时，包含部分老年代分区(混合收集)，这样即使堆内存很大时，也可以限制收集范围，从而降低停顿。

> 开启选项：-XX:+UseG1GC



| 参数                    | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| -XX:+UseSerialGC        | Serial New + Serial Old，启用串行垃级收集器                  |
| -XX:+UseParallelGC      | Parallel Scavenge + Parallel Old，启用并行垃圾收集器         |
| -XX:+UseParallelOldGC   | Parallel Scavenge + Parallel Old                             |
| -XX:+UseParNewGC        | ParNew + SerialOld                                           |
| -XX:+UseConcMarkSweepGC | ParNew + CMS，启用该选项后-XX:+UseParNewGC自动启用。         |
| -XX:+UseG1GC            | G1，用于多核、大内存的机器。它在保持高吞吐量的情况下，高概率满足GC暂停时间的目标。 |



## 15、JVM性能监控：JDK命令行工具

假如我们需要排查定位一些系统问题，知识、经验是关键基础，数据是依据，那么工具就是运用知识处理数据的手段。这里面指的数据包括：运行日志、异常堆栈、GC日志、线程快照、堆转储快照等等。 经常使用适当的虚拟机监控和分析的工具可以加快我们分析数据、定位解决问题的速度。



我们可能都知道，在JDK的安装目录中有 "java.exe"、"javac.exe"这两个命令行工具，但可能很多人都不了解bin目录中其它命令行的作用。每当JDK版本更新后，这个目录下总是默默的多出了一些命令行。接下来，我们就要去了解这些工具，主要去了解监视虚拟机和故障处理的一些工具。



### jps: 虚拟机进程查看工具

> 可以列出当前系统正在运行的虚拟机进程，命令格式: jps [options][hostid]

![image-20200417211238118](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417211238118.png) 

![1597715317648](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597715317648.png) 

> 杀进程: taskkill -pid 4556 -f



### jstat: 虚拟机统计信息监视工具

用于监视虚拟机各种运行状态信息的命令行工具，如: 类装载、内存、垃圾收集，是在没有图形页面纯文本控制台中定位运行期虚拟机性能问题的首选工具

~~~
命令格式: jstat [options]    [vmid]   [interval(ms)]   [count]
         jstat [-命令选项]  [进程号]   [间隔时间/毫秒]   [查询次数]
~~~

![image-20200417211325634](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20200417211325634.png) 

> 类加载统计: jstat -class 12896

![1597715847800](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597715847800.png) 

> 垃圾回收统计: jstat -gcutil 12896 250 20
> （每隔250毫秒查询一次进程12896的垃圾收集情况，一共查询20次）

![1597716145146](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597716145146.png) 

~~~
S0 —  Heap上的 Survivor space 0 区已使用空间的百分比(幸存1区)
S1 —  Heap上的 Survivor space 1 区已使用空间的百分比(幸存2区)
E  —  Heap上的 Eden space 区已使用空间的百分比(伊甸园区)
O  —  Heap上的 Old space 区已使用空间的百分比(老年代)
M  —  Meta space 区已使用空间的百分比(元数据区|永久代)
CCS — 压缩使用比例
YGC — Young GC 年轻代垃圾回收次数
YGCT– Young GC Time 年轻代垃圾回收消费时间(单位秒)
FGC — Full GC java堆(年轻代与老年代)垃圾回收次数
FGCT– Full GC Time  java堆(年轻代与老年代)垃圾回收消费时间(单位秒)
GCT — GC Time 垃圾回收消耗总时间(单位秒)
~~~

> 详细参考: <https://www.cnblogs.com/sxdcgaq8080/p/11089841.html> 



### jinfo: Java配置信息查看工具

用于实时的查看虚拟机各项参数

~~~shell
命令格式: jinfo [option] vmid

# 查看2734虚拟机的所有参数
jinfo -flags 12896

# 查看12896虚拟机MaxHeapSize属性的值
jinfo -flag MaxHeapSize 12896
~~~

![1597718951523](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597718951523.png)  



### jmap: Java内存映像工具

Heap Dump 是 Java进程所使用的内存情况在某一时间的一次快照。以文件的形式持久化到磁盘中。

~~~shell
命令格式: jmap [option] vmid

# 将12896进程所使用的内存情况映像存储到dump.hprof文件
jmap -dump:format=b,file=F:/dump.hprof 12896
~~~

![1597719695215](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597719695215.png)  





### jhat: Java堆转储快照分析工具

> 与jmap搭配使用，用于分析dump文档，内置了http服务器，可以在浏览器中进行分析

Java堆内存OOM是经常会见到的异常，"java.lang.OutOfMemoryError" 跟着会提示 "Java heap space",要解决这个区域的异常，一般的手段是先通过内存映像分析工具对Dump出来的堆转储快照进行分析

```shell
jhat F:/dump.hprof
```

![1597720198570](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597720198570.png) 

![1597720220217](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597720220217.png) 



MAT也可以查看jmap导出的快照数据



### jstack: Java堆栈跟踪工具

用于生成虚拟机当前时刻的线程快照，线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合。用于分析线程死锁、死循环、请求外部资源时间过长等常见原因。

~~~shell
# 显示指定进程的线程快照
# 语法格式: jstack pid
jstack 12896

# 将某个进程的线程信息导出到abc.txt文件
jstack 12896 > abc.txt
~~~

![1597721049832](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597721049832.png) 





## 16、JVM性能监控：CPU飙高排查实战

项目准备(在linux系统模拟生产环境): 

```txt
运行项目: nohup java -jar xxx.jar > 日志文件 &
         nohup java -jar ROOT.jar > nohup.out &

输出日志: tail -f nohup.out


nohup: (no hangup) 不挂断地运行, 后台运行
&: 关闭终端后程序还在后台运行
```

![1597722947387](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597722947387.png)  

浏览器请求: http://192.168.12.132:8088/findAll

![1597723028912](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597723028912.png) 

生产环境排查 CPU 飙高, 解决方案:

![image-20210113152617209](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/image-20210113152617209.png)



### 具体操作

1. 使用 top 命令查看内存情况，得到CPU飙高的进程ID:

```
top
```

![1597723160431](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597723160431.png) 



2. 查看某个进程内部线程情况: top -p 2087 -H

![1597725197394](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597725197394.png) 



3. 把消耗CPU最高的线程PID转换成16进制(默认十进制): printf "%x" 2109

![1597725838959](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597725838959.png) 



4. 导出某个进程的线程信息到日志文件: jstack 2087 > thread.log

![1597731346579](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597731346579.png) 



5. 查看thread.log日志文件，根据线程ID，进行模糊查询:

```shell
vi thread.log

/83d
```

![1597731453921](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597731453921.png) 

![1597731491630](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597731491630.png) 



6. 找到了UserService.java:10行

![1597731642512](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597731642512.png) 





## 17、JVM性能监控：死锁排查实战

杀死 2087 进程，重新运行项目: kill -9 2087

![1597732772418](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597732772418.png) 

浏览器请求(产生死锁): 

![1597731922773](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597731922773.png) 

**生产环境排查 死锁, 通过jstack命令 可以直接识别死锁**:

导出某个进程的线程信息: jstack 10789 > thread.log

```txt
// 发现一个java级别的死锁
Found one Java-level deadlock:
=============================
"Thread-5":
  waiting to lock monitor 0x00007f904c004538 (object 0x00000000f1108d98, a java.lang.Object),
  which is held by "Thread-4"
"Thread-4":
  waiting to lock monitor 0x00007f904c0048a8 (object 0x00000000f1108da8, a java.lang.Object),
  which is held by "Thread-5"
  
// 死锁的详细信息
Java stack information for the threads listed above:
===================================================
"Thread-5":
        at cn.itcast.service.UserService.lambda$findOne$1(UserService.java:40)
        - waiting to lock <0x00000000f1108d98> (a java.lang.Object)
        - locked <0x00000000f1108da8> (a java.lang.Object)
        at cn.itcast.service.UserService$$Lambda$393/2066072829.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
"Thread-4":
        at cn.itcast.service.UserService.lambda$findOne$0(UserService.java:26)
        - waiting to lock <0x00000000f1108da8> (a java.lang.Object)
        - locked <0x00000000f1108d98> (a java.lang.Object)
        at cn.itcast.service.UserService$$Lambda$392/2132434886.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

![1597733303410](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597733303410.png) 





## 18、JVM性能监控：JDK可视化工具

JDK除了提供大量的命令行工具外，还有两个功能强大的可视化工具: JConsole和VisualVM ,这两个工具是JDK的正式成员，功能非常强大！

### jconsole: Java监视与管理控制台

通过jdk/bin目录下的 jconsole.exe启动JConsole，将会自动的搜索出本机的所有java虚拟机进程。

![1597736876722](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597736876722.png) 

在本地进程中会列出本地 正在运行的Java虚拟机列表，也可以远程连接其他服务器的Java服务器，现在我们选择JConsoleTest的进程，并且将堆的大小固定在100MB，每个隔一小段时间向集合中装入128KB的数据，并注意JConsole监控中个数据指标的变化。

~~~java
package cn.itcast.jvm;

import java.util.ArrayList;
import java.util.List;

/**
* VM Args: -Xms150m -Xmx150m -XX:+UseSerialGC
**/
public class JConsoleTest {
	static class OOMObject{
		public byte[] placeholder = new byte[128*1024];
	} 
    public static void fillHeap(int num) throws InterruptedException {
		List<OOMObject> list = new ArrayList<OOMObject>();
		for (int i = 0; i < num; i++) {
			System.out.println("i = " + i);
            Thread.sleep(100);
        	list.add(new OOMObject());
        } 
        System.gc();
		Thread.sleep(5000);
	}
    public static void main(String[] args) throws InterruptedException {
		fillHeap(500);
		fillHeap(1000);
	}
}
~~~

![1597737198094](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597737198094.png)  

![1597737302009](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597737302009.png) 



### VisualVM: 多合一故障处理工具

All in One Java Troubleshooting Tool，目前为止JDK发布的最强大的运行监视和故障处理程序，除了基本的运行监视、故障处理之外，它还设计了可插拔的插件功能，可以根据需要添加一些优秀的插件时间更强大的功能。它的性能分析功能甚至比一些专业的收费工具也不逊色。

双击JDK/bin目录下的 jvisualvm.exe，启动VisualVM工具。

![1597737559247](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597737559247.png)  

~~~
左侧列表显示的是本地java虚拟机的活动列表，点击一个进程可以进入监控状态
~~~



**概述页签展示了 当前监控java虚拟机的配置信息**  

![1597737582721](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597737582721.png)  



**监视页签展示了,CPU 类加载 堆 线程的基本监控信息**  

![1597737695668](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597737695668.png)  



**线程的信息，可以转储线程快照，也可以打开线程快照进行分析**

![1597737738870](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597737738870.png)  



**抽样器，可以查看方法的时间占用情况，也可以查看内存占用的实时情况**  

![1597737820398](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597737820398.png)  



![1597737876681](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597737876681.png)  



### Visual GC：插件安装

**操作步骤:**

+ 第一步:

  ![1597825832096](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597825832096.png) 

+ 第二步: 选择资料中已下载的: com-sun-tools-visualvm-modules-visualgc_1.nbm

  ![1597825943091](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597825943091.png) 

+ 第三步: 安装

  ![1597826015588](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597826015588.png) 

  ![1597826034352](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597826034352.png) 

  ![1597826065342](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597826065342.png) 

  ![1597826090201](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597826090201.png) 

+ 第四步: Visual GC界面

  ![1597826141889](就业技术加强-03-JVM虚拟机性能优化：运行时数据区-垃圾收集.assets/1597826141889.png) 

  



## 19、JVM：常见面试题总结

+ 问题1: 说一下JVM的主要组成部分？及作用？

  ```txt
  类加载器（ClassLoader）
  运行时数据区（Runtime Data Area）
  执行引擎（Execution Engine）
  本地库接口（Native Interface）
  
  组件的作用: 
  首先通过类加载器（ClassLoader）会把 Java 代码转换成字节码，运行时数据区（Runtime Data Area）再把字节码加载到内存中，而字节码文件只是 JVM 的一套指令集规范，并不能直接交给底层操作系统去执行，因此需要特定的命令解析器执行引擎（Execution Engine），将字节码翻译成底层系统指令，再交由CPU去执行，而这个过程中需要调用其他语言的本地库接口（Native Interface）来实现整个程序的功能。
  ```

+ 问题2: 说一下JVM运行时数据区？详细介绍下每个区域的作用? 

  ```txt
  1. 程序计数器（Program Counter Register）: 当前线程所执行的字节码的行号指示器，字节码解析器的工作是通过改变这个计数器的值，来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能，都需要依赖这个计数器来完成。
  
  2. Java 虚拟机栈（Java Virtual Machine Stacks）: 用于存储局部变量表、操作数栈、动态链接、方法出口等信息。
  
  3. 本地方法栈（Native Method Stack）: 与虚拟机栈的作用是一样的，只不过虚拟机栈是服务 Java 方法的，而本地方法栈是为虚拟机调用 Native 方法服务的。
  
  4. Java 堆（Java Heap）: Java 虚拟机中内存最大的一块，是被所有线程共享的，几乎所有的对象实例都在这里分配内存。
  
  5. 方法区（Methed Area）: 用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译后的代码等数据。
  ```

+ 问题3: 说一下堆栈的区别？

  ```txt
  1. 功能方面: 堆是用来存放对象的，栈是用来执行程序的。
  
  2. 共享性: 堆是线程共享的，栈是线程私有的。
  
  3. 空间大小: 堆大小远远大于栈。
  ```

+ 问题4: Java中都有哪些类加载器 

  ```txt
  1. 启动类加载器（Bootstrap ClassLoader），是虚拟机自身的一部分，用来加载Java_HOME/jre/lib/
  
  2. 扩展类加载器（Extension ClassLoader）：负责加载jre/lib/ext目录。
  
  3. 应用程序类加载器（Application ClassLoader）。负责加载用户类路径（classpath）上的指定类库，我们可以直接使用这个类加载器。一般情况，如果我们没有自定义类加载器默认就是用这个加载器。
  
  4. 自定义类加载器：通过继承ClassLoader抽象类，实现loadClass方法。
  ```

+ 问题5: 什么是双亲委派模型？

  ```txt
  如果一个类加载器收到了类加载的请求，它首先不会自己去加载这个类，而是把这个请求委派给父类加载器去完成，每一层的类加载器都是如此，这样所有的加载请求都会被传送到顶层的启动类加载器中，只有当父加载无法完成加载请求（它的搜索范围中没找到所需的类）时，子加载器才会尝试去加载类。
  ```

+ 问题6: 说一下类装载的执行过程？  

  ```txt
  类装载分为以下 5 个步骤:
  
  1. 加载: 根据查找路径找到相应的 class 文件然后导入。
  2. 检查: 检查加载的 class 文件的正确性。
  3. 准备: 给类中的静态变量分配内存空间。
  4. 解析: 虚拟机将常量池中的符号引用替换成直接引用的过程。符号引用就理解为一个标示，而在直接引用直接指向内存中的地址。
  5. 初始化: 对静态变量和静态代码块执行初始化工作。
  ```

+ 问题7: 怎么判断对象是否可以被回收？

  ```txt
  一般有两种方法来判断:
  
  1. 引用计数器：为每个对象创建一个引用计数，有对象引用时计数器 +1，引用被释放时计数 -1，当计数器为 0 时就可以被回收。它有一个缺点不能解决循环引用的问题。
  
  2. 可达性分析：从 GC Roots 开始向下搜索，搜索所走过的路径称为引用链。当一个对象到 GC Roots 没有任何引用链相连时，则证明此对象是可以被回收的。
  ```

+ 问题8: Java 中都有哪些引用类型？ 

  ```txt
  强引用：发生 gc 的时候不会被回收。
  软引用：有用但不是必须的对象，在发生内存溢出之前会被回收。
  弱引用：有用但不是必须的对象，在下一次GC时会被回收。
  虚引用（幽灵引用/幻影引用）：无法通过虚引用获得对象，用 PhantomReference 实现虚引用，虚引用的用途是在gc 时返回一个通知。
  ```

+ 问题9: 说一下 JVM 有哪些垃圾回收算法？

  ```txt
  1. 标记-清除算法：标记无用对象，然后进行清除回收。缺点：效率高，无法清除垃圾碎片。
  
  2. 标记-整理算法：标记无用对象，让所有存活的对象都向一端移动，然后直接清除掉端边界以外的内存。
  
  3. 复制算法：按照容量划分二个大小相等的内存区域，当一块用完的时候将活着的对象复制到另一块上，然后再把已使用的内存空间一次清理掉。缺点：内存使用率不高，只有原来的一半。
  
  4. 分代算法：根据对象存活周期的不同将内存划分为几块，一般是新生代和老年代，新生代基本采用复制算法，老年代采用标记整理或标记清除算法。
  ```

+ 问题10: 说一下 JVM 有哪些垃圾收集器？

  ```txt
  1. Serial: 最早的单线程串行垃圾回收器，针对新生代，采用复制算法。
  2. Serial Old: 是 Serial 垃圾收集器的老年代版本，同样也是单线程的，采用"标记-整理"算法。
  3. ParNew: 是Serial的多线程版本，针对新生代，采用复制算法。
  4. Parallel Scavenge: 吞吐量优先的收集器，和ParNew收集器类似，是新生代收集器，使用复制算法的
     并行多线程收集器，可以牺牲等待时间换取系统的吞吐量。
  5. Parallel Old: 是 Parallel 老年代版本，Parallel Old 使用的是"标记-整理"算法。
  6. CMS(Concurrent Mark Sweep): 一种以获得最短停顿时间为目标的收集器，采用“标记-清除”算法。
  7. G1(Garbage First): 为解决CMS算法产生空间碎片和其它一系列的问题缺陷，HotSpot提供了另外一种
     垃圾回收策略，G1算法，一种兼顾吞吐量和停顿时间的 GC 实现，是 JDK 9 以后的默认 GC 选项。
  ```

+ 问题11: 新生代垃圾回收器和老生代垃圾回收器都有哪些？有什么区别？

  ```txt
  - 新生代回收器: Serial、ParNew、Parallel Scavenge
  - 老年代回收器: Serial Old、Parallel Old、CMS
  - 整堆回收器: G1
  
  新生代垃圾回收器一般采用的是复制算法，复制算法的优点是效率高，缺点是内存利用率低。
  老年代回收器一般采用的是标记-整理的算法进行垃圾回收。
  ```

+ 问题12: 简述分代垃圾回收器是怎么工作的？

  ```txt
  分代回收器有两个分区：老年代和新生代，新生代默认占比是 1/3，老年代的默认占比是 2/3。
  
  新生代使用的是复制算法，新生代里有 3 个分区：Eden、To Survivor、From Survivor，它们的默认占比是8:1:1，它的执行流程如下:
  
  把 Eden + From Survivor 存活的对象放入 To Survivor 区；清空 Eden 和 From Survivor 分区；
  From Survivor 和 To Survivor 分区交换，From Survivor 变 To Survivor，To Survivor 变 From
  Survivor。
  
  每次在 From Survivor 到 To Survivor 移动时都存活的对象，年龄就 +1，当年龄到达 15（默认配置是 15）时，升级为老年代。大对象也会直接进入老年代。
  
  老年代当空间占用到达某个值之后就会触发全局垃圾收回，一般使用标记整理的执行算法。以上这些循环往复就构成了整个分代垃圾回收的整体执行流程。
  ```

+ 问题13: Minor GC与Full GC分别在什么时候发生？

  ```txt
  Minor GC: 新生代内存不够用时候发生MGC也叫YGC。
  Full GC: JVM堆内存不够的时候发生FGC。
  ```

+ 问题14: 有没有在生产环境下排过错，说说过程?

  ```txt
  可以说案例分析里面的案例，也可以说CPU飙高的排查
  ```

+ 问题15: 说一下 你知道的JVM 性能监控的工具？

  ```txt
  JDK 自带了很多监控工具，都位于 JDK 的 bin 目录下，其中最常用的是 jconsole 和 jvisualvm 这两款视图监控工具。
  jps jinfo jstat jmap jstack jhat
  
  jconsole: 用于对 JVM 中的内存、线程和类等进行监控。
  jvisualvm: JDK自带的全能分析工具，可以分析: 内存快照、线程快照、程序死锁、监控内存的变化、gc变化等。
  ```

+ 问题16: 常用的 JVM 调优的参数都有哪些？

  ```txt
  -Xms2g                   堆初始化内存为 2g
  -Xmx2g                   堆最大内存为 2g
  -Xmn500M                 设置新生代为500m
  -XX:PermSize=500M        1.8之后采用 -XX:MetaspaceSize=200m
  -XX:MaxPermSize=500M     1.8之后采用 -XX:MaxMetaspaceSize=200m
  -XX:NewRatio=4           设置年轻代和老年代的内存比例为 1:4
  -XX:SurvivorRatio=8      设置新生代 Eden 和 Survivor 比例为 8:1:1
  –XX:+UseParNewGC         指定使用 ParNew + Serial Old 垃圾回收器组合(串行)
  -XX:+UseParallelOldGC    指定使用 ParNew + ParNew Old 垃圾回收器组合(并行)
  -XX:+UseConcMarkSweepGC  指定使用 ParNew + CMS 垃圾回收器组合(并行)
  -XX:+UseG1GC             指定使用 G1 垃圾回收器(并行)
  -XX:+PrintGC             开启打印 gc 信息
  -XX:+PrintGCDetails      打印 gc 详细信息。
  -XX:MaxDirectMemorySize  设置直接内存的大小
  ```



我们的程序上线运行,最好是开启打印JVM的GC,并且保存文件中

```
-XX:+PrintGCDetails -Xloggc:F:\gc1.log
```

