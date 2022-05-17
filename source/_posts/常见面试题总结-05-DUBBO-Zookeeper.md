---
title: 常见面试题总结(05)-DUBBO&Zookeeper
tags:
  - 笔记
  - 就业技术加强
  - DUBBO
  - Zookeeper
  - 集群
  - 优化
  - 常见面试题总结
categories:
  - 常见面试题总结
date: 2021-01-28 17:40:55
---

## 问题1：Dubbo与SpringCloud区别 

两个没什么关联，如果硬要说区别，有以下几点:

通信方式不同: 
Dubbo使用的是RPC通信，而Spring Cloud使用的是 HTTP RESTful方式通信。
a）Dubbo底层使用Netty这样的NIO框架，基于TCP协议传输，配合Hession序列化完成RPC通信。
b）SpringCloud基于Http协议+Rest接口远程调用的通信，相对来说，Http请求会有更大的报文，占的带宽也会更多。但是REST相比RPC更为灵活，服务提供方和调用方的依赖只依靠一纸契约，不存在代码级别的强依赖，这在强调快速演化的微服务环境下，显得更为合适。

组成部分不同:
Dubbo是SOA时代的产物，它的关注点主要在于服务的调用，流量分发、流量监控和熔断。
Spring Cloud诞生于微服务架构时代，考虑的是微服务治理的方方面面，另外由于依托了Spirng、Spirng Boot的优势之上，两个框架在开始目标就不一致，Dubbo定位服务治理、Spirng Cloud是一个生态(更完善)。



## 问题2： Dubbo只能从注册中心获取服务吗?

不是，也可以使用直连，直连方式是不需要从注册中心获取服务。
在开发及测试环境下，经常需要绕过注册中心，只测试指定服务提供者，这时可能需要点对点直连方式。

```java
@Reference(url = "dubbo://localhost:24567")
private IUserservice userservice;
```

直连方式会失去负载均衡的能力,所以不适合生产环境。

## 问题3： Dubbo服务提供者一定要需要开启，消费者才可以开启吗?

```txt
服务提供者不一定需要开启，因为dubbo有启动时检查,如果check=false,服务提供者就不需要开启。
```

## 问题4： Dubbo默认调用服务超时时长是多少?

服务超时时长是: 1000毫秒；重试两次

## 问题5： Dubbo集群容错模式有哪些? 默认容错模式是什么?【重点】

Dubbo集群容错模式有6种: Failover、Failfast、Failsafe、Faillback、Forking、Broadcast
默认容错模式是: Failover(失败自动切换)

## 问题6： Dubbo注册中心有哪些?【重点】

Dubbo默认支持五种注册中心:
1. Multicast
2. Zookeeper(推荐)
3. Nacos
4. Redis
5. Simple

## 问题7： Dubbo注册中心zookeeper挂掉，消费者还可以正常调用服务者吗?

可以正常调用，因为消费者从注册中心，获取服务地址后会在本地缓存。

## 问题8： Dubbo可以对调用结果进行缓存吗?

可以对调用结果进行缓存(默认是没有的),可以通过cache属性配置结果缓存(lru, threadlocal, jcache)

## 问题9： Dubbo屏蔽与熔错是什么?

屏蔽: 相当于熔断, 不发起远程调用。
     mock=force:return null

熔错: 相当于降级, 在远程调用失败时，返回null值，不抛出异常，对消费者不产生调用影响。
     mock=fail:return null

## 问题10： Dubbo可以整合Hystrix吗?

可以,用Hystrix框架来提供服务降级方法。

## 问题11： Dubbo负载均衡策略有哪些，默认负载均衡策略是什么?【重点】

```txt
负载均衡策略有4种:
1. random(随机)
2. roundrobin(轮询)
3. leastactive(最少活跃数)
4. consistanthash(一致性hash)

默认负载均衡策略是: random (随机)
```

## 问题12： Dubbo注册中心zookeeper挂掉，负载均衡还有效吗?

有效。因为消费者从zookeeper注册中心获取服务地址后会在本地缓存。

## 问题13： Dubbo项目如何对服务升级?

使用服务提供者的版本号隔离，对服务进行灰度发布。

