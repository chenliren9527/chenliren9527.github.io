---
title: 就业技术加强(05)-DUBBO&Zookeeper(2)
tags:
  - 笔记
  - 就业技术加强
  - DUBBO
  - Zookeeper
  - 集群
  - 优化
categories:
  - 就业技术加强
abbrlink: 19642
date: 2021-01-28 17:34:01
---

## 0、课程目标

目标1: 能够配置dubbo-springboot工程

目标2: 能够配置dubbo的容错和降级

目标3: 能够说出dubbo的负载均衡策略

目标4: 能够应用dubbo的灰度发布

目标5: 能够说出dubbo的常见面试题

## 1、DUBBO架构

Apache Dubbo 是一款高性能、轻量级的开源 Java 服务框架。



Apache Dubbo |ˈdʌbəʊ| 提供了六大核心能力：面向接口代理的高性能RPC调用，智能容错和负载均衡，服务自动注册和发现，高度可扩展能力，运行期流量调度，可视化的服务治理与运维。



![image-20201225191632016](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225191632016.png)



**图例说明**：

- 图中小方块 Protocol, Cluster, Proxy, Service, Container, Registry, Monitor 代表层或模块，蓝色的表示与业务有交互，绿色的表示只对 Dubbo 内部交互。
- 图中背景方块 Consumer, Provider, Registry, Monitor 代表部署逻辑拓扑节点。
- 图中蓝色虚线为初始化时调用，红色虚线为运行时异步调用，红色实线为运行时同步调用。
- 图中只包含 RPC 的层，不包含 Remoting 的层，Remoting 整体都隐含在 Protocol 中。



**调用关系说明：**

0. 服务容器负责启动，加载，运行服务提供者。

1. 服务提供者在启动时，向注册中心注册自己提供的服务。

2. 服务消费者在启动时，向注册中心订阅自己所需的服务。

3. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。

4. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。

5. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

 

【软负载均衡】软件负载均衡则是通过在服务器上安装的特定的负载均衡软件或是自带负载均衡模块完成对请求的分配派发。如：轮询法、随机法、源地址哈希法、最小连接数法等。在消费方中声明服务的时候可以指定负载均衡的策略，dubbo在返回的服务地址列表中使用负载均衡策略选择一个服务地址；默认是使用随机法。



## 2、DUBBO SpringBoot

使用DUBBO 及 Spring Boot的工程，通过如下的案例来了解其整合。

### 2.1、导入案例工程

#### 1）创建空工程

![image-20201225211919776](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225211919776.png)

![image-20201225213243775](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225213243775.png)

![image-20201225213413917](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225213413917.png)

创建好一个空工程是如下的目录结构：

![image-20201225213511930](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225213511930.png)

#### 2）导入案例工程

复制 `资料\案例工程\dubbo-springboot` 到刚刚创建的空工程目录下；

![image-20201225213705391](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225213705391.png)

![image-20201225213739109](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225213739109.png)

![image-20201225213826129](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225213826129.png)



导入完成后如下：

![image-20201225213844081](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225213844081.png)

`goods-service` 表示商品系统，既是服务提供者也是服务消费者；

`order-service` 表示订单系统，既是服务提供者也是服务消费者；



### 2.2、注册中心

Dubbo默认支持五种注册中心： 推荐使用 [Zookeeper 注册中心](http://dubbo.apache.org/zh-cn/docs/user/references/registry/zookeeper.html) 

[Zookeeper](http://zookeeper.apache.org/) 是 Apacahe Hadoop 的子项目，是一个树型的目录服务，支持变更推送，适合作为 Dubbo 服务的注册中心，工业强度较高，可用于生产环境，并推荐使用。



先使用windows版本；解压 `资料\zookeeper-3.4.14.zip` ；

进入解压后目录并双击启动 `zookeeper-3.4.14\bin\zkServer.cmd`

![image-20201225214233070](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225214233070.png)

### 2.3、导入DUBBO监控中心

可以到 dubbo-admin.zip(https://github.com/apache/dubbo-admin) 下载

git clone https://github.com/apache/dubbo-admin.git(克隆仓库)

![image-20201225215303175](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225215303175.png)



#### 1）解压

解压 `资料\dubbo-admin-master.zip` 到创建的空工程目录下如下：

![image-20201225214505292](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225214505292.png)

#### 2）导入工程

将在工程目录下的 `dubbo-admin-master` 导入到IDEA中：

![image-20201225214529395](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225214529395.png)

![image-20201225214633695](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225214633695.png)

![image-20201225214654381](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225214654381.png)



## 3、DUBBO优化配置

### 3.1、启动时检查

#### 1）问题

- 在实际的项目开发时，如果服务提供者没有启动，这个时候消费者就会报错影响项目的运行；
- 在项目之间如果出现服务之间相互调用，那么也会出现启动时错误

![image-20201225220128072](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225220128072.png)

#### 2）简介

官方文档: <http://dubbo.apache.org/zh-cn/docs/user/demos/preflight-check.html>

+ dubbo 缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止 Spring 初始化完成，以便上线时，能及早发现问题，默认 `check="true"`。
+ 可以通过 `check="false"` 关闭检查，比如，测试时，有些服务不关心，或者出现了循环依赖，必须有一方先启动。
+ 另外，如果你的 Spring 容器是懒加载的，或者通过 API 编程延迟引用服务，请关闭 check，否则服务临时不可用时，会抛出异常，拿到 null 引用，如果 `check="false"`，总是会返回引用，当服务恢复时，能自动连上。

#### 3）使用

- **方式一**：注解中局部使用`@DubboReference(check=true|false)` ；check的默认值为`true`



```java
@RestController
@RequestMapping("/order")
public class OrderController {

    /**
     * check = false 表示启动时不检查服务是否可用
     */
    @DubboReference(
            check = false
    )
    private GoodsService goodsService;
```



- **方式二**：全局配置在`application.yml`文件中；

+ 配置(格式一)：dubbo.reference.(包名+接口类名).check=false

  ```yml
  ...
  dubbo:
    registry:
      address: zookeeper://127.0.0.1:2181
      timeout: 10000
    reference:
      com.itheima.goods.service.GoodsService:
        check: false
  ...
  ```

  配置(格式二)：dubbo.consumer.check=false

  ```yaml
  # 关闭全部的服务接口启动时不检查
  dubbo:
    consumer:
    	check: false
  ```

  **注意:**

  如果全局配置了check，局部也配置了check, dubbo会采用并且的方式得到最终的check；

  ​     **启动时检查check = 全局的check && 局部的check**



+ 能解决什么问题？

  ```txt
  如果在开发时候可能一个服务已依赖很多提供者，如果这个时候仅仅在开发order服务，其他的服务就必须运行起来，才能够去开发，这样可能会影响开发效率。开启false后只需要开启当前开发的服务
  ```

+ 到底要不要把check改成false？

  ```开txt
  开发时改成false，上线时尽量改成true
  ```

+ 用全局配置还是局部配置，为什么？

  ```txt
  用全局的配置；修改灵活方便
  ```

### 3.2、结果缓存

结果缓存，用于加速热门数据的访问速度，Dubbo 提供声明式缓存，以减少用户加缓存的工作量。

**缓存类型**

- `lru` 基于最近最少使用原则删除多余缓存，保持最热的数据被缓存。
- `threadlocal` 当前线程缓存，比如一个页面渲染，用到很多 portal，每个 portal 都要去查用户信息，通过线程缓存，可以减少这种多余访问。
- `jcache` 与 [JSR107](http://jcp.org/en/jsr/detail?id=107') 集成，可以桥接各种缓存实现。

```java
@RequestMapping("/order")
public class OrderController {

    /**
     * check = false 表示启动时不检查服务是否可用
     * cache = "lru" 调用服务的所有方法时候使用 最近最少使用 淘汰策略方式删除缓存结果
     * methods 则是针对服务中的特定方法
     */
    @DubboReference(
            check = false,
            //cache = "lru",
            methods = {@Method(name = "findGoodsByOrderId", cache = "lru")}
    )
    private GoodsService goodsService;

    @Resource
    private OrderService orderService;
```

> 不管是开发还是生产环境；一般都不开启。缓存要根据系统业务逻辑设计、处理

### 3.3、集群容错

为了服务的稳定性，一般会有多个服务提供者提供服务，也即`服务集群`；而 `消费者` 在调用服务的时候如果遇到一个提供者失败之后可以再次尝试其它提供者获取服务。这也称为 `集群容错`。

#### 3.3.1、案例1

- **需求**

> 访问订单系统（order-service） http://localhost:9200/order/1 获取商品系统（goods-service）中订单对应的商品列表；其中，在商品系统中根据订单查询商品需要耗时2000毫秒。
>
> 商品系统goods-service 提供3台集群节点服务器分别对外的服务端口为：（9100/20880）（9101/20881）（9102/20882）

- **实现**

**1、修改GoodsServiceImpl的findGoodsByOrderId方法**

![image-20201226112039952](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226112039952.png)

**2、部署3台goods-service集群节点**

修改application.yml配置文件如下：

![image-20201226112223289](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226112223289.png)



这时可以打开 `GoodsApplication` 启动第1台商品系统。

**配置其它两台商品系统启动项**：

![image-20201226112508971](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226112508971.png)

![image-20201226112603750](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226112603750.png)

![image-20201226112706216](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226112706216.png)



重复刚才的操作，并修改对应名称和启动参数。

![image-20201226112758378](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226112758378.png)

**3、测试**

a，正常启动（不要debug启动） `GoodsApplication`  `GoodsApplication20881` `GoodsApplication20882` 

b，修改 `OrderController` 结果缓存的设置去掉；

![image-20201226114207165](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226114207165.png)

c，启动 `OrderApplication`

![image-20201226114124980](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226114124980.png)

f，访问 http://localhost:9200/order/1；再查看后台中的3台商品系统控制台的输出情况。



>发现在调用商品系统服务的时候，如果失败的话；那么会尝试从3台服务中都去获取服务；直到获取到数据或者所有服务器都访问失败。这是DUBBO默认的集群容错策略：failover 适用于读取数据。
>
>如果对应的服务是写数据失败进行重试，会引发什么问题呢？
>
>

#### 3.3.2、案例2

> 访问订单系统（order-service） http://localhost:9200/order/add 生成订单；同时对商品系统（goods-service）中商品库存进行扣减；其中，在商品系统中扣减库存商品需要耗时2000毫秒。

**分析**：因为DUBBO远程服务调用的超时时间是1000毫秒，重试2次；像上述的需求，如果按照默认方式执行因为扣减库存时间超时，那么会进行重试调用其它服务，这样在3台商品服务系统中都尝试扣减库存，引起库存扣减异常。

**解决方法**：

方式一、设置DUBBO引用服务的超时时间；如：

![image-20201226115728281](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226115728281.png)



方式二、设置DUBBO的集群容错策略为：`failfast` 一旦失败则不再重试

![image-20201226115833609](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226115833609.png)

修改容错方式之后；重启 `OrderApplication` 访问 http://localhost:9200/order/add 查看可以发现，一旦商品服务调用失败，即使有其它的同类商品服务也不会进行重试。



#### 3.3.3、容错说明

**在集群调用失败时，Dubbo默认提供了6种集群容错模式，缺省为 failover 重试。**

 + Failover Cluster(默认): 失败自动切换，当出现失败，重试其它服务器。通常用于读操作，但重试会带来更长延迟。可通过`retries="2"` 来设置重试次数(不包含第一次)。
  + Failfast Cluster: 快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。
  + Failsafe Cluster: 失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。
  + Failback Cluster: 失败自动恢复，后台记录失败请求，**定时重发**。通常用于消息通知操作。
  + Forking Cluster: 并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 `forks="2"` 来设置最大并行数。
  + Broadcast Cluster: 广播调用所有提供者，逐个调用，任意一台报错则报错。通常用于通知所有提供者更新缓存或日志等本地资源信息。(暂不支持)



![image-20201225231520135](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201225231520135.png)

各节点关系：

- 这里的 `Invoker` 是 `Provider` 的一个可调用 `Service` 的抽象，`Invoker` 封装了 `Provider` 地址及 `Service` 接口信息
- `Directory` 代表多个 `Invoker`，可以把它看成 `List` ，但与 `List` 不同的是，它的值可能是动态变化的，比如注册中心推送变更
- `Cluster` 将 `Directory` 中的多个 `Invoker` 伪装成一个 `Invoker`，对上层透明，伪装过程包含了容错逻辑，调用失败后，重试另一个
- `Router` 负责从多个 `Invoker` 中按路由规则选出子集，比如读写分离，应用隔离等
- `LoadBalance` 负责从多个 `Invoker` 中选出具体的一个用于本次调用，选的过程包含了负载均衡算法，调用失败后，需要重选



>如何选择容错模式？
>
>failover —— 读操作；可以重试
>failfast —— 写操作；只发起一次调用，失败立即报错

### 3.4、服务降级

#### 3.4.1、Mock本地伪装

本地伪装 [1](http://dubbo.apache.org/zh/docs/v2.7/user/examples/local-mock/#fn:1) 通常用于服务降级，比如订单下单的时候，当商品服务提供方全部挂掉后，客户端不抛出异常，而是通过 Mock 数据返回失败。

**1）代码配置**

假设在已知 商品服务 未开发好的情况下，订单需要开发测试；那么可以在代码上配置如下：

- mock=force:return null（**屏蔽,熔断**）表示**消费方**对该服务的方法调用都直接返回 null 值，**不发起远程调用**。用来屏蔽不重要服务不可用时对调用方的影响。

![image-20201226122210843](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226122210843.png)

- mock=fail:return null（**容错,降级**）表示**消费方**对该服务的方法调用在失败后，再返回 null 值，不抛异常。用来容忍不重要服务不稳定时对调用方的影响。

![image-20201226122251180](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226122251180.png)



**2）控制台设置**

访问： http://localhost:7001/governance/consumers

![image-20201226173332734](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226173332734.png)



屏蔽效果（熔断）：

![image-20201226173541048](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226173541048.png)



容错效果（降级）：

![image-20201226173624847](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226173624847.png)

>屏蔽: 屏蔽后，将不发起远程调用，直接在客户端返回空对象
>容错: 容错后，当远程调用失败时，返回空对象。



#### 3.4.2、整合Hystrix

DUBBO中也可以使用 `Hystrix` 进行熔断；具体配置步骤如下：

1）添加依赖

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>2.2.6.RELEASE</version>
        </dependency>
```

2）配置

a、在 `OrderApplication` 启动引导类中添加 `@EnableCircuitBreaker` 注解

![image-20201226173953338](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226173953338.png)

b、在 `OrderController` 中配置如下：

![image-20201226174212538](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226174212538.png)



3）测试

重新启动 `OrderApplication` 访问：http://localhost:9200/order/1

![image-20201226174423596](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226174423596.png)

### 3.5、负载均衡策略

在集群负载均衡时，Dubbo 提供了多种均衡策略，缺省为 `random` 随机调用。

#### 3.5.1、策略说明

**Random LoadBalance**

- **随机**，按权重设置随机概率。
- 在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。

![image-20201226182346717](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226182346717.png)

**RoundRobin LoadBalance**

- **轮询**，按公约后的权重设置轮询比率。
- 存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。

![image-20201226182453983](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226182453983.png)

**LeastActive LoadBalance**

- **最少活跃调用数**，相同活跃数的随机，活跃数指调用前后计数差。
- 使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。

![image-20201226182525381](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226182525381.png)

**ConsistentHash LoadBalance**

- **一致性 Hash**，相同参数的请求总是发到同一提供者。
- 当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。
- 算法参见：http://en.wikipedia.org/wiki/Consistent_hashing
- 缺省只对第一个参数 Hash
- 缺省用 160 份虚拟节点

![image-20201226182554085](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226182554085.png)

#### 3.5.2、案例应用

**需求**：商品服务系统goods-service中有3台服务提供者，这3台服务机器配置性能都相当。在订单服务系统order-service调用的时候希望可以轮流调用 `findGoodsByOrderId`

**实现方式一**：代码中设置负载均衡策略为：轮询

![image-20201226185110709](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226185110709.png)

测试时候分别访问：http://localhost:9200/order/1 请求地址3次；然后到商品服务的服务器控制台看输出即可。



**实现方式二**：在监控中心中设置商品服务的对应方法的负载均衡策略为：轮询

![image-20201226184449059](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226184449059.png)

![image-20201226184726027](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226184726027.png)

![image-20201226184740153](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226184740153.png)

### 3.6、灰度发布

**灰度发布**：是指在黑与白之间，能够平滑过渡的一种发布方式。在其上可以进行A/B testing，即让一部分用户继续用产品特性A，一部分用户开始用产品特性B，如果用户对B没有什么反对意见，那么逐步扩大范围，把所有用户都迁移到B上面来。

在 Dubbo 中为同一个服务配置多个版本当一个接口实现，出现不兼容升级时，可以用版本号过渡，版本号不同的服务相互间不引用。

可以按照以下的步骤进行版本迁移：

1. 在低压力时间段，先升级一半提供者为新版本
2. 再将所有消费者升级为新版本
3. 然后将剩下的一半提供者升级为新版本

**案例需求**：

> 将商品服务goods-service发布了新版的GoodsService，但是订单系统order-service在使用中不能停止等待商品服务的更新。所以需要保留一部分旧版的商品服务，给订单系统调用使用。同时也需要在订单系统可以使用到最新的商品服务。

**分析**：

![image-20201226191244872](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226191244872.png)



**实现**：

1）修改 `商品服务goods-service` 的配置文件添加版本配置信息`1.0`如下：

![image-20201226192950328](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226192950328.png)

2）非debug 启动 `GoodsAppliaction` 

![image-20201226193037516](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226193037516.png)

3）修改 `商品服务goods-service` 的配置文件添加版本配置信息如`2.0`下：

![image-20201226193110872](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226193110872.png)

4）非debug 启动 `GoodsAppliaction20881` 

![image-20201226193201098](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226193201098.png)

5）修改 `订单服务order-service`的配置文件添加版本配置信息如下：

![image-20201226193251540](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226193251540.png)

6） 启动 `OrderAppliaction` 

![image-20201226193351856](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226193351856.png)

可以从访问中看出；订单服务只会调用版本号对应的服务，同样的服务可通过不同的版本号区分开来；从而实现灰度发布。

## 4、Zookeeper集群

### 4.1、搭建

Zookeeper单机版的安装搭建可以参考《Linux安装zookeeper(单机版).md》

Zookeeper集群版的安装搭建可以参考《Linux安装zookeeper(集群版).md》

### 4.2、应用

#### 1）配置DUBBO监控中心

打开 `dubbo-admin-master\dubbo-admin\src\main\resources\application.properties` 文件；修改

`dubbo.registry.address`的配置项内容为如下：

```properties
dubbo.registry.address=zookeeper://192.168.12.135:3181?backup=192.168.12.135:3182,192.168.12.135:3183
```

![image-20201226194240161](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226194240161.png)

#### 2）配置商品服务系统

打开 `dubbo-springboot\goods\goods-service\src\main\resources\application.yml` 文件；修改

`dubbo.registry.address`的配置项内容为如下：

```yml
server:
  port: ${port:9100}

spring:
  application:
    name: goods-service
dubbo:
  registry:
    address: zookeeper://192.168.12.135:3181?backup=192.168.12.135:3182,192.168.12.135:3183
    timeout: 10000
  scan:
    base-packages: com.itheima.goods.service.impl
  protocol:
    name: dubbo
    port: ${dport:20880}
  provider:
    version: 2.0
```

![image-20201226194527791](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226194527791.png)

#### 3）配置订单服务系统

打开 `dubbo-springboot\order\order-service\src\main\resources\application.yml` 文件；修改

`dubbo.registry.address`的配置项内容为如下：

```yml
server:
  port: 9200

spring:
  application:
    name: order-service
dubbo:
  registry:
    address: zookeeper://192.168.12.135:3181?backup=192.168.12.135:3182,192.168.12.135:3183
    timeout: 10000
  scan:
    base-packages: com.itheima.order.service.impl
  protocol:
    name: dubbo
    port: 30880
  consumer:
    check: false
    version: 2.0
```

![image-20201226194643398](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226194643398.png)



可以到监控中心中查看注册中心的状态；比如注册中心的集群节点3台中，如果其中1台宕机了；也不会影响使用。

![image-20201226194812251](就业技术加强-05-DUBBO-Zookeeper-2.assets/image-20201226194812251.png)

## 5、常见面试题

- **问题1**：Dubbo与SpringCloud区别 

```
两个没什么关联，如果硬要说区别，有以下几点:

通信方式不同: 
Dubbo使用的是RPC通信，而Spring Cloud使用的是 HTTP RESTful方式通信。
a）Dubbo底层使用Netty这样的NIO框架，基于TCP协议传输，配合Hession序列化完成RPC通信。
b）SpringCloud基于Http协议+Rest接口远程调用的通信，相对来说，Http请求会有更大的报文，占的带宽也会更多。但是REST相比RPC更为灵活，服务提供方和调用方的依赖只依靠一纸契约，不存在代码级别的强依赖，这在强调快速演化的微服务环境下，显得更为合适。

组成部分不同:
Dubbo是SOA时代的产物，它的关注点主要在于服务的调用，流量分发、流量监控和熔断。
Spring Cloud诞生于微服务架构时代，考虑的是微服务治理的方方面面，另外由于依托了Spirng、Spirng Boot的优势之上，两个框架在开始目标就不一致，Dubbo定位服务治理、Spirng Cloud是一个生态(更完善)。
```



+ **问题2**： Dubbo只能从注册中心获取服务吗?

  ```txt
  不是，也可以使用直连，直连方式是不需要从注册中心获取服务。
  在开发及测试环境下，经常需要绕过注册中心，只测试指定服务提供者，这时可能需要点对点直连方式。
  
  @Reference(url = "dubbo://localhost:24567")
  private IUserservice userservice;
  
  直连方式会失去负载均衡的能力,所以不适合生产环境。
  ```

+ **问题3**： Dubbo服务提供者一定要需要开启，消费者才可以开启吗?

  ```txt
  服务提供者不一定需要开启，因为dubbo有启动时检查,如果check=false,服务提供者就不需要开启。
  ```

+ **问题4**： Dubbo默认调用服务超时时长是多少?

  ```txt
  服务超时时长是: 1000毫秒；重试两次
  ```

+ **问题5**： Dubbo集群容错模式有哪些? 默认容错模式是什么?【重点】

  ```txt
  Dubbo集群容错模式有6种: Failover、Failfast、Failsafe、Faillback、Forking、Broadcast
  默认容错模式是: Failover(失败自动切换)
  ```

+ **问题6**： Dubbo注册中心有哪些?【重点】

  ```txt
  Dubbo默认支持五种注册中心:
  1. Multicast
  2. Zookeeper(推荐)
  3. Nacos
  4. Redis
  5. Simple
  ```

+ **问题7**： Dubbo注册中心zookeeper挂掉，消费者还可以正常调用服务者吗?

  ```txt
  可以正常调用，因为消费者从注册中心，获取服务地址后会在本地缓存。
  ```

+ **问题8**： Dubbo可以对调用结果进行缓存吗?

  ```txt
  可以对调用结果进行缓存(默认是没有的),可以通过cache属性配置结果缓存(lru, threadlocal, jcache)
  ```

+ **问题9**： Dubbo屏蔽与熔错是什么?

  ```txt
  屏蔽: 相当于熔断, 不发起远程调用。
       mock=force:return null
  
  熔错: 相当于降级, 在远程调用失败时，返回null值，不抛出异常，对消费者不产生调用影响。
       mock=fail:return null
  ```

+ **问题10**： Dubbo可以整合Hystrix吗?

  ```txt
  可以,用Hystrix框架来提供服务降级方法。
  ```

+ **问题11**： Dubbo负载均衡策略有哪些，默认负载均衡策略是什么?【重点】

  ```txt
  负载均衡策略有4种:
  1. random(随机)
  2. roundrobin(轮询)
  3. leastactive(最少活跃数)
  4. consistanthash(一致性hash)
  
  默认负载均衡策略是: random (随机)
  ```

+ **问题12**： Dubbo注册中心zookeeper挂掉，负载均衡还有效吗?

  ```txt
  有效。因为消费者从zookeeper注册中心获取服务地址后会在本地缓存。
  ```

+ **问题13**： Dubbo项目如何对服务升级?

```
使用服务提供者的版本号隔离，对服务进行灰度发布。
```

