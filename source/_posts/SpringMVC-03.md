---
title: SpringMVC(03)
tags:
  - 笔记
  - ssm框架
  - Spring
  - SpringMVC
  - ssm整合（SpringMVC+spring+mybatis）
categories:
  - ssm框架
abbrlink: 54365
date: 2020-11-12 21:05:35
---

### 00、学习目标

1、了解SpringMVC拦截器 反馈

2、掌握ssm整合（SpringMVC+spring+mybatis）

### 01、ssm整合（一）整合步骤分析

#### 1.1 目标

分析ssm整合实现方案，整合流程。

#### 1.2 什么是SSM整合？

1. SpringMVC Spring Mybatis ，简称ssm
2. Spring整合SpringMVC
3. Spring整合Mybatis

![image-20200705090022491](SpringMVC-03.assets/image-20200705090022491.png)



#### 1.3 整合方案

1. <font color=red><b>注解 + XML 【推荐】</b></font>

   注解： 自定义的类（dao、service）

   XML： jar包中的类（DataSource，DataSourceTransactionManager）

2. 纯XML

3. 纯注解

#### 1.4 整合流程

1. 单独搭建Spring开发环境
2. 单独搭建SpringMVC开发环境
3. Spring整合SpringMVC
4. 单独搭建mybatis环境
5. Spring整合Mybatis**[ssm整合完成]**

![image-20200705090621971](SpringMVC-03.assets/image-20200705090621971.png)





### 02、ssm整合（二）单独Spring环境

#### 2.1 目标

搭建并测试spring环境.

#### 2.2 实现步骤

1. 创建项目：springmvc03_ssm，添加依赖
2. 编写User实体类，封装数据
3. 编写UserService接口、实现类
4. 编写applicationContext.xml
5. 测试

#### 2.3 实现代码

1、创建项目：springmvc03_ssm，添加依赖

![image-20200705091424179](SpringMVC-03.assets/image-20200705091424179.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
  <modelVersion>4.0.0</modelVersion>  
  <groupId>com.itheima</groupId>  
  <artifactId>springmvcday03_ssm</artifactId>  
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <dependencies>
    <!--SpringMVC支持包-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.0.2.RELEASE</version>
    </dependency>

    <!--为了使用Spring声明式事务-->
    <!--SpringJDBC包 , 虽然你没有使用jdbctemplate，但是你用上事务管理器，spring事务管理器要使用该依赖-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.0.2.RELEASE</version>
    </dependency>
    <!--Spring事务支持包-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>5.0.2.RELEASE</version>
    </dependency>
    <!--AspectJ支持包  切入点表达式-->
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.8.13</version>
    </dependency>

    <!--Spring测试支持包-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>5.0.2.RELEASE</version>
    </dependency>

    <!--数据库驱动包-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.47</version>
    </dependency>
    <!--连接池-->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.0.31</version>
    </dependency>

    <!--MyBatis核心包-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.4.5</version>
    </dependency>
    <!--***Spring整合MyBatis支持包***-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.2</version>
    </dependency>

    <!--junit支持包-->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
    </dependency>

    <!--jstl支持包-->
    <dependency>
      <groupId>jstl</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>

    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
    </dependency>

    <!--日志依赖-->
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.17</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
      <version>1.7.29</version>
    </dependency>
  </dependencies>
</project>

```

2、编写User实体类，封装数据

```java
package com.itheima.model;

import java.util.Date;

public class User {

    private Integer id;

    private String username;

    private Date birthday;

    private String sex;

    private String address;


    public User() {
    }

    public User(Integer id, String username, Date birthday, String sex, String address) {
        this.id = id;
        this.username = username;
        this.birthday = birthday;
        this.sex = sex;
        this.address = address;
    }

    /**
     * 获取
     * @return id
     */
    public Integer getId() {
        return id;
    }

    /**
     * 设置
     * @param id
     */
    public void setId(Integer id) {
        this.id = id;
    }

    /**
     * 获取
     * @return username
     */
    public String getUsername() {
        return username;
    }

    /**
     * 设置
     * @param username
     */
    public void setUsername(String username) {
        this.username = username;
    }

    /**
     * 获取
     * @return birthday
     */
    public Date getBirthday() {
        return birthday;
    }

    /**
     * 设置
     * @param birthday
     */
    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    /**
     * 获取
     * @return sex
     */
    public String getSex() {
        return sex;
    }

    /**
     * 设置
     * @param sex
     */
    public void setSex(String sex) {
        this.sex = sex;
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
        return "User{id = " + id + ", username = " + username + ", birthday = " + birthday + ", sex = " + sex + ", address = " + address + "}";
    }
}

```

3、编写UserSevice接口、实现

```java
package com.itheima.service;

import com.itheima.model.User;

import java.util.List;

public interface UserService {

    public List<User> findAll();
}

```

```java
package com.itheima.service.impl;

import com.itheima.model.User;
import com.itheima.service.UserService;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserServiceImpl implements UserService {


    @Override
    public List<User> findAll() {
        System.out.println("查询用户...");
        return null;
    }
}

```

4、编写applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--扫描service包-->
    <context:component-scan base-package="com.itheima.service"/>


</beans>
```

5、测试

```java
package com.itheima.test;

import com.itheima.model.User;
import com.itheima.service.UserService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.List;

/*
spring整合junit
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class AppTest {

    @Autowired
    private UserService userService;

    @Test
    public void test01(){
        List<User> list = userService.findAll();

    }
}

```

#### 2.4 小结

**工程结构图：**

![1605142719659](SpringMVC-03.assets/1605142719659.png)



### 03、ssm整合（三）单独SpringMVC环境

#### 3.1 目标

独立搭建springMVC环境。

#### 3.2 步骤

1. web.xml配置前端（核心）控制器（前端控制，过滤器，加载springmvc核心配置文件）
2. springmvc.xml(1，视图解析器，包扫描,注解驱动,静态访问)
3. UserController控制器
4. list.jsp页面(只是随便来点数据即可)
5. 测试

#### 3.3 代码

1、先把当前模块转换web模块，web.xml配置前端控制器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	version="2.5">

	<!--1. 核心控制器-->
	<servlet>
		<servlet-name>dispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<!--2. 加载springmvc的配置文件-->
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc.xml</param-value>
		</init-param>
		<!--服务器启动的时候创建核心控制器的对象-->
		<load-on-startup>1</load-on-startup>
	</servlet>
	
	<servlet-mapping>
		<servlet-name>dispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

	<!--3. 配置过滤器-->
	<filter>
		<filter-name>characterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<!--指定过滤器使用码表-->
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

2、springmvc.xml springmvc的核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--1. 视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/pages/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--2. 开启包扫描-->
    <context:component-scan base-package="com.itheima.controller"/>

    <!--3. 注解驱动-->
    <mvc:annotation-driven/>

    <!--4.解决静态资源访问问题-->
    <mvc:default-servlet-handler/>
</beans>
```

3、UserController控制器

```java
package com.itheima.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/list")
    public String list(){
        return "list";
    }
}

```

4、list.jsp页面

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h1>员工列表...</h1>
</body>
</html>

```



5、测试

![1597974119060](SpringMVC-03.assets/1597974119060.png)

​		注意==这里只是测试环境，没有数据==





### 04、ssm整合（四）Spring整合SpringMVC

#### 4.1 目标

掌握spring整合springmvc整合步骤。

Spring整合SpringMVC核心： 在web.xml中通过spring提供的一个监听器，加载applicationContext.xml

#### 4.2 关键点

在项目启动时候，加载web.xml；web.xml文件已经指定加载springmvc配置文件，但是没有加载ApplicationContext.xml文件，所以导致spring的环境没有被加载加载，所以service层没有扫描到，所以web目前是没法使用service的代码。 -==解决方案：在web.xml中要加载applicationContext.xml==

&nbsp;![image-20200705102301191](SpringMVC-03.assets/image-20200705102301191.png)





#### 4.3 步骤

1. 配置web.xml，配置spring提供的ServletContext监听器： ContextLoaderListener  (ServletContextListener )
2. 修改controller，注入service

#### 4.4 代码

1、配置web.xml，配置spring提供的ServletContext监听器： ContextLoaderListener

![1605144146368](SpringMVC-03.assets/1605144146368.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	version="2.5">

	<!--1. 核心控制器-->
	<servlet>
		<servlet-name>dispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<!--2. 加载springmvc的配置文件-->
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc.xml</param-value>
		</init-param>
		<!--服务器启动的时候创建核心控制器的对象-->
		<load-on-startup>1</load-on-startup>
	</servlet>
	
	<servlet-mapping>
		<servlet-name>dispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

	<!--3. 配置过滤器-->
	<filter>
		<filter-name>characterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<!--指定过滤器使用码表-->
		<init-param>
			<param-name>encoding</param-name>
			<param-value>utf-8</param-value>
		</init-param>
	</filter>
	
	<filter-mapping>
		<filter-name>characterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>


	<!--spring整合springmvc核心的一步：利用spring提供的监听器去加载applicationContext的配置文件即可-->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	<!--配置全局参数指定文件的路径-->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:applicationContext.xml</param-value>
	</context-param>
</web-app>
```

2、修改controller，注入service

```java
package com.itheima.controller;

import com.itheima.model.User;
import com.itheima.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import java.util.List;

@Controller
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping("/list")
    public String list(HttpServletRequest request){
        List<User> list = userService.findAll();
        request.setAttribute("list",list);
        return "list";
    }
}

```



#### 4.5 小结

​	**spring整合springmvc核心步骤？**

   - 在web.xml文件上创建ContextLoaderListener监听器加载applicationContext.xml文件

     



### 05、ssm整合（五）单独Mybatis环境

#### 5.1 目标

单独使用mybatis查询数据库

#### 5.2 实现步骤

1. 编写dao接口（如果你不是使用注解开发，还有映射文件Mapper.xml）
2. 编写SqlMapConfig.xml
3. 编写测试类

#### 5.3 实现代码

1. 编写dao接口

   ​	**先创建表**

   ```sql
   CREATE TABLE `user` (
     `id` INT(11) NOT NULL AUTO_INCREMENT,
     `username` VARCHAR(30) DEFAULT NULL,
     `birthday` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
     `sex` CHAR(2) DEFAULT NULL,
     `address` VARCHAR(30) DEFAULT NULL,
     PRIMARY KEY (`id`)
   ) ENGINE=INNODB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;
   
   INSERT  INTO `user`(`id`,`username`,`birthday`,`sex`,`address`) VALUES (1,'李成硕','2020-08-17 07:43:47','男','上海'),(2,'阿祖','2020-08-17 07:43:44','女','香港'),(3,'慧慧','2020-08-20 08:13:46','女','北京');
   
   ```

   **再创建userDao**

   ```java
   package com.itheima.dao;
   
   import com.itheima.model.User;
   import org.apache.ibatis.annotations.Select;
   
   import java.util.List;
   
   public interface UserDao {
   
       @Select("select * from user")
       List<User> findAll();
   }
   
   ```

2. 编写SqlMapConfig.xml

   **需要先引入db.properties**

   **db.properties**

   ```properties
   jdbc.url=jdbc:mysql:///db1
   jdbc.username=root
   jdbc.password=root
   jdbc.driver=com.mysql.jdbc.Driver
   ```

   log4j.properties

   ```properties
   log4j.rootLogger=debug,stdout
   log4j.appender.stdout=org.apache.log4j.ConsoleAppender
   log4j.appender.stdout.Target=System.out
   log4j.appender.stdout.layout=org.apache.log4j.SimpleLayout
   ```

   

   **SqlMapConfig.xml**

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
   
       <!--加载外部的properties配置-->
       <properties resource="db.properties"></properties>
   
   
       <!--设置java类型别名-->
       <typeAliases>
           <!--将整个包下所有的类名设置了别名，别名（小名）：类名-->
           <package name="com.itheima.model"></package>
       </typeAliases>
   
   
       <!--数据库环境配置-->
       <environments default="mysql">
           <!--使用MySQL环境-->
           <environment id="mysql">
               <!--事务管理器：JDBC类型-->
               <transactionManager type="JDBC"></transactionManager>
               <!--连接池：内置POOLED-->
               <dataSource type="POOLED">
                   <property name="driver" value="${jdbc.driver}"></property>
                   <property name="url" value="${jdbc.url}"></property>
                   <property name="username" value="${jdbc.username}"></property>
                   <property name="password" value="${jdbc.password}"></property>
               </dataSource>
           </environment>
       </environments>
   
   
       <!--加载映射文件-->
       <mappers>
          <!--  Dao接口所在包-->
           <package name="com.itheima.dao"></package>
       </mappers>
   </configuration>
   ```

3. 编写测试类

   ```java
   package com.itheima.test;
   
   import com.itheima.dao.UserDao;
   import com.itheima.model.User;
   import org.apache.ibatis.session.SqlSession;
   import org.apache.ibatis.session.SqlSessionFactory;
   import org.apache.ibatis.session.SqlSessionFactoryBuilder;
   import org.junit.Test;
   
   import java.util.List;
   
   public class MybatisTest {
   
   
       @Test
       public void test01(){
           //1、创建sqlSession工程的构造器
           SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
   
           //2. 通过sqlSession的构造器得到SqlSessionFactory
           SqlSessionFactory sqlSessionFactory = builder.build(MybatisTest.class.getResourceAsStream("/sqlMapConfig.xml"));
   
           //3. 通过SqlSessionFactory得到SqlSession
           SqlSession sqlSession = sqlSessionFactory.openSession();
   
   
           //4. 通过SqlSession得到接口的代理对象
           UserDao userDao = sqlSession.getMapper(UserDao.class);
   
   
           //5. 调用方法
           List<User> list = userDao.findAll();
           System.out.println("集合对象："+list);
   
           //6. 关闭sqlSession
           sqlSession.close();
       }
   
   }
   
   ```





### 06、ssm整合（六）Spring整合Mybatis环境、测试

#### 6.1 目标

1. 完成Spring整合Mybatis。配置：applicationContext.xml
2. 关键点： 把SqlSessionFactory的创建，交给SpringIOC容器完成。



#### 6.2 实现步骤

1. 配置applicationContext.xml
2. 修改service，注入dao，直接调用dao方法

#### 6.3 代码实现

1、配置applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--扫描service包-->
    <context:component-scan base-package="com.itheima.service"/>


   <!-- ================================spring整合mybatis========================-->

    <!--1. 加载db.properties的配置文件-->
    <context:property-placeholder  location="classpath:db.properties"/>


    <!--2.创建连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!--3. 扫描dao包，注意：如果你的dao层是使用mybatis，不需要使用@Repository注解创建dao实现类对象-->
    <!--MapperScannerConfigurer扫描dao包的时候，扫到接口会创建代理对象并且交给spring容器的-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.itheima.dao"/>
    </bean>

    <!--4. 创建SqlSession工厂-->
    <bean class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!--别名配置-->
        <property name="typeAliasesPackage" value="com.itheima.model"/>
    </bean>



    <!--=====================================添加事务管理=============================================-->
    <!--1. 创建事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>



    <!--2.配置事务通知规则：规定哪些方法需要事务，哪些方法不需要事务
        propagation ： 事务传播行为  REQUIRED(必须) SUPPORTS(可有可无)
     -->
    <tx:advice id="ad" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="find*" propagation="SUPPORTS"/>
            <tx:method name="select*" propagation="SUPPORTS"/>
            <tx:method name="get*" propagation="SUPPORTS"/>
            <tx:method name="query*" propagation="SUPPORTS"/>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>


    <!--3. 配置切面-->
    <aop:config>
        <!--切入点表达式 事务是针对service层-->
        <aop:pointcut id="pt" expression="execution(* com.itheima.service.impl.*.*(..))"/>
        <!--配置通知-->
        <aop:advisor advice-ref="ad" pointcut-ref="pt"/>
    </aop:config>



</beans>
```

2、修改service，注入dao，直接调用dao方法

```java
package com.itheima.service.impl;

import com.itheima.dao.UserDao;
import com.itheima.model.User;
import com.itheima.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;


    @Override
    public List<User> findAll() {
        return userDao.findAll();
    }
}

```

#### 6.4 测试

   ```java
package com.itheima.test;

import com.itheima.dao.UserDao;
import com.itheima.model.User;
import com.itheima.service.UserService;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.List;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class MybatisTest {

    @Autowired
  private UserService userService;


    @Test
    public void test01(){
        //5. 调用方法
        List<User> list = userService.findAll();
        System.out.println("集合对象："+list);

    }

}

   ```

### 07、ssm整合（七）ssm整合，页面实现

#### 7.1 目标

启动项目，测试ssm环境。实现页面显示。

#### 7.2 列表页面

控制器代码：

```java
package com.itheima.controller;

import com.itheima.model.User;
import com.itheima.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import java.util.List;

@Controller
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping("/list")
    public String list(HttpServletRequest request){
        List<User> list = userService.findAll();
        request.setAttribute("list",list);
        return "list";
    }
}

```



list.jsp

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

<table border="1">
    <tr>
        <th>账户ID</th>
        <th>用户名称</th>
        <th>用户生日</th>
        <th>性别</th>
        <th>地址</th>
    </tr>

    <c:forEach items="${list}" var="user">
        <tr>
            <td>${user.id}</td>
            <td>${user.username}</td>
            <td>${user.birthday}</td>
            <td>${user.sex}</td>
            <td>${user.address}</td>
        </tr>
    </c:forEach>


</table>
</body>
</html>

```

#### 7.3 效果

可以显示用户列表，但是日期格式不对：

![1596079594809](SpringMVC-03.assets/1596079594809.png)

#### 7.4 优化

优化日期格式：

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%--<%@ taglib prefix="ftm" uri="http://java.sun.com/jsp/jstl/fmt" %>--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

<table border="1">
    <tr>
        <th>账户ID</th>
        <th>用户名称</th>
        <th>用户生日</th>
        <th>性别</th>
        <th>地址</th>
    </tr>

    <c:forEach items="${list}" var="user">
        <tr>
            <td>${user.id}</td>
            <td>${user.username}</td>
                <%--
                    日期格式不对解决方案：
                         方案一： 使用jstl的函数库的dateformat标签
                             <ftm:formatDate value="${user.birthday}" pattern="yyyy-MM-dd"/>
                         方案二： 由于el表达式获取对象的属性都是依赖getter方法，那么修改getBirthday方法
                         --%>
            <td>
                ${user.birthday}
            </td>
            <td>${user.sex}</td>
            <td>${user.address}</td>
        </tr>
    </c:forEach>


</table>
</body>
</html>

```

**User实体类的修改**

![1605148710113](SpringMVC-03.assets/1605148710113.png)

```java
package com.itheima.model;

import java.text.SimpleDateFormat;
import java.util.Date;

public class User {

    private Integer id;

    private String username;

    private Date birthday;

    private String sex;

    private String address;


    public User() {
    }

    public User(Integer id, String username, Date birthday, String sex, String address) {
        this.id = id;
        this.username = username;
        this.birthday = birthday;
        this.sex = sex;
        this.address = address;
    }

    /**
     * 获取
     * @return id
     */
    public Integer getId() {
        return id;
    }

    /**
     * 设置
     * @param id
     */
    public void setId(Integer id) {
        this.id = id;
    }

    /**
     * 获取
     * @return username
     */
    public String getUsername() {
        return username;
    }

    /**
     * 设置
     * @param username
     */
    public void setUsername(String username) {
        this.username = username;
    }

    /**
     * 获取
     * @return birthday
     */
    public String getBirthday() {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        return dateFormat.format(birthday);
    }

    /**
     * 设置
     * @param birthday
     */
    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    /**
     * 获取
     * @return sex
     */
    public String getSex() {
        return sex;
    }

    /**
     * 设置
     * @param sex
     */
    public void setSex(String sex) {
        this.sex = sex;
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
        return "User{id = " + id + ", username = " + username + ", birthday = " + birthday + ", sex = " + sex + ", address = " + address + "}";
    }
}

```



优化后效果：

![image-20200605132642878](SpringMVC-03.assets/image-20200605132642878.png)

#### 7,5 小结

工程结构图：

![1597979781138](SpringMVC-03.assets/1597979781138.png)







### 08、基于RESTFUL案例（一）部署UI （列表）

#### 引入资源

拷贝`springmvc03\04 素材` 中的所有资源到项目webapp中：

![image-20200605132959188](SpringMVC-03.assets/image-20200605132959188.png)



#### 修改控制器

因为我们使用restful风格的地址，一个地址表示多次请求，通过请求方式区分。 所以这里我们修改一下访问 路径、以及控制器方法返回值。

![1605148985640](SpringMVC-03.assets/1605148985640.png)

```java
package com.itheima.controller;

import com.itheima.model.User;
import com.itheima.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import java.util.List;

@Controller
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping("/list")
    public String list(HttpServletRequest request){
        List<User> list = userService.findAll();
        request.setAttribute("list",list);
        return "user/user-list";
    }
}

```

#### 修改页面

user-list.jsp 页面修改取值：

![1605149015345](SpringMVC-03.assets/1605149015345.png)

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ include file="../base.jsp"%>
<!DOCTYPE html>
<html>

<head>
    <!-- 页面meta -->
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>数据 - AdminLTE2定制版</title>
    <meta name="description" content="AdminLTE2定制版">
    <meta name="keywords" content="AdminLTE2定制版">
    <!-- Tell the browser to be responsive to screen width -->
    <meta content="width=device-width,initial-scale=1,maximum-scale=1,factory-scalable=no" name="viewport">
</head>

<body>
<div id="frameContent" class="content-wrapper" style="margin-left:0px;">
    <section class="content-header">
        <h1>
            系统管理
            <small>用户管理</small>
        </h1>
        <ol class="breadcrumb">
            <li><a href="all-admin-index.html"><i class="fa fa-dashboard"></i> 首页</a></li>
        </ol>
    </section>
    <section class="content">
        <div class="box box-primary">
            <div class="box-header with-border">
                <h3 class="box-title">用户列表</h3>
            </div>
            <div class="box-body">
                <div class="table-box">
                    <div class="pull-left">
                        <div class="form-group form-inline">
                            <div class="btn-group">
                                <button type="button" class="btn btn-default" title="添加" onclick='location.href="${pageContext.request.contextPath}/pages/user/user-add.jsp"'>
                                    <i class="fa fa-file-o"></i> 添加
                                </button>
                                <button type="button" class="btn btn-default" title="删除" onclick='deleteuser()'>
                                    <i class="fa fa-file-o"></i> 删除
                                </button>
                            </div>
                        </div>
                    </div>
                    <div class="box-tools pull-right">
                        <div class="has-feedback">
                            <input type="text" class="form-control input-sm" placeholder="搜索">
                            <span class="glyphicon glyphicon-search form-control-feedback"></span>
                        </div>
                    </div>
                    <table id="dataList" class="table table-bordered table-striped table-hover dataTable">
                        <thead>
                        <tr>
                            <th class="sorting">选择</th>
                            <th class="sorting">账户ID</th>
                            <th class="sorting">用户名称</th>
                            <th class="sorting">用户生日</th>
                            <th class="sorting">性别</th>
                            <th class="sorting">地址</th>
                        </tr>
                        </thead>
                        <tbody>
                        <c:forEach items="${list}" var="user">
                            <tr>
                                <td><input type="radio" name="id" value="${user.id}"/></td>
                                <td>${user.id}</td>
                                <td>${user.username}</td>
                                <td>${user.birthday}</td>
                                <td>${user.sex}</td>
                                <td>${user.address}</td>
                            </tr>
                        </c:forEach>
                        </tbody>
                    </table>
                </div>
            </div>
            <div class="box-footer">


            </div>
        </div>

    </section>
</div>
</body>

</html>
```

#### 小结

此时，list.jsp就可以删除了，最终效果如下

![image-20200605142417056](SpringMVC-03.assets/image-20200605142417056.png)



### 09 把用户列表同步请求修改为异步

#### 9.1 步骤

1. 导入jquery
2. 修改user-list.jsp，页面加载完毕的时候发出异步的请求
3. 修改UserController的list方法，返回的值修改为json.



#### 9.2实现

1. 导入jquery

   ![1605149398751](SpringMVC-03.assets/1605149398751.png)

2. 修改user-list.jsp，页面加载完毕的时候发出异步的请求

   ![1605149936567](SpringMVC-03.assets/1605149936567.png)

   ![1605149965670](SpringMVC-03.assets/1605149965670.png)

   ```jsp
   <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
   <%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <%@ include file="../base.jsp"%>
   <!DOCTYPE html>
   <html>
   
   <head>
       <!-- 页面meta -->
       <meta charset="utf-8">
       <meta http-equiv="X-UA-Compatible" content="IE=edge">
       <title>数据 - AdminLTE2定制版</title>
       <meta name="description" content="AdminLTE2定制版">
       <meta name="keywords" content="AdminLTE2定制版">
       <script src="${pageContext.request.contextPath}/js/jquery-3.3.1.min.js"></script>
       <!-- Tell the browser to be responsive to screen width -->
       <meta content="width=device-width,initial-scale=1,maximum-scale=1,factory-scalable=no" name="viewport">
   </head>
   
   <body>
   <div id="frameContent" class="content-wrapper" style="margin-left:0px;">
       <section class="content-header">
           <h1>
               系统管理
               <small>用户管理</small>
           </h1>
           <ol class="breadcrumb">
               <li><a href="all-admin-index.html"><i class="fa fa-dashboard"></i> 首页</a></li>
           </ol>
       </section>
       <section class="content">
           <div class="box box-primary">
               <div class="box-header with-border">
                   <h3 class="box-title">用户列表</h3>
               </div>
               <div class="box-body">
                   <div class="table-box">
                       <div class="pull-left">
                           <div class="form-group form-inline">
                               <div class="btn-group">
                                   <button type="button" class="btn btn-default" title="添加" onclick='location.href="${pageContext.request.contextPath}/pages/user/user-add.jsp"'>
                                       <i class="fa fa-file-o"></i> 添加
                                   </button>
                                   <button type="button" class="btn btn-default" title="删除" onclick='deleteuser()'>
                                       <i class="fa fa-file-o"></i> 删除
                                   </button>
                               </div>
                           </div>
                       </div>
                       <div class="box-tools pull-right">
                           <div class="has-feedback">
                               <input type="text" class="form-control input-sm" placeholder="搜索">
                               <span class="glyphicon glyphicon-search form-control-feedback"></span>
                           </div>
                       </div>
                       <table id="dataList" class="table table-bordered table-striped table-hover dataTable">
                           <thead>
                           <tr>
                               <th class="sorting">选择</th>
                               <th class="sorting">账户ID</th>
                               <th class="sorting">用户名称</th>
                               <th class="sorting">用户生日</th>
                               <th class="sorting">性别</th>
                               <th class="sorting">地址</th>
                           </tr>
                           </thead>
                           <tbody>
   
                           </tbody>
                       </table>
                   </div>
               </div>
               <div class="box-footer">
               </div>
           </div>
   
       </section>
   </div>
   </body>
   
   <script>
   
       //页面加载完毕，得到用户的列表
       $(function(){
           list();
       })
   
       //得到用户列表方法
       function list(){
           $.ajax({
               url:"/user", //请求路径
               type:"get", //请求方式
               dataType:"json", //服务器返回的数据格式
               success:function(userList){
                   // 定义一个变量保存拼接的结果
                   var resultHtml ="";
                   //遍历userList
                   for(var user of userList){
                       //每一个user就应该是一个tr标签
                       resultHtml+='<tr>\n' +
                           '                                <td><input type="radio" name="id" value='+user.id+'/></td>\n' +
                           '                                <td>'+user.id+'</td>\n' +
                           '                                <td>'+user.username+'</td>\n' +
                           '                                <td>'+user.birthday+'</td>\n' +
                           '                                <td>'+user.sex+'</td>\n' +
                           '                                <td>'+user.address+'</td>\n' +
                           '                            </tr>';
                   }
                   //设置tbody的标签体内容
                   $("tbody").html(resultHtml);
               }
           });
   
       }
   
   
   </script>
   
   </html>
   ```

   

3. 修改UserController的list方法，返回的值修改为json.

```java
package com.itheima.controller;

import com.itheima.model.User;
import com.itheima.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import java.util.List;

@Controller
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping(method = RequestMethod.GET)
    @ResponseBody
    public List<User> list(){
        List<User> list = userService.findAll();
        return list;
    }
}

```





### 10、基于RESTFUL案例（二）添加用户

#### 10.1 流程

添加用户流程如下：

1、页面点击添加，

![image-20200605135628085](SpringMVC-03.assets/image-20200605135628085.png)

2、进入添加页面，

![image-20200605140319401](SpringMVC-03.assets/image-20200605140319401.png)

3、添加保存后，提交后台处理，最终重定向到列表

#### 10.2 思考

这里的birthday是日期类型，我们页面的日期字符串可以自动封装为日期类型吗？

#### 10.3 步骤

1、user-add.jsp检查表单提交地址

2、编写控制器

3、编写service

4、编写dao

5、编写日期类型转换器、配置转换器

#### 10.4 实现



1、user-add.jsp检查表单提交地址

第一：修改地址

![1597992476309](SpringMVC-03.assets/1597992476309.png)

```html
 <form id="editForm" action="${pageContext.request.contextPath}/user" method="post">
```

第二：还需要修改日历控件：

![image-20200705153454397](SpringMVC-03.assets/image-20200705153454397.png)

![1596091821388](SpringMVC-03.assets/1596091821388.png)

第三： 修改性别单选框的value值

![1597993594956](SpringMVC-03.assets/1597993594956.png)



2、编写控制器， 添加add()方法， 处理请求类型是`post`路径为`/user` 的请求



```java
package com.itheima.controller;

import com.itheima.model.User;
import com.itheima.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import java.util.List;

@Controller
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping(method = RequestMethod.GET)
    @ResponseBody
    public List<User> list(){
        List<User> list = userService.findAll();
        return list;
    }


    @RequestMapping(method = RequestMethod.POST)
    public String add(User user){
        userService.save(user);
        //添加完毕之后，回到列表页面,列表页面会发出异步请求，请求用户列表
        return "user/user-list";
    }



}

```

3、编写service

```java
package com.itheima.service;

import com.itheima.model.User;

import java.util.List;

public interface UserService {

    public List<User> findAll();

    //添加用户
    void save(User user);
}

```

```java
package com.itheima.service.impl;

import com.itheima.dao.UserDao;
import com.itheima.model.User;
import com.itheima.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;


    @Override
    public List<User> findAll() {
        return userDao.findAll();
    }

    //添加用户
    @Override
    public void save(User user) {
        userDao.save(user);
    }
}

```

4、编写dao

```java
package com.itheima.dao;

import com.itheima.model.User;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;

import java.util.List;


public interface UserDao {

    @Select("select * from user")
    List<User> findAll();

    @Insert("insert into user values(null,#{username},#{birthday},#{sex},#{address})")
    void save(User user);
}

```

5、编写日期类型转换器、配置转换器

```java
package com.itheima.utils;

import org.springframework.core.convert.converter.Converter;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class StringToDateConverter implements Converter<String,Date> {
    @Override
    public Date convert(String source) {
        try {
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
            Date date =  simpleDateFormat.parse(source);
            return date;
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }
}

```

`springMVC.xml`  这里修改3点：1、创建类型转换器；2、l把类型转换器交给类型转换器的工厂；  3、把类型转换器的工厂交给注解驱动启动。



截图代码：

![1605151318941](SpringMVC-03.assets/1605151318941.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--1. 视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/pages/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--2. 开启包扫描-->
    <context:component-scan base-package="com.itheima.controller"/>



    <!--4.解决静态资源访问问题-->
    <mvc:default-servlet-handler/>


    <!--配置日期类型转换器-->
    <!--1.创建日期类型转换器对象-->
    <bean id="stringToDateConverter" class="com.itheima.utils.StringToDateConverter"/>

    <!--2. 把类型转换器交给类型转换器的工厂-->
    <bean id="conversionFactory" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <ref bean="stringToDateConverter"/>
            </set>
        </property>
    </bean>

    <!--3. 把类型转换器的工厂交给注解驱动启动 , 注意： 该语句是从上面剪切下来的-->
    <mvc:annotation-driven conversion-service="conversionFactory"/>
</beans>
```



### 10、基于RESTFUL案例（二）删除用户

#### 10.1 需求

异步删除用户信息，每次只能删除一个用户。

![image-20200605154405046](SpringMVC-03.assets/image-20200605154405046.png)

#### 10.2 步骤

1、在user-list页面添加deleteuser的方法

2、编写控制器

3、编写service

4、编写dao

#### 10.3 实现

1、在user-list页面添加deleteuser的方法

```js
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ include file="../base.jsp"%>
<!DOCTYPE html>
<html>

<head>
    <!-- 页面meta -->
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>数据 - AdminLTE2定制版</title>
    <meta name="description" content="AdminLTE2定制版">
    <meta name="keywords" content="AdminLTE2定制版">
    <!-- Tell the browser to be responsive to screen width -->
    <meta content="width=device-width,initial-scale=1,maximum-scale=1,factory-scalable=no" name="viewport">
</head>

<body>
<div id="frameContent" class="content-wrapper" style="margin-left:0px;">
    <section class="content-header">
        <h1>
            系统管理
            <small>用户管理</small>
        </h1>
        <ol class="breadcrumb">
            <li><a href="all-admin-index.html"><i class="fa fa-dashboard"></i> 首页</a></li>
        </ol>
    </section>
    <section class="content">
        <div class="box box-primary">
            <div class="box-header with-border">
                <h3 class="box-title">用户列表</h3>
            </div>
            <div class="box-body">
                <div class="table-box">
                    <div class="pull-left">
                        <div class="form-group form-inline">
                            <div class="btn-group">
                                <button type="button" class="btn btn-default" title="添加" onclick='location.href="${pageContext.request.contextPath}/pages/user/user-add.jsp"'>
                                    <i class="fa fa-file-o"></i> 添加
                                </button>
                                <button type="button" class="btn btn-default" title="删除" onclick='deleteuser()'>
                                    <i class="fa fa-file-o"></i> 删除
                                </button>
                            </div>
                        </div>
                    </div>
                    <div class="box-tools pull-right">
                        <div class="has-feedback">
                            <input type="text" class="form-control input-sm" placeholder="搜索">
                            <span class="glyphicon glyphicon-search form-control-feedback"></span>
                        </div>
                    </div>
                    <table id="dataList" class="table table-bordered table-striped table-hover dataTable">
                        <thead>
                        <tr>
                            <th class="sorting">选择</th>
                            <th class="sorting">账户ID</th>
                            <th class="sorting">用户名称</th>
                            <th class="sorting">用户生日</th>
                            <th class="sorting">性别</th>
                            <th class="sorting">地址</th>
                        </tr>
                        </thead>
                        <tbody>
                        <c:forEach items="${list}" var="user">
                            <tr>
                                <td><input type="radio" name="id" value="${user.id}"/></td>
                                <td>${user.id}</td>
                                <td>${user.username}</td>
                                    <%--
                                       formatDate: 格式化日期
                                         value： 需要格式化的提前
                                         pattern: 格式表达式
                                    --%>
                                <td>${user.birthday}</td>
                                <td>${user.sex}</td>
                                <td>${user.address}</td>
                            </tr>
                        </c:forEach>
                        </tbody>
                    </table>
                </div>
            </div>
            <div class="box-footer">


            </div>
        </div>

    </section>
</div>
</body>

<script>

   //删除用户
    function deleteuser(){
        //1. 获取到选中的id
       var id = $("input[type='radio']:checked").val();
        //2. 判断是否有选中的id
        if(id){
            //3. 如果选中id，那么发出异步请求
           $.ajax({
               url:"/user/"+id,
               type:"delete",  //注意：表单的请求方式只有get与post，但是异步请求是支持任意中请求方式的。所以不需要配置过滤器
               success:function(result){
                    if(result=='ok'){
                        //删除成功
                        alert("删除成功!");
                        location.reload();//重新加载当前页面
                    }else{
                        //删除失败
                        alert("删除失败");
                    }
               }
           });

        }else{
            //4. 如果没有选中id，那么提示用户
            alert("没有选中要删除的用户,请选择!")
        }



    }


</script>

</html>
```

2、编写控制器,添加delete的方法

```java
package com.itheima.controller;

import com.itheima.model.User;
import com.itheima.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import java.util.List;

@Controller
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping(method = RequestMethod.GET)
    @ResponseBody
    public List<User> list(){
        List<User> list = userService.findAll();
        return list;
    }


    @RequestMapping(method = RequestMethod.POST)
    public String add(User user){
        userService.save(user);
        //添加完毕之后，回到列表页面,列表页面会发出异步请求，请求用户列表
        return "user/user-list";
    }


    @RequestMapping(value ="/{id}" ,method = RequestMethod.DELETE)
    @ResponseBody //如果是返回一个字符串，那么就直接返回字符串，如果是java对象，那么会先转换为json再返回。
    public String delete(@PathVariable("id") Integer id){
        userService.delete(id);
        //添加完毕之后，回到列表页面,列表页面会发出异步请求，请求用户列表
        return "ok";
    }



}

```

3、编写service,添加delete方法

```java
package com.itheima.service;

import com.itheima.model.User;

import java.util.List;

public interface UserService {

    public List<User> findAll();

    //添加用户
    void save(User user);

    //根据id删除用户
    void delete(Integer id);
}

```

```java
package com.itheima.service.impl;

import com.itheima.dao.UserDao;
import com.itheima.model.User;
import com.itheima.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;


    @Override
    public List<User> findAll() {
        return userDao.findAll();
    }

    //添加用户
    @Override
    public void save(User user) {
        userDao.save(user);
    }

    @Override
    public void delete(Integer id) {
        userDao.delete(id);
    }
}

```

4、编写dao,添加delete方法

```java
package com.itheima.dao;

import com.itheima.model.User;
import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;

import java.util.List;


public interface UserDao {

    @Select("select * from user")
    List<User> findAll();

    @Insert("insert into user values(null,#{username},#{birthday},#{sex},#{address})")
    void save(User user);

    @Delete("delete from user where id = #{id}")
    void delete(Integer id);
}

```

#### 10.4 测试

选择一条记录，

![image-20200605154939116](SpringMVC-03.assets/image-20200605154939116.png)

点击删除，

![image-20200605155013834](SpringMVC-03.assets/image-20200605155013834.png)

点击确定，列表重新刷新，少了一条记录

![image-20200605155034013](SpringMVC-03.assets/image-20200605155034013.png)





