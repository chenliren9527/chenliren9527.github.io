---
title: MySQL存储过程&视图
tags:
  - 笔记
  - 数据库
  - MySQL
categories:
  - 数据库
date: 2020-09-21 14:13:31
---


# MySQL存储过程&视图&JDBC-笔记

## 回顾

1. 能够使用内连接进行多表查询

   隐式内连接: SELECT * FROM 表1, 表2 WHERE 条件;

   显示内连接: SELECT * FROM 表1 INNER JOIN 表2 ON 表连接条件 WHERE 查询条件;

2. 能够使用左外和右外连接进行多表查询

   左外: SELECT * FROM 表1 LEFT OUTER  JOIN 表2 ON 表连接条件 WHERE 查询条件;

   右外: SELECT * FROM 表1 RIGHT OUTER JOIN 表2 ON 表连接条件 WHERE 查询条件;

3. 能够使用子查询

   一个查询的结果作为另一个查询的一部分

   SELECT * FROM 表名 WHERE 字段名=(SELECT MAX(字段名) FROM 表名);

4. 能够理解多表查询的规律

   1. 明确查询哪些表
   2. 明确表连接的条件
   3. 后续的查询

5. 能够理解事务的概念

   由多条SQL语句组成一个功能,这多条SQL语句形成一个事务

   事务中的多条SQL语句要么都执行,要么都不执行,具有原子性的

   START TRANSACTION;

   指定多条SQL语句

   COMMIT;/ROLLBACK;

6. 理解mysql索引的作用

   提高查询效率

   

## 学习目标

1. 会使用mysql字符串函数
2. 会使用mysql日期函数
3. 能够独立完成mysql综合练习
4. 理解mysql表自关联
5. 能够理解JDBC的概念
6. 能够使用JDBC实现对单表数据增、删、改、查



## 准备数据

```sql
CREATE DATABASE day18;
use day18;

-- 部门表
CREATE TABLE dept (
  id INT PRIMARY KEY PRIMARY KEY, -- 部门id
  dname VARCHAR(50), -- 部门名称
  loc VARCHAR(50) -- 部门位置
);

-- 添加4个部门
INSERT INTO dept(id,dname,loc) VALUES 
(10,'教研部','北京'),
(20,'学工部','上海'),
(30,'销售部','广州'),
(40,'财务部','深圳');

-- 职务表，职务名称，职务描述
CREATE TABLE job (
  id INT PRIMARY KEY,
  jname VARCHAR(20),
  description VARCHAR(50)
);

-- 添加4个职务
INSERT INTO job (id, jname, description) VALUES
(1, '董事长', '管理整个公司，接单'),
(2, '经理', '管理部门员工'),
(3, '销售员', '向客人推销产品'),
(4, '文员', '使用办公软件');

-- 员工表
CREATE TABLE emp (
  id INT PRIMARY KEY, -- 员工id
  ename VARCHAR(50), -- 员工姓名
  job_id INT, -- 职务id
  mgr INT , -- 上级领导
  joindate DATE, -- 入职日期
  salary DECIMAL(7,2), -- 工资
  bonus DECIMAL(7,2), -- 奖金
  dept_id INT, -- 所在部门编号
  CONSTRAINT emp_jobid_ref_job_id_fk FOREIGN KEY (job_id) REFERENCES job (id),
  CONSTRAINT emp_deptid_ref_dept_id_fk FOREIGN KEY (dept_id) REFERENCES dept (id)
);

-- 添加员工
INSERT INTO emp(id,ename,job_id,mgr,joindate,salary,bonus,dept_id) VALUES 
(1001,'孙悟空',4,1004,'2000-12-17','8000.00',NULL,20),
(1002,'卢俊义',3,1006,'2001-02-20','16000.00','3000.00',30),
(1003,'林冲',3,1006,'2001-02-22','12500.00','5000.00',30),
(1004,'唐僧',2,1009,'2001-04-02','29750.00',NULL,20),
(1005,'李逵',4,1006,'2001-09-28','12500.00','14000.00',30),
(1006,'宋江',2,1009,'2001-05-01','28500.00',NULL,30),
(1007,'刘备',2,1009,'2001-09-01','24500.00',NULL,10),
(1008,'猪八戒',4,1004,'2007-04-19','30000.00',NULL,20),
(1009,'罗贯中',1,NULL,'2001-11-17','50000.00',NULL,10),
(1010,'吴用',3,1006,'2001-09-08','15000.00','0.00',30),
(1011,'沙僧',4,1004,'2007-05-23','11000.00',NULL,20),
(1012,'李逵',4,1006,'2001-12-03','9500.00',NULL,30),
(1013,'小白龙',4,1004,'2001-12-03','30000.00',NULL,20),
(1014,'关羽',4,1007,'2002-01-23','13000.00',NULL,10);

-- 工资等级表
CREATE TABLE salarygrade (
  grade INT PRIMARY KEY,
  losalary INT,
  hisalary INT
);

-- 添加5个工资等级
INSERT INTO salarygrade(grade,losalary,hisalary) VALUES 
(1,7000,12000),
(2,12010,14000),
(3,14010,20000),
(4,20010,30000),
(5,30010,99990);
```



##  视图(了解)

### 目标

学习视图的创建，修改，查看，删除



#### 视图的概述

视图（View）是一种虚拟存在的表。通俗的讲，视图就是一条SELECT语句执行后返回的结果集。所以我们在创建视图的时候，主要的工作就落在创建这条SQL查询语句上。视图相对于普通的表的优势主要包括以下几项:

- 简单：使用视图的用户完全不需要关心后面对应的表的结构、关联条件和筛选条件，对用户来说已经是过滤好的复合条件的结果集。

- 安全：使用视图的用户只能访问他们被允许查询的结果集，对表的权限管理并不能限制到某个行某个列，但是通过视图就可以简单的实现。




#### 创建视图

创建视图的语法为：

```mysql
CREATE [OR REPLACE] VIEW 视图名称 AS 查询语句;
```



**示例 , 创建city_country_view视图 , 执行如下SQL** 

![1595558413341](MySQL存储过程-视图.assets/1595558413341.png)

```mysql
-- 创建一个视图
CREATE OR REPLACE VIEW view_emp_dept AS SELECT e.*, d.dname, d.loc FROM emp e INNER JOIN dept d ON e.dept_id=d.id;
```



#### 使用视图

使用视图的语法为：

```sql
SELECT * FROM 视图名称;
```



#### 查看视图

从 MySQL 5.1 版本开始，使用 SHOW TABLES 命令的时候不仅显示表的名字，同时也会显示视图的名字，而不存在单独显示视图的 SHOW VIEWS 命令。

```mysql
SHOW TABLES;
```

**结果：**

![1595428494282](MySQL存储过程-视图.assets/1595428494282.png)



**如果需要查询某个视图的定义，可以使用 SHOW CREATE VIEW 命令进行查看** 

```mysql
-- 查看创建视图的sql语句
SHOW CREATE VIEW view_emp_dept;
```

![1595558554713](MySQL存储过程-视图.assets/1595558554713.png)



#### 修改视图

修改视图的语法为：

```mysql
ALTER VIEW 视图名称 AS 查询语句;
```



#### 删除视图

**语法 :**

```mysql
DROP VIEW [IF EXISTS] 视图名称;
```

**示例 , 删除视图city_country_view**  

```mysql
DROP VIEW view_emp_dept;
```



### 小结

视图在以后什么时候可以去使用？

复杂的多表查询语句,频繁使用,就可以使用视图来保存这个复杂的SQL语句



创建视图、 删除视图、 查看视图

```
创建视图: CREATE [OR REPLAC]E VIEW 视图名称 AS 查询语句;
删除视图: DROP VIEW [IF EXISTS] 视图名称;
查看视图: SHOW TABLES;
使用视图: SELECT * FROM 视图名称;
```






# MySQL的内置函数



## MySQL字符串函数和数学函数（了解）

### 目标

了解MySQL中字符串函数和数学函数



MySQL中字符串函数如下:

| **函数**                               | **描述**                                                     | **实例**                                                  |
| -------------------------------------- | ------------------------------------------------------------ | --------------------------------------------------------- |
| **CHAR_LENGTH(s)**                     | 返回字符串 s 的字符数                                        | SELECT CHAR_LENGTH("ZhangSan") AS 长度;                   |
| **CONCAT(s1,s2...sn)**                 | 字符串 s1,s2 等多个字符串合并为一个字符串                    | SELECT CONCAT("SQL ", "itcast ", "Gooogle ", "Facebook"); |
| **LOWER(s)**                           | 将字符串 s 的所有字母变成小写字母                            | SELECT LOWER('HelloWorld');                               |
| **UPPER(s)**                           | 将字符串转换为大写                                           | SELECT UPPER("HelloWorld");                               |
| **SUBSTR(s, start, length)**           | 从字符串 s 的 start 位置截取长度为 length 的子字符串，从1开始计数 | SELECT SUBSTR("HelloWorld", 7, 3);                        |
| **TRIM(s)**                            | 去掉字符串 s 开始和结尾处的空格                              | SELECT TRIM('   itheima   ')                              |
| **REPLACE(字符串, 源字符串,新字符串)** | 将字符串中的源字符串换成新的字符串                           | SELECT REPLACE('abcde','bc','xyz');                       |

**示例代码**

```sql
-- 显示所有员工的姓氏，截取  从1开始计数
SELECT SUBSTR(ename,1,1) FROM emp;

-- 显示所有员工姓名和名字的长度
SELECT ename, CHAR_LENGTH(ename) FROM emp;

-- 将所有姓刘的员工，姓氏替换为牛
SELECT e.`ename` 原名,REPLACE(e.`ename`,'刘','牛')   FROM emp e;

-- 将所有员工的编号和姓名拼接在一起
SELECT CONCAT(e.`id`,e.`ename`)  FROM emp e;
```



MySQL中数学函数:

| **函数**                  | **说明**             | **案例**                  |
| ------------------------- | -------------------- | ------------------------- |
| **RAND()**                | 返回 0 到 1 的随机数 | SELECT RAND();            |
| **ROUND(小数, 保留几位)** | 四舍五入保留几位小数 | SELECT ROUND(1.23456, 3); |

**示例代码**

```mysql
-- 统计每个部门的平均薪资,保留2位小数
SELECT e.`dept_id`, ROUND(AVG(e.`salary`),2)  FROM  emp e GROUP BY e.`dept_id`;
```

![1595412796544](MySQL存储过程-视图.assets/1595412796544.png)





## MySQL日期函数(了解)

### 目标

了解日期函数



MySQL日期函数

| **函数**             | **说明**                          | **案例**                                   |
| -------------------- | --------------------------------- | ------------------------------------------ |
| **addDate(date, n)** | 计算起始日期 date 加上 n 天的日期 | SELECT ADDDATE("2017-06-15", 10);          |
| **curDate()**        | 返回当前日期                      | SELECT CURDATE();                          |
| **dateDiff(d1,d2)**  | 计算日期 d1->d2 之间相隔的天数    | SELECT DATEDIFF('2001-01-01','2001-02-02') |
| **now()**            | 返回当前日期和时间                | SELECT NOW()                               |
| **year(日期)**       | 获取指定日期的年份                | SELECT YEAR(NOW())                         |

**示例代码**

```mysql
-- 统计每个员工入职的天数
SELECT e.ename,  DATEDIFF(CURDATE() ,e.joindate)  FROM emp e; 

-- 统计每个员工的工龄 
SELECT e.ename,  ROUND(DATEDIFF(CURDATE(),e.joindate)/365,0)  FROM emp e; 

-- 查询2001年入职的员工
SELECT  *  FROM emp e WHERE YEAR(e.`joindate`)=2001; 

-- 统计入职19年以上的员工有多少个
SELECT  *  FROM emp e WHERE DATEDIFF(CURDATE(),e.joindate)/365 > 19;
```



## MySQL高级函数CASE和IF

### 目标

学习MySQL高级函数CASE和IF



#### CASE高级函数

在查询代码的过程中，可能我们需要对查询的结果进行判断, 就会使用到CASE函数,  语法如下:

```mysql
-- case表达式语法1
select 字段名1, 字段名2, case 字段名
	when 值1 then 返回的值
	when 值2 then 返回的值
	...
	else
                 上面都不符合返回的值
    end  列名
  from 表名;
  
 -- case表达式语法2
select 字段名1, 字段名2, case 
	when 判断条件1 then 返回的值
	when 判断条件1 then 返回的值
	...
	else
                 上面条件都不成立返回的值
    end
  from 表名;
  
-- case表达式功能
实现分支条件判断，与java的switch结构类似， 
当字段的值与when的值匹配时返回 then 后面值， 都不符合返回else的值
```

**举列：**

```mysql
-- 需求:查询每个员工的工资等级并排序
-- 工资等级在1显示为 '努力赚钱'
-- 工资等级在2显示为 '小康生活'
-- 工资等级在3显示为 '可以娶媳妇'
-- 工资等级在4显示为 '可以买车'
-- 工资等级在5显示为 '可以买房'
-- 工资等级不在以上列表中显示为 '土豪'

-- 分析
-- 确定表
员工信息：emp表
工资等级：salarygrade表
-- 确定表连接
SELECT e.ename, e.salary , s.grade,
    CASE s.grade
    WHEN 1 THEN '努力赚钱'
    WHEN 2 THEN '小康生活'
    WHEN 3 THEN '可以娶媳妇'
    WHEN 4 THEN '可以买车'
    WHEN 5 THEN '可以买房'
    ELSE '土豪'
    END  建议信息
FROM emp e INNER JOIN salarygrade s 
ON e.salary BETWEEN s.losalary AND s.hisalary
ORDER BY s.grade DESC;


-- 语法2 解决
SELECT e.ename, e.salary , s.grade,
    CASE 
    WHEN s.grade=1 THEN '努力赚钱'
    WHEN s.grade=2 THEN '小康生活'
    WHEN s.grade=3 THEN '可以娶媳妇'
    WHEN s.grade=4 THEN '可以买车'
    WHEN s.grade=5 THEN '可以买房'
    ELSE '土豪'
    END  建议信息
FROM emp e INNER JOIN salarygrade s 
ON e.salary BETWEEN s.losalary AND s.hisalary
ORDER BY s.grade DESC;
```



#### IF高级函数

IF函数也是用于条件判断,  语法如下

```mysql
IF(条件, '条件成立返回的值', '条件不成立返回的值')
```

例子：

```mysql
-- 工资(月薪)+奖金大于等于20000的员工 显示'收入不错'，否则显示'继续努力'
SELECT e.`ename`, e.`salary` + IFNULL(e.`bonus`, 0) 总收入, IF(e.`salary`+IFNULL(e.`bonus`, 0)>=20000, "收入不错", "继续努力") FROM emp e;
```





## 综合案例练习

### 目标

练习上面学习过的函数



#### **练习1**

需求：计算员工的日薪(按30天)，截断保留二位小数

![1595432330140](MySQL存储过程-视图.assets/1595432330140.png)

```mysql
-- 需求：计算员工的日薪(按30天)，截断保留二位小数
SELECT e.`ename`, ROUND(e.`salary`/30,2) 日薪  FROM emp e;
```



#### 练习2

需求：计算出员工的年薪，并且以年薪降序排序

![1595427707940](MySQL存储过程-视图.assets/1595427707940.png)

```mysql
-- 需求：计算出员工的年薪，并且以年薪排序 降序
SELECT ename, salary 月薪, bonus 奖金, (salary + IFNULL(bonus, 0)) * 12 年薪 FROM emp ORDER BY 年薪 DESC;
```



#### 练习3

找出奖金少于5000或者没有获得奖金的员工的信息

![1595427763714](../../../../Documents/Java笔记/就业班/2.数据库/day18(MySQL存储过程&视图)/资料/img/1595427763714.png)

```mysql
-- 需求： 找出奖金少于5000或者没有获得奖金的员工的信息

SELECT e.`ename`,e.bonus 奖金  FROM emp e WHERE e.bonus<5000 OR e.bonus IS NULL;
```



#### 练习4

需求：返回员工职务名称及其从事此职务的最低工资

![1595428068993](MySQL存储过程-视图.assets/1595428068993.png)

```mysql
-- 需求4： 返回员工职务名称及其从事此职务的最低工资
-- 查看哪些表:  emp e , job
-- 清除笛卡尔积 e.job_id = j.id
-- 条件      分组 group by e.job_id
-- 查询字段  
SELECT  j.`jname`, MIN(e.`salary`)  FROM emp e INNER JOIN  job j
ON e.`job_id` = j.`id`  
GROUP BY j.`id`;
```



#### 练习5

需求：返回工龄超过10年，且2月份入职的员工信息

![1595428107322](MySQL存储过程-视图.assets/1595428107322.png)

```mysql
-- 需求5：返回工龄超过10年，且2月份入职的员工信息

SELECT  * FROM emp e WHERE DATEDIFF(NOW(),e.`joindate`)/365>10 AND MONTH(e.`joindate`)=2;
```



#### 练习6

需求：返回与 猪八戒 同一年入职的员工

![1595428140902](MySQL存储过程-视图.assets/1595428140902.png)

```mysql
-- 需求6：返回与 猪八戒 同一年入职的员工

-- 1. 找到二师兄入职的年份
SELECT YEAR(e.`joindate`) FROM emp e WHERE e.`ename`='猪八戒';

-- 2. 找到猪八戒 同一年入职的员工
SELECT * FROM emp e  WHERE YEAR(e.`joindate`) = (SELECT YEAR(e.`joindate`) FROM emp e WHERE e.`ename`='猪八戒') AND e.`ename`!='猪八戒';
```



#### 练习7

需求：返回工资为二等级（工资等级表）的职员名字（员工表）、部门名称（部门表）

![1595428184689](MySQL存储过程-视图.assets/1595428184689.png)

```mysql
-- 需求7： 返回工资为二等级（工资等级表）的职员名字（员工表）、部门名称（部门表）

-- 确认表   emp e , dept d ,  salarygrade  s

-- 清除笛卡尔积 e.dept_id = d.id  , e.salary between s.losalary and s.hisalary

-- 额外条件  s.grande=2

-- 确认字段 

SELECT e.`ename`,d.`dname`,s.`grade` FROM  emp e
INNER JOIN dept d ON e.`dept_id` = d.id
INNER JOIN salarygrade s ON e.`salary` BETWEEN s.`losalary` AND s.`hisalary`
WHERE s.`grade`=2;

```



#### 练习8

需求：涨工资：董事长2000 经理1500 其他800

![1595428240162](../../../../Documents/Java笔记/就业班/2.数据库/day18(MySQL存储过程&视图)/资料/img/1595428240162.png)

```mysql
-- 练习8：涨工资：董事长2000 经理1500 其他800


-- 确认表   emp e ,job j

-- 清除笛卡尔积  e.job_id = j.id 

-- 额外条件 无

-- 确认字段 

SELECT e.ename, j.`jname`, e.`salary`,
  CASE j.`jname`
  WHEN '董事长' THEN e.`salary`+2000
  WHEN '经理' THEN e.`salary`+1500
  ELSE  e.`salary`+800
  END 涨后  
  FROM emp e
INNER JOIN job j ON e.job_id = j.`id`;
```





## 存储过程入门

### 目标

学习存储过程的创建，修改，查看，删除



####  存储过程和函数概述

存储过程和存储函数是**事先经过编译并存储在数据库中的一段 SQL 语句**的集合，调用存储过程和函数可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。

**优点：**

1 . 减少数据在数据库和应用服务器之间的传输，提高效率。



**缺点：** 

1. 由于存储过程是需要把SQL语句保存在MySQL的服务器上，在真实环境中，普通程序根本是不可能可以接触到真实数据库环境的，那么会造成调试非常的麻烦。 

  2. 数据库的迁移，存储过程是没法迁移。



![1597547355013](MySQL存储过程-视图.assets/1597547355013.png)

####  创建存储过程

在MySQL中的存储过程或者是存储函数是没有{}表示范围，以BEGIN开头，以END结尾

**语法**

```mysql
CREATE PROCEDURE procedure_name([proc_parameter[,...]])
BEGIN
-- 多条SQL语句
END;
```

**示例**

```mysql
-- 创建一个打印hello world的存储过程
-- 注意： mysql中默认结束语句使用; 但是创建一个存储过程出现了多次; 时候mysql就不知道你的函数的范围在哪里
-- 所以我们在定义函数或者是存储过程的时候一定要修改默认的结束符号。 delimiter 关键字就可以修改结束符号
-- 这个语句则代表了mysql的结束符号以及变成$
DELIMITER $  
CREATE PROCEDURE print_hello()
BEGIN
  SELECT 'hello world';
END $
DELIMITER ;  -- 把MySQL的结束符号修改回;
```

**DELIMITER**

该关键字用来声明SQL语句的分隔符 , 告诉 MySQL 解释器，该段命令是否已经结束了，MySQL是否可以执行了。

默认情况下，delimiter是分号;。在命令行客户端中，如果有一行命令以分号结束，那么回车后，MySQL将会执行该命令。



#### 调用存储过程

**语法**

```mysql
CALL procedure_name();
```

**示例：**

```mysql
-- 调用print_hello的存储过程
CALL print_hello();
```



#### 查看存储过程（了解）

```mysql
SELECT * FROM mysql.proc WHERE db='数据库名';

-- 查询存储过程的状态信息
SHOW PROCEDURE STATUS;

-- 查询某个存储过程的定义
SHOW CREATE PROCEDURE print_hello;
```

**结果：**

![1595429063604](MySQL存储过程-视图.assets/1595429063604.png)

![1595429071503](MySQL存储过程-视图.assets/1595429071503.png)

![1595429075754](MySQL存储过程-视图.assets/1595429075754.png)



#### 删除存储过程

**语法**

```mysql
DROP PROCEDURE [IF EXISTS] sp_name；
```

**示例**

```mysql
-- 删除存储过程
DROP PROCEDURE print_hello;
```



### 小结

创建存储过程

```mysql
DELIMITER $
CREATE PROCEDURE 存储过程名()
BEGIN
	SQL语句;
END $
DELIMITER ;
```



调用存储过程

```mysql
CALL 存储过程名();
```



删除存储过程

```mysql
DROP PROCEDURE 存储过程名();
```




## 存储过程编程-变量(了解)

存储过程是可以编程的，意味着可以使用变量，表达式，控制结构，来完成比较复杂的功能。



### 目标

在存储过程中使用变量



#### DECLARE定义变量

通过`DECLARE`可以定义一个局部变量，该变量的作用范围只能在`BEGIN…END`块中，语法: 

```mysql
DECLARE var_name  type [DEFAULT 值];
```

**示例**

```sql
DELIMITER $
CREATE PROCEDURE pro_test01()
BEGIN
	-- 定义一个变量
	DECLARE number INT DEFAULT 5; -- int n = 5;
	SELECT number + 10;
END $

DELIMITER ;

CALL pro_test01();
```

**效果**

![1596889239231](MySQL存储过程-视图.assets/1596889239231.png)



#### SET给变量赋值

直接赋值使用 SET，可以赋常量或者赋表达式，具体语法如下：

```mysql
SET var_name = expr [, var_name = expr] ... 
```

**示例**



```mysql
DELIMITER $
CREATE PROCEDURE pro_test02()
BEGIN
  --  定义一个变量
  DECLARE str VARCHAR(30) DEFAULT '中国';
  -- 给变量设置值   通过set命令给变量设置值
  SET str = '我爱java，java不爱我';
  -- 查看变量
  SELECT str;
END $
DELIMITER ;

CALL pro_test02();
```

**效果**

![1595572831537](MySQL存储过程-视图.assets/1595572831537.png)



#### 通过SELECT... INTO方式进行赋值操作 

```mysql
DELIMITER $
CREATE PROCEDURE pro_test03()
BEGIN
    -- 定义一个变量可以用于接收查询字段数值
    DECLARE num INT;
    -- 查询员工的总数给num赋值   给变量赋值的方式二： select 字段名 into 变量名 from 表 
    SELECT COUNT(*) INTO num FROM emp;
    -- 查看变量的值
    SELECT num;
END $
DELIMITER ;

CALL pro_test03();
```

**效果**

![1595573066601](MySQL存储过程-视图.assets/1595573066601.png)





#### IF关键条件判断

**语法格式**

```mysql
if 条件1 then 代码1
[elseif 条件2 then 代码2] ...
[else 代码n]
end if;
```

**示例**

需求：根据定义的身高变量，判定当前身高的所属的身材类型

180 及以上 ----------> 身材高挑

170 - 180 ---------> 标准身材

170 以下 ----------> 一般身材

![1595429514160](MySQL存储过程-视图.assets/1595429514160.png)

```mysql
DELIMITER $
CREATE PROCEDURE pro_test04()
BEGIN 
  -- 定义一个变量存储身高
  DECLARE height INT DEFAULT 175;
  -- 定义一个变量存储结果
  DECLARE result VARCHAR(10);
  -- 判断身高
  IF height>=180 THEN 
       SET result = '身材高挑';
  ELSEIF height>=170 THEN 
       SET result = "标准身材";
  ELSE 
       SET result = "一般身材";      
  -- 结束if语句
  END IF;
  
  SELECT result;
END $
DELIMITER ;

-- 调用存储过程
CALL pro_test04();
```



### 小结

IF语句的格式

```sql
IF 条件1 THEN 代码1;
[ELSEIF 条件2 THEN 代码2]
[ELSE 代码3]
END IF;
```



##  存储过程编程-参数和返回值

### 目标

在存储过程中传递参数和返回数据



####  语法格式 

```mysql
CREATE PROCEDURE procedure_name([in/out/inout] 参数名 参数类型)
...
IN :  该参数可以作为输入，也就是需要调用方传入值 , 默认
OUT:  该参数作为输出，也就是该参数可以作为返回值
INOUT: 既可以作为输入参数，也可以作为输出参数
```

**示例**

需求： 根据定义的身高变量，判定当前身高的所属的身材类型 

![1595429577377](MySQL存储过程-视图.assets/1595429577377.png)

#### IN-输入参数

```mysql
-- 传入参数  IN 代表是输入参数
DELIMITER $
CREATE PROCEDURE pro_test05(IN height INT)
BEGIN 
  -- 定义一个变量存储结果
  DECLARE result VARCHAR(10);
  -- 判断身高
  IF height>=180 THEN 
       SET result = '身材高挑';
  ELSEIF height>=170 THEN 
       SET result = "标准身材";
  ELSE 
       SET result = "一般身材";
  -- 结束if语句
  END IF;
  
  SELECT result;
END $
DELIMITER ;

CALL pro_test05(163);
```



#### OUT-输出参数(相当于返回值)

需求:根据传入的身高变量，获取当前身高的所属的身材类型   

**示例**

```mysql
-- OUT 代表的是输出参数
DELIMITER $
CREATE PROCEDURE pro_test06(IN height INT, OUT result VARCHAR(10))
BEGIN 
  -- 判断身高
  IF height>=180 THEN 
       SET result = '身材高挑';
  ELSEIF height>=170 THEN 
       SET result = "标准身材";
  ELSE 
       SET result = "一般身材";      
  -- 结束if语句
  END IF;
END $
DELIMITER ;


-- 调用的时候输出参数需要使用一个类似于变量的内容去接收即可。
CALL pro_test06(175,@abc); -- @符号表示的是本次会话，只要不关闭连接，abc这个变量都可以使用。 

SELECT @abc;
```

**效果**

![1595429638534](MySQL存储过程-视图.assets/1595429638534.png)

> 扩展
>
> @: 表示用户会话变量
>
> @@: 表示系统变量

例如

```sql
@xxx: 在变量名称前面加上“@”符号，叫做用户会话变量，表示本次会话只要不关闭连接,这个变量都可以使用。
```



## 存储过程编程-循环

### 目标

在存储过程中使用循环




#### WHILE循环

**语法结构**

```mysql
WHILE 条件 DO
	代码;
END WHILE;
```

**需求:**计算从 1加到n的值

![1595429704908](MySQL存储过程-视图.assets/1595429704908.png)

```mysql
-- 需求： 定义一个存储过程求1~n的和
DELIMITER $
CREATE PROCEDURE pro_test07(IN n INT)
BEGIN 
	-- 定义求和变量
	DECLARE SUM INT DEFAULT 0;
	-- 定义变量i, i从1变化到n
	DECLARE i INT DEFAULT 1;
	
	WHILE i <= n DO
		SET SUM = SUM + i; -- 求和
		SET i = i+1; -- i + 1
	END WHILE;
	
	SELECT SUM;
END $
DELIMITER ;

-- 调用存储过程
CALL pro_test07(5);
```



#### REPEAT循环

有条件的循环控制语句，当满足条件的时候退出循环 。WHILE是满足条件才执行，REPEAT是满足条件就退出循环。

语法结构 :

```mysql
REPEAT
	重复执行的代码;
UNTIL 条件
END REPEAT;

REPEAT当UNTIL的条件为true结束了循环
```

需求：计算从 1加到n的值 

![1595429762072](MySQL存储过程-视图.assets/1595429762072.png)

```mysql
-- 需求：定义一个存储过程求1~n的和
DELIMITER $
CREATE PROCEDURE pro_test08(IN n INT)
BEGIN 
	-- 定义一个变量存储的求和结果
	DECLARE SUM INT DEFAULT 0;
	-- 定义变量i, 从1变化到n
	DECLARE i INT DEFAULT 1;
	
	REPEAT
		SET SUM = SUM + i;
		SET i = i + 1;
	UNTIL i > n
	END REPEAT;
	
	SELECT SUM;
	
END $
DELIMITER ;

-- 调用存储过程
CALL pro_test08(100);
```



#### LOOP语句

LOOP实现简单的循环，退出循环的条件需要使用其他的语句定义，通常可以使用 LEAVE 语句实现，具体语法如下：

```mysql
LOOP
  循环体;
END LOOP



标签名: LOOP
  IF 结束循环的条件 THEN 
    LEAVE 标签名;	-- LEAVE相当于Java中的break;
  END IF;
  
  循环体;
END LOOP 标签名;
```

如果增加退出循环的语句，那么 LOOP 语句可以用来实现简单的死循环。

标签名: 为循环前缀名，当使用leave关键字退出循环时要设置这个名字



**LEAVE语句**

![1595429908291](MySQL存储过程-视图.assets/1595429908291.png)

用来从标注的流程构造中退出，通常和 BEGIN ... END 或者循环一起使用。下面是一个使用 LOOP 和 LEAVE 的简单例子 , 退出循环：

```mysql
-- 需求：定义一个存储过程求1~n的和
DELIMITER $
CREATE PROCEDURE pro_test09(IN n INT)
BEGIN 
	-- 定义一个变量存储的求和结果
	DECLARE SUM INT DEFAULT 0;
	-- 定义变量i, 从1变化到n
	DECLARE i INT DEFAULT 1;
	
	getSum:LOOP
		-- 结束循环的条件
		IF i > n THEN
			LEAVE getSum; -- 离开LOOP循环, 相当于break结束循环
		END IF;
		SET SUM = SUM + i;
		SET i = i + 1;
	END LOOP getSum;
	
	SELECT SUM;
END $
DELIMITER ;

-- 调用函数
CALL pro_test09(100);
```

==注意: leave后面必须设置循环前缀名, 否则报错==



#### 游标/光标(了解)

结果集: 通过查询语句查到的数据就是结果集

==游标是用来存储查询结果集的数据类型==, 在存储过程和函数中可以使用光标对结果集进行循环的处理。游标的使用

包括游标的声明、OPEN、FETCH 和 CLOSE，其语法分别如下。

**声明游标：**

```mysql
DECLARE 游标名字 CURSOR FOR 查询语句;
```

**OPEN 游标**

```mysql
OPEN 游标名字;
```

**FETCH 游标：**

```mysql
FETCH 游标名字 INTO 变量名1 ,变量2...
```

**CLOSE 游标：**

```mysql
CLOSE 游标名字;
```

**示例需求 :**由于公司业务拓展，当前新增了一张表user表，将emp表中的id和ename数据保存到user表中。

```sql
CREATE TABLE USER (
	id INT,
	ename VARCHAR(50)
);
```

**示例代码:**

```mysql
DELIMITER $
CREATE PROCEDURE moveData()
BEGIN
   -- 定义两个变量存储数据
   DECLARE lid INT;
   DECLARE lNAME VARCHAR(30);
   -- 定义一个变量用于结束循环语句
   DECLARE num INT DEFAULT 1;
   
   -- 定义一个游标存储查询的结果
   DECLARE result_data CURSOR FOR SELECT e.id,e.ename FROM emp e;
   -- 定义mysql游标退出机制 ， 游标不断抓取的数据的时候，如果一旦没有抓取到数据，就会触发该机制，然后把num设置为0.
   DECLARE EXIT HANDLER FOR NOT FOUND SET num=0;
   
   
   -- 打开游标
   OPEN result_data;
   
   REPEAT
       -- 抓取里面的数据 每FETCH一次就是抓取一行的数据
       FETCH result_data INTO lid, lNAME;
       -- 把数据插入user表中
       INSERT INTO USER VALUES(lid, lNAME);
   UNTIL num=0
   END REPEAT;
   
   -- 关闭游标
   CLOSE result_data;
END $
DELIMITER ;

CALL moveData();
```



## 存储函数

函数与存储过程的区别在于，存储过程不能有返回值，函数可以返回值。

**语法结构**

```mysql
CREATE FUNCTION 函数名(参数列表)
RETURNS 返回值类型
BEGIN
	代码;
END;
```

### 案例

####  需求

定义一个存储函数, 计算指定员工编号的年薪返回 ;

![1595430186593](../../../../Documents/Java笔记/就业班/2.数据库/day18(MySQL存储过程&视图)/资料/img/1595430186593.png)

```mysql
-- 定义一个存储函数, 计算指定员工编号的年薪返回;
DELIMITER $
CREATE FUNCTION getMoney(eid INT)
RETURNS INT
BEGIN
	-- 定义个变量保存年薪
	DECLARE money INT DEFAULT 0;
	-- 计算某个员工的年薪 = (月工资+奖金) * 12
	SELECT (salary + IFNULL(bonus, 0)) * 12 年薪 INTO money FROM emp WHERE id=eid;
	
	RETURN money;
END $
DELIMITER ;

-- 调用函数
SELECT getMoney(1009);
```



### 小结

函数和存储过程的区别

存储过程没有返回值,函数可以有返回值





## 总结

1. 会使用mysql字符串函数

   char_length: 获取字符串的长度

   concat: 连接字符串

   substr: 截取字符串

   trim: 去掉左右空格

2. 会使用mysql日期函数

   adddate(date, n): 在date的时间基础上加上n天

   datediff(date1, date2): date1-date2相差的天数

   curdate(): 当前日期 年月日

   curtime(): 当前时间 时分秒

   now(): 当前日期时间 年月日 时分秒

   year(date): 获取年

   month(date): 获取月份

3. 能够独立完成mysql综合练习

   各种函数的练习

4. 理解mysql表自关联

   自己连接自己, 找到两张表的表连接条件