---
title: Spring(01)
tags:
  - 笔记
  - ssm框架
  - Spring
  - 控制反转
  - IOC
  - 依赖注入
  - DI
  - Spring的生命周期
categories:
  - ssm框架
date: 2020-11-05 21:05:40
---

## 学习目标

1，能够描述spring框架
2，能够理解spring的IOC的容器   
3，能够编写spring的IOC的入门案例
4，能够说出spring的bean标签的配置
5，能够理解 Bean的实例化方法
6，能够理解Bean的属性注入方法
7，能够理解复杂类型的属性注入



new 类名()

今天学习spring： 今天spring将我们带来创建对象，给对象赋予属性值。 



##  框架概述与Spring重要性

### **什么是框架？**

**框架是软件的半成品**，已经预先提供了一些功能，我们在框架的基础上做开发，框架已经有的功能可以直接使用。



### **框架的好处**

​	简化开发。

​	A. 简化开发，因为已经提供了一些通用性的功能的实现， 必须要在搭建好环境的情况下才能简化开发.

​	B. 框架是优秀的程序员写出的优秀的代码形成的工具类，具有可扩展性，稳定性更加好。 





### **三层架构与常见的框架**

![1561941552995](Spring-01.assets/1561941552995.png)

**Web层（表现层）**

- struts  

- springmvc

  

**Service层（业务层）**

- spring 



**Dao层（持久层）**

- hibernate
- mybatis
- spring data jpa 
- jdbctemplate



### **Spring重要性**



Spring是一个家族体系（spring的全家桶），可以完成JavaEE开发中几乎所有问题！！（Spring可能会一统Java开发的天下）



Spring项目官网：<https://spring.io/projects>



Spring家族常见的项目如下：

1）Spring  Framework (spring框架   IOC  AOP 事务管理，SpringMVC等)

2）Spring Security（权限校验）

3）Spring Data( 一统持久层天下  mysql，oracle，mongoDB，solr等 )

4）Spring Boot（简化Spring开发）

5）Spring Cloud（分布式开发）

6）Spring AMQP（整合RabbitMQ 消息队列）

........





## Spring（Framework）介绍

### **简介**

Spring （Framework）框架拥有7个核心模块，都是为JavaEE开发提供基础功能。



### **体系结构**

![1563325384580](Spring-01.assets/1563325384580.png)



1)  ==Spring Core （IOC容器）：创建对象，给对象属性赋值==   ***

2）==Spring AOP（面向切面编程）： 事务管理，自定义AOP编程==  ***   aspect   object 

3）Spring ORM ：Spring整合ORM框架（Hibernate/JPA）*

4)  Spring JDBC : 提供JdbcTemplate的，对Jdbc简单封装 * 

5）Spring Web： Spring整合Web（Servlet，Filter）*

6）==SpringMVC：Spring提供MVC表现层模块==  ***

7）Spring Context: 国际化，事件驱动编程

### 小结：

- **spring最核心的功能有哪些？**

  - spring core  核心模块： 创建对象，给对象赋予属性值
  - spring aop 面向切面编程： 解决事务管理，日志记录这些问题
  - springmvc:  web层的框架，用于取代servlet的。属于servlet的框架。

   



## 程序中解耦应用-工厂模式解耦

### **需求**

由于客户的数据库环境经常变化，希望更换数据库时候，只需要修改dao的代码，不需要修改service。



### **实现**



### 案例类图

![1604539093399](Spring-01.assets/1604539093399.png)





### UserDao接口：

```java
package com.itheima.dao;

public interface UserDao {

    public void findAll();
}

```



UserDaoMysqlImpl实现，这里假设项目需要连接MySQL数据库，所以先编写UserDaoMySQLImpl

```java
package com.itheima.dao.impl;

import com.itheima.dao.UserDao;

public class UserDaoMysqlImpl implements UserDao {
    @Override
    public void findAll() {
        System.out.println("mysql查询全部....");
    }
}

```



3）编写Service接口和实现



### UserService接口：

```java
package com.itheima.service;

import com.itheima.dao.UserDao;
import com.itheima.dao.impl.UserDaoMysqlImpl;

public interface UserService {

    public void findAll();


}

```



### UserServiceImpl实现类：

```java
package com.itheima.service.impl;

import com.itheima.dao.UserDao;
import com.itheima.dao.impl.UserDaoMysqlImpl;
import com.itheima.service.UserService;

public class UserServiceImpl implements UserService {

    private UserDao userDao = new UserDaoMysqlImpl();

    @Override
    public void findAll() {
        userDao.findAll();
    }
}

```



### 4）编写测试类

```java
package com.itheima.test;

import com.itheima.service.UserService;
import com.itheima.service.impl.UserServiceImpl;
import org.junit.Test;

public class AppTest {

    @Test
    public void test01(){
        UserService userService = new UserServiceImpl();
        userService.findAll();
    }
}

```

### 问题

这时，项目数据需要从mysql迁移到oracle，意味Dao实现类需要更新，但为了遵守==开闭原则==，我们采用新建UserDaoOracleImpl新的Dao实现类的方式进行扩展。同时，UserServiceImpl业务层需要修改代码来切换Dao实现：

注意：==开闭原则，是软件设计的一种方式，对修改关闭，对扩展/新增开放！==



Oracle版本的Dao实现

```java
package com.itheima.dao.impl;

import com.itheima.dao.UserDao;

public class UserDaoOracleImpl implements UserDao {

    @Override
    public void findAll() {
        System.out.println("oracle查询全部...");
    }
}

```



**==存在的问题==**

Service层通过硬编码方式切换Dao实现

```java
package com.itheima.service.impl;

import com.itheima.dao.UserDao;
import com.itheima.dao.impl.UserDaoMysqlImpl;
import com.itheima.dao.impl.UserDaoOracleImpl;
import com.itheima.service.UserService;

public class UserServiceImpl implements UserService {

//    private UserDao userDao = new UserDaoMysqlImpl();

    //UserServiceImpl 完全依赖了UserDaoMysqlImpl
    private UserDao userDao = new UserDaoOracleImpl(); //更换实现类的时候，我们又发现一个问题：我们的程序高耦合的问题
    //高耦合的问题的解决方案：使用工厂设计模式去解决耦合度的问题的。


    @Override
    public void findAll() {
        userDao.findAll();
    }
}

```

这时我们应该可以发现，如果每次Dao进行扩展，都去Service需要修改源代码切换到新的Dao实现，这种做法耦合度太高！！！

怎么呢？  可以使用**工厂模式**解决Service和Dao的耦合问题！！！



### 程序解耦解决方案

![1604538576302](Spring-01.assets/1604538576302.png)

**建立bean.properties，在properties文件先描述项目中所有需要创建的对象**

```
userDao=com.itheima.dao.impl.UserDaoOracleImpl
userService=com.itheima.service.impl.UserServiceImpl
```



**创建Beanfactory读取配置文件**

```java
package com.itheima.utils;

import sun.security.util.Resources;

import java.util.ResourceBundle;

public class BeanFactory {

    private  static  ResourceBundle resourceBundle;

    static{
        //读取配置文件
        resourceBundle = Resources.getBundle("bean");
    }


    public static Object getBean(String beanName){ //userDao
        try {
            //获取该接口对应的实现类的类全名
            String className = resourceBundle.getString(beanName);
            //利用反射技术创建该类的对象
            Object obj = Class.forName(className).newInstance();
            return obj;
        } catch (Exception e) {
           throw new  RuntimeException(e);
        }

    }
}

```





**改造的Service**

```java
package com.itheima.service.impl;

import com.itheima.dao.UserDao;
import com.itheima.dao.impl.UserDaoMysqlImpl;
import com.itheima.dao.impl.UserDaoOracleImpl;
import com.itheima.service.UserService;
import com.itheima.utils.BeanFactory;

public class UserServiceImpl implements UserService {

//    private UserDao userDao = new UserDaoMysqlImpl();

    //UserServiceImpl 完全依赖了UserDaoMysqlImpl
   // private UserDao userDao = new UserDaoOracleImpl(); //更换实现类的时候，我们又发现一个问题：我们的程序高耦合的问题
    //高耦合的问题的解决方案：使用工厂设计模式去解决耦合度的问题的。


    //工厂设计模式解决高耦合的问题
    private UserDao userDao = (UserDao) BeanFactory.getBean("userDao");


    @Override
    public void findAll() {
        userDao.findAll();
    }
}

```



## 程序中解耦应用-控制反转

### 什么是IOC和DI？****

**IOC，Inverse Of Control  控制反转**

1. 把创建对象的权力，由自己创建(new)改为工厂（IOC容器）来创建
2. 最终解决的是创建对象的耦合问题。



new 类名(); 创建对象的主动权在我们手上，我们把创建对象主动权交给了工厂，这个我们则称作为控制反转 。



**DI，Denpedency Injection 依赖注入**   

1. 把对象属性赋值的权力，由自己使用硬编码给对象赋值改为工厂（IOC容器）来完成

  2. 最终解决的是给=对象属性赋值==的耦合问题。






## SpringIOC容器入门案例

###  **步骤**

1.创建maven项目，导入spring-context依赖，导入junit依赖

2.编写User类

3.编写applicationContext.xml声明对象的创建

4.测试



### **实现**

**1.创建项目，导入spring-context依赖**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>springday01_02xml</artifactId>
    <version>1.0-SNAPSHOT</version>



    <dependencies>
        <!--spring核心依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
        <!--junit依赖-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>


    
</project>
```



**2.编写User类**

```java
package com.itheima.model;

public class User {
    
    public User() {
        System.out.println("User对象被创建了..构造器被调用..");
    }
}

```



**3.编写bean.xml(spring的核心配置文件)声明对象的创建**

**名称：**建议叫bean.xml 或 applicationContext.xml

**位置:**  通常放在项目类路径下，即 resources目录下

文件约束头，可以查询spring官方文档或者复制老师的



**最终bean.xml内容如下：**

![1568772302092](Spring-01.assets/1568772302092.png)

![1568772349601](Spring-01.assets/1568772349601.png)

![1568772452288](Spring-01.assets/1568772452288.png)



```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--bean标签： bean标签的作用就是帮助我们创建一个类的对象-->
    <bean id="user" class="com.itheima.model.User"/>
</beans>
```



**测试类内容如下：**

```java
package com.itheima.test;

import com.itheima.model.User;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppTest {

    @Test
    public void test01(){
        //1. 创建spring的容器，加载ApplicationContext.xml文件，然后创建的对象则全部会进入spring的容器中
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        //2. 从容器中根据对象的id值查找对应的对象
        User user = (User) context.getBean("user");
        System.out.println("得到的对象："+ user);
    }
}

```



### **小结**

**IOC创建对象的步骤?**

- 导入spring的依赖 spring-context

- 自定义一个类

- 编写spring核心配置文件： applicationContext.xml  <bean id =  class="">

- 编写测试类

  

## SpringIOC容器创建对象细节(bean标签常用的属性)

**bean标签配置细节**

![1563330494898](Spring-01.assets/1563330494898.png)



**代码**

1.bean.xml配置细节代码演示



```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--bean标签： bean标签的作用就是帮助我们创建一个类的对象-->
    <!--bean标签常用的属性：
            id： 对象在spring容器中的唯一标识，注意：不能同名
            class: 创建对象的完整类名
            name: 对象在spring容器中的名字作用与id一样，只不过name可以有多个名字
            scope: 创建对象的时候使用的模式（singleton 单例设计模式，prototype 多例模式）,默认情况是单例模式
            init-method: 创建对象后调用的方法,仅对于单例设计模式有效
            destroy-method : 销毁对象前对调用的方法，仅对于单例设计模式有效
            lazy-init: 单例设计模式是有饿汉、懒汉式，lazy-init是否为懒汉模式
    -->

    <bean id="user" name="user2,user3" scope="singleton" init-method="init" destroy-method="destroy" lazy-init="false"  class="com.itheima.model.User"/>
</beans>
```



2.对应的测试代码



```java
package com.itheima.test;

import com.itheima.model.User;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppTest {

    @Test
    public void test01(){
        //1. 创建spring的容器，加载ApplicationContext.xml文件，然后创建的对象则全部会进入spring的容器中
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        //2. 从容器中根据对象的id值查找对应的对象
      /*  User user1 = (User) context.getBean("user");
        User user2 = (User) context.getBean("user");
        System.out.println("得到的对象："+ user1);
        System.out.println("得到的对象："+ user2);*/

      //关闭容器
        context.close();

    }

}

```





### 小结

- **bean标签常用的属性,以及每个标签的作用?**
  	- id  : spring对象在spring容器中唯一标识
  	- class ：创建的对象的类全名
  	- name : 作用与id一致，只不过可以存在多个name
  	- init-method ： 创建对象后调用的方法
  	- destroy-method :  销毁对象前调用的方法
  	- scope : 指定spring创建对象的时候使用单例（singleton，默认）还是多例(prototype)
  	- lazy-init : 指定使用懒汉还是饿汉单例设计模式,默认是饿汉单例模式

> Spring默认是单例的，不要使用非静态的成员变量，否则会发生数据逻辑混乱。正因为单例所以不是线程安全的。
>
> ```
> package com.riemann.springbootdemo.controller;
> 
> import org.springframework.context.annotation.Scope;
> import org.springframework.stereotype.Controller;
> import org.springframework.web.bind.annotation.RequestMapping;
> 
> /**
>  * @author riemann
>  * @date 2019/07/29 22:56
>  */
> @Controller
> public class ScopeTestController {
> 
>     private int num = 0;
> 
>     @RequestMapping("/testScope")
>     public void testScope() {
>         System.out.println(++num);
>     }
> 
>     @RequestMapping("/testScope2")
>     public void testScope2() {
>         System.out.println(++num);
>     }
> 
> }
> ```
>
> 
>
> 我们首先访问 http://localhost:8080/testScope，得到的答案是1；然后我们再访问 http://localhost:8080/testScope2，得到的答案是 2。
>
> 
>
> **得到的不同的值，这是线程不安全的。**
>
> 
>
> 接下来我们再来给controller增加作用多例 @Scope("prototype")
>
> 
>
> ```
> package com.riemann.springbootdemo.controller;
> 
> import org.springframework.context.annotation.Scope;
> import org.springframework.stereotype.Controller;
> import org.springframework.web.bind.annotation.RequestMapping;
> 
> /**
>  * @author riemann
>  * @date 2019/07/29 22:56
>  */
> @Controller
> @Scope("prototype")
> public class ScopeTestController {
> 
>     private int num = 0;
> 
>     @RequestMapping("/testScope")
>     public void testScope() {
>         System.out.println(++num);
>     }
> 
>     @RequestMapping("/testScope2")
>     public void testScope2() {
>         System.out.println(++num);
>     }
> 
> }
> ```
>
> 
>
> 我们依旧首先访问 http://localhost:8080/testScope，得到的答案是1；然后我们再访问 http://localhost:8080/testScope2，得到的答案还是 1。
>
> **解决方案**
>
> 1. 不要在controller中定义成员变量。
>
> 2. 万一必须要定义一个非静态成员变量时候，则通过注解@Scope(“prototype”)，将其设置为多例模式。
>
> 3. 在Controller中使用ThreadLocal变量

## SpringIOC容器(几种方式【了解】

### **目标**

1. 容器接口结构图
2. 代码实现不同的方式创建容器

### **容器接口结构图**

![1563333342298](Spring-01.assets/1563333342298.png)





### **代码**



```java
package com.itheima.test;

import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;

public class AppTest2 {

    @Test
    public void test01(){
        /*
            ApplicationContext：spring容器的顶级接口(ctrl+h快捷键可以查看该类的体系，向下去查看)
                ----------| ClsspathXmlApplicationContext  使用类路径加载配置文件
                ----------| FileSystemXmlApplicationContext  使用绝对路径加载配置文件(不会使用的)
                ----------| AnnotationConfigApplicationContext  加载注解配置类的（后天演示去看）

         */
        ApplicationContext context = new FileSystemXmlApplicationContext("H:/applicationContext.xml");
        Object user = context.getBean("user");
        System.out.println("对象："+user);

    }

}

```



### **小结**

- **有几种类型的容器**

  - ClasspathXmlApplicationContext  类路径加载配置文件
  - FileSystemXmlApplicationContext  绝对路径加载配置文件 
  - AnnotationConfigApplicationContext  加载注解配置类

  

  

##  SpringIOC容器创建对象的三种方式

### **目标**

掌握创建对象的三种方式：

方式1：默认无参数构造函数创建对象（推荐方式）

方式2：工厂类的静态方法创建对象 

方式3：工厂类的实例方法创建对象 



### **默认无参数构造函数创建对象（最常用）**

步骤：

1. 编写Person类

2. 编写BeanFactory工厂类 

3. 配置bean_ioc.xml
4. 测试



#### 实现：

1. 编写Person类

```java
package com.itheima.model;

public class Person {

    public Person() {
        System.out.println("Person类被创建对象了，构造器被调用了...");
    }
}

```

2. 编写BeanFactory工厂类 

```java
package com.itheima.utils;

import com.itheima.model.Person;

public class BeanFactory {

    //静态的方法创建对象
    public static Person getPerson(){
        return new Person();
    }


    //实例方法创建对象
    public Person getPersonInstance(){
        return new Person();
    }
}

```

3. 在resources目录下建立bean_ioc.xml配置

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
<!--
    目标：学习创建spring对象的三种方式
        1. 利用无参的构造器创建对象(推荐使用)
        2. 调用工厂静态的方法创建对象(了解)
        3. 调用工厂实例方法创建对象（了解）-->

    <!--1. 利用无参的构造器创建对象(推荐使用)-->
    <bean id="p1" class="com.itheima.model.Person"/>

    <!--2. 调用工厂静态的方法创建对象(了解)
            class： 编写的的静态方法所属的类（工厂类）
            factory-method: 创建对象的静态方法
      -->
    <bean id="p2" class="com.itheima.utils.BeanFactory" factory-method="getPerson"/>


    <!--3. 调用工厂实例方法创建对象（了解）
            1. 创建工厂的对象
            2. 引用工厂对象调用实例方法才能创建对象
    -->
    <!--1. 创建工厂的对象-->
    <bean id="factory" class="com.itheima.utils.BeanFactory"/>
    <!--创建Person对象
            factory-bean: 引用工厂对象的id值
    -->
    <bean id="p3" factory-bean="factory" factory-method="getPersonInstance"/>
</beans>
```





**3.测试**

```java
package com.itheima.test;

import com.itheima.model.Person;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;

public class AppTest3 {

    @Test
    public void test01(){
        ApplicationContext context = new ClassPathXmlApplicationContext("bean_ioc.xml");
        Person person = (Person) context.getBean("p3");
        System.out.println("对象："+person);

    }

}

```



### **小结**

1. **创建对象的三种方式**

   - 无参构造器创建对象
   - 工厂静态的方法创建对象
   - 工厂实例的方法创建对象








## SpringIOC容器依赖注入A-带参数构造函数

### **什么是依赖注入(denpendices injection)？**

所谓DI, 即依赖注入，关注的就是怎么==给对象的属性赋值==





### spring给对象赋予属性值（依赖注入）的方式

- 利用带参构造方法给属性赋值

- 利用对象的setter方法给属性赋值

- p空间利用setter方法给属性赋值

  



### **构造器赋值**

步骤：

1.在Person类添加属性及带参数的构造方式

2.配置bean_di.xml

3.测试



### 实现：

**1.创建新的Person类，在Person类添加属性及其带参数的构造方式**

```java
package com.itheima.model;

public class Person {

    private int id;

    private String name;

    private String sex;

    public Person(int id, String name, String sex) {
        this.id = id;
        this.name = name;
        this.sex = sex;
    }

    public Person(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public Person() {
    }

    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                '}';
    }
}

```





**2.创建配置bean.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--学习目标：学习di，就是学习创建对象之后，如何给对象的属性赋值
        给对象的属性赋值的方式：
                1. 方式一：  利用带参构造器赋值
                2.方式二：  利用setter方法赋值
                3. 方式三： p空间利用setter方法赋值
    -->
    <!--1. 方式一：  利用带参构造器赋值-->
    <bean id="p1" class="com.itheima.model.Person">
        <!--调用带参的构造器需要使用constructor-arg标签
            name指定构造器的形参的名字。
        -->
        <constructor-arg name="id" value="110"/>
        <constructor-arg name="name" value="狗娃"/>
        <constructor-arg name="sex" value="女"/>
    </bean>

    <bean id="p2" class="com.itheima.model.Person">
        <!--调用带参的构造器需要使用constructor-arg标签
            index：指定形参索引值
        -->
        <constructor-arg index="1" value="狗剩"/>
        <constructor-arg index="0" value="220"/>
    </bean>


    <bean id="p3" class="com.itheima.model.Person">
        <!--调用带参的构造器需要使用constructor-arg标签
            按照构造器的形参顺序赋值
        -->
        <constructor-arg  value="330"/>
        <constructor-arg  value="铁蛋"/>
        <constructor-arg  value="男"/>
    </bean>



</beans>
```



**3.测试**

```java
package com.itheima.test;

import com.itheima.model.Person;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppTest4 {

    @Test
    public void test01(){
        ApplicationContext context = new ClassPathXmlApplicationContext("bean_di.xml");
        Person person = (Person) context.getBean("p3");
        System.out.println("对象："+person);

    }

}

```



### 小结

- 使用构造方法注入属性值需要使用哪个标签

  - \<constructor-arg name=构造器的形参名字 index=形参的索引值   value=值/>






##  SpringIOC容器依赖注入set方法（最常见）

### 步骤

1.在Person类提供setter方法

2.配置bean_di.xml

3.测试



### 实现

**1.在Person类为属性提供setter方法**

```java
package com.itheima.model;

public class Person {

    private int id;

    private String name;

    private String sex;

    public Person(int id, String name, String sex) {
        this.id = id;
        this.name = name;
        this.sex = sex;
    }

    public Person(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public Person() {
    }


    public void setId(int id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                '}';
    }
}

```



**2.配置bean_di.xml**

```xml
   <!--2.方式二：  利用setter方法赋值-->
    <bean id="p4" class="com.itheima.model.Person">
        <!--利用setter方法设置需要使用property标签-->
        <property name="id" value="440"/>
        <property name="name" value="川建国"/>
        <property name="sex" value="男"/>
    </bean>

```



**3.测试**

```java
package com.itheima.test;

import com.itheima.model.Person;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppTest4 {

    @Test
    public void test01(){
        ApplicationContext context = new ClassPathXmlApplicationContext("bean_di.xml");
        Person person = (Person) context.getBean("p4");
        System.out.println("对象："+person);

    }

}

```



### **小结**

- spring认为哪些一个对象存在属性的必须条件是什么？ 使用set方法注入属性值使用哪个标签

  - 必须存在setter的方法
  - \<property>

  



## SpringIOC容器依赖注入p名称空间

### **分析**

p名称空间注入，本质也是setter方法注入，p名称空间就是setter方法的简化版



### **实现**

#### 步骤：

1.Person类也需要为属性提供setter方法

2.配置bean_di.xml

3.测试



#### 实现：

1.Person类也需要为属性提供setter方法

```java
package com.itheima.model;

public class Person {

    private int id;

    private String name;

    private String sex;

    public Person(int id, String name, String sex) {
        this.id = id;
        this.name = name;
        this.sex = sex;
    }

    public Person(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public Person() {
    }


    public void setId(int id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return "Person{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                '}';
    }
}

```





### 2.配置bean_di.xml

**先导入p名称空间**   alt+enter

![1563337640679](Spring-01.assets/1563337640679.png)



然后使用p名称空间给对应属性注入内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--学习目标：学习di，就是学习创建对象之后，如何给对象的属性赋值
        给对象的属性赋值的方式：
                1. 方式一：  利用带参构造器赋值
                2.方式二：  利用setter方法赋值
                3. 方式三： p空间利用setter方法赋值
    -->
   

    <!--3. 方式三： p空间利用setter方法赋值-->
    <bean id="p5" class="com.itheima.model.Person" p:id="550" p:name="拜登" p:sex="妖"/>


</beans>
```







3.测试

```java
package com.itheima.test;

import com.itheima.model.Person;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppTest4 {

    @Test
    public void test01(){
        ApplicationContext context = new ClassPathXmlApplicationContext("bean_di.xml");
        Person person = (Person) context.getBean("p5");
        System.out.println("对象："+person);

    }

}

```



### **小结**

- **给对象的属性注入值的方式：**

  - 利用带参的构造方法
  - 利用stter的方法，property标签
  - 利用setter的方法，利用p空间。

  



## SpringIOC容器给集合属性赋值



### 步骤：

1.在Person2类提供一些集合类型属性

2.配置bean_di.xml

3.测试



### 实现：

**1.在Person2类提供一些集合类型**

```java
package com.itheima.model;

import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.Set;

public class Person2 {

    private List<String> list;

    private Set<String> set;

    private Map<String,String> map;

    private String[] arr;

    public void setList(List<String> list) {
        this.list = list;
    }

    public void setSet(Set<String> set) {
        this.set = set;
    }

    public void setMap(Map<String, String> map) {
        this.map = map;
    }

    public void setArr(String[] arr) {
        this.arr = arr;
    }

    @Override
    public String toString() {
        return "Person2{" +
                "list=" + list +
                ", set=" + set +
                ", map=" + map +
                ", arr=" + Arrays.toString(arr) +
                '}';
    }
}

```



**2.配置bean_di.xml**

```xml
 <!--给对象的集合属性赋值-->
    <bean id="p6" class="com.itheima.model.Person2">
        <property name="list">
            <list> <!--list类型的属性对应的是list标签-->
                <value>习大大</value>
                <value>川建国</value>
                <value>拜登</value>
            </list>
        </property>
        <property name="set">
            <set> <!--set类型对应的标签-->
                <value>彭丽媛</value>
                <value>伊万卡</value>
                <value>乔碧萝</value>
            </set>
        </property>
        <property name="map">
            <!--map类型的属性对应map标签-->
            <map>
                <entry key="小马" value="凤姐"/>
                <entry key="一凡" value="红牛"/>
                <entry key="王成" value="大长腿"/>
            </map>
        </property>
        
        <property name="arr">
            <!--数组类型-->
            <array>
                <value>1</value>
                <value>2</value>
                <value>3</value>
            </array>
        </property>


    </bean>

```



**3.测试**

```java
package com.itheima.test;

import com.itheima.model.Person;
import com.itheima.model.Person2;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppTest5 {

    @Test
    public void test01(){
        ApplicationContext context = new ClassPathXmlApplicationContext("bean_di.xml");
        Person2 person = (Person2) context.getBean("p6");
        System.out.println("对象："+person);

    }

}

```



### 总结

- **属性如果是list、set、map、数组分别使用哪些标签设置属性值?**
  - list
  - set
  - map
  - array



## 基于注解的 IOC 配置（一）环境搭建

### **目标**

实现基于注解的ioc容器。



### 步骤：

1.创建项目，导入spring-context依赖

2.编写User实体类

3.在User类使用IOC注解创建对象

4.配置bean.xml开启IOC注解扫描

5.测试



### 实现：

1.创建项目，导入spring-context依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>springday01_03ann</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!--spring核心包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>

        <!--junit测试包-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>

    
</project>
```



**2.编写User实体类**

```java
package com.itheima.model;

import org.springframework.stereotype.Component;
/*
@Component注解的作用： 相当于bean标签的一样，创建该类的对象。
注意： 使用Component注解的时候，如果没有指定对象在容器中的名字，默认的名字就是当前的类名，但是首字母要小写。User---user
 */
@Component("user1") //user1就是该对象在spring容器存储的名字
public class User {

    public User() {
        System.out.println("User无参的构造器被调用了，user对象被创建了..");
    }
}

```





**4.配置bean.xml开启IOC注解扫描（***）**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--指定spring 的扫描的包,开启包扫描
        base-package: 指定要扫描的包，包括指定包的子孙包
    -->
    <context:component-scan base-package="com.itheima.model"/>


</beans>
```





**5.测试**

```java
package com.itheima.test;

import com.itheima.model.User;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppTest01 {

    @Test
    public void test01(){
        //1. 创建容器
        ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
        //2. 从容器中查找对象
        User user = (User) context.getBean("user1");
        System.out.println("user对象："+ user);

    }
}

```



### **小结**

-  使用注解开发我们需要使用哪些进行哪些操作？ 
   -  导入依赖 spring-context
   -  编写实体类,@Component
   -  在spring核心配置文件上开启注解扫描： <context:component-scan base-package="扫描包名">
   -  测试







##  基于注解的 IOC 配置（二）用于创建对象的注解

### **需求**

<font color=red>@Component</font>    创建对象加入容器。举例：工具类。

**@Controller**： 创建对象加入容器。同@Component一样。一般用于表现层的注解。
**@Service**：       创建对象加入容器。同@Component一样。一般用于业务层的注解。
**@Repository**：创建对象加入容器。同@Component一样。 一般用于持久层的注解。



### **实现**



```java
package com.itheima.model;

import org.springframework.stereotype.Component;
import org.springframework.stereotype.Controller;
import org.springframework.stereotype.Repository;
import org.springframework.stereotype.Service;

/*
@Component注解的作用： 相当于bean标签的一样，创建该类的对象。
注意： 使用Component注解的时候，如果没有指定对象在容器中的名字，默认的名字就是当前的类名，但是首字母要小写。User---user
 */
//@Component("user1") //user1就是该对象在spring容器存储的名字
//@Repository("user1")
//@Service("user1")
@Controller("user1")
public class User {

    public User() {
        System.out.println("User无参的构造器被调用了，user对象被创建了..");
    }
}

```





## 基于注解的 IOC 配置（三）注入数据@Autowired修饰字段

### **@Autowired注解说明**

1. 作用： 注入数据，从IOC容器获取另一个对象来注入。 


### 实现

**1.编写Teacher的实体类**

```java
package com.itheima.model;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class Teacher {

    /*
         @Autowired作用：
                1. 从spring容器中查找类型与该属性匹配的值注入

          @Autowired查找数据的规则：
            1. 如果容器中只有一个与该属性匹配类型的值，那么直接注入。
            2. 如果spring容器中出现了多个与该属性匹配的值，那么根据id与属性名进行匹配。
     */
    @Autowired
    private  int id;

    @Autowired
    private String name;

    @Override
    public String toString() {
        return "Teacher{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}

```



3.配置bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--指定spring 的扫描的包,开启包扫描
        base-package: 指定要扫描的包，包括指定包的子孙包
    -->
    <context:component-scan base-package="com.itheima.model"/>


    <!--创建一个int类型的值-->
    <bean id="num1" class="java.lang.Integer">
        <constructor-arg value="110"/>
    </bean>

    <bean id="id"  class="java.lang.Integer">
        <constructor-arg value="220"/>
    </bean>

    <!--创建一个String类型的值-->
    <bean class="java.lang.String">
        <constructor-arg value="安安"/>
    </bean>

</beans>
```



4.测试

```java
package com.itheima.test;

import com.itheima.model.Teacher;
import com.itheima.model.User;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppTest02 {

    @Test
    public void test01(){
        //1. 创建容器
        ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
        //2. 从容器中查找对象
        Teacher teacher = (Teacher) context.getBean("teacher");
        System.out.println("对象："+ teacher);

    }
}

```



### 小结

 - **AutoWired使用前提：**
   - spring的容器必须存在与该属性类型匹配的值
 - @AutoWired自动注入的规则：
   - 如果spring容器中只有一个与属性类型匹配的值，那么会直接的注入
     	- 如果spring容器存在两个或者两个以上的与属性类型匹配的值，那么默认根据id值与属性名匹配进行注入。



## 基于注解的 IOC 配置（四）注入数据B-@Autowired修饰方法

### **说明**

@Autowired修饰方法时，作用与修饰字段基本一样。



### 步骤：

1.编写Teacher的实体类，把@AutoWired修饰id的字段拿掉

2.在Teacher实体添加方法，在方法上面使用@Autowired修饰setId方法

3.配置bean.xml

4.测试



### 实现：

1.修改Teacher的实体类



```java
package com.itheima.model;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class Teacher {

    /*
         @Autowired作用：
                1. 从spring容器中查找类型与该属性匹配的值注入

          @Autowired查找数据的规则：
            1. 如果容器中只有一个与该属性匹配类型的值，那么直接注入。
            2. 如果spring容器中出现了多个与该属性匹配的值，那么根据id与属性名进行匹配。
     */
  //  @Autowired
    private  int id;

    @Autowired
    private String name;

    /*
          @Autowired修饰一个方法的时候：
                1. 该方法会自动被调用。
                2. spring会从容器中查找与该形参类型匹配的值给形参注入。
     */
    @Autowired
    public void setId(int id) {
        this.id = id;
    }

    @Override
    public String toString() {
        return "Teacher{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}

```





3.配置bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--指定spring 的扫描的包,开启包扫描
        base-package: 指定要扫描的包，包括指定包的子孙包
    -->
    <context:component-scan base-package="com.itheima.model"/>


    <!--创建一个int类型的值-->
    <bean id="num1" class="java.lang.Integer">
        <constructor-arg value="110"/>
    </bean>

    <bean id="id"  class="java.lang.Integer">
        <constructor-arg value="220"/>
    </bean>

    <!--创建一个String类型的值-->
    <bean class="java.lang.String">
        <constructor-arg value="安安"/>
    </bean>

</beans>
```



4.测试

```java
package com.itheima.test;

import com.itheima.model.Teacher;
import com.itheima.model.User;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppTest02 {

    @Test
    public void test01(){
        //1. 创建容器
        ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
        //2. 从容器中查找对象
        Teacher teacher = (Teacher) context.getBean("teacher");
        System.out.println("对象："+ teacher);

    }
}

```





##  基于注解的 IOC 配置（四）注入数据C-@Qualifier

### **@Qualifier注解说明**

1. 如果想让@Autowired只根据指定的对象（别名）名称实现依赖注入，要配置@Qualifier注解
2. @Qualifier 通常要配合@Autowired一起使用。（在纯注解开发中可以单独使用@Qualifier）



### 步骤：

1.编写Teacher2实体类

2.在Teacher2在字段上，同时使用@Autorwird与@Qualifier注解

3.配置bean.xml

4.测试



### 实现：



1.在Teacher2在字段上，同时使用@Autorwird与@Qualifier注解

```java
package com.itheima.model;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

@Component
public class Teacher2 {

    /*
     @Qualifier() 指定名称从spring容器中查找数据， @Qualifier注解不会单独去使用，一般都会配合@AutoWired注解一起去使用的。
     */
    @Autowired
    @Qualifier("num1")
    private  int id;

    @Autowired
    private String name;

    @Override
    public String toString() {
        return "Teacher{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}

```



3.配置bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--指定spring 的扫描的包,开启包扫描
        base-package: 指定要扫描的包，包括指定包的子孙包
    -->
    <context:component-scan base-package="com.itheima.model"/>


    <!--创建一个int类型的值-->
    <bean id="num1" class="java.lang.Integer">
        <constructor-arg value="110"/>
    </bean>

    <bean id="num2"  class="java.lang.Integer">
        <constructor-arg value="220"/>
    </bean>

    <!--创建一个String类型的值-->
    <bean class="java.lang.String">
        <constructor-arg value="安安"/>
    </bean>

</beans>
```



4.测试

```java
package com.itheima.test;

import com.itheima.model.Teacher;
import com.itheima.model.Teacher2;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppTest03 {

    @Test
    public void test01(){
        //1. 创建容器
        ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
        //2. 从容器中查找对象
        Teacher2 teacher = (Teacher2) context.getBean("teacher2");
        System.out.println("对象："+ teacher);

    }
}

```







##  基于注解的 IOC 配置（六）注入数据 @Value

### **@Value注解说明**

1. 直接给简单类型的字段赋值（相当于property标签的value属性）
2. 获取配置文件值。 （在纯注解开发时候使用）



### 步骤：

1.编写User2实体

2.在User2使用@Value注解

3.配置bean.xml

4.测试



### 实现：

1.编写User2实体

```java
package com.itheima.model;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;
/*
@Value注解的作用：
    1. 直接给简单类型的属性赋值
    2. 加载配置文件的数据给属性赋值


加载properties文件的方式：
    1. xml方式加载   <context:property-placeholder location="db.properties"/>
    2. 注解的方式加载  @PropertySource("classpath:db.properties")


注意： spring容易在创建的时候会往容器中存储的一个键为username的属性，值就是计算机名字.
       以后为了避免这种冲突，我们数据库的配置文件我们一般都加上jdbc作为前缀

 */
@Component
//@PropertySource("classpath:db.properties")
public class User2 {

    @Value("330")
    private int id;

    @Value("马云")
    private String name;

    @Value("56")
    private int age;

    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.driverClass}")
    private String driverClass;
    @Value("${jdbc.username}")
    private String username;
    @Value("${jdbc.password}")
    private String password;


    @Override
    public String toString() {
        return "User2{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", url='" + url + '\'' +
                ", driverClass='" + driverClass + '\'' +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}

```





2. properties文件

   ```properties
   jdbc.url=jdbc:mysql:///travel
   jdbc.driverClass=com.mysql.jdbc.Driver
   jdbc.username=root
   jdbc.password=root
   ```

   





3. 配置bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--指定spring 的扫描的包,开启包扫描
        base-package: 指定要扫描的包，包括指定包的子孙包
    -->
    <context:component-scan base-package="com.itheima.model"/>

    <context:property-placeholder location="db.properties"/>


    <!--创建一个int类型的值-->
    <bean id="num1" class="java.lang.Integer">
        <constructor-arg value="110"/>
    </bean>

    <bean id="num2"  class="java.lang.Integer">
        <constructor-arg value="220"/>
    </bean>

    <!--创建一个String类型的值-->
    <bean class="java.lang.String">
        <constructor-arg value="安安"/>
    </bean>

</beans>
```







**4.测试**

```java
package com.itheima.test;

import com.itheima.model.Teacher2;
import com.itheima.model.User2;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppTest04 {

    @Test
    public void test01(){
        //1. 创建容器
        ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
        //2. 从容器中查找对象
        User2 user2 = (User2) context.getBean("user2");
        System.out.println("对象："+ user2);

    }
}

```



### 小结

 -  @Value注解作用：
    	
    给属性注入值


​    	

- 加载配置文件：

  - 方式一： 使用注解加载： @PropertiesSource  注解加载
  - 方式二： 使用标签加载。 <!--加载配置文件--><context:property-placeholder location="classpath:jdbc.properties"/>

  

##  基于注解的 IOC 配置对象范围与生命周期相关注解

### **注解说明**

@Scope

@PostConstruct

@PreDestroy

@Lazy

```xml
<bean id=""  class="" 
      scope=""                使用@Scope注解取代
      init-method=""          使用@PostConstruct注解取代
      destroy-method=""       使用@PreDestroy注解取代
      lazy-init=""            使用@Lazy注解取代
/>
```



### 步骤：

1.编写User3实体类

2.在User3实体添加相应的注解

3.配置bean.xml

4.测试



### 实现：

1.编写User3实体类



```java
package com.itheima.model;

import org.springframework.context.annotation.Lazy;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

/*
@Scope  指定是否为单例还是多例模式

@PostConstruct  创建调用后调用的方法

@PreDestroy  销毁对象前调用的方法

@Lazy  是否为懒汉单例设计模式
 */
@Component
@Scope("singleton")
@Lazy(true)
public class User3 {

    @PostConstruct
    public void init(){
        System.out.println("init方法被调用，user对象被创建了");
    }

    @PreDestroy
    public void destroy(){
        System.out.println("destroy方法被调用，user对象马上要被销毁了");
    }


    public User3() {
        System.out.println("User3无参的构造器被调用了，");
    }
}

```





3.配置bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--指定spring 的扫描的包,开启包扫描
        base-package: 指定要扫描的包，包括指定包的子孙包
    -->
    <context:component-scan base-package="com.itheima.model"/>

    <context:property-placeholder location="classpath:db.properties"/>


    <!--创建一个int类型的值-->
    <bean id="num1" class="java.lang.Integer">
        <constructor-arg value="110"/>
    </bean>

    <bean id="num2"  class="java.lang.Integer">
        <constructor-arg value="220"/>
    </bean>

    <!--创建一个String类型的值-->
    <bean class="java.lang.String">
        <constructor-arg value="安安"/>
    </bean>

</beans>
```



4.测试



```java
package com.itheima.test;

import com.itheima.model.User2;
import com.itheima.model.User3;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppTest05 {

    @Test
    public void test01(){
        //1. 创建容器
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:bean.xml");
        //2. 从容器中查找对象
     /*   User3 user3 = (User3) context.getBean("user3");
        System.out.println("对象："+ user3);*/
        context.close();

    }
}

```













