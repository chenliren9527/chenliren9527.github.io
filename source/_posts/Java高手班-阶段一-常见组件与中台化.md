---
title: 阶段一 第四章 常见组件与中台化
tags:
  - 中台
  - Java高手课
categories:
  - 阶段一 中台战略与组件化开发专题课程
date: 2021-12-27 23:19:1
---

# 常见组件与中台化

# 1. 中台概述

## 1.1. 中台概念

随着互联网公司的崛起，“中台”这个词也进入了人们的视线。BAT 等公司纷纷推出了自己的中台系统。那么，什么是中台系统?

任何一个软件系统都是通过帮助客户解决问题来实现价值的。针对不同的需求会建立不同的软件项目。这些软件项目包含客户端的应用和后台管理配置的应用。久而久之就形成了固定的“前台”和“后台”系统，而且大家都在乐此不疲地开发着类似的业务系统。

但是，时间一长大家就发现了，这些系统中有一些部分大同小异，在做第二个项目的时候并不用将所有的功能重写，可以把之前项目中那些共有的模块拿出来，稍作修改就可以在新项目中应用了。中台系统就是将“后台”系统中那些针对技术，业务，组织的通用“模块/服务”从原来固定的项目中抽离出来，并且使之能够成为一个自治的服务提供给更多的“前台”使用。

中台是 IT 信息化过程中经验总结的产物，他是前人归纳总结出来的方法论，也是解决问题的思路。它把这些经验和方法从具体的场景中抽离出来，为的是服务于更多的场景，即**能力复用**。

中台的催生基石是**能力共享**。如果中台所提供的能力无法被共享，那就不是中台能力。**如果中台只服务于一个前端应用，那就不是中台。**

中台是为业务服务的，是需要根据企业业务演进逐渐积累而成的，因此**中台的建设不是一蹴而就的。**

**要让中台模式在企业中发挥作用，对企业本身也是有一定要求的，比如企业要有一定规模，业务比较丰富，值得去提炼共性元素形成共享能力。**

## 1.2. 中台背景

许多人通过阿里巴巴推出的中台战略而得知中台这一概念，其实中台是一个旧名词，在很早的时候就出现过，只不过一直没有被真正的应用起来，在2017年通过《企业IT架构转型之道：阿里巴巴中台战略思想与架构实战》一书热度上升起来，那么阿里提出的中台究竟是什么。

**阿里中台主要应用于业务种类繁杂、个人/企业会员数量庞大、不同种类业务相互交叉依赖，需要根据需求变化快速响应调整的O2O线上线下协同的电商场景。**阿里中台将技术、数据、业务能力从前台剥离，形成技术中台提供系统性的后端服务，为前台零售电商业务提供支撑，中台主要分为业务中台与数据中台两大块。

业务中台主要针对其电商交易业务，具体包括售前、售中、售后三部分，业务中台被划分为许多中心服务，即会员中心、商品中心、交易中心、评价中心、店铺中心、支付中心、营销中心、库存中心，贯穿整个电商业务流程。数据中台主要提供大数据相关的计算、存储、共享能力，将线上与线下的数据源进行连接打通，进而实现决策分析、精准营销等战略手段。

![1597655324153](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/1597655324153.png) 

阿里提出中台战略之后，滴滴、京东等互联网厂商陆续推出对应的中台战略，同时数据中台、业务中台、技术中台、内容中台等多种形式中台出现。

## 1.3. 中台分类

中台是企业级共享能力平台，下图展示的是中台的架构体系。

![1597658262030](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/1597658262030.png) 

- 业务中台

业务中台泛指代表企业核心业务特征的中台结构，更多的是对业务的支持，实现后端业务资源到前台应用能力的转化，业务中台多由不同的业务板块组成。

- 技术中台

技术中台整合和包装了云基础设施，以及在其上建设的各种技术中间件，比如微服务、分布式缓存、消息队列、搜索引擎、分布式数据库等，并在此基础上建设和封装了简单易用的能力接口。技术中台作为工具和组件，为建设前台应用和业务中台提供了基础设施重用的能力，大大缩短了它们的建设周期。

- 研发中台

研发中台是关注系统开发效率的管理平台。软件开发和系统建设是一项工程，涉及项目管理、团队协作、流程、测试、部署、运营、监控等方面。 如何将在企业应用开发过程中的最佳实践沉淀为可重用的能力，从而更好地快速迭代开发创新型的应用，也是很多企业目前的一个关注点。

研发中台为应用开发提供了流程、质量管控和持续交付的能力，包括敏捷开发管理、开发流水线、部署流水线、持续交付。

例如开发流水线则涉及源代码的版本管理、分支的创建、合并和提交，半成品的构建、存储和使用以及产成品的构建。将产品部署到指定环境并上线运行是部署流水线的职责。

- 数据中台

数据中台泛指通过数据处理、分析等技术，对企业内外部海量数据进行采集、计算、存储、加工、分析等一系列活动，凸显数据价值，加强企业对数据的利用。数据中台提供数据能力直接服务于业务。

# 2. 常用组件服务介绍

## 2.1. 通用权限系统

### 2.1.1. 功能概述

对于企业中的项目绝大多数都需要进行用户权限管理、认证、鉴权、加密、解密、XSS防跨站攻击等。这些功能整体实现思路基本一致，但是大部分项目都需要实现一次，这无形中就形成了巨大的资源浪费。本项目就是针对这个问题，提供了一套通用的权限解决方案----品达通用权限系统。

品达通用权限系统基于SpringCloud(Hoxton.SR1)  +SpringBoot(2.2.2.RELEASE) 的微服务框架，具备通用的用户管理、资源权限管理、网关统一鉴权、XSS防跨站攻击等多个模块，支持多业务系统并行开发，支持多服务并行开发，可以作为后端服务的开发脚手架。核心技术采用SpringBoot、Zuul、Nacos、Fegin、Ribbon、Hystrix、JWT Token、Mybatis Plus等主要框架和中间件。

本项目具有两个主要功能特性：

- 用户权限管理
  
  具有用户、部门、岗位、角色、菜单管理，并通过网关进行统一的权限认证

- 微服务开发框架
  
  本项目同时也是一个微服务开发框架，集成了基础的公共组件，包括数据库、缓存、日志、表单验证、对象转换、防注入和接口文档管理等工具。

系统架构：

![image-20201207094701062](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20201207094701062.png)

技术架构：

![image-20201207101527251](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20201207101527251.png)

工程结构：

![image-20201207095017082](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20201207095017082.png)

项目服务有两个：网关服务和权限服务

| 应用             | 端口   | 说明   | 启动命令                           |
| -------------- | ---- | ---- | ------------------------------ |
| pd-gateway     | 8760 | 网关服务 | java -jar pd-gateway.jar &     |
| pd-auth-server | 8764 | 权限服务 | java -jar pd-auth-server.jar & |

本系统是基于当前非常流行的前后端分离开发方式开发。

环境要求：

- JDK ： 1.8 + 

- Maven： 3.3 + 

- Mysql： 5.6.0 + 

- Redis： 4.0 + 

- Nacos： 1.3.0

- Node： 11.3+（集成npm）

### 2.1.2. 应用场景

1、新项目开发

如果是新项目开发，可以在品达通用权限系统的基础上进行相关的业务开发，其实就是将通用权限系统当做开发脚手架在此基础之上快速开始业务开发。

2、已有项目集成

对于已经开发完成的项目，可以直接集成通用权限系统进行权限管理、认证鉴权。

### 2.1.3. 重点代码介绍

网关服务：

```
BaseFilter：基础网关过滤器
TokenContextFilter：解析token中的用户信息过滤器
AccessFilter：权限验证过滤器
```

权限服务：

```
LoginController：登录控制器
MenuController：菜单管理控制器
ResourceController：资源管理控制器
RoleController：角色管理控制器
UserController：用户管理控制器
OrgController：组织管理控制器
StationController：岗位管理控制器
```

### 2.1.4. 使用说明

#### 2.1.4.1 新项目开发

如果是新项目开发，可以在品达通用权限系统的基础上进行相关的业务开发，其实就是将通用权限系统当做开发脚手架在此基础之上快速开始业务开发。

本小节通过一个商品服务的案例来讲解如何基于品达通用权限系统进行新业务的开发。

##### 2.1.4.1.1 数据库环境搭建

创建数据库pd_goods并创建表pd_goods_info：

![image-20200415092939839](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200415092939839.png)

##### 2.1.4.1.2 后端业务功能开发

###### 2.1.4.1.2.1 创建工程

在品达通用权限系统基础上创建商品服务相关模块，如下图：

![image-20200415093512088](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200415093512088.png)

```file
pd-goods              #商品服务父工程
├── pd-goods-entity      #实体
├── pd-goods-server      #服务
```

###### 2.1.4.1.2.2 pd-goods-entity开发

第一步：配置pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>pd-goods</artifactId>
        <groupId>com.itheima</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>pd-goods-entity</artifactId>
    <description>接口服务实体模块</description>
    <dependencies>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-tools-common</artifactId>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-core</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus</artifactId>
        </dependency>
    </dependencies>
</project>
```

第二步：创建商品实体类

```java
package com.itheima.pinda.goods.entity;

import com.baomidou.mybatisplus.annotation.TableName;
import com.itheima.pinda.base.entity.Entity;
import lombok.*;
import lombok.experimental.Accessors;
import java.math.BigDecimal;
import java.time.LocalDateTime;

@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Accessors(chain = true)
@TableName("pd_goods_info")
public class GoodsInfo extends Entity<Long> {

    private static final long serialVersionUID = 1L;

    /**
     * 商品编码
     */
    private String code;

    /**
     * 商品名称
     */
    private String name;

    /**
     * 国条码
     */
    private String barCode;

    /**
     * 品牌表id
     */
    private Integer brandId;

    /**
     * 一级分类id
     */
    private Integer oneCategoryId;

    /**
     * 二级分类id
     */
    private Integer twoCategoryId;

    /**
     * 三级分类id
     */
    private Integer threeCategoryId;

    /**
     * 商品的供应商id
     */
    private Integer supplierId;

    /**
     * 商品售价价格
     */
    private BigDecimal price;

    /**
     * 商品加权平均成本
     */
    private BigDecimal averageCost;

    /**
     * 上下架状态:0下架，1上架
     */
    private boolean publishStatus;

    /**
     * 审核状态: 0未审核，1已审核
     */
    private boolean auditStatus;

    /**
     * 商品重量
     */
    private Float weight;

    /**
     * 商品长度
     */
    private Float length;

    /**
     * 商品重量
     */
    private Float height;

    /**
     * 商品宽度
     */
    private Float width;

    /**
     * 颜色
     */
    private String color;

    /**
     * 生产日期
     */
    private LocalDateTime productionDate;

    /**
     * 商品有效期
     */
    private Integer shelfLife;

    /**
     * 商品描述
     */
    private String descript;

}
```

第三步：创建商品操作对应的多个DTO类

```java
package com.itheima.pinda.goods.dto;

import com.itheima.pinda.goods.entity.GoodsInfo;
import lombok.*;
import lombok.experimental.Accessors;
import java.time.LocalDateTime;

@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Accessors(chain = true)
public class GoodsInfoPageDTO extends GoodsInfo {
    private LocalDateTime startCreateTime;
    private LocalDateTime endCreateTime;
}
```

```java
package com.itheima.pinda.goods.dto;

import com.itheima.pinda.goods.entity.GoodsInfo;

public class GoodsInfoSaveDTO extends GoodsInfo {
}
```

```java
package com.itheima.pinda.goods.dto;

import com.itheima.pinda.goods.entity.GoodsInfo;

public class GoodsInfoUpdateDTO extends GoodsInfo {
}
```

###### 2.1.4.1.2.3 pd-goods-server开发

第一步：配置pom.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>pd-goods</artifactId>
        <groupId>com.itheima</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>pd-goods-server</artifactId>
    <description>接口服务启动模块</description>
    <dependencies>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-tools-log</artifactId>
        </dependency>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-tools-swagger2</artifactId>
        </dependency>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-tools-validator</artifactId>
        </dependency>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-tools-xss</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>fastjson</artifactId>
                    <groupId>com.alibaba</groupId>
                </exclusion>
                <exclusion>
                    <groupId>com.google.guava</groupId>
                    <artifactId>guava</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>com.google.guava</groupId>
                    <artifactId>guava</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>com.google.guava</groupId>
                    <artifactId>guava</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.ow2.asm</groupId>
            <artifactId>asm</artifactId>
            <version>${asm.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.tomcat.embed</groupId>
                    <artifactId>tomcat-embed-websocket</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-undertow</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-json</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-tools-databases</artifactId>
        </dependency>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-tools-dozer</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-context</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-goods-entity</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>
    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

第二步：导入资料中提供的配置文件

![image-20200415095309623](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200415095309623.png)

第三步：在配置中心Nacos中创建pd-goods-server.yml

![image-20200415095706237](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200415095706237.png)

配置文件内容如下：

```yaml
# 在这里配置 权限服务 所有环境都能使用的配置
pinda:
  mysql:
    database: pd_goods
  swagger:
    enabled: true
    docket:
      core:
        title: 核心模块
        base-package: com.itheima.pinda.goods.controller

server:
  port: 8767
```

第四步：编写启动类

```java
package com.itheima.pinda;

import com.itheima.pinda.validator.config.EnableFormValidator;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.hystrix.EnableHystrix;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.env.Environment;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import java.net.InetAddress;
import java.net.UnknownHostException;

@SpringBootApplication
@EnableDiscoveryClient
@EnableHystrix
@EnableFeignClients(value = {
        "com.itheima.pinda",
})
@EnableTransactionManagement
@Slf4j
@EnableFormValidator
public class GoodsServerApplication {
    public static void main(String[] args) throws UnknownHostException {
        ConfigurableApplicationContext application = SpringApplication.run(GoodsServerApplication.class, args);
        Environment env = application.getEnvironment();
        log.info("\n----------------------------------------------------------\n\t" +
                        "应用 '{}' 运行成功! 访问连接:\n\t" +
                        "Swagger文档: \t\thttp://{}:{}/doc.html\n\t" +
                        "----------------------------------------------------------",
                env.getProperty("spring.application.name"),
                InetAddress.getLocalHost().getHostAddress(),
                env.getProperty("server.port"));

    }
}
```

第五步：导入资料中提供的配置类

![image-20200415100417595](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200415100417595.png)

第六步：创建Mapper接口

```java
package com.itheima.pinda.goods.dao;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.itheima.pinda.goods.entity.GoodsInfo;
import org.springframework.stereotype.Repository;

/**
 * Mapper 接口
 */
@Repository
public interface GoodsInfoMapper extends BaseMapper<GoodsInfo> {
}
```

第七步：创建Service接口和实现类

```java
package com.itheima.pinda.goods.service;

import com.baomidou.mybatisplus.extension.service.IService;
import com.itheima.pinda.goods.entity.GoodsInfo;

public interface GoodsInfoService extends IService<GoodsInfo> {
}
```

```java
package com.itheima.pinda.goods.service.impl;

import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.itheima.pinda.goods.dao.GoodsInfoMapper;
import com.itheima.pinda.goods.entity.GoodsInfo;
import com.itheima.pinda.goods.service.GoodsInfoService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Slf4j
@Service
public class GoodsInfoServiceImpl extends ServiceImpl<GoodsInfoMapper, GoodsInfo> implements GoodsInfoService {
}
```

第八步：创建Controller

```java
package com.itheima.pinda.goods.controller;

import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.itheima.pinda.base.BaseController;
import com.itheima.pinda.base.R;
import com.itheima.pinda.base.entity.SuperEntity;
import com.itheima.pinda.database.mybatis.conditions.Wraps;
import com.itheima.pinda.database.mybatis.conditions.query.LbqWrapper;
import com.itheima.pinda.dozer.DozerUtils;
import com.itheima.pinda.goods.dto.GoodsInfoPageDTO;
import com.itheima.pinda.goods.dto.GoodsInfoSaveDTO;
import com.itheima.pinda.goods.dto.GoodsInfoUpdateDTO;
import com.itheima.pinda.goods.entity.GoodsInfo;
import com.itheima.pinda.goods.service.GoodsInfoService;
import com.itheima.pinda.log.annotation.SysLog;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@Slf4j
@Validated
@RestController
@RequestMapping("/goodsInfo")
@Api(value = "GoodsInfo", tags = "商品信息")
public class GoodsInfoController extends BaseController {
    @Autowired
    private DozerUtils dozer;
    @Autowired
    private GoodsInfoService goodsInfoService;

    /**
     * 分页查询商品信息
     *
     * @param data 分页查询对象
     * @return 查询结果
     */
    @ApiOperation(value = "分页查询商品信息", notes = "分页查询商品信息")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "current", value = "当前页", dataType = "long", paramType = "query", defaultValue = "1"),
            @ApiImplicitParam(name = "size", value = "每页显示几条", dataType = "long", paramType = "query", defaultValue = "10"),
    })
    @GetMapping("/page")
    @SysLog("分页查询商品信息")
    public R<IPage<GoodsInfo>> page(GoodsInfoPageDTO data) {
        Page<GoodsInfo> page = getPage();
        LbqWrapper<GoodsInfo> wrapper = Wraps.lbQ();

        wrapper.like(GoodsInfo::getName, data.getName())
                .like(GoodsInfo::getCode, data.getCode())
                .eq(GoodsInfo::getBarCode, data.getBarCode())
                .geHeader(GoodsInfo::getCreateTime, data.getStartCreateTime())
                .leFooter(GoodsInfo::getCreateTime, data.getEndCreateTime())
                .orderByDesc(GoodsInfo::getCreateTime);

        goodsInfoService.page(page, wrapper);
        return success(page);
    }

    @ApiOperation(value = "查询商品信息", notes = "查询商品信息")
    @GetMapping("/list")
    @SysLog("查询商品信息")
    public R<List<GoodsInfo>> list(GoodsInfoPageDTO data) {

        LbqWrapper<GoodsInfo> wrapper = Wraps.lbQ();

        wrapper.like(GoodsInfo::getName, data.getName())
                .like(GoodsInfo::getCode, data.getCode())
                .eq(GoodsInfo::getBarCode, data.getBarCode())
                .geHeader(GoodsInfo::getCreateTime, data.getStartCreateTime())
                .leFooter(GoodsInfo::getCreateTime, data.getEndCreateTime())
                .orderByDesc(GoodsInfo::getCreateTime);

        return success(goodsInfoService.list(wrapper));
    }

    /**
     * 查询商品信息
     *
     * @param id 主键id
     * @return 查询结果
     */
    @ApiOperation(value = "查询商品信息", notes = "查询商品信息")
    @GetMapping("/{id}")
    @SysLog("查询商品信息")
    public R<GoodsInfo> get(@PathVariable Long id) {
        return success(goodsInfoService.getById(id));
    }

    /**
     * 新增商品信息
     *
     * @param data 新增对象
     * @return 新增结果
     */
    @ApiOperation(value = "新增商品信息", notes = "新增商品信息不为空的字段")
    @PostMapping
    @SysLog("新增商品信息")
    public R<GoodsInfo> save(@RequestBody @Validated GoodsInfoSaveDTO data) {
        GoodsInfo GoodsInfo = dozer.map(data, GoodsInfo.class);
        goodsInfoService.save(GoodsInfo);
        return success(GoodsInfo);
    }

    /**
     * 修改商品信息
     *
     * @param data 修改对象
     * @return 修改结果
     */
    @ApiOperation(value = "修改商品信息", notes = "修改商品信息不为空的字段")
    @PutMapping
    @SysLog("修改商品信息")
    public R<GoodsInfo> update(@RequestBody @Validated(SuperEntity.Update.class) GoodsInfoUpdateDTO data) {
        GoodsInfo GoodsInfo = dozer.map(data, GoodsInfo.class);
        goodsInfoService.updateById(GoodsInfo);
        return success(GoodsInfo);
    }

    /**
     * 删除商品信息
     *
     * @param ids 主键id
     * @return 删除结果
     */
    @ApiOperation(value = "删除商品信息", notes = "根据id物理删除商品信息")
    @SysLog("删除商品信息")
    @DeleteMapping
    public R<Boolean> delete(@RequestParam("ids[]") List<Long> ids) {
        goodsInfoService.removeByIds(ids);
        return success();
    }
}
```

##### 2.1.4.1.3 配置网关路由规则

在Nacos中的pd-gateway.yml中新增商品服务相关的路由配置，内容如下：

```yaml
zuul:
  #  debug:
  #    request: true
  #  include-debug-header: true
  retryable: false
  servlet-path: /         # 默认是/zuul , 上传文件需要加/zuul前缀才不会出现乱码，这个改成/ 即可不加前缀
  ignored-services: "*"   # 忽略eureka上的所有服务
  sensitive-headers:  # 一些比较敏感的请求头，不想通过zuul传递过去， 可以通过该属性进行设置
  #  prefix: /api #为zuul设置一个公共的前缀
  #  strip-prefix: false     #对于代理前缀默认会被移除   故加入false  表示不要移除
  routes:  # 路由配置方式
    authority:  # authority是路由名称，可以随便定义，但是path和service-id需要一一对应
      path: /authority/**
      serviceId: pd-auth-server
    goods:
      path: /goods/**
      serviceId: pd-goods-server
```

##### 2.1.4.1.4 前端开发

可以将pinda-authority-ui作为前端开发脚手架，基于此工程开发商品服务相关页面。资料中已经提供了开发完成的前端工程，直接运行即可。

##### 2.1.4.1.5 配置菜单和资源权限

启动网关服务、权限服务、商品服务、前端工程，使用管理员账号登录，配置商品服务相关的菜单和对应的资源权限。

![image-20200415102224580](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200415102224580.png)

![image-20200415102314102](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200415102314102.png)

goodsInfo:add    新增        POST    /goodsInfo
goodsInfo:update    修改        PUT    /goodsInfo
goodsInfo:delete    删除        DELETE    /goodsInfo
goodsInfo:view    列表        GET    /goodsInfo/page

##### 2.1.4.1.6 配置角色

启动网关服务和权限服务，使用管理员账号登录。创建新角色并进行配置(菜单权限和资源权限)和授权(为用户分配角色)。

![image-20200415105314675](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200415105314675.png)

![image-20200415105358366](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200415105358366.png)

![image-20200415105457620](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200415105457620.png)

#### 2.1.4.2 已有项目集成

本小节通过一个已经完成开发的TMS(品达物流)项目来展示如何进行已有项目集成的过程。

##### 2.1.4.2.1 TMS调整

###### 2.1.4.2.1.1 页面菜单

对于已经完成相关业务开发的项目，可以将其前端系统的页面通过iframe的形式内嵌到通用权限系统的前端页面中，这就需要对其前端系统的页面进行相应的修改。因为原来的TMS系统前端页面的左侧菜单和导航菜单都在自己页面中展示，现在需要将这些菜单配置到通用权限系统中，通过权限系统的前端系统来展示。

![image-20200423103130632](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200423103130632.png)

###### 2.1.4.2.1.2 请求地址

为了能够进行鉴权相关处理，需要将TMS前端发送的请求首先经过通用权限系统的网关进行处理：

![img](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/clip_image002-1587612549680.jpg)

##### 2.1.4.2.2 网关路由配置

配置通用权限系统的网关路由规则，将针对TMS的请求转发到TMS相关服务：

```yaml
zuul:
  retryable: false
  servlet-path: /
  ignored-services: "*"   # 忽略eureka上的所有服务
  sensitive-headers:  # 一些比较敏感的请求头，不想通过zuul传递过去， 可以通过该属性进行设置
  routes:  # 路由配置方式
    authority: 
      path: /authority/**
      serviceId: pd-auth-server
    pay:
      path: /pay/**
      serviceId: pd-ofpay-server
    web-manager:
      path: /web-manager/**
      serviceId: pd-web-manager
    web-xczx:
      path: /xczx/api/**
      url: http://xc-main-java.itheima.net:7291/api/
```

##### 2.1.4.2.3 通用权限系统配置

###### 2.1.4.2.3.1 菜单配置

登录通用权限系统，配置TMS项目相应的菜单：

![image-20200423113627553](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200423113627553.png)

###### 2.1.4.2.3.2 资源权限配置

资源权限都是关联到某个菜单下的，所以要配置资源权限需要先选中某个菜单，然后就可以配置相关资源权限了：

![image-20200423114115893](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200423114115893.png)

###### 2.1.4.2.3.3 角色配置

登录通用权限系统，在角色管理菜单中配置TMS项目中使用到的角色：

![image-20200423114347953](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200423114347953.png)

角色创建完成后可以为角色配置菜单权限和资源权限：

![image-20200423114722781](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200423114722781.png)

完成角色的菜单权限和资源权限配置后可以将角色授权给用户：

![image-20200423115004298](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200423115004298.png)

![image-20200423115042703](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20200423115042703.png)

## 2.2. 注册登录服务

### 2.2.1. 功能概述

登录认证几乎是任何一个系统的标配，web 系统、APP、PC 客户端等都需要注册、登录、认证。

以淘宝为例，如果我们想要下单，首先需要注册一个账号。拥有了账号之后，我们需要输入用户名、密码完成登录过程。之后如果你在一段时间内再次进入系统，是不需要输入用户名和密码的，只有在长时间不登录的情况下访问系统才需要再次输入用户名和密码。 

本服务提供多种登录方式：用户名密码登录、手机验证码登录、邮箱登录、微信扫码登录、微博登录、qq登录，登录成功自动完成注册。

工程结构：

![image-20201211112954510](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20201211112954510.png)

项目服务有两个：网关服务和注册登录服务

| 应用           | 端口   | 说明     | 启动命令                         |
| ------------ | ---- | ------ | ---------------------------- |
| auth-gateway | 8782 | 网关服务   | java -jar auth-gateway.jar & |
| auth         | 8783 | 注册登录服务 | java -jar auth.jar &         |

![image-20201211134913609](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20201211134913609.png)

项目依赖环境：

- mysql
- redis
- nacos

### 2.2.2. 应用场景

针对互联网用户：

- 用户名密码登录/注册
- 手机验证码登录/注册
- 电子邮箱登录/注册
- 微信登录/注册
- 微博登录/注册
- qq登录/注册

### 2.2.3. 使用说明

第一步：部署网关服务和注册登录服务并成功启动

![image-20201211140800511](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20201211140800511.png)

第二步：开发自己的业务应用

第三步：在网关服务配置文件中配置路由规则，由网关服务进行jwt校验

```yaml
spring:
  cloud:
    # 路由网关配置
    gateway:
      # 配置路由规则
      routes:
        # 采用自定义路由 ID（有固定用法，不同的 id 有不同的功能，详见：
        - id: CUST-AUTH
          # 采用 LoadBalanceClient 方式请求，以 lb:// 开头，后面的是注册在 Nacos 上的服务名
          uri: lb://cust-auth
          # Predicate 翻译过来是“谓词”的意思，必须，主要作用是匹配用户的请求，有很多种用法
          predicates:
            - Path=/cust/**
          filters:
            - StripPrefix= 1
        # 配置其他业务微服务
        - id: CUST-AUTH-DEMO
          uri: lb://cust-auth-demo
          predicates:
            - Path=/demo/**
          filters:
            - StripPrefix= 1
        # 配置其他业务微服务
        - id: CUST-USER
          uri: lb://cust-user
          predicates:
            - Path=/user/**
          filters:
            - StripPrefix= 1
```

第四步：修改业务应用页面，请求时需要将token放在请求头中提交

## 2.3. 分布式ID

### 2.3.1. 功能概述

ID，全称Identifier，中文翻译为标识符，是用来唯一标识对象或记录的符号。比如我们每个人都有自己的身份证号，这个就是我们的标识符，有了这个唯一标识，就能快速识别出每一个人。

**在计算机世界里，复杂的分布式系统中，经常需要对大量的数据、消息、HTTP 请求等进行唯一标识。**比如对于分微服务架构的系统中，服务间相互调用需要唯一标识，幂等处理，调用链路分析，日志追踪的时候都需要使用这个唯一标识，此时我们的系统就迫切的需要一个全局唯一的ID。

另外随着社会的发展，各种金融、电商、支付、等系统中产生的数据越来越多，对数据库进行分库分表是比较常见的，而分库后则需要有一个唯一ID来标识一条数据或消息，单个数据库的自增ID显然不能满足需求，此时也会需要一个能够生成全局唯一ID的系统。

工程结构：

![image-20201210152930524](Java%E9%AB%98%E6%89%8B%E7%8F%AD-%E9%98%B6%E6%AE%B5%E4%B8%80-%E5%B8%B8%E8%A7%81%E7%BB%84%E4%BB%B6%E4%B8%8E%E4%B8%AD%E5%8F%B0%E5%8C%96.assets/image-20201210152930524.png)

### 2.3.2. 应用场景

**1、全局唯一**

这个最简单，就是说不能出现重复的ID，既然是唯一标识，这是最基本的要求。

**2、趋势递增**

**先来了解下什么是趋势递增？**

简单说就是在一段时间内，生成的ID是递增的趋势，而不强求下一个ID必须大于前一ID。例如在一段时间内生成的ID在【0，1000】之间，过段时间生成的ID在【1000，2000】之间。

**为什么要趋势递增？**

目前大部分的互联网公司使用了开源的MySQL数据库，存储引擎选择InnoDB。MySQL InnoDB引擎中使用的是聚集索引，由于多数RDBMS数据库使用B-tree的数据结构来存储索引数据，在主键的选择上面我们应该尽量使用有序的主键，这样在插入新的数据时B-tree的结构不会时常被打乱重塑，能有效的提高存取效率。

**3、单调递增**

通俗的说就是下一个ID一定大于上一个ID，例如事务版本号、IM增量消息、排序等特殊需求。

**4、信息安全**

如果ID是连续递增的，那么恶意用户可以根据当前ID推测出下一个ID，爬取系统中数据的工作就非常容易实现，直接按照顺序访问指定URL即可；如果是订单号就更加危险，竞争对手可以直接知道系统一天的总订单量。所以在一些应用场景下，会需要ID无规则、不规则，切不易被破解。

### 2.3.4. 使用说明

分布式ID生成系统部署完成后，第三方系统接入即可直接获取ID。

1. 引入distributedid-client依赖：在项目pom.xml添加坐标
   
   ```xml
   <dependencies>
        <dependency>
             <groupId>com.itheima.distributedid</groupId>
             <artifactId>distributedid-client</artifactId>
             <version>1.0-SNAPSHOT</version>
        </dependency>
   </dependencies>
   ```

```
2. 分布式ID生成系统客户端配置，在项目resources目录下编辑distributedid_client.properties

   ```properties
   #服务器地址
   distributedid.server=211.103.136.244:7315
   #部署多个的话，可自行添加
   #distributedid.server=211.103.136.244:7315,ip2:port,...
   #超时时间
   distributedid.readTimeout=5000
   distributedid.connectTimeout=5000
```

3. 获取ID时，直接调用即可
   
   ```java
   Long id = 0L;
   //从服务端获取自增型ID
   id = DistributedId.autoincrementId("your service name");
   
   //本地生成雪花算法ID
   id = DistributedId.snowflake();
   
   //从服务端获取雪花算法ID
   id = DistributedId.snowflakeFromServer();
   ```
