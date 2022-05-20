---
title: Elasticsearch_win安装
tags:
  - 笔记
  - 项目实战二前置课程
  - Elasticsearch
categories:
  - 项目实战二前置课程
abbrlink: 5172
date: 2020-12-13 13:41:27
---

##### 注意

> ElasticSearch是使用java开发的，且本版本的es需要的jdk版本要是1.8以上，所以安装ElasticSearch 之前保证JDK1.8+安装完毕，并正确的配置好JDK环境变量，否则启动ElasticSearch失败。

##### 下载

ElasticSearch的官方地址： https://www.elastic.co/products/elasticsearch

![1572261702892](elasticsearch-win安装.assets/1572261702892.png) 

linux下载：https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.6.2.tar.gz

win下载：https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.6.2.zip







##### 服务器安装步骤

lasticSearch分为Linux和Window版本，基于我们主要学习的是ElasticSearch的Java客户端的使用，所以我们课程中使用的是安装较为简便的Window版本，项目上线后，公司的运维人员会安装Linux版的ES供我们连接使用。

> 1：下载ES压缩包
> 2：启动ES服务端
> 3：安装ES的图形化界面插件客户端
>
> ​	3-1：先解压`elasticsearch-head-master.zip` 文件
> ​	3-2：把解压的内容复制到tomcat的webapps下的ROOT目录下（先把ROOT删除干净）.
> ​	3-3：然后启动tomcat。bin/startup.bat
> ​	3-4:  打开浏览器，输入 http://localhost:8080

具体实现如下：

1：在资料中已经提供了下载好的6.6.2的压缩包：`elasticsearch-6.6.2.zip`。进行解压。如下：

![1572261901519](elasticsearch-win安装.assets/1572261901519.png) 

2：点击ElasticSearch下的bin目录下的elasticsearch.bat启动，控制台显示的日志：

![1572261981080](elasticsearch-win安装.assets/1572261981080.png) 

![1572262010829](elasticsearch-win安装.assets/1572262010829.png)



然后在浏览器访问：`http://localhost:9200`  得到如下信息，说明安装成功了:

```json
{
  "name": "bCqCwDk",
  "cluster_name": "elasticsearch",
  "cluster_uuid": "3UnrQY63SgqSsqnVdJdytg",
  "version": {
    "number": "6.6.2",
    "build_flavor": "default",
    "build_type": "zip",
    "build_hash": "3bd3e59",
    "build_date": "2019-03-06T15:16:26.864148Z",
    "build_snapshot": false,
    "lucene_version": "7.6.0",
    "minimum_wire_compatibility_version": "5.6.0",
    "minimum_index_compatibility_version": "5.0.0"
  },
  "tagline": "You Know, for Search"
}
```







##### 客户端安装步骤

```
3：安装ES的图形化界面插件客户端

	3-1：先解压elasticsearch-head-master.zip 文件
	3-2：把解压的内容复制到tomcat的webapps下的ROOT目录下（先把ROOT删除干净）.
	3-3：然后启动tomcat。bin/startup.bat
	3-4:  打开浏览器，输入 http://localhost:8081 (如果出现端口冲突，可以修改conf/server.xml中的端口为别的端口。)
```

大家把下面这个压缩包解压就可以运行了：

![1572262185106](elasticsearch-win安装.assets/1572262185106.png) 

这个tomcat的端口是8081，也可以自己去修改。

![1572262289551](elasticsearch-win安装.assets/1572262289551.png) 





==注意：Elasticsearch的客户端默认连接不上服务端的，会报以下错误：==

![1572262364959](elasticsearch-win安装.assets/1572262364959.png) 

原因：Elasticsearch的客户端  跨域 访问  服务端 ，产生跨域错误。

解决办法:

在Elasticsearch的配置文件添加跨域配置：

```properties
#跨域问题
http.cors.enabled: true
http.cors.allow-origin: "*"
```

![1572262461466](elasticsearch-win安装.assets/1572262461466.png) 







**小结**

==1：注：完整的安装好的客户端在 资料 > apache-tomcat-8.5.42-配置好的客户端master==直接启动即可。

2：端口不同互相访问会出现跨域的问题，es服务/config/elasticsearch.yml 需要如下配置：

```properties
http.cors.enabled: true
http.cors.allow-origin: "*"
```

配置完毕以后记得重启。