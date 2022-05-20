---
title: SpringCloud(2)
tags:
  - 笔记
  - 项目实战二前置课程
  - SpringCloud
categories:
  - 项目实战二前置课程
abbrlink: 3663
date: 2020-12-22 13:35:39
---

## 课程目标

目标1：能够使用Feign进行远程调用 

目标2：能够搭建Spring Cloud Gateway网关服务 

目标3：能够配置Spring Cloud Gateway路由过滤器 

目标4：能够编写Spring Cloud Gateway全局过滤器 

目标5：能够搭建Spring Cloud Config配置中心服务 

目标6：能够使用Spring Cloud Bus实时更新配置 



## 01、Feign：介绍与使用

在前面的学习中，使用了Ribbon的负载均衡功能，大大简化了远程调用时的代码： 

```java
// 定义服务实例访问URL
String url = "http://user-service/user/" + id;
return restTemplate.getForObject(url, String.class);
```

但是我们要思考一个问题? **微服务 与 微服务 之间调用问题**

+ 调用URL问题 (我们自己拼接，代码不够优雅)
+ 请求参数问题 (我们自己封装请求参数？如果请求参数多，封装起来就会变得特别麻烦，何况不同的请求封装请求参数格式还不一样)

这就是我们接下来要学习Feign的目的，它可以把这些问题全部伪装成一个Feign的http客户端接口，默认集成RestTemplate和Ribbon。**Feign存在的目的就是为了简化微服务之间的调用**。 

### 1.1 介绍

Feign也叫伪装： 

![1572166999223](SpringCloud-2.assets/001.png)&nbsp;

Feign可以把Rest的请求进行隐藏，**伪装SpringMVC的Controller一样，又类似于Mybatis的Mapper接口**。你不用再自己拼接url，拼接参数等等操作，一切的一切都交给Feign去做。

项目主页：https://github.com/OpenFeign/feign 

![1572166999223](SpringCloud-2.assets/1572166999223.png)

### 1.2 使用【掌握】

> 目标：使用Feign客户端调用微服务

**操作步骤**

+ 第一步：配置依赖(在user-consumer中添加如下依赖):

  ```xml
  <!-- 配置openfeign启动器 -->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
  </dependency>
  ```

+ 第二步：编写Feign的客户端(在user-consumer中编写Feign客户端接口类):

  ```java
  package cn.itcast.consumer.client;
  
  import cn.itcast.consumer.pojo.User;
  import org.springframework.cloud.openfeign.FeignClient;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.PathVariable;
  
  @FeignClient("user-service")//生成一个代理对象， 并生成bean
  public interface UserClient {
  
      @GetMapping("/user/{id}")
      User findOne(@PathVariable("id") Long id);
  }
  ```

  + Feign客户端必须是一个接口，Feign会通过JDK动态代理，创建代理对象，并生成一个Bean。

  + @FeignClient注解：声明一个Feign客户端，value或name属性指定服务id。

  + Feign客户端接口中的方法，完全采用SpringMVC的注解，Feign会根据SpringMVC的注解生成请求URL，封装请求参数，并实现http远程调用，得到调用结果。

+ 第三步：编写控制器（创建新的控制器类ConsumerFeignController，注入UserClient访问） 

  ```java
  package cn.itcast.consumer.controller;
  
  import cn.itcast.consumer.client.UserClient;
  import cn.itcast.consumer.pojo.User;
  import lombok.extern.slf4j.Slf4j;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.PathVariable;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RestController;
  
  @RestController
  @RequestMapping("/cf")
  public class ConsumerFeignController {
      @Autowired(required = false)
      private UserClient userClient;
      
      @GetMapping("/{id}")
      public User findOne(@PathVariable("id") Long id){
          return userClient.findOne(id);
      }
  }
  ```

+ 第四步：开启Feign的支持（在ConsumerApplication启动类上，添加@EnableFeignClients注解）

  ```java
  package cn.itcast.consumer;
  
  import org.springframework.boot.SpringApplication;
  import org.springframework.cloud.client.SpringCloudApplication;
  import org.springframework.cloud.client.loadbalancer.LoadBalanced;
  import org.springframework.cloud.openfeign.EnableFeignClients;
  import org.springframework.context.annotation.Bean;
  import org.springframework.web.client.RestTemplate;
  
  /** @SpringBootApplication
   @EnableDiscoveryClient
   @EnableCircuitBreaker */
  @SpringCloudApplication
  @EnableFeignClients // 开启Feign客户端
  public class ConsumerApplication {
      public static void main(String[] args){
          SpringApplication.run(ConsumerApplication.class, args);
      }
      @Bean
      @LoadBalanced
      public RestTemplate restTemplate(){
          return new RestTemplate();
      }
  }
  ```

  > 说明：Feign中已经自动集成了Ribbon负载均衡，因此不需要自己定义RestTemplate进行负载均衡的配置。 

+ 第五步：启动测试，访问地址 http://localhost:8080/cf/1

  ![1572170381541](SpringCloud-2.assets/1572170381541.png) 



## 02、Feign：Ribbon的支持

+ Feign中本身已经集成了Ribbon依赖和自动配置:

  ![1572191899479](SpringCloud-2.assets/1572191899479.png)

+ Fegin内置的Ribbon默认设置了请求超时时长，可以通过手动配置来修改这个超时时长，同时负载均衡策略在轮询的基础上增加了服务节点重试机制，一旦超时，会自动向下一个服务节点重新发起请求

  ```yaml
  ribbon:
    ConnectTimeout: 2000 # 建立连接的超时时长(默认,注意这是连接微服务超时时间,不能更改)
    ReadTimeout: 1000 # 读取响应数据超时时长(注意这是调用微服务获取响应数据超时时间，默认1000)
    MaxAutoRetries: 0 # 调用的当前节点的重试次数(默认)
    MaxAutoRetriesNextServer: 1 # 其他节点还需重试多少个服务节点(默认)
    OkToRetryOnAllOperations: false # true所有超时请求都重试(默认false，代表只重试Get请求)，如果不能保证被调用服务的幂等性，请设置为false。
  ```

  什么是幂等性？例如user-consumer调用user-service新增一个用户，但是因为网络波动，没有及时响应数据，这时如果重试，导致又插入一个同样的用户，这样岂不是数据重复？幂等性指的便是发送多次同样的请求，产生的结果是唯一的。

  说明：com.netflix.client.config.DefaultClientConfigImpl.java类中的默认配置

  ![1575388873283](SpringCloud-2.assets/1575388873283.png) 

  ![1575388991009](SpringCloud-2.assets/1575388991009.png) 

  ![1572193520578](SpringCloud-2.assets/1572193520578.png)


- 测试重试机制步骤：

  - 修改user-consumer配置：

    ```yaml
    ribbon:
      ConnectTimeout: 2000 # 建立连接的超时时长(默认,注意这是连接微服务超时时间,不能更改)
      ReadTimeout: 2300 # 读取响应数据超时时长(注意这是调用微服务获取响应数据超时时间)
      MaxAutoRetries: 0 # 当前服务器的重试次数(默认)
      MaxAutoRetriesNextServer: 1 # 重试多少个服务节点(默认)
      OkToRetryOnAllOperations: false # true所有超时请求都重试(默认false，代表只重试Get请求)，如果不能保证被调用服务的幂等性，请设置为false。
    ```

  - user-service添加睡眠时间2秒，并**只启动一个节点**：

    ```java
    @GetMapping("/{id}")
    public User findOne(@PathVariable("id") Long id)  {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return userService.findOne(id);
    }
    ```

  - 访问user-consumer的http://localhost:8080/cf/1

  - F12打开浏览器控制台，network中可以发现，偶尔访问时需要用时4s多，偶尔只需要2s多（user-service睡眠了2s），出现4s多是因为刚好第一次访问的是停止的节点，进行重试机制，去访问第二个节点。

    结果一：

    ![image-20200821143737354](SpringCloud-2.assets/image-20200821143737354.png)

    结果二：

    ![image-20200821143800228](SpringCloud-2.assets/image-20200821143800228.png)



## 03、Feign：Hystrix的支持

+ Feign默认也有对Hystrix做了集成（只不过，默认情况下是关闭的）：

  ![1572196163561](SpringCloud-2.assets/1572196163561.png) 

+ 需要通过下面的参数来开启:

  ```yaml
  feign:
    hystrix:
      enabled: true # 开启Feign的熔断功能(线程隔离与熔断) 
      
  # 线程隔离超时时间
  hystrix:
    command:
      default:
        execution:
          isolation:
            thread:
              timeoutInMilliseconds: 1000
  ```

  **注意**：若想要保证被调用服务只要**至少一个可用**就不超时，保证重试机制可用，Hystrix线程隔离的超时时间应该比ribbon重试的总时间要大，比如当前案例中，Hystrix线程隔离的超时时间应该大于等于 （ReadTimeout * 调用节点总次数（当前节点调用次数 + 重试节点调用次数）：2300 * 2 = 4600 (连接时间不计算在内)

+ 编写Feign的fallback服务降级逻辑

  + 定义UserClientFallback实现类，实现UserClient客户端接口，作为服务降级处理类，并生成Bean

    ```java
    package cn.itcast.consumer.client.fallback;
    
    import cn.itcast.consumer.client.UserClient;
    import cn.itcast.consumer.pojo.User;
    import org.springframework.stereotype.Component;
    
    @Component // 生成bean
    public class UserClientFallback implements UserClient {
        @Override
        public User findOne(Long id) {
            User user = new User();
            user.setId(1L);
            user.setName("用户异常");
            return user;
        }
    }
    ```

  + 在UserClient客户端接口中指定服务降级处理类

    ```java
    @FeignClient(value = "user-service", fallback = UserClientFallback.class)
    public interface UserClient {
        @GetMapping("/user/{id}")
        User findOne(@PathVariable("id") Long id);
    }
    ```

  + 测试(关闭user-service所有节点服务，然后在页面访问):

    ![1572229457585](SpringCloud-2.assets/1572229457585.png)&nbsp;



## 04、Feign：日志级别【了解】

前面讲过，通过 logging.level.xx=debug 来设置日志级别。然而这个对Fegin客户端而言不会产生效果。Fegin的日志由fegin.Logger.Level实例来输出。我们只需要创建一个fegin.Logger.Level日志级别的Bean即可。

+ 在user-consumer的配置文件中设置cn.itcast包下的日志级别都为debug

  ```properties
  logging:
    level:
      cn.itcast: debug
  ```

+ 在user-consumer启动类中，定义fegin.Logger.Level实例

  ```java
  @SpringCloudApplication
  @EnableFeignClients // 开启Feign客户端
  public class ConsumerApplication {
      
      public static void main(String[] args){
          // 运行spring应用
          SpringApplication.run(ConsumerApplication.class, args);
      }
      @Bean
      @LoadBalanced // 负载均衡注解
      public RestTemplate restTemplate(){
          return new RestTemplate();
      }
      // Feign日志级别
      @Bean
      public Logger.Level feignLoggerLevel(){
          return Logger.Level.FULL;
      }
  }
  ```

  这里指定的Level级别是FULL，feign.Logger.Level支持4种级别： 

  + NONE：不记录任何日志信息，这是默认值。 
  + BASIC：仅记录请求的方法，URL以及响应状态码和执行时间 
  + HEADERS：在BASIC的基础上，额外记录了请求和响应的头信息 
  + FULL：记录所有请求和响应的明细，包括请求头、请求体、响应头、响应体。 

+ 重启项目，Fegin客户端调用微服时，就可以输出访问的日志:

  ![1572231843263](SpringCloud-2.assets/1572231843263.png)



## 05、Gateway：网关介绍

官网学习地址：<https://spring.io/projects/spring-cloud-gateway#learn>

> Spring Cloud Gateway简介

+ Gateway基于Spring 5.0+、Spring Boot 2.0+、WebFlux、Netty等技术开发的网关服务。 
+ Gateway基于Filter链提供网关基本功能：断言、路由、过滤、限流等。
+ Gateway为微服务架构提供简单、有效、统一的API路由管理方式。 
+ Gateway是替代Netflix公司Zuul的一套解决方案。

Gateway组件的核心是一系列的过滤器，通过这些过滤器可以将客户端发送的请求转发（路由）到对应的微服务。 Spring Cloud Gateway是加在整个微服务最前面的防火墙和代理器，隐藏微服务节点IP与端口信息，从而达到保护微服务的目的。Spring Cloud Gateway本身也是一个微服务，且必须连接到Eureka服务注册中心，因为需要拉取服务列表，才能做转发请求。

> Gateway加入后的架构

![1572232735021](SpringCloud-2.assets/1572232735021.png)

说明：不管是来自于客户端（PC或移动端）的请求，还是服务内部调用。一切对服务的请求都可经过网关，然后再由网关来实现鉴权、动态路由等等操作。Gateway就是我们微服务调用的统一入口。 

**核心概念**：

+ 路由（route）：由一个ID、一个目标URI、一组断言工厂、一组过滤器组成。如果断言为真，该请求就会 路由到 目标URI。 
+ 断言（Predicate）：用断言工厂去匹配请求URL。如果能匹配，断言为真。

+ 过滤器（Filter）：用过滤器过滤请求，例如过滤请求URL、过滤请求参数、过滤请求头等。



## 06、Gateway：快速入门

### 6.1 创建模块

+ 填写基本信息

  ![1572233975329](SpringCloud-2.assets/1572233975329.png)

  ![1572234035717](SpringCloud-2.assets/1572234035717.png)

+ 添加依赖

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
           http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <parent>
          <artifactId>springcloud-demo</artifactId>
          <groupId>cn.itcast</groupId>
          <version>1.0-SNAPSHOT</version>
      </parent>
      <modelVersion>4.0.0</modelVersion>
      <artifactId>gateway-server</artifactId>
  
      <dependencies>
          <!-- 配置eureka客户端启动器 -->
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
          </dependency>
          <!-- 配置gateway启动器(基于netty运行，所在不需要tomcat启动器) -->
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-gateway</artifactId>
          </dependency>
      </dependencies>
  </project>
  ```

### 6.2 编写启动类

在gateway-server中创建cn.itcast.GatewayApplication启动类

```java
package cn.itcast;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class GatewayApplication {
    public static void main(String[] args){
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

### 6.3 编写配置

在gateway-server中创建application.yml文件，内容如下：

```properties
server:
  port: 10010
spring:
  application:
    name: api-gateway

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka,http://localhost:8762/eureka
```

**注意：必须设置拉取服务为true(默认也是为true)，因为需要转发请求到具体服务，或者使用默认值**



### 6.4 编写路由规则

+ 启动三个Spring Boot应用:

  ![1572235671256](SpringCloud-2.assets/1572235671256.png) 

+ 需要用网关来代理user-service服务，先看一下控制面板中的服务状态:

  ![1572235875152](SpringCloud-2.assets/1572235875152.png)

+ 修改gateway-server的application.yml文件:

  ```properties
  server:
    port: 10010
  spring:
    application:
      name: api-gateway
    cloud:
      gateway:
        routes:
          # 路由id, 路由信息的唯一标识, 可以随意写
          - id: user-service-route
            # 路由的目标服务地址
            uri: http://127.0.0.1:9001
            # 断言，Path: 匹配路由映射路径
            predicates:
              - Path=/**
  
  eureka:
    client:
      service-url:
        defaultZone: http://localhost:8761/eureka,http://localhost:8762/eureka
  ```

  + 将符合 Path 规则的一切请求，都代理或路由到 uri 参数指定的地址 
  + 本例中，我们将路径中包含有 /** 开头的请求，代理到http://127.0.0.1:9001 

### 6.5 启动测试

访问的路径中需要加上配置规则的映射路径，我们访问：http://localhost:10010/user/1

![1572237032054](SpringCloud-2.assets/1572237032054.png)



## 07、Gateway：面向服务路由

在刚才的路由规则中，把路径对应的服务地址写死了！如果同一服务有多个实例的话，这样做显然不合理。 应该根据服务的id，去Eureka注册中心查找服务对应的所有实例列表，然后进行动态路由！ 

+ 修改映射配置，通过服务名称获取

  因为已经配置了Eureka客户端，可以从Eureka获取服务的地址信息。修改application.yml文件： 

  ```properties
  server:
    port: 10010
  spring:
    application:
      name: api-gateway
    cloud:
      gateway:
        routes:
          # 路由id,可以随意写
          - id: user-service-route
            # 代理的服务地址；lb表示负载均衡(从eureka中根据服务id获取服务实例)
            uri: lb://user-service
            # 断言，Path: 配置路由映射路径
            predicates:
              - Path=/**
  
  eureka:
    client:
      service-url:
        defaultZone: http://localhost:8761/eureka,http://localhost:8762/eureka
  ```

  > 路由配置中uri所用的协议为lb时（以uri: lb://user-service为例），gateway将使用 LoadBalancerClient把user-service通过eureka解析为实际的主机和端口，并进行ribbon负载均衡。

  ![1572257331337](SpringCloud-2.assets/1572257331337.png)

+ 启动测试

  再次启动，这次gateway进行代理时，会利用Ribbon进行负载均衡访问：

  ![1572259349840](SpringCloud-2.assets/1572259349840.png)

  > 说明: spring-cloud-gateway网关服务，默认就已经集成了Ribbon负载均衡(轮询算法) 




## 08、Gateway：路由前缀

### 8.1 添加前缀

在gateway中可以通过配置网关过滤器PrefixPath，实现映射路径中的地址添加前缀，修改application.yml文件：

```yaml
server:
  port: 10010
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        # 路由id,可以随意写
        - id: user-service-route
          # 代理的服务地址；lb表示负载均衡(从eureka中根据服务id获取服务实例)
          uri: lb://user-service
          # 断言，配置路由映射路径
          predicates:
            - Path=/**
          filters:
            # 添加前缀: 指定路由路径需要添加的前缀
            - PrefixPath=/user

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka,http://localhost:8762/eureka
```

通过 **PrefixPath=/xxx** 来指定了路由要添加的前缀。

+ PrefixPath=/user，转换效果： http://localhost:10010/1 => http://localhost:9001/user/1 

+ PrefixPath=/user/abc，转换效果： http://localhost:10010/1 => http://localhost:9001/user/abc/1 以此类推。

  ![1572260447715](SpringCloud-2.assets/1572260447715.png)

### 8.2 去除前缀

在gateway中可以通过配置网关过滤器StripPrefix，实现映射路径中的地址去除前缀，修改application.yml文件：

```properties
server:
  port: 10010
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        # 路由id,可以随意写
        - id: user-service-route
          # 代理的服务地址；lb表示负载均衡(从eureka中根据服务id获取服务实例)
          uri: lb://user-service
          # 断言，Path: 配置路由映射路径
          predicates:
            - Path=/**
          filters:
            # 去除前缀: 1去除一个前缀，2去除两个前缀，以此类推
            - StripPrefix=1

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka,http://localhost:8762/eureka
```

通过 StripPrefix=1 来指定了路由要去掉的前缀个数。如：路径 /api/user/1 将会被代理到 /user/1。 

+ StripPrefix=1，转换效果： http://localhost:10010/api/user/1 => http://localhost:9001/user/1
+ StripPrefix=2，转换效果：http://localhost:10010/api/user/1 => http://localhost:9001/1

 以此类推。

 ![1572261006650](SpringCloud-2.assets/1572261006650.png)

> 作用：通过路由前缀可区分微服务，且可以对用户隐藏服务真实url



## 09、Gateway：过滤器介绍

> 过滤器简介

Gateway作为网关的其中一个重要功能，就是实现请求的鉴权。而这个动作往往是通过网关提供的过滤器来实现的。前一章中的路由前缀 章节中的功能也是使用过滤器实现的。

+ Gateway自带过滤器有30多个，常见自带过滤器有：

  | 过滤器名称           | 说明                         |
  | :------------------- | :--------------------------- |
  | AddRequestHeader     | 对匹配上的请求添加Header     |
  | AddRequestParameters | 对匹配上的请求添加参数       |
  | AddResponseHeader    | 对从网关返回的响应添加Header |
  | StripPrefix          | 对匹配上的请求路径去除前缀   |

  ![1572262078572](SpringCloud-2.assets/1572262078572.png)

  详细的说明在: <a href='https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.0.RC1/reference/html/#gatewayfilter-factories'>官网链接</a>

+ 过滤器类型:

  + 全局过滤器：自定义全局过滤器，需实现 GlobalFilter、Ordered两个接口。不需要在配置文件中配置，作用在所有的路由上。
  + 局部过滤器：通过spring.cloud.routes.filters 配置在具体路由下，只作用在当前路由上，自带的30几个过滤器都可以配置。

> 过滤器执行生命周期

Spring Cloud Gateway 的 Filter 的生命周期有两个(前置过滤与后置过滤)：“pre” 和 “post”。“pre”和 “post” 分别会在请求执行前调用 或 请求执行后调用。

![1572265550032](SpringCloud-2.assets/1572265550032.png)&nbsp;

过滤器使用场景(例举):

+ 请求鉴权: 一般在请求执行前，如果发现没有访问权限，直接就拦截返回空。 
+ 异常处理: 一般在请求执行后，记录异常并返回。 
+ 服务调用时长统计: 在请求执行前记录时间，在请求执行后计算该服务的调用时间。 



## 10、Gateway：配置默认过滤器

我们可以将Spring Cloud Gateway自带的过滤器配置成默认过滤器: 不是针对一个路由；而是对全部路由有效。(相当于全局过滤器)

```properties
server:
  port: 10010
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      # 默认过滤器，对全部路由有效
      default-filters:
        # 添加响应头过滤器，添中一个响应头为name，值为admin
        - AddResponseHeader=name,admin
      routes:
        # 路由id,可以随意写
        - id: user-service-route
          # 代理的服务地址；lb表示负载均衡(从eureka中根据服务id获取服务实例)
          uri: lb://user-service
          # 断言，Path: 配置路由映射路径
          predicates:
            - Path=/api/user/**
          filters:
            # 去除前缀: 1去除一个前缀，2去除两个前缀，以此类推
            - StripPrefix=1

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka,http://localhost:8762/eureka
```

运行测试:

![1572265183926](SpringCloud-2.assets/1572265183926.png)

## 11、Gateway：自定义全局过滤器

> 需求：模拟一个登录的校验。基本逻辑：如果请求中有token参数，则认为请求有效，放行。 

+ 在gateway-server模块中编写全局过滤器: MyGlobalFilter

  ```java
  package cn.itcast.filter;
  
  import org.apache.commons.lang.StringUtils;
  import org.springframework.cloud.gateway.filter.GatewayFilterChain;
  import org.springframework.cloud.gateway.filter.GlobalFilter;
  import org.springframework.core.Ordered;
  import org.springframework.http.HttpStatus;
  import org.springframework.stereotype.Component;
  import org.springframework.web.server.ServerWebExchange;
  import reactor.core.publisher.Mono;
  
  /** 自定义全局过滤器 */
  @Component
  public class MyGlobalFilter implements GlobalFilter, Ordered {
      @Override
      public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
          System.out.println("==全局过滤器MyGlobalFilter==");
          String token = exchange.getRequest().getQueryParams().getFirst("token");
          if (StringUtils.isBlank(token)){
              // 设置响应状态码: 401 未授权
              exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
              // 返回响应完成
              return exchange.getResponse().setComplete();
          }
          // 继续执行其他过滤器
          return chain.filter(exchange);
      }
      @Override
      public int getOrder() {
          // 值越小越先执行
          return 1;
      }
  }
  ```

+ 测试访问

  + 访问 http://localhost:10010/api/user/1，页面提示401

  + 访问 http://localhost:10010/api/user/1?token=abc

    ![1572337878029](SpringCloud-2.assets/1572337878029.png) 



## 12、Gateway：集成Hystrix

> 目标: spring-cloud-gateway集成Hystrix实现线程隔离。

- 第一步：在gateway-server中，添加hystrix启动器

  ```xml
  <!-- 配置hystrix启动器 -->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
  </dependency>
  ```

- 第二步：配置线程隔离时间

  ```properties
  # 线程隔离
  hystrix:
    command:
      default:
        execution:
          isolation:
            thread:
              timeoutInMilliseconds: 1000
  ```

- 第三步：在默认过滤器中配置Hystrix过滤器(对全部路由有效)

  ```properties
  spring:
    cloud:
      gateway:
        # 配置默认过滤器(对全部路由有效)
        default-filters:
        # 添加响应头，响应头的名称为name 值为admin
        - AddResponseHeader=name,admin
        - name: Hystrix  # 配置Hystrix过滤器
          args:          # 配置两个参数
            name: fallbackcmd
            fallbackUri: forward:/fallback
  ```

- 第四步：创建FallbackController.java控制器，提供服务降级方法

  ```java
  package cn.itcast.controller;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.RestController;
  
  @RestController
  public class FallbackController {
      @GetMapping("/fallback")
      public String fallback(){
          return "您好，服务器正忙，请稍候再试。。。";
      }
  }
  ```

- 第五步：测试

  ![1575308843439](SpringCloud-2.assets/1575308843439.png) 



## 13、Gateway：高可用

+ 启动多个Gateway服务，自动注册到Eureka，就可以形成集群。

  + Gateway被内部微服务访问，自动负载均衡(Ribbon)。 

  + Gateway被外部访问，如PC端、移动端等。它们无法通过Ribbon进行负载均衡，那么该怎么办？

    此时，可以使用其它的服务网关，来对Gateway做负载均衡。比如:【Nginx、Apache、F5】

   

+ Gateway与Feign的区别

  + Gateway 作为整个应用的入口，接收所有的请求，如PC、移动端等，并且将不同的请求路由至不同的微服务，大部分情况下用作权限鉴定、流量控制。
  + Feign 主要用于微服务与微服务之间的调用。 



## 14、Config：配置中心介绍

在分布式系统中，由于微服务数量特别多，配置文件分散在不同的微服务中，不方便管理。为了更方便管理配置文件，就需要统一管理的配置中心。Spring Cloud 提供了Spring Cloud Config，它支持配置文件统一管理，可以把全部微服务的配置文件放在Git远程仓库（GitHub、gitee码云）。

使用Spring Cloud Config配置中心后的架构如下图：

![1572343652379](SpringCloud-2.assets/1572343652379.png)

官网学习文档：<https://cloud.spring.io/spring-cloud-static/spring-cloud-config/2.2.0.RC1/reference/html/> 

> 配置中心本质上也是一个微服务，同样需要注册到Eureka服务注册中心！ 



## 15、Config：Git配置管理

### 15.1 远程Git仓库

+ 知名的Git远程仓库有: **国外的GitHub **和 **国内的码云 **，但是使用GitHub时，国内的用户经常遇到的问题是访问速度太慢，有时候还会出现无法连接的情况。如果希望体验更好一些，可以使用国内的Git托管服务——码云。
+ 码云访问地址：https://gitee.com/

### 15.2 创建远程仓库

​	首先要使用码云上的私有远程git仓库需要先注册帐号；请先自行访问网站并注册帐号，然后使用帐号登录码云控制台并创建公开仓库。

![image-20200821171258929](SpringCloud-2.assets/image-20200821171258929.png)



​	![image-20200821171423596](SpringCloud-2.assets/image-20200821171423596.png)



### 15.3 创建配置文件

​	在新建的仓库中创建需要被统一配置管理的配置文件。

​	**配置文件的命名方式：**

 	{application}-{profile}.yml 或 {application}-{profile}.properties 

+ application为应用名称 

+ profile用来区分: 开发环境、测试环境、生产环境等。

  如user-dev.yml，表示用户微服务开发环境下使用的配置文件。这里将user-service工程的配置文件application.yml文件的内容复制作为user-dev.yml文件的内容，具体配置如下：

![image-20200821171806401](SpringCloud-2.assets/image-20200821171806401.png)

​	![image-20200821173428745](SpringCloud-2.assets/image-20200821173428745.png)

​	提交user-dev.yml配置文件之后，gitee中的仓库如下：

 ![image-20200821173616065](SpringCloud-2.assets/image-20200821173616065.png)



## 16、Config：搭建配置中心微服务

### 16.1 创建模块

+ 创建配置中心微服务模块

  ![1572419995349](SpringCloud-2.assets/1572419995349.png)

  ![1572420032713](SpringCloud-2.assets/1572420032713.png) 

+ 添加依赖，修改pom.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
           http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <parent>
          <artifactId>springcloud-demo</artifactId>
          <groupId>cn.itcast</groupId>
          <version>1.0-SNAPSHOT</version>
      </parent>
      <modelVersion>4.0.0</modelVersion>
      <artifactId>config-server</artifactId>
      
      <dependencies>
          <!-- 配置eureka客户端启动器 -->
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
          </dependency>
          <!-- 配置config服务端(依赖了web启动器) -->
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-config-server</artifactId>
          </dependency>
      </dependencies>
  </project>
  ```

### 16.2 启动类

创建配置中心模块config-server启动类ConfigServerApplication.java:

```java
package cn.itcast;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableDiscoveryClient // 开启eureka客户端
@EnableConfigServer // 开启配置中心服务
public class ConfigServerApplication {
    
    public static void main(String[] args){
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

### 16.3 配置文件

创建配置中心工程config-server的配置文件application.yml:

```yaml
server:
  port: 12000

spring:
  application:
    name: config-server

  cloud:
    config:
      server:
        git:
          # 配置git地址
          uri: https://gitee.com/liu-kun1024/master.git

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka, http://localhost:8762/eureka

```

> 注意上面的 spring.cloud.config.server.git.uri 则是在码云创建的仓库地址
>
> 默认配置属性类：MultipleJGitEnvironmentProperties.java

### 16.4 启动测试

启动eureka注册中心和配置中心，然后访问http://localhost:12000/user-dev.yml ，查看能否输出在码云存储管理的user-dev.yml文件。并且可以在gitee上修改user-dev.yml，然后刷新上述测试地址也能及时到最新数据。

![image-20200821173236977](SpringCloud-2.assets/image-20200821173236977.png)

> 注意事项：更新配置文件到本地的两种方式：
>
> 1、重启config-server
>
> 2、浏览器请求查询某个配置文件，例如本例中的http://localhost:12000/user-dev.yml



## 17、Config：获取配置中心配置

前面已经完成了配置中心微服务的搭建，下面我们就需要改造一下用户微服务user-service，配置文件信息不再由微服务项目提供，而是从配置中心获取。如下对user-service工程进行改造。

### 17.1 添加依赖

+ 在user-service模块中，添加如下依赖:

  ```xml
  <!-- 配置config启动器，相当于config客户端  -->
  <dependency>
    	<groupId>org.springframework.cloud</groupId>
    	<artifactId>spring-cloud-starter-config</artifactId>
  </dependency>
  ```

### 17.2 修改配置

+ 删除user-service模块中的application.yml文件（因为该文件从配置中心获取）

+ 创建user-service模块bootstrap.yml配置文件(config客户端配置文件)

  ```properties
  spring:
    cloud:
      config:
        # 与远程仓库中的配置文件的application保持一致
        name: user
        # 与远程仓库中的配置文件的profile保持一致
        profile: dev
        # 与远程仓库中的分支名保持一致
        label: master
        # 配置去哪里发现
        discovery:
          # 启用配置中心
          enabled: true
          # 配置中心服务id
          service-id: config-server
  # 配置eureka
  eureka:
    client:
      service-url: # EurekaServer地址,多个地址以','隔开
        defaultZone: http://localhost:8761/eureka,http://localhost:8762/eureka
  ```

  user-service模块，修改后的结构:

  ![1572426539208](SpringCloud-2.assets/1572426539208.png)&nbsp;

  + bootstrap.yml文件也是Spring Boot的默认配置文件，而且其加载的时间相比于application.yml更早。 
  + application.yml和bootstrap.yml虽然都是Spring Boot的默认配置文件，但是定位却不相同。bootstrap.yml可以理解成系统级别的一些参数配置，这些参数一般是不会变动的。application.yml可以用来定义应用级别的参数。
  + 总结就是：bootstrap.yml文件相当于项目启动时的引导文件，内容相对固定。application.yml文件是微服务的一些常规配置参数，变化比较频繁。

### 17.3 启动测试

启动注册中心、配置中心、用户服务user-service，如果启动没有报错其实已经使用上配置中心内容，可以到注册中心查看，也可以检验user-service的服务。

![1572427096538](SpringCloud-2.assets/1572427096538.png)

> 注意事项：config客户端，也就是本例中的user-service在启动的时候会发送http请求到config-server，config-server也会拉取最新的配置到本地缓存，再把最近的配置返回给user-service，user-service也会以最新的配置来启动容器。



## 18、Spring Cloud Bus：消息总线介绍

> 存在问题

前面已经完成了将微服务中的配置文件集中存储在远程Git仓库，并且通过配置中心微服务从Git仓库拉取配置文件，当用户微服务启动时会连接配置中心获取配置信息从而启动用户微服务。如果我们更新Git仓库中的配置文件，那用户微服务是否可以及时接收到新的配置信息并更新呢？ 

> 测试是否更新Git仓库中的配置文件

+ 修改在码云上的user-dev.yml文件，添加一个属性test.name

  ![1572428018674](SpringCloud-2.assets/1572428018674.png) 

+ 修改user-service工程中的UserController.java

  ```java
  package cn.itcast.user.controller;
  
  import cn.itcast.user.pojo.User;
  import cn.itcast.user.service.UserService;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.beans.factory.annotation.Value;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.PathVariable;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RestController;
  
  @RestController
  @RequestMapping("/user")
  public class UserController {
  
      @Autowired
      private UserService userService;
      
      @Value("${test.name}")
      private String name;
  
      /** 根据主键id查询用户 */
      @GetMapping("/{id}")
      public User findOne(@PathVariable("id")Long id){
          System.out.println("配置文件中的test.name = " + name);
          return userService.findOne(id);
      }
  }
  ```

+ 启动测试

  + 依次启动Eureka、配置中心微服务、用户微服务、然后修改Git仓库中的配置信息，访问用户微服务，查看输出内容。http://localhost:9001/user/1

    ![1572429589291](SpringCloud-2.assets/1572429589291.png)

    ![1572429551939](SpringCloud-2.assets/1572429551939.png) 

  + 结论：通过查看用户微服务控制台的输出结果可以发现，我们对于Git仓库中配置文件的修改并没有及时更新到用户微服务，只有重启用户微服务才能生效。【重启用户微服务效果】

    ![1572429752529](SpringCloud-2.assets/1572429752529.png)

  + 如果想在不重启微服务的情况下更新配置该如何实现呢? 可以使用Spring Cloud Bus来实现配置的自动更新。需要注意的是Spring Cloud Bus底层是基于RabbitMQ实现的，默认使用本地的消息队列服务，所以需要提前启动本地RabbitMQ服务。

> Spring Cloud Bus 消息总线介绍

Spring Cloud Bus是用轻量的消息代理将分布式的节点连接起来，可以用于广播配置文件的更改或者服务的监控管理。一个关键的思想就是，消息总线可以为微服务做监控，也可以实现应用程序之间相互通信。Spring Cloud Bus可选的消息代理有RabbitMQ和Kafka。

使用了Bus之后的架构图: 

![1572430080736](SpringCloud-2.assets/1572430080736.png)

## 19、Spring Cloud Bus：改造配置中心

+ 在config-server模块的pom.xml文件中加入Spring Cloud Bus相关依赖

  ```xml
  <!-- 配置spring-cloud-bus消息总线 -->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-bus</artifactId>
  </dependency>
  <!-- 配置rabbit消息中间件 -->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
  </dependency>
  ```

+ 在config-server模块中修改application.yml文件

  属性配置类：RabbitProperties.java与WebEndpointProperties.java

  ```properties
  server:
    port: 12000
  
  spring:
    application:
      name: config-server
    cloud:
      config:
        server:
          git:
            uri: https://gitee.com/lixiaohua7/config.git
  
    # rabbitmq的配置信息；如下配置的rabbit都是默认值，其实可以完全不配置
    rabbitmq:
      host: localhost
      port: 5672
      username: guest
      password: guest
  
  eureka:
    client:
      service-url:
        defaultZone: http://localhost:8761/eureka,http://localhost:8762/eureka
  
  management:
    endpoints:
      web:
        exposure:
          # 暴露触发消息总线的地址(发送消息到rabbitmq)
          include: bus-refresh
  ```



## 20、Spring Cloud Bus：改造用户服务

+ 在user-service模块的pom.xml中加入Spring Cloud Bus相关依赖

  ```xml
  <!-- 配置spring-cloud-bus消息总线 -->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-bus</artifactId>
  </dependency>
  <!-- 配置rabbit消息中间件 -->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
  </dependency>
  <!-- 配置actuator监控管理springboot应用的启动器，动态刷新配置 -->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  ```

+ 修改user-service模块的bootstrap.yml

  ```properties
  spring:
    cloud:
      config:
        # 与远程仓库中的配置文件的application保持一致
        name: user
        # 与远程仓库中的配置文件的profile保持一致
        profile: dev
        # 与远程仓库中的版本保持一致
        label: master
        discovery:
          # 启用配置中心
          enabled: true
          # 配置中心服务id
          service-id: config-server
    # rabbitmq的配置信息；如下配置的rabbit都是默认值，其实可以完全不配置
    rabbitmq:
      host: localhost
      port: 5672
      username: guest
      password: guest
  
  # 配置eureka
  eureka:
    client:
      service-url: # EurekaServer地址,多个地址以','隔开
        defaultZone: http://localhost:8761/eureka,http://localhost:8762/eureka
  ```

+ 改造user-service模块的UserController.java

  ![1572450619755](SpringCloud-2.assets/1572450619755.png) 

+ 启动测试

  前面已经完成了配置中心微服务和用户微服务的改造，下面来测试一下，当我们修改了Git仓库中的配置文件，用户微服务是否能够在不重启的情况下自动更新配置信息。

  

+ 测试步骤

  + 第一步：依次启动Eureka、配置中心微服务、用户微服务

  + 第二步：访问用户微服务查看输出结果

  + 第三步：修改Git仓库中配置文件内容

  + 第四步：使用Postman或者RESTClient工具发送POST方式，请求访问地址

    http://127.0.0.1:12000/actuator/bus-refresh

    ![1572456439768](SpringCloud-2.assets/1572456439768.png)

  + 第五步：访问用户微服务系统控制台查看输出结果

    ![1572456622779](SpringCloud-2.assets/1572456622779.png)

  #### 说明

  + Postman或者RESTClient是一个可以模拟浏览器发送各种请求（POST、GET、PUT、DELETE等）的工具。
  + 请求地址http://127.0.0.1:12000/actuator/bus-refresh中 /actuator是固定的，/bus-refresh对应的是配置中心config-server中的application.yml文件的配置项include的内容。
  + 请求http://127.0.0.1:12000/actuator/bus-refresh地址的作用是访问配置中心的消息总线服务，消息总线服务接收到请求后会向消息队列中发送消息，用户微服务会监听消息队列。当用户微服务接收到队列中的消息后，会重新从配置中心获取最新的配置信息。

## 21、Spring Cloud：技术体系综合应用

![1572451364036](SpringCloud-2.assets/1572451364036.png)



## 课程总结

+ Feign：服务调用(简化微服务调用)
+ Spring Cloud Gateway: 统一入口，服务网关(路由到微服务)，统一鉴权、分发路由等
+ Spring Cloud Config: 配置中心(配置文件统一管理)
+ Spring Cloud Bus: 消息总线(动态更新配置文件)