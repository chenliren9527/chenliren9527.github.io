---
title: 前端(04)-jquery
tags:
  - 笔记
  - 前端
  - jquery
categories:
  - 前端
date: 2020-10-18 12:45:34
---

## 01.学习目标


### 学习目标

1)       能够引入jQuery

2)       能够使用jQuery的基本选择器

<font color=red>3) 	      能够使用jQuery的层级选择器</font>

<font color=red>4) 	能够使用jQuery的属性选择器</font>

<font color=red>5) 	      能够使用jQuery的基本过滤选择器</font>

6)       能够使用jQuery的属性过滤选择器

<font color=red>7) 	      能够使用jQuery的DOM操作的方法</font>

<font color=red>8) 	      能够完成隔行换色案例</font>

<font color=red>9) 	      能够完成购物车的案例</font>

## 02.jquery框架下载与导入【理解】

### 目标

掌握jquery的概念与导入使用



### jquery介绍

jquery是js的框架，==提高js的DOM开发效率,简化开发==。在以后企业中如果遇到传统项目前端框架就会使用jquery。

jquery框架宗旨：write less do more

jquery框架由于是最早的js框架，所以发展最为成熟，有丰富的jquery的插件。

jquery插件：是对一个功能独立的封装，可以在任意项目中使用：比如：分页插件，二维码插件，轮播图插件



### 使用实现步骤

1. 框架的下载： http://www.jquery.com/

2. 本地下载， 素材中提供的

   ![image-20200718090124980](前端-04-jquery.assets/image-20200718090124980.png)

    jquery就是对js的代码封装，所以jquery的库就是js文件

   版本的区别：

   版本号： 1.x.x 版本支持IE9以下的浏览器

   ​                 2.x.x/3.x.x 版本支持IE9和IE9以上的浏览器 【推荐使用，功能最全】

   开发版与生产版本的：

   ​			xxx.js 开发版本： 体积大，里面有注释，换行等

   ​			xxx.min.js 生产版本（压缩版）【推荐使用】： 体积小，里面没有注释和换行，用于生产环境

   总结：推荐使用3.x.x的生成版本

   

3. 创建项目和网页

   ![image-20200718091255181](前端-04-jquery.assets/image-20200718091255181.png)

   导入jquery到项目中

   ![image-20200718091423290](前端-04-jquery.assets/image-20200718091423290.png) 

4. 在网页上引用jquery的js文件， 这样网页才可以使用jquery的代码

### 实现代码

html页面上导入js文件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script type="text/javascript" src="../js/jquery-3.3.1.min.js"></script>
    <script>
        //原生js的写法
     /*   window.onload=function(){
            var content = document.getElementById("username").value;
            alert("获取到内容"+content);
        }*/


        //jq写法
        $(function(){
            var content = $("#username").val();
            alert("获取到内容： "+content);
        });



    </script>
</head>
<body>
    <input type="text" id="username" value="张三"/>
</body>
</html>
```

### 运行效果

![image-20200718091800758](前端-04-jquery.assets/image-20200718091800758.png)

### jquery常见错误

错误描述： 经常开发人员忘记导入jquery的js文件，导致jquery代码没有效果

代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script>
        /*编写jquery的代码，进行体验*/

        //jq的页面加载完成事件
        $(function () {
            alert("hello jquery");
        })
    </script>
</head>
<body>

</body>
</html>
```

运行访问

![image-20200718092142297](前端-04-jquery.assets/image-20200718092142297.png)



### 小结

* **jquery是什么**

  - jquery是js的框架(js的工具)

* **jquery以后企业中开发使用开发版本还是生产版本（压缩版）？**

  - 生产版本

* **jquery的使用步骤?**

  - 把jquery文件导入项目中

  - 在页面上引入jquery的文件

  - 直接使用

    



## 03.jq与js区别：加载事件【理解】

### 目标

1. 掌握jquery加载完成事件的语法

2. 掌握jquery与js的加载完成事件的区别



### 实现代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script type="text/javascript" src="../js/jquery-3.3.1.min.js"></script>
    <script>
        //js的加载事件
        /*window.onload = function(){
            alert("呵呵");
        }

        window.onload = function(){
            alert("嘻嘻");
        }

*/

        //jq的事件加载
     /*
       jq事件加载的格式：
             $(function(){

                })
        */

      $(function(){
          alert("呵呵");
      })

        $(function(){
            alert("嘻嘻");
        })


    //js事件加载与jq的事件加载区别： jq事件加载允许多个，js只能有一个。 


    </script>
</head>
<body>

</body>
</html>
```

### 小结

1. **jquery加载完成事件的语法**

   $(function(){

   });




## 04.jq与js区别：对象与事件【理解】

### 目标

掌握获取jquery对象的方式

掌握jquery对象注册事件方式



### 讲解代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>js和jq区别</title>
    <script type="text/javascript" src="../js/jquery-3.3.1.min.js"></script>
</head>
<body>
<input type="button" id="b1" value="我是JS对象">
<input type="button" id="b2" value="我是JQ对象">
</body>
<script type="text/javascript">

    /*
    * 获取对象语法不同：
    *   js对象获取的语法： document.getElement系列方法获取
    *   jq对象获取的语法： $("选择器") ， 选择器类似css的选择器
    * 注册事件语法不同：
    *   js对象注册点击事件语法： js对象.onclick = function(){...};
    *   jq对象注册点击事件语法： jq对象.click(function(){...});
    * */

    //使用js对象注册点击事件并弹出value属性值

    //找到按钮
    var btn1 = document.getElementById("b1");

    //给按钮注册事件
    btn1.onclick = function(){
        alert(this.value);
    }


    //使用jq对象注册点击事件并弹出value属性值

    //找到按钮
    var btn2 = $("#b2");

    //给按钮注册事件
    btn2.click(function(){
       alert(this.value);
    });

</script>

```

### 运行效果

![1560476724917](前端-04-jquery.assets/1560476724917.png)

### 小结

* 获取jquery对象的方式

  ​	$("选择器")

  

* jquery对象注册事件方式

  jq对象.事件名(function(){});

  


## 05.jq与js区别：jq与js对象的转换【理解】

### 疑问

* 以后我们是使用js对象还是jq对象操作dom？

  ```
  尽量使用jq对象，因为jq对象提供了大量丰富成员操作DOM，提高开发效率
  ```

### 目标

掌握js对象转换为jq对象



### 讲解代码

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
    <script src="js/jquery-3.3.1.min.js"></script>
    <script>
        $(function() {
            var b1 = document.getElementById("b1");
            b1.onclick = function () {
                //获取用户名的input标签对象，由于使用的是jq方法获取，这里是jq丢向
                var inputObj = $("#username");
                //jq对象转换为js对象
                alert("js对象获取数据：" + inputObj[0].value);
                alert("jq对象获取数据：" + inputObj.val());

            }
        })

    </script>
</head>
<body>
    <input type="button" id="b1" value="我是JS对象">
    <input type="text" id="username" value="admin"/>
</body>
</html>

```



### 运行效果

!![1602686829568](前端-04-jquery.assets/1602686829568.png)

### 小结

**为什么学习js对象转换为jq对象**

​	- js与jq 的方法是不通用的

**js对象转换为jq对象**

​	- $(js对象)




## 06.选择器：基本选择器【应用】

### 疑问

1. jq具体简化了js什么操作？

   > DOM操作，对标签元素的增删改查

2. 增、删、改、查先学习那个？

   > 查

### 目标

掌握基本选择器查询jq对象



### 基本选择器语法

| **基本选择器** | **语法**    |
| -------------- | ----------- |
| **ID**选择器   | $("#ID")    |
| **类选择器**   | $(".类名")  |
| **标签选择器** | $("标签名") |

### 实现代码

```html
<script type="text/javascript">
		// jquery有一个修改样式的方法： css(样式名，样式值)
        // 1) 改变 id 为one的元素的背景色为红色
		$("#b1").click(function(){
			//id选择器
			$("#one").css("background-color","red");
		});

        // 2) 改变元素名为 <div> 的所有元素的背景色为 红色。
        $("#b2").click(function(){
			//标签选择器
			$("div").css("background-color","red");
        });

        // 3) 改变 class 为 mini 的所有元素的背景色为 红色
        $("#b3").click(function(){
			//类选择器
			$(".mini").css("background-color","red");
        });
    </script>
```

### 小结

基本选择器有哪些?

>  $("#id属性值")   id
>
>  $(".class属性值") 类
>
>  $("标签名") 标签



## 07.选择器：层级选择器【应用】

### 目标

掌握成绩选择器



### 层级选择器语法

![1568773034920](前端-04-jquery.assets/1568773034920.png)

### 实现代码

```html
<script type="text/javascript">
    //1) 改变 <body> 内所有 <div> 的背景色为红色
    $("#b1").click(function(){
        //找到body下面的所有div
        $("body div").css("background-color","red");
    });

    //2) 改变 <body> 内子 <div> 的背景色为 红色
    $("#b2").click(function(){
        $("body>div").css("background-color","red");
    });

    //3) 改变 id为two的下一个div兄弟元素的背景色为红色
    $("#b3").click(function(){
        $("#two+div").css("background-color","red");
    });

</script>
```

## 08.选择器：属性选择器【应用】

### 目标

掌握属性选择器的使用



### 属性选择器语法

![1546828424868](前端-04-jquery.assets/1546828424868.png)

### 实现代码

```html
	<script type="text/javascript">        //1) 含有属性title 的div元素背景色为红色		$("#b1").click(function(){			$("div[title]").css("background-color","red");		});        //2) 属性title值等于test的div元素背景色为红色        $("#b2").click(function(){            $("div[title=test]").css("background-color","red");        });        //3) 属性title值不等于test的div元素(没有title属性的也将被选中)背景色为红色        $("#b3").click(function(){            $("div[title!=test]").css("background-color","red");        });        //4) 选取有属性id的div元素，然后在结果中选取属性title值等于“test”的 div 元素背景色为红色        $("#b4").click(function(){            $("div[id][title=test]").css("background-color","red");        });	</script>
```

### 小结

属性选择器有哪些?

> [属性名]
>
> [属性名=属性值]
>
> [属性名!=属性值]
>
> [属性名\]\[属性名=属性值\]
>
> 
>
> 



## 09.选择器：基本过滤选择器【应用】

### 目标

掌握基本过滤器的使用



### 基本过滤选择器语法

![1546828777714](前端-04-jquery.assets/1546828777714.png)

### 实现代码

```html
<script type="text/javascript">	//1) 改变第一行的背景色为浅灰色	$("#b1").click(function(){		$("tr:first").css("background-color","gray");	})	//2) 改变最后一行的背景色为lightgreen浅绿色    $("#b2").click(function(){        $("tr:last").css("background-color","lightgreen");    })	//3) 改变除第1行和最后1行的其它行背景色为lightyellow浅黄色    $("#b3").click(function(){        $("tr:not(:first):not(:last)").css("background-color","lightyellow");    })	//4) 改变索引值为偶数的行背景色为lightpink浅粉色，下标从0开始    $("#b4").click(function(){        $("tr:even").css("background-color","lightpink");    })	//5) 改变索引值为奇数的行背景色为aquamarine深蓝色    $("#b5").click(function(){        $("tr:odd").css("background-color","aquamarine");    })	//6) 改变索引值大于 3 的tr元素的背景色为oldlace浅米色    $("#b6").click(function(){        $("tr:gt(3)").css("background-color","oldlace");    })	//7) 改变索引值等于 3 的tr元素的背景色为antiquewhite古董白    $("#b7").click(function(){        $("tr:eq(3)").css("background-color","antiquewhite");    })	//8) 改变索引值为小于 3 的tr元素背景色为yellowgreen黄绿色    $("#b8").click(function(){        $("tr:lt(3)").css("background-color","yellowgreen");    })	</script>
```

### 按钮1到按钮3的效果

![1566528751822](前端-04-jquery.assets/1566528751822.png)

### 按钮1到按钮5的效果

![1566528778361](前端-04-jquery.assets/1566528778361.png)

### 按钮6到按钮8效果

![1566528800739](前端-04-jquery.assets/1566528800739.png)





## 10.选择器：表单属性过滤选择器【应用】

### 目标

掌握表单属性过滤器的使用



### 表单属性过滤选择器语法

![1556496581195](前端-04-jquery.assets/1556496581195.png)

### 实现代码

```html
	<script type="text/javascript">        //1) val() 方法改变表单内可用文本框 <input> 元素的值		$("#b1").click(function(){			$("input[type=text]:enabled").val("呵呵");		});        //2) val() 方法改变表单内不可用 <input> 元素的值        $("#b2").click(function(){            $("input[type=text]:disabled").val("哈哈，二追三");        });        //3) length 属性获取单选框与复选框选中的个数        $("#b3").click(function(){			var nums = $("input:checked").length;			alert("选中个数："+nums);        });        //4) text 属性获取下拉列表选中的内容        $("#b4").click(function(){            var options = $(":selected");			for(var i = 0; i<options.length ; i++){			    var option = options[i]; //option目前是js对象			    alert(option.value);			}        });	</script>
```



## 11.选择器案例：表格隔行换色【应用】

### 需求

![1549980909856](前端-04-jquery.assets/1549980909856.png)

页面加载后偶数行为粉红色，奇数行为黄色

### 实现效果

![1546831197598](前端-04-jquery.assets/1546831197598.png)

### 实现代码

```html
<script type="text/javascript" src="../js/jquery-3.3.1.min.js"></script>	<script>		$("tr:gt(1):even").css("background-color","pink");		$("tr:gt(1):odd").css("background-color","yellow");	</script>
```



## 12.DOM操作：数据【应用】

### 目标

掌握标签体数据的操作

掌握表单元素值的操作



### 语法

标签存放数据的位置：

​	1.非表单标签：标签体的数据

​     2.表单标签：value属性的数据



![1546832019929](前端-04-jquery.assets/1546832019929.png)

### 实现代码

```html
  <script type="text/javascript">		   /*		   	val() 操作标签的value属性值		   	html()\text() 操作标签体内容		    */		// 设置myinput的value属性值		 $("#b1").click(function(){			$("#myinput").val("李四");		 });		// 获取myinput的value属性值并弹框显示        $("#b2").click(function(){            //获取			alert($("#myinput").val());        });		//设置mydiv标签体html代码'<h2>itheima</h2>'        $("#b3").click(function(){			$("#mydiv").html("<h2>itheima</h2>");        });		//获取mydiv标签体html代码并弹框显示        $("#b4").click(function(){			alert($("#mydiv").html());        });		//设置mydiv标签体text文本内容为'<h2>itheima</h2>'        $("#b5").click(function(){            $("#mydiv").text("<h2>itheima</h2>"); //使用text方法的时候，里面所有内容都会变成普通文本        });		//获取mydiv标签体text文本内容并弹框显示        $("#b6").click(function(){            alert($("#mydiv").text());        });	   </script>
```

### 小结

标签体数据的操作

> html()||text()

表单元素值的操作

> val()

## 13.DOM操作：属性【应用】

### 目标

掌握标签属性数据的操作



### 语法

![1546832444911](前端-04-jquery.assets/1546832444911.png)

### 实现代码

```html
<script type="text/javascript">			//获取北京节点的name属性值		$("#b1").click(function(){			alert($("#bj").attr("name"));		})		//设置北京节点的name属性的值为dabeijing        $("#b2").click(function(){            $("#bj").attr("name","dabeijing");        })		//新增北京节点的description属性 属性值是didu        $("#b3").click(function(){            //如果没有该属性的时候就是新增            $("#bj").attr("description","didu");        })		//删除北京节点的name属性        $("#b4").click(function(){			$("#bj").removeAttr("name");        })		//获得hobby的选中状态        $("#b5").click(function(){			//alert("被选中了吗？"+$("#hobby").attr("checked"));			//如果操作的属性是一个boolean类型，那么建议使用prop方法操作			alert("被选中了吗？"+$("#hobby").prop("checked"));        })	</script>
```

### 小结

标签属性数据的操作

> attr(属性名[,属性值])/removeAttr(属性名) : 适合操作大多数属性
>
> prop(属性名[,属性值])/removeProp(属性名)： 适合操作boolean类型属性



## 14.DOM操作：样式【应用】

### 目标

掌握使用jq操作标签样式



### dom样式操作方式

html标签设置样式是通过class属性与style属性完成的

```html
<标签 class="类样式1 类样式2 ..." style=""></标签>
```

### jquery操作dom样式的语法

class的操作

![1546833089770](前端-04-jquery.assets/1546833089770.png)

 style的样式操作

![1546833109218](前端-04-jquery.assets/1546833109218.png)

### 实现代码

```html
<script type="text/javascript">			// 采用属性attr设置类样式(改变id=one的类样式)，类样式名为second		$("#b1").click(function(){			$("#one").attr("class","second");		});				// 使用addClass方式增加类样式        $("#b2").click(function(){            $("#one").addClass("second");        });        		// 使用removeClass方式删除类样式        $("#b3").click(function(){            $("#one").removeClass("second");        });		// 切换样式        $("#b4").click(function(){            $("#one").toggleClass("second");        });		// 通过css()获得id为one背景颜色        $("#b5").click(function(){           alert($("#one").css("background-color"));        }); 		// 通过css()设置id为one背景颜色为绿色        $("#b6").click(function(){            $("#one").css("background-color","green");        });	</script>   
```

### 小结

使用jq操作标签样式

> attr(class,类选择器名字）
>
> addClass()
>
> removeClass
>
> ToggleClass()切换
>
> css()



## 15.DOM操作：节点创建-修改【应用】

### 目标

掌握标签元素的创建、移动操作

### 语法

![image-20200717100058161](前端-04-jquery.assets/image-20200717100058161.png)

### 实现代码

```html
	<script type="text/javascript">		//将反恐放置到city的后面		$("#b1").click(function(){            var fk = $("#fk").clone(); //克隆			$("#city").append(fk);		});				//将反恐放置到city的最前面        $("#b2").click(function(){            $("#city").prepend($("#fk"));        });				//将反恐放在天津前面        $("#b3").click(function(){			$("#tj").before($("#fk"));        });				//将反恐放在天津后面        $("#b4").click(function(){            $("#tj").after($("#fk"));        });				//创建一个广州的节点，加到城市中：<li id="gz" name="guangzhou">广州</li>        $("#b5").click(function(){			//创建节点：			var gz =  $("<li id=\"gz\" name=\"guangzhou\">广州</li>");			$("#city").append(gz);        });	</script>
```

### 小结

标签元素的创建、移动操作

> append()    添加到最后一个子节点
>
> prepend()   添加到第一个子节点
>
> before()     前面
>
> after()   后面
>
> $("标签的字符串“）  创建一个节点



## 16.DOM操作：节点删除【应用】

### 目标

掌握元素对象的自杀与他杀



### 语法

![1546834581936](前端-04-jquery.assets/1546834581936.png)

### 实现代码

```html
<script type="text/javascript">	   //从city中删除<li id="bj" name="beijing">北京</li>节点	   $("#b1").click(function(){			$("#bj").remove();	   })	   	   //删除city中所有的子节点，观察city本身有没有删除       $("#b2").click(function(){			$("#city").empty();       })</script>   
```

### 小结

元素对象的自杀与他杀的语法

> 自杀， jq对象.remove()
>
> 他杀， jq对象.empty();



## 17.DOM案例：QQ表情【应用】

### 目标

掌握标签的克隆与应用场景



### 需求效果

![1560487113268](前端-04-jquery.assets/1560487113268.png)

### 代码实现

```html
<script>    // 找到li下面的所有img图片,给img图片注册点击事件    $("li img").click(function(){       var img =  $(this).clone(); //clone方法是jq的方法。        $(".word").append(img);    });</script>
```



### 小结

标签的克隆与应用场景

> clone()



## 18.循环遍历【应用】

### 目标

掌握不同循环方式的使用



### 修改idea的js校验规范

效果

![1568790743265](前端-04-jquery.assets/1568790743265.png)

### 修改后

![1566544763100](前端-04-jquery.assets/1566544763100.png)

修改后的效果，就不报错了

![1568790822499](前端-04-jquery.assets/1568790822499.png)

### 实现代码

<font color=red>注意：素材中提供的引用jquery的js文件路径是错误的，需要添加“../”</font>

```html
<script type="text/javascript">    /*    * 一共有常见4种循环方式：    * 1.js原始fori循环    * 2.ES6标准的for..of循环【推荐】    * 3.jquery对象的each方法循环    * 4.jquery全局的each方法循环    * */    // 最原始方式fori    $("#b1").click(function(){        var options = $("option");        for(var i =0;i<options.length ; i++){            var option = options[i];//遍历出来的对象是js对象            alert($(option).text());        }    });    // es6支持新语法 for 。of    $("#b2").click(function(){        var options = $("option");        for(var option of options){            alert($(option).text());        }    });    // jq的方法  each    $("#b3").click(function(){        var options = $("option");        //i：遍历过程中的索引值        //option: 遍历出来元素        options.each(function(i,option){            alert("索引值："+i+" 值："+ $(option).text());        });    });    // jq的方法   全局方法each    $("#b4").click(function(){        var options = $("option");        $.each(options,function(i,option){            alert("索引值："+i+" 值："+ $(option).text());        });    });</script>
```



### 效果

![image-20191230161253753](前端-04-jquery.assets/image-20191230161253753.png)



### 小结

不同循环方式的使用

> fori
>
> for-of
>
> each
>
> each



## 19.事件：事件介绍【应用】

### 目标

掌握常用事件有哪些

掌握jquery注册事件的链式写法



### 事件介绍

![image-20200717101729434](前端-04-jquery.assets/image-20200717101729434.png)

### 代码讲解

```html
<script type="text/javascript">        $(function () {           /* //得到焦点改变背景色pink		    $("#t1").focus(function(){		       $(this).css("background-color","pink");            });            //失去焦点改变背景色            $("#t1").blur(function(){                $(this).css("background-color","");            });            //松开键，把英文改成大写            $("#t1").keyup(function(){                //获取内容                var content = $(this).val();                //大写                content = content.toUpperCase();                //重新设置                $(this).val(content)            });*/            //链式写法            $("#t1").focus(function(){                $(this).css("background-color","pink");            }).blur(function(){                $(this).css("background-color","");            }).keyup(function(){                //获取内容                var content = $(this).val();                //大写                content = content.toUpperCase();                //重新设置                $(this).val(content)            })        })    </script>
```

### 效果

![1560488254332](前端-04-jquery.assets/1560488254332.png)

### 小结

常用事件有哪些

> click   单击
>
> focus  获取焦点
>
> blur  失去焦点
>
> mouseover  鼠标经过
>
> keyup  键盘松开

jquery注册事件的链式写法

> jq对象.事件名（）.事件名()。。。



## 20.事件：on绑定与off解绑【理解】

### 目标

使用on绑定事件

使用off解绑事件



### 使用on绑定事件的语法

![image-20200212221010950](前端-04-jquery.assets/image-20200212221010950.png)

### 使用off解绑事件

![image-20200212221030406](前端-04-jquery.assets/image-20200212221030406.png)

### 实现代码

```html
<script type="text/javascript">    //注意：如果直接在标签中使用了on事件名这种静态的注册方式，off方法是没法解绑的，如果真的需要解绑，只能删除onxx这个属性。    function  test() {        alert("按钮3");    }    //给按钮1使用事件名方式注册点击事件，弹出“按钮1”    $("#b1").click(function(){        alert($(this).val());    })    //使用on注册事件方式给b2注册点击事件，弹出“按钮2”    $("#b2").on("click",function(){        alert($(this).val());    });    //给unbind注册点击事件实现给b2解绑事件    $("#unbind").click(function(){        $("#b2").off();//解绑所有        $("#b1").off();//解绑所有        $("#b3").removeAttr("onclick");    });</script>
```

### 效果

![1566547547269](前端-04-jquery.assets/1566547547269.png)



### 小结

on绑定事件

> on(事件名，function(){})

off解绑事件

> jq对象.off(事件名)
>
> 



## 21.事件：on动态元素绑定【应用】

### 目标

使用on给动态添加元素绑定事件



##### on可以给已有和动态创建元素绑定事件，语法如下

![image-20200212221636402](前端-04-jquery.assets/image-20200212221636402.png)

### 效果

![image-20200212221607012](前端-04-jquery.assets/image-20200212221607012.png)

### 实现代码

```html
<script type="text/javascript">    //给每个li添加点击事件，弹出自己的文本内容    //注意： 这种注册方法只能针对现有元素进行事件的注册，对于未来新增的元素是不起作用。/*    $("li").click(function(){        alert($(this).text());    });*/    //动态注册：对未来新增的元素也是起作用的。    $("#city").on("click","li",function(){        alert($(this).text());    })    //点击按钮事件，动态添加li    $("#b1").click(function(){        var city = $("#t1").val();        var item = $("<li>"+city+"</li>");        $("#city").append(item);        $("#t1").val("");    });</script>
```

### 小结

1. 给页面已有元素注册事件方式有哪些？

   >  j

2. 给页面已有和新增内容都注册事件的方式？

   > 



## 22.DOM案例-购物车【应用】

### 需求

1) 实现添加商品，如果商品名为空，提示：商品名不能为空。如果不为空，则添加成功一行，清空文本框的内容，图片使用固定一张。

2) 实现删除一行商品的功能，删除前弹出确认对话框。

3) 实现全选/全不选的效果

### 效果

![1546834761360](前端-04-jquery.assets/1546834761360.png)

### 实现代码

```html
<script type="text/javascript" src="../js/jquery-3.3.1.min.js" ></script><script>    //添加商品    function addRow(){        //1. 得到商品的名字        var pname = $("#pname").val();        //2. 创建tr        var tr = $("<tr>\n" +            "            <td>\n" +            "                <input type=\"checkbox\" name=\"item\" />\n" +            "            </td>\n" +            "            <td><img src=\"img/sx.jpg\" /></td>\n" +            "            <td>\n" +            "               "+pname+"\n" +            "            </td>\n" +            "            <td>\n" +            "                <a href=\"javascript:void(0)\" onclick=\"deleteRow(this)\">删除</a>\n" +            "            </td>\n" +            "        </tr>");        //3. 把tr添加到tbody中        $("#cart").append(tr);    }    //删除    function deleteRow(aNode){        //parent() 获取父节点的方法       var trNode =  $(aNode).parent().parent();        //自杀        trNode.remove();    }    //全选与反选功能    function selectAll(allNode){        var flag = $(allNode).prop("checked"); //全选按钮的checked状态。        $("input[name=item]").prop("checked",flag); //商品的复选的checked状态    }</script>
```



