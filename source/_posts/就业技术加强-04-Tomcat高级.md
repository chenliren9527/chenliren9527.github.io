---
title: 就业技术加强(04)-Tomcat高级
tags:
  - 笔记
  - 就业技术加强
  - Tomcat的体系结构
  - Tomcat的请求流程
  - 配置Tomcat服务器
  - 配置Tomcat的调优方案
  - Jmeter进行性能测试
categories:
  - 就业技术加强
date: 2021-01-26 18:57:30
---

## 课程目标

目标1: 能够描述Tomcat的体系结构

目标2: 能够说出Tomcat的请求流程

目标3: 能够配置Tomcat服务器

目标4: 能够配置Tomcat的调优方案

目标5: 能够用Jmeter进行性能测试



## 01、Tomcat：组成结构

> 目标: 掌握和了解Tomcat的组成结构

**下载地址:** https://tomcat.apache.org/download-80.cgi

![1592287315100](就业技术加强-04-Tomcat高级.assets/1592287315100.png) 

**组成结构:**

+ 解压apache-tomcat-8.5.42.zip:

  ![image-20210120094133774](就业技术加强-04-Tomcat高级.assets/image-20210120094133774.png)

  ![image-20210126090317957](就业技术加强-04-Tomcat高级.assets/image-20210126090317957.png)

  ![image-20210120094243563](就业技术加强-04-Tomcat高级.assets/image-20210120094243563.png)

  

  ![1592346951570](就业技术加强-04-Tomcat高级.assets/1592346951570.png)  

- bin: 启动`startup.bat/sh`，关闭`shutdown.bat/sh`，配置jvm`catalina.bat/sh`

  其中`.sh` 都是linux的可执行文件，`.bat`都是windows的可执行文件。

  **建议：如果你不想搞混淆，你可以在windows中把.sh删掉，在linux中把.bat删掉。**

  ![1592347154621](就业技术加强-04-Tomcat高级.assets/1592347154621.png) 

- conf: 配置目录，`server.xml`核心配置文件，配置端口，线程池等。

- lib: 存储jar包目录。

- logs: 日志目录，tomcat启动和运行项目的日志目录，方便的查看的日志信息。

  ```shell
  tail -f logs/catalina.out
  ```

- temp: 临时目录，文件上传产生的临时文件存放的目录。

- webapps: 项目的部署目录。

- work: jsp编译以后生成类的目录。





## 02、Tomcat：整体架构

> 目标: 了解tomcat服务器的整体架构。

**Tomcat服务器的核心**

就是负责处理请求(request)与响应(response)。

![image-20210114102038230](就业技术加强-04-Tomcat高级.assets/image-20210114102038230.png)





**Tomcat整体架构图:**

![img](就业技术加强-04-Tomcat高级.assets/1377406-20190413151238887-1461861651.png)

![1592347759453](就业技术加强-04-Tomcat高级.assets/1592347759453.png) 

| 组件      | 职责                                                         |
| --------- | ------------------------------------------------------------ |
| Server    | 即服务器，每个Tomcat程序启动后，就是一个server。             |
| Service   | Service是服务器的内部组件，一个Server包含多个Service。它将若干个Connector组件绑定到一个Container的Engine上(**用来接受请求和处理请求**) |
| Connector | 连接器，负责接收请求与返回响应，**监听端口和规范协议。默认是：http协议和端口8080 。是一种动态资源比如：jsp/servlet，还有一种是AJP协议，监听的端口是：8009 ，处理静态资源比较高效，Connector是Tomcat的连接器，其主要任务是负责处理浏览器发送过来的请求，并创建一个Request和Response的对象用于和浏览器交换数据，然后产生一个线程用于处理请求，Connector会把Request和Response对象传递给该线程，该线程的具体的处理过程是Container容器的事了。** |
| Container | 顶级父容器: 包含了四个子容器(Engine、Host、Context、Wrapper) |
| Engine    | 处理引擎，所有的请求都是通过处理引擎处理的。                 |
| Host      | 主机容器(虚拟主机)，接收处理Engine发送过来的请求，并分发到指定的Context默认的是:**ROOT** |
| Context   | 上下文容器，一个Context对应一个Web Application，接收Host的调用请求，并分发到指定的Wrapper组件。 |
| Wrapper   | Servlet容器: 代表一个Servlet，它负责管理一个Servlet，包括Servlet的装载、初始化、执行以及资源回收。 |

有了上面的概念的理解，就可以简单的想象一下Tomcat的处理过程：

![image-20210114110809327](就业技术加强-04-Tomcat高级.assets/image-20210114110809327.png)

1. 首先请求发送给服务器。

2. 服务器使用相应的服务进行处理。

3. 先通过不同的连接器请求后发送给处理引擎。

4. 处理引擎通过对虚拟主机的分析，发送给相应的虚拟主机。

5. 虚拟主机使用相应的应用进行响应。

简言之，就是请求会先发送到连接器，连接器转给处理引擎进行处理。



## 03、Tomcat监控工具

+ jdk自带两个监控工具

  ![1592351486452](就业技术加强-04-Tomcat高级.assets/1592351486452.png)   

+ jconsole(Java Console): java监视管理控制台

  ![1592352170171](就业技术加强-04-Tomcat高级.assets/1592352170171.png) 

+ jvisualvm(Java VisualVM): java可视化监控台

  ![1592352241673](就业技术加强-04-Tomcat高级.assets/1592352241673.png) 







## 04、Tomcat：配置Server和Service

> 目标: 了解Server的作用。



#### 配置Server

Tomcat服务器的配置主要集中在tomcat/conf目录下: catalina.policy、catalina.properties、context.xml、server.xml、tomcat-users.xml、web.xml文件。

server.xml是Tomcat服务器的核心配置文件，包含了Tomcat所有的配置。

**操作说明:**

+ Server是server.xml的根元素，用于启动或停止Tomcat服务器，默认使用的实现类是org.apache.catalina.core.StandardServer。

  ```xml
  <Server port="8005" shutdown="SHUTDOWN" className="org.apache.catalina.core.StandardServer">
  	...
  </Server>
  ```

+ 相关属性:

  + className: 实现Server容器的类名，默认org.apache.catalina.core.StandardServer。
  + port: 设置服务的端口，默认为8005。
  + shutdown: 用于关闭tomcat服务的命令，默认为SHUTDOWN。




#### 配置Service

![1592380586656](就业技术加强-04-Tomcat高级.assets/1592380586656.png) 

Service元素的属性:

+ className: 实现Service的类名，默认是org.apache.catalina.core.StandardService。
+ name: 服务的名称，默认为Catalina。

Service内嵌的子元素: Executor、Connector、Engine

+ Executor 配置Connector连接器可以共享的执行器(包含线程池)。

+ Connector 配置Connector连接器接收请求、返回响应。 

+ Engine 配置引擎作为处理请求的入口，引擎实现分析请求中包含的HTTP头，并传递它们到适当的主机。

  ```xml
  <Service name="Catalina">
      <Executor name="tomcatThreadPool"/>
      <Connector executor="tomcatThreadPool" port="8080"/>
      <Engine name="Catalina" defaultHost="localhost"/>
  </Service>
  ```

  说明: 一个Server服务器，可以包含多个Service服务。





## 05、Tomcat：配置Executor

Executor用来定义多个Connector连接器可以共享的线程池，如果想添加一个执行器，配置如下:

```xml
<!-- 执行器线程池 -->
<Executor name="tomcatThreadPool"
          namePrefix="catalina-exec-"
          maxThreads="800"
          minSpareThreads="20"
          maxIdleTime="60000"
          className="org.apache.catalina.core.StandardThreadExecutor"/>
```

![img](就业技术加强-04-Tomcat高级.assets/20151022113805905.png)

**属性说明:**

| 属性                    | 含义                                                         |
| ----------------------- | ------------------------------------------------------------ |
| name                    | 执行器的名称，用于在Connector中指定。                        |
| namePrefix              | 所创建的每个线程的名称前缀，一个单独的线程名称为 namePrefix-thread-Number。 |
| maxThreads              | 线程池中最大线程数。                                         |
| maxSpareTHreads         | 最大空闲线程数，若空闲时间大于最大空闲时间，则回收，小于则继续存活，等待被调度。 |
| minSpareThreads         | 最小空闲线程数，无论如何都会存活的线程数.                    |
| maxIdleTime             | 最大空闲时间，超过这个空闲时间，且线程数大于最小空闲数的，都会被回收，默认值为60000毫秒。 |
| acceptCount             | 最大等待队列数，请求并发大于tomcat线程池的处理能力，则被放入等待队列等待被处理。 |
| maxQueueSize            | 在被执行前最大线程排队数量，默认为Int的最大值，也就是广义的无限。除非特殊情况，这个值不需要更改，否则会有请求不会被处理的情况发生。 |
| prestartminSpareThreads | 启动前tomcat最小空闲线程数与minSpareThreads的值一样，不设置时默认为: true。 |
| threadPriority          | 线程池中线程优先级，默认值为5，值从1到10。                   |
| className               | 线程池实现类，默认实现类为org.apache.catalina.core.StandardThreadExecutor。 |

![1592387010363](就业技术加强-04-Tomcat高级.assets/1592387010363.png)  

说明: 如果不配置执行器，那么Connector连接器也会创建自己的线程池。





## 06、Tomcat：配置Connector

> 目标: 了解Connector连接器的作用。

#### 6.1 连接器

Connector用于**接收请求、返回响应**，默认情况下: server.xml配置了两个连接器，一个支持HTTP协议，一个支持AJP协议。大多数情况下，我们并不需要新增连接器配置，只是根据需要对已有连接器进行优化。

```xml
<!-- HTTP协议: 处理动态和静态资源 -->
<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
<!-- AJP协议: 整合Apache -->
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
```

**属性说明**

+ port: 端口号，Connector用于创建服务端Socket并进行监听，以等待客户端请求链接。如果该属性设置为0，Tomcat将会随机选择一个可用的端口号给当前Connector使用。 

+ protocol: 当前Connector支持的访问协议。**默认为 HTTP/1.1** 就是NIO，并采用自动切换机制选择一个基于 JAVA NIO的连接器或者基于本地APR的连接接器（根据本地是否含有Tomcat的本地库判定）。如果不希望采用上述自动切换的机制，而是明确指定协议，可以使用以下值。

  + Http协议:

    > Tomcat7: bio(阻塞式I/O模式)  Tomat8:nio(非阻塞式I/O模式)
    >
    > **性能: BIO < NIO < NIO2 < APR**
    >
    > APR: 就是从操作系统级别解决异步IO问题(需要安装依赖库)。

    ```xml
    org.apache.coyote.http11.Http11NioProtocol: 非阻塞式Java NIO模式
    org.apache.coyote.http11.Http11Nio2Protocol: 非阻塞式JAVA NIO2模式
    org.apache.coyote.http11.Http11AprProtocol: 操作系统级别解决异步IO(需要安装依赖库)
    ```

    ```xml
    <Connector port="8080" protocol="org.apache.coyote.http11.Http11Nio2Protocol"
                   connectionTimeout="20000"
                   redirectPort="8443" />
    ```

    ![1592384530380](就业技术加强-04-Tomcat高级.assets/1592384530380.png) 

    

+ connectionTimeOut: 连接等待超时时间，单位为(毫秒)，-1表示不超时。

+ redirectPort: 当前Connector不支持SSL请求，接收到了一个请求，将请求重定向到指定的端口。

+ executor: 指定共享线程池的名称。

+ URIEncoding: 用于指定URI的字符编码。Tomcat8.x: UTF-8，Tomcat7.x: ISO-8859-1。

+ acceptCount: 当所有可能的请求处理线程都在使用时，传入的连接请求的最大队列长度。当队列满时收到的任何请求都将被拒绝。缺省值为100。

  完整配置如下:

  ```xml
  <Connector port="8080"
             protocol="HTTP/1.1"
             maxThreads="1000"
             minSpareThreads="100"
             acceptCount="1100"
             connectionTimeout="20000"
             redirectPort="8443"
             URIEncoding="UTF-8"/>
  ```



**什么是AJP呢？**

AJP（Apache JServer Protocol） AJPv13协议是面向包的。WEB服务器和Servlet容器通过TCP连接来交互；为了节省SOCKET创建的昂贵代价，WEB服务器会尝试维护一个永久TCP连接到servlet容器，并且在多个请求和响应周期过程会重用连接。

![image-20210114104700649](就业技术加强-04-Tomcat高级.assets/image-20210114104700649.png)

有两种方式请求web服务:
第一种：请求tomcat服务器8080 端口 直接请求
第二种：生产环境常用，在web用户和tomcat之间架设一个web服务器，用户首先请求Web服务器，然后由web服务器来请求tomcat,这样做的好处是在前面的web服务器可以做集群和缓存，如果web服务器和tomcat做的是短连接，这样的话就需要经常重新连接效率低，tomcat在这里做了一个优化，架设了一个连接器叫做ajp的长连接，这样就减少了创建连接和关闭连接的次数，但是AJP服务只有Apache服务器才能使用。我们一般不适用apache服务，我们一般是使用Nginx+tomcat的架构，所以用不着AJP协议，所以把AJP连接器禁用；其实换个角度想想如果一个AJP服务我们用不到但是开起着，肯定会造成资源的浪费。



## 07、Tomcat：容器Container

Tomcat设计了4种容器: Engine、Host、Context、Wrapper。这4种容器不是平行关系，而是父子关系。Tomcat通过一种分层的架构，使得Servlet容器具有很好的灵活性。

![1560686115687](就业技术加强-04-Tomcat高级.assets/1560686115687-1576634260081.png) 

组件的含义: 

| 容器    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| Engine  | 引擎容器，用来管理多个虚拟主机，一个Service最多只能有一个Engine，但是一个引擎可包含多个Host。**单机部署多应用** |
| Host    | 代表一个虚拟主机，或者说一个站点，可以给Tomcat配置多个虚拟主机地址，而一个虚拟主机下可包含多个Context。 |
| Context | 表示一个Web应用程序，一个Web应用可包含多个Wrapper(Servlet)。 |
| Wrapper | 表示一个Servlet，Wrapper作为容器中的最底层，不能包含子容器。 |

Tomcat采用了组件化的设计，它的构成组件都是可配置的，其中最外层的是Server，其他组件按照一定的格式要求配置在服务器中。

![1560854103740](就业技术加强-04-Tomcat高级.assets/1560854103740-1576634260082.png) 

那么，Tomcat是怎么管理这些容器的呢？你会发现这些容器具有父子关系，形成一个树形结构，你可能马上就想到了设计模式中的组合模式。没错，Tomcat就是用组合模式来管理这些容器的。具体实现方法是，所有容器组件都实现了Container接口。

![1592552626354](就业技术加强-04-Tomcat高级.assets/1592552626354.png) 





## 08、Tomcat：配置Engine

Engine引擎容器，是容器对外提供功能的入口，用于管理各个Realm(角色安全管理)、Host等。

```xml
<Engine name="Catalina" defaultHost="localhost">
	...
</Engine>
```

**属性说明:**

+ name: 用于指定Engine的名称，默认为Catalina。
+ defaultHost: 默认使用的虚拟主机名称，当客户端请求指向的主机无效时，将交由默认的虚拟主机处理， 默认为localhost。





## 09、Tomcat：配置Host

Host元素用于配置一个虚拟主机，可以嵌入子元素: Listener、Valve、Realm、Context。



**Value子标签介绍**

Valve子标签是用来配置Tomcat日志的。

directory: 日志存放的目录

prefix: 日志文件的开头名称

suffix: 日志文件的后缀

pattern: 日志文件内容的格式

```xml
<Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
	<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
		prefix="localhost_access_log" suffix=".txt"
		pattern="%h %l %u %t &quot;%r&quot; %s %b" />

</Host>
```



**Host标签介绍**

```xml
<Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
    ...
</Host>
```

**属性说明**

+ name: 当前的虚拟主机名称(**域名、localhost**)，Engine包含的Host必须存在一个名称与Engine的defaultHost一致。
+ appBase: Web应用部署的基础目录(可以是绝对目录，相对路径)，默认为webapps。
+ unpackWARs: 设置是否自动解压war包(true: 解压、false: 不解压)。
+ autoDeploy: 在运行时定期检测并自动部署Web应用。

**域名映射**

+ 修改C:\Windows\System32\drivers\etc\hosts文件:

  ```properties
  127.0.0.1 		www.weixin.com
  127.0.0.1 		www.yaoyiyao.com
  127.0.0.1 		www.yueyiyue.com
  ```

**单机部署多应用**

+ 单机部署多个项目: 说的是能不能在tomcat中部署多个项目，并且都不需要指定项目路径，解决方案就是配置多个HOST。

  ```xml
  <Host name="www.7bhd.com"  appBase="webapps1" unpackWARs="true" autoDeploy="true">
  </Host>
  
  <Host name="www.lxhsq.com"  appBase="webapps2" unpackWARs="true" autoDeploy="true">
  </Host>
  ```

  ![1592559369692](就业技术加强-04-Tomcat高级.assets/1592559369692.png)  





## 10、Tomcat：配置Context

Context用于配置一个Web应用，默认的配置如下: 

```xml
<Context docBase="ROOT" path="/" />
```

**属性描述**

+ docBase: Web应用目录或者war包的部署路径。可以是绝对路径，也可以是相对于Host appBase的相对路径。

+ path: Web应用访问的上下文路径。如果我们Host名为localhost，则该web应用访问的根路径为:

  http://localhost:8080/ + path指定的路径。

  配置一(相对路径):

  ```xml
  <Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
       <Context docBase="docs" path="/"></Context>
       <Valve className="org.apache.catalina.valves.AccessLogValve"
              directory="logs" prefix="localhost_access_log" suffix=".txt"
              pattern="%h %l %u %t &quot;%r&quot; %s %b" />
  </Host>
  ```

  配置二(绝对路径):

  ```xml
  <Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
       <Context docBase="F:\docs" path="/app"></Context>
       <Valve className="org.apache.catalina.valves.AccessLogValve"
              directory="logs" prefix="localhost_access_log" suffix=".txt"
              pattern="%h %l %u %t &quot;%r&quot; %s %b" />
  </Host>
  ```

  



## 11、Tomcat：配置web.xml

面试题: 在web应用中有这三个主页，走哪一个，如何修改默认主页？

```
index.html
index.htm
index.jsp
```



修改conf/web.xml

```html
<welcome-file-list>
    <welcome-file>hehe.html</welcome-file>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
</welcome-file-list>
```



## 12、Tomcat：请求流程

+ 设计了这么多层次的容器，Tomcat是怎么确定每一个请求应该由哪个Wrapper容器里的Servlet来处理的呢？答案是，Tomcat是用Mapper组件来完成这个任务的。

+ Mapper组件的功能就是将用户请求的URL定位到一个Servlet，它的工作原理是: Mapper组件里保存了Web应用的配置信息，其实就是容器组件与访问路径的映射关系，比如Host容器里配置的域名、Context容器里的Web应用路径，以及Wrapper容器里Servlet映射的路径，你可以想象这些配置信息就是一个多层次的Map。

+ 当一个请求到来时，Mapper组件通过解析请求URL里的域名和路径，再到自己保存的Map里去查找，就能定位到一个Servlet。一个请求URL最后只会定位到一个Wrapper容器，也就是一个Servlet。

+ 下面的示意图中，就描述了 当用户请求链接 ``` http://www.itcast.cn/bbs/findAll ``` 之后, 是如何找到最终处理业务逻辑的servlet。

  ![1592707253701](就业技术加强-04-Tomcat高级.assets/1592707253701.png) 





## 13、Jmeter：安装和使用

> 目标: 掌握Jmeter工具的安装与使用

#### 13.1 Jmeter安装

**操作步骤**

+ 官方网址: <https://jmeter.apache.org/>

  ![1592709585279](就业技术加强-04-Tomcat高级.assets/1592709585279.png) 

+ 下载地址: <https://mirrors.tuna.tsinghua.edu.cn/apache/jmeter/binaries/>

  ![1592709733607](就业技术加强-04-Tomcat高级.assets/1592709733607.png) 

+ 学习参考地址: https://www.cnblogs.com/jessicaxu/p/7501770.html 

+ 解压apache-jmeter-5.3.zip

  ![1592710717369](就业技术加强-04-Tomcat高级.assets/1592710717369.png)  

+ 双击打开`bin/jmeter.bat`，并修改语言

  ![1592710867844](就业技术加强-04-Tomcat高级.assets/1592710867844.png)

  ![1592710902743](就业技术加强-04-Tomcat高级.assets/1592710902743.png)   

  ![1592711535193](就业技术加强-04-Tomcat高级.assets/1592711535193.png) 



#### 13.2 Jmeter使用

![image-20210114202813985](就业技术加强-04-Tomcat高级.assets/image-20210114202813985.png)





+ 添加线程组

  ![1592713027567](就业技术加强-04-Tomcat高级.assets/1592713027567.png) 

  ![1592713106163](就业技术加强-04-Tomcat高级.assets/1592713106163.png) 

  ![1592713249676](就业技术加强-04-Tomcat高级.assets/1592713249676.png)  

+ 添加测试用例

  ![1592713346081](就业技术加强-04-Tomcat高级.assets/1592713346081.png) 

  ![1592713583001](就业技术加强-04-Tomcat高级.assets/1592713583001.png)  

+ 添加察看结果树

  ![1592713780600](就业技术加强-04-Tomcat高级.assets/1592713780600.png)

  ![1592713883862](就业技术加强-04-Tomcat高级.assets/1592713883862.png) 

+ 添加聚合报告

  ![1592713912679](就业技术加强-04-Tomcat高级.assets/1592713912679.png) 

  ![1592713958135](就业技术加强-04-Tomcat高级.assets/1592713958135.png)  

  ```txt
  Label: 每个请求的名称，比如HTTP请求等
  Samples: 发给服务器的请求数量
  Average: 单个请求的平均响应时间 毫秒ms
  Median:  50%请求的响应时间 毫秒ms
  90%Line: 90%请求的响应时间 毫秒ms
  95%Line: 95%请求的响应时间 毫秒ms
  99%Line: 99%请求的响应时间 毫秒ms
  Min: 最小的响应时间 毫秒ms
  Max: 最大的响应时间 毫秒ms
  Error%: 错误率=错误的请求的数量/请求的总数
  Throughput: 吞吐量，默认情况下表示每秒完成的请求数（Request per Second）
  Received KB/sec: 每秒从服务器端接收到的数据量
  Sent KB/sec: 每秒从客户端发送的请求的数量
  ```

+ 部署项目(ROOT.war)到tomcat，进行压测

  ![1592722392145](就业技术加强-04-Tomcat高级.assets/1592722392145.png) 

  修改server.xml:

  ![1592722424112](就业技术加强-04-Tomcat高级.assets/1592722424112.png) 

  用Jmeter压测:

  ![1592726639653](就业技术加强-04-Tomcat高级.assets/1592726639653.png) 

  ![1592727468967](就业技术加强-04-Tomcat高级.assets/1592727468967.png) 

  

+ 测试的原则

  + 一般没有业务逻辑的接口是不会优先考虑去测试。

  + 一般压测在最开始都去测试哪些并发比较大的接口或者业务（抢购，秒杀，注册，登录），也告诉各位一个道理: 不是所有的接口都需要进行测试。

    





## 14、Tomcat：配置调优

#### 14.1 禁用AJP连接器

默认状态下，Tomcat会启动AJP服务，占用8009端口。

AJP是什么: AJP（Apache JServer Protocol）是为Tomcat与Apache服务器之间通信而定制的协议，能提供较高的通信速度和效率。

Tomcat虽然是一个Web服务器，可以对静态资源进行解析，但它最主要的作用是提供Servlet和jsp容器，对静态资源的解析肯定不如一些专业的Http服务器(apache、nginx)。所以通常生产环境会把Tomcat和其他http服务器搭配使用，所有请求都访问http服务器，如果访问的资源是Servlet或jsp则http服务器将请求交由Tomcat处理，如果静态资源http服务器直接处理，ajp就是Apache服务器和Tomcat服务器通信的主要协议，如果你不使用Tomcat和Apache服务器整合的话，这个AJP协议服务是没必要开启的。

注释它就可以禁用AJP，提升一点点CPU利用率:

```xml
<!-- <Connector port="8009" protocol="AJP/1.3" redirectPort="8443"/> -->
```

![1592788676280](就业技术加强-04-Tomcat高级.assets/1592788676280.png) 



#### 14.2 Executor线程池调优

默认Tomcat会为每个连接器创建一个线程池(最大线程数200)，服务启动时，默认创建10个空闲线程等待用户请求。如果想要多个Connector连接器都用同一个线程池，就可以让他们都绑定到同一个执行器。

修改执行器:

```xml
<!-- 
  name: 执行器名称
  namePrefix: 线程名称前缀
  maxThreads: 最大线程数
  minSpareThreads: 最小空闲线程数
  maxIdleTime: 最大空闲时间
 -->
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
          maxThreads="600" minSpareThreads="50"
          maxIdleTime="60000"/>
```

修改Connector节点，增加executor属性:

```xml
<Connector port="8080" protocol="org.apache.coyote.http11.Http11Nio2Protocol"
           executor="tomcatThreadPool"
           connectionTimeout="20000"
           redirectPort="8443" />
```

1 最大线程数50，初始为10

2 最大线程数100，初始为10

3 最大线程数200，初始为10

4 最大线程数200，初始为200

5 最大线程数500，初始为500

6 最大线程数5000，初始为5000



最大线程要根据自己的实际情况合理设置，设置越大会耗费内存和CPU，因为CPU疲于线程上下文切换，没有精力提供请求服务了当然允许的最大线程连接数还受制于操作系统的内核参数设置，设置多大要根据自己的需求与环境。





#### 14.3 Connector调优

给连接器配置了线程池，就可以不用配置Connector默认的线程池

```xml
<!-- 配置连接器Connector -->
<Connector port="8080" protocol="org.apache.coyote.http11.Http11Nio2Protocol"
           maxThreads="600"  
           minSpareThreads="100"
           maxIdleTime="60000"
           URIEncoding="UTF-8"
           connectionTimeout="2000"
           acceptCount="300"
           redirectPort="8443"/>
```

连接器参数:

+ maxThreads: 最大线程数，默认值是: 200。
+ minSpareThreads: 最小空闲线程数，默认值是: 10。
+ URIEncoding: 设置请求URL的编码方式(针对get请求)。
+ connnectionTimeout: 网络连接超时，一般设置2000毫秒，设置为0: 表示永不超时。
+ acceptCount: 最大排队数，当请求超过maxThreads时，会把请求放到队列中，超过最大排队数的请求将不予处理，默认为100。



#### 14.4 设置NIO2运行模式

Tomcat连接器IO模式: BIO、NIO、NIO2、APR，四种方式性能差别大，APR性能最优，BIO性能最差。

+ BIO(阻塞式I/O模式): Tomcat7默认。

+ NIO(非阻塞式I/O模式): Tomat8默认。

  > Tomcat8提供了NIO2，效率要比NIO快些，所以如果是Tomcat8之前的版本推荐使用NIO，如果是Tomcat8及之后的版本都使用NIO2。

+ APR(Apache Portable Runtime): 就是从操作系统级别解决异步IO问题(需要安装依赖库)。

```
org.apache.coyote.http11.Http11NioProtocol: 非阻塞式Java NIO模式
org.apache.coyote.http11.Http11Nio2Protocol: 非阻塞式JAVA NIO2模式
org.apache.coyote.http11.Http11AprProtocol: 操作系统级别解决异步IO(需要安装依赖库)
```



#### 14.5 Tomcat架构调优

前面我们已经通过jvm、tomcat自身来提升tomcat的性能，但tomcat的性能是有极限的，接下来我们聊聊如何在架构层面，对Tomcat进行优化。这里面我们主要讨论两个优化思路: **动静分离**、**Tomcat集群**

搭建Tomcat的集群，目前比较流程的做法就是通过Nginx来实现Tomcat集群的负载均衡。

![1592805253907](就业技术加强-04-Tomcat高级.assets/1592805253907.png)  

**动静分离**

+ 什么是动静分离:

  动静分离是让网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们就可以根据静态资源的特点将其做缓存操作，这就是网站静态化处理的核心思路，动静分离简单的概括是: 动态文件与静态文件的分离。

+ 为什么需要动静分离:

  Tomcat虽然能够处理静态资源，但它处理静态资源的能力不如apache或nginx等专门的web服务器，Tomcat是处理Servlet/jsp程序的首选。而我们的java项目中会有好多静态资源，这些静态资源的请求全部走Tomcat的话一方面是性能不高，另一方面每个请求都会占用Tomcat宝贵的线程资源。所以在互联网高并发项目中动静分离就显得十分重要。

+ 静态资源: 一般客户端发送请求到web服务器，web服务器直接找到对应的资源，原封不动的直接返回给客户端（浏览器），客户端解析并渲染显示出来。

+ 动态资源: 一般客户端请求的动态资源，web服务器端的程序(Servlet/jsp)处理过后生成对应的相应结果，返回给客户端解析渲染处理。

  > 动静分离: Nginx处理静态资源、Tomcat处理动态资源。

**Tomcat集群**

+ 互联网架构常用名词: 分布式、集群、负载均衡。

  + 分布式: 一个任务分给多台机器去做，减少单个任务的执行时间。
  + 集群: 集群是一组协同工作的服务集合,一般由两个或者两个以上的服务器组成。
  + 负载均衡: 将负载（任务）平衡的分摊到各个执行单元上。

+ Tomcat集群: 指定的是2台或2台以上的tomcat部署同样的项目，协同提供服务。

  + 提高性能: 单台tomcat处理能力有限，多台tomcat一起提供服务会起到事半功倍的效果。

  + 防止单点故障: 另一方面，单台tomcat提供服务 很容易出现单点故障，集群可以实现高可用。

    > 缺点: 虽然集群能够大大提高服务性能，实现高可用，但有了集群之后也带来很多架构处理的问题: session共享、分布式锁等等。



## 15、Tomcat：JVM调优

Tomcat是一款Java应用，那么JVM的配置便与其运行性能密切相关，而JVM优化的重点则集中在内存分配和GC策略的调整上，因为内存会直接影响服务的运行效率和吞吐量，JVM垃圾回收机制则会不同程度地导致程序运行中断。可以根据应用程序的特点，选择不同的垃圾回收策略，调整JVM垃圾回收策略，可以极大减少垃圾回收次数，提升垃圾回收效率，改善程序运行性能。

#### 15.1 JVM内存分配

+ JVM内存参数

  | 参数                 | 参数作用                                | 优化建议                |
  | -------------------- | --------------------------------------- | ----------------------- |
  | -server              | 启动Server，以服务端模式运行            | 服务端模式建议开启      |
  | -Xms                 | 最小堆内存                              | 建议与-Xmx设置相同      |
  | -Xmx                 | 最大堆内存                              | 建议设置为可用内存的80% |
  | -XX:MetaspaceSize    | 元空间初始内存(jdk1.7 PremSize非堆内存) | 不建议修改              |
  | -XX:MaxMetaspaceSize | 元空间最大内存(jdk1.7 PremSize非堆内存) | 不建议修改              |

  windows系统(catalina.bat):

  ```bat
  set JAVA_OPTS=-server -Xms256m -Xmx256m -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=256m
  ```

  ![1592731742041](就业技术加强-04-Tomcat高级.assets/1592731742041.png) 




设置堆大小为256M，并行垃圾回收器

```shell
set JAVA_OPTS=-server -Xms128m -Xmx128m -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=256m ‐XX:+UseParallelGC
```

设置堆大小为128M，并行垃圾回收器

设置堆大小为512M，并行垃圾回收器

设置堆大小为1024M，并行垃圾回收器

设置堆大小为8192M，并行垃圾回收器

设置堆大小为8192M，G1垃圾回收器





## 16、Linux部署Tomcat远程调优

+ 查看Tomcat默认的垃圾收集器:

  + 上传apache-tomcat-8.5.42.tar.gz到linux系统。

  + 配置`tomcat/bin/catalina.sh`，并启动tomcat:

    ```shell
    JAVA_OPTS="-server -Xms512m -Xmx512m -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=256m -Djava.rmi.server.hostname=192.168.100.80 -Dcom.sun.management.jmxremote.port=8999 -Dcom.sun.management.jmxremote.rmi.port=8999 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false"
    ```

    ![1592732319620](就业技术加强-04-Tomcat高级.assets/1592732319620.png) 

  + 打开jconsole，远程连接tomcat

    ![1592732843840](就业技术加强-04-Tomcat高级.assets/1592732843840.png) 

    ![1592732872727](就业技术加强-04-Tomcat高级.assets/1592732872727.png) 

  + 查看远程tomcat的概要信息

    ![1592811987281](就业技术加强-04-Tomcat高级.assets/1592811987281.png)    

    


## 16、Tomcat：常见面试题总结

+ 问题1: 您对BIO与NIO的理解?

  ```txt
  BIO: 传统的java.io包，同步、阻塞。服务器实现模式为一个连接一个线程，服务器端为每一个客户端的连接请求都需要启动一个线程进行处理。
  
  NIO: JDK1.4引入的java.nio包，采用多路复用技术，同步非阻塞。服务器实现模式为客户端的连接请求都会注册到多路复用器上，用同一个线程接收所有连接请求。
  ```

+ 问题2: 网站的并发量是多少?

  ```txt
  具体多大我不能直接给你一个具体数字，因为tomcat并发量我们需要通过压测软件测试我才能知道当前tomcat在当前电脑的这种的配置下具体是多少，要根据具体报告来得到答案，根据我自己在上一家公司的经验: 一台8核8代i7 16G内存的电脑进行项目核心接口测试大概是:800-1500 稳定在:1100左右。
  ```

+ 问题3: tomcat性能如何调优?

  ```txt
  1. 对Tomcat的配置文件进行调优
     a.禁用AJP连接器
     b.配置Executor线程池(最大线程数量、最小线程数量、回收空闲线程)
     c.配置Connector连接器，设置为NIO2运行模式
  2. 对Tomcat架构进行调优
  	a.单台Tomcat承载量有限,用nginx对tomcat做集群，实现负载均衡
  	b.Tomcat比较擅长动态资源JSP/Servlet的处理,静态资源交给Nginx去处理  CDN
  3. JVM调优
  	a.调整堆内存的大小
  	b.选择合适的垃圾收集器
  	
  4.代码优化  ArrayList/LinkedList
  ```

+ 问题4: 您用过哪些压测工具?

  ```txt
  1. apache的ab命令
  2. Jmeter 压力测试工具
  3. JCStress 多线程调试工具
  ```





## 17、扩展-代码优化

优化，不仅仅是在运行环境进行优化，还需要在代码本身做优化，如果代码本身存在性能问题，那么在其他方面再怎么优化也不可能达到效果最优的。



**尽可能使用局部变量**

调用方法时传递的参数以及在调用中创建的临时变量都保存在栈中速度较快，其他变量，如静态变量、实例变量等，都在堆中创建，速度较慢。另外，栈中创建的变量，随着方法的运行结束，这些内容就没了，不需要额外的垃圾回收。

**尽量减少对变量的重复计算**

明确一个概念，对方法的调用，即使方法中只有一句语句，也是有消耗的。所以例如下面的操作：

```java
for (int i = 0; i < list.size(); i++) {
	...
}
```

建议替换为：

```java
int length = list.size();
for (int i = 0,  i < length; i++) {
	...
}
```



这样，在list.size()很大的时候，就减少了很多的消耗。



**尽量采用懒加载的策略，即在需要的时候才创建**

```
String str = "aaa";
if (i == 1) {
　　list.add(str);
}

//建议替换成
if (i == 1) {
　　String str = "aaa";
　　// ...
　　list.add(str);
}
```



**异常不应该用来控制程序流程**

```
异常对性能不利。抛出异常首先要创建一个新的对象，Throwable接口的构造函数调用名为fillInStackTrace()的本地同步方 法，fillInStackTrace()方法检查堆栈，收集调用跟踪信息。只要有异常被抛出，Java虚拟机就必须调整调用堆栈，因为在处理过程中创建了一个新的对象。异常只能用于错误处理，不应该用来控制程序流程。
```



**不要将数组声明为public static final**

因为这毫无意义，这样只是定义了引用为static final，数组的内容还是可以随意改变的，将数组声明为public更是一个安全漏洞，这意味着这个数组可以被外部类所改变。

public static final int[] arr = new int[10];

int[] arr = new int[20];

arr[0] = 10;



**不要创建一些不使用的对象，不要导入一些不使用的类**

这毫无意义，如果代码中出现"The value of the local variable i is not used"、"Theimport java.util is never used"，那么请删除这些无用的内容



**程序运行过程中避免使用反射**

反射是Java提供给用户一个很强大的功能，功能强大往往意味着效率不高。不建议在程序运行过程中使用尤其是频繁使用反射机制，特别是 Method的invoke方法。如果确实有必要，一种建议性的做法是将那些需要通过反射加载的类在项目启动的时候通过反射实例化出一个对象并放入内存。



**使用数据库连接池和线程池**(池化技术)

这两个池都是用于重用对象的，前者可以避免频繁地打开和关闭连接，后者可以避免频繁地创建和销毁线程。



**容器初始化时尽可能指定长度**

容器初始化时尽可能指定长度，如：new ArrayList<>(10); new HashMap<>(32); 避免容器长度不足时，扩容带来的性能损耗。



**ArrayList随机遍历快，LinkedList添加删除快**

**使用Entry遍历Map**

```java
Map<String,String> map = new HashMap<>();
for (Map.Entry<String,String> entry : map.entrySet()) {
    String key = entry.getKey();
    String value = entry.getValue();
}
```

避免使用这种方式：

```java
Map<String,String> map = new HashMap<>();
for (String key : map.keySet()) {
    String value = map.get(key);
}
```



**不要手动调用System.gc();**



**String尽量少用正则表达式**

正则表达式虽然功能强大，但是其效率较低，除非是有需要，否则尽可能少用。
replace() 不支持正则
replaceAll() 支持正则

如果仅仅是字符的替换建议使用replace()。



**对资源的close()建议分开操作**

```java
try {
    XXX.close();
    YYY.close();
} catch (Exception e) {
    ...
}

// 建议改为
try {
    XXX.close();
} catch (Exception e) {
    ...
}

try {
    YYY.close();
} catch (Exception e) {
    ...
}
```

