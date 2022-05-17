---
title: maven基础
tags:
  - 笔记
  - maven
  - 动态代理
categories:
  - maven
date: 2020-10-21 12:27:00
---

## 01.学习目标



1)	能够了解Maven的作用
2)	能够理解Maven仓库的作用
3)	能够理解Maven的坐标概念
4)	能够掌握Maven的安装
5)	能够掌握IDEA配置本地Maven
6)	能够使用IDEA创建JavaWeb的Maven工程
8)	能够掌握依赖引入的配置方式
9)	能够了解依赖范围的概念



## 02.Maven介绍与作用

### 目标

理解maven介绍



### 直接复制jar包到lib目录内的问题?

> 问题1： 到处项目复制jar包，导致磁盘jar包冗余
>
> 问题2：版本冲突
>
> 问题3： 如果需要的ar包本地没有，需要自己去网络查找下载比较麻烦
>
> 总结：jar包的管理冗余，重用性不好



### maven介绍

==maven是项目管理构建工具==，maven是apache基金会开发的开源的构建jar的工具或服务器。可以非常方便的获取jar包，构建项目使用。并且还可以管理项目打包，测试，安装，上传。



### maven的作用

1. jar包的管理【核心功能】

   > 以后所有的jar包都找maven获取

2. 项目管理工具（项目生命周期管理）

   > maven可以对象项目生成、编译、测试、打包、安装、管理项目
   >
   > 清理：删除编译的文件夹(target)
   >
   > 编译：将编写java源代码编译成class文件
   >
   > 测试：编译并运行所有junit单元测试
   >
   > 打包：将项目进行打包
   >
   > 安装：将打包的项目安装到本地的maven里面给内自己项目使用

3. 创建聚合项目（以后maven高级的时候讲解）【核心功能】

   > 将多个模块构建成一个模块去使用

### 小结

* maven是什么?

  ==mave是什么： maven项目管理构建工具==

* **maven的功能有哪些？**

  - 管理项目

    - 管理项目的jar
    - 项目的生命周期（maven可以对项目进行清理、编译、测试、打包、安装）

  - 构建项目(多个模块聚合成一个项目)

    ​	

## 03.Maven的仓库【理解】

### 目标

1、理解maven仓库的作用

2、理解maven仓库的类型



### Maven仓库介绍

maven仓库里面拥有所有的jar包，以后项目使用jar包就从maven仓库获取



### 项目使用Maven管理jar包构建项目原理

![1603027196176](maven基础.assets/1603027196176.png)

### Maven仓库类型

![1547600211217](maven基础.assets/1547600211217.png)

官方中央仓库地址：<https://repo1.maven.org/maven2/>

> 官方允许浏览器和maven服务器访问
>
> 缺点:  中央仓库是国外服务器, 下载较慢

第三方阿里云仓库：http://maven.aliyun.com/nexus/content/groups/public/

> 注意，阿里云不对浏览器访问开放，只对maven服务器访问开发
>
> 优点:  一般都是国内的服务器, 下载速度快

> 注意:  第三方与中央仓库都有共同的缺点,  只能下载不能上传

私服[maven高级的时候讲解]

> 介绍: 就是自己公司搭建的服务器
>
> 优点: 不仅可以下载还可以上传
>
> 经验:  一般大公司都使用自己私服 ,  团队内可以下载, 又可以将好用的jar上传到服务器中共享给团队其他成员使用

### 本地仓库的说明

![image-20200309094102162](maven基础.assets/image-20200309094102162.png)

![image-20200309094126208](maven基础.assets/image-20200309094126208.png)

![image-20200309094214221](maven基础.assets/image-20200309094214221.png)

### 小结

1. **仓库的作用**

   - maven的仓库是用于存储jar

   

2. **maven找jar的顺序**

   情况1：maven-----本地仓库

   情况2： maven----本地仓库---------远程仓库

   

3. **远程仓库有哪几种？**

   - maven官方： 中央仓库
   - 第三方：阿里云
   - 私服

   

## 04.Maven的坐标【理解】

### 目标

掌握坐标的作用

理解坐标的组成部分



### 为什么学习坐标？

因为仓库里面的jar包有很多，需要通过坐标获取里面指定的jar包 , ==坐标是jar在仓库的唯一标识==





### 坐标由3部分组成

![1547600948690](maven基础.assets/1547600948690.png)

groupId，对应仓库里面第一层目录

artifactId，对应仓库里面第二层目录

version，对应仓库里面第三层目录

### 坐标配置放在哪里？

> pom.xml配置文件中，每个模块项目都有这个配置文件
>
> （Project Object Model） 项目对象模型， 因为这个配置文件将每个jar包看成一个对象模型进行管理操作，在配置文件中可以对每个jar包进行灵活管理配置



### 引用junit的jar包坐标例子

引用仓库里面的junit的jar包,在配置文件pom.xml中做如下配置:

```xml
<dependency>   					<!--依赖，就是导入jar包的含义  -->
<groupId>junit</groupId>        <!--组织名  -->
<artifactId>junit</artifactId>  <!--项目名.模块名  -->
<version>4.12</version>			<!--版本号  -->
<scope>test</scope>				<!--依赖范围，后面讲解  -->
</dependency>
```

![1547601240806](maven基础.assets/1547601240806.png)&nbsp;

本地仓库的jar包位置

![1547601274264](maven基础.assets/1547601274264.png)&nbsp;

<font color=red>注意：<br>一般情况下:  坐标中groupId对应仓库中jar包位置的第一层目录<br>特殊情况下:  如果groupId配置的目录名字中有"."点,  就代表多层目录, 比如: groupId的信息为javax.servlet,  多一个一个点就会多一层目录<br>其他artifactId与version:  无论配置什么信息, 每个标签都只对应一层目录</font>

![1547601337151](maven基础.assets/1547601337151.png)&nbsp;

##### 小结

* **坐标的作用**
  	- 坐标是定位一个jar的唯一标识
* **坐标的组成**
  - groupId  组织名
  - artifactId  项目名
  - version 版本


## 05.Maven的安装与配置【应用】

### 目标

掌握maven的安装

掌握maven配置本地仓库与远程仓库



### 实现步骤

1. 下载maven（素材中提供）

   官网地址：http://maven.apache.org/download.cgi

   maven是apache开源项目，专门为java提供的项目构建工具

   本地提供

   ![1552441861870](maven基础.assets/1552441861870.png)

2. 解压完成安装

   ![1552441878999](maven基础.assets/1552441878999.png)

3. 介绍maven服务器的目录结构

   ![image-20200818095155642](maven基础.assets/image-20200818095155642.png)

   ![1552442092919](maven基础.assets/1552442092919.png)

   ![1552442131546](maven基础.assets/1552442131546.png)

4. 配置maven全局配置文件conf/settings.xml

   * 绑定本地仓库

     ![1552442193580](maven基础.assets/1552442193580.png)

     ```xml
     <localRepository>本地仓库地址</localRepository>
     ```

   * 绑定远程仓库，阿里云

     > 远程仓库的地址在笔记md上复制，不要再pdf上复制，否则莫名其妙会加上中文空格

     ```xml
     	<mirror>
     	 <id>nexus</id>
     	 <mirrorOf>*</mirrorOf> 
     	 <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
     	</mirror>	
     ```

     将上面信息配置到如下位置

     ![1552442423314](maven基础.assets/1552442423314.png)

   * 配置全局的jdk编译语言级别版本（如果不配这个，默认编译级别是1.5，太低，idea运行的会有警告）

     ```xml
      <profile>
         <id>development</id>
         <activation>
           <jdk>1.8</jdk>
           <activeByDefault>true</activeByDefault>
         </activation>
         <properties>
           <maven.compiler.source>1.8</maven.compiler.source>
           <maven.compiler.target>1.8</maven.compiler.target>
           <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
         </properties>
       </profile>
     ```

     将上面内容添加到如下配置文件位置

     ![1552442631991](maven基础.assets/1552442631991.png)

5. 设置环境变量，到处都可以使用maven的命令

   ![1603244635285](maven基础.assets/1603244635285.png)

   

6. 测试是否安装配置成功，如下信息代表配置成功

   ![1552442824587](maven基础.assets/1552442824587.png)

### 小结

* **给maven配置环境变量有哪些?**

  - 配置MAVEN_HOME
  - 配置path环境变量

  

* **安装maven，给全局配置文件settings.xml配置什么信息？**

  - 本地仓库

- 远程仓库

  - 配置jdk编译的版本

  

## 06.IDEA绑定本地Maven【应用】

### 目标

1.掌握idea绑定maven工具，以后使用idea操作maven

2.使用idea设置当前工作空间（project项目）绑定maven

3.使用idea设置未来的工作空间（project项目）绑定maven



### 当前项目绑定Maven实现步骤

![1603245071828](maven基础.assets/1603245071828.png)

运行参数绑定

1. 设置运行参数

   -DarchetypeCatalog=internal  ，用于设置任何配置信息都从本地缓存中拿。有一些模板信息maven默认从远程仓库下载获取，如果设置了这个参数第一次从远程拿，以后从本地拿（这就要求大家第一次玩maven必须联网，1~5M不等）

   ```properties
   -DarchetypeCatalog=internal
   ```

   配置效果如下

   ![image-20200725102805575](maven基础.assets/image-20200725102805575.png)

==注意：如果启动maven的时候控制台是乱码，那么请配置编码 -DarchetypeCatalog=internal -Dfile.encoding=UTF-8==

配置自动刷新maven

![image-20200106143050374](maven基础.assets/image-20200106143050374.png)



### 配置新项目绑定Maven实现步骤

1. 绑定本地maven软件

   2020版本效果

   ![image-20200929104254232](maven基础.assets/image-20200929104254232.png)

   

   2017、2018、2019版本

   ![1603028595222](maven基础.assets/1603028595222.png)

   ![1603028635972](maven基础.assets/1603028635972.png)

![1567219417919](maven基础.assets/1567219417919.png)

![1603245788304](maven基础.assets/1603245788304.png)

### 小结

   idea关联maven做了哪些设置

		1. 先找到maven, 配置maven的路径与settings、本地仓库
		2. Runner, 
			3. import





## 07.Maven骨架web工程1-创建项目【了解】

### 目标

1、理解使用官方骨架方式创建maven工程

2、理解maven的web工程目录结构



### Maven骨架创建maven项目

创建module

![image-20200725104353139](maven基础.assets/image-20200725104353139.png) 

根据骨架创建模块

> 骨架：是maven官方apache提供的创建项目模板

![image-20200725104515239](maven基础.assets/image-20200725104515239.png)

设置坐标信息

![1603246339676](maven基础.assets/1603246339676.png)

下一步

![1603246358811](maven基础.assets/1603246358811.png)

设置磁盘目录名

![1603246452232](maven基础.assets/1603246452232.png)

version

成功信息

![image-20200818105200493](maven基础.assets/image-20200818105200493.png)

创建骨架项目初始化结构

![image-20200929110315669](maven基础.assets/image-20200929110315669.png) 

### maven项目目录结构规范

> maven工程以后是由maven管理的，对项目目录结构maven是由规范要求的适合进行管理
>
> ![1603247071676](maven基础.assets/1603247071676.png))

<img src=assets/image-20200525233332109.png width="50%"> 

分析

![image-20200929111130781](maven基础.assets/image-20200929111130781.png) 

必须有结构

![image-20200818105802124](maven基础.assets/image-20200818105802124.png) 

### 完善目录结构

手动创建的目录如下

![image-20200929111453082](maven基础.assets/image-20200929111453082.png) 

这是刷新前的效果

![image-20200818110239684](maven基础.assets/image-20200818110239684.png)

刷新后的效果

![image-20200929111622722](maven基础.assets/image-20200929111622722.png)   

> 注意：上面手动创建后需要在maven窗口进行刷新，让对应的目录变色，只有变色才说明目录结构正确

如果上面的maven窗口没有，可以点击如下出来

![image-20200818110417921](maven基础.assets/image-20200818110417921.png) 

### 小结

1、理解官方的骨架方式创建maven工程

​	   勾选：![image-20200308211151567](maven基础.assets/image-20200308211151567.png)

​		选着模板：![image-20200308211215771](maven基础.assets/image-20200308211215771.png)

2、理解maven的web工程目录结构

​	  

## 08.Maven骨架web工程2-pom.xml【记忆】

### 目标

理解pom文件的相关信息



### 介绍pom.xml文件

> 每个项目都有maven的核心配置文件，主要用于配置项目依赖的jar包坐标与使用的插件

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <!--当前模块的坐标-->
  <!--组织名-->
  <groupId>com.itheima</groupId>
  <!--项目名-->
  <artifactId>day35_01</artifactId>
  <!--版本-->
  <version>1.0-SNAPSHOT</version>

  <!--打包方式
      1. jar : javase项目使用jar
      2. war  javaweb的项目使用war
      3. pom  如果是多个模块聚合成一个项目，使用pom
  -->
  <packaging>war</packaging>


  <!--配置maven的编译版本 ， 刚刚已经在settings已经配置,所以该信息可以省略-->
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
  </properties>


  <!--配置依赖的， 每一个依赖就是dependency标签， -->
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>


  <!--maven的插件，maven的插件根据个人喜好，有些人不喜欢使用maven插件，有些人喜好使用maven插件 -->
  <build>
    <finalName>day35_01</finalName>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
        <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_war_packaging -->
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.8.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.22.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-war-plugin</artifactId>
          <version>3.2.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>

```

<font color=red>总结：骨架方式生成的pom.xml文件生成了很多没有用的信息</font>

### 小结

* **pom.xml中必须有哪些信息？**

- 项目的坐标信息
  - 项目的打包方式
  - 依赖



## 09.Maven骨架web工程3-开发Servlet【了解】

### 目标

在maven项目中开发servlet



### 创建Servlet

中main/java鼠标右键创建servlet



注意，勾上如图

![image-20200309111804486](maven基础.assets/image-20200309111804486.png)&nbsp;

### 代码效果

```java
package com.itheima.web;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/demo1")
public class Demo1Servlet extends javax.servlet.http.HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        out.write("哈哈");
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```



==发现上面servlet的类报错，是由于没有引用servlet的jar包==

### pom.xml导入Servlet的jar包引用依赖

```xml
 <dependency>      <groupId>javax.servlet</groupId>      <artifactId>javax.servlet-api</artifactId>      <version>3.1.0</version>    </dependency>
```

检查下maven窗口是否有导入

![image-20200818112550532](maven基础.assets/image-20200818112550532.png) 

截图

![image-20200725114012617](maven基础.assets/image-20200725114012617.png)

### 导入后观察Servlet已经不报错了

![image-20200725114026057](maven基础.assets/image-20200725114026057.png)

### DemoServlet的代码

输出:hello maven javaweb archetype

```java
package com.itheima.web;import javax.servlet.ServletException;import javax.servlet.annotation.WebServlet;import javax.servlet.http.HttpServletRequest;import javax.servlet.http.HttpServletResponse;import java.io.IOException;import java.io.PrintWriter;@WebServlet("/demo1")public class Demo1Servlet extends javax.servlet.http.HttpServlet {    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {        response.setContentType("text/html;charset=utf-8");        PrintWriter out = response.getWriter();        out.write("哈哈");    }    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {        doPost(request, response);    }}
```

### 部署项目运行

> maven工程需要手动部署项目到tomcat上

![1603030191715](maven基础.assets/1603030191715.png)

点击OK

![image-20200725114502271](maven基础.assets/image-20200725114502271.png)

启动项目

### 运行效果

访问欢迎页面

![1603248779200](maven基础.assets/1603248779200.png)



### 注意运行部署的位置

![1603248906842](maven基础.assets/1603248906842.png)

### 小结

* 开发Servlet的步骤

  0.搭建maven工程目录结构

  1.在pom.xml中引用servlet依赖jar包

  2.开发servlet代码

  3.手动部署项目到tomcat上

  4.运行访问

  



## 10.javaToweb的插件安装【应用】

1. **打开设置找到plugins**

   ![1603031164000](maven基础.assets/1603031164000.png)

2. 重启idea ,发现多了一个javaToweb的菜单

   ![1603031321838](maven基础.assets/1603031321838.png)

   









## 11. Maven自定义web工程【重点,每一位都需要写】

### 目标

了解使用idea自定义方式创建Maven工程



### 实现步骤

1. 创建maven项目，不用向导骨架

   ![1547606684341](maven基础.assets/1547606684341.png)


2. 输入组织名与项目名

   ![1603031411226](maven基础.assets/1603031411226.png)

   

   ### ![1603249453321](maven基础.assets/1603249453321.png)

==注意： 当前结构还没有不符合javaweb项目的结构==



3. 使用插件把当前模块转换为javaweb的项目

![1603249493797](maven基础.assets/1603249493797.png)

![1603249542577](maven基础.assets/1603249542577.png)



4. 开发servlet

   导入依赖

![1603249597560](maven基础.assets/1603249597560.png)



servlet代码

```java
package com.itheima.web;import javax.servlet.ServletException;import javax.servlet.annotation.WebServlet;import javax.servlet.http.HttpServlet;import javax.servlet.http.HttpServletRequest;import javax.servlet.http.HttpServletResponse;import java.io.IOException;import java.io.PrintWriter;@WebServlet("/demo1")public class Demo1Servlet extends HttpServlet {    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {        response.setContentType("text/html;charset=utf-8");        PrintWriter out = response.getWriter();        out.write("嘻嘻");    }    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {        doPost(request, response);    }}
```

1. 部署运行

   访问欢迎页面

   ![image-20200725121009680](maven基础.assets/image-20200725121009680.png)

   访问servlet

   ![1603249769945](maven基础.assets/1603249769945.png)

### 小结

* **使用maven插件帮助创建了什么？**
  - 帮我们生成webapp目录
    - 在pom文件中添加打包方式



## 12.项目管理1—clean命令【重点】

### 目标

通过生命周期管理命令管理项目并且掌握命令执行方式

### 生命周期命令介绍

![1547570295556](maven基础.assets/1547570295556.png)

### clean命令介绍

因为有的时候编译器的编译结果是错误的， 一般解决办法先清除原来的编译结果（删除target），再重新编译

> 场景描述：代码已经修改，但是target目录下的输出class字节码文件依然是旧的
>
> 解决方案：执行clean命令清除旧编译结果

### 使用命令方式

1. 方式一：传统方式使用dos命令

   ![1603032313708](maven基础.assets/1603032313708.png)

2. 方式二：使用idea开发工具内置集成的maven工具操作（推荐方式）

   ![1603032380800](maven基础.assets/1603032380800.png)

## 13.项目管理2—其他命令【重点】

### 目标

了解complie\test\install生命周期命令

掌握package生命周期命令



### complie命令

##### 作用

用于对main目录下的java代码重新编译

##### 效果

![1603032490717](maven基础.assets/1603032490717.png)

### test命令

##### 作用

先编译再运行test目录下的所有单元测试代码

##### 应用场景

在企业中，偶尔会更新上线的项目代码（给客户使用的项目），在给客户之前会将所有的单元测试全部运行一遍，观察是否通过，就会使用maven的test命令非常方便看，而不是一个一个的运行浪费时间

##### 使用要求

==测试包、测试类类名必须以test开始或者结尾，否则只会编译不会运行==

##### 效果

![1603251362450](maven基础.assets/1603251362450.png)



##### 中文乱码问题

使用test命令运行测试类输出中文默认会乱码，不建议在test测试代码里面直接输出中文

##### 解决测试中文乱码【了解】

a.设置当前项目maven码表为utf8

b.所有生命周期的命令都是使用插件完成的，要求插件必须使用最新版本

```xml
<?xml version="1.0" encoding="UTF-8"?><project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">    <modelVersion>4.0.0</modelVersion>    <groupId>javaee130</groupId>    <artifactId>day35_03_javaweb_selfdefine_plugin</artifactId>    <version>1.0-SNAPSHOT</version>  <packaging>war</packaging>  <properties>    <!--解决测试中文乱码第一步：设置码表UTF-8-->    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  </properties>  <dependencies>    <dependency>      <groupId>junit</groupId>      <artifactId>junit</artifactId>      <version>4.12</version>      <scope>test</scope>    </dependency>    <dependency>      <groupId>javax.servlet</groupId>      <artifactId>javax.servlet-api</artifactId>      <version>3.1.0</version>      <scope>provided</scope>    </dependency>  </dependencies>  <!--解决测试中文乱码第二步：设置使用高版本的插件-->  <build>    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->      <plugins>        <plugin>          <artifactId>maven-clean-plugin</artifactId>          <version>3.1.0</version>        </plugin>        <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_war_packaging -->        <plugin>          <artifactId>maven-resources-plugin</artifactId>          <version>3.0.2</version>        </plugin>        <plugin>          <artifactId>maven-compiler-plugin</artifactId>          <version>3.8.0</version>        </plugin>        <plugin>          <artifactId>maven-surefire-plugin</artifactId>          <version>2.22.1</version>        </plugin>        <plugin>          <artifactId>maven-war-plugin</artifactId>          <version>3.2.2</version>        </plugin>        <plugin>          <artifactId>maven-install-plugin</artifactId>          <version>2.5.2</version>        </plugin>        <plugin>          <artifactId>maven-deploy-plugin</artifactId>          <version>2.8.2</version>        </plugin>      </plugins>    </pluginManagement>  </build></project>
```

解决乱码运行的效果

![image-20200309142325595](maven基础.assets/image-20200309142325595.png)

### package命令【重要】

##### 作用

将当前项目进行打包的

##### 效果

![image-20200309142628743](maven基础.assets/image-20200309142628743.png)

### intall命令

##### 作用

将当前项目安装到本地仓库，给自己其他项目引用构建使用

##### 控制台运行效果

![image-20200309142948566](maven基础.assets/image-20200309142948566.png)

##### 本地仓库安装效果

![image-20200309143034830](maven基础.assets/image-20200309143034830.png)

### 总结

maven的命令有哪些？

	- clean         删除target目录- compile    编译- test       测试，执行test目录下的所有测试方法，但是该命令有bug，有中文乱码问题- package 打包- install   安装







## 14.依赖管理1-依赖插件【了解】

### 目标

掌握使用tomcat插件运行项目



### maven的插件介绍

maven通过插件来扩展maven工具的功能，maven的项目管理工具的命令就是使用了插件。maven有很多插件，其中最知名有2个，一个是jdk编译语言级别插件和一个tomcat7运行插件 ,插件可以根据个人的兴趣爱好选择使用，实际开发中，有部分人喜欢使用插件，有部分人不喜欢。



### 实现步骤

1. jdk插件用于设置编译级别【了解】

2. tomcat7插件

   配置插件代码

   ```xml
   <build>    <plugins>      <!-- jdk版本编译语言级别插件（了解），因为一般都采用全局settings.xml配置了,如果当前项目不使用全局就可以使用这个插件 -->      <plugin>        <groupId>org.apache.maven.plugins</groupId>        <artifactId>maven-compiler-plugin</artifactId>        <version>3.2</version>        <configuration>          <source>1.8</source>          <target>1.8</target>          <encoding>UTF-8</encoding>          <showWarnings>true</showWarnings>        </configuration>      </plugin>      <!--tomcat7的插件，官方只提供了tomcat7,没有提供tomcat8          tomcat7插件运行项目的优点： 支持webapp目录下资源修改的热部署                                     就是webapp目录下资源的修改不用重新部署理解识别          tomcat7插件运行代部署资源位置介绍：                  webapp部署位置：                      为什么不需要webapp目录下资源重新部署？答：tomcat7插件是直接运行webapp目录下的资源                  java代码部署位置（类路径）：                      target/classes目录下          注意： 使用插件不可以在页面里面点击浏览器图标运行      -->      <plugin>        <groupId>org.apache.tomcat.maven</groupId>        <artifactId>tomcat7-maven-plugin</artifactId>        <version>2.2</version>        <configuration>          <port>8080</port><!--设置插件使用端口，可以修改-->          <path>/</path>   <!--设置项目路径，可以修改-->          <uriEncoding>UTF-8</uriEncoding>  <!--tomcat7的get方式提交中文默认会乱码，这里配置码表解决get提交中文乱码-->          <server>tomcat7</server>   <!--插件服务器的名字-->        </configuration>      </plugin>    </plugins>  </build>
   ```

   使用插件运行

   

![1603034207305](maven基础.assets/1603034207305.png)

注意： 由于tomcat7没有解决servlet的jar冲突问题，所以需要在pom文件中配置servlet的依赖范围

```xml
  <dependency>      <groupId>javax.servlet</groupId>      <artifactId>javax.servlet-api</artifactId>      <version>3.1.0</version>      <scope>provided</scope> <!-- 依赖范围-->    </dependency>
```



### 运行效果

   ![1603253153345](maven基础.assets/1603253153345.png)





## 15.依赖管理2-依赖范围【重点】

### 目标

1. 掌握在pom.xml引用jar的依赖
2. 理解引用依赖中依赖范围的概念

### 依赖jar包

疑问1：依赖jar包不会写怎么办？

答：在企业中都是根据项目组内提供pom.xml模板直接copy使用



疑问2：个人使用的技术用到了模板pom.xml中没有的依赖怎么办？

答：面向百度，官方手册，还有mvn坐标网站

mvn坐标网站： http://mvnrepository.com/ ，不会写可以在这里参照jar包的坐标依赖写法

![image-20200309151450831](maven基础.assets/image-20200309151450831.png)

点击“MyBatis”连接

![image-20200309151545652](maven基础.assets/image-20200309151545652.png)

点击版本3.5.4

![image-20200309151619326](maven基础.assets/image-20200309151619326.png)

```xml
 <!--引用junit依赖-->    <dependency>      <groupId>junit</groupId>      <artifactId>junit</artifactId>      <version>4.12</version>      <scope>test</scope>      <!--      scope标签：用于设置当前jar的依赖范围，就是设置jar包在哪里使用      -->    </dependency>    <!--引用servlet的依赖-->    <dependency>      <groupId>javax.servlet</groupId>      <artifactId>javax.servlet-api</artifactId>      <version>3.1.0</version>      <scope>provided</scope>    </dependency>  </dependencies>
```

### 依赖范围

##### 什么是依赖范围

`<scope>依赖范围</scope>`

==依赖范围是指依赖在什么阶段起作用， 我们的阶段分为：编译时、运行时、测试时三个阶段。==



常用的依赖范围有以下四种，其中默认是：compile

##### 依赖范围见下表

scope标签配置依赖范围的关键字有如下：

| 依赖范围编译时 | 编译时 | 测试时 | 运行时 |
| -------------- | ------ | ------ | ------ |
| compile(默认)  | Y      | Y      | Y      |
| provided       | Y      | Y      | -      |
| runtime        | -      | Y      | Y      |
| test           | -      | Y      | -      |



### 示例：test依赖范围

##### 介绍

设置jar包只能在test目录下运行使用

##### 测试需求

效果描述：配置junit的jar包的使用范围为test， 在main主体位置使用junit代码，发现不能使用

##### 演示步骤

pom.xml设置junit的jar包

```xml
<!--导入junit的jar包  --><dependency>  <groupId>junit</groupId>  <artifactId>junit</artifactId>  <version>4.11</version>  <scope>test</scope></dependency>
```

在main/java的里面Servlet使用junit单元测试查看效果

![image-20200818164405230](maven基础.assets/image-20200818164405230.png)

##### test应用场景

专门用于junit的jar包设置依赖范围，目的是不给运行时使用，也就是不给到客户



### 示例：provided依赖范围

##### 介绍

设置jar包在主体main和测试test的位置可以使用，运行时不能使用

> 因为主体main与运行时可能存在冲突，导致jar不能在2个位置同时运行



##### provided应用场景

专门给servlet的jar包使用

> 主体位置没有servlet的jar包所以需要导入使用，servlet的jar包有个特点在运行时已存在这个jar包（就是tomcat上已经有了这个jar包）
>
> tomcat服务器上已存在的servlet的jar包截图
>
> ![image-20200818165018735](maven基础.assets/image-20200818165018735.png)
>
> 主体导入了一个，服务器上已存在了一个，这两个不能同时存在，否则会产生冲突



##### 冲突演示【tomcat7maven插件运行】

主体导入servlet的jar包，并设置运行时可用，就是将主体的jar包给到运行时去运行，这样就会和运行时已有的jar包产生冲突

```xml
<!--导入servlet的jar包--><dependency>  <groupId>javax.servlet</groupId>  <artifactId>javax.servlet-api</artifactId>  <version>3.1.0</version>  <!--如果没有配置scope默认为compile,就是所有位置可用--></dependency>
```

现在使用tomcat7插件运行servlet, 会发现冲突

> servlet的jar包正式服务器是没有冲突的

![image-20200818165539165](maven基础.assets/image-20200818165539165.png)



##### 解决冲突

设置servlet的jar包只能在主体或测试使用，不可以在运行时使用

```xml
 <!--导入servlet的依赖-->    <dependency>      <groupId>javax.servlet</groupId>      <artifactId>javax.servlet-api</artifactId>      <version>3.1.0</version>      <scope>provided</scope>  <!-- 对于编译的时候有作用，测试的时候有作用 -->    </dependency>
```

再次运行tomcat7插件访问servlet

![image-20200818165735566](maven基础.assets/image-20200818165735566.png)

### 示例：runtime依赖范围

##### 介绍

设置jar包在测试test和运行时可用，在主体main不可用



##### runtime应用场景

专门设置mysql驱动包使用的依赖范围

> 分析：mysql驱动包为什么在主体位置不用？
>
> jdk提供： JDBC的接口
>
> mysql驱动包提供： JDBC接口的实现类
>
> 答：因为在主体main中开发人员编写代码只使用JDBC接口



##### 测试

pom.xml中没有引用依赖mysql驱动包

```java
@Test    public void test02() throws Exception {        //1.加载mysql的驱动包        Class.forName("com.mysql.jdbc.Driver");        //2. 获取连接        Connection connection = DriverManager.getConnection("jdbc:mysql:///db1", "root", "root");        System.out.println(connection);    }
```

 在main/java目录下编写jdbc操作不会报错，说明不需要mysql驱动包

![image-20200818170701800](maven基础.assets/image-20200818170701800.png)

运行代码会报错，因为运行时没有导入mysql驱动包依赖

![image-20200725162359412](maven基础.assets/image-20200725162359412.png)

##### 解决方案

在pom.xml中导入mysql驱动包依赖使用范围为runtime

```xml
<dependency>      <groupId>mysql</groupId>      <artifactId>mysql-connector-java</artifactId>      <version>5.1.47</version>      <scope>runtime</scope>  <!--在运行与测试的时候起作用-->    </dependency>
```

##### 运行效果

![image-20200725162519077](maven基础.assets/image-20200725162519077.png)

### 小结



1. 常用的依赖范围有哪些

   > complie(默认),  编译、运行、测试
   >
   > provided    编译，测试
   >
   > runtime     运行，测试
   >
   > test   测试

   




## 16.将已有项目转换为Maven工程【应用】

目标： 清楚每一个资源文件放在maven目录的那个结构。

原有项目

![1603261750043](maven基础.assets/1603261750043.png) 

创建一个自定义maven工程+插件优化结构效果

![image-20200929161959558](maven基础.assets/image-20200929161959558.png)

![image-20200929162016411](maven基础.assets/image-20200929162016411.png) 

![image-20200929162049540](maven基础.assets/image-20200929162049540.png)

pom.xml导入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?><project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">    <modelVersion>4.0.0</modelVersion>    <groupId>com.itheima</groupId>    <artifactId>day35_autocomplete</artifactId>    <version>1.0-SNAPSHOT</version>  <packaging>war</packaging>  <dependencies>    <dependency>      <groupId>junit</groupId>      <artifactId>junit</artifactId>      <version>4.12</version>      <scope>test</scope>    </dependency>    <!--servlet-->    <dependency>      <groupId>javax.servlet</groupId>      <artifactId>javax.servlet-api</artifactId>      <version>3.1.0</version>      <scope>provided</scope>    </dependency>    <!--mysql驱动-->    <dependency>      <groupId>mysql</groupId>      <artifactId>mysql-connector-java</artifactId>      <version>5.1.40</version>      <scope>runtime</scope>    </dependency>    <!--mybatis-->    <dependency>      <groupId>org.mybatis</groupId>      <artifactId>mybatis</artifactId>      <version>3.4.5</version>    </dependency>    <dependency>      <groupId>log4j</groupId>      <artifactId>log4j</artifactId>      <version>1.2.17</version>    </dependency>    <!--BeanUtils-->    <dependency>      <groupId>commons-beanutils</groupId>      <artifactId>commons-beanutils</artifactId>      <version>1.9.2</version>      <scope>compile</scope>    </dependency>    <!--jackson-->    <dependency>      <groupId>com.fasterxml.jackson.core</groupId>      <artifactId>jackson-databind</artifactId>      <version>2.3.3</version>    </dependency>    <dependency>      <groupId>com.fasterxml.jackson.core</groupId>      <artifactId>jackson-core</artifactId>      <version>2.3.3</version>    </dependency>    <dependency>      <groupId>com.fasterxml.jackson.core</groupId>      <artifactId>jackson-annotations</artifactId>      <version>2.3.3</version>    </dependency>  </dependencies></project>
```

导入配置文件

![image-20200929162232522](maven基础.assets/image-20200929162232522.png) 

将原有项目中的src目录下的代码复制到maven工程总蓝java目录下

![image-20200929162422317](maven基础.assets/image-20200929162422317.png) 

将原有项目中web目录下js目录和index.jsp文件复制到maven工程的webapp目录下

![image-20200929162511054](maven基础.assets/image-20200929162511054.png) 

最终maven工程结构

![image-20200929162546745](maven基础.assets/image-20200929162546745.png) 

 

创建mybatis的Dao映射配置文件

![image-20200929164056412](maven基础.assets/image-20200929164056412.png)

磁盘资源目录效果

![image-20200929164134411](maven基础.assets/image-20200929164134411.png)



代码

![image-20200929164325371](maven基础.assets/image-20200929164325371.png) 

```xml
<?xml version="1.0" encoding="UTF-8" ?><!DOCTYPE mapper        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"        "http://mybatis.org/dtd/mybatis-3-mapper.dtd"><!-- namespace名称空间，名称空间代表该xml文件映射是哪个接口--><mapper namespace="com.itheima.dao.UserDao">        <select id="findByName" resultType="user">        SELECT * FROM USER WHERE NAME LIKE CONCAT(#{name},'%') LIMIT 0,4    </select></mapper>
```

![1603262786643](maven基础.assets/1603262786643.png)



## 17.删除maven项目再打开【重点】

### 删除项目





![1603263125358](maven基础.assets/1603263125358.png)

### 恢复项目

1. 导入模块

![1603069124665](maven基础.assets/1603069124665.png)

2. 选择模块，注意这时候不是选择iml文件了，而是选择整个模块

![1603069202402](maven基础.assets/1603069202402.png)

![1603069223943](maven基础.assets/1603069223943.png)

==接下来一路next即可==



## 18.清除下载失败标记文件【重点】

### maven的问题

以后大家使用了本地仓库没有jar包，默认会去远程仓库下载，在下载过程中网络断了，maven会在本地仓库jar包对应位置生成一个错误标记文件xxx.lastUpdated,   一旦产生了这个文件，下次网络恢复了，也不会去下载这个这个jar包。如何解决？

答： 清除失败标记文件xxx.lastUpdated

### 目标

解决下载jar包失败以后无法再次下载的问题



### 解决方案

进入本地仓库

![image-20200106171959060](maven基础.assets/image-20200106171959060-1595636444172.png)

==缺失jar的解决方案一：==

搜索*.lastUpdate

![image-20200106172126432](maven基础.assets/image-20200106172126432-1595636444173.png)

![、image-20200106172205368](maven基础.assets/image-20200106172205368-1595636444173.png)

==缺失jar的解决方案二： 如果你依赖报红，那么老钟个人建议删除某一个版本的整个文件夹，然后刷新即可重新下载。==

![1603263915742](maven基础.assets/1603263915742.png)

![1603263993057](maven基础.assets/1603263993057.png)





## 19.JDK动态代理优化SqlSession资源开启与关闭【重点】

### 目标

使用jdk动态代理增强每一个dao数据库方法初始化资源和关闭资源



### 优化需求

对于每个业务层方法都需要如下操作

```java
package com.itheima.service;import com.itheima.dao.UserDao;import com.itheima.entity.User;import com.itheima.util.MybatisUtils;import org.apache.ibatis.session.SqlSession;import java.util.List;/** * @author 黑马程序员 */public class UserService {    //根据搜索条件name查找用户列表    public List<User> findByName(String name){        //调用dao根据搜索条件获取数据        SqlSession sqlSession = MybatisUtils.getSession();        UserDao userDao = sqlSession.getMapper(UserDao.class);        List<User> userList = userDao.findByName(name);        //关闭资源        MybatisUtils.closeSession(sqlSession);        return userList;    }}
```

<font color=red>1. 初始化SqlSession资源</font>

<font color=red>2.释放SqlSession资源 </font>

们可以将这些代码抽取出来，使用动态代理增强每个dao数据库的方法，实现统一初始化与释放资源 



### 创建Dao工厂类实现动态代理增强每个dao的方法

DaoFactory代码

![image-20200929170958175](maven基础.assets/image-20200929170958175.png)&nbsp;

```java
package com.itheima.utils;import org.apache.ibatis.session.SqlSession;import java.lang.reflect.InvocationHandler;import java.lang.reflect.Method;import java.lang.reflect.Proxy;public class DaoFactory {    //获取所有Dao的代理对象。 增强Dao的所有的方法。    public static Object getDaoInstance(Class daoClass){ //daoClass : 获取的Dao的类型      Object daoProxy =   Proxy.newProxyInstance(DaoFactory.class.getClassLoader(), new Class[]{daoClass}, new InvocationHandler() {            @Override            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {                //1. 得到SqlSession                SqlSession sqlSession = SqlSessionUtils.getSqlSession();                //得到接口代理对象                Object mapper = sqlSession.getMapper(daoClass);                //让目标方法执行                Object result = method.invoke(mapper,args);                //最后，提交并关闭                SqlSessionUtils.close(sqlSession);                return result;            }        });      return daoProxy;    }}
```

### 优化ContactService代码

```java
package com.itheima.service;import com.itheima.dao.UserDao;import com.itheima.model.User;import com.itheima.utils.DaoFactory;import com.itheima.utils.SqlSessionUtils;import org.apache.ibatis.session.SqlSession;import java.util.List;public class UserService {    public List<User> findByName(String name){        UserDao userDao = (UserDao) DaoFactory.getDaoInstance(UserDao.class);        List<User> list  = userDao.findByName(name);        return list;    }}
```



### 小结

jdk动态代理的作用？

​	- 增强某个对象的某个方法
