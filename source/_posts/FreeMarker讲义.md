---
title: FreeMarker讲义
tags:
  - 笔记
  - FreeMarker
  - 页面静态化
  - Spring MVC
categories:
  - ssm框架
date: 2021-01-31 23:00:57
---

 

# 1.  FreeMarker

## 1.1. 概述

FreeMarker是一款模板引擎： 即一种基于模板和要改变的数据， 并用来生成输出文本（HTML网页、电子邮件、配置文件、源代码等）的通用工具。 它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件。

HTML静态化也是某些缓存策略使用的手段，对于系统中频繁使用数据库查询但是内容更新很小的应用，可以使用FreeMarker将HTML静态化。

## 1.2. 工作原理

![img](FreeMarker讲义.assets/clip_image002.jpg)

**模板**：就是一份已经写好了基本内容，有着固定格式的文档，其中空出或者用占位符标识的内容，由使用者来填充，不同的使用者给出的数据是不同的。在模板中的占位符，在模板运行时，由模板引擎来解析模板，并采用动态数据替换占位符部分的内容。

FreeMarker与Web容器无关，即在Web运行时，它并不知道Servlet或HTTP。它不仅可以用作表现层的实现技术，而且还可以用于生成Html，XML，JSP或Java 等文件。

 

## 1.3. 优缺点

l **优点**

1、FreeMarker不支持Java脚本代码；所以可以彻底的分离表现层和业务逻辑；

2、提高开发效率。开发过程中，界面设计和开发人员可以并行工作；不必等待完成页面原形后，再开发程序；

3、开发过程中的人员分工更加明确。使用FreeMarker后，作为界面开发人员，只专心创建HTML文件、图像以及Web页面的其他可视化方面，不用理会数据；而程序开发人员则专注于系统实现，负责为页面准备要显示的数据。

 

l **缺点**

1. 生成静态的HTML页面后，数据更新可能不及时；

2. 需要学习FreeMarker模版语言。而且FreeMarker中的变量必须要赋值，如果不赋值，那么就会抛出异常。想避免错误就要应用if/elseif/else 指令进行判段，如果对每一个变量都判断的话，那么则反而增加了编程的麻烦。FreeMarker的map限定key必须是string，其他数据类型无法操作。

# 2.  第一个FreeMarker例子

## 2.1. 搭建maven项目

搭建java项目即可。FreeMarker的依赖坐标为：

```xml
		<dependency>
			<groupId>org.freemarker</groupId>
			<artifactId>freemarker</artifactId>
			<version>2.3.23</version>
		</dependency>

```



 

## 2.2. 控制台输出

![img](FreeMarker讲义.assets/clip_image004.jpg)

## 2.3. 文件输出

![img](FreeMarker讲义.assets/clip_image006.jpg)

# 3.  模版

详见《FreeMarker_Manual_zh_CN.pdf》的第26页。

![img](FreeMarker讲义.assets/clip_image008.jpg)

# 4.  基本语法

一些常见的符号说明：

```html
${ }插值；只能输出数值、日期或者字符串，其它类型不能输出。

<#freemarker命令

<#-- 注释 -->

<@使用自定义命令

??是判断对象是否为空

?函数调用

```



 

## 4.1. 命令

### 4.1.1.  自定义对象输出

![img](FreeMarker讲义.assets/clip_image010.jpg)

 

模版文件内容：

![img](FreeMarker讲义.assets/clip_image012.jpg)

### 4.1.2.  控制

语法：

```html
<#if condition>
...
<#elseif condition2>
...
<#elseif condition3>
...
...
<#else>
...
</#if>
这里：
condition ， condition2 等：表达式将被计算成布尔值。

```

```html
关键之：gt ：比较运算符“大于”；gte ：比较运算符“大于或等于”；lt ：比较运算符“小于”；lte ：比较运算符“小于或等于”
```



![img](FreeMarker讲义.assets/clip_image014.jpg)

### 4.1.3.  循环

语法：

```html
<#list sequence as item>
...
</#list>
这里：
sequence ：表达式将被算作序列或集合
item ：循环变量（不是表达式）的名称

```



 

![img](FreeMarker讲义.assets/clip_image016.jpg)

 

模板中可以如下遍历：

![img](FreeMarker讲义.assets/clip_image018.jpg)

## 4.2. 变量

语法：`<#assign name=value>`

示例：

```html
<#-- 定义字符变量 -->
<#assign str="Hello World.">
${str}

<hr>
<#-- 定义数值变量 -->
<#assign num=123>
${num}

<hr>
<#-- 定义布尔变量；但是${}只能输出数值或字符串，需要转换为字符串；?sring 的意思是调用boolean类型的内置函数string -->
<#assign bool=true>
${bool?string}
${bool?string("Yes","No")}

<hr>
<#-- 显示日期  .now 是当前时间 -->
${.now}---${.now?string("yyyy年MM月dd日 HH:mm:ss")}

<hr>
<#-- 字符串转日期 -->
<#assign dd ="2000-01-01"?date("yyyy-MM-dd")>
${dd?string("yyyy年MM月dd日")}

<hr>
<#-- 获取范围字符 -->
${"我和黑马程序员的故事"[2..6]}

```

注：local一般使用在宏或函数里面。

## 4.3. 空值

在freemarker中对于空值必须手动处理。

在插值中处理空值：或者 

```html
${emp.name!} 表示name为空时什么都不显示

${emp.name!(“名字为空”)}  表示name为空时显示 名字为空

${(emp.company.name)!} 表示如果company对象为空则什么都不显示，!只用在最近的那个属性判断；所以如果遇上有自定义类型（导航）属性时，需要使用括号

${bool???string} 表示：首先??表示判断bool是否为空返回true或者false，然后对返回的值调用其内置函数string

<#if str??> 表示去判断str变量是否为空值，空则true，不空为false

```



模板示例：

```html
<#-- ${emp.name!}的使用 -->
${emp.name!}

<hr>
<#-- ${emp.name!()}的使用 -->
${emp.name!("名字为空。。。")} 
---- 或者不加小括号也一样  ----
${emp.name!"名字为空。。。"} 

<hr>
<#-- ${bool???string}的使用 -->
bool???string = ${bool???string}

<hr>
<#-- ?? 判断一个值是否为空，空返回true,否则false -->
<#assign str="">
<#if str??>
  str为空值
<#else>
  str不为空值
</#if>

```



# 5.  列表与Map

## 5.1. 列表

```html
<#assign nums = [1,2,3,4,5]>
<#assign nums =6..10 >

全部输出：
<#list nums as no>
	${no}
</#list>

<br>
输出第1个到第4个：
<#list nums[0..3] as no>
	${no}
</#list>

```



 

## 5.2. Map

Freemarker对于map的key而言，其**key的类型必须为字符串**。

```html
处理map数据：<br>
<#assign map = {"name":"heima1","age":10}>
<#list map?keys as key>
	${key}----${map["${key}"]} <br>
</#list>

```



# 6.  自定义命令macro

通过macro可以自定义命令，然后可以传递参数；在macro标签之间可以嵌套其它命令。另外；macro传递参数值时若参数有默认值则要把有默认值的参数放置在参数列表的最后。

```html
<#-- 第一个参数为命令的名称，是必须的；之后的为参数列表，可以添加多个 -->
自定义命令：
<#macro sayHello name>
	Hello ${name}
</#macro>

<@sayHello name="黑马"/>

<hr>
自定义命令，多参数，默认参数：
<#macro printNum name num=3>
	<#list 1..num as n>
	Hello ${name} --- ${n}<br>
	</#list>
</#macro>

<@printNum name="黑马"/>

```



 

# 7.  Spring MVC整合FreeMarker

Spring mvc 主要是使用freemarker用来展示数据；所以只需要配置它对应的视图解析器即可。

## 7.1. 搭建maven工程

Pom.xml文件内容为：

 ```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>cn.itcast.freemarker</groupId>
	<artifactId>spring-freemarker</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>

	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.10</version>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
			<version>4.1.3.RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>4.1.3.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.freemarker</groupId>
			<artifactId>freemarker</artifactId>
			<version>2.3.23</version>
		</dependency>

		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.7.5</version>
		</dependency>

		<!-- JSP相关 -->
		<dependency>
			<groupId>jstl</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>servlet-api</artifactId>
			<version>2.5</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>jsp-api</artifactId>
			<version>2.0</version>
			<scope>provided</scope>
		</dependency>

	</dependencies>


	<build>
		<plugins>
			<!-- java编译插件 -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.2</version>
				<configuration>
					<source>1.7</source>
					<target>1.7</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
			<!-- 配置Tomcat插件 -->
			<plugin>
				<groupId>org.apache.tomcat.maven</groupId>
				<artifactId>tomcat7-maven-plugin</artifactId>
				<configuration>
					<port>8080</port>
					<path>/freemarker</path>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>

 ```



 

## 7.2. 配置

Spring MVC的配置文件：spring-freemarker-servlet.xml文件内容为：

 ```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.1.xsd
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.1.xsd 
	http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.1.xsd
	http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.1.xsd">

	<mvc:annotation-driven />

	<!-- 定义处理器 -->
	<context:component-scan base-package="cn.itcast.freemarker.controller"></context:component-scan>
	
	<bean id="freeMarkerConfigurer" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
		<property name="templateLoaderPath" value="/WEB-INF/ftl"></property>
	</bean>
	
	<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
		<property name="prefix" value=""></property>
		<property name="suffix" value=".ftl"></property>
		<property name="order" value="1"></property>
		<property name="contentType" value="text/html;charset=UTF-8"></property>
	</bean>

	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<!-- 设置资源前缀路径 -->
		<property name="prefix" value="/WEB-INF/jsp/"></property>
		<!-- 设置资源的后缀 -->
		<property name="suffix" value=".jsp"></property>
		<property name="order" value="2"></property>
	</bean>

	<!-- 解决servlet映射路径配置为/后，访问静态资源404问题 -->
	<mvc:default-servlet-handler/>
	

</beans>

 ```



 

配置web.xml文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	id="WebApp_ID" version="3.0">

	<filter>
		<filter-name>characterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>utf-8</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>characterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>


	<!-- 配置spring mvc前端控制器 -->
	<servlet>
		<servlet-name>spring-freemarker</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:spring/spring-freemarker-servlet.xml</param-value>
		</init-param>
		<!-- 随着应用服务器启动而初始化 若不配置，则第一次请求后完成初始化 -->
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>spring-freemarker</servlet-name>
		<!-- springmvc的映射规则： 可以配置：/、/xxx/*、*.* 不可配置：/* 与应用服务器配置默认冲突 -->
		<url-pattern>/</url-pattern>
	</servlet-mapping>


	<display-name>spring-freemarker</display-name>
	<welcome-file-list>
		<welcome-file>index.jsp</welcome-file>
	</welcome-file-list>
</web-app>

```



 

## 7.3. 测试

编写HelloController分别测试转发到ftl和jsp的方法：

![img](FreeMarker讲义.assets/clip_image020.jpg)

 

# 8.  Nginx配置静态化

如果淘淘商城需要首页静态化，则需要在页面中所有动态加载和jsp标签语法都需要改变；然后在nginx的配置文件nginx.conf中添加：

```text
server {
        listen       80;
        server_name  www.taotao.com taotao.com;

		proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header Host $host;
		
		set $staticIndex "D:/itcast/taotao_index";
		root $staticIndex;
		
		location ~*(/|/index.html)$ {
			set $indexHtml $staticIndex/index.html;
			if (!-e $indexHtml ){
				proxy_pass http://127.0.0.1:9091;
				# break;
			}
			rewrite $indexHtml break;
			proxy_connect_timeout 600;
			proxy_read_timeout 600;
		}
		
        location / {
			proxy_pass http://127.0.0.1:9091;
			proxy_connect_timeout 600;
			proxy_read_timeout 600;
        } 
    }

```

