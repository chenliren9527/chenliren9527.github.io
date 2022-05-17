---
title: 前端(01)-HTML5
tags:
  - 笔记
  - 前端
  - HTML5
categories:
  - 前端
date: 2020-09-28 12:48:00
---

# 《HTML5-笔记》

今天内容特点： 没有逻辑、不需要同学们去理解，只需要记忆。练习2次。 

# 学习目标

1**、能够使用idea创建html文档** 

​    可以

**2、能够使用h1~h6、hr、p、br 等与文本有关的标签** 

​		标题

​        水平线

​         段落

​       换行

3、能够使用有序列表ol-li和无序列表ul-li显示列表内容 

4、能够使用块标签div与内联标签span

5、能够使用图片img标签把图片显示在页面中 

​		`<img src="路径"/>`

6、能够使用超链接a标签跳转到一个新页面 

​		`<a href="链接地址">`

7、能够使用table、tr、td标签定义表格 

​		table

​             tr

​                 th||td



**8、能够制作黑马旅游网的首页** 

​		可以

**9、能够使用表单form标签创建表单容器** 

​			form

10、能够使用表单中常用的input标签创建表单项项 

  - 普通文本输入框 type=text

  - 密码框  type=password

  - 单选框  type=radio

  - 复选框  type=checkbox 

  - 文件上传表单项： type=file

- 提交按钮 type=submit

- 重置 type=reset

- 普通 type=button

  

11、能够使用表单select标签定义下拉选择输入项 

​	select

​        option

12、能够使用表单textarea标签定义文本域 

​		<textarea >



# 学习内容

## 01、HTML的概述

### 目标

​	HTML的概念是什么，它是用来做什么的?‘



### 概念

HTML就是网页，Hyper Text Markup Language ==超文本标记语言==

XML：eXtensaible Markup Language 可扩展标记语言

1. **超文本：**超级的文本 ， 普通文本只能编写文字， 超级文本可以控制文本的颜色、大小，并且可以插入图片等其他多媒体的资源。   
2. **标记语言：**该门语言就是有==标签构成的==，没有任何的逻辑。学习该门语言非常的简单。

3. **容错性：** 浏览器的对该语言的容错性非常强。  松散型语言

### 运行方式

![1551669429404](前端-01-HTML5.assets/1551669429404.png)

1. HTML保存在：服务器上的，也称为Web服务器。
2. 运行在：本地浏览器。先将网页从服务器上下载到本地，再由浏览器解析运行。

### 小结

1. **什么是HTML?** 

   ​    超文本标记语言

2. **html文件的运行方式：**

   ​	 直接使用浏览器运行即可

## 02、HTML5的介绍

### 目标

​	什么是HTML5，它有什么特点？

### HTML5的作用

第5个版本，主要用来支持移动端的网页，如：手机，平板

![1551669453294](前端-01-HTML5.assets/1551669453294.png)

​	2014年10月29日，经过几乎8年的艰辛努力，HTML5标准规范终于最终制定完成了，并已公开发布，这是一次重大的革新。HTML5将会取代1999年制定的HTML 4.01、XHTML 1.0标准，以期能在互联网应用迅速发展的时候，使网络标准达到符合当代的网络需求，为桌面和移动平台带来无缝衔接的丰富内容。  

​	目前支持Html5的浏览器包括Firefox（火狐浏览器），IE9及其更高版本，Chrome（谷歌浏览器），Safari，Opera等。不同的浏览器之间是有差异的，同一个网页在不同的浏览器上运行结果可能不同。

![1551669532678](前端-01-HTML5.assets/1551669532678.png)

### HTML5的作品

![1551669555295](前端-01-HTML5.assets/1551669555295.png)

### 小结

**什么是HTML5，它有什么特点？**

- 支持移动端的设备



## 03、网页的基本结构、使用idea创建网页

### 目标

​	如何创建HTML？

### 常见的HTML编辑器

1. **HBuilder**

   ![1551669763877](前端-01-HTML5.assets/1551669763877.png)

2. **Adobe Dreamweaver CS**

   ![1551669807940](前端-01-HTML5.assets/1551669807940.png)

3. **SublimeText**

   ![1551669828420](前端-01-HTML5.assets/1551669828420.png)

4. Visual Studio Code 

   微软公司第一次向开发者们提供了一款真正的跨平台编辑器。

   ![1551769940609](前端-01-HTML5.assets/1551769940609.png)

5. **NotePad++**

   ![1551669847102](前端-01-HTML5.assets/1551669847102.png)

### 使用记事本创建

```html
<font size="20px" color=red >我爱香港
<img width="250px" height="250px" src="h://美女/1.jpg"/> 
```



### HTML的基本结构

注意：以后我们编写html代码一定要符合html代码的结构。

一个html网页的机构必须有有head（头），body(体)

**HTML的结构**

```html
<html>
    <head>
    	头信息： 标题、网页保存时候使用码表还有浏览器打开时候使用码表
    </head>
    <body>
         体部分,网页的正文部分
    </body>
    
</html>


```



**案例：**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <!-- 网页保存使用码表与浏览器解析的时候使用码表 -->
    <meta charset="UTF-8">
    <!--网页标题-->
    <title>这是第一个网页</title>
</head>
<body>
    <!--网页正文部分-->
    大家买车票了吗？没有，真好，可以留校学习！
</body>
</html>
```

**html的注释**

```html
<!-- 这个是注释 -->
```





### 使用IntelliJ IDEA创建html

​	一个网页项目建议按如下目录创建结构

![1551670240611](前端-01-HTML5.assets/1551670240611.png)

1. 创建静态Web工程

   ![1551670035481](前端-01-HTML5.assets/1551670035481.png)

2. 指定工程名和保存位置

   ![1551670125978](前端-01-HTML5.assets/1551670125978.png)

3. 创建HTML文件，选择html5的版本

   ![1551670139700](前端-01-HTML5.assets/1551670139700.png)

4. 创建HTML文件

5. 点右上角一排浏览器按钮运行，idea会使用内置的服务器在指定的浏览器上运行。

   ![1551670171151](前端-01-HTML5.assets/1551670171151.png)

6. 在浏览器上运行的结果

   ![1551670186424](前端-01-HTML5.assets/1551670186424.png)

7. 访问地址

   ![1551670197659](前端-01-HTML5.assets/1551670197659.png)

8. 通常不需要修改浏览器文件的地址，如果要修改可以在这里操作

   ![1551670268837](前端-01-HTML5.assets/1551670268837.png)

### 小结

| **标签名** | **作用**                |
| ---------- | ----------------------- |
| **html**   | 根标签                  |
| **head**   | 头信息， 网页标题，码表 |
| **body**   | 网页正文部门            |
| **注释**   | <!-- 注释 -->           |



## 04、文本标签的学习

### 目标

1.  标签的分类
2.  文本标签的学习

### HTML标签的分类

| **有否有主体**   | **格式**                     |
| ---------------- | ---------------------------- |
| **有主体标签**   | <开始标签> 标签体</结束标签> |
| **没有主体标签** | <标签名/>                    |

| **是否换行** | **特点**         |
| ------------ | ---------------- |
| **块标签**   | 独占一行         |
| **内联标签** | 不需要独占一行的 |

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>标签分类</title>
    <!--标签分类：-->
        <!--1. 是否有标签的主体划分
                比如：
                    a. <font size="12px" color='green'>大家好</font>  有主体标签 , 如果一个标签需要封装数据，则需要标签体（标签主体）
                    b. 比如： 换行标签，<br/> 功能比较单一，不需要封装数据
        -->

        <!--2. 根据是否独占一行划分
            独占一行标签： 块级标签    比如： div,h1
            不需要独占一行标签： 内联标签， 比如： <span>

        -->

</head>
<body>

大家好, <br/><font size="12px" color='green'>中秋快乐</font><br/>

<!--块级标签-->
<div>AAA</div>
<span>BBB</span><span>Cccc</span>

</body>
</html>
```







### 文本标签介绍

![1551670393595](前端-01-HTML5.assets/1551670393595.png)

![1551670419113](前端-01-HTML5.assets/1551670419113.png)

### 代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>文本标签</title>

    <!--
        注意： 如果需要控制数据的样式，我们是通过属性去控制，属性格式： <开始标签 属性名="属性值"></结束标签>   <标签名 属性名="属性值" />
常用的文本标签
    h1~h6 表示标题标签，
         align="center"  对齐方式
    hr : 水平线标签
         color： 颜色
         size： 粗细
         width: 宽度
         align: 对齐的方式， left、center、right
    font ： 字体标签

    b标签： 加粗

    i标签： 斜体

    br  : 换行标签

    p  ： 段落标签



     -->
</head>
<body>
<h1 align="center">习大大召开第三次新疆工作会议</h1>
<h2 align="center">习大大召开第三次新疆工作会议</h2>
<h3 align="center">习大大召开第三次新疆工作会议</h3>
<h4 align="center">习大大召开第三次新疆工作会议</h4>
<h5 align="center">习大大召开第三次新疆工作会议</h5>
<h6 align="center">习大大召开第三次新疆工作会议</h6>
<!--水平线标签-->
<hr color="green" size="5px" width="200px" align="left"/>
时隔6年，中央召开第三次新疆工作座谈会。
习近平总书记在会上<i><b>强调</b></i>，<br/>“做好<font color="red">新疆工作</font>是全党全国的大事”。
<p>
记得2014年到新疆考察时，总书记就指出，“新疆工作在党和国家工作全局中具有特殊重要的战略地位”。
</p>
<p>
是全党全国的大事”“具有特殊重要的战略地位”，正说明新疆发展和稳定，关系全国改革发展稳定大局，关系祖国统一、民族团结、国家安全，关系中华民族伟大复兴。
</p>
</body>
</html>
```



### 小结

| 说说下面文本标签的作用 | **功能** |
| ---------------------- | -------- |
| **h1~h6**              | 标题     |
| **font**               | 字体     |
| **br**                 | 换行     |
| **p**                  | 段落     |
| **hr**                 | 水平线   |
| **b**                  | 加粗     |
| **i**                  | 斜体     |



## 05、案例：制作黑马公司介绍

### 目标

​	使用已经学习的文本标签，制作如下的公司介绍的页面

### 效果

![1551670516854](前端-01-HTML5.assets/1551670516854.png)

### 步骤

1. “公司介绍”，需要使用标题标签完成 。例如：\<h3>
2. 两条橙色的线使用水平线完成
3. “中关村黑马程序员训练营” 需要使用字体标签完成 
4. “传智播客” 需要斜体\<i> 和 粗体\<b> 组合完成
5. 这个文档被划分成4个段落，每一个段落之间有定义的间隔，需要使用段落标签\<p>完成，如果要缩进目前可以使用两个全角空格。后面还有其它实现方式。
6. 第2行与第3行是一个普通的换行，需要使用\<br/>完成
7. 最下面的页脚使用2号字体，灰色，居中。

### 代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<p>
<h2>公司简介</h2>
<hr color="orange"/>
<p><font color="red">"中关村黑马程序员训练营"</font>是由<i><b>传智播客</b></i>联合中关村软件园、CSDN， 并委托传智播客进行教学实施的软件开发高端培训机构，致力于服务各大软件企业，解决当前软件开发技术飞速发展， 而企业招不到优秀人才的困扰。
目前，“中关村黑马程序员训练营”已成长为行业“学员质量好、课程内容深、企业满意”的移动开发高端训练基地， 并被评为中关村软件园重点扶持人才企业。</p>
<p>黑马程序员的学员多为大学毕业后，有理想、有梦想，想从事IT行业，而没有环境和机遇改变自己命运的年轻人。 黑马程序员的学员筛选制度，远比现在90%以上的企业招聘流程更为严格。任何一名学员想成功入学“黑马程序员”， 必须经历长达2个月的面试流程，这些流程中不仅包括严格的技术测试、自学能力测试，还包括性格测试、压力测试、 品德测试等等测试。毫不夸张地说，黑马程序员训练营所有学员都是精挑细选出来的。百里挑一的残酷筛选制度确 保学员质量，并降低企业的用人风险。</p>
<p>中关村黑马程序员训练营不仅着重培养学员的基础理论知识，更注重培养项目实施管理能力，并密切关注技术革新， 不断引入先进的技术，研发更新技术课程，确保学员进入企业后不仅能独立从事开发工作，更能给企业带来新的技术体系和理念。</p>
<p>一直以来，黑马程序员以技术视角关注IT产业发展，以深度分享推进产业技术成长，致力于弘扬技术创新，倡导分享、 开放和协作，努力打造高质量的IT人才服务平台。</p>

<hr color="orange"/>


<div style="color: gray;text-align: center">
江苏传智播客教育科技股份有限公司<br/>
版权所有Copyright © 2006-2018, All Rights Reserved 苏ICP备16007882
</div>

<!--<center>
<font color="gray">
江苏传智播客教育科技股份有限公司<br/>
版权所有Copyright © 2006-2018, All Rights Reserved 苏ICP备16007882
</font>
</center>-->

</body>
</html>
```





## 06、列表标签：有序列表和无序列表

### 目标

1. 有序列表
2. 无序列表

### 作用

![1551670658102](前端-01-HTML5.assets/1551670658102.png)

### 案例需求

​	制作如图所示的菜单列表，左边列是有序列表，右边列是无序列表

### 效果

![1551670700487](前端-01-HTML5.assets/1551670700487.png)

### 代码

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>列表标签</title>
<!--
    列表标签类型：
        ol-li  有序列表标签
            type属性： 控制序号
        ul-li  无序列标签
            type属性：
-->


</head>
<body>
    今天中午吃什么呢？
        <ol type="1">
            <li>猪脚饭</li>
            <li>鸭脚饭</li>
            <li>鸡脚饭</li>
            <li>煲仔饭</li>
        </ol>

    今晚去哪里？
    <ul type="circle">
        <li>宿舍</li>
        <li>网吧</li>
        <li>洗脚城</li>
        <li>图书馆</li>
    </ul>



</body>
</html>
```



### 小结

1. **有序列表使用什么标签？**

   	ol-li

2. **无序列表使用什么标签？**

   ​     ul-li



## 07、块标签与内联标签

### 目标

​	学习span标签和div标签的作用和区别

### 作用

![1551767026042](前端-01-HTML5.assets/1551767026042.png)

### 区别

#### div作用

![1551670828108](前端-01-HTML5.assets/1551670828108.png)

​	\<div> 元素是块级元素，浏览器会在其前后显示折行。它是可用于组合其他 HTML 元素的容器。 \<div> 元素是英文division，起到分割的意思。如果与 CSS 一同使用，\<div> 元素可用于对大的内容块设置样式属性。\<div> 元素的一个常见的用途是文档布局。

#### span作用

![1551670912590](前端-01-HTML5.assets/1551670912590.png)

​	\<span>元素是内联元素，可用作文本的容器。当与 CSS 一同使用时，\<span> 元素可用于为部分文本设置样式属性。

### 案例

​	通过代码认识div和span的功能和特点

### 效果

![1551670962184](前端-01-HTML5.assets/1551670962184.png)

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title></head><body><span style="background-color: yellow;color: red">万维网联盟，</span>又称W3C理事会1994年10月在麻省理工学院计算机科学实验室成立<div style="background-color: yellow;color: red">万维网联盟，</div>又称W3C理事会1994年10月在麻省理工学院计算机科学实验室成立</body></html>
```



### 小结

**div标签和span标签的主要区别是什么？**

- div需要独占一行
- span不需要独占一行



## 08、dl-dt-dd列表标签(项目列表)

### 目标

学习dl-dt-dd标签的使用

### 介绍

用于网页布局比较多，一种列表显示方式

dl: 是列表的容器

dt: 是列表的标题 title

dd: 是列表的内容，(描述 description)

### 语法

```html
<dl>  <dt>标题</dt>    <dd>内容</dd>  <dd>可以有多个</dd></dl>
```

### 案例![1601260400832](前端-01-HTML5.assets/1601260400832.png)



```java
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title>    <!--项目列表标签           dl:             dt:               dd:    --></head><body><h3>公司架构</h3>    <dl>        <dt>JAVA教研部</dt>        <!--dd标签自带缩进效果-->        <dd>广武老师</dd>        <dd>华弟老师</dd>        <dd>张平老师</dd>        <dd>小钟</dd>        <dt>学工部</dt>        <dd>宝晴老师</dd>        <dd>灵涛老师</dd>        <dd>大长腿老师</dd>    </dl></body></html>
```



### 小结

​	项目列标签到组成：

​		 dl---dt---dd



## 09、实体字符

### 目标



### 为什么需要使用实体字符

有些符号在html里面是有着特殊含义，如果我们需要原样显示特殊符号，我们则需要使用实体字符。



​	当页面上需要使用一些特殊符号的时候怎么办呢？

![1551683446920](前端-01-HTML5.assets/1551683446920.png)

### 案例代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title>    <!--为什么需要学习实体字符（转义字符）：            原因： 因为某些字符已经被赋予了特殊的含义，如果需要原样显示这些特殊符号，我们则需要使用实体字符。        常用实体字符：              大于号        &gt;              小于号        &lt;              空格          &nbsp;    --></head><body>    &lt;h1&gt;标签是一个标题标签    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;同学们一定要加强表达能力，表达能力是非常非常非常重要!<br/>《java程序员从入门到删库》 作者：&copy;钟哥老师<br/>青岛龙虾节,每一只脚是50块钱 最终解析权归属于： &reg;青岛小王子</body></html>
```

### 常用的实体字符表

![1551683521415](前端-01-HTML5.assets/1551683521415.png)

### 小结

1. **为什么需要学实体标签**

   - 某些标签是有着特殊含义的，如果需要原样显示这些特殊含义的标签，则需要使用实体字符。

2. **常用的实体标签**

   - \&gt;   大于

   - \&lt;   小于

   - \&nbsp  空格

   - \&copy；  版权

   - \&reg;   注册商标

     


## 10、图像标签(重点)

### 目标

​	如何在页面上显示图片

![1551683566145](前端-01-HTML5.assets/1551683566145.png)

### 基本语法

```html
没有主体标签<img src="指定图片的相对地址"/>
```

### 常用属性

![](前端-01-HTML5.assets/1551767064737.png)

### 代码

```java
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title><!--    img 图像标签        常用属性：            src: 图像路径            width ： 宽度            height: 高度            title  ：  鼠标移动到图片上的文字说明            alt: 没法加载图片的时候的文字说明--></head><body>    <img src="img/3.jpg" width="500px" height="300px" alt="这只是一个美女" title="俄罗斯进口"></body></html>
```



### 小结

图像标签的名字是什么，哪个属性是必须的？

​		img,   

​       src



## 11、案例：家用电器排行榜

### 目标

制作家用电器排行榜案例

![1551683693058](前端-01-HTML5.assets/1551683693058.png) 

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>家用电器排行榜</title></head><body><h3>家用电器排行榜</h3><hr color="orange" width="380px" align="left"/><img src="img/tv01.jpg">索尼KLV-40R476A  55英寸  ¥ 3599.00<hr color="orange" width="380px" align="left"/><img src="img/tv02.jpg">海信LED93247  50英寸  ¥ 2486.00<hr color="orange" width="380px" align="left"/><img src="img/tv03.jpg">三星98EAC  60英寸  ¥ 4777.00<hr color="orange" width="380px" align="left"/><img src="img/tv04.jpg">创维42E5CHR  42英寸  ¥ 2799.00<hr color="orange" width="380px" align="left"/></body></html>
```



### 小结

在这个案例中我们用到了哪些标签？





## 12、超链接标签：基本使用

### 目标

1. 链接标签的基本语法

   ![1551683842614](前端-01-HTML5.assets/1551683842614.png)

2. 点链接调用发邮件的客户端

### 作用

- 可以用于跳转页面
- 可以用于锚点定位



### 语法

```html
<a href="链接的地址">显示的文字</a><a href="链接的地址">   <img src=""/></a>
```

![1551767292933](前端-01-HTML5.assets/1551767292933.png)

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>超链接标签</title>    <!-- a标签，超链接标签        作用1： 跳转页面        a标签常用属性：               href：  跳转资源的地址               target: _blank (新开一个窗口打开) _self(在当前窗口打开，默认)    --></head><body>    <!--跳转到站内资源-->    <a href="2标签的分类.html">查看标签的分类</a><br/>    <!--空连接-->    <a href="#">空链接，没有地址</a><br/>    <!--跳转到站外资源-->    <a href="http://www.itcast.cn" title="点击不见两万,但是可以赚十万" target="_blank" >传智播客</a></body></html>
```

### 调用发邮件客户端语法

```html
    如果同学们缺乏动力，请邮件给我，帮你找回自信：<a href="mailTo://567898@qq.com">邮箱</a>
```

### 小结

超链接标签的作用是什么？哪个属性用于跳转到其它页面？

	跳转资源



## 13、链接标签：设置锚点(重点)

### 目标

如何使用锚点定位到同一个网页中不同的位置？

### 锚点的步骤

锚点的作用：如果一个页面比较长，可以在这个页面不同的位置定义锚点。就可以通过链接跳转到锚点，就可以跳转到同一个网页的不同位置。格式：#锚点名字，可以是字母或数字。

#### 创建锚点

```
<a name="锚点名字">文字</a>
```

#### 使用锚点

```
<a href="#锚点名字">文字</a>
```



### 案例代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>超链接标签</title></head><body><!--  a标签： 超链接标签   作用：    1. 可以用于跳转其他页面（链接其他的资源）    2. 锚点定位   常用的属性：        href: 链接的地址。        target : _self 默认， 当前窗口打开。  _blank 独立一个窗口打开--><!--用于链接外部的资源-->学习java来<a href="http://www.itcast.cn">传智播客</a><br/><!--用于链接内部的资源文件--><a href="03_文本标签.html" target="_blank">点击带你学习文本标签</a><br/><!--启动邮件客户端-->欢迎加入我们，联系我们请发送邮件至：<a href="mailTo://1234567@itcast.cn">1234567@itcast.cn</a><br/><a name="top" style="color: red;font-size: 32px">顶部</a>一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。一年前，习近平总书记对首个“中国医师节”作出的重要指示言犹在耳。他对广大医务人员全心全意为人民健康服务的重要贡献充分肯定，对敬佑生命、救死扶伤、甘于奉献、大爱无疆的精神高度赞誉，激励着广大医务人员为增进人民健康作出新贡献、为健康中国建设谱写新篇章，激励着全社会关心爱护医务人员、形成尊医重卫的良好氛围。<br/><a href="#top" style="color:yellow;font-size: 32px">回到顶部</a></body></html>
```



### 小结

1. 如何定义锚点？

   ```
   
   ```

2. 如何跳转到锚点？

   ```
   
   ```

   

## <font color="red">14、表格标签：基本使用</font>(重点)

### 目标

表格的结构是怎样的  

### 表格的作用

1. 可以进行网页布局(使用div布局代替)
2. 显示服务器的数据

### 表格的结构标签

![1551684133580](前端-01-HTML5.assets/1551684133580.png)

![1551684144361](前端-01-HTML5.assets/1551684144361.png)

### 案例：表格的表头

![1601275080796](前端-01-HTML5.assets/1601275080796.png)

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>表格标签</title>    <!--表格标签的组成元素：            table: 代表是一个表格            tr:    一行            td:    一个单元格            th  : 表头单元格， 表头默认的样式就是居中加粗。            caption: 表格标题标签      常用的属性：            border: 边框大小            width: 宽度            height: 高度            align: 对齐方式            cellspacing : 单元格与单元格之间间距            cellpadding : 数据到单元格边框之间的间距,前提不要使用居中对齐     表格结构组成：         一个表格可以被分为：                thead : 表头, 0~n次                tbody : 表体  1~n次                tfoot : 表尾  0~n次--></head><body>    <table border="1px" width="300px" height="200px" cellspacing="0px" cellpadding="20px" align="center" >        <caption>2020年度最帅人员名单</caption>        <thead>            <!--第一行-->            <tr>                <th>姓名</th>                <th>电话</th>                <th>地址</th>            </tr>        </thead>        <tbody>            <!--第二行-->            <tr align="center">                <td>小明</td>                <td>12345</td>                <td>广州</td>            </tr>            <!--第三行-->            <tr >                <td>小马</td>                <td>110</td>                <td>日本</td>            </tr>        </tbody>    </table></body></html>
```



### 小结

| **表格结构** | **标签** |
| ------------ | -------- |
| **整个表格** | table    |
| **行**       | tr       |
| **列标题**   | th       |
| **单元格**   | td       |
| 表格的标题   | caption  |



## 15、案例：学生成绩表

### 目标

![1601276573257](前端-01-HTML5.assets/1601276573257.png)

### 表格属性

![1551684166720](../../../../../../../assets/1551684166720.png)

### 步骤

1. 表格有标题，使用caption标签
2. 使用thead、tbody、tfoot对表格进行分块
3. 第一行有表格列标题，使用th标签
4. 表格的每一行都居中，使用tr标签的align属性居中对齐
5. 成绩和总成绩分别有跨行和跨列的要求
6. 使用属性要设置表格之间的间隔
7. 使用属性设置文本与表格边框之间的距离

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>表格的案例</title></head><body><table border="1px" bgcolor="#87ffa5" cellspacing="0px" align="center" width="500px" height="300px">    <caption>学生成绩表</caption>    <!--第一行-->    <tr>        <th>编号</th>        <th>姓名</th>        <th>性别</th>        <th>成绩</th>    </tr>    <!--第二行-->    <tr align="center">        <td>100</td>        <td>潘金莲</td>        <td>女</td>        <td>80</td>    </tr>    <!--第三行-->    <tr align="center">        <td>200</td>        <td>武大郎</td>        <td>男</td>        <!--90这个单元格需要占两行, rowspan 指定单元格占的行数-->        <td rowspan="2">90</td>    </tr>    <!--第四行-->    <tr align="center">        <td>300</td>        <td>红太狼</td>        <td>女</td>    </tr>    <!--第一行-->    <tr align="center">        <td>总成绩</td>        <!--colspan ： 指定该单元格占的列数-->        <td colspan="3">900</td>    </tr></table></body></html>
```



### 小结

以上案例中用到了哪些标签？

​		table

​        caption

​        tr

​       th

​      td



## <font color="red">16、表单标签: 介绍和基本属性</font>（最重要）

### 目标

1. 表单标签的作用
2. 表单标签有哪些属性

### 作用

==用于将浏览器端的数据发送给服务器、==

包含其它的表单项：文本框，单选框，下拉列表等

![1564040259129](前端-01-HTML5.assets/1564040259129.png)

![1551770463520](../../../../../../../assets/1551770463520.png)



### 常用属性

![1551770534147](../../../../../../../assets/1551770534147.png)

![1564040840844](前端-01-HTML5.assets/1564040840844.png) 

？ 用来分隔服务器的地址与参数

& 如果有多个参数，使用&分隔

### 案例：表单登录

![1551770885387](../../../../../../../assets/1551770885387.png)

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title><!--    一个表单的根标签form标签， 所有表单项标签都必须在form标签的内部。    表单常用的属性：        action ： 指定表单提交的地址        method ： 指定提交的方式，默认是get的请求方式    get与post请求方式区别：        1. get提交的数据出现在地址上，post提交的数据不会出现在地址栏上。        2. get提交的数据大小有限制，1kb。post请求方式是没有大小限制的。    注意： 表单项如果没有name的属性值，不允许提交    数据提交请求的格式： 服务器url地址?参数名=值1&参数名2=值2...    --></head><body><h3>用户登陆</h3>    <form action="http://www.itcast.cn" method="get">        <!--普通的文本输入框-->        用户名: <input type="text" name="userName"/><br/>        <!--密码框 -->        密&nbsp;&nbsp;码: <input type="password" name="password"/><br/>        <!--提交按钮-->       <input type="submit" value="登录"/>    </form></body></html>
```

### 小结

1. **表单标签的作用?**

- 把数据提交给服务器

2. **表单标签有哪些属性?**
   - action : 指定提交的地址
     - method 	 : 指定提交的方式



## <font color="red">17、表单：文本框、密码框、单选框、复选框</font>

### 目标

​	结合表格布局，制作如图所示的注册页面

### 效果

![1551770654462](../../../../../../../assets/1551770654462.png)

### 步骤

1. 整个表单由8行2列组成，第1列显示文本，可以在td中使用label标签
2. 用户名、密码、性别、爱好、照片使用input标签，设置不同的type属性
3. 学历使用select，个人简介使用textarea
4. 最后1行跨2列，注册、清空、按钮的type分别是submit、reset、button

![1551770703353](前端-01-HTML5.assets/1551770703353.png)

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>用户注册</title>    <!--如果一个表单项不能输入值的，那么必须要指定value属性值，value属性值才是真正提交给服务器的数据--></head><body><form action="http://www.itcast.cn" method="get">    <table>        <caption>            <h3 align="left">用户注册</h3>        </caption>        <!--第一行-->        <tr>            <td>用户名:</td>            <td>                <!--普通文本输入框-->                <input type="text" name="userName"/>            </td>        </tr>        <!--第二行-->        <tr>            <td>密码:</td>            <td>                <!--密码框-->                <input type="password" name="pwd"/>            </td>        </tr>        <!--第三行-->        <tr>            <td>性别:</td>            <td>                <!--单选框                注意： 同一组单选框只能够选择其中的一个，分组是以name的属性值决定                -->                <input type="radio" name="gender" checked="checked" value="man">男                <input type="radio" name="gender" value="woman" >女            </td>        </tr>        <!--第四行-->        <tr>            <td>爱好:</td>            <td>                <!--复选框-->                <input type="checkbox" name="hobbit" checked="checked" value="java"/> java                <input type="checkbox" name="hobbit" value="mysql"/> mysql                <input type="checkbox" name="hobbit" value="php" /> php            </td>        </tr>        <!--第五行-->        <tr>            <td>学历:</td>            <td>                <!--下拉框-->                <select name="xueli">                    <option value="yjs">研究生</option>                    <option value="bk">本科</option>                    <option value="dz">大专</option>                    <option value="gz">高中</option>                    <option value="cz">初中</option>                </select>            </td>        </tr>        <!--第五行-->        <tr>            <td>照片:</td>            <td>                <!--文件上传表单项-->                <input type="file" name="image"/>            </td>        </tr>        <!--第六行-->        <tr>            <td>个人简介</td>            <td>                <!--文本域，文本域的特点：多行多列-->                <textarea name="intro" cols="30" rows="10"></textarea>            </td>        </tr>        <!--第一行-->        <tr>            <td colspan="2">                <!--提交按钮-->                <input type="submit" value="注册"/>                <!--重置按钮-->                <input type="reset" value="清空"/>                <!--普通按钮 需要配合js事件才有作用的。-->                <input type="button" value="按钮" />            </td>        </tr>    </table></form></body></html>
```

### 小结

1. 上面学的所有的标签都是：表单项 input

2. 只是type不同：

3. 下拉列表：select

4. 多行文本域：textarea

   



## 18、表单: 其它控件的学习

### 目标

​	表单中其它控件的作用和常用属性

### 表单控件

![1551770781384](../../../../../../../assets/1551770781384.png)

![1551770803741](前端-01-HTML5.assets/1551770803741.png)

![1551770820907](前端-01-HTML5.assets/1551770820907.png)



## 19、HTML5中新增的type类型(了解)

### 目标

HTML5中新增的几个type作用

### 效果

![](../../../../../../../assets/1551771077442.png)

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>Title</title></head><body><form>生日：<input type="date" name="birthday"/><br/>邮箱：<input type="email" name="email"/><br/>颜色：<input type="color" name="color"/><br/>年龄：<input type="number" name="age"/><br/>    <input type="submit" value="提交"/></form></body></html>
```

### 小结

| **type属性值** | **作用** |
| -------------- | -------- |
| **date**       | 日期     |
| **email**      | 邮箱     |
| **color**      | 颜色     |
| **number**     | 数字     |



## 20、案例：黑马旅游页面的布局

### 目标

![/](前端-01-HTML5.assets/1551684229194.png)

### 步骤

1. 创建最外面的表格，边框、单元格间隔、单元格与内容的间隔都设置成0，宽度100%，居中对齐
2. 上面的4行，设置为thead
3. 中间部分设置为tbody
4. 最后2行，设置为tfoot
5. 创建7行tr和td

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>黑马旅游首页</title></head><body><table width="100%" cellpadding="0px" cellspacing="0px">    <!--网页的头部分-->    <thead>       <!--第一行-->        <tr>            <td></td>        </tr>       <!--第二行-->       <tr>           <td></td>       </tr>       <!--第三行-->       <tr>           <td></td>       </tr>       <!--第四行-->       <tr>           <td></td>       </tr>    </thead>    <!--网页体部分-->    <tbody>        <tr>            <td></td>        </tr>    </tbody>    <!--页脚（页尾）-->    <tfoot>        <tr>            <td></td>        </tr>        <tr>            <td></td>        </tr>    </tfoot></table></body></html>
```



## 21、案例：黑马旅游页面的头部

### 目标

![1551684497512](前端-01-HTML5.assets/1551684497512.png)

### 步骤

1. 第1行：直接在里面插入一张图片top_banner.jpg即可，图片的宽度使用100%

2. 第2行：
   1. 使用表格嵌套，插入一个1行3列的表格。边框、单元格间隔、单元格与内容的间隔都设置成0，宽度100%。
   2. 表格只有1行，全部居中
   3. 第1列的图片：logo.jpg，第2列的图片search.png，宽度设置为500，第3列图片hotel_tel.png
3. 第3行：
   1. 插入一个1行10列的表格
   2. 边框、单元格间隔、单元格与内容的间隔都设置成0，宽度100%，背景色设置#ffc900。
   3. 在每个单元格中输入一个菜单项，每个菜单项上可以加上a标签，单元格的高度设置为45
   4. tr对齐居中，则表格中的文字居中
   5. 上一级的td对齐居中，则表格整个位置居中。
4. 这里只要插入一张图片banner_3.jpg即可，宽度为100%

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>黑马旅游首页</title>    <!--编写css样式，去除超链接下划线与改变颜色-->    <style>        a{            color: black;            text-decoration: none;        }    </style></head><body><table width="100%" cellspacing="0px" cellpadding="0px">    <thead>        <!--第一行-->        <tr>            <td>                <img src="img/top_banner.jpg" width="100%">            </td>        </tr>        <!--第二行-->        <tr>            <td>                <table width="100%" cellspacing="0px" cellpadding="0px">                    <tr align="center">                        <td>                            <img src="img/logo.jpg"/>                        </td>                        <td>                            <img src="img/search.png" width="500px"/>                        </td>                        <td>                            <img src="img/hotel_tel.png"/>                        </td>                    </tr>                </table>            </td>        </tr>        <!--第三行-->        <tr>            <td>                <table width="100%" cellspacing="0px" cellpadding="0px">                    <tr align="center" bgcolor="orange">                        <td height="40px"><a href="#">首页</a></td>                        <td><a href="#">门票</a></td>                        <td><a href="#">酒店</a></td>                        <td><a href="#">国内游</a></td>                        <td><a href="#">酒店</a></td>                        <td><a href="#">国内游</a></td>                        <td><a href="#">酒店</a></td>                        <td><a href="#">国内游</a></td>                        <td><a href="#">酒店</a></td>                        <td><a href="#">国内游</a></td>                    </tr>                </table>            </td>        </tr>        <!--第四行-->        <tr>            <td>                <img src="img/banner_3.jpg" width="100%"/>            </td>        </tr>    </thead>        <tbody>    </tbody>    <tfoot>    </tfoot></table></body></html>
```



## 22、案例：黑马旅游页面的主体

### 目标

![](前端-01-HTML5.assets/1551684497513.png)

### 步骤

1. "黑马精选"、"国内游"、"境外游"三个部分嵌套一个6行1列的表格，边框、内边距、单元格间距都为0，表格宽度为95%，表格设置居中。
2. 其中1，3，5行在td中添加图标icon_5.jpg、icon_6.jpg、icon_7.jpg和文字，下面使用hr添加水平线，粗细2，颜色为：#ffc900，td单元格的高度为80
3. 第2行嵌套一个1行4列的表格，每个单元格插入1张图片jiangxuan_1.jpg和文字。文字使用p。价格使用font标签设置为红色。
4. 第4行嵌套一个1行2列的表格，左边1列单元格插入1张图片guonei_1.jpg，宽度30%，右边1列嵌套一个2行3列的表格，每个单元格插入1张图片和文字。
5. 第6行与第4行的制作方法相同，左边1列的图片名为jiangwai_1.jpg，修改文字和图片即可

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>黑马旅游首页</title>    <!--编写css样式，去除超链接下划线与改变颜色-->    <style>        a{            color: black;            text-decoration: none;        }    </style></head><body><table width="100%" cellspacing="0px" cellpadding="0px">    <thead>        <!--第一行-->        <tr>            <td>                <img src="img/top_banner.jpg" width="100%">            </td>        </tr>        <!--第二行-->        <tr>            <td>                <table width="100%" cellspacing="0px" cellpadding="0px">                    <tr align="center">                        <td>                            <img src="img/logo.jpg"/>                        </td>                        <td>                            <img src="img/search.png" width="500px"/>                        </td>                        <td>                            <img src="img/hotel_tel.png"/>                        </td>                    </tr>                </table>            </td>        </tr>        <!--第三行-->        <tr>            <td>                <table width="100%" cellspacing="0px" cellpadding="0px">                    <tr align="center" bgcolor="orange">                        <td height="40px"><a href="#">首页</a></td>                        <td><a href="#">门票</a></td>                        <td><a href="#">酒店</a></td>                        <td><a href="#">国内游</a></td>                        <td><a href="#">酒店</a></td>                        <td><a href="#">国内游</a></td>                        <td><a href="#">酒店</a></td>                        <td><a href="#">国内游</a></td>                        <td><a href="#">酒店</a></td>                        <td><a href="#">国内游</a></td>                    </tr>                </table>            </td>        </tr>        <!--第四行-->        <tr>            <td>                <img src="img/banner_3.jpg" width="100%"/>            </td>        </tr>    </thead>    <tbody>        <!--第一行-->        <tr>            <td>                <img src="img/icon_5.jpg">黑马精选                <hr color="orange"/>            </td>        </tr>        <!--第二行-->        <tr>            <td>                <table width="100%" cellspacing="0px" cellpadding="0px">                    <tr>                        <td>                            <img src="img/jiangxuan_1.jpg" />                            <p style="font-size: 12px">上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>                            <font color="red">&yen;899</font>                        </td>                        <td>                            <img src="img/jiangxuan_1.jpg" />                            <p style="font-size: 12px">上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>                            <font color="red">&yen;899</font>                        </td>                        <td>                            <img src="img/jiangxuan_1.jpg" />                            <p style="font-size: 12px">上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>                            <font color="red">&yen;899</font>                        </td>                        <td>                            <img src="img/jiangxuan_1.jpg" />                            <p style="font-size: 12px">上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>                            <font color="red">&yen;899</font>                        </td>                    </tr>                </table>            </td>        </tr>        <!--第三行-->        <tr>            <td>                <img src="img/icon_6.jpg">国内游                <hr color="orange"/>            </td>        </tr>        <!--第四行-->        <tr>            <td>                <table width="100%" cellspacing="0px" cellpadding="0px">                    <tr>                        <td>                            <img src="img/guonei_1.jpg"/>                        </td>                        <td>                            <table width="100%" cellspacing="0px" cellpadding="0px">                                <tr>                                    <td>                                        <img src="img/jiangxuan_2.jpg" />                                        <p style="font-size: 12px">上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>                                        <font color="red">&yen;699</font>                                    </td>                                    <td>                                        <img src="img/jiangxuan_2.jpg" />                                        <p style="font-size: 12px">上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>                                        <font color="red">&yen;699</font>                                    </td>                                    <td>                                        <img src="img/jiangxuan_2.jpg" />                                        <p style="font-size: 12px">上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>                                        <font color="red">&yen;699</font>                                    </td>                                </tr>                                <tr>                                    <td>                                        <img src="img/jiangxuan_2.jpg" />                                        <p style="font-size: 12px">上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>                                        <font color="red">&yen;699</font>                                    </td>                                    <td>                                        <img src="img/jiangxuan_2.jpg" />                                        <p style="font-size: 12px">上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>                                        <font color="red">&yen;699</font>                                    </td>                                    <td>                                        <img src="img/jiangxuan_2.jpg" />                                        <p style="font-size: 12px">上海直飞三亚5天4晚自由行(春节预售+亲子/蜜月/休闲游首选+豪华酒店任选+接送机)</p>                                        <font color="red">&yen;699</font>                                    </td>                                </tr>                            </table>                        </td>                    </tr>                </table>            </td>        </tr>    </tbody>    <tfoot>    </tfoot></table></body></html>
```



## 23、案例：黑马旅游页面的底部

### 目标

![1551684792614](前端-01-HTML5.assets/1551684792614.png)

### 步骤

1. 第1行插入图片footer_service.png，宽度100%
2. 第2行输入文字，背景色\#ffc900，内容居中，大小2，颜色用灰色

### 代码

```html
<!DOCTYPE html><html lang="en"><head>    <meta charset="UTF-8">    <title>黑马旅游首页</title></head><body><table width="100%" cellpadding="0" cellspacing="0" align="center">    <!--上部分-->    <thead>        <!--1行-->        <tr>            <td>                <img src="img/top_banner.jpg" width="100%">            </td>        </tr>        <!--2行-->        <tr>            <td>                <!--表格嵌套-->                <table width="100%" cellpadding="0" cellspacing="0" align="center">                    <!--1行3列-->                    <tr align="center">                        <td>                            <img src="img/logo.jpg">                        </td>                        <td>                            <input type="text" size="50" placeholder="请输入线路名字">                            <input type="submit" value="  搜索  ">                        </td>                        <td>                            <img src="img/hotel_tel.png">                        </td>                    </tr>                </table>            </td>        </tr>        <!--3行：导航条-->        <tr>            <td>                <table width="100%" cellpadding="0" cellspacing="0" align="center">                    <!--1行10列-->                    <tr align="center" bgcolor="orange">                        <td height="45">首页</td>                        <td>国内游</td>                        <td>国内游</td>                        <td>国内游</td>                        <td>国内游</td>                        <td>国内游</td>                        <td>国内游</td>                        <td>国内游</td>                        <td>国内游</td>                        <td>国内游</td>                    </tr>                </table>            </td>        </tr>        <!--4行：轮播图-->        <tr>            <td>                <img src="img/banner_3.jpg" width="100%">            </td>        </tr>    </thead>    <!--主体-->    <tbody>        <!--5行-->        <tr>            <td>                <!--6行，1列-->                <table width="95%" cellpadding="0" cellspacing="0" align="center">                    <!--1行-->                    <tr>                        <td height="80">                            <img src="img/icon_5.jpg">黑马精选                            <hr color="orange" size="2"/>                        </td>                    </tr>                    <!--1行-->                    <tr>                        <td>                            <table width="100%" cellpadding="0" cellspacing="0" align="center">                                <!--1行4列-->                                <tr>                                    <td>                                        <img src="img/jiangxuan_1.jpg"><br/>                                        <p>上海直飞三亚5天4晚自由行(春节预售+<br/>亲子/蜜月/休闲游首选+豪华酒店任选+<br/>接送机)</p>                                        <font color="red">&yen;8999</font>                                    </td>                                    <td>                                        <img src="img/jiangxuan_1.jpg"><br/>                                        <p>上海直飞三亚5天4晚自由行(春节预售+<br/>亲子/蜜月/休闲游首选+豪华酒店任选+<br/>接送机)</p>                                        <font color="red">&yen;8999</font>                                    </td>                                    <td>                                        <img src="img/jiangxuan_1.jpg"><br/>                                        <p>上海直飞三亚5天4晚自由行(春节预售+<br/>亲子/蜜月/休闲游首选+豪华酒店任选+<br/>接送机)</p>                                        <font color="red">&yen;8999</font>                                    </td>                                    <td>                                        <img src="img/jiangxuan_1.jpg"><br/>                                        <p>上海直飞三亚5天4晚自由行(春节预售+<br/>亲子/蜜月/休闲游首选+豪华酒店任选+<br/>接送机)</p>                                        <font color="red">&yen;8999</font>                                    </td>                                </tr>                            </table>                        </td>                    </tr>                    <!--1行-->                    <tr>                        <td height="80">                            <img src="img/icon_6.jpg">国内游                            <hr color="orange" size="2"/>                        </td>                    </tr>                    <!--1行-->                    <tr>                        <td>                            <table width="100%" cellpadding="0" cellspacing="0" align="center">                                <!--1行2列-->                                <tr>                                    <td>                                        <img src="img/guonei_1.jpg">                                    </td>                                    <td>                                        <table width="100%" cellpadding="0" cellspacing="0" align="center">                                            <!--2行3列-->                                            <tr>                                                <td>                                                    <img src="img/jiangxuan_2.jpg"><br/>                                                    <p>上海直飞三亚5天4晚自由行(春节预售+<br/>亲子/蜜月/休闲游首选+豪华酒店任选)</p>                                                    <font color="red">&yen;8999</font>                                                </td>                                                <td>                                                    <img src="img/jiangxuan_2.jpg"><br/>                                                    <p>上海直飞三亚5天4晚自由行(春节预售+<br/>亲子/蜜月/休闲游首选+豪华酒店任选)</p>                                                    <font color="red">&yen;8999</font>                                                </td>                                                <td>                                                    <img src="img/jiangxuan_2.jpg"><br/>                                                    <p>上海直飞三亚5天4晚自由行(春节预售+<br/>亲子/蜜月/休闲游首选+豪华酒店任选)</p>                                                    <font color="red">&yen;8999</font>                                                </td>                                            </tr>                                            <tr>                                                <td>                                                    <img src="img/jiangxuan_2.jpg"><br/>                                                    <p>上海直飞三亚5天4晚自由行(春节预售+<br/>亲子/蜜月/休闲游首选+豪华酒店任选)</p>                                                    <font color="red">&yen;8999</font>                                                </td>                                                <td>                                                    <img src="img/jiangxuan_2.jpg"><br/>                                                    <p>上海直飞三亚5天4晚自由行(春节预售+<br/>亲子/蜜月/休闲游首选+豪华酒店任选)</p>                                                    <font color="red">&yen;8999</font>                                                </td>                                                <td>                                                    <img src="img/jiangxuan_2.jpg"><br/>                                                    <p>上海直飞三亚5天4晚自由行(春节预售+<br/>亲子/蜜月/休闲游首选+豪华酒店任选)</p>                                                    <font color="red">&yen;8999</font>                                                </td>                                            </tr>                                        </table>                                    </td>                                </tr>                            </table>                        </td>                    </tr>                </table>            </td>        </tr>    </tbody>    <!--下部分-->    <tfoot>        <!--6行-->        <tr>            <td>                <img src="img/footer_service.png" width="100%">            </td>        </tr>        <!--7行-->        <tr>            <td align="center" height="45" bgcolor="orange">                <font color="grey">江苏传智播客教育科技股份有限公司 版权所有Copyright 2006-2018, All Rights Reserved 苏ICP备16007882</font>            </td>        </tr>    </tfoot></table></body></html>
```



