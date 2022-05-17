---
title: 常见面试题总结(06)-Spring框架相关的常见面试题
tags:
  - 笔记
  - 就业技术加强
  - 源码分析
  - Spring
  - 常见面试题总结
categories:
  - 常见面试题总结
date: 2021-01-29 23:28:56
---

## **面试题1**：Spring框架中应用了哪些常见的设计模式

1. 单例模式:
   保证一个类仅有一个实例，Spring管理的bean默认就是单例的。
   
2. 策略模式:
   定义一系列的算法，把它们一个个封装起来，并且使它们可相互替换，Spring中的资源加载。
  
3. 工厂方法模式:
   Spring中的FactoryBean就是典型的工厂方法模式。
   
4. 代理模式:
   Spring的Proxy模式在aop中有体现，比如JdkDynamicAopProxy和Cglib2AopProxy。
   
5. 观察者模式:
   Spring中Observer模式常用的地方是listener的实现, 如ApplicationListener。
   
6. 模板方法模式:
   定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。Spring框架用到了很多抽像类。

## **面试题2**：Spring框架中的IoC是什么?

1. Spring IoC(Inversion of Control)指的是控制反转，IoC容器负责bean的实例化、管理bean与bean之间的依赖关系，它帮我们完成bean之间的依赖注入，控制权交由Spring容器，从而实现bean与bean之间松耦合。
2. 以前控制权是当前bean对象，现在控制权交由Spring容器，这种行为就是控制反转(IoC)。

## **面试题3**： Spring框架中的单例bean线程安全吗?

不，Spring框架中的单例bean不是线程安全的。

Spring作用域（scope）配置的区别：
- 非线程安全：singleton（默认）: Spring容器只存在一个共享的bean实例。
- 线程安全：  prototype:        每次对bean的请求都会创建一个新的bean实例。

## **面试题4**：Spring框架中的AOP是什么?

1. AOP(Aspect-Oriented Programming)面向切面编程，对bean做增强处理的。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
2. 主要的意图是: 将日志记录、事务处理、异常处理、性能统计、安全控制等代码从原有业务逻辑代码中分离出来，将它们抽取到非业务逻辑的方法中，利用AOP切面可以将这些方法切入，从而达到原有的业务逻辑代码效果。

