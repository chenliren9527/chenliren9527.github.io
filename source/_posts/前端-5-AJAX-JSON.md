---
title: 前端(5)-AJAX&JSON
tags:
  - 笔记
  - 前端
  - AJAX
  - JSON
  - jQuery
  - 跨域
  - json格式字符串的转换
  - json转换工具Jackson
categories:
  - 前端
date: 2020-10-20 13:23:23
---

## 01.复习-学习目标

### 学习目标

1. 能够理解异步（ajax）的概念

2. 能够了解原生js的ajax

3. 能够使用jQuery3.0的$.get()新增签名进行访问

4. 能够使用jQuery3.0的$.post()新增签名进行访问

5. 能够使用json转换工具Jackson进行json格式字符串的转换

6. <font color=red>能够完成用户名是否存在的查重案例</font>

7. <font color=red>能够完成自动补全的案例</font>




## 02.ajax介绍与应用场景【理解】

### 目标

理解ajax异步概念与应用场景



### ajax(阿贾克斯)介绍

Asynchronous JavaScript And XML 异步的js和xml实现请求与处理（异步请求）

异步使用的技术：==js== 和 xml

xml作为以前的异步请求传输数据格式，由于体积过大，被==json==代替  

> ==ajax又叫异步处理请求， 又叫无刷新处理请求  ，又叫页面局部刷新处理请求==
>
> 何为异步？  
>
> ​      同步请求：浏览器通过主线程发送有阻塞的请求， 浏览器需要死等服务器的响应
>
> ​      异步请求：是js发送异步请求，通过创建子线程发送请求，没有阻塞浏览器的主线程



### 同步与异步详细处理过程

#### 同步方式发送请求过程

![1603154979186](AJAX-JSON.assets/1603154979186.png)

==同步请求的特点： 服务器没有响应之前浏览器会卡死不动==

客户端发请求者

> 浏览器主线程发送同步阻塞的请求， 此时浏览器处于页面空白等待状态

服务器

> 服务器处理同步请求，会响应一个完整页面给前端

客户端接收响应者

> 浏览器会接收一个完整页面进行显示， 也叫刷新了页面

特点：同步请求也叫整个页面刷新的请求

#### 异步方式发送请求过程

![1603155258448](AJAX-JSON.assets/1603155258448.png)

客户端发请求者

> js发送异步请求，异步请求会创建子线程发送，这样就不会阻塞浏览器的主线程

服务器

> 处理异步请求，响应数据给js

客户端接收响应者

> js接收数据，通过DOM更新页面局部位置数据

异步：请求一个页面局部位置的数据

### 异步无刷新AJAX应用场景

1. 异步检验用户名

   ![1546995371381](AJAX-JSON.assets/1546995371381.png) 

2. 内容自动补全

   ![image-20200717113652591](AJAX-JSON.assets/image-20200717113652591.png) 

### 小结

* **什么是ajax？**

  ajax是异步交互技术， 也是被称作为局部刷新技术

* **ajax应用场景?** 

  * 内容补全
  * 用户名校验

  



## 03.js原生ajax：后端处理【了解 】

### 目标

使用js原生实现ajax异步请求的后端实现



### 需求

实现用户注册功能异步校验用户名是否存在

### 效果

![1550220360024](AJAX-JSON.assets/1550220360024.png)





### 环境搭建

创建项目 

js_ajax.html注册页面素材

![1602816339424](AJAX-JSON.assets/1602816339424.png) 

````html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<p>
    用户名:<input type="text" id="username" name="username"><span id="info"></span>
</p>
<script type="text/javascript" src="js/jquery-3.3.1.min.js"></script>
<script type="text/javascript">
    //目标：提交异步请求验证用户名是否已被注册
  

</script>
</body>
</html>
````

导入jquery

![1602816401289](AJAX-JSON.assets/1602816401289.png) 

导入过滤器处理乱码

![1602816456343](AJAX-JSON.assets/1602816456343.png)&nbsp;

### 分析实现

![image-20200815100148406](AJAX-JSON.assets/image-20200815100148406.png)

> //目标：处理校验用户名异步请求
> //1.获取请求数据用户名
> //2.校验用户名是否存在
> //3.存在,返回“已被注册”
> //3.不存在，返回“可以使用”

### CheckUserNameServlet代码

```java
package com.itheima.web;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/checkUserName")
public class CheckUserNameServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        //1. 获取用户名
        String username = request.getParameter("username");
        //2 。校验用户名
        if("admin".equals(username)){
            out.write("已被注册");
        }else{
            out.write("可以使用");
        }


    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```



### 小结

异步请求后端返回什么数据给前端?

> 字符串格式的数据

## 04.js原生ajax：前端处理【理解】

### 目标

能够说出js原生实现ajax的过程(面试题)



### 发送异步请求语法

原生api方法，原生异步核心js对象XMLHttpRequest

![1546996175109](AJAX-JSON.assets/1546996175109.png)

### ajax连接状态

                // readyState存有ajax连接状态，从 0 到 4 发生变化。
                // 0: 请求未初始化
                // 1: 服务器连接已建立
                // 2: 请求已接收
                // 3: 请求处理中
                // 4: 请求已完成，且响应已就绪

![1556551658775](AJAX-JSON.assets/1556551658775.png)

实现步骤

![image-20200815100915863](AJAX-JSON.assets/image-20200815100915863.png)

### 01.js_ajax.html实现代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="/js/jquery-3.3.1.min.js"></script>
    <script>

        $(function(){
            //1. 当前页面加载完毕的时候就给username注册一个失去焦点的事件
            $("#username").blur(function(){
                var username = $(this).val();
                if(username) { //字符串的时候空串为false，有内容为true
                    //2. 获取username输入的值, 创建ajax 的引擎
                    var xmlHttpRequest = new XMLHttpRequest();
                    //3. 设置请求的参数
                    /*  open的方法有两个参数：
                        参数一： 请求的方式
                        参数二： 请求地址*/
                    xmlHttpRequest.open("GET", "/checkUserName?username="+username);

                    //4. 发送请求
                    xmlHttpRequest.send();

                    //5. 使用ajax引擎注册一个监听事件，监听服务器的状态。
                    xmlHttpRequest.onreadystatechange=function(){ //浏览器与服务器的状态发生改变的时候都会触发该方法
                        if(xmlHttpRequest.readyState==4 && xmlHttpRequest.status==200){
                                //xmlHttpRequest.readyState==4 代表服务器已经响应完毕给ajax引擎，   xmlHttpRequest.status==200 代表是状态码是200
                               if(xmlHttpRequest.responseText=="已被注册"){
                                   $("#info").css("color","red");
                               }else{
                                   $("#info").css("color","green");
                               }
                               $("#info").html(xmlHttpRequest.responseText); // xmlHttpRequest.responseText 就是服务器响应回来的文本数据
                        }

                    }
                }
            });
        });



    </script>
</head>
<body>
    用户名：<input type="text" id="username"/><span id="info"></span>
</body>
</html>
```

部署项目设置

![image-20200719095814257](AJAX-JSON.assets/image-20200719095814257.png)

### 小结

**说出js原生实现ajax的过程(面试题)**

- 创建ajax的引擎
- 设置请求参数（请求方式+请求地址）
- 发送请求
- 监听服务器的状态

## 05.JQ实现AJAX：get方法【重点】

### 目标

理解全局对象get方法实现异步请求



### jq五种常用ajax方法语法

![image-20200927095737762](AJAX-JSON.assets/image-20200927095737762.png)

### 实现代码

![1602823982448](AJAX-JSON.assets/1602823982448.png)&nbsp;

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="/js/jquery-3.3.1.min.js"></script>
    <script>


            /*
            jquery发出get请求
                格式：
                     $.get(url, [data], [callback], [type])
                     url : 请求地址
                     data: 请求的参数
                        格式1："参数名1=参数值1&参数名2=参数2.." （不推荐）
                        json格式 : {"参数名1":值1,"参数名2"：值2} (推荐)
                    callback：会调用函数，服务器如果成功响应回来则调用该函数
                    type: 服务器返回的数据的格式类型， text(文本，默认)，json()
            */


        /*

            方式一： 参数是拼接字符串的方式

          //1. 给username注册失去焦点事件
        $(function(){
            $("#username").blur(function(){
                //2. 获取username的值
                var username = $(this).val();
                //3. 判断输入的值是否空，如果不为空则发出请求
                if(username){
                    //4. 发出请求
                    $.get("/checkUserName","username="+username,function(responseText){
                        if(responseText=="已被注册"){
                            $("#info").css("color","red");
                        }else{
                            $("#info").css("color","green");
                        }
                        $("#info").html(responseText);
                    });
                }
            })
        });*/




            // 方式二： Es6标准的拼接字符串的新语法
            /*
                1. 变量与字符串拼接新语法格式： `字符串${变量}`
                2. let可以声明变量， let与var区别： let语法更加严谨，比如： let声明变量{}有有限制作为范围的作用， 2。 let声明的变量可以重复声明变量

            $(function(){
                $("#username").blur(function(){
                    //2. 获取username的值
                    let username = $(this).val();
                    //3. 判断输入的值是否空，如果不为空则发出请求
                    if(username){
                        //4. 发出请求
                        $.get("/checkUserName",`username=${username}`,function(responseText){
                            if(responseText=="已被注册"){
                                $("#info").css("color","red");
                            }else{
                                $("#info").css("color","green");
                            }
                            $("#info").html(responseText);
                        });
                    }
                })
            }); */



            //格式三： 发送的数据使用json格式。
            $(function(){
                $("#username").blur(function(){
                    //2. 获取username的值
                    let username = $(this).val();
                    //3. 判断输入的值是否空，如果不为空则发出请求
                    if(username){
                        //4. 发出请求
                        $.get("/checkUserName",{"username":username},function(responseText){
                            if(responseText=="已被注册"){
                                $("#info").css("color","red");
                            }else{
                                $("#info").css("color","green");
                            }
                            $("#info").html(responseText);
                        });
                    }
                })
            });




    </script>
</head>
<body>
     用户名：<input type="text" id="username"/><span id="info"></span>
</body>
</html>
```

### 小结

* get异步请求的参数有几个?哪一个参数是必须的?

  >  url : 发送地址
  >
  >  data : 发送的数据
  >
  >  function() : 会调用函数
  >
  >  type: 服务器响应回来的数据格式

* 异步请求传递请求参数数据格式有几种?分别是什么?

  > "参数名=参数值1&参数名2=参数值2"
  >
  > {“key”：值}

  

## 06.调试运行【应用】

### 目标

掌握异步请求前端与后端调试代码操作



### 前端调试

1. 使用浏览器的开发者工具

2. 设置断点

3. 每一行运行

   ![1551924776740](AJAX-JSON.assets/1551924776740.png)

### 后端调试

1. 设置断点

2. 每一行运行

   ![image-20191231101536022](AJAX-JSON.assets/image-20191231101536022.png)



### 小结

- 后端与前端的代码在哪里调试?

  > 前端：浏览器
  >
  > 后端：idea

- 如何知道前端发送异步请求成功到后端了？

  > 进入后端第一句代码调试

- 如何知道服务器处理是否成功？

  > 回到前端的回调函数



## 07.JQ实现AJAX：post方法【重点】

### 目标

理解全局对象post方法发送异步请求



### 实现步骤

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="/js/jquery-3.3.1.min.js"></script>
    <script>


            /*
            jquery发出post请求
                格式：
                     $.post(url, [data], [callback], [type])
                     url : 请求地址
                     data: 请求的参数
                        格式1："参数名1=参数值1&参数名2=参数2.." （不推荐）
                        json格式 : {"参数名1":值1,"参数名2"：值2} (推荐)
                    callback：会调用函数，服务器如果成功响应回来则调用该函数
                    type: 服务器返回的数据的格式类型， text(文本，默认)，json()
            */


            //格式三： 发送的数据使用json格式。
            $(function(){
                $("#username").blur(function(){
                    //2. 获取username的值
                    let username = $(this).val();
                    //3. 判断输入的值是否空，如果不为空则发出请求
                    if(username){
                        //4. 发出请求
                        $.post("/checkUserName",{"username":username},function(responseText){
                            if(responseText=="已被注册"){
                                $("#info").css("color","red");
                            }else{
                                $("#info").css("color","green");
                            }
                            $("#info").html(responseText);
                        });
                    }
                })
            });




    </script>
</head>
<body>
     用户名：<input type="text" id="username"/><span id="info"></span>
</body>
</html>
```



### 小结

post方法提交异步请求语法

> $.post(url,data,fucntion(),type)

post提交异步请求比get好在哪里

> 发送的数据更加安全，而且没有大小限制



## 08.JQ实现AJAX：ajax方法【重点】

### 目标

使用ajax方法实现异步请求

### 需求

实现异步登录

### 效果

![1560567283190](AJAX-JSON.assets/1560567283190.png)

![1550284875130](AJAX-JSON.assets/1550284875130.png) 

页面素材

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>用户登录</title>
    <script type="text/javascript" src="js/jquery-3.3.1.min.js"></script>
</head>
<body>
<h2>用户登录</h2>
<form action="login" method="post" id="loginForm">
    <table>
        <tr>
            <td>用户名</td>
            <td><input type="text" name="username" id="username" placeholder="请输入用户名"/></td>
        </tr>
        <tr>
            <td>密码</td>
            <td><input type="password" name="password" id="password" placeholder="请输入密码"/></td>
        </tr>
        <tr>
            <td colspan="2"><input type="button" value="登录" id="btnLogin"/></td>
        </tr>
    </table>
</form>
<script type="text/javascript">
   

</script>
</body>
</html>
```

### ajax方法语法

```html
语法：$.ajax(json对象)  --api文档写法$.ajax([settings])
json对象常用属性如下：
	url	    服务器访问地址
	async	设置后台发送类型，是异步或同步发送。通常省略
			默认值：true，异步发送请求
	data	发送给服务器的数据的格式
    		格式1：键=值
			格式2：{键:值} 里面可以有多个键值对
	method	发送请求的方式：GET或POST，早期的版本这个属性叫:type
    		默认的请求方式，GET方式
	dataType	服务器返回的数据类型
				取值可以是 xml, html, script, json, text, _default等
	success	服务器响应成功以后调用的回调函数
			回调函数的参数是服务器返回的数据
	error	如果服务器出现异常调用这个函数
```



### 实现代码

##### LoginServlet代码

![image-20200815112021118](AJAX-JSON.assets/image-20200815112021118.png)&nbsp;

```java
package com.itheima.web;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/loginServlet")
public class LoginServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        if("admin".equals(username) && "123".equals(password)){
            out.write("true");
        }else{
            out.write("false");
        }
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

##### 04.jq_ajax.html

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>用户登录</title>
    <script type="text/javascript" src="js/jquery-3.3.1.min.js"></script>
</head>
<body>
<h2>用户登录</h2>
<form action="login" method="post" id="loginForm">
    <table>
        <tr>
            <td>用户名</td>
            <td><input type="text" name="username" id="username" placeholder="请输入用户名"/></td>
        </tr>
        <tr>
            <td>密码</td>
            <td><input type="password" name="password" id="password" placeholder="请输入密码"/></td>
        </tr>
        <tr>
            <td colspan="2"><input type="button" value="登录" id="btnLogin"/></td>
        </tr>
    </table>
</form>
<script type="text/javascript">
/*

    语法：$.ajax(json对象)  --api文档写法$.ajax({json格式数据})

    json对象常用属性如下：
        url	    服务器访问地址
        async	设置后台发送类型，是异步或同步发送。通常省略 默认值：true，异步发送请求
        data	发送给服务器的数据的格式
                格式1：键=值
                格式2：{键:值} 里面可以有多个键值对
        method	发送请求的方式：GET或POST，早期的版本这个属性叫:type  默认的请求方式，GET方式
        dataType	服务器返回的数据类型 取值可以是 xml, html, script, json, text, _default等
        success	服务器响应成功以后调用的回调函数,  回调函数的参数是服务器返回的数据
        error	如果服务器出现异常调用这个函数
*/
    $("#btnLogin").click(function(){
        //1. 获取用户名与密码
        let username = $("#username").val();
        let password = $("#password").val();

        //2. 如果用户名与密码不为空，发送ajax请求
        if(username && password){
            $.ajax({
                url:"/loginServlet",  //请求地址
                data:`username=${username}&password=${password}`, //请求的参数
                method:"post",  //请求的方式
                success:function(info){  //成功的回调函数
                    if(info=="true"){
                        alert(`${username}登陆成功`);
                    }else{
                        alert("登陆失败");
                    }
                }
            });

        }else{
            //3. 如果用户名或者密码为空，提示用户不能为空
            alert("用户名或者密码不能为空");
        }






    });



</script>
</body>
</html>
```

运行效果与目标效果一致

### 同步与异步测试

效果图： 

![1603162010596](AJAX-JSON.assets/1603162010596.png)



同步：会阻塞主线程

异步：不会阻塞主线程， 会创建单独的子线程运行

**模拟服务器响应时间较长：（让servlet睡眠10秒钟）**

```java
package com.itheima.web;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/loginServlet")
public class LoginServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        if("admin".equals(username) && "123".equals(password)){
            out.write("true");
        }else{
            out.write("false");
        }
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```





异步使用测试

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>用户登录</title>
    <script type="text/javascript" src="js/jquery-3.3.1.min.js"></script>
</head>
<body>
<h2>用户登录</h2>
<form action="login" method="post" id="loginForm">
    <table>
        <tr>
            <td>用户名</td>
            <td><input type="text" name="username" id="username" placeholder="请输入用户名"/></td>
        </tr>
        <tr>
            <td>密码</td>
            <td><input type="password" name="password" id="password" placeholder="请输入密码"/></td>
        </tr>
        <tr>
            <td colspan="2"><input type="button" value="登录" id="btnLogin"/></td>
        </tr>
    </table>
</form>
    <input type="button" onclick="alert('哎呀被点了')" value="按钮"/>
<script type="text/javascript">
/*

    语法：$.ajax(json对象)  --api文档写法$.ajax({json格式数据})

    json对象常用属性如下：
        url	    服务器访问地址
        async	设置后台发送类型，是异步或同步发送。通常省略 默认值：true，异步发送请求
        data	发送给服务器的数据的格式
                格式1：键=值
                格式2：{键:值} 里面可以有多个键值对
        method	发送请求的方式：GET或POST，早期的版本这个属性叫:type  默认的请求方式，GET方式
        dataType	服务器返回的数据类型 取值可以是 xml, html, script, json, text, _default等
        success	服务器响应成功以后调用的回调函数,  回调函数的参数是服务器返回的数据
        error	如果服务器出现异常调用这个函数
*/
    $("#btnLogin").click(function(){
        //1. 获取用户名与密码
        let username = $("#username").val();
        let password = $("#password").val();

        //2. 如果用户名与密码不为空，发送ajax请求
        if(username && password){
            $.ajax({
                url:"/loginServlet",  //请求地址
                async:false, //默认值true，默认是发异步的请求，false为同步请求
                data:`username=${username}&password=${password}`, //请求的参数
                method:"post",  //请求的方式
                success:function(info){  //成功的回调函数
                    if(info=="true"){
                        alert(`${username}登陆成功`);
                    }else{
                        alert("登陆失败");
                    }
                },
                error:function(){ //服务器异常调用的函数
                    alert("服务器压力过大，请稍后");
                }
            });

        }else{
            //3. 如果用户名或者密码为空，提示用户不能为空
            alert("用户名或者密码不能为空");
        }






    });



</script>
</body>
</html>
```



### 演示错误回调函数弹出友好信息

修改服务器端代码，让其发生异常

![1603162248836](AJAX-JSON.assets/1603162248836.png)

重新部署运行

![image-20200719114349227](AJAX-JSON.assets/image-20200719114349227.png)



### 小结

ajax方法使用语法

> $.ajax({
>
> url: xx,     请求地址
>
> data:xx,    请求参数 
>
> method:xx,   请求方式
>
> async: xx， 是否为异步
>
> success:xx,  请求成功回调函数
>
> error:xx   请求失败的回调函数
>
> })

ajax方法的好处

> 1.可以设置是否异步请求
>
> 2.有错误回调函数



## 09.JQ3.0实现AJAX：签名方法【应用】

### 目标

掌握使用jq3.0新增post签名方法发送异步请求



### get签名方法方式

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>用户登录</title>
    <script type="text/javascript" src="js/jquery-3.3.1.min.js"></script>
</head>
<body>
<h2>用户登录</h2>
<form action="login" method="post" id="loginForm">
    <table>
        <tr>
            <td>用户名</td>
            <td><input type="text" name="username" id="username" placeholder="请输入用户名"/></td>
        </tr>
        <tr>
            <td>密码</td>
            <td><input type="password" name="password" id="password" placeholder="请输入密码"/></td>
        </tr>
        <tr>
            <td colspan="2"><input type="button" value="登录" id="btnLogin"/></td>
        </tr>
    </table>
</form>
    <input type="button" onclick="alert('哎呀被点了')" value="按钮"/>
<script type="text/javascript">
/*

    语法：$.ajax(json对象)  --api文档写法$.ajax({json格式数据})

    json对象常用属性如下：
        url	    服务器访问地址
        async	设置后台发送类型，是异步或同步发送。通常省略 默认值：true，异步发送请求
        data	发送给服务器的数据的格式
                格式1：键=值
                格式2：{键:值} 里面可以有多个键值对
        method	发送请求的方式：GET或POST，早期的版本这个属性叫:type  默认的请求方式，GET方式
        dataType	服务器返回的数据类型 取值可以是 xml, html, script, json, text, _default等
        success	服务器响应成功以后调用的回调函数,  回调函数的参数是服务器返回的数据
        error	如果服务器出现异常调用这个函数
*/
    $("#btnLogin").click(function(){
        //1. 获取用户名与密码
        let username = $("#username").val();
        let password = $("#password").val();

        //2. 如果用户名与密码不为空，发送ajax请求
        if(username && password){
            $.get({
                url:"/loginServlet",  //请求地址
                async:false, //默认值true，默认是发异步的请求，false为同步请求
                data:`username=${username}&password=${password}`, //请求的参数
                method:"post",  //请求的方式
                success:function(info){  //成功的回调函数
                    if(info=="true"){
                        alert(`${username}登陆成功`);
                    }else{
                        alert("登陆失败");
                    }
                },
                error:function(){ //服务器异常调用的函数
                    alert("服务器压力过大，请稍后");
                }
            });

        }else{
            //3. 如果用户名或者密码为空，提示用户不能为空
            alert("用户名或者密码不能为空");
        }






    });



</script>
</body>
</html>
```

### post签名方法方式

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>用户登录</title>
    <script type="text/javascript" src="js/jquery-3.3.1.min.js"></script>
</head>
<body>
<h2>用户登录</h2>
<form action="login" method="post" id="loginForm">
    <table>
        <tr>
            <td>用户名</td>
            <td><input type="text" name="username" id="username" placeholder="请输入用户名"/></td>
        </tr>
        <tr>
            <td>密码</td>
            <td><input type="password" name="password" id="password" placeholder="请输入密码"/></td>
        </tr>
        <tr>
            <td colspan="2"><input type="button" value="登录" id="btnLogin"/></td>
        </tr>
    </table>
</form>
    <input type="button" onclick="alert('哎呀被点了')" value="按钮"/>
<script type="text/javascript">
/*

    语法：$.post(json对象)  --api文档写法$.ajax({json格式数据})

    json对象常用属性如下：
        url	    服务器访问地址
        async	设置后台发送类型，是异步或同步发送。通常省略 默认值：true，异步发送请求
        data	发送给服务器的数据的格式
                格式1：键=值
                格式2：{键:值} 里面可以有多个键值对
        method	发送请求的方式：GET或POST，早期的版本这个属性叫:type  默认的请求方式，GET方式
        dataType	服务器返回的数据类型 取值可以是 xml, html, script, json, text, _default等
        success	服务器响应成功以后调用的回调函数,  回调函数的参数是服务器返回的数据
        error	如果服务器出现异常调用这个函数
*/
    $("#btnLogin").click(function(){
        //1. 获取用户名与密码
        let username = $("#username").val();
        let password = $("#password").val();

        //2. 如果用户名与密码不为空，发送ajax请求
        if(username && password){
            $.post({
                url:"/loginServlet",  //请求地址
                async:false, //默认值true，默认是发异步的请求，false为同步请求
                data:`username=${username}&password=${password}`, //请求的参数
                success:function(info){  //成功的回调函数
                    if(info=="true"){
                        alert(`${username}登陆成功`);
                    }else{
                        alert("登陆失败");
                    }
                },
                error:function(){ //服务器异常调用的函数
                    alert("服务器压力过大，请稍后");
                }
            });

        }else{
            //3. 如果用户名或者密码为空，提示用户不能为空
            alert("用户名或者密码不能为空");
        }






    });



</script>
</body>
</html>
```

### 小结

jq3.0新增get/post签名方法的使用语法

> $.get||post({
>
> url: xx,     请求地址
>
> data:xx,    请求参数 
>
> method:xx,   请求方式
>
> async: xx， 是否为异步
>
> success:xx,  请求成功回调函数
>
> error:xx   请求失败的回调函数
>
> })



## 10.JSON格式：介绍和对象【应用】

### 目标

1、理解JSON数据格式

2、掌握JSON对象数据解析



### 什么是JSON

JavaScript Object Notation   js对象标记

==json对象是符合指定格式的字符串对象==

这种格式与js的对象格式一模一样， 所以js技术可以很好解析json对象数据


### 轻量级

```
传输数据的体积小， 与xml相比
```



### 数据交换

就是不同软件之间的传递数据就是数据交换， 或者是不同系统之间传递数据

<img src="../../../../Documents/Java笔记/就业班/3.前端/day34(Ajax&json)/笔记/assets/image-20200717150444041.png" width="60%"> 



### JSON对象格式

![image-20191230223359629](AJAX-JSON.assets/image-20191230223359629.png)

### js访问json格式的数据格式示例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script>

        /*
             单个对象是 : {}
             数组或者集合： []
            json格式：{"key1":value1,"key2":value2..}
            json访问属性格式： json对象.属性名(key)
            json访问数组||集合格式： json对象[索引值]

         */

        //1. 普通单个java对象
        /*
            Person p = new Person("狗娃",12);
         */
        var person = {"name":"狗娃","age":12};
        //alert("姓名："+ person.name +" 年龄："+ person.age);


        //2. java对象在集合中
        /*
            List list = new ArrayList();
            list.add(new Person("狗娃",12));
            list.add(new Person("狗剩",18));
         */
        var personList =[{"name":"狗娃","age":12},{"name":"狗剩","age":18}];
        alert("访问2号员工，姓名："+personList[1].name  +" 年龄："+ personList[1].age);



        //3. 单个对象的某个属性是集合对象。
        /*
            List list = new ArrayList();
            list.add(new Address(中国，"广州"));
            list.add(new Address("中国,"东莞"));

            Person person = new Person();
            pername.setName("狗娃");
            person.setAddresses(list);
         */
        var person2 = {"name":"狗娃","addresses":[{"country":"中国","city":"广州"},{"country":"中国","city":"东莞"}]}
        alert("访问person的姓名与第二个地址：name:"+ person2.name+" 第二个地址："+person2.addresses[1].city );



    </script>
</head>
<body>

</body>
</html>
```

效果

![image-20200927115257773](AJAX-JSON.assets/image-20200927115257773.png)

### 小结



1. json格式

> {“key”，value}

2. json访问的格式：

> 1. 访问属性 ： json对象.属性名
> 2. 访问数组:    json对象[索引值]



​     

## 11.JSON格式：格式转换【应用】

### 目标

能够理解服务器端如何返回json字符串数据

掌握jackson的作用

掌握jackson的核心类对象与方法



### 为什么java对象转换为json字符串？

![image-20200717151722903](AJAX-JSON.assets/image-20200717151722903.png)

后端从数据库获取的是java对象数据，然而异步请求响应给前端需要使用json字符串， 所以要将java对象转换为json字符串

### 常用第三方工具

![1547003633648](AJAX-JSON.assets/1547003633648.png)

### JackSon核心api语法

| 核心类ObjectMapper                         |                                  |
| ------------------------------------------ | -------------------------------- |
| String  writeValueAsString(Object object); | 将java对象转换为json字符串的语法 |



### jackson工具的使用步骤

1. 导入jackson的jar包
2. 可以使用jackson提供的工具类进行对java对象转换为json字符串

### User实体类代码

```java
package com.itheima.model;

import java.util.List;

public class User {

    private String name;
    private int age;

    private List<Address> addressList;


    public List<Address> getAddressList() {
        return addressList;
    }

    public void setAddressList(List<Address> addressList) {
        this.addressList = addressList;
    }

    public User() {
    }


    public User(String name, int age, List<Address> addressList) {
        this.name = name;
        this.age = age;
        this.addressList = addressList;
    }

    public User(String name, int age) {
        this.name = name;
        this.age = age;
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

    public String toString() {
        return "User{name = " + name + ", age = " + age + "}";
    }
}

```

**Address类**

```java
package com.itheima.model;

public class Address {

    private String country;

    private String city;


    public Address() {
    }

    public Address(String country, String city) {
        this.country = country;
        this.city = city;
    }

    /**
     * 获取
     * @return country
     */
    public String getCountry() {
        return country;
    }

    /**
     * 设置
     * @param country
     */
    public void setCountry(String country) {
        this.country = country;
    }

    /**
     * 获取
     * @return city
     */
    public String getCity() {
        return city;
    }

    /**
     * 设置
     * @param city
     */
    public void setCity(String city) {
        this.city = city;
    }

    public String toString() {
        return "Address{country = " + country + ", city = " + city + "}";
    }
}

```



### Jackson转换json代码

```java
package com.itheima.test;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.itheima.model.Address;
import com.itheima.model.User;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/*
jackson核心： ObjectMapper

jackson的核心方法：writeValueAsString


 */
public class JsonTest {

    public static void main(String[] args) throws JsonProcessingException {
        //1普通的java对象
        User user  = new User("狗娃",12);

        //Jackson的核心类： ObjectMapper
        ObjectMapper objectMapper = new ObjectMapper();
        //jackson的核心方法：writeValueAsString
        String javaBean = objectMapper.writeValueAsString(user);

        System.out.println("javabean的json格式："+ javaBean);

        //2.Map集合
        Map<String,Object> map = new HashMap<>();
        map.put("name","铁蛋");
        map.put("age",12);
        String mapJson = objectMapper.writeValueAsString(map);
        System.out.println("map的json格式："+ mapJson);


        //3.  List集合
        List<User> list = new ArrayList<>();
        list.add(new User("铁蛋",12));
        list.add(new User("狗蛋",18));
        String listJson = objectMapper.writeValueAsString(list);
        System.out.println("list集合的json:"+ listJson);


        // 混合对象
        List<Address> addressList = new ArrayList<>();
        addressList.add(new Address("中国","广州"));
        addressList.add(new Address("中国","东莞"));

        User user2 = new User("旺财",12,addressList);
        String user2Json = objectMapper.writeValueAsString(user2);
        System.out.println("混合对象的json:" + user2Json);


    }
}

```

### 效果

![1603176512433](AJAX-JSON.assets/1603176512433.png)

### 小结

jackson的核心类对象与方法是什么?

> 核心类：ObjectMapper
>
> 核心方法：writeValueasString


## 12.内容自动补全：环境搭建【应用】

### 需求

​	在输入框输入关键字，下拉框中异步显示与该关键字相关的用户名称

### 效果

​      将素材中查询search.html页面放入到本项目中

​	![1546958775299](AJAX-JSON.assets/1546958775299.png)      

### 环境搭建

在素材中页面导入到项目中、jar、工具类、配置文件

![1603010372361](AJAX-JSON.assets/1603010372361.png) 

页面原型素材

![image-20200927151458368](AJAX-JSON.assets/image-20200927151458368.png) 

导入页面原型

<img src="../../../../Documents/Java笔记/就业班/3.前端/day34(Ajax&json)/笔记/assets/image-20200717154849022.png" width="60%" > 

修改配置文件名数据库信息db.properties

```properties
jdbc.url=jdbc:mysql:///db1
jdbc.username=root
jdbc.password=root
jdbc.driver=com.mysql.jdbc.Driver
```

执行数据库sql

```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) DEFAULT NULL,
  `password` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`)
);

INSERT INTO `user` VALUES ('1', '张三', '123');
INSERT INTO `user` VALUES ('2', '李四', '123');
INSERT INTO `user` VALUES ('3', '王五', '123');
INSERT INTO `user` VALUES ('4', '赵六', '123');
INSERT INTO `user` VALUES ('5', '田七', '123');
INSERT INTO `user` VALUES ('6', '孙八', '123');
INSERT INTO `user` VALUES ('7', '张三丰', '123');
INSERT INTO `user` VALUES ('8', '张无忌', '123');
INSERT INTO `user` VALUES ('9', '李寻欢', '123');
INSERT INTO `user` VALUES ('10', '王维', '123');
INSERT INTO `user` VALUES ('11', '李白', '123');
INSERT INTO `user` VALUES ('12', '杜甫', '123');
INSERT INTO `user` VALUES ('13', '李贺', '123');
INSERT INTO `user` VALUES ('14', '李逵', '123');
INSERT INTO `user` VALUES ('15', '宋江', '123');
INSERT INTO `user` VALUES ('16', '王英', '123');
INSERT INTO `user` VALUES ('17', '鲁智深', '123');
INSERT INTO `user` VALUES ('18', '武松', '123');
INSERT INTO `user` VALUES ('19', '张薇', '123');
INSERT INTO `user` VALUES ('20', '张浩', '123');

```

### 创建实体类

```java
package com.itheima.model;

public class User {

    private int id;
    private String name;
    private String password;


    public User() {
    }

    public User(int id, String name, String password) {
        this.id = id;
        this.name = name;
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
        return "User{id = " + id + ", name = " + name + ", password = " + password + "}";
    }
}

```



## 13.内容自动补全：后端处理【应用】

### 目标

后端根据条件查询用户列表数返回json字符串给前端



### 实现步骤

![image-20200927154603863](AJAX-JSON.assets/image-20200927154603863.png)

### UserDao代码

```java
package com.itheima.dao;

import com.itheima.model.User;
import org.apache.ibatis.annotations.Select;

import java.util.List;

public interface UserDao {

    //根据name进行模糊查询
    @Select("SELECT * FROM USER WHERE NAME LIKE CONCAT(#{name},'%') LIMIT 0,4")
    List<User> findByName(String name);
}

```

### UserService代码

```java
package com.itheima.service;

import com.itheima.dao.UserDao;
import com.itheima.model.User;
import com.itheima.utils.SqlSessionUtils;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.session.SqlSession;

import java.util.List;

public class UserService {

    public List<User> findByName(String name){
        SqlSession sqlSession = SqlSessionUtils.getSqlSession();
        UserDao userDao = sqlSession.getMapper(UserDao.class);
        List<User> list  = userDao.findByName(name);
        //关闭资源
        SqlSessionUtils.close(sqlSession);
        return list;
    }
}

```

### SelectUserServlet代码

> //目标：处理异步用户搜索请求
> //1.获取请求数据参数name搜索条件
> //2.调用业务根据name查找符合的用户列表
> //3.将用户列表数据转换为json字符串返回

```java
package com.itheima.web;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.itheima.model.User;
import com.itheima.service.UserService;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.List;

@WebServlet("/userServlet")
public class UserServlet extends HttpServlet {

    private UserService userService = new UserService();

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
      //  response.setContentType("text/html;charset=utf-8"); //响应的内容是文本格式的html代码，
        response.setContentType("text/json;charset=utf-8"); //响应的内容是文本格式的html代码，如果后端写了，那么前端则不需要写dataType
        PrintWriter out = response.getWriter();

        //1. 获取用户名
        String name = request.getParameter("name");

        //2. 根据用户名查找用户列表
        List<User> list = userService.findByName(name);

        //3. 创建ObjectMapper对象，把列表转换为json
        String json = new ObjectMapper().writeValueAsString(list);

        //4. 把json写出
        out.write(json);

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

### 小结

后端根据条件查询用户列表数返回json字符串给前端

> json字符串的应用： 将java对象转换为json字符串返回



## 14.内容自动补全：前端处理【应用】

### 目标

提交异步请求并更新页面显示数据

掌握内容自动补全什么时候发送异步请求



### 实现步骤

![image-20200927160946108](AJAX-JSON.assets/image-20200927160946108.png)

### search.html代码

> //目标1：发送异步查询用户列表请求
> //1.给文本框注册keyup事件
> //2.获取输入的条件值
> //3.提交异步请求，传递搜索条件name,查询符合的用户列表返回
> //4.获取返回的数据，更新到页面上显示
> //4.1 循环遍历userList,拼接每一个用户名的html代码 //4.2 将拼接好的列表html代码设置到div(id=show)标签体内 //4.3 让div(id=show)显示出来
>
> //目标2：实现用户名点击显示到文本框
> //1.给div(id=show)里面的所有div注册动态点击事件
> //2.获取当前div标签体的文本内容数据 //3.将数据设置到文本框的value值 //4.让div(id=show)隐藏

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>自动完成</title>
    <style type="text/css">
        .content {
            width: 400px;
            margin: 30px auto;
            text-align: center;
        }

        input[type='text'] {
            box-sizing: border-box;
            width: 280px;
            height: 30px;
            font-size: 14px;
            border: 1px solid #38f;
        }

        input[type='button'] {
            width: 100px;
            height: 30px;
            background: #38f;
            border: 0;
            color: #fff;
            font-size: 15px;
        }

        #show {
            box-sizing: border-box;
            position: relative;
            left: 7px;
            font-size: 14px;
            width: 280px;
            border: 1px solid dodgerblue;
            text-align: left;
            border-top: 0;
            /*一开始是隐藏不可见*/
            display: none;
        }

        #show div {
            padding: 4px;
            background-color: white;
        }

        #show div:hover {
            /*鼠标移上去背景变色*/
            background-color: #3388ff;
            color: white;
        }
    </style>
    <script type="text/javascript" src="js/jquery-3.3.1.min.js"></script>
</head>
<body>
<div class="content">
    <img alt="传智播客" src="img/logo.png"><br/><br/>
    <input type="text" name="word" id="word">
    <input type="button" value="搜索一下">
    <div id="show"></div>
</div>

<script type="text/javascript">
    //1. 给的输入框注册键盘松开的事件
    $("#word").keyup(function(){
        //2. 判断输入框的内容是否为空， 如果不为空就发出ajax的请求
        var username = $(this).val();
        if(username){
            $.ajax({
                url:"/userServlet",
                data:{"name":username},
                method:"get",
                //dataType:"json",  //服务器返回的数据类型，默认是text格式。
                success:function(userList){
                    //3. 得到用户列表的时候，遍历用户列表 每一个用户就是拼接一个div
                    var html = "";
                    for(var user of userList){
                        html+=`<div>${user.name}</div>`;
                    }
                    //4. 把拼接的字符串交给id为show标签体
                    $("#show").html(html);
                    //让show的div标签显示出来
                    $("#show").show();
                }
            });

        }else{
            //
            $("#show").hide(); //hide（） 隐藏  show() 显示
        }
    });


    //鼠标选中提示，自动补全到输入框

    //1. 找到show的div注册鼠标点击
    $("#show").on("click","div",function(){
        //2. 获取当前被点击的内容
        var content = $(this).text();
        //3. 把点击内容补全到输入框
        $("#word").val(content);
        //4. 让提示框隐藏
        $("#show").hide();
    })




</script>
</body>
</html>

```

### 小结

内容自动补全什么时候发送异步请求

> keyup事件





## 15.跨域问题演示与解决【扩展】

### 目标

了解ajax默认无法跨域访问资源数据

理解如何解决跨域问题



### 跨域问题说明

什么是跨域： A服务器的程序访问了B服务器的程序，这个会产生域名、端口、协议的不同的情况，这个访问的过程称作为跨域。



#### 介绍

ajax默认不允许跨域访问，跨域如下3个信息中一个不一样就是跨域 访问

* 协议
* 域名
* 端口

#### 同源策略(同域介绍，就是协议，域名，端口一样)。

| **URL**                                                    | **说明**         | 是否允许Aajx通信 |
| ---------------------------------------------------------- | ---------------- | ---------------- |
| `http://www.a.com/a.js`<br>  `http://www.a.com/b.js `      | 同一域名         | 允许             |
| `http://www.a.com/a.js`  <br>  `http://www.b.com/a.js`     | 不同域名         | 不允许           |
| `http://www.a.com:8000/a.js` <br>  `http://www.a.com/b.js` | 同一域名不同端口 | 不允许           |
| `https://www.a.com/a.js`  <br/>  `http://www.a.com/b.js`   | 同一域名不同协议 | 不允许           |

http协议默认端口是80

https协议默认端口是443

### 跨域问题演示

#### 创建2个跨域项目

1. 创建2个Web模块

![image-20191230224824488](AJAX-JSON.assets/image-20191230224824488.png)&nbsp;

#### 导入jquery到项目项目中

![image-20200717111453511](AJAX-JSON.assets/image-20200717111453511.png) 

#### 部署8080服务器项目

![image-20200717111647326](AJAX-JSON.assets/image-20200717111647326.png)

![image-20200717111714181](AJAX-JSON.assets/image-20200717111714181.png)

#### 部署8888服务器项目

![image-20200717111738975](AJAX-JSON.assets/image-20200717111738975.png)

注意端口要与上面的服务器设置不一样,否则端口会有冲突

![image-20200717111804485](AJAX-JSON.assets/image-20200717111804485.png)



#### 8080服务器上项目index.jsp代码

![image-20200717112233919](AJAX-JSON.assets/image-20200717112233919.png) 

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="/js/jquery-3.3.1.min.js"></script>
    <script>
        //发送一个异步的请求
        $.ajax({
            url:"http://localhost:8080/demo1",
            success:function(info){
                alert("8080服务器的响应数据："+info);
            }
        })
    </script>
</head>
<body>

</body>
</html>
```



#### 8888服务器项目上Servlet代码

![image-20200717112253627](AJAX-JSON.assets/image-20200717112253627.png) 

```java
package com.itheima.web;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/demo1")
public class Demo1Servlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        out.write("记得给钱");
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

#### 启动2个服务器

![image-20200717105327542](AJAX-JSON.assets/image-20200717105327542.png)

#### 8080进行跨域访问8888报错

访问8080服务器项目资源index.jsp页面, 没有任何效果, 使用F12查看报错

```url
http://localhost:8080/ajax_cross_01_war_exploded/
```

![image-20200717112032521](AJAX-JSON.assets/image-20200717112032521.png)

跨域的错误，如下所示：

![image-20200717112118050](AJAX-JSON.assets/image-20200717112118050.png)

### 解决跨域演示

#### 跨域响应头解决方案

设置响应头语法，后台代码在被请求的Servlet中添加Header设置：

```java
response.setHeader("Access-Control-Allow-Origin", "*");
```

> Access-Control-Allow-Origin这个Header在W3C标准里用来检查该跨域请求是否可以被通过，如果值为*则表明当前页面可以跨域访问。默认的情况下是不允许的。

#### 8888端口服务器的Servlet代码

```java
package com.itheima.web;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

@WebServlet("/demo1")
public class Demo1Servlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        response.setHeader("Access-Control-Allow-Origin","*"); //允许其他的服务器进行跨域的访问
        PrintWriter out = response.getWriter();
        out.write("记得给钱");
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

#### 重新部署8888服务器上的项目

![image-20200717112602399](AJAX-JSON.assets/image-20200717112602399.png) 

#### 8080再次进行跨域访问8888成功

![image-20200717112355286](AJAX-JSON.assets/image-20200717112355286.png)

