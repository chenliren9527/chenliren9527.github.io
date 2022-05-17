---
title: SpringMVC(01)
tags:
  - 笔记
  - ssm框架
  - Spring
  - SpringMVC
  - 源码分析
  - RequestMapping
  - SpringMVC的参数绑定
  - SpringMVC的自定义类型转换器
  - 常用注解
categories:
  - ssm框架
date: 2020-11-09 21:05:35
---

##  01. 学习目标

**1、介绍SpringMVC框架** 

**2、能够实现SpringMVC的环境搭建** 

**3、能够说出入门案例的执行过程** 

**4、说出入门案例中涉及的SpringMVC常用组件** 

**5、掌握RequestMapping的使用** 

**6、掌握SpringMVC的参数绑定** 

**7、掌握SpringMVC的自定义类型转换器的使用** 

**8、掌握SpringMVC的常用注解** 




## 02. SpringMVC基本概念三层架构与mvc

### 2.1 目标

1. 什么是三层架构？ 
2. 什么是mvc？  

### 2.2 概念

三层架构

![1597711848018](SpringMVC-01.assets/1597711848018.png)



#### 2.3 什么是mvc

m ：model 实体类， 封装数据

v： view 视图(html||jsp)

c: controller  控制器 servlet 

mvc开发模式是用于web层一套开发模式





### 2.4 小结

- mvc分别指什么？

  m: model

  V:view 视图层

  C:在以前是Servetl控制的




## 03. SpringMVC基本概念SpringMVC介绍



### 3.1 目标

​	了解springmvc的基本介绍

### 3.2 概念

​	==SpringMVC 是一种基于 Java 的实现 MVC框架==的请求驱动类型的轻量级 Web 框架， 属于 SpringFrameWork 的后续产品，已经融合在 Spring Web Flow 里面。 

​	Spring 框架提供了构建 Web 应用程序的全功能 MVC 模块。使用 Spring 可插入的 MVC 架构，从而在使用 Spring 进行 WEB 开发时，可以选择使用 Spring的 Spring MVC 框架或集成其他 MVC 开发框架，如 Struts1(现在一般不用)， Struts2 等。

SpringMVC 已经成为目前最主流的 MVC 框架之一， 从 Spring3.0 的发布， 就已全面超越 Struts2，成为最优秀的 MVC 框架。==它通过一套注解，让一个简单的 Java 类成为<font color=red>处理请求</font>的控制器==，而无须实现任何接口。同时它还支持RESTful 编程风格的请求。

- servlet： 称作为侵入式代码, 因为你编写的servlet一定要实现Servlet接口。==弊端： 侵入式让用户代码产生对框架的依赖，这些代码不能在框架外使用，不利于代码的复用。==

- controller : 非侵入式代码， ==编写Controller的时候不需要继承或者实现任何的类或者接口==。 优势： 框架的代码不需要侵入到业务代码中，对框架的依赖性没有那么强，用户不会过多感觉到框架的存在。





## 04. SpringMVC入门:创建项目、添加依赖、配置web.xml



### 4.1 步骤

1.创建web项目，导入依赖（spring-webmvc）

2.配置web.xml，配置（核心）前端控制器

3.配置springMVC.xml核心配置文件

4.编写控制器

5.在webapp目录添加一个目录pages，然后在pages目录下创建一个success.jsp文件



### 4.2 实现

1.创建web项目，导入依赖（spring-webmvc）

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
  <modelVersion>4.0.0</modelVersion>  
  <groupId>com.itheima</groupId>  
  <artifactId>springmvcday01</artifactId>  
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <dependencies>
    <!--springmvc的依赖-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.0.2.RELEASE</version>
    </dependency>
  </dependencies>
</project>

```



2.配置web.xml，配置核心前端控制器（DispathcerServlet）（***）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	version="2.5">

	<!--1. 配置核心控制器(前端控制器)-->
	<servlet>
		<servlet-name>dispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<!--2. 利用核心控制器加载springmvc核心配置文件-->
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc.xml</param-value>
		</init-param>
		<!--servlet对象默认情况是第一次访问的时候创建该类的对象，然后才去加载springmvc的配置文件
			导致的问题：第一个访问的用户的体验糟糕，因为效率低。
			解决方案： 让服务器一启动的时候就创建DispatcherServlet的对象，然后就去加载配置文件.
		-->
		<load-on-startup>1</load-on-startup>

	</servlet>

	
	<servlet-mapping>
		<servlet-name>dispatcherServlet</servlet-name>
		<!--映射路径本质上可以随意写，但是今天建议同学们都使用*.do-->
		<url-pattern>*.do</url-pattern>
	</servlet-mapping>


</web-app>
```



3.配置springMVC.xml



```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--1. 配置视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--配置前缀,查找页面的路径-->
        <property name="prefix" value="/pages/"/>
        <!--配置后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--2. 开启注解扫描-->
    <context:component-scan base-package="com.itheima.controller"/>

    <!--3. 开启注解驱动 ,
           启动springmvc的三大组件与注册类型转换器
    注意： 导入名称空间的时候千万别导入cache的,是需要导入mvc的-->
    <mvc:annotation-driven/>
</beans>
```







4.编写控制器

```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/*
@Controller： 如果一个普通的java类添加了Controller注解，那么该类就是一个控制器.
 */
@Controller
public class HelloController {

    /*
         @RequestMapping用于指定方法的映射路径的。 访问路径
     */
    @RequestMapping("/hello.do")  //注意： /与.do是可以省略不写的
    public String hello(){
        System.out.println("hello方法被调用了...");
        return "success"; //方法的返回值代表了视图的名称，会经过视图解析器去加工。/pages/success.jsp
    }

}

```



5.在webapp目录添加一个目录pages，然后在pages目录下创建一个success.jsp文件

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h1>这是一个成功的页面..</h1>
</body>
</html>

```





### 4.3 小结

- **springmvc入门的步骤**
  - 导入了依赖spring-webmvc
  - 在web.xml文件上配置前端控制器DispatcherServlet，加载springmvc的核心配置文件
  - 编写springmvc的核心配置文件（1，视图解析器 2 。包扫描 3，开启注解驱动）
  - 编写控制器(@Controller , @RequestMapping)





## 05. SpringMVC入门执行流程

重点关注的是：核心控制器做了什么事情？执行流程?

时序图



![1604886282225](SpringMVC-01.assets/1604886282225.png)



## 06. SpringMVC入门源码分析(应付面试)



### 6.1 三大组件及其作用

1）处理器映射器： HandlerMapping （RequestMapingHandlerMapping）

- 找到对应的方法


2）处理器适配器：HandlerAdapter  （RequestMappingHandlerAdapter）

- 执行对应的方法

3）视图解析器：ViewResolver ( InternalResourceViewResolver ) 

- 对方法的返回值进行加工处理，得到一个完整路径

  



### 6.2 源码分析

1. 启动处理器映射器查找对应的方法

   ![1569121406254](SpringMVC-01.assets/1569121406254.png)

![1569121569941](SpringMVC-01.assets/1569121569941.png)

![1569121690256](SpringMVC-01.assets/1569121690256.png)



![1569122683059](SpringMVC-01.assets/1569122683059.png)





### 6.3面试题



1. **SpringMVC执行流程中涉及三大组件**
   - 处理器映射器: 找到对应的方法
   - 处理器的适配器： 执行对应的方法
   - 视图解析器： 对方法的返回值进行加工处理，得到一个完整的视图名称



2.  **请说出SpringMVC的执行流程？**
    - 浏览器发出的请求会交给核心控制器
    - 核心控制器会启动处理器的映射器找到对应的方法
    - 核心控制器会启动处理器的适配器执行对应的方法
    - 核心控制器还会启动视图解析器对方法的返回值进行加工的处理

​     

## 07. @RequestMapping注解使用

### 7.1 目标

学习@RequestMapping注解的使用：作用、注解属性



@RequestMapping注解作用：给方法配置映射路径



@Requestmapping作用范围： 可以写在类上，可以在方法上。

​	



### 7.2 实现



```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/*
学习目标： 学习@RequstMapping的注解

@RequestMapping注解作用：给方法配置映射路径

@Requestmapping作用范围： 可以写在类上，可以在方法上。

访问路径：http://localhost:8080/项目根路径/类上映射路径/方法上的映射路径

疑问： 类上添加@RequestMapping意义何在？ 分模块去管理映射路径
 */
@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/addUser")
    public String addUser(){
        System.out.println("添加用户成功");
        return "success"; //视图的名称
    }
}

```



### 小结

 - @RequestMapping可以作用于哪里？ 分别代表什么作用
   - @RequestMapping用于类上或者方法上。
     	- 类上： 代表着当前的Controller所属的模块
     - 方法上： 代表了方法的映射路径





## 08. 请求参数绑定（一）简单类型作为参数

### 8.1 目标

springMVC如何封装请求参数。





### 8.2  步骤

1.编写UserController，编写save方法

2.在浏览器访问传递参数



### 8.3 实现

1.编写UserController，编写save方法

```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/*
学习目标： 学习@RequstMapping的注解

@RequestMapping注解作用：给方法配置映射路径

@Requestmapping作用范围： 可以写在类上，可以在方法上。

访问路径：http://localhost:8080/项目根路径/类上映射路径/方法上的映射路径

疑问： 类上添加@RequestMapping意义何在？ 分模块去管理映射路径
 */
@Controller
@RequestMapping("/user")
public class UserController {

 
    /*
     springmvc会自动的帮助我们接收参数，但是前提：参数的名称必须与形参的名称一致。
      注意： 使用springmvc的时候我们建议大家都使用包装类型，因为当参数与形参不一致的时候
      默认会给形参注入null，基本数据类型是没法接收null值的。
     */
    @RequestMapping("/save")
    public String save(Integer id,String name){
        System.out.println("接收到的请求参数：id="+id+" name="+name);
        return "success"; //视图的名称
    }


}

```






## 09. 请求参数绑定（二）pojo类型作为参数

### 9.1 目标

会使用对象封装请求数据。



### 9.2 步骤

1.编写user.jsp页面，提供表单传递参数

2.编写User的实体类

3.在UserController添加update方法

4.测试



### 9.3 实现

1.编写user.jsp页面，提供表单传递参数

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <form action="/user/update.do" method="get">
        用户名：<input type="text" name="userName"/><br/>
        年龄：<input type="text" name="age"/><br/>
        地址：<input type="text" name="address"/><br/>
        <input type="submit" value="提交"/>
    </form>
</body>
</html>

```

2.编写User的实体类(如果需要让springmvc自动封装参数，那么要求实体类的属性名要与参数名一致)

```java
package com.itheima.model;

public class User {

    private String userName;

    private Integer age;

    private String address;


    public User() {
    }

    public User(String userName, Integer age, String address) {
        this.userName = userName;
        this.age = age;
        this.address = address;
    }

    /**
     * 获取
     * @return userName
     */
    public String getUserName() {
        return userName;
    }

    /**
     * 设置
     * @param userName
     */
    public void setUserName(String userName) {
        this.userName = userName;
    }

    /**
     * 获取
     * @return age
     */
    public Integer getAge() {
        return age;
    }

    /**
     * 设置
     * @param age
     */
    public void setAge(Integer age) {
        this.age = age;
    }

    /**
     * 获取
     * @return address
     */
    public String getAddress() {
        return address;
    }

    /**
     * 设置
     * @param address
     */
    public void setAddress(String address) {
        this.address = address;
    }

    public String toString() {
        return "User{userName = " + userName + ", age = " + age + ", address = " + address + "}";
    }
}

```



3.在UserController添加update方法

```java
package com.itheima.controller;

import com.itheima.model.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/*
@RequestMapping注解作用：给方法配置映射路径



@Requestmapping作用范围： 可以写在类上，可以在方法上。

注意：
    1. 如果类上配置了路径，那么方法的访问路径： 类上路径+方法的路径
    2. 类上添加@RequestMapping一般起到的作用就是分模块访问。
 */
@Controller
@RequestMapping("/user")  //代表的模块名
public class UserController {



    /*
        javabean封装数据要注意的事项：
            1. 参数名必须与实体对象的属性名一直
            2. 封装参数底层是依赖setter方法，所以一定要提供setter方法
     */
    @RequestMapping("/update")
    public String update(User user) {
        System.out.println("接收到的参数："+user);
        return "success"; //视图的名称
    }




}

```



3.测试

![1564200182753](SpringMVC-01.assets/1564200182753.png)









## 10. 包装Pojo对象（对象里面有对象）



### 10.1 步骤

1.添加用户表单

2.添加了Address的实体类，修改User的实体类的地址类型

3.在UserController添加update方法不需要改变



### 10.2 实现



1.添加用户表单

![1597720920210](SpringMVC-01.assets/1597720920210.png)

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<form action="/user/update.do" method="get">
    用户名：<input type="text" name="userName"/><br/>
    年龄：<input type="text" name="age"/><br/>
    省份：<input type="text" name="address.province"/><br/>
    城市：<input type="text" name="address.city"/><br/>
    <input type="submit" value="提交"/>
</form>
</body>
</html>

```



2.添加了Address的实体类，修改User的实体类的地址类型

```java
package com.itheima.model;

//地址
public class Address {

    private String province; //省份

    private String city; //城市


    public Address() {
    }

    public Address(String province, String city) {
        this.province = province;
        this.city = city;
    }

    /**
     * 获取
     * @return province
     */
    public String getProvince() {
        return province;
    }

    /**
     * 设置
     * @param province
     */
    public void setProvince(String province) {
        this.province = province;
    }

    /**
     * 获取
     * @return city
     */
    public String getCity() {
        return city;
    }

    /**
     * 设置
     * @param city
     */
    public void setCity(String city) {
        this.city = city;
    }

    public String toString() {
        return "Address{province = " + province + ", city = " + city + "}";
    }
}

```



在User添加Address对象

```java
package com.itheima.model;

public class User {

    private String userName;

    private Integer age;

    private Address address;


    public User() {
    }

    public User(String userName, Integer age, Address address) {
        this.userName = userName;
        this.age = age;
        this.address = address;
    }

    /**
     * 获取
     * @return userName
     */
    public String getUserName() {
        return userName;
    }

    /**
     * 设置
     * @param userName
     */
    public void setUserName(String userName) {
        this.userName = userName;
    }

    /**
     * 获取
     * @return age
     */
    public Integer getAge() {
        return age;
    }

    /**
     * 设置
     * @param age
     */
    public void setAge(Integer age) {
        this.age = age;
    }

    /**
     * 获取
     * @return address
     */
    public Address getAddress() {
        return address;
    }

    /**
     * 设置
     * @param address
     */
    public void setAddress(Address address) {
        this.address = address;
    }

    public String toString() {
        return "User{userName = " + userName + ", age = " + age + ", address = " + address + "}";
    }
}

```

3.UserController不需要做出改变

```java
package com.itheima.controller;

import com.itheima.model.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/*
学习目标： 学习@RequstMapping的注解

@RequestMapping注解作用：给方法配置映射路径

@Requestmapping作用范围： 可以写在类上，可以在方法上。

访问路径：http://localhost:8080/项目根路径/类上映射路径/方法上的映射路径

疑问： 类上添加@RequestMapping意义何在？ 分模块去管理映射路径
 */
@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/addUser")
    public String addUser(){
        System.out.println("添加用户成功");
        return "success"; //视图的名称
    }

    /*
     springmvc会自动的帮助我们接收参数，但是前提：参数的名称必须与形参的名称一致。
      注意： 使用springmvc的时候我们建议大家都使用包装类型，因为当参数与形参不一致的时候
      默认会给形参注入null，基本数据类型是没法接收null值的。
     */
    @RequestMapping("/save")
    public String save(Integer id,String name){
        System.out.println("接收到的请求参数：id="+id+" name="+name);
        return "success"; //视图的名称
    }

    /*
        javabean封装数据要注意的事项：
            1. 参数名必须与实体对象的属性名一直
            2. 封装参数底层是依赖setter方法，所以一定要提供setter方法
     */
    @RequestMapping("/update")
    public String update(User user) {
        System.out.println("接收到的参数："+user);
        return "success"; //视图的名称
    }


}

```





3.测试

![1604890266257](SpringMVC-01.assets/1604890266257.png)



## 11. 集合对象



### 11.1 步骤

1.修改实体类,修改地址为List类型

2.修改页面,复制多一个表单出来

3.添加UserController的update2方法



### 11.2 实现

1. 修改实体类,修改地址为List类型

```java
package com.itheima.model;

import java.util.List;

public class User {

    private String userName;

    private Integer age;

    private List<Address> addressList;


    public User() {
    }

    public User(String userName, Integer age, List<Address> addressList) {
        this.userName = userName;
        this.age = age;
        this.addressList = addressList;
    }

    /**
     * 获取
     * @return userName
     */
    public String getUserName() {
        return userName;
    }

    /**
     * 设置
     * @param userName
     */
    public void setUserName(String userName) {
        this.userName = userName;
    }

    /**
     * 获取
     * @return age
     */
    public Integer getAge() {
        return age;
    }

    /**
     * 设置
     * @param age
     */
    public void setAge(Integer age) {
        this.age = age;
    }

    /**
     * 获取
     * @return addressList
     */
    public List<Address> getAddressList() {
        return addressList;
    }

    /**
     * 设置
     * @param addressList
     */
    public void setAddressList(List<Address> addressList) {
        this.addressList = addressList;
    }

    public String toString() {
        return "User{userName = " + userName + ", age = " + age + ", addressList = " + addressList + "}";
    }
}

```

2.修改页面,复制多一个表单出来

![1604890793151](SpringMVC-01.assets/1604890793151.png)

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<form action="/user/update.do" method="get">
    用户名：<input type="text" name="userName"/><br/>
    年龄：<input type="text" name="age"/><br/>
    省份：<input type="text" name="address.province"/><br/>
    城市：<input type="text" name="address.city"/><br/>
    <input type="submit" value="提交"/>
</form>

<h2>=============================================</h2>

<form action="/user/update2.do" method="get">
    用户名：<input type="text" name="userName"/><br/>
    年龄：<input type="text" name="age"/><br/>
    省份1：<input type="text" name="addressList[0].province"/><br/>
    城市1：<input type="text" name="addressList[0].city"/><br/>
    省份1：<input type="text" name="addressList[1].province"/><br/>
    城市1：<input type="text" name="addressList[1].city"/><br/>
    <input type="submit" value="提交"/>
</form>

</body>
</html>

```



3. Controller的方法



```java
package com.itheima.controller;

import com.itheima.model.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/*
@RequestMapping注解作用：给方法配置映射路径



@Requestmapping作用范围： 可以写在类上，可以在方法上。

注意：
    1. 如果类上配置了路径，那么方法的访问路径： 类上路径+方法的路径
    2. 类上添加@RequestMapping一般起到的作用就是分模块访问。
 */
@Controller
@RequestMapping("/user")  //代表的模块名
public class UserController {




    /*
        javabean封装数据要注意的事项：
            1. 参数名必须与实体对象的属性名一直
            2. 封装参数底层是依赖setter方法，所以一定要提供setter方法
     */
    @RequestMapping("/update2")
    public String update2(User user) {
        System.out.println("接收到的参数："+user);
        return "success"; //视图的名称
    }




}

```





3.测试

![1604890911215](SpringMVC-01.assets/1604890911215.png)

![1565669765689](SpringMVC-01.assets/1565669765689.png)



## 12. 求参数绑定（三）请求参数乱码

### 12.1 目标

解决请求参数中文乱码问题。



### 12.2 问题分析

GET

​        Tomcat8或以上版本： 中文不会乱码

​        Tomcat7或以下版本：中文会乱码

​               解决办法：

​                       在Tomcat7的conf/server.xml 添加URIEncoding="UTF-8"

![1565670186710](SpringMVC-01.assets/1565670186710.png)

POST

​          默认情况下（不管什么版本的Tomtcat），SpringMVC接收的中文都是乱码的

   

### 12.3 实现



SpringMVC提供针对POST请求字符编码过滤器，在web.xml配置该过滤器即可！！

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	version="2.5">

	<!--使用springmvc之后，如果post请求出现了乱码，我们只需要把字符编码过滤器配置上即可，有现成-->
	<filter>
		<filter-name>characterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<!--注意： 使用该滤器的时候一定要配置编码，使用过滤器的初始化参数指定-->
		<init-param>
			<param-name>encoding</param-name>
			<param-value>utf-8</param-value>
		</init-param>
	</filter>

	<filter-mapping>
		<filter-name>characterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>


</web-app>
```





## 13. 请求参数绑定@RequestParam实现参数绑定

### 13.1 需求

当请求参数名称与方法形参不一致时候，如何封装请求数据？

**@RequestParam**

1. 应用场景：当请求参数名称与方法形参不一致时候使用
2. 建立请求参数与方法形参的映射关系

### 13.2 步骤

1.在UserController编写find方法

2.测试



### 13.3 实现

1.在EmpController编写find方法

```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
@RequestMapping("/emp")
public class EmpController {

    /*
        目标：讲解@RequestParam注解

        @RequestParam作用：解决参数名与形参名不一致的情况去使用

        @RequestParam常用的属性：
            1. name : 指定参数的名字
            2.defaultValue 该参数的默认值，如果没有该参数传递过来，则使用默认值
            3. required 该参数是一定要传递的


     */
    @RequestMapping("/find")
    public String find(@RequestParam(name = "id1",defaultValue ="999",required = true) Integer id,@RequestParam(name="name1",defaultValue = "狗娃") String name){
        System.out.println("id："+ id+" name:"+name);
        return "success";
    }


}

```







2.测试



![1604891990956](SpringMVC-01.assets/1604891990956.png)



## 14. 请求参数绑定自定义类型转换器

### 14.1 目标

在项目中我们需要从前端传递日期类型（2019-07-01）到后端，但是SpringMVC无法转换字符串为Date日期类型。这时我们可以通过自定义类型转换器，实现把请求参数值转换为指定的类型。

![1604892998052](SpringMVC-01.assets/1604892998052.png)





### 14.2 步骤

1. 自定义一个类实现Converter接口
2. 实现Converter接口中方法,把字符串转换为Date的业务逻辑
3. 在springmvc配置文件注册类型转换器



### 14.3 实现

1.定义类型转换器



```java
package com.itheima.utils;

import org.springframework.core.convert.converter.Converter;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
/*

    自定义类型转换器的步骤：
            1. 自定义一个类实现Converter接口
            2. 实现Converter接口中方法,把字符串转换为Date的业务逻辑
            3. 在springmvc配置文件注册类型转换器

*/
public class StringToDateConverter implements Converter<String,Date> {

    @Override
    public Date convert(String source) { //参数传入的字符串， 1990-12-12
        try {
            SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
            Date date = dateFormat.parse(source);
            return date;
        } catch (ParseException e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }
    }
}

```



3.配置类型转换器

![1604893635617](SpringMVC-01.assets/1604893635617.png)

![1604893655639](SpringMVC-01.assets/1604893655639.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--1. 配置视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--配置前缀,查找页面的路径-->
        <property name="prefix" value="/pages/"/>
        <!--配置后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--2. 开启注解扫描-->
    <context:component-scan base-package="com.itheima.controller"/>





    <!--====================================注册类型转换器======================================-->

    <!--1.创建类型转换器的对象-->
    <bean id="stringToDateConverter" class="com.itheima.utils.StringToDateConverter"/>


    <!--2.把类型转换器的对象交给类型转换器的工厂对象-->
    <bean id="conversionFactory" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <!--converters这个是ConversionServiceFactoryBean的属性，而且是set集合的属性-->
        <property name="converters">
            <set>
                <ref bean="stringToDateConverter"/>
            </set>
        </property>
    </bean>


    <!--3.把类型转换器工厂对象交给注解驱动去启动-->
    <!--3. 开启注解驱动 ,
          启动springmvc的三大组件与注册类型转换器
   注意： 导入名称空间的时候千万别导入cache的,是需要导入mvc的-->
    <mvc:annotation-driven conversion-service="conversionFactory"/>
</beans>
```

### 小结

自定义类型转换器步骤？

   1. 自定义一个类实现Converter接口，实现接口方法

   2. 在springmvc文件上注册类型转换器

      a. 创建类型转换器的对象

      b. 把类型转换器的对象交给类型转换器的工厂

      c. 把类型转换器的工厂交给注解驱动启动.









## 15. 请求参数绑定（六）servletApi对象作为方法参数



### 15.1 步骤

1.在UserController添加delete方法

2.测试



### 15.2 实现

1.在UserController添加delete方法



```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.util.Date;

@Controller
@RequestMapping("/user")
public class UserController {



    /*
        目标：在Controller中使用Servlet的API(request,response,session..)

        解决方案： 如果需要在Controller中去使用这些ApI，我们只需要在形参中直接声明即可，springmvc会自动给我们注入这些对象
     */
    @RequestMapping("/delete")
    public void delete(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String id = request.getParameter("id");
        System.out.println("得到的参数："+id);
        response.sendRedirect("/pages/success.jsp");
    }


}

```



2.测试

![1604894148318](SpringMVC-01.assets/1604894148318.png)





