---
title: 乐优电商(04)-品牌图片上传&商品规格【OSS】
tags:
  - 笔记
  - 项目实战二
  - springcloud
  - 微服务
  - 图片上传
  - OSS
  - 阿里云
categories:
  - 项目实战二
date: 2020-12-28 12:59:56
---

## 01、课程目标

- 实现OSS文件上传
- 了解商品规格数据结构设计思路
- 实现商品规格查询





## 02、图片上传客户端：上传文件插件说明



**回顾页面上传图片三要素：**

1）必须是post请求

2）必须是多部件类型form表单（type="multipart/form-data"）

3）必须有一个type=“file”文件上传项



这三要素指的是form表单，我们的vue中没有这个表单，所以我们换一种方式，也就是做一个Vue的文件上传插件来实现文件上传，但是如果我们自己去开发这个插件，那会浪费很多时间，所以我们找一个现成的，直接放到项目中就可以用了，我们只需要看懂即可，关于这个文件上传插件的开发大家可以参考这个文件：

![1575424402339](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575424402339.png) 

看这个文章的这一项：

![1575424714898](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575424714898.png) 



有兴趣的同学可以看看，但我们的重点在微服务中。





## 03、图片本地上传：搭建本地图片服务器

图片服务器我们可以选择tomcat，也可以选择nginx，如何选择他们？

那就看谁处理静态资源更加好了，从这方面来看，nginx肯定是首选了，所以我们去nginx下的html目录下创建一个brand-img的文件夹，然后把图片上传到该目录即可，接下来我们就开始来创建图片上传的微服务。

在nginx的根目录中找到html文件夹，在里面创建一个放置品牌logo的文件夹。

![image-20200720144620364](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/image-20200720144620364.png)

在brand-logo中手动放一张图片

![image-20200720144932214](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/image-20200720144932214.png)

通过访问地址访问效果如下：

![image-20200315092558170](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/image-20200315092558170.png)



刚才的新增实现中，我们并没有上传图片，接下来我们一起完成图片上传逻辑。

文件的上传并不只是在品牌管理中有需求，以后的其它服务也可能需要，因此我们创建一个独立的微服务，专门处理各种上传。



## 04、图片本地上传：创建图片上传微服务



### 1）创建module

![1605833520130](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1605833520130.png)

![1552144636424](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1552144636424.png)



### 2）添加依赖

我们需要EurekaClient和web依赖：

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

    <artifactId>ly-upload</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <version>2.1.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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



### 3）编写配置

```yaml
server:
  port: 8082
spring:
  application:
    name: upload-service
  servlet:
    multipart:
      max-file-size: 5MB # 限制文件上传的大小
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
```

需要注意的是，我们添加了限制文件大小的配置





### 4）启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class LyUploadApplication {
    public static void main(String[] args) {
        SpringApplication.run(LyUploadApplication.class, args);
    }
}
```

结构：

![1552144792255](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1552144792255.png) 



### 5）在ly-gateway中注册网关配置

```yml
        - id: upload-service
          uri: lb://upload-service
          predicates:
            - Path=/api/upload/**
          filters:
            - StripPrefix=2
```





## 05、图片本地上传：上传功能初步实现

### 1）在application.yml定义变量



```yml
ly:
  upload:
    imagePath: D:\leyou_projects\javaee145\software\nginx-1.16.0\html\brand-logo
    imageUrl: http://localhost/brand-logo/     
```

### 2）编写Controller

点击新增品牌页面的上传图片按钮，即可看到上传图片请求：

![1552144440922](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1552144440922.png)



从图中很容易看出编写controller需要知道4个内容：

- 请求方式：上传肯定是POST
- 请求路径：/upload/image
- 请求参数：文件，参数名是file，SpringMVC会封装为一个接口：MultipleFile
- 返回结果：这里上传与表单分离，文件不是跟随表单一起提交，而是单独上传并得到结果，随后把结果跟随表单提交。因此上传成功后需要返回一个可以访问的文件的url路径

代码如下：

```java
package com.leyou.upload.controller;

import com.leyou.upload.service.UploadService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

/**
 * 上传
 */
@RestController
public class UploadController {
    @Autowired
    private UploadService uploadService;

    /**
     * 本地图片上传
     */
    @PostMapping("/image")
    public ResponseEntity<String> uploadImage(@RequestParam("file") MultipartFile file){
        String imageUrl = uploadService.uploadImage(file);
        return ResponseEntity.ok(imageUrl);
    }
}


```



### 3）编写Service

完成文件上传：

```java
package com.leyou.upload.service;

import com.leyou.common.exception.pojo.ExceptionEnum;
import com.leyou.common.exception.pojo.LyException;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.File;
import java.io.IOException;
import java.util.UUID;

/**
 *
 */
@Service
public class UploadService {

    @Value("${ly.upload.imagePath}")
    private String imagePath; // 图片保存路径
    @Value("${ly.upload.imageUrl}")
    private String imageUrl;// 图片的访问路径

    public String uploadImage(MultipartFile file) {
        //生成随机文件名称
        String uuid = UUID.randomUUID().toString();
        //获取文件的后缀名
        String originalFilename = file.getOriginalFilename();
        String extName = originalFilename.substring(originalFilename.lastIndexOf(".")); // .jpg
        //最终的文件名称
        String fileName = uuid+extName;

        try {
            //保存文件到nginx目录
            file.transferTo(new File(imagePath,fileName));

            //访问路径：http://localhost/brand-logo/3.jpg
            return imageUrl+fileName;
        } catch (IOException e) {
            e.printStackTrace();
            throw new LyException(ExceptionEnum.FILE_UPLOAD_ERROR);
        }

    }
}


```





### 4）测试上传

启动上传文件微服务和网关微服务，然后打开页面测试：

![image-20200315095935513](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/image-20200315095935513.png)



## 06、图片本地上传：上传文件格式判断

现在图片已经能够上传了，但是还不够完善，文件的上传类型我们还没有判断，我们不能让用户什么东西都往上面传，所以我们需要判断一下上传图片的格式。

mime类型，是浏览器识别的文件类型，这些类型我们不用背，随便找个tomcat里面都有：

![1575429326515](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575429326515.png) 

去web.xml中搜索即可，我们允许那种类型就选择哪个，这里我们用的是多个，所以用一个集合来存比较好，我们把这个集合作为一个全局变量，然后在上传方法中去判断，代码如下

```java
package com.leyou.upload.service;

import com.leyou.common.exception.pojo.ExceptionEnum;
import com.leyou.common.exception.pojo.LyException;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.util.UUID;

/**
 *
 */
@Service
public class UploadService {

    @Value("${ly.upload.imagePath}")
    private String imagePath; // 图片保存路径
    @Value("${ly.upload.imageUrl}")
    private String imageUrl;// 图片的访问路径

    public String uploadImage(MultipartFile file) {
        //判断该文件是否为图片流
        try {
            BufferedImage image = ImageIO.read(file.getInputStream());
            if(image==null){
                //代表该文件不是图片
                throw new LyException(ExceptionEnum.INVALID_FILE_TYPE);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }


        //生成随机文件名称
        String uuid = UUID.randomUUID().toString();
        //获取文件的后缀名
        String originalFilename = file.getOriginalFilename();
        String extName = originalFilename.substring(originalFilename.lastIndexOf(".")); // .jpg
        //最终的文件名称
        String fileName = uuid+extName;

        try {
            //保存文件到nginx目录
            file.transferTo(new File(imagePath,fileName));

            //访问路径：http://localhost/brand-logo/3.jpg
            return imageUrl+fileName;
        } catch (IOException e) {
            e.printStackTrace();
            throw new LyException(ExceptionEnum.FILE_UPLOAD_ERROR);
        }

    }
}



```





## 07、图片本地上传：修改nginx上传文件大小限制

我们上传一个超过1M大小的文件

![1552145659243](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1552145659243.png)

发现收到错误响应：

![1552145695272](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1552145695272.png)



这是nginx对文件大小的限制（默认为1M），我们`leyou.conf`中的server的外面，设置大小限制为5M：

```
client_max_body_size 5m;
```

如下图：

![1575430521729](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575430521729.png) 



重启nginx，再次测试，可以了，完美！

```
nginx.exe -s reload
```

> nginx 下的logs文件夹可以看到相关日志 如果nginx可以在日志中查看

前端的代码已经限制最大上传2m，如果要大于这个值，需要改动代码：

![1578445546507](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1578445546507.png) 

![1603763675075](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1603763675075.png)



## 08、图片云存储：云存储介绍

讲课前，我们先思考一下，之前上传的功能，有没有什么问题？

上传本身没有任何问题，问题出在保存文件的方式，我们是保存在服务器机器，就会有下面的问题：

- 单机器存储，存储能力有限
- 无法进行水平扩展，因为多台机器的文件无法共享,会出现访问不到的情况
- 数据没有备份，有单点故障风险
- 并发能力差

这个时候，最好使用分布式文件存储来代替本地文件存储。





### 1）什么是分布式文件系统

分布式文件系统（Distributed File System）是指文件系统管理的物理存储资源不一定直接连接在本地节点上，而是通过计算机网络与节点相连。 

通俗来讲：

- 传统文件系统管理的文件就存储在本机。
- 分布式文件系统管理的文件存储在很多机器，这些机器通过网络连接，要被统一管理。无论是上传或者访问文件，都需要通过管理中心来访问

常见的分布式文件系统有谷歌的GFS、HDFS（Hadoop）、TFS（淘宝）、Fast DFS（淘宝）等。

不过，企业自己搭建分布式文件系统成本较高，对于一些中小型企业而言，使用云上的文件存储，是性价比更高的选择。



### 2）分布式存储方案

- 七牛：     https://www.qiniu.com/
- 阿里云： https://www.aliyun.com/product/oss
- 腾讯云： https://cloud.tencent.com/product/cos
- 华为云： https://www.huaweicloud.com/product/obs.html
- 亚马逊： https://aws.amazon.com/cn/s3/

七牛我们用过了，亚马逊国外的 就不用了，华为刚出先不用，那么阿里云和腾讯云两个中我们可以选择阿里云，因为阿里云做云存储比腾讯云早点，我用它。







### 3）阿里云对象存储OSS介绍

从官方进去，找到产品分类，找到对象存储OSS即可：

![1575439951112](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575439951112.png) 

点击链接，去到对象存储页面：

![1575440028426](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575440028426.png) 

阿里的OSS就是一个文件云存储方案，简介：

> 阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。其数据设计持久性不低于99.999999999%，服务设计可用性不低于99.99%。具有与平台无关的RESTful API接口，您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。
>
> 您可以使用阿里云提供的API、SDK接口或者OSS迁移工具轻松地将海量数据移入或移出阿里云OSS。数据存储到阿里云OSS以后，您可以选择标准类型（Standard）的阿里云OSS服务作为移动应用、大型网站、图片分享或热点音视频的主要存储方式，也可以选择成本更低、存储期限更长的低频访问类型（Infrequent Access）和归档类型（Archive）的阿里云OSS服务作为不经常访问数据的备份和归档。





## 09、图片云存储：阿里云对象存储OSS

### 1）开通OSS访问

点击立即开通按钮，就可以开通服务，没有登录会跳转到登录页面：

![1575440350543](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575440350543.png) 

开通后，即可以进入管理控制台：

![1575440806451](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575440806451.png) 



### 2）对象存储OSS的基本概念

OSS中包含一些概念，我们来认识一下：

相关概念可以查看官方链接： https://help.aliyun.com/document_detail/31817.html



- 存储类型（Storage Class）

  OSS 提供标准、低频访问、归档三种存储类型，全面覆盖从热到冷的各种数据存储场景。其中标准存储类型提供高可靠、高可用、高性能的对象存储服务，能够支持频繁的数据访问；低频访问存储类型适合长期保存不经常访问的数据（平均每月访问频率 1 到 2 次），存储单价低于标准类型；归档存储类型适合需要长期保存（建议半年以上）的归档数据，在三种存储类型中单价最低。详情请参见[存储类型介绍](https://help.aliyun.com/document_detail/51374.html#concept-fcn-3xt-tdb)。

- ==存储空间（Bucket）==

  存储空间是您用于存储对象（Object）的容器，所有的对象都必须隶属于某个存储空间。存储空间具有各种配置属性，包括地域、访问权限、存储类型等。您可以根据实际需求，创建不同类型的存储空间来存储不同的数据。创建存储空间请参见[创建存储空间](https://help.aliyun.com/document_detail/31842.html#concept-ntj-wx1-5db)。

- 对象/文件（Object）

  对象是 OSS 存储数据的基本单元，也被称为 OSS 的文件。对象由元信息（Object Meta）、用户数据（Data）和文件名（Key）组成。对象由存储空间内部唯一的 Key 来标识。对象元信息是一组键值对，表示了对象的一些属性，比如最后修改时间、大小等信息，同时您也可以在元信息中存储一些自定义的信息。

- 地域（Region）

  地域表示 OSS 的数据中心所在物理位置。您可以根据费用、请求来源等选择合适的地域创建 Bucket。详情请参见 [OSS 已开通的Region](https://help.aliyun.com/document_detail/31837.html#concept-zt4-cvy-5db)。

- ==访问域名（Endpoint）==

  Endpoint 表示 OSS 对外服务的访问域名。OSS 以 HTTP RESTful API 的形式对外提供服务，当访问不同地域的时候，需要不同的域名。通过内网和外网访问同一个地域所需要的域名也是不同的。具体的内容请参见[各个 Region 对应的 Endpoint](https://help.aliyun.com/document_detail/31837.html#concept-zt4-cvy-5db)。

- ==访问密钥（AccessKey）==

  AccessKey（简称 AK）指的是访问身份验证中用到的 AccessKeyId 和 AccessKeySecret。OSS 通过使用 AccessKeyId 和 AccessKeySecret 对称加密的方法来验证某个请求的发送者身份。AccessKeyId 用于标识用户；AccessKeySecret 是用户用于加密签名字符串和 OSS 用来验证签名字符串的密钥，必须保密。获取 AccessKey 的方法请参见[创建 AccessKey](https://help.aliyun.com/document_detail/53045.html#concept-53045-zh)。





### 3）创建一个Bucket（存储桶）

在控制台的右侧，可以看到一个`新建Bucket`按钮：

![1575441974929](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575441974929.png) 

点击后，弹出对话框，填写基本信息：

![1575442212986](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575442212986.png) 

填写完基本信息后，点击确定按钮就可以完成Bucket的创建

==**注意点：**==

- bucket：存储空间名称，名字只能是字母、数字、中划线
- 区域：即服务器的地址，这里选择了离我们最近的深圳
- Endpoint：选中区域后，会自动生成一个Endpoint地址，这将是我们访问OSS服务的域名的组成部分
- 存储类型：默认
- 读写权限：这里我们选择公共读，否则每次访问都需要额外生成签名并校验，比较麻烦。敏感数据设置为私有即可！
- 日志：不开通



### 4）设置跨域访问(***)

由于我们的域名是leyou.com，而阿里云的域名是aliyuncs.com，所以会产生跨域访问，我们设置下：

![1575444412056](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575444412056.png)

 

点击 ![1575444758298](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575444758298.png) 设置跨域规则如下：

![image-20200315104844853](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/image-20200315104844853.png) 



### 5）创建AccessKey

有了bucket就可以进行文件上传或下载了。不过，为了安全考虑，我们给阿里云账户开通一个子账户，并设置对OSS的读写权限。

点击屏幕右上角的个人图像，然后点击访问控制：

![1575442872330](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575442872330.png) 

在跳转的页面中，选择用户，并新建一个用户： 

![1575443068711](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575443068711.png) 

然后填写用户信息：

![1575443013764](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575443013764.png) 

然后会为你生成用户的AccessKeyID和AccessKeySecret，下图马赛克部分：

![1575443194990](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575443194990.png) 

```
LTAI4GE2hZZF1iCZEXagEeHg
PDHcwUktVKqwzYO0x9DEhETrQrQwtW
```



妥善保管，不要告诉任何人！

接下来，我们需要给这个用户添加对OSS的控制权限。

进入这个新增的用户详情页面： 

![1575443396855](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575443396855.png) 

点击添加权限，会进入权限选择页面，输入oss进行搜索，然后选择`管理对象存储服务（OSS）`权限： 

![1575443529053](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575443529053.png) 



到此全部准备工作已经准备好了！







## 10、图片云存储：上传方案说明

https://www.aliyun.com/product/oss?spm=5176.19720258.J_8058803260.12.e9392c4aM4Jct2

![1603765994241](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1603765994241.png)

在控制台的右侧，点击`开发者指南`按钮，即可查看帮助文档： 

![1575445994148](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575445994148.png) 

然后在弹出的新页面的左侧菜单中找到开发者指南：

![1603766049642](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1603766049642.png) 

可以看到上传文件中，支持多种上传方式，并且因为提供的Rest风格的API，任何语言都可以访问OSS实现上传。



我们可以直接使用java代码来实现把图片上传到OSS，不过这样以来文件会先从客户端浏览器上传到我们的服务端tomcat，然后再上传到OSS，效率较低，如图： 

![1575446365802](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575446365802.png) 

以上方法有三个缺点：

- 上传慢。先上传到应用服务器，再上传到OSS，网络传送比直传到OSS多了一倍。如果直传到OSS，不通过应用服务器，速度将大大提升，而且OSS采用BGP带宽，能保证各地各运营商的速度。
- 扩展性差。如果后续用户多了，应用服务器会成为瓶颈。
- 费用高。需要准备多台应用服务器。由于OSS上传流量是免费的，如果数据直传到OSS，不通过应用服务器，那么将能省下几台应用服务器。

在阿里官方的最佳实践中，推荐了更好的做法： 

![1575448077824](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575448077824.png) 

直接从前端（客户端浏览器）上传文件到OSS。



### 1）web前端直传分析

阿里官方文档中，对于web前端直传又给出了3种不同方案：

- [JavaScript客户端签名直传](https://help.aliyun.com/document_detail/31925.html?spm=a2c4g.11186623.2.10.6c5762121wgIAS#concept-frd-4gy-5db)：客户端通过JavaScript代码完成签名，然后通过表单直传数据到OSS。
- [服务端签名后直传](https://help.aliyun.com/document_detail/31926.html?spm=a2c4g.11186623.2.11.6c5762121wgIAS#concept-en4-sjy-5db)：客户端上传之前，由服务端完成签名，前端获取签名，然后通过表单直传数据到OSS。
- [服务端签名直传并设置上传回调](https://help.aliyun.com/document_detail/31927.html?spm=a2c4g.11186623.2.12.6c5762121wgIAS#concept-qp2-g4y-5db)：服务端完成签名，并且服务端设置了上传后回调，然后通过表单直传数据到OSS。OSS回调完成后，再将应用服务器响应结果返回给客户端。



各自有一些优缺点：

- JavaScript客户端签名直传：
  - 优点：在客户端通过JavaScript代码完成签名，无需过多配置，即可实现直传，非常方便。
  - 问题：客户端通过JavaScript把AccesssKeyID 和AccessKeySecret写在代码里面有泄露的风险
- 服务端签名，JavaScript客户端直传：
  - 优点：Web端向服务端请求签名，然后直接上传，不会对服务端产生压力，而且安全可靠
  - 问题：服务端无法实时了解用户上传了多少文件，上传了什么文件
- 服务端签名直传并设置上传回调：
  - 优点：包含服务端签名后客户端直传的所有优点，并且可以设置上传回调接口路径。上传结束后，OSS会主动将上传的结果信息发送给回调接口。

经过一番分析，大家觉得我们会选哪种方案？

这里我们选择第二种，因为我们并不需要了解用户上传的文件的情况。

### 2）服务器端签名后直传流程

服务端签名后直传的原理如下：

1. 用户发送上传Policy请求到应用服务器（我们的微服务）。
2. 应用服务器返回上传Policy和签名给用户。
3. 用户直接上传数据到OSS。

流程图：

![1552311833528](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1552311833528.png)

根据上面的流程，我们需要做的事情包括：

- 改造文件上传组件，达成下面的目的：
  - 上传文件前，向服务端发起请求，获取签名
  - 上传时，修改上传目的地到阿里OSS服务，并携带上传的签名
- 编写服务端接口，接收请求，生成签名后返回



开发思路：

1）前端点击上传的时候，请求微服务获取签名（修改url，开启oss签名need-signature）
2）ly-upload微服务生成签名，返回给前端（参考阿里的Java API生成并返回签名）
3）前端再次发起请求，把图片提交到OSS（已经做好）



## 11、图片云存储：改造上传组件

文件上传组件是我们自定义组件，用法可以参考课前资料的：《自定义组件用法》。

而且，我们的上传组件内部已经实现了对阿里OSS的支持了：

![1552314430650](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1552314430650.png)

所以我们只需要在`BrandForm`页面中，给`v-upload`组件，加上`need-signature`属性，并且指定一个获取签名的url地址即可：

![1605839717704](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1605839717704.png)

自定义的上传组件中的核心代码：Upload.vue

![1552315238995](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1552315238995.png)

这段代码的核心思路：

- 判断needSignature是否为true，true代表需要签名，false代表不需要
- 如果false，直接上传文件到url
- 如果为true，则把url作为签名接口路径，访问获取签名
- 拿到响应结果后，其中可不仅仅包括签名，而是包括：
  - host：上传到阿里的url路径（**bucket + endpoint**）
  - policy：上传的策略
  - signature：签名
  - accessId：就是AccessKeyID
  - dir：在oss的bucket中的子文件夹的名称，可以不写。

因此我们需要在签名返回时，携带这些参数。



刷新页面，可以看到请求已经发出：

![1552315014021](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1552315014021.png)









## 12、图片云存储：OSS上传处理器编写

查看接口文档：

![1575449278319](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575449278319.png) 

按照接口文档在ly-upload项目的UploadController，添加getOssSignature方法

```java
/**
     * 生成阿里云的OSS签名
     */
    @GetMapping("/signature")
    public ResponseEntity<Map<String, String>> signature(){
        Map<String, String> resultMap = uploadService.signature();
        return ResponseEntity.ok(resultMap);
    }

```





## 13、图片云存储：OSS上传后台业务层代码(*)

接下来需要写业务层代码，但是这些代码应该是阿里云提供的，它也确实提供了相应的代码：

下面给出了一段示例代码：

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

    String accessId = "<yourAccessKeyId>"; // 请填写您的AccessKeyId。
    String accessKey = "<yourAccessKeySecret>"; // 请填写您的AccessKeySecret。
    String endpoint = "oss-cn-hangzhou.aliyuncs.com"; // 请填写您的 endpoint。
    String bucket = "bucket-name"; // 请填写您的 bucketname 。
    String host = "http://" + bucket + "." + endpoint; // host的格式为 bucketname.endpoint
    // callbackUrl为 上传回调服务器的URL，请将下面的IP和Port配置为您自己的真实信息。
    String callbackUrl = "http://88.88.88.88:8888";
    String dir = "user-dir-prefix/"; // 用户上传文件时指定的前缀。

    OSSClient client = new OSSClient(endpoint, accessId, accessKey);
    
    try {
        long expireTime = 30;
        long expireEndTime = System.currentTimeMillis() + expireTime * 1000;
        Date expiration = new Date(expireEndTime);
        PolicyConditions policyConds = new PolicyConditions();
        policyConds.addConditionItem(PolicyConditions.COND_CONTENT_LENGTH_RANGE, 0, 1048576000);
        policyConds.addConditionItem(MatchMode.StartWith, PolicyConditions.COND_KEY, dir);

        String postPolicy = client.generatePostPolicy(expiration, policyConds);
        byte[] binaryData = postPolicy.getBytes("utf-8");
        String encodedPolicy = BinaryUtil.toBase64String(binaryData);
        String postSignature = client.calculatePostSignature(postPolicy);

        Map<String, String> respMap = new LinkedHashMap<String, String>();
        respMap.put("accessid", accessId);
        respMap.put("policy", encodedPolicy);
        respMap.put("signature", postSignature);
        respMap.put("dir", dir);
        respMap.put("host", host);
        respMap.put("expire", String.valueOf(expireEndTime / 1000));
        // respMap.put("expire", formatISO8601Date(expiration));

        JSONObject jasonCallback = new JSONObject();
        jasonCallback.put("callbackUrl", callbackUrl);
        jasonCallback.put("callbackBody",
                          "filename=${object}&size=${size}&mimeType=${mimeType}&height=${imageInfo.height}&width=${imageInfo.width}");
        jasonCallback.put("callbackBodyType", "application/x-www-form-urlencoded");
        String base64CallbackBody = BinaryUtil.toBase64String(jasonCallback.toString().getBytes());
        respMap.put("callback", base64CallbackBody);

        JSONObject ja1 = JSONObject.fromObject(respMap);
        // System.out.println(ja1.toString());
        response.setHeader("Access-Control-Allow-Origin", "*");
        response.setHeader("Access-Control-Allow-Methods", "GET, POST");
        response(request, response, ja1.toString());

    } catch (Exception e) {
        // Assert.fail(e.getMessage());
        System.out.println(e.getMessage());
    }
}
```

可以看到代码分4部分：

- 初始化OSSClient，这是阿里SDK提供的工具
- 生成文件上传策略policy（文件大小、文件存储目录等）
- 生成许可签名
- 封装结果并返回

接下来，我们逐个完成。

### 1）引入阿里sdk的依赖

```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>3.4.2</version>
    <scope>compile</scope>
</dependency>
```



### 2）抽取签名配置

上面的示例代码中，把很多的配置硬编码到代码中，我们统一抽取放到application.yml中

```yaml
ly:
  oss:
    accessKeyId: LTAID13FA8uCV
    accessKeySecret: CgWgVzbulWQ2fagXvScAmYBniQL
    host: http://ly-upload-server.oss-cn-shenzhen.aliyuncs.com # 访问oss的域名，很重要bucket + endpoint
    endpoint: oss-cn-shenzhen.aliyuncs.com # 你的服务的端点，不一定跟我一样
    dir: "" # 保存到bucket的某个子目录
    expireTime: 20 # 过期时间，单位是S
    maxFileSize: 5242880 #文件大小限制，这里是5M
```



### 3）编写配置类加载配置

```java
package com.leyou.upload.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

/**
 * 读取Oss相关配置
 *  让@ConfigurationProperties注解生效两种方法：
 *     1）在启动类使用@EnableConfigurationProperties(OssProperties.class)开启配置读取
 *     2）在当前类上直接使用@Component注解
 */
@Data
@Component
@ConfigurationProperties(prefix = "ly.oss")
public class OssProperties {
    private String accessKeyId;
    private String accessKeySecret;
    private String host;
    private String endpoint;
    private String dir;
    private Long expireTime;
    private Long maxFileSize;
}


```



### 4）生成签名客户端

```java
package com.leyou.upload.config;

import com.aliyun.oss.OSS;
import com.aliyun.oss.OSSClientBuilder;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Oss初始化工具类对象配置
 */
@Configuration
public class OssConfig {
    /*
    @Autowired
    private OssProperties ossProps*/;

    /**
     * 注意：在@Bean的方法的参数上面可以直接注入IOC容器的对象，不需要任何注解
     * @param ossProps
     * @return
     */
    @Bean
    public OSS createOss(OssProperties ossProps){
        return new OSSClientBuilder().build(
                ossProps.getEndpoint(), 
                ossProps.getAccessKeyId(), 
                ossProps.getAccessKeySecret());
    }

}

```

### 5）业务层代码添加如下内容

```java
public Map<String,String> signature() {
        try {
            long expireTime = ossProps.getExpireTime();
            long expireEndTime = System.currentTimeMillis() + expireTime * 1000;
            Date expiration = new Date(expireEndTime);
            // PostObject请求最大可支持的文件大小为5 GB，即CONTENT_LENGTH_RANGE为5*1024*1024*1024。
            PolicyConditions policyConds = new PolicyConditions();
            policyConds.addConditionItem(PolicyConditions.COND_CONTENT_LENGTH_RANGE, 0, ossProps.getMaxFileSize());
            policyConds.addConditionItem(MatchMode.StartWith, PolicyConditions.COND_KEY, ossProps.getDir());

            String postPolicy = ossClient.generatePostPolicy(expiration, policyConds);
            byte[] binaryData = postPolicy.getBytes("utf-8");
            String encodedPolicy = BinaryUtil.toBase64String(binaryData);
            String postSignature = ossClient.calculatePostSignature(postPolicy);

            Map<String, String> respMap = new LinkedHashMap<String, String>();
            respMap.put("accessId", ossProps.getAccessKeyId());//注意：这个accessId必须和Upload.vue页面的key一致
            respMap.put("policy", encodedPolicy);
            respMap.put("signature", postSignature);
            respMap.put("dir", ossProps.getDir());
            respMap.put("host", ossProps.getHost());
            respMap.put("expire", String.valueOf(expireEndTime)); //注意：因为Upload.vue页面使用毫秒单位来判断超时时间

            return respMap;
        } catch (Exception e) {
            System.out.println(e.getMessage());
            throw new LyException(ExceptionEnum.INVALID_NOTIFY_SIGN);
        } finally {
            ossClient.shutdown();
        }
    }


```



### 6）重启微服务测试

![1575454168088](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575454168088.png) 







## 14、图片云存储：隐藏阿里云的url地址

在刚才的业务中，我们直接把我们在阿里的域名对外暴露了，如果要隐藏域名信息，我们可以使用一个自定义域名。

另外，最好保持与我们自己的域名一致，我们可以使用：`http://image.leyou.com`，然后使用nginx反向代理，最终再指向阿里服务器域名。

首先，修改服务器端返回的请求域名，修改ly-upload项目中的application.yml文件：

```yaml
ly:
  oss:
    host: http://image.leyou.com
```

然后在nginx中设置反向代理leyou.conf：

```nginx
server {
	listen       80;
	server_name  image.leyou.com;
	location / {
		proxy_pass   http://ly-upload-server.oss-cn-shenzhen.aliyuncs.com;
	}
}
```

重启后测试，一切OK

![1575454842821](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575454842821.png) 







## 15、商品规格：规格组和规格参数表结构说明

### 1）规格属性内容

商品中都有属性，不同商品，属性往往不同，这一部分数据很重要，我们一起来看看：

我们看下京东中商品的规格属性：

一款华为手机的属性：

![1575455655488](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575455655488.png) 

一款空调的属性：

![1575455706369](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575455706369.png) 

我们发现，不同商品的属性名称竟然不同，假如我们要把属性放入一张表去保存，表字段该如何设计？

别着急，我们再看另一个手机的属性：

三星手机的规格属性：

![1526087142454](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1526087142454.png) 

我们发现，虽然不同商品，规格不同。但是同一分类的商品，比如都是手机，其规格参数名称是一致的，但是值不一样。

也就是说，商品的规格参数应该是与分类绑定的。**每一个分类都有统一的规格参数模板，但不同商品其参数值可能不同**。

因此：

- 规格参数的名称（key）与值（value）应该分开来保存
- 一个分类，对应一套规格参数模板，只有规格参数key，没有值
- 一个分类对应多个商品，每个商品的规格值不同，每个商品对应一套规格的值



### 2）规格与规格组

值我们暂且不管，新增商品时，再来填写规格参数值即可，我们先思考**规格参数模板**（key）该如何设计。

来看下规格参数的结构：

![1534634716761](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1534634716761.png) 

- 规格数据首先要分组，组内再有不同的规格参数
- 一个分类规格模板中，有多个规格组
- 每个规格组中，包含多个规格参数

从面向对象的思想来看，我们规格参数和规格组分别是两类事务，并且组与组内参数成**一对多关系**，因此可以有两个类分别描述他们，那么从数据库设计来看，也就对应两张不同的表：



- 规格组：tb_spec_group
  - 一个商品分类下有多个规格组
- 规格参数：tb_spec_param
  - 一个规格组下，有多个规格参数

如图：

![1534636185452](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1534636185452.png)

大家接下来要思考的就是：

- 描述规格组需要哪些属性？
- 描述规格参数需要哪些属性？

想清楚上面的问题，就知道表该怎么设计了。



### 3）规格组表结构

规格参数分组表：tb_spec_group

```mysql
CREATE TABLE `tb_spec_group` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `cid` bigint(20) NOT NULL COMMENT '商品分类id，一个分类下有多个规格组',
  `name` varchar(32) NOT NULL COMMENT '规格组的名称',
  `create_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `key_category` (`cid`)
) ENGINE=InnoDB AUTO_INCREMENT=15 DEFAULT CHARSET=utf8 COMMENT='规格参数的分组表，每个商品分类下有多个规格参数组';
```

规格组有3个字段：

- id：主键
- cid：商品分类id，一个分类下有多个模板
- name：该规格组的名称。





### 4）规格参数表结构

规格参数表：tb_spec_param

```mysql
CREATE TABLE `tb_spec_param` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `cid` bigint(20) NOT NULL COMMENT '商品分类id',
  `group_id` bigint(20) NOT NULL,
  `name` varchar(128) NOT NULL COMMENT '参数名',
  `numeric` tinyint(1) NOT NULL COMMENT '是否是数字类型参数，true或false',
  `unit` varchar(128) DEFAULT '' COMMENT '数字类型参数的单位，非数字类型可以为空',
  `generic` tinyint(1) NOT NULL COMMENT '是否是sku通用属性，true或false',
  `searching` tinyint(1) NOT NULL COMMENT '是否用于搜索过滤，true或false',
  `segments` varchar(1024) DEFAULT '' COMMENT '数值类型参数，如果需要搜索，则添加分段间隔值，如CPU频率间隔：0.5-1.0',
  `create_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `key_group` (`group_id`),
  KEY `key_category` (`cid`)
) ENGINE=InnoDB AUTO_INCREMENT=24 DEFAULT CHARSET=utf8 COMMENT='规格参数组下的参数名';
```

按道理来说，我们的规格参数就只需要记录参数名、组id、商品分类id即可。但是这里却多出了很多字段，为什么？





#### 数值类型

某些规格参数可能为数值类型，我们需要标记出来，并且记录单位：

​	 ![1534636623281](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1534636623281.png)

我们有两个字段来描述：

- numberic：是否为数值类型
  - true：数值类型
  - false：不是数值类型
- unit：参数的单位





#### 搜索过滤

打开一个搜索页，我们来看看过滤的条件：

![1526090072535](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1526090072535.png)

你会发现，过滤条件中的屏幕尺寸、运行内存、网路、机身内存、电池容量、CPU核数等，在规格参数中都能找到：

 ![1526090228171](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1526090228171.png)

也就是说，规格参数中的数据，将来会有一部分作为搜索条件来使用。我们可以在设计时，将这部分属性标记出来，将来做搜索的时候，作为过滤条件。

与搜索相关的有两个字段：

- searching：标记是否用作过滤
  - true：用于过滤搜索
  - false：不用于过滤
- segments：某些数值类型的参数，在搜索时需要按区间划分，这里提前确定好划分区间
  - 比如电池容量，0~2000mAh，2000mAh~3000mAh，3000mAh~4000mAh

乐优商城是一个全品类的电商网站，因此商品的种类繁多，每一件商品，其属性又有差别。为了更准确描述商品及细分差别，抽象出两个概念：SPU和SKU，了解一下。





#### 通用属性

有一个generic属性，代表通用属性，我们在商品数据结构时再聊。







## 16、商品规格：商品规格模块准备工作

### 1）页面请求

首先是规格组管理，我们点击菜单中的`规格参数`：

![1575532225534](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575532225534.png) 

打开规格参数页面，可以看到页面发生了变化，看到如下内容：

![1552659142800](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1552659142800.png)

可以看到规格参数页面的左侧是一个商品分类的树，右侧暂时是空白。那么问题来了：

我们这里是规格管理，为什么会显示商品分类信息呢？



因为规格是跟商品分类绑定的，我们在看到规格参数时，肯定希望知道接下来要管理的是哪个分类下的规格参数。

所以首先会展现商品分类树，并且提示你要选择商品分类，才能看到规格参数的模板。



此时，我们点击分类，当点击到某个3级分类时，页面发生了变化：



页面右侧出现了数据表格，不过还没有数据。同时，我们查看页面控制台，可以看到请求已经发出：

![1552488184332](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1552488184332.png)

这个请求查询的应该就是当前分类下的规格参数组信息了。



规格组的数据有了，但是我们点击的菜单是==规格参数==，那规格参数在哪里显示呢？

当我们选中`主体`这个规格组时，其它规格组信息都隐藏了，表格变成了一个空白表格。

通过表头可以看出，这个表格显示的应该是`主体`这个规格组内的规格参数信息。

此时查看控制台，发现页面发起了新的请求：

![1552661146177](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1552661146177.png)

可以看出，这个是根据规格组的id，查询规格参数的请求



<font size=5><b>页面显示说明：</b></font>

==**所以第一步我们要先展示规格组，第二步点击了具体的规格组后，再根据规格组id去查询规格参数**==

![1575536583343](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575536583343.png) 



### 2）规格组entity对象

中添加实体类：

```java
package com.leyou.item.entity;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;

import java.util.Date;

@Data
@TableName("tb_spec_group")
public class SpecGroup {

    @TableId(type = IdType.AUTO)
    private Long id;

    private Long cid;

    private String name;

    private Date createTime;

    private Date updateTime;
}
```



### 3）规格参数entity对象

```java
package com.leyou.item.entity;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;

import java.util.Date;

@Data
@TableName("tb_spec_param")
public class SpecParam {
    @TableId(type = IdType.AUTO)
    private Long id;
    private Long cid;
    private Long groupId;
    private String name;
    private Boolean numeric;
    private String unit;
    private Boolean generic;
    private Boolean searching;
    private String segments;
    private Date createTime;
    private Date updateTime;
}
```



### 4）规格组的Mapper

```java
package com.leyou.item.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.leyou.item.entity.SpecGroup;

public interface SpecGroupMapper extends BaseMapper<SpecGroup> {
}
```



### 5）规格参数的Mapper

```java
package com.leyou.item.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.leyou.item.entity.SpecParam;

public interface SpecParamMapper extends BaseMapper<SpecParam> {
}
```



### 6）商品规格的Service

```java
package com.leyou.item.service;

import com.leyou.item.mapper.SpecGroupMapper;
import com.leyou.item.mapper.SpecParamMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@Transactional
public class SpecService {

    @Autowired
    private SpecGroupMapper groupMapper;

    @Autowired
    private SpecParamMapper paramMapper;

}
```

我们现在是准备工作，先把大体代码结构写好，再写具体业务。

### 7）商品规格的Controller

```java
package com.leyou.item.controller;

import com.leyou.item.service.SpecService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SpecController {

    @Autowired
    private SpecService specService;

}
```



好了，准备工作我们做完了，接下来编写具体的接口代码了！







## 17、商品规格：根据分类查询规格组



### 1）编写处理器

在SpecController中添加如下方法

```java
package com.leyou.item.controller;

import com.leyou.item.pojo.SpecGroup;
import com.leyou.item.service.SpecService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

/**
 * 规格
 */
@RestController
public class SpecController {
    @Autowired
    private SpecService specService;

    /**
     * 根据分类ID查询规格组
     */
    @GetMapping("/spec/groups/of/category")
    public ResponseEntity<List<SpecGroup>> findSpecGroupsByCid(@RequestParam("id") Long id){
        List<SpecGroup> specGroups = specService.findSpecGroupsByCid(id);
        return ResponseEntity.ok(specGroups);
    }
}


```



### 2）编写Service

在SpecService添加如下方法

```java
package com.leyou.item.service;

import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.core.toolkit.CollectionUtils;
import com.baomidou.mybatisplus.core.toolkit.Wrappers;
import com.leyou.common.exception.pojo.ExceptionEnum;
import com.leyou.common.exception.pojo.LyException;
import com.leyou.item.mapper.SpecGroupMapper;
import com.leyou.item.mapper.SpecParamMapper;
import com.leyou.item.pojo.SpecGroup;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

/**
 * 规格业务
 */
@Service
@Transactional
public class SpecService {
    @Autowired
    private SpecGroupMapper specGroupMapper;
    @Autowired
    private SpecParamMapper specParamMapper;

    public List<SpecGroup> findSpecGroupsByCid(Long id) {
        //1.封装条件
        SpecGroup specGroup = new SpecGroup();
        specGroup.setCid(id);
        QueryWrapper<SpecGroup> queryWrapper = Wrappers.query(specGroup);
        
        //2.执行查询，获取结果
        List<SpecGroup> specGroups = specGroupMapper.selectList(queryWrapper);
        
        //3.处理和返回结果
        if(CollectionUtils.isEmpty(specGroups)){
            throw new LyException(ExceptionEnum.SPEC_NOT_FOUND);
        }
        return specGroups;
    }
}


```



重启微服务，测试，页面规格组数据加载了！







## 18、商品规格：根据规格组查询规格参数

查看文档：

![1578395494883](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1578395494883.png) 



### 1）编写处理器

在SpecController中添加如下方法

```java
      /**
     * 根据条件查询规格参数（gid,cid,searching）
     */
    @GetMapping("/spec/params")
    public ResponseEntity<List<SpecParam>> findSpecParams(
            @RequestParam(value = "gid",required = false) Long gid,
            @RequestParam(value = "cid",required = false) Long cid,
            @RequestParam(value = "searching",required = false) Boolean searching
    ){
        List<SpecParam> specParams = specService.findSpecParams(gid,cid,searching);
        return ResponseEntity.ok(specParams);
    }
```



### 2）编写Service

在SpecService添加如下方法

```java
public List<SpecParam> findSpecParams(Long gid, Long cid, Boolean searching) {
        //判断gid和cid不能同时存在
        if(gid!=null && cid!=null){
            throw new LyException(ExceptionEnum.INVALID_PARAM_ERROR);
        }

        //1.封装条件  注意：MyBatis-Plus底层自动判断NULL值，则不作为查询条件
        SpecParam specParam = new SpecParam();
        specParam.setGroupId(gid);
        specParam.setCid(cid);
        specParam.setSearching(searching);

        QueryWrapper<SpecParam> queryWrapper = Wrappers.query(specParam);

        //2.执行查询，获取结果
        List<SpecParam> specParams = specParamMapper.selectList(queryWrapper);

        //3.处理和返回结果
        if(CollectionUtils.isEmpty(specParams)){
            throw new LyException(ExceptionEnum.SPEC_NOT_FOUND);
        }
        return specParams;
    }
```



重启微服务，查询页面规格参数数据时出现了异常！

![1575541404707](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575541404707.png) 

这是什么导致的呢？

仔细阅读错误说明，说sql语句有误，仔细观察发现Sql语句中有一个字段，名为`numeric`，这个字段名称与数据库名称冲突，这种情况下我们都需要对字段进行转义，变为普通字符串。





## 19、商品规格：sql语句中出现mysql关键字处理

sql是由mybatis-plus生成的，我们该如何修改Sql语句中的字段呢？

通过mybatis-plus的@TableField注解，来声明这个字段对应的列名：

```java
@Data
@TableName("tb_spec_param")
public class SpecParam {
    @TableId(type = IdType.AUTO)
    private Long id;
    private Long cid;
    private Long groupId;
    private String name;
    @TableField("`numeric`")
    private Boolean numeric;
    private String unit;
    private Boolean generic;
    private Boolean searching;
    private String segments;
    private Date createTime;
    private Date updateTime;
}
```

重启后  打开页面

![1575541714644](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575541714644.png) 

在页面刷新：

![1575541783067](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575541783067.png) 

控制台输出的sql：

发现已经加入了反引号处理：

![1575542458197](乐优电商-04-品牌图片上传-商品规格【OSS】.assets/1575542458197.png) 



## 20、课程总结





1）图片上传

​    1.1 本地上传：把图片上传都nginx服务器    复习后台如何接收文件

   1.2 阿里云OSS上传：  服务端签名后上传

​         1）前端：v-upload文件上传插件     

```html
<v-upload v-model="brand.image" :multiple="false" :pic-width="250" :pic-height="90"
                        url="/upload/signature" need-signature/>
```



​      2）生成签名给前端（重点）

​           学会如何在SpringBoot项目中抽取配置（application.yml） 使用@Value或@ConfigurationProperties读取

​            查看阿里云的OSS API抽取



2）商品规格（重点）

​           2.1 学习商品分类-规格组-规格参数之间的关系

​          2.2 规格组和规格参数表结构

​         2.3 写出根据分类查询规格组 和 通用的查询规格参数的方法（根据规格组）



乐优商城的模块

1）后台商品管理（* - **）

2）商品搜索（***）

3）商品详情（**）

4）用户模块（用户注册，用户登录，鉴权）（**）

5）购物车（*）

6）订单管理（*）

7）微信支付（*）

















