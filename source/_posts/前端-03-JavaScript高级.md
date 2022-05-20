---
title: 前端(03)-javascript高级
tags:
  - 笔记
  - 前端
  - javascript
categories:
  - 前端
abbrlink: 473
date: 2020-10-07 12:47:11
---





# 学习目标

1**、能够在JS中定义命名函数和匿名函数** 

**2、能够使用JS中常用的事件** 

**3、能够使用数组及其常用的方法** 

**4、能够掌握事件的注册方式** 

**5、能够使用window对象常用的方法** 

**6、能够使用location对象常用的方法和属性** 

**7、能够使用history对象常用的方法** 

**8、能够使用domcument对象的方法来查找节点** 

**9、能够使用JavaScript对CSS样式进行操作** 







# 学习内容

## <font color="red">命名函数和匿名函数</font>

### 目标

命名函数和匿名函数的使用

### 函数的概述

什么是函数：类似于Java中方法，关键字：function。Java中所有的方法都是必须起名字

两种函数：命名函数和匿名函数

### 命名函数的格式

[ 内容表示可选 ]  <必须的>

```javascript
function 函数名(参数) {
   方法体;
   [return 返回值]
}
```

###  函数要注意的事项

1. 函数没有返回的类型
2. 形参不能使用var关键字声明形参
3. 如果函数有返回值，则加上return，否则就不加。
4. 函数没有重载的概念，后定义的同名函数会直接覆盖前面定义的同名变量
5. 在JS中形参的个数与实参的个数无关，只会通过方法名去调用，建议参数个数要匹配。
6. 调用方法的时候实参是传递给内部维护的arguments数组对象的

### 自定义函数

#### 需求

定义一个函数实现加法功能

#### 代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script>
        /*函数的定义格式：
            function 函数名(参数) {
               方法体;
               [return 返回值]
            }

          函数要注意的事项：
            1. 定义函数的时候形式参数不能用var关键字。
            2. 在javascript中的函数是没有返回值类型，如果需要返回结果直接使用return关键字即可，不返回也可以。
            3. 在javascript中是没有函数重载的概念，后定义的同名函数会直接覆盖前面的同名函数的。
            4. 在javascript中调用函数是不根据形式参数的个数去调用的，只是根据了函数名去调用。
            5. 在javascript任何一个函数内部都隐式的维护了一个arguments的数组对象， 调用函数的时候实参其实是直接
            传递给了arguments数组的，然后再从arguments数组里面获取元素交给形参。
            6. 虽然我们不需要定义形参也可以得到实参，但是我都建议同学们去定义形参，因为让调用者见名知意。

        */

        //需求： 定义一个函数做两个参数的加法功能
        function sum(a,b){
            var result = a+b;
           document.write("两个参数总和："+ result);
           // return result;
        }


        function sum(){
          /*  var result = a+b+c;
            document.write("三个参数总和："+ result);*/
          var sum = 0;
          for(var i =0; i<arguments.length ; i++){
                sum+=arguments[i];
           }
           document.write("总和："+ sum);

        }


        //调用函数
        sum(1,2,3); //实参
        /*var c = sum(1,2);
        document.write("返回值："+ c);*/


        //构造器
        function Person(name,age){
            this.name = name;
            this.age = age;
        }


        var p = new Person("狗娃",12);




    </script>
</head>
<body>

</body>
</html>
```

### 隐藏数组的执行过程

1. 调用的时候传递的是实参，实参先将所有的值赋值给函数内部的隐藏数组arguments
2. 在函数内部使用形参的时候，数据从隐藏的数组中去取出值，然后再计算。

![1551950685737](../../../../../../../assets/1551950685737.png)

### 案例

需求：在函数内部输出隐藏数组的长度和每个元素



#### 代码

```html
 function sum(){
          /*  var result = a+b+c;
            document.write("三个参数总和："+ result);*/
          var sum = 0;
          for(var i =0; i<arguments.length ; i++){
                sum+=arguments[i];
           }
           document.write("总和："+ sum);

        }


        //调用函数
        sum(1,2,3); //实参
```

### 匿名函数

#### 语法

```javascript
//定义匿名函数
var 变量名 = function(形参) {
   return 返回值;
}

```

#### 函数调用代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script>
        /*javascript中匿名函数
        * 匿名函数： 没有名字的函数
        *
        * 匿名函数一般都需要配合事件去使用的，不能单独去使用。因为匿名函数没有函数名没法调用。
        * 如果真的需要调用匿名函数我们可以定义一个变量去指向该函数。
        * */



        //需求：使用匿名函数计算两个参数的总和
        var sum = function (a,b){
            var result = a+b;
            document.write("总和："+ result);
        }

        //调用
        sum(1,2);



    </script>
</head>
<body>

</body>
</html>
```

### 小结

1. 定义函数的关键字是什么？函数的返回值需要指定数据类型吗？

   ​	function ,   不需要

2. 函数是否有重载？

   - 没有

3. 每个函数内部都有一个隐藏数组名是：

   - arguments

   



## 案例：轮播图

### 目标

1. 函数的应用
2. 定时函数的使用

### 方法说明

| 方法                                    | **描述**                                                     |
| --------------------------------------- | ------------------------------------------------------------ |
| **document.getElementById("id")**       | 通过id得到一个元素，如果网页中有相同ID的元素，得到第1个。    |
| **window.setInterval("函数名()",时间)** | window可以省略，每过一段时间调用一次函数。<br />参数1：被调用的函数<br />参数2：每隔多久，单位是毫秒 |
| **window.setInterval(函数名,时间)**     | 功能上与第1种相同，没有双引号，没有小括号。                  |

```
因为浏览器解析网页的时候从上往下解析的，先遇到脚本就会执行，这时候不能得到后面加载的网页元素。
结论：JS代码要写在网页元素的后面
```

### 需求

每过3秒中切换一张图片的效果，一共5张图片，当显示到最后1张的时候，再次显示第1张。

### 效果

![1551952153233](../../../../../../../assets/1551952153233.png)

### 步骤

1. 创建HTML页面，页面有一个img标签，id为pic，宽度600。
2. body的背景色为黑色，内容居中。
3. 五张图片的名字依次是0~4.jpg，放在项目的img文件夹下，图片一开始的src为第0张图片。
4. 编写函数：changePicture()，使用setInterval()函数，每过3秒调用一次。
5. 定义全局变量：num=1。
6. 在changePicture()方法中，设置图片的src属性为img/num.jpg。
7. 判断num是否等于5，如果等于5，则num=0；否则num++。

### 代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script>

        //需求：实现轮播图

        //1. 定义变量记录当前要切换的图片的名字
        var num = 0;

        //2. 定义一函数切换图片
        function changeImage(){
            //找到图片标签
            var img = document.getElementById("img");
            num++;
            //修改src属性
            img.src = "img/"+num+".jpg";
            if(num==4){
                num=-1;
            }

        }


        //3. 开启定时任务调用切换图片函数
        window.setInterval(changeImage,1000);


    </script>
</head>
<body>
    <img id="img" src="img/0.jpg" width="400px" height="300px"/>
</body>
</html>
```

### 小结

| 方法                                                         | **描述**               |
| ------------------------------------------------------------ | ---------------------- |
| **document.getElementById("id")**<br />                      | 根据id查找标签对象     |
| **window.setInterval("函数名()",时间)**   <br />**window.setInterval(函数名,时间)** | 指定相隔毫秒数调用方法 |



## 设置事件的两种方式

### 目标

使用命名函数和匿名函数设置事件

### 什么是事件

在网页中操作的时候，会激活各种事件，如：鼠标移动，点击按钮，双击，失去焦点等。我们可以通过JS代码来对这些事件编程，当激活这些事件的时候，实现相应的功能。可以让网页"活"起来，与用户有交互的功能。

### 设置事件的两种方式

1. 方式一：html元素直接注册事件

```html
<input type="button" onclick="clickMe()" id="btn">

function clickMe() {
   //事件处理函数
}
```



2. 方式二：js对象注册函数

```javascript
document.getElementById("btn").onclick = function() {  //事件处理函数}
```



### 事件处理案例

#### 需求

有两个按钮，点第1个弹出一个信息框，点第2个弹出另一个信息框。分别使用两种不同的方式激活事件

#### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title></head><body>    <input id="b1" type="button" onclick="test()" value="按钮"/></body><script>    /* 事件： 事件就是指发生了指定的事件之后会触发某一个方法    *    *  事件的注册方式：    *        方式一： 直接在html元素中注册(命名函数注册)    *                格式： <标签名  onXXX="函数名()" />    *    *        方式二： 可以直接找到js对象注册。    *                格式：  js对象.onXXX = function(){}    *    *       推荐使用： js对象注册方式, 因为js代码可以与html代码分离，提高代码可维护性/    * */    //需求： 给按钮注册事件单击时间    //找到按钮标签对象    var button = document.getElementById("b1");    //给button对象注册事件    button.onclick=function(){        alert("哎呀，被点了..");    }    //方式一：    function test(){        alert("舒服...");    }</script></html>
```

### 小结

1. html元素如何注册事件

       ```html

   <html标签  on事件名="函数()">
       ```

   

2. js对象如何注册事件(推荐使用)

   ​	js对象.on事件名=function(){}



## <font color="red">事件介绍：onload、onclick、ondblclick</font>

### 目标

1. 加载完毕事件
2. 鼠标单击
3. 鼠标双击

### 加载完成事件

1. 事件名：所有的事件都以on开头
2. 示例：页面加载完毕以后，才执行相应的JS代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title>    <script>        /*onload事件： 当指定的对象被加载完毕之后会触发方法        * */        //body标签被加载完毕的时候就会触发的方法     /*   function ready(){            alert("body内容已经加载完毕了..");        }*/        //浏览器窗口内容被加载完毕了        window.onload = function(){            //浏览器窗口内容被加载完毕了            alert("aaa");        }    </script></head><body ></body></html>
```



### 鼠标点击事件

1. 单击：onclick
2. 双击：ondblclick

### 案例演示

需求：单击复制文本内容，双击清除文本内容

思路：文本框.value = "内容"   文本框.value=""

![1551952995946](前端-03-jquery高级.assets/1551952995946.png)

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>单击和双击事件</title></head><body>姓名： <input type="text" id="t1"> <br/><br/>姓名： <input type="text" id="t2"> <br/><br/><input type="button" value="单击复制/双击清除" id="btn"><script type="text/javascript">   //需求： 给按钮注册单击与双击时间，但是是复制，双击是清除   //找到按钮    var btn = document.getElementById("btn");    var input1 = document.getElementById("t1");    var input2 = document.getElementById("t2");    //注册单击事件   btn.onclick=function(){        input2.value = input1.value;   }    //注册双击事件    btn.ondblclick = function(){       input1.value="";       input2.value="";    }</script></body></html>
```

### 小结

1. 加载完成事件名：onload
2. 单击事件名：  onclick
3. 双击事件名： ondbclick



## 事件介绍：鼠标移入和移出

### 目标

1. 鼠标移入
2. 鼠标移出
3. this关键字

### 事件名

1. 移入：onmouseover
2. 移出：onmouseout

### 示例

需求：将鼠标移动到img上显示图片，移出隐藏图片。图片设置边框，宽500px

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title>    <script>      //需求：鼠标进入的时候显示美女，鼠标移出的时候让美女消失        window.onload=function(){            //1. 找到图片对象            var img = document.getElementById("img");            //方式一： 普通方式          /*  //2. 给图片注册鼠标移入事件            img.onmouseover= function(){                img.src="img/a.jpg";            }            //3. 给图片注册鼠标移出事件            img.onmouseout=function(){                img.src="";            }*/        }        //方式二： 使用this关键字实现        function showImage(imgObj){          //  var img = document.getElementById("img");            imgObj.src="img/a.jpg";        }      function hiddenImage(imgObj){          //  var img = document.getElementById("img");          imgObj.src="";      }    </script></head><body>      <!--this代表是当前对象，也就是触发该事件的标签对象-->      <img width="400px" height="300px" onmouseover="showImage(this)" onmouseout="hiddenImage(this)" id="img"  alt="鼠标进入看美女，鼠标出去消失"/></body></html>
```

### this关键字的作用

  this代表的了当前对象。 

1. 出现在控件的事件方法中：

```
  <img width="400px" height="300px" onmouseover="showImage(this)" />
```

2. 出现在匿名函数的代码中：

```
   img.onmouseover= function(){                this.src="img/a.jpg";            }
```

### 小结

1. 鼠标移上： onmouseover
2. 鼠标移出： onmouseout



## 事件介绍：得到焦点、失去焦点、改变事件

### 目标

1. 得到焦点
2. 失去焦点
3. 改变事件

### 事件名

1. 得到焦点：onfocus
2. 失去焦点： onblur

### 焦点事件示例

### 需求：

​	获取焦点的时候请把input标签的内容清空，失去焦点还原。

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title>    <script>      // 需求：输入框获取到焦点的时候清空文本，失去焦点的时候检查用户输入的内容是是空，则还原        window.onload = function(){            //1. 找到输入框            var username = document.getElementById("username");            //2. 注册获取焦点事件            username.onfocus = function(){                username.value="";            }            //3. 注册失去焦点事件            username.onblur=function(){                //判断内容是否为空,如果是空则还原默认值                var content = username.value;                if(content==""){                    username.value="用户名由6位英文字母组成";                }            }        }    </script></head><body>    用户名：<input type="text" id="username" value="用户名由6位英文字母组成"/></body></html>
```

### 改变事件示例

事件名：onchange，文本框的改名事件等到失去焦点以后才激活。



#### 效果

![1566355600750](前端-03-jquery高级.assets/1566355600750.png)

#### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>改变事件</title></head><body>城市：<select id="city">    <option value="广州">广州</option>    <option value="深圳">深圳</option>    <option value="东莞">东莞</option></select><hr/>选择的城市：<input type="text" id="eng"><hr/><script type="text/javascript">   //需求： 下拉列表内容发生改变的时候，把下拉列表的value属性值拷贝到input标签中    var city = document.getElementById("city");    var eng = document.getElementById("eng");    //给city下拉框注册改变事件    city.onchange=function(){        eng.value = this.value;    }</script></body></html>
```

### 小结

1. 得到焦点事件名：onfocus
2. 失去焦点事件名：onblur
3. 改变事件事件名：onchange



## 内置对象：数组对象

### 目标

1. 数组的创建的方式
2. 数组中的方法

### 创建数组的四种方式

数组在JS中是一个内置对象

| **创建数组的方式**           | **说明**                        |
| ---------------------------- | ------------------------------- |
| new Array()                  | 创建长度为0的数组               |
| new Array(n)                 | 创建一个指定长度的数组，长度为n |
| new Array(元素1,元素2,元素3) | 指定数组中每个元素创建一个数组  |
| [元素1,元素2,元素3]          | 指定数组中每个元素创建一个数组  |

### 数组的特点

1. 创建数组，元素的类型可以不同
2. 数组的长度可以动态变化的
3. 数组中还包含了方法

### 创建数组代码演示

```java
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title>    <script>        /*数组的创建方式        *       方式一 : new Array( )  创建一个长度为0的数组对象。  推荐        *       方式二：  new Array(length) 指定长度创建数组对象。        *        *       方式三：  new Array(元素1，元素2.。) 指定元素创建数组对象        *       方式四：  [元素1，元素2...]  指定元素创建数组对象        *        *        * javascript数组的特点：        *      1. javascript中的数组是可以存储任意类型数据。        *      2. javacript中的数组长度是可变的。        * */        //创建一个长度为0的数组对象。        var arr1= new Array();        arr1[0]="狗娃";        arr1[5]="狗剩";        for(var i = 0; i<arr1.length ;i++){            document.write(arr1[i]+",");        }        document.write("<br/>");        document.write("数组的长度："+ arr1.length +"<br/>");        var arr2= new Array(10);        document.write("arr2数组的长度："+ arr2.length +"<br/>");        //创建数组的时候马上指定元素        var arr3 = ["狗娃","狗剩","铁蛋"];        for(var i = 0; i<arr3.length ;i++){            document.write(arr3[i]+",");        }    </script></head><body></body></html>
```





### 常用方法

| **方法名**          | **功能**                                                     |
| ------------------- | ------------------------------------------------------------ |
| **concat()**        | 用于拼接一个或多个数组                                       |
| **reverse()**       | 用于数组的反转                                               |
| **join(separator)** | 用于将整个数组使用分隔符拼接成一个字符串。相当于split()反操作 |
| **sort()**          | 排序，如果要对数字进行排序，必须指定比较器。默认是按字符串进行排序 |
| **pop()**           | 删除并返回数组的最后一个元素                                 |
| **push()**          | 添加一个元素到数组的最后面                                   |



### 方法演示案例

#### 效果



#### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title>    <script>        /*数组常用的方法                concat()  用于拼接一个或多个数组,并返回一个新的数组               join(separator) 用于将整个数组使用分隔符拼接成一个字符串。相当于java中的Arrays.toString(),只不过javascript中可以指定元素与元素之间的分隔符。                reverse()   用于数组的反转                sort()  排序，如果要对数字进行排序，必须指定比较器。默认是按字符串进行排序                pop() 删除并返回数组的最后一个元素                push() 添加一个元素到数组的最后面        */        var arr1 = ["狗娃","狗剩","铁蛋"];        var arr2 = ["牛娃","牛剩","牛蛋"];        //concat合并数组        var arr3 = arr1.concat(arr2);        //删除并返回最后一个元素        var lastItem = arr3.pop();        document.write("被删除的最后一个元素："+ lastItem+"<br/>");        //添加到最后        arr3.push("AA");        document.write("数组的元素："+ arr3.join(",")+"<br/>");        var arr4= [10,3,13,21];        //对数组的元素进行排序        function compare(a,b){            return a-b;        }        arr4.sort(compare);        arr4.reverse();        document.write("arr4数组的元素："+ arr4.join(",")+"<br/>");    </script></head><body ></body></html>
```

### 小结

| **方法名**          | **功能**                                     |
| ------------------- | -------------------------------------------- |
| **concat()**        | 拼接两个数组，返回一个拼接后的数组对象       |
| **reverse()**       | 翻转                                         |
| **join(separator)** | 指定分隔符拼接数组的元素内容，返回一个字符串 |
| **sort()**          | 排序，对数字排序一定要传入比较器             |
| **pop()**           | 弹出最后一个元素                             |
| **push()**          | 在数组最后添加一个元素s                      |



## 使用JS修改CSS的样式

### 目标

JS操作CSS的两种方式

### 在JS中操作CSS属性命名上的区别

以前CSS都是写死的，现在可以通过JS代码来修改CSS，让CSS也"活"起来。

| **CSS中写法** | **JS中的写法** | **说明**               |
| ------------- | -------------- | ---------------------- |
| **color**     | color          | 一个单词样式名字不变的 |
| **font-size** | fontSize       | 多个单词使用驼峰命名法 |

### 方式一

一条语句只能修改一个样式

```
元素对象.style.样式名 = 样式值;
```



### 方式二

1. 先定义一个类名
2. 使用类名一次修改多个样式

```
元素对象.className= 类名;
```



### 示例：点按钮，修改p标签的字体、颜色、大小

### 效果

![1552049542846](前端-03-jquery高级.assets/1552049542846.png) 

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>使用JS修改样式</title>    <style>        .myclass {            color: red;            font-size: 40px;            font-family: 隶书;        }    </style></head><body><p id="p1">第一自然段</p><p id="p2">第二自然段</p><hr/><input type="button"  value="改变行内样式" id="b1"><input type="button" value="改变类样式" id="b2"><script type="text/javascript">/*    修改html元素的css样式的方式：            方式一： 通过操作style内置对象实现。            格式：  js对象.style.css样式名 = 样式值。            方式二： 直接使用类选择器         格式：  js对象.className = 类选择器名字。 */    var b1 = document.getElementById("b1");    var p1 = document.getElementById("p1");    var b2 = document.getElementById("b2");    var p2 = document.getElementById("p2");    //给按钮1注册事件    b1.onclick = function(){        p1.style.fontSize="32px";        p1.style.backgroundColor="red";    }    //使用类选择器去修改    b2.onclick=function(){        p2.className="myclass";    }</script></body></html>
```

### 网页变化

![1564295776176](前端-03-jquery高级.assets/1564295776176.png) 

### 小结

JS修改CSS样式的两种方式

	- js对象.style.css样式名 = 样式值- js对象.className = 类选择器名称；







## 案例：鼠标移动到文字上显示提示文字

### 目标

![1552049859046](前端-03-jquery高级.assets/1552049859046.png)

### 技术点

| **css样式名：display** | **说明**                   |
| ---------------------- | -------------------------- |
| **none**               | 这个元素在网页不可见       |
| **block**              | 将元素设置成块级元素(可见) |
| **inline**             | 将元素设置成内联元素(可见) |

### 步骤

1)	网页上有一个a标签链接，a标签下有一个不可见的div。
2)	鼠标移到a标签，设置div的样式，让div可见
3)	鼠标移出a标签，设置div的样式，让div不可见。

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>JS修改css样式</title>    <style>        .info {            border: dashed black 1px;            border-radius: 5px;            width: 180px;            font-size: 14px;            text-align: center;            height: 25px;            /*行高与高度相同，垂直居中*/            line-height: 25px;            background-color: yellow;        }        a:link {            text-decoration: none;        }        a:hover {            color: red;        }    </style></head><body><a href="04_轮播图.html" id="company">传智播客</a><a href="04_轮播图.html" id="123"><div id="info" style="display: none">点我可以看到精彩内容哦</div></a><script type="text/javascript">    //1. 查找到a标签    var company = document.getElementById("company");    var div = document.getElementById("info");    //2. 注册鼠标经过事件    company.onmouseover = function(){        div.style.display = "block"; //显示出来        div.className = "info";    }    //3. 注册鼠标移开事件    company.onmouseout=function(){        div.style.display="none";    }</script></body></html>
```

### 小结

隐藏元素使用什么样式？

​	



## 内置对象：日期对象

### 目标

日期对象方法的使用

### 创建日期对象

```
var date = new Date();  //创建现在的时间
```

### 日期对象的方法

![1552052803821](前端-03-jquery高级.assets/1552052803821.png)



### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title>    <script>        /*           Date对象是用于获取当前的系统时间         */        //获取当前系统时间        var date = new Date();        document.write("年："+date.getFullYear()+"<br/>");        document.write("月："+(date.getMonth()+1)+"<br/>");        document.write("日："+date.getDate()+"<br/>");        document.write("时："+date.getHours()+"<br/>");        document.write("分："+date.getMinutes()+"<br/>");        document.write("秒："+date.getSeconds()+"<br/>");        document.write("当前系统时间："+date.toLocaleString()+"<br/>");    </script></head><body></body></html>
```

### 小结

说说下面方法的作用

	- new Date();  创建了日期对象- toLocalString() 返回本地的日期格式的字符串



## BOM：location对象

### 目标

1. 什么是BOM编程
2. location对象的方法和属性

### 什么是BOM

BOM：Browser Object Model 浏览器对象模型，javascript==内置了很多对象描述了浏览器的每一个部分==，我们操作这些BOM对象的时候就相当于操作了浏览器，从而可以控制浏览器的有些行为。

![1552052941943](前端-03-jquery高级.assets/1552052941943.png)

### BOM对象

| **BOM常用对象** | **作用**       |
| --------------- | -------------- |
| **window**      | 浏览器窗口     |
| **location**    | 访问的地址对象 |
| **history**     | 历史记录对象   |

### location对象

在面向对象程序中，学习对象的：方法，属性

属性可以设置set，也可以获取get。

| 属性     | 功能                                                         |
| -------- | ------------------------------------------------------------ |
| **href** | 默认属性<br />获取：得到当前访问的地址<br />设置：跳转到指定的地址 |
| search   | 获取?后面的参数                                              |

1. 省略href属性：如果只给location对象赋值，默认操作的是href属性
2. 与window对象的关系：location其实是window对象的一个属性

### location常用方法

| location的方法 | 描述                                       |
| -------------- | ------------------------------------------ |
| **reload()**   | 相当于浏览器上刷新按钮，重载加载当前页面。 |

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title>    <script>       /* location代表了浏览器的地址栏，操作location对象就相当于操作浏览器的地址栏       *       *   如何获取到location对象。       *        window.location , 而且window可以省略不写。       *       *    常用属性：       *            href : 获取与设置url地址栏       *            search 设置或获取 href 属性中跟在问号后面的部分。 参数.       *    常用的方法：       *            reload(); 重新加载（相当于刷新）       *       * */       //获取地址栏的url        function getUrl(){            alert("地址栏的url："+ location.href);        }       //设置地址栏的url       function setUrl(){           location.href="http://www.itcast.cn";       }       //获取参数        function getData(){            alert(location.search);        }        //刷新        function refresh(){            location.reload();        }    </script></head><body>    <input type="button" value="获取地址栏的url" onclick="getUrl()"/>    <input type="button" value="设置地址栏的url" onclick="setUrl()"/>    <input type="button" value="获取参数" onclick="getData()"/>    <input type="button" value="刷新" onclick="refresh()"/></body></html>
```

### 小结

1. location学习了哪些属性：

   - href 设置与获取地址栏
   - search 获取url参数

2. 学习了哪些方法

   - reload（） 

3. 它与window是什么关系?

   - location是属于window对象一个属性对象。




## BOM：history对象

### 目标

history对象有哪些方法和属性

### 作用

访问之前访问过的历史记录

Chrome：ctrl +shift +del 清除历史记录

| 方法      | 作用                                             |
| --------- | ------------------------------------------------ |
| forward() | 相当于前进，如果浏览器后退按钮不可用这个方法无效 |
| back()    | 相当于后退                                       |
| go(整数)  | 正整数：前进多个页面，负整数：后退多个页面       |

注：浏览器上的前进和后退按钮可以点的时候，这个代码才起作用。

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title>    <script>        function forwardTest(){            // history.forward();            history.go(1);        }    </script></head><body>    <h1>这个是1.html</h1>    <a href="2.html">去到2.html</a>    <input type="button" value="前进" onclick="forwardTest()"/></body></html>
```



2.html

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title>    <script>        function backTest(){          //  history.back();            history.go(-1);        }    </script></head><body>    <h1>这个是2.html</h1>    <input type="button" value="后退" onclick="backTest()"/></body></html>
```





### 小结

| **方法**      | **作用** |
| ------------- | -------- |
| **forward()** | 前进     |
| **back()**    | 后退     |



## 案例：倒计时页面跳转

### 目标

window对象与location对象的综合案例应用

### window中计时器有关的方法

| **window中的方法**                      | **作用**                                               |
| --------------------------------------- | ------------------------------------------------------ |
| **setTimeout(函数名,** **间隔毫秒数)**  | 过多久以后调用1次指定的函数，只调用一次                |
| **clearTimeout(计时器)**                | 清除上面的计时器                                       |
| **setInterval(函数名,** **间隔毫秒数)** | 每隔一段时间调用一次函数<br />返回值：整数类型的计时器 |
| **clearInterval(计时器)**               | 清除上面的计时器，让计时器失效。                       |



### 需求

页面上显示一个倒计时5秒的数字，到了5秒以后跳转到另一个页面

### 效果

![1552053355566](前端-03-jquery高级.assets/1552053355566.png) 

### 分析

1. 在页面上创建一个span用于放置变化的数字。
2. 定义一个全局变量为5，每过1秒调用1次自定义refresh()函数
3. 编写refresh()函数，修改span中的数字
4. 判断变量是否为0，如果是0则跳转到新的页面
5. 否则变量减1。并修改span中的数字

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>倒计时页面跳转</title></head><body><span id="timer">5</span>秒以后跳转<script type="text/javascript">        //1. 定义一个变量记录当前的倒计时时间    var time = 5;    //2. 定义一个方法去修改span标签里面的时间    function changeTime(){        time--;        //找到span标签        var timerSpan = document.getElementById("timer");        //修改span标签的标签体内容        timerSpan.innerHTML=time;        if(time==0){            //跳转到首页            location.href="http://www.itheima.com";        }    }    //3. 开启定时任务调用方法    window.setInterval(changeTime,1000);</script></body></html>
```

### 小结

1. 每过一段时间执行的方法是：setInterval
2. 跳转到另一个页面使用：location.href
3. 更新数字显示使用：innerHTML



## 案例：会动的时钟

### 目标

​	页面上有两个按钮，一个开始按钮，一个暂停按钮。点开始按钮时间开始走动，点暂停按钮，时间不动。再点开始按钮，时间继续走动。

![1552053500102](前端-03-jquery高级.assets/1552053500102.png)

### 步骤

1.	在页面上创建一个h1标签，用于显示时钟，设置颜色和大小。
2.	一开始暂停按钮不可用，设置disabled属性，disabled=true表示不可用。
3.	创建全局的变量，用于保存计时器
4.	为了防止多次点开始按钮出现bug，点开始按钮以后开始按钮不可用，暂停按钮可用。点暂停按钮以后，暂停按钮不可用，开始按钮可用。设置disabled=true。
5.	点开始按钮，在方法内部每过1秒中调用匿名函数，在匿名函数内部得到现在的时间，并将得到的时间显示在h1标签内部。
6.	暂停的按钮在方法内部清除clearInterval()的计时器。

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>会动的时钟</title></head><body><h1 style="color: darkgreen" id="clock">现在的时间</h1><hr/><input type="button" value="开始" id="btnStart"><input type="button" value="暂停" id="btnPause" disabled="disabled"><script type="text/javascript">    /*        window对象的定时任务的方法：               1， setInterval(方法，毫秒数) 每隔指定的毫秒数调用指定的方法,返回定时任务的id               2. clearInterval（timerId） 根据定时任务的id好取消定时任务。               3. setTimeOut(方法，毫秒数) 相隔指定的毫秒数调用指定的方法，但是由始至终只会调用一次。               4. clearTimeOut(timerId)  根据定时任务的id号去取消定时任务.     */    //1. 找到开始按钮与结束的按钮    var startBtn = document.getElementById("btnStart");    var pauseBtn = document.getElementById("btnPause");    //定义一个函数获取当前的系统时间，并且设置在    function showTime(){        //获取当前的系统时间        var curTime = new Date().toLocaleString();        //找到现实时间的标签        var clock = document.getElementById("clock");        //把时间显示在标签体中        clock.innerHTML = curTime;    }    var timerId = 0;    //点击开始按钮需要做什么呢？    startBtn.onclick =function(){        showTime();        //开启定时任务，调用showTime的方法        timerId = setInterval(showTime,1000); //开启定时任务的时候，会返回当前定时任务的id.        //设置开始按钮不可点击        startBtn.disabled ="disabled";        pauseBtn.disabled="";    }    //停止按钮的点击时间    pauseBtn.onclick = function(){        window.clearInterval(timerId);        startBtn.disabled ="";        pauseBtn.disabled="disabled";    }</script></body></html>
```

### 小结

1. **清除计时器的方法:** 

   ​	clearInterval(定时任务的id)

## value，innerHTML和innerText的区别

### 目标

学习innerHTML和innerText的区别

### 语法

这两个属性都是用来修改元素主体内容

| **元素的属性** | **作用**                                                     |
| -------------- | ------------------------------------------------------------ |
| **innerHTML**  | 获得：一个元素内部所有的标签和内容<br />设置：修改元素主体中HTML，如果内容中包含标签，标签也是起作用的 |
| **innerText**  | 获得：一个元素内部纯文本，不会包含任何的标签<br />设置：修改元素主体中纯文本，如果内容中包含了标签，标签不起作用。 |
| value          | 设置标签的value属性                                          |



### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title></head><body>    用户名：<input id="userName" type="text"/><br/>    性别:<span id="gender"> </span></body><script>    /* value :  操作标签的value属性值     innerHTML: 操作标签体的内容，可以是文本也可以是子标签     innerText: 操作标签体内容，只能是文本     */    document.getElementById("userName").value="呵呵";    //操作    document.getElementById("gender").innerText="<a href='#'>男</a>";</script></html>
```

### 小结

| **元素的属性** | **作用**                                                     |
| -------------- | ------------------------------------------------------------ |
| **innerHTML**  | 代表的是标签的标签体内容,不单止可以设置文本内容，而且还可以设置标签元素 |
| **innerText**  | 代表的是标签的标签体内容,只能设置文本内容，所有的数据都会转换为普通文本 |
| value          | 设置标签的value属性                                          |



## window对象：三个对话框方法

### 目标

有哪些弹出的对话框

### 语法

![1552053909245](前端-03-jquery高级.assets/1552053909245.png)

### confirm案例演示

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title>    <script>        //确认框        var flag = window.confirm("今晚约吗？");        if(flag){            document.write("老地方见...");        }else{            document.write("明晚约！");        }    </script></head><body></body></html>
```

### 小结

| **window中的方法**                                     | **说明**                                        |
| ------------------------------------------------------ | ----------------------------------------------- |
| **setInterval()<br />setTimeout()**                    | 每过一段时间调用一次函数<br />过一段时间执行1次 |
| **clearTimeout(timerId) <br />clearInterval(timerId)** | 清除上面的计时器                                |
| **alert()<br />prompt()<br />confirm()**               | 信息框<br />输入框<br />确认框                  |



## DOM查找元素的四个方法

### 目标

1. 什么是DOM编程
2. 学习通过DOM查找元素的方法

### 节点与DOM树

DOM：Document Object Model 文档对象模型，操作网页中各种元素，属性，文本等

DOM编程： 浏览器在解析html页面的时候，每遇到一个标签都会创建一个对应的对象进行描述，==浏览器显示出来的内容其实都是这些标签对象的属性信息==，DOM编程就是指我们要获取到这些标签对象即可操作这些对象的属性，从而改变浏览器的显示效果。 



![1602058294809](前端-03-jquery高级.assets/1602058294809.png)

### 查找节点的方法

| **获取元素的方法**                          | **作用**                 |
| ------------------------------------------- | ------------------------ |
| **document.getElementById("id")**           | 通过id得到一个元素       |
| **document.getElementsByTagName("标签名")** | 通过标签名得到一组元素   |
| **document.getElementsByName("name")**      | 通过name属性得到一组元素 |
| **document.getElementsByClassName("类名")** | 通过类名得到一组元素     |

### 案例：查找节点

#### 步骤

1. 点第1个按钮，给所有tr奇数行和偶数行添加不同的背景色
2. 点击第2个按钮，选中所有商品
3. 点击第3个按钮给所有的a链接添加href属性。

#### 效果

![1552054117143](前端-03-jquery高级.assets/1552054117143.png)

#### 代码

```html
<!DOCTYPE html><html><head>    <meta charset="UTF-8">    <title>根据标签的属性找元素</title>    <style>        table {            width: 350px;            border-collapse: collapse;        }        td {            text-align: center;            padding: 3px;        }    </style></head><body><input type="button" value="(通过标签名)给tr的奇数行添加背景色" onclick="changeBackground()"/><input type="button" value="(通过name属性)选中所有的商品" onclick="checkAll()"/><input type="button" value="(通过类名)给a添加链接" onclick="setURL()"/><hr/><table border="1">    <tr>        <td>100</td>        <td>100</td>        <td>100</td>    </tr>    <tr>        <td>200</td>        <td>200</td>        <td>200</td>    </tr>    <tr>        <td>300</td>        <td>300</td>        <td>300</td>    </tr>    <tr>        <td>400</td>        <td>400</td>        <td>400</td>    </tr></table><hr/><input type="checkbox" name="product"> 自行车<input type="checkbox" name="product"> 电视机<input type="checkbox" name="product"> 洗衣机<hr/><a class="company">传智播客</a><br/><a class="company">传智播客</a><br/><a class="company">传智播客</a><br/><script type="text/javascript">    /*        找标签对象的方法：            1.document.getElementById("id") 通过id得到一个元素 ，返回单个对象            2.document.getElementsByTagName("标签名") 通过标签名得到一组元素, 返回一个数组对象            3.document.getElementsByName("name") 通过name属性得到一组元素, 返回一个数组对象           4. document.getElementsByClassName("类名") 通过类名得到一组元素, 返回一个数组对象     */    function changeBackground(){        //找到所有的tr标签        var trs = document.getElementsByTagName("tr");//根据标签名找标签，不管找到几个都是返回一个数组对象        for(var i = 0 ; i<trs.length ; i++){            var tr = trs[i];            if(i%2!=0){                //奇数                tr.style.backgroundColor="green";            }else{                //偶数                tr.style.backgroundColor="yellow";            }        }    }    function checkAll(){        //根据name的属性值找到所有的复选框        var productList = document.getElementsByName("product");        for(var i = 0 ; i<productList.length ; i++) {            var product = productList[i];            product.checked=true;        }    }    function setURL(){        var companyList = document.getElementsByClassName("company")        for(var i = 0 ; i<companyList.length ; i++) {            var company = companyList[i];            company.href="http://www.itcast.cn";        }    }</script></body></html>
```

### 小结

| **获取元素的方法**                           | **作用**                     |
| -------------------------------------------- | ---------------------------- |
| **document.getElementById("id")**            | 根据id查找，返回的是单个对象 |
| **document.getElementsByTagName ("标签名")** | 根据标签名查找               |
| **document.getElementsByName("name")**       | 根据name属性值查找           |
| **document.getElementsByClassName("类名")**  | 根据class属性值查找          |



## DOM树修改的方法

### 目标

1. DOM树上如何创建节点
2. DOM树上如何添加元素节点
3. 删除DOM树上的元素

### 添加元素的步骤

1. 创建元素
2. 把这个元素添加到DOM树中

### 创建元素的方法

| **创建元素的方法**                   | **作用**         |
| ------------------------------------ | ---------------- |
| **document.createElement("标签名")** | 创建一个新的元素 |

### 修改DOM树的方法

| **将元素挂到DOM树上的方法**        | **作用**                     |
| ---------------------------------- | ---------------------------- |
| **父元素.appendChild(子元素)**     | 添加成父元素的最后一个子元素 |
| **直接父元素.removeChild(子元素)** | 删除父元素下一个子元素       |
| **元素.remove()**                  | 自杀，删除自己               |

### 设置属性

| 方法                    | 作用           |
| ----------------------- | -------------- |
| setAttribute(key,value) | 设置属性       |
| removeArribute(key)     | 删除指定的属性 |



### 通过关系找节点的属性

| **遍历节点的属性**  | **说明**               |
| ------------------- | ---------------------- |
| **childNodes**      | 得到当前元素所有子节点 |
| **firstChild**      | 得到第1个子节点        |
| **lastChild**       | 得到最后1个子节点      |
| **parentNode**      | 得到父元素             |
| **nextSibling**     | 得到下一个兄弟节点     |
| **previousSibling** | 得到上一个兄弟节点     |

### 小结

1. 创建元素：createElement
2. 添加成最后一个子元素：appendChild
3. 删除子元素：removeChild
4. 删除本身：remove



### 操作节点对象的方法

```java
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>操作节点对象的方法</title>    <script>        /*            修改与创建节点：                   document.createElement("标签名")  创建标签对象                   标签对象.setAttribute(属性名,属性值) 给标签设置属性值                   父元素.appendChild（子元素） 添加子节点a         */        var i = 1;        function createBtn(){            //创建标签元素            var inputNode = document.createElement("input");            //设置属性            inputNode.setAttribute("type","button");            inputNode.setAttribute("value","按钮"+i++ +"号");            //找到body标签            var body = document.getElementsByTagName("body")[0];            //给body标签添加一个子节点            body.appendChild(inputNode);        }    </script></head><body>    <input type="button" value="添加按钮" onclick="createBtn()" /></body></html>
```









## 案例：学生信息管理

### 目标

1.	当添加按钮“添加一行数据”时，文本框中的数据添加到表格中且文本框置空； 
2.	当点击表格中的“删除”时，该行数据被删除，删除前确认
3.	当点击表格第一行的复选框的时候，下面每一行都选中。当取消的时候，下面每一行都取消。

### 效果

![1552054419916](前端-03-jquery高级.assets/1552054419916.png)

### 步骤

1. 添加的实现：当点击按钮时，得到文本框中的文本，如果文本框为空，则提示错误。
2. 创建tr节点；把整个tr中的内容使用innerHTML直接设置到tr的内部。
3. 把tr追加到tbody元素中；注：tbody无论源代码中是否写了，在浏览器中始终存在。
4. 添加一行之后，清空学号与姓名的文本框内容
5. 删除操作：将当前点击按钮所在的一行tr，从tbody中删除。
6. 如何得到点击按钮所在的行? 先得到按钮的父节点td，td的父节点tr，然后删除。
7. 点击上面的复选框，则下面所有复选框的选择状态和上面的一样。

### 代码

```html
<!DOCTYPE html><html><head>    <meta charset="UTF-8">    <title>学生信息管理</title>    <style type="text/css">        table {            width: 500px;            border-collapse: collapse;        }        td {            text-align: center;        }        th {            background-color: lightgrey;        }    </style>    <script type="text/javascript">        /*            元素.remove() 自杀，自身删除自身            直接父元素.removeChild(子元素)  父节点删除子节点         */        //添加学生信息        function addRow(){            //1获取客户输入的内容（学号与姓名）            var name = document.getElementById("name").value;            var num = document.getElementById("num").value;            //2. 创建一行            var trNode = document.createElement("tr");            //3 定义一个字符串保存tr要保存的内容            var content = "<td>\n" +                "                <input type=\"checkbox\" name=\"item\">\n" +                "            </td>\n" +                "            <td>"+num+"</td>\n" +                "            <td>"+name+"</td>\n" +                "            <td>\n" +                "                <input type=\"button\" value=\"删除\" onclick=\"deleteRow(this)\">\n" +                "            </td>";            //4. 给tr设置标签体内容            trNode.innerHTML=content;            //5. 把行添加到表格中            var tbody = document.getElementsByTagName("tbody")[0];            tbody.appendChild(trNode);        }        //删除功能        function deleteRow(btnObj){            //通过按钮找到要删除的tr            var trNode = btnObj.parentNode.parentNode;            //删除            //trNode.remove(); //自杀            var tbody = document.getElementsByTagName("tbody")[0];            tbody.removeChild(trNode);        }        //全选与反选功能        function selectAll(checkAllNode){            //找到所有的复选框            var items = document.getElementsByName("item");            for(var i =0 ; i<items.length ; i++){                items[i].checked = checkAllNode.checked;            }        }    </script></head><body><div>    <table border="1" cellspacing="0" cellpadding="3">        <tr>            <th>                <input type="checkbox" id="all" onclick="selectAll(this)">            </th>            <th>学号</th>            <th>姓名</th>            <th>操作</th>        </tr>        <tr>            <td>                <input type="checkbox" name="item">            </td>            <td>00001</td>            <td>潘金莲</td>            <td>                <input type="button" value="删除" onclick="deleteRow(this)">            </td>        </tr>        <tr>            <td>                <input type="checkbox" name="item">            </td>            <td>00002</td>            <td>鲁智深</td>            <td>                <input type="button" value="删除" onclick="deleteRow(this)">            </td>        </tr>    </table>    <br />    学号：<input type="text" id="num"  />    姓名：<input type="text" id="name"  />    <input type="button" id="addBtn" value="添加" onclick="addRow()"/></div></body></html>
```

### 小结

- document.createElement(标签名）  创建对象
- setArrtibute(key,value)  设置属性
- removeAttribte(key)  删除属性   
- 父元素,appendChild(子元素)   爸爸添加元素
- 直接父元素.removeChild(子元素)   爸爸删除你
- 元素.remove()  自杀
- parentNode  找父节点



