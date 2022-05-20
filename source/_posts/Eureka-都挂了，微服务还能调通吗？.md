---
title: Eureka 都挂了，微服务还能调通吗？
tags:
  - Spring Cloud
  - 面试
categories:
  - 面试
abbrlink: 13921
date: 2021-01-14 18:55:48
---

# Eureka 都挂了，微服务还能调通吗？                    

这是一位公众号读者遇到的面试题。

老实说，这个问题并不难。

如果你做过微服务开发，这个面试题应该能够立马答出来，如果你没做过微服务开发，但是学过一些 Spring Cloud 组件的用法，这个问题可能要稍微想一下，但是也应该能够答出来。





## 1.原因分析

为什么 Eureka 关闭后服务还能调用呢？我们先来看一张简单的服务调用图：

![20200504160930](Eureka-都挂了，微服务还能调通吗？.assets/20200504160930.png)

我来说一下这个流程：

1. Eureka 作为一个服务注册中心启动。
2. Provider 和 Consumer 分别作为服务启动，并且注册到 Eureka 上面去，以 provider 为例，provider 注册时会告诉 eureka，我叫 provider，我的地址是 xx.xx.xx.xx，我的端口是 xx，我的 xx 是  xx，就是说，provider 会将自己的一些元数据信息告诉 eureka；同理，consumer 也是如此。
3. 接下来，consumer 要调用 provider 的接口，但是它不知道 provider 的地址是什么，他只知道要调用的服务叫  provider，于是 consumer 找到 eureka，从 eureka 上查询出来 provider  的具体地址和端口，这个具体的地址和端口，可能是一个，也可能是多个（集群化部署）。
4. consumer 获取到 provider 的地址和端口之后，接下来就直接去调用 provider 了。

从上面一个流程图中，大家可以看出来，一旦 consumer 获取到 provider 的具体地址，接下来的调用其实就没有 eureka 什么事了。

所以，我们说一旦 Eureka 挂了，微服务是可以调通的，**但是是有前提的**。

什么前提？就是 provider 的地址没变！如果 provider 换了一个 IP 地址或者端口，这个时候，consumer  就无法及时感知到这种变化，就会调不通。当 Eureka 没有挂掉的时候，provider 的 IP 变化这种事情，可以通过 Eureka 让  consumer 感知到，进而对调用地址作出调整，现在 Eureka 挂了，consumer 就无法感知了。



## 2.相关原理

Eureka 本身可以分为两大部分，Eureka Server 和 Eureka Client。

我们先来看 Eureka Server：

### 2.1 Eureka Server

Eureka Server 主要对外提供了三个功能：

1. 服务注册，所有的服务都注册到 Eureka Server 上面来，这是 Eureka 基本功能。
2. 提供注册表，注册表就是所有注册上来服务的一个列表，Eureka 内部通过一个二层缓存机制来维护这个注册表。Eureka Client 在调用服务时，需要获取这个注册表，一般来说，这个注册表会缓存下来，如果缓存失效，则直接获取最新的注册表。
3. 同步状态，Eureka Client 通过注册、心跳等机制，和 Eureka Server 同步当前客户端的状态，以便 Eureka Client 能够及时感知到变化。

### 2.2 Eureka Client

服务要注册到 Eureka 上面去，这种注册本身就是一个 HTTP 请求，但是自己手写注册过程的话太过于繁琐，Eureka Client 可以帮助我们简化注册过程。

一般来说，Eureka Client 有这样一些功能：

##### 服务注册

服务提供者将自己注册到服务注册中心（Eureka Server），需要注意，所谓的服务提供者，只是一个业务上的划分，本质上他就是一个 Eureka Client。当 Eureka Client 向 Eureka Server 注册时，他需要提供自身的一些元数据信息，例如 IP  地址、端口、名称、运行状态等等，将来服务消费者获取到的也是这些信息。

##### 获取注册信息

Eureka Client 从 Eureka Server 上获取服务的注册信息，**并将其缓存在本地**，这句是关键。

当 Eureka Client 在需要调用远程服务时，会从该信息中查找远程服务所对应的 IP 地址、端口等信息。Eureka Client 上缓存的服务注册信息会定期更新(30 秒)，如果 Eureka Server 返回的注册表信息与本地缓存的注册表信息不同的话，Eureka  Client 会自动处理。

这里，也涉及到两个属性：

1. 一个是是否允许获取注册表信息：`eureka.client.fetch-registry=true`。
2. 另一个是 Eureka Client 上缓存的服务注册信息，定期更新的时间间隔，默认 30 秒，可以通过如下属性自行修改：`eureka.client.registry-fetch-interval-seconds=30`。

##### 服务续约

Eureka Client 注册到 Eureka Server 上之后，默认情况下，Eureka CLient 每隔 30 秒就要向 Eureka Server 发送一条心跳消息，来告诉 Eureka Server 我还在运行。

如果 Eureka Server 连续 90 秒都有没有收到 Eureka Client 的续约消息（连续三次没发送），它会认为 Eureka Client 已经掉线了，会将掉线的 Eureka Client 从当前的服务注册列表中剔除。

这里有两个相关的属性（一般不建议修改）：

1. `eureka.instance.lease-renewal-interval-in-seconds` 表示服务的续约时间，默认是 30 秒。
2. `eureka.instance.lease-expiration-duration-in-seconds` 表示服务失效时间，默认是 90 秒。

##### 服务下线

服务下线当 Eureka Client 下线时，它会主动发送一条消息，告诉 Eureka Server ，我下线啦。

从上面的介绍可以看出，Eureka Client 会自动拉取、更新以及缓存 Eureka Server 中的信息，这样，即使 Eureka Server 所有节点都宕机，Eureka Client 依然能够获取到想要调用服务的地址（前提是服务地址没有发生变化）。

好了，本文就先说这么多，其实东西不难，感兴趣的小伙伴感觉去试试吧～