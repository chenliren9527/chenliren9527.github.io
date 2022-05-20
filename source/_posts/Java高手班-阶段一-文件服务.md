---
title: 阶段一 第二章 文件服务
tags:
  - 文件服务
  - 文件处理
  - 分片上传
  - 上传附件
  - 打包下载
  - 分片合并
  - Java高手课
categories:
  - 阶段一 中台战略与组件化开发专题课程
abbrlink: 12374
date: 2021-12-27 23:08:41
---

# **文件服务**

## 1. 需求背景

文件的上传、下载功能是软件系统常见的功能，包括上传文件、下载文件、查看文件等。例如：电商系统中需要上传商品的图片、广告视频，办公系统中上传附件，社交类系统中上传用户头像等等。

文件上传下载大致流程为：

![1585711616132](文件服务.assets/1585711616132.png)  

这种方式开发起来简单、直接，但是有一些问题：

- 重复开发： 比如对接某个OSS(Object Storage Service,简称OSS)服务商， 每个应用都需要对接该服务商，重复工作
- 扩展性差： 当需要切换服务商时，所有涉及到的应用都需要修改、测试、上线



基于以上原因，微服务体系下的应用系统一般都有一个文件服务，用于统一管理文件上传下载等功能，大型电商系统甚至有独立的文件、图片、视频服务。此时架构体系变为：

![1585712078385](文件服务.assets/1585712078385.png) 

这种方式提供一个独立的文件微服务，该微服务向应用系统提供统一的上传、下载、查看接口，应用系统调用方式相同，并且屏蔽了底层对外调用OSS服务的接口，即使以后迁移OSS服务商，应用层面的系统也不需要变动。

这种模式也有一个小问题，比如我们调用了阿里云的OSS服务，如果所有的下载、查看功能都调用文件服务，那么文件服务的网络流量将会有非常大的压力。所以常用的做法是这样的：

![1585712407005](文件服务.assets/1585712407005.png) 

## 2. 核心功能

文件服务的核心功能是： **上传**和**下载**， 另一方面，除了这两个核心功能，还需要其他非功能性要求：

- 可用性：作为基础性服务，可用性要求非常高
- 配置性：OSS服务商配置、上传下载方式等内容
- 扩展性：能方便的进行扩展，如添加新的OSS服务商等

本课程的文件服务提供两种类型的服务：

​	1、面对应用系统的通用附件服务

​			提供统一的上传接口，屏蔽底层的存储方案（本地存储、FastDFS、阿里云存储、七牛云存储等），可独立运行服务

​	2、面对用户的网盘服务

​			有文件夹和文件的概念，支持大文件分片上传、合并

## 3. 存储策略

### 3.1 本地存储

本地存储，即将上传的文件存储在本地磁盘，并通过本地提供的Nginx服务来对外提供文件的下载和查看等功能。

### 3.2 FastDFS存储

FastDFS存储，即将上传的文件存储在FastDFS分布式文件存储系统中，并通过FastDFS结合Nginx提供的服务来对外提供文件的下载和查看等功能。

### 3.3 云存储

云存储，即将上传的文件存储在第三方云平台上，例如阿里云OSS、七牛云OSS服务等，并通过这些第三方提供的OSS服务来对外提供文件的下载和查看等功能。

## 4. 技术设计

本课程的文件服务以品达通用权限系统为脚手架，在此基础之上进行开发。为了能够提供统一的上传接口从而屏蔽底层的存储方案，需要进行相应的接口设计：

![image-20200424152528386](文件服务.assets/image-20200424152528386.png)



FileStrategy：文件策略顶层接口

AbstractFileStrategy：抽象文件策略处理类，实现FileStrategy接口。实现主要的文件上传处理流程，但是真正上传的过程需要其子类来完成。

LocalServiceImpl：具体的文件策略处理类，是AbstractFileStrategy的子类，负责将上传的文件保存在本地磁盘。

FastDfsServiceImpl：具体的文件策略处理类，是AbstractFileStrategy的子类，负责将上传的文件保存到FastDFS上。

AliServiceImpl：具体的文件策略处理类，是AbstractFileStrategy的子类，负责将上传的文件保存到阿里云OSS上。

`注意：本课程提供的存储策略有以上三种方式（即本地存储、FastDFS存储、阿里云OSS存储），后期也可以根据需要扩展其他的存储策略。这种设计方式其实就是策略模式的一个具体应用。`

## 5. 文件服务开发

### 5.1 环境搭建

#### 5.1.1 数据库环境搭建

第一步：创建pd_files数据库

~~~sql
create database pd_files character set utf8mb4;
~~~

第二步：在pd_files数据库中创建pd_attachment和pd_file数据表

~~~sql
CREATE TABLE `pd_attachment` (
  `id` bigint(20) NOT NULL COMMENT 'ID',
  `biz_id` varchar(64) DEFAULT NULL COMMENT '业务ID',
  `biz_type` varchar(255) DEFAULT NULL COMMENT '业务类型\n#AttachmentType',
  `data_type` varchar(255) DEFAULT 'IMAGE' COMMENT '数据类型\n#DataType{DIR:目录;IMAGE:图片;VIDEO:视频;AUDIO:音频;DOC:文档;OTHER:其他}',
  `submitted_file_name` varchar(255) DEFAULT '' COMMENT '原始文件名',
  `group_` varchar(255) DEFAULT '' COMMENT 'FastDFS返回的组\n用于FastDFS',
  `path` varchar(255) DEFAULT '' COMMENT 'FastDFS的远程文件名\n用于FastDFS',
  `relative_path` varchar(255) DEFAULT '' COMMENT '文件相对路径',
  `url` varchar(255) DEFAULT '' COMMENT '文件访问链接\n需要通过nginx配置路由，才能访问',
  `file_md5` varchar(255) DEFAULT NULL COMMENT '文件md5值',
  `context_type` varchar(255) DEFAULT '' COMMENT '文件上传类型\n取上传文件的值',
  `filename` varchar(255) DEFAULT '' COMMENT '唯一文件名',
  `ext` varchar(64) DEFAULT '' COMMENT '后缀\n (没有.)',
  `size` bigint(20) DEFAULT '0' COMMENT '大小',
  `org_id` bigint(20) DEFAULT NULL COMMENT '组织ID\n#c_core_org',
  `icon` varchar(64) DEFAULT '' COMMENT '图标',
  `create_month` varchar(10) DEFAULT NULL COMMENT '创建年月\n格式：yyyy-MM 用于统计',
  `create_week` varchar(10) DEFAULT NULL COMMENT '创建时处于当年的第几周\nyyyy-ww 用于统计',
  `create_day` varchar(12) DEFAULT NULL COMMENT '创建年月日\n格式： yyyy-MM-dd 用于统计',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `create_user` bigint(11) DEFAULT NULL COMMENT '创建人',
  `update_time` datetime DEFAULT NULL COMMENT '最后修改时间',
  `update_user` bigint(11) DEFAULT NULL COMMENT '最后修改人',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=COMPACT COMMENT='附件';


CREATE TABLE `pd_file` (
  `id` bigint(20) NOT NULL COMMENT '主键',
  `data_type` varchar(255) DEFAULT 'IMAGE' COMMENT '数据类型\n#DataType{DIR:目录;IMAGE:图片;VIDEO:视频;AUDIO:音频;DOC:文档;OTHER:其他}',
  `submitted_file_name` varchar(255) DEFAULT '' COMMENT '原始文件名',
  `tree_path` varchar(255) DEFAULT ',' COMMENT '父目录层级关系',
  `grade` int(11) DEFAULT '1' COMMENT '层级等级\n从1开始计算',
  `is_delete` bit(1) DEFAULT b'0' COMMENT '是否删除\n#BooleanStatus{TRUE:1,已删除;FALSE:0,未删除}',
  `folder_id` bigint(20) DEFAULT '0' COMMENT '父文件夹ID',
  `url` varchar(1000) DEFAULT '' COMMENT '文件访问链接\n需要通过nginx配置路由，才能访问',
  `size` bigint(20) DEFAULT '0' COMMENT '文件大小\n单位字节',
  `folder_name` varchar(255) DEFAULT '' COMMENT '父文件夹名称',
  `group_` varchar(255) DEFAULT '' COMMENT 'FastDFS组\n用于FastDFS',
  `path` varchar(255) DEFAULT '' COMMENT 'FastDFS远程文件名\n用于FastDFS',
  `relative_path` varchar(255) DEFAULT '' COMMENT '文件的相对路径 ',
  `file_md5` varchar(255) DEFAULT '' COMMENT 'md5值',
  `context_type` varchar(255) DEFAULT '' COMMENT '文件类型\n取上传文件的值',
  `filename` varchar(255) DEFAULT '' COMMENT '唯一文件名',
  `ext` varchar(64) DEFAULT '' COMMENT '文件名后缀 \n(没有.)',
  `icon` varchar(64) DEFAULT '' COMMENT '文件图标\n用于云盘显示',
  `create_month` varchar(10) DEFAULT NULL COMMENT '创建时年月\n格式：yyyy-MM 用于统计',
  `create_week` varchar(10) DEFAULT NULL COMMENT '创建时年周\nyyyy-ww 用于统计',
  `create_day` varchar(12) DEFAULT NULL COMMENT '创建时年月日\n格式： yyyy-MM-dd 用于统计',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `create_user` bigint(20) DEFAULT NULL COMMENT '创建人',
  `update_time` datetime DEFAULT NULL COMMENT '最后修改时间',
  `update_user` bigint(20) DEFAULT NULL COMMENT '最后修改人',
  `source` varchar(10) DEFAULT 'inner' COMMENT '文件来源：inner, outer',
  PRIMARY KEY (`id`) USING BTREE,
  FULLTEXT KEY `FU_TREE_PATH` (`tree_path`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=COMPACT COMMENT='文件表';
~~~

注：SQL脚本位置为 `文件服务/资料/数据库/pd_files.sql`



```
前面课程已经提到，本课程的文件服务提供两种类型的服务：
1、面对应用系统的通用附件服务
	提供统一的上传接口，屏蔽底层的存储方案（本地存储、FastDFS、阿里云存储、七牛云存储等），可独立运行服务。上传的文件相关信息保存在pd_attachment表中。
2、面对用户的网盘服务
	提供统一的上传接口，屏蔽底层的存储方案，有文件夹和文件的概念，支持大文件分片上传、断点续传。上传的文件相关信息保存在pd_file表中。
```

pd_attachment表为附件表，具体表结构如下：

![image-20200430104238608](文件服务.assets/image-20200430104238608.png)



pd_file表为文件/文件夹信息表，具体表结构如下：

![image-20200430114544818](文件服务.assets/image-20200430114544818.png)

#### 5.1.2 Nacos环境搭建

第一步：安装Nacos并进行配置(略)

第二步：在Nacos中创建命名空间file-server

![image-20200429165919785](文件服务.assets/image-20200429165919785.png)

![image-20200429170033028](文件服务.assets/image-20200429170033028.png)

第三步：在Nacos中将配置文件导入到file-server命名空间

![image-20200429170157579](文件服务.assets/image-20200429170157579.png)

注：yml配置文件位置为 `文件服务/资料/Nacos/nacos_config_export_2020-04-30 11_47_56.zip`

#### 5.1.3 Nginx环境搭建

当文件存储策略为本地存储或者FastDFS存储时，需要使用Nginx服务来对外提供文件的下载和查看等功能。

第一步：安装Nginx(略)

第二步：配置Nginx_HOME/conf/nginx.conf

注：可参照 `文件服务/资料/Nginx/nginx.conf` 进行配置

![image-20200429173715882](文件服务.assets/image-20200429173715882.png)

#### 5.1.4 maven工程环境搭建

说明：本项目的开发并不是从零开始搭建开发环境，而是在一个名为品达通用权限系统的基础上进行文件服务的开发。`品达通用权限系统是传智播客提供的一个开发平台（脚手架），其中提供了一系列的基础组件并且已经实现了权限管理、认证、鉴权、JWT解析等功能。用户可以在此平台基础上开发自己的业务功能。`

品达通用权限系统位置: `文件服务/资料/初始工程（脚手架）/pinda-authority`



具体搭建过程：

第一步：导入品达通用权限系统到IDEA并修改pd-apps模块中的pom.xml文件

~~~xml
<!-- 开发环境 -->
<profile>
    <id>dev</id>
    <activation>
        <!--默认激活配置-->
        <activeByDefault>true</activeByDefault>
    </activation>
    <properties>
        <!--当前环境-->
        <pom.profile.name>dev</pom.profile.name>
        <!--Nacos配置中心地址-->
        <pom.nacos.ip>127.0.0.1</pom.nacos.ip>
        <pom.nacos.port>8848</pom.nacos.port>
        <!--Nacos配置中心命名空间ID-->
        <pom.nacos.namespace>
            53c655b3-babd-4402-82f1-33b7e4c90111
        </pom.nacos.namespace>
    </properties>
</profile>
~~~

第二步：在品达通用权限系统平台基础之上创建文件服务对应的maven工程pd-file

![image-20200424171904437](文件服务.assets/image-20200424171904437.png)

第三步：在pd-file下创建pd-file-entity子模块并配置pom.xml文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>pd-file</artifactId>
        <groupId>com.itheima</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>pd-file-entity</artifactId>
    <description>文件服务实体模块</description>
    <dependencies>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-tools-common</artifactId>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-core</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus</artifactId>
        </dependency>
    </dependencies>
</project>
~~~

第四步：将文件服务相关的实体类、DTO、domain等导入pd-file-entity中

![image-20200424172324728](文件服务.assets/image-20200424172324728.png)

注：文件服务相关实体类位置 `文件服务/资料/文件服务相关实体类`

第五步：在pd-file下创建pd-file-server子模块并配置pom.xml文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>pd-file</artifactId>
        <groupId>com.itheima</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>pd-file-server</artifactId>
    <description>文件服务启动模块</description>
    <dependencies>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-auth-api</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>com.google.guava</groupId>
                    <artifactId>guava</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-tools-log</artifactId>
        </dependency>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-tools-swagger2</artifactId>
        </dependency>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-tools-validator</artifactId>
        </dependency>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-tools-xss</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>fastjson</artifactId>
                    <groupId>com.alibaba</groupId>
                </exclusion>
                <exclusion>
                    <groupId>com.google.guava</groupId>
                    <artifactId>guava</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>com.google.guava</groupId>
                    <artifactId>guava</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.ow2.asm</groupId>
            <artifactId>asm</artifactId>
            <version>${asm.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.tomcat.embed</groupId>
                    <artifactId>tomcat-embed-websocket</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-undertow</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-json</artifactId>
        </dependency>
        <!-- 测试 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-file-entity</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-tools-databases</artifactId>
        </dependency>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>pd-tools-dozer</artifactId>
        </dependency>
        <!-- @RefreshScope 需要使用 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-context</artifactId>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        <!-- 阿里云oss -->
        <dependency>
            <groupId>com.aliyun.oss</groupId>
            <artifactId>aliyun-sdk-oss</artifactId>
        </dependency>
        <!-- 七牛oss -->
        <dependency>
            <groupId>com.qiniu</groupId>
            <artifactId>qiniu-java-sdk</artifactId>
        </dependency>
        <!--  FastDFS -->
        <dependency>
            <groupId>com.github.tobato</groupId>
            <artifactId>fastdfs-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
            <scope>compile</scope>
        </dependency>
    </dependencies>
    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
~~~

第六步：导入配置文件到pd-file-server中

![image-20200424172708976](文件服务.assets/image-20200424172708976.png)

注：文件服务相关配置文件位置: `文件服务/资料/文件服务相关配置文件`

第七步：在pd-file-server中创建启动类

~~~java
package com.itheima.pinda;

import com.itheima.pinda.validator.config.EnableFormValidator;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.hystrix.EnableHystrix;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.env.Environment;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import java.net.InetAddress;
import java.net.UnknownHostException;

@SpringBootApplication
@EnableDiscoveryClient
@EnableHystrix
@EnableFeignClients(value = {
        "com.itheima.pinda",
})
@EnableTransactionManagement
@Slf4j
@EnableFormValidator
public class FileServerApplication {
    public static void main(String[] args) throws UnknownHostException {
        ConfigurableApplicationContext application = SpringApplication.run(FileServerApplication.class, args);
        Environment env = application.getEnvironment();
        log.info("\n----------------------------------------------------------\n\t" +
                        "应用 '{}' 运行成功! 访问连接:\n\t" +
                        "Swagger文档: \t\thttp://{}:{}/doc.html\n\t" +
                        "数据库监控: \t\thttp://{}:{}/druid\n" +
                        "----------------------------------------------------------",
                env.getProperty("spring.application.name"),
                InetAddress.getLocalHost().getHostAddress(),
                env.getProperty("server.port"),
                "127.0.0.1",
                env.getProperty("server.port"));

    }
}
~~~

第八步：将配置类导入pd-file-server中

![image-20200424181720459](文件服务.assets/image-20200424181720459.png)

注：文件服务相关配置类位置: `文件服务/资料/文件服务配置类`

第九步：在pd-file-server中导入工具类

![image-20200424181910324](文件服务.assets/image-20200424181910324.png)

注：文件服务相关工具类位置: `文件服务/资料/文件服务工具类`

第十步：在pd-file-server中导入配置属性类

![image-20200424182009395](文件服务.assets/image-20200424182009395.png)

注：文件服务相关配置属性类位置: `文件服务/资料/文件服务配置属性类`

第十一步：在pd-file-server中导入Mapper接口

![image-20200501001642286](文件服务.assets/image-20200501001642286.png)

注：文件服务相关Mapper接口位置: 文件服务/资料/文件服务相关Mapper接口

### 5.2 文件处理策略

由于我们当前的文件服务需要给客户端提供统一的服务接口，这就需要文件服务屏蔽底层的具体文件存储方式，所以对文件处理策略进行如下类和接口的设计：

![image-20200424152528386](文件服务.assets/image-20200424152528386.png)

#### 5.2.1 FileStrategy

FileStrategy是文件处理策略顶层接口，是对文件处理的顶层抽象，具体代码如下：

~~~java
package com.itheima.pinda.file.strategy;

import com.itheima.pinda.file.domain.FileDeleteDO;
import com.itheima.pinda.file.entity.File;
import org.springframework.web.multipart.MultipartFile;
import java.util.List;
/**
 * 文件策略接口
 */
public interface FileStrategy {
    /**
     * 文件上传
     *
     * @param file 文件
     * @return 文件对象
     */
    File upload(MultipartFile file);

    /**
     * 删除文件
     *
     * @param list 列表
     */
    boolean delete(List<FileDeleteDO> list);
}
~~~

#### 5.2.2 AbstractFileStrategy

AbstractFileStrategy是抽象文件策略处理类，实现了FileStrategy接口。

AbstractFileStrategy实现主要的文件上传、删除的处理流程，例如异常情况的判断，文件对象的封装等，但是真正上传的处理过程需要其子类来完成，因为不同的存储方案处理方式是不同的。

~~~java
package com.itheima.pinda.file.strategy.impl;

import com.itheima.pinda.exception.BizException;
import com.itheima.pinda.file.domain.FileDeleteDO;
import com.itheima.pinda.file.entity.File;
import com.itheima.pinda.file.enumeration.IconType;
import com.itheima.pinda.file.properties.FileServerProperties;
import com.itheima.pinda.file.strategy.FileStrategy;
import com.itheima.pinda.file.utils.FileDataTypeUtil;
import com.itheima.pinda.utils.DateUtils;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.io.FilenameUtils;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.multipart.MultipartFile;
import java.time.LocalDateTime;
import java.util.List;
import static com.itheima.pinda.exception.code.ExceptionCode.BASE_VALID_PARAM;

/**
 * 文件抽象策略处理类
 */
@Slf4j
public abstract class AbstractFileStrategy implements FileStrategy {
    private static final String FILE_SPLIT = ".";
    @Autowired
    protected FileServerProperties fileProperties;

    protected FileServerProperties.Properties properties;

    /**
     * 上传文件
     *
     * @param multipartFile
     * @return
     */
    @Override
    public File upload(MultipartFile multipartFile) {
        try {
            if (!multipartFile.getOriginalFilename().contains(FILE_SPLIT)) {
                throw BizException.wrap(BASE_VALID_PARAM.build("缺少后缀名"));
            }

            File file = File.builder()
                    .isDelete(false).
                submittedFileName(multipartFile.getOriginalFilename())
                    .contextType(multipartFile.getContentType())
                    .dataType(FileDataTypeUtil.getDataType(multipartFile.getContentType()))
                    .size(multipartFile.getSize())
                    .ext(FilenameUtils.getExtension(multipartFile.getOriginalFilename()))
                    .build();
            file.setIcon(IconType.getIcon(file.getExt()).getIcon());
            setDate(file);
            uploadFile(file, multipartFile);
            return file;
        } catch (Exception e) {
            log.error("e={}", e);
            throw BizException.wrap(BASE_VALID_PARAM.build("文件上传失败"));
        }
    }

    /**
     * 获取下载地址前缀
     */
    protected String getUriPrefix() {
        if (StringUtils.isNotEmpty(properties.getUriPrefix())) {
            return properties.getUriPrefix();
        } else {
            return properties.getEndpoint();
        }
    }

    /**
     * 具体类型执行上传操作
     *
     * @param file
     * @param multipartFile
     * @throws Exception
     */
    protected abstract void uploadFile(File file, MultipartFile multipartFile) throws Exception;

    private void setDate(File file) {
        LocalDateTime now = LocalDateTime.now();
        file.setCreateMonth(DateUtils.formatAsYearMonthEn(now))
                .setCreateWeek(DateUtils.formatAsYearWeekEn(now))
                .setCreateDay(DateUtils.formatAsDateEn(now));
    }

    @Override
    public boolean delete(List<FileDeleteDO> list) {
        if (list.isEmpty()) {
            return true;
        }
        boolean flag = false;
        for (FileDeleteDO file : list) {
            try {
                delete(file);
                flag = true;
            } catch (Exception e) {
                log.error("删除文件失败", e);
            }
        }
        return flag;
    }

    /**
     * 具体执行删除方法， 无需处理异常
     */
    protected abstract void delete(FileDeleteDO file);

}
~~~

#### 5.2.3 LocalServiceImpl

LocalServiceImpl是AbstractFileStrategy的子类，负责处理存储策略为本地时的文件上传和删除操作。为了使程序能够动态选择具体的策略处理类，可以提供一个配置类，在配置类中定义LocalServiceImpl，具体代码如下：

~~~java
package com.itheima.pinda.file.storage;

import cn.hutool.core.util.StrUtil;
import com.itheima.pinda.file.domain.FileDeleteDO;
import com.itheima.pinda.file.entity.File;
import com.itheima.pinda.file.properties.FileServerProperties;
import com.itheima.pinda.file.strategy.impl.AbstractFileStrategy;
import com.itheima.pinda.utils.StrPool;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import java.nio.file.Paths;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.util.UUID;
import static com.itheima.pinda.utils.DateUtils.DEFAULT_MONTH_FORMAT_SLASH;
/**
 * 本地上传配置
 */
@EnableConfigurationProperties(FileServerProperties.class)
@Configuration
@ConditionalOnProperty(name = "pinda.file.type", havingValue = "LOCAL")
@Slf4j
public class LocalAutoConfigure {
    /**
     * 本地文件策略处理类
     */
    @Service
    public class LocalServiceImpl extends AbstractFileStrategy {
        private void buildClient() {
            properties = fileProperties.getLocal();
        }

        /**
         * 上传文件
         * @param file
         * @param multipartFile
         * @throws Exception
         */
        @Override
        protected void uploadFile(File file, MultipartFile multipartFile) throws Exception {
            buildClient();
            //生成文件名
            String fileName = UUID.randomUUID().toString() + StrPool.DOT + file.getExt();
            //日期文件夹，例如：2020\04
            String relativePath = Paths.get(LocalDate.now().format(DateTimeFormatter.ofPattern(DEFAULT_MONTH_FORMAT_SLASH))).toString();
            // web服务器存放的绝对路径，例如：D:\\uploadFiles\\oss-file-service\\2020\\04
            String absolutePath = Paths.get(properties.getEndpoint(), properties.getBucketName(), relativePath).toString();
            //目标输出文件
            java.io.File outFile = new java.io.File(Paths.get(absolutePath, fileName).toString());
            //向目标文件写入数据
            org.apache.commons.io.FileUtils.writeByteArrayToFile(outFile, multipartFile.getBytes());

            String url = new StringBuilder(getUriPrefix())
                    .append(StrPool.SLASH)
                    .append(properties.getBucketName())
                    .append(StrPool.SLASH)
                    .append(relativePath)
                    .append(StrPool.SLASH)
                    .append(fileName)
                    .toString();
            //替换掉windows环境的\路径
            url = StrUtil.replace(url, "\\\\", StrPool.SLASH);
            url = StrUtil.replace(url, "\\", StrPool.SLASH);
            file.setUrl(url);
            file.setFilename(fileName);
            file.setRelativePath(relativePath);
        }

        /**
         * 文件删除
         * @param file
         */
        @Override
        protected void delete(FileDeleteDO file) {
            java.io.File ioFile = 
                new java.io.File(Paths.get(properties.getEndpoint(), 
                                           properties.getBucketName(), 
                                           file.getRelativePath(), 
                                           file.getFileName()).toString());
            
            org.apache.commons.io.FileUtils.deleteQuietly(ioFile);
        }
    }
}
~~~

通过上面的代码可以看到，在进行文件上传和文件删除时都会使用到配置文件中的配置项，关于本地文件处理策略的配置如下：

~~~yaml
pinda:
  mysql:
    database: pd_files
  nginx:
    ip: ${spring.cloud.client.ip-address} #正式环境要将该ip设置成nginx对应的公网ip
    port: 10000   #正式环境需要将该ip设置成nginx对应的公网端口
  file:
    type: LOCAL
    local:
      uriPrefix: http://${pinda.nginx.ip}:${pinda.nginx.port}
      bucket-name: oss-file-service
      endpoint: D:\uploadFiles
~~~

#### 5.2.4 FastDfsServiceImpl

FastDfsServiceImpl是AbstractFileStrategy的子类，负责处理存储策略为FastDFS时的文件上传和删除操作。为了使程序能够动态选择具体的策略处理类，可以提供一个配置类，在配置类中定义FastDfsServiceImpl，具体代码如下：

~~~java
package com.itheima.pinda.file.storage;

import com.github.tobato.fastdfs.domain.fdfs.StorePath;
import com.github.tobato.fastdfs.service.FastFileStorageClient;
import com.itheima.pinda.file.domain.FileDeleteDO;
import com.itheima.pinda.file.entity.File;
import com.itheima.pinda.file.properties.FileServerProperties;
import com.itheima.pinda.file.strategy.impl.AbstractFileStrategy;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

/**
 * FastDFS配置
 */
@EnableConfigurationProperties(FileServerProperties.class)
@Configuration
@Slf4j
@ConditionalOnProperty(name = "pinda.file.type", havingValue = "FAST_DFS")
public class FastDfsAutoConfigure {
    /**
     * FastDFS文件策略处理类
     */
    @Service
    public class FastDfsServiceImpl extends AbstractFileStrategy {
        @Autowired
        private FastFileStorageClient storageClient; //操作FastDFS的客户端

        /**
         * 上传文件
         * @param file
         * @param multipartFile
         * @throws Exception
         */
        @Override
        protected void uploadFile(File file, MultipartFile multipartFile) 
            throws Exception {
            //调用FastDFS客户端将文件上传到FastDFS
            StorePath storePath = 
                storageClient.uploadFile(multipartFile.getInputStream(), 
                                         multipartFile.getSize(), 
                                         file.getExt(), 
                                         null);
            
            file.setUrl(fileProperties.getUriPrefix() + 
                        storePath.getFullPath());
            file.setGroup(storePath.getGroup());
            file.setPath(storePath.getPath());
        }

        /**
         * 文件删除
         * @param file
         */
        @Override
        protected void delete(FileDeleteDO file) {
            //调用FastDFS客户端删除文件
            storageClient.deleteFile(file.getGroup(), file.getPath());
        }
    }
}
~~~

通过上面代码可以看到要使用FastDFS提供的客户端FastFileStorageClient来实现文件的上传和删除，这就需要在文件服务对应的配置文件中进行如下配置：

~~~yaml
pinda:
  mysql:
    database: pd_files
  nginx:
    ip: ${spring.cloud.client.ip-address} #正式环境要将该ip设置成nginx对应的公网ip
    port: 10000  #正式环境需要将该ip设置成nginx对应的公网端口
  file:
    type: FAST_DFS  
    uriPrefix: http://172.17.0.115:8188/ #存储类型为FAST_DFS时使用
#FAST_DFS配置
fdfs:
  soTimeout: 1500
  connectTimeout: 600
  thumb-image:
    width: 150
    height: 150
  tracker-list:
    - 172.17.0.115:22122
  pool:
    #从池中借出的对象的最大数目
    max-total: 153
    max-wait-millis: 102
    jmx-name-base: 1
    jmx-name-prefix: 1
~~~

#### 5.2.5 AliServiceImpl

AliServiceImpl是AbstractFileStrategy的子类，负责处理存储策略为阿里云OSS时的文件上传和删除操作。为了使程序能够动态选择具体的策略处理类，可以提供一个配置类，在配置类中定义AliServiceImpl，可以参照阿里云OSS官方提供的示例代码(https://help.aliyun.com/document_detail/32011.html?spm=a2c4g.11186623.6.769.652763282djHGw)进行文件的上传和删除。

具体代码如下：

~~~java
package com.itheima.pinda.file.storage;

import cn.hutool.core.util.StrUtil;
import com.alibaba.fastjson.JSONObject;
import com.aliyun.oss.OSS;
import com.aliyun.oss.OSSClientBuilder;
import com.aliyun.oss.model.*;
import com.itheima.pinda.file.domain.FileDeleteDO;
import com.itheima.pinda.file.entity.File;
import com.itheima.pinda.file.properties.FileServerProperties;
import com.itheima.pinda.file.strategy.impl.AbstractFileStrategy;
import com.itheima.pinda.utils.StrPool;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import java.nio.file.Paths;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;
import java.util.UUID;
import static com.itheima.pinda.utils.DateUtils.DEFAULT_MONTH_FORMAT_SLASH;

/**
 * 阿里云OSS配置
 */
@EnableConfigurationProperties(FileServerProperties.class)
@Configuration
@Slf4j
@ConditionalOnProperty(name = "pinda.file.type", havingValue = "ALI")
public class AliOssAutoConfigure {
    /**
     * 阿里云OSS文件策略处理类
     */
    @Service
    public class AliServiceImpl extends AbstractFileStrategy {
        /**
         * 构建阿里云OSS客户端
         * @return
         */
        private OSS buildClient() {
            properties = fileProperties.getAli();
            return new OSSClientBuilder().
                                    build(properties.getEndpoint(),
                                            properties.getAccessKeyId(),
                                            properties.getAccessKeySecret());
        }

        protected String getUriPrefix() {
            if (StringUtils.isNotEmpty(properties.getUriPrefix())) {
                return properties.getUriPrefix();
            } else {
                String prefix = properties.
                    			getEndpoint().
                    			contains("https://") ? "https://" : "http://";
                return prefix + properties.getBucketName() + "." + 
                    properties.getEndpoint().replaceFirst(prefix, "");
            }
        }

        /**
         * 上传文件
         * @param file
         * @param multipartFile
         * @throws Exception
         */
        @Override
        protected void uploadFile(File file, MultipartFile multipartFile) 
            throws Exception {
            OSS client = buildClient();
            //获得OSS空间名称
            String bucketName = properties.getBucketName();
            if (!client.doesBucketExist(bucketName)) {
                //创建存储空间
                client.createBucket(bucketName);
            }

            //生成文件名
            String fileName = UUID.randomUUID().toString() + 
                				StrPool.DOT + 
                				file.getExt();
            //日期文件夹，例如：2020\04
            String relativePath = 
                Paths.get(LocalDate.now().
                          format(DateTimeFormatter.
                 ofPattern(DEFAULT_MONTH_FORMAT_SLASH))).
                toString();
            // web服务器存放的相对路径
            String relativeFileName = relativePath + StrPool.SLASH + fileName;
            relativeFileName = StrUtil.replace(relativeFileName, "\\\\", 
                                               StrPool.SLASH);
            relativeFileName = StrUtil.replace(relativeFileName, "\\", 
                                               StrPool.SLASH);
            //对象元数据
            ObjectMetadata metadata = new ObjectMetadata();
            metadata.setContentDisposition("attachment;fileName=" + 
                                           file.getSubmittedFileName());
            metadata.setContentType(file.getContextType());

            //上传请求对象
            PutObjectRequest request = 
                new PutObjectRequest(bucketName, relativeFileName, 
                                     multipartFile.getInputStream(), 
                                     metadata);
            //上传文件到阿里云OSS空间
            PutObjectResult result = client.putObject(request);

            log.info("result={}", JSONObject.toJSONString(result));

            String url = getUriPrefix() + StrPool.SLASH + relativeFileName;
            url = StrUtil.replace(url, "\\\\", StrPool.SLASH);
            url = StrUtil.replace(url, "\\", StrPool.SLASH);
            // 写入文件表
            file.setUrl(url);
            file.setFilename(fileName);
            file.setRelativePath(relativePath);

            file.setGroup(result.getETag());
            file.setPath(result.getRequestId());

            //关闭阿里云OSS客户端
            client.shutdown();
        }

        /**
         * 文件删除
         * @param file
         */
        @Override
        protected void delete(FileDeleteDO file) {
            OSS client = buildClient();
            //获得OSS空间名称
            String bucketName = properties.getBucketName();
            // 删除文件
            client.deleteObject(bucketName, file.getRelativePath() + 
                                StrPool.SLASH + file.getFileName());
            //关闭阿里云OSS客户端
            client.shutdown();
        }
    }
}
~~~

通过上面代码可以看到要使用阿里云OSS提供的客户端OSS来实现文件的上传和删除，这就需要在文件服务对应的配置文件中进行如下配置：

~~~yaml
pinda:
  mysql:
    database: pd_files
  nginx:
    ip: ${spring.cloud.client.ip-address} #正式环境要将该ip设置成nginx对应的公网ip
    port: 10000  #正式环境需要将该ip设置成nginx对应的公网端口
  file:
    type: ALI
    ali:
      # 请填写自己的阿里云存储配置
      bucket-name: bladex-loan
      endpoint: http://oss-cn-qingdao.aliyuncs.com
      access-key-id: LTAI4FhtimFAiz6iLGJSiJui  
      access-key-secret: SsU15qaPwpF1x5xMqwc0XzGuY92fnc
~~~

### 5.3 接口开发-上传附件

#### 5.3.1 接口文档

上传附件接口要完成的操作主要有两个：

- 将客户端提交的文件上传到指定存储位置（具体存储位置由配置文件配置的存储策略确定）
- 将上传的文件信息保存到数据库的pd_attachment表中

接口文档如下：

![image-20200427211221746](文件服务.assets/image-20200427211221746.png)

![image-20200428121728825](文件服务.assets/image-20200428121728825.png)

#### 5.3.2 代码实现

第一步：创建AttachmentController并提供文件上传方法

~~~java
package com.itheima.pinda.file.controller;

import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.core.toolkit.Wrappers;
import com.baomidou.mybatisplus.core.toolkit.support.SFunction;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.itheima.pinda.base.BaseController;
import com.itheima.pinda.base.R;
import com.itheima.pinda.file.dto.AttachmentDTO;
import com.itheima.pinda.file.dto.AttachmentRemoveDTO;
import com.itheima.pinda.file.dto.AttachmentResultDTO;
import com.itheima.pinda.file.dto.FilePageReqDTO;
import com.itheima.pinda.file.entity.Attachment;
import com.itheima.pinda.file.service.AttachmentService;
import com.itheima.pinda.utils.BizAssert;
import io.swagger.annotations.*;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.collections.MapUtils;
import org.apache.commons.lang3.ArrayUtils;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.List;
import java.util.Map;
import static com.itheima.pinda.exception.code.ExceptionCode.BASE_VALID_PARAM;
import static java.util.stream.Collectors.groupingBy;
/**
 * 附件处理控制器
 */
@RestController
@RequestMapping("/attachment")
@Slf4j
@Api(value = "附件", tags = "附件")
public class AttachmentController extends BaseController {
    @Autowired
    private AttachmentService attachmentService;

    /**
     * 上传附件
     */
    @ApiOperation(value = "附件上传", notes = "附件上传")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "isSingle", value = "是否单文件", dataType = "boolean", paramType = "query"),
            @ApiImplicitParam(name = "id", value = "文件id", dataType = "long", paramType = "query"),
            @ApiImplicitParam(name = "bizId", value = "业务id", dataType = "long", paramType = "query"),
            @ApiImplicitParam(name = "bizType", value = "业务类型", dataType = "long", paramType = "query"),
            @ApiImplicitParam(name = "file", value = "附件", dataType = "MultipartFile", allowMultiple = true, required = true),
    })
    @PostMapping(value = "/upload")
    public R<AttachmentDTO> upload(
            @RequestParam(value = "file") MultipartFile file,
            @RequestParam(value = "isSingle", required = false, defaultValue = "false") Boolean isSingle,
            @RequestParam(value = "id", required = false) Long id,
            @RequestParam(value = "bizId", required = false) String bizId,
            @RequestParam(value = "bizType") String bizType) {
        
        // 忽略路径字段,只处理文件类型
        if (file.isEmpty()) {
            return fail(BASE_VALID_PARAM.build("请求中必须至少包含一个有效文件"));
        }
        AttachmentDTO attachment = attachmentService.upload(file, 
                                                            id, 
                                                            bizType, 
                                                            bizId, 
                                                            isSingle);

        return success(attachment);
    }
}
~~~

第二步：创建AttachmentService接口

~~~java
package com.itheima.pinda.file.service;

import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.baomidou.mybatisplus.extension.service.IService;
import com.itheima.pinda.file.dto.AttachmentDTO;
import com.itheima.pinda.file.dto.AttachmentResultDTO;
import com.itheima.pinda.file.dto.FilePageReqDTO;
import com.itheima.pinda.file.entity.Attachment;
import org.springframework.web.multipart.MultipartFile;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.List;

/**
 * 附件业务接口
 */
public interface AttachmentService extends IService<Attachment> {
    /**
     * 上传附件
     *
     * @param file 文件
     * @param id 附件id
     * @param bizType 业务类型
     * @param bizId 业务id
     * @param isSingle 是否单个文件
     * @return
     */
    AttachmentDTO upload(MultipartFile file, Long id, String bizType, 
                         String bizId, Boolean isSingle);

}
~~~

第三步：创建AttachmentServiceImpl实现类

~~~java
package com.itheima.pinda.file.service.impl;

import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.core.toolkit.Wrappers;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.itheima.pinda.base.id.IdGenerate;
import com.itheima.pinda.database.mybatis.conditions.Wraps;
import com.itheima.pinda.database.mybatis.conditions.query.LbqWrapper;
import com.itheima.pinda.dozer.DozerUtils;
import com.itheima.pinda.exception.BizException;
import com.itheima.pinda.file.dao.AttachmentMapper;
import com.itheima.pinda.file.domain.FileDO;
import com.itheima.pinda.file.domain.FileDeleteDO;
import com.itheima.pinda.file.dto.AttachmentDTO;
import com.itheima.pinda.file.dto.AttachmentResultDTO;
import com.itheima.pinda.file.dto.FilePageReqDTO;
import com.itheima.pinda.file.entity.Attachment;
import com.itheima.pinda.file.entity.File;
import com.itheima.pinda.file.enumeration.DataType;
import com.itheima.pinda.file.properties.FileServerProperties;
import com.itheima.pinda.file.service.AttachmentService;
import com.itheima.pinda.file.strategy.FileStrategy;
import com.itheima.pinda.utils.DateUtils;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.ArrayUtils;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.time.LocalDateTime;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

/**
 * 附件业务实现类
 */
@Slf4j
@Service
public class AttachmentServiceImpl extends ServiceImpl<AttachmentMapper, Attachment> implements AttachmentService {
    @Autowired
    private IdGenerate<Long> idGenerate;
    @Autowired
    private DozerUtils dozer;
    @Autowired
    private FileStrategy fileStrategy;
    @Autowired
    private FileServerProperties fileProperties;

    /**
     * 上传附件
     *
     * @param multipartFile 文件
     * @param id 附件id
     * @param bizType 业务类型
     * @param bizId 业务id
     * @param isSingle 是否单个文件
     * @return
     */
    @Override
    public AttachmentDTO upload(MultipartFile multipartFile, Long id, String bizType, String bizId, Boolean isSingle) {
        //根据业务类型来判断是否生成业务id
        if (StringUtils.isNotEmpty(bizType) && StringUtils.isEmpty(bizId)) {
            bizId = String.valueOf(idGenerate.generate());
        }
        File file = fileStrategy.upload(multipartFile);

        Attachment attachment = dozer.map(file, Attachment.class);

        attachment.setBizId(bizId);
        attachment.setBizType(bizType);
        setDate(attachment);

        if (isSingle) {
            super.remove(Wraps.<Attachment>lbQ().eq(Attachment::getBizId, bizId).eq(Attachment::getBizType, bizType));
        }

        if (id != null && id > 0) {
            //当前端传递了文件id时，修改这条记录
            attachment.setId(id);
            super.updateById(attachment);
        } else {
            attachment.setId(idGenerate.generate());
            super.save(attachment);
        }

        AttachmentDTO dto = dozer.map(attachment, AttachmentDTO.class);
        return dto;
    }

    private void setDate(Attachment file) {
        LocalDateTime now = LocalDateTime.now();
        file.setCreateMonth(DateUtils.formatAsYearMonthEn(now));
        file.setCreateWeek(DateUtils.formatAsYearWeekEn(now));
        file.setCreateDay(DateUtils.formatAsDateEn(now));
    }
}
~~~

#### 5.3.3 接口测试

第一步：启动Nacos配置中心，pd-file-server.yml配置内容如下：

~~~yaml
pinda:
  mysql:
    database: pd_files
  nginx:
    ip: ${spring.cloud.client.ip-address} #正式环境要将该ip设置成nginx对应的公网ip
    port: 10000  #正式环境需要将该ip设置成nginx对应的公网端口
  swagger:
    enabled: true
    docket:
      file:
        title: 文件服务
        base-package: com.itheima.pinda.file.controller

  file:
    type: LOCAL # LOCAL ALI MINIO FAST_DFS  
    local:
      uriPrefix: http://${pinda.nginx.ip}:${pinda.nginx.port}
      bucket-name: oss-file-service
      endpoint: D:\uploadFiles
    ali:
      # 请填写自己的阿里云存储配置
      bucket-name: bladex-loan
      endpoint: http://oss-cn-qingdao.aliyuncs.com
      access-key-id: LTAI4FhtimFAiz6iLGJSiJui  
      access-key-secret: SsU15qaPwpF1x5xMqwc0XzGuY92fnc

#FAST_DFS配置
fdfs:
  soTimeout: 1500
  connectTimeout: 600
  thumb-image:
    width: 150
    height: 150
  tracker-list:
    - 111.231.76.210:22122
  pool:
    #从池中借出的对象的最大数目
    max-total: 153
    max-wait-millis: 102
    jmx-name-base: 1
    jmx-name-prefix: 1


server:
  port: 8765
~~~

第二步：启动Nginx服务

第三步：启动文件服务

第四步：访问接口文档，地址为http://localhost:8765/doc.html

![image-20200428122508460](文件服务.assets/image-20200428122508460.png)

![image-20200428122607756](文件服务.assets/image-20200428122607756.png)

可以看到上传的文件已经保存到了本地磁盘：

![image-20200428122743952](文件服务.assets/image-20200428122743952.png)

同时上传的文件信息也已经保存到了pd_attachment表中：

![image-20200428122912010](文件服务.assets/image-20200428122912010.png)

可以通过Nginx提供的Http服务来访问上传的文件：

http://192.168.3.19:10000/oss-file-service/2020/04/527e00c7-245c-4848-ac2c-5245b4823e3a.zip

注：可以修改Nacos中的pd-file-server.yml配置文件，将存储策略改为ALI和FAST_DFS来测试文件的存储策略是否发生了变化。

### 5.4 接口开发-根据id删除附件

#### 5.4.1 接口文档

根据id删除附件接口要完成的操作主要有两个：

- 将客户端上传的文件从指定存储位置（具体存储位置由配置文件配置的存储策略确定）删除
- 将文件信息从数据库的pd_attachment表中删除

根据id删除附件功能的接口文档如下：

![image-20200428145846716](文件服务.assets/image-20200428145846716.png)

#### 5.4.2 代码实现

第一步：在AttachmentController中提供文件删除的方法

~~~java
@ApiOperation(value = "删除文件", notes = "删除文件")
@ApiImplicitParams({
    @ApiImplicitParam(name = "ids[]", value = "文件ids", dataType = "array", paramType = "query"),
})
@DeleteMapping
public R<Boolean> remove(@RequestParam(value = "ids[]") Long[] ids) {
    attachmentService.remove(ids);
    return success(true);
}
~~~

第二步：在AttachmentService接口中扩展remove方法

~~~java
/**
* 删除附件
*
* @param ids
*/
void remove(Long[] ids);
~~~

第三步：在AttachmentServiceImpl实现类中实现remove方法

~~~java
/**
*根据id删除附件
* @param ids
*/
@Override
public void remove(Long[] ids) {
    if (ArrayUtils.isEmpty(ids)) {
        return;
    }
    //查询数据库
    List<Attachment> list = super.list(Wrappers.<Attachment>lambdaQuery().
                                       in(Attachment::getId, ids));
    if (list.isEmpty()) {
        return;
    }
    //删除数据库中的记录
    super.removeByIds(Arrays.asList(ids));

    //对象格式处理
    List<FileDeleteDO> fileDeleteDOList =
        list.stream().map((fi) -> FileDeleteDO.builder()
                          .relativePath(fi.getRelativePath()) //文件在服务器的相对路径
                          .fileName(fi.getFilename()) //唯一文件名
                          .group(fi.getGroup()) //fastDFS返回的组 用于FastDFS
                          .path(fi.getPath()) //fastdfs 的路径
                          .build())
        .collect(Collectors.toList());
    //删除文件
    fileStrategy.delete(fileDeleteDOList);
}
~~~

#### 5.4.3 接口测试

第一步：启动Nacos配置中心

第二步：启动Nginx服务

第三步：启动文件服务

第四步：访问接口文档，地址为http://localhost:8765/doc.html

![image-20200428154444350](文件服务.assets/image-20200428154444350.png)

可以看到pd_attachment表中对应的记录已经删除掉了，对应的文件也已经被删除掉了。

### 5.5 接口开发-根据业务类型/业务id删除附件

#### 5.5.1 接口文档

根据业务类型/业务id删除附件接口要完成的操作主要有两个：

- 将客户端上传的文件从指定存储位置（具体存储位置由配置文件配置的存储策略确定）删除
- 将文件信息从数据库的pd_attachment表中删除

根据业务类型/业务id删除附件功能的接口文档如下：

![image-20200428160024666](文件服务.assets/image-20200428160024666.png)

![image-20200428160221111](文件服务.assets/image-20200428160221111.png)

#### 5.5.2 代码实现

第一步：在AttachmentController中提供根据业务类型/业务id删除文件的方法

~~~java
@ApiOperation(value = "根据业务类型或业务id删除文件", 
              notes = "根据业务类型或业务id删除文件")
@DeleteMapping(value = "/biz")
public R<Boolean> removeByBizIdAndBizType(
    									@RequestBody 
                                        AttachmentRemoveDTO dto) {
    attachmentService.removeByBizIdAndBizType(dto.getBizId(), 
                                              dto.getBizType());
    return success(true);
}
~~~

第二步：在AttachmentService接口中扩展removeByBizIdAndBizType方法

~~~java
/**
* 根据业务id/业务类型删除附件
*
* @param bizId
* @param bizType
*/
void removeByBizIdAndBizType(String bizId, String bizType);
~~~

第三步：在AttachmentServiceImpl实现类中实现removeByBizIdAndBizType方法

~~~java
/**
* 根据业务id和业务类型删除附件
*
* @param bizId
* @param bizType
*/
@Override
public void removeByBizIdAndBizType(String bizId, String bizType) {
    //根据业务类和业务id查询数据库
    List<Attachment> list = super.list(
        Wraps.<Attachment>lbQ()
        .eq(Attachment::getBizId, bizId)
        .eq(Attachment::getBizType, bizType));
    if (list.isEmpty()) {
        return;
    }
    
    //根据id删除文件
    remove(list.stream().mapToLong(
        Attachment::getId).boxed().toArray(Long[]::new));
}
~~~

#### 5.5.3 接口测试

第一步：启动Nacos配置中心

第二步：启动Nginx服务

第三步：启动文件服务

第四步：访问接口文档，地址为http://localhost:8765/doc.html

![image-20200428165332463](文件服务.assets/image-20200428165332463.png)

可以看到pd_attachment表中对应的记录已经删除掉了，对应的文件也已经被删除掉了。

### 5.6 接口开发-根据文件id打包下载附件

#### 5.6.1 接口文档

根据文件id打包下载附件接口分两种情况进行下载：

1、如果客户端提交的文件id只有一个，则下载对应的原始文件

2、如果客户端提交的文件id有多个，则将对应的多个原始文件进行压缩，最终下载的是压缩后的文件

接口文档如下：

![image-20200428171359800](文件服务.assets/image-20200428171359800.png)

#### 5.6.2 代码实现

第一步：在AttachmentController中提供根据文件id打包下载文件的方法

~~~java
/**
* 下载一个文件或多个文件打包下载
*
* @param ids      文件id
* @param response
* @throws Exception
*/
@ApiOperation(value = "根据文件id打包下载", notes = "根据附件id下载多个打包的附件")
@GetMapping(value = "/download", produces = "application/octet-stream")
public void download(
    @ApiParam(name = "ids[]", value = "文件id 数组")
    @RequestParam(value = "ids[]") Long[] ids,
    HttpServletRequest request, HttpServletResponse response) throws Exception {
    BizAssert.isTrue(ArrayUtils.isNotEmpty(ids), 
                     BASE_VALID_PARAM.build("附件id不能为空"));
    //根据文件id下载文件
    attachmentService.download(request, response, ids);
}
~~~

第二步：中AttachmentService接口中扩展download方法

~~~java
/**
* 根据文件id下载附件
*
* @param request
* @param response
* @param ids
* @throws Exception
*/
void download(HttpServletRequest request, 
              HttpServletResponse response, 
              Long[] ids) throws Exception;
~~~

第三步：在AttachmentServiceImpl实现类中实现download方法

~~~java
@Autowired
private FileBiz fileBiz;

/**
* 根据文件id下载文件
* @param request
* @param response
* @param ids
* @throws Exception
*/
@Override
public void download(HttpServletRequest request, 
                     HttpServletResponse response, 
                     Long[] ids) throws Exception {
    //根据文件id查询数据库
    List<Attachment> list = 
        (List<Attachment>) super.listByIds(Arrays.asList(ids));
    down(request, response, list);
}

/**
* 文件下载
* @param request
* @param response
* @param list
* @throws Exception
*/
private void down(HttpServletRequest request, HttpServletResponse response, 
                  List<Attachment> list) throws Exception {
    if (list.isEmpty()) {
        throw BizException.wrap("您下载的文件不存在");
    }
    List<FileDO> fileDOList = 
        list.stream().map((file) ->FileDO.builder()
                                 .url(file.getUrl())
                                 .submittedFileName(file.getSubmittedFileName())
                                 .size(file.getSize())
                                 .dataType(file.getDataType())
                                 .build())
        						 .collect(Collectors.toList());
    fileBiz.down(fileDOList, request, response);
}
~~~

第四步：创建FileBiz，统一进行文件下载

~~~java
package com.itheima.pinda.file.biz;

import cn.hutool.core.util.StrUtil;
import com.itheima.pinda.file.domain.FileDO;
import com.itheima.pinda.file.enumeration.DataType;
import com.itheima.pinda.file.utils.ZipUtils;
import com.itheima.pinda.utils.NumberHelper;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

/**
 * 文件和附件的一些公共方法
 */
@Component
@Slf4j
public class FileBiz {
    /**
     * 构建新文件名称
     * @param filename
     * @param order
     * @return
     */
    private static String buildNewFileName(String filename, Integer order) {
        return StrUtil.strBuilder(filename).
            insert(filename.lastIndexOf("."), "(" + order + ")").toString();
    }

    /**
     * 下载文件
     * @param list
     * @param request
     * @param response
     * @throws Exception
     */
    public void down(List<FileDO> list, 
                     HttpServletRequest request, 
                     HttpServletResponse response) throws Exception {
        
        int fileSize = list.stream().filter(
            (file) -> file != null &&
            !DataType.DIR.eq(file.getDataType()) && 
            StringUtils.isNotEmpty(file.getUrl()))
                .mapToInt(
            		(file) -> NumberHelper.intValueOf0(file.getSize())).sum();
        
        String extName = list.get(0).getSubmittedFileName();
        if (list.size() > 1) {
            extName = StringUtils.substring(extName, 0, 
                                  StringUtils.lastIndexOf(extName, ".")) + 
                					"等.zip";
        }

        Map<String, String> map = new LinkedHashMap<>(list.size());
        Map<String, Integer> duplicateFile = new HashMap<>(list.size());
        list.stream()
                //过滤不符合要求的文件
                .filter((file) -> file != null && !DataType.DIR.eq(file.getDataType()) && StringUtils.isNotEmpty(file.getUrl()))
                //循环处理相同的文件名
                .forEach((file) -> {
                    String submittedFileName = file.getSubmittedFileName();
                    if (map.containsKey(submittedFileName)) {
                        if (duplicateFile.containsKey(submittedFileName)) {
                            duplicateFile.put(submittedFileName, duplicateFile.get(submittedFileName) + 1);
                        } else {
                            duplicateFile.put(submittedFileName, 1);
                        }
                        submittedFileName = buildNewFileName(submittedFileName, duplicateFile.get(submittedFileName));
                    }
                    map.put(submittedFileName, file.getUrl());
                });


        ZipUtils.zipFilesByInputStream(map, Long.valueOf(fileSize), extName, request, response);
    }
}
~~~

#### 5.6.3 接口测试

第一步：启动Nacos配置中心

第二步：启动Nginx服务

第三步：启动文件服务

第四步：访问接口文档，地址为http://localhost:8765/doc.html

![image-20200428192949971](文件服务.assets/image-20200428192949971.png)

### 5.7 接口开发-根据业务类型/业务id打包下载

#### 5.7.1 接口文档

根据业务类型/业务id打包下载文件接口分两种情况进行下载：

1、如果根据业务类型和业务id匹配到的文件只有一个，则下载对应的原始文件

2、如果根据业务类型和业务id匹配到的文件有多个，则将对应的多个原始文件进行压缩，最终下载的是压缩后的文件

接口文档如下：

![image-20200428193858972](文件服务.assets/image-20200428193858972.png)

#### 5.7.2 代码实现

第一步：在AttachmentController中提供根据业务类型和业务id打包下载的方法

~~~java
/**
* 根据业务类型或者业务id其中之一，或者2个同时打包下载文件
*
* @param bizIds   业务id
* @param bizTypes 业务类型
*
*/
@ApiImplicitParams({
    @ApiImplicitParam(name = "bizIds[]", value = "业务id数组", dataType = "array", paramType = "query"),
    @ApiImplicitParam(name = "bizTypes[]", value = "业务类型数组", dataType = "array", paramType = "query"),
})
@ApiOperation(value = "根据业务类型/业务id打包下载", notes = "根据业务id下载一个文件或多个文件打包下载")
@GetMapping(value = "/download/biz", produces = "application/octet-stream")
public void downloadByBiz(
    @RequestParam(value = "bizIds[]", required = false) String[] bizIds,
    @RequestParam(value = "bizTypes[]", required = false) String[] bizTypes,
    HttpServletRequest request, HttpServletResponse response) throws Exception {
    BizAssert.isTrue(!(ArrayUtils.isEmpty(bizTypes) && ArrayUtils.isEmpty(bizIds)), BASE_VALID_PARAM.build("附件业务id和业务类型不能同时为空"));
    attachmentService.downloadByBiz(request, response, bizTypes, bizIds);
}
~~~

第二步：在AttachmentService接口中扩展downloadByBiz方法

~~~java
/**
* 根据业务id和业务类型下载附件
*
* @param request
* @param response
* @param bizTypes
* @param bizIds
* @throws Exception
*/
void downloadByBiz(HttpServletRequest request, HttpServletResponse response, 
                   String[] bizTypes, String[] bizIds) throws Exception;
~~~

第三步：在AttachmentServiceImpl实现类中实现downloadByBiz方法

~~~java
/**
* 根据业务id和业务类型下载附件
*
* @param request
* @param response
* @param bizTypes
* @param bizIds
* @throws Exception
*/
@Override
public void downloadByBiz(HttpServletRequest request, HttpServletResponse response, String[] bizTypes, String[] bizIds) throws Exception {
    List<Attachment> list = super.list(
        Wraps.<Attachment>lbQ()
        .in(Attachment::getBizType, bizTypes)
        .in(Attachment::getBizId, bizIds));

    down(request, response, list);
}
~~~

#### 5.7.3 接口测试

第一步：启动Nacos配置中心

第二步：启动Nginx服务

第三步：启动文件服务

第四步：访问接口文档，地址为http://localhost:8765/doc.html

![image-20200428194415419](文件服务.assets/image-20200428194415419.png)

### 5.8 导入其他接口代码

本课程重点讲解文件的上传、下载、文件处理策略的实现，对于纯数据库操作相关的功能不再作为重点讲解，此处我们直接导入这些代码即可。

#### 5.8.1 接口导入-分页查询附件

接口文档：

![image-20200430152747062](文件服务.assets/image-20200430152747062.png)

AttachmentController代码：

~~~java
/**
* 分页查询附件
*
*/
@ApiOperation(value = "分页查询附件", notes = "分页查询附件")
@ApiImplicitParams({
    @ApiImplicitParam(name = "current", value = "当前页", dataType = "long", paramType = "query", defaultValue = "1"),
    @ApiImplicitParam(name = "size", value = "每页显示几条", dataType = "long", paramType = "query", defaultValue = "10"),
})
@GetMapping(value = "/page")
public R<IPage<Attachment>> page(FilePageReqDTO data) {
    Page<Attachment> page = getPage();
    attachmentService.page(page, data);
    return success(page);
}
~~~

AttachmentService接口：

~~~java
/**
* 查询附件分页数据
*
* @param page
* @param data
* @return
*/
IPage<Attachment> page(Page<Attachment> page, FilePageReqDTO data);
~~~

AttachmentServiceImpl类：

~~~java
/**
* 查询附件分页数据
*
* @param page
* @param data
* @return
*/
public IPage<Attachment> page(Page<Attachment> page, FilePageReqDTO data) {
    Attachment attachment = dozer.map(data, Attachment.class);

    // ${ew.customSqlSegment} 语法一定要手动eq like 等 不能用lbQ!
    LbqWrapper<Attachment> wrapper = Wraps.<Attachment>lbQ()
        .like(Attachment::getSubmittedFileName, attachment.getSubmittedFileName())
        .like(Attachment::getBizType, attachment.getBizType())
        .like(Attachment::getBizId, attachment.getBizId())
        .eq(Attachment::getDataType, attachment.getDataType())
        .orderByDesc(Attachment::getId);
    return baseMapper.page(page, wrapper);
}
~~~

#### 5.8.2 接口导入-根据业务类型/业务id查询附件

接口文档：

![image-20200430153452092](文件服务.assets/image-20200430153452092.png)

AttachmentController代码：

~~~java
@ApiOperation(value = "查询附件", notes = "查询附件")
@ApiResponses(
    @ApiResponse(code = 60103, message = "文件id为空")
)
@GetMapping
public R<List<AttachmentResultDTO>> findAttachment(@RequestParam(value = "bizTypes", required = false) String[] bizTypes,
                                                   @RequestParam(value = "bizIds", required = false) String[] bizIds) {
    //不能同时为空
    BizAssert.isTrue(!(ArrayUtils.isEmpty(bizTypes) && ArrayUtils.isEmpty(bizIds)), BASE_VALID_PARAM.build("业务类型不能为空"));
    return success(attachmentService.find(bizTypes, bizIds));
}
~~~

AttachmentService接口：

~~~java
/**
* 根据业务类型和业务id查询附件
*
* @param bizTypes
* @param bizIds
* @return
*/
List<AttachmentResultDTO> find(String[] bizTypes, String[] bizIds);
~~~

AttachmentServiceImpl类：

~~~java
/**
* 根据业务类型和业务id查询附件
*
* @param bizTypes
* @param bizIds
* @return
*/
public List<AttachmentResultDTO> find(String[] bizTypes, String[] bizIds) {
        return baseMapper.find(bizTypes, bizIds);
}
~~~

### 5.9 导入网盘服务接口

前面我们已经完成了文件服务中的附件服务相关接口的开发，附件服务最终是将上传的文件信息保存在pd_attachment表中。

本小节要完成的是文件服务中的网盘服务功能，此功能最终是将上传的文件信息保存在pd_file表中。网盘服务和附件服务非常类似，只是多了一个文件夹的概念。

#### 5.9.1 导入FileController

~~~java
package com.itheima.pinda.file.controller;

import com.baomidou.mybatisplus.core.metadata.IPage;
import com.itheima.pinda.base.BaseController;
import com.itheima.pinda.base.R;
import com.itheima.pinda.dozer.DozerUtils;
import com.itheima.pinda.file.dto.FilePageReqDTO;
import com.itheima.pinda.file.dto.FileUpdateDTO;
import com.itheima.pinda.file.dto.FolderDTO;
import com.itheima.pinda.file.dto.FolderSaveDTO;
import com.itheima.pinda.file.entity.File;
import com.itheima.pinda.file.manager.FileRestManager;
import com.itheima.pinda.file.service.FileService;
import com.itheima.pinda.log.annotation.SysLog;
import io.swagger.annotations.*;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.validation.Valid;
import javax.validation.constraints.NotNull;

/**
 * 文件前端控制器
 */
@Validated
@RestController
@RequestMapping("/file")
@Slf4j
@Api(value = "文件表", tags = "文件表")
public class FileController extends BaseController {
    @Autowired
    private FileService fileService;
    @Autowired
    private FileRestManager fileRestManager;
    @Autowired
    private DozerUtils dozerUtils;

    /**
     * 查询单个文件信息
     *
     * @param id
     * @return
     */
    @ApiOperation(value = "查询文件", notes = "查询文件")
    @GetMapping
    public R<File> get(@RequestParam(value = "id") Long id) {
        File file = fileService.getById(id);
        if (file != null && file.getIsDelete()) {
            return success(null);
        }
        return success(file);
    }

    /**
     * 获取文件分页
     */
    @ApiOperation(value = "分页查询文件", notes = "获取文件分页")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "current", value = "当前页", dataType = "long", paramType = "query", defaultValue = "1"),
            @ApiImplicitParam(name = "size", value = "每页显示几条", dataType = "long", paramType = "query", defaultValue = "10"),
    })
    @GetMapping(value = "/page")
    public R<IPage<File>> page(FilePageReqDTO data) {
        return success(fileRestManager.page(getPage(), data));
    }

    /**
     * 上传文件
     */
    @ApiOperation(value = "上传文件", notes = "上传文件 ")
    @ApiResponses({
            @ApiResponse(code = 60102, message = "文件夹为空"),
    })
    @ApiImplicitParams({
            @ApiImplicitParam(name = "source", value = "文件来源", dataType = "String", paramType = "query"),
            @ApiImplicitParam(name = "userId", value = "用户id", dataType = "long", paramType = "query"),
            @ApiImplicitParam(name = "folderId", value = "文件夹id", dataType = "long", paramType = "query"),
            @ApiImplicitParam(name = "file", value = "附件", dataType = "MultipartFile", allowMultiple = true, required = true),
    })
    @RequestMapping(value = "/upload", method = RequestMethod.POST)
    public R<File> upload(
            @NotNull(message = "文件夹不能为空")
            @RequestParam(name = "source", defaultValue = "inner") String source,
            @RequestParam(name = "userId", required = false) Long userId,
            @RequestParam(value = "folderId") Long folderId,
            @RequestParam(value = "file") MultipartFile simpleFile) {
        //1，先将文件存在本地,并且生成文件名
        log.info("contentType={}, name={} , sfname={}", simpleFile.getContentType(), simpleFile.getName(), simpleFile.getOriginalFilename());
        // 忽略路径字段,只处理文件类型
        if (simpleFile.getContentType() == null) {
            return fail("文件为空");
        }

        File file = fileService.upload(simpleFile, folderId, source, userId);

        return success(file);
    }


    /**
     * 保存文件夹
     */
    @ApiResponses({
            @ApiResponse(code = 60000, message = "文件夹为空"),
            @ApiResponse(code = 60001, message = "文件夹名称为空"),
            @ApiResponse(code = 60002, message = "父文件夹为空"),
    })
    @ApiOperation(value = "新增文件夹", notes = "新增文件夹")
    @RequestMapping(value = "", method = RequestMethod.POST)
    public R<FolderDTO> saveFolder(@Valid @RequestBody FolderSaveDTO folderSaveDto) {
        //2，获取身份

        FolderDTO folder = fileService.saveFolder(folderSaveDto);
        return success(folder);
    }

    /**
     * 修改文件、文件夹信息
     *
     * @param fileUpdateDTO
     * @return
     */
    @ApiOperation(value = "修改文件/文件夹名称", notes = "修改文件/文件夹名称")
    @ApiResponses({
            @ApiResponse(code = 60100, message = "文件为空"),
    })
    @RequestMapping(value = "", method = RequestMethod.PUT)
    public R<Boolean> update(@Valid @RequestBody FileUpdateDTO fileUpdateDTO) {
        // 判断文件名是否有 后缀
        if (StringUtils.isNotEmpty(fileUpdateDTO.getSubmittedFileName())) {
            File oldFile = fileService.getById(fileUpdateDTO.getId());
            if (oldFile.getExt() != null && !fileUpdateDTO.getSubmittedFileName().endsWith(oldFile.getExt())) {
                fileUpdateDTO.setSubmittedFileName(fileUpdateDTO.getSubmittedFileName() + "." + oldFile.getExt());
            }
        }
        File file = dozerUtils.map2(fileUpdateDTO, File.class);

        fileService.updateById(file);
        return success(true);
    }

    /**
     * 根据Ids进行文件删除
     *
     * @param ids
     * @return
     */
    @ApiOperation(value = "根据Ids进行文件删除", notes = "根据Ids进行文件删除  ")
    @DeleteMapping(value = "/ids")
    public R<Boolean> removeList(@RequestParam(value = "ids[]") Long[] ids) {
        Long userId = getUserId();
        return success(fileService.removeList(userId, ids));
    }

    /**
     * 下载一个文件或多个文件打包下载
     *
     * @param ids
     * @param response
     * @throws Exception
     */
    @ApiOperation(value = "下载一个文件或多个文件打包下载", notes = "下载一个文件或多个文件打包下载")
    @GetMapping(value = "/download", produces = "application/octet-stream")
    public void download(
            @ApiParam(name = "ids[]", value = "文件id 数组")
            @RequestParam(value = "ids[]") Long[] ids,
            HttpServletRequest request, HttpServletResponse response) throws Exception {
        fileRestManager.download(request, response, ids, null);
    }

}
~~~

#### 5.9.2 导入StatisticsController

~~~java
package com.itheima.pinda.file.controller;

import com.itheima.pinda.base.BaseController;
import com.itheima.pinda.base.R;
import com.itheima.pinda.file.domain.FileStatisticsDO;
import com.itheima.pinda.file.dto.FileOverviewDTO;
import com.itheima.pinda.file.dto.FileStatisticsAllDTO;
import com.itheima.pinda.file.service.FileService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import java.time.LocalDateTime;
import java.util.List;

/**
 * 文件统计接口
 */
@Slf4j
@RestController
@RequestMapping("/statistics")
@Api(value = "Statistics", tags = "统计接口")
public class StatisticsController extends BaseController {
    @Autowired
    private FileService fileService;

    @ApiOperation(value = "云盘首页数据概览", notes = "云盘首页数据概览")
    @GetMapping(value = "/overview")
    public R<FileOverviewDTO> overview(@RequestParam(name = "userId", required = false) Long userId) {
        return success(fileService.findOverview(userId, null, null));
    }

    @ApiOperation(value = "按照类型，统计各种类型的 大小和数量", notes = "按照类型，统计当前登录人各种类型的大小和数量")
    @GetMapping(value = "/type")
    public R<List<FileStatisticsDO>> findAllByDataType(@RequestParam(name = "userId", required = false) Long userId) {
        return success(fileService.findAllByDataType(userId));
    }

    @ApiOperation(value = "按照时间统计各种类型的文件的数量和大小", notes = "按照时间统计各种类型的文件的数量和大小 不指定时间，默认查询一个月")
    @GetMapping(value = "")
    public R<FileStatisticsAllDTO> findNumAndSizeToTypeByDate(@RequestParam(name = "userId", required = false) Long userId,
                                                              @RequestParam(value = "startTime", required = false) LocalDateTime startTime,
                                                              @RequestParam(value = "endTime", required = false) LocalDateTime endTime) {
        return success(fileService.findNumAndSizeToTypeByDate(userId, startTime, endTime));
    }
}
~~~

#### 5.9.3 导入FileRestManager

~~~java
package com.itheima.pinda.file.manager;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.core.metadata.IPage;
import com.itheima.pinda.context.BaseContextHandler;
import com.itheima.pinda.file.constant.FileConstants;
import com.itheima.pinda.file.dto.FilePageReqDTO;
import com.itheima.pinda.file.entity.File;
import com.itheima.pinda.file.service.FileService;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import static com.itheima.pinda.utils.StrPool.DEF_PARENT_ID;

/**
 * 文件 公共代码 管理类
 */
@Component
public class FileRestManager {
    @Autowired
    private FileService fileService;

    public IPage<File> page(IPage<File> page, FilePageReqDTO filePageReq) {
        //类型和文件夹id同时为null时， 表示查询 全部文件
        if (filePageReq.getFolderId() == null && filePageReq.getDataType() == null) {
            filePageReq.setFolderId(DEF_PARENT_ID);
        }

        QueryWrapper<File> query = new QueryWrapper<>();
        LambdaQueryWrapper<File> lambdaQuery = query.lambda()
                .eq(File::getIsDelete, false)
                .eq(filePageReq.getDataType() != null, File::getDataType, filePageReq.getDataType())
                .eq(filePageReq.getFolderId() != null, File::getFolderId, filePageReq.getFolderId())
                .like(StringUtils.isNotEmpty(filePageReq.getSubmittedFileName()), File::getSubmittedFileName, filePageReq.getSubmittedFileName());

        query.orderByDesc(String.format("case when %s='DIR' THEN 1 else 0 end", FileConstants.DATA_TYPE));
        lambdaQuery.orderByDesc(File::getCreateTime);

        fileService.page(page, lambdaQuery);
        return page;
    }

    public void download(HttpServletRequest request, HttpServletResponse response, Long[] ids, Long userId) throws Exception {
        userId = userId == null || userId <= 0 ? BaseContextHandler.getUserId() : userId;
        fileService.download(request, response, ids, userId);
    }
}
~~~

#### 5.9.4 导入FileService

~~~java
package com.itheima.pinda.file.service;

import com.baomidou.mybatisplus.extension.service.IService;
import com.itheima.pinda.file.domain.FileAttrDO;
import com.itheima.pinda.file.domain.FileStatisticsDO;
import com.itheima.pinda.file.dto.FileOverviewDTO;
import com.itheima.pinda.file.dto.FileStatisticsAllDTO;
import com.itheima.pinda.file.dto.FolderDTO;
import com.itheima.pinda.file.dto.FolderSaveDTO;
import com.itheima.pinda.file.entity.File;
import org.springframework.web.multipart.MultipartFile;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.time.LocalDateTime;
import java.util.List;
/**
 * 业务接口
 * 文件表
 *
 */
public interface FileService extends IService<File> {
    /**
     * 保存文件夹
     *
     * @param folderSaveDto 文件夹
     * @return
     */
    FolderDTO saveFolder(FolderSaveDTO folderSaveDto);

    /**
     * 根据文件id下载文件，并统计下载次数
     *
     * @param request  请求
     * @param response 响应
     * @param ids      文件id集合
     * @param userId   用户id
     * @throws Exception
     */
    void download(HttpServletRequest request, HttpServletResponse response,
                  Long[] ids, Long userId) throws Exception;

    /**
     * 根据文件id和用户id 删除文件或者文件夹
     *
     * @param userId 用户id
     * @param ids    文件id集合
     * @return
     */
    Boolean removeList(Long userId, Long[] ids);

    /**
     * 根据文件夹id查询
     *
     * @param folderId
     * @return
     */
    FileAttrDO getFileAttrDo(Long folderId);

    /**
     * 文件上传
     *
     * @param simpleFile 文件
     * @param folderId   文件夹id
     * @return
     */
    File upload(MultipartFile simpleFile, Long folderId);

	/**
	 * 文件上传
	 *
	 * @param simpleFile
	 * @param folderId
	 * @param source
	 * @param userId
	 * @return
	 */
	File upload(MultipartFile simpleFile, Long folderId, String source, Long userId);

    /**
     * 首页概览
     *
     * @param userId
     * @param startTime
     * @param endTime
     * @return
     */
    FileOverviewDTO findOverview(Long userId, LocalDateTime startTime, LocalDateTime endTime);

    /**
     * 首页个人文件发展概览
     *
     * @param userId
     * @param startTime
     * @param endTime
     * @return
     */
    FileStatisticsAllDTO findAllByDate(Long userId, LocalDateTime startTime, LocalDateTime endTime);


    /**
     * 按照 数据类型分类查询 当前人的所有文件的数量和大小
     *
     * @param userId
     * @return
     */
    List<FileStatisticsDO> findAllByDataType(Long userId);

    /**
     * 查询下载排行前20的文件
     *
     * @param userId
     * @return
     */
    List<FileStatisticsDO> downTop20(Long userId);

    /**
     * 根据日期查询，特定类型的数量和大小
     *
     * @param userId
     * @param startTime
     * @param endTime
     * @return
     */
    FileStatisticsAllDTO findNumAndSizeToTypeByDate(Long userId, LocalDateTime startTime, LocalDateTime endTime);

    /**
     * 根据日期查询下载大小
     *
     * @param userId
     * @param startTime
     * @param endTime
     * @return
     */
    FileStatisticsAllDTO findDownSizeByDate(Long userId, LocalDateTime startTime,
                                            LocalDateTime endTime);
}
~~~

#### 5.9.5 导入FileServiceImpl

~~~java
package com.itheima.pinda.file.service.impl;

import com.baomidou.mybatisplus.core.toolkit.Wrappers;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.itheima.pinda.database.mybatis.conditions.Wraps;
import com.itheima.pinda.database.mybatis.conditions.update.LbuWrapper;
import com.itheima.pinda.dozer.DozerUtils;
import com.itheima.pinda.file.biz.FileBiz;
import com.itheima.pinda.file.dao.FileMapper;
import com.itheima.pinda.file.domain.FileAttrDO;
import com.itheima.pinda.file.domain.FileDO;
import com.itheima.pinda.file.domain.FileDeleteDO;
import com.itheima.pinda.file.domain.FileStatisticsDO;
import com.itheima.pinda.file.dto.FileOverviewDTO;
import com.itheima.pinda.file.dto.FileStatisticsAllDTO;
import com.itheima.pinda.file.dto.FolderDTO;
import com.itheima.pinda.file.dto.FolderSaveDTO;
import com.itheima.pinda.file.entity.File;
import com.itheima.pinda.file.enumeration.DataType;
import com.itheima.pinda.file.enumeration.IconType;
import com.itheima.pinda.file.service.FileService;
import com.itheima.pinda.file.strategy.FileStrategy;
import com.itheima.pinda.utils.BizAssert;
import com.itheima.pinda.utils.DateUtils;
import lombok.Getter;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.ArrayUtils;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;
import java.util.stream.Stream;
import static com.itheima.pinda.exception.code.ExceptionCode.BASE_VALID_PARAM;
import static com.itheima.pinda.utils.StrPool.DEF_PARENT_ID;
import static com.itheima.pinda.utils.StrPool.DEF_ROOT_PATH;
import static java.util.stream.Collectors.groupingBy;

/**
 * 业务实现类
 * 文件表
 */
@Slf4j
@Service
public class FileServiceImpl extends ServiceImpl<FileMapper, File> implements FileService {
    @Autowired
    private DozerUtils dozerUtils;

    @Autowired
    private FileBiz fileBiz;
    @Resource
    private FileStrategy fileStrategy;

    @Override
    public File upload(MultipartFile simpleFile, Long folderId) {
        FileAttrDO fileAttrDO = this.getFileAttrDo(folderId);
        String treePath = fileAttrDO.getTreePath();
        String folderName = fileAttrDO.getFolderName();
        Integer grade = fileAttrDO.getGrade();

        File file = fileStrategy.upload(simpleFile);
        file.setFolderId(folderId);
        file.setFolderName(folderName);
        file.setGrade(grade);
        file.setTreePath(treePath);
        super.save(file);
        return file;
    }

	@Override
	public File upload(MultipartFile simpleFile, Long folderId, String source, Long userId) {
		FileAttrDO fileAttrDO = this.getFileAttrDo(folderId);
		String treePath = fileAttrDO.getTreePath();
		String folderName = fileAttrDO.getFolderName();
		Integer grade = fileAttrDO.getGrade();

		File file = fileStrategy.upload(simpleFile);
		file.setFolderId(folderId);
		file.setFolderName(folderName);
		file.setGrade(grade);
		file.setTreePath(treePath);

		file.setSource(source);
		if (userId != null) {
			file.setCreateUser(userId);
		}

		super.save(file);
		return file;
	}

	@Override
    public FileAttrDO getFileAttrDo(Long folderId) {
        String treePath = DEF_ROOT_PATH;
        String folderName = "";
        Integer grade = 1;
        if (folderId == null || folderId <= 0) {
            return new FileAttrDO(treePath, grade, folderName, DEF_PARENT_ID);
        }
        File folder = this.getById(folderId);

        if (folder != null && !folder.getIsDelete() && DataType.DIR.eq(folder.getDataType())) {
            folderName = folder.getSubmittedFileName();
            treePath = StringUtils.join(folder.getTreePath(), folder.getId(), DEF_ROOT_PATH);
            grade = folder.getGrade() + 1;
        }
        BizAssert.isTrue(grade <= 10, BASE_VALID_PARAM.build("文件夹层级不能超过10层"));
        return new FileAttrDO(treePath, grade, folderName, folderId);
    }


    @Override
    public FolderDTO saveFolder(FolderSaveDTO folderSaveDto) {
        File folder = dozerUtils.map2(folderSaveDto, File.class);
        if (folderSaveDto.getFolderId() == null || folderSaveDto.getFolderId() <= 0) {
            folder.setFolderId(DEF_PARENT_ID);
            folder.setTreePath(DEF_ROOT_PATH);
            folder.setGrade(1);
        } else {
            File parent = super.getById(folderSaveDto.getFolderId());
            BizAssert.notNull(parent, BASE_VALID_PARAM.build("父文件夹不能为空"));
            BizAssert.isFalse(parent.getIsDelete(), BASE_VALID_PARAM.build("父文件夹已经被删除"));
            BizAssert.equals(DataType.DIR.name(), parent.getDataType().name(), BASE_VALID_PARAM.build("父文件夹不存在"));
            BizAssert.isTrue(parent.getGrade() < 10, BASE_VALID_PARAM.build("文件夹层级不能超过10层"));
            folder.setFolderName(parent.getSubmittedFileName());
            folder.setTreePath(StringUtils.join(parent.getTreePath(), parent.getId(), DEF_ROOT_PATH));
            folder.setGrade(parent.getGrade() + 1);
        }
        if (folderSaveDto.getOrderNum() == null) {
            folderSaveDto.setOrderNum(0);
        }
        folder.setIsDelete(false);
        folder.setDataType(DataType.DIR);
        folder.setIcon(IconType.DIR.getIcon());
        setDate(folder);
        super.save(folder);
        return dozerUtils.map2(folder, FolderDTO.class);
    }

    private void setDate(File file) {
        LocalDateTime now = LocalDateTime.now();
        file.setCreateMonth(DateUtils.formatAsYearMonthEn(now))
                .setCreateWeek(DateUtils.formatAsYearWeekEn(now))
                .setCreateDay(DateUtils.formatAsDateEn(now));
    }

    public boolean removeFile(Long[] ids, Long userId) {
        LbuWrapper<File> lambdaUpdate =
                Wraps.<File>lbU()
                        .in(File::getId, ids)
                        .eq(File::getCreateUser, userId);
        File file = File.builder().isDelete(Boolean.TRUE).build();

        return super.update(file, lambdaUpdate);
    }

    @Override
    public Boolean removeList(Long userId, Long[] ids) {
        if (ArrayUtils.isEmpty(ids)) {
            return Boolean.TRUE;
        }
        List<File> list = super.list(Wrappers.<File>lambdaQuery().in(File::getId, ids));
        if (list.isEmpty()) {
            return true;
        }
        super.removeByIds(Arrays.asList(ids));

        fileStrategy.delete(list.stream().map((fi) -> FileDeleteDO.builder()
                .relativePath(fi.getRelativePath())
                .fileName(fi.getFilename())
                .group(fi.getGroup())
                .path(fi.getPath())
                .file(false)
                .build())
                .collect(Collectors.toList()));
        return true;
    }

    @Override
    public void download(HttpServletRequest request, HttpServletResponse response, Long[] ids, Long userId) throws Exception {
        if (ids == null || ids.length == 0) {
            return;
        }
        List<File> list = (List<File>) super.listByIds(Arrays.asList(ids));

        if (list == null || list.size() == 0) {
            return;
        }
        List<FileDO> listDo = list.stream().map((file) ->
                FileDO.builder()
                        .dataType(file.getDataType())
                        .size(file.getSize())
                        .submittedFileName(file.getSubmittedFileName())
                        .url(file.getUrl())
                        .build())
                .collect(Collectors.toList());
        fileBiz.down(listDo, request, response);
    }


    @Override
    public FileOverviewDTO findOverview(Long userId, LocalDateTime startTime, LocalDateTime endTime) {
        InnerQueryDate innerQueryDate = new InnerQueryDate(userId, startTime, endTime).invoke();
        startTime = innerQueryDate.getStartTime();
        endTime = innerQueryDate.getEndTime();

        List<FileStatisticsDO> list = baseMapper.findNumAndSizeByUserId(userId, null, "ALL", startTime, endTime);
        FileOverviewDTO.FileOverviewDTOBuilder builder = FileOverviewDTO.myBuilder();

        long allSize = 0L;
        int allNum = 0;
        for (FileStatisticsDO fs : list) {
            allSize += fs.getSize();
            allNum += fs.getNum();
            switch (fs.getDataType()) {
                case DIR:
                    builder.dirNum(fs.getNum());
                    break;
                case IMAGE:
                    builder.imgNum(fs.getNum());
                    break;
                case VIDEO:
                    builder.videoNum(fs.getNum());
                    break;
                case DOC:
                    builder.docNum(fs.getNum());
                    break;
                case AUDIO:
                    builder.audioNum(fs.getNum());
                    break;
                case OTHER:
                    builder.otherNum(fs.getNum());
                    break;
                default:
                    break;
            }
        }
        builder.allFileNum(allNum).allFileSize(allSize);
        return builder.build();
    }

    @Override
    public FileStatisticsAllDTO findAllByDate(Long userId, LocalDateTime startTime, LocalDateTime endTime) {
        InnerQueryDate innerQueryDate = new InnerQueryDate(userId, startTime, endTime).invoke();
        startTime = innerQueryDate.getStartTime();
        endTime = innerQueryDate.getEndTime();
        List<String> dateList = innerQueryDate.getDateList();
        String dateType = innerQueryDate.getDateType();

        //不完整的数据
        List<FileStatisticsDO> list = baseMapper.findNumAndSizeByUserId(userId, dateType, null, startTime, endTime);

        //按月份分类
        Map<String, List<FileStatisticsDO>> map = list.stream().collect(groupingBy(FileStatisticsDO::getDateType));

        List<Long> sizeList = new ArrayList<>();
        List<Integer> numList = new ArrayList<>();

        dateList.forEach((date) -> {
            if (map.containsKey(date)) {
                List<FileStatisticsDO> subList = map.get(date);

                Long size = subList.stream().mapToLong(FileStatisticsDO::getSize).sum();
                Integer num = subList.stream().filter((fs) -> !DataType.DIR.eq(fs.getDataType()))
                        .mapToInt(FileStatisticsDO::getNum).sum();
                sizeList.add(size);
                numList.add(num);
            } else {
                sizeList.add(0L);
                numList.add(0);
            }
        });

        return FileStatisticsAllDTO.builder().dateList(dateList).numList(numList).sizeList(sizeList).build();
    }


    @Override
    public List<FileStatisticsDO> findAllByDataType(Long userId) {
        List<DataType> dataTypes = Arrays.asList(DataType.values());
        List<FileStatisticsDO> list = baseMapper.findNumAndSizeByUserId(userId, null, "ALL", null, null);

        Map<DataType, List<FileStatisticsDO>> map = list.stream().collect(groupingBy(FileStatisticsDO::getDataType));

        return dataTypes.stream().map((type) -> {
            FileStatisticsDO fs = null;
            if (map.containsKey(type)) {
                fs = map.get(type).get(0);
            } else {
                fs = FileStatisticsDO.builder().dataType(type).size(0L).num(0).build();
            }
            return fs;
        }).collect(Collectors.toList());
    }

    @Override
    public List<FileStatisticsDO> downTop20(Long userId) {
        return baseMapper.findDownTop20(userId);
    }

    @Override
    public FileStatisticsAllDTO findNumAndSizeToTypeByDate(Long userId, LocalDateTime startTime, LocalDateTime endTime) {
        return common(userId, startTime, endTime,
                (qd) -> baseMapper.findNumAndSizeByUserId(qd.getUserId(), qd.getDateType(), "ALL", qd.getStartTime(), qd.getEndTime()));
    }

    @Override
    public FileStatisticsAllDTO findDownSizeByDate(Long userId, LocalDateTime startTime,
                                                   LocalDateTime endTime) {
        return common(userId, startTime, endTime,
                (qd) -> baseMapper.findDownSizeByDate(qd.getUserId(), qd.getDateType(), qd.getStartTime(), qd.getEndTime()));
    }

    /**
     * 抽取公共查询公共代码
     *
     * @param userId    用户id
     * @param startTime 开始时间
     * @param endTime   结束时间
     * @param function  回调函数
     * @return
     */
    private FileStatisticsAllDTO common(Long userId, LocalDateTime startTime, LocalDateTime endTime, Function<InnerQueryDate, List<FileStatisticsDO>> function) {
        InnerQueryDate innerQueryDate = new InnerQueryDate(userId, startTime, endTime).invoke();
        List<String> dateList = innerQueryDate.getDateList();

        List<FileStatisticsDO> list = function.apply(innerQueryDate);

        //按月份分类
        Map<String, List<FileStatisticsDO>> map = list.stream().collect(groupingBy(FileStatisticsDO::getDateType));

        List<Long> sizeList = new ArrayList<>(dateList.size());
        List<Integer> numList = new ArrayList<>(dateList.size());

        List<Integer> dirNumList = new ArrayList<>(dateList.size());

        List<Long> imgSizeList = new ArrayList<>(dateList.size());
        List<Integer> imgNumList = new ArrayList<>(dateList.size());

        List<Long> videoSizeList = new ArrayList<>(dateList.size());
        List<Integer> videoNumList = new ArrayList<>(dateList.size());

        List<Long> audioSizeList = new ArrayList<>(dateList.size());
        List<Integer> audioNumList = new ArrayList<>(dateList.size());

        List<Long> docSizeList = new ArrayList<>(dateList.size());
        List<Integer> docNumList = new ArrayList<>(dateList.size());

        List<Long> otherSizeList = new ArrayList<>(dateList.size());
        List<Integer> otherNumList = new ArrayList<>(dateList.size());

        dateList.forEach((date) -> {
            if (map.containsKey(date)) {
                List<FileStatisticsDO> subList = map.get(date);

                Function<DataType, Stream<FileStatisticsDO>> stream = (dataType) -> subList.stream().filter((fs) -> !dataType.eq(fs.getDataType()));
                Long size = stream.apply(DataType.DIR).mapToLong(FileStatisticsDO::getSize).sum();
                Integer num = stream.apply(DataType.DIR).mapToInt(FileStatisticsDO::getNum).sum();
                sizeList.add(size);
                numList.add(num);

                Integer dirNum = subList.stream().filter((fs) -> DataType.DIR.eq(fs.getDataType()))
                        .mapToInt(FileStatisticsDO::getNum).sum();
                dirNumList.add(dirNum);

                add(imgSizeList, imgNumList, subList, DataType.IMAGE);
                add(videoSizeList, videoNumList, subList, DataType.VIDEO);
                add(audioSizeList, audioNumList, subList, DataType.AUDIO);
                add(docSizeList, docNumList, subList, DataType.DOC);
                add(otherSizeList, otherNumList, subList, DataType.OTHER);

            } else {
                sizeList.add(0L);
                numList.add(0);
                dirNumList.add(0);
                imgSizeList.add(0L);
                imgNumList.add(0);
                videoSizeList.add(0L);
                videoNumList.add(0);
                audioSizeList.add(0L);
                audioNumList.add(0);
                docSizeList.add(0L);
                docNumList.add(0);
                otherSizeList.add(0L);
                otherNumList.add(0);
            }
        });

        return FileStatisticsAllDTO.builder()
                .dateList(dateList)
                .numList(numList).sizeList(sizeList)
                .dirNumList(dirNumList)
                .imgNumList(imgNumList).imgSizeList(imgSizeList)
                .videoNumList(videoNumList).videoSizeList(videoSizeList)
                .audioNumList(audioNumList).audioSizeList(audioSizeList)
                .docNumList(docNumList).docSizeList(docSizeList)
                .otherNumList(otherNumList).otherSizeList(otherSizeList)
                .build();
    }

    private void add(List<Long> sizeList, List<Integer> numList, List<FileStatisticsDO> subList, DataType dt) {
        Function<DataType, Stream<FileStatisticsDO>> stream =
                dataType -> subList.stream().filter(fs -> dataType.eq(fs.getDataType()));

        Long size = stream.apply(dt).mapToLong(FileStatisticsDO::getSize).sum();
        Integer num = stream.apply(dt).mapToInt(FileStatisticsDO::getNum).sum();
        sizeList.add(size);
        numList.add(num);
    }

    @Getter
    private static class InnerQueryDate {
        private LocalDateTime startTime;
        private LocalDateTime endTime;
        private List<String> dateList;
        private String dateType;
        private Long userId;

        public InnerQueryDate(Long userId, LocalDateTime startTime, LocalDateTime endTime) {
            this.userId = userId;
            this.startTime = startTime;
            this.endTime = endTime;
        }

        public InnerQueryDate invoke() {
            if (startTime == null) {
                startTime = LocalDateTime.now().plusDays(-9);
            }
            if (endTime == null) {
                endTime = LocalDateTime.now();
            }
            endTime = LocalDateTime.of(endTime.toLocalDate(), LocalTime.MAX);
            dateList = new ArrayList<>();
            dateType = DateUtils.calculationEn(startTime, endTime, dateList);
            return this;
        }
    }
}
~~~

#### 5.9.6 扩展FileMapper接口方法

~~~java
/**
* 查询下次次数前20的文件
*
* @param userId
* @return
*/
List<FileStatisticsDO> findDownTop20(@Param("userId") Long userId);

/**
* 统计时间区间内文件的下次次数和大小
*
* @param userId
* @param dateType  日期类型 {MONTH:按月;WEEK:按周;DAY:按日} 来统计
* @param startTime
* @param endTime
* @return
*/
List<FileStatisticsDO> findDownSizeByDate(@Param("userId") Long userId,
                                          @Param("dateType") String dateType,
                                          @Param("startTime") LocalDateTime startTime,
                                          @Param("endTime") LocalDateTime endTime);
~~~

### 5.10 接口开发-分片上传

#### 5.10.1 分片上传介绍

前面的课程我们已经实现了普通的附件服务和网盘服务，如果上传的文件比较小，可以直接使用这两个服务即可。如果上传的文件比较大，例如要上传一个500M或者1G的视频文件（或者更大），这就需要分片上传了。那么什么是分片上传呢？

分片上传就是把一个大文件进行分片，一片一片的上传到服务端，最后由服务端进行分片的合并。

要实现分片上传需要前端和后端配合来完成。在进行分片上传时，一般是由前端对要上传的大文件进行分片，然后分多次将这些分片上传到服务端，所有分片都上传到服务端后，在服务端将分片合并为原始的大文件。采用大文件分片并发上传，可以极大的提高文件的上传效率。

#### 5.10.2 前端分片上传插件webuploader

WebUploader是由Baidu WebFE(FEX)团队开发的一个简单的以HTML5为主，FLASH为辅的现代文件上传组件。在现代的浏览器里面能充分发挥HTML5的优势，同时又不摒弃主流IE浏览器，沿用原来的FLASH运行时，兼容IE6+，iOS 6+, android 4+。

官网地址：http://fex.baidu.com/webuploader/

分片与并发结合，将一个大文件分割成多块，并发上传，极大地提高大文件的上传速度。

当网络问题导致传输错误时，只需要重传出错分片，而不是整个文件。另外分片传输能够更加实时的跟踪上传进度。

由于本课程主要为后端服务开发，所以前端部分不再开发，直接从资料中获得使用即可。

资料位置：`文件服务\资料\分片上传\前端`

直接打开index.html页面，选择要上传的大文件，可以看到发送了多次请求，每次请求会上传此大文件的一个分片：

![image-20200514101901437](文件服务.assets/image-20200514101901437.png)

注：由于目前后端服务还没有开发，所以上传会失败。

#### 5.10.3 后端代码实现

##### 5.10.3.1 接口文档

![image-20200513191533974](文件服务.assets/image-20200513191533974.png)

![image-20200513191617164](文件服务.assets/image-20200513191617164.png)

##### 5.10.3.2 代码开发

第一步：创建FileChunkController并提供分片上传方法uploadFile

~~~java
package com.itheima.pinda.file.controller;

import com.itheima.pinda.base.BaseController;
import com.itheima.pinda.base.R;
import com.itheima.pinda.dozer.DozerUtils;
import com.itheima.pinda.file.domain.FileAttrDO;
import com.itheima.pinda.file.dto.chunk.FileChunksMergeDTO;
import com.itheima.pinda.file.dto.chunk.FileUploadDTO;
import com.itheima.pinda.file.entity.File;
import com.itheima.pinda.file.manager.WebUploader;
import com.itheima.pinda.file.properties.FileServerProperties;
import com.itheima.pinda.file.service.FileService;
import com.itheima.pinda.file.strategy.FileChunkStrategy;
import com.itheima.pinda.file.strategy.FileStrategy;
import com.itheima.pinda.file.utils.FileDataTypeUtil;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
/**
 * 分片上传
 */
@RestController
@Slf4j
@RequestMapping("/chunk")
@CrossOrigin
@Api(value = "分片上传", tags = "分片上传，需要webuploder.js插件进行配合使用")
public class FileChunkController extends BaseController {
    @Autowired
    private FileServerProperties fileProperties;
    @Autowired
    private FileService fileService;
    @Autowired
    private FileStrategy fileStrategy;
    @Autowired
    private WebUploader webUploader;
    @Autowired
    private DozerUtils dozerUtils;
    /**
     * 分片上传
     * @param fileUploadDTO
     * @param multipartFile
     * @return
     */
    @ApiOperation(value = "分片上传", notes = "分片上传")
    @PostMapping(value = "/upload")
    public R<FileChunksMergeDTO> uploadFile(FileUploadDTO fileUploadDTO,
                                            @RequestParam(value = "file", required = false) MultipartFile multipartFile) throws Exception {

        if (multipartFile == null || multipartFile.isEmpty()) {
            log.error("分片上传分片为空");
            return fail("分片上传分片为空");
        }

        //  存放分片文件的服务器绝对路径 ，例如 D:\\uploadfiles\\2020\\04
        String uploadFolder = FileDataTypeUtil.getUploadPathPrefix(fileProperties.getStoragePath());

        if (fileUploadDTO.getChunks() == null || fileUploadDTO.getChunks() <= 0) {
            //没有分片，按照普通文件上传处理
            File file = fileStrategy.upload(multipartFile);
            file.setFileMd5(fileUploadDTO.getMd5());
            
            fileService.save(file);

            return success(null);
        } else {
            //为上传的文件准备好对应的位置
            java.io.File targetFile = webUploader.getReadySpace(fileUploadDTO, uploadFolder);

            if (targetFile == null) {
                return fail("分片上传失败");
            }
            //保存上传文件
            multipartFile.transferTo(targetFile);

            //封装信息给前端，用于分片合并
            FileChunksMergeDTO mergeDTO = new FileChunksMergeDTO();
            mergeDTO.setSubmittedFileName(multipartFile.getOriginalFilename());
            dozerUtils.map(fileUploadDTO,mergeDTO);

            return success(mergeDTO);
        }
    }
}
~~~

第二步：在配置属性类中添加storagePath属性和对于的get、set方法

~~~java
public String getStoragePath() {
    return storagePath;
}

public void setStoragePath(String storagePath) {
    this.storagePath = storagePath;
}

//指定分片上传时临时存放目录
private String storagePath ;
~~~

第三步：创建WebUploader分片上传工具类

~~~java
package com.itheima.pinda.file.manager;

import com.itheima.pinda.file.dto.chunk.FileUploadDTO;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import java.io.IOException;
/**
 * 分片上传工具类
 */
@Service
@Slf4j
public class WebUploader2 {
    /**
     * 为上传的文件创建对应的保存位置,若上传的是分片，则会创建对应的文件夹结构和tmp文件
     *
     * @param fileUploadDTO 上传文件的相关信息
     * @param path 文件保存根路径
     * @return
     */
    public java.io.File getReadySpace(FileUploadDTO fileUploadDTO, String path) {
        //创建上传文件所需的文件夹
        if (!this.createFileFolder(path, false)) {
            return null;
        }

        //将上传的分片保存在此目录中
        String fileFolder = fileUploadDTO.getName();

        if (fileFolder == null) {
            return null;
        }

        //文件上传路径更新为指定文件信息签名后的临时文件夹，用于后期合并
        path += "/" + fileFolder;

        if (!this.createFileFolder(path, true)) {
            return null;
        }

        //分片上传，指定当前分片文件的文件名
        String newFileName = String.valueOf(fileUploadDTO.getChunk());
        return new java.io.File(path, newFileName);
    }

    /**
     * 创建存放分片上传的文件的文件夹
     *
     * @param file   文件夹路径
     * @param hasTmp 是否有临时文件
     * @return
     */
    private boolean createFileFolder(String file, boolean hasTmp) {
        //创建存放分片文件的临时文件夹
        java.io.File tmpFile = new java.io.File(file);
        if (!tmpFile.exists()) {
            try {
                tmpFile.mkdirs();
            } catch (SecurityException ex) {
                log.error("无法创建文件夹", ex);
                return false;
            }
        }

        if (hasTmp) {
            //创建临时文件，用来记录上传分片文件的修改时间，用于清理长期未完成的垃圾分片
            tmpFile = new java.io.File(file + ".tmp");
            if (tmpFile.exists()) {
                return tmpFile.setLastModified(System.currentTimeMillis());
            } else {
                try {
                    tmpFile.createNewFile();
                } catch (IOException ex) {
                    log.error("无法创建tmp文件", ex);
                    return false;
                }
            }
        }
        return true;
    }
}
~~~

第四步：修改Nacos配置中心的pd-file-server.yml文件，加入storagePath配置项

##### 5.10.3.3 接口测试

第一步：启动Nacos配置中心

第二步：启动Nginx服务

第三步：启动文件服务

第四步：访问分片上传页面，进行大文件上传

可以看到，上传完成后，对应的分片上传所需目录、临时文件、分片文件都已经创建成功了：

![image-20200514185803364](文件服务.assets/image-20200514185803364.png)

![](文件服务.assets/image-20200514185803364.png)

![image-20200514185825012](文件服务.assets/image-20200514185825012.png)

### 5.11 接口开发-分片合并

前面我们已经完成了分片上传的接口，本小节需要完成的是将这些分片文件合并为原始文件并按照配置文件配置的存储策略保存到相应位置。由于不同的存储方式对应的分片合并方式也不同，所以我们需要提供不同的分片合并处理策略。具体接口设计如下：

![image-20200518110627666](文件服务.assets/image-20200518110627666.png)

#### 5.11.1 FileChunkStrategy

FileChunkStrategy是分片文件处理策略顶层接口，是对分片文件处理的顶层抽象，具体代码如下：

~~~java
package com.itheima.pinda.file.strategy;

import com.itheima.pinda.base.R;
import com.itheima.pinda.file.dto.chunk.FileChunksMergeDTO;
import com.itheima.pinda.file.entity.File;
/**
 * 文件分片处理策略接口
 */
public interface FileChunkStrategy {
    /**
     * 分片合并
     *
     * @param merge
     * @return
     */
    R<File> chunksMerge(FileChunksMergeDTO merge);
}
~~~

#### 5.11.2 AbstractFileChunkStrategy

AbstractFileChunkStrategy是抽象分片策略处理类，实现了FileChunkStrategy接口。AbstractFileChunkStrategy实现主要的分片合并处理流程，例如：分片临时存储路径获取、分片数量的检查、合并后临时分片文件清理、合并后将文件信息保存到数据库等，但是真正分片合并的处理过程需要其子类来完成，因为不同的存储方案处理方式是不同的。

由于在进行分片合并处理过程中需要锁，在资料中(`文件服务\资料\分片上传\后端`)已经提供了工具类，直接导入项目使用即可。

AbstractFileChunkStrategy代码如下：

~~~java
package com.itheima.pinda.file.strategy.impl;

import com.itheima.pinda.base.R;
import com.itheima.pinda.file.dto.chunk.FileChunksMergeDTO;
import com.itheima.pinda.file.entity.File;
import com.itheima.pinda.file.enumeration.IconType;
import com.itheima.pinda.file.properties.FileServerProperties;
import com.itheima.pinda.file.service.FileService;
import com.itheima.pinda.file.strategy.FileChunkStrategy;
import com.itheima.pinda.file.utils.FileLock;
import com.itheima.pinda.file.utils.FileDataTypeUtil;
import com.itheima.pinda.utils.DateUtils;
import com.itheima.pinda.utils.NumberHelper;
import com.itheima.pinda.utils.StrPool;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.io.FileUtils;
import org.springframework.beans.factory.annotation.Autowired;
import java.io.IOException;
import java.nio.file.Paths;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.locks.Lock;
/**
 * 文件分片处理 抽象策略类
 */
@Slf4j
public abstract class AbstractFileChunkStrategy implements FileChunkStrategy {
    @Autowired
    protected FileService fileService;
    @Autowired
    protected FileServerProperties fileProperties;

    protected FileServerProperties.Properties properties;

    /**
     * 分片合并
     * @param info
     * @return
     */
    @Override
    public R<File> chunksMerge(FileChunksMergeDTO info) {
        //   570de89d476e6a5ba371f5fdd8d7920b.avi
        String filename = new StringBuilder(info.getName()).append(StrPool.DOT).append(info.getExt()).toString();
        //分片合并
        R<File> result = chunksMerge(info, filename);

        if (result.getIsSuccess() && result.getData() != null) {
            //文件名
            File filePo = result.getData();

            LocalDateTime now = LocalDateTime.now();
            filePo.setDataType(FileDataTypeUtil.getDataType(info.getContextType()))
                    .setCreateMonth(DateUtils.formatAsYearMonthEn(now))
                    .setCreateWeek(DateUtils.formatAsYearWeekEn(now))
                    .setCreateDay(DateUtils.formatAsDateEn(now))
                    .setSubmittedFileName(info.getSubmittedFileName())
                    .setIsDelete(false)
                    .setSize(info.getSize())
                    .setFileMd5(info.getMd5())
                    .setContextType(info.getContextType())
                    .setFilename(filename)
                    .setExt(info.getExt())
                    .setIcon(IconType.getIcon(info.getExt()).getIcon());

            //将上传的文件信息保存到数据库
            fileService.save(filePo);
            return R.success(filePo);
        }
        return result;
    }

    /**
     * 分片合并
     * @param info
     * @param fileName
     * @return
     */
    private R<File> chunksMerge(FileChunksMergeDTO info, String fileName) {
        //获得分片文件存储的路径 D:\\chunks\\2020\\05
        String path = FileDataTypeUtil.getUploadPathPrefix(fileProperties.getStoragePath());
        int chunks = info.getChunks();
        String folder = info.getName();
        String md5 = info.getMd5();
        int chunksNum = this.getChunksNum(Paths.get(path, folder).toString());

        //检查是否满足合并条件：分片数量是否足够
        if (chunks == chunksNum) {
            //同步指定合并的对象
            Lock lock = FileLock.getLock(folder);
            try {
                lock.lock();
                //检查是否满足合并条件：分片数量是否足够
                List<java.io.File> files = new ArrayList<>(Arrays.asList(this.getChunks(Paths.get(path, folder).toString())));
                if (chunks == files.size()) {
                    //按照名称排序文件，这里分片都是按照数字命名的

                    //这里存放的文件名一定是数字
                    files.sort((f1, f2) -> NumberHelper.intValueOf0(f1.getName()) - NumberHelper.intValueOf0(f2.getName()));

                    R<File> result = merge(files, fileName, info);

                    //清理：文件夹，tmp文件
                    this.cleanSpace(folder, path);
                    return result;
                }
            } catch (Exception ex) {
                log.error("数据分片合并失败", ex);
                return R.fail("数据分片合并失败");
            } finally {
                //解锁
                lock.unlock();
                //清理锁对象
                FileLock.removeLock(folder);
            }
        }

        log.error("文件[签名:" + md5 + "]数据不完整，可能该文件正在合并中");
        return R.fail("数据不完整，可能该文件正在合并中, 也有可能是上传过程中某些分片丢失");
    }

    /**
     * 子类实现具体的合并操作
     *
     * @param files    分片文件
     * @param fileName 唯一名 含后缀
     * @param info     分片信息
     * @return
     * @throws IOException
     */
    protected abstract R<File> merge(List<java.io.File> files,  String fileName, FileChunksMergeDTO info) throws IOException;

    /**
     * 清理分片上传的相关数据
     * 文件夹，tmp文件
     *
     * @param folder 文件夹名称
     * @param path   上传文件根路径
     * @return
     */
    protected boolean cleanSpace(String folder, String path) {
        //删除分片文件夹
        java.io.File garbage = new java.io.File(Paths.get(path, folder).toString());
        if (!FileUtils.deleteQuietly(garbage)) {
            return false;
        }
        //删除tmp文件
        garbage = new java.io.File(Paths.get(path, folder + ".tmp").toString());
        if (!FileUtils.deleteQuietly(garbage)) {
            return false;
        }
        return true;
    }

    /**
     * 获取指定文件的分片数量
     *
     * @param folder 文件夹路径
     * @return
     */
    private int getChunksNum(String folder) {
        java.io.File[] filesList = this.getChunks(folder);
        return filesList.length;
    }

    /**
     * 获取指定文件的所有分片
     *
     * @param folder 文件夹路径
     * @return
     */
    private java.io.File[] getChunks(String folder) {
        java.io.File targetFolder = new java.io.File(folder);
        return targetFolder.listFiles((file) -> {
            if (file.isDirectory()) {
                return false;
            }
            return true;
        });
    }
}
~~~

#### 5.11.3 LocalChunkServiceImpl

LocalChunkServiceImpl是AbstractFileChunkStrategy的子类，负责处理存储策略为本地时的分片文件合并操作。为了使程序能够动态选择具体的策略处理类，可以将LocalChunkServiceImpl定义在LocalAutoConfigure配置类中，具体代码如下：

~~~java
/**
* 本地分片文件处理策略类
*/
@Service
public class LocalChunkServiceImpl extends AbstractFileChunkStrategy {
    /**
         *分片合并
         * @param files    分片文件
         * @param fileName 唯一名 含后缀
         * @param info     分片信息
         * @return
         * @throws IOException
     */
    @Override
    protected R<File> merge(List<java.io.File> files, String fileName, FileChunksMergeDTO info) throws IOException {
        properties = fileProperties.getLocal();

        //日期目录
        String relativePath = Paths.get(LocalDate.now().format(DateTimeFormatter.ofPattern(DateUtils.DEFAULT_MONTH_FORMAT_SLASH))).toString();

        //合并后文件的存储路径 例如：D:\\uploadFiles\\oss-file-service\\2020\\05
        String path = Paths.get(properties.getEndpoint(), properties.getBucketName(), relativePath).toString();

        //上传文件存放目录，如果不存在则创建
        java.io.File uploadFolder = new java.io.File(path);
        if(!uploadFolder.exists()){
            uploadFolder.mkdirs();
        }

        //创建合并后的文件
        java.io.File outputFile = new java.io.File(Paths.get(path, fileName).toString());
        if (!outputFile.exists()) {
            boolean newFile = outputFile.createNewFile();
            if (!newFile) {
                return R.fail("创建文件失败");
            }
            try (FileChannel outChannel = new FileOutputStream(outputFile).getChannel()) {
                //同步nio 方式对分片进行合并, 有效的避免文件过大导致内存溢出
                for (java.io.File file : files) {
                    try (FileChannel inChannel = new FileInputStream(file).getChannel()) {
                        inChannel.transferTo(0, inChannel.size(), outChannel);
                    } catch (FileNotFoundException ex) {
                        log.error("文件转换失败", ex);
                        return R.fail("文件转换失败");
                    }
                    //删除分片
                    if (!file.delete()) {
                        log.error("分片[" + info.getName() + "=>" + file.getName() + "]删除失败");
                    }
                }
            } catch (FileNotFoundException e) {
                log.error("文件输出失败", e);
                return R.fail("文件输出失败");
            }

        } else {
            log.warn("文件[{}], fileName={}已经存在", info.getName(), fileName);
        }

        String url = new StringBuilder(properties.getUriPrefix()).
                    append(bucketName).append(StrPool.SLASH).
                    append(relativePath).append(StrPool.SLASH).
                    append(fileName).toString();
        File filePo = File.builder()
            .relativePath(relativePath)
            .url(StringUtils.replace(url, "\\", StrPool.SLASH))
            .build();
        return R.success(filePo);
    }
}
~~~

#### 5.11.4 FastDfsChunkServiceImpl

FastDfsChunkServiceImpl是AbstractFileChunkStrategy的子类，负责处理存储策略为FastDFS时的分片文件合并操作。为了使程序能够动态选择具体的策略处理类，可以将FastDfsChunkServiceImpl定义在FastDfsAutoConfigure配置类中，具体代码如下：

~~~java
/**
* FastDfs分片文件处理策略类
*/
@Service
public class FastDfsChunkServiceImpl extends AbstractFileChunkStrategy {
    @Autowired
    protected AppendFileStorageClient storageClient;

    /**
         * 分片合并
         * @param files    分片文件
         * @param fileName 唯一名 含后缀
         * @param info     分片信息
         * @return
         * @throws IOException
    */
    @Override
    protected R<File> merge(List<java.io.File> files, String fileName, FileChunksMergeDTO info) throws IOException {
        StorePath storePath = null;

        for (int i = 0; i < files.size(); i++) {
            java.io.File file = files.get(i);

            FileInputStream in = FileUtils.openInputStream(file);
            if (i == 0) {
                storePath = storageClient.uploadAppenderFile(null, in,
                                                             file.length(), info.getExt());
            } else {
                storageClient.appendFile(storePath.getGroup(), storePath.getPath(),
                                         in, file.length());
            }
        }
        if (storePath == null) {
            return R.fail("上传失败");
        }

        String url = new StringBuilder(fileProperties.getUriPrefix())
            .append(storePath.getFullPath())
            .toString();
        File filePo = File.builder()
            .url(url)
            .group(storePath.getGroup())
            .path(storePath.getPath())
            .build();
        return R.success(filePo);
    }
}
~~~

#### 5.11.5 AliChunkServiceImpl

AliChunkServiceImpl是AbstractFileChunkStrategy的子类，负责处理存储策略为阿里云OSS时的分片文件合并操作。为了使程序能够动态选择具体的策略处理类，可以将AliChunkServiceImpl定义在AliOssAutoConfigure配置类中，具体代码如下：

~~~java
/**
* 阿里云OSS分片文件处理策略类
*/
@Service
public class AliChunkServiceImpl extends AbstractFileChunkStrategy {
    private OSS buildClient() {
        properties = fileProperties.getAli();
        return new OSSClientBuilder().build(properties.getEndpoint(), properties.getAccessKeyId(),
                                            properties.getAccessKeySecret());
    }

    /**
         * 分片合并
         * @param files    分片文件
         * @param fileName 唯一名 含后缀
         * @param info     分片信息
         * @return
         * @throws IOException
    */
    @Override
    protected R<File> merge(List<java.io.File> files, String fileName, FileChunksMergeDTO info) throws IOException {
        OSS client = buildClient();
        String bucketName = properties.getBucketName();

        //日期文件夹
        String relativePath = LocalDate.now().format(DateTimeFormatter.ofPattern(DEFAULT_MONTH_FORMAT_SLASH));
        // web服务器存放的相对路径
        String relativeFileName = relativePath + StrPool.SLASH + fileName;

        ObjectMetadata metadata = new ObjectMetadata();
        metadata.setContentDisposition("attachment;fileName=" + info.getSubmittedFileName());
        metadata.setContentType(info.getContextType());
        //步骤1：初始化一个分片上传事件。
        InitiateMultipartUploadRequest request = new InitiateMultipartUploadRequest(bucketName, relativeFileName, metadata);
        InitiateMultipartUploadResult result = client.initiateMultipartUpload(request);
        // 返回uploadId，它是分片上传事件的唯一标识，您可以根据这个ID来发起相关的操作，如取消分片上传、查询分片上传等。
        String uploadId = result.getUploadId();

        // partETags是PartETag的集合。PartETag由分片的ETag和分片号组成。
        List<PartETag> partETags = new ArrayList<PartETag>();
        for (int i = 0; i < files.size(); i++) {
            java.io.File file = files.get(i);
            FileInputStream in = FileUtils.openInputStream(file);

            UploadPartRequest uploadPartRequest = new UploadPartRequest();
            uploadPartRequest.setBucketName(bucketName);
            uploadPartRequest.setKey(relativeFileName);
            uploadPartRequest.setUploadId(uploadId);
            uploadPartRequest.setInputStream(in);
            // 设置分片大小。除了最后一个分片没有大小限制，其他的分片最小为100KB。
            uploadPartRequest.setPartSize(file.length());
            // 设置分片号。每一个上传的分片都有一个分片号，取值范围是1~10000，如果超出这个范围，OSS将返回InvalidArgument的错误码。
            uploadPartRequest.setPartNumber(i + 1);

            // 每个分片不需要按顺序上传，甚至可以在不同客户端上传，OSS会按照分片号排序组成完整的文件。
            UploadPartResult uploadPartResult = client.uploadPart(uploadPartRequest);

            // 每次上传分片之后，OSS的返回结果会包含一个PartETag。PartETag将被保存到partETags中。
            partETags.add(uploadPartResult.getPartETag());
        }

        /* 步骤3：完成分片上传。 */
        // 排序。partETags必须按分片号升序排列。
        partETags.sort(Comparator.comparingInt(PartETag::getPartNumber));

        // 在执行该操作时，需要提供所有有效的partETags。OSS收到提交的partETags后，会逐一验证每个分片的有效性。当所有的数据分片验证通过后，OSS将把这些分片组合成一个完整的文件。
        CompleteMultipartUploadRequest completeMultipartUploadRequest =
            new CompleteMultipartUploadRequest(bucketName, relativeFileName, uploadId, partETags);

        CompleteMultipartUploadResult uploadResult = client.completeMultipartUpload(completeMultipartUploadRequest);

        String url = new StringBuilder(properties.getUriPrefix())
            .append(relativePath)
            .append(StrPool.SLASH)
            .append(fileName)
            .toString();
        File filePo = File.builder()
            .relativePath(relativePath)
            .group(uploadResult.getETag())
            .path(uploadResult.getRequestId())
            .url(StringUtils.replace(url, "\\", StrPool.SLASH))
            .build();

        // 关闭OSSClient。
        client.shutdown();
        return R.success(filePo);
    }
}
~~~

#### 5.11.6 分片合并接口

接口文档：

![image-20200518114353207](文件服务.assets/image-20200518114353207.png)

![image-20200518114411802](文件服务.assets/image-20200518114411802.png)



在FileChunkController中提供分片合并方法，直接调用分片处理策略类完成分片合并操作：

~~~java
@Autowired
private FileChunkStrategy fileChunkStrategy;//分片文件处理策略

/**
* 分片合并
* @param info
* @return
*/
@ApiOperation(value = "分片合并", notes = "所有分片上传成功后，调用该接口对分片进行合并")
@PostMapping(value = "/merge")
public R<File> saveChunksMerge(FileChunksMergeDTO info) {
    log.info("info={}", info);

    return fileChunkStrategy.chunksMerge(info);
}
~~~

