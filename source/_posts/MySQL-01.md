---
title: MySQL(01)
tags:
  - 笔记
  - 数据库
  - MySQL
  - MySQL基础
  - SQL
  - SQL建库、建表
  - SQL添删改查
  - SQL排序
categories:
  - 数据库
abbrlink: 46604
date: 2020-09-17 16:04:09
---


# MySQL基础语法

## 学习目标

1. 能够使用SQL语句建库、建表
2. 能够使用SQL语句进行数据的添删改查操作
3. 能够使用SQL语句进行排序



# 学习内容

## 数据库的基本知识

### 目标

1. 学习数据库的概念
2. 了解常用的数据库



#### 什么是数据库

存储数据的仓库



#### 数据的存储方式

1. **数据保存在内存**

   ```java
   int[] arr = new int[]{1, 2, 3, 4};
   ArrayList<Integer>list = new ArrayList<Integer>();
   list.add(1);
   list.add(2);
   ```

   new出来的对象存储在堆中，堆是内存中的一小块空间

   优点：内存速度快
   缺点：断电/程序退出，数据就清除了，内存价格贵

2. **数据保存在普通文件**
   优点：永久保存
   缺点：查找，增加，修改，删除数据比较麻烦，效率低

3. **数据保存在数据库**
   优点：永久保存，通过SQL语句比较方便的操作数据库，数据库是对大量的信息进行管理的高效的解决方案



#### 常见数据库

![常见数据库](mysql-01.assets/常见数据库.PNG)
**<font color="red">Oracle</font>**：收费的大型数据库，Oracle公司的产品。Oracle收购SUN公司，收购MYSQL。

![1596868707747](mysql-01.assets/1596868707747.png)

**<font color="red">MySQL</font>**：开源免费的数据库，小型的数据库。已经被Oracle收购了。MySQL6.x版本也开始收费。

![1596868680479](mysql-01.assets/1596868680479.png)

**DB2** ：IBM公司的数据库产品,收费的。常应用在银行系统中。
**SQLServer**：MicroSoft 公司收费的中型的数据库。C#、.net等语言常使用。
**SyBase**：已经淡出历史舞台。提供了一个非常专业数据建模的工具PowerDesigner。
**SQLite**: 嵌入式的小型数据库，应用在手机端。

#### 常用数据库

**MySQL**，**Oracle**

在web应用中，使用的最多的就是MySQL数据库，原因如下：

1. 开源、免费
2. 功能足够强大，足以应付web应用开发（最高支持千万级别的并发访问）



### 小结

1. 说出数据库的概念：存储数据的仓库
2. 说出常用的数据库：MySQL, Oracle



## MySQL数据库卸载

### 目标

了解MySQL数据库卸载



![](mysql-01.assets/unload.png)



## 数据库的安装

### 目标

学习MySQL数据库软件的安装



### 讲解

安装过程：

1. 文件复制的过程，解压文件到指定的目录下

   ![1550408706301](mysql-01.assets/1550408706301.png)

2. 服务器的配置

   ![1550408729170](mysql-01.assets/1550408729170.png)

   端口号是：3306

   管理员名字叫：root

   可以远程访问：![1550408779650](mysql-01.assets/1550408779650.png)



### 小结

1. MySQL安装过程的两个步骤：解压复制    配置信息

2. MySQL端口号是：3306
3. 管理员名字：root



## MySQL目录结构

### 目标

了解MySQL目录结构

```sql
│-- bin：mysql相关的可执行文件*.exe
     │-- MySQLInstanceConfig.exe mysql的配置程序
│-- data: mysql自带的数据库文件(不用关注)
│-- include: c语言的头文件(不用关注)
│-- lib: 存放mysql使用到的dll动态库(相当于jar包，不用关注)
│-- my.ini mysql的配置文件,配置了mysql的相关信息
```



## 命令行客户端连接服务器

### 目标

1. 学习启动和关闭mysql服务
2. 学习登录mysql



#### 打开和关闭mysql服务(了解)

![1550408825267](mysql-01.assets/1550408825267.png)
![mysql启动02](mysql-01.assets/mysql启动02.png)





#### 登录mysql服务器

![1550289380140](mysql-01.assets/1550289380140.png)

​	MySQL是一个需要账户名密码登录的数据库，登陆后使用，它提供了一个默认的root账号，使用安装时设置的密码即可登录

1. 登录格式1：在DOS命令行：`mysql -u用户名 -p密码` 

> u和p后面没有空格

例如：   

   ```sql
mysql -uroot -proot
   ```

![MYSQL登录01](mysql-01.assets/MYSQL登录01.png)

   

后输入密码方式：

   ```sql
mysql -uroot -p回车
下一行输入密码
   ```


2. 登录格式2：`mysql -hip地址 -u用户名 -p密码`
   例如：

   ```sql
   mysql -h127.0.0.1 -uroot -proot
   ```

   

3. 退出MySQL：`exit`或`quit`



### 小结

1. 连接到本机的mysql

   ```sql
   mysql -u账号 -p密码
   ```

2. 连接到指定主机的mysql

   ```sql
   mysql -hip地址 -u账号 -p密码
   ```

   

## 图形界面SQLyog客户端

### 目标

1. 学习SQLyog的安装
2. 学习使用SQLyog连接mysql

#### 讲解

![1550289366973](mysql-01.assets/1550289366973.png)

​	SQLyog是业界著名的Webyog公司出品的一款简洁高效、功能强大的图形化MySQL数据库管理工具。使用SQLyog可以快速直观地让您从世界的任何角落通过网络来维护远端的MySQL数据库

![SQLYog介绍](mysql-01.assets/SQLYog介绍.png)

1. 双击![SQLYog程序](mysql-01.assets/SQLYog程序.png)
2. 一直下一步，直到出现下面对话框
   ![SQLYog注册](mysql-01.assets/SQLYog注册.png)
3. 输入名称和秘钥
4. 重启SQLyog即可
5. 使用SQLyog登录数据库
   ![SQLyog使用01.png](mysql-01.assets/SQLyog使用01.png)





![1597196571164](mysql-01.assets/1597196571164.png)

![1597196642007](mysql-01.assets/1597196642007.png)

### 小结

1. SQLyog的安装?  

   一直下一步

2. 使用SQLyog连接mysql?  

   ![SQLyog使用01.png](mysql-01.assets/SQLyog使用01.png)

   

## 服务器与数据库、表、记录的关系

### 目标

学习MySQL服务器与数据库、表、记录的关系



#### 讲解

![1550409365086](mysql-01.assets/1550409365086.png)



**<font color="red">关系型数据库的核心单元是表,有行有列</font>**



### 小结

MySQL服务器与数据库、表、记录的关系？

MySQL服务器是一个服务端软件,可以管理很多的数据库,一个数据库相当于一个文件夹,一个数据库中可以包含很多表,一张表相当于一个文件,表中可以存储很多的数据,一条数据就称为一条记录

![1600307964925](mysql-01.assets/1600307964925.png)





## SQL语句的分类和语法

### 目标

学习SQL的概念和作用

#### 什么是SQL

(**S**tructured **Q**uery **L**anguage) 结构化查询语言，简称SQL。



#### SQL作用

通过SQL语句我们可以**方便**的操作数据库、表、数据。
SQL是数据库管理系统都需要遵循的规范。不同的数据库生产厂商都支持SQL语句，但都有特有内容。
![SQL规范](mysql-01.assets/SQL规范.png)



#### SQL语句分类

1. <font color="red">**DDL**</font>(Data Definition Language) 数据定义语言
   用来定义数据库对象：数据库，表，列等。关键字：create，drop，alter等

   

2. <font color="red">**DML**</font>(Data Manipulation Language) 数据操作语言
   用来对数据库中表的数据进行增删改。关键字：insert，delete，update等

   

3. <font color="red">**DQL**</font>(Data Query Language) 数据查询语言
   对数据库进行数据查询，关键字select

   

4. DCL(Data Control Language)数据控制语言(了解)

   是用来设置或更改数据库用户或角色权限的语句，这个比较少用到



#### SQL通用语法

1. SQL语句可以单行或多行书写，以分号结尾。

2. 可使用空格和缩进来增强语句的可读性。

3. MySQL数据库的SQL语句不区分大小写，关键字建议使用大写。

   ```sql
   SELECT
      * 
         FROM 
             abc;
   ```

4. 3种注释
   单行注释: -- 注释
   多行注释: `/*注释*/`
   MySQL特有的单行注释：# 注释

   

### 小结

1. SQL的作用？

   ```
   通过SQL语句可以很方便的操作数据库,表,记录
   ```

   

2. SQL的分类？

   ```
   DDL: 数据定义语言  (操作数据库,表)
   DML: 数据操作语句  (操作数据的增删改)
   DQL: 数据查询语句  (查询数据)
   ```

   

   

## DDL创建数据库(重要)

### 目标

学习创建数据库的三种语法



### 讲解

#### 创建数据库

1. 直接创建数据库

   ```sql
   CREATE DATABASE 数据库名;
   ```

2. 判断是否存在并创建数据库

   ```sql
   CREATE DATABASE IF NOT EXISTS 数据库名;
   ```

3. 创建数据库并指定字符集(编码表)

   ```sql
   CREATE DATABASE 数据名 DEFAULT CHARACTER SET 字符集;
   ```


4. 具体操作：

* 直接创建数据库db1

  ```sql
  CREATE DATABASE db1;
  ```

  ![直接创建数据库](mysql-01.assets/直接创建数据库.png)

* 判断是否存在并创建数据库db2

  ```sql
  CREATE DATABASE IF NOT EXISTS db2;
  ```

  ![判断是否存在并创建数据库](mysql-01.assets/判断是否存在并创建数据库.png)

* 创建数据库db3并指定字符集为gbk

  ```sql
  CREATE DATABASE db2 CHARACTER SET gbk;
  ```

  ![创建数据库并指定字符集](mysql-01.assets/创建数据库并指定字符集.png)



#### 查看数据库

1. 查看所有的数据库

```sql
SHOW DATABASES;
```

   ![查看所有数据库](mysql-01.assets/查看所有数据库.png)

2. 查看某个数据库的定义信息

```sql
SHOW CREATE DATABASE 数据名;
```

   ![查看某个数据库的定义信息](mysql-01.assets/查看某个数据库的定义信息.png)



### 小结

1. 创建数据库语法

   ```sql
   CREATE DATABASE 数据库名;
   ```

   

2. 查看有哪些数据库

   ```sql
   SHOW DATABASES;
   ```

   

## DDL修改和删除数据库(重要)

### 目标

1. 学习修改数据库的字符集
2. 学习删除数据库



#### 修改数据库字符集

```sql
ALTER DATABASE 数据库名 DEFAULT CHARACTER SET 字符集;

ALTER: 表示修改
DATABASE: 数据库
```

具体操作：

* 将db3数据库的字符集改成utf8

  ```sql
  ALTER DATABASE db3 DEFAULT CHARACTER SET utf8;
  ```

  ![修改数据库字符集](mysql-01.assets/修改数据库字符集.png)

#### 删除数据库

```sql
DROP DATABASE 数据库名;

DROP: 删除
```

具体操作：

* 删除db2数据库

  ```sql
  DROP DATABASE db2;
  ```

  ![删除数据库](mysql-01.assets/删除数据库.png)



### 小结

1. 修改数据库的字符集格式？

   ```sql
   ALTER DATABASE 数据库名 DEFAULT CHARACTER SET 新字符集;
   ```

   

2. 删除数据库格式？

   ```sql
   DROP DATABASE 数据库名;
   ```

   

## DDL使用数据库

### 目标

1. 学习切换数据库语法
2. 查看正在使用的数据库



### 讲解

1. 查看正在使用的数据库

   ```sql
   SELECT DATABASE();
   ```

2. ## 使用/切换数据库

   ```sql
   USE 数据库名;
   ```

具体操作：

* 查看正在使用的数据库

  ```sql
  SELECT DATABASE();
  ```

  ![查看正在使用的数据库](mysql-01.assets/查看正在使用的数据库.png)

* 使用db1数据库

  ```sql
  USE db1;
  ```

  ![使用db1数据库](mysql-01.assets/使用db1数据库.png)



### 小结

| DDL语句操作数据库 | 关键字                                                |
| ----------------- | ----------------------------------------------------- |
| 创建              | CREATE DATABASE 数据库名;                             |
| 修改              | ALTER DATABASE 数据库名 DEFAULT CHARACTER SET 字符集; |
| 查看              | SHOW DATABASES;                                       |
| 删除              | DROP DATABASE 数据库名;                               |



## DDL创建表(重要)

### 目标

学习DDL创建表



>**前提先使用某个数据库**（db1）

#### 创建表

```sql
CREATE TABLE 表名 (字段名1 字段类型1, 字段名2 字段类型2);
```

建议写成如下格式：

```sql
CREATE TABLE 表名 (
    字段名1 字段类型1,
    字段名2 字段类型2
);
```

#### MySQL数据类型

MySQL中的我们常使用的数据类型如下：
![MYSQL常用数据类型](mysql-01.assets/MYSQL常用数据类型.png)

详细的数据类型如下(不建议详细阅读！)

![1550410505421](mysql-01.assets/1550410505421.png)

具体操作: 

创建student表包含id,name,birthday字段

```sql
CREATE TABLE student (
      id INT,
      name VARCHAR(20),
      birthday DATE
);
```



### 小结

1. 创建表语句

   ```sql
   CREATE TABLE 表名 (
   	字段名1 字段类型1,
       字段名2 字段类型2
   );
   ```

   

2. 常用数据类型

   ```
   int: 整数
   double: 小数
   date: 日期
   varchar(length) 可变长度字符串
   ```

   

## DDL查看表(重要)

### 目标

1. 学习查看某个数据库中的所有表
2. 学习查看表结构

### 讲解

1. 查看某个数据库中的所有表

   ```sql
   SHOW TABLES;
   ```

2. 查看表结构

   ```sql
   DESC 表名;
   ```

3. 查看创建表的SQL语句

   ```sql
   SHOW CREATE TABLE 表名;
   ```

  具体操作：

* 查看MySQL数据库中的所有表

  ```sql
  SHOW TABLES;
  ```

  ![查看某个数据库中的所有表](mysql-01.assets/查看某个数据库中的所有表.png)

* 查看student表的结构

  ```sql
  DESC student;
  ```

  ![查看student表的结构](mysql-01.assets/查看student表的结构.png)

* 查看student的创建表SQL语句

  ```sql
  SHOW CREATE TABLE student;
  ```

  ![查看student的创建表SQL语句](mysql-01.assets/查看student的创建表SQL语句.png)

### 小结

1. 查看某个数据库中的所有表

   ```sql
   SHOW TABLES;
   ```

   

2. 查看表结构

   ```sql
   DESC 表名;
   ```

   

3. 查看创建表的SQL语句

   ```sql
   SHOW CREATE TABLE 表名;
   ```

   

## DDL删除表(重要)

### 目标

1. 学习删除表语法
2. 学习快速创建一个表结构相同的表



### 讲解

#### 快速创建一个表结构相同的表(复制表)

```sql
CREATE TABLE 表名 LIKE 其他表;
```

具体操作：

- 创建s1表，s1表结构和student表结构相同

  ```sql
  CREATE TABLE s1 LIKE student;
  ```


#### 删除表

1. 直接删除表

   ```sql
   DROP TABLE 表名;
   ```

2. 判断表是否存在并删除表

   ```sql
   DROP TABLE IF EXISTS 表名;
   ```

具体操作：

* 直接删除表s1表

  ```sql
  DROP TABLE s1;
  ```

  ![直接删除表](mysql-01.assets/直接删除表.png)

* 判断表是否存在并删除s1表

  ```sql
  DROP TABLE IF EXISTS s1;
  ```

  ![判断表存在并删除](mysql-01.assets/判断表存在并删除.png)

### 小结

1. 快速创建一个表结构相同的表

   ```sql
   CREATE TABLE 表名 LIKE 其他表;
   ```

   

2. 删除表语法

   ```sql
   DROP TABLE 表名;
   ```

   

## DDL修改表结构

### 目标

学习修改表结构的语法



### 讲解

所有的修改表结构的语句都是:  <font color="red">**ALTER TABLE 表名 XXX;**</font>

> 修改表结构使用不是很频繁，只需要了解，等需要使用的时候再回来查即可

1. 添加表一列

   ```sql
   ALTER TABLE 表名 ADD 字段名 字段类型;
   ```

   具体操作：

   * 为学生表添加一个新的字段remark,类型为varchar(20)

     ```sql
     ALTER TABLE student ADD remark VARCHAR(20);
     ```

     ![添加字段](mysql-01.assets/添加字段.png)

2. 修改字段类型

   ```sql
   ALTER TABLE 表名 MODIFY 字段名 新类型;
   ```

   具体操作：

   * 将student表中的remark字段的改成varchar(100)

     ```sql
     ALTER TABLE student MODIFY remark VARCHAR(100);
     ```

     ![修改字段类型](mysql-01.assets/修改字段类型.png)

3. 修改字段名

   ```sql
   ALTER TABLE 表名 CHANGE 老字段名 新字段名 类型;
   ```

   具体操作：

   * 将student表中的remark字段名改成intro，类型varchar(30)

     ```sql
     ALTER TABLE student CHANGE remark intro varchar(30);
     ```

     ![修改表字段名称](mysql-01.assets/修改表字段名称.png)

4. 删除字段

   ```sql
   ALTER TABLE 表名 DROP 字段名;
   ```

   具体操作：

   * 删除student表中的字段intro

     ```sql
     ALTER TABLE student DROP intro;
     ```

     ![删除字段](mysql-01.assets/删除字段.png)

5. 修改表名

   ```sql
   RENAME TABLE 表名 TO 新表名;
   ```

具体操作：

* 将学生表student改名成student2，再删除student2表

  ```sql
   RENAME TABLE student TO student2;
   DROP TABLE student2;
  ```

  ![修改表名](mysql-01.assets/修改表名.png)

6. 修改表的字符集

   ```sql
   ALTER TABLE 表名 DEFAULT CHARACTER SET 新字符集;
   ```

   具体操作：

   * 将sutden2表的编码修改成gbk

     ```sql
     ALTER TABLE student2 character set gbk;
     ```

      ![修改字符集](mysql-01.assets/修改字符集.png)



### 小结

1. 所有修改表前面的语法都是相同的？

   ```sql
   ALTER TABLE 表名 xxx;
   ```

   

2. 添加字段：

   ```sql
   ALTER TABLE 表名 ADD 字段名 字段类型;
   ```

   

3. 修改字段类型：

   ```sql
   ALTER TABLE 表名 MODIFY 字段名 新类型;
   ```

   

4. 修改字段名和类型：

   ```sql
   ALTER TABLE 表名 CHANGE 老字段名 新字段名 类型; 
   ```

   

5. 删除一列：

   ```sql
   ALTER TABLE 表名 DROP 字段名;
   ```

   


## DML插入记录(重点)

### 目标

学习DML往表中添加记录



### 讲解

DML是对表中的数据进行增删改

创建student表包含id,name,birthday,sex,address字段。

```sql
CREATE TABLE student (
      id INT,
      name VARCHAR(20),
      birthday DATE,
      sex char(2),
      address varchar(50)
);
```

#### 插入全部字段

* 所有的字段名都写出来

  ```sql
  INSERT INTO 表名 (字段名1, 字段名2, 字段名3, ...) VALUES (值1, 值2, 值3, ...);
  ```

* 不写字段名

  ```sql
  INSERT INTO 表名 VALUES (值1, 值2, 值3, ...);
  ```



#### 插入部分数据

```sql
只需要指定要插入数据的字段
INSERT INTO 表名 (字段名1, 字段名2...) VALUES (字段值1, 字段值2);
```

<font color="red">**没有添加数据的字段会使用NULL**</font>

1. 具体操作:

   * 插入部分数据，往学生表中添加 id, name, age, sex数据

   ```sql
   INSERT INTO student (id, NAME, age, sex) VALUES (1, '张三', 20, '男');
   ```

   ![添加部分数据](mysql-01.assets/添加部分数据.png)

   * 向表中插入所有字段

     * 所有的字段名都写出来

     ```sql
     INSERT INTO student (NAME, id, age, sex, address) VALUES ('李四', 2, 23, '女', '广州');
     ```

     ![所有字段都添加数据](mysql-01.assets/所有字段都添加数据.png)

     * 不写字段名

     ```sql
     INSERT INTO student VALUES (3, '王五', 18, '男', '北京');
     ```

     ![添加所有字段数据](mysql-01.assets/添加所有字段数据.png)



#### 注意

> - 值与字段必须对应，个数相同，类型相同
> - 值的数据大小必须在字段的长度范围内
> - 除了数值类型外，其它的字段类型的值必须使用引号引起。（建议单引号）
> - 如果要插入空值，可以不写字段，或者插入NULL



### 小结

1. 向表中添加一条完整记录：

   ```sql
   所有的字段都写出来
   INSERT INTO 表名 (字段名1, 字段名2, ...) VALUES (值1, 值2, ...);
   
   不写字段名
   INSERT INTO 表名 VALUES (值1, 值2, ...);
   ```

   

## DOS命令窗口操作数据乱码问题

### 目标

学习解决DOS命令行中文乱码



### 讲解

>当我们使用DOS命令行进行SQL语句操作如有有中文会出现乱码，导致SQL执行失败
>![DOS中文乱码01](mysql-01.assets/DOS中文乱码01.png)
>错误原因:因为MySQL的客户端设置编码是utf8，而系统的DOS命令行编码是gbk，编码不一致导致的乱码
>![1551157215427](mysql-01.assets/1551157215427.png)



解决方案：

1. 快捷设置

   ```sql
   在DOS命令行输入：
   set names gbk;
   ```

> 注意：以上方式为临时方案，退出DOS命令行就失效了，需要每次都配置

2. 修改MySQL安装目录下的my.ini文件，重启服务所有地方生效。此方案将所有编码都修改了[**不建议**]
   ![DOS中文乱码04](mysql-01.assets/DOS中文乱码04.png)



### 小结

如何解决DOS命令行乱码

```sql
在DOS命令行输入：
set names gbk;
```




## DML更新表记录(重要)

### 目标

学习DML更新表记录



### 讲解

1. 不带条件修改数据

   ```sql
   UPDATE 表名 SET 字段名=字段值;
   ```

2. 带条件修改数据

   ```sql
   UPDATE 表名 SET 字段名=字段值 WHERE 条件;
   ```

   

3. 具体操作：

   * 不带条件修改数据，将所有的性别改成女

     ```sql
     UPDATE student SET sex='女';
     ```

     ![修改所有数据](mysql-01.assets/修改所有数据.png)

   * 带条件修改数据，将id号为2的学生性别改成男

     ```sql
     UPDATE student SET sex='男' WHERE id=2;
     ```

     ![带条件修改](mysql-01.assets/带条件修改.png)

   * 一次修改多个列，把id为3的学生，年龄改成26岁，address改成北京

     ```sql
     UPDATE student SET age=26, address='北京' WHERE id=3;
     ```

     ![一次性修改2个字段](mysql-01.assets/一次性修改2个字段.png)

### 小结

1. 不带条件的更新数据库记录

   ```sql
   UPDATE 表名 SET 字段名=值;
   ```

   

2. 带条件更新数据库记录

   ```sql
   UPDATE 表名 SET 字段名=值 WHERE 条件;
   ```

   

## DML删除表记录(重要)

### 目标

学习DML删除表记录



### 讲解

1. 带条件删除数据

   ```sql
   DELETE FROM 表名 WHERE 条件;
   ```

2. 不带条件删除数据

   ```sql
   DELETE FROM 表名;
   ```

3. 具体操作：

   * 带条件删除数据，删除id为3的记录

     ```sql
     DELETE FROM student WHERE id=3;
     ```

     ![删除满足条件的记录](mysql-01.assets/删除满足条件的记录.png)

   * 不带条件删除数据,删除表中的所有数据

     ```sql
     DELETE FROM student;
     ```

     ![删除所有记录](mysql-01.assets/删除所有记录.png)



### 小结

1. 指定条件删除

   ```sql
   DELETE FROM 表名 WHERE 条件;
   ```

2. 没有条件删除所有的记录

   ```sql
   DELETE FROM 表名;
   ```



## DQL没有条件的简单查询(重要)

### 目标

学习DQL简单查询



### 讲解

>注意：查询不会对数据库中的数据进行修改，只是一种显示数据的方式。


#### 查询表中所有列数据

1. 写出查询每列的名称

```sql
SELECT 字段名1, 字段名2, 字段名3 FROM 表名;
```

   具体操作：

   ```sql
SELECT id, NAME ,age, sex, address FROM student;
   ```

   ![查询所有列](mysql-01.assets/查询所有列.png)

2. 使用*表示所有列

   ```sql
   SELECT * FROM 表名;
   ```

   具体操作：

   ```sql
   SELECT * FROM student;
   ```

   ![查询所有列](mysql-01.assets/查询所有列.png)


查询student表中的name 和 age 列

```sql
SELECT NAME, age FROM student;
```

![查询指定字段](mysql-01.assets/查询指定字段.png)

#### 别名查询

1. 查询时给列、表指定别名需要使用AS关键字

2. 使用别名的好处是方便观看和处理查询到的数据

   ```sql
   SELECT 字段名1 AS 别名, 字段名2 AS 别名... FROM 表名;
   ```

   

> 注意: AS关键字可以省略

3. 具体操作：

   * 查询sudent表中name 和 age 列，name列的别名为”姓名”，age列的别名为”年龄”

   ```sql
   SELECT NAME AS 姓名, age AS 年龄 FROM student;
   ```

   ![查询字段别名](mysql-01.assets/查询字段别名.png)



#### 清除重复值

1. 查询指定列并且结果不出现重复数据

   ```sql
   SELECT DISTINCT 字段名 FROM 表名;
   ```

2. 具体操作：

   * 查询address列并且结果不出现重复的address

   ```sql
   SELECT DISTINCT address 城市 FROM student;
   ```

   ![1550289641391](mysql-01.assets/1550289641391.png)

#### 查询结果参与运算

1. 某列数据和固定值运算

   ```sql
   SELECT 字段名 + 值 FROM 表名;
   ```

2. 某列数据和其他列数据参与运算

   ```sql
   SELECT 字段1 + 字段2 FROM 表名;
   ```

   >注意: 参与运算的必须是数值类型

3. 需求：

   * 添加数学，英语成绩列，给每条记录添加对应的数学和英语成绩
   * 查询的时候将数学和英语的成绩相加

4. 实现：

* 修改student表结构,添加数学和英语成绩列

  ```sql
  ALTER TABLE student ADD math INT;
  ALTER TABLE student ADD english INT;
  ```

* 给每条记录添加对应的数学和英语成绩
  ![添加数学和英语成绩](mysql-01.assets/添加数学和英语成绩.png)

* 查询math + english的和

  ```sql
  SELECT math + english FROM student;
  ```

  ![查询math和english的和](mysql-01.assets/查询math和english的和.png)

  >结果确实将每条记录的math和english相加，但是效果不好看

* 查询math + english的和使用别名”总成绩”

  ```sql
  SELECT math + english 总成绩 FROM student;
  ```

  ![组合查询结果取别名](mysql-01.assets/组合查询结果取别名.png)

* 查询所有列与math + english的和并使用别名”总成绩”

  ```sql
  SELECT *, math + english 总成绩 FROM student;
  ```

  ![查询所有列数据和参与运算](mysql-01.assets/查询所有列数据和参与运算.png)

* 查询姓名、年龄，将每个人的数学增加10分

  ```sql
  SELECT name, math + 10 FROM student;
  ```



### 小结

1. 简单查询格式

   ```sql
   SELECT 字段名1, 字段名2, 字段名3, ... FROM 表名;
   ```

   

2. 定义别名

   ```sql
   SELECT 字段名1 AS 别名1, 字段名2 AS 别名2, 字段名3, ... FROM 表名;
   
   AS可以省略
   SELECT 字段名1 别名1, 字段名2 别名2, 字段名3, ... FROM 表名;
   ```

   

3. 去除重复行

   ```sql
   SELECT DISTINCT 字段名 FROM 表名;
   ```




### DML_DQL小结

| DML语句操作 | 关键字                                     |
| ----------- | ------------------------------------------ |
| 添加        | INSERT INTO 表名 (字段名) VALUES (字段值); |
| 修改        | UPDATE 表名 SET 字段名=值;                 |
| 删除        | DELETE FROM 表名;                          |
| 查询        | SELECT * FROM 表名;                        |



## DQL查询语句-条件查询(重要)

前面我们的查询都是将所有数据都查询出来，但是有时候我们只想获取到**满足条件**的数据

### 目标

学习条件查询语法格式

### 讲解


语法格式：

```sql
SELECT * FROM 表名 WHERE 条件;
```

流程：取出表中满足条件的记录

#### 准备数据

```sql
CREATE TABLE student3 (
  id int,
  name varchar(20),
  age int,
  sex varchar(5),
  address varchar(100),
  math int,
  english int
);

INSERT INTO student3(id,NAME,age,sex,address,math,english) VALUES (1,'马云',55,'男','杭州',66,78),(2,'马化腾',45,'女','深圳',98,87),(3,'马景涛',55,'男','香港',56,77),(4,'柳岩',20,'女','湖南',76,65),(5,'柳青',20,'男','湖南',86,NULL),(6,'刘德华',57,'男','香港',99,99),(7,'马德',22,'女','香港',99,99),(8,'德玛西亚',18,'男','南京',56,65);
```


#### 比较运算符

`>`大于 
`<`小于
`<=`小于等于
`>=`大于等于
`=`等于
`<>`、`!=`不等于

具体操作：

* 查询math分数大于80分的学生

```sql
SELECT * FROM student3 WHERE math>80;
```

![](mysql-01.assets/where查询01.png)

* 查询english分数小于或等于80分的学生

```sql
SELECT * FROM student3 WHERE english<=80;
```

![](mysql-01.assets/where查询02.png)

* 查询age等于20岁的学生

```sql
SELECT * FROM student3 WHERE age=20;
```

![](mysql-01.assets/where查询03.png)

* 查询age不等于20岁的学生

```sql
SELECT * FROM student3 WHERE age!=20;
SELECT * FROM student3 WHERE age<>20;
```

![](mysql-01.assets/where查询04.png)



#### 逻辑运算符

`and(&&)` 多个条件同时满足
`or(||)` 多个条件其中一个满足
`not(!)` 不满足

具体操作：

* 查询age大于35且性别为男的学生(两个条件同时满足)

```sql
SELECT * FROM student3 WHERE  age>35 AND sex='男';
```

![](mysql-01.assets/where查询05.png)

* 查询age大于35或性别为男的学生(两个条件其中一个满足)

```sql
SELECT * FROM student333 WHERE age>35 OR sex='男';
```

![](mysql-01.assets/where查询06.png)

* 查询id是1或3或5的学生

```sql
SELECT * FROM student3 WHERE id=1 OR id=3 OR id=5;
```

![](mysql-01.assets/where查询08.png)



**in关键字**
语法格式：

```sql
SELECT * FROM 表名 WHERE 字段名 IN (值1, 值2);
```

`in`里面的每个数据都会作为一次条件，只要满足条件的就会显示

具体操作：

* 查询id是1或3或5的学生

```sql
SELECT * FROM student3 WHERE id IN (1,3,5);
```

![](mysql-01.assets/where查询08.png)


* 查询id不是1或3或5的学生

```sql
SELECT * FROM student3 WHERE id NOT IN (1,3,5);
```

![](mysql-01.assets/where查询07.png)

#### 范围

```sql
SELECT * FROM 表名 WHERE 字段名 BETWEEN 值1 AND 值2;
```

比如：`age BETWEEN 80 AND 100`
相当于： `age>=80 && age<=100` 

具体操作：

* 查询english成绩大于等于75，且小于等于90的学生

```sql
SELECT * FROM student3 WHERE english>=75 AND english<=90;
SELECT * FROM student3 WHERE english BETWEEN 75 AND 90;
```

![](mysql-01.assets/where查询09.png)



### 小结

比较运算符

```
>
<
>=
<=
!=, <>
=
```

逻辑运算符

```
&& AND
|| OR
! NOT
```

IN

```
SELECT * FROM 表名 WHERE 字段名 IN (值1, 值2, 值3);
```

BETWEEN  AND

```
SELECT * FROM 表名 WHERE 字段名 BETWEEN 值1 AND 值2;
```




## 模糊查询like(重要)

### 目标

学习模糊查询语法格式

![](mysql-01.assets/where查询11.png)



### 讲解

`LIKE`  像什么什么一样

```sql
SELECT * FROM 表名 WHERE 字段名 LIKE '通配符字符串';
```

满足`通配符字符串`规则的数据就会显示出来

MySQL通配符有两个：
`%`: 表示任意多个字符
`_`: 表示一个字符

具体操作：

* 查询姓马的学生

```sql
SELECT * FROM student3 WHERE NAME LIKE '马%';
```

![](mysql-01.assets/where查询10.png)

* 查询姓名中包含'德'字的学生

```sql
SELECT * FROM student3 WHERE NAME LIKE '%德%';
```

![](mysql-01.assets/where查询11.png)

* 查询姓马，且姓名有三个字的学生

```sql
SELECT * FROM student3 WHERE NAME LIKE '马__';
```

![](mysql-01.assets/where查询12.png)

### 小结

模糊查询格式

```sql
SELECT * FROM 表名 WHERE 字段名 LIKE '通配符字符串';
```

`%`: 表示任意多个字符
`_`: 表示一个字符



## DQL查询语句-排序(重要)

### 目标

学习对查询的数据进行排序

![](mysql-01.assets/orderby01.png)



### 讲解

通过`ORDER BY`子句，可以将查询出的结果进行排序(排序只是显示方式，不会影响数据库中数据的顺序)

```sql
SELECT * FROM 表名 ORDER BY 字段名 ASC|DESC;
```

**ASC**: 升序
**DESC**: 降序

1.2.1 单列排序

单列排序就是使用一个字段排序

具体操作：

* 查询所有数据,使用年龄降序排序

```sql
select * FROM student3 order by age DESC;
```

![](mysql-01.assets/orderby01.png)

1.2.2 组合排序

组合排序就是先按第一个字段进行排序，如果第一个字段相同，才按第二个字段进行排序，依次类推。
上面的例子中，年龄是有相同的。当年龄相同再使用math进行排序

```sql
SELECT * FROM 表名 WHERE 条件 ORDER BY 字段名 [ASC|DESC], 字段名 [ASC|DESC];
```

具体操作：

* 查询所有数据,在年龄降序排序的基础上，如果年龄相同再以数学成绩降序排序

```sql
SELECT * FROM student3 ORDER BY age DESC, math DESC;
```

![](mysql-01.assets/orderby02.png)

### 小结

1. 排序的关键字

   ```sql
   ORDER BY 字段名 [ASC|DESC];
   
   |: 表示或者
   []: 表示可省略
   ```

2. 升序：ASC, 默认

3. 降序：DESC



## 总结

1. 能够使用SQL语句建库、建表
   建库: CREATE DATABASE 数据库名;

建表: CREATE TABLE 表名 (字段名 字段类型);

2. 能够使用SQL语句进行数据的添删改查操作

   添加数据: INSERT INTO 表名 (字段名1, 字段名2, ...) VALUES (值1, 值2, ...);

   修改数据: UPDATE 表名 SET 字段名=值;

   删除数据: DELETE FROM 表名;

   查询数据: SELECT * FROM 表名;

3. 能够使用SQL语句进行排序

   SELECT * FROM 表名 ORDER BY 字段名 [ASC|DESC];

   默认是升序

   ASC: 表示升序

   DESC: 表示降序

