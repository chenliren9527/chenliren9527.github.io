---
title: 前端(02)-CSS&JavaScript基础
tags:
  - 笔记
  - 前端
  - CSS
  - JavaScript
categories:
  - 前端
date: 2020-09-30 12:47:29
---



# 学习目标

1、能够使用CSS的基本选择器选择元素 

​	      标签

​        类

​      id

​    通用

**2、能够使用CSS的扩展选择器选择元素** 

​		父选择器  子选择器{}

**3、能够使用常见的CSS属性** 

**4、能够说出盒子模型的属性** 

​        border

​        padding 

​        margin

**5、能够制作黑马旅游网的注册页面** 

​		可以

**6、能够说出五种原始的数据类型** 

​		number string boolean object undefined 

**7、能够使用JS中常用的运算符** 

​		可以

**8、能够使用JS中的流程控制语句** 

​		可以





# 学习内容

## 01、什么是CSS

### 目标

1. 什么是CSS
2. CSS有什么用

### 概念

CSS：层叠样式表 Cascading Style Sheet 用于HTML的==美化操作==，因为CSS的出现，让HTML的一些与美化有关的标签被淘汰了。如：font center

![1562743701250](前端-02-CSS-JavaScript基础.assets/1562743701250.png)



### 案例：CSS的作用

​	将一个表格中所有的单元格居中，如果使用以前的方式，每个td或tr都要设置align属性为center，而使用css则方便得多。

### 效果

![1551771191107](前端-02-CSS-JavaScript基础.assets/1551771191107.png)

### 代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        /*样式*/
        tr{
            text-align: center;
            color: blue;
        }

    </style>

</head>
<body>
<h2 align="center">学生成绩表</h2>
<table border="1" width="500" align="center">
    <tr>
        <td>11</td>
        <td>111</td>
        <td>111</td>
    </tr>
    <tr>
        <td>111</td>
        <td>111</td>
        <td>111</td>
    </tr>
    <tr>
        <td>111</td>
        <td>111</td>
        <td>111</td>
    </tr>
</table>
</body>
</html>
```

### 小结

| **CSS的好处** | **描述**                                |
| ------------- | --------------------------------------- |
| **功能上**    | 美化功能更加专业,更加强大               |
| **耦合性**    | html代码与css代码分离开，降低代码耦合度 |



## 02、CSS在网页中的3种位置

### 目标

1. CSS的规则
2. CSS出现的三种位置

### CSS出现的三种位置

1. 行内样式：出现在标签的头部，以style属性存在
2. 内部样式：出现在HTML文件内，以style标签的方式存在
3. 外部样式：出现在HTML文件外部，以单独的CSS文件存在

### CSS语法规则

![1551771303307](前端-02-CSS-JavaScript基础.assets/1551771303307.png)

| **规范**     | **说明**                                                     |
| ------------ | ------------------------------------------------------------ |
| **大括号**   | 所有的样式要写在2个大括号之间                                |
| **样式名**   | 左边是样式名字，取值是固定的，由CSS规定的。<br />所有的单词小写，如果有多个单词，使用短横分隔 |
| **样式值**   | 样式名与样式值之间用冒号隔开<br />每个样式名后面的样式值也是固定的 |
| **样式结尾** | 每一行样式以分号结尾                                         |
| **注释**     | 与Java多行注释一样：  /*  */                                 |

### 行内样式

![1551771360803](前端-02-CSS-JavaScript基础.assets/1551771360803.png) 

特点：出现在标签的头部，以style属性存在，只对这一个标签起作用。



### 内部样式

![1551771385547](前端-02-CSS-JavaScript基础.assets/1551771385547.png) 

特点：以style标签的方式存在HTML的内部，对整个HTML中的元素起作用。

### 外部样式

特点：以CSS文件的方式存在外部，使用的时候必须要导入到HTML中。它可以对多个HTML文件起作用。

![1551771443994](前端-02-CSS-JavaScript基础.assets/1551771443994.png)

### 三种样式的代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>编写css样式的位置</title>
<!--
    1. 行内样式：出现在开始标签，以style属性存在
        例子：
            <a href="#" style="color: black;text-decoration: none;">传智播客</a><br/>

        弊端： 样式复用性极差.

    2. 内部样式：出现在HTML文件内，以style标签的方式存在

    例子：
          <style>
                a{
                    color: black;
                    text-decoration: none;
                }
            </style>
    弊端： 复用性还不够好，html代码与css代码还存在着耦合性。


    3. 外部样式：出现在HTML文件外部，以单独的CSS文件存在（推荐使用）

     例子：  <link href="css/1.css" rel="stylesheet"/>
-->

    <!--注意： 使用link标签引入css文件一定要指定该文件与html页面的关系。  -->
    <link href="css/1.css" rel="stylesheet"/>




</head>
<body>
<a href="#">传智播客</a><br/>
<a href="#" >黑马程序员</a><br/>
</body>
</html>
```

### 小结

1. **行内样式有什么特点？**

   ​			只能对当前标签起作用。 

2. **内部样式有什么特点？**

   ​		 只能作用于当前页面

3. **外部样式有什么特点？**

   ​		可以作用于所有页面


## <font color="red">03 、CSS基本选择器</font>

### 目标

学习CSS的四种基本选择器

### 选择器作用

如果要给网页中某个元素(标签)设置样式，首先必须先选择元素。选择器的作用就是用来选择元素的（==选择器的作用就是找到指定的元素==）。

### 语法格式

```css
选择器 {
  样式名:样式值;
}
```

![1551771636816](前端-02-CSS-JavaScript基础.assets/1551771636816.png)

### 选择器之间优先级

```
通用选择器 < 标签选择器 < 类选择器 < ID选择器
```

### 案例演示

#### 需求

通过代码演示上面CSS的导入，各种基本选择器的使用

#### 效果

 

#### 代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>基本选择器</title>
    <style>

        /*   选择器的作用：查找html的元素。
         基本选择器:
           1. 标签选择器:   对选中的标签都起作用
                  格式：
                       标签名{
                          样式名1：样式值1;
                          样式名2：样式值2;
                       }

           2. 类选择器： 对选中的class标签都起作用, 使用前提，标签必须有一个class的属性值
                   格式：
                      .class属性值{
                           样式名1：样式值1;
                          样式名2：样式值2;
                      }
               注意： class的属性值一定不能以数字开头。

           3. ID选择器： 对选中的id值的标签起作用， 使用前提：html的元素必须有一个id的属性值
                   格式：
                          #ID属性值{
                               样式名1：样式值1;
                               样式名2：样式值2;
                          }

                 注意： 使用id选择器的时候，建议id值在当前页面要唯一。

          4. 通用选择器: 对所有的标签都起作用

               格式：
                          *{
                               样式名1：样式值1;
                               样式名2：样式值2;
                          }

           选择器的优先级： id选择器--类选择器---标签选择器---通用选择器
     */

        /*标签选择器*/
        p{
            color: red;
        }


        /*类选择器*/
        .one{
            color: chartreuse;
        }


        /*id选择器 */
        #two{
            color: blue;
        }


        /*通用选择器*/
        *{
            color: orange;
        }


    </style>

</head>
<body>
<p>
    11111
</p>
<p id="two" class="one">
    我们是害虫
</p>
<!--首先使用class属性进行分类-->
<p  class="one">
    33333
</p>
<h2 class="one">我是标题</h2>

<span>aaa</span>


</body>
</html>
```

### 小结

| 选择器名 | 格式            |
| -------- | --------------- |
| 通用     | *{}             |
| 标签     | 标签名{ }       |
| 类       | .class属性值{ } |
| ID       | #id属性值{ }    |



## 04、CSS扩展选择器：层级、属性选择器

### 目标

1. 层级选择器
2. 属性选择器

### 什么是扩展选择器

因为基本选择器选择的方式不够灵活，扩展选择器是在基本选择器的组合，形成更多，更灵活的选择方式。

### 常用的扩展选择器

![1551944648449](前端-02-CSS-JavaScript基础.assets/1551944648449.png)

### 扩展选择器案例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        /*
     层级选择器的格式：对子孙元素都起作用
             父选择器 子选择器{
                 样式：
             }


           属性选择器：
                 [属性名]{
                     样式..
                 }
     */

        /*层级选择器  作用于span标签中p标签*/
        span p{
            color: red;
        }

        /*对含有title属性的元素起作用*/
        [title]{
            color: chartreuse;
        }

        /*对div标签含有title属性的元素起作用*/
        div[title]{
            color: blue;
        }

        /*对div标签含有title属性,并且属性值为two的元素起作用*/
        div[title=two]{
            color: orange;
        }
    </style>

</head>
<body>


<div>
    <span>
        <p title="one">这是段落1</p>
    </span>
</div>

<p>
    段落2
</p>

<div title="two">
    这是第一个DIV
</div>

<div title="three">
    这是第二个DIV
</div>

</body>
</html>
```

### 小结

**层级选择器格式**

​		父选择器  子选择器{样式}

**属性选择器格式**

​		[属性名]{样式}



## 扩展选择器：伪类、并集、交集选择器

### 目标

1. 伪类选择器
2. 并集选择器
3. 交集选择器

### 语法

![1562745616606](前端-02-CSS-JavaScript基础.assets/1562745616606.png)

### 代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        /*
        伪类选择器的作用： 对于标签处于某种转态下进行样式化

            选择器:visited{ }  已经被访问过状态

            选择器:link{ }    没有被访问过的状态

            选择器:hover{ }  鼠标经过状态

            选择器:active{}   激活转态

        注意：
            1. 在 CSS 定义中，a:hover 必须被置于 a:link 和 a:visited 之后，才是有效的。

            2. 在 CSS 定义中，a:active 必须被置于 a:hover 之后，才是有效的。



        并集选择器：
                格式：
                        选择器1，选择器2{样式 }   对选择器1与选择器2使用相同的样式..

        交集选择器：
                格式：
                        标签名.class选择器||#id选择器{ }  对标签中的某一个元素起作用。



        */





        /*已经被访问过状态*/
        a:visited{
            color: gray;
        }


        /*没有被访问过的状态*/
        a:link{
            color: black;
            text-decoration: none;
        }

        /*鼠标经过状态*/
        a:hover{
            color: orange;
            text-decoration: underline;
        }



        /*鼠标经过状态*/
        a:active{
            color: green;
        }

        /*对input标签获取焦点的状态进行样式化*/
        input:focus{
            background-color: #8cff47;
        }

        /*并集选择器*/
        p,span{
            color: red;
        }

        div.first{
            color: red;
        }

    </style>
</head>
<body>
<a href="#1">传智播客</a>
<a href="#2">传智播客</a>
<a href="#3">传智播客</a>
<a href="#4">传智播客</a>

<hr/>
用户名：<input type="text"><br/>
用户名：<input type="text">

<hr/>
<p class="one">
    Hello Girl!
</p>
<span>
     Hello Boy!
 </span>

<hr/>
<div class="first">
    11111
</div>
<br/>
<div id="second">
    22222
</div>
<br/>
<div>
    333333
</div>
</body>
</html>
```

### 小结

| 扩展选择器       | 选择器符号                                                   |
| ---------------- | ------------------------------------------------------------ |
| 伪类选择器选择器 | 选择器：link 没有被访问过的样式<br/>选择器:visited 被访问过的样式<br/>选择器：hover 鼠标经过的样式<br/>选择器：active 激活样式<br/>选择器:focus 获取焦点的样式 |
| 并集选择器选择器 | 选择器1，选择器2{}                                           |
| 交集选择器       | 标签选择器类选择器\|\|id选择器                               |



## 常见样式：背景样式

### 目标

学习背景样式有哪些属性

### 样式名和值

![1551772698804](前端-02-CSS-JavaScript基础.assets/1551772698804.png)

### 效果

![1551772735047](前端-02-CSS-JavaScript基础.assets/1551772735047.png)

### 代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>背景样式</title>
    <style>
        body{
            /*设置背景颜色*/
            background-color: #4dcaff;
            /*背景图片*/
            background-image: url("img/girl1.jpg");
            /*设置背景图不重复*/
            background-repeat: no-repeat;
            /*设置背景图片的大小
            background-size: 1366px 600px;*/
            /*设置背景图片的起始位置*/
            background-position: 500px 100px;

        }

    </style>
</head>
<body>
    外交部：美又演一出反华闹剧
</body>
</html>
```

### 小结

| **属性名**              | **属性取值**         |
| ----------------------- | -------------------- |
| **background-color**    | 设置背景颜色         |
| **background-image**    | 设置背景图片         |
| **background-repeat**   | 设置背景是否重复     |
| **background-position** | 设置背景坐标起始位置 |
| **background-size**     | 设置背景图片的宽高   |



## 常用样式：文本样式和字体样式

### 目标

1. 文本样式
2. 字体样式
3. 制作案例：诗歌

### 文本样式

![1552181469390](前端-02-CSS-JavaScript基础.assets/1552181469390.png)

### 字体样式

![1551774415181](前端-02-CSS-JavaScript基础.assets/1551774415181.png)

### 案例：诗歌

![1551774444336](前端-02-CSS-JavaScript基础.assets/1551774444336.png)

### 步骤

3.	标题文字大小18px，颜色#06F，居中对齐
4.	每段文字缩进2em
5.	段中的行高是22px
6.	"胸怀中满溢着幸福，只因你就在我眼前",加粗，倾斜，蓝色
7.	最后一段，颜色#390; 下划线，鼠标移上去指针变化。
8.	美字加粗，颜色color:#F36，大小18px;
9.	文席慕容，三个字，12px，颜色#999，不加粗

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>初相遇</title>    <style>        /*1. 标题文字大小18px，颜色#06F，居中对齐*/        h1{            font-size: 18px;            color: #06F;            text-align: center;        }        /*2. 每段文字缩进2em*/        p{            text-indent: 2em;        }        /*3. 段中的行高是22px*/        p{            line-height: 22px;        }        /*4. "胸怀中满溢着幸福，只因你就在我眼前",加粗，倾斜，蓝色*/        #xiong{            font-weight: bolder;            font-style: italic;            color: blue;        }        /*5. 最后一段，颜色#390; 下划线，鼠标移上去指针变化。*/        p:last-child{            color: #390;            text-decoration: underline;            cursor: pointer;        }        /*6. 美字加粗，颜色color:#F36，大小18px;*/        .mei{            color:#36F;            font-size: 18px;            font-weight: bolder;        }        /*7. 文席慕容，三个字，12px，颜色#999，不加粗*/        h1 span{            font-size: 12px;            color: #999;            font-weight: normal;        }        /*8.设置div的宽度为400px , 文本居中对齐*/        div{            width: 400px;            text-align: center;            margin: auto;        }    </style></head><body><div>    <h1>初相遇 <span>文/席慕容</span></h1>    <p><span class="mei">美</span>丽的梦和美丽的诗一样，都是可遇而不可求的，常常在最没能料到的时刻里出现。</p>    <p>我喜欢那样的梦，在梦里，一切都可以重新开始，一切都可以慢慢解释，心里甚至还能感觉到，所有被浪费的时光竟然都能重回时的狂喜与感激。        <span id="xiong">胸怀中满溢着幸福，只因你就在我眼前，</span>对我微笑，一如当年。</p>    <p>我喜欢那样的梦，明明知道你已为我跋涉千里，却又觉得芳草鲜美，落落英缤纷，好像你我才初相遇。</p></div></body></html>
```



### 小结

| **属性名**          | **作用**   |
| ------------------- | ---------- |
| **color**           | 颜色       |
| **line-height**     | 行高       |
| **text-decoration** | 文本修饰符 |
| **text-indent**     | 缩进       |
| **text-align**      | 对齐方式   |

| **属性名**      | **作用** |
| --------------- | -------- |
| **font-size**   | 字体大小 |
| **font-style**  | 字体样式 |
| **font-weight** | 字体粗细 |



## 两种盒子模型

### 目标

​	两种盒子模型的区别

### 概述

​	任何一个网页元素包含由这些属性组成：内容(content)、内边距(padding)、边框(border)、外边距(margin)， 这些属性我们可以用日常生活中的常见事物——盒子作一个比喻来理解，所以叫它盒子模型。

- 内容（content）就是盒子里装的东西
- 内边距(padding)就是怕盒子里装的东西（贵重的）损坏而添加的泡沫或者其它抗震的辅料；
- 边框(border)就是盒子本身了；
- 外边界(margin)则说明盒子摆放的时候的不能全部堆在一起，要留一定空隙保持通风，同时也为了方便取出。

### 盒子模型的属性

两种盒子模型：

1. 标准盒子模型content-box：宽和高会被内边距，边框撑大。设置的是内容的宽和高
2. 怪异盒子模型border-box：设置了固定的宽和高，设置内边距和边框，网页元素会被挤压。

![1551774655949](前端-02-CSS-JavaScript基础.assets/1551774655949.png)

### 如何计算盒子的尺寸

​	在盒子模型中，最重要的还是如何理解元素的实际尺寸。
 	盒子模型分为两种，分别是：标准盒模型 和 怪异盒模型。绝大多数元素的尺寸默认是按照标准盒模型计算的。下面是元素的尺寸分别在两种盒模型下计算规则：

![1551774782003](前端-02-CSS-JavaScript基础.assets/1551774782003.png)

### 标准盒子模型

![1551774810940](前端-02-CSS-JavaScript基础.assets/1551774810940.png)

### 怪异盒子模型

![1551774834485](前端-02-CSS-JavaScript基础.assets/1551774834485.png)

### 计算方式

```
标准盒子：实际宽度=内容宽度+内边距+边框宽度实际高度=内容高度+内边距+边框高度
```

```
怪异盒子：实际宽度=宽度实际高度=高度
```

### 案例：盒子模型

#### 效果

![1551774996564](前端-02-CSS-JavaScript基础.assets/1551774996564.png)

#### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>盒子模型</title>    <style>        /* 盒子模型: 在浏览器看来，任何一个html元素都是有边框的，相当于一个盒子存储了数据一样。           我们学习盒子模型主要是学习是三个属性：                 border: 边框                 padding: 内边距   数据到边框的距离                 margin：外边距    元素与元素边框之间距离           盒子的类型：               1. 标准的盒子（默认）：content-box  标准盒子宽高的计算方式 = 数据大小+padding+border               2. 怪异的盒子：border-box 怪异盒子的宽高计算方式 = 数据大小。        */        #one{            width: 100px;            height: 100px;            /*设置边框线型、大小*/            border:solid 1px;            /*设置左内边距*/            padding-left: 20px;            /*设置该盒子为标准盒子  100+20+2=122 */            box-sizing: content-box;            /*设置盒子的下外边距*/            margin-bottom: 100px;            margin-left: 500px;        }        #two{            width: 100px;            height: 100px;            /*设置边框线型、大小*/            border:solid 1px;            padding-left: 20px;            /*设置该盒子为怪异盒子*/            box-sizing: border-box;        }    </style></head><body><div id="one">div1</div><div id="two">div2</div></body></html>
```



### 小结

- 盒子模型可以分为哪些类型？ 如何设置 ？
  - 标准盒子   boxzing=content-box   
  - 怪异盒子  boxzing=border-box   



## 常用样式：边框有关

### 目标

边框有哪些属性和取值

### 边框的样式

![1551775156839](前端-02-CSS-JavaScript基础.assets/1551775156839.png)

### 内边距和外边距的写法

| **内边距的写法**           | **含义** |
| -------------------------- | -------- |
| **padding-top:10px;**      | 上内边距 |
| **padding-left:10px;**     | 左       |
| **padding-bottom:  10px;** | 下       |
| **padding-right:10px;**    | 右       |

| **外边距的写法**           | **含义** |
| -------------------------- | -------- |
| **margin-top:10px;**       | 上       |
| **margin-left:10px;**      | 左       |
| **margin-bottom:   10px;** | 下       |
| **margin-right:10px;**     | 右       |

### 案例：边框的使用

#### 效果

![1551775249504](前端-02-CSS-JavaScript基础.assets/1551775249504.png)

### 步骤

1. 盒子模型宽和高分别是200px，边框使用double线形，红色，1px粗细
2. 上内边距10px，右内边距20px，下内边距30px，左内边距40px
3. 使用缩写的1句话的方式实现
4. 对div整个居中
5. 设置圆角

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>边框样式</title>    <style>        #box{            /*1. 盒子模型宽和高分别是200px，边框使用double线形，红色，1px粗细*/            width: 200px;            height: 200px;            border: double red 5px;            /*2. 上内边距10px，右内边距20px，下内边距30px，左内边距40px*/         /*   padding-top: 10px;            padding-right: 20px;            padding-bottom: 30px;            padding-left: 40px;*/            /*3. 使用缩写的1句话的方式实现, 如果缺失具体的某一个值，那么就是对边一致*/            padding: 10px 20px 30px 40px  ;            /*4. 对div内容整个居中*/            text-align: center;            /*5. 设置圆角*/            border-radius: 10px;            /*水平居中*/            margin: auto;        }    </style></head><body><div id="box">    我喜欢那样的梦，明明知道你已为我跋涉千里，却又觉得芳草鲜美，落落英缤纷，好像你我才初相遇。    我喜欢那样的梦，明明知道你已为我跋涉千里，却又觉得芳草鲜美，落落英缤纷，好像你我才初相遇。</div></body></html>
```

### 关于块级元素居中

![1551775342231](前端-02-CSS-JavaScript基础.assets/1551775342231.png)

### 小结

| **属性**          | **边框样式** |
| ----------------- | ------------ |
| **border-style**  | 线条         |
| **border-width**  | 设置粗细     |
| **border-color**  | 颜色         |
| **border-radius** | 圆角         |



## 案例：用户的注册

### 目标

使用CSS样式排版出如图所示的效果

### 效果

![1551775397597](前端-02-CSS-JavaScript基础.assets/1551775397597.png)

### 步骤

2. table居中,宽300px，高180px; 边框solid 1px gray
3. td的文字对齐居中，字体大小14px
4. table添加背景，不平铺，图片背景的宽度和高度与table的宽和高一样。
5. 用户名和密码文本框使用类样式，也可以使用其它选择器。宽150px，高32px，边框用实线，圆角5px，1个宽度，黑色
6. 使用伪类样式，当鼠标移动到文本框上的时候，变成虚线橙色边框。得到焦点，背景色变成浅黄色
7. 文本框前面有头像，密码框前面有键盘，左内边距35

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>黑马旅游注册</title>    <style type="text/css">        /*1. table居中,宽300px，高180px; 边框solid 1px gray*/        table{            width: 300px;            height: 180px;            border: solid 1px gray;        }        /*2. td的文字对齐居中，字体大小14px*/        td{            text-align: center;            font-size: 14px;        }        /*3. table添加背景，不平铺，图片背景的宽度和高度与table的宽和高一样。*/        table{            background-image: url("img/bg1.jpg");            background-repeat: no-repeat;            background-size: 300px 180px;        }        /*4. 用户名和密码文本框使用类样式，也可以使用其它选择器。宽150px，高32px，边框用实线，1个宽度，黑色 圆角5px，*/        .info{            width: 150px;            height: 32px;            border: solid 1px black;            border-radius: 5px;        }        /*5. 使用伪类样式，当鼠标移动到文本框上的时候，变成虚线橙色边框。得到焦点，背景色变成浅黄色*/        .info:hover{            border: dashed 1px orange;        }        .info:focus{            background-color: #faff95;        }        /*6. 文本框前面有头像，密码框前面有键盘，左内边距35*/        #username{            background-image: url("img/head.png");            background-repeat: no-repeat;            padding-left: 35px;        }        #password{            background-image: url("img/keyboard.png");            background-repeat: no-repeat;            padding-left: 35px;        }    </style></head><body><div id="content">    <!--2列4行-->    <form action="server">        <table>            <tr>                <th colspan="2">黑马旅游注册</th>            </tr>            <tr>                <td>用户名：</td>                <td>                    <input class="info" type="text" name="username" id="username">                </td>            </tr>            <tr>                <td>密码：</td>                <td>                    <input class="info" type="password" name="password" id="password">                </td>            </tr>            <tr>                <td colspan="2">                    <input type="image" src="img/btn.jpg">                </td>            </tr>        </table>    </form></div></body></html>
```



## JavaScript概述和体验

### 目标

1. JavaScript的作用
2. 编写第1个JavaScript代码

### JavaScript的基本概述  liveScript

​	在1995年时，由Netscape公司在网景导航者浏览器上首次设计实现而成。因为Netscape与Sun合作，Netscape管理层希望它外观看起来像Java，因此取名为JavaScript。 
​	JavaScript是一种属于网络的脚本语言,已经被广泛用于Web应用开发,常用来为网页添加各式各样的动态功能,为用户提供更流畅美观的浏览效果。



### 为什么要JavaScript

1. ==网页需要与用户进行交互，JS可以提升用户与浏览器的交互==。
2. 网页上表单，用户输入的数据需要验证。

![1551945857805](前端-02-CSS-JavaScript基础.assets/1551945857805.png)

### 前端各技术的作用

| **技术**       | **作用**               |
| -------------- | ---------------------- |
| **HTML**       | 描述一个网页的结构     |
| **CSS**        | 对页面的数据进行样式化 |
| **JavaScript** | 与用户交互             |



### JS体验案例

#### 需求

​	使用JS在网页上输出5个Hello World

#### 效果

![1551945949527](前端-02-CSS-JavaScript基础.assets/1551945949527.png) 

#### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title>    <script>        for(var i = 0 ; i<5 ; i++){            document.write("hello world<br/>");        }    </script></head><body></body></html>
```

### JavaScript与Java的区别

| **特点**     | **Java**                                               | **JavaScript**                                               |
| ------------ | ------------------------------------------------------ | ------------------------------------------------------------ |
| **面向对象** | 面向对象的语言                                         | 基于对象的语言，javascript中有对象存在，但是不符合面向对象三大特征。封装、继承、多态 |
| **运行方式** | 编译解析型语言                                         | 解析型语言                                                   |
| **跨平台**   | 只有有jvm即可                                          | 只有有浏览器即可                                             |
| **数据类型** | 强类型语言, 声明变量之前一定要明确数据类型  int age=20 | 弱类型语言  ，声明变量是不需要确定具体的数据类型             |
| **大小写**   | 严格区分                                               | 严格区分                                                     |

### 小结

1. **什么是JS？**

- js是运行浏览器端的一门脚本语言。
  - 脚本语言：一定要有寄生环境

2. **说说网页上各种技术的作用：HTML，CSS，JS。**

- html  负责网页结构
  - css  美化
  - js  负责与用户交互



## script标签的使用说明

### 目标

1. JS语言的三个组成部分
2. \<script>标签的说明

### JavaScript语言组成

| **组成部分** | **作用**                                                     |
| ------------ | ------------------------------------------------------------ |
| DOM          | Document Object Model 文档对象模型，用于操作网页中各种元素和标签 |
| BOM          | Browser Object Model 浏览器对象模型，用于操作浏览器中各种对象。如：window |
| ECMA Script  | 脚本语言规则，==制定JS脚本的核心基础，基础知识,基础语法部分所有浏览器都是通用的，没有浏览器的兼用性问题== |

### script的编写方式

1. 方式一:  可以直接在html网页中添加一个<script>标签即可

2. 方式二： 引入外部的js文件。 使用<script>标签引入即可

    

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>2js代码编写方式</title>    <!--        1. 方式一:  可以直接在html网页中添加一个<script>标签即可        例子：                 <script>                    //弹出框                    //alert("呵呵,我怎么感觉你们好急哦！");                    document.write("呵呵，不急不急！");                </script>        2. 方式二： 引入外部的js文件。 使用<script>标签引入即可        js注释：            //  单行注释            /*  多行注释 */        js常用的两个方法：                alter()   弹出框                document.write()  向网页输出指定的数据    注意： 如果一个script标签一旦用于引入js文件，那么该script标签的标签体不能再编写任何的js代码。    -->    <script src="js/1.js"></script>    <script>        alert("呵呵,哈哈");    </script></head><body></body></html>
```



### JavaScript的注释

![1551946466936](前端-02-CSS-JavaScript基础.assets/1551946466936.png)

### 小结

1. **JS由哪三个部分组成？**

   - ECMAScript  基于语法

   - BOM 浏览器对象模型

   - DOM 文档对象模型

      

2. **script标签有哪两个属性？**

   - src 
   - type

   

   

## 变量的定义

### 目标

1. 变量的定义的语法
2. var关键字的使用

### 定义语法

| **数据类型** | **Java**中定义变量                | **JS中定义变量** |
| ------------ | --------------------------------- | ---------------- |
| **整数**     | int i = 5;                        | var              |
| **浮点数**   | float f = 3.14; 或 double d=3.14; | var              |
| **布尔**     | boolean b = true;                 | var              |
| **字符**     | char c = 'a';                     | var              |
| **字符串**   | String str =   "abc";             | var              |

### 注意事项

1. 关于弱类型？

   ```
   同一个变量可以赋值为不同的数据类型
   ```

2. 在JS中的字符和字符串引号？

   ```
   在JS中没有字符和字符串区分，只有字符串，字符串既可以使用单引号，也可以使用双引号。
   ```

3. var定义变量的特点

   1. 在js中是没有字符的概念，只有字符串。 字符串可以使用‘’或者“”括起来
   2. 在js中没有变量名重复声明概念，后声明变量会覆盖前面定义变量。
   3. var关键字可以省略不写，但是不建议。
   4. {}是没法限制变量的作用范围的。
   5. 一个变量存储的数据类型是可以随时发生变化。

### 案例

#### 需求

​	输出不同类型的数据

#### 效果

![1564126867830](前端-02-CSS-JavaScript基础.assets/1564126867830.png) 

#### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>变量的声明</title>    <script>        /*            变量的声明格式：  var  变量名 = 数据；            变量要注意的事项:                1. 在js中是没有字符的概念，都是字符串,编写字符串的时候可以使用单引号或者双引号                2. 在js是没有变量重复的概念。后定义同名变量会直接覆盖前面定义同名变量                3. js声明变量的时候可以省略var不写，但是建议同学们都写上                4. {}对var声明变量是没有限制范围的作用                5. 一个变量存储的数据类型可以随时发生变化         */        //整数        {            var age =19;        }        age = "哈哈";        document.write("老钟的真实年龄："+ age+ "<br/>");        //浮点数        var height = 175.5;        document.write("老钟的真实身高："+ height+ "<br/>");        //布尔值        var flag = true;        document.write("老钟是否已婚："+ flag+ "<br/>");        //字符        var poker = "A";        document.write("老钟手上的牌："+ poker+ "<br/>");        //字符串        var str = '霸王';        document.write("老钟洗发水："+ str+ "<br/>");    </script></head><body></body></html>
```

### 小结

1. **在JS中定义变量使用哪个关键字？**  

   ​	var 

2. **JS是弱类型还是强类型？**

   弱类型  

3. **有没有字符和字符串的区别？** 

   ​	只有字符串

## 五种数据类型

### 目标

JS中有哪五种数据类型

### 五种原始数据类型

| **关键字**    | **说明**                                 |
| ------------- | ---------------------------------------- |
| **number**    | 数值型，包含：整数和浮点数               |
| **boolean**   | 布尔型，包含：true或false                |
| **string**    | 字符串类型，包含字符和字符串             |
| **object**    | 对象类型，包含自定义的对象，系统内置对象 |
| **undefined** | 未定义的数据类型，没有初始化的变量。     |

### typeof操作符

作用：判断一个变量的数据类型，系统自带函数

写法：

- typeof 变量名 
- typeof(变量名)

### 案例：数据类型的演示

#### 需求

分别输出整数、浮点数、字符串(单引号和双引号)、布尔、未定义、对象 的数据类型

#### 效果

 

#### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>js的数据类型</title>    <script>        /*            数据类型要注意的事项：                1. 整数与小数都是属于number类型                2. 字符与字符串都是属于string类型                3. 一个变量的数据类型是取决于该变量存储的值                4. undefined类型只有一个值就是undefined。                5.js数据类型： number、 boolean、 string ,undefined ， object         */        //整数        var a = 1;        document.write("a="+a+" 数据类型："+(typeof a)+"<br/>");        //浮点数        var b = 3.14;        document.write("b="+b+" 数据类型："+(typeof b)+"<br/>");        //布尔        var c = true;        document.write("c="+c+" 数据类型："+(typeof c)+"<br/>");        //字符串与字符        var d = 'a';        document.write("d="+d+" 数据类型："+(typeof d)+"<br/>");        var e = 'hello';        document.write("e="+e+" 数据类型："+(typeof e)+"<br/>");        // undefined类型        var f ;        document.write("f="+f+" 数据类型："+(typeof f)+"<br/>");        var g = null;        document.write("g="+g+" 数据类型："+(typeof g)+"<br/>");    </script></head><body></body></html>
```

### 小结

- javascript的数据类型

  - ```
    number、 boolean、 string ,undefined ， object
    ```

## 数据类型的转换函数

### 目标

学习JS中类型转换函数

### 全局函数

概念：在JS脚本任意位置都可以直接使用的函数，在JS内部带了一些全局函数。

### 

| **转换函数**     | **作用**                                                     |
| ---------------- | ------------------------------------------------------------ |
| **parseInt()**   | 将字符串转成整数，如果前面一段可以转则转，否则不要           |
| **parseFloat()** | 功能同上，将字符串转成浮点数                                 |
| **isNaN()**      | 判断字符串是否能转成数字，如果能转返回false，不是数字返回true |



### 案例：字符串转数字

#### 需求：

1. 创建两个字符串的变量，保存2个数字
2. 对两个数字进行加的运算，输出它们的计算结果
3. 使用两个小数再进行计算
4. 判断一个字符串能否转换成数字，看看输出结果

#### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>数据类型转换</title>    <script>        /*        为什么需要进行数据类型转换：             从网页中获取到的所有数据都是字符串类型，如果我们需要拿这些数据去运算，那么就必须进行数据类型转换。         数据类型转换的函数：                1. parseInt()  把字符串转换为整数,从字符串的第一个字符开始一直往后去遍历，一直取到非数字字符为止，把前面的数字字符串转换为整数                2. parseFloat（） 把字符串转换为小数，从字符串的第一个字符开始一直往后去遍历，一直取到非数字字符为止，把前面的数字字符串转换为小数                3. isNaN()  is not a number  判断字符串是否为纯数字字符构成， 不是纯数字字符组成返回true，        */        var num1 = "123ab12";        var result1= parseInt(num1);        document.write("result1 = "+ result1+"<br/>"); //123        var num2 = "123ab12";        var result2= parseFloat(num2);        document.write("result2 = "+ result2+"<br/>"); //123.2        var str3 = "123";        document.write(isNaN(str3)); //true    </script></head><body></body></html>
```

### 小结

- 数据类型转换相关的方法

  - paseInt

  - parseFloat

  - isNaN

    





## 常用的运算符

### 目标

学习JS中的各种运算符

### 算术运算符

算术运算符用于执行两个变量或值的算术运算

![1551947559080](前端-02-CSS-JavaScript基础.assets/1551947559080.png) 

### 赋值运算符

赋值运算符用于给JavaScript 变量赋值

![1551947658451](前端-02-CSS-JavaScript基础.assets/1551947658451.png) 

### 比较运算符

比较运算符用于逻辑语句的判断，从而确定给定的两个值或变量是否相等。

![1551947697070](前端-02-CSS-JavaScript基础.assets/1551947697070.png) 

**数字可以与字符串进行比较，字符串可以与字符串进行比较。字符串与数字进行比较的时候会先把字符串转换成数字然后再进行比较**

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>js的运算符</title>    <script>        /*         疑问：== 与 === 的区别             1. ==比较的是值相等， === 比较值与类型都要相等。          */        var num1 = 1; //基本数据类型        var num2 = new Number(1); //包装类型        document.write("num1==num2?"+(num1==num2)+"<br/>");        document.write("num1===num2?"+(num1===num2)+"<br/>");        /*        字符串与数字比较， 字符串与字符串比较规则            1. 在js中字符串是可以与数字进行比较的，在使用比较运算符的时候js会自动把            字符串转换为number类型再比较。            2.字符串与字符串的比较规则：                    a. 能够找到对应位置不同的字符，那么就比较第一个不同的字符。                    b . 如果两个字符串没有对应不同的字符，那么比较是两个字符串的长度。        */        var str = "18a";        var num3= 3;        document.write("str>num3?"+(str>num3)+"<br/>");        var str1= "abc";        var str2= "ab";        document.write("str1>str2?"+(str1>str2)+"<br/>");    </script></head><body></body></html>
```

### 逻辑运算符

逻辑运算符用来确定变量或值之间的逻辑关系，支持短路运算

![1551947786567](前端-02-CSS-JavaScript基础.assets/1551947786567.png) 

逻辑运算符不存在单与&、单或|

### 三元运算符

![1551947843411](前端-02-CSS-JavaScript基础.assets/1551947843411.png) 

### 小结

- **运算符 === 有什么作用？**

  比较值与类型都需要相等

  



## 流程控制：if 语句

### 目标

JS中if语句使用非boolean类型的条件

### if 语句：

在一个指定的条件成立时执行代码。 

```
if(条件表达式) {     //代码块;}
```



### if...else 语句

在指定的条件成立时执行代码，当条件不成立时执行另外的代码。

```
if(条件表达式) {     //代码块; } else {     //代码块; }
```



### if...else if....else 语句

使用这个语句可以选择执行若干块代码中的一个。

```
if (条件表达式) {     //代码块; } else if(条件表达式) {     //代码块; } else {     //代码块; }
```

### 使用非布尔类型的表达式

建议不要使用非布尔类型的表达式

| **数据类型**            | **为真** | **为假**     |
| ----------------------- | -------- | ------------ |
| **number**              | 非0      | 0            |
| **string**              | 字符串   | 空字符串  "" |
| **undefined**           |          | 为假         |
| **NaN(Not a   Number)** |          | 为假         |
| **object**              | 有值     | null         |

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>js的if语句</title>    <script>        /*            if语句特殊之处：                1. js的if语句的条件可以放任意类型的数据。                        number  0为false ， 非0为true.                        bolean                        string  空串为false， 非空串为true                        undefined  false                        object  null 为false ， 非null为true         */        var sleep = false;        var num ;        if(num){            document.write("小锤子敲死你！");        }else{            document.write("大锤子砸死你！");        }    </script></head><body></body></html>
```

### 小结

1. if语句的特点

   - if条件可以放任意类型的数据

     

## 流程控制：switch语句

### 目标

1. switch语句与Java的区别
2. window对象的方法

### 语法一：case后使用常量，与Java相同

```
switch(变量名) {   case 常量值:      break;   case 常量值：      break;   default:       break; }
```



### 语法二：case后使用表达式

```
switch(true) {  //这里的变量名写成true   case 表达式:     //如：n > 5     break;   case 表达式:     break;   default:     break; }
```

### BOM相关的方法

| **window对象的方法名**                   | **作用**                                     |
| ---------------------------------------- | -------------------------------------------- |
| **string   prompt("提示信息","默认值”)** | 出现一个输入框，输入信息。返回值是string类型 |
| **alert("提示信息")**                    | 出现一个信息框，只有一个确定按钮             |

输入框

![1562746273460](前端-02-CSS-JavaScript基础.assets/1562746273460.png)

信息提示框

![1562746197150](前端-02-CSS-JavaScript基础.assets/1562746197150.png)

window.prompt(),  window.alert()

如果是window对象的方法或属性，window可以省略。上面的方法：prompt()  alert()

#### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title>    <script>        var money = prompt("请输入你要给我的金额");        document.write("本次获取到金额是："+money);    </script></head><body></body></html>
```



### 案例：判断一个学生的等级

通过prompt输入的分数，如果90~100之间，输出优秀。80~89之间输出良好。60~79输出及格。60以下输出不及格。其它分数输出:分数有误。

#### 效果

![1551948291900](前端-02-CSS-JavaScript基础.assets/1551948291900.png)

#### 分析

1. 使用prompt得到输入的分数
2. 使用switch对分数进行判断
3. 如果在90到100之间，则输出优秀，其它依次类推。

#### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title>    <script>        var score = prompt("请输入您的分数：");        switch(true){            case score>=90 && score<=100:                document.write("优秀");                break;            case score>=80 && score<=89:                document.write("良好");                break;            case score>=60 && score<=79:                document.write("及格");                break;            case score>=0 && score<=59:                document.write("补考");                break;            default:                document.write("非法成绩！");                break;        }    </script></head><body></body></html>
```

### 小结

​	如果switch的case后面要使用表达式，switch后面值要写成什么？ true

​		

## 流程控制：循环语句

### 目标

使用循环实现9x9乘法表

### while语句：

当指定的条件为 true 时循环执行代码

```
while (条件表达式) {     需要执行的代码; }
```



### do-while语句：

最少执行1次循环

```
do {     需要执行的代码; } while (条件表达式)
```



### for 语句

循环指定次数

```
for (var i=0; i<10; i++) {     需要执行的代码; }
```



### break和continue

```
break: 结束整个循环continue：跳过本次循环，执行下一次循环
```



### 案例：乘法表

#### 需求

以表格的方式输出乘法表，其中行数通过用户输入

#### 效果

![1551950186004](前端-02-CSS-JavaScript基础.assets/1551950186004.png)

#### 分析

1. 先制作一个没有表格，无需用户输入的9x9乘法表
2. 由用户输入乘法表的行数
3. 使用循环嵌套的方式，每个外循环是一行tr，每个内循环是一个td
4. 输出每个单元格中的计算公式
5. 给表格添加样式，设置内间距

#### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>乘法表</title>    <style type="text/css">        table {            /*细边框样式，单元格之间没有间隔*/            border-collapse: collapse;        }        td {            padding: 5px;        }        td:hover {            background-color: yellow;        }    </style>    <script>        // 1. 弹出一个输入框       var rows =  prompt("请输入乘法表的行数");       document.write("<table border='1px'>");       for(var i =1; i<=rows ;i++){ //控制的行数           document.write("<tr>");           for(var j = 1 ; j<=i ; j++){                document.write("<td>");                document.write(j+"*"+i+"="+(i*j));                document.write("</td>");           }           document.write("</tr>");       }        document.write("</table>");    </script></head><body></body></html>
```

## 在浏览器中调试代码

### 目标

如何在浏览器中调试JS代码

### 设置断点

注：设置断点以后要重新刷新页面才会在断点停下来

![1564128455120](前端-02-CSS-JavaScript基础.assets/1564128455120.png)

### 语法错误

![1551947495069](前端-02-CSS-JavaScript基础.assets/1551947495069.png)

