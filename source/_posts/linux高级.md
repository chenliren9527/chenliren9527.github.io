---
title: linux高级
tags:
  - 笔记
  - 项目部署
  - mysql安装
  - nginx安装
  - tomcat安装
  - redis安装
  - nginx反向代理
  - linux
categories:
  - linux
abbrlink: 4695
date: 2020-11-04 12:20:23
---



## 01.学习目标

### **学习目标**

1. 能够使用Linux中的网络管理命令

2. 能够使用Linux中SSH免密登录命令

4. 能够使用Linux中防火墙配置命令

5. 能够在linux系统中安装jdk软件

6. 能够在linux系统中安装mysql软件

7. 能够在linux系统中安装redis软件

8. 能够在linux系统中安装tomcat软件

9. 能够在linux系统的tomcat中部署发布项目

10. 能够使用nginx实现反向代理





## 02.网络管理2-网络服务管理【理解】

systemctl start|stop|restart|reload  服务名称

### 疑问

* 电脑操作系统能不能上网由什么决定？

  由网络服务是否开启决定。



### 网络服务管理命令

![1547519925832](linux高级.assets/1547519925832.png)

客户端操作

![image-20200626093310117](linux高级.assets/image-20200626093310117.png)

![image-20200626093428783](linux高级.assets/image-20200626093428783.png)

服务器操作

![image-20200626093525447](linux高级.assets/image-20200626093525447.png)

![image-20200626093707177](linux高级.assets/image-20200626093707177.png)

客户端操作

![image-20200626093804897](linux高级.assets/image-20200626093804897.png)

 如果断开了, 需要重新连接会话

### 小结

* 如果linux系统不能上网（内网和外网），需要检查什么服务？

  - 检查网络服务器是否开启。  systemctl status  network



## 03.网络管理3-网卡激活与关闭管理【理解】

### 目标

操作每个网络的管理（网卡的管理）



### 查看网卡的名字

通过ifconfig可以查看网卡列表

```
ifconfig
```

![image-20200308093322742](linux高级.assets/image-20200308093322742.png)

所有linux系统的==网卡都有对应的配置文件==：/etc/sysconfig/network-scripts/目录下

### 关闭网卡实现步骤

1. 找到网卡配置文件ens33

   ![1547520503417](linux高级.assets/1547520503417.png)

2. 编辑网卡配置文件

   ```shell
   vim ifcfg-ens33
   ```

   配置文件信息

   ![image-20200308093623204](linux高级.assets/image-20200308093623204.png)

3. 修改配置ONBOOT，修改为关闭网卡

   ONBOOT属性用于管理网卡的启动或关闭

   ONBOOT=yes 代表激活网卡，启动网卡

   ONBOOT=no 代表关闭网卡

   ```properties
   ONBOOT=no
   ```

4. 重启网络服务才可以识别最新的修改

   ```
   systemctl restart network
   ```

   

5. 测试是否可以联网

   ![1547520637778](linux高级.assets/1547520637778.png)&nbsp;

### 激活网卡实现步骤

 1. 修片配置文件ifcfg-ens33，开启网卡

    ```
    ONBOOT=yes
    ```

2. 重启网络服务

   ```shell
   systemctl restart network
   ```

   

### 小结

* linux系统是否可以联网（内网或外网）需要保证哪2个前提？

  * 网络服务是否开启
  * 网卡配置文件是配置是激活的状态。

  



## 04.网络管理4-配置静态ip【大家都需要配置】

### 配置静态ip的意义

* 当前我们linux的ip地址是动态分配的，是有可能某一天突然间就变化。 以后我们编写的程序，我们都需要链接linux某些服务的，如果ip地址经常发生改变，那么会导致我们的程序出错。 配置静态ip就是一个固定不变的ip地址，确保我们每次运行java程序都可以不是因为ip地址导致出现异常。

### 目标

掌握linux系统配置静态ip



### ip配置的类型

dhcp,  ip是动态分配生成的，适合个人电脑

==static, ip是静态不变的，适合服务器，ip不能变==

> 以后服务器是对外提供服务的，ip不可以变化，否则客户段找不到服务器

### windwos配置静态ip配置与配置说明

| 内容     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| ip地址   | 例如：192.168.56.106<br>ip是一个网络中的唯一标识，由网络号+主机号组成，比如上面192.168.56就是网络号（一个网络号代表一个网络），106是主机号 |
| 子网掩码 | 是用于描述ip地址中的网络号，网络号由几个数字组成，就有几个255，例如192.168.56.106，子网掩码就是为255.255.255.0 |
| 网关     | （又叫调制解调器，又叫光猫）作用用于不同网络之间的通信，使用路由器连接不同网络的网关，每个网卡都有网关，查看网卡可以获取网关（一般网卡都有默认，可以通过查看网卡的网关获取，例如：192.168.56.2） |
| DNS      | (Domain Name System)域名解析服务器，作用是根据提供的域名获取服务器真实ip。<br>`例子：我们通过域名www.baidu.com访问百度，这个域名会给到DNS服务器，DNS服务器会根据域名获取到百度真实ip地址，再进行底层的ip通信。`<br>所以如果需要虚拟机上外网需要配置DNS.常见DNS服务器有如下 ：<br>电信DNS（一般）：114.114.114.114<br>Google的DNS（较差）：8.8.8.8<br>阿里DNS（最优）： 223.5.5.5 |

> 域名解析服务器：  我们提供一个域名给到DNS， DNS就会返回域名对应真实服务器的ip回来
>
> DNS里面存储的数据结果类似如下；
>
> ​		www.baid.com   - 》  182.61.200.7
>
> ​        www.itcast.cn   -》 219238.20.100
>
> 其他DNS了解：https://www.feiniaomy.com/post/333.html

### 查询vmnet8网卡nat模式的网关信息



![1547521349089](linux高级.assets/1547521349089.png)

### linux系统配置静态ip实现步骤

修改linux的网卡配置文件ifcfg-ens33，里面设置静态ip

进入配置文件目录

```shell
cd /etc/sysconfig/network-scripts/
```

编辑配置文件

```shell
vim ifcfg-ens33
```

如图进行修改并保存

![image-20200308102616779](linux高级.assets/image-20200308102616779.png)

内容

IPADDR 配置ip地址

GATEWAY配置网关

NETMASK 配置子网掩码

DNS1配置DNS

```properties
IPADDR=192.168.179.128  （ip地址,ip地址不能乱写的，前三位必须与网关一样）GATEWAY=192.168.179.2     (网关vm8网卡是一样)NETMASK=255.255.255.0    (子网掩码)DNS1=223.5.5.5DNS2=114.114.114.114
```

3.重启网络服务

```shell
systemctl restart network
```

### 小结

* 因为dhcp是动态生成ip会导致服务器的ip经常变化，如何解决？

  - 配置静态ip

* 配置静态ip的步骤

  * 找到ifcfg-ens33网卡的配置文件
  * 修改网卡配置文件，修改dhcp修改static
  * 添加网卡的信息
  * 重启网络服务

  



## 04.网络管理5-虚拟机快照【应用】

### 思考

虚拟机由于其不稳定型，虚拟机经常通过某些配置或安装软件后导致系统不可用，一般我们的做法是重装虚拟机linux系统解决，那么这样操作很浪费时间，有没有什么好办法不用重装系统呢？

```
在系统可用（没有问题）的时候进行设置系统快照，在系统不可用的时候恢复快照就可以
```



### 目标

掌握给虚拟机linux系统设置快照

### 快照介绍

虚拟机“快照”是虚拟机磁盘文件在==某个时间点及时的复本备份==。系统崩溃或系统异常，你可以通过使用恢复到快照指定时间点系统状态。当升级应用和服务器及给它们打补丁的时候，快照是救世主。VMware 快照是 VMwareWorkstation 里的一个特色功能。

### 实现步骤

1. 生成快照

   点击如图菜单，进行管理快照

   ![1567043806053](linux高级.assets/1567043806053.png)

   新建快照，生成当前操作系统磁盘文件的复本（相当于备份一个系统）

   ![1567043829467](linux高级.assets/1567043829467.png)

   ![1567043841417](linux高级.assets/1567043841417.png)

2. 当系统不可用时，通过恢复指定快照进行恢复系统使用

   ![1567043976394](linux高级.assets/1567043976394.png)

##### 

## 05.网络管理6-克隆虚拟电脑【应用】

### 疑问

* 企业开发中经常需要准备多台一模一样的虚拟机电脑linux系统环境，那么如何根据当前的虚拟电脑快速搭建多台一样的虚拟电脑呢？

  虚拟机克隆，可以克隆出多台一模一样的虚拟电脑

### 目标

使用虚拟机的克隆快速复制出一模一样的虚拟电脑

### 实现步骤

1. 关闭当前虚拟机linux系统

2. 选中点击当前linux系统虚拟电脑，再鼠标右键如图操作

   ![1547522348624](linux高级.assets/1547522348624.png) 

   

3. 修改克隆出来的静态ip

   > 需要将克隆出来的电脑itcast修改静态ip为192.168.56.107， 要与itheima142不一样
   >
   > ![image-20200829100943644](linux高级.assets/image-20200829100943644.png)

   


### 小结

* VMware虚拟机克隆的作用是什么？

  





## 06.网络管理8-查询网络进程使用端口号【理解】

### 疑问

* 在企业开发中经常要查询软件使用的端口号，那么如何查询linux系统某一个运行软件使用的端口？

  使用查看网络进程端口号命令就可以

### 目标

掌握查看网络通信服务监听使用的端口号

### 命令语法

![1550628649970](linux高级.assets/1550628649970.png)

-u,  查询udp协议通信的程序

### 实现步骤

netstat -nutlp    #查看端口、udp、tcp、正在监听、显示程序名的所有程序

![1547525909682](linux高级.assets/1547525909682.png)

### 小结

* 查看使用tcp或udp协议进行网络通信的进程端口命令是什么？

  netstat -ntlp    只查tcp协议

  netstat -nulp   只查udp协议

  netstat -nutlp   只查udp与tcp协议



## 07.防火墙管理【理解】

### 目标

掌握防火墙服务的开启与关闭

掌握防火墙对网络服务进程的限制访问。



### 防火墙作用

是防止外界访问系统内部的程序，防火墙允许哪个软件访问外网，这个软件才可以进行网络通信，

linux系统安装的任何软件默认都不可以访问外网，必须经过防火墙的放行才可以。

### 防火墙服务的管理

![1547526158672](linux高级.assets/1547526158672.png)

### 开放端口允许外部连接

介绍

```
外网或内网需要连接到当前系统内的程序进行操作，需要 linux 系统防火墙开放程序端口，否则无法访问。
```

语法

```shell
firewall-cmd  --zone =public  --add-port =端口/tcp  --permanent
```

命令介绍

![1567044079852](linux高级.assets/1567044079852.png)

事例：假如开放 mysql 软件的端口 3306，允许外界访问

```shell
# 开放3306端口firewall-cmd --zone =public --add-port =3306/tcp --permanent# 修改了防火墙配置需要重启防火墙服务Systemctl restart firewalld
```

### 移出端口不允许外部连接

介绍

```
外网或内网不需要连接到当前系统内的程序进行操作，则 linux 系统关闭开放的程序端口。
```

语法

```shell
firewall-cmd  --zone =public  --remove-port =端口/tcp  --permanent
```

命令解释

![1567044189780](linux高级.assets/1567044189780.png)

事例：移除开放 mysql 软件的端口 3306，不允许外界访问

```shell
firewall-cmd  --zone =public  --remove-port =3306/tcp  --permanentSystemctl restart firewalld
```

### 小结

* 防火墙的服务管理

  ```shell
  systemctl start firewalld  -- 启动防火墙systemctl stop firewalld   -- 关闭防火墙systemctl status firewalld -- 查询防火墙状态systemctl restart firewalld --重启防火墙
  ```

  

* 防火墙的作用

  1、开放端口

  开放tcp与udp的端口

  ```shell
  firewall-cmd  --zone =public  --add-port =端口/tcp  --permanentfirewall-cmd  --zone =public  --add-port =端口/udp  --permanent
  ```

  

  2、移出开放端口

  ```shell
  firewall-cmd  --zone =public  --remove-port =端口/tcp  --permanentfirewall-cmd  --zone =public  --remove-port =端口/udp  --permanent
  ```

  > 注意：修改配置后需要重启服务器服务



## 08.SSH有密登录和免密登录【应用】

### 目标

理解SSH协议

理解非对称加密流程原理



### SSH协议介绍

SSH 是一种网络协议，是 Secure Shell（安全外壳协议）的缩写，用于计算机之间的加密登录。SSH 登录有

SSH协议的作用：远程登录其他的操作系统

两种验证机制：
1) 基于口令的安全验证（有密登录）
2) 基于秘钥登录的验证方式（免密登录，使用秘钥加密登录【非对称加密】）



### 登录远程服务器的语法

```
ssh ip地址
```



### 有密登录效果

![1604455004494](linux高级.assets/1604455004494.png)

### 免密登录效果



### 目标128这台机器免密登录到129的机器上实现原理流程

**公钥:**  加密的数据

**私钥：**解密数据的。

![1604455579020](linux高级.assets/1604455579020.png)



### 实现免密登录步骤

1. 在128生成一对公钥和私钥 (generate, 生成)，使用rsa算法

   ```
   ssh-keygen
   ```

2. 将公钥发送给129服务器,必须给出129的密码才接收公钥

   ```
   ssh-copy-id 192.168.179.129
   ```

3. 发送登录请求，实现免密登录

### 效果

![1604455751905](linux高级.assets/1604455751905.png)



发送公钥给129的机器

![image-20200829112225277](linux高级.assets/image-20200829112225277.png)

免密登录

![image-20200829112249831](linux高级.assets/image-20200829112249831.png)





### 小结

* SSH免密登录实现步骤？

  1、生成一对秘钥（非对称加密学）

  2、发送公钥给目标机子

  3、利用ssh ip免密登录

## 09.部署项目1-软件安装命令rpm与yum【理解】

### 目标

掌握rpm安装软件



### linux常用安装软件方式

1、rpm安装,  适合本地安装

2、yum安装， 适合在线网络下载安装

### RPM介绍

RedHat Package Manager(RPM)， 擅长安装软件



### rpm的作用

​	查询已安装的软件

​	安装软件

​	卸载软件

### rpm的语法

![image-20200307195407532](linux高级.assets/image-20200307195407532.png)

> rpm操作的都是以rpm为结尾的软件包

### rpm常用命令

```
rpm -qa #查询所有已安装软件rpm -ivh 软件包  #安装指定的软件包rpm -e --nodeps 软件包  #不验证软件依赖相关性卸载指定软件包rpm -ivhU * --nodeps --force  #不验证依赖与相关、强制安装软件包，或将已存在的更新升级			*代表安装当前目录下所有的rpm软件包
```

、

### YUM安装软件

在线下载安装命令：yum（全称为 Yellow dog Updater  Modified）作用：用于自动从服务器上下载相应的软件包，自动安装，并且自动下载它的依赖包。

### yum语法

![image-20200308142103253](linux高级.assets/image-20200308142103253.png)

注意：使用yum安装的软件，就要使用yum卸载软件

### rpm与yum的区别

| 方式                | 区别                                                         |
| ------------------- | ------------------------------------------------------------ |
| yum在线下载安装方式 | 不仅下载当前软件包还会自动下载依赖的软件包安装；yum底层使用的是rpm，对rpm进行了进一步包装与优化 |
| rpm在线下载安装方式 | 仅仅下载当前指定的软件包，对于依赖不会下载，所以在线下载rpm是麻烦的 |

​    yum，不仅可以在线安装指定的一个软件并且会将依赖的所有软件都进行安装，非常方便



### 小结

* rpm的常用作用有哪些？

  查询已安装的软件

  进行安装软件

  卸载已安装的软件

* rpm与yum的区别

  yum会自动下载安装指定的软件并且会下载与安装依赖的所有软件

  rpm只会下载安装当前指定的一个软件

  yum的本质就使用的是rpm，但是优化封装了rpm

## 10.部署项目2-jdk安装【应用】

### 疑问

* linux上运行java代码是否需要安装jdk？

  必须安装。

### 实现步骤

1. 将jdk软件上传到linux系统/usr/soft目录下

   

   ![1604378522092](linux高级.assets/1604378522092.png)

2. 先进入/usr/soft目录将jdk压缩文件解压到/usr/local

   ```shell
   tar -xvf jdk-8u162-linux-x64.tar.gz -C /usr/local
   ```

3. 配置linux的jdk环境变量，操作一个/etc/profile环境变量配置文件

   编辑配置文件配置环境变量

   ```
   vim /etc/profile
   ```

   在文件里面的末位添加如下配置

   ```properties
   #set java environmentJAVA_HOME=/usr/local/jdk1.8.0_162PATH=$JAVA_HOME/bin:$PATHexport JAVA_HOME  PATH
   ```

4. 重载环境变量配置文件

   ```shell
   source /etc/profile
   ```

   查看jdk环境变量是否配置成功，如下信息说明成功

   ![1550633028213](linux高级.assets/1550633028213.png)

### 小结

* 环境变量的配置文件叫什么？

  ​		/etc/profile 

* 修改过linux的环境变量文件需要注意什么？

  ​		重新加载才可生效， source /etc/profile 



## 11.部署项目3-mysql安装-启动-远程授权【应用】

### 目标

根据步骤安装Mysql软件



### 本地安装

1. 解压安装文件（创建一个解压后的目录，再进行解压）

   ```
   mkdir /usr/local/mysqltar -xvf mysql-community-5.6.45.tar.gz -C /usr/local/mysql
   ```

   ![1569307688860](linux高级.assets/1569307688860.png)

2. 查看解压后的文件列表

   ![1569307740363](linux高级.assets/1569307740363.png)

3. 执行安装命令

   ```
   rpm -ivhU * --nodeps --force
   ```

   

4. 执行安装的效果

   ![1569307811680](linux高级.assets/1569307811680.png)

### mysql本地登录使用

1. 启动mysql服务

   ```shell
   systemctl start mysqld
   ```

   > 将mysql加到系统服务中并设置开机启动【可以不用设置，默认mysql就开机启动】
   >
   > systemctl enable mysqld

2. 登录mysql，root用户默认没有密码

   ```shell
   mysql -uroot -p
   ```

3. 在mysql中修改当前用户root的密码，密码修改为root

   ```mysql
   set password = password('root'); 
   ```

### mysql远程登录使用

1. 开启mysql的远程登录权限，默认情况下mysql为安全起见，不支持远程登录mysql，所以需要设置开启，并且刷新权限缓存。远程登录mysql的权限登录mysql后输入如下命令  DCL

   ```mysql
   grant all privileges on *.* to 'root'@'%' identified by 'root';flush privileges;
   ```

   ![image-20200308145718046](linux高级.assets/image-20200308145718046.png)

2. 开放Linux的对外访问的端口3306

   ```shell
   #开放的端口永久保存到防火墙firewall-cmd --zone=public --add-port=3306/tcp --permanent#重启防火墙systemctl restart firewalld
   ```

   ![image-20200308145808842](linux高级.assets/image-20200308145808842.png)

windows主机mysql客户端就可以连接了，效果如下

![1552327912949](linux高级.assets/1552327912949.png)

### 小结



* 本地安装mysql

  * 解压
  * 使用rpm名进行安装
  * 启动mysql的服务
  * 登录root用户，然后修改密码

  

* linux如何让远程访问mysql数据库服务器？

  - 授权，允许远程可以登录root用户
  - 开放3306端口



## 12.部署项目4-linux版本redis1-安装【应用】

### 目标

在linux系统上安装redis



### linux版本的Redis介绍

从官方下载linux版本redis是c语言源代码，需要安装c语言编译器



### 本地安装

进入soft

```shell
cd /usr/soft
```



解压安装文件

```
tar -xvf gcc-c++_all.tar.gz -C /usr/local
```

![1569308803589](linux高级.assets/1569308803589.png)

进入解压后的目录，显示文件列表

```
cd /usr/local/gcc-c++_all
```

![1569308830178](linux高级.assets/1569308830178.png)



执行安装命令

```
rpm -ivhU * --nodeps --force
```

​          

安装的效果

![1569308919389](linux高级.assets/1569308919389.png)



进入soft目录

```shell
cd /usr/soft
```



解压redis的压缩文件

```shell
tar -xvf redis-3.2.11.tar.gz -C /usr/local
```



进入/usr/local/redis-3.2.11目录

```shell
cd /usr/local/redis-3.2.11/
```



执行make命令，用于编译c语言源代码

```shell
make
```

![1548291333634](linux高级.assets/1548291333634.png)

执行安装,必须在redis-3.2.11目录里面（make  PREFIX=安装真正安装的路径 install）

```shell
make PREFIX=/usr/local/redis install
```

   ![1548291415096](linux高级.assets/1548291415096.png)

   安装好的目录结构

   ![1548291504849](linux高级.assets/1548291504849.png)



### 小结

- 在linux版本的redis是使用什么语言开发？需要下载安装什么才能编译？

  > C语言开发
  >
  > gcc-c++_all

## 13.部署项目5-linux版本redis2-启动与停止【应用】

### 目标

在linux系统上操作redis的启动与停止



### 前端模式启动

命令：./redis-server

特点：霸占整个终端，导致无法运行其他命令

 ![1548291718432](linux高级.assets/1548291718432.png)



### 后端启动（推荐后端启动）

特点：后台运行，不影响运行其他命令

==执行Ctrl+C结束redis的服务器的运行==

![1558055652954](linux高级.assets/1558055652954.png)

将源代码目录中/usr/local/redis-3.2.11/redis.conf复制到当前安装bin目录下

```
cp /usr/local/redis-3.2.11/redis.conf .
```

![1548291871941](linux高级.assets/1548291871941.png)

修改redis.conf配置文件内容

```shell
vim redis.conf
```

查找daemonize,输入如下命令后回车

```shell
/daemonize
```

输入i进入编辑模式

```
i
```

修改`daemonize no`为`daemonize yes`，代表后端启动

```
daemonize yes
```

以后启动服务器端必须指明redis.conf进行启动

```
./redis-server redis.conf
```

![1548292088587](linux高级.assets/1548292088587.png)

启动客户端

```shell
./redis-cli
```

![1548292183726](linux高级.assets/1548292183726.png)



### 停止redis的2种方式

1. 杀死进程方式

   ```
   kill -9  进程号
   ```

   特点：强制杀死进程，容易导致redis如果正在持久化就可能会失败数据丢失

   ![1548292366752](linux高级.assets/1548292366752.png)

2. 客户端发送关闭命令（推荐）

   ```
   ./redis-cli shutdown
   ```

   特点：如果正在持久化先会等持久化完成后再关闭服务器

   ![1548292415863](linux高级.assets/1548292415863.png)



### 小结

1、在linux系统上操作redis的启动

​	  设置redis.conf配置文件`daemonize yes`

​	  启动服务器指定配置文件`./redis-server redis.conf`

2、redis的停止

​	   `./redis-cli shutdown`



## 14.部署项目6-linux版本redis3-客户端远程连接

### 目标

掌握使用客户端连接远程redis服务器和注意问题



### 讲解

1. 测试客户端是否可以远程连接到linux的redis

   ![1548292553147](linux高级.assets/1548292553147.png)

2. 无法连接，分析原因

   开放端口6379

   ```shell
   #开放的端口永久保存到防火墙，重启防火墙或重启电脑才会永久生效firewall-cmd --zone=public --add-port=6379/tcp --permanent#重启防火墙 systemctl restart firewalld
   ```

   redis在3.2.x版本以后进行了安全升级，默认只允许本地访问，不允许远程访问，这个安全设置在redis.conf中有配置，默认只允许本地127.0.0.1访问

   ```
   bind 127.0.0.1
   ```

3. 解决实现步骤

   修改配置文件redis.conf，增加允许远程看可以连接这个ip访问redis，指定的ip是当前linux系统的ip（redis服务器的ip）

   ```
   bind 127.0.0.1 (当前linux机器的ip:所有)192.168.56.106
   ```

   ![image-20200829155540932](linux高级.assets/image-20200829155540932.png)

   重启redis服务器

   ![1551403668897](linux高级.assets/1551403668897.png)

   ![1548293020352](linux高级.assets/1548293020352.png)&nbsp;

### 小结

客户端连接远程redis注意的问题：

	- 开放6379的端口- 修改redis.conf配置文件， bind  linux自己的ip地址



## 15.部署项目7-tomcat安装【应用】

### 目标

掌握在linux上安装tomcat软件

### 实现步骤

1. 将软件上传到linux系统/usr/soft目录下

   ```shell
   cd /usr/soft
   ```

2. 解压压缩包到/usr/local目录下

   ```shell
   tar -xvf apache-tomcat-8.5.27.tar.gz  -C /usr/local/
   ```

3. 进入bin目录，启动tomcat服务器

   ```shell
   ./startup.sh
   ```

4. 开发linux系统防火墙8080端口

   ```shell
   firewall-cmd --zone=public --add-port=8080/tcp --permanentsystemctl restart firewalld
   ```

5. windows客户端使用浏览器访问linux的8080端口tomcat

   ![1560917340918](linux高级.assets/1560917340918.png)

6. 进入bin目录下，关闭服务器。关闭服务器以后，浏览器不能再访问

   ```shell
   ./shutdown.sh
   ```

   ![1552328175791](linux高级.assets/1552328175791.png)

### 小结

* linux版本的tomcat的启动与关闭命令？

  startup.sh

  shutdown.sh



## 16.项目发布到linux运行【应用】

##### 目标

将黑马旅游项目部署到linux的tomcat运行



##### 分析linux目前环境

jdk软件

mysql软件

redis软件

tomcat软件

linux系统有以上软件环境可以运行黑马旅游前台项目了

##### 环境准备【已开启则忽略】

开启tomcat

![1567817247444](linux高级.assets/1567817247444.png)

访问服务器

![1567817288610](linux高级.assets/1567817288610.png)

启动redis

![1567817379831](linux高级.assets/1567817379831.png)



##### 本地黑马旅游数据库导出

![1551404176769](linux高级.assets/1551404176769-1593128408195.png)

![1551404205915](linux高级.assets/1551404205915-1593128408196.png)

##### 远程导入数据库到linux

![1551404224525](linux高级.assets/1551404224525-1593128408196.png)

![1551404247520](linux高级.assets/1551404247520-1593128408199.png)

##### 本地项目打包

##### 发布项目

**修改jedis.properties文件**



![1604462225160](linux高级.assets/1604462225160.png)

修改配置文件db.properties文件内容(linux版本的mysql传输命令默认使用的不是utf8, 所以需要设置)

![1604461595315](linux高级.assets/1604461595315.png)

修改日志代理类的代码

![1604461689565](linux高级.assets/1604461689565.png)

打包

![1551404797389](linux高级.assets/1551404797389-1593128408199.png)

将war包部署到linux的tomcat上

![1567817773274](linux高级.assets/1567817773274.png)

启动linux的tomcat

![1551404869315](linux高级.assets/1551404869315-1593128408200.png)

运行

![1567817980104](linux高级.assets/1567817980104.png)

## 17.nginx介绍与作用【理解】

##### 目标

理解nginx是处理高并发请求的http协议通信==服务器==

##### 介绍

nginx(NG)是http协议通信服务器，专门用于传输数据的服务器,  适合处理高并发请求。

##### Nginx 与 tomcat 区别

⚫ Nginx

> 1.专门用于传输数据的,并且对传输数据进行加密与压缩
>
> 2.处理静态资源的请求与响应
>
> 3.由于只负责传输数据, 占用内存很小,  所以可以处理高并发请求

⚫ Tomcat

>  1.处理静态资源与动态资源的请求与响应,  
>
>  2.由于处理动态资源, 进行业务逻辑与算法的控制, 所以会占用内存很多, 所以不可以高并发

<font color=red>**总结**</font>

```
以后使用ng接收高并发的请求，之后分发给多台tomcat处理请求,  这个过程叫NG的负载均衡
```



##### nginx作用

1.发布静态资源（）

2.反向代理,将请求分发给tomcat, 负载均衡（真正的作用）

> 终极目标:  实现负载均衡（项目二学习）,将请分发给多台tomcat



## 18.linux版nginx的安装与发布静态资源【应用】

##### 目标

掌握在linux上安装nginx与发布静态资源

##### nginx安装

> 下载地址：http://nginx.org/en/download.html

1. 将软件压缩文件上传到linux上

   ![1548295126251](linux高级.assets/1548295126251-1593128487338.png)

2. 解压到/usr/local目录下

   ```shell
   cd /usr/soft
   ```

   解压

   ```shell
   tar -xvf nginx-1.15.3.tar.gz -C /usr/local
   ```

   

   ![1548295168945](linux高级.assets/1548295168945-1593128487339.png)

3. nginx是用c语言发开，并使用编译环境gcc进行编译并安装，在安装之前需要安装依赖环境

   安装依赖环境，

   ```shell
   #正则表达式的库：1.4Myum -y install pcre pcre-devel#数据压缩的库：132Kyum -y install zlib zlib-devel #安全证书的库：6.4Myum -y install openssl openssl-devel
   ```

   

   配置安装命令，进入

   ```shell
   cd /usr/local/nginx-1.15.3
   ```

   

   > 目录下执行(就是配置nginx默认安装在哪里，和搜索要安装的资源),默认安装到/usr/local/nginx目录

   ```
   ./configure
   ```

   ![image-20200318101558209](linux高级.assets/image-20200318101558209.png)

   编译并安装，进入nginx解压后的目录

   ```
   make && make install
   ```

4. nginx的目录介绍

   进入安装的nginx目录

   ```shell
   cd /usr/local/nginx
   ```

   ![1567821006415](linux高级.assets/1567821006415.png)

   ![image-20200829165024757](linux高级.assets/image-20200829165024757.png)

5. 启动nginx相关命令

   ![1548295942335](linux高级.assets/1548295942335-1593128487340.png)

   <font color=red>NG服务器使用的是80端口</font>

6. 开放端口

   ```shell
   #开放的端口永久保存到防火墙firewall-cmd --zone=public --add-port=80/tcp --permanent#重启防火墙systemctl restart firewalld
   ```

   ![1548296035402](linux高级.assets/1548296035402-1593128487340.png)

7. 客户端使用浏览器访问nginx

   ![1567821139026](linux高级.assets/1567821139026.png)

   <font color=red>http协议访问服务器默认端口是80，也就是端口不写就是80</font>

   <font color=red>https协议访问服务器默认端口是443，也就是端口不写就是443</font>

##### 发布静态资源

1. 将静态资源发布到nginx/html目录下

   ![1567821339794](linux高级.assets/1567821339794.png)

2. 使用nginx -s reload命令重新的加载

3. 浏览器访问

   ![1567821375069](linux高级.assets/1567821375069.png)

##### nginx常见命令(nginx/sbin/nginx可执行文件)

| **启动与关闭的命令** | **说明**                                        |
| -------------------- | ----------------------------------------------- |
| ./nginx              | 启动nginx                                       |
| ./nginx -s reload    | 重新启动nginx，重新读取配置文件 conf/nginx.conf |
| ./nginx -s stop      | 关闭nginx服务器                                 |

##### nginx.conf配置文件介绍

![1548296316655](linux高级.assets/1548296316655-1593128487341.png)

![1561689083242](linux高级.assets/1561689083242-1593128487341.png)

##### 小结

安装linux版本的nginx的注意：

​	1、先安装3个依赖库（数据压缩、安全证书、正则表达式）

​    2、nginx软件官方是c语言源代码，使用gcc先编译后安装

​    3、开放80端口，外部才可以访问

nginx目录结构

​		html: 发布静态资源根目录

​        conf：核心配置文件nginx.conf

​        sbin：里面nginx可执行命令

关于命令语法:

​		启动命令：./nginx

​        重载命令：./nginx  -s reload



## 19.windows版nginx的安装与发布静态资源【应用】

##### 目标

掌握windows版本nginx的安装与发布静态资源



##### windows安装

1. 解压完成完成安装

   ![1548297475184](linux高级.assets/1548297475184-1593128487341.png)

2. 启动nginx，使用浏览器访问

   双击nginx.exe（会一闪而过，正常现象）

   ![1548297502615](linux高级.assets/1548297502615-1593128487341.png)

##### 发布静态资源

将静态资源发布D:\LY目录内

![1561690437016](linux高级.assets/1561690437016-1593128487341.png)

修改配置文件nginx.conf

![1561690494028](linux高级.assets/1561690494028-1593128487341.png)

<font color=red>修改了配置文件需要重启nginx服务器</font>

![1561690525767](linux高级.assets/1561690525767-1593128487341.png)

使用浏览器访问

![1561690534911](linux高级.assets/1561690534911-1593128487342.png)



##### 小结

**发布静态资源的方式**

	- 放入html文件夹- 修改nginx.conf文件修改根路径

**nginx启动、关闭、重新加载命令？**

 - ./nginx

 - ./nginx  - s  stop

 - ./nginx  - s  reload

   

## 20.正向代理介绍【理解】

##### 目标

理解正向代理的介绍与作用



##### 介绍

解决个人电脑(客户端)无法远程访问其他资源的方式,  使用一个代理服务器解决

##### 原理架构图

![image-20200626163529841](linux高级.assets/image-20200626163529841.png)&nbsp;

客户端与代理服务器形成一个LAN局域网

##### 作用

（1）对客户端访问授权，上网进行认证
（2） 可以做缓存，加速访问资源
（4）代理可以记1，对外隐藏用户信息

##### 小结

核心作用：正向代理解决访问无法访问的资源

正向代理的特点：客户端电脑与代理服务器是局域网，  代理服务器与目标服务器是外网

## 21.反向代理介绍【理解】

##### 目标

理解反向代理介绍与作用

##### 介绍

反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

##### 架构图

![1604472419528](linux高级.assets/1604472419528.png)

![1548298216831](linux高级.assets/1548298216831-1593128487342.png)

反向代理是代理服务器与原始服务器形成局域网，原始服务器没有暴露在外网上。

![1567822759087](linux高级.assets/1567822759087.png)

##### 作用

1.保证内网服务器的安全。由于原始服务器没有暴露在外网上，所以外网无法攻击服务器。

2.实现反向代理(可以将代理服务器的请求分发给一个tomcat)。终极目标是实现负载均衡(分发给多个tomcat)。

![1548298488751](linux高级.assets/1548298488751-1593128487342.png)



##### 小结

反向代理特点：  客户发送请求都给代理服务器，代理服务器分发请求给目标服务器

​                             客户与代理服务器在外网上，代理服务器与目标服务器在内网（局域网）

反向代理作用：

​			1、保证内网服务器的安全

​            2、实现反向代理：提升访问并发量

## 22.nginx实现反向代理【应用】

##### 目标

nginx实现反向代理tomcat服务器



##### 反向代理实现需求

用户请求交给nginx（反向代理服务器），nginx将请求交给tomcat去处理，处理完成交回给nginx，nginx交给用户。tomcat是原始服务器，不暴露在外网上。



##### 实现步骤

1. 启动nginx

2. 修改nginx.conf配置文件，增加代理tomcat

   在server配置的上面增加如下命令：

   ```
   #增加代理tomcat服务器upstream test2{server localhost:8080;}
   ```

   ![image-20200318112321480](linux高级.assets/image-20200318112321480.png)

   修改location访问资源使用代理

   修改location里面的代码如下

   ```shell
   location / {			            #root   html;  注释本地资源的访问            index  index.html index.htm;            #配置所有请求访问代理            proxy_pass http://test2;        }
   ```

   ![image-20200318112656410](linux高级.assets/image-20200318112656410.png)

3. 重载nginx

   进入sbin目录,执行nginx重启命令，如下

   ```
   cd ../sbin./nginx -s reload
   ```

   ![1548299125500](linux高级.assets/1548299125500-1593128487343.png)

4. 使用客户端浏览器访问nginx就可以实现访问黑马旅游项目

   ![1561692004994](linux高级.assets/1561692004994-1593128487343.png)

##### 小结

* 反向代理的好处？

  1.保证内网tomcat的安全

  2.反向代理的目标提升服务器并发访问量
