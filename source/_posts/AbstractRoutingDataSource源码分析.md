---
title: AbstractRoutingDataSource源码分析
tags:
  - 笔记
  - 数据库
  - 多数据源
  - 源码分析
  - 动态切换数据源
categories:
  - 数据库
date: 2022-05-16 20:52:27
---

spring为我们提供了AbstractRoutingDataSource抽象类，该类就是**实现动态切换数据源**的关键。

我们看下该类的类图，其实现了DataSource接口。

![](AbstractRoutingDataSource源码分析.assets/2022-05-16-22-25-38-image.png)

我们看下它的getConnection方法的逻辑，其首先调用determineTargetDataSource来获取数据源，再获取数据库连接。很容易猜想到就是这里来决定具体使用哪个数据源的。

![](AbstractRoutingDataSource源码分析.assets/2022-05-16-22-27-27-image.png)

进入到determineTargetDataSource方法，我们可以看到它先是调用determineCurrentLookupKey获取到一个lookupKey，然后根据这个key去resolvedDataSources里去找相应的数据源。

![](AbstractRoutingDataSource源码分析.assets/2022-05-16-22-27-38-image.png)

看下该类定义的几个对象，defaultTargetDataSource是默认数据源，resolvedDataSources是一个map对象，存储所有主从数据源。

所以，关键就是这个lookupKey的获取逻辑，决定了当前获取的是哪个数据源，然后执行getConnection等一系列操作。determineCurrentLookupKey是AbstractRoutingDataSource类中的一个抽象方法，而它的返回值是你所要用的数据源dataSource的key值，有了这个key值，resolvedDataSource（这是个map,由配置文件中设置好后存入的）就从中取出对应的DataSource，如果找不到，就用配置默认的数据源。

所以，通过扩展AbstractRoutingDataSource类，并重写其中的determineCurrentLookupKey()方法，可以实现数据源的切换。
