---
title: MongoDB
tags:
  - 笔记
  - 数据库
  - MongonDB
categories:
  - 数据库
date: 2021-01-31 18:18:14
---

 

# 1.  MongonDB

## 1.1.  概述

MongoDB 是一个跨平台的，面向文档的数据库，是当前 NoSQL 数据库产品中最热门的一种。它介于关系数据库和非关系数据库之间，是非关系数据库当中功能最丰富，最像关系数据库的产品。它支持的数据结构非常松散，是类似JSON 的 BSON 格式，因此可以存储比较复杂的数据类型。

MongoDB 的官方网站地址是：http://www.mongodb.org/

![img](MongoDB讲义.assets/clip_image001.png) 

 

## 1.2.  特点

MongoDB 最大的特点是他支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。它是一个面向集合的,模式自由的文档型数据库。

具体特点总结如下：

（1）面向集合存储，易于存储对象类型的数据

（2）模式自由

（3）支持动态查询

（4）支持完全索引，包含内部对象

（5）支持复制和故障恢复

（6）使用高效的二进制数据存储，包括大型对象（如视频等）

（7）自动处理碎片，以支持云计算层次的扩展性

（8）支持 Python，PHP，Ruby，Java，C，C#，Javascript，Perl 及 C++语言的驱动程序，社区中也提供了对 Erlang 及.NET 等平台的驱动程序

（9） 文件存储格式为 BSON（一种 JSON 的扩展）

## 1.3.  体系结构

MongoDB 的逻辑结构是一种层次结构。主要由：

文档(document)、集合(collection)、数据库(database)这三部分组成的。逻辑结构是面向用户

的，用户使用 MongoDB 开发应用程序使用的就是逻辑结构。

（1）MongoDB 的文档（document），相当于关系数据库中的一行记录。

（2）多个文档组成一个集合（collection），相当于关系数据库的表。

（3）多个集合（collection），逻辑上组织在一起，就是数据库（database）。

（4）一个 MongoDB 实例支持多个数据库（database）。

文档(document)、集合(collection)、数据库(database)的层次结构如下图:

![img](MongoDB讲义.assets/clip_image003.jpg) 

下表是MongoDB与MySQL数据库逻辑结构概念的对比

| MongoDb           | 关系型数据库Mysql |
| ----------------- | ----------------- |
| 数据库(databases) | 数据库(databases) |
| 集合(collections) | 表(table)         |
| 文档(document)    | 行(row)           |

 

# 2.  安装与启动

## 2.1.  安装

详见《MongoDB windows版安装说明.docx》

 

为了方便操作也可以安装MongoDB的一个图形界面管理工具；解压“资料\robo3t-1.2.1-windows-x86_64-3e50a65.zip”然后双击运行“robo3t.exe”即可。

## 2.2.  启动

### 2.2.1.  创建存放数据目录

随便在系统中创建一个存放MongoDB的数据的目录；如：

![img](MongoDB讲义.assets/clip_image005.jpg)

 

### 2.2.2.  编写启动脚本

找到安装MongoDB的目录的bin目录下；

创建如下的启动脚本文件startMongoDB.bat；文件内容如下：

  ```
.\mongod.exe --dbpath=D:\database\mongodb\data\db3_4
  ```

--dbpath是刚刚创建的数据存放目录；

之后启动MongoDB就直接双击startMongoDB.bat即可。

![img](MongoDB讲义.assets/clip_image007.jpg)

默认连接的端口号为：27017

## 2.3.  连接

在安装目录的bin目录路径中双击“mongo.exe”即可

 

# 3.  基本操作

## 3.1.  选择或创建数据库

使用use 数据库名称 即可选择数据库，如果该数据库不存在会自动创建

```
  use itcastdb  
```

  ![img](MongoDB讲义.assets/clip_image008.png)  

 

## 3.2.  新增

文档相当于关系数据库中的记录

首先定义一个文档变量，格式为变量名称={}; 例如：

 ```
obj = {"name":"传智播客","gender":"男","age":12,"address":"吉山"}
 ```

![img](MongoDB讲义.assets/clip_image010.jpg)  

 

接下来就是将这个变量存入MongoDB 

格式为：

  `db.集合名称.save(变量);  `

这里的集合就相当于关系数据库中的表。例如：

![img](MongoDB讲义.assets/clip_image012.jpg)

这样就在user集合中存入文档。如果这个user集合不存在，就会自动创建。

当然，也可以不用定义变量，直接把变量值放入save方法中也是可以的。

  ```
db.user.save({"name":"黑马程序员","gender":"女","age":8,"address":"广州"});
  ```

 ![img](MongoDB讲义.assets/clip_image014.jpg)  

 

为了方便后期测试，预加数据：

```
db.user.save({"name":"李雷","gender":"男","age":15,"address":"北京"});
db.user.save({"name":"韩梅梅","gender":"女","age":14,"address":"上海"});
db.user.save({"name":"露西","gender":"女","age":13,"address":"纽约"});
db.user.save({"name":"莉莉","gender":"女","age":13,"address":"纽约"});
db.user.save({"name":"Jim Green","gender":"男","age":12,"address":"华盛顿"});
db.user.save({"name":"王大叔","gender":"男","age":40,"address":"深圳"});

```



 

## 3.3.  查询

### 3.3.1.  查询全部文档

语法：`db.集合名称.find();`

```
【说明】查询user集合中的所有文档
【示例】db.user.find();

```



![img](MongoDB讲义.assets/clip_image016.jpg)  

发现每条文档会有一个叫_id的字段，这个相当于原来关系数据库中表的主键，当在插入文档记录时没有指定该字段，MongDB会自动创建，其类型是ObjectID类型。

在插入文档记录时指定该字段也是可以的，其类型可以使ObjectID类型，也可以是MongoDB支持的任意类型。 例如：

 ```
db.user.save({"_id":123,"name":"JBL","gender":"男","age":12,"address":"吉山"});
 ```

![img](MongoDB讲义.assets/clip_image018.jpg)  

 

重新查询全部：

![img](MongoDB讲义.assets/clip_image020.jpg)

### 3.3.2.  条件查询

语法：`db.集合名称.find({“属性名”:”条件值”});`

  ```
【说明】查询集合中某个属性的值为 *** 的所有文档

【示例】db.user.find({"address":"吉山"});

  ```

![img](MongoDB讲义.assets/clip_image022.jpg)  

 

### 3.3.3.  查询第一条

语法：`db.集合名称.findOne();`

  ```
【说明】查询集合中的第一个文档
【示例】db.user.findOne();

  ```

![img](MongoDB讲义.assets/clip_image024.jpg)  

 

## 3.4.  更新

语法：`db.集合名称.update({更新条件键值对},{要更新的键值对});`

  ```
【说明】根据更新条件键值对，更新第二个参数对应的属性
【示例】db.user.update({"name":"传智播客"},{"address":"广州吉山"});

  ```

![img](MongoDB讲义.assets/clip_image026.jpg)     

```
更新后文档只剩下_id 和address两个字段了。
如果要保留其它字段值的话；需要使用MongoDB提供的修改器$set 来实现，请看下列示例：

db.user.update({"name":"黑马程序员"},{$set:{"address":"广州吉山"}});

```



![img](MongoDB讲义.assets/clip_image028.jpg)  

 

## 3.5.  删除

### 3.5.1.  删除文档

语法：db.集合名称.remove({删除条件键值对});

 ```
【说明】删除集合中所有符合条件的文档
【示例】db.user.remove({"address":"广州吉山"});

 ```

![img](MongoDB讲义.assets/clip_image030.jpg)  

 

### 3.5.2.  删除集合

语法：`db.集合名称.drop()`

```
【说明】用于从数据库中删除集合；删除的时候可以使用show collections查看当前数据库下的所有集合
【示例】
db.test.save({"name":"aa"});
db.test.drop();

```



![img](MongoDB讲义.assets/clip_image032.jpg)  

 

### 3.5.3.  删除数据库

语法：`db.dropDatabase()`

```
【说明】删除当前数据库，默认为 test，你可以使用 db 命令查看当前数据库名。
【示例】
use itcast_test;
db.dropDatabase();
```



  ![img](MongoDB讲义.assets/clip_image034.jpg)     

 

# 4.  高级查询

## 4.1.  模糊查询

MongoDB的模糊查询是通过正则表达式的方式实现的。格式为：

 /模糊查询字符串/

```
【示例1】查询名字中带有“雷”的文档
db.user.find({"name":/雷/});

```

![img](MongoDB讲义.assets/clip_image036.jpg)     

```
【示例2】查询名字以“李”开头的文档
db.user.find({"name":/^李/});
```



![img](MongoDB讲义.assets/clip_image038.jpg)  

 

## 4.2.  null值处理

找出集合中某属性值为空或者不存在该属性的文档；与之前的条件查询是一样的，条件值写为null就可以了。

```
【示例】先修改一个address空的记录，然后查询address为空的记录
/*先更新address为null*/
db.user.update({"_id":123},{$set:{"address":null}});
/*查询address为null或者不存在该属性的文档*/
db.user.find({"address":null});

```

![img](MongoDB讲义.assets/clip_image040.jpg)  

 

## 4.3.  大于小于

<, <=, >, >= 这个操作符也很常用，格式如下

```
db.collection.find({ "field" : { $gt: value } } ); // 大于: field > value
db.collection.find({ "field" : { $lt: value } } ); // 小于: field < value
db.collection.find({ "field" : { $gte: value } } ); // 大于等于: field >= value
db.collection.find({ "field" : { $lte: value } } ); // 小于等于: field <= value
```



```
【示例】查询年龄大于等于14岁的用户
/*查询age大于等于14的用户*/
db.user.find({"age":{$gte:14}});

```

![img](MongoDB讲义.assets/clip_image042.jpg)  

 

## 4.4.  不等于

不等于使用$ne操作符。

```sql
  /*查询性别不等于男的用户*/  db.user.find({"gender":{$ne:"男"}});     
```



![img](MongoDB讲义.assets/clip_image044.jpg)  

 

## 4.5.  判断属性是否存在

判断属性是否存在使用$exists操作符。

```
/*查询address属性存在的文档*/
db.user.find({"address":{$exists:true}});

```

![img](MongoDB讲义.assets/clip_image046.jpg)  

 

## 4.6.  包含与不包含

包含使用$in操作符；不包含使用$nin操作符。

```sql
/*查询address在北京、上海的用户*/
db.user.find({"address":{$in:["北京","上海"]}});

/*查询address不在北京、上海的用户*/
db.user.find({"address":{$nin:["北京","上海"]}});

```



![img](MongoDB讲义.assets/clip_image048.jpg)  

 

## 4.7.  统计记录数

统计记录条件使用count()方法。

```sql
/*统计集合中的所有文档*/
db.user.count();

/*统计集合中所有address在纽约的用户*/
db.user.count({"address":{$in:["纽约"]}});

```

  ![img](MongoDB讲义.assets/clip_image050.jpg)  

 

## 4.8.  条件连接-并且

如果需要查询同时满足两个以上条件，需要使用$and操作符将条件进行关联。（相当于SQL的and）

格式为：`$and:[ { },{ },{  } ]`

```sql
/*查询age大于等于14并且 年龄小于等于20的用户*/
db.user.find({$and:[{"age":{$gte:14}},{"age":{$lte:20}}]});
   
```

 ![img](MongoDB讲义.assets/clip_image052.jpg)  

 

## 4.9.  条件连接-或者

如果两个以上条件之间是或者的关系，我们使用$or操作符进行关联，与前面$and的使用方式相同

格式为：`$or:[ { },{ },{  } ]`

```sql
/*查询age小于等于15 或者 gender为女的用户*/
db.user.find({$or:[{"age":{$lte:15}},{"gender":"女"}]});
```

![img](MongoDB讲义.assets/clip_image054.jpg)  

 

# 5.  java连接MongoDB

## 5.1.  添加依赖

```java
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>3.6.3</version>
</dependency>

```



## 5.2.  查询

### 5.2.1.  查询全部

 ```java

@Test
public void queryTest(){
    //创建连接对象
    MongoClient mongoClient = new MongoClient("127.0.0.1", 27017);
    //获取要操作的数据库
    MongoDatabase database = mongoClient.getDatabase("itcastdb");
    //获取集合
    MongoCollection<Document> collection = database.getCollection("user");

    //查询集合所有文档
    FindIterable<Document> documents = collection.find();

    if (documents != null) {
        //遍历输出集合中的内容
        for (Document document : documents) {
            System.out.println("_id = " + document.get("_id"));
            System.out.println("name = " + document.getString("name"));
            System.out.println("gender = " + document.getString("gender"));
            System.out.println("age = " + document.getDouble("age"));
            System.out.println("address = " + document.getString("address"));
            System.out.println("------------------------------------------------");
        }
    }
}

 ```

MongoDB的数字类型默认使用64位浮点型数值。{“x”：3.14}或{“x”：3}。对于整型值，可以在保存的时候使用NumberInt（4字节符号整数），{“x”:NumberInt(“3”)} 或NumberLong（8字节符号整数）{“x”:NumberLong(“3”)}

### 5.2.2.  条件查询

MongoDB使用BasicDBObject类型封装查询条件，构造方法的参数为key 和value。

![img](MongoDB讲义.assets/clip_image056.jpg)

### 5.2.3.  模糊查询

构建模糊查询条件是通过正则表达式的方式来实现的

（1）完全匹配Pattern pattern = Pattern.compile("^name$");

（2）右匹配Pattern pattern = Pattern.compile("^.*name$");

（3）左匹配Pattern pattern = Pattern.compile("^name.*$");

（4）模糊匹配Pattern pattern = Pattern.compile("^.*name.*$");

 

![img](MongoDB讲义.assets/clip_image058.jpg)

### 5.2.4.  大于小于查询

在MongoDB提示符下条件json字符串为{ age: { $gte :14} } ，对应的java代码也是BasicDBObject 的嵌套。

![img](MongoDB讲义.assets/clip_image060.jpg)

### 5.2.5.  条件连接-并且

![img](MongoDB讲义.assets/clip_image062.jpg)

### 5.2.6.  条件连接-或者

![img](MongoDB讲义.assets/clip_image064.jpg)

## 5.3.  新增

```java
@Test
public void insertTest() {
    //创建连接对象
    MongoClient mongoClient = new MongoClient("127.0.0.1", 27017);
    //获取要操作的数据库
    MongoDatabase database = mongoClient.getDatabase("itcastdb");
    //获取集合
    MongoCollection<Document> collection = database.getCollection("user");

    Map<String, Object> map = new HashMap<String, Object>();
    map.put("name", "芒果");
    map.put("gender", "女");
    map.put("age", 16);
    map.put("address", "广州");

    Document document = new Document(map);//或者直接对document对象设置值
    collection.insertOne(document);
}

```

 

## 5.4.  更新

```java
@Test
public void updateTest() {
    //创建连接对象
    MongoClient mongoClient = new MongoClient("127.0.0.1", 27017);
    //获取要操作的数据库
    MongoDatabase database = mongoClient.getDatabase("itcastdb");
    //获取集合
    MongoCollection<Document> collection = database.getCollection("user");

    //更新条件
    BasicDBObject condition = new BasicDBObject("name", "芒果");

    //更新内容
    BasicDBObject updateObject = new BasicDBObject("$set", new BasicDBObject("address", "北京"));

    collection.updateMany(condition, updateObject);
}

```

 

## 5.5.  删除

```java
public void deleteTest() {
    //创建连接对象
    MongoClient mongoClient = new MongoClient("127.0.0.1", 27017);
    //获取要操作的数据库
    MongoDatabase database = mongoClient.getDatabase("itcastdb");
    //获取集合
    MongoCollection<Document> collection = database.getCollection("user");

    //删除条件
    BasicDBObject condition = new BasicDBObject("name", "芒果");

    collection.deleteMany(condition);

```

 

 

​                                               