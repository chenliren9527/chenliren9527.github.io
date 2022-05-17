---
title: 就业技术加强(06)-源码分析(2)-Mybatis
tags:
  - 笔记
  - 就业技术加强
  - 源码分析
  - Mybatis
categories:
  - 就业技术加强
date: 2021-01-29 18:27:00
---

## 0、课程目标

- 理解mybatis框架执行流程

- 掌握通过debug跟踪学习源码方式

- 熟悉mybatis框架的核心组件

- 理解mybatis框架整体架构设计

- 掌握mybatis框架相关的常见面试题



## 1、执行流程

### 1.1、准备工作

1）导入演示数据；创建mysql数据库 `mybatis-demo` 将 `资料\数据库数据\mybatis-demo.sql`文件导入到刚刚创建的数据库中。可看到只有一张非常简单的表 `tb_user`

2）导入演示工程；创建IDEA空工程，将 `资料\演示工程\mybatis-demo` 导入IDEA中。

### 1.2、执行流程

![mybatis](就业技术加强-06-源码分析-2-Mybatis.assets/mybatis01.png)

+ **第1步: 加载配置文件**
  + 核心配置文件: mybatis-config.xml (配置类型别名、环境、设置信息等)
  + sql语句映射文件：UserMapper.xml (定义CRUD语句、缓存等)
+ **第2步: 解析配置文件**
  + 用SAX解析XML文件中的内容，然后把内容全部封装到Configuration对象

  + 创建SqlSessionFactory，把Configuration对象作为参数传入

+ **第3步: 获取SqlSession对象**
  + 调用DefaultSqlSessionFactory获取操作数据库会话对象SqlSession
  + 创建DefaultSqlSession，把Executor、Configuration对象作为参数传入
+ **第4步: 通过SqlSession操作数据库**
  + SqlSession直接通过相关api方法，操作数据库
  + SqlSession获取mapper接口代理对象，操作数据库（底层还是SqlSession调用api方法）
+ **第5步: 通过Executor执行器，操作数据库**
  + 从MappedStatement对象中获取BoundSql对象(sql语句与参数)
  + 操作二级缓存 与 一级缓存
  + 设置sql语句预编译 与 sql语句参数
  + 调用StatementHandler对象访问数据库
  + 调用ResultSetHandler处理Result结果集，完成数据封装与映射

## 2、配置解析

在mybatis中配置解析主要是解析 `mybatis-config.xml`总配置文件及 `mappers.xml` 映射配置文件。那么要了解配置文件的解析需要从配置文件的加载作为起始断点。

![image-20210103164935253](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103164935253.png)



### 2.1、DEBUG操作说明

在开始调试之前，简单回顾一下 IDEA 中 debug 调整中各个操作的表示意思：

![image-20210103161246274](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103161246274.png)

```
1、显示执行点：回到当前断点执行处，常用在调试多处时要回到当下断点执行地方的时候
2、执行并跳到下一行：执行完当前断点所在行并跳转到下一行
3、（步入）进入方法：根据当前行进入对应的方法，如果是多个方法则可选择进入；所以是进入某个方法的意思
4、（强制步入）进入方法；与上述一样；只是它可以进入JDK等原生方法，能力更强
5、（步出）退出方法：与步入是反操作，可以在进入方法之后退出该方法并跳转到进入方法的入口处（不能再进入方法执行）
6、回退上一步：可以回退到进入某个方法入口处并可重新进入方法；常用于需要多次调试进入某个方法的时候
7、执行到光标处：将断点执行到光标所在行
8、表达式计算：可以对断点过程中的变量、表达式等进行计算
```



### 2.2、properties解析

从起始断点了解对于 `mybatis-config.xml` 文件中的 `properties`的解析源码

![image-20210103165219360](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103165219360.png)

![image-20210103170023067](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103170023067.png)

![image-20210103170030377](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103170030377.png)

![image-20210103170052379](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103170052379.png)

![image-20210103170104539](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103170104539.png)

![image-20210103170647175](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103170647175.png)

>如果有一个同名的属性在4个地方都指定了；那么优先级为：
>
>1、代码中设置的配置属性；
>
>2、url中设置的配置属性；
>
>3、resource中设置的配置属性；
>
>4、子标签property中设置的配置属性；
>
>**问题**：一个公司开发时为了衔接旧系统，如果要同时可用 url 和 resource 两个属性对应的配置文件，该怎么办？

### 2.3、settings解析

![image-20210103175506541](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103175506541.png)

![image-20210103175851440](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103175851440.png)

### 2.4、environments解析

![image-20210103175906612](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103175906612.png)

![image-20210103175915851](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103175915851.png)

### 2.5、mappers解析

![image-20210103183546292](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103183546292.png)

![image-20210103183720602](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103183720602.png)

![image-20210103183753752](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103183753752.png)

![image-20210103183902082](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103183902082.png)

![image-20210103183932867](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103183932867.png)

![image-20210103184041919](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103184041919.png)

![image-20210103184344790](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103184344790.png)

![image-20210103184351565](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103184351565.png)

创建完毕 `configuration` 对象之后，根据这些解析的数据配置信息构建 `SqlSessionFactory`

![image-20210103185053999](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103185053999.png)

![image-20210103185113989](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103185113989.png)

**解析流程小结**：

```
第一步: 通过MyBatis框架的Resources.getResourceAsStream()方法把MyBatis框架的核心配置文件，加载到内存中。底层通过类加载器找到该文件，再通过io流把配置文件读取到内存中。

第二步: 当调用SqlSessionFactoryBuilder.build()方法构建SqlSessionFactory会话工厂对象时，它会创建XMLConfigBuilder对象，它底层封装了SAX，用该对象的parse方法来解析MyBatis的核心配置文件，并且会把解析好的数据设置到Configuration对象中。
    比如: 
    - <properties/>元素  设置到Configuration的variables属性中。
    - <typeAliases/>元素  设置到Configuration的typeAliasRegistry属性中。
    - <settings/>元素  设置到Configuration的多个属性中。
    - <environment/>元素  设置到Configuration的environment属性中。
    - <mappers/>元素  设置到Configuration的mapperRegistry与mappedStatements属性中。
       mapperRegistry: 存储数据访问接口信息
       mappedStatements: 该map集合存储sql映射语句信息
       里面的每个MappedStatement，封装维护了一个<select|update|delete|insert>节点.
       MappedStatement对象中的SqlSource属性封装了sql语句与参数信息。
   
 最后用SqlSessionFactory接口实现类DefaultSqlSessionFactory来创建SqlSessionFactory对象。它需要Configuration配置信息对象作为构建参数。
 
```

### 2.6、SqlSessionManager了解

另外一个实现了SqlSessionFactory的SqlSessionManager提供了一个本地线程变量，每当通过startManagedSession()获得session实例的时候，会在本地线程保存session实例。因为SqlSessionManager同时实现了SqlSessionFactory和SqlSession接口，所以可以直接使用sqlSession中对应的方法；如果开启了startManagedSession()方法，那么每次使用sqlSession中对应的方法对应的sqlSession都是在本地线程保存session实例，也就是同一个。

也就是多次使用SqlSessionManager的时候，在同一个线程都是使用1个SqlSession。

1）**使用**

```java
        SqlSessionManager sqlSessionManager = SqlSessionManager.newInstance(sqlSessionFactory);
        sqlSessionManager.startManagedSession();
        User user1 = sqlSessionManager.selectOne("com.itheima.mybatis.mapper.UserMapper.findById", 1);
        System.out.println(user1);
        sqlSessionManager.close();

```

2）**源码**

![image-20210103191520009](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103191520009.png)

![image-20210103191531518](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103191531518.png)

![image-20210103191541524](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210103191541524.png)

> SqlSessionManager 同时实现了SqlSessionFactory和SqlSession接口，所以可以直接使用sqlSession中对应的方法。在开启的时候将sqlSession绑定到本地线程变量中，在执行sqlSession的方法的时候使用动态代理，先获取本地线程变量中的sqlSession，然后再利用其执行对应的方法。在同一个线程中第二次使用的时候也是同一个sqlSession。关闭的时候清空本地线程变量。

## 3、执行过程

### 3.1、创建SqlSession

![image-20210104113857296](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104113857296.png)

![image-20210104113857296](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104113917724.png)

![image-20210104114034354](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104114034354.png)

![image-20210104114051268](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104114051268.png)

![image-20210104114109737](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104114109737.png)

![image-20210104114120142](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104114120142.png)

![image-20210104114129363](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104114129363.png)

最终创建 `DefaultSqlSession`

![image-20210104114138243](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104114138243.png)

### 3.2、执行原生查询

先查看直接使用了 `sqlSession.selectOne()` 方法，也就是直接使用`sqlSession`原生方法进行查询的源码执行过程：

![image-20210104120157436](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104120157436.png)

![image-20210104120211221](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104120211221.png)

![image-20210104120225866](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104120225866.png)

![image-20210104120246517](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104120246517.png)

![image-20210104155336912](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104155336912.png)

![image-20210104155442561](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104155442561.png)

![image-20210104155503280](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104155503280.png)

![image-20210104160336642](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104160336642.png)

![image-20210104160352655](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104160352655.png)

![image-20210104160410567](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104160410567.png)

![image-20210104160503897](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104160503897.png)

>如果要开启二级缓存（mapper级别缓存）
>
>1、在Mapper.xml映射文件中添加 <cache/>
>
>2、在处理完上一次的查询之后需要 sqlSession.commit() 提交将结果缓存起来；其它sqlSession再次查询时可从缓存中取数据
>
>注意：一级缓存（sqlSession级别）是默认开启，只要是同一个sqlSession的同条件查询都会使用缓存

### 3.3、执行接口查询

1）获取Mapper注册对象并创建Mapper代理对象

![image-20210104171547318](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104171547318.png)

![image-20210104171557248](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104171557248.png)

![image-20210104171603068](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104171603068.png)

![image-20210104171609214](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104171609214.png)

![image-20210104172331531](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104172331531.png)

2）**执行查询**

![image-20210104173710822](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104173710822.png)

![image-20210104173826926](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104173826926.png)

![image-20210104173851455](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104173851455.png)

![image-20210104173857284](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104173857284.png)

![image-20210104173911303](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104173911303.png)

![image-20210104174005663](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104174005663.png)

![image-20210104174053950](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104174053950.png)

![image-20210104174104397](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104174104397.png)

![image-20210104174118690](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104174118690.png)

> 发现使用了接口查询的做法在底层也是使用 sqlSession对应的方法在执行。

## 4、架构设计

### 4.1、核心组件

![1569290611885](就业技术加强-06-源码分析-2-Mybatis.assets/mybatis02-1609753999232.png) 

| 序号 | 组件名称                 | 组件描述                                                     |
| ---- | ------------------------ | ------------------------------------------------------------ |
| 1    | Configuration            | 用于封装MyBatis框架的所有配置信息                            |
| 2    | SqlSessionFactoryBuilder | 用于解析封装核心配置文件的内容，构建SqlSessionFactory核心会话工厂对象 |
| 3    | SqlSessionFactory        | 会话工厂接口，底层封装了Configuration配置信息对象，通过工厂方法openSession创建SqlSession对象 |
| 4    | SqlSession               | 会话接口，底层封装了事务对象、执行器，与数据库交互的会话，提供了数据库增删改查的操作方法 |
| 5    | Executor                 | 执行器接口，负责事务管理、数据库连接、缓存的维护、sql语句预编译、设置参数、调用StatementHandler处理器 |
| 6    | StatementHandler         | 语句处理器接口，负责封装jdbc的三个Statement接口，直接jdbc操作数据库。 |
| 7    | MappedStatement          | 每个MappedStatement，封装了一个<select\|update\|delete\|insert>节点映射语句信息 |
| 8    | SqlSource                | sql源接口,根据用户传递的parameterObject，动态生成sql语句，将信息封装到BoundSql对象中 |
| 9    | BoundSql                 | 封装sql语句、参数映射、参数值信息                            |
| 10   | ParameterHandler         | 参数处理器接口，负责sql语句中所需要的参数处理                |
| 11   | ResultSetHandler         | 结果集处理器接口，负责对ResultSet结果集进行进行处理          |
| 12   | TypeHandler              | 类型处理器，完成java数据类型和jdbc数据类型之间的映射和转换   |

**Executor 执行器类图**

![image-20210104184204347](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104184204347.png)

**StatementHandler 语句处理器类图**

![image-20210104193542261](就业技术加强-06-源码分析-2-Mybatis.assets/image-20210104193542261.png)

### 4.3、架构

![img](就业技术加强-06-源码分析-2-Mybatis.assets/wps1.jpg)

**接口层**：提供给外部使用的接口API，开发人员通过这些本地API来操纵数据库。接口层一接收到调用请求就会调用数据处理层来完成具体的数据处理。

 

**数据处理层**：负责具体的SQL查找、SQL解析、SQL执行和执行结果映射处理等。主要目的是根据调用的请求完成一次数据库操作。

 

**支撑层**：负责最基础的功能支撑，包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的东西。为上层的数据处理层提供最基础的支撑。

## 5、常见面试题

**问题1**： 谈谈你对MyBatis框架的理解?

```txt
1. MyBatis是一个持久层框架，它主要是对jdbc操作数据库的封装。 
   目的: 让持久层开发变得更加方便与简单。
2. MyBatis框架有两种配置文件: 核心配置文件、sql语句映射文件
3. MyBatis框架支持两种方式访问数据库: 传统API方式、mapper接口访问方式(面向接口编程)
4. MyBatis框架为了查询性能好，提供了一级缓存与二级缓存
5. MyBatis框架开发持久层的要点就是: 定义sql语句、在mapper接口定义对应的方法
```

**问题2**： MyBatis框架执行流程能和我说一下吗?

```txt
1、加载mybatis配置文件
2、通过mybatis环境等配置信息构造SqlSessionFactory即会话工厂；
3、由会话工厂创建sqlSession即会话，操作数据库需要通过sqlSession进行；
4、利用接口Mapper或者sqlSession中原生方法操作数据库；
5、在解析配置过程中会创建MappedStatement对象，它包装了mybatis配置信息及sql映射信息等。mapper.xml文件中一个sql对应一个MappedStatement对象；
6、MappedStatement对sql执行输入参数进行定义，Executor通过 MappedStatement在执行sql前将输入的java对象映射至sql中，输入参数映射就是jdbc编程中对preparedStatement设置参数；
7、MappedStatement对sql执行输出结果进行定义，Executor通过 MappedStatement在执行sql后将输出结果映射至java对象中，输出结果映射过程相当于jdbc编程中对结果的解析处理过程。
```



**问题3**：java程序可以通过jdbc操作数据库,为什么还需要用MyBatis框架呢?

```txt
题目背景: 考察对MyBatis框架的认识

答题思路:
jdbc开发存在的问题:
	1. sql语句直接在java代码中写死了，是硬编码，不方便维护，不灵活
	2. jdbc程序操作数据库，对结果集封装处理很麻烦，有很多重复代码
	3. jdbc程序每次访问操作数据库，都需要创建一个Connection连接对象，
	    操作完成需要释放Connection对象。造成系统资源浪费

MyBatis框架的优点:
	1.MyBatis框架通过xml配置文件的方式，配置sql语句。
	  让sql语句与java代码分离，结构清晰，方便维护
	2.MyBatis框架提供了自动将sql语句执行的结果，映射封装到java对象上，减少了重复代码的编写
	3.MyBatis框架提供了连接池功能，可以复用Connection对象，支持更好的利用系统资源
	
开发方便，重用性高
```



**问题4**： 描述MyBatis框架支持的开发方式?

```txt
题目背景: 考察对mybatis框架使用是否熟悉

答题思路:
	MyBatis框架支持两种开发的方式:
	第一种: 传统提供的API方式，即通过SqlSession.insert|update|delete|select方式
	第二种: mapper接口代理开发方法，即开发的时候编写一个接口，和对应的接口映射文件，在运行
	       时，MyBatis框架通过动态代理技术，创建接口的代理对象，完成数据库操作。
```

**问题5**: MyBatis框架的mapper接口开发，是否支持方法的重载?

```txt
题目背景: 考察对mybatis框架是否有深入的一些理解

答题思路: 
	1.mybatis框架的mapper接口开发原则：
	  1.1 mapper接口映射文件中的【namespace】属性值，必须是mapper接口的全限定名称
	     【包名+类名】
	  1.2 mapper接口映射文件中的【id】属性值，与mapper接口中的【方法名称】一致
	  1.3 mapepr接口映射文件中的【resultType】属性值，与mapper接口中【方法返回值】一致
		
	2.那么就是说，在mybatis框架中，找到目标方法的唯一标识是：
		namesapce+id（这里的id就是接口方法名称）
		
	3.在java语言中方法重载指的是方法名称一样，参数类型或者个数不一样。结合2和3点，
	  
最终结论mapper接口中方法不支持重载
```



**问题6**: MyBatis框架中#{}与${}区别?

```txt
题目背景: 考察对mybatis框架使用是否熟悉
select * from account where id = ${id} ==> select * from account where id = 1
select * from account where id = #{id} ==> select * from account where id = 1

答题思路：
	1.在mybatis框架中,#{}是一个占位符，相当于jdbc中的问号【?】，在执行的时候底层是通过
	  PreparedStatement进行预编译，和参数设置值。可以有效防止sql注入
	
    2.在mybatis框架中，${}是一个字符串拼接符，在执行sql语句预编译之前就会替换。
      会有sql注入的风险
      
    3.在使用上的区别：
      #{}: 在sql语句where条件、set修改、values()添加时使用。
      ${}: sql语句前面部分使用。(数据库列名、表名)
```

**问题7**：描述MyBatis框架中有哪些Executor执行器?

```txt
题目背景: 考察对mybatis框架是否有深入的一些理解

答题思路:
	1.在mybatis框架中，有四种类型的Executor执行器:
		SimpleExecutor、ReuseExecutor、BatchExecutor、CachingExecutor
	
	2.SimpleExecutor:
		特点：每次执行select|update|insert|delete操作，底层都会创建一个Statement对象，
		      执行完成立即关闭Statement对象
		
	3.ReuseExecutor:
		特点：每次执行select|update|insert|delete操作，都会以当前的sql语句为key，
		     查找缓存中是否有Statement对象，如果有直接使用；如果没有则创建一个Statement
		     对象。简单理解，可以重复使用Statement对象
		
	4.BatchExecutor:
		特点：底层通过jdbc的addBatch执行批处理操作
		
	5.CachingExecutor:
		特点：在以上三个Executor执行器的基础上，支持二级缓存
```

**问题8**: 描述MyBatis框架的缓存支持?

```txt
题目背景: 考察对mybatis框架是否有深入的一些理解
答题思路:
1.在mybatis框架中，支持一级缓存和二级缓存
2.一级缓存是指SqlSession级别的缓存，在同一个会话SqlSession内部有效，每一次数据库操作一
	  级缓存都存在。
3.二级缓存是指SqlSessionFactory级别的缓存，二级缓存的范围更大，可以在不同的
	  SqlSession之间共享。二级缓存默认是全局开启的；但是应用时需要再mapper文件中添加<cache/>。
4.在企业项目中，一般不推荐直接使用mybatis框架的二级缓存，原因是二级缓存以 命名空间 为单位；如果其它的命名空间也同样使用了某张表数据，那么在更新之后；原来的缓存是不会发生改变的，所以会存在读取脏数据的可能。
5.在实际企业项目中，可以考虑在业务层，结合相关的缓存中间件实现缓存服务。常用的缓存中间件有（redis、memcached）
```

