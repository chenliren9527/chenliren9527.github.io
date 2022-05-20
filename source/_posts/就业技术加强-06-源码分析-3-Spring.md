---
title: 就业技术加强(06)-源码分析(3)-Spring
tags:
  - 笔记
  - 就业技术加强
  - 源码分析
  - Spring
categories:
  - 就业技术加强
abbrlink: 22208
date: 2021-01-29 18:27:32
---

## 0、课程目标

目标1: 了解spring框架体系结构

目标2: 了解spring框架容器初始化流程

目标3: 掌握spring框架相关的常见面试题

## 1、工厂体系结构

### 1.1、导入演示工程

将 `资料\演示工程\spring-demo` 导入到IDEA中。

![image-20210104204133991](就业技术加强-06-源码分析-3-Spring.assets/image-20210104204133991.png)

### 1.2、工厂继承关系

关于 `ApplicationContext` 的继承关系：

![image-20210104210840859](就业技术加强-06-源码分析-3-Spring.assets/image-20210104210840859.png)

关于 `ClassPathXmlApplicationContext` 的继承关系：

先找到 `BeanFactory` 然后按 `ctrl + h` 

![image-20210104211815072](就业技术加强-06-源码分析-3-Spring.assets/image-20210104211815072.png)

| 名称                                  | 描述                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| BeanFactory                           | spring框架工厂体系结构的顶层接口，提供了基础规范：获取bean对象、bean的作用范围、bean的类型。 |
| ListableBeanFactory                   | BeanFactory接口中的getBean方法只能获取单个对象。ListableBeanFactory可以获取多个对象 |
| HierarchicalBeanFactory               | 提供父容器的访问功能。在一个spring应用中，支持有多个BeanFactory，并且可以设置为它们的父子关系。比如ssm框架整合中的两个IoC容器 |
| ApplicationContext                    | 项目中直接使用的工厂接口，它同时继承了ListableBeanFactory和HierarchicalBeanFactory接口 |
| AbstractApplicationContext            | ApplicationContext工厂抽象类，提供了IoC容器初始化公共实现    |
| AbstractRefreshableApplicationContext | 在AbstractApplicationContext基础上，增加了IoC容器重建支持    |
| AbstractXmlApplicationContext         | 增加了配置文件解析处理                                       |
| ClassPathXmlApplicationContext        | 项目中，直接使用的工厂实现类。从类的根路径下加载配置文件，创建spring IoC容器 |
| DefaultListableBeanFactory            | 在spring框架工厂体系结构中，它是最强大的工厂类，也是我们最终创建的IoC容器，它内部持有了一系列Map集合。 |

## 2、容器初始化

### 2.1、初始化时序图

![image-20210105154237352](就业技术加强-06-源码分析-3-Spring.assets/image-20210105154237352.png)

### 2.2、核心源码

从初始化开始时候调试起：

![image-20210105113050523](就业技术加强-06-源码分析-3-Spring.assets/image-20210105113050523.png)

![image-20210105113056730](就业技术加强-06-源码分析-3-Spring.assets/image-20210105113056730.png)

![image-20210105113103235](就业技术加强-06-源码分析-3-Spring.assets/image-20210105113103235.png)

上述使用到了 `ClassPathXmlApplicationContext` 

```java
public class ClassPathXmlApplicationContext extends AbstractXmlApplicationContext {
     
    // 资源配置文件成员变量，是一个数组，支持多个spring的配置文件
     @Nullable
    private Resource[] configResources;

     // 默认构造方法
    public ClassPathXmlApplicationContext() {
    }

    // 如果已经存在一个ioc容器，可以在构造的时候设置【父】容器
    public ClassPathXmlApplicationContext(ApplicationContext parent) {
        super(parent);
    }

     // 1. 根据xxx.xml配置文件，创建ioc容器
    public ClassPathXmlApplicationContext(String configLocation)throws BeansException {
        this(new String[]{configLocation}, true, (ApplicationContext)null);
    }

    ..........................................

   /**
    * 2. 方法说明：
    *	根据xml文件的定义，以及父容器，创建一个新的ClassPathXmlApplicationContext
    *
    *参数说明：
    *	configLocations：xml配置文件数组
    *	refresh：是否要重新创建ioc容器。加载全部bean的定义和创建所有的单例对象
    *	parent：父容器
    */
    public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, 
                  @Nullable ApplicationContext parent) throws BeansException {
        super(parent);// 设置父容器
    
        // 根据提供的路径，处理成配置文件数组(以分号、逗号、空格、tab、换行符分割)
        this.setConfigLocations(configLocations);
        if (refresh) {
            this.refresh();// 3. 【核心方法】：该方法表示初始化（或者重建）ioc容器。即可以把原
                           // 来的ApplicationContext销毁，重新执行初始化创建
        }
    }
    ..........................................
}
```



也使用到了`AbstractApplicationContext`

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader implements ConfigurableApplicationContext {
     ..........................................
     /**
     * 4.方法说明：
     *	【核心方法】：该方法表示初始化（或者重建）ioc容器。即可以把原来的
     *   ApplicationContext销毁，重新执行初始化创建 
     */
    public void refresh() throws BeansException, IllegalStateException {
      
       // 创建ioc容器，同步加锁，保障线程安全
       synchronized (this.startupShutdownMonitor) {
			// 准备工作：记录容器启动的时间 和 容器激活状态
			prepareRefresh();

			// 【关键步骤】 获取新的bean工厂(obtainFreshBeanFactory)
           	 //	1.根据配置文件中的内容，解析成一个个Bean定义实例（BeanDefinition）
             //	2.将一个个Bean定义实例，注册到BeanFactory中
           	 // 3.细节：此时还没有真正创建对应的bean对象
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// 为BeanFactory设置其它信息:
           	 // 1.设置bean类加载器
             // 2.设置Bean表达式解析器
           	 // 3.设置BeanPostProcessor（bean后置处理器）
           	 // 4.设置资源加载器
             // ....
			prepareBeanFactory(beanFactory);

			try {
				// 设置BeanFactoryPostProcessors(预留方法)
				postProcessBeanFactory(beanFactory);

			    // 调用BeanFactoryPostProcessor各个实现类的 
                 // postProcessBeanFactory(factory) 方法
                 // 可以理解为是给BeanFactory提供的一种扩展机制。比如可以让我们的
                 // BeanFactory实现BeanFactoryPostProcessor接口，增强该BeanFactory的功能
				invokeBeanFactoryPostProcessors(beanFactory);

				// 注册BeanPostProcessor的实现类：
                  // 1.该接口有两个方法：
                  // postProcessBeforeInitialization(), 在init-method属性指定的方法前调用
                  // postProcessAfterInitialization(),  在init-method属性指定的方法后调用
				registerBeanPostProcessors(beanFactory);

				// 初始化国际化支持的资源文件
				initMessageSource();

				// 初始化ApplicationContext事件广播器
				initApplicationEventMulticaster();

				// 模板方法：用于特殊bean的初始化，默认是空实现（在api中如果预留了一些方法
                 // 实现是空，表示该方法是留给子类自我实现。那么这些方法称为：钩子方法）
				onRefresh();

				// 注册事件监听器：监听器需要实现ApplicationListener接口
				registerListeners();

                  // 【关键步骤】实例化所有非延迟加载的单例bean对象
                  // <bean id="userDao" class="com.itheima.spring.dao.impl.UserDaoImpl" 
                  //     lazy-init="false" scope="singleton"/>
				finishBeanFactoryInitialization(beanFactory);
                
                  // 发布广播事件。ApplicationContext初始化完成
				finishRefresh();
			}catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization 
                                   - " + "cancelling refresh attempt: " + ex);
				}

                 // 如果发生异常，需要销毁已经创建的singleton对象
				destroyBeans();

                  // 将active状态设置为false
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
    }  
    ..........................................   
}
```

### 2.3、主体流程

![spring-ioc-refresh](就业技术加强-06-源码分析-3-Spring.assets/spring-ioc-refresh.png)



### 2.4、准备刷新容器

![image-20210105144856121](就业技术加强-06-源码分析-3-Spring.assets/image-20210105144856121.png)

```java
/**
 * 方法说明：创建bean容器前的预备工作
 * 1.设置容器启动时间
 * 2.设置closed状态
 * 3.设置active状态
 */
protected void prepareRefresh() {
	// 容器启动时间（当前系统时间）
	this.startupDate = System.currentTimeMillis();
	// closed状态，false表示容器不是关闭状态
	this.closed.set(false);
	// active状态,true表示容器是激活状态
	this.active.set(true);

	if (logger.isInfoEnabled()) {
		logger.info("Refreshing " + this);
	}

	// 初始化加载属性资源文件。该方法是空实现，留给子类实现特殊处理
	initPropertySources();

	// 准备校验系统环境属性资源。比如使用@Value注解中系统环境中取值：
	// @Value("${jdbc.driver}")
	getEnvironment().validateRequiredProperties();

	// 初始化早期应用的事件集合，它是一个Set
	this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

### 2.5、创建BeanFactory

![image-20210105175539262](就业技术加强-06-源码分析-3-Spring.assets/image-20210105175539262.png)

![image-20210105175546970](就业技术加强-06-源码分析-3-Spring.assets/image-20210105175546970.png)

![image-20210105175553774](就业技术加强-06-源码分析-3-Spring.assets/image-20210105175553774.png)

跟进 `loadBeanDefinitions(beanFactory)` 方法查看，在配置文件中定义的两个Bean是否被解析为 `BeanDefinition`

![image-20210105183046353](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183046353.png)

![image-20210105183141914](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183141914.png)

![image-20210105183154837](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183154837.png)

![image-20210105183204568](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183204568.png)

![image-20210105183213763](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183213763.png)

![image-20210105183223953](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183223953.png)

![image-20210105183234918](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183234918.png)

![image-20210105183245051](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183245051.png)

![image-20210105183257891](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183257891.png)

![image-20210105183311617](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183311617.png)

![image-20210105183321121](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183321121.png)

![image-20210105183330641](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183330641.png)

![image-20210105183402461](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183402461.png)

![image-20210105183500635](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183500635.png)

![image-20210105183537248](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183537248.png)

![image-20210105183547930](就业技术加强-06-源码分析-3-Spring.assets/image-20210105183547930.png)



在解析创建完 `BeanDefinition` 后返回 `BeanFactory`

![image-20210105175908999](就业技术加强-06-源码分析-3-Spring.assets/image-20210105175908999.png)



### 2.6、创建单例Bean

到这一步，已经完成了spring配置文件的解析，将相关的配置信息封装成了`BeanDefinition`对象，同时将`BeanDefinition`对象，注册到了BeanFactory中。
**需要注意**：真正的Bean实例对象还没有创建，接下跟进，创建实例化bean的流程：
 跟进入口 `refresh()`方法中的  `finishBeanFactoryInitialization(beanFactory)`

先跟进源码：

![image-20210105205747877](就业技术加强-06-源码分析-3-Spring.assets/image-20210105205747877.png)

![image-20210105205756511](就业技术加强-06-源码分析-3-Spring.assets/image-20210105205756511.png)

![image-20210105205807580](就业技术加强-06-源码分析-3-Spring.assets/image-20210105205807580.png)

![image-20210105205814639](就业技术加强-06-源码分析-3-Spring.assets/image-20210105205814639.png)

![image-20210105205840012](就业技术加强-06-源码分析-3-Spring.assets/image-20210105205840012.png)

![image-20210105205845904](就业技术加强-06-源码分析-3-Spring.assets/image-20210105205845904.png)

![image-20210105205857038](就业技术加强-06-源码分析-3-Spring.assets/image-20210105205857038.png)

![image-20210105205905194](就业技术加强-06-源码分析-3-Spring.assets/image-20210105205905194.png)

![image-20210105205913962](就业技术加强-06-源码分析-3-Spring.assets/image-20210105205913962.png)

![image-20210105205923051](就业技术加强-06-源码分析-3-Spring.assets/image-20210105205923051.png)

![image-20210105205936363](就业技术加强-06-源码分析-3-Spring.assets/image-20210105205936363.png)

![image-20210105205947661](就业技术加强-06-源码分析-3-Spring.assets/image-20210105205947661.png)

![image-20210105210406145](就业技术加强-06-源码分析-3-Spring.assets/image-20210105210406145.png)



在创建完Bean之后；在如下方法对Bean属性进行设置值：

![image-20210105210641842](就业技术加强-06-源码分析-3-Spring.assets/image-20210105210641842.png)

![image-20210105210718506](就业技术加强-06-源码分析-3-Spring.assets/image-20210105210718506.png)

再小结如下：

```
1.调用【AbstractApplicationContext】finishBeanFactoryInitialization方法：
	初始化创建全部单例【singleton】对象的入口方法
		
2.调用【DefaultListableBeanFactory】preInstantiateSingletons方法：
	循环遍历创建所有（非抽象、非延迟加载、是单例）的bean对象入口方法
		
3.调用【AbstractBeanFactory】getBean方法：
	直接调用本类中的doGetBean方法
	
4.调用【AbstractBeanFactory】doGetBean方法：
	从容器中获取目标bean。如果目标bean已经存在，直接获取返回；如果目标bean不存在，则创建目标bean
	
5.调用【AbstractAutowireCapableBeanFactory】createBean方法：
	加载目标bean字节码Class，创建bean对象
	
6.调用【AbstractAutowireCapableBeanFactory】doCreateBean方法：
	实例化bean，设置依赖注入，进行后置回调处理
	
7.调用【AbstractAutowireCapableBeanFactory】createBeanInstance方法：
	通过三种实例化bean方式实例化（工厂方法、构造方法依赖注入、无参数构造方法）
	
8.调用【AbstractAutowireCapableBeanFactory】instantiateBean方法：
	使用无参数构造方法实例化bean
	
9.调用【SimpleInstantiationStrategy】instantiate方法：
	通过反射技术实例化bean对象
```

## 3、常见面试题

**面试题1**：Spring框架中应用了哪些常见的设计模式

```txt
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
```

**面试题2**：Spring框架中的IoC是什么?

```text
1. Spring IoC(Inversion of Control)指的是控制反转，IoC容器负责bean的实例化、管理bean与bean之间的依赖关系，它帮我们完成bean之间的依赖注入，控制权交由Spring容器，从而实现bean与bean之间松耦合。
2. 以前控制权是当前bean对象，现在控制权交由Spring容器，这种行为就是控制反转(IoC)。
```

**面试题3**： Spring框架中的单例bean线程安全吗?

```txt
不，Spring框架中的单例bean不是线程安全的。

Spring作用域（scope）配置的区别：
- 非线程安全：singleton（默认）: Spring容器只存在一个共享的bean实例。
- 线程安全：  prototype:        每次对bean的请求都会创建一个新的bean实例。
```

**面试题4**：Spring框架中的AOP是什么?

```txt
1. AOP(Aspect-Oriented Programming)面向切面编程，对bean做增强处理的。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
2. 主要的意图是: 将日志记录、事务处理、异常处理、性能统计、安全控制等代码从原有业务逻辑代码中分离出来，将它们抽取到非业务逻辑的方法中，利用AOP切面可以将这些方法切入，从而达到原有的业务逻辑代码效果。
```