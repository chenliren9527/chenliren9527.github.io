---
title: 乐优电商(11)-用户注册【RabbitMQ】
tags:
  - 笔记
  - 项目实战二
  - springcloud
  - 微服务
  - RabbitMQ
  - SpringDataRedis
  - 阿里短信
  - Redis
categories:
  - 项目实战二
abbrlink: 11033
date: 2021-01-07 20:27:48
---

## 01、课程目标

- 会使用SpringDataRedis

- 会使用阿里短信SDK发送短信
- 了解面向接口开发方式
- 实现数据校验功能
- 实现短信发送功能
- 实现注册功能





## 02、数据同步：思路分析及常量准备

我们已经完成了对MQ的基本学习和认识。接下来，我们就改造项目，实现搜索服务、商品静态页的数据同步。

### 1）思路分析

> 发送方：商品微服务

- 什么时候发？

  当商品服务对商品进行新增和上下架的时候，需要发送一条消息，通知其它服务。

- 发送什么内容？

  对商品的增删改时其它服务可能需要新的商品数据，但是如果消息内容中包含全部商品信息，数据量太大，而且并不是每个服务都需要全部的信息。因此我们**只发送商品id**，其它服务可以根据id查询自己需要的信息。

> 接收方：搜索微服务、静态页微服务

- 接收消息后如何处理？
  - 搜索微服务：
    - 上架：添加新的数据到索引库
    - 下架：删除索引库数据
  - 静态页微服务：
    - 上架：创建新的静态页
    - 下架：删除原来的静态页

![商品上下架数据同步](./day11-用户注册【RabbitMQ】.assets/商品上下架数据同步.png)



### 2）常量准备

在`ly-common`中编写一个常量类，记录将来会用到的Exchange名称、Queue名称、routing_key名称

```java
package com.leyou.common.constants;

/**
 * @author 黑马程序员
 */
public abstract class MQConstants {

    public static final class Exchange {
        /**
         * 商品服务交换机名称
         */
        public static final String ITEM_EXCHANGE_NAME = "ly.item.exchange";
    }

    public static final class RoutingKey {
        /**
         * 商品上架的routing-key
         */
        public static final String ITEM_UP_KEY = "item.up";
        /**
         * 商品下架的routing-key
         */
        public static final String ITEM_DOWN_KEY = "item.down";
    }

    public static final class Queue{
        /**
         * 搜索服务，商品上架的队列
         */
        public static final String SEARCH_ITEM_UP = "search.item.up.queue";
        /**
         * 搜索服务，商品下架的队列
         */
        public static final String SEARCH_ITEM_DOWN = "search.item.down.queue";

        /**
         * 商品详情服务，商品上架的队列
         */
        public static final String PAGE_ITEM_UP = "page.item.up.queue";
        /**
         * 商品详情服务，商品下架的队列
         */
        public static final String PAGE_ITEM_DOWN = "page.item.down.queue";
    }
}
```

这些常量我们用一个图来展示用在什么地方：

![image-20200323142920428](./day11-用户注册【RabbitMQ】.assets/image-20200323142920428.png)







## 03、数据同步：商品微服务发送消息

我们先在商品微服务`ly-item`中实现发送消息。

### 1）引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### 2）配置文件

我们在application.yml中添加一些有关RabbitMQ的配置：

```yaml
spring:
  rabbitmq:
    host: 127.0.0.1
    username: leyou
    password: leyou
    virtual-host: /leyou
    publisher-confirms: true
    template:
      retry:
        enabled: true
        initial-interval: 10000ms
        max-interval: 80000ms
        multiplier: 2
```

- template：有关`AmqpTemplate`的配置
  - retry：失败重试
    - enabled：开启失败重试
    - initial-interval：第一次重试的间隔时长
    - max-interval：最长重试间隔，超过这个间隔将不再重试
    - multiplier：下次重试间隔的倍数，此处是2即下次重试间隔是上次的2倍
  - exchange：缺省的交换机名称，此处配置后，发送消息如果不指定交换机就会使用这个
- publisher-confirms：生产者确认机制，确保消息会正确发送，如果发送失败会有错误回执，从而触发重试

### 3）Json消息序列化方式（选做）

需要注意的是，默认情况下，AMQP会使用JDK的序列化方式进行处理，传输数据比较大，效率太低。我们可以自定义消息转换器，使用JSON来处理：

```java
package com.leyou.item.config;

import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 配置json消息转换
 */
@Configuration
public class RabbitConfig {

    @Bean
    public Jackson2JsonMessageConverter jsonMessageConverter(){
        return new Jackson2JsonMessageConverter();
    }

}


```

### 4）改造GoodsService

改造GoodsService中的商品上下架功能，发送消息，注意用静态导入方式，导入在ly-common中定义的常量：

```java
package com.leyou.item.service;

import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import com.leyou.common.constant.MQConstants;
import com.leyou.common.exception.pojo.ExceptionEnum;
import com.leyou.common.exception.pojo.LyException;
import com.leyou.common.pojo.PageResult;
import com.leyou.common.utils.BeanHelper;
import com.leyou.item.dto.SpuDTO;
import com.leyou.item.entity.*;
import com.leyou.item.mapper.SkuMapper;
import com.leyou.item.mapper.SpuDetailMapper;
import com.leyou.item.mapper.SpuMapper;
import org.apache.commons.lang3.StringUtils;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.util.CollectionUtils;
import tk.mybatis.mapper.entity.Example;

import java.util.Date;
import java.util.List;
import java.util.stream.Collectors;

import static com.leyou.common.constant.MQConstants.RoutingKey.ITEM_UP_KEY;

@Service
@Transactional
public class GoodsService {

    ……

    @Autowired
    private AmqpTemplate amqpTemplate;


    
   public void updateSaleable(Long id, Boolean saleable) {

        try {
            Spu spu = new Spu();
            spu.setId(id);
            spu.setSaleable(saleable);

            spuMapper.updateById(spu);//注意：updateById该方法会自动判断NULL不更新的

            //定义上下架的routingKey
            String routingKey = saleable ? MQConstants.RoutingKey.ITEM_UP_KEY : MQConstants.RoutingKey.ITEM_DOWN_KEY;
            //发送消息给MQ
            amqpTemplate.convertAndSend(MQConstants.Exchange.ITEM_EXCHANGE_NAME,routingKey,id);
        } catch (Exception e) {
            e.printStackTrace();
            throw new LyException(ExceptionEnum.UPDATE_OPERATION_FAIL);
        }
    }

    
    ……
}

```

注意：此刻不能启动项目测试，因为rabbitMQ中还没有交换机和队列。





## 04、数据同步：搜索微服务接收消息

搜索服务接收到消息后要做的事情：

- 上架：添加新的数据到索引库
- 下架：删除索引库数据

我们需要两个不同队列，监听不同类型消息。

### 1）引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```



### 2）添加配置

```yaml
spring:
  rabbitmq:
    host: 127.0.0.1
    username: leyou
    password: leyou
    virtual-host: /leyou
```

这里只是接收消息而不发送，所以不用配置template相关内容。

### 3）Json消息序列化配置

不过，不要忘了消息转换器（**如果生产者没有加消息转换器，那么消费者也不要加**）：

![1553262961034](./day11-用户注册【RabbitMQ】.assets/1553262961034.png) 

```java
/**
 * @author 黑马程序员
 */
@Configuration
public class RabbitConfig {

    @Bean
    public Jackson2JsonMessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```



### 4）编写监听器

 ![1553263050354](./day11-用户注册【RabbitMQ】.assets/1553263050354.png) 

代码：

```java
package com.leyou.search.listener;

import com.leyou.common.constants.MQConstants;
import com.leyou.search.service.SearchService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * 商品上下架处理
 */
@Component
@Slf4j
public class ItemListener {

    @Autowired
    private SearchService searchService;

    /**
     * 商品上架->创建商品的索引库
     */
    @RabbitListener(bindings =
        @QueueBinding(
                value = @Queue(name = MQConstants.Queue.SEARCH_ITEM_UP),
                exchange = @Exchange(name = MQConstants.Exchange.ITEM_EXCHANGE_NAME,type = ExchangeTypes.TOPIC),
                key = MQConstants.RoutingKey.ITEM_UP_KEY
        )
    )
    public void createIndex(Long id){
        try {
            searchService.createIndex(id);
            log.info("【索引同步】索引创建成功");
        } catch (Exception e) {
            e.printStackTrace();
            log.error("【索引同步】索引创建失败，原因："+e.getMessage());
        }
    }

    /**
     * 商品下架->删除商品的索引库
     */
    @RabbitListener(bindings =
    @QueueBinding(
            value = @Queue(name = MQConstants.Queue.SEARCH_ITEM_DOWN),
            exchange = @Exchange(name = MQConstants.Exchange.ITEM_EXCHANGE_NAME,type = ExchangeTypes.TOPIC),
            key = MQConstants.RoutingKey.ITEM_DOWN_KEY
    )
    )
    public void deleteIndex(Long id){
        try {
            searchService.deleteIndex(id);
            log.info("【索引同步】索引删除成功");
        } catch (Exception e) {
            e.printStackTrace();
            log.error("【索引同步】索引删除失败，原因："+e.getMessage());
        }
    }
}



```



### 5）编写创建和删除索引方法

这里因为要创建和删除索引，我们需要在SearchService中拓展两个方法，创建和删除索引：

```java
 public void createIndex(Long id) {
        //1.根据spuId查询SpuDTO
        SpuDTO spuDTO = itemClient.findSpuDTOBySpuId(id);
        //2.把SpuDTO转换为Goods
        Goods goods = buildGoods(spuDTO);
        //3.保存Goods到ES中
        searchRepository.save(goods);
    }

    public void deleteIndex(Long id) {
        searchRepository.deleteById(id);
    }
```



## 05、数据同步：静态页服务接收消息

商品静态页服务接收到消息后的处理：

- 上架：创建新的静态页
- 下架：删除原来的静态页

与前面搜索服务类似，也需要两个队列来处理。

### 1）引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### 2）添加配置

```yaml
spring:
  rabbitmq:
    host: 127.0.0.1
    username: leyou
    password: leyou
    virtual-host: /leyou
```

这里只是接收消息而不发送，所以不用配置template相关内容。

### 3）Json消息序列化配置

不过，不要忘了消息转换器：

![1577093458724](./day11-用户注册【RabbitMQ】.assets/1577093458724.png)  



```java
/**
 * @author 黑马程序员
 */
@Configuration
public class RabbitConfig {

    @Bean
    public Jackson2JsonMessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}

```



### 4）编写监听器

 ![1577093412587](./day11-用户注册【RabbitMQ】.assets/1577093412587.png) 

代码：

```java
package com.leyou.page.listener;

import com.leyou.common.constants.MQConstants;
import com.leyou.page.service.PageService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * 商品上下架处理
 */
@Component
@Slf4j
public class ItemListener {

    @Autowired
    private PageService pageService;

    /**
     * 商品上架->创建商品的静态页
     */
    @RabbitListener(bindings =
        @QueueBinding(
                value = @Queue(name = MQConstants.Queue.PAGE_ITEM_UP),
                exchange = @Exchange(name = MQConstants.Exchange.ITEM_EXCHANGE_NAME,type = ExchangeTypes.TOPIC),
                key = MQConstants.RoutingKey.ITEM_UP_KEY
        )
    )
    public void createStaticPage(Long id){
        try {
            pageService.createStaticPage(id);
            log.info("【索引同步】静态页创建成功");
        } catch (Exception e) {
            e.printStackTrace();
            log.error("【索引同步】静态页创建失败，原因："+e.getMessage());
        }
    }

    /**
     * 商品下架->删除商品的静态页
     */
    @RabbitListener(bindings =
    @QueueBinding(
            value = @Queue(name = MQConstants.Queue.PAGE_ITEM_DOWN),
            exchange = @Exchange(name = MQConstants.Exchange.ITEM_EXCHANGE_NAME,type = ExchangeTypes.TOPIC),
            key = MQConstants.RoutingKey.ITEM_DOWN_KEY
    )
    )
    public void deleteStaticPage(Long id){
        try {
            pageService.deleteStaticPage(id);
            log.info("【索引同步】静态页删除成功");
        } catch (Exception e) {
            e.printStackTrace();
            log.error("【索引同步】静态页删除失败，原因："+e.getMessage());
        }
    }
}


```

### 5）添加删除页面方法

创建商品详情静态页面的方法我们之前已经添加过了，只需要添加删除静态页的方法即可：

```java
/**
     * 生成每个商品的详情静态页面
     */
    public void createStaticPage(Long id){

        //1. 创建Context对象，用于读取动态数据
        Context context = new Context();
        context.setVariables(getDetailData(id));

        //2. 读取模板文件  item.html
        String templateName = itemTemplate+".html";

        //3. 使用模板引擎对象生成静态页面
        /**
         * 参数一： 模板名称。默认会自动到classpath下的templates目录读取该文件
         * 参数二：Context对象
         * 参数三：输出流
         */
        PrintWriter writer = null;
        try {
            writer = new PrintWriter(new File(itemDir,id+".html"));
            templateEngine.process(templateName,context,writer);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }finally {
            //注意：必须关闭IO流，否则后续无法删除文件
            writer.close();
        }
    }

    public void deleteStaticPage(Long id) {
        //1.读取静态页文件
        File file = new File(itemDir,id+".html");
        //2.删除
        if(file.exists()){
            file.delete();
        }
    }
```





## 06、数据同步：测试数据同步

### 查看RabbitMQ控制台

重新启动项目，并且登录RabbitMQ管理界面：http://192.168.13.111:15672

可以看到，交换机已经创建出来了：

![1577093964631](./day11-用户注册【RabbitMQ】.assets/1577093964631.png) 

队列也已经创建完毕：

![1577094058227](./day11-用户注册【RabbitMQ】.assets/1577094058227.png) 

并且队列都已经绑定到交换机：

 ![1577094169361](./day11-用户注册【RabbitMQ】.assets/1577094169361.png) 

第一次搜索：美图

<img src="../../../../../../../Documents/笔记/10.项目二/day11、用户注册【RabbitMQ】/assets/1599463380091.png" alt="1599463380091" style="zoom:67%;" />

后台下架其中一件商品：

&nbsp;<img src="../../../../../../../Documents/笔记/10.项目二/day11、用户注册【RabbitMQ】/assets/1599463420645.png" alt="1599463420645" style="zoom:67%;" />

第二次搜索，发现少了一件商品

&nbsp;<img src="../../../../../../../Documents/笔记/10.项目二/day11、用户注册【RabbitMQ】/assets/1599463456351.png" alt="1599463456351" style="zoom:67%;" />







## 07、用户注册：注册功能分析

完成了商品的详情展示，下一步自然是购物了。不过购物之前要完成用户的注册和登录等业务，我们打开注册页面分析下需求：

![image-20200325091227767](./day11-用户注册【RabbitMQ】.assets/image-20200325091227767.png)

 

综上要完成注册功能需要的技术点有：

加密技术

redis技术(**)

mq消息中间件

发短信





## 08、用户注册：Redis的回顾



### 1）NoSQL

Redis是目前非常流行的一款NoSql数据库。

> 什么是NoSql？

![1535363817109](./day11-用户注册【RabbitMQ】.assets/1535363817109.png)



常见的NoSql产品：

![1535363864689](./day11-用户注册【RabbitMQ】.assets/1535363864689.png)





### 2）Redis简介

> Redis的网址：

[官网](http://redis.io/)：速度很慢，几乎进去不去啊。

[中文网站](http://www.redis.cn/)：有部分翻译的官方文档，英文差的同学的福音



> 历史：

![1535364502694](./day11-用户注册【RabbitMQ】.assets/1535364502694.png)

> 特性：

 ![1535364521915](./day11-用户注册【RabbitMQ】.assets/1535364521915.png)





### 3）Redis安装

课前资料中已经准备了安装软件：

![1577100527718](./day11-用户注册【RabbitMQ】.assets/1577100527718.png) 

解压完成后，双击`redis-server.exe`即可，然后使用`redis-destop-manager`链接即可。





### 4）Redis与Memcache

Redis和Memcache是目前非常流行的两种NoSql数据库，读可以用于服务端缓存。两者有怎样的差异呢？

- 从实现来看：
  - redis：单线程
  - Memcache：多线程
- 从存储方式来看：
  - redis：支持数据持久化和主从备份，数据更安全
  - Memcache：数据存于内存，没有持久化功能 
- 从功能来看：
  - redis：除了基本的k-v 结构外，支持多种其它复杂结构、事务等高级功能
  - Memcache：只支持基本k-v 结构
- 从可用性看：
  - redis：支持主从备份、数据分片、哨兵监控
  - memcache：没有分片功能，需要从客户端支持

可以看出，Redis相比Memcache功能更加强大，支持的数据结构也比较丰富，已经不仅仅是一个缓存服务。而Memcache的功能相对单一。

一些面试问题：Redis缓存击穿问题、缓存雪崩问题。





### 5）Redis命令：help



通过`help`命令可以让我们查看到Redis的指令帮助信息：

在`help`后面跟上`空格`，然后按`tab`键，会看到Redis对命令分组的组名：

 ![](./day11-用户注册【RabbitMQ】.assets/test.gif)

主要包含：

- @generic：通用指令
- @string：字符串类型指令
- @list：队列结构指令
- @set：set结构指令
- @sorted_set：可排序的set结构指令
- @hash：hash结构指令

其中除了@generic以为的，对应了Redis中常用的5种数据类型：

- String：等同于java中的，`Map<String,String>`
- list：等同于java中的`Map<String,List<String>>`
- set：等同于java中的`Map<String,Set<String>>`
- sort_set：可排序的set
- hash：等同于java中的：`Map<String,Map<String,String>>`

可见，Redis中存储数据结构都是类似java的map类型。Redis不同数据类型，只是`'map'`的值的类型不同。



### 6）Redis命令：通用指令

> keys

获取符合规则的键名列表。

- 语法：keys pattern

  示例：keys *	(查询所有的键)

  ![img](./day11-用户注册【RabbitMQ】.assets/wpsD8A4.tmp.jpg)

这里的pattern其实是正则表达式，所以语法基本是类似的

> exists

判断一个键是否存在，如果存在返回整数1，否则返回0

- 语法：EXISTS key

- 示例：

  ![img](./day11-用户注册【RabbitMQ】.assets/wps979F.tmp.jpg)

> del

DEL：删除key，可以删除一个或多个key，返回值是删除的key的个数。

- 语法：DEL key [key … ]

- 示例：

  ![img](./day11-用户注册【RabbitMQ】.assets/wpsF22.tmp.jpg)

> expire

- 语法：

  ```
  EXPIRE key seconds
  ```

- 作用：设置key的过期时间，超过时间后，将会自动删除该key。

- 返回值：

  - 如果成功设置过期时间，返回1。
  - 如果key不存在或者不能设置过期时间，返回0

> TTL

TTL：查看一个key的过期时间

- 语法：`TTL key`
- 返回值：
  - 返回剩余的过期时间
  - -1：永不过期
  - -2：已过期或不存在
- 示例：

![img](./day11-用户注册【RabbitMQ】.assets/wpsFAFA.tmp.jpg) 

> persist

- 语法：

  ```
  persist key
  ```

- 作用：

  移除给定key的生存时间，将这个 key 从带生存时间 key 转换成一个不带生存时间、永不过期的 key 。

- 返回值：

  - 当生存时间移除成功时，返回 1 .
  - 如果 key 不存在或 key 没有设置生存时间，返回 0 .

- 示例：

![img](./day11-用户注册【RabbitMQ】.assets/wpsCE15.tmp.jpg) 

### 7）Redis命令：字符串指令

字符串结构，其实是Redis中最基础的K-V结构。其键和值都是字符串。类似Java的Map<String,String>

![img](./day11-用户注册【RabbitMQ】.assets/wps9794.tmp.jpg)

常用指令：

| 语法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [SET key value](http://www.runoob.com/redis/strings-set.html) | 设置指定 key 的值                                            |
| [GET key](http://www.runoob.com/redis/strings-get.html)      | 获取指定 key 的值。                                          |
| [GETRANGE key start end](http://www.runoob.com/redis/strings-getrange.html) | 返回 key 中字符串值的子字符                                  |
| [INCR key](http://www.runoob.com/redis/strings-incr.html)    | 将 key 中储存的数字值增一。                                  |
| [INCRBY key increment](http://www.runoob.com/redis/strings-incrby.html) | 将 key 所储存的值加上给定的增量值（increment） 。            |
| [DECR key](http://www.runoob.com/redis/strings-decr.html)    | 将 key 中储存的数字值减一。                                  |
| [DECRBY key decrement](http://www.runoob.com/redis/strings-decrby.html) | key 所储存的值减去给定的减量值（decrement） 。               |
| [APPEND key value](http://www.runoob.com/redis/strings-append.html) | 如果 key 已经存在并且是一个字符串， APPEND 命令将 value 追加到 key 原来的值的末尾。 |
| [STRLEN key](http://www.runoob.com/redis/strings-strlen.html) | 返回 key 所储存的字符串值的长度。                            |
| [MGET key1  key2 ...](http://www.runoob.com/redis/strings-mget.html) | 获取所有(一个或多个)给定 key 的值。                          |
| [MSET key value key value ...](http://www.runoob.com/redis/strings-mset.html) | 同时设置一个或多个 key-value 对。                            |

### 8）Redis命令：hash结构命令

Redis的Hash结构类似于Java中的Map<String,Map<String,Stgring>>，键是字符串，值是另一个映射。结构如图：

![11](./day11-用户注册【RabbitMQ】.assets/wps47DA.tmp.jpg)

这里我们称键为key，字段名为  hKey， 字段值为 hValue



 常用指令：

> **HSET、HSETNX和HGET（添加、获取）**

HSET

- 介绍：
  - ![img](./day11-用户注册【RabbitMQ】.assets/wps55A2.tmp.jpg) 
  - Redis Hset 命令用于为哈希表中的字段赋值 。
  - 如果哈希表不存在，一个新的哈希表被创建并进行 HSET 操作。
  - 如果字段已经存在于哈希表中，旧值将被覆盖。
- 返回值：
  - 如果字段是哈希表中的一个新建字段，并且值设置成功，返回 1 。
  - 如果哈希表中域字段已经存在且旧值已被新值覆盖，返回 0
- 示例：

![img](./day11-用户注册【RabbitMQ】.assets/wps55B3.tmp.jpg) 

 

> HGET

- 介绍：

![img](./day11-用户注册【RabbitMQ】.assets/wps55D5.tmp.jpg) 

```
Hget 命令用于返回哈希表中指定字段的值。 
```

- 返回值：返回给定字段的值。如果给定的字段或 key 不存在时，返回 nil
- 示例：

![img](./day11-用户注册【RabbitMQ】.assets/wps55E5.tmp.jpg) 

> HGETALL

- 介绍：

![img](./day11-用户注册【RabbitMQ】.assets/wps562A.tmp.jpg) 

- 返回值：

指定key 的所有字段的名及值。返回值里，紧跟每个字段名(field name)之后是字段的值(value)，所以返回值的长度是哈希表大小的两倍

- 示例：

  ![img](./day11-用户注册【RabbitMQ】.assets/wps562B.tmp.jpg)

> HKEYS

- 介绍

![img](./day11-用户注册【RabbitMQ】.assets/wps563C.tmp.jpg) 

- 示例：

  ![img](./day11-用户注册【RabbitMQ】.assets/wps563D.tmp.jpg) 



> HVALS

![img](./day11-用户注册【RabbitMQ】.assets/wps564D.tmp.jpg) 

- 注意：这个命令不是HVALUES，而是HVALS，是value 的缩写：val

- 示例：

  ![img](./day11-用户注册【RabbitMQ】.assets/wps565E.tmp.jpg) 

> **HDEL（删除）**

Hdel 命令用于删除哈希表 key 中的一个或多个指定字段，不存在的字段将被忽略。

![img](./day11-用户注册【RabbitMQ】.assets/wps565F.tmp.jpg) 

- 语法：

  HDEL key field1 [field2 ... ]

- 返回值：

  被成功删除字段的数量，不包括被忽略的字段

- 示例：

  ![img](./day11-用户注册【RabbitMQ】.assets/wps566F.tmp.jpg) 





### 10）Redis的持久化：RDB

Redis有两种持久化方案：RDB和AOF



> 触发条件

RDB是Redis的默认持久化方案，当满足一定的条件时，Redis会自动将内存中的数据全部持久化到硬盘。

条件在redis.conf文件中配置，格式如下：

```
save (time) (count)
```

当满足在time（单位是秒）时间内，至少进行了count次修改后，触发条件，进行RDB快照。

例如，默认的配置如下：

![1535375586633](./day11-用户注册【RabbitMQ】.assets/1535375586633.png)

> 基本原理

RDB的流程是这样的：

- Redis使用fork函数来复制一份当前进程（父进程）的副本（子进程）
- 父进程继续接收并处理请求，子进程开始把内存中的数据写入硬盘中的临时文件
- 子进程写完后，会使用临时文件代替旧的RDB文件



### 11）Redis的持久化：AOF

> 基本原理

AOF方式默认是关闭的，需要修改配置来开启：

```
appendonly yes # 把默认的no改成yes
```

AOF持久化的策略是，把每一条服务端接收到的写命令都记录下来，每隔一定时间后，写入硬盘的AOF文件中，当服务器重启后，重新执行这些命令，即可恢复数据。



AOF文件写入的频率是可以配置的：

![1535376414724](./day11-用户注册【RabbitMQ】.assets/1535376414724.png)



> AOF文件重写

当记录命令过多，必然会出现对同一个key的多次写操作，此时只需要记录最后一条即可，前面的记录都毫无意义了。因此，当满足一定条件时，Redis会对AOF文件进行重写，移除对同一个key的多次操作命令，保留最后一条。默认的触发条件：

![1535376351400](./day11-用户注册【RabbitMQ】.assets/1535376351400.png)

主从





## 09、用户注册：SpringDataRedis的使用(*)

之前，我们使用Redis都是采用的Jedis客户端，不过既然我们使用了SpringBoot，为什么不使用Spring对Redis封装的套件呢？

### 1）Spring Data Redis

官网：<http://projects.spring.io/spring-data-redis/>

![1527250056698](./day11-用户注册【RabbitMQ】.assets/1527250056698.png)                                    

Spring Data Redis，是Spring Data 家族的一部分。 对Jedis客户端进行了封装，与spring进行了整合。可以非常方便的来实现redis的配置和操作。 

### 2）RedisTemplate基本操作

与以往学习的套件类似，Spring Data 为 Redis 提供了一个工具类：RedisTemplate。里面封装了对于Redis的五种数据结构的各种操作，包括：

- redisTemplate.opsForValue() ：操作字符串  <key, String>
- redisTemplate.opsForHash() ：操作hash <key, Map<String, Object>>
- redisTemplate.opsForList()：操作list <key,  List<Object>>
- redisTemplate.opsForSet()：操作set  <key,  Set<Object>>
- redisTemplate.opsForZSet()：操作zset  <key,  Set<Object>>

例如我们对字符串操作比较熟悉的有：get、set等命令，这些方法都在 opsForValue()返回的对象中有：

![1535376791994](./day11-用户注册【RabbitMQ】.assets/1535376791994.png)



其它一些通用命令，如del，可以通过redisTemplate.xx()来直接调用。

 ![1535376896536](./day11-用户注册【RabbitMQ】.assets/1535376896536.png)



### 3）StringRedisTemplate

RedisTemplate在创建时，可以指定其泛型类型：

- K：代表key 的数据类型
- V: 代表value的数据类型

注意：这里的类型不是Redis中存储的数据类型，而是Java中的数据类型，RedisTemplate会自动将Java类型转为Redis支持的数据类型：字符串、字节、二二进制等等。

![1527250218215](./day11-用户注册【RabbitMQ】.assets/1527250218215.png)

不过RedisTemplate默认会采用JDK自带的序列化（Serialize）来对对象进行转换。生成的数据十分庞大，因此一般我们都会指定key和value为String类型，这样就由我们自己把对象序列化为json字符串来存储即可。



因为大部分情况下，我们都会使用key和value都为String的RedisTemplate，因此Spring就默认提供了这样一个实现：

 ![1527256139407](./day11-用户注册【RabbitMQ】.assets/1527256139407.png)

### 4）测试

我们新建一个测试项目，然后在项目中引入Redis启动器：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.leyou</groupId>
    <artifactId>redis</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
        <relativePath/>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
    </dependencies>
</project>
```

然后在配置文件中指定Redis地址：

```yaml
spring:
  redis:
    host: 127.0.0.1
```

编写启动类

```java
package cn.itcast;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 *
 */
@SpringBootApplication
public class RedisApplication {
    public static void main(String[] args) {
        SpringApplication.run(RedisApplication.class,args);
    }
}

```





然后就可以直接注入`StringRedisTemplate`对象了：

```java
package com.itheima;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.BoundHashOperations;
import org.springframework.data.redis.core.BoundValueOperations;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.concurrent.TimeUnit;

/**
 *
 */
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = RedisApplication.class)
public class RedisDemo {

    @Autowired
    private StringRedisTemplate redisTemplate;

    @Test
    public void testSet(){
        /**
         * redisTemplate.opsForXXX(): 一般用于操作普通类型（字符串，基本对象）
         * redisTemplate.boundXXXOps()： 一般用于操作复杂类型（Hash,Set,List,ZSet）
         */
        redisTemplate.opsForValue().set("name","jack",50, TimeUnit.SECONDS);

        /*BoundValueOperations<String, String> boundValueOps = redisTemplate.boundValueOps("name");
        boundValueOps.set("jack");
        boundValueOps.expire(20,TimeUnit.SECONDS);*/
    }

    @Test
    public void tesetGet(){
        /**
         * redisTemplate.opsForXXX(): 一般用于操作普通类型（字符串，基本对象）
         * redisTemplate.boundXXXOps()： 一般用于操作复杂类型（Hash,Set,List,ZSet）
         */
        String name = redisTemplate.opsForValue().get("name");
        System.out.println(name);

       /* BoundValueOperations<String, String> boundValueOps = redisTemplate.boundValueOps("name");
        boundValueOps.set("jack");
        boundValueOps.expire(20,TimeUnit.SECONDS);*/
    }
}




```

![1577159922463](./day11-用户注册【RabbitMQ】.assets/1577159922463.png) 







## 10、用户注册：加密技术



**加密从大的范畴来说分两类：**

- 密钥加密：提前生成密钥，安全级别高，比较复杂，一般用于文件类的加密。比如git。

- 加盐加密：直接指定加盐规则即可，灵活方便，适用于密码加密。从加盐的方式上又分两类，动态加盐加密和固定加盐加密，动态加盐加密更安全。

**加盐加密：**

- 固定加盐加密：加盐规则固定，反复加密结果是一样的，比如md5就可以实现固定加盐加密。
- 动态加盐加密：加盐规则是动态的，每次加密结构都不一样。比如spring的加密方式。多次加密的效果如下：导入工具包：

```xml
<dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-core</artifactId>
        </dependency>
```





```java
package com.itheima;

import org.junit.Test;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

/**
 * BCrpt加密
 */
public class BCryptDemo {

    /**
     * $2a$10$K3QbgSfuNgPJKqAkFuNjauyTiOIfWNlFcE9IBLsPiGBfmGcswqcBy
     * $2a$10$FQMt/VbUzDmwVLVsHFWu7OoNQ0TzKeF8VDDROrHmWpJpw3CPzF.b6
     * $2a$10$T6P96V.JJR5UFCXRmiFeT.6.1t8rxkXIl78rkHxUW4BSYZAmxl2XW
     */
    @Test
    public void testEncode(){
        String pwd = "123456";

        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        String encodePwd = passwordEncoder.encode(pwd);
        System.out.println(encodePwd);
    }

    @Test
    public void testMatch(){
        String pwd = "123456";
        String encodePwd = "$2a$10$T6P96V.JJR5UFCXRmiFeT.6.1t8rxkXIl78rkHxUW4BSYZAmxl2XW";

        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        /**
         * 参数一：原始密码
         * 参数二：加密的密码
         */
        System.out.println(passwordEncoder.matches(pwd,encodePwd));
    }

}



```





## 11、用户注册：开通阿里短信服务

### 1） 创建阿里云用户并授权

这一步，我们之前在使用阿里OSS服务时已经做过了，这里只需要给用户授予SMS短信服务权限即可。

![image-20200325102006712](./day11-用户注册【RabbitMQ】.assets/image-20200325102006712.png)

### 2） 进入阿里SMS服务主界面，通过“快速入门”来准备使用SMS服务相关的操作。

由此处进入阿里云SMS主界面

![image-20200325102223729](./day11-用户注册【RabbitMQ】.assets/image-20200325102223729.png)

![image-20200325102354296](./day11-用户注册【RabbitMQ】.assets/image-20200325102354296.png)

### 3） 申请短信签名

进入阿里云SMS服务主界面，找到左侧“国内消息”

![image-20200325102526385](./day11-用户注册【RabbitMQ】.assets/image-20200325102526385.png)



添加签名

![image-20200325102748777](./day11-用户注册【RabbitMQ】.assets/image-20200325102748777.png)

一般非礼拜天时间段，两个小时内有结果。

![image-20200325102837568](./day11-用户注册【RabbitMQ】.assets/image-20200325102837568.png)

如果未通过，查看原因并修改，再次申请。

### 4） 申请短信模板

顾名思义，短信模板，就是发短信的内容模板，模板中有一个验证码占位符，可以纯数字类型传参，其余文字描述部分，一旦确定，后期无法修改。

如图所示，打开短信模板申请界面

![image-20200325103148090](./day11-用户注册【RabbitMQ】.assets/image-20200325103148090.png)

添加模板信息

![image-20200325103435557](./day11-用户注册【RabbitMQ】.assets/image-20200325103435557.png)

一般非礼拜天时间段，两个小时内有结果。





## 12、用户注册：阿里在线短信调试

申请完签名和模板之后，我们可以借助阿里的在线调试短信功能来测试我们的短信签名和模板。

https://help.aliyun.com/document_detail/55288.html?spm=5176.8195934.1283918.5.34ed6a7dFB19N9

继续回到“快速入门”，点击“发送短信”：

![image-20200325103835577](./day11-用户注册【RabbitMQ】.assets/image-20200325103835577.png)

点击如图所示SendSms，进入测试界面：

![image-20200325103918058](./day11-用户注册【RabbitMQ】.assets/image-20200325103918058.png)

填写调试短信的信息：

![image-20200325104508868](./day11-用户注册【RabbitMQ】.assets/image-20200325104508868.png)

填入各类参数后，点击 `发起调用`按钮即可，然后该手机会受到这样一条短信：

![1577259801071](./day11-用户注册【RabbitMQ】.assets/1577259801071.png) 





## 13、用户注册：搭建短信微服务

因为系统中不止注册一个地方需要短信发送，因此我们将短信发送抽取为微服务：`ly-sms`，凡是需要的地方都可以使用。

另外，因为短信发送API调用时长的不确定性，为了提高程序的响应速度，短信发送我们都将采用异步发送方式，即：

- 短信服务监听MQ消息，收到消息后发送短信。
- 其它服务要发送短信时，通过MQ通知短信微服务。



![用户注册发送短信流程](./day11-用户注册【RabbitMQ】.assets/用户注册发送短信流程.png)



### 1）创建module

![1577263325821](./day11-用户注册【RabbitMQ】.assets/1577263325821.png) 



### 2）依赖坐标

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>leyou</artifactId>
        <groupId>com.leyou</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>ly-sms</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.aliyun</groupId>
            <artifactId>aliyun-java-sdk-core</artifactId>
            <version>4.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>com.leyou</groupId>
            <artifactId>ly-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```



### 3）编写启动类

```java
package com.leyou;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class LySmsApplication {
    public static void main(String[] args) {
        SpringApplication.run(LySmsApplication.class, args);
    }
}
```



### 4）编写application.yml

```yml
server:
  port: 8085
spring:
  application:
    name: sms-service
  rabbitmq:
    host: 127.0.0.1
    username: leyou
    password: leyou
    virtual-host: /leyou
```





## 14、用户注册：封装短信工具

### 1） 获取SMS发短信的代码

![image-20200325105752131](./day11-用户注册【RabbitMQ】.assets/image-20200325105752131.png)

具体代码如下：

```java
import com.aliyuncs.CommonRequest;
import com.aliyuncs.CommonResponse;
import com.aliyuncs.DefaultAcsClient;
import com.aliyuncs.IAcsClient;
import com.aliyuncs.exceptions.ClientException;
import com.aliyuncs.exceptions.ServerException;
import com.aliyuncs.http.MethodType;
import com.aliyuncs.profile.DefaultProfile;
/*
pom.xml
<dependency>
  <groupId>com.aliyun</groupId>
  <artifactId>aliyun-java-sdk-core</artifactId>
  <version>4.0.3</version>
</dependency>
*/
public class SendSms {
    public static void main(String[] args) {
        DefaultProfile profile = DefaultProfile.getProfile("cn-hangzhou", "<accessKeyId>", "<accessSecret>");
        IAcsClient client = new DefaultAcsClient(profile);

        CommonRequest request = new CommonRequest();
        request.setMethod(MethodType.POST);
        request.setDomain("dysmsapi.aliyuncs.com");
        request.setVersion("2017-05-25");
        request.setAction("SendSms");
        request.putQueryParameter("RegionId", "cn-hangzhou");
        request.putQueryParameter("PhoneNumbers", "xxx");
        request.putQueryParameter("SignName", "xxx");
        request.putQueryParameter("TemplateCode", "xxx");
        request.putQueryParameter("TemplateParam", "{\"checkcode\":\"666\"}");
        try {
            CommonResponse response = client.getCommonResponse(request);
            System.out.println(response.getData());
        } catch (ServerException e) {
            e.printStackTrace();
        } catch (ClientException e) {
            e.printStackTrace();
        }
    }
}
```



### 2）抽取短信相关的属性

我们首先把一些常量抽取到application.yml中：

```yaml
ly:
  sms:
    accessKeyID: LTAIfmmL26haCK0b # 你自己的accessKeyId
    accessKeySecret: pX3RQns9ZwXs75M6Isae9sMgBLXDfY # 你自己的AccessKeySecret
    signName: 乐优商城 # 签名名称
    verifyCodeTemplate: SMS_133976814 # 模板名称
    domain: dysmsapi.aliyuncs.com # 域名
    action: SendSMS # API类型，发送短信
    version: 2017-05-25 # API版本，固定值
    regionID: cn-hangzhou # 区域id
    code: checkcode # 短信模板中验证码的占位符
```

然后提供解析配置文件的配置类：

```java
package com.leyou.sms.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

/**
 * 读取SMS配置属性
 */
@Component
@ConfigurationProperties(prefix = "ly.sms")
@Data
public class SmsProperties {

    /**
     * 账号
     */
    String accessKeyID;
    /**
     * 密钥
     */
    String accessKeySecret;
    /**
     * 短信签名
     */
    String signName;
    /**
     * 短信模板
     */
    String verifyCodeTemplate;
    /**
     * 发送短信请求的域名
     */
    String domain;
    /**
     * API版本
     */
    String version;
    /**
     * API类型
     */
    String action;
    /**
     * 区域
     */
    String regionID;
    /**
     * 短信模板中验证码的占位符
     */
    String code;

}

```

### 3）参数KEY的静态变量

```java
package com.leyou.sms.config;

/**
 * @author 黑马程序员
 */
public final class SmsConstants {

    /**
     * 请求参数
     */
    public static final String SMS_PARAM_REGION_ID = "RegionId";
    public static final String SMS_PARAM_KEY_PHONE = "PhoneNumbers";
    public static final String SMS_PARAM_KEY_SIGN_NAME = "SignName";
    public static final String SMS_PARAM_KEY_TEMPLATE_CODE = "TemplateCode";
    public static final String SMS_PARAM_KEY_TEMPLATE_PARAM= "TemplateParam";

    /**
     * 响应结果
     */
    public static final String SMS_RESPONSE_KEY_CODE = "Code";
    public static final String SMS_RESPONSE_KEY_MESSAGE = "Message";

    /**
     * 状态
     */
    public static final String OK = "OK";
}
```

如图：

![image-20200325112906251](./day11-用户注册【RabbitMQ】.assets/image-20200325112906251.png) 

### 4）阿里客户端交给spring管理

首先，把发请求需要的客户端注册到Spring容器：

```java
package com.leyou.config;

import com.aliyuncs.DefaultAcsClient;
import com.aliyuncs.IAcsClient;
import com.aliyuncs.profile.DefaultProfile;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 短信配置类
 */
@Configuration
public class SmsConfig {

    @Bean
    public IAcsClient iAcsClient(SmsProperties smsProps){
        DefaultProfile profile = DefaultProfile.getProfile(smsProps.getRegionID(), smsProps.getAccessKeyID(), smsProps.getAccessKeySecret());
        IAcsClient client = new DefaultAcsClient(profile);
        return client;
    }
}

```



### 5）发送短信工具类

我们把阿里提供的demo进行简化和抽取，封装一个工具类：

```java
package com.leyou.sms.utils;

import com.aliyuncs.CommonRequest;
import com.aliyuncs.CommonResponse;
import com.aliyuncs.IAcsClient;
import com.aliyuncs.exceptions.ClientException;
import com.aliyuncs.exceptions.ServerException;
import com.aliyuncs.http.MethodType;
import com.leyou.common.utils.JsonUtils;
import com.leyou.sms.config.SmsProperties;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.Map;

/**
 * SMS工具类
 */
@Component
@Slf4j
public class SmsHelper {

    @Autowired
    private IAcsClient client;
    @Autowired
    private SmsProperties smsProps;

    /**
     * 发送验证码短信
     */
    public void sendCodeMsg(String phone,String code){
        CommonRequest request = new CommonRequest();
        request.setMethod(MethodType.POST);
        request.setDomain(smsProps.getDomain());
        request.setVersion(smsProps.getVersion());
        request.setAction(smsProps.getAction());
        request.putQueryParameter("RegionId", smsProps.getRegionID());
        request.putQueryParameter("PhoneNumbers", phone);
        request.putQueryParameter("SignName", smsProps.getSignName());
        request.putQueryParameter("TemplateCode", smsProps.getVerifyCodeTemplate());
        request.putQueryParameter("TemplateParam", "{\""+smsProps.getCode()+"\":\""+code+"\"}");
        try {
            CommonResponse response = client.getCommonResponse(request);
            //System.out.println(response.getData());

            //判断是否成功
            Map respMap = JsonUtils.toMap(response.getData(),String.class,String.class);
            if(respMap.get("Code").equals("OK") && respMap.get("Message").equals("OK")){
                log.info("【短信API】发送短信成功");
            }else{
                log.info("【短信API】发送短信失败，原因："+respMap.get("Message"));
            }

        } catch (Exception e) {
            e.printStackTrace();
            log.info("【短信API】发送短信失败，原因："+e.getMessage());
        }
    }

}


```



## 15、用户注册：MQ监听器异步发送短信

接下来，编写消息监听器，当接收到消息后，我们发送短信。

### 1）改造ly-common中的MQ常量

不要忘了，几个队列和交换机的名称，定义到`ly-common`中：

```java
package com.leyou.common.constants;

/**
 * @author 黑马程序员
 */
public abstract class MQConstants {

    public static final class Exchange {
        // ... 略
        /**
         * 消息服务交换机名称
         */
        public static final String SMS_EXCHANGE_NAME = "ly.sms.exchange";
    }

    public static final class RoutingKey {
        // ... 略
        /**
         * 消息服务的routing-key
         */
        public static final String VERIFY_CODE_KEY = "sms.verify.code";
    }


    public static final class Queue{
        // ... 略
        /**
         * 消息服务的队列
         */
        public static final String SMS_VERIFY_CODE_QUEUE = "sms.verify.code.queue";
    }
}
```



###2）编写监听器

在`ly-sms`项目编写短信消息监听器：

```java
ppackage com.leyou.sms.listener;

import com.leyou.common.constants.MQConstants;
import com.leyou.sms.utils.SmsHelper;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.Map;

/**
 * 短信发送监听器
 */
@Component
public class SmsListener {

    @Autowired
    private SmsHelper smsHelper;

    /**
     * 接收短信内容
     */
    @RabbitListener(bindings =
        @QueueBinding(
                value = @Queue(name = MQConstants.Queue.SMS_VERIFY_CODE_QUEUE),
                exchange = @Exchange(name = MQConstants.Exchange.SMS_EXCHANGE_NAME,type = ExchangeTypes.TOPIC),
                key = MQConstants.RoutingKey.VERIFY_CODE_KEY
        )
    )
    public void sendCodeMsg(Map<String,String> msgMap){
        //1.接收内容
        String phone = msgMap.get("phone");
        String code = msgMap.get("code");
        //2.调用短信发送API
        smsHelper.sendCodeMsg(phone,code);
    }
}


```

我们注意到，消息体是一个Map，里面有两个属性：

- phone：电话号码
- code：短信验证码



### 3）单元测试

```java
package com.leyou;

import com.leyou.common.constants.MQConstants;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.HashMap;
import java.util.Map;

/**
 *
 */
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = LySmsApplication.class)
public class SmsTest {

    @Autowired
    private AmqpTemplate amqpTemplate;

    @Test
    public void testSendCodeMsg(){
        Map<String,String> msgMap = new HashMap<>();
        msgMap.put("phone","13060801954");
        msgMap.put("code","537343");
        amqpTemplate.convertAndSend(
                MQConstants.Exchange.SMS_EXCHANGE_NAME,
                MQConstants.RoutingKey.VERIFY_CODE_KEY,
                msgMap);

        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}

```



查看RabbitMQ控制台，发现交换机已经创建：

 ![1527239600218](./day11-用户注册【RabbitMQ】.assets/1527239600218.png) 





## 16、用户注册：搭建用户中心

### 1） 创建ly-pojo-user对象模块

![image-20200810131729071](./day11-用户注册【RabbitMQ】.assets/image-20200810131729071.png)

### 2） 创建ly-user工程并导入jar包

![1577328413615](./day11-用户注册【RabbitMQ】.assets/1577328413615.png)  

> pom文件加入依赖坐标

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>leyou</artifactId>
        <groupId>com.leyou</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>ly-user</artifactId>

    <dependencies>
        <!--nacos客户端-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>2.1.0.RELEASE</version>
        </dependency>
        <!--web启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- mysql驱动 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!-- mybatis-plus启动器 -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.1.0</version>
        </dependency>
        <!-- 实体类包 -->
        <dependency>
            <groupId>com.leyou</groupId>
            <artifactId>ly-pojo-user</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--redis-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!--mq消息中间件-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <!--测试-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>com.leyou</groupId>
            <artifactId>ly-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### 3） 提供启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
@MapperScan("com.leyou.user.mapper")
public class LyUserApplication {
    public static void main(String[] args) {
        SpringApplication.run(LyUserApplication.class,args);
    }
}
```

### 4） 提供配置文件

```yml
server:
  port: 8086
spring:
  application:
    name: user-service
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/leyou?serverTimezone=UTC&characterEncoding=utf8&useUnicode=true&useSSL=true
    username: root
    password: root
  redis:
    host: 127.0.0.1
  rabbitmq:
    host: 127.0.0.1
    virtual-host: /test
    username: test
    password: test
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
mybatis-plus:
  type-aliases-package: com.leyou.user.pojo
  configuration:
    map-underscore-to-camel-case: true
logging:
  level:
    com.leyou: debug
```



### 5） 添加路由网关

我们修改`ly-gateway`配置，添加路由规则，对`ly-user`进行路由:

![1577329581013](./day11-用户注册【RabbitMQ】.assets/1577329581013.png) 

```yml
- id: user-service
  uri: lb://user-service
  predicates:
    - Path=/api/user/**
  filters:
    - StripPrefix=2
```



## 17、用户注册：后台代码准备

### 1）在ly-pojo-user模块中提供实体类

```java
package com.leyou.user.entity;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;

import java.util.Date;

@TableName("tb_user")
@Data
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String username;
    private String password;
    private String phone;
    private Date createTime;
    private Date updateTime;
}
```



### 2）Mapper

```java
package com.leyou.user.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.leyou.user.entity.User;

public interface UserMapper extends BaseMapper<User> {
}
```



### 3）Service

```java
package com.leyou.user.service;

import com.leyou.user.mapper.UserMapper;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@Transactional
public class UserService {

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private AmqpTemplate amqpTemplate;

    @Autowired
    private StringRedisTemplate redisTemplate;

}
```



### 4）Controller

```java
package com.leyou.user.controller;

import com.leyou.user.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {
    
    @Autowired
    private UserService userService;
    
}
```





## 18、用户注册：校验数据是否唯一

### 1）接口说明

#### 接口路径

```
GET /check/{data}/{type}
```

#### 参数说明

| 参数 | 说明                                 | 是否必须 | 数据类型 | 默认值 |
| ---- | ------------------------------------ | -------- | -------- | ------ |
| data | 要校验的数据                         | 是       | String   | 无     |
| type | 要校验的数据类型：1，用户名；2，手机 | 是       | Integer  | 无     |

#### 返回结果

返回布尔类型结果：

- true：可用
- false：不可用

状态码：

- 200：校验成功
- 400：参数有误
- 500：服务器内部异常





### 2）Controller

因为有了接口，我们可以不关心页面，所有需要的东西都一清二楚：

- 请求方式：GET
- 请求路径：/check/{param}/{type}
- 请求参数：param,type
- 返回结果：true或false

```java
package com.leyou.user.controller;

import com.leyou.user.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

/**
 *
 */
@RestController
public class UserController {
    @Autowired
    private UserService userService;

    /**
     * 数据校验
     */
    @GetMapping("/check/{data}/{type}")
    public ResponseEntity<Boolean> checkData(
            @PathVariable("data") String data,
            @PathVariable("type") Integer type
    ){
        Boolean isCanUse = userService.checkData(data,type);
        return ResponseEntity.ok(isCanUse);
    }
}




```

### 3）Service

```java
package com.leyou.user.service;

import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.core.toolkit.Wrappers;
import com.leyou.common.exception.pojo.ExceptionEnum;
import com.leyou.common.exception.pojo.LyException;
import com.leyou.user.mapper.UserMapper;
import com.leyou.user.pojo.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 *
 */
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;

    public Boolean checkData(String data, Integer type) {
        User user = new User();
        switch (type){
            case 1:
                user.setUsername(data);
                break;
            case 2:
                user.setPhone(data);
                break;
            default:
                throw new LyException(ExceptionEnum.INVALID_PARAM_ERROR);
        }
        QueryWrapper queryWrapper = Wrappers.query(user);
        return userMapper.selectCount(queryWrapper)==0;
    }
}

```



### 4）测试

我们查看数据库中的数据：

![1577330768054](./day11-用户注册【RabbitMQ】.assets/1577330768054.png) 

然后启动微服务，在浏览器调用接口，测试用户名：

![1577330914146](./day11-用户注册【RabbitMQ】.assets/1577330914146.png) 

![1577330871961](./day11-用户注册【RabbitMQ】.assets/1577330871961.png) 



测试手机号码：

![1577330973722](./day11-用户注册【RabbitMQ】.assets/1577330973722.png) 

![1577331011749](./day11-用户注册【RabbitMQ】.assets/1577331011749.png) 





## 19、用户注册：发送短信验证码

短信微服务已经准备好，我们就可以继续编写用户中心接口了。

### 1）思路分析

这里的业务逻辑是这样的：

- 1）我们接收页面发送来的手机号码
- 2）生成一个随机验证码
- 3）将验证码保存在服务端
- 4）发送短信，将验证码发送到用户手机



那么问题来了：验证码保存在哪里呢？

验证码有一定有效期，一般是5分钟，我们可以利用Redis的过期机制来保存。





### 2）接口文档

#### 功能说明

用户输入手机号，点击发送验证码，前端会把手机号发送到服务端。服务端生成随机验证码，长度为6位，纯数字。并且调用短信服务，发送验证码到用户手机。

步骤：

- 接收用户请求，手机号码
- 验证手机号格式
- 生成验证码
- 保存验证码到redis
- 发送RabbitMQ消息到ly-sms

#### 接口路径

```
POST /code
```

#### 参数说明

| 参数  | 说明           | 是否必须 | 数据类型 | 默认值 |
| ----- | -------------- | -------- | -------- | ------ |
| phone | 用户的手机号码 | 是       | String   | 无     |

#### 返回结果

无

状态码：

- 204：请求已接收
- 400：参数有误
- 500：服务器内部异常





### 3）Controller

```java
  /**
     * 发送验证码
     */
    @PostMapping("/code")
    public ResponseEntity<Void> sendCodeMsg(@RequestParam("phone") String phone){
        userService.sendCodeMsg(phone);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
```



### 4）在ly-common常量类中提供注册时短信验证码在redis中的key的前缀

![image-20200810144558843](./day11-用户注册【RabbitMQ】.assets/image-20200810144558843.png)

```java
/*注册时短信验证码在redis中的key的前缀*/
public static final String REDIS_KEY_PRE = "REDIS_KEY_PRE";
```

### 5）Service

```java
    public void sendCodeMsg(String phone) {
        //1.生成随机验证码
        //RandomStringUtils: 生成随机数子字符串
        String code = RandomStringUtils.randomNumeric(6);

        //2.把验证码存入redis
        redisTemplate.opsForValue().set("REDIS_KEY_PRE_"+phone,code,4, TimeUnit.DAYS);

        //3.把手机号和验证码发送给MQ
        Map<String,String> msgMap = new HashMap<>();
        msgMap.put("phone",phone);
        msgMap.put("code",code);
        amqpTemplate.convertAndSend(
                MQConstants.Exchange.SMS_EXCHANGE_NAME,
                MQConstants.RoutingKey.VERIFY_CODE_KEY,
                msgMap);
    }
```

> 报错问题：
>
> ![image-20210112152354387](乐优电商-11-用户注册【RabbitMQ】.assets/image-20210112152354387.png)
>
> 原因：
>
> 因为ly-sms 中已经是由了转换器，但是ly-user没有添加转换器，添加即可
>
> ![image-20210112152443776](day11-课堂笔记.assets/image-20210112152443776.png)



## 20、用户注册：编写用户注册业务

### 1）接口说明

#### 功能说明

用户页面填写数据，发送表单到服务端，服务端实现用户注册功能。需要对用户密码进行加密存储，使用MD5加密，加密过程中使用随机码作为salt加盐。另外还需要对用户输入的短信验证码进行校验。

- 验证短信验证码
- 生成盐
- 对密码加密
- 写入数据库

#### 接口路径

```
POST /register
```

#### 参数说明

form表单格式

| 参数     | 说明                                     | 是否必须 | 数据类型 | 默认值 |
| -------- | ---------------------------------------- | -------- | -------- | ------ |
| username | 用户名，格式为4~30位字母、数字、下划线   | 是       | String   | 无     |
| password | 用户密码，格式为4~30位字母、数字、下划线 | 是       | String   | 无     |
| phone    | 手机号码                                 | 是       | String   | 无     |
| code     | 短信验证码                               | 是       | String   | 无     |

#### 返回结果

无返回值。

状态码：

- 201：注册成功
- 400：参数有误，注册失败
- 500：服务器内部异常，注册失败



### 2）Controller

```java
  /**
     * 用户注册
     */
    @PostMapping("/register")
    public ResponseEntity<Void> register(
            User user,
            @RequestParam("code") String code
    ){
        userService.register(user,code);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
```



### 3）在用户微服务中提供加密对象



```java
package com.leyou.user.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

import java.security.SecureRandom;

/**
 * 加密配置类
 */
@Configuration
public class PasswordConfig {

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder(){
        return new BCryptPasswordEncoder();
    }
}

```



### 4）Service

基本逻辑：

- 1）校验短信验证码
- 2）对密码加密
- 3）写入数据库

代码：

```java
public void register(User user, String code) {
        //1.判断验证码是否一致
        String redisCode = redisTemplate.opsForValue().get("REDIS_KEY_PRE_"+user.getPhone());
        if(redisCode==null || !redisCode.equals(code)){
            throw new LyException(ExceptionEnum.INVALID_VERIFY_CODE);
        }
        try {
            //2.对密码加密
            user.setPassword(passwordEncoder.encode(user.getPassword()));
            //3.保存用户数据
            userMapper.insert(user);
        } catch (Exception e) {
            e.printStackTrace();
            throw new LyException(ExceptionEnum.INSERT_OPERATION_FAIL);
        }
    }
```





## 21、课程总结





1）数据同步（采用MQ异步模式）

​    1.1 业务细节（发送内容：商品ID，如何设计MQ的元素：使用哪种消息模式，有什么交换机，队列，路由）

​     1.2 生产者：商品微服务上下架的时候，发送ID给MQ

​     1.3 搜索微服务消费方：编写MQ监听器，接收ID，创建索引库或删除索引库

​     1.4 静态页服务消费方：编写MQ监听器，接收ID，创建静态页或删除静态页



2）用户注册

​     2.1 数据唯一性校验

​     2.2 发送短信验证码（采用MQ异步模式）

​     2.3 用户注册