---
title: Spring(02)
tags:
  - 笔记
  - ssm框架
  - Spring
  - 控制反转
  - IOC
  - 依赖注入
  - DI
  - 动态代理
  - JdbcTemplate
categories:
  - ssm框架
abbrlink: 8201
date: 2020-11-07 21:05:45
---







## 1. 今日学习目标

1. 掌握JdbcTemplate使用
2. 完成spring IOC案例实现
3. 完成spring整合junit

4. 理解什么动态代理
5. 掌握动态代理的两种实现方式
6. 完成动态代理案例



##  2,JdbcTemplate概述与入门(使用传统的方式使用jdbctemplate)

### 目标

掌握jdbctemplate的入门步骤



### 步骤

#### jdbctemplate的基本概述

JdbcTemplate是spring提供的一个模板类，它是对jdbc的简单封装。用于支持持久层的操作。==它的特点是：简单、方便，功能不够mybatis强大,以后我们还是以mybatis为主==


![1566204269065](Spring-02.assets/1566204269065.png)

#### 准备数据

```sql
/*创建账户表*/
create table account(
	id int primary key auto_increment,
	name varchar(40),
	money float
)ENGINE=InnoDB character set utf8 collate utf8_general_ci;

/*初始化新增三个账户*/
insert into account(name,money) values('小明',1000);
insert into account(name,money) values('小花',1000);
insert into account(name,money) values('小王',1000);
```

图一：

![1566204645523](Spring-02.assets/1566204645523.png)



#### 案例演示



##### 1. 配置pom.xml，导入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>springday02_01jdbctemplate</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!--spring核心包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        <!--jdbctemplate-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>

        <!--数据库驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.46</version>
        </dependency>

        <!--连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.10</version>
        </dependency>

        <!--单元测试-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>


    </dependencies>

</project>
```

**2.在resources目录下创建db.properties配置文件**

```properties
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql:///db1
username=root
password=root
```



##### 编写案例代码

```java
package com.itheima.test;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.pool.DruidDataSourceFactory;
import org.junit.Test;
import org.springframework.jdbc.core.JdbcTemplate;

import javax.sql.DataSource;
import java.io.IOException;
import java.util.Properties;

public class JdbcTemplateTest {

    //目标： jdbctemlate的基本使用
    @Test
    public void test01() throws Exception {
        //1. 加载properties的配置文件
        Properties properties = new Properties();
        properties.load(JdbcTemplateTest.class.getResourceAsStream("/db.properties"));
        //2. 创建druid的连接池
        DataSource dataSource = DruidDataSourceFactory.createDataSource(properties);
        //3. 创建jdbctemplate的核心类对象
        JdbcTemplate jdbcTemplate = new JdbcTemplate();

        //4. 给jdbctemplate配置连接池
        jdbcTemplate.setDataSource(dataSource);

        //5. 准备sql语句，使用jdbctemplate执行sql语句
        String sql = "insert into account values(null,?,?)";
        //sql语句的参数
        Object[] param = {"老钟",800};
        //执行sql语句
        jdbcTemplate.update(sql,param);
    }
}

```

图一：

![1566205410851](Spring-02.assets/1566205410851.png)



### 小结

​	**jdbctemplate的核心类与核心方法：**

			- jdbctemplate
			- update()(增删改)|| query()||queryForObject()  查询 





## 3. spring IOC管理JdbcTemplate

**思路分析**

- spring容器应该有哪些对象？
  - 加载配置文件
  - 连接池
  - jdbctemplate

**创建db2.properties文件**

```properties
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql:///db1
jdbc.username=root
jdbc.password=root
```



##### 编写bean.xml(创建jdbctemplate、连接池的对象)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--1. 加载配置文件-->
    <context:property-placeholder location="classpath:db2.properties"/>

    <!--2. 创建连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <!--给连接池设置数据库的四大参数-->
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>


    <!--3. 创建jdbctemplate对象-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!--ref: 引用id值为指定的对象-->
        <property name="dataSource" ref="dataSource"/>
    </bean>


</beans>
```



##### 在测试包创建JdbcTemplateTest2测试代码

```java
package com.itheima.test;

import com.alibaba.druid.pool.DruidDataSourceFactory;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.jdbc.core.JdbcTemplate;

import javax.sql.DataSource;
import java.util.Properties;

public class JdbcTemplateTest2 {

    //目标： 利用spring的ioc管理了jdbctemplate然后使用
    @Test
    public void test01() throws Exception {
       //1. 创建spring的容器   classpath:代表的是从resources目录中查找资源，目前可以不添加，但是ssm整合之后是一定要添加的，所以建议同学们现在就习惯
        ApplicationContext context  = new ClassPathXmlApplicationContext("classpath:bean.xml");
        //2. 从spring的容器中查找jdbctemplate
        JdbcTemplate jdbcTemplate = (JdbcTemplate) context.getBean("jdbcTemplate");
        //3. 准备sql语句，执行sql语句
        String sql = "delete from account where name=?";
        //如果只有一个参数的时候那就没必要封装到数组中了，直接给update方法即可
        jdbcTemplate.update(sql,"老钟");
    }
}

```



### 小结

- 使用spring的ioc管理jdbctemplate的步骤

  1. 准备db.properties
  2. 编写bean.xml的核心配置文件
     1. 加载properties文件，得到数据库参数
     2. 创建DruidDataSource
     3. 创建jdbctemplate核心类对象
  3. 测试使用





## 4. JdbcTemplate实现完整增删改操作



### 目标

​	使用jdbctemplate完成对account的增删改操作	



### 案例需求

需求：把jdbctemplate交给spring管理，实现以下功能：
	1.新增一个账户
	2.根据用户id修改用户
	3.根据用户id删除用户

### 步骤

在JdbcTemplateTest2添加增删改的方法

```java
package com.itheima.test;

import com.alibaba.druid.pool.DruidDataSourceFactory;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.jdbc.core.JdbcTemplate;

import javax.sql.DataSource;
import java.util.Properties;

public class JdbcTemplateTest2 {

    //目标： 利用spring的ioc管理了jdbctemplate然后使用
    @Test
    public void testDelete() throws Exception {
       //1. 创建spring的容器   classpath:代表的是从resources目录中查找资源，目前可以不添加，但是ssm整合之后是一定要添加的，所以建议同学们现在就习惯
        ApplicationContext context  = new ClassPathXmlApplicationContext("classpath:bean.xml");
        //2. 从spring的容器中查找jdbctemplate
        JdbcTemplate jdbcTemplate = (JdbcTemplate) context.getBean("jdbcTemplate");
        //3. 准备sql语句，执行sql语句
        String sql = "delete from account where name=?";
        //如果只有一个参数的时候那就没必要封装到数组中了，直接给update方法即可
        jdbcTemplate.update(sql,"老钟");
    }


    //目标：增加
    @Test
    public void testAdd() throws Exception {
        //1. 创建spring的容器   classpath:代表的是从resources目录中查找资源，目前可以不添加，但是ssm整合之后是一定要添加的，所以建议同学们现在就习惯
        ApplicationContext context  = new ClassPathXmlApplicationContext("classpath:bean.xml");
        //2. 从spring的容器中查找jdbctemplate
        JdbcTemplate jdbcTemplate = (JdbcTemplate) context.getBean("jdbcTemplate");
        //3. 准备sql语句，执行sql语句
       String sql = "insert into account values(null,?,?)";
       //把sql的参数封装到object 数组中
        Object[] param = {"宝晴",180000};
        jdbcTemplate.update(sql,param);

    }

    //目标：修改
    @Test
    public void testUpdate() throws Exception {
        //1. 创建spring的容器   classpath:代表的是从resources目录中查找资源，目前可以不添加，但是ssm整合之后是一定要添加的，所以建议同学们现在就习惯
        ApplicationContext context  = new ClassPathXmlApplicationContext("classpath:bean.xml");
        //2. 从spring的容器中查找jdbctemplate
        JdbcTemplate jdbcTemplate = (JdbcTemplate) context.getBean("jdbcTemplate");
        //3. 准备sql语句，执行sql语句
        String sql = "update account set money = ? where name=?";
        //sql语句对应的参数
        Object[] param = {16000,"宝晴"};

        jdbcTemplate.update(sql,param);


    }
}

```



### 小结

- 使用jdbctemplate的增删改使用哪个方法？
  - update(sql,Object[])
  - update(sql,一个参数)





## 5. JdbcTemplate完成查询



### 目标

	- 使用jdbctemplate完成查询
	- 能够使用rowmapper结果处理器与BeanPropertyRowMapper结果处理器



### 需求

1.  查询单个对象，使用RowMapper结果映射器

  2.  查询多个对象，使用RowMapper结果映射器
  3.  查询单个、多个对象，使用BeanPropertyMapper结果映射器

#### 自定义Account类

```java
package com.itheima.domain;

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



#### 自定义RowMapper多个对象代码实现

![1604713710165](Spring-02.assets/1604713710165.png)

```java
package com.itheima.test;

import com.itheima.domain.Account;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

/*
目标： 学习使用jdbctemplate完成的查询
 */
public class JdbcTemplateTest3 {


    //目标： 自定义结果映射器查询全部 ， 查询多个对象使用的方法是query
    @Test
    public void test01(){
        //1. 创建spring的容器
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:bean.xml");

        //2. 从容器中查找jdbctemplate
        JdbcTemplate  jdbcTemplate = (JdbcTemplate) context.getBean("jdbcTemplate");

        //3. 准备sql语句
        String sql = "select * from account";

        //4. 自定义结果映射器，执行查询
        /*
            RowMapper（接口）：结果映射器, 结果映射器的作用就是把查询的结果封装到对象中


         */
        List<Account> list =  jdbcTemplate.query(sql, new RowMapper<Account>() {
           /*
                ResultSet: 结果集,保存了当前遍历到的数据
                i :当前遍历到的结果集的索引值
                mapRow方法作用： 每遍历一条数据的时候都会调用一次mapRow方法，然后把当前行的数据封装到  ResultSet中。
            */
            @Override
            public Account mapRow(ResultSet rs, int i) throws SQLException {
                //每一行数据就对应一个Account对象
                Account account = new Account();
                account.setId(rs.getInt("id"));
                account.setName(rs.getString("name"));
                account.setMoney(rs.getDouble("money"));
                return account;
            }
        });

        System.out.println("查询结果："+list);

    }





}

```

#### 自定义RowMappe单个对象实现

```java
    //目标： 自定义结果映射器查询单个对象 ， 查询单个对象使用的方法是queryForObject
    @Test
    public void testForObject(){
        //1. 创建spring的容器
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:bean.xml");

        //2. 从容器中查找jdbctemplate
        JdbcTemplate  jdbcTemplate = (JdbcTemplate) context.getBean("jdbcTemplate");

        //3. 准备sql语句
        String sql = "select * from account where id = ?";

        //4. 自定义结果映射器，执行查询
        Account account = jdbcTemplate.queryForObject(sql, new RowMapper<Account>() {
            @Override
            public Account mapRow(ResultSet rs, int i) throws SQLException {
                //每一行数据就对应一个Account对象
                Account account = new Account();
                account.setId(rs.getInt("id"));
                account.setName(rs.getString("name"));
                account.setMoney(rs.getDouble("money"));
                return account;
            }
        }, 1);
        System.out.println("查询的对象："+account);
    }
```



#### BeanPropertyRowMapper实现多个对象的查询

```java
    //目标： 使用现有的结果映射器查询全部对象
    @Test
    public void testForList2(){
        //1. 创建spring的容器
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:bean.xml");

        //2. 从容器中查找jdbctemplate
        JdbcTemplate  jdbcTemplate = (JdbcTemplate) context.getBean("jdbcTemplate");

        //3. 准备sql语句
        String sql = "select * from account";
        //BeanPropertyRowMapper 就是一个已经实现RowMapper接口的实现类。
        List<Account> list = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Account>(Account.class));
        System.out.println("查询的结果："+ list);
    }



    //目标： 使用现有的结果映射器查询单个对象
    @Test
    public void testForObj(){
        //1. 创建spring的容器
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:bean.xml");

        //2. 从容器中查找jdbctemplate
        JdbcTemplate  jdbcTemplate = (JdbcTemplate) context.getBean("jdbcTemplate");

        //3. 准备sql语句
        String sql = "select * from account where name=?";
        //BeanPropertyRowMapper 就是一个已经实现RowMapper接口的实现类。
        Account account = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<Account>(Account.class),"宝晴");
        System.out.println("查询的结果："+ account);
    }


```



### 小结

- 查询一个对象使用哪个方法？

  - queryForObject（sql,new BeanPropertyRowMapper(),参数）

  

- 查询多个对象使用哪个方法？

  - query（sql,new BeanPropertyRowMapper(),参数）

  





## 6. spring IOC案例实现(重点，重点练习的是springioc的管理)

### 案例需求

​	编写dao与service层，使用spring管理dao与service、jdbctemplate、连接池对象，实现查询所有联系人。



### 案例实现(xml版本)

#### 创建一个新模块springday02_02xml，配置pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>day43_spring02_1jdbctemplate</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>

        <!--jdbctemplate的核心包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>


        <!--测试
        -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>

        <!--mysql驱动包-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.40</version>
        </dependency>

        <!--连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.10</version>
        </dependency>

    </dependencies>

    
</project>
```



#### 开发代码

###### 编写实体类

```java
package com.itheima.domain;

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



###### 编写持久层

- dao接口

```java
package com.itheima.dao;

import com.itheima.domain.Account;

import java.util.List;

public interface AccountDao {

    //查询所有的用户
    List<Account> findAll();
}

```

- dao实现类

  ```java
  package com.itheima.dao.impl;
  
  import com.itheima.dao.AccountDao;
  import com.itheima.domain.Account;
  import org.springframework.jdbc.core.BeanPropertyRowMapper;
  import org.springframework.jdbc.core.JdbcTemplate;
  
  import java.util.List;
  
  public class AccountDaoImpl implements AccountDao {
  
      private JdbcTemplate jdbcTemplate;
  
      public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
          this.jdbcTemplate = jdbcTemplate;
      }
  
      @Override
      public List<Account> findAll() {
          String sql = "select * from account";
          List<Account> list = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Account>(Account.class));
          return list;
      }
  }
  
  ```

  

#### 编写业务层

- service接口

  ```java
  package com.itheima.service;
  
  import com.itheima.domain.Account;
  
  import java.util.List;
  
  public interface AccountService {
  
      //查询所有的用户
      List<Account> findAll();
  }
  
  ```

  

- service实现类

  ```java
  package com.itheima.service.impl;
  
  import com.itheima.dao.AccountDao;
  import com.itheima.domain.Account;
  import com.itheima.service.AccountService;
  
  import java.util.List;
  
  public class AccountServiceImpl implements AccountService {
  
      private AccountDao accountDao;
  
      public void setAccountDao(AccountDao accountDao) {
          this.accountDao = accountDao;
      }
  
      @Override
      public List<Account> findAll() {
          return accountDao.findAll();
      }
  }
  
  ```

  

###### 编写bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--1. 加载db.properties文件-->
    <context:property-placeholder location="classpath:db.properties"/>

    <!--2. 创建druid的连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>


    <!--3.创建jdbctemplate-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>


    <!--4. 创建AccountDaoImpl对象-->
    <bean id="accountDao" class="com.itheima.dao.impl.AccountDaoImpl">
        <!--property标签是给对象设置属性使用，是否有指定的数据只看setxx方法-->
        <property name="jdbcTemplate" ref="jdbcTemplate"/>
    </bean>


    <!--5.创建AccountServiceImpl对象-->
    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"/>
    </bean>





</beans>
```



###### 编写测试类

```java
package com.itheima.test;

import com.itheima.service.AccountService;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppTest {

    @Test
    public void test01(){
        //创建spring 的容器
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:bean.xml");
        AccountService accountService = (AccountService) context.getBean("accountService");
        System.out.println("用户："+accountService.findAll());
    }
}

```



### xml与注解混合版本(最为常用)

#### 需求

1. 将上述版本AccountDao的实现类对象与AccountService实现类对象从bean.xml文件上抽离出去，使用注解创建。
  2. xml文件只留下dataSource，jdbctemplate。 

#### 实现思路

- dao层的代码使用@Rspository.  @Autowired
- Service层代码使用@Service ，   @Autowired
- 开启注解扫描

​	

#### 案例实现

1. springday02_02xml的模块，然后修改模块名springday02_02xml_ann

   ​	==注意： 修改模块名需要修改三个地方：==

   		1. 修改模块名
   		2. 修改iml文件，文件名必须与模块名一样
   		3. 修改pom文件的项目名

2. 改造用户dao实现类

```java
package com.itheima.dao.impl;

import com.itheima.dao.AccountDao;
import com.itheima.domain.Account;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public class AccountDaoImpl implements AccountDao {

    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    @Override
    public List<Account> findAll() {
        String sql = "select * from account";
        List<Account> list = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Account>(Account.class));
        return list;
    }
}

```



3. 改造客户service实现类

```java
package com.itheima.service.impl;

import com.itheima.dao.AccountDao;
import com.itheima.domain.Account;
import com.itheima.service.AccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service("accountService")
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;


    @Override
    public List<Account> findAll() {
        return accountDao.findAll();
    }
}

```



###### **4. 配置bean.xml**

![1604717136204](Spring-02.assets/1604717136204.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--1. 加载db.properties文件-->
    <context:property-placeholder location="classpath:db.properties"/>

    <!--2. 创建druid的连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>


    <!--3.创建jdbctemplate-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>


    <!--开启注解扫描-->
    <context:component-scan base-package="com.itheima"/>

</beans>
```



**测试**

```java
package com.itheima.test;

import com.itheima.service.AccountService;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppTest {

    @Test
    public void test01(){
        //创建spring 的容器
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:bean.xml");
        AccountService accountService = (AccountService) context.getBean("accountService");
        System.out.println("用户："+accountService.findAll());
    }
}

```





### 完全注解基础版本



#### 实现步骤

1. 编写一个配置类用于取代bean.xml文件。
2. 在类上添加@Configuration注解与@ComponentScan注解用于做包扫描
3. 使用@Bean在方法上用于创建jdbcTemplate与数据源





#### 代码实现

1. 把springday02_02xml_ann模块复制修改模块springday02_03ann

   ![1604717348879](Spring-02.assets/1604717348879.png)

2. 编写配置类

```java
package com.itheima.utils;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.jdbc.core.JdbcTemplate;

import javax.sql.DataSource;

/*
该类的作用就是用于取代bean.xml文件的

@Configuration: 表示该类是一个配置类，用于取代配置文件，即使不写该注解也可以。

 */
@Configuration
@PropertySource("classpath:db.properties")
@ComponentScan("com.itheima")
public class SpringConfigration {

    @Value("${jdbc.driverClassName}")
    private String  driverClassName;

    @Value("${jdbc.url}")
    private String  url;

    @Value("${jdbc.username}")
    private String  username;

    @Value("${jdbc.password}")
    private String  password;

    /*
     @Bean注解的作用：
           1. 一个方法添加了Bean注解,那么该方法会被自动调用
           2. 该方法的返回值会被存储到spring的容器中。
     */
    @Bean
    public DataSource createDataSource(){
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(driverClassName);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }

    /*
   @Bean注解的作用：
         1. 一个方法添加了Bean注解,那么该方法会被自动调用
         2. 该方法的返回值会被存储到spring的容器中。
         3. bean调用的方法如果有形参，会自动从容器中去搜索相同类型的参数注入
   */
    @Bean
    public JdbcTemplate createJdbcTemplate(DataSource dataSource){
       JdbcTemplate jdbcTemplate = new JdbcTemplate();
       jdbcTemplate.setDataSource(dataSource);
       return jdbcTemplate;
    }


}

```



###### 改造测试类

![1604718017002](Spring-02.assets/1604718017002.png)

```java
package com.itheima.test;

import com.itheima.service.AccountService;
import com.itheima.utils.SpringConfigration;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppTest {

    @Test
    public void test01(){
        //创建spring 的容器AnnotationConfigApplicationContext加载的是配置类
        ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfigration.class);
        AccountService accountService = (AccountService) context.getBean("accountService");
        System.out.println("用户："+accountService.findAll());
    }
}

```



### 小结

- **使用纯注解的时候我们学习哪些注解？**
  -   @Configuration   表示该类是一个配置类
  -   @PropertySource   用于加载db.properties文件的
  -   @ComponentScan  包扫描
  -   @Bean   会自动调用， 方法的返回值存储到spring的容器中， 如果方法上有形参会自动从spring的容器中搜索类型匹配的注入





## 7. spring整合junit(常用)

### 为什么需要使用spring整合junit

1. 主要是为了方便测试

2. Spring整合Junit后，创建容器，可以使用注解完成。

3. 如果要使用Spring整合Junit方便测试，需要先在项目中引入spring-test的支持包。

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-test</artifactId>
       <version>5.0.2.RELEASE</version>
   </dependency>
   ```

### 基于完全xml版本整合(混合版本与该方式是一样)



#### 配置pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>springday02_02xml</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>

        <!--jdbctemplate的核心包-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>


        <!--测试
        -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>

        <!--mysql驱动包-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.40</version>
        </dependency>

        <!--连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.10</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>

    </dependencies>


</project>
```



#### 编写测试类

```java
package com.itheima.test;

import com.itheima.service.AccountService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

/*
    spring整合junit的好处：
        1. 不要自己创建容器了。
        2. 获取某一个类的对象的时候使用自动注入即可，不需要再查询
    SPring整合junit要使用的到注解：
        1.@RunWith(SpringJUnit4ClassRunner.class) 这个注解指定测试的时候使用SpringJUnit4ClassRunner这个类去执行方法
            SpringJUnit4ClassRunner这个类会帮助我们创建容器。
       2.@ContextConfiguration("classpath:bean.xml")  指定容器加载的配置文件

 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:bean.xml")
public class AppTest {

    @Autowired
    private AccountService accountService;

    @Test
    public void test01(){
        System.out.println("用户："+accountService.findAll());
    }
}

```



### 基于完全注解版本整合



#### 导入依赖

![1604719020744](Spring-02.assets/1604719020744.png)



#### 编写测试类

```java
package com.itheima.test;

import com.itheima.service.AccountService;
import com.itheima.utils.SpringConfigration;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=SpringConfigration.class)
public class AppTest {

    @Autowired
    private AccountService accountService;


    @Test
    public void test01(){
        System.out.println("用户："+accountService.findAll());
    }
}

```



### 小结

- **spring整合junit的好处与步骤？**
  - 好处：
    - 不需要再手动创建spring 的容器
    - 不需要我们去查找指定的对象，只需要根据类型自动注入即可
  - 步骤
    - 导入依赖spring-test
    - @RunWith(SpringJUnit4ClassRunner.class)
    - @ContextConfiguration指定加载的配置文件



## 8. 动态代理回顾

### 代理模式概述

#### 代理模式的作用

​	增强某一个对象的某些功能，而且是在不修改源码的情况下。 



### 动态代理两种实现方式



#### 需求:增强List对象的add方法

```java
package com.itheima.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.ArrayList;
import java.util.List;

/*
动态代理的作用：增强某个对象的某些功能

动态代理的核心类: Proxy

创建代理的对象核心方法：newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
        loader： 类加载器
        interfaces: 代理对象实现的接口
        InvocationHandler: 处理器

Proxy实现的动态代理的弊端： 代理的对象必须要存在接口。、

 */
public class Demo1 {

    public static List getListProxy(){
        //1. 维护一个被代理的对象
        List list = new ArrayList();

       List listProxy = (List) Proxy.newProxyInstance(Demo1.class.getClassLoader(), new Class[]{List.class}, new InvocationHandler() {
            //method: 代表了当前被调用的方法
           //args:代表了当前调用该方法传递的实参
           @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                //2.得到当前调用的方法名
               String methodName = method.getName();
               Object result = null;
               //3. 判断是否是需要被增强的方法
               if("add".equalsIgnoreCase(methodName)){
                   //4. 如果是需要被增强的方法，让被代理对象执行方法，然后再增强
                   result =  method.invoke(list,args);
                   System.out.println("添加了:"+args[0]);
               }else{
                   //5. 如果不是要增强的方法，使用被代理对象执行方法即可
                   result =   method.invoke(list,args);
               }

               //6. 方法执行完毕之后，不管方法有没有返回值都定义一个变量接收方法执行的结果，并返回
                return result;
            }
        });
       return listProxy;
    }

    public static void main(String[] args) {
        List list = getListProxy();//得到代理对象
        list.add("川建国"); //调用代理对象的任何一个方法都会触发invoke方法
        list.add("拜登");
        list.add("希拉里");
        list.add("奥巴马");
        System.out.println("集合："+ list);
    }

}

```



#### proxy类实现动态代理

##### 需求

使用proxy类实现动态代理，代理AccountService的所有方法，要求所有的方法开始执行前做事务的管理

##### 要求：

​	proxy实现动态代理，要求被代理对象必须要实现接口。

##### 实现

```java
package com.itheima.utils;

import com.itheima.service.AccountService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

//使用proxy类实现动态代理，代理AccountService的所有方法，要求所有的方法开始执行前做日志记录。

public class BeanFactory {


    //创建一个方法得到AccountServiceImpl的代理对象
    public static AccountService getAccountServiceProxy(){
        //1. 得到被代理对象
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:bean.xml");
        AccountService accountService = (AccountService) context.getBean("accountService");

        AccountService accountServiceProxy = (AccountService) Proxy.newProxyInstance(BeanFactory.class.getClassLoader(), accountService.getClass().getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                try {
                    System.out.println("开始事务");
                    //1. 直接使用被代理对象执行当前的方法
                    Object result =  method.invoke(accountService,args);
                    System.out.println("提交事务....");
                    return result;
                } catch (Exception e) {
                    e.printStackTrace();
                    System.out.println("事务的回滚...");
                } finally {
                    System.out.println("关闭资源...");
                }
                return null;
            }
        });
        return accountServiceProxy;
    }


    public static void main(String[] args) {
        //通过getAccountServiceProxy方法得到的AccountService的代理对象的方法全部已经被增强
        AccountService accountServiceProxy = getAccountServiceProxy();
        accountServiceProxy.add();
        System.out.println("查询的对象："+ accountServiceProxy.findAll());

    }

}

```





#### Enhancer实现动态代理（CGLIB）

##### 需求

使用enhancer类实现动态代理，给AccountServiceImpl类的所有方法进行增强，添加事务管理

##### 要求：

​	要求被代理的对象所属类可以没有实现任何接口，只需要要求该类不是final修饰的即可。



**实现:**

```java
package com.itheima.utils;

import com.itheima.service.AccountService;
import com.itheima.service.impl.AccountServiceImpl;
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
/*
需求： 使用enhancer类实现动态代理，给AccountServiceImpl类的所有方法进行增强，添加事务管理

Enhancer好处： 这种方式实现的动态代理是不需要依赖接口的，因为底层是通过继承的方式实现。

 */


public class BeanFactory2 {


    //创建一个方法得到AccountServiceImpl的代理对象
    public static AccountService getAccountServiceProxy(){
        //1. 得到被代理对象
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:bean.xml");
        AccountService accountService = (AccountService) context.getBean("accountService");
        /*
            superClass： 被代理对象的类
            MethodInterceptor: 相当于Proxy中InvocationHandler一样，处理器
         */
        AccountService accountServiceProxy = (AccountService) Enhancer.create(AccountServiceImpl.class, new MethodInterceptor() {
            /*
                method ； 当前调用的方法
                objects: 调用方法的实参
             */
            @Override
            public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                try {
                    System.out.println("开启事务..");
                    //被代理对象执行原始方法
                    Object result = method.invoke(accountService,objects);
                    System.out.println("提交事务");
                    return result;
                } catch (Exception e) {
                    e.printStackTrace();
                    System.out.println("回滚事务..");
                } finally {
                    System.out.println("关闭资源..");
                }
                return null;
            }
        });
        return accountServiceProxy;
    }


    public static void main(String[] args) {
        //通过getAccountServiceProxy方法得到的AccountService的代理对象的方法全部已经被增强
        AccountService accountServiceProxy = getAccountServiceProxy();
        accountServiceProxy.add();
        System.out.println("查询的对象："+ accountServiceProxy.findAll());

    }

}

```

