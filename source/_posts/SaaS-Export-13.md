---
title: SaaS-Export(13)
tags:
  - 笔记
  - 项目实战一
  - PDF报表
  - Jaspersoft Studio
  - jaspser模板
  - jaspser
categories:
  - 项目实战一
date: 2020-12-04 16:09:44
---



### 学习目标

1、理解Jaspersoft的开发流程

2、能够使用模板工具Jaspersoft Studio绘制模板

3、理解数据的两种填充方式

4、完成报运单的PDF导出





### 1. 商品销售统计排行

#### 需求分析

产品销售量要求按前5名统计，并以柱状图的形式显示。

#### 柱状图

图1：参考官网实例

![1561690533938](SaaS-Export-13.assets/1561690533938.png)

图2：代码如下

![1561690555449](SaaS-Export-13.assets/1561690555449.png)

```js
option = {
    xAxis: {
        type: 'category',
        data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    },
    yAxis: {
        type: 'value'
    },
    series: [{
        data: [120, 200, 150, 80, 70, 110, 130],
        type: 'bar'
    }]
};
```



#### 步骤

1. 数据库SQL实现
2. dao接口
3. dao接口映射
4. service接口
5. service实现
6. controller控制器
7. 页面

#### 实现

1. 数据库SQL实现

   ```sql
   -- 产品销售量要求按前5名统计，并以柱状图的形式显示。
   SELECT p.`product_no` `name`,SUM(p.`amount`) `value` FROM co_contract_product p 
   WHERE p.`company_id` = '1' GROUP BY p.`product_no`  
   ORDER BY SUM(p.`amount`) DESC LIMIT 5;
   ```

2. dao接口

   ```java
   package cn.itcast.dao.stat;
   
   import java.util.List;
   import java.util.Map;
   
   public interface StatDao {
   
      
   
       //统计产品的销售额
       List<Map<String,Object>> getSellData(String companyId);
   }
   
   ```

3. dao接口映射

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="cn.itcast.dao.stat.StatDao">
      
   
       <!--//统计产品的销售额
       List<Map<String,Object>> getSellData(String companyId);-->
       <select id="getSellData" resultType="map">
          SELECT p.`product_no` `name`,SUM(p.`amount`) `value` FROM co_contract_product p
           WHERE p.`company_id` = #{companyId} GROUP BY p.`product_no`
           ORDER BY SUM(p.`amount`) DESC LIMIT 5
       </select>
   
   </mapper>
   ```

4. service接口

   ```java
   public interface StatService {
   
      //获取产品销售额
       List<Map<String,Object>> getSellData(String companyId);
   }
   ```

5. service实现

   ```java
   package cn.itcast.service.stat.impl;
   
   import cn.itcast.dao.stat.StatDao;
   import cn.itcast.service.stat.StatService;
   import com.alibaba.dubbo.config.annotation.Service;
   import org.springframework.beans.factory.annotation.Autowired;
   
   import java.util.List;
   import java.util.Map;
   
   @Service
   public class StatServiceImpl implements StatService {
   
       @Autowired
       private StatDao statDao;
   
   
       //获取产品的销售前五名
     @Override
       public List<Map<String, Object>> getSellData(String companyId) {
           return statDao.getSellData(companyId);
       }
   }
   
   ```

6. controller控制器

   ```java
   package cn.itcast.web.controller.stat;
   
   import cn.itcast.service.stat.StatService;
   import cn.itcast.web.controller.BaseController;
   import com.alibaba.dubbo.config.annotation.Reference;
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.ResponseBody;
   
   import java.util.List;
   import java.util.Map;
   
   @Controller
   @RequestMapping("/stat")
   public class StatController extends BaseController {
   
       @Reference
       private StatService statService;
   
     
   
      
        /*
          作用：获取产品的销售额前五名
          url: /stat/getSellData.do
          参数：无
          返回数据： 获取产品的销售额前五名
       */
       @RequestMapping("/getSellData")
       @ResponseBody
       public List<Map<String,Object>> getSellData(){
           return statService.getSellData(getLoginUserCompanyId());
       }
   
   }
   
   ```

7. 页面

   参考

   

   ![1557770823575](SaaS-Export-13.assets/1557770823575-1572448333995.png)

   stat-sell.jsp

   ==页面的div的宽度不够宽，需要调整到900或者1000==

   ![1599786289143](SaaS-Export-13.assets/1599786289143.png)

   ```jsp
   <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <%@ include file="../base.jsp"%>
   <!DOCTYPE html>
   <html>
   
   <head>
       <!-- 页面meta -->
       <meta charset="utf-8">
       <meta http-equiv="X-UA-Compatible" content="IE=edge">
       <title>数据 - AdminLTE2定制版</title>
       <meta name="description" content="AdminLTE2定制版">
       <meta name="keywords" content="AdminLTE2定制版">
       <!-- Tell the browser to be responsive to screen width -->
       <meta content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no" name="viewport">
       <!-- 页面meta /-->
   </head>
   <body>
   <div id="frameContent" class="content-wrapper" style="margin-left:0px;">
       <section class="content-header">
           <h1>
               统计分析
               <small>厂家销量统计</small>
           </h1>
       </section>
       <section class="content">
           <div class="box box-primary">
               <div id="main" style="width: 1000px;height:400px;"></div>
           </div>
       </section>
   </div>
   </body>
   
   <script src="../plugins/jQuery/jquery-2.2.3.min.js"></script>
   <script src="../../plugins/echarts/echarts.min.js"></script>
   <script type="text/javascript">
       $.ajax({
           url:"/stat/getSellData.do",
           type:"get",
           dataType:"json", //服务返回的数据类型
           success:function(resultData){
               // 基于准备好的dom，初始化echarts实例
               var myChart = echarts.init(document.getElementById('main'));
   
               //定义一个数组存储所有的产品名字 name
               var nameArray = [];
               //定义一个数组存储所有的产品值  value
               var valueArray = [];
   
               for(var i = 0 ; i<resultData.length ; i++){
                  var obj =  resultData[i]; //
                   nameArray[i] = obj.name;
                   valueArray[i] = obj.value;
   
               }
   
   
               // 指定图表的配置项和数据
               option = {
                   xAxis: {
                       type: 'category',
                       data: nameArray
                   },
                   yAxis: {
                       type: 'value'
                   },
                   series: [{
                       data: valueArray,
                       type: 'bar'
                   }]
               };
   
   
               // 使用刚指定的配置项和数据显示图表。
               myChart.setOption(option);
           }
       })
   </script>
   
   </html>
   ```



### 2. 系统访问压力图

#### 需求分析

按小时统计访问人数，通过折线图显示（也可以叫做系统访问压力图）。统计st_sys_log表如下：

![1557771012158](SaaS-Export-13.assets/1557771012158-1572448333996.png)

#### 折线图

图1：参考官网

![1561690679757](SaaS-Export-13.assets/1561690679757.png)

图2：

![1561690691441](SaaS-Export-13.assets/1561690691441.png)

```js
option = {
    xAxis: {
        type: 'category',
        data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    },
    yAxis: {
        type: 'value'
    },
    series: [{
        data: [820, 932, 901, 934, 1290, 1330, 1320],
        type: 'line'
    }]
};

```



#### 步骤

1. 数据库SQL实现
2. dao接口
3. dao接口映射
4. service接口
5. service实现
6. controller控制器
7. 页面

#### 实现

1. 数据库SQL实现

   ```sql
   -- 按小时统计访问人数，通过折线图显示（也可以叫做系统访问压力图）
   -- 按照小时分组
   -- 左外连接：左表为主，左表的数据不管是否符合条件全部都会显示
   -- mysql中有一个内置的函数： ifnull(字段，默认值)
   SELECT i.`A1` `name`, IFNULL(temp.num,0) `value` FROM st_online_info i LEFT JOIN (
   SELECT DATE_FORMAT(l.`time`,'%H') h,COUNT(*) num
   FROM st_sys_log l GROUP BY DATE_FORMAT(l.`time`,'%H')) temp
   ON i.`A1` = temp.h
   
   ```

2. dao接口

   ```java
   package cn.itcast.dao.stat;
   
   import java.util.List;
   import java.util.Map;
   
   public interface StatDao {
   
   
         //统计每小时访问人数
       List<Map<String,Object>> getOnLineData();
   }
   
   ```

3. dao接口映射

   ```java
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="cn.itcast.dao.stat.StatDao">
     
      <!--//统计每小时访问人数
       List<Map<String,Object>> getOnLineData();-->
       <select id="getOnLineData" resultType="map">
           SELECT i.`A1` `name`, IFNULL(temp.num,0) `value` FROM st_online_info i LEFT JOIN (
           SELECT DATE_FORMAT(l.`time`,'%H') h,COUNT(*) num
           FROM st_sys_log l GROUP BY DATE_FORMAT(l.`time`,'%H')) temp
           ON i.`A1` = temp.h
       </select>
   
   </mapper>
   ```

4. service接口

   ```java
   package cn.itcast.service.stat;
   
   import java.util.List;
   import java.util.Map;
   
   public interface StatService {
   
   
   
       //统计每小时访问人数
       List<Map<String,Object>> getOnLineData();
   
   }
   
   ```

5. service实现

   ```java
   package cn.itcast.service.stat.impl;
   
   import cn.itcast.dao.stat.StatDao;
   import cn.itcast.service.stat.StatService;
   import com.alibaba.dubbo.config.annotation.Service;
   import org.springframework.beans.factory.annotation.Autowired;
   
   import java.util.List;
   import java.util.Map;
   
   @Service
   public class StatServiceImpl implements StatService {
   
       @Autowired
       private StatDao statDao;
   
     
       //统计每小时访问人数
       @Override
       public List<Map<String, Object>> getOnLineData() {
           return statDao.getOnLineData();
       }
   }
   
   ```

6. controller控制器

   ```java
   package cn.itcast.web.controller.stat;
   
   import cn.itcast.service.stat.StatService;
   import cn.itcast.web.controller.BaseController;
   import com.alibaba.dubbo.config.annotation.Reference;
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.ResponseBody;
   
   import java.util.List;
   import java.util.Map;
   
   @Controller
   @RequestMapping("/stat")
   public class StatController extends BaseController {
   
       @Reference
       private StatService statService;
   
     /*
         作用：统计每小时访问人数
         url: /stat/getOnlineData.do
         参数：无
         返回数据： 统计每小时访问人数
      */
       @RequestMapping("/getOnLineData")
       @ResponseBody
       public List<Map<String,Object>> getOnLineData(){
           return statService.getOnLineData();
       }
   }
   
   ```

7. 页面

   参考：

   ![1557771534207](SaaS-Export-13.assets/1557771534207-1572448333996.png)

   stat-online.jsp

   ```jsp
   <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <%@ include file="../base.jsp"%>
   <!DOCTYPE html>
   <html>
   
   <head>
       <!-- 页面meta -->
       <meta charset="utf-8">
       <meta http-equiv="X-UA-Compatible" content="IE=edge">
       <title>数据 - AdminLTE2定制版</title>
       <meta name="description" content="AdminLTE2定制版">
       <meta name="keywords" content="AdminLTE2定制版">
       <!-- Tell the browser to be responsive to screen width -->
       <meta content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no" name="viewport">
       <!-- 页面meta /-->
   
   </head>
   <body>
   <div id="frameContent" class="content-wrapper" style="margin-left:0px;">
       <section class="content-header">
           <h1>
               统计分析
               <small>在线人数统计</small>
           </h1>
       </section>
       <section class="content">
           <div class="box box-primary">
               <div id="main" style="width: 600px;height:400px;"></div>
           </div>
       </section>
   </div>
   </body>
   
   <script src="../plugins/jQuery/jquery-2.2.3.min.js"></script>
   <script src="../../plugins/echarts/echarts.min.js"></script>
   <script type="text/javascript">
       $.ajax({
           url:"/stat/getOnLineData.do",
           type:"get",
           dataType:"json", //服务返回的数据类型
           success:function(resultData){
               // 基于准备好的dom，初始化echarts实例
               var myChart = echarts.init(document.getElementById('main'));
   
               //定义一个数组存储所有的产品名字 name
               var nameArray = [];
               //定义一个数组存储所有的产品值  value
               var valueArray = [];
   
               for(var i = 0 ; i<resultData.length ; i++){
                   var obj =  resultData[i]; //
                   nameArray[i] = obj.name;
                   valueArray[i] = obj.value;
   
               }
   
   
               // 指定图表的配置项和数据
               option = {
                   xAxis: {
                       type: 'category',
                       data: nameArray
                   },
                   yAxis: {
                       type: 'value'
                   },
                   series: [{
                       data: valueArray,
                       type: 'line'
                   }]
               };
   
   
   
               // 使用刚指定的配置项和数据显示图表。
               myChart.setOption(option);
           }
       })
   </script>
   
   </html>
   ```

#### 测试

![1557771684477](SaaS-Export-13.assets/1557771684477-1572448333996.png)



### 3. PDF报表概述

#### 概述

​	在企业级应用开发中，报表生成、报表打印下载是其重要的一个环节。在之前的课程中我们已经学习了报表中比较重要的一种：Excel报表。其实除了Excel报表之外，PDF报表也有广泛的应用场景，例如货运详情，货运单等。

#### JasperReport框架的介绍

​	JasperReport是一个强大、灵活的报表生成工具，能够展示丰富的页面内容，并将之转换成PDF，HTML，或者XML格式。该库完全由Java写成，可以用于在各种Java应用程序，包括J2EE，Web应用程序中生成动态内容。只需要将JasperReport引入工程中即可完成PDF报表的编译、显示、输出等工作。

​	在开源的JAVA报表工具中，JASPER Report发展是比较好的，比一些商业的报表引擎做得还好，如支持了十字交叉报表、统计报表、图形报表，支持多种报表格式的输出，如PDF、RTF、XML、CSV、XHTML、TEXT、DOCX以及OpenOffice。

​	数据源支持更多，常用 JDBC SQL查询、XML文件、CSV文件 、HQL（Hibernate查询），HBase，JAVA集合等。还允许你义自己的数据源，通过JASPER文件及数据源，JASPER就能生成最终用户想要的文档格式。

![img](SaaS-Export-13.assets/clip_image002.jpg)

#### JasperReport生命周期(重点)

​	通常我们提到PDF报表的时候,浮现在脑海中的是最终的PDF文档文件。在JasperReports中，这只是报表生命周期的最后阶段。通过JasperReports生成PDF报表一共要经过三个阶段，我们称之为 JasperReport的生命周期，这三个阶段为：<font color=red>设计（Design）阶段、执行（Execution）阶段以及输出（Export）阶段</font>，如下图所示：

![img](SaaS-Export-13.assets/clip_image001.png)

* 设计阶段（Design）：<font color=red>画模板</font>
  所谓的报表设计就是创建一些模板，模板包含了报表的布局与设计，包括执行计算的复杂公式、可选的从数据源获取数据的查询语句、以及其它的一些信息。模板设计完成之后，我们将模板保存为JRXML文件（JR代表JasperReports）,其实就是一个XML文件。
* 执行阶段（Execution）：<font color=red>模板 + 数据</font>
  使用以JRXML文件编译为可执行的二进制文件（即.Jasper文件）结合数据进行执行，填充报表数据
* 输出阶段（Export）：<font color=red>展示。 将模板和数据一起展示。</font>
  数据填充结束，可以指定输出为多种形式的报表

#### JasperReport执行流程（重点+）

![1558185485478](SaaS-Export-13.assets/1558185485478.png)

1. JRXML:报表填充模板，本质是一个XML.
   JasperReport已经封装了一个dtd，只要按照规定的格式写这个xml文件，那么jasperReport就可以将其解析最终生成报表，但是jasperReport所解析的不是我们常见的.xml文件，而是.jrxml文件，其实跟xml是一样的，只是后缀不一样。

2. Jasper:由JRXML模板编译生成的二进制文件，用于代码填充数据。
   解析完成后JasperReport就开始编译.jrxml文件，将其编译成.jasper文件，因为JasperReport只可以对.jasper文件进行填充数据和转换，这步操作就跟我们java中将java文件编译成class文件是一样的
3. .Jrprint:当用数据填充完Jasper后生成的文件，用于输出报表。
   这一步才是JasperReport的核心所在，它会根据你在xml里面写好的查询语句来查询指定是数据库，也可以控制在后台编写查询语句，参数，数据库。在报表填充完后，会再生成一个.jrprint格式的文件（读取jasper文件进行填充，然后生成一个jrprint文件）
4. Exporter:决定要输出的报表为何种格式，报表输出的管理类。
5. Jasperreport	可以输出多种格式的报表文件，常见的有Html,PDF,xls等



### 4. Jaspersoft Studio 模板工具

#### 概述

​	Jaspersoft Studio是JasperReports库和JasperReports服务器的基于Eclipse的报告设计器; 它可以作为Eclipse插件或作为独立的应用程序使用。Jaspersoft Studio允许您创建包含图表，图像，子报表，交叉表等的复杂布局。您可以通过JDBC，TableModels，JavaBeans，XML，Hibernate，大数据（如Hive），CSV，XML / A以及自定义来源等各种来源访问数据，然后将报告发布为PDF，RTF， XML，XLS，CSV，HTML，XHTML，文本，DOCX或OpenOffice。

​	Jaspersoft Studio 是一个可视化的报表设计工具,使用该软件可以方便地对报表进行可视化的设计，设计结果为格式.jrxml 的 XML 文件，并且可以把.jrxml 文件编译成.jasper 格式文件方便 JasperReport 报表引擎解析、显示。

#### 安装配置

1. 到JasperReport官网下载 https://community.jaspersoft.com/community-download

   ![1558185952754](SaaS-Export-13.assets/1558185952754.png)

2. 下载后，安装：TIB_js-studiocomm_6.5.0.final_windows_x86_64.exe。 直接下一步下一步即可。

3. 主界面

   ![1558186944087](SaaS-Export-13.assets/1558186944087.png)


#### 基本使用

##### 如何创建模板？

1. 打开Jaspersoft Studio ，新建一个project, 步骤： File -> New -> Project-> JasperReports Project

   ![img](SaaS-Export-13.assets/clip_image001-1558186971568.png)

2. 下一步，输入项目名称：

   ![1558187047734](SaaS-Export-13.assets/1558187047734.png)

3. 创建JasperReport模板

   图1：

   ![1558187131571](SaaS-Export-13.assets/1558187131571.png)

   图2：

   ![1558187184582](SaaS-Export-13.assets/1558187184582.png)

   图3：

   ![1558187228576](SaaS-Export-13.assets/1558187228576.png)

   图4：

   ![1558188161418](SaaS-Export-13.assets/1558188161418.png)

##### 面版说明

* Title(标题)：只在整个报表的第一页的最上端显示。只在第一页显示，其他页面均不显示。

* Page Header(页头，页眉)：在整个报表中每一页都会显示。在第一页中，出现的位置在 Title Band的下面。在除了第一页的其他页面中Page Header 的内容均在页面的最上端显示。

* Page Footer(页脚)：在整个报表中每一页都会显示。显示在页面的最下端。一般用来显示页码。

* Detail 1(详细)：报表内容，每一页都会显示。

* Column Header(列头)：Detail中打印的是一张表的话，这Column Header就是表中列的列头。

* Column Footer(表尾)：Detail中打印的是一张表的话，这Column Footer就是表中列的表尾。

* Summary(最后要显示内容)：表格的合计段，出现在整个报表的最后一页中，在Detail 1 Band后面。主要是用来做报表的合计显示。

##### 完善模板、编译模板

第一步：可以把不需要的面版删除：这里只留了Title、Summary

![1558188445259](SaaS-Export-13.assets/1558188445259.png)

第二步：拖入Image到面版中，输入url地址：http://www.itcast.cn/2018czgw/images/logo.png

![1558188616398](SaaS-Export-13.assets/1558188616398.png)

第三步：拖入Static Text，静态文本

![1558188861621](SaaS-Export-13.assets/1558188861621.png)

第四步：点击预览

![1558188902869](SaaS-Export-13.assets/1558188902869.png)

第四步：现在已经创建了jrxml模板，现在需要编译它：

![1558188982833](SaaS-Export-13.assets/1558188982833.png)

编译结果：

![1558189010403](SaaS-Export-13.assets/1558189010403.png)



### 5. java整合jaspser模板文件

#### 需求

实现出口报运模块列表，点击下载，展示pdf文件：

![1558189557644](SaaS-Export-13.assets/1558189557644.png)

#### 依赖

在export_web_manager项目中添加jasper依赖包：

```xml
<!--jasper依赖包-->
<dependency>
  <groupId>net.sf.jasperreports</groupId>
  <artifactId>jasperreports</artifactId>
  <version>6.5.0</version>
</dependency>
<dependency>
  <groupId>org.olap4j</groupId>
  <artifactId>olap4j</artifactId>
  <version>1.2.0</version>
</dependency>
<dependency>
  <groupId>com.lowagie</groupId>
  <artifactId>itext</artifactId>
  <version>2.1.7</version>
</dependency>
```

#### 拷贝jasper文件

1. 拷贝pdf模板编译后的jasper文件到项目中

   ![1558189809273](SaaS-Export-13.assets/1558189809273.png)

2. 拷贝到项目中

   &nbsp;![1558189859496](SaaS-Export-13.assets/1558189859496.png)

#### 编写控制器类

```java
package cn.itcast.web.controller.cargo;

import cn.itcast.domain.cargo.*;
import cn.itcast.service.cargo.ContractService;
import cn.itcast.service.cargo.ExportProductService;
import cn.itcast.service.cargo.ExportService;
import cn.itcast.vo.ExportProductVo;
import cn.itcast.vo.ExportResult;
import cn.itcast.vo.ExportVo;
import cn.itcast.web.controller.BaseController;
import com.alibaba.dubbo.config.annotation.Reference;
import com.github.pagehelper.PageInfo;
import net.sf.jasperreports.engine.*;
import org.apache.cxf.jaxrs.client.WebClient;
import org.springframework.beans.BeanUtils;
import org.springframework.stereotype.Controller;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import java.io.InputStream;
import java.util.HashMap;
import java.util.List;

@Controller
@RequestMapping("/cargo/export")
public class ExportController extends BaseController {

    @Reference
    private ContractService contractService;

    @Reference
    private ExportService exportService;

    @Reference
    private ExportProductService exportProductService ;

    
 /*
       作用： 下载报运单
       url:  /cargo/export/exportPdf.do?id=4028d3cf567275410156735276210001
       参数：报运单的id
       返回值 : 下载
    */
    @RequestMapping("/exportPdf")
    @ResponseBody
    public void exportPdf(String id) throws Exception {

        //1. 得到模板的输入流
        InputStream inputStream = session.getServletContext().getResourceAsStream("/jasper/test01.jasper");

        //2.把模板与数据填充， 拿到JasperPrint对象（数据+模板结合）
        /*
        fillReport(InputStream inputStream, Map<String, Object> parameters, JRDataSource dataSource)
                inputStream: 模板的输入流
                parameters: 需要被填充的参数，不需要被遍历的
                dataSource: 数据源，需要被遍历的数据
         */

        JasperPrint jasperPrint = JasperFillManager.fillReport(inputStream, new HashMap<>(), new JREmptyDataSource());


        //3. 把pdf文件输出
        /*
            exportReportToPdfStream(JasperPrint jasperPrint, OutputStream outputStream)
                    jasperPrint： jasperprint的对象
                    outputStream: 输出的目标地址的输出流对象
         */
        JasperExportManager.exportReportToPdfStream(jasperPrint,response.getOutputStream());

    }

}

```

#### 测试

第一步：点击下载

![1558191025176](SaaS-Export-13.assets/1558191025176.png)

第二步: 查看pdf文件

![1558191044298](SaaS-Export-13.assets/1558191044298.png)



### 6. java整合jaspser模板文件 中文处理

#### 问题

显示的pdf没有中文字体：

![1558191044298](SaaS-Export-13.assets/1558191044298.png)

配置的模板中有中文描述：

![1558191167364](SaaS-Export-13.assets/1558191167364.png)

#### 原因

中文需要设置字体，且项目要引入字体模板。这样才可以正常显示中文。

#### 解决

第一步：修改模板、重新编译

![1575082191951](SaaS-Export-13.assets/1575082191951.png)

第二步：把重新编译后的jasper文件拷贝到项目中

&nbsp;![1558191464287](SaaS-Export-13.assets/1558191464287.png)

第三步：项目引入中文字体文件

&nbsp;![1558191568745](SaaS-Export-13.assets/1558191568745.png)

第四步：测试

注意：如果jasper模板文件名称变了，需要修改PdfController加载的文件名称。

![1558191695053](SaaS-Export-13.assets/1558191695053.png)





### 7. 数据填充（一）参数map填充

#### 需求

<font color=red>通过java代码往jasper模板中传入参数。</font>

#### 步骤

第一步： <font color=blue>定义模板。</font>先定义模板、在模板中定义一些参数。

模板预览：

![1558443280343](SaaS-Export-13.assets/1558443280343.png)

第二步：<font color=blue>导出PDF。</font>通过java代码，往模板中设置参数，导出pdf。

效果预览：

![1558443338162](SaaS-Export-13.assets/1558443338162.png)



#### 第一步：定义模板

① 拖入frame

![1558443973301](SaaS-Export-13.assets/1558443973301.png)

② 设置frame边框

![1558444152991](SaaS-Export-13.assets/1558444152991.png)



③ 拖入Line到frame、设置宽度与frame一致。

![1558444362840](SaaS-Export-13.assets/1558444362840.png)

④ 设置h为1px后

![1558444481113](SaaS-Export-13.assets/1558444481113.png)

⑤ 预览

![1558444994344](SaaS-Export-13.assets/1558444994344.png)

⑥设置Parameter

图1：

![1558445082217](SaaS-Export-13.assets/1558445082217.png)

图2：

![1558445120800](SaaS-Export-13.assets/1558445120800.png)

⑦ 制作模板

![1558445349401](SaaS-Export-13.assets/1558445349401.png)

⑧ 修改文字大小、修改文字字体、编译模板生成jasper文件、把jasper文件拷贝到项目中。

#### 第二步：导出PDF

① 拷贝jasper文件到项目中

&nbsp;![1558445442666](SaaS-Export-13.assets/1558445442666.png)

② 编写控制器

```java
package cn.itcast.web.controller.cargo;

import cn.itcast.domain.cargo.*;
import cn.itcast.service.cargo.ContractService;
import cn.itcast.service.cargo.ExportProductService;
import cn.itcast.service.cargo.ExportService;
import cn.itcast.vo.ExportProductVo;
import cn.itcast.vo.ExportResult;
import cn.itcast.vo.ExportVo;
import cn.itcast.web.controller.BaseController;
import com.alibaba.dubbo.config.annotation.Reference;
import com.github.pagehelper.PageInfo;
import net.sf.jasperreports.engine.*;
import org.apache.cxf.jaxrs.client.WebClient;
import org.springframework.beans.BeanUtils;
import org.springframework.stereotype.Controller;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import java.io.InputStream;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Controller
@RequestMapping("/cargo/export")
public class ExportController extends BaseController {

    @Reference
    private ContractService contractService;

    @Reference
    private ExportService exportService;

    @Reference
    private ExportProductService exportProductService ;

   
   /*
       作用： 下载报运单
       url:  /cargo/export/exportPdf.do?id=4028d3cf567275410156735276210001
       参数：报运单的id
       返回值 : 下载
    */
    @RequestMapping("/exportPdf")
    @ResponseBody
    public void exportPdf(String id) throws Exception {

        //1. 得到模板的输入流
        InputStream inputStream = session.getServletContext().getResourceAsStream("/jasper/test02.jasper");

        //2.把模板与数据填充， 拿到JasperPrint对象（数据+模板结合）
        /*
        fillReport(InputStream inputStream, Map<String, Object> parameters, JRDataSource dataSource)
                inputStream: 模板的输入流
                parameters: 需要被填充的参数，不需要被遍历的
                dataSource: 数据源，需要被遍历的数据
         */
        Map<String,Object> map = new HashMap<>();
        map.put("userName","狗娃");
        map.put("email","1234@itcast.cn");
        map.put("companyName","X内教育");
        map.put("deptName","学工部");



        JasperPrint jasperPrint = JasperFillManager.fillReport(inputStream,map, new JREmptyDataSource());


        //3. 把pdf文件输出
        /*
            exportReportToPdfStream(JasperPrint jasperPrint, OutputStream outputStream)
                    jasperPrint： jasperprint的对象
                    outputStream: 输出的目标地址的输出流对象
         */
        JasperExportManager.exportReportToPdfStream(jasperPrint,response.getOutputStream());

    }


}

```

③ 测试

![1558445486943](SaaS-Export-13.assets/1558445486943.png)



### 8. 数据填充（二）JDBC数据源 A 配置数据源 （不常用，了解即可）

#### 分析

> 数据源填充数据分为：JDBC数据源填充数据（数据库连接）、JavaBean填充数据（list集合）

> 如图：

![1558450067316](SaaS-Export-13.assets/1558450067316.png)

#### 配置JDBC数据源

图1：

&nbsp;![1558450269214](SaaS-Export-13.assets/1558450269214.png)

图2：

&nbsp;![1558450306369](SaaS-Export-13.assets/1558450306369.png)

图3：配置数据库连接信息

&nbsp;![1558450439131](SaaS-Export-13.assets/1558450439131.png)

图4：选择驱动、点击完成

&nbsp;![1558450620489](SaaS-Export-13.assets/1558450620489.png)

图5：配置结果

&nbsp;![1558450635061](SaaS-Export-13.assets/1558450635061.png)



### 9. 数据填充（三）JDBC数据源 B 制作模板

#### 需求

制作模板，预览效果为：

![1558450908809](SaaS-Export-13.assets/1558450908809.png)

#### 制作模板

##### 第一步：新建模板

① 新建jasper模板

&nbsp;![1558451118255](SaaS-Export-13.assets/1558451118255.png)

输入模板名称：

&nbsp;![1558451184874](SaaS-Export-13.assets/1558451184874.png)

下一步，完成

&nbsp;![1558451228367](SaaS-Export-13.assets/1558451228367.png)

##### 第二步：模板制作

①创建空白模板后，并将不需要的Band删除， 只留下如下面版：

&nbsp;![1558451363404](SaaS-Export-13.assets/1558451363404.png)

②新建数据源

图1：

&nbsp;![1558451452321](SaaS-Export-13.assets/1558451452321.png)

图2：

&nbsp;![1558451663532](SaaS-Export-13.assets/1558451663532.png)

图3： 删除不需要的字段，只留下如下四个字段：

&nbsp;![1558451741103](SaaS-Export-13.assets/1558451741103.png)

③ 新建数据源后，自动生成如下四个Fields

&nbsp;![1558451872573](SaaS-Export-13.assets/1558451872573.png)

④ 构造模板

&nbsp;![1558452269949](SaaS-Export-13.assets/1558452269949.png)

⑤ 预览效果

&nbsp;![1558452308540](SaaS-Export-13.assets/1558452308540.png)

##### 第三步：制作细节

①由于模板数据有空格，需要去除数据的空格

图1：双击空白区域

&nbsp;![1558452453825](SaaS-Export-13.assets/1558452453825.png)

图2：

![1558452468759](SaaS-Export-13.assets/1558452468759.png)

图3：预览

![1558452491790](SaaS-Export-13.assets/1558452491790.png)

②由于没有边框，现在设置边框

图1：

![1558452595871](SaaS-Export-13.assets/1558452595871.png)

图2：预览 (设置字体)

![1558452619170](SaaS-Export-13.assets/1558452619170.png)







### 10. 数据填充（四）JDBC数据源 C 代码实现

#### 预览

导出数据源配置的表数据，效果如下：

![1558453705712](SaaS-Export-13.assets/1558453705712.png)

这里的数据是从pe_user表查询出来的，在jasper模板中配置的数据源。

#### 实现

第一步：拷贝生成的jasper文件到项目中

&nbsp;![1558453664469](SaaS-Export-13.assets/1558453664469.png)

第二步：实现导出PDF

```java
package cn.itcast.web.controller.cargo;

import cn.itcast.domain.cargo.*;
import cn.itcast.service.cargo.ContractService;
import cn.itcast.service.cargo.ExportProductService;
import cn.itcast.service.cargo.ExportService;
import cn.itcast.vo.ExportProductVo;
import cn.itcast.vo.ExportResult;
import cn.itcast.vo.ExportVo;
import cn.itcast.web.controller.BaseController;
import com.alibaba.dubbo.config.annotation.Reference;
import com.github.pagehelper.PageInfo;
import net.sf.jasperreports.engine.*;
import org.apache.cxf.jaxrs.client.WebClient;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import javax.sql.DataSource;
import java.io.InputStream;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.UUID;

@Controller
@RequestMapping("/cargo/export")
public class ExportController extends BaseController {

    @Reference
    private ContractService contractService;

    @Reference
    private ExportService exportService;

    @Reference
    private ExportProductService exportProductService;

  

     @Autowired
    private DataSource dataSource;

    /*
       作用： 下载报运单
       url:  /cargo/export/exportPdf.do?id=4028d3cf567275410156735276210001
       参数：报运单的id
       返回值 : 下载
    */
    @RequestMapping("/exportPdf")
    @ResponseBody
    public void exportPdf(String id) throws Exception {

        //1. 得到模板的输入流
        InputStream inputStream = session.getServletContext().getResourceAsStream("/jasper/test03.jasper");

        //2.把模板与数据填充， 拿到JasperPrint对象（数据+模板结合）
        /*
        fillReport(InputStream inputStream, Map<String, Object> parameters, JRDataSource dataSource)
                inputStream: 模板的输入流
                parameters: 需要被填充的参数，不需要被遍历的
                dataSource: 数据源，需要被遍历的数据
         */


        JasperPrint jasperPrint = JasperFillManager.fillReport(inputStream,new HashMap<>(), dataSource.getConnection());


        //3. 把pdf文件输出
        /*
            exportReportToPdfStream(JasperPrint jasperPrint, OutputStream outputStream)
                    jasperPrint： jasperprint的对象
                    outputStream: 输出的目标地址的输出流对象
         */
        JasperExportManager.exportReportToPdfStream(jasperPrint,response.getOutputStream());

    }



}
```



### 11. 数据填充（五）JavaBean数据源(重点)

#### 需求

刚才我们实现了jdbc数据源，直接查询表中的数据填充jasper模板。但是很多时候我们需要对数据进行处理，再把处理后的数据填充到jasper模板。所以，我们就需要把处理后的数据用javabean封装，这里就用到了javabean数据源。

我们的实现步骤分为下面2步：

1. 制作jasper模板
2. 导出pdf

#### 制作模板

![1558604428239](SaaS-Export-13.assets/1558604428239.png)

#### 导出pdf

先把编译后的jasper模板文件拷贝到项目中，再在控制器添加如下方法：

```java
package cn.itcast.web.controller.cargo;

import cn.itcast.domain.cargo.*;
import cn.itcast.domain.system.User;
import cn.itcast.service.cargo.ContractService;
import cn.itcast.service.cargo.ExportProductService;
import cn.itcast.service.cargo.ExportService;
import cn.itcast.vo.ExportProductVo;
import cn.itcast.vo.ExportResult;
import cn.itcast.vo.ExportVo;
import cn.itcast.web.controller.BaseController;
import com.alibaba.dubbo.config.annotation.Reference;
import com.github.pagehelper.PageInfo;
import net.sf.jasperreports.engine.*;
import net.sf.jasperreports.engine.data.JRBeanCollectionDataSource;
import org.apache.cxf.jaxrs.client.WebClient;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import javax.sql.DataSource;
import java.io.InputStream;
import java.util.*;

@Controller
@RequestMapping("/cargo/export")
public class ExportController extends BaseController {

    @Reference
    private ContractService contractService;

    @Reference
    private ExportService exportService;

    @Reference
    private ExportProductService exportProductService;


    @Autowired
    private DataSource dataSource;

      /*
       作用： 下载报运单
       url:  /cargo/export/exportPdf.do?id=4028d3cf567275410156735276210001
       参数：报运单的id
       返回值 : 下载
    */
    @RequestMapping("/exportPdf")
    @ResponseBody
    public void exportPdf(String id) throws Exception {

        //1. 得到模板的输入流
        InputStream inputStream = session.getServletContext().getResourceAsStream("/jasper/test04.jasper");

        //2.把模板与数据填充， 拿到JasperPrint对象（数据+模板结合）
        /*
        fillReport(InputStream inputStream, Map<String, Object> parameters, JRDataSource dataSource)
                inputStream: 模板的输入流
                parameters: 需要被填充的参数，不需要被遍历的
                dataSource: 数据源，需要被遍历的数据
         */

        List<User> list = new ArrayList<>() ;
        for (int i = 0; i < 10; i++) {
            User user = new User();
            user.setUserName("家锦"+i+"号");
            user.setEmail("jiajin@163.com");
            user.setCompanyName("网易");
            user.setDeptName("传销部");
            list.add(user);
        }
        //把集合封装到数据源中
        JRBeanCollectionDataSource dataSource = new JRBeanCollectionDataSource(list);


        JasperPrint jasperPrint = JasperFillManager.fillReport(inputStream,new HashMap<>(),dataSource);


        //3. 把pdf文件输出
        /*
            exportReportToPdfStream(JasperPrint jasperPrint, OutputStream outputStream)
                    jasperPrint： jasperprint的对象
                    outputStream: 输出的目标地址的输出流对象
         */
        JasperExportManager.exportReportToPdfStream(jasperPrint,response.getOutputStream());

    }


}
```

#### 效果

![1558604755417](SaaS-Export-13.assets/1558604755417.png)





### 12. 数据填充（六）分组报表(了解)

#### 需求

根据企业进行分组：

![1558608298675](SaaS-Export-13.assets/1558608298675.png)



#### 制作模板

① 新建模板：test06_group.jrxml

②新建Fields

&nbsp;![1558605826007](SaaS-Export-13.assets/1558605826007.png)

③ 新建组

图1：

![1558605705620](SaaS-Export-13.assets/1558605705620.png)

图2：

&nbsp;![1558608235853](SaaS-Export-13.assets/1558608235853.png)

图3：

&nbsp;![1558605767618](SaaS-Export-13.assets/1558605767618.png)

④ 编辑模板 - 组

图1：按照哪个字段进行分组，就拖入对应字段

![1558605931519](SaaS-Export-13.assets/1558605931519.png)

图2：编辑组名称

&nbsp;![1558606247929](SaaS-Export-13.assets/1558606247929.png)

图3：双击组名称

![1558606318806](SaaS-Export-13.assets/1558606318806.png)

图4： 再次拖入分组字段到Footer中

![1558606421774](SaaS-Export-13.assets/1558606421774.png)

图5：输入统计信息放入Footer

&nbsp;![1558606533309](SaaS-Export-13.assets/1558606533309.png)

⑤ 添加Field、完成模板制作

&nbsp;![1558606872984](SaaS-Export-13.assets/1558606872984.png)

#### 导出pdf

先把编译后的jasper模板文件拷贝到项目中，再在控制器添加如下方法：

```java
package cn.itcast.web.controller.cargo;

import cn.itcast.domain.cargo.*;
import cn.itcast.domain.system.User;
import cn.itcast.service.cargo.ContractService;
import cn.itcast.service.cargo.ExportProductService;
import cn.itcast.service.cargo.ExportService;
import cn.itcast.vo.ExportProductVo;
import cn.itcast.vo.ExportResult;
import cn.itcast.vo.ExportVo;
import cn.itcast.web.controller.BaseController;
import com.alibaba.dubbo.config.annotation.Reference;
import com.github.pagehelper.PageInfo;
import net.sf.jasperreports.engine.*;
import net.sf.jasperreports.engine.data.JRBeanCollectionDataSource;
import org.apache.cxf.jaxrs.client.WebClient;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import javax.sql.DataSource;
import java.io.InputStream;
import java.util.*;

@Controller
@RequestMapping("/cargo/export")
public class ExportController extends BaseController {

    @Reference
    private ContractService contractService;

    @Reference
    private ExportService exportService;

    @Reference
    private ExportProductService exportProductService;

    @Autowired
    private DataSource dataSource;

  
    /*
       作用： 下载报运单
       url:  /cargo/export/exportPdf.do?id=4028d3cf567275410156735276210001
       参数：报运单的id
       返回值 : 下载
    */
    @RequestMapping("/exportPdf")
    @ResponseBody
    public void exportPdf(String id) throws Exception {

        //1. 得到模板的输入流
        InputStream inputStream = session.getServletContext().getResourceAsStream("/jasper/test05.jasper");

        //2.把模板与数据填充， 拿到JasperPrint对象（数据+模板结合）
        /*
        fillReport(InputStream inputStream, Map<String, Object> parameters, JRDataSource dataSource)
                inputStream: 模板的输入流
                parameters: 需要被填充的参数，不需要被遍历的
                dataSource: 数据源，需要被遍历的数据
         */

        List<User> list = new ArrayList<>() ;
        for (int j = 0; j < 5; j++) {
            for (int i = 0; i < 10; i++) {
                User user = new User();
                user.setUserName("家锦"+i+"号");
                user.setEmail("jiajin@163.com");
                user.setCompanyName("网易"+j);
                user.setDeptName("传销部");
                list.add(user);
            }
        }
        //把集合封装到数据源中
        JRBeanCollectionDataSource dataSource = new JRBeanCollectionDataSource(list);


        JasperPrint jasperPrint = JasperFillManager.fillReport(inputStream,new HashMap<>(),dataSource);


        //3. 把pdf文件输出
        /*
            exportReportToPdfStream(JasperPrint jasperPrint, OutputStream outputStream)
                    jasperPrint： jasperprint的对象
                    outputStream: 输出的目标地址的输出流对象
         */
        JasperExportManager.exportReportToPdfStream(jasperPrint,response.getOutputStream());

    }



}
```



### 13. 数据填充（七）图形报表 

#### 需求

根据统计需求，统计数据，导出pdf以图形方式进行统计分析。参考如下：

![1558617569927](SaaS-Export-13.assets/1558617569927.png)

#### 制作模板

①创建模板、创建Fields

![1558616286362](SaaS-Export-13.assets/1558616286362.png)

②添加Charts

图1：

![1558616334074](SaaS-Export-13.assets/1558616334074.png)

图2：

![1558616398120](SaaS-Export-13.assets/1558616398120.png)

③设置饼图属性

图1：

![1558616531299](SaaS-Export-13.assets/1558616531299.png)

图2：

![1558617354792](SaaS-Export-13.assets/1558617354792.png)

图3：

![1558617418528](SaaS-Export-13.assets/1558617418528.png)

图3：

&nbsp;![1558617475780](SaaS-Export-13.assets/1558617475780.png)

④最后pdf模板

&nbsp;![1558616862575](SaaS-Export-13.assets/1558616862575.png)

<font color=red>注意:  设置了标题，一定设置 标题 +  饼图的字体。</font>

#### 导出pdf

先把编译后的jasper模板文件拷贝到项目中，再在控制器添加如下方法：

```java
package cn.itcast.web.controller.cargo;

import cn.itcast.domain.cargo.*;
import cn.itcast.domain.system.User;
import cn.itcast.service.cargo.ContractService;
import cn.itcast.service.cargo.ExportProductService;
import cn.itcast.service.cargo.ExportService;
import cn.itcast.vo.ExportProductVo;
import cn.itcast.vo.ExportResult;
import cn.itcast.vo.ExportVo;
import cn.itcast.web.controller.BaseController;
import com.alibaba.dubbo.config.annotation.Reference;
import com.github.pagehelper.PageInfo;
import net.sf.jasperreports.engine.*;
import net.sf.jasperreports.engine.data.JRBeanCollectionDataSource;
import org.apache.cxf.jaxrs.client.WebClient;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import javax.sql.DataSource;
import java.io.InputStream;
import java.util.*;

@Controller
@RequestMapping("/cargo/export")
public class ExportController extends BaseController {

    @Reference
    private ContractService contractService;

    @Reference
    private ExportService exportService;

    @Reference
    private ExportProductService exportProductService;

   
    /*
       作用： 下载报运单
       url:  /cargo/export/exportPdf.do?id=4028d3cf567275410156735276210001
       参数：报运单的id
       返回值 : 下载
    */
    @RequestMapping("/exportPdf")
    @ResponseBody
    public void exportPdf(String id) throws Exception {

        //1. 得到模板的输入流
        InputStream inputStream = session.getServletContext().getResourceAsStream("/jasper/test06.jasper");

        //2.把模板与数据填充， 拿到JasperPrint对象（数据+模板结合）
        /*
        fillReport(InputStream inputStream, Map<String, Object> parameters, JRDataSource dataSource)
                inputStream: 模板的输入流
                parameters: 需要被填充的参数，不需要被遍历的
                dataSource: 数据源，需要被遍历的数据
         */

        List<Map<String,Object>> list = new ArrayList<>();
        for (int i = 0; i <5 ; i++) {
            Map<String,Object> map = new HashMap<>();
            map.put("title","李总传销第"+i+"号事业部");
            map.put("value",new Random().nextInt(100));
            list.add(map);
        }


        //把集合封装到数据源中
        JRBeanCollectionDataSource dataSource = new JRBeanCollectionDataSource(list);


        JasperPrint jasperPrint = JasperFillManager.fillReport(inputStream,new HashMap<>(),dataSource);


        //3. 把pdf文件输出
        /*
            exportReportToPdfStream(JasperPrint jasperPrint, OutputStream outputStream)
                    jasperPrint： jasperprint的对象
                    outputStream: 输出的目标地址的输出流对象
         */
        JasperExportManager.exportReportToPdfStream(jasperPrint,response.getOutputStream());

    }



}
```



### 14. 报运单详情PDF报表

#### 需求

完成报运详情的PDF报表打印：

1. 原始的Excel报表如下：

![1558620867089](SaaS-Export-13.assets/1558620867089.png)

数据来源分析：

![1562643765725](SaaS-Export-13.assets/1562643765725.png)

现在把Excel报表显示的表运单详情，改为PDF形成导出：

![1558620909614](SaaS-Export-13.assets/1558620909614.png)

#### 制作模板

报运单的jasper模板文件的制作虽然比较麻烦，但是用到的技术我们都在前面使用了。所以大家直接从资料中拷贝已经制作好的模板文件即可：

![1558620992786](SaaS-Export-13.assets/1558620992786.png)

#### 导出PDF

大家需要先在项目中引入BeanMapUtils，调用其方法把对象数据拷贝到Map中。再实现导出报运单：

```java
package cn.itcast.web.controller.cargo;

import cn.itcast.domain.cargo.*;
import cn.itcast.domain.system.User;
import cn.itcast.service.cargo.ContractService;
import cn.itcast.service.cargo.ExportProductService;
import cn.itcast.service.cargo.ExportService;
import cn.itcast.vo.ExportProductVo;
import cn.itcast.vo.ExportResult;
import cn.itcast.vo.ExportVo;
import cn.itcast.web.controller.BaseController;
import cn.itcast.web.utils.BeanMapUtils;
import com.alibaba.dubbo.config.annotation.Reference;
import com.github.pagehelper.PageInfo;
import net.sf.jasperreports.engine.*;
import net.sf.jasperreports.engine.data.JRBeanCollectionDataSource;
import org.apache.cxf.jaxrs.client.WebClient;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.sql.DataSource;
import java.io.InputStream;
import java.util.*;

@Controller
@RequestMapping("/cargo/export")
public class ExportController extends BaseController {

    @Reference
    private ContractService contractService;

    @Reference
    private ExportService exportService;

    @Reference
    private ExportProductService exportProductService;

   
  
    /*
       作用： 下载报运单
       url:  /cargo/export/exportPdf.do?id=4028d3cf567275410156735276210001
       参数：报运单的id
       返回值 : 下载
    */
    @RequestMapping("/exportPdf")
    @ResponseBody
    public void exportPdf(String id) throws Exception {

        //通知浏览器以附件的形式下载
        response.setHeader("content-disposition","attachment;filename=export.pdf");

        //1. 得到模板的输入流
        InputStream inputStream = session.getServletContext().getResourceAsStream("/jasper/export.jasper");

        //2.把模板与数据填充， 拿到JasperPrint对象（数据+模板结合）

        //根据id查找到报运单
        Export export = exportService.findById(id);
        //目标： 由于export是不需要遍历的数据应该存放到Map中
        Map<String, Object> map = BeanMapUtils.beanToMap(export);//map的key就是对象的属性名， 值：属性值

        //查找该报运单下的所有货物
        ExportProductExample exportProductExample = new ExportProductExample();
        exportProductExample.createCriteria().andExportIdEqualTo(id);
        List<ExportProduct> productList = exportProductService.findAll(exportProductExample);


        //把集合封装到数据源中
        JRBeanCollectionDataSource dataSource = new JRBeanCollectionDataSource(productList);


        JasperPrint jasperPrint = JasperFillManager.fillReport(inputStream,map,dataSource);


        //3. 把pdf文件输出
        /*
            exportReportToPdfStream(JasperPrint jasperPrint, OutputStream outputStream)
                    jasperPrint： jasperprint的对象
                    outputStream: 输出的目标地址的输出流对象
         */
        JasperExportManager.exportReportToPdfStream(jasperPrint,response.getOutputStream());

    }



}
```









