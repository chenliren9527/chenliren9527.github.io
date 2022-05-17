---
title: 乐优电商(12)-JWT+RSA微服务鉴权
tags:
  - 笔记
  - 项目实战二
  - springcloud
  - 微服务
  - Swagger
  - JWT
  - RSA
  - 用户鉴权
  - 无状态认证
  - 登录
  - CAS
  - 单点登录
  - Spring的生命周期
categories:
  - 项目实战二
date: 2021-01-09 22:34:44
---

## 01、课程目标

- 了解什么是无状态登录
- 了解JWT原理
- 实现登录授权功能
- 实现首页登录状态判断





## 02、用户注册：服务端数据校验

刚才虽然实现了注册，但是服务端并没有进行数据校验，而前端的校验是很容易被有心人绕过的。所以我们必须在后台添加数据校验功能：

我们这里会使用Hibernate-Validator框架完成数据校验：

而SpringBoot的web启动器中已经集成了相关依赖：

![1577336953660](乐优电商-12-JWT-RSA微服务鉴权.assets/1577336953660.png) 



### 1）什么是Hibernate Validator

Hibernate Validator是Hibernate提供的一个开源框架，使用注解方式非常方便的实现服务端的数据校验。

官网：http://hibernate.org/validator/

![1577337059910](乐优电商-12-JWT-RSA微服务鉴权.assets/1577337059910.png) 



**hibernate Validator** 是 Bean Validation 的参考实现 。

Hibernate Validator 提供了 JSR 303 规范中所有内置 constraint（约束） 的实现，除此之外还有一些附加的 constraint。

在日常开发中，Hibernate Validator经常用来验证bean的字段，基于注解，方便快捷高效。





### 2）Bean校验的注解

常用注解如下：

| **Constraint**                                     | **详细信息**                                                 |
| -------------------------------------------------- | ------------------------------------------------------------ |
| **@Valid**                                         | 被注释的元素是一个对象，需要检查此对象的所有字段值           |
| **@Null**                                          | 被注释的元素必须为 null                                      |
| **@NotNull**                                       | 被注释的元素必须不为 null                                    |
| **@AssertTrue**                                    | 被注释的元素必须为 true                                      |
| **@AssertFalse**                                   | 被注释的元素必须为 false                                     |
| **@Min(value)**                                    | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值     |
| **@Max(value)**                                    | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值     |
| **@DecimalMin(value)**                             | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值     |
| **@DecimalMax(value)**                             | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值     |
| **@Size(max,   min)**                              | 被注释的元素的**数字**大小必须在指定的范围内                 |
| **@Digits   (integer, fraction)**                  | 被注释的元素必须是一个数字，其值必须在可接受的范围内         |
| **@Past**                                          | 被注释的元素必须是一个过去的日期                             |
| **@Future**                                        | 被注释的元素必须是一个将来的日期                             |
| **@Pattern(value)**                                | 被注释的元素必须符合指定的正则表达式                         |
| **@Email**                                         | 被注释的元素必须是电子邮箱地址                               |
| **@Length**                                        | 被注释的**字符串**的大小必须在指定的范围内                   |
| **@NotEmpty**                                      | 被注释的字符串的必须非空                                     |
| **@Range**                                         | 被注释的元素必须在合适的范围内                               |
| **@NotBlank**                                      | 被注释的字符串的必须非空                                     |
| **@URL(protocol=,host=,   port=,regexp=, flags=)** | 被注释的字符串必须是一个有效的url                            |
| **@CreditCardNumber**                              | 被注释的字符串必须通过Luhn校验算法，银行卡，信用卡等号码一般都用Luhn计算合法性 |

> **@NotEmpty、@NotBlank、@NotNull的区别**
>
> @NotEmpty 用在集合类上面
> @NotBlank 用在String上面
> @NotNull  用在基本类型上
>
> **@NotEmpty**
>
> ```java
> 
> /**
> * Asserts that the annotated string, collection, map or array is not {@code null} or empty.
> *
> * @author Emmanuel Bernard
> * @author Hardy Ferentschik
> */
> ```
>
> 也就是说，加了@NotEmpty的String类、Collection、Map、数组，是不能为null或者长度为0的（String、Collection、Map的isEmpty()方法）。
>
> **@NotBlank**
>
> ```java
> /**
> * Validate that the annotated string is not {@code null} or empty.
> * The difference to {@code NotEmpty} is that trailing whitespaces are getting ignored.
> *
> * @author Hardy Ferentschik
> */
> ```
>
> “The difference to {@code NotEmpty} is that trailing whitespaces are getting ignored.” 
>
> 和{@code NotEmpty}不同的是，尾部空格被忽略，也就是说，纯空格的String也是不符合规则的。所以才会说@NotBlank用于String。
>
> **@NotNull**
>
> ```java
> /**
> * The annotated element must not be {@code null}.
> * Accepts any type.
> *
> * @author Emmanuel Bernard
> */
> ```
>
> 不能为null。

### 3）给User添加校验

在ly-pojo父工程中导入hibernate校验数据的jar包

```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
</dependency>
```

我们在User对象的部分属性上添加注解：

```java
@TableName("tb_user")
@Data
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    @Size(min = 4, max = 10, message = "用户名格式不正确")
    private String username;
    @Length(min = 4, max = 10, message = "密码格式不正确")
    private String password;
    private String phone;
    private Date createTime;
    private Date updateTime;
}
```



### 4）在controller上进行控制

在controller中只需要给需要做校验的User添加 **@Valid注解**即可。

![1577345073689](乐优电商-12-JWT-RSA微服务鉴权.assets/1577345073689.png) 





### 5）测试

我们故意填错，然后SpringMVC会自动返回错误信息：

![1577345163508](乐优电商-12-JWT-RSA微服务鉴权.assets/1577345163508.png) 



如果需要自定义返回结果，可以这么写：

> BindingResult 必须紧跟在**@Valid注解**后面

```java
 /**
     * 用户注册
     * BindingResult: 封装了所有字段验证失败的错误信息
     */
    @PostMapping("/register")
    public ResponseEntity<Void> register(@Valid User user, BindingResult result, @RequestParam("code") String code){
        if(result.hasErrors()){
            //取出错误信息
            String errorMsg = result.getFieldErrors()
                                        .stream()
                                        .map(FieldError::getDefaultMessage)
                                        .collect(Collectors.joining("|"));

            //自行封装异常信息
            throw new LyException(500,errorMsg);
        }

        userService.register(user,code);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
```

我们在User参数后面跟一个BindingResult参数，不管校验是否通过，都会进入方法内部。如何判断校验是否通过呢？

BindingResult中会封装错误结果，我们通过result.hashErrors来判断是否有错误，然后通过result.getFieldErrors来获取错误信息。

再次测试：

![1577345745350](乐优电商-12-JWT-RSA微服务鉴权.assets/1577345745350.png) 



## 03、API文档：Swagger的介绍

完成了用户中心接口的开发，接下来我们就要测试自己的接口了，而且为了方便前端调用和参考，我们最好提供一份更直观的api文档，这里我们介绍一个工具，叫做swagger-ui

什么是swagger呢？swagger是对Open-API的一种实现。那么，什么是OpenAPI呢？

### 1）什么是OpenAPI

随着互联网技术的发展，现在的网站架构基本都由原来的后端渲染，变成了：前端渲染、前后端分离的形态，而且前端技术和后端技术在各自的道路上越走越远。  前端和后端的唯一联系，变成了API接口；API文档变成了前后端开发人员联系的纽带，变得越来越重要。

没有API文档工具之前，大家都是手写API文档的，在什么地方书写的都有，而且API文档没有统一规范和格式，每个公司都不一样。这无疑给开发带来了灾难。

OpenAPI规范（OpenAPI Specification 简称OAS）是Linux基金会的一个项目，试图通过定义一种用来描述API格式或API定义的语言，来规范RESTful服务开发过程。目前V3.0版本的OpenAPI规范已经发布并开源在github上 。

官网：https://github.com/OAI/OpenAPI-Specification





### 2）什么是swagger（丝袜哥）？

OpenAPI是一个编写API文档的规范，然而如果手动去编写OpenAPI规范的文档，是非常麻烦的。而Swagger就是一个实现了OpenAPI规范的工具集。

官网：https://swagger.io/

看官方的说明：



Swagger包含的工具集：

- **Swagger编辑器：** Swagger Editor允许您在浏览器中编辑YAML中的OpenAPI规范并实时预览文档。
- **Swagger UI：** Swagger UI是HTML，Javascript和CSS资产的集合，可以从符合OAS标准的API动态生成漂亮的文档。
- **Swagger Codegen：**允许根据OpenAPI规范自动生成API客户端库（SDK生成），服务器存根和文档。
- **Swagger Parser：**用于解析来自Java的OpenAPI定义的独立库
- **Swagger Core：**与Java相关的库，用于创建，使用和使用OpenAPI定义
- **Swagger Inspector（免费）：** API测试工具，可让您验证您的API并从现有API生成OpenAPI定义
- **SwaggerHub（免费和商业）：** API设计和文档，为使用OpenAPI的团队构建。





## 04、API文档：Swagger的使用



### 1）快速入门

SpringBoot已经集成了Swagger，使用简单注解即可生成swagger的API文档。

#### 引入依赖

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.8.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.8.0</version>
</dependency>
```

#### 编写配置

```java
package com.leyou.user.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .host("localhost:8086")
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.leyou.user.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("乐优商城用户中心")
                .description("乐优商城用户中心接口文档")
                .version("1.0")
                .build();
    }
}
```

#### 启动测试

重启服务，访问：http://localhost:8086/swagger-ui.html

就能看到swagger-ui为我们提供的API页面了：

![1554549137117](乐优电商-12-JWT-RSA微服务鉴权.assets/1554549137117.png)

可以看到我们编写的4个接口，任意点击一个，即可看到接口的详细信息：

![1554554238765](乐优电商-12-JWT-RSA微服务鉴权.assets/1554554238765.png)

可以看到详细的接口声明，包括：

- 请求方式：
- 请求路径
- 请求参数
- 响应等信息

点击右上角的`try it out!`还可以测试接口：

![1554554464170](乐优电商-12-JWT-RSA微服务鉴权.assets/1554554464170.png)

填写参数信息，点击execute，可以发起请求并测试：

![1554554533108](乐优电商-12-JWT-RSA微服务鉴权.assets/1554554533108.png)







### 2）自定义接口说明

刚才的文档说明中，是swagge-ui根据接口自动生成，不够详细。如果有需要，可以通过注解自定义接口声明。常用的注解包括：

```java
/**
 @Api：修饰整个类，描述Controller的作用
 @ApiOperation：描述一个类的一个方法，或者说一个接口
 @ApiParam：单个参数描述
 @ApiModel：用对象来接收参数
 @ApiProperty：用对象接收参数时，描述对象的一个字段
 @ApiResponse：HTTP响应其中1个描述
 @ApiResponses：HTTP响应整体描述
 @ApiIgnore：使用该注解忽略这个API
 @ApiError ：发生错误返回的信息
 @ApiImplicitParam：一个请求参数
 @ApiImplicitParams：多个请求参数
 */
```

示例：

```java
/**
     * 校验数据是否可用
     * @param data
     * @param type
     * @return
     */
@GetMapping("/check/{data}/{type}")
@ApiOperation(value = "校验用户名数据是否可用，如果不存在则可用")
@ApiResponses({
    @ApiResponse(code = 200, message = "校验结果有效，true或false代表可用或不可用"),
    @ApiResponse(code = 400, message = "请求参数有误，比如type不是指定值")
})
public ResponseEntity<Boolean> checkUserData(
    @ApiParam(value = "要校验的数据", example = "lisi") @PathVariable("data") String data,
    @ApiParam(value = "数据类型，1：用户名，2：手机号", example = "1") @PathVariable(value = "type") Integer type) {
    return ResponseEntity.ok(userService.checkData(data, type));
}
```

查看文档：

![1554555057087](乐优电商-12-JWT-RSA微服务鉴权.assets/1554555057087.png)





## 05、授权中心：有状态认证及无状态认证

### 1）什么是有状态认证？

有状态认证，即服务端需要记录每次会话的客户端信息，从而识别客户端身份，根据用户身份进行请求的处理，典型的设计如tomcat中的session。

例如登录：用户登录后，我们把登录者的信息保存在服务端session中，并且给用户一个cookie值，记录对应的session。然后下次请求，用户携带cookie值来，我们就能识别到对应session，从而找到用户的信息。

缺点是什么？

- 服务端保存大量数据，增加服务端压力
- 服务端保存用户状态，无法进行水平扩展（无法很好的集群）
- 客户端请求依赖服务端，多次请求必须访问同一台服务器



### 2）什么是无状态认证

微服务集群中的每个服务，对外提供的都是Rest风格的接口。而Rest风格的一个最重要的规范就是：服务的无状态性，即：

- 服务端不保存任何客户端请求者信息
- **客户端的每次请求必须具备自描述信息，通过这些信息识别客户端身份**

带来的好处是什么呢？

- 客户端请求不依赖服务端的信息，任何多次请求不需要必须访问到同一台服务
- 服务端的集群和状态对客户端透明
- 服务端可以任意的迁移和伸缩
- 减小服务端存储压力

### 3）有状态认证解决方案

#### 集中式（单机版）认证流程

![1577356732084](乐优电商-12-JWT-RSA微服务鉴权.assets/1577356732084.png) 

#### 分布式认证流程（单点登录  CAS）

<img src="乐优电商-12-JWT-RSA微服务鉴权.assets/1602316346035.png" alt="1602316346035" style="zoom:67%;" /> 



### 4）无状态分布式认证解决方案(*)

无状态认证的流程：

- 当客户端第一次请求服务时，服务端对用户进行信息认证（登录）
- 认证通过，将**用户信息进行加密形成token**，返回给客户端（保存到Cookie中），作为登录凭证
- 客户端以后每次请求，客户端都携带Cookie中的认证信息的token
- 服务端对token进行解密，判断是否有效（身份合法性校验）。
- 服务端从token中解析出登录用户信息（用户ID，用户角色等）

流程图：

 ![1527300483893](乐优电商-12-JWT-RSA微服务鉴权.assets/1527300483893.png) 



整个无状态认证过程中，最关键的点是什么？

**token的安全性！！！**

因为token是识别客户端身份的唯一标示，如果加密不够严密，被人伪造那就完蛋了。

采用何种方式加密才是安全可靠呢？

我们将采用`JWT + RSA非对称加密`





## 06、授权中心：JWT+RSA鉴权

### 1）JWT简介

JWT，全称是Json Web Token， 是JSON风格轻量级的授权和身份认证规范，可实现无状态、分布式的Web应用授权；官网：https://jwt.io  （JWT，生成Token加密字符串的一个标准或格式！）

![1577360704059](乐优电商-12-JWT-RSA微服务鉴权.assets/1577360704059.png) 



GitHub上jwt的java客户端：https://github.com/jwtk/jjwt



### 2）JWT数据格式

JWT包含三部分数据：

- Header：头部，通常头部有两部分信息：

  - 声明类型(typ)，这里是JWT
  - 签名算法（alg），自定义

  我们会对头部进行base64编码，得到第一部分数据

  如图所示：头部是不具备安全性的。

  ![image-20200326105519850](乐优电商-12-JWT-RSA微服务鉴权.assets/image-20200326105519850.png)

- Payload：载荷，就是有效数据，一般包含下面信息：

  - 用户身份信息（注意，这里因为采用base64加密，可解密，因此不要存放敏感信息）
  - tokenID((JTI/id)：当前这个JWT的唯一标示
  - 注册声明：如token的签发时间，过期时间(exp)，签发人等

  这部分也会采用base64加密，得到第二部分数据

  如图所示：载荷也不具备安全性。

  ![image-20200326110145687](乐优电商-12-JWT-RSA微服务鉴权.assets/image-20200326110145687.png)

- Signature：签名，是整个数据的认证信息。一般根据前两步的数据，再加上服务的的密钥（secret）（不要泄漏，最好周期性更换），通过加密算法生成。用于验证整个数据完整和可靠性

如果所示：签名中决定整个token是否安全的关键在盐。

![image-20200326110424475](乐优电商-12-JWT-RSA微服务鉴权.assets/image-20200326110424475.png)

生成的数据格式：

![1527322512370](乐优电商-12-JWT-RSA微服务鉴权.assets/1527322512370.png)

可以看到分为3段，每段就是上面的一部分数据





### 3）登录和鉴权流程：JWT+HS256算法

登录和鉴权流程：

![1](乐优电商-12-JWT-RSA微服务鉴权.assets/1-1599622478793.png)



- 授权流程：
  - 1、用户请求登录，携带用户名密码到授权中心
  - 2、授权中心携带用户名密码，到用户中心查询用户
  - 3、查询如果正确，生成JWT凭证
  - 4、返回JWT给用户



- 鉴权流程：
  - 1、用户请求某微服务功能，携带JWT
  - 2、**微服务将jwt交给授权中心校验**
  - 3、**授权中心返回校验结果到微服务**
  - 4、微服务判断校验结果，成功或失败
  - 5、失败则直接返回401
  - 6、成功则处理业务并返回



因为JWT签发的token中已经包含了用户的身份信息，并且每次请求都会携带，这样服务的就无需保存用户信息，甚至无需去数据库查询，完全符合了Rest的无状态规范。



但是，上面的流程存在一个问题是，因为JWT校验所需的盐统一存放在授权中心，所以每个微服务每次都需要把JWT交给授权中心进行校验，这样效率就降低了！能不能改善呢？可以的，这时可以使用RSA非对称加密来完善以上流程。



### 4）RSA非对称加密

加密技术是对信息进行编码和解码的技术，编码是把原来可读信息（又称明文）译成代码形式（又称密文），其逆过程就是解码（解密），加密技术的要点是加密算法，加密算法可以分为两大类，三小类：  

- **可逆加密**

  - 对称加密，如AES

    - 基本原理：将明文分成N个组，然后使用密钥对各个组进行加密，形成各自的密文，最后把所有的分组密文进行合并，形成最终的密文。
    - 优势：算法公开、计算量小、加密速度快、加密效率高
    - 缺陷：双方都使用同样密钥，安全性得不到保证 

    

  - 非对称加密，如RSA

    - 基本原理：同时生成两把密钥：私钥和公钥，**私钥隐秘保存，公钥可以下发给信任客户端**
    - 私钥加密，持有私钥或公钥才可以解密
    - 公钥加密，持有私钥才可解密
    - 优点：安全，难以破解
    - 缺点：算法比较耗时

  

  

- **不可逆加密**，如MD5，SHA，HS加密

  - 基本原理：加密过程中不需要使用[密钥](https://baike.baidu.com/item/%E5%AF%86%E9%92%A5)，输入明文后由系统直接经过加密算法处理成密文，这种加密后的数据是无法被解密的，无法根据密文推算出明文。



RSA算法历史：

1977年，三位数学家Rivest、Shamir 和 Adleman 设计了一种算法，可以实现非对称加密。这种算法用他们三个人的名字缩写：RSA



### 5）登录和鉴权流程：JWT+RSA

有了非对称加密，我们就可以改变签名和验签的方式了：

- 生成RSA密钥对，私钥存放在授权中心，公钥下发给微服务
- 在授权中心利用私钥对JWT签名
- 在微服务利用公钥验证签名有效性

因为非对称加密的特性，不用担心公钥泄漏问题，因为公钥是无法伪造签名的，但要**确保私钥的安全和隐秘**。

非对称加密后的授权和鉴权流程：



![2](乐优电商-12-JWT-RSA微服务鉴权.assets/2.png)



用户只需要与微服务交互，不用访问授权中心，效率大大提高！

接下来让我们撸起袖子，开始写代码吧！





## 07、授权中心：准备工作-RSA工具

### 1）RSA工具类

RSA工具类负责对RSA密钥的创建、读取功能：

```java
package com.leyou.common.auth.utils;

import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.security.*;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.Base64;

/**
 * @author 黑马程序员
 */
public class RsaUtils {

    private static final int DEFAULT_KEY_SIZE = 2048;
    /**
     * 从文件中读取公钥
     *
     * @param filename 公钥保存路径，相对于classpath
     * @return 公钥对象
     * @throws Exception
     */
    public static PublicKey getPublicKey(String filename) throws Exception {
        byte[] bytes = readFile(filename);
        return getPublicKey(bytes);
    }

    /**
     * 从文件中读取密钥
     *
     * @param filename 私钥保存路径，相对于classpath
     * @return 私钥对象
     * @throws Exception
     */
    public static PrivateKey getPrivateKey(String filename) throws Exception {
        byte[] bytes = readFile(filename);
        return getPrivateKey(bytes);
    }

    /**
     * 获取公钥
     *
     * @param bytes 公钥的字节形式
     * @return
     * @throws Exception
     */
    private static PublicKey getPublicKey(byte[] bytes) throws Exception {
        bytes = Base64.getDecoder().decode(bytes);
        X509EncodedKeySpec spec = new X509EncodedKeySpec(bytes);
        KeyFactory factory = KeyFactory.getInstance("RSA");
        return factory.generatePublic(spec);
    }

    /**
     * 获取密钥
     *
     * @param bytes 私钥的字节形式
     * @return
     * @throws Exception
     */
    private static PrivateKey getPrivateKey(byte[] bytes) throws NoSuchAlgorithmException, InvalidKeySpecException {
        bytes = Base64.getDecoder().decode(bytes);
        PKCS8EncodedKeySpec spec = new PKCS8EncodedKeySpec(bytes);
        KeyFactory factory = KeyFactory.getInstance("RSA");
        return factory.generatePrivate(spec);
    }

    /**
     * 根据密文，生存rsa公钥和私钥,并写入指定文件
     *
     * @param publicKeyFilename  公钥文件路径
     * @param privateKeyFilename 私钥文件路径
     * @param secret             生成密钥的密文
     */
    public static void generateKey(String publicKeyFilename, String privateKeyFilename, String secret, int keySize) throws Exception {
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
        SecureRandom secureRandom = new SecureRandom(secret.getBytes());
        keyPairGenerator.initialize(Math.max(keySize, DEFAULT_KEY_SIZE), secureRandom);
        KeyPair keyPair = keyPairGenerator.genKeyPair();
        // 获取公钥并写出
        byte[] publicKeyBytes = keyPair.getPublic().getEncoded();
        publicKeyBytes = Base64.getEncoder().encode(publicKeyBytes);
        writeFile(publicKeyFilename, publicKeyBytes);
        // 获取私钥并写出
        byte[] privateKeyBytes = keyPair.getPrivate().getEncoded();
        privateKeyBytes = Base64.getEncoder().encode(privateKeyBytes);
        writeFile(privateKeyFilename, privateKeyBytes);
    }

    private static byte[] readFile(String fileName) throws Exception {
        return Files.readAllBytes(new File(fileName).toPath());
    }

    private static void writeFile(String destPath, byte[] bytes) throws IOException {
        File dest = new File(destPath);
        if (!dest.exists()) {
            dest.createNewFile();
        }
        Files.write(dest.toPath(), bytes);
    }
}

```



### 2）测试RSA



我们在`ly-common`中导入test依赖并编写测试类：

> pom.xml

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

```

> 准备存放公钥和私钥的空文件夹

![image-20200326113731457](乐优电商-12-JWT-RSA微服务鉴权.assets/image-20200326113731457.png)

> 测试类

```java
package com.leyou;

import com.leyou.common.auth.utils.RsaUtils;
import org.junit.Test;

import java.security.PrivateKey;
import java.security.PublicKey;

/**
 * 测试Rsa工具类
 */
public class RsaTest {

    public String publicKeyFile = "D:\\leyou_projects\\javaee143\\software\\rsa\\rsa-key.pub";
    public String privateKeyFile = "D:\\leyou_projects\\javaee143\\software\\rsa\\rsa-key";

    /**
     * 生成公私钥密钥对
     */
    @Test
    public void genKey() throws Exception {
        RsaUtils.generateKey(publicKeyFile,privateKeyFile,"itcast",1);
    }

    /**
     * 读取公钥
     */
    @Test
    public void getPublicKey() throws Exception {
        PublicKey publicKey = RsaUtils.getPublicKey(publicKeyFile);
        System.out.println(publicKey);
    }

    /**
     * 读取私钥
     */
    @Test
    public void getPrivateKey() throws Exception {
        PrivateKey privateKey = RsaUtils.getPrivateKey(privateKeyFile);
        System.out.println(privateKey);
    }
}


```

我们运行`generateKey()`，然后到指定的目录中查看：

![1599633060602](乐优电商-12-JWT-RSA微服务鉴权.assets/1599633060602.png)

公钥和私钥已经生成了，接下来我们获取公钥和私钥，我们运行 `getKey()` 方法，查看控制台：

&nbsp;![1599633164490](乐优电商-12-JWT-RSA微服务鉴权.assets/1599633164490.png)







## 08、授权中心：准备工作-JWT工具类

### 1）依赖导入

我们需要先在`ly-common`中引入JWT依赖：

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.10.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.10.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.10.5</version>
    <scope>runtime</scope>
</dependency>
```

### 2）载荷对象：Payload

JWT中，会保存载荷数据，我们计划存储3部分：

- id：jwt的id
- 用户信息：用户数据，不确定，可以是任意类型
- 过期时间：Date

为了方便后期获取，我们定义一个类来封装：

```java
package com.leyou.common.auth.pojo;

import lombok.Data;

import java.util.Date;

/**
 * @author 黑马程序员
 */
@Data
public class Payload<T> {
    private String id;
    private T info;
    private Date expiration;
}
```



### 3）封装用户信息类：UserInfo

这里我们假设用户信息包含3部分：

- id：用户id
- username：用户名
- role：角色

载荷:UserInfo

```java
package com.leyou.common.auth.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 * 存放JWt的载荷中的登录用户信息
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class UserInfo {
    private Long id;//用户ID
    private String username;//用户名称
    private String role;//用户角色
}

```



### 4）JWT工具

```java
package com.leyou.common.auth.utils;

import com.leyou.common.utils.JsonUtils;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jws;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.joda.time.DateTime;

import java.security.PrivateKey;
import java.security.PublicKey;
import java.util.Base64;
import java.util.UUID;

/**
 * @author: 黑马程序员
 **/
public class JwtUtils {

    private static final String JWT_PAYLOAD_USER_KEY = "user";

    /**
     * 私钥加密token
     *
     * @param info   载荷中的数据
     * @param privateKey 私钥
     * @param expire     过期时间，单位分钟
     * @return JWT
     */
    public static String generateTokenExpireInMinutes(Object info, PrivateKey privateKey, int expire) {
        return Jwts.builder()
                //claim: 往Jwt的载荷存入数据
                .claim(JWT_PAYLOAD_USER_KEY, JsonUtils.toString(info))
                //往Jwt的载荷存入数据，设置固定id的key
                .setId(createJTI())
                //往Jwt的载荷存入数据，设置固定exp的key
                .setExpiration(DateTime.now().plusMinutes(expire).toDate())
                //设置token的签名
                .signWith(privateKey, SignatureAlgorithm.RS256)
                .compact();
    }

    /**
     * 私钥加密token
     *
     * @param info   载荷中的数据
     * @param privateKey 私钥
     * @param expire     过期时间，单位秒
     * @return JWT
     */
    public static String generateTokenExpireInSeconds(Object info, PrivateKey privateKey, int expire) {
        return Jwts.builder()
                .claim(JWT_PAYLOAD_USER_KEY, JsonUtils.toString(info))
                .setId(createJTI())
                .setExpiration(DateTime.now().plusSeconds(expire).toDate())
                .signWith(privateKey, SignatureAlgorithm.RS256)
                .compact();
    }

    /**
     * 公钥解析token
     *
     * @param token     用户请求中的token
     * @param publicKey 公钥
     * @return Jws<Claims>
     */
    private static Jws<Claims> parserToken(String token, PublicKey publicKey) {
        return Jwts.parser().setSigningKey(publicKey).parseClaimsJws(token);
    }

    private static String createJTI() {
        return new String(Base64.getEncoder().encode(UUID.randomUUID().toString().getBytes()));
    }

    /**
     * 获取token中的用户信息
     *
     * @param token     用户请求中的令牌
     * @param publicKey 公钥
     * @return 用户信息
     */
    public static <T> Payload<T> getInfoFromToken(String token, PublicKey publicKey, Class<T> userType) {
        Jws<Claims> claimsJws = parserToken(token, publicKey);
        Claims body = claimsJws.getBody();
        Payload<T> claims = new Payload<>();
        claims.setId(body.getId());
        claims.setInfo(JsonUtils.toBean(body.get(JWT_PAYLOAD_USER_KEY).toString(), userType));
        claims.setExpiration(body.getExpiration());
        return claims;
    }

    /**
     * 获取token中的载荷信息
     *
     * @param token     用户请求中的令牌
     * @param publicKey 公钥
     * @return 用户信息
     */
    public static <T> Payload<T> getInfoFromToken(String token, PublicKey publicKey) {
        Jws<Claims> claimsJws = parserToken(token, publicKey);
        Claims body = claimsJws.getBody();
        Payload<T> claims = new Payload<>();
        claims.setId(body.getId());
        claims.setExpiration(body.getExpiration());
        return claims;
    }
}
```



### 5）测试

在RsaUtilsTest加入测试方法

> 生成token

```java
package com.leyou;

import com.leyou.common.auth.utils.JwtUtils;
import com.leyou.common.auth.utils.Payload;
import com.leyou.common.auth.utils.RsaUtils;
import com.leyou.common.auth.utils.UserInfo;
import org.junit.Test;

import java.security.PrivateKey;
import java.security.PublicKey;

/**
 *
 */
public class JwtTest {
    public String publicKey = "D:\\leyou_projects\\javaee145\\software\\rsa\\rsa-key.pub";
    public String privateKey = "D:\\leyou_projects\\javaee145\\software\\rsa\\rsa-key";

    /**
     * 生成token
     * @throws Exception
     */
    @Test
    public void testGenToken() throws Exception {
        /**
         * 参数一：用户对象
         * 参数二：私钥对象
         * 参数三：过期时间
         */
        UserInfo userInfo = new UserInfo(1L,"小苍","admin");
        PrivateKey privateKey = RsaUtils.getPrivateKey(this.privateKey);
        String token = JwtUtils.generateTokenExpireInMinutes(userInfo,privateKey,30);
        System.out.println(token);
    }

    /**
     * 校验合法性并取出用户信息
     */
    @Test
    public void testValid() throws Exception {
        String token = "eyJhbGciOiJSUzI1NiJ9.eyJ1c2VyIjoie1wiaWRcIjoxLFwidXNlcm5hbWVcIjpcIuWwj-iLjVwiLFwicm9sZVwiOlwiYWRtaW5cIn0iLCJqdGkiOiJaakkwTVRFM01HRXROMkpqTkMwME5UWXhMVGt3TWpndE56UTBNVE0zTmpSaE16SmgiLCJleHAiOjE2MTAxODY1OTl9.DJAD3Y6hgz0CwoVkOizcwyYjesEIB2M9-gSkWGY3u96bnvOQydI9-wpS-OS7r4Vi5FdDJriFDA0Z2a2trAkr46lhTxGKJgEruDJzzPKHiKLT6FGE9WerJg4InHm3_wIYIqcQHgc8oQ656apdKCxQWu2Qwf8WcYkXXNmI4I-HKFCE1sfv1gk8IOSVkjMYK8bdXVOF6L7XT5URc59z-aAmGn0SfbJWJFamX2881VrkK-dL1aPWCXk1EUKJEjlpKajUwQMlDbkLAODf4mY2WufWUpyGtU4lD_swF0YUGbBgqDqrldphU_QdCnlFg7Jc4JDWYLVmxkMTcUkFSrOj8cLk3w";
        PublicKey publicKey = RsaUtils.getPublicKey(this.publicKey);
        Payload<UserInfo> payload = null;

        try {
            payload = JwtUtils.getInfoFromToken(token, publicKey, UserInfo.class);
            System.out.println("验证成功");

            UserInfo info = payload.getInfo();
            System.out.println(info);
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("验证失败");
        }

    }


}


```

控制台：

![1577413012770](乐优电商-12-JWT-RSA微服务鉴权.assets/1577413012770.png) 



> 解析token

```java
 /**
     * 解析token，获取用户信息
     */
    @Test
    public void testParseToken() throws Exception {
        String token = "eyJhbGciOiJSUzI1NiJ9.eyJ1c2VyIjoAie1wiaWRcIoxLFwibmFtZVwiOlwiamFja1wiLFwicm9sZVwiOlwiYWRtaW5cIn0iLCJqdGkiOiJNREF5WXpFNFl6QXRabUZrTmkwME56UTFMVGczTjJZdE1tSXpZamxoWkdKbVlXVTMiLCJleHAiOjE2MDIzODkzNTN9.eEdnnXBMc0Dhd3tDoYcnwI2RNQa8M6DZ6TKwzRgu9efGobNHFqVaA5f1gbWn1Pfmw6v9tSJL5PdFkoxh3YaRFBnrnkww0G2XlQ3mjwrFlOF4rkInzUj8WQQOCu_Ov6avnIOjy1JSe7K0oo1rXxUWENA5lLFEoNdWbPnUE-EinTGH7CRvHvni-MvzfTKCwN9T-CJ0FpaKN2zpSoOZ-E2McLA-Opj5bw7QkDx8ewTih8ayXCBdHKSq5BR-KRZMihgNG-ApOrqY-gqIRzCK6Bh5Fv27uRPzciL7454P5SIjQTSfWor3Vy_asaHQtVpJHCrK6UbYu9jKdZ7z3YwFQahaiQ";
        //获取公钥
        PublicKey publicKey = RsaUtils.getPublicKey(pubPathKey);

        Payload<UserInfo> infoFromToken = null;
        try {
            infoFromToken = JwtUtils.getInfoFromToken(token, publicKey, UserInfo.class);

            System.out.println("认证成功");
            UserInfo info = infoFromToken.getInfo();
            System.out.println(info);
        }catch (Exception e){
            System.out.println("认证失败："+e.getMessage());
        }

    }

```

控制台：

![1577413491326](乐优电商-12-JWT-RSA微服务鉴权.assets/1577413491326.png) 





## 09、授权中心：搭建授权微服务

授权中心的主要职责：

- 用户登录鉴权：
  - 接收用户的登录请求，
  - 通过用户中心的接口校验用户名密码
  - 使用私钥生成JWT并返回
- 用户登录状态校验
  - 判断用户是否登录，其实就是token的校验
- 用户登出
  - 用户选择退出登录后，要让token失效
- 用户登录状态刷新
  - 用户登录一段时间后，JWT可能过期，需要刷新有效期

接下来，我们逐一完成上述功能



### 1）提供用户微服务对外的feign接口模块

因为授权中心微服务需要调用用户微服务的方法，所以需要提供用户微服务Feign接口模块

创建ly-client-user模块

&nbsp;<img src="乐优电商-12-JWT-RSA微服务鉴权.assets/1599635150210.png" alt="1599635150210" style="zoom: 67%;" />



导入jar包

```xml
<dependencies>
    <dependency>
        <groupId>com.leyou</groupId>
        <artifactId>ly-pojo-user</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

### 2）创建项目

![1577413691589](乐优电商-12-JWT-RSA微服务鉴权.assets/1577413691589.png) 

### 3）导入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>leyou</artifactId>
        <groupId>com.leyou</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>ly-auth</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>2.1.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>com.leyou</groupId>
            <artifactId>ly-client-user</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>com.leyou</groupId>
            <artifactId>ly-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```



### 3）启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class LyAuthApplication {
    public static void main(String[] args) {
        SpringApplication.run(LyAuthApplication.class, args);
    }
}
```



### 4）配置文件`application.yml`

```yml
server:
  port: 8087
spring:
  application:
    name: auth-service
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
```



### 5）在网关加入路由规则

```yml
spring:
  cloud:
    gateway:
      routes:
        # 其他省略
        - id: auth-service
          uri: lb://auth-service
          predicates:
            - Path=/api/auth/**
          filters:
            - StripPrefix=2
```



项目结构：

![1577414077459](乐优电商-12-JWT-RSA微服务鉴权.assets/1577414077459.png) 







## 10、授权中心：用户微服务提供登录Feign接口

### 1）分析

接下来，我们需要在`ly-auth`编写一个接口，对外提供登录授权服务。

登录授权流程我们上面已经分析过，基本流程如下：：

- 1、用户请求登录，携带用户名密码到授权中心
- 2、授权中心携带用户名密码，到用户中心查询用户
- 3、查询如果正确，生成JWT凭证，查询错误则返回400,
- 4、返回JWT给用户



### 2）ly-user编写Controller

> 用户中心必须对外提供查询接口，方便ly-auth做用户名密码校验。

在ly-user中提供查询用户信息的处理器

```java
 package com.leyou.auth.controller;

import com.leyou.auth.service.AuthService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletResponse;

    /**
     * 根据用户名和密码查询用户
     */
    @GetMapping("/query")
    public ResponseEntity<User> findUserByUserNameAndPwd(
            @RequestParam("username") String username,
            @RequestParam("password") String password
    ){
        User loginUser = userService.findUserByUserNameAndPwd(username,password);
        return ResponseEntity.ok(loginUser);
    }

```

### 3）ly-user编写Service

```java
 public User findUserByUserNameAndPwd(String username, String password) {
        //1.判断用户名是否存在
        User user = new User();
        user.setUsername(username);
        QueryWrapper<User> queryWrapper = Wrappers.query(user);
        User loginUser = userMapper.selectOne(queryWrapper);

        if(loginUser==null){
            throw new LyException(ExceptionEnum.INVALID_USERNAME_PASSWORD);
        }

        //2.用户存在，判断密码是否匹配
        if(!passwordEncoder.matches(password,loginUser.getPassword())){
            throw new LyException(ExceptionEnum.INVALID_USERNAME_PASSWORD);
        }

        //3.密码匹配，返回User对象
        return loginUser;
    }
```

### 4）ly-client-user提供feign接口

```java
package com.leyou.user.client;

import com.leyou.user.pojo.User;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

/**
 * 用户接口
 */
@FeignClient("user-service")
public interface UserClient {

    /**
     * 根据用户名和密码查询用户
     */
    @GetMapping("/query")
    public User findUserByUserNameAndPwd(
            @RequestParam("username") String username,
            @RequestParam("password") String password
    );
}



```



## 11、授权中心：授权微服务登录认证方法

### 1） 提供授权相关配置文件

```yml
ly:
  jwt:
    pubKeyPath: D:\ly_guangzhou126\software\rsa_key\jwt_key.pub # 公钥地址
    priKeyPath: D:\ly_guangzhou126\software\rsa_key\jwt_key # 私钥地址
    cookie:
      expire: 30 #过期时间设置 单位分钟
      cookieName: LY_TOKEN # cookie名称
      cookieDomain: leyou.com # cookie的域
```

### 2） 提供解析配置文件的配置类

通过分析spring中bean对象的完整生命周期，来总结一个解析配置文件的方案。

Spring的Bean对象完整生命周期如图：

![1554617122160](乐优电商-12-JWT-RSA微服务鉴权.assets/1554617122160.png)

Spring Bean在Spring Bean Factory Container中完成其整个生命周期：以下是完成其生命周期所需的各种内容：

1. Spring容器从XML文件或@Configuration中bean的定义中实例化bean(IOC)。
2. Spring依据配置中指定的属性，为bean填充所有属性(DI)。
3. 如果bean实现BeanNameAware接口，spring调用setBeanName()方法，并传递bean的id。
4. 如果bean实现BeanFactoryAware接口，spring将调用setBeanFactory()方法，并把自己作为参数。
5. 如果bean实现ApplicationContextAware接口，spring将调用setApplicationContext()方法，并把ApplicationContext实例作为参数。
6. 如果存在与bean关联的任何BeanPostProcessors（后处理器），则调用preProcessBeforeInitialization()方法。比如Autowired等依赖注入功能是在此时完成。
7. 如果Bean实现了InitializingBean接口，则调用bean的afterPropertiesSet()方法。
8. 如果为bean指定了init-method，那么将调用bean的init方法。
9. 最后，如果存在与bean关联的任何BeanPostProcessors，则将调用postProcessAfterInitialization（）方法。
10. 如果关闭容器，则调用destroy方法



![1606797065448](乐优电商-12-JWT-RSA微服务鉴权.assets/1606797065448.png)

> **bean标签配置细节**
>
> ![1563330494898](乐优电商-12-JWT-RSA微服务鉴权.assets/1563330494898.png)

最终解析配置文件的配置类为：

```java
package com.leyou.auth.config;

import com.leyou.common.auth.utils.RsaUtils;
import lombok.Data;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import java.security.PrivateKey;
import java.security.PublicKey;

/**
 *
 */
@Data
@Component("jwt")
@ConfigurationProperties(prefix = "ly.jwt")
public class JwtProperties{
    private String pubKeyPath;
    private String priKeyPath;

    private PublicKey publicKey;//公钥对象
    private PrivateKey privateKey;//私钥对象

    // 因为这个是spring对象，以下方式无法获取到正确数据
  /*  public JwtProperties() throws Exception {
        publicKey = RsaUtils.getPublicKey(pubKeyPath);
        privateKey = RsaUtils.getPrivateKey(priKeyPath);
    }*/

   @PostConstruct // 该注解把该方法标记为init方法
    public void initMethod() throws Exception {
        publicKey = RsaUtils.getPublicKey(pubKeyPath);
        privateKey = RsaUtils.getPrivateKey(priKeyPath);
    }

    //接收cookie的内容
    //属性名是和yml的cookie一致
    private CookiePojo cookie = new CookiePojo();



    @Data
    public class CookiePojo{
         //属性名是和yml的cookie的内容一致
        private Integer expire;
        private String cookieName;
        private String cookieDomain;
    }


}


```



### 3） 提供认证的处理器

```java
package com.leyou.auth.controller;

import com.leyou.auth.service.AuthService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 *
 */
@RestController
public class AuthController {

    @Autowired
    private AuthService authService;

    /**
     * 登录
     */
    @PostMapping("/login")
    public ResponseEntity<Void> login(@RequestParam("username") String username,
                                      @RequestParam("password") String password,
                                      HttpServletRequest request,
                                      HttpServletResponse response){
        authService.login(username,password,request,response);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
}

```

### 4） 提供认证的service

```java
package com.leyou.auth.service;

import com.leyou.auth.config.JwtProperties;
import com.leyou.common.auth.utils.JwtUtils;
import com.leyou.common.auth.utils.UserInfo;
import com.leyou.common.utils.CookieUtils;
import com.leyou.user.client.UserClient;
import com.leyou.user.pojo.User;
import net.bytebuddy.asm.Advice;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 *
 */
@Service
public class AuthService {
    @Autowired
    private UserClient userClient;
    @Autowired
    private JwtProperties jwtProps;

    public void login(String username, String password,
                      HttpServletRequest request, HttpServletResponse response) {

        //1.验证用户名和密码
        User loginUser = userClient.findUserByUserNameAndPwd(username,password);

        //2.生成token，并写回token到Cookie中
        //2.1 创建UserInfo对象
        UserInfo userInfo = new UserInfo(loginUser.getId(),loginUser.getUsername(),"admin");
        //2.2 生成token，并写回token到Cookie中
        createTokenAndWriteCookie(userInfo,response);
    }

    /**
     * 生成token，并写回token到Cookie中
     * @param userInfo
     * @param response
     */
    public void createTokenAndWriteCookie(UserInfo userInfo, HttpServletResponse response) {
        String token = JwtUtils.generateTokenExpireInMinutes(
                userInfo,
                jwtProps.getPrivateKey(),
                jwtProps.getCookie().getExpire());
        //写出token到Cookie中
        CookieUtils.newCookieBuilder()
                .response(response)
                .name(jwtProps.getCookie().getCookieName())
                .value(token)
                .domain(jwtProps.getCookie().getCookieDomain())
                .build();
    }
}



```

### 5） 启动项目测试

查看浏览器是否保存了Cookie

&nbsp;![1599641413736](乐优电商-12-JWT-RSA微服务鉴权.assets/1599641413736.png)





## 12、授权中心：首页判断登录状态

虽然cookie已经成功写入，但是我们首页的顶部，登录状态依然没能判断出用户信息：

![1577439747531](乐优电商-12-JWT-RSA微服务鉴权.assets/1577439747531.png) 

我们思考一下，应该如何判断用户是否登录呢？

### 1）步骤分析

传统登录校验的步骤：

- 1）用户请求到达服务端，会自动携带cookie
- 2）cookie中包含sessionId，tomcat根据sessionId获取session
- 3）从session中读取用户信息，判断是否存在
- 4）存在，证明已经登录；不存在，证明登录超时或未登录

我们现在使用的是无状态登录，不存在session，而是把用户身份写入了token，是否需要发请求到服务端进行校验呢？

肯定需要的，因为token需要通过公钥解析才知道是否有效。

分析一下步骤：

- 1）页面向后台发起请求，携带cookie
- 2）后台获取cookie中的LY_TOKEN
- 3）校验token是否有效
  - 无效：登录失效
  - 有效：解析出里面的用户信息，返回到页面

接下来，我们就分步实现上述功能。

### 2）页面JS代码

首先是页面发起请求，校验cookie。

页面的顶部已经被我们封装为一个独立的Vue组件，在`/js/pages/shortcut.js`中

![1577443772586](乐优电商-12-JWT-RSA微服务鉴权.assets/1577443772586.png) 

打开js，发现里面已经定义好了Vue组件，并且在created函数中，查询用户信息：

![1577443831722](乐优电商-12-JWT-RSA微服务鉴权.assets/1577443831722.png) 

查看网络控制台，发现发起了请求：

![1527553282028](乐优电商-12-JWT-RSA微服务鉴权.assets/1527553282028.png) 

因为token在cookie中，因此本次请求肯定会携带token信息在头中。



### 3）校验用户登录状态

我们在`ly-auth`中定义用户的校验接口，通过cookie获取token，然后校验通过返回用户信息。

- 请求方式：GET
- 请求路径：/verify
- 请求参数：无，不过我们需要从cookie中获取token信息
- 返回结果：UserInfo，校验成功返回用户信息；校验失败，则返回401

> controller代码：

```java
    /**
     * 验证用户是否登录，同时获取登录用户信息
     */
    @GetMapping("/verify")
    public ResponseEntity<UserInfo> verify(HttpServletRequest request,
                                           HttpServletResponse response){
        UserInfo userInfo = authService.verify(request,response);
        return ResponseEntity.ok(userInfo);
    }

```

> service代码：

```java
  public UserInfo verify(HttpServletRequest request, HttpServletResponse response) {
        //1.从Cookie获取token
        String token = CookieUtils.getCookieValue(request,jwtProps.getCookie().getCookieName());

        //2.验证token是否合法
        Payload<UserInfo> payload = JwtUtils.getInfoFromToken(token, jwtProps.getPublicKey(), UserInfo.class);

        //3.从token中取出用户信息
        return payload.getInfo();
    }
```

### 4）测试

![1577444090272](乐优电商-12-JWT-RSA微服务鉴权.assets/1577444090272.png) 

页面效果：

![1577444141968](乐优电商-12-JWT-RSA微服务鉴权.assets/1577444141968.png) 







## 13、课程总结



1）区分有状态认证 和 无状态认证（***）

2）JWT + RSA：

​     JWT： 生成token的格式（头部，载荷（用户数据），签名）

​    RSA： 用于生成JWT格式的token的签名，为了提高效率，使用RSA解密

3）搭建授权中心（重点）

​     3.1 登录方法（验证用户名和密码对不对，JwtUtils生成token，写出Cookie给浏览器）

​     3.2 获取登录用户名（从Cookie取出token，验证token，取出token的用户数据）



















