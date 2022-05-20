---
title: JavaWeb(05)-el、jstl和mvc
tags:
  - 笔记
  - JavaWeb
  - jstl
  - el表达式
  - mvc
categories:
  - JavaWeb
date: 2020-10-14 19:42:34
---
#  学习目标

1、能够在jsp中编写java代码 



2、能够说出el表达式的作用 



3、能够使用el表达式获取javabean的属性 



4、能够使用jstl标签库的if标签 

​	

5、能够使用jstl标签库的foreach标签 



# 学习内容

## 1. JSP的优势

### 目标

1. 了解JSP的特点
2. 体验第1个JSP的编写

### 概念

JSP：Java Server Page 运行在服务器上的Java页面。在浏览器上看到的结果其实是JSP在服务器上运行的结果。

HTML: 静态页面，运行在浏览器上	

### 各种技术的特点

| **技术**    | **特点**                                          |
| ----------- | ------------------------------------------------- |
| **HTML**    | 只能编写html标签                                  |
| **Servlet** | 编写java代码非常方便， 编写html代码会感觉非常麻烦 |
| **JSP**     | 编写java代码与编写html代码都非常的方便。          |



### 2. JSP的第1个案例

#### 需求

在浏览器上输出服务器当前的时间，并且设置样式

#### Servlet代码

```jsp
package com.itheima.web;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.text.SimpleDateFormat;
import java.util.Date;

/*
学习目标：体验jsp


需求： 向页面输出当前的系统时间


 */
@WebServlet("/demo1")
public class Demo1Servlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        //得到当前系统时间
        String curTime = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss").format(new Date());
        //向页面输出数据
        out.write("<h1>当前系统时间是：<span style='color:green'>"+curTime+"</span></h1>");
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```



#### jsp代码代码完成

```java
<%@ page import="java.text.SimpleDateFormat" %>
<%@ page import="java.util.Date" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>体验jsp</title>
    <style>
        h1{
            border: 1px dashed yellow;
        }

        span{
            color: chartreuse;
        }
    </style>
</head>
<body>
<%
    //得到当前系统时间
    String curTime = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss").format(new Date());
%>

<h1>当前的系统时间：<span><%out.write(curTime);%></span></h1>

</body>
</html>

```



### 小结

  1. JSP的优势

     - 既可以编写java代码也可以编写html代码。 
     
       
     

<font color="red" size="3px">疑问：</font> 

  - **为什么jsp可以直接使用out、request、response对象**
    - 因为我们编写的代码落入jsp翻译文件中的service方法内部，而service方法在一开始的时候就声明大量的对象提供给你们去使用
  - **jsp可以编写java代码也可以编写html代码是如何做到的？**
    	- servlet编写java代码自然可以的
        	- jsp之所以可以编写html代码，其实本质上也是在做一个拼接字符串的操作。 

## 2. JSP运行原理

### 目标

了解JSP与Servlet之间是什么关系

### JSP的执行过程

1. JSP由Tomcat翻译成Java代码，而这个Java类继承HttpJspBase，而HttpJspBase继承HttpServlet.
2. 将Servlet编译成字节码文件，由JVM来执行

![1544403634800](assets/1544403634800.png)

### 疑问

1. Servlet生成在哪个位置？

   ```
   C:\Users\ztl\.IntelliJIdea2017.3\system\tomcat\Tomcat_8_5_5_142project_7\work\Catalina\localhost\day30\org\apache\jsp
   ```

   ![1564968822973](assets/1564968822973.png) 

2. JSP和Servlet是什么关系？

   ```java
   public final class demo01_jsp extends org.apache.jasper.runtime.HttpJspBase
   public abstract class HttpJspBase extends HttpServlet 
结论：JSP本质上就是一个Servlet
   
   原来的HTML，使用out.write()输出
   原来的Java代码，原样不动的复制过去了
   ```
   
3. Servlet的生命周期

   ```
   init()
   service()  --> 我们在JSP中写的所有的代码，都是在_jspService()方法中
   destory()
   ```

   


### 小结

1. JSP的执行过程有哪两个阶段？

   - 翻译
   - 编译
   
2. JSP与Servlet是什么关系？

   - jsp本质上就是一个servlet

   

3. 我们编写的代码全部落入那个方法内部

   - service方法里面

   

## 3. JSP的3种脚本元素

### 目标

​	学习使用JSP的三种脚本元素

1. 脚本表达式
2. 代码片段
3. 声明



### JSP脚本表达式

#### 语法格式

```
<%=变量名或表达式%>   作用： 相当于out.write() 输出一个数据到页面上。
```

#### 代码演示

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

<%--
jsp编写java代码的三种方式
    1. 代码片段
    2. 脚本表达式
    3. jsp声明

脚本表达式的格式：
    <%=
        变量||常量
     %>

脚本表达式的作用： 相当于out.write()输出语句
--%>

<%
    String info = "行动大于一切！";
%>


<%--
<%out.write(info);%>
--%>

<%--脚本表达式--%>
<%=info%>

</body>
</html>

```



### JSP代码片段

#### 语法和作用

```
<%
   Java代码，只要符合Java语法规则都可以;
%>
```



#### 代码

```jsp
<%@ page import="java.util.ArrayList" %>
<%@ page import="java.util.List" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>jsp编写java代码的三种方式</title>
</head>
<body>
<%--
jsp编写java代码的三种方式
    1. 代码片段
    2. 脚本表达式
    3. jsp声明

代码片段格式：
    <%
        java 代码
     %>
代码片段的特点：
    1. 一个jsp里面可以编写多个代码片段
    2. 代码片段的代码都落入到service方法的内部
--%>

<%
    List<String> list = new ArrayList<>();
    list.add("狗娃");
    list.add("狗剩");
    list.add("狗蛋");
%>

<table>
    <tr>
        <th>姓名</th>
    </tr>
    <%
        for (String name : list) {
    %>
    <tr>
        <td><%out.write(name);%></td>
    </tr>

    <% } %>
</table>






</body>
</html>

```



### JSP的声明

#### 语法和作用

```
<%! 声明成员变量或方法 %>
声明方法很少使用，因为方法写在类中，只需要在JSP上创建对象调用方法即可
主要定义全局变量
```

####  

#### 代码

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

<%--
jsp编写java代码的三种方式
    1. 代码片段
    2. 脚本表达式
    3. jsp声明

jsp的声明的格式：
    <%!
        成员变量或者是成员方法的
     %>

--%>



<%!
    //定义一个了一个成员方法
    public void sum(int a, int b){
        int result = a+b;
        System.out.println("结果："+result);
    }


%>


<%
    //在service的方法里面调用
    sum(1,2);
%>



</body>
</html>

```

### 小结

| **组成部分**      | 特点                          | **语法**                  |
| ----------------- | ----------------------------- | ------------------------- |
| **JSP代码片段**   | 代码全部都落入service方法内部 | <%java代码%>              |
| **JSP脚本表达式** | 相当于out.write()             | <%=变量%>                 |
| **JSP声明**       | 用于声明成员变量或者成员方法  | <%!成员变量\|\|成员方法%> |



## 4. JSP的三大指令

### 目标

学习JSP中三大指令和它的作用

### 三大指令是

1. page(重点)
2. taglib(重点)
3. include

### 指令的格式

```jsp
<%@指令名 属性名="属性值"%>
```

### taglib指令

1. 作用：标签库指令，用于导入其他的标签库的，让jsp页面的标签更加丰富。

2.  语法：

```jsp
<%@taglib prefix="前缀" uri="标签库标识" %>
```

### include指令（静态包含）

1. 作用：用于静态包含其它的页面

2. 语法： 

```jsp
<%@include file="被包含的JSP页面"%>
```

### include指令演示

#### 需求

一个index.jsp和login.jsp包含另一个head.jsp

#### 代码

```jsp
<%@page contentType="text/html;charset=utf-8" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>黑马旅游网</title>
    <!-- Bootstrap -->
    <link href="css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" type="text/css" href="css/common.css">
    <link rel="stylesheet" type="text/css" href="css/index.css">
</head>
<body>
<!--引入头部-->
<div id="header">
     <%--底层是使用了服务器去读取路径的，/如果被服务器读取的时候/ 代表了http://localhost:8080/项目根路径--%>
    <%@include file="/header.jsp" %>
</div>
<!-- banner start-->
<section id="banner">
    <div id="carousel-example-generic" class="carousel slide" data-ride="carousel" data-interval="2000">
        <!-- Indicators -->
        <ol class="carousel-indicators">
            <li data-target="#carousel-example-generic" data-slide-to="0" class="active"></li>
            <li data-target="#carousel-example-generic" data-slide-to="1"></li>
            <li data-target="#carousel-example-generic" data-slide-to="2"></li>
        </ol>
        <!-- Wrapper for slides -->
        <div class="carousel-inner" role="listbox">
            <div class="item active">
                <img src="images/banner_1.jpg" alt="">
            </div>
            <div class="item">
                <img src="images/banner_2.jpg" alt="">
            </div>
            <div class="item">
                <img src="images/banner_3.jpg" alt="">
            </div>
        </div>
        <!-- Controls -->
        <a class="left carousel-control" href="#carousel-example-generic" role="button" data-slide="prev">
            <span class="glyphicon glyphicon-chevron-left" aria-hidden="true"></span>
            <span class="sr-only">Previous</span>
        </a>
        <a class="right carousel-control" href="#carousel-example-generic" role="button" data-slide="next">
            <span class="glyphicon glyphicon-chevron-right" aria-hidden="true"></span>
            <span class="sr-only">Next</span>
        </a>
    </div>
</section>
<!-- banner end-->
<!-- 旅游 start-->
<section id="content">
    <!-- 黑马精选start-->
    <section class="hemai_jx">
        <div class="jx_top">
            <div class="jx_tit">
                <img src="images/icon_5.jpg" alt="">
                <span>黑马精选</span>
            </div>
            <!-- Nav tabs -->
            <ul class="jx_tabs" role="tablist">
                <li role="presentation" class="active">
                    <span></span>
                    <a href="#popularity" aria-controls="popularity" role="tab" data-toggle="tab">人气旅游</a>
                </li>
                <li role="presentation">
                    <span></span>
                    <a href="#newest" aria-controls="newest" role="tab" data-toggle="tab">最新旅游</a>
                </li>
                <li role="presentation">
                    <span></span>
                    <a href="#theme" aria-controls="theme" role="tab" data-toggle="tab">主题旅游</a>
                </li>
            </ul>
        </div>
        <div class="jx_content">
            <!-- Tab panes -->
            <div class="tab-content">
                <div role="tabpanel" class="tab-pane active" id="popularity">
                    <div class="row">
                        <div class="col-md-3">
                            <a href="javascript:;">
                                <img src="images/jiangxuan_4.jpg" alt="">
                                <div class="has_border">
                                    <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                    <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                                </div>
                            </a>
                        </div>
                        <div class="col-md-3">
                            <a href="javascript:;">
                                <img src="images/jiangxuan_4.jpg" alt="">
                                <div class="has_border">
                                    <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                    <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                                </div>
                            </a>
                        </div>
                        <div class="col-md-3">
                            <a href="javascript:;">
                                <img src="images/jiangxuan_4.jpg" alt="">
                                <div class="has_border">
                                    <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                    <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                                </div>
                            </a>
                        </div>
                        <div class="col-md-3">
                            <a href="javascript:;">
                                <img src="images/jiangxuan_4.jpg" alt="">
                                <div class="has_border">
                                    <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                    <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                                </div>
                            </a>
                        </div>
                    </div>
                </div>
                <div role="tabpanel" class="tab-pane" id="newest">
                    <div class="row">
                        <div class="col-md-3">
                            <a href="javascript:;">
                                <img src="images/jiangxuan_1.jpg" alt="">
                                <div class="has_border">
                                    <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                    <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                                </div>
                            </a>
                        </div>
                        <div class="col-md-3">
                            <a href="javascript:;">
                                <img src="images/jiangxuan_1.jpg" alt="">
                                <div class="has_border">
                                    <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                    <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                                </div>
                            </a>
                        </div>
                        <div class="col-md-3">
                            <a href="javascript:;">
                                <img src="images/jiangxuan_1.jpg" alt="">
                                <div class="has_border">
                                    <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                    <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                                </div>
                            </a>
                        </div>
                        <div class="col-md-3">
                            <a href="javascript:;">
                                <img src="images/jiangxuan_1.jpg" alt="">
                                <div class="has_border">
                                    <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                    <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                                </div>
                            </a>
                        </div>
                    </div>
                </div>
                <div role="tabpanel" class="tab-pane" id="theme">
                    <div class="row">
                        <div class="col-md-3">
                            <a href="javascript:;">
                                <img src="images/jiangxuan_2.jpg" alt="">
                                <div class="has_border">
                                    <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                    <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                                </div>
                            </a>
                        </div>
                        <div class="col-md-3">
                            <a href="javascript:;">
                                <img src="images/jiangxuan_2.jpg" alt="">
                                <div class="has_border">
                                    <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                    <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                                </div>
                            </a>
                        </div>
                        <div class="col-md-3">
                            <a href="javascript:;">
                                <img src="images/jiangxuan_2.jpg" alt="">
                                <div class="has_border">
                                    <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                    <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                                </div>
                            </a>
                        </div>
                        <div class="col-md-3">
                            <a href="javascript:;">
                                <img src="images/jiangxuan_2.jpg" alt="">
                                <div class="has_border">
                                    <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                    <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                                </div>
                            </a>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
    <!-- 黑马精选end-->
    <!-- 国内游 start-->
    <section class="hemai_jx">
        <div class="jx_top">
            <div class="jx_tit">
                <img src="images/icon_6.jpg" alt="">
                <span>国内游</span>
            </div>
        </div>
        <div class="heima_gn">
            <div class="guonei_l">
                <img src="images/guonei_1.jpg" alt="">
            </div>
            <div class="guone_r">
                <div class="row">
                    <div class="col-md-4">
                        <a href="route_detail.html">
                            <img src="images/jiangxuan_4.jpg" alt="">
                            <div class="has_border">
                                <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                            </div>
                        </a>
                    </div>
                    <div class="col-md-4">
                        <a href="route_detail.html">
                            <img src="images/jiangxuan_4.jpg" alt="">
                            <div class="has_border">
                                <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                            </div>
                        </a>
                    </div>
                    <div class="col-md-4">
                        <a href="route_detail.html">
                            <img src="images/jiangxuan_4.jpg" alt="">
                            <div class="has_border">
                                <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                            </div>
                        </a>
                    </div>
                    <div class="col-md-4">
                        <a href="route_detail.html">
                            <img src="images/jiangxuan_4.jpg" alt="">
                            <div class="has_border">
                                <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                            </div>
                        </a>
                    </div>
                    <div class="col-md-4">
                        <a href="route_detail.html">
                            <img src="images/jiangxuan_4.jpg" alt="">
                            <div class="has_border">
                                <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                            </div>
                        </a>
                    </div>
                    <div class="col-md-4">
                        <a href="route_detail.html">
                            <img src="images/jiangxuan_4.jpg" alt="">
                            <div class="has_border">
                                <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                            </div>
                        </a>
                    </div>
                </div>
            </div>
        </div>
    </section>
    <!-- 国内游 end-->
    <!-- 境外游 start-->
    <section class="hemai_jx">
        <div class="jx_top">
            <div class="jx_tit">
                <img src="images/icon_7.jpg" alt="">
                <span>境外游</span>
            </div>
        </div>
        <div class="heima_gn">
            <div class="guonei_l">
                <img src="images/jiangwai_1.jpg" alt="">
            </div>
            <div class="guone_r">
                <div class="row">
                    <div class="col-md-4">
                        <a href="route_detail.html">
                            <img src="images/jiangxuan_4.jpg" alt="">
                            <div class="has_border">
                                <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                            </div>
                        </a>
                    </div>
                    <div class="col-md-4">
                        <a href="route_detail.html">
                            <img src="images/jiangxuan_4.jpg" alt="">
                            <div class="has_border">
                                <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                            </div>
                        </a>
                    </div>
                    <div class="col-md-4">
                        <a href="route_detail.html">
                            <img src="images/jiangxuan_4.jpg" alt="">
                            <div class="has_border">
                                <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                            </div>
                        </a>
                    </div>
                    <div class="col-md-4">
                        <a href="route_detail.html">
                            <img src="images/jiangxuan_4.jpg" alt="">
                            <div class="has_border">
                                <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                            </div>
                        </a>
                    </div>
                    <div class="col-md-4">
                        <a href="route_detail.html">
                            <img src="images/jiangxuan_4.jpg" alt="">
                            <div class="has_border">
                                <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                            </div>
                        </a>
                    </div>
                    <div class="col-md-4">
                        <a href="route_detail.html">
                            <img src="images/jiangxuan_4.jpg" alt="">
                            <div class="has_border">
                                <h3>上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</h3>
                                <div class="price">网付价<em>￥</em><strong>889</strong><em>起</em></div>
                            </div>
                        </a>
                    </div>
                </div>
            </div>
        </div>
    </section>
    <!-- 境外游 end-->
</section>
<!-- 旅游 end-->
<!--导入底部-->
<div id="footer"></div>
<!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
<script src="js/jquery-3.3.1.js"></script>
<!-- Include all compiled plugins (below), or include individual files as needed -->
<script src="js/bootstrap.min.js"></script>
<!--导入布局js，共享header和footer-->
<script type="text/javascript" src="js/include.js"></script>
</body>
</html>
```

#### 效果

![1563115464530](assets/1563115464530.png)

### 小结

**三大指令是什么？分别的作用是？**

1. taglib
2. include





## <font color="red">5. JSP的page指令</font>

### page指令概述

1. 作用：设置JSP网页中一些属性，告诉web容器，将一个JSP页面翻译Servlet的参数。

2. 位置：可以放在网页任何一个位置，但建议在网页最上面。

### 导包的属性

```
language="java"  指定JSP页面使用的编译语言，默认是Java,而且只有Java
import="java.util.*" 作用：导包
```

#### 方式一

```jsp
<%@ page import="java.util.ArrayList" %>
<%@ page import="java.util.List" %>
每个import导入一个包
```

#### 方式二

```jsp 
<%@ page import="java.util.Date,java.text.SimpleDateFormat" %>
一个import导入多个包，包之间使用逗号隔开
```



### 与编码相关的属性

``` jsp
contentType="text/html; charset=utf-8"
JSP上汉字必须添加这个属性，否则汉字会有乱码的问题
相当于Servlet中response.setContentType("text/html;charset=UTF-8")
```



### 与错误相关的属性

```jsp
<%@page contentType="text/html;charset=UTF-8" import="java.util.ArrayList,java.util.List" language="java" errorPage="error.jsp" %>
<%--
page指令常用的属性：
    1. contentType    相当于servlet里面response.setContentType("text/html;charset=utf-8")
    2. language    当前jsp使用的语言
    3.import       导包
         3.1  使用多个page指令导包
         3.2  使用一个page指令导包，包与包之间使用，分割
   4.erorPage   配置错误页面，如果当前页面一旦出错，直接跳转到错误页面中
   5. isErrorPage  表示当前页面是一个错误的页面，错误页面可以直接使用exception对象。
--%>
<html>
<head>
    <title>Title</title>
</head>
<body>

<%
    List<String> list = new ArrayList<>();
    int i = 1/0; //出现异常
%>

</body>
</html>

```

``` jsp
<%@ page import="java.io.PrintWriter" %>
<%@ page import="java.io.PrintStream" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" isErrorPage="true" %>
<html>
<head>
    <title>Title</title>

</head>
<body>
<h2>由于商品的价格比较受欢迎导致访问人数过多，请稍后！</h2>

<%
//    收集异常的日志
    PrintWriter printWriter = new PrintWriter("H:/20201014.log");
    exception.printStackTrace(printWriter);
    printWriter.close();

%>

</body>
</html>

```

![1564972718285](assets/1564972718285.png)

### 错误页面的跳转的三种配置方式

#### 方式一：errorPage

```
<%@ page contentType="text/html;charset=utf-8" language="java" errorPage="404.jsp" %>
```



#### 方式二：错误码

在web.xml配置文件中指定错误码的方式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <!--根据转态码配置错误页面-->
    <error-page>
        <!--错误的转态码-->
        <error-code>500</error-code>
        <location>/error.jsp</location>
    </error-page>





</web-app>
```



#### 方式三：指定错误的类型

```xml

    <!--根据异常的类型配置错误页面-->
    <error-page>
        <exception-type>java.lang.ArithmeticException</exception-type>
        <location>/error.jsp</location>
    </error-page>
    

```

### 小结

| **属性名**                                         | **作用**                                                     |
| -------------------------------------------------- | ------------------------------------------------------------ |
| **language="java"**                                | 指定JSP页面使用语言                                          |
| **import="{package.class \| package.\*}"**         | 导入包：<br />1. 每个import包入一个包<br />2. 一个import导入多个包，使用逗号分隔 |
| **errorPage="relative_url"**                       | 当前页面出错，转发到哪个错误页面                             |
| **isErrorPage="true\|false"**                      | 当前是否是一个错误页面,如果是错误页面可以多一个exception的内置对象。 |
| **contentType="text/html;charset=characterSet ]"** | 指定页面类型和编码，不指定会有乱码                           |



## 6. JSP的动作(了解)

### 三个动作

jsp的内置标签，可以直接使用。以jsp开头

![1563116192201](assets/1563116192201.png)

### \<jsp:include>

1. 作用：用来动态包含另一个JSP页面

2. 语法： 

   ```jsp
   <jsp:include page="被包含页面"/>
   ```

### include动作与指令的区别

|                   | 静态包含                         | 动态包含                  |
| ----------------- | -------------------------------- | ------------------------- |
| **语法格式**      | \<%@include file="URL"%>         | <jsp:include page="URL"/> |
| **包含的方式**    | 在运行之前就把包含的页面拷贝过来 | 在运行的时候才输出的      |
| **生成Servlet个数 | 1个                              | 2个                       |
| 变量共享          | 共享使用                         | 不能共享使用              |

### 案例演示

#### 需求

1. 使用动态包含和静态包含分别实现index.jsp中包含header.jsp
2. 观察tomcat的work目录下生成几个Servlet

![1563116071877](assets/1563116071877.png)

#### 代码实现

7.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<%--

静态包含与动态包含区别是什么：
    1. 静态包含的时候会产生1个Servlet的翻译文件呢？动态包含的时候会产生2个Servlet的翻译文件呢？
    2. 静态包含的变量是可以共享使用的。


静态包含要注意的事项： 被包含的页面不要保留html、head、body这些标签。但是一定要保留page指令
--%>

<%--静态包含--%>

<%--
<%@include file="/8网页头部分.jsp"%>
--%>

<%--动态包含--%>
<jsp:include page="/8网页头部分.jsp"/>

<h1>这个是jsp体部分</h1>
</body>
</html>

```

8.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    String name = "呵呵";
%>

<h2>这个是jsp的头部分</h2>

```



#### 注意

如果2个页面中有相同的变量，不能使用静态包含，而应该使用动态包含。

#### 效果

![1564975498762](assets/1564975498762.png) 

![1564975571322](assets/1564975571322.png) 



## 7. forward和param动作

### 目标

1. forward作用
2. param的功能

### forward

1. 功能：用于转发，相当于request.getRequestDispatcher("/URL").forward(request, response)
2. 语法

```jsp
<jsp:forward page="/页面地址"/>
```

### param

1. 功能：给forward和include提供参数名和值
2. 语法：做为子标签存在

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

    <%
        //JSP使用forward标签传递参数的时候的乱码解决方案
        request.setCharacterEncoding("UTF-8");
    %>
    <%--该标签的作用就是实现请求转发--%>
    <jsp:forward page="/10.jsp">
        <jsp:param name="username" value="狗娃"/>
        <jsp:param name="password" value="abc123"/>
    </jsp:forward>


</body>
</html>

```

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<%
    String username = request.getParameter("username");
    String password = request.getParameter("password");
%>

<%
    out.write(username+"<br/>");
    out.write(password+"<br/>");
%>

</body>
</html>

```



### 小结

1. forward作用：请求转发
2. param的功能： 传递参数



### 面试题目

**说出jsp页面的九大内置对象：** 

- reuqest----------request
- response-----------response
- session-----------session
- out-------------response.getWriter()
- exception---------异常
- pageContext--------- 当前页面的上下文对象
- application----------ServletContext
- config---------ServletCOnfig
- page---------------this







## 8. jsp的最佳实践方式（工作应用场景）

在实际工作中，一般情况下我们都允许jsp直接编写java代码，因为会导致结构非常混乱，后期难以维护项目。 

所以在实际工作中，==我们一般都是使用servlet产生数据，jsp只需要显示数据即可，要做到jsp不出现一句java代码。==





## 9. EL的概念和它的两个作用

### 目标

​	什么是EL？

​	它有什么作用？

### 引入案例



#### Servlet代码

```java
package com.itheima.web;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.text.SimpleDateFormat;
import java.util.Date;

/*
学习目标：产生数据交给jsp的去显示


需求：

 */
@WebServlet("/demo2")
public class Demo2Servlet extends HttpServlet {



    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();

        String city = "东莞";
        //为了把数据共享给jsp，所以存储到域中
        request.setAttribute("city",city);
        //因为数据存储到request域中，所以我们需要请求转发
        request.getRequestDispatcher("/el/1体验el表达式.jsp").forward(request,response);



    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

#### jsp的代码（显示数据）

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<%--<%
    Object city = request.getAttribute("city");
    out.write("男人的天堂："+ city);
%>--%>

<%--

el表达式作用：
    1. 从域中查找数据   域对象.getAttribute("city");
    2. 并且把查找到的数据输出。out.write()

el表达式的格式：
     ${name}


--%>



    el表达式获取city: ${city}


</body>
</html>

```

### EL表达式的作用

#### 什么是EL

Expression Lanuage 表达式语言

#### 作用

为了简化JSP页面上Java代码

1. 取出作用域中值,并且输出

**el表达式的格式：**

```
${变量名或表达式}
```

### 小结

1.  **el表达式的格式如何？**

​         - ${存储在域的name}

2.   **el表达式的作用？**
    - 从域中取出数据
    - 把取出的数据输出。





## 10. EL表达式的搜索顺序

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<%
    pageContext.setAttribute("city","北京");
    request.setAttribute("city","上海");
    session.setAttribute("city","广州");
    application.setAttribute("city","深圳");
//    el表达式从域中去搜索数据的时候，搜索的顺序是从小到大搜索： pageContext---request----session---application

//如果四大域对象中存在同名的数据，我们需要指定获取某一个域中的数据的时候，我需要使用el表达式的内置对象去获取

%>

pageContext的数据：${pageScope.city}<br/>
request的数据：${requestScope.city}<br/>
session的数据：${sessionScope.city}<br/>
application的数据：${applicationScope.city}<br/>


</body>
</html>

```



## 11. pageContext对象与el表达式的内置

### 目标

分别使用JSP代码和EL从作用域中取数据

### 从指定的作用域中获取数据

#### 什么是页面域

- 对象名：pageContext，内置对象，可以直接在JSP上使用

- 范围：只在一个JSP页面上起作用

- 作用域大小比较： pageContext < request < session < application

- 底层数据结构：上面四个作用域底层结构都是Map


#### 四个与页面域有关的方法

![1552907105674](assets/1552907105674.png)

### pageContext的基本使用与作用

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<%
/*
   pageContext代表的当前页面的上下文对象，当前页面运行环境
    pageContext也是一个域对象。

    pageContext存在的方法：
        setAttribute（name，value）  存储数据
        getAttribute(name)  获取数据
        removeAttribute(name)  删除数据
        findAttribute(name) ,可以从四大域对象搜索数据， 这个方法其实就是el表达式底层使用的方法
*/

%>
<%
    session.setAttribute("city1","广州");
    request.setAttribute("city2","深圳");

%>

<%=pageContext.findAttribute("city1")%><br/>
<%=pageContext.findAttribute("city2")%><br/>


</body>
</html>

```





## <font color="red">12. 使用EL取出不同数据类型的值（非常重点）</font>

### 目标

使用EL，分别从JavaBean，Map，集合List，数组中取出它们的值

### 效果

![1564987665239](assets/1564987665239.png) 

### 代码

```jsp
<%@ page import="com.itheima.model.Person" %>
<%@ page import="java.util.List" %>
<%@ page import="java.util.ArrayList" %>
<%@ page import="java.util.HashMap" %>
<%@ page import="java.util.Map" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>5el表达式取出不同类型的数据</title>
</head>
<body>
<%
    Person person = new Person("狗娃",12,"女");
    //存储到域
    pageContext.setAttribute("person",person);
    /*
        取出javabean的属性数据的格式：
            ${javabean.属性名}

          el表达式取出javabean属性数据要注意的事项：
            1. el表达式取出javabean的属性数据底层是依赖getxxx方法,与成员变量无关
     */
%>
<h1>================javabean的数据===========</h1>
姓名：${person.name} <br/>
年龄： ${person.age}<br/>
性别：${person.gender} <br/>
生日：${person.birthday} <br/>

<h1>================List的数据===========</h1>
<%
    List<String> list = new ArrayList<>();
    list.add("武汉");
    list.add("广州");
    list.add("青岛");
    list.add("北京");
    pageContext.setAttribute("list",list);
    /*
        el表达式取出List集合的数据格式：
                ${list集合对象[索引值]}
     */
%>

大家暂时不要去的城市： ${list[2]}


<h1>===============数组的数据===========</h1>
<%
    String[] arr = {"狗娃","狗剩","狗蛋","枸杞"};
    pageContext.setAttribute("arr",arr);

     /*
        el表达式取出数组的数据格式：
                ${数组对象[索引值]}
     */
%>

哪个不是人： ${arr[3]}


<h1>===============Map的数据===========</h1>
<%
    Map<String,String> map = new HashMap<>();
    map.put("item1","凤姐");
    map.put("item2","乔碧萝");
    map.put("item3","如花");
    map.put("item4-1","张平");
    pageContext.setAttribute("map",map);
    /*
        el表达式取出map的数据：
                格式1： map.key
                格式2： map['key']
     */
%>


奥巴马的媳妇： ${map.item1} <br/>
你们的媳妇： ${map['item4-1']} <br/>


<%
    //el表达式任何一个对象是否存在某个属性是依赖getXX的方法。


   /* ${pageContext.request.contextPath} 获取项目的根路径，
    底层代码
    ((HttpServletRequest)pageContext.getRequest()).getContextPath();
    */
%>


<a href="${pageContext.request.contextPath}/el/1体验el表达式.jsp">1体验el表达式.jsp</a>



</body>
</html>

```

#### 键名中有特殊字符的写法

```jsp
${作用域变量名["键"]}
```



### 小结

使用EL中从域中取出不同数据类型的写法

| 数据类型 | 写法                            |
| -------- | ------------------------------- |
| JavaBean | ${javabean.属性名}              |
| List     | ${list[索引值]}                 |
| 数组     | ${数组[索引值]}                 |
| Map集合  | \${map.key} <br/>​\${map['key']} |



## 13. EL中各种运算符[重点]

### 目标

学习EL中各种运算符

### 算术运算符

| 算术运算符 | 说明 | 范例       | 结果 |
| ---------- | ---- | ---------- | ---- |
| +          | 加   | ${1+1}     | 2    |
| -          | 减   | ${2-1}     | 1    |
| *          | 乘   | ${1*1}     | 1    |
| /或div     | 除   | ${5 div 2} | 2.5  |
| %或mod     | 取余 | ${5 mod 2} | 1    |

​    

### 比较运算符

| 关系运算符 | 说明                              | 范例      | 结果  |
| ---------- | --------------------------------- | --------- | ----- |
| ==或 eq    | 等于(equal)                       | ${1 eq 1} | true  |
| != 或 ne   | 不等于(not equal)                 | ${1 != 1} | false |
| <    或 lt | 小于(Less than)                   | ${1 lt 2} | true  |
| <=或 le    | 小于等于(Less than or equal)      | ${1 <= 1} | true  |
| >    或 gt | 大于(Greater than)                | ${1 > 2}  | false |
| >=或 ge    | 大于等于(Greater than or   equal) | ${1 >=1}  | true  |

 

### 逻辑运算符 

| 逻辑运算符 | 说明     | 范例                | 结果  |
| ---------- | -------- | ------------------- | ----- |
| && 或 and  | 交集(与) | ${true and false}   | false |
| \|\| 或 or | 并集(或) | ${true \|\| false } | true  |
| ! 或 not   | 非       | ${not true}         | false |

### 三元运算符
```jsp
${条件表达式?真值:假值}
```

### 判空运算符

```
${empty 表达式}
```

### 代码

```jsp
<%@ page import="java.util.List" %>
<%@ page import="java.util.ArrayList" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<%
    int num1=1;
    int num2 =2;
    pageContext.setAttribute("num1",num1);
    pageContext.setAttribute("num2",num2);
%>

    <%--结论：el表达式的内部是可以运算的。--%>
    运算结果：${num1+num2}  <br/>
    <%
        /*
            三目运算符(三元运算符)
            格式：
                   布尔表达式？值1:值2
       */
       int age = 19;
       pageContext.setAttribute("age",age);
    %>

    年龄：${age>=18?"成年人":"未成年人"}  <br/>

    <%
//        empty 判断一个对象是否为空还有集合元素是否为0
        List<String> list =  new ArrayList<>();
        pageContext.setAttribute("list",list);

    %>

    集合是否为空？${list==null}<br/>
    集合是否为空？${empty list}<br/>

</body>
</html>

```

### 小结

  1. empty运算符如何使用？ 

     	- 使用的格式： ${empty 对象}
      - empty的作用
        	- 判断对象是否为null
        	- 判断集合的元素个是否为0

​     



## 14. JSTL的概念和作用

### 目标

1. 什么是JSTL
2. 它的作用是什么

### 什么是JSTL

JSP Standard Tag Language  JSP标准标签库，一组自定义标签。我们直接使用这组元素就可以了，本质上还是运行JAVA代码。

如下几个定制标签库：

1. core 核心标签库：用于页面的流程控制。if  for 
2. sql标签库：在JSP页面上直接访问数据库
3. xml标签库：在JSP页面上解析XML文件
4. fmt标签库：用于国际化和格式化
5. function标签库：16个字符串函数

### JSTL的使用步骤

1. 导包，复制到WEB-INF/lib

   ![1564989251056](assets/1564989251056.png) 

2. 创建JSP页面，在JSP页面上使用以下指令

   ```
   <%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
   ```

3. 在JSP页面上使用标签

   

### 小结

- 使用jstl标库的步骤？

  - 导包
  - 页面是使用taglib指令引入。
  - 页面就可以直接使用标签
  
  



## 15. JSTL的标签：if标签

### 目标

在JSP页面上使用if标签

### 作用

用于一个条件的判断

#### 属性

| 属性名 | 是否支持EL | 属性类型        | 描述                                     |
| ------ | ---------- | --------------- | ---------------------------------------- |
| test   | 取值使用EL | 返回boolean类型 | 如果test的值是true，就执行标签的主体内容 |

### 示例：if标签的案例

​                                                           

#### 代码

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%--
uri ： 表示标签库的引用地址
prefix : 前缀, 前缀名是可以任意的，前缀名代表以后你使用该标签库的标签的时候必须以指定前缀名开头。
        核心库标签的前缀名一般我们都是使用c作为前缀
--%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>Title</title>

</head>
<body>
<%
        //模拟：如果session中存储了一个user，代表该用户已经登陆。
        //session.setAttribute("user","毕展鹏");
/*

        if标签的使用格式：

        <c:if test=布尔表达式>
             布尔表达式为true的时候显示的内容。
        </c:if>
*/

%>
    <c:if test="${user==null}">
         <a href="#">请登陆</a>
    </c:if>

    <c:if test="${user!=null}">
        <div>欢迎：<span style="color: red">${user}</span></div>
    </c:if>
</body>
</html>

```

### 小结

- if标签格式

   - <c:if test=布尔表达式>
    
     布尔表达式为true的情况下显示
    
     \</c:if\>



## 16. JSTL的标签：choose标签

### 目标

学习使用choose,when,otherwise三个标签的使用

### 作用

用于2个以上的判断条件，多分支的语句

#### 属性

| 标签名    | 作用                                                         |
| --------- | ------------------------------------------------------------ |
| choose    | 一个容器，包含下面的2个标签。相当于java中swtich语句          |
| when      | 其中一个判断条件，也有一个属性test指定判断条件，相当于case。when可以出现多个 |
| otherwise | 如果上面所有的条件都不满足，执行这个，相当于default          |





### 案例：通过输入分数，得到评级

#### 需求

​	使用choose标签完成刚刚登陆状态的案例



#### 代码

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%--
uri ： 表示标签库的引用地址
prefix : 前缀, 前缀名是可以任意的，前缀名代表以后你使用该标签库的标签的时候必须以指定前缀名开头。
        核心库标签的前缀名一般我们都是使用c作为前缀
--%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>Title</title>

</head>
<body>
<%
        //需求：如果session中存储了一个user，代表该用户已经登陆。
        session.setAttribute("user","毕展鹏");

/*

        choose标签的适用情况： 存在多种情况下去使用

        choose标签的使用格式：
                <c:choose>
                    <c:when test="条件1">
                        符合条件1显示内容
                    </c:when>
                    <c:when test="条件2">
                        符合条件2显示内容
                    </c:when>
                     <c:when test="条件3">
                        符合条件3显示内容
                    </c:when>
                    ....出现n次when.....
                    <c:otherwise>
                        不符合上述情况执行的代码
                    <c:otherwise>
                </c:choose>
*/

%>

    <c:choose>
        <c:when test="${user==null}">
            <a href="#">请登陆</a>
        </c:when>
        <c:otherwise>
            <div>欢迎：<span style="color: red">${user}</span></div>
        </c:otherwise>
    </c:choose>


</body>
</html>

```

### 小结

1. choose标签的组成结构

   ​	![1602659229547](assets/1602659229547.png)
   
    	
   

## <font color="red">17. JSTL的标签：forEach标签</font>

### 目标

循环遍历标签的使用

### forEach的作用



#### 属性

![1552909020357](assets/1552909020357.png)

#### varStatus属性

![1552909056306](assets/1552909056306.png)

### 案例：遍历集合

#### 需求：

学生信息：包含编号，姓名，性别，成绩。在Servlet中得到所有的学生信息，转发到JSP页面，在JSP页面上使用forEach标签，从List中输出所有的学生信息。

注：页面域pageContext只能用在JSP上，Servlet中不能使用

![1602660332451](assets/1602660332451.png)

 

#### 代码

##### Person.java

```java
package com.itheima.model;

public class Person {

    private String name;

    private int age;

    private String gender;


    public Person() {
    }

    public Person(String name, int age, String gender) {
        this.name = name;
        this.age = age;
        this.gender = gender;
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
     * @return age
     */
    public int getAge() {
        return age;
    }

    /**
     * 设置
     * @param age
     */
    public void setAge(int age) {
        this.age = age;
    }

    /**
     * 获取
     * @return gender
     */
    public String getGender() {
        return gender;
    }

    /**
     * 设置
     * @param gender
     */
    public void setGender(String gender) {
        this.gender = gender;
    }



    public String getBirthday() {
        return "2020-12-12";
    }

    public String toString() {
        return "Person{name = " + name + ", age = " + age + ", gender = " + gender + "}";
    }
}

```

##### JSP代码

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page import="java.util.List" %>
<%@ page import="java.util.ArrayList" %>
<%@ page import="com.itheima.model.Person" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>


</head>
<body>
<%
  /*  foreach标签的格式：
        <c:foreach items="要遍历的目标" var="遍历出来的元素存储在域中的名字">

        </c:foreach>

   foreach标签常用属性：
        items: 要遍历目标对象
        var:  遍历出来的每一个元素存储在域的名字
        varStatus: foreach标签在遍历的过程中会产生大量的遍历信息，比如： index（元素的索引值）,count(序号)..这时候
        foreach标签把这些遍历的信息封装到一个对象中 new Status(index,count). varStatus就只该遍历的迭代信息对象存储在
        域的名字
        begin: 开始遍历的索引值
        end: 结束遍历的索引值
        step: 遍历的步长 默认的步长是1


   */

    List<Person> list = new ArrayList<>();
    list.add(new Person("狗娃",12,"女"));
    list.add(new Person("狗剩",19,"男"));
    list.add(new Person("铁蛋",20,"女"));
    list.add(new Person("黑娃",18,"男"));
    pageContext.setAttribute("list",list);



%>


<table border="1px" align="center" width="400px" >
    <tr>
        <th>序号</th>
        <th>姓名</th>
        <th>年龄</th>
        <th>性别</th>
    </tr>

    <c:forEach items="${list}" var="person" step="2" varStatus="status">
        <tr>
            <td>${status.count}</td>
            <td>${person.name}</td>
            <td>${person.age}</td>
            <td>${person.gender}</td>
        </tr>
    </c:forEach>

</table>


</body>
</html>

```

### 小结

1. forEach标签的作用是：

2. 以下两个属性的作用是： 
   
   



## <font color="red">18. 开发模式</font>

### 目标

什么是MVC，并且使用MVC开发自己项目

### 什么是开发模式

开发模式就是前人在开发的过程中积累的经验，这些经验非常合适我们管理项目。 提高项目可维护性.



### 概念



####  **JSP Model1 第一代**

JSP Model1是JavaWeb早期的模型，它适合小型Web项目，开发成本低！Model1第一代时期，服务器端只有JSP页面，所有的操作都在JSP页面中，连访问数据库的API也在JSP页面中完成。也就是说，所有的东西都在一起，对后期的维护和扩展极为不利。

![1602661617203](assets/1602661617203.png)

==缺点： 所有代码都落入到jsp页面上，回到后期维护成本非常高。==

####  **JSP Model2(MVC) 第二代**

JSP Model1第二代有所改进，把业务逻辑的内容放到了JavaBean中，而JSP页面只负责显示，servlet负责处理请求与调度的工作。虽然第二代比第一代好了些，但还让JavaBean做了过多的工作，java Bean依然要封装数据、处理业务和操作数据库等混合在一起了。 

JSP Model 2 Model2使用到的技术有：Servlet、JSP、JavaBean。Model2 是MVC设计模式在Java语言的具体体现。l JSP：视图层，用来与用户打交道。负责接收用来的数据，以及显示数据给用户；l Servlet：控制层，负责找到合适的模型对象来处理业务逻辑，转发到合适的视图；l JavaBean：模型层，完成具体的业务工作，例如：转账等。 

![1602662054813](assets/1602662054813.png)

==缺点： javabean与Servlet的任务还是比较繁重。==



#### **MVC+三层架构**

JSP模式是理论基础，但实际开发中，我们常将服务器端程序，根据逻辑进行分层。一般比较常见的是分三层，我们称为：经典三层体系架构。三层分别是：表示层、业务逻辑层、数据访问层。l 表示层：又称为 web层，与浏览器进行数据交互的。l 业务逻辑层：又称为service层，专门用于处理业务数据的。l 数据访问层：又称为dao层，与数据库进行数据交换的。将数据库的一条记录与JavaBean进行对应。



![1602662644520](./assets/1602662644520.png)

### 小结

MVC是哪三个单词缩写：

- model
- view
- controller



##  19. 案例part1：项目结构的搭建、数据访问层、业务层

### 目标

1. 搭建MVC和三层架构
2. 编写数据访问层和业务层

### 案例需求

使用三层架构和MVC模式开发代码，完成用户显示列表功能

### 案例效果

![1552910157095](assets/1552910157095.png)

### 案例分析

![1602663167886](assets/1602663167886.png)

### 实现步骤

#### SQL语句

```mysql
create table contact( 
   id int primary key auto_increment , 
   name varchar(20) not null , 
   sex char(1) , 
   age int(3) unsigned ,   -- 无符号
   address varchar(10) ,   -- 籍贯
   qq varchar(18) , 
   email varchar(25) 
 ) ;
 
-- 插入记录
insert  into contact(name,sex,age,address,qq,email) values 
('猪八戒','男',25,'广东','834523234','zhuzhuxia@itcast.cn'),
('貂蝉','女',18,'湖南','59869834','diaochan@qq.com'),
('孙悟空','男',28,'湖南','87967822','wukong@itheima.com'),
('周瑜','男',25,'广西','2743759345','zhou@163.com');

select * from contact;
```

#### 导入jar包

mybatis

jstl标签

log4j 

mysql驱动





![1564994330947](assets/1564994330947.png) 

#### sqlMapConfig.xml配置文件

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

#### db.properties

```properties
jdbc.url=jdbc:mysql:///db1
jdbc.username=root
jdbc.password=root
jdbc.driver=com.mysql.jdbc.Driver
```







#### 实体类

```java
package com.itheima.model;

public class Contact {

    private int id;

    private String name;

    private String sex;

    private int age;

    private String address;

    private String qq;

    private String email;


    public Contact() {
    }

    public Contact(int id, String name, String sex, int age, String address, String qq, String email) {
        this.id = id;
        this.name = name;
        this.sex = sex;
        this.age = age;
        this.address = address;
        this.qq = qq;
        this.email = email;
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
     * @return age
     */
    public int getAge() {
        return age;
    }

    /**
     * 设置
     * @param age
     */
    public void setAge(int age) {
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

    /**
     * 获取
     * @return qq
     */
    public String getQq() {
        return qq;
    }

    /**
     * 设置
     * @param qq
     */
    public void setQq(String qq) {
        this.qq = qq;
    }

    /**
     * 获取
     * @return email
     */
    public String getEmail() {
        return email;
    }

    /**
     * 设置
     * @param email
     */
    public void setEmail(String email) {
        this.email = email;
    }

    public String toString() {
        return "Contact{id = " + id + ", name = " + name + ", sex = " + sex + ", age = " + age + ", address = " + address + ", qq = " + qq + ", email = " + email + "}";
    }
}

```

#### SqlSessionUtils工具类

```java
package com.itheima.utils;

import org.apache.ibatis.session.SqlSession;
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
        InputStream inputStream = SqlSessionUtils.class.getResourceAsStream("/sqlMapConfig.xml");
         factory = builder.build(inputStream);
    }

    public static SqlSessionFactory getFactory() {
        return factory;
    }

    public static void setFactory(SqlSessionFactory factory) {
        SqlSessionUtils.factory = factory;
    }


    //提供sqlSession
    public static SqlSession getSqlSession(){
        SqlSession sqlSession = factory.openSession();
        return sqlSession;
    }


    //关闭SqlSession
    public static void close(SqlSession sqlSession){
        //如果是增删改操作是需要提交事务
        sqlSession.commit();
        //关闭
        sqlSession.close();
    }


}
```



### 按照三层架构的方式分包



![1602663923381](assets/1602663923381.png)





### 导入静态页面



![1602664027167](assets/1602664027167.png)

## 20. part2： 显示所有联系人

### 流程图

![1602664795871](assets/1602664795871.png)



### ContactDao代码

```java
package com.itheima.dao;

import com.itheima.model.Contact;
import org.apache.ibatis.annotations.Select;

import java.util.List;

public interface ContactDao {

    //查询所有的联系人
    List<Contact> findAll();
}

```

### ContactDao.xml文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace名称空间，名称空间代表该xml文件映射是哪个接口-->
<mapper namespace="com.itheima.dao.ContactDao">

    <select id="findAll" resultType="contact">
        select * from contact
    </select>

</mapper>
```





### ContactService代码

```java
package com.itheima.service;

import com.itheima.dao.ContactDao;
import com.itheima.model.Contact;
import com.itheima.utils.SqlSessionUtils;
import org.apache.ibatis.session.SqlSession;

import java.util.List;

public class ContactService {

    //查询所有的联系人
    public List<Contact> findAll(){
        SqlSession sqlSession = SqlSessionUtils.getSqlSession();
        ContactDao contactDao = sqlSession.getMapper(ContactDao.class);
        List<Contact> list = contactDao.findAll();
        SqlSessionUtils.close(sqlSession);
        return list;
    }
}

```

### ContactServlet

```java
package com.itheima.web;

import com.itheima.model.Contact;
import com.itheima.service.ContactService;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;

@WebServlet("/contactServlet")
public class ContactServlet extends HttpServlet {

    private ContactService contactService = new ContactService();

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");

        //1.得到有的联系人
        List<Contact> list = contactService.findAll();

        //2. 把联系人存储到域中
        request.setAttribute("list",list);

        //3. 请求转发到list.jsp
        request.getRequestDispatcher("/list.jsp").forward(request,response);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

### list.jsp页面修改

```java
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
<!-- 网页使用的语言 -->
<html lang="zh-CN">
<head>
    <!-- 指定字符集 -->
    <meta charset="utf-8">
    <!-- 使用Edge最新的浏览器的渲染方式 -->
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <!-- viewport视口：网页可以根据设置的宽度自动进行适配，在浏览器的内部虚拟一个容器，容器的宽度与设备的宽度相同。
    width: 默认宽度与设备的宽度相同
    initial-scale: 初始的缩放比，为1:1 -->
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- 上述3个meta标签*必须*放在最前面，任何其他内容都*必须*跟随其后！ -->
    <title>Bootstrap模板</title>

    <!-- 1. 导入CSS的全局样式 -->
    <link href="css/bootstrap.min.css" rel="stylesheet">
    <!-- 2. jQuery导入，建议使用1.9以上的版本 -->
    <script src="js/jquery-2.1.0.min.js"></script>
    <!-- 3. 导入bootstrap的js文件 -->
    <script src="js/bootstrap.min.js"></script>
    <style type="text/css">
        td, th {
            text-align: center;
        }
    </style>
</head>
<body>
<div class="container">
    <h3 style="text-align: center">显示所有联系人</h3>
    <table border="1" class="table table-bordered table-hover">
        <tr class="success">
            <th>编号</th>
            <th>姓名</th>
            <th>性别</th>
            <th>年龄</th>
            <th>籍贯</th>
            <th>QQ</th>
            <th>邮箱</th>
            <th>操作</th>
        </tr>

       <c:forEach items="${list}" var="contact">
            <tr>
                <td>${contact.id}</td>
                <td>${contact.name}</td>
                <td>${contact.sex}</td>
                <td>${contact.age}</td>
                <td>${contact.address}</td>
                <td>${contact.qq}</td>
                <td>${contact.email}</td>
                <td><a class="btn btn-default btn-sm" href="修改联系人.html">修改</a>&nbsp;<a class="btn btn-default btn-sm" href="修改联系人.html">删除</a></td>
            </tr>
       </c:forEach>

        <tr>
            <td colspan="8" align="center"><a class="btn btn-primary" href="添加联系人.html">添加联系人</a></td>
        </tr>
    </table>
</div>
</body>
</html>

```

==注意：访问的时候一定要先访问servlet，由servlet产生数据然后转发到list.jsp页面上，千万别直接访问list.jsp==


