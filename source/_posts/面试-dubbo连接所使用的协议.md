---
title: 面试-dubbo连接所使用的协议
tags:
  - 笔记
  - 面试
  - dubbo
categories:
  - 面试
abbrlink: 27160
date: 2021-01-15 16:12:41
---

# dubbo连接所使用的协议

## dubbo://协议

### 使用场景:

​	Dubbo 缺省协议采用单一长连接和 NIO 异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况。

反之，Dubbo 缺省协议不适合传送大数据量的服务，比如传文件，传视频等，除非请求量很低。

### 特性:

​	缺省协议，使用基于 minai a `1.1.7` 和 hessian `3.2.1` 的 tbremoting 交互。

- 连接个数：单连接

- 连接方式：长连接

- 传输协议：TCP

- 传输方式：NIO 异步传输

- 序列化：Hessian 二进制序列化

- 适用范围：传入传出参数数据包较小（建议小于100K），消费者比提供者个数多，单一消费者无法压满提供者，尽量不要用 dubbo 协议传输大文件或超大字符串。

- 适用场景：常规远程服务方法调用

  

## rmi://协议

### 使用场景:

​	RMI 协议采用 JDK 标准的 `java.rmi.*` 实现，采用阻塞式短连接和 JDK 标准序列化方式。

注意：如果正在使用 RMI 提供服务给外部访问 ，同时应用里依赖了老的 common-collections 包的情况下，存在反序列化安全风险 。

### 特性

- 连接个数：多连接

- 连接方式：短连接

- 传输协议：TCP

- 传输方式：同步传输

- 序列化：Java 标准二进制序列化

- 适用范围：传入传出参数数据包大小混合，消费者与提供者个数差不多，可传文件。

- 适用场景：常规远程服务方法调用，与原生RMI服务互操作

  

## hessian://协议

### 使用场景:

​	Hessian协议用于集成 Hessian 的服务，Hessian 底层采用 Http 通讯，采用 Servlet 暴露服务，Dubbo 缺省内嵌 Jetty 作为服务器实现。

Dubbo 的 Hessian 协议可以和原生 Hessian 服务互操作，即：

- 提供者用 Dubbo 的 Hessian 协议暴露服务，消费者直接用标准 Hessian 接口调用
- 或者提供方用标准 Hessian 暴露服务，消费方用 Dubbo 的 Hessian 协议调用。

### 特性

- 连接个数：多连接

- 连接方式：短连接

- 传输协议：HTTP

- 传输方式：同步传输

- 序列化：Hessian二进制序列化

- 适用范围：传入传出参数数据包较大，提供者比消费者个数多，提供者压力较大，可传文件。

- 适用场景：页面传输，文件传输，或与原生hessian服务互操作

  

## http://协议

### 使用场景:

​	基于 HTTP 表单的远程调用协议，采用 Spring 的 HttpInvoker 实现

### 特性

- 连接个数：多连接
- 连接方式：短连接
- 传输协议：HTTP
- 传输方式：同步传输
- 序列化：表单序列化
- 适用范围：传入传出参数数据包大小混合，提供者比消费者个数多，可用浏览器查看，可用表单或URL传入参数，暂不支持传文件。
- 适用场景：需同时给应用程序和浏览器 JS 使用的服务。



## webservice://协议

### 使用场景:

​	基于 WebService 的远程调用协议，基于 Apache CXF 的 `frontend-simple` 和 `transports-http` 实现 。可以和原生 WebService 服务互操作，即：

- 提供者用 Dubbo 的 WebService 协议暴露服务，消费者直接用标准 WebService 接口调用，
- 或者提供方用标准 WebService 暴露服务，消费方用 Dubbo 的 WebService 协议调用。  

### 特性

- 连接个数：多连接

- 连接方式：短连接

- 传输协议：HTTP

- 传输方式：同步传输

- 序列化：SOAP 文本序列化

- 适用场景：系统集成，跨语言调用

  

## thrift://协议

### 使用场景:

​	当前 dubbo 支持的 thrift 协议是对 thrift 原生协议的扩展，在原生协议的基础上添加了一些额外的头信息，比如 service name，magic number 等。

使用 dubbo thrift 协议同样需要使用 thrift 的 idl compiler 编译生成相应的 java 代码，后续版本中会在这方面做一些增强。



## memcached://协议

​	基于 memcached实现的 RPC 协议。

### 注册 memcached 服务的地址

```java
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("memcached://10.20.153.11/com.foo.BarService?category=providers&dynamic=false&application=foo&group=member&loadbalance=consistenthash"));

```

### 在客户端引用

在客户端使用：

```xml
<dubbo:reference id="cache" interface="java.util.Map" group="member" />
```

或者，点对点直连：

```xml
<dubbo:reference id="cache" interface="java.util.Map" url="memcached://10.20.153.10:11211" />
```

也可以使用自定义接口：

```xml
<dubbo:reference id="cache" interface="com.foo.CacheService" url="memcached://10.20.153.10:11211" />

```

方法名建议和 memcached 的标准方法名相同，即：get(key), set(key, value), delete(key)。

如果方法名和 memcached 的标准方法名不相同，则需要配置映射关系：

```xml
<dubbo:reference id="cache" interface="com.foo.CacheService" url="memcached://10.20.153.10:11211" p:set="putFoo" p:get="getFoo" p:delete="removeFoo" />

```



## redis://协议

​	基于 Redis实现的 RPC 协议。

### 注册 redis 服务的地址

```java
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("redis://10.20.153.11/com.foo.BarService?category=providers&dynamic=false&application=foo&group=member&loadbalance=consistenthash"));

```

### 在客户端引用

在客户端使用：

```xml
<dubbo:reference id="store" interface="java.util.Map" group="member" />
```

或者，点对点直连：

```xml
<dubbo:reference id="store" interface="java.util.Map" url="redis://10.20.153.10:6379" />
```

也可以使用自定义接口：

```xml
<dubbo:reference id="store" interface="com.foo.StoreService" url="redis://10.20.153.10:6379" />

```

方法名建议和 redis 的标准方法名相同，即：get(key), set(key, value), delet(key)。

如果方法名和 redis 的标准方法名不相同，则需要配置映射关系：

```xml
<dubbo:reference id="cache" interface="com.foo.CacheService" url="memcached://10.20.153.10:11211" p:set="putFoo" p:get="getFoo" p:delete="removeFoo" />

```

