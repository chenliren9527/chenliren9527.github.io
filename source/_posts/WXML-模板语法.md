---
title: WXML 模板语法
tags:
  - 小程序
  - 微信
  - 笔记
categories:
  - 微信小程序
abbrlink: 48684
date: 2023-04-05 21:10:06
---

# WXML模板语法

## 数据绑定

### 1. 数据绑定的基本原则

1. 在 data 中定义数据
2. 在 WXML 中使用数据

### 2. 在 data 中定义页面的数据

在页面对应的 .js 文件中，把数据定义到 data 对象中即可：

![image-20230405182344632](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405182344632.png)

### 3. Mustache 语法的格式

把data中的数据绑定到页面中渲染，使用 **Mustache** **语法**（双大括号）将变量包起来即可。语法格式为：

![image-20230405182411632](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405182411632.png)

### 4. Mustache 语法的应用场景

Mustache 语法的主要应用场景如下

- 绑定内容
- 绑定属性
- 运算（三元运算、算术运算等）

### 5. 动态绑定内容

页面的数据如下：

![image-20230405182515815](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405182515815.png)

页面的结构如下：

![image-20230405182530821](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405182530821.png)

### 6. 动态绑定属性

页面的数据如下：

![image-20230405182557996](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405182557996.png)

页面的结构如下：

![image-20230405182603426](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405182603426.png)

### 7. 三元运算

页面的数据如下：

![image-20230405185903255](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405185903255.png)

页面的结构如下：

![image-20230405185915525](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405185915525.png)

### 8. 算数运算

页面的数据如下：

![image-20230405185930990](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405185930990.png)

页面的结构如下：

![image-20230405185939356](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405185939356.png)

## 事件绑定

### 1. 什么是事件

事件是渲染层到逻辑层的通讯方式。通过事件可以将用户在渲染层产生的行为，反馈到逻辑层进行业务的处理。

![image-20230405190012024](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190012024.png)

### 2. 小程序中常用的事件

![image-20230405190031107](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190031107.png)

### 3. 事件对象的属性列表

当事件回调触发的时候，会收到一个事件对象 event，它的详细属性如下表所示：

![image-20230405190045445](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190045445.png)

### 4. target 和 currentTarget 的区别

target 是触发该事件的源头组件，而 currentTarget 则是当前事件所绑定的组件。举例如下：

![image-20230405190139299](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190139299.png)

点击内部的按钮时，点击事件以冒泡的方式向外扩散，也会触发外层 view 的 tap 事件处理函数。

此时，对于外层的 view 来说：

- e.target 指向的是触发事件的源头组件，因此，e.target 是内部的按钮组件
- e.currentTarget 指向的是当前正在触发事件的那个组件，因此，e.currentTarget 是当前的 view 组件

### 5. bindtap 的语法格式

在小程序中，不存在 HTML 中的 onclick 鼠标点击事件，而是通过 tap 事件来响应用户的触摸行为。

1. 通过 bindtap，可以为组件绑定 tap 触摸事件，语法如下：


   ![image-20230405190227891](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190227891.png)

2. 在页面的 .js 文件中定义对应的事件处理函数，事件参数通过形参 event（一般简写成 e） 来接收：

   ![image-20230405190241232](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190241232.png)

### 6. 在事件处理函数中为 data 中的数据赋值

通过调用 this.setData(dataObject) 方法，可以给页面 data 中的数据重新赋值，示例如下：

![image-20230405190357161](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190357161.png)

### 7. 事件传参

小程序中的事件传参比较特殊，不能在绑定事件的同时为事件处理函数传递参数。例如，下面的代码将不能正常工作：

![image-20230405190422109](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190422109.png)

因为小程序会把 bindtap 的属性值，统一当作事件名称来处理，相当于要调用一个名称为 btnHandler(123) 的事件处理函数。

可以为组件提供 data-* 自定义属性传参，其中 * 代表的是参数的名字，示例代码如下：

![image-20230405190455053](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190455053.png)

最终：

- info 会被解析为参数的名字
- 数值 2 会被解析为参数的值

在事件处理函数中，通过 event.target.dataset.参数名 即可获取到具体参数的值，示例代码如下：

![image-20230405190526588](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190526588.png)

### 8. bindinput 的语法格式

在小程序中，通过 input 事件来响应文本框的输入事件，语法格式如下：

1. 通过 bindinput，可以为文本框绑定输入事件：

   ![image-20230405190603909](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190603909.png)

2. 在页面的 .js 文件中定义事件处理函数：

   ![image-20230405190618670](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190618670.png)

### 9. 实现文本框和 data 之间的数据同步

实现步骤：

1. 定义数据

2. 渲染结构

3. 美化样式

4. 绑定 input 事件处理函数

定义数据：

![image-20230405190658465](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190658465.png)

渲染结构：

![image-20230405190707441](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190707441.png)

美化样式：

![image-20230405190715837](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190715837.png)

绑定 input 事件处理函数：

![image-20230405190723410](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190723410.png)

## 条件渲染

### 1. wx:if

在小程序中，使用` wx:if="{{condition}}" `来判断是否需要渲染该代码块：

![image-20230405190805050](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190805050.png)

也可以用 wx:elif 和 wx:else 来添加 else 判断：

![image-20230405190811674](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405190811674.png)

### 2. 结合 `<block> `使用 wx:if

如果要一次性控制多个组件的展示与隐藏，可以使用一个` <block></block> `标签将多个组件包装起来，并在`<block> `标签上使用 wx:if 控制属性，示例如下：

![image-20230405195637084](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405195637084.png)

注意：` <block>` 并不是一个组件，它只是一个包裹性质的容器，不会在页面中做任何渲染。

### 3. hidden

在小程序中，直接使用 hidden="{{ condition }}" 也能控制元素的显示与隐藏：

![image-20230405195719964](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405195719964.png)

### 4. wx:if 与 hidden 的对比

1. 运行方式不同

   - wx:if 以动态创建和移除元素的方式，控制元素的展示与隐藏

   - hidden 以切换样式的方式（display: none/block;），控制元素的显示与隐藏

2. 使用建议

   - 频繁切换时，建议使用 hidden

   -  控制条件复杂时，建议使用 wx:if 搭配 wx:elif、wx:else 进行展示与隐藏的切换

## 列表渲染

### 1. wx:for

通过 wx:for 可以根据指定的数组，循环渲染重复的组件结构，语法示例如下：

![image-20230405195923797](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405195923797.png)

默认情况下，当前循环项的索引用 index 表示；当前循环项用 item 表示。

### 2. 手动指定索引和当前项的变量名*

- 使用 wx:for-index 可以指定当前循环项的索引的变量名
- 使用 wx:for-item 可以指定当前项的变量名

示例代码如下：

![image-20230405200016926](WXML-%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405200016926.png)

### 3. wx:key 的使用

类似于 Vue 列表渲染中的 **:key**，小程序在实现列表渲染时，也建议为渲染出来的列表项指定唯一的 key 值，从而提高渲染的效率，示例代码如下：

![image-20230405200056188](4.WXML%20%E6%A8%A1%E6%9D%BF%E8%AF%AD%E6%B3%95.assets/image-20230405200056188.png)
