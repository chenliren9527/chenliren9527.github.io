---
title: 就业技术加强(02)-MySQL高级
tags:
  - 笔记
  - 就业技术加强
  - MySQL
  - 索引优化
  - 慢查询
  - Explain执行计划
categories:
  - 就业技术加强
date: 2021-01-24 17:19:10
---

##  课程目标

目标1: 能够描述索引的作用及优劣势

目标2: 能够说出索引的底层数据结构

目标3: 能够区别聚簇索引和非聚簇索引

目标4: 能够使用Explain执行计划

目标5: 能够描述SQL优化的方案



## 00、MySQL体系结构

![1592233894420](就业技术加强-02-MySQL高级.assets/image-20210116181507529.png)



**客户端连接**

​    支持接口：支持的客户端连接，例如 C、Java、PHP 等语言来连接 MySQL 数据库。

**第一层：网络连接层**

​    连接池：管理、缓冲用户的连接，线程处理等需要缓存的需求。

**第二层：核心服务层**

​    管理服务和工具：系统的管理和控制工具，例如备份恢复、复制、集群等。 

​    SQL 接口：接受 SQL 命令，并且返回查询结果。

​    查询解析器：验证和解析 SQL 命令，例如过滤条件、语法结构等。 

​    查询优化器：在执行查询之前，使用默认的一套优化机制进行优化sql语句。

​    缓存：如果缓存当中有想查询的数据，则直接将缓存中的数据返回。没有的话再重新查询。

**第三层：存储引擎层**

​    插件式存储引擎：管理和操作数据的一种机制，包括(存储数据、如何更新、查询数据等)

**第四层：系统文件层**

​    文件系统：配置文件、数据文件、日志文件、错误文件、二进制文件等等的保存。



## 01、MySQL：性能优化概述

> 目标: 了解为什么要进行MySQL优化及优化方案。

在应用开发的过程中，由于前期数据量少，开发人员编写的SQL语句或者数据库整体解决方案都更重视在功能上的实现，但是当应用系统正式上线后，随着生产环境中数据量的急剧增长，很多SQL语句和数据库整体方案开始逐渐显露出了性能问题，对生产的影响也越来越大，此时MySQL数据库的性能问题成为系统应用的瓶颈，因此需要进行MySQL数据库的性能优化。

#### 为什么要进行数据库优化

1. 避免网站页面出现访问错误

   由于数据库连接timeout产生页面5xx错误

   由于慢查询造成页面无法加载

2. 增加数据库的稳定性

   很多数据库问题都是由于低效的查询引起的

3. 优化用户体验

   流畅页面的访问速度




#### 性能下降的原因

+ 数据量太大。
+ 没有建立索引。
+ 建立的索引失效，建立了索引，在真正执行时有用上建立的索引(通过执行explain来解决)。
+ 关联查询太多join查询语句写的不好，各种连接，各种子查询导致用不上索引。
+ 服务器调优及配置参数导致，如果设置的不合理，不恰当，也会导致性能下降，sql变慢（专业DBA）。
+ 系统架构的问题。



#### 常见优化方案【重点】

+ 索引优化: 添加适当索引（index）。

+ sql优化: 写出高质量的sql，避免索引失效。

+ 关联查询: 不允许超过3张表，如果超过说明你表设计不合理，就进行优化和合并。

+ 设计优化: 表的设计合理化(符合3NF，有时候要进行反三范式操作）。

+ 架构优化: 分表技术(水平分割、垂直分割)主从复制，读写分离。

+ 配置优化: 对MySQL配置优化（配置最大并发数my.ini, 调整缓存大小）。

  > 调整MySQL参数配置（DBA）也需要硬件上的支持 
  >
  > ![1592233894420](就业技术加强-02-MySQL高级.assets/1592233894420.png)

+ 考虑其他的存储方式（redis/mongodb/es/solr等）。

+ 直接修改MySQL内核（阿里）。

+ 硬件优化: 增加硬件（CPU/内存/硬盘）。

  ![1592233924746](就业技术加强-02-MySQL高级.assets/1592233924746.png)  



## 02、MySQL：存储引擎

**存储引擎介绍**

在生活中，引擎就是整个机器运行的核心(发动机)，不同的引擎具备不同的功能，应用于不同的场景之中。

![image-20210117174704515](就业技术加强-02-MySQL高级.assets/image-20210117174704515.png)

Oracle、SqlServer 等数据库只有一种存储引擎。而MySQL针对不同的需求，配置不同的存储引擎，就会让数据库采取不同处理数据的方式和扩展功能。

简单的理解：MySQL中不同的存储引擎，使用不同的方式将数据保存到文件中。

MySQL 支持的存储引擎有很多，常用的有三种：InnoDB、MyISAM、MEMORY。

特性对比

​    MyISAM 存储引擎：访问快，不支持事务和外键操作。

​    InnoDB 存储引擎：支持事务和外键操作，支持并发控制，占用磁盘空间大。(MySQL 5.5版本后默认)

​    MEMORY 存储引擎：内存存储，速度快，不安全。适合小量快速访问的数据。

**注意: MySQL5.5后默认的存储引擎是InnoDB。**



**查看存储引擎**

我们可以通过命令查看MySQL数据库有9种数据存储引擎: 

```sql
show engines;
```

![1592041303054](就业技术加强-02-MySQL高级.assets/1592041303054.png) 



**修改存储引擎**

```sql
-- 修改存储引擎为MyISAM
ALTER TABLE 表名 ENGINE = MyISAM;

-- 修改存储引擎为InnoDB
ALTER TABLE 表名 ENGINE = InnoDB;
```



## 03、MySQL：磁盘存储和索引概念

> 目标: 掌握和了解MySQL的索引原理及相关概念

+ **磁盘存取示意图**

  每次执行SQL从磁盘中查找数据称为磁盘I/O，而磁盘IO至少要经历磁盘寻道、磁盘旋转、数据读取等等操作，非常消耗性能，所以对于读取数据，最大的优化就是减少磁盘I/O。

  ![img](就业技术加强-02-MySQL高级.assets/mysql004.png) 



什么是数据结构: 计算机存储、组织数据的方式。常见的数据结构有数组，二叉树，红黑树，堆，栈，队列等。设计这些数据结构的目的就是为：统一数据的存储和快速查找数据。选择不同的数据结构，它性能都不一样。

什么是索引：是帮助数据库快速查询数据的技术。**在MySQL中索引使用B+Tree(N叉树)数据结构**，索引类似新华字典的索引目录，可以通过索引目录快速查到你想要的字快速查找数据。索引是解决SQL性能问题的重要手段之一，使用索引可以帮助用户解决大多数的SQL性能问题。


查询MySQL一页的大小:

```sql
SHOW GLOBAL STATUS LIKE 'innodb_page_size';
```



+ **执行sql的过程**

  + 会发起一起磁盘IO (寻道 + 旋转 + 读取数据 + 返回数据存储 内存的过程)。7200转/min，旋转一周需要8.33ms，寻道约10ms。

  + 如果条件不满足就会不停的寻道和发起IO。

    ![1592234072751](就业技术加强-02-MySQL高级.assets/没有索引.png)

  

+ 怎么解决这个问题呢？没错就是优化表，对表的数据进行重新编排和建立目录映射。其实就是优化数据的存储结构，就是建立索引。

  ![1592234072751](就业技术加强-02-MySQL高级.assets/1592234072751.png)  

  

+ **小结**

+ + 每次SQL语句都会进全表的匹配查询，每一次的匹配都是一次IO的寻道的过程，这个性能极低。这样为什么当数据越来越大的时候，SQL的执行速度变慢的原因也是罪魁祸首，怎么解决：索引。
  + 建立索引的目的: 就是对数据重新整理的过程，目的**减少IO的寻道的次数**。
  + **索引是一种数据结构（B+tree）**这种数据结构：就是对表的数据重新整理的过程，把表的记录建立映射关系。






## 04、MySQL：索引底层实现

> 目标: 了解MySQL索引底层实现

Btree 还是 B+tree 都是通过最原始的数据的结构 **二叉树** 演变而来。

#### 4.1 二叉树

参考网站: https://www.cs.usfca.edu/~galles/visualization/BST.html

![1592234099209](就业技术加强-02-MySQL高级.assets/1592234099209.png)  



![image-20210116221348314](就业技术加强-02-MySQL高级.assets/image-20210116221348314.png)

为了加快数据的查找，可以维护**二叉查找树**，每个节点分别包含索引键和一个指向对应数据记录的物理地址的指针，这样就可以运用二叉查找在一定的复杂度内获取相应的数据，从而快速的检索出符合条件的记录。

二叉树作为索引的缺点：从二叉树的查找过程了来看，最坏的情况下磁盘IO的次数由树的高度来决定。二叉树只能存两个节点，层级越来越大越来越深，发生磁盘的IO会越频繁。

从前面分析情况来看，减少磁盘IO的次数就必须要压缩树的高度，让瘦高的树尽量变成矮胖的树，所以B-Tree强势登场。

#### 4.2 B-Tree

参考网站: https://www.cs.usfca.edu/~galles/visualization/BTree.html 

> **[B-tree](http://baike.baidu.com/subview/363832/363832.htm)树即[B树](http://baike.baidu.com/subview/298408/298408.htm)**，B即**Balanced(平衡的意思)**，**B-Tree**又称为多路平衡查找树。因为B树的原英文名称为B-Tree，B-Tree是为磁盘等待外存设备设计的一种平衡查找树。每个节点包含key和data。

系统从磁盘读取数据到内存时以磁盘块block为基本单位的，位于同一个磁盘块中的数据会被一次性读取出来，不是需要什么取什么。

B树是一种多路平衡搜索树，它类似普通的二叉树，但是Btree允许每个节点有更多的子节点。   

![image-20210116223652300](就业技术加强-02-MySQL高级.assets/image-20210116223652300.png)

完整示意图:

![img](就业技术加强-02-MySQL高级.assets/20190813145314735-1582474963201.png) 

模拟查找关键字29的过程:

+ 找到根节点对应的磁盘块1，读入内存。【磁盘I/O操作第1次】
+ 比较关键字29在区间（17,35）。
+ 找到磁盘块3，读入内存。【磁盘I/O操作第2次】
+ 比较关键字29在区间（26,30）。
+ 找到磁盘块8，读入内存。【磁盘I/O操作第3次】
+ 在磁盘块8中的关键字列表中找到关键字29。

> 分析上面过程，发现需要3次磁盘I/O操作。由于内存中的关键字是一个有序表结构，可以利用二分法查找提高效率。而3次磁盘I/O操作是影响整个B-Tree查找效率的决定因素。B-Tree相对于二叉树缩减了节点个数，使每次磁盘I/O取到内存的数据都发挥了作用，从而提高了查询效率。

**特点说明**

- **平衡查询树，它对数据会进行自我平衡，**它比二叉树的层级要低，所以查询的性能要比二叉树高很多。
- 和二叉树一样比父节点大的数据存储在右边，小的存储在左边。
- 度（degree)节点的数据存储个数。度越深代表存储的数据越密，树的层级和高度就越低。越利于搜索和存储数据。评价一个索引的好坏一定是进入索引的次数越小越快。
- 节点中数据key从左到右递增排列

**缺点说明**

- Btree数据是存储到每个节点中，所以每次查询的和维护的时候就会维护索引值又维护了数据，这样会就是造成内存的浪费和性能的消耗。这也是B+TREE优化的地方。
- 为了提升度的长度，还需要对这种数据结构进行优化，所以它的升华版B+Tree诞生了。

#### 4.3 B+Tree

参考网站: https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html 

>在B-Tree基础上进行优化，使其更适合实现外存储索引结构。InnoDB就是存储引擎就是用B+Tree。 
>
>在B-Tree中每一个节点存储空间有限，如果data数据较大，会导致每个节点key太小，当数据量很大时也会导致B-Tree深度较大，增大查询的磁盘IO次数，影响查询效率。
>
>在B+Tree中，所有数据记录节点都是按照键值大小顺序存放在同一层叶子节点上，而非叶子节点上只存储key值信息，可以大大增大每个节点存储的key值的数量，降低B+Tree的高度。

 B+树是Btree的变体，也是一种多路平衡查找树，B+树的示意图为:

![image-20191211212214287](就业技术加强-02-MySQL高级.assets/image-20191211212214287.png) 



![image-20210111162040037](就业技术加强-02-MySQL高级.assets/image-20210111162040037.png)

**特点说明**

+ 非叶子节点只存储 key 值，一个磁盘块可以存储更多的key，叶子节点存储key和数据，所有叶子节点之间都有连接指针，提高区间访问能力。

B+Tree**索引的性能分析**

```
一般使用磁盘I/O次数评价索引结构的优劣

B+Tree的度一般会超过100，因此h非常小 (一般为3到5之间)，减少磁盘的IO次数，性能非常稳定
B+Tree叶子节点有顺序指针，更容易做范围查询
存储器读取数据按磁盘块读取 
每个磁盘块的大小为扇区（页）的2的N次方
每个扇区的最小单位512B不同的生产厂家不同
```

**小结**

你对索引理解是什么？

+ 建立索引的作用：减少磁盘IO，提高数据的查询效率。

+ 索引是一种数据结构。MySQL的索引使用B+Tree。

+ B+Tree 数据结构

  ​    非叶子节点只存储 key 值。

  ​    所有数据存储在叶子节点。

  ​    所有叶子节点之间都有连接指针。



#### 4.4 MyISAM非聚簇

非聚簇是指: MyISAM索引文件和数据文件是分离的，叶子节点存储的是数据的磁盘地址，非主键索引和主键索引类似。

![img](就业技术加强-02-MySQL高级.assets/mysql010.png) 





![1592042928198](就业技术加强-02-MySQL高级.assets/1592042928198.png) 

说明: .frm 表结构文件、.MYD 表数据文件、.MYI 表索引文件。



#### 4.5 InnoDB聚簇

>数据文件: 本身就是索引文件
>表数据文件本身就是按照B+Tree组织的一个索引结构文件
>主键索引-叶节点包含了完整的数据记录，非主键索引的叶子节点指向主键.

![img](就业技术加强-02-MySQL高级.assets/mysql009.png) 

+ InnoDB: 没有.MYD和.MYI，.frm文件（包括的数据结构和索引结构），其数据文件对应于ibdata(1..n)文件中。
+ InnoDB: 在实际的开发中一定要建立主键，因为索引都是通过主键索引获取和查找的。在开发过程中如果你使用的是Innodb引擎一定要建立一个主键列。



BAT面试题:

对于InnoDB引擎如果不建立主键MySQL数据库会怎么处理?

1. MySQL会找唯一字段作为主键
2. 如果没有唯一字段MySQL生成伪列当做主键，开发人员看不见。

为什么主键使用UUID不好？为什么使用自增主键最好？

1. UUID是字符串，排序时需要计算字符串的ASCII码值，降低效率。
2. UUID不是连续的，增加数据时对B+树的改动较大，降低效率。



InnoDB使用主键索引和普通索引查询时如何走索引的，哪个效率高?

```sql
select * from t where id = 20;

select * from t where name = Jim;
```





## 05、MySQL：索引优劣势&注意事项

#### 5.1 索引优势

+ 可以通过建立唯一索引或者主键索引，保证数据库表中每一行数据的唯一性。

+ 建立索引，减少表的检索行数，大大提高查询性能。

+ 在表连接的连接条件可以加速表与表之间的相连。



#### 5.2 索引劣势

+ 在创建索引和维护索引会耗费时间，随着数据量的增加而增加。

+ 索引文件会占用物理空间，除了数据表需要占用物理空间之外，每一个索引还会占用一定的物理空间。

+ 当对表的数据进行INSERT，UPDATE，DELETE 的时候，索引也要动态的维护，这样就会降低增删改数据的速度。(建立索引会占用磁盘空间的索引文件。一般情况这个问题不太严重，但如果你在一个大表上创建了多种组合索引，索引文件的会膨胀很快)。


#### 5.3 注意事项【重点】

+ 建立索引后需要注意的问题?

  ```txt
  1. 增，删，改都会触发索引重建，并且会产生锁。
  2. 删除尽量逻辑删除，不要物理删除，防止索引重建（is_delete=0 未删除  1 删除）
     什么叫逻辑删除: update table where is_delete =0 where id =1
  3. 尽量不要更新索引的列。
  ```

+ 哪些列适合建立索引?

  + 不经常变动的列: 主键、电话、手机号码、邮箱、订单编号、UUID、身份证。

- 哪些列不适合建立索引?
  - 经常变动的列: 用户名，余额。
  - 数据比例种类太少的列（数据离散程度不高，可能存在大量重复数据的列）: 年龄、性别、状态。

 **公式: 用当前表中某列的去重数/表总记录数 == 1(接近1)，那么这种列就特别适合建立索引。**



## 06、MySQL：索引的分类

+ 主键索引: primary key（不为空且唯一）

+ 唯一索引: unique index（唯一）

+ 普通索引: index(id) 

+ 联合索引(组合索引):

  + primary key(id, name): 联合主键索引
  + unique index(id, name): 联合唯一索引
  + index(id, name): 联合普通索引

+ 全文索引: fulltext index(字段名) 用于搜索很长一篇文章的时候，效果比较好。

  说明: 最好还是用全文搜索服务Elasticsearch、solr来解决。





## 07、MySQL：索引的操作

#### 7.1 查看索引

+ 语法

  ```sql
  -- SHOW INDEX FROM 表名
  show index from system_user;
  ```

  ![1592112078094](就业技术加强-02-MySQL高级.assets/1592112078094.png) 


#### 7.2 创建索引

+ 语法

  ```sql
  -- 创建主键索引: alter table 表名 add primary key(列名)
  ALTER TABLE goods ADD PRIMARY KEY (id);
  
  -- 创建唯一索引: CREATE UNIQUE INDEX 索引名 ON 表名(列名)
  CREATE UNIQUE INDEX idx_name ON goods(NAME);
  
  -- 创建普通索引: CREATE INDEX 索引名 ON 表名(列名)
  CREATE INDEX idx_price ON goods(price);
  ```

+ 查询对比

  + 没有使用索引查询

    ```sql
    -- 根据手机号码查询
    select * from system_user where user_phone = '15400008514';
    ```

    ![1592111749197](就业技术加强-02-MySQL高级.assets/1592111749197.png) 

  + 使用索引查询

    ```sql
    -- 创建索引
    create index user_phone_index on system_user(user_phone);
    -- 根据手机号码查询
    select * from system_user where user_phone = '15400008514';
    ```

    ![1592111992442](就业技术加强-02-MySQL高级.assets/1592111992442.png) 



- 测试某些列不适合建立索引

  - 通过性别查询用户

    ```sql
    -- 通过性别查询用户
    SELECT * FROM SYSTEM_USER WHERE user_sex=1 LIMIT 100000000;
    ```

    ![image-20210109113127071](就业技术加强-02-MySQL高级.assets/image-20210109113127071.png)

  - 给性别建立索引

    ```sql
    -- 给性别建立索引
    CREATE INDEX user_sex_index ON SYSTEM_USER(user_sex);
    
    -- 通过性别查询用户
    SELECT * FROM SYSTEM_USER WHERE user_sex=1 LIMIT 100000000;
    ```

    ![image-20210109113217352](就业技术加强-02-MySQL高级.assets/image-20210109113217352.png)

#### 7.3 删除索引

+ 语法

  ```sql
  -- 删除主键索引(非自增长): alter table 表名 drop primary key
  ALTER TABLE goods DROP PRIMARY KEY;
  
  -- 删除唯一索引和普通索引: DROP INDEX 索引名 ON 表名
  DROP INDEX idx_name ON goods;
  DROP INDEX idx_price ON goods;
  ```

#### 7.4 查看索引详解

+ 语法

  ```sql
  -- SHOW INDEX FROM 表名
  show index from system_user;
  ```

  ![1592112078094](就业技术加强-02-MySQL高级.assets/1592112078094.png) 

  + Table: 表的名称。
  + Non_unique: 如果索引不能包括重复词则为0，如果可以重复则为1。
  + Key_name: 索引的名称。
  + Seq_in_index: 索引中的序列号，从1开始。
  + Column_name: 列名称。
  + Collation: 列以什么方式存储在索引中。在MySQL中，‘A’(升序)或NULL(无分类)。
  + **Cardinality**: 基数。**基数**越大，越适合建立索引，当进行查询时，MySQL使用该索引的机会就越大。
  + Sub_part: 如果列只是被部分地编入索引，则为被编入索引的字符的数目。如果整列被编入索引，则为NULL。
  + Packed: 关键字如何被压缩。如果没有被压缩，则为NULL。
  + Null: 如果列含有NULL，则含有YES。如果没有，则该列含有NO。
  + Index_type: 索引存储数据结构（BTREE, FULLTEXT, HASH, RTREE）。
  + Comment: 评论。




## 08、MySQL：执行计划Explain【重点】

> 目标: 掌握explain执行计划的使用



### 8.1 语法

```mysql
-- 使用Explain关键字 放到sql语句前
explain select * from system_user where id = 1;
```

![1592140051173](就业技术加强-02-MySQL高级.assets/1592140051173.png) 

### 8.2 作用

+ 通过explain可以让我们知道SQL语句的执行顺序。
+ 通过explain可以让我们知道SQL查询语句是否命中索引。
+ 通过explain可以分析查询语句或表结构的性能瓶颈。



### 8.3 列名

| 列名                                 | 说明                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| <font color="red">**id**</font>      | 查询标识符，SQL执行的顺序标识符，SQL从大到小的执行。         |
| select_type                          | 显示本行是简单或复杂查询t。如果查询有任何复杂的子查询，则最外层标记为PRIMARY（DERIVED、UNION、UNION RESUlT） |
| <font color="red">**table**</font>   | 访问哪张表                                                   |
| partitions                           | 在分区数据库中，一个表可以分布在一个或多个数据库分区中       |
| <font color="red">**type**</font>    | 访问类型（ALL、index、range、ref、eq_ref、const、system）    |
| possible_keys                        | 显示MySQL可能采用哪些索引来优化查询                          |
| <font color="red">**key**</font>     | 显示MySQL决定采用哪个索引来优化查询                          |
| <font color="red">**key_len**</font> | 显示索引所使用的字节数                                       |
| ref                                  | 显示查询时所引用到的索引(关联查询)                           |
| rows                                 | 粗略估算整个查询需要检查的行数(如果值越大，查询速度就越慢)   |
| filtered                             | 过滤行的百分比                                               |
| Extra                                | 额外信息（用了where条件、用了排序、用了分组）等              |

#### 8.3.1 id 查询标识符【重点】

> id查询标识符，SQL执行的顺序的标识，SQL从大到小的执行。

+ id值相同，由上到下执行。

  ```sql
  -- id值相同, 由上到下执行
  EXPLAIN SELECT * from employee a, department b, customer c where a.dep_id = b.id and a.cus_id = c.id;
  ```

  ![1599573587839](就业技术加强-02-MySQL高级.assets/1599573587839.png)  

+ id值不同，由大到小执行。

  ```sql
  -- id值不同, 由大到小执行
  EXPLAIN SELECT * from department WHERE id = (SELECT id from employee WHERE id=(SELECT id from customer WHERE id = 1));
  ```

  ![1599573550579](就业技术加强-02-MySQL高级.assets/1599573550579.png)  

+ id值相同不同，不同的由大到小执行，相同的由上到下执行。

  ```sql
  -- id值相同 不同都存在, 不同的由大到小执行，相同的由上到下执行 deriverd 衍生出来的虚表
  EXPLAIN select * from department d, (select dep_id from employee group by dep_id) t where d.id = t.dep_id;
  ```

  ![1599573491950](就业技术加强-02-MySQL高级.assets/1599573491950.png)  

  

#### 8.3.2 select_type 查询类型

查询类型: 主要用来区分是普通查询、联合查询、子查询等。

| 查询类型     | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| SIMPLE       | 简单查询，查询中不包含子查询或者union                        |
| PRIMARY      | 查询中如果包含子查询或union，最外层查询则被标记为primary     |
| SUBQUERY     | 查询时包含了子查询                                           |
| DERIVED      | from列表中包含了子查询就会标记为derived(派生表)，查询结果放在临时表中 |
| UNION        | 查询时包含了union查询                                        |
| UNION RESULT | UNION合并的结果集                                            |

```sql
-- union联合查询
EXPLAIN select * from employee e LEFT JOIN department d on e.dep_id = d.id
UNION 
select * from employee e RIGHT JOIN department D ON e.dep_id = d.id;
```

![1599573795574](就业技术加强-02-MySQL高级.assets/1599573795574.png)  

>union: 取并集，过滤重复。（如果两条SQL语句出现相同的就会过滤掉）
>
>union all: 取并集，不过滤重复。



#### 8.3.3 table 表

> 这个查询是访问哪张表



#### 8.3.4 type 访问类型【重点】

衡量你的sql语句性能好坏的参考指标，<font color="red">**以消除ALL为己任**</font>，如果出现ALL代表没有命中索引，是全表查询。![1592189404654](就业技术加强-02-MySQL高级.assets/1592189404654.png) 

+ system: 主键、唯一索引扫描(表中只有一行数据，`MyISAM`引擎)

  ```sql
  EXPLAIN SELECT id FROM customer WHERE id = 1;
  ```

  ![image-20210109152553676](就业技术加强-02-MySQL高级.assets/image-20210109152553676.png)

+ const: 主键、唯一索引扫描(主键列、唯一列查询)

  ```sql
  -- 表示通过索引一次就找到了，const用于primary key 或者 unique索引。
  explain select * from system_user where id = 1;
  ```

  ![1599575058066](就业技术加强-02-MySQL高级.assets/1599575058066.png)  

+ eq_ref: 主键、唯一索引扫描(关联查询比较时用)

  ```sql
  -- 常见于主键或唯一索引扫描，一对一的情况下会出现比较多
  EXPLAIN select * from employee e, department d where e.id = d.id;
  ```

  ![1599575109903](就业技术加强-02-MySQL高级.assets/1599575109903.png)  

+ ref: 非唯一索引扫描(index)

  ```sql
  EXPLAIN select * from system_user where user_phone = '15400008514';
  ```

  ![1599575236809](就业技术加强-02-MySQL高级.assets/1599575236809.png)  

+ range: 范围索引扫描(between、in、>=、like)等操作

  ```sql
  EXPLAIN select * from system_user where id < 3;
  EXPLAIN SELECT * FROM SYSTEM_USER WHERE id IN (3, 5, 7, 8);
  EXPLAIN select * from system_user where user_phone like '1540000851%';
  ```

  ![1599575288190](就业技术加强-02-MySQL高级.assets/1599575288190.png)  

+ index: 全索引扫描，遍历整个索引树

  ```sql
  -- index（覆盖索引）index类型遍历整个索引树
  explain select id from employee;
  ```

  ![1599575368079](就业技术加强-02-MySQL高级.assets/1599575368079.png)  

+ ALL: 全表扫描，MySQL将遍历全表数据直至找到匹配的行(**不走索引**)

  ```sql
  -- 全表进行扫描，从硬盘当中读取数据，如果出现了All 数据量非常大, 一定要去做优化。
  explain select * from employee;
  ```

  ![1599575400134](就业技术加强-02-MySQL高级.assets/1599575400134.png)  

  > 说明: 一般来说，保证查询至少达到range级别，最好能达到ref。
  >
  > 注意: 数据很少的情况下，MySQL优化器可能会直接走表查询而不会走索引查询。因为MySQL优化器觉得走表或走索引是差不多的。



#### 8.3.5 possible_keys

> 显示MySQL可能采用哪些索引来优化查询

```mysql
-- 为dep_id创建普通索引
create index dept_index on employee(dep_id);
--  可能不会用到索引，实际用到索引
explain select dep_id from employee;
```

![1599575621636](就业技术加强-02-MySQL高级.assets/1599575621636.png)  

```mysql
-- 可能会使用索引，实际没用到索引
explain select * from employee e, department d where e.dep_id = d.id;
```

![1599575716375](就业技术加强-02-MySQL高级.assets/1599575716375.png)  



#### 8.3.6 key 采用索引【重点】

> 显示MySQL决定采用哪个索引来优化查询

```sql
explain select * from employee where id = 1;
```

![1599575762875](就业技术加强-02-MySQL高级.assets/1599575762875.png)  



#### 8.3.7 key_len 索引长度

```mysql
-- 表示索引所使用的字节数，可以通过该列计算出使用的索引长度
explain select * from employee where dep_id = 1 and name = '鲁班' and age = 10;
```

![1599575852194](就业技术加强-02-MySQL高级.assets/1599575852194.png)  



#### 8.3.8 ref 引用索引

> 显示查询时所引用到的索引。

```mysql
-- 显示查询时所引用到的索引
explain select e.dep_id from employee e, department d, customer c where e.dep_id = d.id and e.cus_id = c.id and e.name = '鲁班';
```

![1599576594218](就业技术加强-02-MySQL高级.assets/1599576594218.png)  



#### 8.3.9 rows 检查行数【重点】

> 粗略估算整个查询需要检查的行数(如果值越大，查询速度就越慢)

它体现建立索引以后，优化的行数，越小速度越快，如果你建立索引，type=ref并且也命中到了，但是row还很大，那说明该表已经没有建立索引的意义了，已经优化到了极限(索引不是万能的)，你需要通过其它持术手段来优化。

比如：分库分表、缓存、把数据迁移到es/solr。

```sql
create index user_name_index on system_user(user_name);

explain select * from system_user where user_name like '田%';
```

![1599576836770](就业技术加强-02-MySQL高级.assets/1599576836770.png)  



#### 8.3.10 Extra 额外信息

> 额外信息（用了where条件、用了排序、用了分组）等

```sql
-- Using where; Using filesort
explain select * from employee where dep_id = 1 ORDER BY cus_id;
```

![1599577027124](就业技术加强-02-MySQL高级.assets/1599577027124.png)  

```mysql
-- Using where; Using temporary; Using filesort
explain select cus_id from employee where dep_id in (1,2,3) GROUP BY cus_id;
```

![1599577158033](就业技术加强-02-MySQL高级.assets/1599577158033.png)  





## 09、MySQL：避免索引失效【重点】

> 目标: 掌握如何避免查询时索引失效问题。

- 全部匹配

  ```sql
  -- 删除部门索引 
  drop index dept_index on employee;
  -- 创建联合索引
  create index name_dep_age_idx on employee(name, dep_id, age);
  -- 索引列全部用上(命中索引)
  explain select * from employee where name = '鲁班' and dep_id = 1 and age = 10;
  ```

  ![1599577268986](就业技术加强-02-MySQL高级.assets/1599577268986.png)  

- 联合索引的最左匹配原则

  ```sql
  -- 去掉name条件(索引失效)
  explain select * from employee where dep_id = 1 and age = 10;
  ```

  ![1599577322890](就业技术加强-02-MySQL高级.assets/1599577322890.png)  

  ```sql
  -- 去掉dep_id(命中name索引)
  explain select * from employee where name = '鲁班' and age = 10;
  ```

  ![1599577361710](就业技术加强-02-MySQL高级.assets/1599577361710.png)  

  ```sql
  -- 顺序错乱不会影响最左匹配(命中索引)
  explain select * from employee where dep_id = 1 and age = 10 and name = '鲁班';
  ```

  ![1599577393428](就业技术加强-02-MySQL高级.assets/1599577393428.png)  

- 索引列使用函数

  ```sql
  -- 在name列上加去除空格的函数(索引失效)
  explain select * from employee where TRIM(name) = '鲁班' and dep_id = 1 and age = 10;
  ```

  ![1599577431480](就业技术加强-02-MySQL高级.assets/1599577431480.png)  

- 范围条件

  ```sql
  -- 联合索引使用范围条件(范围后的索引失效)
  explain select * from employee where name = '鲁班' and dep_id > 1 and age = 10;
  ```

  ![1599577477665](就业技术加强-02-MySQL高级.assets/1599577477665.png)  

- 比较运算符(!=或者<>、>、>=)索引可能失效

  ```sql
  -- 创建age_index索引
  create index age_index on employee(age);
  -- 使用不等于(!=或者<>)索引失效
  explain select * from employee where age != 10;
  -- 使用大于(>)索引失效
  explain select * from employee where age > 10;
  -- 使用大于等于(>=)索引失效
  explain select * from employee where age >= 10;
  ```

  ![1599577592670](就业技术加强-02-MySQL高级.assets/1599577592670.png)  

- is not null 索引失效

  ```sql
  -- 创建name_index索引
  create index name_index on employee(name);
  -- is not null
  explain select * from employee where name is not NULL;
  ```

  ![1599577670066](就业技术加强-02-MySQL高级.assets/1599577670066.png)  

- like 索引失效

  ```sql
  -- %开头 (索引失效)
  explain select * from employee where name like '%鲁';
  
  -- x%结尾(命中索引) 推荐写法
  explain select * from employee where name like '鲁%';
  ```

  ![1599577726430](就业技术加强-02-MySQL高级.assets/1599577726430.png) 

  ![1599577762233](就业技术加强-02-MySQL高级.assets/1599577762233.png)  

- 字符串不加引号(索引失效)

  ```sql
  explain select * from employee where name = '200';
  explain select * from employee where name = 200;
  ```

  ![1599577819804](就业技术加强-02-MySQL高级.assets/1599577819804.png) 

- 使用or(索引失效)

  ```sql
  explain select * from employee where name = '鲁班' or age > 10;
  ```

  ![1599577883906](就业技术加强-02-MySQL高级.assets/1599577883906.png)  

- 使用全索引

  ```sql
  -- 全索引: 需要查询的列全部是索引列
  -- 上面情况会触发全表扫描，不过若使用了全索引，则会只扫描索引文件
  explain select name,dep_id,age from employee where name = '鲁班' or age > 10;
  ```

  ![1599578072028](就业技术加强-02-MySQL高级.assets/1599578072028.png)  




**小结**

- 联合索引，**采用最左匹配原则**。
- 索引列，最好不要出现(!=、<>、>、>=、or、is not null、内置函数、数据类型要正确)。
- 如果采用like查询，%在后面走索引。





## 10、MySQL：排序与分组优化

> 如果排序的字段没有建立索引，会使用文件排序（USING filesort），文件排序的效率是相对较低的，我们要避免文件排序。



### 10.1 order by排序

+ 注意1:

  ```sql
  -- 如果排序的字段没有建立索引，会使用文件排序（USING filesort），文件排序的效率是相对较低的，我们要避免文件排序
  explain select * from employee order by name, dep_id, age;
  ```

![1599578194884](就业技术加强-02-MySQL高级.assets/1599578194884.png)  

+ 注意2:

  ```sql
  -- select语句使用到联合索引中的列(最左匹配原则) 使用索引排序
  explain select * from employee where name='鲁班' order by dep_id, age;
  ```

  ![1599578266654](就业技术加强-02-MySQL高级.assets/1599578266654.png)  

+ 注意3:

  ```sql
  -- 联合索引在order by后面顺序不一致。会使用文件排序
  explain select * from employee where name='鲁班' order by age, dep_id;
  ```

  ![1599578376769](就业技术加强-02-MySQL高级.assets/1599578376769.png)  

    

+ 注意4:

  ```sql
  -- 排序使用一升一降，会使用文件排序
  explain select * from employee where name='鲁班' order by dep_id desc, age asc;
  ```

  ![1599578517938](就业技术加强-02-MySQL高级.assets/1599578517938.png)  

<font color="red">**我们的目标是消灭 Using filesort**</font>: 避免排序时使用文件排序。

**注意: order by本身是不会走索引，如果你想要order by走索引，那么必须前面的where一定要有索引，并且遵循最左匹配原则。**



### 10.2 group by分组

> 同order by情况类似，分组必定触发排序。【了解】

```sql
explain select dep_id from employee group by dep_id, age;
explain select dep_id from employee group by name, dep_id, age;
```

![1599579005114](就业技术加强-02-MySQL高级.assets/1599579005114.png) 

![1599579028386](就业技术加强-02-MySQL高级.assets/1599579028386.png)  





## 11、MySQL：大数据量分页优化

分页是我们经常使用的功能，在数据量少时单纯的使用limit m, n 不会感觉到性能的影响，但我们的数据达到成百上千万时，就会明显查询速度越来越低。

```mysql
-- 测试一下大数据量分页查询
-- limit 0,20 时间: 0.002s
select * from system_user limit 0, 20;
-- limit 10000,20 时间: 0.006s
select * from system_user limit 10000, 20;
-- limit 100000,20 时间: 0.048s
select * from system_user limit 100000, 20;
-- limit 1000000,20 时间: 0.991s
select * from system_user limit 1000000, 20;
-- limit 3000000,20 时间: 2.251s
select * from system_user limit 3000000, 20;
```



### 11.1 使用id限定

```mysql
-- 使用id限定方案，将上一页的ID传递过来，根据id范围进行分页查询
-- 通过程序的设计，持续保留上一页的ID，并且ID保证自增
-- limit 3000000,20 时间: 2.266s -> 时间: 0.005s (使用条件有些苛刻 但效率非常高)
select * from system_user where id > 3000000 limit 20;
```



### 11.2 子查询优化

```mysql
-- 通过explain发现，之前我们没有利用到索引，这次我们利用索引查询出对应的所有ID
-- 在通过关联查询，查询出对应的全部数据，性能有了明显提升(失去了默认的按主键id排序)
-- limit 3000000,20 时间:  2.251s -> 时间: 1.364s
explain select * from system_user u, (select id from system_user limit 3000000, 20) t where u.id = t.id;

-- 需要指定id排序
explain select * from system_user u, (select id from system_user ORDER by id limit 3000000, 20) t where u.id = t.id;
```





## 12、MySQL：小表驱动大表



### 12.1 exists介绍

exists：表示存在，当子查询有结果，就会显示主查询中的数据。当子查询没有结果，就不会显示主查询中的数据。

```sql
SELECT * FROM employee WHERE EXISTS (SELECT * FROM employee WHERE id=8);
```



### 12.2 关联查询

MySQL表关联的算法是 Nest Loop Join(嵌套循环连接)，是通过驱动表的结果集作为循环基础数据，然后一条一条地通过该结果集中的数据作为过滤条件到下一个表中查询数据，然后合并结果。如果小的循环在外层，外层表只需要锁5次，如果1000在外，则需要锁1000次，从而浪费资源，增加消耗。这就是为什么要小表驱动大表。

> 总结: 多表关联查询时，一定要让小表驱动大表。



### 12.3 in&exists

```sql
-- 使用in 时间: 8.292ms
select * from system_user where id in (select id from department);

使用department表中数据作为外层循环 5次
for(select id from department d)
	每次循环执行employee表中的查询 300万次
	for( select * from system_user e where e.dep_id=d.id)
```

```sql
-- 使用exists 时间: 14.771ms
SELECT * FROM system_user s WHERE EXISTS (SELECT 1 FROM department d WHERE d.id = s.id);

使用employee表中数据作为外层循环 300万次
for(select * from system_user e)
	每次循环执行department表中的查询 5次
	for( select 1 from department d where d.id = e.dep_id)
```



**小结** 

+ 当A表数据多于B表中的数据时，使用in优于exists。
+ 当A表数据小于B表中的数据时，使用exists优于in。
+ 如果两张表数据量差不多，那么它们的执行性能差不多。





## 13、MySQL：慢查询性能分析

### 13.1 慢查询日志介绍

MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阀值的语句，MySQL的日志是跟踪MySQL性能瓶颈的最快和最直接的方式了，系统性能出现瓶颈的时候，首先要打开慢查询日志，进行跟踪，尽快的分析和排查出执行效率较慢的SQL ,及时解决避免造成不好的影响。

### 13.2 开启慢查询日志

```sql
-- 查看慢查询日志变量
show variables like '%slow_query_log%';
show variables like '%slow_query_log_file%';
show variables like '%long_query_time%';

-- 开启方式一: 只对当前数据库生效，MySQL重启后，会失效 0=OFF 1=ON
set global slow_query_log=1;

-- 开启方式二: 想永久生效，提供配置文件my.ini(推荐)
slow_query_log=1
slow_query_log_file=日志文件存储路径
# 设置慢查询的阈值(默认: 10s)
long_query_time=0.2;
```

![1599580290012](就业技术加强-02-MySQL高级.assets/1599580290012.png)  

```ini
# 开启MySQL慢查询日志,要配置在[mysqld]下方
slow_query_log=1
slow_query_log_file=F:/mysqlslowquery.log
long_query_time=0.2
```

![1599581149494](就业技术加强-02-MySQL高级.assets/1599581149494.png) 

![1599581172576](就业技术加强-02-MySQL高级.assets/1599581172576.png) 

重启MySQL服务:

![1592212408979](就业技术加强-02-MySQL高级.assets/1592212408979.png) 

执行查询:

![1592212487993](就业技术加强-02-MySQL高级.assets/1592212487993.png) 

查看mysqlslowquery.log:

![1599581399892](就业技术加强-02-MySQL高级.assets/1599581399892.png)  

### 13.3 慢查询日志分析

慢查询日志中可能会出现很多的日志记录，我们可以通过慢查询日志工具进行分析，MySQL默认安装了
mysqldumpslow.pl工具实现对慢查询日志信息的分析。

> 检查是否已安装perl环境: perl -v
>
> 说明: 需要安装perl脚本环境下载地址: <https://www.perl.org/get.html> 

```sql
-- 得到返回记录集最多的10个SQL。
perl mysqldumpslow.pl -s r -t 10 F:\mysqlslowquery.log
-- 得到访问次数最多的10个SQL
perl mysqldumpslow.pl -s c -t 10 F:\mysqlslowquery.log

Count: 2（执行了多少次）
Time=6.99s(13s) 平均执行的时间（总的执行时间）
Lock=0.00s(0s)（等待锁的时间）
Rows=1661268.0（每次返回的记录数） (3322536)（总共返回的记录数)
username[password]@[localhost]
```

![1599581881028](就业技术加强-02-MySQL高级.assets/1599581881028.png)   

参数说明:

```txt
-s 按照那种方式排序
   c: 访问统计
   l: 锁定时间
   r: 返回记录
   al: 平均锁定时间
   ar: 平均访问记录数
   at: 平均查询时间
   
-t 是top，返回多少条数据。

-g 可以跟上正则匹配模式，大小写不敏感。
```

![1592210568746](就业技术加强-02-MySQL高级.assets/1592210568746.png)   



你平时如何找到查询慢的SQL,以及如何优化!

1. 开启慢查询日志(DBA)

2. 查找慢查询SQL

3. explain进行慢查询SQL分析

4. SQL语句调优，数据库服务器参数调优(DBA)





## 14、MySQL：索引的选择

### 14.1 建立索引的时机

- 在中小型公司，刚开始开发项目的时候不需要关注太多的性能问题，一般不去建立索引。

- 项目上线运行一段时间，刚开始查询很快，后面就查询很慢，可能需要考虑建立索引。

- 用工具分析慢查询SQL日志文件，如果有查询慢的SQL，把慢查询SQL通过explain执行计划进行分析，然后建立相关的索引。 

- 当你发现一张表的数据量，每天增长的非常大，那这个也需要考虑建立索引了。千万不要等到数据达到几百万甚至上千万的时候去建立索引。

- 如果表的数据量不是很大（几千，几万，几十万）这种表就不需要建立索引。

  > 看到一个表就要去建立索引？千万不能有这样的想法!!!

### 14.2 适合建立索引

+ 表的主键列(primary key)
+ 频繁作为查询条件的列应该创建索引(比如银行系统银行帐号，电信系统的手机号)
+ 查询中与其它表关联的列，外键关系建立索引(比如员工表的部门外键)
+ 查询中排序的列，排序列若通过索引去访问将大大提升排序速度(索引能够提高检索速度和排序速度)
+ 查询中统计或分组的列

### 14.3 不适合建立索引 

+ 记录比较少的表
+ 频繁更新的列不适合建立索引(每次更新不单单更新数据，还要更新索引)
+ 索引提高了查询的速度，同时也会降低更新表的速度，因为建立索引后，如果对表进行INSERT, UPDATE, DELETE, MySQL不仅要保存数据，还要保存索引(delete删除做逻辑删除，更新的时候最好不要更新索引列)
+ where条件里用不到的列
+ 数据重复的列，如果某列包含了许多重复的内容，比如表中的某列为国籍、性别、数据的差异率不高，这种列建立索引就没有太大的意义。

> **公式：用当前表中某列的去重数/表总记录数 ==1（接近1），这种列就特别适合建立索引。**





## 15、MySQL：常见面试题总结

+ 问题1: 下面查询语句，索引的使用情况?

  ```sql
  -- 建立联合索引(a,b,c)，请说出下列条件的索引使用情况
  select * from table where a=4 
   	使用到索引a
   	
  select * from table where a=4 and b=6 
  	使用到了索引a,b
  	
  select * from table where a=4 and c=5 and b=6 
  	使用到了索引a,b,c
  	
  select * from table where b=4 or b=5 
  	没使用到索引
  	
  select * from table where a=4 and c=6 
  	使用到索引a
  	
  select * from table where a=4 and b>5 and c=6 
  	使用到索引a,b
  	
  select * from table where a=4 and b like 'test%' and c=6
  	使用索引a,b b条件相当于范围查询
  	
  select * from table where a=4 order by b,c
  	使用到索引a 不会产生Using FileSort
  	
  select * from table where b=5 order by a
  	没使用索引 产生Using Filesort
  	
  select * from table where b=5 order by c
  	没使用索引 产生Using Filesort
  	
  select * from table where a=5 group by b,c
  	使用索引a 不会产生Using temporary
  ```

+ 问题2: 什么是索引?

  ```txt
  数据库索引的本质是: 数据结构 是一种b+tree的数据结构，它有二叉树的特征，同时解决平衡和深度的问题，这种数据结构能够帮助我们快速的获取数据库中的数据。
  ```

+ 问题3: 索引的作用?

  ```txt
  当表中的数据量越来越大时，索引对于性能的影响愈发重要。索引优化应该是对查询性能优化最有效的手段。索引能够轻易将查询性能提高好几倍。有了索引相当于我们给数据库的数据加了目录一样，可以快速的找到数据，简单来说是提高数据查询的效率。
  ```

+ 问题4: 索引的分类?

  ```txt
  1.普通索引
  2.主键索引
  3.唯一索引
  4.联合索引（组合索引）
  5.全文索引
  ```

+ 问题5: 索引的原理?

  ```txt
  索引的实现本质上是为了让数据库能够快速查找数据，而单独维护的数据结构，mysql实现索引主要使用的两种数据结构: hash 和 B+Tree，我们比较常用的MyISAM和InnoDB存储引擎都是基于B+Tree的。
  
  hash:（hash索引在MySQL比较少用）他以把数据的索引以hash形式组织起来，因此当查找某一条记录的时候，速度非常快。但是因为是hash结构，每个键只对应一个值，而且是散列的方式分布，所以他并不支持范围查找和排序等功能。
  
  B+树:B+Tree是(MySQL使用最频繁的一个索引数据结构)数据结构，B+Tree每个节点可以存放多个数据，相比二叉树，树的高度更低，磁盘IO更少，查询效率更高。因为是树型结构，所以更适合用来处理排序，范围查找等功能。
  ```

+ 问题6: 索引的优点?

  ```txt
  1. 可以通过建立唯一索引或者主键索引,保证数据库表中每一行数据的唯一性。
  2. 建立索引,可以大大提高检索的数据,以及减少表的检索行数。
  3. 建立索引,在表连接条件时,可以加速表与表直接的相连。
  4. 建立索引,在分组和排序时,可以减少查询时分组和排序所消耗的时间。
  5. 建立索引,在查询中使用索引可以提高性能。
  ```

+ 问题7: 索引的缺点?

  ```txt
  1. 在创建索引和维护索引时,会耗费时间,随着数据量的增加而增加。
  2. 索引文件会占用物理空间,除了数据表需要占用物理空间之外,每一个索引还会占用一定的物理空间。
  3. 当对表进行INSERT,UPDATE,DELETE的时候,索引也要维护,这样就会降低数据的维护速度(建立索引会占用磁盘空间的索引文件。一般情况这个问题不太严重，但如果你在一个大表上创建了多种组合索引，索引文件的会膨胀很快)。
  ```

+ 问题8: 如何分析索引使用情况?

  ```txt
  explain显示了MySQL如何使用索引来处理select语句以及连接表。可以帮助选择更好的索引和写出更优化的查询语句。简单讲，它的作用就是分析查询性能。explain关键字的使用方法很简单，就是把它放在select查询语句的前面。MySQL查看是否使用索引，简单的看type类型就可以。如果它是all，那说明这条查询语句遍历了所有的行，并没有使用到索引。
  ```

+ 问题9: 哪些字段适合加索引?

  ```txt
  1. 在经常需要搜索的列上添加索引,可以加快搜索的速度。
  2. 主键列上可以确保列的唯一性。
  3. 在表与表的而连接条件上加上索引,可以加快连接查询的速度。
  4. 在经常需要排序(order by),分组(group by)和的distinct列上加索引可以加快排序查询的时间。
  ```

+ 问题10: 哪些字段不适合加索引

  ```txt
  1. 查询中很少使用到的列不应该创建索引,如果建立了索引然而还会降低mysql的性能和增大了空间需求。
  2. 很少数据的列也不应该建立索引,比如一个性别字段0或者1,在查询中,结果集的数据占了表中数据行的比例比较大,MySQL需要扫描的行数很多,增加索引,并不能提高效率。
  3. 定义为text和image和bit数据类型的列不应该增加索引。
  4. 当表的修改(UPDATE,INSERT,DELETE)操作远远大于检索(SELECT)操作时不应该创建索引,这两个操作是互斥的关系。
  ```

+ 问题11: 哪些情况会造成索引失效?

  ```txt
  1. 如果条件中有or，即使其中有条件带索引也不会使用(这也是为什么尽量少用or的原因)。
  2. 索引字段的值不能有null值，有null值会使该列索引失效。
  3. 对于联合索引，不是使用的第一部分，则不会使用索引（最左原则）。
  4. like查询以%开头。
  5. 如果列类型是字符串，那一定要在条件中将数据使用单引号引用起来,否则不使用索引。
  6. 在索引的列上使用表达式或者函数会使索引失效。
  ```

+ 问题12: 联合索引最左原则?

  ```txt
  在MySQL建立联合索引时会遵循最左前缀匹配的原则，即最左优先，在检索数据时从联合索引的最左边开始匹配，组合索引的第一个字段必须出现在查询组句中，这个索引才会被用到, 如创建组合索引a,b,c那么查询条件中只使用b和c是使用不到索引的。
  ```

+ 问题13: 聚簇索引和非聚簇索引?

  ```txt
  1. MyISAM——非聚簇索引
     MyISAM存储引擎采用的是非聚簇索引，非聚簇索引的数据表和索引表是分开存储的。非聚簇索引的主键索引和辅助索引几乎是一样的，只是主索引不允许重复，不允许空值，他们的叶子结点的key都存储指向键值对应的数据的物理地址。
  
  2. InnoDB——聚簇索引
     聚簇索引的数据和主键索引存储在一起，主键索引的叶子结点存储的是键值对应的数据本身，辅助索引的叶子结点存储的是键值对应的数据的主键键值。因此主键的值长度越小越好，类型越简单越好。
  ```

+ 问题14: in和exists区别?

  ```txt
  1.当A表数据多于B表中的数据时，使用in优于exists。
  2.当A表数据小于B表中的数据时，使用exists优于in。
  3.如果两张表数据量差不多，那么它们的执行性能差不多。
  ```

+ 问题15: 我有三个表 A,B,C 现在有一个select * from A,B,C你能告诉我？A,B,C三个表在查询的的执行顺序是什么?

  ```txt
  一定通过explain查询id的值才能决定。如果排id相同那么至上而下运行。如果id不同，大的先执行。
  ```

+ 问题16: like查询中哪那些会走索引那些不会走索引？

  ```sql
  -- b 建立了一个索引
  select * from table where b like '%xxxx%' -- 不会
  select * from table where b like 'xxxx%'  -- 会
  select * from table where  b like '%xxxx' -- 不会
  ```

+ 问题17: MySQL事务隔离级别?

  ![1593759912209](就业技术加强-02-MySQL高级.assets/1593759912209.png)  

   

+ 问题18: MySQL中锁的分类?

  ```txt
  1.按操作分: 读锁(共享锁)、写锁(排它锁)
  2.按粒度分: 表锁、页锁、行锁  MySQL一页是16kB
  3.思想的层面分: 悲观锁、乐观锁
  ```

+ 问题19: MySQL中有几种连接查询?

  ```txt
  1. 内连接(inner join): 只有两个元素表相匹配的才能在结果集中显示。
  	
  2. 外连接：
     2.1 左外连接(left join): 左边为驱动表，驱动表的数据全部显示，匹配表的不匹配的不会显示。
     2.2 右外连接(right join): 右边为驱动表，驱动表的数据全部显示，匹配表的不匹配的不会显示。
     
     union:	联合查询,将多个查询的结果组合在一起, 去重
     union all: 联合查询,将多个查询的结果组合在一起, 不去重
  ```

+ 问题20: MySQL如何综合性优化?

  ```txt
  1.选择表合适存储引擎: 
    MyISAM: 以读和插入操作为主，只有少量的更新和删除，并且对事务的完整性，并发性要求不是很高的。
    InnoDB: 事务处理，以及并发条件下要求数据的一致性。除了插入和查询外，包括很多的更新和删除。
  
  1.1 合理的设计表(满足3范式)
  
  2.索引优化: 
    -- 表一定要建立主键索引。
    -- 数据量大的表应该有索引。
    -- 经常与其他表进行连接的表，在连接字段上应该建立索引。
    -- 经常出现在Where子句中的字段，特别是大表的字段，应该建立索引。
    -- 索引应该建在选择性高的字段上（sex 性别这种就不适合）。
    -- 索引应该建在小字段上，对于大的文本字段甚至超长字段，不要建索引。
    -- 频繁进行数据操作的表，不要建立太多的索引。
    -- 删除无用的索引，避免对执行计划造成负面影响。
  
  3.sql语句优化:
    -- SELECT语句务必指明列的名称（避免直接使用select * ）
    -- SQL语句要避免造成索引失效的写法
    -- SQL语句中IN包含的值不应过多
    -- 如果排序字段没有用到索引，就尽量少排序
    -- 尽量少用or
    -- 尽量用union all代替 union
    -- 避免在where子句中对字段进行null值判断
    -- 不建议使用%前缀模糊查询
    -- 避免在where子句中对列进行表达式或函数操作
  
  4.缓存优化:
     为了提高查询速度，我们可以通过不同的方式去缓存我们的结果从而提高响应效率。当我们的数据库打开了Query Cache（简称QC）功能后，数据库在执行SELECT语句时，会将其结果放到QC中，当下一次处理同样的SELECT请求时，数据库就会从QC取得结果，而不需要去数据表中查询。如果缓存命中率非常高的话，有测试表明在极端情况下可以提高效率238%。
  
  5.读写分离:
    如果数据库的使用场景读的操作比较多的时候，为了避免写的操作所造成的性能影响 可以采用读写分离的架构，读写分离，解决的是，数据库的写入，影响了查询的效率。读写分离的基本原理是让主数据库处理事务性增、改、删操作（INSERT、UPDATE、DELETE），而从数据库处理SELECT查询操作。
  
  6.MySQL的分库分表:
    数据量越来越大时，单体数据库无法满足要求，可以考虑分库分表
    两种拆分方案：
       垂直拆分, 水平拆分
       表的垂直拆分:就是把原来一个有很多列的表拆分成多个表。
       通常垂直拆分可以按以下原则进行：
  		1、把不常用的字段表单独存放到一个表中。
  		2、把大字段独立存放到一个表中。
  	3、把经常一起使用的字段放到一起。
  	
  	表的水平拆分: 是为了解决单表数据量过大的问题，水平拆分的表每一个表的结构都是完全一致的，将数据平均分为N份
  	
    分库分表常用中间件: MyCat、Sharding-JDBC
  
  运维
  修改数据库的配置
  
  增加硬件
  ```

  