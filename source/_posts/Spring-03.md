---
title: Spring(03)
tags:
  - 笔记
  - ssm框架
  - Spring
  - 控制反转
  - IOC
  - 依赖注入
  - DI
  - AOP
  - 面向切面
  - 事务
  - 声明式事务
  - 注解
  - 编程式事务
categories:
  - ssm框架
date: 2020-11-08 21:05:45
---

## 01.什么是AOP

所谓面向切面编程，就是我希望在目标对象（有无接口都可以）上扩展原有代码的功能，但可以在不修改源码的基础上来完成！！！



### 1.1 AOP的应用场景



需要把项目中的通用的**非核心**业务逻辑，从核心业务逻辑中进行抽离，这时就是AOP的应用场景



例如：

日志处理

事务管理

权限控制（粗粒度权限控制,），后面会学习专业的权限控制框架



### 1.2小结

- **AOP编程是什么？AOP编程有哪些应用场景？**
  	- AOP面向切面编程
   - 应用场景：
     - 日志管理
     - 事务管理



## 02 . Aop编程专业术语

### **2.1目标**

什么是连接点（JointPoint），切入点（Pointcut），通知（Advice），切面（Aspect）目标对象（Target），代理（Proxy）



### **2.2概念**

![1604796614101](Spring-03.assets/1604796614101.png)



### 2.3 小结

 - 什么是连接点、切入点、通知、切面，代理对象、目标对象？

   - 连接点 :  被代理的所有方法
   - 切入点:  被代理对象的被增强方法
   - 通知 : 增强的代码
   - 切面 :  切入点+通知
   - 目标对象 : 被代理对象
   - 代理对象: 代理对象





## 03. Aop编程基于XML的AOP配置 （重点）

### **3.1 需求**

采用输出日志作为示例。 访问service方法自动记录日志

​                  

### 3.2 步骤

1.创建项目，导入spring-aop,aspectjweare依赖

2.创建UserService接口和实现

3.创建切面类（增强的代码）

4.配置切面

5.测试



### 3.3 实现

1.创建springday03_01aop_xml项目，导入spring-aop和aspectjweare依赖



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>day44_spring03_aop01</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!--spring 核心包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>

        <!--使用aop编程-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>

        <!--因为aop编程需要使用切入点表达式，所以需要导入切入点表达式-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.13</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>

    </dependencies>
    
</project>

```



2.创建UserService接口和实现（目标对象）

```java
package com.itheima.service;

public interface UserService {

    public void  save();
}

```

```java
package com.itheima.service.impl;

import com.itheima.service.UserService;

//目标对象：被代理对象
public class UserServiceImpl implements UserService{

    @Override
    public void save() {
        System.out.println("保存用户...");
    }
}

```



3.创建LogAspect切面类， 这里说是一个切面类，但是本质上该类还不是一个切面类，因为一个切面=切入点+通知，这里仅仅编写了通知的代码。 

```java
package com.itheima.utils;
/*
创建LogAspect切面类， 这里说是一个切面类，但是本质上该类还不是一个切面类，
因为一个切面=切入点+通知，这里仅仅编写了通知的代码。


 注意： 该类仅仅编写通知的代码而已
 */
public class LogAspect {

    public void log(){
        System.out.println("记录日志..");
    }
}

```



4.创建bean.xml文件，配置切面类

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--1. 创建目标对象-->
    <bean id="userService" class="com.itheima.service.impl.UserServiceImpl"/>

    <!--2.创建通知类的对象-->
    <bean id="logAspect" class="com.itheima.utils.LogAspect"/>


    <!--3. 配置一个切面 切面 = 通知+ 切入点 -->
    <aop:config>
        <!--切入点: 切入点表达式就是指明那个类的哪个方法需要被增强-->
        <aop:pointcut id="pt" expression="execution(* com.itheima.service.impl.UserServiceImpl.save())"/>
        <!--把通知与切入点融合一起形成一个切面-->
        <aop:aspect ref="logAspect">
            <!--method:指定使用通知类的哪个方法去增强-->
            <aop:before method="log" pointcut-ref="pt"/>
        </aop:aspect>
    </aop:config>

</beans>
```



5.测试

```java
package com.itheima.test;

import com.itheima.service.UserService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:bean.xml")
public class AppTest {

    @Autowired
    private UserService userService; //注意： 一旦有了切面编程，这里注入的是UserServiceImpl的代理对象

    @Test
    public void test01(){
        userService.save();
    }
}

```



### **3.4 小结**

   - AOP编程有哪些步骤？

     -    创建被代理对象
     -    创建一个通知类的对象
     -    配置切面
      -    \<aop:config> 
       -    \<aop:pointcut> 切入点
        -    \<aop:aspect>
             - \<aop:before>

     


## 04. Aop编程切入点表达式【重点】

### **4.1 目标**

掌握切入点表达式语法



### 4.2 切入点表达式作用

用于声明要在目标对象的哪些方法进行切入通知！！

语法： execution( 修饰符 返回值类型 包路径.类名称.方法名称( 参数列表类型 )  )



### 4.3 步骤

1.复制bean.xml，改为bean_pointcut.xml

2.修改bean_pointcut.xml

3.修改测试类加载的配置文件



### 4.4 实现

1. 复制bean.xml，改为bean_pointcut.xml

2. 修改bean_pointcut.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--1. 创建目标对象-->
    <bean id="userService" class="com.itheima.service.impl.UserServiceImpl"/>

    <!--2.创建通知类的对象-->
    <bean id="logAspect" class="com.itheima.utils.LogAspect"/>


    <!--3. 配置一个切面 切面 = 通知+ 切入点 -->
    <aop:config>
        <!--
            切入点表达式的作用： 指定哪些类的哪些方法需要增强

            切入点表达式的语法： execution(【权限修饰符】 方法返回值类型 包名.类名.方法名(形式参数))


            1. 权限修饰符：
                   a. 省略不写
                   b. 精准匹配
            2. 方法的返回值类型
                    a.精准匹配  void 、 int...
                    b.模糊匹配 *
            3. 包名
                    a.精准匹配  比如：com.itheima.service
                    b.一级路径模糊匹配  *
                    c. 多级路径模糊匹配  ..
            4. 类名
                    a.精准匹配
                    b.模糊匹配 *
            5. 方法名
                    a.精准匹配
                    b.模糊匹配 *

             6.形参列表
                     a.精准匹配
                    b.模糊匹配 ..
        -->
        <aop:pointcut id="pt" expression="execution(public * com..*Impl.*(..))"/>
        <!--把通知与切入点融合一起形成一个切面-->
        <aop:aspect ref="logAspect">
            <!--method:指定使用通知类的哪个方法去增强-->
            <aop:before method="log" pointcut-ref="pt"/>
        </aop:aspect>
    </aop:config>

</beans>
```

3.修改测试类加载的配置文件

```java
package com.itheima.test;

import com.itheima.service.UserService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:bean_pointcut.xml")
public class AppTest {

    @Autowired
    private UserService userService; //注意： 一旦有了切面编程，这里注入的是UserServiceImpl的代理对象

    @Test
    public void test01(){
        userService.save(12);
    }
}

```



### 4.5 小结

1. **切入点表达式的作用:** 指定目标对象的哪些方法需要被增强

   

2. **切入点表达式的语法：** execution(权限修饰符 返回值类型 包名.类名.方法名(形参列表))

   

3. **切入点表达式每一个成分如何编写:**

   	- 权限修饰符： 精准匹配、省略不写
   			- 返回值类型： *  精准匹配
   	 	 			- 包名：  精准  *   ..
   	 		 			- 类名&方法名： 精准   *
   	 			 			- 形参列表：精准匹配  .. 

   ​		




## 05. Aop编程通知类型 A 介绍

### 5.1 通知的类型

前置通知：在执行目标对象方法之前执行  aop:before

后置通知：在执行目标对象方法之后执行 (异常不执行) aop:after-returning

异常通知：在执行目标对象方法里面发生异常时候执行 aop:after-throwing

最终通知：在执行完目标对象方法后始终执行的方法就是最终通知(异常也执行) aop:after



try{

​      前置通知

​       执行目标方法      

​      后置通知

}catch(Exception e){

​      异常通知

 }finally{

​      最终通知

}



环绕通知：自定义如何执行目标方法  aop:around



## 06. Aop编程通知类型-前置、后置、异常、最终通知演示

### **6.1 目标**

1. 会定义各种通知类型（前置、后置、异常、最终）
2. 掌握各种通知类型的执行顺序



### 6.2 步骤：

1.复制springday03_01aop_xml，改为springday03_02aop_xml

2.在LogAspect切面（通知）类添加各种通知方法

3.bean.xml配置各种通知

4.测试



### 6.3 实现

1.复制springday03_01aop_xml，改为springday03_02aop_xml

![1604800819490](Spring-03.assets/1604800819490.png)



2.在LogAspect切面类添加各种通知方法

```java
package com.itheima.utils;
/*
创建LogAspect切面类， 这里说是一个切面类，但是本质上该类还不是一个切面类，
因为一个切面=切入点+通知，这里仅仅编写了通知的代码。


 注意： 该类仅仅编写通知的代码而已
 */
//通知类
public class LogAspect {

    //前置通知
    public void before(){
        System.out.println("====前置通知===");
    }

    //后置通知
    public void afterReturning(){
        System.out.println("====后置通知===");
    }


    //异常通知
    public void afterThrowing(){
        System.out.println("====异常通知===");
    }


    //最终通知
    public void after(){
        System.out.println("====最终通知===");
    }
}

```



3.bean.xml配置各种通知

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--1. 创建目标对象-->
    <bean id="userService" class="com.itheima.service.impl.UserServiceImpl"/>

    <!--2.创建通知类的对象-->
    <bean id="logAspect" class="com.itheima.utils.LogAspect"/>


    <!--3. 配置一个切面 切面 = 通知+ 切入点 -->
    <aop:config>
        <!--切入点: 切入点表达式就是指明那个类的哪个方法需要被增强-->
        <aop:pointcut id="pt" expression="execution(* com.itheima.service.impl.UserServiceImpl.*(..))"/>
        <!--把通知与切入点融合一起形成一个切面-->
        <aop:aspect ref="logAspect">
            <!--method:指定使用通知类的哪个方法去增强-->
            <aop:before method="before" pointcut-ref="pt"/>
            <aop:after-returning method="afterReturning" pointcut-ref="pt"/>
            <aop:after-throwing method="afterThrowing" pointcut-ref="pt"/>
            <aop:after method="after" pointcut-ref="pt"/>
        </aop:aspect>
    </aop:config>

</beans>
```



4.测试

```java
package com.itheima.test;

import com.itheima.service.UserService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:bean.xml")
public class AppTest {

    @Autowired
    private UserService userService; //注意： 一旦有了切面编程，这里注入的是UserServiceImpl的代理对象

    @Test
    public void test01(){
        userService.save();
    }
}

```



### 6.4 小结

- 通知的类型

  - 前置通知    before
  - 后置通知   after-returning
  - 异常通知    after-throwing
  - 最终通知    after



## 07. Aop编程（六）通知类型 C 环绕通知	【重点】

### 7.1 目标

会使用环绕通知。



### 7.2 步骤

1. 拷贝bean.xml文件改名为bean_around.xml，删除原本的通知类型,添加环绕通知的方法
2. 在LogAspect添加环绕通知方法
3. bean_around.xml配置环绕通知
4. 测试



### 7.3 实现

 1. 拷贝bean.xml文件改名为bean_around.xml，删除原本的通知类型,添加环绕通知的方法

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    
        <!--1. 创建目标对象-->
        <bean id="userService" class="com.itheima.service.impl.UserServiceImpl"/>
    
        <!--2.创建通知类的对象-->
        <bean id="logAspect" class="com.itheima.utils.LogAspect"/>
    
    
        <!--3. 配置一个切面 切面 = 通知+ 切入点 -->
        <aop:config>
            <!--切入点: 切入点表达式就是指明那个类的哪个方法需要被增强-->
            <aop:pointcut id="pt" expression="execution(* com.itheima.service.impl.UserServiceImpl.*(..))"/>
            <!--把通知与切入点融合一起形成一个切面-->
            <aop:aspect ref="logAspect">
                <!--环绕通知-->
               <aop:around method="around" pointcut-ref="pt"/>
            </aop:aspect>
        </aop:config>
    
    </beans>
    ```

    

2. 在LogAspect添加环绕通知方法

```java
//通知类
public class LogAspect {


    /*
        环绕通知的方法的要求：
                1. 环绕通知的方法必须有返回值，这个返回值就是目标方法执行后的结果
                2. 环绕通知的方法必须有一个形参： ProceedingJoinPoint  代表了当前被执行的方法，非常类似于动态代理的method
     */
    public Object around(ProceedingJoinPoint pj){ //ProceedingJoinPoint代表当前执行方法


        try {
            System.out.println("====前置通知===");
            //放行目标方法
            Object result = pj.proceed(); //目标方法执行后的返回值结果，不管有没有值都接收返回
            System.out.println("====后置通知======");
            return result;
        } catch (Throwable throwable) {
            throwable.printStackTrace();
            System.out.println("====异常通知=====");
        } finally {
            System.out.println("=====最终通知=======");
        }
        return null;

    }
}

```



3. 测试

```java
package com.itheima.test;

import com.itheima.service.UserService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:bean_around.xml")
public class AppTest {

    @Autowired
    private UserService userService; //注意： 一旦有了切面编程，这里注入的是UserServiceImpl的代理对象

    @Test
    public void test01(){

        int result = userService.save();
        System.out.println("结果："+ result);
    }
}

```



### 7.4 小结

- ProceedingJoinPoint代表了什么？如何让目标方法执行

  - 





## 08. SpringAop编程XML方式实现(最常用)【重点】

### 8.1 目标  

​	使用AOP面向切面编程为service添加事务管理，事务管理知识模拟即可。



### 8.2 需求

​	使用Aop为AccountServiceImpl的所有方法添加一个环绕通知，在方法执行之前、之后、异常、最终添加对应的事务管理，事务管理只需要模拟即可。 要求纯xml的方式实现



### 8.3步骤

1.创建springday03_03aop_tx_xml，导入相关依赖

2.编写AccountService接口和实现

3.编写事务通知类

4.配置事务切面类

5.测试



### 8.4 实现

1. 创建springday03_03aop_tx_xml，导入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ithema</groupId>
    <artifactId>day44_spring03_aop_xml</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies>
        <!--spring 核心包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>

        <!--使用aop编程-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>

        <!--因为aop编程需要使用切入点表达式，所以需要导入切入点表达式-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.13</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.0.2.RELEASE</version>
            <scope>test</scope>
        </dependency>

    </dependencies>


</project>
```



2. 编写AccountService接口和实现（目标对象）

```java
package com.itheima.service;

public interface AccountService {

    void update();
}

```



```java
package com.itheima.service.imp;

import com.itheima.service.AccountService;

public class AccountServiceImpl implements AccountService {
    @Override
    public void update() {
        System.out.println("更新用户..");
    }
}

```



3. 编写事务通知类

```java
package com.itheima.utils;

import org.aspectj.lang.ProceedingJoinPoint;

//切面类
public class LogAspect {


    //增强的代码， 通知
    public void transaction(ProceedingJoinPoint jp){

        try {
            System.out.println("=======前置通知：开启事务=======");
            //目标方法执行
            jp.proceed();
            System.out.println("=======后置通知：提交事务=======");

        } catch (Throwable throwable) {
            throwable.printStackTrace();
            System.out.println("=======异常通知：回滚事务=======");

        } finally {
            System.out.println("=======最终通知：释放资源=======");

        }

    }
}


```



4. 配置事务切面类(***)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--1. 创建目标对象-->
    <bean id="userService" class="com.itheima.service.imp.AccountServiceImpl"/>

    <!--2.创建通知类对象-->
    <bean id="transactionAspect" class="com.itheima.utils.TransactionAspect"/>

    <!--3.配置切面-->
    <aop:config>
        <!--切入点-->
        <aop:pointcut id="pt" expression="execution(* com.itheima.service.impl.*.*(..))"/>
        <!--切面 -->
        <aop:aspect ref="transactionAspect">
            <!--环绕通知-->
            <aop:around method="tx" pointcut-ref="pt"/>
        </aop:aspect>

    </aop:config>


</beans>
```



5. 测试

测试类

```java
package com.itheima.test;

import com.itheima.service.AccountService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:bean.xml")
public class AppTest {

    @Autowired
    private AccountService accountService;

    @Test
    public void test01(){
        accountService.update();
    }
}

```







## 09. Aop 编程注解方式实现（XML+注解）【重点】

### 9.1 目标

把xml实现aop编程，改为注解实现



### 9.2 需求 

​	使用Aop为AccountServiceImpl的所有方法添加一个环绕通知，在方法执行之前、之后、异常、最终添加对应的事务管理，事务管理只需要模拟即可。 要求xml+注解混合使用，只要求把切面部分

### 9.3 分析

![1604804080065](Spring-03.assets/1604804080065.png)





### 9.4 步骤

1.复制springday03_03aop_tx_xml，改为springday03_04aop_tx_xml_anno

2.在LogAspect上添加注解

3.修改bean.xml配置

4.测试



### 9.5 实现：

1.复制springday03_03aop_tx_xml，改为springday03_04aop_tx_xml_anno





2.在TransactionAspect上添加注解

```java
package com.itheima.utils;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

//切面类= 通知+切入点
@Component
@Aspect
public class TransactionAspect {

    /*
      切入点表达式
       注意： 这个切入点表达式的id就是为当前的方法名
     */
    /*@Pointcut("execution(* com.itheima.service.impl.*.*(..))")
    public void pt(){}

*/

    //环绕通知
  //方式一：   @Around("pt()") //通知类型必须引入切入点表达式的id，就是方法名
   //方式二：
    @Around("execution(* com.itheima.service.impl.*.*(..))")
    public Object tx(ProceedingJoinPoint pj){
        try {
            System.out.println("=====开启事务=====");
            Object result = pj.proceed();
            System.out.println("=====提交事务=====");
            return result;
        } catch (Throwable throwable) {
            throwable.printStackTrace();
            System.out.println("=====回滚事务=====");
        } finally {
            System.out.println("======关闭资源==========");
        }
        return null;
    }

}

```



3.修改bean.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--1. 创建目标对象-->
    <bean id="userService" class="com.itheima.service.impl.AccountServiceImpl"/>

    <!--2.开启包扫描 : component-scan这里的包扫描只能扫描： @Component，@Service，@Controller,@AutoWired.，但是是没法扫描Aop的相关的注解-->
    <context:component-scan base-package="com.itheima"/>

    <!--3.扫描aop的注解, 专门用于扫描@Aspect，@pointCut,@Around这些注解-->
    <aop:aspectj-autoproxy/>
</beans>
```



4.测试

```java
package com.itheima.test;

import com.itheima.service.AccountService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:bean.xml")
public class AppTest {

    @Autowired
    private AccountService accountService;

    @Test
    public void test01(){
        accountService.update();
    }
}

```





### 9.6 小结

- **注解方式的aop编程需要使用哪些注解, 每个注解的作用是什么？** 

  - @Aspect   表示是一个切面类

  - @PointCut   切入点表达式

  - @Around\@Before\@AfterReturning\@AfterThrowing\@After   

    

注意：如果使用了以上注解一定要开启aop的注解扫描



## 10. springAop编程零配置实现纯注解

### 10.1 目标：

使用纯注解的方式实现事务管理



### **10.1需求**

把上面的案例，改造为纯注解形式



**分析**

![1604805399551](Spring-03.assets/1604805399551.png)



### 10.2 步骤：

1.复制springday03_04aop_tx_xml_anno，改为springday03_05aop_anno

2.编写Spring配置类，在配置类添加注解

3.在业务类添加@Service注解

4.修改测试类，改为注解方式启动



实现：

1.复制springday03_04aop_tx_xml_anno，改为springday03_05aop_anno



2.编写Spring配置类，在配置类添加注解

```java
package com.itheima.utils;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration //表示该类是一个配置类，可以省略不写
@ComponentScan("com.itheima")
@EnableAspectJAutoProxy
public class SpringConfiguration {
}

```



3.在业务类添加@Service注解

```java
package com.itheima.service.impl;

import com.itheima.service.AccountService;
import org.springframework.stereotype.Service;

@Service
public class AccountServiceImpl implements AccountService {
    @Override
    public void update() {
        System.out.println("更新用户..");
    }
}

```







4.修改测试类，改为注解方式启动

```java
package com.itheima.test;

import com.itheima.service.AccountService;
import com.itheima.utils.SpringConfiguration;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfiguration.class)
public class AppTest {

    @Autowired
    private AccountService accountService;

    @Test
    public void test01(){
        accountService.update();
    }
}

```



### 10.3 小结

- 纯注解的方式我需要学习哪个新的注解

  - @EnableAspectJAutoProxy  //开启aop的注解扫描

  



## 11. 事务基本概念

### 11.1 目标

了解什么是事务？事务的特性是如何？



### 11.2 事务的概念



**1. 什么是事务？**

事务，一组最小单位的执行单元（一组SQL语句），这个最小单位要么一起成功，要么一起失败！！



**2.事务的四个特性（ACID）？**

原子性（A）：如果使用事务控制对操作进行控制，这些操作要么一起成功，要么一起失败！！

一致性（C）：事务可以让数据库从一个一致性状态切换到另一个一致性的状态（转账总额不变）

隔离性（I）：多个并发事务之间应该相互隔离。

​          如果多个并发事务隔离出问题，可能导致以下现象：

​          1）脏读：一个事务读到另一个未提交的数据（必须防止的）

​           2）不可重复读：一个事务读到另一个已经提交的更新（update）数据

​           3）幻读（虚读） ：   一个事务读到另一个已经提交的插入（insert）数据     



​           可以通过设置数据库的隔离级别，防止以上的现象的发现：

|                                                  | 脏读 | 不可重复读 | 幻读（虚读） |
| ------------------------------------------------ | ---- | ---------- | ------------ |
| read uncommited（脏读）                          | N    | N          | N            |
| read committed(解决了脏读，但是会出现不可重复读) | Y    | N          | N            |
| repeatable read (解决不可重复读，但是出现幻读)   | Y    | Y          | N            |
| serilizable(可序列化)                            | Y    | Y          | Y            |

  

注意：

​     1）mysql的默认隔离级别：repeatable read

​           oracle的默认隔离级别： read committed

​      2）隔离级别越高，事务并发性能越差





持久性（D）：事务一旦提交（commit），数据应该永久保存下来。





### 11.3 小结

- 什么是事务，事务的四大特性是什么？

  - 事务是最小的执行单元，要不一起成功，要不一起失败.
  - 事务的四大特性： ACID





## 12. Spring事务管理介绍



### 12.1 目标

回顾jdbc操作事务的相关代码



### 12.2 原生JDCB实现



回顾之前的JDBC事务管理：

​    开启事务： connection.setAutoCommit(false)

​    提交事务：connection.commit()

​    回滚事务： connection.rollback();



JDBC业务操作

```java
public void UserServiceImpl{
    
    public void save(){
        //获取连接
        Connection conn = JDBCUtils.getConnection();
        
        //====开启事务（取消自动提交）======
        conn.setAutoCommit(false);
        
        try{
            //预编译sql语句
            String sql = "insert into user(username,age) vales(?,?)";
            PreparedStatement stmt = conn.preparedStatement(sql);

            //设置参数
            stmt.setParameter(1,'张三');
            stmt.setParameter(2,20);

            //执行SQL
            stmt.executeUpdate();
            
            //====提交事务====
            conn.commit()
        }catch(Exception e){
            e.printStackOver();
             
            === 回滚事务======
            conn.rollback();    
        }finally{
               //连接放回连接池
           JDBCUtils.releaseConn(conn);
        }
     
    }
    
    
}



```

==Spring事务管理：==

第一： ==JavaEE 体系进行分层开发， 事务控制位于业务层， Spring 提供了分层设计业务层
的事务处理解决方案。===

第二：Spring的事务管理，分为 声明式事务管理 与 编程式事务管理器。

​       Spring事务管理分为：

​				声明式事务管理，底层是AOP思想（重点掌握）

​                编程式事务管理，底层不是AOP思想（了解）

### 小结

- spring提供事务管理是管理service层。 

- spring事务管理分为： 
  - 声明式事务管理(重点)
  - 编程式的事务管理(了解)

## 13. Spring事务管理 Api

首先在项目导入spring-tx包，再查看Api

```xml
 <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
```



|-  PlatformTransactionManager    顶层接口 (spring-tx) （Spring提供用于进行事务控制的方法的接口）

​     |- DataSourceTransactionManager   实现类  (spring-jdbc\mybatis)

​                                Spring提供用于进行事务控制的方法的实现代码，底层调用JDBC事务控制代码

​      |- HibernateTransactionManager 

​                                实现类（spring-orm）(spring提供对Hibernate事务控制实现)

​      |- JpaTransactionManager  实现 （spring-orm）(spring提供对Jpa事务控制实现)





注意：下面是PlatformTransactionManager   接口的方法（导入spirng-tx才可以看到）

![1563761586735](Spring-03.assets/1563761586735.png)

  DataSourceTransactionManager 实现（导入spring-jdbc才有的）           

```xml
<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
```

​      



## 14. Spring声明式事务管理 XML 实现（一）环境准备

### 14.1 需求

使用DataSourceTransactionManager事务管理器对service进行真正意义上事务控制。



### 14.2 实现

步骤：

1.创建springday03_06tx_xml项目，导入依赖,db.properties

2.编写AccountDao接口和实现

3.编写AccountService接口和实现

4.编写bean.xml（没有事务管理器）

5.编写测试



实现：

1.创建springday03_06tx_xml项目，导入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>spring04_04_spring_tx_xml</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!--mysql + druid + jdbcTemplate + spring ioc + spring-aop -->
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.46</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.6</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.7</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>


</project>
```

**db.properties**

```prop
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql:///db1
jdbc.username=root
jdbc.password=root
```



2.编写Account实体，AccountDao接口和实现

```java
package com.itheima.model;

public class Account {

    private int id;

    private String name;

    private double money;


    public Account() {
    }

    public Account(int id, String name, double money) {
        this.id = id;
        this.name = name;
        this.money = money;
    }

    /**
     * 获取
     * @return id
     */
    public int getId() {
        return id;
    }

    /**
     * 设置
     * @param id
     */
    public void setId(int id) {
        this.id = id;
    }

    /**
     * 获取
     * @return name
     */
    public String getName() {
        return name;
    }

    /**
     * 设置
     * @param name
     */
    public void setName(String name) {
        this.name = name;
    }

    /**
     * 获取
     * @return money
     */
    public double getMoney() {
        return money;
    }

    /**
     * 设置
     * @param money
     */
    public void setMoney(double money) {
        this.money = money;
    }

    public String toString() {
        return "Account{id = " + id + ", name = " + name + ", money = " + money + "}";
    }
}

```



3. 编写AccountDao的接口与实现类

```java
package com.itheima.dao;

import com.itheima.model.Account;

public interface AccountDao {

    void save(Account account);
}

```



```java
package com.itheima.dao.impl;

import com.itheima.dao.AccountDao;
import com.itheima.model.Account;
import org.springframework.jdbc.core.JdbcTemplate;

public class AccountDaoImpl implements AccountDao {

    private JdbcTemplate jdbcTemplate;

    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public void save(Account account) {
        String sql = "insert into account values(null,?,?)";
        Object[] param = {account.getName(),account.getMoney()};
        jdbcTemplate.update(sql,param);
    }
}

```





3.编写AccountService接口和实现

```java
package com.itheima.service;

import com.itheima.model.Account;

public interface AccountService {

    void save(Account account);
}

```



```java
package com.itheima.service.impl;

import com.itheima.dao.AccountDao;
import com.itheima.model.Account;
import com.itheima.service.AccountService;

public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    @Override
    public void save(Account account) {
        accountDao.save(account);
        int i = 1/0; //这里出现异常， 如果没有事务管理器：一条成功，一条失败.
                    //一旦有事务管理器管理： 要不一起成功，要不就一起失败
        accountDao.save(account);
    }
}

```



4.编写bean.xml（没有事务控制）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">


    <!--1. 加载properties配置文件-->
    <context:property-placeholder location="classpath:db.properties"/>


    <!--2.创建连接池对象-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>


    <!--3. 创建jdbctemplate对象-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>


    <!--4.创建AccountDaoImpl对象-->
    <bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate"/>
    </bean>


    <!--5. 创建AccountServiceImpl对象-->
    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"/>
    </bean>



</beans>

```





5.编写测试

```java
package com.itheima.test;

import com.itheima.model.Account;
import com.itheima.service.AccountService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:bean.xml")
public class AppTest {

    @Autowired
    private AccountService accountService;

    @Test
    public void test01(){
        Account account = new Account();
        account.setName("狗娃");
        account.setMoney(1000);
        accountService.save(account);

    }
}

```



查看运行结果，发现程序抛出异常，但是数据依然成功插入了1条，证明当前环境没有事务控制！

![1563765141788](Spring-03.assets/1563765141788.png)







## 15. Spring声明式事务管理 XML 实现（二）事务配置【重点】

### 15.1 需求

通过XML的Spring声明式事务来对业务层方法进入切入事务



### 15.2 步骤：

1. 在spring03_06tx_xml项目导入spring-tx依赖

2. 在bean.xml添加事务配置

3. 测试



### 15.3 实现：

1.在spring03_06tx_xml项目导入spring-tx依赖



spring-tx依赖

```xml
 <!--事务管理依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
```

spring-aop依赖（**spring事务管理必须依赖spring-aop依赖**，已经是导入了）

```xml
<dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-aop</artifactId>
     <version>5.0.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.7</version>
</dependency>
```



2.在bean.xml添加事务配置



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--suppress ALL -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">


    <!--1. 加载properties配置文件-->
    <context:property-placeholder location="classpath:db.properties"/>


    <!--2.创建连接池对象-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>


    <!--3. 创建jdbctemplate对象-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>


    <!--4.创建AccountDaoImpl对象-->
    <bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate"/>
    </bean>


    <!--5. 创建AccountServiceImpl对象 目标对象-->
    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"/>
    </bean>

    <!--6.创建事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>


    <!--7.配置事务的通知规则:  配置哪些方法是需要事务，哪些方法是不需要事务的-->
    <tx:advice id="ad" transaction-manager="transactionManager">
        <tx:attributes>
            <!--配置哪些方法要事务，哪些方法不需要事务， 查询的方法是不需要事务，其他的方法都需要事务
                name: 方法名
                propagation: 事务的传播行为，规定哪些方法要事务，哪些方法不需要事务,常用的值： REQUIRED（必须有事务）,SUPPORTS(可有可无)有没有事务都可以

                企业使用规则：查询在前，其他在后
            -->
            <tx:method name="find*" propagation="SUPPORTS"/>
            <tx:method name="get*" propagation="SUPPORTS"/>
            <tx:method name="query*" propagation="SUPPORTS"/>
            <tx:method name="select*" propagation="SUPPORTS"/>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>



    <!--8. 配置切面-->
    <aop:config>
        <!--切入点表达式-->
        <aop:pointcut id="pt" expression="execution(* com.itheima.service.impl.*.*(..))"/>
        <!--配置通知
            advice-ref : 引用通知规则的id
            pointcut-ref: 切入点表达式的引用
        -->
        <aop:advisor advice-ref="ad" pointcut-ref="pt"/>
    </aop:config>


</beans>

```



3.测试

```java
package com.itheima.test;

import com.itheima.model.Account;
import com.itheima.service.AccountService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:bean.xml")
public class AppTest {

    @Autowired
    private AccountService accountService;

    @Test
    public void test01(){
        Account account = new Account();
        account.setName("狗娃");
        account.setMoney(1000);
        accountService.save(account);

    }
}

```



### 15.4 小结

- **声明式事务bean.,xml操作步骤**

  1. 创建事务管理器对象

2. 配置事务通知规则：哪些方法要有事务，哪些方法没有事务
  3. 配置切面

  

  


## 16. Spring声明式事务管理 注解实现（XML+注解 , 了解）

### 16.1 需求

把上面XML的Spring声明式事务改造为注解方式



**分析**

![1563767552209](Spring-03.assets/1563767552209.png)

注意：事务管理器依然还是bean来配置！！！



### 16.2 步骤：

1.复制springday03_06tx_xml，改为springday03_07tx_xml_anno

2.在业务类上添加@Transactional注解

3.修改bean.xml开启事务注解扫描

4.测试



### 16.7 实现

1. 复制springday03_06tx_xml，改为springday03_07tx_xml_anno





2.在业务类上添加@Transactional注解

![1604821677436](Spring-03.assets/1604821677436.png)

```java
package com.itheima.service.impl;

import com.itheima.dao.AccountDao;
import com.itheima.model.Account;
import com.itheima.service.AccountService;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@Transactional  //代表了该类的所有方法都需要使用事务管理器，而且每一个Service的类你都需要要指定。
                //@Transactional  也可以用于方法上,配置事务的隔离级别
public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    @Override
    //@Transactional(propagation = Propagation.REQUIRED)
    public void save(Account account) {
        accountDao.save(account);
       // int i = 1/0; //这里出现异常， 如果没有事务管理器：一条成功，一条失败.
                    //一旦有事务管理器管理： 要不一起成功，要不就一起失败
        accountDao.save(account);
    }
}

```





3.修改bean.xml开启事务注解扫描

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--suppress ALL -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">


    <!--1. 加载properties配置文件-->
    <context:property-placeholder location="classpath:db.properties"/>


    <!--2.创建连接池对象-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>


    <!--3. 创建jdbctemplate对象-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>


    <!--4.创建AccountDaoImpl对象-->
    <bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate"/>
    </bean>


    <!--5. 创建AccountServiceImpl对象 目标对象-->
    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"/>
    </bean>

    <!--6.创建事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>


    <!--开启@Transactional注解扫描-->
    <tx:annotation-driven transaction-manager="transactionManager"/>

</beans>

```



4.测试

```java
package com.itheima.test;

import com.itheima.model.Account;
import com.itheima.service.AccountService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:bean.xml")
public class AppTest {

    @Autowired
    private AccountService accountService;

    @Test
    public void test01(){
        Account account = new Account();
        account.setName("狗娃");
        account.setMoney(1000);
        accountService.save(account);

    }
}

```



### 16.8 小结 

​	事务管理使用的那个注解：@Transactional	

​	开启@Transactional的注解使用哪个标签：<tx:annotation-driven/>



## 17. Spring声明式事务管理 零配置实现 （纯注解）

### 17.1 需求

把XML+注解版本改为纯注解



**分析**



![1565151439795](Spring-03.assets/1565151439795.png)

![1565151742305](Spring-03.assets/1565151742305.png)





### 17.2 步骤

1. 复制springday03_07tx_xml_anno，改为springday03_08tx_anno

2. 编写配置类，添加相应注解

3. 在Service类和Dao类添加注解

4. 修改测试类，改为注解方式加载



### 17.3 实现

1.复制springday03_07tx_xml_anno，改为springday03_08tx_anno





2.编写配置类，添加相应注解, 编写SpringConfig配置类

```java
package com.itheima.utils;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.*;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;

@Configuration  //表示是一个配置类
@EnableTransactionManagement  //开启注解扫描，扫描的是@Transactional
@ComponentScan("com.itheima") //扫描的是@Service相关的注解
@Import(JdbcConfig.class) //引用另外一个配置类
public class SpringConfiguration {




}

```

3. 编写JdbcConfig配置类

```java
package com.itheima.utils;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.PropertySource;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

import javax.sql.DataSource;

@PropertySource("classpath:db.properties")//加载配置文件properties
public class JdbcConfig {

    @Value("${jdbc.driverClassName}")
    private String driverClassName;

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean  //该方法会被自动调用，返回值自动存储到spring的容器中
    public DataSource createDataSource(){
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(driverClassName);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }


    @Bean  //该方法会被自动调用，返回值自动存储到spring的容器中
    public JdbcTemplate createJdbcTemplate(DataSource dataSource){
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        jdbcTemplate.setDataSource(dataSource);
        return jdbcTemplate;
    }


    @Bean  //该方法会被自动调用，返回值自动存储到spring的容器中
    public DataSourceTransactionManager createDataSourceTransactionManager(DataSource dataSource){
        DataSourceTransactionManager manager = new DataSourceTransactionManager();
        manager.setDataSource(dataSource);
        return manager;
    }
}

```

4. 修改测试类，改为注解方式加载

```java
package com.itheima.test;

import com.itheima.model.Account;
import com.itheima.service.AccountService;
import com.itheima.utils.SpringConfiguration;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfiguration.class)
public class AppTest {

    @Autowired
    private AccountService accountService;

    @Test
    public void test01(){
        Account account = new Account();
        account.setName("狗娃");
        account.setMoney(1000);
        accountService.save(account);

    }
}

```



### 17.4 小结

-  **纯注解开发我们学习哪些新注解**
-  @Configuration  //表示是一个配置类
   @EnableTransactionManagement  //开启注解扫描，扫描的是@Transactional
   @ComponentScan("com.itheima") //扫描的是@Service相关的注解
   @Import(JdbcConfig.class) //引用另外一个配置类



## 18. Spring编程式事务【了解】

### 18.1 需求

使用Spring编程式事务，意思就是用写代码的方式直接在业务类加入Spring事务控制！



### 18.2 步骤：

1.复制项目springday03_06tx_xml，改为springday03_09tx_code

2.bean.xml创建事务管理器，创建事务模板

3.在业务方法上编码实现事务控制

4.测试



### 18.3 实现

1.复制项目spring04_04_spring_tx_xml，改为spring04_07_spring_tx_code







2.在applicationContext.xml创建事务管理器，创建事务模板对象

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--suppress ALL -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">


    <!--1. 加载properties配置文件-->
    <context:property-placeholder location="classpath:db.properties"/>


    <!--2.创建连接池对象-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>


    <!--3. 创建jdbctemplate对象-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>


    <!--4.创建AccountDaoImpl对象-->
    <bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate"/>
    </bean>


    <!--5. 创建AccountServiceImpl对象 目标对象-->
    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="transactionTemplate" ref="transactionTemplate"/>
    </bean>

    <!--6.创建事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>


    <!--7.创建事务管理模板-->
   <bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
       <property name="transactionManager" ref="transactionManager"/>
   </bean>

</beans>

```



3.在业务方法上编码实现事务控制(***)

```java
package com.itheima.service.impl;

import com.itheima.dao.AccountDao;
import com.itheima.model.Account;
import com.itheima.service.AccountService;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.TransactionCallback;
import org.springframework.transaction.support.TransactionTemplate;

public class AccountServiceImpl implements AccountService {

    private AccountDao accountDao;

    private TransactionTemplate transactionTemplate;

    public void setTransactionTemplate(TransactionTemplate transactionTemplate) {
        this.transactionTemplate = transactionTemplate;
    }

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    @Override
    public void save(Account account) {
        //创建一个事务的回滚器
        TransactionCallback callback = new TransactionCallback() {
            @Override
            public Object doInTransaction(TransactionStatus transactionStatus) {
                //要进行事务管理的代码
                accountDao.save(account);
                int i = 1/0; //这里出现异常， 如果没有事务管理器：一条成功，一条失败.
                //一旦有事务管理器管理： 要不一起成功，要不就一起失败
                accountDao.save(account);
                return null;
            }
        };

        //使用事务的模板执行事务的回滚器
        transactionTemplate.execute(callback);


    }
}

```



4.测试

```java
package com.itheima.test;

import com.itheima.model.Account;
import com.itheima.service.AccountService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:bean.xml")
public class AppTest {

    @Autowired
    private AccountService accountService;

    @Test
    public void test01(){
        Account account = new Account();
        account.setName("狗娃");
        account.setMoney(1000);
        accountService.save(account);

    }
}

```











