---
title: 常见面试题总结(06)-Mybatis框架相关的常见面试题
tags:
  - 笔记
  - 就业技术加强
  - 源码分析
  - Mybatis
  - 常见面试题总结
categories:
  - 常见面试题总结
abbrlink: 48214
date: 2021-01-29 23:29:13
---

## **问题1**： 谈谈你对MyBatis框架的理解?

1. MyBatis是一个持久层框架，它主要是对jdbc操作数据库的封装。 
   目的: 让持久层开发变得更加方便与简单。
2. MyBatis框架有两种配置文件: 核心配置文件、sql语句映射文件
3. MyBatis框架支持两种方式访问数据库: 传统API方式、mapper接口访问方式(面向接口编程)
4. MyBatis框架为了查询性能好，提供了一级缓存与二级缓存
5. MyBatis框架开发持久层的要点就是: 定义sql语句、在mapper接口定义对应的方法

## **问题2**： MyBatis框架执行流程能和我说一下吗?

1. 加载mybatis配置文件
2. 通过mybatis环境等配置信息构造SqlSessionFactory即会话工厂；
3. 由会话工厂创建sqlSession即会话，操作数据库需要通过sqlSession进行；
4. 利用接口Mapper或者sqlSession中原生方法操作数据库；
5. 在解析配置过程中会创建MappedStatement对象，它包装了mybatis配置信息及sql映射信息等。mapper.xml文件中一个sql对应一个MappedStatement对象；
6. MappedStatement对sql执行输入参数进行定义，Executor通过 MappedStatement在执行sql前将输入的java对象映射至sql中，输入参数映射就是jdbc编程中对preparedStatement设置参数；
7. MappedStatement对sql执行输出结果进行定义，Executor通过 MappedStatement在执行sql后将输出结果映射至java对象中，输出结果映射过程相当于jdbc编程中对结果的解析处理过程。



## **问题3**：java程序可以通过jdbc操作数据库,为什么还需要用MyBatis框架呢?

**题目背景: 考察对MyBatis框架的认识**

**答题思路:**
jdbc开发存在的问题:

1. sql语句直接在java代码中写死了，是硬编码，不方便维护，不灵活
2. jdbc程序操作数据库，对结果集封装处理很麻烦，有很多重复代码
3. jdbc程序每次访问操作数据库，都需要创建一个Connection连接对象，
   	    操作完成需要释放Connection对象。造成系统资源浪费

MyBatis框架的优点:

1. MyBatis框架通过xml配置文件的方式，配置sql语句。
   让sql语句与java代码分离，结构清晰，方便维护
2. MyBatis框架提供了自动将sql语句执行的结果，映射封装到java对象上，减少了重复代码的编写
3. MyBatis框架提供了连接池功能，可以复用Connection对象，支持更好的利用系统资源	
   开发方便，重用性高



## **问题4**： 描述MyBatis框架支持的开发方式?

**题目背景: 考察对mybatis框架使用是否熟悉**

**答题思路:**
	MyBatis框架支持两种开发的方式:
	第一种: 传统提供的API方式，即通过SqlSession.insert|update|delete|select方式
	第二种: mapper接口代理开发方法，即开发的时候编写一个接口，和对应的接口映射文件，在运行
	       时，MyBatis框架通过动态代理技术，创建接口的代理对象，完成数据库操作。

## **问题5**: MyBatis框架的mapper接口开发，是否支持方法的重载?

**题目背景: 考察对mybatis框架是否有深入的一些理解**

**答题思路: **

1. mybatis框架的mapper接口开发原则：

   1.1 mapper接口映射文件中的【namespace】属性值，必须是mapper接口的全限定名称
   【包名+类名】
   1.2 mapper接口映射文件中的【id】属性值，与mapper接口中的【方法名称】一致
   1.3 mapepr接口映射文件中的【resultType】属性值，与mapper接口中【方法返回值】一致

2. 那么就是说，在mybatis框架中，找到目标方法的唯一标识是：
   namesapce+id（这里的id就是接口方法名称）		

3. 在java语言中方法重载指的是方法名称一样，参数类型或者个数不一样。结合2和3点，

最终结论mapper接口中方法不支持重载



## **问题6**: MyBatis框架中#{}与${}区别?

**题目背景: 考察对mybatis框架使用是否熟悉**
select * from account where id = ${id} ==> select * from account where id = 1
select * from account where id = #{id} ==> select * from account where id = 1

答题思路：

1. 在mybatis框架中,#{}是一个占位符，相当于jdbc中的问号【?】，在执行的时候底层是通过
   PreparedStatement进行预编译，和参数设置值。可以有效防止sql注入
2. 在mybatis框架中，${}是一个字符串拼接符，在执行sql语句预编译之前就会替换。
     会有sql注入的风险	
3. 在使用上的区别：
     #{}: 在sql语句where条件、set修改、values()添加时使用。
     ${}: sql语句前面部分使用。(数据库列名、表名)

## **问题7**：描述MyBatis框架中有哪些Executor执行器?

**题目背景: 考察对mybatis框架是否有深入的一些理解**

答题思路:

1. 在mybatis框架中，有四种类型的Executor执行器:
   		SimpleExecutor、ReuseExecutor、BatchExecutor、CachingExecutor2.SimpleExecutor:
2. 特点：每次执行select|update|insert|delete操作，底层都会创建一个Statement对象，
   	      执行完成立即关闭Statement对象	
3. ReuseExecutor:
   	特点：每次执行select|update|insert|delete操作，都会以当前的sql语句为key，
   	     查找缓存中是否有Statement对象，如果有直接使用；如果没有则创建一个Statement
   	     对象。简单理解，可以重复使用Statement对象
4. BatchExecutor:
   	特点：底层通过jdbc的addBatch执行批处理操作
5. CachingExecutor:
   	特点：在以上三个Executor执行器的基础上，支持二级缓存



## **问题8**: 描述MyBatis框架的缓存支持?

**题目背景: 考察对mybatis框架是否有深入的一些理解**
答题思路:

1. 在mybatis框架中，支持一级缓存和二级缓存
2. 一级缓存是指SqlSession级别的缓存，在同一个会话SqlSession内部有效，每一次数据库操作一
   	  级缓存都存在。
3. 二级缓存是指SqlSessionFactory级别的缓存，二级缓存的范围更大，可以在不同的
   	  SqlSession之间共享。二级缓存默认是全局开启的；但是应用时需要再mapper文件中添加`<cache/>`。
4. 在企业项目中，一般不推荐直接使用mybatis框架的二级缓存，原因是二级缓存以 命名空间 为单位；如果其它的命名空间也同样使用了某张表数据，那么在更新之后；原来的缓存是不会发生改变的，所以会存在读取脏数据的可能。
5. 在实际企业项目中，可以考虑在业务层，结合相关的缓存中间件实现缓存服务。常用的缓存中间件有（redis、memcached）

