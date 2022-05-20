---
title: 就业技术加强(05)-Linux安装zookeeper(单机版)
tags:
  - 笔记
  - 就业技术加强
  - Linux
  - zookeeper
  - 单机
categories:
  - 就业技术加强
abbrlink: 28816
date: 2021-01-28 22:57:53
---



### 1、安装说明

**下载地址**：http://www.apache.org/dyn/closer.cgi/zookeeper

**安装环境**

`CentOS 7.0+` `Jdk1.8+`

> 如果安装过docker；那么在CentOS中查看Ip的时候；前缀是172.16.\*.\* 类似的ip地址的时候；可以重启系统网络；执行如下命令：
>
> systemctl restart network

### 2、安装步骤

#### 2.1、上传安装包

```shell
# 创建目录
mkdir -p /usr/local/zookeeper
cd /usr/local/zookeeper

# 将 资料\zookeeper-3.4.14.tar.gz 上传到CentOS系统的 /usr/local/zookeeper 目录下

```

#### 2.2、解压

```shell
# 解压
tar -zxvf zookeeper-3.4.14.tar.gz
```

#### 2.3、创建存放数据目录

```shell
# 创建存放zookeeper的数据目录
mkdir /usr/local/zookeeper/zookeeper-3.4.14/data
```



#### 2.4、修改配置

```shell
# 复制配置
cd /usr/local/zookeeper/zookeeper-3.4.14/conf
cp zoo_sample.cfg zoo.cfg

#修改配置文件
vi zoo.cfg

#将第12 行的存放数据的目录修改为前面创建的目录 dataDir=/tmp/zookeeper 修改为如下
dataDir=/usr/local/zookeeper/zookeeper-3.4.14/data

```

![image-20201225163552123](../../../../Documents/Java笔记/就业班/12.就业技术加强/05-DUBBO&Zookeeper/资料/assets/image-20201225163552123.png)

#### 2.5、启动

```shell
# 启动
/usr/local/zookeeper/zookeeper-3.4.14/bin/zkServer.sh start

# 查看状态
/usr/local/zookeeper/zookeeper-3.4.14/bin/zkServer.sh status
```



### 3、关闭防火墙

zookeeper使用2181端口号，为了能对外正常使用zookeeper，需要开放2181端口号，或者关闭防火墙。

```shell
# 对外开放2181端口
firewall-cmd --zone=public --add-port=2181/tcp --permanent

–zone：作用域
–add-port=2181/tcp：添加端口，格式为：端口/通讯协议
–permanent：永久生效，没有此参数重启后失效

#重启防火墙
firewall-cmd --reload

# 查看已经开放的端口
firewall-cmd --list-ports

#停止防火墙
systemctl stop firewalld.service

#禁止防火墙开机启动
systemctl disable firewalld.service
```



### 4、注意事项

如果使用DUBBO的时候将linux中的zookeeper作为注册中心使用的话；那么在项目启动的时候连接zookeeper会出现超时报如下的错误：

```
Caused by: java.lang.IllegalStateException: zookeeper not connected
	at org.apache.dubbo.remoting.zookeeper.curator.CuratorZookeeperClient.<init>(CuratorZookeeperClient.java:80) ~[dubbo-2.7.8.jar:2.7.8]
	... 38 common frames omitted
```

需要在项目的配置中设置一下zookeeper的连接超时时间（默认3000，可以设置为10000）：

![image-20201225165903034](assets/image-20201225165903034.png)