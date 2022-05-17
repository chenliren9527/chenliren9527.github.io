---
title: JavaWeb(02)-request请求对象
tags:
  - 笔记
  - JavaWeb
  - HTTP协议
  - Request域对象 
  - Request对象做请求转发
  - HTTP请求参数的乱码
  - HTTP协议请求
categories:
  - JavaWeb
date: 2020-10-10 16:26:01
---

# 《request请求对象-笔记》



# 学习目标

1、能够理解HTTP协议请求内容 

2、能够处理HTTP请求参数的乱码问题 

3、能够使用Request域对象 

4、能够使用Request对象做请求转发 



# 学习内容

## 1.web模块的导入方式



1. 把模块拷贝到当前的Project中。

2. 先把模块拷贝到project中。

   ![1602290558405](JavaWeb-02-request请求对象.assets/1602290558405.png)

3. 从project中导入当前的模块

![1596587801069](JavaWeb-02-request请求对象.assets/1596587801069.png)

![1596587816320](JavaWeb-02-request请求对象.assets/1596587816320.png)

3. 让模块关联tomcat的jar

   ![1602290739872](JavaWeb-02-request请求对象.assets/1602290739872.png)

   ![1596588008377](JavaWeb-02-request请求对象.assets/1596588008377.png)

![1596588055860](JavaWeb-02-request请求对象.assets/1596588055860.png)

4. 把导入的模块修复一下,修复为web模块

5. ![1602290872041](JavaWeb-02-request请求对象.assets/1602290872041.png)

   ![1596588237850](JavaWeb-02-request请求对象.assets/1596588237850.png)





## 2.HTTP协议的概念

### 目标

1. 什么是HTTP
2. 它有什么特点

### 概念

![1555154341398](JavaWeb-02-request请求对象.assets/1555154341398.png) 

作用：浏览器中浏览网页的时候，使用的协议

HTTP:  Hyper Text Transfer Protocol ==超文本传输协议==，传输HTML网页。

HTML: Hyper Text Markup Language 超文本标记语言

默认端口号：Web服务器默认使用的端口号是80，可以省略。



### http协议的作用

![1602291568426](JavaWeb-02-request请求对象.assets/1602291568426.png)



==http协议的作用：==  规范浏览器与服务器数据传输的格式



### 为什么需要学习HTTP协议(http协议的应用场景)

我们学习http协议，我们就非常清楚浏览器与服务器通讯时要使用的格式。



**应用场景一：** 简单外挂工具、刷屏软件、偷菜

![1566696549571](JavaWeb-02-request请求对象.assets/1566696549571.png)



**应用场景二： 智能家居**

只能家居一般都是手机发送指令给家庭电器。 

发送数据给家庭设备的时候需要符合http协议（手动拼接字符串）。设备那边也是需要解析。





### 小结

1. **http协议的作用**

   - 规范浏览器与服务器数据传输的格式



## 3.HTTP请求的三个组成

### 目标

请求有哪三个组成部分

### 什么是请求

由浏览器发送给服务器的所有的数据

### 查看HTTP请求

1. 在HTML页面上，制作2个表单提交页面，用户名和密码，get提交和post提交按钮，查看HTTP请求

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>登录表单</title>
   </head>
   <body>
   <h2>GET请求</h2>
   <form action="login" method="get">
       用户名：<input type="text" name="username"><br>
       密码：<input type="password" name="password"><br>
       <input type="submit" value="登录">
   </form>
   
   <hr/>
   <h2>POST请求</h2>
   <form action="login" method="post">
       用户名：<input type="text" name="username"><br>
       密码：<input type="password" name="password"><br>
       <input type="submit" value="登录">
   </form>
   </body>
   </html>
   ```

2. 查看浏览器与服务器的通讯，按F12打开窗口

   ![1552551173484](JavaWeb-02-request请求对象.assets/1552551173484.png)

### 请求的组成

![1564623694262](JavaWeb-02-request请求对象.assets/1564623694262.png) 

### 小结

**请求由哪三个组成部分？分别是？**

- 请求信息= 请求行+请求头+请求体



## 4.请求信息的组成：请求行

### 目标

1. 请求行的格式
2. POST和GET请求的区别

### 请求行组成

请求行 = 请求方式+url地址+ 协议版本

```html
GET /day27/demo1?username=jack&password=123 HTTP/1.1
POST /day27/demo1 HTTP/1.1
```

==请求行 ：== 

GET方法数据是在请求行中发送的

### POST与GET的区别

|            | **POST方式**     | **GET方式**                                             |
| ---------- | ---------------- | ------------------------------------------------------- |
| **地址栏** | 不会出现请求参数 | 会出现请求的参数                                        |
| **大小**   | 没有大小限制     | 有大小限制的，理论上是1kb，实际上每个不同浏览器都不一样 |
| **安全性** | 安全性高         | 安全性低                                                |
| **缓存**   | 不会使用缓存的   | 是会使用缓存的                                          |

### 小结

1. **请求行由哪三个组成部分？**

   - 请求行 = 请求方式+ url+ 协议版本

2. **GET方法和POST方法传递数据有什么区别？**

   ​	![1602293533469](JavaWeb-02-request请求对象.assets/1602293533469.png)






## 5.请求信息的组成：请求头、请求体(了解即可)

### 目标

了解常见请求头的作用

### 常用请求头

| 请求头            | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| referer           | 得到从哪个页面跳转过来的，上一个页面是什么<br />![1564625238679](assets/1564625238679.png) |
| if-modified-since | 网页在本地缓存的时间，如果本地没有使用缓存，则看不到这个请求头。<br />![1564625366737](assets/1564625366737.png) |
| user-agent        | 告诉服务器，本地浏览器和操作系统的类型<br />![1564625465806](assets/1564625465806.png) |
| connection        | TCP传输层协议连接状态，HTTP应用层协议，HTTP运行在TCP之上<br />![1564625595245](assets/1564625595245.png) |
| host              | 访问的主机名和端口号<br />![1564625687058](assets/1564625687058.png) |



### 请求体

就是浏览器发送给服务器的数据

1. GET：方法没有请求体，数据在请求行中发送
2. POST：有请求体，数据在请求体中





## 6.请求的方法：request对象获取请求行有关的方法

### 目标

与请求行相关的方法

### 什么是HttpServletRequest对象

​	概述：这是一个接口，实现类由Tomcat编写，并且由Tomcat实例化这个对象。传递给doGet或doPost方法，我们只需要在doGet或doPost方法中去使用这两个对象就可以了。

![1552552065703](JavaWeb-02-request请求对象.assets/1552552065703.png)

### 常用的API

### 

| **HttpServletRequest对象的方法**  | **功能描述**       |
| --------------------------------- | ------------------ |
| **String getMethod()**            | 得到请求方式       |
| **String   getRequestURI()**      | 得到URI            |
| **StringBuffer  getRequestURL()** | 得到URL            |
| **String   getProtocol()**        | 得到协议和版本     |
| String getRemoteAddr（）          | 获取客户端的ip地址 |

### 案例

#### 需求

创建一个RequestLineServlet，用于获取请求行中相关信息的方法，并且输出到网页上。

#### 效果

![1564626643972](JavaWeb-02-request请求对象.assets/1564626643972.png) 

#### 代码

```java
package com.itheima.request;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
/*
学习目标： 通过servlet的方法获取请求行的信息

请求行 = 请求方式+url + 协议版本

HttpServletRequest常用的方法：
                    1.  getMethod()   获取请求的方式(重点)
                    2. getRequestURL（）  获取请求的地址
                    3. getRequestURI（）  获取请求的地址(重点)
                    4. getProtocol()   获取协议版本
                    5. getRemoteAddress  获取客户端的ip地址(重点)
                    6. getContextPath()  获取当前项目根路径(重点，后期会常用，现在大家没法感受它的作用)

 */

@WebServlet("/requestLine")
public class _01RequestLIneServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();

        // 1.  getMethod()   获取请求的方式
        String method = request.getMethod();
        out.write("请求的方式："+ method+"<br/>");

        //2. 获取请求的路径
        String url = request.getRequestURL().toString();
        out.write("请求的URL路径："+ url+"<br/>");

        //3. 获取请求的路径
        String uri = request.getRequestURI();
        out.write("请求的URI路径："+ uri+"<br/>");


//        4. getProtocol()   获取协议版本
        String protocol = request.getProtocol();
        out.write("协议版本："+ protocol+"<br/>");

//       5. getRemoteAddress  获取客户端的ip地址
        String ip = request.getRemoteAddr();
        out.write("ip地址："+ ip+"<br/>");

//       6. getContextPath 获取项目的根路径
        String contextPath = request.getContextPath();
        out.write("项目根路径："+ contextPath+"<br/>");





    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

### 小结

| **HttpServletRequest对象的方法**  | **功能描述**   |
| --------------------------------- | -------------- |
| **String getMethod()**            | 获取请求方式   |
| **String   getRequestURI()        | 获取请求路径   |
| **StringBuffer  getRequestURL()** | 获取请求路径   |
| **String   getProtocol()**        | 协议版本       |
| **String getRemoteAddr**()        | ip地址         |
| getContextPath()                  | 获取项目根路径 |

## 7.请求的方法：与请求头有关的方法

### 目标

request对象中与请求头相关的方法

### 请求头相关方法

| **请求方法**                                | **功能描述**                   |
| ------------------------------------------- | ------------------------------ |
| **String   getHeader(String headName)**     | 通过请求头的键，得到请求头的值 |
| **Enumeration\<String>   getHeaderNames()** | 得到所有请求头的键             |

| **Enumeration接口中方法**       | **说明**                   |
| ------------------------------- | -------------------------- |
| **boolean   hasMoreElements()** | 判断是否还有下一个元素     |
| **E   nextElement()**           | 得到当前元素，向下移动一行 |

### 方法示例

#### 需求

1. 得到user-agent请求头的值
2. 得到所有的请求 



#### 代码

```java
package com.itheima.request;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.Enumeration;
/*
学习目标： 通过servlet的方法获取请求头的信息


HttpServletRequest常用的方法：
                    1.  getHeader(String headName) 根据请求头的key获取请求头的值  （重点）
                    2. getHeaderNames()   获取所有请求头的key

 */

@WebServlet("/requestHeader")
public class _02RequestHeaderServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();

        //1. 根据请求头的key获取请求头的值
      /*  String browser = request.getHeader("User-Agent");
        out.write("当前你使用的浏览器版本是："+ browser);*/

      //获取所有的请求头的key
        Enumeration<String> e = request.getHeaderNames(); //返回的是一个老款迭代器
        while(e.hasMoreElements()){
            //迭代出来的是请求头的键
            String key = e.nextElement();
            String value = request.getHeader(key);
            out.write(key+" : "+ value+"<br/>");
        }

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

### 小结

| **请求方法**                                | **功能描述**                 |
| ------------------------------------------- | ---------------------------- |
| **String getHeader(String   headName)**     | 根据请求头的键获取请求头的值 |
| **Enumeration\<String>   getHeaderNames()** | 获取请求头的所有键           |



## 8.请求头案例：判断浏览器的类型

### 目标

判断客户端是什么类型的浏览器：Edge、Chrome、Safari、Firefox、IE浏览器或其它

### 效果

![1552556601532](JavaWeb-02-request请求对象.assets/1552556601532.png)

### 分析

1. 得到user-agent请求头，请求头的名字不区分大小写。
2. 判断它的值是否包含contains()指定了字符串，如果包含则表示是指定的浏览器。
3. 否则就输出其它的浏览器

### 代码

```java
package com.itheima.request;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.Enumeration;
/*
学习目标： 通过user-agent请求头判断出用户使用的浏览器版本

 */

@WebServlet("/browser")
public class _03BrowserServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();

        //获取客户使用的浏览器版本
        String browser = request.getHeader("user-agent");
        System.out.println("浏览器版本："+ browser);
        if(browser.contains("Chrome")){
            out.write("我猜你是的是谷歌");
        }else if(browser.contains("Firefox")){
            out.write("我猜你是的是火狐");
        }else if(browser.contains("Trident")){
            out.write("我猜你是的是IE");
        }

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

### 小结

**案例的实现思路**

1. 通过getHeader（user-agent）获取浏览器版本信息
2. 通过一些标记去判断使用浏览版本



## <font color="red">9.请求的方法：获取请求参数(从这里开始，这一个点都使劲敲)</font>

### 目标

使用request中的方法得到表单或地址栏上提交的参数值

### 请求对象的方法

| **方法名**                                     | **描述**                                                     |
| ---------------------------------------------- | ------------------------------------------------------------ |
| **String getParameter(String   name)**         | 通过参数名得到参数值，只能获得一个值                         |
| **String[]   getParameterValues(String name)** | 通过一个参数名得到一组同名的值，<br />返回字符串的数组，用于复选框 |
| **Enumeration\<String>   getParameterNames()** | 得到所有的参数名                                             |
| **Map<String,String[]>   getParameterMap()**   | 得到表单所有的参数名和值，封装成Map对象<br />键：参数名，值：参数值(是一个数组) |

### 获取参数的案例

#### 需求

1. 通过上面的方法得到注册页面的用户名和性别，输出到浏览器
2. 得到所有的爱好，输出到浏览器
3. 得到所有表单的名字和值，输出到浏览器
4. 得到封装好的表单数据Map对象，输出到浏览器

#### 效果

![1564629915794](JavaWeb-02-request请求对象.assets/1564629915794.png) 

#### 代码

##### HTML

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>注册页面</title>
</head>
<body>
<h2>用户注册</h2>
<form action="/day27/parameter" method="get">
    用户名： <input type="text" name="name"><br/>
    性别: <input type="radio" name="gender" value="男" checked="checked"/>男
    <input type="radio" name="gender" value="女"/>女 <br/>
    城市：
    <select name="city">
        <option value="广州">广州</option>
        <option value="深圳">深圳</option>
        <option value="珠海">珠海</option>
    </select>
    <br/>
    爱好：
    <input type="checkbox" name="hobby" value="上网"/>上网
    <input type="checkbox" name="hobby" value="上学"/>上学
    <input type="checkbox" name="hobby" value="上车"/>上车
    <input type="checkbox" name="hobby" value="上吊"/>上吊
    <br/>
    <input type="submit" value="注册"/>
</form>
</body>
</html>
```

##### Servlet代码

```java
package com.itheima.request;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.Arrays;
import java.util.Enumeration;
/*
学习目标： 通过request请求对象获取请求参数


request获取请求参数相关的方法：
        1. String getParameter(String   name)   根据参数的name获取参数值 (重点)
        2、String[]   getParameterValues(String name)  根据参数的name获取参数值，对着复选框同一个参数名有多个值的情况下去使用(重点)
        3. Enumeration<String>   getParameterNames()  获取所有请求参数的name
        4. Map<String,String[]>   getParameterMap()   获取所有的请求请求，并且把请求参数封装到一个map对象中返回。

 */

@WebServlet("/parameter")
public class _04ParameterServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();

//        ======================重点==========================
        //getParameter（） 根据参数的name获取参数的value
        String name = request.getParameter("name");
        String gender = request.getParameter("gender");
        String city = request.getParameter("city");
        String[] hobby = request.getParameterValues("hobby");

        out.write("name="+name+"<br/>" );
        out.write("gender="+gender+"<br/>" );
        out.write("city="+city+"<br/>" );
        out.write("hobby="+ Arrays.toString(hobby)+"<br/>" );


//        ======================getParameterNames了解即可=====================
        System.out.println("参数名：");
        Enumeration<String> e = request.getParameterNames();
        while(e.hasMoreElements()){
            String key = e.nextElement();
            System.out.println("\t"+key);
        }

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

### 小结

| **方法名**                                   | **描述**                                      |
| -------------------------------------------- | --------------------------------------------- |
| **String getParameter(String name)**         | 根据参数的name获取参数的值                    |
| **String[] getParameterValues(String name)** | 根据参数的name获取参数的值,针对复选框去使用的 |
| **Map<String,String[]> getParameterNames()** | 获取所有参数的name                            |



## <font color="red">10.BeanUtils工具类的使用</font>

### 目标

BeanUtils工具类中方法的使用

### BeanUtils工具类给我们带来作用

	1. BeanUtils会自动帮我们封装参数
	2. BeanUtils针对八种基本数据类型的属性，都会自动进行类型转换



### 什么是BeanUtils

Bean:   JavaBean，实体类，POJO

BeanUtils是Apache Commons组件的成员之一，主要用于简化JavaBean封装数据的操作。

![1552556927816](JavaWeb-02-request请求对象.assets/1552556927816.png)

BeanUtils是第三方组织写的工具类，要使用它，得先下载对应的工具jar包。

下载地址：http://commons.apache.org/

### 相关的jar包

![1552557572028](JavaWeb-02-request请求对象.assets/1552557572028.png)

### BeanUtils常用方法

| BeanUtils方法                                          | 说明                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| public static void populate(Object target, Map source) | 作用：将source中对应的键复制到target对应的属性中，如果不相同的属性忽略，如果数据类型不同，会自动转换。<br />参数1：要复制到的JavaBean对象<br />参数2：从哪里复制数据的Map |



### 案例：BeanUtils的使用

#### 需求

1. 得到表单所有的数据封装成Map
2. 使用BeanUtils封装成一个User对象，并且输出到浏览器。

#### 步骤

**==jar包必须放在web/WEB-INF/lib这个目录下==**

![1552557742580](JavaWeb-02-request请求对象.assets/1552557742580.png)

#### 效果

![1564630904312](JavaWeb-02-request请求对象.assets/1564630904312.png) 

#### 代码

##### JavaBean

```java
package com.itheima.model;

import java.util.Arrays;

public class User {

    private String name;

    private String gender;

    private String city;

    public User() {
    }

    public User(String name, String gender, String city) {
        this.name = name;
        this.gender = gender;
        this.city = city;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public void setCity(String city) {
        this.city = city;
    }



    public String getName() {
        return name;
    }

    public String getGender() {
        return gender;
    }

    public String getCity() {
        return city;
    }


    public String toString() {
        return "User{name = " + name + ", gender = " + gender + ", city = " + city + "}";
    }
}

```

##### Servlet

```java
package com.itheima.request;

import com.itheima.model.User;
import org.apache.commons.beanutils.BeanUtils;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.lang.reflect.InvocationTargetException;
import java.util.Arrays;
import java.util.Enumeration;
import java.util.Map;
/*
学习目标： 通过request请求对象获取请求参数，然后使用Beanutils把请求参数封装到javaBean(实体类)对象。

request获取请求参数相关的方法：
          4. Map<String,String[]>   getParameterMap()   获取所有的请求参数，并且把请求参数封装到一个map对象中返回。
易错点：
    1. 在web模块导入jar的时候一定要在web目录下创建一个WEB-INF目录，然后在WEB-INF目录下创建一个lib文件夹。
    2. Beanutils底层依赖实体类的Setter方法封装参数的，一旦没有setxxx的方法，那么没法封装
    3. 请求参数的名字一定要与实体类的属性名对应，否则没法封装
 */

@WebServlet("/parameter2")
public class _05ParameterServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        // 获取所有的请求参数，并且把请求参数封装到一个map对象中返回。
        // name ,gowau
        //gender, 男
        Map<String, String[]> map = request.getParameterMap();
        //使用Beanutils封装参数
        User user = new User();
        try {
            //populate方法就会读取map中数据交给user对象
            BeanUtils.populate(user,map);
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }

        out.write("封装的参数："+ user);


    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

### 小结

| **BeanUtils常用方法**                                        | **说明**                       |
| ------------------------------------------------------------ | ------------------------------ |
| **public   static void populate(Object target, Map  source)** | 把请求参数自动封装到javabean中 |



## 11.请求参数值汉字乱码的问题

### 目标

解决POST方法提交乱码的问题

### 请求参数值产生乱码的原因

![1602312115315](JavaWeb-02-request请求对象.assets/1602312115315.png)

### POST方法乱码解决方法

1. 方法：

   ```
   //设置请求的编码
   request.setCharacterEncoding("utf-8");
   ```

2. 位置：必须放在所有得到参数值的方法前面

3. 必须与HTML页面编码一致

### Tomcat8.0中GET方法乱码的问题

		- tomcat8以下版本不管get还是post都会乱码的。
		- tomcat8开始只有post请求有乱码，get请求没有乱码。 

### 小结

1. POST乱码如何解决

   - request.setCharacterEncoding("与页面一样编码")

2. GET乱码如何解决

   - 不需要解决，因为没有乱码



## <font color="red">12.请求域有关的方法</font>

### 目标

1. 有哪三个作用域
2. 操作作用域的方法

### 什么是作用域

==request对象是一个域对象==

作用域是服务器内存中一块区域，==用于实现不同的Servlet之间数据共享==，实现不用的用户之间数据共享。

作用域底层是Map结构，键是String类型，值是Object类型。作用域中可以存放任意的类型。

![1552559439793](JavaWeb-02-request请求对象.assets/1552559439793.png)

**域的作用：**

1. 域对象本身就是一个容器就相当于是一个集合。  request对象本身就是域对象。
2. 用于不同的Servlet直接实现数据的共享。 





#### 请求域的范围

只在同一次请求中起作用

### 作用域的操作方法

| **request与域有关的方法**               | **作用**                   |
| --------------------------------------- | -------------------------- |
| **Object getAttribute("键")**           | 通过键从请求域中取得一个值 |
| **void setAttribute("键",Object 数据)** | 往请求域中存入键和值       |
| **void removeAttribute("键")**          | 删除请求域中指定的键和值   |



### 案例：作用域的操作方法

#### 需求

1. Servlet创建一个键和值，在同一个Servlet中从请求域中取出信息，打印到浏览器上。
2. 删除请求域中的键值对，再打印到浏览器上。

#### 效果

![1564642001535](JavaWeb-02-request请求对象.assets/1564642001535.png) 

#### 代码

```java
package com.itheima.request;

import com.itheima.model.User;
import org.apache.commons.beanutils.BeanUtils;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.lang.reflect.InvocationTargetException;
import java.util.Map;
/*
学习目标： 学习request请求域相关的方法

request相关的方法：
    1. setAttribute(key,value)  往request请求域中添加数据
    2. getAttribute(key) 从request请求域获取数据
    3. removeAttribute(key) 从request请求域中删除数据

 */

@WebServlet("/attribute")
public class _06AttributeServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();

        //1往request域中存储数据
        request.setAttribute("username","jack");

        //2.从request域中删除数据
        request.removeAttribute("username");

        //2. 从request域中获取数据
        String username = (String) request.getAttribute("username");

        out.write("获取到的数据："+ username);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

### 小结

2. 说说作用域方法的作用：

   | **request域有关的方法**                 | **作用**              |
   | --------------------------------------- | --------------------- |
   | **Object getAttribute("键")**           | 从request域中获取数据 |
   | **void setAttribute("键",Object 数据)** | 往request域中存储数据 |
   | **void removeAttribute("键")**          | 从request域中删除数据 |



## 13.页面的跳转：转发

### 目标

1. 转发的原理
2. 转发的方法

### 页面跳转的方式

- 请求转发

- 请求重定向


### 什么是转发

#### 概念

在服务器端进行的页面跳转，称为转发

#### 原理图

![1602314524502](JavaWeb-02-request请求对象.assets/1602314524502.png)



#### 转发的方法

```
//1. 得到转发器，参数：要跳转的地址
RequestDispatcher dispatcher = request.getRequestDispatcher("/demo7");

//2. 调用转发器forward方法跳转，参数：请求对象，响应对象
dispatcher.forward(request, response);
```

```
request.getRequestDispatcher("/demo7").forward(request,response);
```



### 案例

#### 需求

​	实现从Demo7Servlett中转发到Demo8Servlett

#### 步骤

1. Demo7Servlett向请求域中添加了一个键和值，转发给Demo8Servlett
2. Demo8Servlett就从请求域中取出键和值，打印到浏览器上。

#### 效果

![1564642807111](JavaWeb-02-request请求对象.assets/1564642807111.png) 

#### 代码

##### Demo7Servlett

```java
package com.itheima.request;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.ArrayList;
import java.util.List;

/*
学习目标： 学习请求转发

请求转发的方法：
    request.getRequestDispatcher("/forward2").forward(request,response);

疑问：
    为什么request对象可以共享数据呢？ 07Servlet是如何实现把数据共享给08Servlet?


 */
@WebServlet("/forward1")
public class _07AttributeServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        //往request域中存储数据
        request.setAttribute("username","傻逼");
        //请求转发
        //得到请求转发的转发器
       request.getRequestDispatcher("/forward2").forward(request,response);

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

##### Demo8Servlett

```java
package com.itheima.request;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
/*
学习目标： 学习request请求转发的方法

request相关的方法：
    1. setAttribute(key,value)  往request请求域中添加数据
    2. getAttribute(key) 从request请求域获取数据
    3. removeAttribute(key) 从request请求域中删除数据

 */

@WebServlet("/forward2")
public class _08AttributeServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();

        //从request域中获取数据
        Object username = request.getAttribute("username");
        out.write("forward2从rquest域中获取到的内容："+ username);


    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

### 转发的特点

1. 浏览器地址栏： 不变，因为浏览器只是发出了一次请求
2. 浏览器发出请求次数:  1次
3. 为什么请求转发可以实现数据的共享？ 因为是同一个request对象

  

### 小结

转发使用哪个方法？

- request.getRequestDispatcher("路径").forward(request,respoinse)

## 14.页面的跳转：请求重定向

### 目标

1. 重定向原理
2. 重定向的方法

### 什么是重定向

#### 概念

由浏览器端进行的页面跳转

#### 原理图

![1602315983455](JavaWeb-02-request请求对象.assets/1602315983455.png)

#### 重定向方法

```
response.sendRedirect("跳转的地址")
```



### 重定向案例

#### 需求

从Demo7Servlet重定向到Demo8Servlet

#### 步骤

1. 在Demo7Servlet中向请求域中添加键和值
2. 使用重定向到Demo8Servlet，在TwoServlet中看能否取出请求域的值

#### 效果

![1564643946511](JavaWeb-02-request请求对象.assets/1564643946511.png) 

#### 代码

##### Demo7Servlet

```java
package com.itheima.request;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.ArrayList;
import java.util.List;

/*
学习目标： 学习请求转发

请求转发的方法：
    request.getRequestDispatcher("/forward2").forward(request,response);

请求重定向 ：
    response.sendRedirect("路径")

请求转发与请求重定向的相同点：都可以进行页面跳转。


疑问：
    为什么request对象可以共享数据呢？ 07Servlet是如何实现把数据共享给08Servlet?


 */
@WebServlet("/forward1")
public class _07AttributeServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        //往request域中存储数据
        request.setAttribute("username","傻逼");
        //请求转发
        //得到请求转发的转发器
      // request.getRequestDispatcher("/forward2").forward(request,response);

        //请求重定向
        response.sendRedirect("/day27/forward2");
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

##### Demo8Servlet

```java
package com.itheima.request;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
/*
学习目标： 学习request请求转发的方法

request相关的方法：
    1. setAttribute(key,value)  往request请求域中添加数据
    2. getAttribute(key) 从request请求域获取数据
    3. removeAttribute(key) 从request请求域中删除数据

 */

@WebServlet("/forward2")
public class _08AttributeServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();

        //从request域中获取数据
        Object username = request.getAttribute("username");
        out.write("forward2从rquest域中获取到的内容："+ username);


    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

### 重定向的特点

1. 地址栏：改变，因为两次请求都是浏览器发出

2. 浏览器发出请求次数： 2次

3. 能不能使用request共享数据？不行，因为创建了两个request

   

### 疑问

问：什么时候使用转发，什么时候使用重定向？

- 如果需要使用request域共享数据的时候就使用请求转发
- 如果不需要使用request域对象共享数据则使用请求重定向




### 小结：重定向和转发的区别

| **区别**         | **转发forward()** | **重定向sendRedirect()** |
| ---------------- | ----------------- | ------------------------ |
| **地址栏**       | 不会变化          | 会显示新的地址           |
| **哪里跳转**     | 服务器端          | 浏览器端                 |
| **请求域中数据** | 不会丢失          | 会丢失                   |
| 请求的资源范围   | 本项目的资源      | 任何资源                 |
| 地址上区别       | /资源地址         | /项目的根路径/资源地址   |

### "/"路径说明

- 在web项目中“/”如果是给服务器去使用那么“/”代表了：  http://localhost:8080/项目根路径
- 如果"/"是给浏览器去使用，那么“/”代表了：   http://localhost:8080/

==只有请求转发的时候路径是不需要编写项目的根路径的，其他时候都需要写==

## 15.登录案例part1：项目搭建(不睡觉也要写出来)

### 目标

搭建项目的结构

### 需求

1. 用户名和密码正确，将用户成功的信息保存在请求域中，转发到另一个页面，显示用户登录成功
2. 用户名和密码错误，重定向到另一个html页面，显示登录失败。
3. 使用表示层，业务层，数据访问层的三层结构实现

### 案例流程图

![1602317230044](JavaWeb-02-request请求对象.assets/1602317230044.png)



### 实现步骤

#### 创建用户表

```mysql
create table `user`(
    id int primary key auto_increment,
    username varchar(20),
    password varchar(32)
 );
 
 insert into `user`(username, password) values ('Jack','123'),('Rose','456');
 
 select * from `user`;
```

#### 登录页面

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>用户登录</title>
</head>
<body>
<h2>用户登录</h2>
<form action="/day27_login/userServlet" method="post">
    <table>
        <tr>
            <td>用户名</td>
            <td><input type="text" name="username"/></td>
        </tr>
        <tr>
            <td>密码</td>
            <td><input type="password" name="password"/></td>
        </tr>
        <tr>
            <td colspan="2"><input type="submit" value="登录"/></td>
        </tr>
    </table>
</form>
</body>
</html>
```

#### 创建WEB-INF\lib文件夹，导入jar包

![1564645434661](JavaWeb-02-request请求对象.assets/1564645434661.png) 

#### 实体类User

```java
package com.itheima.domain;

public class User {

    private int id;

    private String username;

    private String password;


    public User() {
    }

    public User( String username, String password) {
        this.username = username;
        this.password = password;
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
     * @return password
     */
    public String getPassword() {
        return password;
    }

    /**
     * 设置
     * @param password
     */
    public void setPassword(String password) {
        this.password = password;
    }

    public String toString() {
        return "User{id = " + id + ", username = " + username + ", password = " + password + "}";
    }
}

```

#### sqlMapConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!--加载外部的properties配置-->
    <properties resource="jdbc.properties"></properties>


    <!--设置java类型别名-->
    <typeAliases>
        <!--将整个包下所有的类名设置了别名，别名（小名）：类名-->
        <package name="com.itheima.entity"></package>
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

#### db.properties

```properties
jdbc.url=jdbc:mysql:///db1
jdbc.username=root
jdbc.password=root
jdbc.driver=com.mysql.jdbc.Driver
```





#### 会话工具类

```java
package com.itheima.utils;

import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.InputStream;

/**
该工具类目的： 获取SqlSessionFactory

 */
public class SqlSessionUtils {

    private static SqlSessionFactory factory;


    static{
        //1. 创建sqlSesionFactory构造器
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        //使用sqlSesionFactory构造器读mybatis核心配置文件
        InputStream inputStream = SqlSessionUtils.class.getResourceAsStream("/SqlMapConfig.xml");
         factory = builder.build(inputStream);
    }

    public static SqlSessionFactory getFactory() {
        return factory;
    }

    public static void setFactory(SqlSessionFactory factory) {
        SqlSessionUtils.factory = factory;
    }
}
```

### 创建对应的包（分层）

![1566720888063](JavaWeb-02-request请求对象.assets/1566720888063.png)









## 16.登录案例part2：数据访问层和业务层

### 目标

1. 编写UserDao代码
2. 编写UserService代码

### 步骤

#### UserDao

通过用户名和密码查询一个用户

#### UserService

业务类直接调用DAO类中相应的方法返回用户对象即可

### 代码

#### UserDao

```java
package com.itheima.dao;

import com.itheima.entity.User;
import org.apache.ibatis.annotations.Select;

public interface UserDao {

    //登录的方法
    @Select("select * from user where username= #{username} and password=#{password}")
    public User login(User user);
}

```

#### UserService

```java
package com.itheima.service;

import com.itheima.dao.UserDao;
import com.itheima.entity.User;
import com.itheima.utils.SqlSessionUtils;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;

//约定： service层代码调用dao层代码
public class UserService {

    public boolean login(User user){
        //得到SessionFactory工厂
        SqlSessionFactory factory = SqlSessionUtils.getFactory();
        //得到sqlSesion
        SqlSession sqlSession = factory.openSession();
        //得到UserDao的接口的代理对象
        UserDao userDao = sqlSession.getMapper(UserDao.class);
        //调用登陆的方法
        User dbUser = userDao.login(user);
        return dbUser!=null; //！=null代表查找到的用户，登录成功

    }
}

```



## 17.登录案例part3：Servlet

### 目标

1. 登录的LoginServlet
2. 登录成功的SuccessServlet

### UserServlet

#### 步骤

1. 解决汉字乱码问题
2. 得到用户提交的用户名和密码
3. 调用业务层实现登录操作
4. 如果用户名不为空表示登录成功
5. 把当前的用户信息放在请求域中，以后放在会话域中
6. 转发到success
7. 如果登录失败重定向到failure.html页面

#### 代码

```java
package com.itheima.web;

import com.itheima.entity.User;
import com.itheima.service.UserService;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/userServlet")
public class LoginServlet extends HttpServlet {

    private UserService userService = new UserService();

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");

        //1. 解决post请求参数的乱码问题
        request.setCharacterEncoding("utf-8");
        //2. 获取请求参数
        String username = request.getParameter("username");
        String password = request.getParameter("password");

        //3. 把请求参数封装到javaBean
        User user = new User(username,password);

        //4. 校验用户名与密码
        boolean flag = userService.login(user);

        if(flag){
            //5。校验成功跳转SuccessServlet
            //把用户名存储到reuqest域对象中
            request.setAttribute("username",username);
            //由于是使用request进行共享数据，所以需要使用请求转发
            request.getRequestDispatcher("/success").forward(request,response);

        }else{
            //6. 校验失败调整到fail.html页面
            //由于不需要共享数据，所以使用请求重定向
            response.sendRedirect("/day27_login/fail.html");

        }





    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```



### SuccessServlet

#### 步骤

1. 设置响应的类型和字符集
2. 得到打印流对象
3. 从请求域中取出用户信息
4. 输出登录成功的信息

#### 代码

```java
package com.itheima.web;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/success")
public class SuccessServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        //从request域获取用户名
        Object username = request.getAttribute("username");
        //
        out.write("欢迎：<font color='red'>"+username+"</font>登陆成功");
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

### fail.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h3>用户名不存在或者密码错误</h3>

</body>
</html>
```









### 小结

1. 用了三层结构：表示层(HTML+Servlet)，业务层，数据访问层
2. DAO层使用了查询方法:
3. 业务层直接调用DAO层中的方法，实现登录
4. Servlet中用到了得到表单参数的方法：getParameter()
5. Servlet中用到了转发和重定向
6. Servlet中用到了请求域有关的方法：getAttribute()取出请求域中的数据





## 18.缺省路径(拓展)

**缺省路径写法**：/或者/*  这两种代表可以匹配任何内容。 



==注意： 任何servlet不允许配置缺省路径，因为一旦配置就会导致整个模块的静态资源无法访问。==



原因： tomcat底层对于访问一个静态的资源的时候，是依赖DefaultServlet去读取静态文件想浏览器输出的。，

而DefaultServlet配置映射路径就是这种缺省路径，如果你一旦配置了缺省路径那么就会导致DefaultServlet失效，那么就没有servlet去读取静态资源了。 



## 19 配置文件的模板创建(拓展)

1. **找到配置的Live template**
   ![1602321286427](JavaWeb-02-request请求对象.assets/1602321286427.png)

![1602321360673](JavaWeb-02-request请求对象.assets/1602321360673.png)



2. **创建模板的组**

![1602321436725](JavaWeb-02-request请求对象.assets/1602321436725.png)

![1602321480931](JavaWeb-02-request请求对象.assets/1602321480931.png)



3. 创建模板

   ![1602321530139](JavaWeb-02-request请求对象.assets/1602321530139.png)

![1602321635058](JavaWeb-02-request请求对象.assets/1602321635058.png)

![1602321686873](JavaWeb-02-request请求对象.assets/1602321686873.png)

![1602321732337](assets/1602321732337.png)