---
title: 就业技术加强(05)-Linux安装zookeeper(集群版)
tags:
  - 笔记
  - 就业技术加强
  - Linux
  - zookeeper
  - 集群
categories:
  - 就业技术加强
abbrlink: 30281
date: 2021-01-28 22:58:04
---



## 1、安装说明

**下载地址**：http://www.apache.org/dyn/closer.cgi/zookeeper

**安装环境**

`CentOS 7.0+` `Jdk1.8+`

> 如果安装过docker；那么在CentOS中查看Ip的时候；前缀是172.16.\*.\* 类似的ip地址的时候；可以重启系统网络；执行如下命令：
>
> systemctl restart network



**集群说明**：

zookeeper的集群需要有至少一般3台独立的zookeeper服务器组成；下面的安装以3台zookeeper组成zookeeper集群为例进行安装说明。如果有更多也是类似的步骤。



## 2、安装步骤

### 2.1、安装配置单台

#### 1）上传安装包

```shell
# 创建目录
mkdir -p /usr/local/zookeeper
cd /usr/local/zookeeper

# 将 资料\zookeeper-3.4.14.tar.gz 上传到CentOS系统的 /usr/local/zookeeper 目录下

```

#### 2）解压

```shell
# 创建目录
mkdir zookeeper-cluster
# 解压
tar -zxvf zookeeper-3.4.14.tar.gz -C ./zookeeper-cluster

#重命名
cd zookeeper-cluster
mv zookeeper-3.4.14 zookeeper01
```

#### 3）创建数据目录及节点ID

```shell
# 创建存放zookeeper的数据目录
mkdir /usr/local/zookeeper/zookeeper-cluster/zookeeper01/data

# 创建zookeeper集群节点的id
vi /usr/local/zookeeper/zookeeper-cluster/zookeeper01/data/myid

#设置里面的内容为 1
```

![image-20201225181042380](就业技术加强-05-Linux安装zookeeper-集群版.assets/image-20201225181042380.png)



#### 4）修改配置

```shell
# 复制配置
cd /usr/local/zookeeper/zookeeper-cluster/zookeeper01/conf
cp zoo_sample.cfg zoo.cfg

#修改配置文件
vi zoo.cfg

#将第12 行的存放数据的目录修改为前面创建的目录 dataDir=/tmp/zookeeper 修改为如下
dataDir=/usr/local/zookeeper/zookeeper-cluster/zookeeper01/data

#修改第14行端口号将原来2181 修改为 3181
clientPort=3181

#在文档最后添加如下信息。格式说明：server.myid文件中的编号=ip:启动监听端口:选举端口
server.1=192.168.12.135:2881:3881
server.2=192.168.12.135:2882:3882
server.3=192.168.12.135:2883:3883
```

![image-20201225181354248](就业技术加强-05-Linux安装zookeeper-集群版.assets/image-20201225181354248.png)



### 2.2、配置其它两台

#### 1）复制节点

```shell
# 复制刚刚配置的第一个集群节点
cd /usr/local/zookeeper/zookeeper-cluster
cp -r zookeeper01 zookeeper02 && cp -r zookeeper01 zookeeper03
```

#### 2）修改zookeeper02

```shell
# 修改 vi /usr/local/zookeeper/zookeeper-cluster/zookeeper02/data/myid 的内容为2

# 打开 vi /usr/local/zookeeper/zookeeper-cluster/zookeeper02/conf/zoo.cfg 

#将第12 行的存放数据的目录修改为如下
dataDir=/usr/local/zookeeper/zookeeper-cluster/zookeeper02/data

#修改第14行端口号修改为 3182
clientPort=3182

#在文档最后添加如下信息。格式说明：server.myid文件中的编号=ip:启动监听端口:选举端口
server.1=192.168.12.135:2881:3881
server.2=192.168.12.135:2882:3882
server.3=192.168.12.135:2883:3883
```

![image-20201225181813677](就业技术加强-05-Linux安装zookeeper-集群版.assets/image-20201225181813677.png)

![image-20201225181742841](就业技术加强-05-Linux安装zookeeper-集群版.assets/image-20201225181742841.png)

#### 3）修改zookeeper03

```shell
# 修改 vi /usr/local/zookeeper/zookeeper-cluster/zookeeper03/data/myid 的内容为3

# 打开 vi /usr/local/zookeeper/zookeeper-cluster/zookeeper03/conf/zoo.cfg 

#将第12 行的存放数据的目录修改为如下
dataDir=/usr/local/zookeeper/zookeeper-cluster/zookeeper03/data

#修改第14行端口号修改为 3183
clientPort=3183

#在文档最后添加如下信息。格式说明：server.myid文件中的编号=ip:启动监听端口:选举端口
server.1=192.168.12.135:2881:3881
server.2=192.168.12.135:2882:3882
server.3=192.168.12.135:2883:3883
```

![image-20201225181847929](就业技术加强-05-Linux安装zookeeper-集群版.assets/image-20201225181847929.png)

![image-20201225181950921](就业技术加强-05-Linux安装zookeeper-集群版.assets/image-20201225181950921.png)

### 2.3、启动

```shell
/usr/local/zookeeper/zookeeper-cluster/zookeeper01/bin/zkServer.sh start
/usr/local/zookeeper/zookeeper-cluster/zookeeper02/bin/zkServer.sh start
/usr/local/zookeeper/zookeeper-cluster/zookeeper03/bin/zkServer.sh start

```



## 3、关闭防火墙

zookeeper使用3181/3182/3183端口号，为了能对外正常使用zookeeper，需要开放端口号，或者关闭防火墙。

```shell
# 对外开放端口
firewall-cmd --zone=public --add-port=3181/tcp --permanent
firewall-cmd --zone=public --add-port=3182/tcp --permanent
firewall-cmd --zone=public --add-port=3183/tcp --permanent

–zone：作用域
–add-port=3181/tcp：添加端口，格式为：端口/通讯协议
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



## 4、注意事项

如果使用DUBBO的时候将linux中的zookeeper作为注册中心使用的话；那么在项目启动的时候连接zookeeper会出现超时报如下的错误：

```
Caused by: java.lang.IllegalStateException: zookeeper not connected
	at org.apache.dubbo.remoting.zookeeper.curator.CuratorZookeeperClient.<init>(CuratorZookeeperClient.java:80) ~[dubbo-2.7.8.jar:2.7.8]
	... 38 common frames omitted
```

需要在项目的配置中设置一下zookeeper的连接超时时间（默认3000，可以设置为10000）：

![image-20201225165903034](assets/image-20201225165903034.png)