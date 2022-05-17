---
title: 乐优电商(07)-构建商品索引库【Elasticsearch】
tags:
  - 笔记
  - 项目实战二
  - springcloud
  - 微服务
  - Elasticsearch
  - SpringDataElasticsearch
categories:
  - 项目实战二
date: 2021-01-02 13:00:42
---

## 01、课程目标

- 独立实现索引库数据写入
- 独立实现基本搜索





## 02、商品搜索：商品搜索思路分析

![商品搜索过程](./乐优电商-07-构建商品索引库【Elasticsearch】.assets/商品搜索过程.jpg)

进行商品搜索的过程应该分为两大步骤：

1）把MySQL的商品数据，导入到Elasticsearch中，构建商品索引库。

2）用户搜索商品，从Elasticsearch检索数据，展示到搜索结果页面。



今天我们主要完成第一个步骤，在Elasticsearch中构成商品索引库！



## 03、商品搜索：设计商品的索引文档对象(*)

构建商品索引库的关键是设计能够满足搜索页展示所需的商品文档对象。接下来逐步分析商品文档对象的数据结构。



### 1）在ly-pojo模块下创建ly-pojo-search

![image-20200721174351397](乐优电商-07-构建商品索引库【Elasticsearch】.assets/image-20200721174351397.png)

### 2）导入ES的jar包

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-elasticsearch</artifactId>
    </dependency>
</dependencies>
```



### 3）商品文档对象(1)：商品列表

![image-20200318114750469](乐优电商-07-构建商品索引库【Elasticsearch】.assets/image-20200318114750469.png)

由上图分析得知，我们要设计文档对象的话，应该是一个Spu对应一个文档。

创建`Goods`对象，如下：

```java
/**
 * 一个SPU对应一个Goods
 */
@Data
@Document(indexName = "goods", type = "docs", shards = 1, replicas = 1)
public class Goods {
    @Id
    @Field(type = FieldType.Keyword)
    private Long id; // spuId  索引，不分词
    @Field(type = FieldType.Keyword, index = false)
    private String subTitle;// 副标题  不索引，不分词
    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String spuName; //spu的名称（因为用于高亮显示，所以 索引，分词）
    @Field(type = FieldType.Keyword, index = false)
    private String skus;// 所有Sku对象信息， 存储json格式，  不索引，不分词
}
```

注意： 其中skus字段是一个集合，默认情况下，ElasticSearch会将集合中每个属性进行索引，而这里sku的信息只是做展示使用，无需索引，所以我们可以将整个skus字段转成json字符串来存储，整个字符串就是一个keyword，不分词不索引，从而提高ES的性能啦。



### 4）商品文档对象(2)：搜索字段

![image-20200318120015701](乐优电商-07-构建商品索引库【Elasticsearch】.assets/image-20200318120015701.png)

搜索条件的内容有一个特征，用户输入的时候，随意性很强，什么都可能输入。

所以我们必须对其分词，而且要索引。

这里，我们专门设计一个all字段用来给索引条件匹配。

all字段的内容，可以设计为：三个分类名称+品牌名称+spuName+subTitle+所有Sku的title



`Goods`对象如下：

```json
/**
 * 一个SPU对应一个Goods
 */
@Data
@Document(indexName = "goods", type = "docs", shards = 1, replicas = 1)
public class Goods {
    @Id
    @Field(type = FieldType.Keyword)
    private Long id; // spuId  索引，不分词
    @Field(type = FieldType.Keyword, index = false)
    private String subTitle;// 副标题  不索引，不分词
    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String spuName; //spu的名称（因为用于高亮显示，所以 索引，分词）
    @Field(type = FieldType.Keyword, index = false)
    private String skus;// 所有Sku对象信息， 存储json格式，  不索引，不分词

    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String all; // 所有需要被搜索的信息，包含三个分类名称+品牌名称+spuName+subTitle+所有Sku的title
}
```



### 5）商品文档对象(3)：固定过滤条件

![image-20200318140406048](乐优电商-07-构建商品索引库【Elasticsearch】.assets/image-20200318140406048.png)

固定过滤条件，指的是，搜索任何商品，都必须具备的过滤条件，只有分类和品牌两个，其中分类指第三级分类。

分类和品牌的数据，是通过Spu列表数据中每个Spu的分类和品牌聚合而来的。

经过分析后，`Goods`对象如下：

```json
/**
 * 一个SPU对应一个Goods
 */
@Data
@Document(indexName = "goods", type = "docs", shards = 1, replicas = 1)
public class Goods {
    @Id
    @Field(type = FieldType.Keyword)
    private Long id; // spuId  索引，不分词
    @Field(type = FieldType.Keyword, index = false)
    private String subTitle;// 副标题  不索引，不分词
    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String spuName; //spu的名称（因为用于高亮显示，所以 索引，分词）
    @Field(type = FieldType.Keyword, index = false)
    private String skus;// 所有Sku对象信息， 存储json格式，  不索引，不分词

    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String all; // 所有需要被搜索的信息，包含三个分类名称+品牌名称+spuName+subTitle+所有Sku的titleb  索引，分词

    @Field(type = FieldType.Long)
    private Long brandId;// 品牌id  索引，不分词
    @Field(type = FieldType.Long)
    private Long categoryId;// 商品3级分类id  索引，不分词
}
```



### 6）商品文档对象(4)：动态过滤条件

只要某个搜索条件，确定了分类范围。那么我们就可以通过每一个分类，获取到当前分类下，所有用来搜索的规格参数，最终汇总到一起，形成了如下的动态过滤条件。

![image-20200318141157862](乐优电商-07-构建商品索引库【Elasticsearch】.assets/image-20200318141157862.png)

动态过滤条件，有一个特征，内容不确定，数量不确定，所以这里我们选择使用Map来存储。



`Goods`对象如下：

```json
/**
 * 一个SPU对应一个Goods
 */
@Data
@Document(indexName = "goods", type = "docs", shards = 1, replicas = 1)
public class Goods {
    @Id
    @Field(type = FieldType.Keyword)
    private Long id; // spuId  索引，不分词
    @Field(type = FieldType.Keyword, index = false)
    private String subTitle;// 副标题  不索引，不分词
    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String spuName; //spu的名称（因为用于高亮显示，所以 索引，分词）
    @Field(type = FieldType.Keyword, index = false)
    private String skus;// 所有Sku对象信息， 存储json格式，  不索引，不分词

    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String all; // 所有需要被搜索的信息，包含三个分类名称+品牌名称+spuName+subTitle+所有Sku的titleb  索引，分词

    @Field(type = FieldType.Long)
    private Long brandId;// 品牌id  索引，不分词
    @Field(type = FieldType.Long)
    private Long categoryId;// 商品3级分类id  索引，不分词

    @Field(type = FieldType.Object)
    private Map<String, Object> specs;// 动态过滤条件，商品规格参数，key是参数名，值是参数值   索引
}
```

注意：在ElasticSearch中Map也会对每一个属性进行索引，且分词的。



### 7）商品文档对象(5)：排序字段

这里我们有两个排序字段，时间和价格。

```json
package com.leyou.search.pojo;

import lombok.Data;
import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

import java.util.List;
import java.util.Map;
import java.util.Set;

/**
 * 商品实体，与ES的文档映射
 * 一个Spu对象对应一个Goods对象
 */
@Data
@Document(indexName = "goods",type = "docs",shards = 1,replicas = 1)
public class Goods {
   @Id
   private Long id;//存储spuId
   @Field(type = FieldType.Text,analyzer = "ik_max_word")
   private String spuName;//spu表的name属性，因为该字段参与高亮显示，所以必须索引，必须分词
   @Field(type = FieldType.Keyword,index = false)
   private String subTitle;//副标题，不索引
   @Field(type = FieldType.Keyword,index = false)
   private String skus;//Spu下的所有Sku数据，由List集合转换成json字符串

   @Field(type = FieldType.Text,analyzer = "ik_max_word")
   private String all;//用于匹配搜索关键词的字段：三个分类名称+品牌名称+spuName+subTitle+所有Sku的title

   private Long categoryId;//分类ID，固定过滤条件，通过categoryId聚合而来
   private Long brandId;//品牌ID，固定过滤条件，通过brandId聚合而来

    /**
     * 动态过滤条件格式：
     *   1）通用参数格式： {"品牌":"华为"}
     *   2）特有参数格式： {"机身颜色":["白色","黑色",...]}
     */
   @Field(type = FieldType.Object)
   private Map<String,Object> specs;//动态过滤条件

   @Field(type = FieldType.Long)
   private Long createTime;//创建时间
   private Set<Long> price;//价格

}

```

注意：在ElasticSearch中，如果拿Set类型的字段域排序，降序排列时，会使用Set集合中最大的值排序，升序排序时，会使用Set集合中最小的值排序，总之，会尽量让当前Spu排在靠上的位置。



```java
package com.leyou.search.pojo;

import lombok.Data;
import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

import java.security.Key;
import java.util.Map;
import java.util.Set;

/**
 * 商品索引库
 * 一个Spu对象，对应一个Goods对象
 */
@Data
@Document(indexName = "goods",type = "docs")
public class Goods {

    @Id
    private Long id;//spuId

    @Field(type = FieldType.Text,analyzer = "ik_max_word")
    private String all;//该字段存放所有被搜索的内容 例如： spu的name，spu的subTitle，SKu的title

    @Field(type = FieldType.Text,analyzer = "ik_max_word")
    private String spuName;//商品名称。注意：其实spuName作为搜索条件已经被all包含了，但是为了高亮显示，该字段还是要索引分词！

    @Field(type = FieldType.Keyword,index = false)
    private String subTitle;//促销信息。 注意：该字段已经包含在all中，可以不索引，不分词

    @Field(type = FieldType.Keyword,index = false)
    private String skus;//所有Sku数据。 注意： List或Map集合类型，在ES默认索引分词的。 List<Sku>转换为json字符串。之所以转换字符串类型，是为了该字段的内容不索引不分词。

    //一个分类数据
    @Field(type = FieldType.Long)
    private Long categoryId;  //索引，不分词

    //一个品牌
    @Field(type = FieldType.Long)
    private Long brandId;//索引，不分词

    @Field(type = FieldType.Object)
    private Map<String,Object> specs;//用于存放该商品的所有过滤的规格参数

    @Field(type = FieldType.Long)
    private Long createTime;//创建时间

    private Set<Long> price;//价格
}

```





## 04、商品搜索：搭建搜索微服务

之前我们学习了Elasticsearch的基本应用。今天就学以致用，搭建搜索微服务，利用elasticsearch实现搜索功能。

### 1）创建module

![1575873023690](乐优电商-07-构建商品索引库【Elasticsearch】.assets/1575873023690.png) 

### 2）导入jar包

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

    <artifactId>ly-search</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>2.1.0.RELEASE</version>
        </dependency>
        <!--springmvc环境-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--调用其他微服务-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>com.leyou</groupId>
            <artifactId>ly-client-item</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--使用springData系列中的springDataElasticsearch操作elasticsearch-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
        <dependency>
            <groupId>com.leyou</groupId>
            <artifactId>ly-pojo-search</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>com.leyou</groupId>
            <artifactId>ly-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
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

### 3）配置application.yml

```yaml
server:
  port: 8083
spring:
  application:
    name: search-service
  data:
    elasticsearch:
      cluster-name: elasticsearch
      cluster-nodes: 127.0.0.1:9300
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
```

### 4）启动类

```java
package com.leyou;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

/**
 * 搜索微服务
 */
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients // 开启远程Feign调用功能
public class LySearchApplication {

    public static void main(String[] args) {
        SpringApplication.run(LySearchApplication.class,args);
    }
}

```

### 5）添加网关路由

在网关ly-gateway的==spring.cloud.gateway.routes== 配置下添加路由：

```yaml
- id: search-service
  uri: lb://search-service
  predicates:
    - Path=/api/search/**
  filters:
    - StripPrefix=2
```



## 05、商品搜索：搜索微服务代码准备工作

### 1）提供repository

```java
package com.leyou.search.repository;

import com.leyou.search.pojo.Goods;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;

/**
 * 搜索Repository
 */
public interface SearchRepository extends ElasticsearchRepository<Goods,Long>{
}

```

### 2）提供service

```java
package com.leyou.search.service;

import com.leyou.search.repository.SearchRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.elasticsearch.core.ElasticsearchTemplate;
import org.springframework.stereotype.Service;

/**
 * 搜索业务
 */
@Service
public class SearchService {
    @Autowired
    private SearchRepository searchRepository; //完成基本CRUD操作
    @Autowired
    private ElasticsearchTemplate esTemplate;//用于高级搜索
}

```

### 3）提供处理器

```java
package com.leyou.search.controller;

import com.leyou.search.service.SearchService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RestController;

/**
 * 搜索Controller
 *
 */
@RestController
public class SearchController {
    @Autowired
    private SearchService searchService;
}

```





## 06、构建索引库：提供商品微服务对外的feign接口

索引库中的数据来自于数据库，我们不能直接去查询商品的数据库，因为真实开发中，每个微服务都是相互独立的，包括数据库也是一样。所以我们只能调用商品微服务提供的接口服务。

再思考我们需要哪些服务：

- 第一：分页查询spu的服务，<font color=green>写过</font>。
- 第二：根据spuId查询sku集合的服务，<font color=red>没写过</font>
- 第三：查询分类下可以用来搜索的规格参数：<font color=green>写过</font>
- 第四：根据spuId查询SpuDetail对象的服务，<font color=red>没写过</font>



### 1）在ly-client父工程的pom文件中导入ly-common依赖

```xml
<dependency>
            <groupId>com.leyou</groupId>
            <artifactId>ly-pojo-item</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>com.leyou</groupId>
            <artifactId>ly-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
```

### 2）根据SpuId查询Sku对象集合

提供controller：

```java
      /**
     * 根据spuId查询Sku列表
     */
    @GetMapping("/sku/of/spu")
    public ResponseEntity<List<Sku>> findSkusBySpuId(@RequestParam("id") Long id){
        List<Sku> skus = goodsService.findSkusBySpuId(id);
        return ResponseEntity.ok(skus);
    }
```

提供service：

```java
 public List<Sku> findSkusBySpuId(Long id) {
        //1.封装条件
        Sku sku = new Sku();
        sku.setSpuId(id);
        QueryWrapper<Sku> queryWrapper = Wrappers.query(sku);
        
        //2.执行查询
        List<Sku> skus = skuMapper.selectList(queryWrapper);
        
        //3.处理结果返回
        if(CollectionUtils.isEmpty(skus)){
            throw new LyException(ExceptionEnum.GOODS_NOT_FOUND);
        }
        return skus;

    }
```

### 3）根据SpuId查询SpuDetail对象

提供controller：

```java
  /**
     * 根据spuId查询SpuDetail
     */
    @GetMapping("/spu/detail")
    public ResponseEntity<SpuDetail> findSpuDetailBySpuId(@RequestParam("id") Long id){
        SpuDetail spuDetail = goodsService.findSpuDetailBySpuId(id);
        return ResponseEntity.ok(spuDetail);
    }
```

提供service：

```java
    public SpuDetail findSpuDetailBySpuId(Long id) {
        SpuDetail spuDetail = spuDetailMapper.selectById(id);
        if(spuDetail==null){
            throw new LyException(ExceptionEnum.GOODS_NOT_FOUND);
        }
        return spuDetail;
    }
```



### 4）最终的item对外feign接口

```java
package com.leyou.item.client;

import com.leyou.common.pojo.PageResult;
import com.leyou.item.dto.SpuDTO;
import com.leyou.item.pojo.Sku;
import com.leyou.item.pojo.SpecParam;
import com.leyou.item.pojo.SpuDetail;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

import java.util.List;

/**
 * 商品远程接口
 */
@FeignClient("item-service")
public interface ItemClient {

    /**
     * 商品分页查询
     */
    @GetMapping("/spu/page")
    public PageResult<SpuDTO> goodsPageQuery(
            @RequestParam(value = "page",defaultValue = "1") Integer page,
            @RequestParam(value = "rows",defaultValue = "5") Integer rows,
            @RequestParam(value = "key",required = false) String key,
            @RequestParam(value = "saleable",required = false) Boolean saleable
    );

    /**
     * 根据spuId查询Sku列表
     */
    @GetMapping("/sku/of/spu")
    public List<Sku> findSkusBySpuId(@RequestParam("id") Long id);

    /**
     * 根据条件查询规格参数（gid,cid,searching）
     */
    @GetMapping("/spec/params")
    public List<SpecParam> findSpecParams(
            @RequestParam(value = "gid",required = false) Long gid,
            @RequestParam(value = "cid",required = false) Long cid,
            @RequestParam(value = "searching",required = false) Boolean searching
    );

     /**
     * 根据spuId查询SpuDetail
     */
    @GetMapping("/spu/detail")
    public SpuDetail findSpuDetailBySpuId(@RequestParam("id")Long id);

}



```





## 07、构建索引库：FeignClient接口测试（*）

### 01）服务的提供方

服务的提供方只需要提供对外的feign接口，上面已经完成了此步骤。

### 02）服务的消费方

导入feign的jar包【已完成】

在启动类上开启feign调用的支持【已完成】

将服务的提供方提供的feign接口引入到项目中【已完成】

测试使用feign接口获取服务提供者提供的数据

```java
package com.leyou;

import com.leyou.client.item.ItemClient;
import com.leyou.item.pojo.Sku;
import com.leyou.item.pojo.SpuDetail;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.List;

/**
 * 测试远程接口
 */
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = LySearchApplication.class)
public class ItemClientTest {

    @Autowired
    private ItemClient itemClient;

    @Test
    public void test1(){
        List<Sku> skus = itemClient.findSkusBySpuId(3L);
        skus.forEach(System.out::println);
    }


    @Test
    public void test2(){
        SpuDetail spuDetail = itemClient.findSpuDetailBySpuId(3L);
        System.out.println(spuDetail);
    }

}


```



## 08、构建索引库：构建Goods对象-基本实现

现在所有的资源都准备好了。

那么，我们开始编写从MySQL数据库把商品数据批量导入到Elasticsearch的方法了



SearchService类：

```java
package com.leyou.search.service;

import com.baomidou.mybatisplus.core.toolkit.CollectionUtils;
import com.leyou.common.pojo.PageResult;
import com.leyou.item.client.ItemClient;
import com.leyou.item.dto.SpuDTO;
import com.leyou.search.pojo.Goods;
import com.leyou.search.repository.SearchRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.Date;
import java.util.List;
import java.util.stream.Collectors;

/**
 *
 */
@Service
public class SearchService {
    @Autowired
    private SearchRepository searchRepository;
    @Autowired
    private ItemClient itemClient;

    /**
     * 导入数据方法（MySQL->ES）
     */
    public void importData(){
        int page = 1;//页码
        int rows = 100;//每次导入量
        long totalPage = 0;//总页数

        //1.分页查询SpuDTO对象
        do {
            PageResult<SpuDTO> pageResult = itemClient.goodsPageQuery(page, rows, null, true);// 注意：只导入上架部分的商品

            //取出所有SpuDTO对象
            List<SpuDTO> spuDTOList = pageResult.getItems();

            //判断不为NULL才处理
            if(CollectionUtils.isNotEmpty(spuDTOList)){

                List<Goods> goodsList = spuDTOList.stream().map(spuDTO -> buildGoods(spuDTO)).collect(Collectors.toList());

                //批量保存到ES
                searchRepository.saveAll(goodsList);
            }

            page++;
            totalPage = pageResult.getTotalPage();//获取总页数
        }while (page<=totalPage);//进入循环条件是：当前页码<=总页数
    }

    /**
     * 把一个SpuDTO对象转换一个Goods对象
     */
    public Goods buildGoods(SpuDTO spuDTO){
        Goods goods = new Goods();

        goods.setId(spuDTO.getId());
        goods.setSpuName(spuDTO.getName());
        goods.setSubTitle(spuDTO.getSubTitle());
        goods.setCategoryId(spuDTO.getCid3());
        goods.setBrandId(spuDTO.getBrandId());
        goods.setCreateTime(new Date().getTime());
        
        goods.setAll(null);
        goods.setPrice(null);
        goods.setSpecs(null);
        goods.setSkus(null);
        return goods;
    }
}


```





## 09、构建索引库：构建Goods对象-封装skus，price，all字段

在上一步中，我们标出了Goods的四个属性还没有值，接下来我们一个一个来完成，这里我们先通过Sku对象集合来封装skus，price，all字段：

```java
 /**
     * 把一个SpuDTO对象转换为一个Goods对象
     * @param spuDTO
     * @return
     */
    public Goods buildGoods(SpuDTO spuDTO){
        Goods goods = new Goods();

        //一、处理all，skus，price属性
        List<Sku> skuList = itemClient.findSkusBySpuId(spuDTO.getId());

        //设计一个List集合，存入所有需要显示的Sku数据
        List<Map<String,Object>> skuMapList = new ArrayList<>();
        if(CollectionUtils.isNotEmpty(skuList)){
            skuList.forEach(sku -> {
                Map<String,Object> skuMap = new HashMap<>();
                skuMap.put("id",sku.getId());
                skuMap.put("price",sku.getPrice());
                skuMap.put("images",sku.getImages());
                skuMapList.add(skuMap);
            });  
        }
        
        String all = spuDTO.getName()+spuDTO.getSubTitle()+skuList.stream().map(Sku::getTitle).collect(Collectors.joining());

        Set<Long> price = skuList.stream().map(Sku::getPrice).collect(Collectors.toSet());

        goods.setId(spuDTO.getId());
        goods.setSpuName(spuDTO.getName());
        goods.setSubTitle(spuDTO.getSubTitle());
        goods.setCreateTime(spuDTO.getCreateTime().getTime());
        goods.setCategoryId(spuDTO.getCid3());
        goods.setBrandId(spuDTO.getBrandId());
        goods.setAll(all);
        goods.setPrice(price);
        goods.setSkus(JsonUtils.toString(skuMapList));
        goods.setSpecs(null);
        return goods;
    }
```





## 10、构建索引库：构建Goods对象-封装specs属性(*)

```java
  /**
     * 把一个SpuDTO对象转换一个Goods对象
     */
    public Goods buildGoods(SpuDTO spuDTO){
        Goods goods = new Goods();

        goods.setId(spuDTO.getId());
        goods.setSpuName(spuDTO.getName());
        goods.setSubTitle(spuDTO.getSubTitle());
        goods.setCategoryId(spuDTO.getCid3());
        goods.setBrandId(spuDTO.getBrandId());
        goods.setCreateTime(new Date().getTime());

        //一、处理all,skus,price三个字段
        //1)根据spuId查询所有Sku对象
        List<Sku> skus = itemClient.findSkusBySpuId(spuDTO.getId());
        //2）只需要sku的id，price，images
        List<Map<String,Object>> skuMapList = new ArrayList();
        //3）打包数据
        skus.forEach(sku -> {
            Map<String,Object> skuMap = new HashMap<>();
            skuMap.put("id",sku.getId());
            skuMap.put("price",sku.getPrice());
            skuMap.put("images",sku.getImages());

            skuMapList.add(skuMap);
        });
        //4）把skuMapList转换json字符串
        String skuJson = JsonUtils.toString(skuMapList);

        //5）处理price
        Set<Long> price = skus.stream().map(Sku::getPrice).collect(Collectors.toSet());

        //6）处理all
        String all = spuDTO.getName()+spuDTO.getSubTitle()+
                skus.stream().map(Sku::getTitle).collect(Collectors.joining(""));

        //二、处理specs属性
        /**
         * 封装思路：Map<String,Object>
         *   1） 根据分类id和searching=true查询用于搜索过滤的规格参数
         *   2） 获取key，和 获取value
         *        key就是规格参数的name
         *   3） value从哪里来？ 根据spuId查询SpuDetail对象 （通用规格参数：generic_spec, 特有规格参数：special_spec）
         *         3.1 判断规格参数的generic属性
         *            如果是true，则取出generic_spec
         *            如果是false，则取出special_spec
         *         3.2 把上一步取出的值放入value
         *   4）把key和value存入一个Map集合
         */
        //1）根据分类id和searching=true查询用于搜索过滤的规格参数
        List<SpecParam> specParams = itemClient.findSpecParams(null, spuDTO.getCid3(), true);
        //2）封装Map的key和value
        Map<String,Object> specsMap = new HashMap<>();
        //根据spuId查询SpuDetail
        SpuDetail spuDetail = itemClient.findSpuDetailBySpuId(spuDTO.getId());

        specParams.forEach(specParam -> {
            String key = specParam.getName();
            Object value = null;

            if(specParam.getGeneric()){
                //取出SpuDetail的generic_spec
                //把json字符串转换Map集合
                Map<Long,Object> genericSpecMap = JsonUtils.toMap(spuDetail.getGenericSpec(),Long.class,Object.class) ;
                //通过规格参数ID到Map中取出对应的规格参数值
                value = genericSpecMap.get(specParam.getId());
            }else{
                //取出SpuDetail的special_spec
                //nativeRead()：专门比较复杂的类型，对象里面包装另另一个对象
                Map<Long,List<Object>> specialSpecMap = JsonUtils.nativeRead(spuDetail.getSpecialSpec(),new TypeReference<Map<Long,List<Object>>>(){});
                value = specialSpecMap.get(specParam.getId());
            }

            //3）把key和value存入一个Map集合
            specsMap.put(key,value);
        });


        goods.setAll(all);
        goods.setPrice(price);
        goods.setSkus(skuJson);

        goods.setSpecs(specsMap);
        return goods;
    }
```





## 11、构建索引库：构建Goods对象-封装specs属性优化

因为过滤参数中有一类比较特殊，就是数值区间，这些区间的值，是商品列表结果中有的才会显示在这里，如果结果中没有个数值区间的，我们不显示。



我们怎么知道结果中有哪些段？先知道结果中有那些尺寸，我们可以用聚合查询加上当前的查询条件，查询这些段，但是我们还需要拿着这些段，去一个个判断，看看那些包含，那些不包含，包含就放入页面显示，不包含就不显示，这样的做法太麻烦，效率不高。



现在的问题是：
1）不知道显示那些段
2）用段来过滤，性能太差



怎么办？这里的设计非常巧妙，当我们把屏幕尺寸存入到索引库的时候，我们不存入值，我们先把它转成段，然后再存入索引库，这样搜索的时候，我们就可以使用term词条查询了，这样的性能就很高了。例如：   5.2英寸    ，我们就存入一个段：   ==5.0-5.5英寸==，   搜索的时候，把==5.0-5.5英寸==当成词条即可。

![1529717362585](乐优电商-07-构建商品索引库【Elasticsearch】.assets/1529717362585.png)

所以我们在存入时要进行分段处理，这段逻辑比较复杂，我已经把它封装成方法，大家直接调用即可：

```java
private String chooseSegment(Object value, SpecParam p) {
    if (value == null || StringUtils.isBlank(value.toString())) {
        return "其它";
    }
    double val = parseDouble(value.toString());
    String result = "其它";
    // 保存数值段
    for (String segment : p.getSegments().split(",")) {
        String[] segs = segment.split("-");
        // 获取数值范围
        double begin = parseDouble(segs[0]);
        double end = Double.MAX_VALUE;
        if (segs.length == 2) {
            end = parseDouble(segs[1]);
        }
        // 判断是否在范围内
        if (val >= begin && val < end) {
            if (segs.length == 1) {
                result = segs[0] + p.getUnit() + "以上";
            } else if (begin == 0) {
                result = segs[1] + p.getUnit() + "以下";
            } else {
                result = segment + p.getUnit();
            }
            break;
        }
    }
    return result;
}

private double parseDouble(String str) {
    try {
        return Double.parseDouble(str);
    } catch (Exception e) {
        return 0;
    }
}
```



优化之后的完整的==**SearchService**==类如下：

```java
 /**
     * 把一个SpuDTO对象转换一个Goods对象
     */
    public Goods buildGoods(SpuDTO spuDTO){
        Goods goods = new Goods();

        goods.setId(spuDTO.getId());
        goods.setSpuName(spuDTO.getName());
        goods.setSubTitle(spuDTO.getSubTitle());
        goods.setCategoryId(spuDTO.getCid3());
        goods.setBrandId(spuDTO.getBrandId());
        goods.setCreateTime(new Date().getTime());

        //一、处理all,skus,price三个字段
        //1)根据spuId查询所有Sku对象
        List<Sku> skus = itemClient.findSkusBySpuId(spuDTO.getId());
        //2）只需要sku的id，price，images
        List<Map<String,Object>> skuMapList = new ArrayList();
        //3）打包数据
        skus.forEach(sku -> {
            Map<String,Object> skuMap = new HashMap<>();
            skuMap.put("id",sku.getId());
            skuMap.put("price",sku.getPrice());
            skuMap.put("images",sku.getImages());

            skuMapList.add(skuMap);
        });
        //4）把skuMapList转换json字符串
        String skuJson = JsonUtils.toString(skuMapList);

        //5）处理price
        Set<Long> price = skus.stream().map(Sku::getPrice).collect(Collectors.toSet());

        //6）处理all
        String all = spuDTO.getName()+spuDTO.getSubTitle()+
                skus.stream().map(Sku::getTitle).collect(Collectors.joining(""));

        //二、处理specs属性
        /**
         * 封装思路：Map<String,Object>
         *   1） 根据分类id和searching=true查询用于搜索过滤的规格参数
         *   2） 获取key，和 获取value
         *        key就是规格参数的name
         *   3） value从哪里来？ 根据spuId查询SpuDetail对象 （通用规格参数：generic_spec, 特有规格参数：special_spec）
         *         3.1 判断规格参数的generic属性
         *            如果是true，则取出generic_spec
         *            如果是false，则取出special_spec
         *         3.2 把上一步取出的值放入value
         *   4）把key和value存入一个Map集合
         */
        //1）根据分类id和searching=true查询用于搜索过滤的规格参数
        List<SpecParam> specParams = itemClient.findSpecParams(null, spuDTO.getCid3(), true);
        //2）封装Map的key和value
        Map<String,Object> specsMap = new HashMap<>();
        //根据spuId查询SpuDetail
        SpuDetail spuDetail = itemClient.findSpuDetailBySpuId(spuDTO.getId());

        specParams.forEach(specParam -> {
            String key = specParam.getName();
            Object value = null;

            if(specParam.getGeneric()){
                //取出SpuDetail的generic_spec
                //把json字符串转换Map集合
                Map<Long,Object> genericSpecMap = JsonUtils.toMap(spuDetail.getGenericSpec(),Long.class,Object.class) ;
                //通过规格参数ID到Map中取出对应的规格参数值
                value = genericSpecMap.get(specParam.getId());
            }else{
                //取出SpuDetail的special_spec
                //nativeRead()：专门比较复杂的类型，对象里面包装另另一个对象
                Map<Long,List<Object>> specialSpecMap = JsonUtils.nativeRead(spuDetail.getSpecialSpec(),new TypeReference<Map<Long,List<Object>>>(){});
                value = specialSpecMap.get(specParam.getId());
            }

            //把所有数字类型的规格参数转换为区间范围
            if(specParam.getNumeric()){
                //如果是数字才转换
                value = chooseSegment(value,specParam);
            }


            //3）把key和value存入一个Map集合
            specsMap.put(key,value);
        });


        goods.setAll(all);
        goods.setPrice(price);
        goods.setSkus(skuJson);

        goods.setSpecs(specsMap);
        return goods;
    }


    private String chooseSegment(Object value, SpecParam p) {
        if (value == null || StringUtils.isBlank(value.toString())) {
            return "其它";
        }
        double val = parseDouble(value.toString());
        String result = "其它";
        // 保存数值段
        for (String segment : p.getSegments().split(",")) {
            String[] segs = segment.split("-");
            // 获取数值范围
            double begin = parseDouble(segs[0]);
            double end = Double.MAX_VALUE;
            if (segs.length == 2) {
                end = parseDouble(segs[1]);
            }
            // 判断是否在范围内
            if (val >= begin && val < end) {
                if (segs.length == 1) {
                    result = segs[0] + p.getUnit() + "以上";
                } else if (begin == 0) {
                    result = segs[1] + p.getUnit() + "以下";
                } else {
                    result = segment + p.getUnit();
                }
                break;
            }
        }
        return result;
    }

    private double parseDouble(String str) {
        try {
            return Double.parseDouble(str);
        } catch (Exception e) {
            return 0;
        }
    }

```





## 12、构建索引库：用测试类将数据源的数据写入索引库

```java
package com.leyou.search.test;

import com.leyou.common.pojo.PageResult;
import com.leyou.item.client.ItemClient;
import com.leyou.item.dto.SpuDTO;
import com.leyou.item.entity.SpecParam;
import com.leyou.search.entity.Goods;
import com.leyou.search.repository.SearchRepository;
import com.leyou.search.service.SearchService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;
import java.util.stream.Collectors;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SearchTest {

    @Autowired
    private ItemClient itemClient;

    @Autowired
    private SearchService searchService;

    @Autowired
    private SearchRepository searchRepository;

    /**
     * 将数据源的数据写入索引库
     */
    @Test
    public void indexWrite(){
        //定义查询的页数，每页条数，总页数
        Integer page = 1, rows = 100;
        Long totalPage = 1L;
        do{
            //查询指定页的Spu数据，不带搜索条件，只查询上架商品数据
            PageResult<SpuDTO> pageResult = itemClient.goodsPageQuery(page, rows, null, true);
            //获取数据列表
            List<SpuDTO> spuDTOS = pageResult.getItems();
            //把Spu集合转成Goods集合
            List<Goods> goodsList = spuDTOS.stream().map(searchService::buildGoods).collect(Collectors.toList());
            //批量写入索引库
            searchRepository.saveAll(goodsList);
            //获取总页数
            totalPage = pageResult.getTotalPage();
            //页码往后自增
            page++;
        }while (page<=totalPage);//当当前页数不大于总页的时候执行
    }

    @Test
    public void findSpecParams(){
        List<SpecParam> specParams = itemClient.findSpecParams(null, 76l, true);
        System.out.println(specParams);
    }
}
```

执行测试代码，然后查看索引库：

![image-20200319114030981](乐优电商-07-构建商品索引库【Elasticsearch】.assets/image-20200319114030981.png)

 

## 13、搜索页渲染：把商品图片导入

现在商品表中虽然有数据，但是所有的图片信息都是无法访问的，看看数据库中给出的图片信息：

![1575543538260](乐优电商-07-构建商品索引库【Elasticsearch】.assets/1575543538260-1601258447820.png) 

现在这些图片的地址指向的是`http://image.leyou.com`这个域名。而之前我们在nginx中做了反向代理，把图片地址指向了阿里巴巴的OSS服务。

我们可以再课前资料中找到这些商品的图片：

![1575543648045](乐优电商-07-构建商品索引库【Elasticsearch】.assets/1575543648045-1601258447822.png) 

并且把这些图片放到nginx目录，然后由nginx对其加载即可。

首先，我们需要把图片放到nginx/html目录，并且解压缩：

![1575543769978](乐优电商-07-构建商品索引库【Elasticsearch】.assets/1575543769978-1601258447822.png) 

修改Nginx中，有关`image.leyou.com`的监听配置，使nginx来加载图片地址：

```nginx
server {
	listen       80;
	server_name  image.leyou.com;
	location /images { # 监听以/images打头的路径，
		root	html;
	}
	location / {
		proxy_pass   https://ly-images.oss-cn-shanghai.aliyuncs.com;
	}
}
```

再次测试：

![1575543999864](乐优电商-07-构建商品索引库【Elasticsearch】.assets/1575543999864-1601258447822.png) 



ok，商品图片都搞定了！

> Nginx反向代理路径说明
>
> `location /images`斜杠其实就是分隔符号，会匹配/后面的内容。并且根据proxy_pass中是否存在斜杠
>
> proxy_pass 有斜杠就会把匹配的内容去除 location中有的。 如果没有斜杠就把匹配的内容带过来。 
>
> 在这个案例中 
>
> http://image.leyou.com/images 代理到 {nginx目录}/html/images
>
> 即完整路径为
>
> http://image.leyou.com/images/9/15/1524.jpg  代理到 {nginx目录}/html/images/9/15/1524.jpg  



## 14、搜索页渲染：common.js工具介绍

为了方便后续的开发，我们在前台系统中定义了一些工具，放在了common.js中：

![1575858309929](乐优电商-07-构建商品索引库【Elasticsearch】.assets/1575858309929.png) 



- 首先对axios进行了一些全局url配置；

- axios请求超时时间；

- 是否允许跨域操作cookie等；

  ![1575858773458](乐优电商-07-构建商品索引库【Elasticsearch】.assets/1575858773458.png) 

- 定义了对象 ly ，也叫leyou，包含了下面的属性：

  - getUrlParam(key)：获取url路径中的参数

    ![1575858701582](乐优电商-07-构建商品索引库【Elasticsearch】.assets/1575858701582.png) 

  - http：axios对象的别名。以后发起ajax请求，可以用ly.http.get()

  - store：localstorage便捷操作，后面用到再详细说明

  - formatPrice：格式化价格，如果传入的是字符串，则扩大100被并转为数字，如果传入是数字，则缩小100倍并转为字符串

    ![1575858879276](乐优电商-07-构建商品索引库【Elasticsearch】.assets/1575858879276.png) 

  - formatDate(val, pattern)：对日期对象val按照指定的pattern模板进行格式化

    ![1575858925601](乐优电商-07-构建商品索引库【Elasticsearch】.assets/1575858925601.png) 

  - stringify：将对象转为参数字符串

  - parse：将参数字符串变为js对象

    ![1575859566989](乐优电商-07-构建商品索引库【Elasticsearch】.assets/1575859566989.png) 



  - 解构表达式

    ![1575860041644](乐优电商-07-构建商品索引库【Elasticsearch】.assets/1575860041644.png) 



## 15、搜索页渲染：渲染搜索页面分析

整个搜索页面需要渲染的部分有两大块：过滤条件部分和商品分页数据部分。

如图：

过滤条件部分

![image-20200319114747090](乐优电商-07-构建商品索引库【Elasticsearch】.assets/image-20200319114747090.png)



商品分页数据部分

![image-20200319114821281](乐优电商-07-构建商品索引库【Elasticsearch】.assets/image-20200319114821281.png)

分页信息

![image-20200724161717630](乐优电商-07-构建商品索引库【Elasticsearch】.assets/image-20200724161717630.png)



## 16、搜索页渲染：搜索页发起请求

### 1）在data中定义变量。

```js
	data: {
            //前端->后端
            searchParams:{
                key:'',
                page:1
            },

            //后端->前端
            items:[],//当前页数据列表
            total:0, //总记录数
            totalPage:0, //总页数
            filterConditions:{}//过滤条件
        }
```



### 2）在methods中提供一个商品分页查询的请求方法

```js
methods:{
            //执行搜索
            loadSearchPage(){
                ly.http.post('/search/page',this.searchParams).then(resp=>{

                })
            }
        },
```



### 3）在钩子函数中调用商品分页查询的请求方法

```js
<script type="text/javascript">
    var vm = new Vue({
        el: "#searchApp",
        data: {
            //前端->后端 参数
            //searchParams: 页码传递的参数
            searchParams:{
                key:'',//关键字
                page:1,//当前页码
            },
            //后端->前端 结果
            items:[],//商品列表
            total:1,//总记录数
            totalPage:1,//总页数
            filterConditions:{},//过滤条件
        },
        created(){
            //设置key关键词
            let key = ly.getUrlParam('key');

            //给参数赋值
            this.searchParams.key = key;

            this.loadSearchPage();
        },
        methods:{
            //商品搜索
            loadSearchPage(){
                ly.http.post('/search/page',this.searchParams).then(resp=>{

                }).catch(e=>{
                    console.log('搜索失败');
                });
            }
        },
        components:{
            lyTop: () => import("./js/pages/top.js")
        }
    });
</script>
```



### 4）通过浏览器f12查看网络中的请求是否成功发送

如图：表示请求已经发送出去

<img src="../../../../../../Documents/笔记/10.项目二/day07、构建商品索引库【Elasticsearch】/assets/1601260060623.png" alt="1601260060623" style="zoom:67%;" />





```js
<script type="text/javascript">
    var vm = new Vue({
        el: "#searchApp",
        data: {
            //前端->后端
            searchParams:{
                key:'',
                page:1,
            },

            //后端->前端
            items:[],
            total:1,//总记录数
            totalPage:1,//总页数
            filterConditions:{},//显示所有的过滤条件
        },
        created(){
            //获取URL的参数
            let key = ly.getUrlParam('key');

            //给key赋值
            this.searchParams.key = key;

            //发出请求
            this.loadSearchPage();
        },
        methods:{

            //商品搜索
            loadSearchPage(){
                ly.http.post('/search/page',this.searchParams).then(resp=>{

                }).catch(e=>{

                });
            }
        },

        components:{
            lyTop: () => import("./js/pages/top.js")
        }
    });
</script>
```





## 17、课程总结



1）商品数据导入

   1.1 设计Goods对象（每个Field如何定义？？？）

   1.2 在ly-item微服务提供Feign方法

   1.3 编写逻辑封装Goods（specs属性！！！）





