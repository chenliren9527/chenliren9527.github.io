---
title: Elasticsearch_linux安装
tags:
  - 笔记
  - 项目实战二前置课程
  - Elasticsearch
categories:
  - 项目实战二前置课程
date: 2020-12-13 13:41:19
---

# elasticsearch Lunix安装

1：下载elasticsearch-6.6.2.tar.gz

![1571756858018](elasticsearch-linux安装.assets/1571756858018.png)

2：解压

```properties
cd /opt
tar -zxvf elasticsearch-6.6.2.tar.gz
```

3：修改config/elasticsearch.yml文件

```
network.host: 192.168.153.147
http.port: 9200
http.cors.enabled: true
http.cors.allow-origin: "*"
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
```

4：启动elasticsearch

```properties
bin/elasticsearch
```

![1571757122567](elasticsearch-linux安装.assets/1571757122567.png)

解决方案：

这是出于系统安全考虑设置的条件。由于ElasticSearch可以接收用户输入的脚本并且执行，为了系统安全考虑， 建议创建一个单独的用户用来运行ElasticSearch。创建elsearch用户组及elsearch用户

```properties
groupadd elsearch
useradd elsearch -g elsearch -p elasticsearch
```

更改elasticsearch文件夹及内部文件的所属用户及组为elsearch:elsearch

```properties
cd /opt
chown -R elsearch:elsearch  elasticsearch-6.6.2
```

5：切换用户到`elsearch`中，执启动

```properties
bin/elasticsearch
```

![1571757873370](elasticsearch-linux安装.assets/1571757873370.png)

启动成功! 浏览器访问：`http://192.168.11.111:9200/`

![1571757907337](elasticsearch-linux安装.assets/1571757907337.png)

如果访问失败请记得关闭防火墙

```properties
# 停止firewall
systemctl stop firewalld.service
# 禁止firewall开机启动
systemctl disable firewalld.service 
```









# elasticsearch启动常见的几个报错

2019.02.12 16:24:31字数 242阅读 4401

[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
切换到root用户，编辑limits.conf添加如下内容

```properties
vi /etc/security/limits.conf
* soft nofile 65536
* hard nofile 65536
* soft nproc 4096
* hard nproc 4096
```

[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
修改/etc/sysctl.conf文件，增加配置vm.max_map_count=262144

```properties
vi /etc/sysctl.conf
vm.max_map_count=262144
sysctl -p
```

执行命令sysctl -p生效

[3]: system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk
问题原因：因为Centos6不支持SecComp，而ES5.2.1默认bootstrap.system_call_filter为true进行检测，所以导致检测失败，失败后直接导致ES不能启动
解决方法：在elasticsearch.yml中配置bootstrap.system_call_filter为false，注意要在Memory下面:

```properties
vim elasticsearch.yml
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
```

