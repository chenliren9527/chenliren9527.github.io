---
title: maven高级
tags:
  - 笔记
  - maven
  - maven私服
  - 依赖版本冲突
  - 传递依赖 
categories:
  - maven
date: 2020-11-14 12:27:10
---

### 00、学习目标

1、理解传递依赖 

2、掌握如何解决依赖版本冲突解决 

3、能够使用maven构建SSM工程 

4、学习使用maven分模块方式构建工程 

5、了解搭建私服的使用 



### 01、maven核心概念（1）maven介绍

#### 1.1 目标

掌握maven基本概念、作用。

#### 1.2 什么是maven？

==maven： 项目管理构建工具==，Maven翻译为“专家”、“内行”；是一个采用纯Java编写的开源项目管理工具，Maven采用了一种被称之为Project Object Model (POM)概念来管理项目，所有的项目配置信息都被定义在一个叫做POM.xml的文件中, 通过该文件Maven可以管理项目的整个声明周期，包括清除、编译、测试、报告、打包、部署等。



**maven项目管理：** 

==1、管理项目jar包==

==2、管理项目的生命周期：==

清理clean:   		删除target目录（直接delete也可以）

编译compile:      重新生成target

测试test：           测试项目

打包package：   把项目打成jar包或war包, 如果聚合项目pom

安装install：        把项目通过install安装到本地库。当我们自己开发的项目组件需要给别的项目去依赖，首先就要先确保项目在本地库中存在，其他项目才可以从本地库去依赖。

部署deploy：       把项目部署的远程仓库，比如私服



**maven作用二：构建项目：** 

把多个模块构建成一个项目。单独给某一个模块提供性能

maven如何对项目进行管理、构建？  pom.xml



#### 1.3 常见的项目管理构建工具

1. ant  (过时)
2. maven
3. gradle  (偏向于移动端的开发)

#### 1.4 小结

> 为什么要用maven，作用是什么？

1. 作用一： 管理项目
   - 管理项目jar
   - 管理项目生命周期
2. 作用二： 构建项目
   - 多个模块构建成一个项目



### 02、maven核心概念（2）仓库

#### 2.1 目标

理解仓库的作用、仓库分类。

#### 2.2 maven的仓库

存储jar包的地方就是仓库。

#### 2.3 仓库分类

1. 本地仓库
2. 远程仓库
   1. 中央仓库
   2. 第三方公共库： 阿里云
   3. 私服

![image-20200523065113529](maven高级.assets/image-20200523065113529.png)



#### 2.4 项目如何查找jar包？

![image-20200523063847859](maven高级.assets/image-20200523063847859.png)

#### 2.5 小结

​	maven项目的jar包一定是存储在本地仓库吗？

​           -最终jar一定会在你的本地仓库出现的。 





### 03、maven核心概念（3）坐标

#### 目标

理解坐标作用、坐标组成部分。

#### 坐标&坐标组成

```xml
<dependency> 
  <groupId>log4j</groupId>  	  组织名: 域名的反写
  <artifactId>log4j</artifactId>  项目名
  <version>1.2.17</version>       版本
 <packaging>war</packaging>    jar(se项目)  war(web项目)  pom(聚合项目)
</dependency> 
```

#### 依赖下载失败的时候该如何处理

 1. **找到本地仓库jar所在的位置**

    - 先找组织名

    - 项目名 

      ![1598060705434](maven高级.assets/1598060705434.png)

 2. **把对应版本把删除，特别注意该版本lastupdate文件，只要有这个文件很多时候都不会重新给你下载。**



3. **刷新你当前模块，这样子就可以重新下载了。**





#### 坐标查找

http://mvnrepository.com

网站上可以搜索具体的组织或项目关键字，之后复制对应的坐标到pom.xml中。如：

![image-20200523064133377](maven高级.assets/image-20200523064133377.png)

#### 3.1 小结

**maven中为什么会出现坐标，作用是什么？**

​		maven坐标的作用： 是jar在仓库中的唯一标识



### 04、maven核心概念（4）依赖 A 依赖范围、依赖传递

#### 4.1 目标

依赖的作用，依赖范围与依赖范围的应用（依赖传递）

#### 4.2 依赖的作用

找jar包。项目添加依赖，就相当于把jar包拷贝到项目中。



所谓依赖范围，就是值一个jar包在编译时、测试时、运行时起不起作用！



#### 4.3 依赖范围

四种常用依赖范围表：

| 依赖范围               | 编译 main | 测试 test | 运行时 target |
| ---------------------- | --------- | --------- | ------------- |
| compile（默认）        | Y         | Y         | Y             |
| provided(serlvet.jar)  | Y         | Y         | -             |
| runtime（mysql驱动类） | -         | Y         | Y             |
| test(junit.jar)        | -         | Y         | -             |



图表解释说明如下：

```
compile
1. 最大的依赖范围
2. 表示在项目依赖的包如果是compile范围，表示在编译、测试、运行时期都有效
3. 会产生依赖传递
4. 举例：几乎所有包都是compile依赖范围

provided
1. 运行的时候不起作用，
2. 如果项目依赖的包如果是provided范围, 表示编译有效，实际运行无效。
3. 不会打入war包/jar包
   4.举例：servlet包，由于tomcat本身有servlet的jar

runtime
1. 运行时期有效
2. 如果项目依赖的包如果是runtime范围，编译无效，运行有效。
3. 会打入war包/jar包
4. 举例：数据库驱动包

test
1. 测试依赖范围
2. 如果项目依赖的包如果是test范围, 只能在测试程序中使用。
3. 不会打入war包/jar包
4. 举例：junit、spring-test
```



#### 4.4 依赖传递

所谓依赖传递，就是当A工程依赖B工程的时候，A工程会把B工程的依赖获取到，这就是指依赖传递。

![1596158451857](maven高级.assets/1596158451857.png)

#### 4.5 小结

1. **常用的有哪四种依赖范围？**

   - compile  （编译 ，测试 ，运行）
   - runtime  (测试，运行)
   - provide (编译，测试)
   - test(测试)

2. **什么是依赖传递**

   ​	- A项目依赖了B项目，B项目的所有依赖都会传递给A	



### 05、maven核心概念（5）依赖 B 可选依赖、依赖冲突、依赖排除

#### 5.1 目标

1. 理解可选依赖的作用
2. 如何解决依赖冲突。

#### 5.2 可选依赖

代码截图：

![1569550899539](maven高级.assets/1569550899539.png)

![1597888362362](maven高级.assets/1597888362362.png)

代码：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>projectA</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!--可选依赖：控制当前项目的依赖是否传递给依赖的项目， 使用optional标签控制
        true代表不传递， false为传递，默认就是为false。-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <!--optional控制 的是可选依赖，true为不传递，false为传递-->
            <optional>false</optional>
        </dependency>
    </dependencies>

    
</project>
```

#### 5.3 依赖冲突

说明：

1、如果2个直接依赖：依赖了同一个坐标不同版本的资源，以配置顺序下方为主。

2、如果2个间接依赖：依赖了同一个坐标不同版本的资源，以配置顺序上方为主。

3、如果1个直接依赖与1个间接依赖：依赖了同一个坐标不同版本的资源，以直接依赖为主。



演示：&nbsp;![1569551606327](maven高级.assets/1569551606327.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>projectA</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!--可选依赖：控制当前项目的依赖是否传递给依赖的项目， 使用optional标签控制
        true代表不传递， false为传递，默认就是为false。-->
        <!-- <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
           optional控制 的是可选依赖，true为不传递，false为传递
            <optional>false</optional>
        </dependency>
        -->

        <!--依赖冲突
                    情况一： 两个直接依赖冲突,以下方为主
                    情况二：两个间接依赖冲突
                    情况三：直接依赖与间接依赖冲突
        -->


        <!--情况一： 两个直接依赖冲突,以下方为主-->
<!--
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
        </dependency>
-->



       <!-- 情况二：两个间接依赖冲突 ， 以上方为主-->
       <!-- <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>


        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
-->

        <!--情况三：直接依赖与间接依赖冲突,使用直接依赖
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
-->



    </dependencies>

    
</project>
```



#### 5.4 依赖排除

第一步：依赖包

![1569551892459](maven高级.assets/1569551892459.png)

第二步：依赖排除

```xml
        <!--依赖排除：两个间接依赖冲突的时候，我们不想使用其中的某一个版本，我们就可以使用依赖排除  -->

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.0.2.RELEASE</version>
            <!--依赖排除，排除该项目spring-core的依赖-->
            <exclusions>
            <exclusion>
                <groupId>org.springframework</groupId><!-- 组织名-->
                <artifactId>spring-core</artifactId> <!--项目名-->
            </exclusion>
            </exclusions>
        </dependency>


        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>



```



#### 5.5 小结

 - **什么是可选依赖？使用哪个关键字**

   ​	可选依赖就是控制一个依赖是否传递，optional关键字

 - **依赖冲突分几种情况？**

   - 两个直接依赖冲突:  使用下方
   - 两个间接依赖冲突:  使用上方
   - 直接依赖与间接依赖冲突:  使用直接依赖

- **依赖排除是使用哪个标签?**

  ​		![1605319657906](maven高级.assets/1605319657906.png)





### 06、maven核心概念（6）分模块项目（一）继承与聚合

#### 6.1 目标

1. 理解分模块创建项目的好处
2. 理解继承与聚合的概念。

#### 6.2 分模块创建项目？

Maven多模块项目,适用于一些比较大的项目，通过合理的模块拆分，实现代码的复用，便于维护和管理。尤其是一些开源框架，也是采用多模块的方式，提供插件集成，用户可以根据需要配置指定的模块。

#### 6.3 分模块创建预览

![image-20200523075712190](maven高级.assets/image-20200523075712190.png)



#### 6.4 传统分包与分模块对比

传统分包与分模块 对比图：

![1569552952710](maven高级.assets/1569552952710.png)

通过对比，思考分模块创建项目的好处有哪些？

1. 项目架构一目了然
2. 分模块创建项目，更易于团队开发维护，提升项目扩展性、可维护性。
3. 分开模块构建可以把某一个模块做出服务，对外提供使用。提高组件复用性



#### 6.5 什么是继承与聚合？

> 继承

实际项目中，可能正要构建一个大型的系统，但又不想一遍又一遍的重复同样的依赖元素，这种情况是经常出现的。不过还好，maven提供了继承机制，<font color=red>项目可以通过parent元素使用继承，可以避免这种重复。</font>当一个项目声明一个parent的时候，它从父项目的POM中继承信息。它也可以覆盖父POM中的值，或者添加一些新的值。

==Maven的继承特性可以将各个模块相同的依赖和插件配置提取出来==，在简化POM的同时还可以促进各个模块配置的一致性。

<font color=red><b>继承，就是为了消除重复依赖配置。</b></font>

通过父项目统一抽取所有子项目公用的配置，子项目继承父项目，父项目公用的依赖会自动引入到子项目中。



> 聚合

聚合就是将一个工程拆分为多个模块。

**Maven的聚合特性**可以帮助我们把项目的多个模块聚合在一起，使用一条命令进行构建，即一条命令实现构建多个项目；



例如：一个父项目，10个小项目。在没有聚合情况下，需要在10个分别进行clean才能清空所有项目，如果有了聚合，只需要父项目那里clean一次，把10个项目全部情况！



==聚合，聚集合并，把各个子项目作为一个整体一起运行,一起管理。==

聚合目的，是为了快速构建运行多个项目





#### 6.6 小结

   继承的作用： 把子模块的公共依赖配置抽取出来，消除重复的依赖配置

   聚合的作用：把子模块当成一个整体一起去管理。



### 07、maven核心概念（7）分模块项目（二）继承与聚合

#### 目标

快速演示项目的继承与聚合。

#### 实现步骤

> 说明

![1569554020379](maven高级.assets/1569554020379.png)

>  第一步：创建父项目 project

父项目中不会写任何一行java代码与除啦pom.xml以外的其他配置，所以src目录可以保留，也可以删除

![image-20200523085440874](maven高级.assets/image-20200523085440874.png)

> 第二步： 创建子项目（子模块）project_domain

图1：新建module

![1605321678657](maven高级.assets/1605321678657.png)



图2：选择聚合项目与父项目

![image-20200523085512388](maven高级.assets/image-20200523085512388.png)

图3：

![image-20200523085526938](maven高级.assets/image-20200523085526938.png)

图4：在project_domain

![1596163301628](maven高级.assets/1596163301628.png)

最后，观察父项目

![1596163463931](maven高级.assets/1596163463931.png)



> 第三步：创建子项目project_dao

![image-20200523085623096](maven高级.assets/image-20200523085623096.png)



父工程的配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>project</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>

    <!--聚合-->
    <modules>
        <module>project_domain</module>
        <module>project_dao</module>
    </modules>

    <!--统一版本管理，定义一个类似于变量去保存版本信息-->
    <properties>
        <spring-version>5.0.2.RELEASE</spring-version>  <!--spring-version标签名是自定义的-->
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring-version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring-version}</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>


</project>
```



#### 小结

1. **父项目的作用是什么？**  抽取公共依赖配置， 把子模块聚合成一个项目

    	

2. **父项目中有什么？** 

   - pom文件
   - src文件可有可无,一般我们都是删除的

3. 继承是哪个标签体现？ 聚合是哪个标签体现？ 继承与聚合要不要我们自行编码实现？

   - parent
   - modules
   - 不用，idea自动实现

### 08、maven核心概念（8）分模块项目（三）父项目版本锁定

#### 目标

理解版本锁定概念、意义。

#### 如何实现版本锁定？

1、父项目的另外一个作用就是<font color=red><b>版本锁定</b></font>

2、版本锁定是什么？  

**版本锁定的作用：**

	1. 父模块锁定子模块的版本
	2. 阻止父模块的所有依赖直接传递，可以让子模块选择传递。减少没必要依赖传递，减轻项目体积

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>project</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>

    <!--聚合-->
    <modules>
        <module>project_domain</module>
        <module>project_dao</module>
    </modules>

    <!--统一版本管理，定义一个类似于变量去保存版本信息-->
    <properties>
        <spring-version>5.0.2.RELEASE</spring-version>  <!--spring-version标签名是自定义的-->
    </properties>


    <!--版本锁定  dependencyManagement标签去实现的
            1. 锁定依赖版本
            2. 一般使用版本锁定，父模块的依赖不会再直接传递给子模块，要求子模块选择性传递。
    -->
    <dependencyManagement>
        <dependencies>

            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>${spring-version}</version>
            </dependency>

            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-test</artifactId>
                <version>${spring-version}</version>
            </dependency>

            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>4.12</version>
            </dependency>
        </dependencies>

    </dependencyManagement>


</project>
```



子模块pom文件

```xml
<?xml version="1.0" encoding="UTF-8"?><project xmlns="http://maven.apache.org/POM/4.0.0"         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">    <parent>        <artifactId>project</artifactId>        <groupId>com.itheima</groupId>        <version>1.0-SNAPSHOT</version>    </parent>    <modelVersion>4.0.0</modelVersion>    <artifactId>project_domain</artifactId>    <dependencies>            <dependency>                <groupId>org.springframework</groupId>                <artifactId>spring-context</artifactId>            </dependency>    </dependencies></project>
```



#### 小结

版本锁定的好处与弊端?

​	好处：

			1. 锁定子模块的版本		2. 减轻子模块的jar体积

​	弊端：

​	麻烦





### 09、私服（1）介绍  （了解   私服仓库服务器）

#### 目标

1. 中央仓库的缺点
2. 什么是私服

#### 中央仓库的缺点

**下载慢-----**由于中央仓库在国外的，所以我们的下载速度比较慢

**黑名单** —— 如果某个IP地址恶意的下载中央仓库内容，例如全公司100台机器使用同一个IP反复下载，这个IP（甚至是IP段）会进入黑名单，因此稍有规模的使用Maven时，应该用Nexus架设私服。

**垃圾内容** —— 由于各种历史原因，中央仓库里面确实存在很多垃圾内容，例如不完整的POM，错误的maven-metadata.xml，主要的责任是开源项目上传内容时不太小心，目前中央仓库正致力于更规范的流程以防止新的垃圾内容进入。

**网络限制----** 如果我们在局域网的情况下我们是没法从中央仓库下载的

#### 私服

1、什么是私服？

- **私服是远程仓库的1种** (搭建在局域网中) 
- 私服的作用 (好处)
  - 解决访问限制问题
  - 即使不能上网也可以下载jar包
  - 可以上传自己的jar包

2、私服的作用

图1：查找jar包的过程

![image-20200523091048056](maven高级.assets/image-20200523091048056.png)





### 10、私服（2）安装

#### 目标

会安装私服

#### 安装

一键安装：双击install-nexus.bat即完成安装！==（使用管理员身份去安装）==

![image-20200523092632892](maven高级.assets/image-20200523092632892.png)

![1598066915919](maven高级.assets/1598066915919.png)

再检查服务是否有启动！如果没有启动一定要启动

![1569556500583](maven高级.assets/1569556500583.png)

#### 小结

1、nexus-2.12.0-01\conf\nexus.properties文件中可以查看访问路径

![image-20200523093500495](maven高级.assets/image-20200523093500495.png)

2、访问路径： http://localhost:8081/nexus

![image-20200523094049449](maven高级.assets/image-20200523094049449.png)



### 11、私服（3）上传项目到私服

#### 目标

能够实现项目上传到私服。

#### 准备

nexus默认分配两个账户

admin  admin123  超级管理员

deployment  deployment123  上传和下载jar包





先配置仓库运行重复部署：

![1569556637593](maven高级.assets/1569556637593.png)

![1569556780772](maven高级.assets/1569556780772.png)

![1605325009964](maven高级.assets/1605325009964.png)

#### 上传配置

项目上传到私服

![1598068032445](maven高级.assets/1598068032445.png)

```xml
第一步：找到maven的settings.xml把以下配置信息配置到<servers>标签中，告诉maven私服的用户名与密码	<server> 		<id>releases</id>		<username>admin</username>		<password>admin123</password>	</server>	<server>		<id>snapshots</id>		<username>deployment</username>		<password>deployment123</password>	</server>第二步：在当前模块的pom.xml配置以下信息   <distributionManagement>  	<repository>  		<id>releases</id> 	    <url>http://localhost:8081/nexus/content/repositories/releases/</url>  	</repository>   	<snapshotRepository>  		<id>snapshots</id>	    <url>http://localhost:8081/nexus/content/repositories/snapshots/</url>  	</snapshotRepository>   </distributionManagement>
```



![1596167478144](maven高级.assets/1596167478144.png)

上传：执行deploy部署命令

![1598068429733](maven高级.assets/1598068429733.png)

查看上传结果：

![1569557113476](maven高级.assets/1569557113476.png)

#### 小结

项目部署到私服的过程是?





### 12、私服（4）私服下载项目、配置阿里云远程仓库

#### 目标

能够实现从私服下载项目。

#### 私服下载资源配置

```xml
<!-- 远程仓库配置 --><mirror>    <!-- ID(唯一) -->    <id>nexus</id>    <!-- 名称 -->    <name>nexus</name>    <!-- 远程仓库的地址 -->    <url>http://127.0.0.1:8081/nexus/content/groups/public/</url>    <!-- 拦截指定仓库的请求 -->    <!-- lib/maven-model-builder-3.6.0.jar/org/apache\maven\model -->    <mirrorOf>central</mirrorOf></mirror>
```

#### 配置阿里云（在自己开发测试时候推荐使用）

```xml
<mirror>    <id>nexus-aliyun</id>    <mirrorOf>central</mirrorOf>    <name>Nexus aliyun</name>    <url>http://maven.aliyun.com/nexus/content/groups/public</url></mirror>
```





### 13、分模块构建ssm项目（1）分析 (重点)

#### 思考

以后的项目都是分模块来构建，如何分模块构建项目 ？有几种方式实现？

#### 分析

![1597900216654](maven高级.assets/1597900216654.png)





### 14、分模块构建ssm项目（2）父项目、聚合项目

#### 目标

创建父项目、添加依赖。

#### 环境准备

准备数据库环境：

![1569567109439](maven高级.assets/1569567109439.png)

#### 案例需求

实现列表显示pe_role角色表的信息

![1569567194503](maven高级.assets/1569567194503.png)

#### 创建父项目

图1：创建项目

![1569567292544](maven高级.assets/1569567292544.png)

图2：删除src目录， 添加依赖

```xml
<?xml version="1.0" encoding="UTF-8"?><project xmlns="http://maven.apache.org/POM/4.0.0"         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">    <modelVersion>4.0.0</modelVersion>    <groupId>com.itheima</groupId>    <artifactId>ssm_parent</artifactId>    <version>1.0-SNAPSHOT</version>    <!--统一定义各个组件的版本-->    <properties>        <spring_version>5.0.2.RELEASE</spring_version>        <aspectj_version>1.8.7</aspectj_version>        <junit_version>4.12</junit_version>    </properties>    <dependencies>        <!--spring核心包-->        <dependency>            <groupId>org.springframework</groupId>            <artifactId>spring-context</artifactId>            <version>${spring_version}</version>        </dependency>        <!--springmvc支持包-->        <dependency>            <groupId>org.springframework</groupId>            <artifactId>spring-webmvc</artifactId>            <version>${spring_version}</version>        </dependency>        <!--aspectj 支持包-->        <dependency>            <groupId>org.aspectj</groupId>            <artifactId>aspectjweaver</artifactId>            <version>${aspectj_version}</version>        </dependency>        <!--spring事务支持-->        <dependency>            <groupId>org.springframework</groupId>            <artifactId>spring-tx</artifactId>            <version>${spring_version}</version>        </dependency>        <!--spring jdbc支持-->        <dependency>            <groupId>org.springframework</groupId>            <artifactId>spring-jdbc</artifactId>            <version>${spring_version}</version>        </dependency>        <!--spring测试支持包-->        <dependency>            <groupId>org.springframework</groupId>            <artifactId>spring-test</artifactId>            <version>${spring_version}</version>        </dependency>        <!--junit-->        <dependency>            <groupId>junit</groupId>            <artifactId>junit</artifactId>            <version>${junit_version}</version>            <scope>test</scope>        </dependency>        <!--mybatis支持包-->        <dependency>            <groupId>org.mybatis</groupId>            <artifactId>mybatis</artifactId>            <version>3.4.5</version>        </dependency>        <!--spring整合mybatis-->        <dependency>            <groupId>org.mybatis</groupId>            <artifactId>mybatis-spring</artifactId>            <version>1.3.1</version>        </dependency>        <!--数据库驱动包-->        <dependency>            <groupId>mysql</groupId>            <artifactId>mysql-connector-java</artifactId>            <version>5.1.30</version>        </dependency>        <!--druid-->        <dependency>            <groupId>com.alibaba</groupId>            <artifactId>druid</artifactId>            <version>1.1.15</version>        </dependency>        <!--jstl支持包-->        <dependency>            <groupId>jstl</groupId>            <artifactId>jstl</artifactId>            <version>1.2</version>        </dependency>        <!--log4j日志包-->        <dependency>            <groupId>log4j</groupId>            <artifactId>log4j</artifactId>            <version>1.2.16</version>        </dependency>        <dependency>            <groupId>org.slf4j</groupId>            <artifactId>slf4j-log4j12</artifactId>            <version>1.7.29</version>        </dependency>    </dependencies></project>
```

#### 小结

核心依赖有哪些？

​		spring, springmvc, mybatis, 数据库，日志

### 15、分模块构建ssm项目（3）实体类子模块

#### 目标

创建model实体类工程。

#### 作用

实体类: 封装数据。  

1. 封装表数据
2. 封装业务数据

#### 创建工程

图1：

&nbsp;![1598078555457](maven高级.assets/1598078555457.png)

图2：编写Role角色实体类

```java
package com.itheima.domain;public class Role {    private String role_id; //角色id    private String name; //角色名字    private String remark; //说明    public Role() {    }    public Role(String role_id, String name, String remark) {        this.role_id = role_id;        this.name = name;        this.remark = remark;    }    /**     * 获取     * @return role_id     */    public String getRole_id() {        return role_id;    }    /**     * 设置     * @param role_id     */    public void setRole_id(String role_id) {        this.role_id = role_id;    }    /**     * 获取     * @return name     */    public String getName() {        return name;    }    /**     * 设置     * @param name     */    public void setName(String name) {        this.name = name;    }    /**     * 获取     * @return remark     */    public String getRemark() {        return remark;    }    /**     * 设置     * @param remark     */    public void setRemark(String remark) {        this.remark = remark;    }    public String toString() {        return "Role{role_id = " + role_id + ", name = " + name + ", remark = " + remark + "}";    }}
```







### 16、分模块构建ssm项目（4）持久化子模块

#### 目标

编写ssm_dao持久化工程。

#### 步骤

1. 创建ssm_dao项目
2. 添加依赖： model模块
3. 编写dao接口

#### 实现

1. 创建ssm_dao项目

2. 添加依赖： ssm_domain

   ```xml
   <?xml version="1.0" encoding="UTF-8"?><project xmlns="http://maven.apache.org/POM/4.0.0"         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">    <parent>        <artifactId>ssm_parent</artifactId>        <groupId>com.itheima</groupId>        <version>1.0-SNAPSHOT</version>    </parent>    <modelVersion>4.0.0</modelVersion>    <artifactId>ssm_dao</artifactId>    <dependencies>        <!--依赖实体类-->        <dependency>            <groupId>com.itheima</groupId>            <artifactId>ssm_domain</artifactId>            <version>1.0-SNAPSHOT</version>        </dependency>    </dependencies></project>
   ```

   

3. 编写dao接口

   ```java
   package com.itheima.dao;import com.itheima.domain.Role;import org.apache.ibatis.annotations.Select;import java.util.List;public interface RoleDao {    //查询所有的角色    @Select("select * from pe_role")    List<Role> findAll();}
   ```

   





### 17、分模块构建ssm项目（3）业务逻辑子模块

#### 目标

创建ssm_service业务工程。

#### 步骤

1. 创建项目：ssm_service
2. 添加依赖：ssm_dao
3. 编写service接口
4. 编写service接口实现

#### 实现

1. 创建项目：ssm_service

2. 添加依赖：ssm_dao

   ```xml
   <?xml version="1.0" encoding="UTF-8"?><project xmlns="http://maven.apache.org/POM/4.0.0"         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">    <parent>        <artifactId>ssm_parent</artifactId>        <groupId>com.itheima</groupId>        <version>1.0-SNAPSHOT</version>    </parent>    <modelVersion>4.0.0</modelVersion>    <artifactId>ssm_service</artifactId>    <!--service需要依赖dao-->    <dependencies>        <dependency>            <groupId>com.itheima</groupId>            <artifactId>ssm_dao</artifactId>            <version>1.0-SNAPSHOT</version>        </dependency>    </dependencies></project>
   ```

   

3. 编写service接口

   ![1569568793589](maven高级.assets/1569568793589.png)

   ```java
   package com.itheima.service;import com.itheima.domain.Role;import java.util.List;public interface RoleService {    List<Role> findAll();}
   ```

   

4. 编写service接口实现

   ```java
   package com.itheima.service.impl;import com.itheima.dao.RoleDao;import com.itheima.domain.Role;import com.itheima.service.RoleService;import org.springframework.beans.factory.annotation.Autowired;import org.springframework.stereotype.Service;import java.util.List;@Servicepublic class RoleServiceImpl implements RoleService {    @Autowired    private RoleDao roleDao;    @Override    public List<Role> findAll() {        return roleDao.findAll();    }}
   ```

   









### 18、分模块构建ssm项目（4）表现层子模块

#### 目标

编写ssm-web工程。

#### 步骤

1. 创建项目：ssm_web
2. 添加依赖： ssm_service
3. 配置web.xml （核心控制器,字符过滤器,监听器）
4. applicationContext.xml （扫描service，整合mybatis， 事务管理）
5. springmvc.xml(视图解析器,包扫描，注解驱动，静态资源)
6. 编写控制器
7. 页面显示
8. 启动测试



```xml
1）编写web.xml    Spring的监听器  字符编码过滤器  核心控制器2）编写applicationContext.xml             Dao层：   Spring整合Mybaits         Service层:  扫描Service包 + 声明式事务3）编写springmvc.xml        Web层： 扫描Controller 视图解析器 注解驱动  处理静态资源 ....
```





#### 预览

![1569571776888](maven高级.assets/1569571776888.png)



#### 实现

1. 创建项目：ssm_web, 转换为web项目

2. 添加依赖： ssm_service

   ![1569569978298](maven高级.assets/1569569978298.png)

3. <font color=red><b>配置web.xml</b></font>

   ```xml
   <?xml version="1.0" encoding="UTF-8"?><web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"	xmlns="http://java.sun.com/xml/ns/javaee"	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"	version="2.5">	<!--1. 配置核心控制器-->	<servlet>		<servlet-name>dispatcherServlet</servlet-name>		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>		<!--2. 加载springmvc.xml文件-->		<init-param>			<param-name>contextConfigLocation</param-name>			<param-value>classpath:springmvc.xml</param-value>		</init-param>		<load-on-startup>1</load-on-startup>	</servlet>		<servlet-mapping>		<servlet-name>dispatcherServlet</servlet-name>		<url-pattern>/</url-pattern>	</servlet-mapping>	<!--3. 字符过滤器-->	<filter>		<filter-name>characterEncodingFilter</filter-name>		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>		<init-param>			<param-name>encoding</param-name>			<param-value>utf-8</param-value>		</init-param>	</filter>	<filter-mapping>		<filter-name>characterEncodingFilter</filter-name>		<url-pattern>/*</url-pattern>	</filter-mapping>	<!--4.监听器-->	<listener>		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>	</listener>	<context-param>		<param-name>contextConfigLocation</param-name>		<param-value>classpath:applicationContext.xml</param-value>	</context-param></web-app>
   ```

4. applicationContext.xml

   **db.properties**

   ```properties
   jdbc.url=jdbc:mysql:///saas-exportjdbc.username=rootjdbc.password=rootjdbc.driver=com.mysql.jdbc.Driver
   ```

   applicationContext.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?><beans xmlns="http://www.springframework.org/schema/beans"       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"       xmlns:aop="http://www.springframework.org/schema/aop"       xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">   
       <!--1.扫描service-->    
       <context:component-scan base-package="com.itheima.service"/>    
       <!--2.spring整合mybatis-->    <!--2,1 加载db配置文件-->    
       <context:property-placeholder location="classpath:db.properties"/>    
       <!--2.2 创建连接池-->    
       <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">       
           <property name="driverClassName" value="${jdbc.driver}"/>        
           <property name="url" value="${jdbc.url}"/>        
           <property name="username" value="${jdbc.username}"/>        
           <property name="password" value="${jdbc.password}"/>    
       </bean>   
       <!--2.3 创建sqlSession工厂-->    
       <bean class="org.mybatis.spring.SqlSessionFactoryBean">        
           <property name="dataSource" ref="dataSource"/>        
           <property name="typeAliasesPackage" value="com.itheima.domain"/>   
       </bean>        <!--2.4 创建一个扫描器扫描dao包-->    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">        <property name="basePackage" value="com.itheima.dao"/>    </bean>    <!--3.事务管理-->    <!--3.1 创建事务管理器-->    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">        <property name="dataSource" ref="dataSource"/>    </bean>    <!--3.2 配置事务通知规则-->    <tx:advice id="ad" transaction-manager="transactionManager">        <tx:attributes>            <tx:method name="find*" propagation="SUPPORTS"/>            <tx:method name="select*" propagation="SUPPORTS"/>            <tx:method name="get*" propagation="SUPPORTS"/>            <tx:method name="query*" propagation="SUPPORTS"/>            <tx:method name="*" propagation="REQUIRED"/>        </tx:attributes>    </tx:advice>    <!--3.3 配置切面-->    <aop:config>        <aop:pointcut id="pt" expression="execution(* com.itheima.service.impl.*.*(..))"/>        <aop:advisor advice-ref="ad" pointcut-ref="pt"/>    </aop:config></beans>
   ```

5. springmvc.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?><beans xmlns="http://www.springframework.org/schema/beans"       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"       xmlns:context="http://www.springframework.org/schema/context"       xmlns:mvc="http://www.springframework.org/schema/mvc"       xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">    <!--1. 视图解析器-->    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">        <property name="prefix" value="/pages/"/>        <property name="suffix" value=".jsp"/>    </bean>    <!--2. 包扫描-->    <context:component-scan base-package="com.itheima.controller"/>    <!--3. 注解驱动-->    <mvc:annotation-driven/>    <!--4.静态资源访问-->    <mvc:default-servlet-handler/></beans>
   ```

6. 编写控制器

   ```java
   package com.itheima.controller;import com.itheima.domain.Role;import com.itheima.service.RoleService;import org.springframework.beans.factory.annotation.Autowired;import org.springframework.stereotype.Controller;import org.springframework.ui.Model;import org.springframework.web.bind.annotation.RequestMapping;import java.util.List;@Controller@RequestMapping("/role")public class RoleController {    @Autowired    private RoleService roleService;    @RequestMapping("list")    public String list(Model model){        //查找所有的角色        List<Role> list = roleService.findAll();        //存储到request域中        model.addAttribute("list",list);        //请求转发到list页面        return "list";    }}
   ```

7. 页面显示

   ![1569571945712](maven高级.assets/1569571945712.png)

   ```jsp
   <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %><%@ page contentType="text/html;charset=UTF-8" language="java" %><html><head>    <title>Title</title></head><body><h3>角色列表</h3><table border="1">    <tr>        <th>角色ID</th>        <th>角色名称</th>        <th>角色备注</th>    </tr>    <c:forEach items="${list}" var="role">        <tr>            <td>${role.role_id}</td>            <td>${role.name}</td>            <td>${role.remark}</td>        </tr>    </c:forEach></table></body></html>
   ```

8. 启动测试

   图1：点击

   ![1569571983521](maven高级.assets/1569571983521.png)

   图2：

   ![1569572008912](maven高级.assets/1569572008912.png)

   图3：部署项目

   ![1569572135082](maven高级.assets/1569572135082.png)

**测试**

![1569572298625](maven高级.assets/1569572298625.png)



​    
