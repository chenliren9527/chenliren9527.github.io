---
title: SaaS-Export(10)
tags:
  - 笔记
  - 项目实战一
  - ApachePOI 
  - excle
  - excle导入导出
categories:
  - 项目实战一
date: 2020-11-29 16:09:28
---



### 学习目标

1、能够理解购销合同模块下附件CRUD的实现思路

2. 独立完成货物上传功能

2、独立完成出货表打印

3、能够理解模板打印

4、能够理解SXSSFWorkbook对象的执行过程



### 1. ApachePOI 介绍

#### 需求说明

在企业级应用开发中，Excel报表是一种最常见的报表需求。Excel报表开发一般分为两种形式：

1、为了方便操作，基于Excel的报表批量上传数据   

2、通过java代码生成Excel报表。

在Saas-Export系统中，也有大量的报表操作，那么接下来的课程就是一起来学习企业级的报表开发。

#### Excel介绍

目前世面上的Excel分为两个大的版本Excel2003和Excel2007及以上两个版本；

两者之间的区别如下：

![img](SaaS-Export-10.assets/clip_image001-1572010629713.png)

Excel2003是一个特有的二进制格式，其核心结构是复合文档类型的结构，存储数据量较小；Excel2007 的核心结构是 XML 类型的结构，采用的是基于 XML 的压缩方式，使其占用的空间更小，操作效率更高。



#### 常见的Excel操作工具

Java中常见的用来操作Excl的方式一般有2种：JXL和POI。

* JXL只能对Excel进行操作,属于比较老的框架，它只支持到Excel 95-2000的版本。现在已经停止更新和维护。

* POI是apache的项目,可对微软的Word,Excel,Ppt进行操作,包括office2003和2007,Excl2003和2007。poi现在一直有更新。所以现在主流使用POI。



#### 什么是POI

​	Apache POI是Apache软件基金会的开源项目，由Java编写的免费开源的跨平台的 Java API，Apache POI提供API给Java语言操作Microsoft Office的功能。

​	Apache POI是目前最流行的操作Microsoft Office的API组件，借助POI可以方便的完成诸如：数据报表生成，数据批量上传，数据备份等工作。



### 2. ApachePOI 入门案例

#### 步骤

1. 创建项目poi_demo
2. 添加依赖
3. 编写测试类，实现导出excel、导入excel

#### 实现

1. 创建项目poi_demo

   ![1557245897648](SaaS-Export-10.assets/1557245897648-1572010629714.png)

2. 添加依赖

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>cn.itcast</groupId>
       <artifactId>poi_demo</artifactId>
       <version>1.0-SNAPSHOT</version>
   
       <dependencies>
           <dependency>
               <groupId>org.apache.poi</groupId>
               <artifactId>poi</artifactId>
               <version>4.0.1</version>
           </dependency>
           <dependency>
               <groupId>org.apache.poi</groupId>
               <artifactId>poi-ooxml</artifactId>
               <version>4.0.1</version>
           </dependency>
           <dependency>
               <groupId>org.apache.poi</groupId>
               <artifactId>poi-ooxml-schemas</artifactId>
               <version>4.0.1</version>
           </dependency>
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
           </dependency>
       </dependencies>
   </project>
   ```

3. 编写测试类，实现导出excel、导入excel

   ```java
   package cn.itcast.test;
   
   import org.apache.poi.hssf.usermodel.HSSFWorkbook;
   import org.apache.poi.ss.usermodel.Row;
   import org.apache.poi.ss.usermodel.Sheet;
   import org.apache.poi.ss.usermodel.Workbook;
   import org.apache.poi.xssf.usermodel.XSSFWorkbook;
   import org.junit.Test;
   
   import java.io.FileInputStream;
   import java.io.FileNotFoundException;
   import java.io.FileOutputStream;
   
   public class AppTest {
   
       /*
           需求： 读取03版本的excel一行数据
        */
       @Test
       public void test01() throws Exception {
           //1. 获取文件的输入流对象
           FileInputStream fileInputStream = new FileInputStream("C:\\Users\\ztl\\Desktop\\day62【saas_export_day10】\\04.工具与资料\\01 excel资源模板\\上传货物模板2.xls");
           //2. 使用文件的输入流创建一个工作簿 03的excel对应实现类:
           Workbook workbook = new HSSFWorkbook(fileInputStream);
   
           //3. 通过工作薄得到工作单
           Sheet sheet = workbook.getSheetAt(0);
   
           //4. 得到行
           Row row = sheet.getRow(0);
   
           //5.获取单元格的内容
           System.out.print(row.getCell(1).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(2).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(3).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(4).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(5).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(6).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(7).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(8).getStringCellValue()+"\t\t");
           System.out.println(row.getCell(9).getStringCellValue());
       }
   
       /*
          需求： 读取03版本所有行内容
        */
       @Test
       public void test02() throws Exception {
           //1. 获取文件的输入流对象
           FileInputStream fileInputStream = new FileInputStream("C:\\Users\\ztl\\Desktop\\day62【saas_export_day10】\\04.工具与资料\\01 excel资源模板\\上传货物模板2.xls");
           //2. 使用文件的输入流创建一个工作簿 03的excel对应实现类:
           Workbook workbook = new HSSFWorkbook(fileInputStream);
   
           //3. 通过工作薄得到工作单
           Sheet sheet = workbook.getSheetAt(0);
   
           //4. 得到行
           Row row = sheet.getRow(0);
   
           //5.获取单元格的内容
           System.out.print(row.getCell(1).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(2).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(3).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(4).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(5).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(6).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(7).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(8).getStringCellValue()+"\t\t");
           System.out.println(row.getCell(9).getStringCellValue());
   
   
           //得到单元格的行数
           int rows = sheet.getPhysicalNumberOfRows();
   
           for (int i = 1; i < rows; i++) {
               row = sheet.getRow(i);
               //获取内容
               System.out.print(row.getCell(1).getStringCellValue()+"\t\t");
               System.out.print(row.getCell(2).getStringCellValue()+"\t\t");
               System.out.print(row.getCell(3).getNumericCellValue()+"\t\t");
               System.out.print(row.getCell(4).getStringCellValue()+"\t\t");
               System.out.print(row.getCell(5).getNumericCellValue()+"\t\t");
               System.out.print(row.getCell(6).getNumericCellValue()+"\t\t");
               System.out.print(row.getCell(7).getNumericCellValue()+"\t\t");
               System.out.print(row.getCell(8).getStringCellValue()+"\t\t");
               System.out.println(row.getCell(9).getStringCellValue());
           }
       }
   
   
   
       /*
          需求： 读取07版本所有行内容， 如果读取07版本的时候使用的实现类
        */
       @Test
       public void test03() throws Exception {
           //1. 获取文件的输入流对象
           FileInputStream fileInputStream = new FileInputStream("C:\\Users\\ztl\\Desktop\\day62【saas_export_day10】\\04.工具与资料\\01 excel资源模板\\上传货物模板.xlsx");
           //2. 使用文件的输入流创建一个工作簿 03的excel对应实现类:XSSFWorkbook
           Workbook workbook = new XSSFWorkbook(fileInputStream);
   
           //3. 通过工作薄得到工作单
           Sheet sheet = workbook.getSheetAt(0);
   
           //4. 得到行
           Row row = sheet.getRow(0);
   
           //5.获取单元格的内容
           System.out.print(row.getCell(1).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(2).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(3).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(4).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(5).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(6).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(7).getStringCellValue()+"\t\t");
           System.out.print(row.getCell(8).getStringCellValue()+"\t\t");
           System.out.println(row.getCell(9).getStringCellValue());
   
   
           //得到单元格的行数
           int rows = sheet.getPhysicalNumberOfRows();
   
           for (int i = 1; i < rows; i++) {
               row = sheet.getRow(i);
               //获取内容
               System.out.print(row.getCell(1).getStringCellValue()+"\t\t");
               System.out.print(row.getCell(2).getStringCellValue()+"\t\t");
               System.out.print(row.getCell(3).getNumericCellValue()+"\t\t");
               System.out.print(row.getCell(4).getStringCellValue()+"\t\t");
               System.out.print(row.getCell(5).getNumericCellValue()+"\t\t");
               System.out.print(row.getCell(6).getNumericCellValue()+"\t\t");
               System.out.print(row.getCell(7).getNumericCellValue()+"\t\t");
               System.out.print(row.getCell(8).getStringCellValue()+"\t\t");
               System.out.println(row.getCell(9).getStringCellValue());
           }
       }
   
   
       /*
        需求： 创建07版本的excel
      */
       @Test
       public void test04() throws Exception {
           //1.创建一个工作薄
           Workbook workbook = new XSSFWorkbook();
           //2.通过工作薄创建工作单
           Sheet sheet = workbook.createSheet("学生信息表");
           //3. 创建行
           Row row = sheet.createRow(0);
           //4. 创建单元格
           row.createCell(1).setCellValue("姓名");
           row.createCell(2).setCellValue("年龄");
           row.createCell(3).setCellValue("身高");
           row.createCell(4).setCellValue("体重");
           //5. 把工作薄输出到指定文件中
           FileOutputStream fileOutputStream = new FileOutputStream("H:/a.xlsx");
           workbook.write(fileOutputStream);
           fileOutputStream.close();
       }
   
   
   }
   
   ```



#### 小结

> Api 说明

| API      | **说明**                                                     |
| -------- | ------------------------------------------------------------ |
| Workbook | 工作薄，Excel的文档对象,针对不同的Excel类型分为：<br />HSSFWorkbook（2003）和XSSFWorkbook（2007） |
| Sheet    | Excel的工作单                                                |
| Row      | Excel的行                                                    |
| Cell     | Excel的格子单元                                              |



### 3. ApachePOI实现货物上传(1)进入上传页

#### 需求

在购销合同列表，点击<font color=red>上传货物</font>，进入上传页：

![1557247440868](SaaS-Export-10.assets/1557247440868.png)

点击后：

!![1585796904657](SaaS-Export-10.assets/1585796904657.png)

#### 步骤

1. 检查页面提交地址
2. 控制器编写方法，实现转发

#### 实现

1. 检查页面提交地址

   ![1557247640462](SaaS-Export-10.assets/1557247640462.png)

2. 控制器编写方法，实现转发

   ![1557247674394](SaaS-Export-10.assets/1557247674394.png)



```java
package cn.itcast.web.controller.cargo;


import cn.itcast.domain.cargo.ContractProduct;
import cn.itcast.domain.cargo.ContractProductExample;
import cn.itcast.domain.cargo.Factory;
import cn.itcast.domain.cargo.FactoryExample;
import cn.itcast.service.cargo.ContractProductService;
import cn.itcast.service.cargo.FactoryService;
import cn.itcast.web.controller.BaseController;
import cn.itcast.web.utils.FileUploadUtil;
import com.alibaba.dubbo.config.annotation.Reference;
import com.github.pagehelper.PageInfo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.MultipartFile;

import java.util.List;

@Controller
@RequestMapping("/cargo/contractProduct")
public class ContractProductController extends BaseController {

    @Reference
    private ContractProductService contractProductService;

    @Reference
    private FactoryService factoryService;

   
 
    /*
        作用 ：进入上传货物的页面
        url :/cargo/contractProduct/toImport.do?contractId=902567ff-1ba5-4258-bccb-9e18586569e4
       参数 : 购销合同的id
       返回值 : 进入上传货物的页面
    */
    @RequestMapping("/toImport")
    public String toImport(String contractId){
        request.setAttribute("contractId",contractId);
        return "cargo/product/product-import";
    }






}

```





### 4. ApachePOI实现货物上传(2)上传货物-Controller

#### 需求

上传货物页面，选择本地文件，进行上传：

![1557247795183](SaaS-Export-10.assets/1557247795183.png)

#### 步骤

1. 后台项目export_web_manager添加poi依赖
2. springmvc.xml配置文件上传解析器(刚刚做图片上传的时候已经完成)
3. 检查页面<font color=red>保存</font>按钮提交地址
4. 后台控制器实现excel文件上传、解析上传的excel

#### 实现

1. 后台项目export_web_manager添加poi依赖

   图1：

   ![1557248432816](SaaS-Export-10.assets/1557248432816.png)

   图2：

   ```xml
   <!--poi坐标-->
   <dependency>
     <groupId>org.apache.poi</groupId>
     <artifactId>poi</artifactId>
     <version>4.0.1</version>
   </dependency>
   <dependency>
     <groupId>org.apache.poi</groupId>
     <artifactId>poi-ooxml</artifactId>
     <version>4.0.1</version>
   </dependency>
   <dependency>
     <groupId>org.apache.poi</groupId>
     <artifactId>poi-ooxml-schemas</artifactId>
     <version>4.0.1</version>
   </dependency>
   ```

2. springmvc.xml配置文件上传解析器==（已经完成了,做七牛云的时候就已经完成）==

   图1：

   ![1557248477518](SaaS-Export-10.assets/1557248477518.png)

   图2：

   ```xml
   <!-- 文件上传解析器-->
   <bean id="multipartResolver"
         class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
       <!-- 设置上传文件的最大尺寸为 5MB -->
       <property name="maxUploadSize" value="5242880"/>
   </bean>
   ```

3. 检查页面<font color=red>保存</font>按钮提交地址

   ![1557248550410](SaaS-Export-10.assets/1557248550410.png)

4. 后台控制器实现excel文件上传、解析上传的excel

   第一步：控制器代码完整实现<font color=red>（重点查看importExcel上传excel方法）</font>

   ```java
      /*
           作用 ：保存上传货物数据
           url :/cargo/contractProduct/import.do
          参数 : 购销合同的id,excel文件
          返回值 : 货物列表
       */
       @RequestMapping("/import")
       public String importExcel(String contractId,MultipartFile file) throws IOException {
           //1.创建工作薄，读取上传文件
           Workbook workbook = new XSSFWorkbook(file.getInputStream());
           //2. 得到工作单
           Sheet sheet = workbook.getSheetAt(0);
   
           //3. 得到文件所有行数
           int rows = sheet.getPhysicalNumberOfRows();
   
           //4. 遍历所有行，读取excel内容, 每一行数据对应ContractProduct.
           for (int index = 1; index < rows; index++) {
               Row row = sheet.getRow(index);
               //每一行对应一个ContractProduct
               ContractProduct contractProduct = new ContractProduct();
   
               //生成厂家
               if( row.getCell(1)!=null){
                   String factoryName = row.getCell(1).getStringCellValue();
                   contractProduct.setFactoryName(factoryName);
               }
   
               //货号
               if( row.getCell(2)!=null){
                   String productNo = row.getCell(2).getStringCellValue();
                   contractProduct.setProductNo(productNo);
               }
   
   
   
               //数量
               if( row.getCell(3)!=null){
                   double cnumber = row.getCell(3).getNumericCellValue();
                   contractProduct.setCnumber((int) cnumber);
               }
   
   
               //包装单位
               if( row.getCell(4)!=null){
                   String packingUnit = row.getCell(4).getStringCellValue();
                   contractProduct.setPackingUnit(packingUnit);
               }
   
   
               //装率
               if( row.getCell(5)!=null){
                   double  loadingRate= row.getCell(5).getNumericCellValue();
                   contractProduct.setLoadingRate(loadingRate+"");
               }
   
   
               //箱数
               if( row.getCell(6)!=null){
                   double boxNum = row.getCell(6).getNumericCellValue();
                   contractProduct.setBoxNum((int) boxNum);
               }
   
   
   
               //单价
               if( row.getCell(7)!=null){
                   double price = row.getCell(7).getNumericCellValue();
                   contractProduct.setPrice(price);
               }
   
   
               //货物描述
               if( row.getCell(8)!=null){
                   String productDesc = row.getCell(8).getStringCellValue();
                   contractProduct.setProductDesc(productDesc);
               }
   
               //要求
               if( row.getCell(9)!=null){
                   String request = row.getCell(9).getStringCellValue();
                   contractProduct.setProductRequest(request);
               }
   
               //补充数据
               //货物所属的购销合同
               contractProduct.setContractId(contractId);
               //该货物所属的生产厂家的id
               Factory factory = factoryService.findByFactoryName(contractProduct.getFactoryName());
               contractProduct.setFactoryId(factory.getId());
   
               //保存数据
               contractProductService.save(contractProduct);
   
           }
           //返回货物列表页面
           return "redirect:/cargo/contractProduct/list.do?contractId="+contractId;
       }
   ```

   

   

   

### 5. ApacheAPO实现货物上传(3)上传货物-Service及Dao



第二步：货运管理服务中， 工厂的接口与实现添加根据工厂名称获取id的方法

<font color=red>FactoryService：</font>

![1568856310002](SaaS-Export-10.assets/1568856310002.png)



<font color=red>FactoryServiceImpl：</font>



```java
package cn.itcast.service.cargo;

import cn.itcast.domain.cargo.Factory;
import cn.itcast.domain.cargo.FactoryExample;
import com.github.pagehelper.PageInfo;

import java.util.List;

/**
 * 工厂模块
 */
public interface FactoryService {

  
 //根据厂家名字查找厂家
    Factory findByFactoryName(String factoryName);
}

```

FactoryServiceImpl实现类

```java
package cn.itcast.service.cargo.impl;

import cn.itcast.dao.cargo.FactoryDao;
import cn.itcast.domain.cargo.Factory;
import cn.itcast.domain.cargo.FactoryExample;
import cn.itcast.service.cargo.FactoryService;
import com.alibaba.dubbo.config.annotation.Service;
import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import org.springframework.beans.factory.annotation.Autowired;

import java.util.Date;
import java.util.List;
import java.util.UUID;

@Service
public class FactoryServiceImpl implements FactoryService {

    @Autowired
    private FactoryDao factoryDao;

   

    //根据厂家名字查找厂家
    @Override
    public Factory findByFactoryName(String factoryName) {
        FactoryExample factoryExample = new FactoryExample();
        factoryExample.createCriteria().andFactoryNameEqualTo(factoryName);
        List<Factory> factoryList = factoryDao.selectByExample(factoryExample);
        //只有去到一个厂家
        Factory factory = factoryList.get(0);
        return factory;
    }
}

```



### 5. 附件管理（一）列表

#### 需求

```reStructuredText
购销合同
   total_amont  合同的总价格 = 货物的价格+附件的价格
   pro_num   货物的数量
   ext_num   附件的数量
货物
  amount  货物的总价
  price   货物的单价
  cnumber  货物的数量
  contractId  所属的购销合同
  
附件
  amount  附件的总价
  price   附件的单价
  cnumber 附件的数量
  contractId  所属的购销合同
  contract_product_id  所属的货物
  
增加附件
   1. 计算出附件的总价
   2. 插入附件数据 
   3.修改购销合同的总价
   4. 修改购销合同的附件种类数量
   5. 更新购销合同
   
修改附件
   1. 计算附件的总价
   2. 更新附件
   3. 重新计算购销合同总价
   4. 更新购销合同
 
删除附件
   1. 根据id找附件
   2. 删除附件
   3. 更新购销合同总价
   4. 更新购销合同种类
   5. 更新购销合同


```



图1：购销了合同列表点击<font color=red>货物</font>

![1557322806618](SaaS-Export-10.assets/1557322806618.png)

图2：进入货物列表，再点击附件

![1557322854790](SaaS-Export-10.assets/1557322854790.png)

图3：进入附件列表和添加附件页面

![1578708818983](SaaS-Export-10.assets/1578708818983.png)

#### 步骤

1. 实现附件的服务类实现类，ExtCProductServiceImpl
2. 编写控制器：ExtCproductController
3. 附件列表显示

#### 实现

1. 实现附件的服务类实现类，ExtCProductServiceImpl

   ```java
   package cn.itcast.service.cargo.impl;
   
   import cn.itcast.dao.cargo.ExtCproductDao;
   import cn.itcast.domain.cargo.ExtCproduct;
   import cn.itcast.domain.cargo.ExtCproductExample;
   import cn.itcsat.service.cargo.ExtCproductService;
   import com.alibaba.dubbo.config.annotation.Service;
   import com.github.pagehelper.PageHelper;
   import com.github.pagehelper.PageInfo;
   import org.springframework.beans.factory.annotation.Autowired;
   
   import java.util.Date;
   import java.util.List;
   import java.util.UUID;
   
   @Service
   public class ExtCproductServiceImpl implements ExtCproductService {
   
       @Autowired
       private ExtCproductDao extCproductDao;
   
       /*
           附件分页查询
        */
       @Override
       public PageInfo<ExtCproduct> findByPage(ExtCproductExample extCproductExample, int pageNum, int pageSize) {
           PageHelper.startPage(pageNum,pageSize);
           //查询数据
           List<ExtCproduct> extCproductList = extCproductDao.selectByExample(extCproductExample);
           //得到附件页面
           PageInfo<ExtCproduct> pageInfo = new PageInfo<>(extCproductList);
           return pageInfo;
       }
   
       /*
          附件条件查询
       */
       @Override
       public List<ExtCproduct> findAll(ExtCproductExample extCproductExample) {
           List<ExtCproduct> extCproductList = extCproductDao.selectByExample(extCproductExample);
           return extCproductList;
       }
   
       /*
           根据id查询附件
       */
       @Override
       public ExtCproduct findById(String id) {
           return extCproductDao.selectByPrimaryKey(id);
       }
   
       /*
         添加附件
     */
       @Override
       public void save(ExtCproduct extCproduct) {
           extCproduct.setId(UUID.randomUUID().toString());
           //有两个字段不能为空
           extCproduct.setCreateTime(new Date());
           extCproduct.setUpdateTime(new Date());
           extCproductDao.insertSelective(extCproduct);
       }
   
       /*
       更新附件
        */
       @Override
       public void update(ExtCproduct extCproduct) {
           //修改更新的时间
           extCproduct.setUpdateTime(new Date());
           extCproductDao.updateByPrimaryKeySelective(extCproduct);
       }
   
       /*
       根据主键删除附件
        */
       @Override
       public void delete(String id) {
           extCproductDao.deleteByPrimaryKey(id);
       }
   
   
   
   
   
   
   
   }
   
   ```

   

2. 编写控制器：ExtCproductController

   ![1557323099378](SaaS-Export-10.assets/1557323099378.png)

   ```java
   package cn.itcast.web.controller.cargo;
   
   
   import cn.itcast.domain.cargo.ExtCproduct;
   import cn.itcast.domain.cargo.ExtCproductExample;
   import cn.itcast.domain.cargo.Factory;
   import cn.itcast.domain.cargo.FactoryExample;
   import cn.itcast.web.controller.BaseController;
   import cn.itcast.web.utils.FileUploadUtil;
   import cn.itcsat.service.cargo.ExtCproductService;
   import cn.itcsat.service.cargo.FactoryService;
   import com.alibaba.dubbo.config.annotation.Reference;
   import com.github.pagehelper.PageInfo;
   import org.apache.poi.ss.usermodel.Row;
   import org.apache.poi.ss.usermodel.Sheet;
   import org.apache.poi.ss.usermodel.Workbook;
   import org.apache.poi.xssf.usermodel.XSSFWorkbook;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Controller;
   import org.springframework.util.StringUtils;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RequestParam;
   import org.springframework.web.multipart.MultipartFile;
   
   import java.io.IOException;
   import java.util.List;
   
   @Controller
   @RequestMapping("/cargo/extCproduct")
   public class ExtCproductController extends BaseController {
   
       @Reference
       private ExtCproductService extCproductService;
   
   
       @Reference
       private FactoryService factoryService;
   
       /*
           作用：进入附件列表页面
           url:/cargo/extCproduct/list.do?contractId=d36401ea-e152-4663-92df-81078f5edb27&contractProductId=be0bd92d-ea0c-41b0-8a1f-ab7bed3d30be
           参数：购销合同id ， contractProductId(货物的id )
           返回值 : 附件列表页面
        */
       @RequestMapping("/list")
       public String list(String contractId,String contractProductId,@RequestParam(defaultValue = "1") Integer pageNum,@RequestParam(defaultValue = "5") Integer pageSize){
           //1. 查询当前货物附件数据,得到pageInfo
           ExtCproductExample extCproductExample = new ExtCproductExample();
           //由于我们只需要查询当前货物的附件
           extCproductExample.createCriteria().andContractProductIdEqualTo(contractProductId);
           PageInfo<ExtCproduct> pageInfo = extCproductService.findByPage(extCproductExample, pageNum, pageSize);
           request.setAttribute("pageInfo",pageInfo);
   
           //2. 查询生成附件的厂家
           FactoryExample factoryExample = new FactoryExample();
           factoryExample.createCriteria().andCtypeEqualTo("附件");
           List<Factory> factoryList = factoryService.findAll(factoryExample);
           request.setAttribute("factoryList",factoryList);
   
   
           //3. 把购销合同id存储到域中
           request.setAttribute("contractId",contractId);
           request.setAttribute("contractProductId",contractProductId);
   
           //请求转到到页面
           return "cargo/extc/extc-list";
       }
   
   
   }
   
   ```

3. 测试 附件列表显示

![1568858760778](SaaS-Export-10.assets/1568858760778.png)





### 6. 附件管理（二）添加

#### 需求

====附件添加=====

1.计算附件总价（单价*数量）

2.插入一条附件表记录

3.计算新合同的总结（原合同总结+附件总价）

4.计算合同的附件数量（原合同的附件数+1）

5.更新合同表的数据



在附件列表页面，添加<font color=red>保存</font>，添加附件：

![1557323157662](SaaS-Export-10.assets/1557323157662.png)

#### 步骤

1. ExtCproductController 处理添加附件请求

2. ExtCproductServiceImpl实现添加附件的方法


#### 实现

1. <font color=red>ExtCproductController处理添加附件的操作方法:edit()</font>

   ```java
   package cn.itcast.web.controller.cargo;
   
   
   import cn.itcast.domain.cargo.ExtCproduct;
   import cn.itcast.domain.cargo.ExtCproductExample;
   import cn.itcast.domain.cargo.Factory;
   import cn.itcast.domain.cargo.FactoryExample;
   import cn.itcast.service.cargo.ExtCproductService;
   import cn.itcast.service.cargo.FactoryService;
   import cn.itcast.web.controller.BaseController;
   import cn.itcast.web.utils.FileUploadUtil;
   import com.alibaba.dubbo.config.annotation.Reference;
   import com.github.pagehelper.PageInfo;
   import org.apache.poi.ss.usermodel.Row;
   import org.apache.poi.ss.usermodel.Sheet;
   import org.apache.poi.ss.usermodel.Workbook;
   import org.apache.poi.xssf.usermodel.XSSFWorkbook;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Controller;
   import org.springframework.util.StringUtils;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RequestParam;
   import org.springframework.web.multipart.MultipartFile;
   
   import java.io.InputStream;
   import java.util.List;
   
   @Controller
   @RequestMapping("/cargo/extCproduct")
   public class ExtCproductController extends BaseController {
   
       @Reference
       private ExtCproductService extCproductService;
   
       @Reference
       private FactoryService factoryService;
   
    /*
       作用：添加附件，更新附件
       url:  /extCproduct/edit.do
       参数：ExtCproduct 附件对象信息
       返回值 : 附件列表页面
    */
       @RequestMapping("/edit")
       public String edit(ExtCproduct extCproduct, MultipartFile productPhoto) throws Exception {
           //附件的创建人
           extCproduct.setCreateBy(getLoginUser().getId());
           //附件的创建人所属的部门
           extCproduct.setCreateDept(getLoginUser().getDeptId());
           //附件的创建人所属的企业id
           extCproduct.setCompanyId(getLoginUserCompanyId());
           //附件的创建人所属的企业名称
           extCproduct.setCompanyName(getLoginUserCompanyName());
   
           if(productPhoto!=null){
               //存储七牛中
               String url = fileUploadUtil.upload(productPhoto); //返回图片上传成功的url
               //把url路径存储到对象
               extCproduct.setProductImage("http://"+url);
           }
   
   
           if(StringUtils.isEmpty(extCproduct.getId())){  //判断一个变量是否为空串,相当于上面的语句
              //添加
              extCproductService.save(extCproduct);
          }else{
              //更新
              extCproductService.update(extCproduct);
          }
          return "redirect:/cargo/extCproduct/list.do?contractId="+extCproduct.getContractId()+"&contractProductId="+extCproduct.getContractProductId();  //访问上面的list方法
       }
   
   
   
   }
   
   ```

2. <font color=red>ExtCproductServiceImpl实现添加附件的方法： save()</font>

   ```java
   package cn.itcast.service.cargo.impl;
   
   import cn.itcast.dao.cargo.ContractDao;
   import cn.itcast.dao.cargo.ExtCproductDao;
   import cn.itcast.domain.cargo.Contract;
   import cn.itcast.domain.cargo.ExtCproduct;
   import cn.itcast.domain.cargo.ExtCproductExample;
   import cn.itcast.service.cargo.ExtCproductService;
   import com.alibaba.dubbo.config.annotation.Service;
   import com.github.pagehelper.PageHelper;
   import com.github.pagehelper.PageInfo;
   import org.springframework.beans.factory.annotation.Autowired;
   
   import java.util.Date;
   import java.util.List;
   import java.util.UUID;
   
   @Service
   public class ExtCproductServiceImpl implements ExtCproductService {
   
       @Autowired
       private ExtCproductDao extCproductDao;
   
       @Autowired
       private ContractDao contractDao;
   
    
       /*
         添加附件
     */
       @Override
       public void save(ExtCproduct extCproduct) {
           extCproduct.setId(UUID.randomUUID().toString());
           //有两个字段不能为空
           extCproduct.setCreateTime(new Date());
           extCproduct.setUpdateTime(new Date());
   
   //        1. 计算出附件的总价
           double amount=0;
           if(extCproduct.getPrice()!=null && extCproduct.getCnumber()!=null){
               amount = extCproduct.getCnumber()*extCproduct.getPrice();
               extCproduct.setAmount(amount);
           }
   //        2. 插入附件数据
           extCproductDao.insertSelective(extCproduct);
   //        3.修改购销合同的总价   购销合同总价 = 购销合同原价格+附件的总价
           //找到购销合同
           Contract contract = contractDao.selectByPrimaryKey(extCproduct.getContractId());
           contract.setTotalAmount(contract.getTotalAmount()+amount);
   //        4. 修改购销合同的附件种类数量
           if(contract.getExtNum()!=null){
               contract.setExtNum(contract.getExtNum()+1);
           }else{
               //第一次添加附件
               contract.setExtNum(1);
           }
   //        5. 更新购销合同
           contractDao.updateByPrimaryKeySelective(contract);
       }
   }
   
   ```



==易错点：展示列表的时候（list.do）一定需要回传contractProductId与contractId==



### 7. 附件管理（三）修改

#### 需求

在附件的列表与添加页面，点击<font color=red>修改</font>：

![1557323838678](SaaS-Export-10.assets/1557323838678.png)

进入修改页面：

![1578711085419](SaaS-Export-10.assets/1578711085419.png)

在修改页面，点击保存，修改附件信息。

#### 步骤

> 进入附件修改页面

* 1. ExtCproductController控制器添加toUpdate()方法, 实现主键查询附件、生产厂家信息
  2. 测试

> 修改保存附件

* 1. ExtCproductService实现update()方法
  2. 测试

#### 实现

> 进入附件修改页面

- 1. ExtCproductController控制器添加toUpdate()方法, 实现主键查询附件、生产厂家信息

     ```java
     package cn.itcast.web.controller.cargo;
     
     
     import cn.itcast.domain.cargo.ExtCproduct;
     import cn.itcast.domain.cargo.ExtCproductExample;
     import cn.itcast.domain.cargo.Factory;
     import cn.itcast.domain.cargo.FactoryExample;
     import cn.itcast.service.cargo.ExtCproductService;
     import cn.itcast.service.cargo.FactoryService;
     import cn.itcast.web.controller.BaseController;
     import cn.itcast.web.utils.FileUploadUtil;
     import com.alibaba.dubbo.config.annotation.Reference;
     import com.github.pagehelper.PageInfo;
     import org.apache.poi.ss.usermodel.Row;
     import org.apache.poi.ss.usermodel.Sheet;
     import org.apache.poi.ss.usermodel.Workbook;
     import org.apache.poi.xssf.usermodel.XSSFWorkbook;
     import org.springframework.beans.factory.annotation.Autowired;
     import org.springframework.stereotype.Controller;
     import org.springframework.util.StringUtils;
     import org.springframework.web.bind.annotation.RequestMapping;
     import org.springframework.web.bind.annotation.RequestParam;
     import org.springframework.web.multipart.MultipartFile;
     
     import java.io.InputStream;
     import java.util.List;
     
     @Controller
     @RequestMapping("/cargo/extCproduct")
     public class ExtCproductController extends BaseController {
     
         @Reference
         private ExtCproductService extCproductService;
     
         @Reference
         private FactoryService factoryService;
     
     
       /*
              作用 ： 进入更新附件的页面
              url : /cargo/extCproduct/toUpdate.do?id=${o.id}&contractId=${contractId}&contractProductId=${o.contractProductId}
             参数 : 附件id，contractId（购销合同id）, 货物的id(货物的id)
             返回值 : extc-update页面
          */
         @RequestMapping("/toUpdate")
         public String toUpdate(String id,String  contractId,String contractProductId){
     
             //1. 根据附件的id查找附件
             ExtCproduct extCproduct =  extCproductService.findById(id);
             //2. 存储到request域中
             request.setAttribute("extCproduct",extCproduct);
     
     
             //3. 生成附件的厂家
             FactoryExample factoryExample = new FactoryExample();
             factoryExample.createCriteria().andCtypeEqualTo("附件");
             List<Factory> factoryList = factoryService.findAll(factoryExample);
             request.setAttribute("factoryList",factoryList);
     
             //4. 合同的id
             request.setAttribute("contractId",contractId);
             //5. 货物的id
             request.setAttribute("contractProductId",contractProductId);
     
             //4. 请求转发到product-update页面
             return "cargo/extc/extc-update";
         }
     
     
        
     }
     
     ```

  2. 测试

> 修改保存附件



====修改附件====

1.计算附件总价（单价*数量）

2.更新附件表一条记录

3.计算合同的总价（原合同的总价-原附件总价+新附件总价）

4.更新合同表的记录



- 1. ExtCproductServiceImpl实现update()方法

     ```java
     package cn.itcast.service.cargo.impl;
     
     import cn.itcast.dao.cargo.ContractDao;
     import cn.itcast.dao.cargo.ExtCproductDao;
     import cn.itcast.domain.cargo.Contract;
     import cn.itcast.domain.cargo.ExtCproduct;
     import cn.itcast.domain.cargo.ExtCproductExample;
     import cn.itcast.service.cargo.ExtCproductService;
     import com.alibaba.dubbo.config.annotation.Service;
     import com.github.pagehelper.PageHelper;
     import com.github.pagehelper.PageInfo;
     import org.springframework.beans.factory.annotation.Autowired;
     
     import java.util.Date;
     import java.util.List;
     import java.util.UUID;
     
     @Service
     public class ExtCproductServiceImpl implements ExtCproductService {
     
         @Autowired
         private ExtCproductDao extCproductDao;
     
         @Autowired
         private ContractDao contractDao;
     
        
          /*
         更新附件
          */
         @Override
         public void update(ExtCproduct extCproduct) {
             ExtCproduct oldExtCproduct = extCproductDao.selectByPrimaryKey(extCproduct.getId());
             //修改更新的时间
             extCproduct.setUpdateTime(new Date());
     //        1. 计算附件的总价
             double amount=0;
             if(extCproduct.getPrice()!=null && extCproduct.getCnumber()!=null){
                 amount = extCproduct.getCnumber()*extCproduct.getPrice();
                 extCproduct.setAmount(amount);
             }
     //        2. 更新附件
             extCproductDao.updateByPrimaryKeySelective(extCproduct);
     //        3. 重新计算购销合同总价   购销合同总价 = 购销合同原总价-原附件总价+新总价
             Contract contract = contractDao.selectByPrimaryKey(extCproduct.getContractId());
             contract.setTotalAmount(contract.getTotalAmount()-oldExtCproduct.getAmount()+amount);
     //        4. 更新购销合同
             contractDao.updateByPrimaryKeySelective(contract);
         }
     
     
     
     }
     
     ```

     



### 8. 附件管理（四）删除

#### 需求

==  删除附件 ===

1.删除附件表的一条记录

2.计算合同的总价（原合同的总价-附件总价）

3.计算合同的附件数量（原附件数量-1）

4.更新合同表的记录





附件列表与添加页面，点击<font color=red>删除</font>：

![1557324330953](SaaS-Export-10.assets/1557324330953.png)

#### 步骤

1. ExtCproductController控制器添加delete()方法,
2. ExtCproductService实现delete()方法

#### 实现

1. ExtCproductController控制器添加delete()方法

   ```java
   package cn.itcast.web.controller.cargo;
   
   
   import cn.itcast.domain.cargo.ExtCproduct;
   import cn.itcast.domain.cargo.ExtCproductExample;
   import cn.itcast.domain.cargo.Factory;
   import cn.itcast.domain.cargo.FactoryExample;
   import cn.itcast.service.cargo.ExtCproductService;
   import cn.itcast.service.cargo.FactoryService;
   import cn.itcast.web.controller.BaseController;
   import cn.itcast.web.utils.FileUploadUtil;
   import com.alibaba.dubbo.config.annotation.Reference;
   import com.github.pagehelper.PageInfo;
   import org.apache.poi.ss.usermodel.Row;
   import org.apache.poi.ss.usermodel.Sheet;
   import org.apache.poi.ss.usermodel.Workbook;
   import org.apache.poi.xssf.usermodel.XSSFWorkbook;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Controller;
   import org.springframework.util.StringUtils;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RequestParam;
   import org.springframework.web.multipart.MultipartFile;
   
   import java.io.InputStream;
   import java.util.List;
   
   @Controller
   @RequestMapping("/cargo/extCproduct")
   public class ExtCproductController extends BaseController {
   
       @Reference
       private ExtCproductService extCproductService;
   
       @Reference
       private FactoryService factoryService;
      /*
            作用 ：删除附件
            url :/cargo/extCproduct/delete.do?id=${o.id}&contractId=${contractId}&contractProductId=${o.contractProductId}
           参数 : 附件的id，购销合同的id,货物的id
           返回值 : 附件列表
        */
       @RequestMapping("/delete")
       public String delete(String id,String  contractId,String contractProductId){
           extCproductService.delete(id);
           return "redirect:/cargo/extCproduct/list.do?contractId="+contractId+"&contractProductId="+contractProductId;
       }
   
   
   
   
   
   
   }
   
   ```

2. ExtCproductServiceImpl实现delete()方法


```java
package cn.itcast.service.cargo.impl;

import cn.itcast.dao.cargo.ContractDao;
import cn.itcast.dao.cargo.ExtCproductDao;
import cn.itcast.domain.cargo.Contract;
import cn.itcast.domain.cargo.ExtCproduct;
import cn.itcast.domain.cargo.ExtCproductExample;
import cn.itcast.service.cargo.ExtCproductService;
import com.alibaba.dubbo.config.annotation.Service;
import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import org.springframework.beans.factory.annotation.Autowired;

import java.util.Date;
import java.util.List;
import java.util.UUID;

@Service
public class ExtCproductServiceImpl implements ExtCproductService {

    @Autowired
    private ExtCproductDao extCproductDao;

    @Autowired
    private ContractDao contractDao;

    /*
    根据主键删除附件
     */
    @Override
    public void delete(String id) {
//        1. 根据id找附件
        ExtCproduct dbExtCproduct = extCproductDao.selectByPrimaryKey(id); //从数据库中读取出来存放内存
//        2. 删除附件
        extCproductDao.deleteByPrimaryKey(id); //删除数据库中
//        3. 更新购销合同总价
        //找到购销合同
        Contract contract = contractDao.selectByPrimaryKey(dbExtCproduct.getContractId());
        contract.setTotalAmount(contract.getTotalAmount()-dbExtCproduct.getAmount());
//        4. 更新购销合同种类
        contract.setExtNum(contract.getExtNum()-1);
//        5. 更新购销合同
        contractDao.updateByPrimaryKeySelective(contract);
    }
}

```







### 9. 出货表导出：需求分析

> 需求：按照船期导出每月出货数据：

![1557332042503](SaaS-Export-10.assets/1557332042503.png)

> 什么是船期？

在购销合同表中，有一个ship_time字段，就是船期。表示交货日期

![1557332161440](SaaS-Export-10.assets/1557332161440.png)

> 出货表导出的数据，如下：

![1557332261501](SaaS-Export-10.assets/1557332261501.png)

> 数据来源分析

根据excel报表展示效果得知数据来源于：购销合同和货物。查询SQL如下：

```sql
-- date_format函数的用法： 
SELECT * FROM co_contract WHERE DATE_FORMAT(ship_time,'%Y-%m') = '2015-01';

-- 根据船期查询货物的数据
SELECT c.`custom_name` customName,c.`contract_no` contractNo,p.`product_no` productNo,
p.`cnumber` cnumber,p.`factory_name` factoryName,c.`delivery_period` deliveryPeriod,
c.`ship_time` shipTime,c.`trade_terms` tradeTerms
FROM co_contract c INNER JOIN co_contract_product p ON c.`id` = p.`contract_id`
WHERE DATE_FORMAT(c.`ship_time`,'%Y-%m')='2015-01' AND c.`company_id`='1';
```



### 10. 出货表导出：数据准备

#### 分析

> 目前导出的SQL有了，查询结果已经包含如下数据：

![1557332899425](SaaS-Export-10.assets/1557332899425.png)

> 但是，现在没有实体类与列表数据对应，怎么办？

> 所以，现在定义封装值对象的VO对象（Value Object）

```java
package cn.itcast.vo;

import java.io.Serializable;
import java.util.Date;

public class ContractProductVo implements Serializable {

    private String customName;		//客户名称
    private String contractNo;		//合同号，订单号
    private String productNo;		//货号
    private Integer cnumber;		//数量
    private String factoryName;		//厂家名称，冗余字段
    private Date deliveryPeriod;	//交货期限
    private Date shipTime;			//船期
    private String tradeTerms;		//贸易条款


    public ContractProductVo() {
    }

    public ContractProductVo(String customName, String contractNo, String productNo, Integer cnumber, String factoryName, Date deliveryPeriod, Date shipTime, String tradeTerms) {
        this.customName = customName;
        this.contractNo = contractNo;
        this.productNo = productNo;
        this.cnumber = cnumber;
        this.factoryName = factoryName;
        this.deliveryPeriod = deliveryPeriod;
        this.shipTime = shipTime;
        this.tradeTerms = tradeTerms;
    }

    /**
     * 获取
     * @return customName
     */
    public String getCustomName() {
        return customName;
    }

    /**
     * 设置
     * @param customName
     */
    public void setCustomName(String customName) {
        this.customName = customName;
    }

    /**
     * 获取
     * @return contractNo
     */
    public String getContractNo() {
        return contractNo;
    }

    /**
     * 设置
     * @param contractNo
     */
    public void setContractNo(String contractNo) {
        this.contractNo = contractNo;
    }

    /**
     * 获取
     * @return productNo
     */
    public String getProductNo() {
        return productNo;
    }

    /**
     * 设置
     * @param productNo
     */
    public void setProductNo(String productNo) {
        this.productNo = productNo;
    }

    /**
     * 获取
     * @return cnumber
     */
    public Integer getCnumber() {
        return cnumber;
    }

    /**
     * 设置
     * @param cnumber
     */
    public void setCnumber(Integer cnumber) {
        this.cnumber = cnumber;
    }

    /**
     * 获取
     * @return factoryName
     */
    public String getFactoryName() {
        return factoryName;
    }

    /**
     * 设置
     * @param factoryName
     */
    public void setFactoryName(String factoryName) {
        this.factoryName = factoryName;
    }

    /**
     * 获取
     * @return deliveryPeriod
     */
    public Date getDeliveryPeriod() {
        return deliveryPeriod;
    }

    /**
     * 设置
     * @param deliveryPeriod
     */
    public void setDeliveryPeriod(Date deliveryPeriod) {
        this.deliveryPeriod = deliveryPeriod;
    }

    /**
     * 获取
     * @return shipTime
     */
    public Date getShipTime() {
        return shipTime;
    }

    /**
     * 设置
     * @param shipTime
     */
    public void setShipTime(Date shipTime) {
        this.shipTime = shipTime;
    }

    /**
     * 获取
     * @return tradeTerms
     */
    public String getTradeTerms() {
        return tradeTerms;
    }

    /**
     * 设置
     * @param tradeTerms
     */
    public void setTradeTerms(String tradeTerms) {
        this.tradeTerms = tradeTerms;
    }

    public String toString() {
        return "ContractPrudctVo{customName = " + customName + ", contractNo = " + contractNo + ", productNo = " + productNo + ", cnumber = " + cnumber + ", factoryName = " + factoryName + ", deliveryPeriod = " + deliveryPeriod + ", shipTime = " + shipTime + ", tradeTerms = " + tradeTerms + "}";
    }
}

```

> 这样，查询的数据就可以封装到ContractProductVo对象中了。

#### 步骤

1. 定义ContractProductVo对象，封装Excel列表数据
2. 编写dao，实现查询excel列表数据
3. 编写service，准备好excel所需要的数据

#### 实现

1. <font color=red>定义ContractProductVo对象，封装Excel列表数据</font>

   图1：

   &nbsp;![1557333191975](SaaS-Export-10.assets/1557333191975.png)

   图2：

   ```java
   package cn.itcast.vo;
   
   import java.io.Serializable;
   import java.util.Date;
   
   public class ContractProductVo implements Serializable {
   
       private String customName;		//客户名称
       private String contractNo;		//合同号，订单号
       private String productNo;		//货号
       private Integer cnumber;		//数量
       private String factoryName;		//厂家名称，冗余字段
       private Date deliveryPeriod;	//交货期限
       private Date shipTime;			//船期
       private String tradeTerms;		//贸易条款
   
   
       public ContractProductVo() {
       }
   
       public ContractProductVo(String customName, String contractNo, String productNo, Integer cnumber, String factoryName, Date deliveryPeriod, Date shipTime, String tradeTerms) {
           this.customName = customName;
           this.contractNo = contractNo;
           this.productNo = productNo;
           this.cnumber = cnumber;
           this.factoryName = factoryName;
           this.deliveryPeriod = deliveryPeriod;
           this.shipTime = shipTime;
           this.tradeTerms = tradeTerms;
       }
   
       /**
        * 获取
        * @return customName
        */
       public String getCustomName() {
           return customName;
       }
   
       /**
        * 设置
        * @param customName
        */
       public void setCustomName(String customName) {
           this.customName = customName;
       }
   
       /**
        * 获取
        * @return contractNo
        */
       public String getContractNo() {
           return contractNo;
       }
   
       /**
        * 设置
        * @param contractNo
        */
       public void setContractNo(String contractNo) {
           this.contractNo = contractNo;
       }
   
       /**
        * 获取
        * @return productNo
        */
       public String getProductNo() {
           return productNo;
       }
   
       /**
        * 设置
        * @param productNo
        */
       public void setProductNo(String productNo) {
           this.productNo = productNo;
       }
   
       /**
        * 获取
        * @return cnumber
        */
       public Integer getCnumber() {
           return cnumber;
       }
   
       /**
        * 设置
        * @param cnumber
        */
       public void setCnumber(Integer cnumber) {
           this.cnumber = cnumber;
       }
   
       /**
        * 获取
        * @return factoryName
        */
       public String getFactoryName() {
           return factoryName;
       }
   
       /**
        * 设置
        * @param factoryName
        */
       public void setFactoryName(String factoryName) {
           this.factoryName = factoryName;
       }
   
       /**
        * 获取
        * @return deliveryPeriod
        */
       public Date getDeliveryPeriod() {
           return deliveryPeriod;
       }
   
       /**
        * 设置
        * @param deliveryPeriod
        */
       public void setDeliveryPeriod(Date deliveryPeriod) {
           this.deliveryPeriod = deliveryPeriod;
       }
   
       /**
        * 获取
        * @return shipTime
        */
       public Date getShipTime() {
           return shipTime;
       }
   
       /**
        * 设置
        * @param shipTime
        */
       public void setShipTime(Date shipTime) {
           this.shipTime = shipTime;
       }
   
       /**
        * 获取
        * @return tradeTerms
        */
       public String getTradeTerms() {
           return tradeTerms;
       }
   
       /**
        * 设置
        * @param tradeTerms
        */
       public void setTradeTerms(String tradeTerms) {
           this.tradeTerms = tradeTerms;
       }
   
       public String toString() {
           return "ContractPrudctVo{customName = " + customName + ", contractNo = " + contractNo + ", productNo = " + productNo + ", cnumber = " + cnumber + ", factoryName = " + factoryName + ", deliveryPeriod = " + deliveryPeriod + ", shipTime = " + shipTime + ", tradeTerms = " + tradeTerms + "}";
       }
   }
   
   ```

2. <font color=red>编写dao，实现查询excel列表数据</font>

   图1：在货物的ContactProductDao接口

   ```java
   package cn.itcast.dao.cargo;
   
   import cn.itcast.domain.cargo.ContractProduct;
   import cn.itcast.domain.cargo.ContractProductExample;
   import cn.itcast.vo.ContractProductVo;
   import org.apache.ibatis.annotations.Param;
   
   import java.util.List;
   
   public interface ContractProductDao {
   
     //根据船期查找货物
       List<ContractProductVo> findByShipTime(@Param("shipTime") String shipTime,@Param("companyId") String companyId);
   
   }
   ```

   

   ![1557333316797](SaaS-Export-10.assets/1557333316797.png)

   图2：dao接口映射

   <font color=green>ContractProductDao.xml</font>

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   
   
     <!--这里不要使用resutMap，也不准使用别名， 因为别名扫描只是扫描domain，没有扫描vo-->
       <select id="findByShipTime" resultType="cn.itcast.vo.ContractProductVo">
           SELECT c.`custom_name` customName,c.`contract_no` contractNo,p.`product_no` productNo,
                   p.`cnumber` cnumber,p.`factory_name` factoryName,c.`delivery_period` deliveryPeriod,
                   c.`ship_time` shipTime,c.`trade_terms` tradeTerms
           FROM co_contract c INNER JOIN co_contract_product p ON c.`id` = p.`contract_id`
           WHERE DATE_FORMAT(c.`ship_time`,'%Y-%m')=#{shipTime} AND c.`company_id`=#{companyId}
       </select>
   
   </mapper>
   ```

3. 编写ContractPrpoductservice，准备好excel所需要的数据

   接口

   ```java
   package cn.itcast.service.cargo;
   
   import cn.itcast.domain.cargo.ContractProduct;
   import cn.itcast.domain.cargo.ContractProductExample;
   import cn.itcast.vo.ContractProductVo;
   import com.github.pagehelper.PageInfo;
   import org.apache.ibatis.annotations.Param;
   
   import java.util.List;
   
   /**
    * 购销合同货物模块
    */
   public interface ContractProductService {
   
      
      //根据船期查找货物
       List<ContractProductVo> findByShipTime(String companyId, String shipTime);
   }
   
   
   ```

   

   ![1557333593073](SaaS-Export-10.assets/1557333593073.png)

   实现

   ![1557333649115](SaaS-Export-10.assets/1557333649115.png)



```java
package cn.itcast.service.cargo.impl;

import cn.itcast.dao.cargo.ContractDao;
import cn.itcast.dao.cargo.ContractProductDao;
import cn.itcast.dao.cargo.ExtCproductDao;
import cn.itcast.domain.cargo.*;
import cn.itcast.service.cargo.ContractProductService;
import cn.itcast.vo.ContractProductVo;
import com.alibaba.dubbo.config.annotation.Service;
import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import org.springframework.beans.factory.annotation.Autowired;

import java.util.Date;
import java.util.List;
import java.util.UUID;

@Service
public class ContractProductServiceImpl implements ContractProductService {

    @Autowired
    private ContractProductDao contractProductDao;

    @Autowired
    private ContractDao contractDao;

    @Autowired
    private ExtCproductDao extCproductDao;

  

     //出货表的sql
    @Override
    public List<ContractPrudctVo> findByShipTime(String companyId, String shipTime) {

        return contractProductDao.findByShipTime(companyId,shipTime);
    }
}

```





### 11. 出货表导出：完整实现

#### 步骤

1. 进入出货表导出页面
2. 页面点击提交，导出excel

#### 实现

1. 进入出货表打印页面

   图1：

   ![1557333912984](SaaS-Export-10.assets/1557333912984.png)

   图2：

   ```java
   package cn.itcast.web.controller.cargo;
   
   
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.RequestMapping;
   
   @Controller
   @RequestMapping("/cargo/contract")
   public class OutProductController {
   
   
       /*
         url:/cargo/contract/print.do
         作用： 进入出货表的页面
         参数： 无
        */
       @RequestMapping("/print")
       public  String print(){
           return "cargo/print/contract-print";
       }
   
   
   }
   
   ```

2. 导入DownLoadUtils工具类

   ![1606633734040](SaaS-Export-10.assets/1606633734040.png)

3. 页面点击提交，导出excel,

   ```java
     /*
             作用： 下载出货表
             url: /cargo/contract/printExcel.do?inputDate=2015-01
             参数：inputDate 出货日期
             返回值 : 无，因为是下载
   
             注意： 如果一个方法是下载文件的时候，必须使用@ResponseBody
          */
       @RequestMapping("/printExcel")
       @ResponseBody  //返回字符串、 返回json、下载
       public void printExcel(String inputDate) throws IOException {
           //1. 创建工作薄
           Workbook workbook = new XSSFWorkbook();
           //2. 创建工作单
           Sheet sheet = workbook.createSheet("出货表");
   
           //3. 合并单元格
           /*
           CellRangeAddress(int firstRow, int lastRow, int firstCol, int lastCol)
                  firstRow： 开始行
                   lastRow:  结束行
                   firstCol:  开始列
                   lastCol:   结束列
            */
           sheet.addMergedRegion(new CellRangeAddress(0,0,1,8));
           //4. 设置列宽
           sheet.setColumnWidth(0,6*256);
           sheet.setColumnWidth(1,21*256);
           sheet.setColumnWidth(2,16*256);
           sheet.setColumnWidth(3,29*256);
           sheet.setColumnWidth(4,11*256);
           sheet.setColumnWidth(5,11*256);
           sheet.setColumnWidth(6,11*256);
           sheet.setColumnWidth(7,11*256);
           sheet.setColumnWidth(8,11*256);
           sheet.setColumnWidth(9,11*256);
   
   
           //标题 第0行  2015-01  2015年1  2015-11
           String title = inputDate.replaceAll("-0","年").replaceAll("-","年")+"月份出货表";
           Row row = sheet.createRow(0);
           Cell cell = row.createCell(1);
           //设置单元格样式
           cell.setCellStyle(bigTitle(workbook));
           //设置内容
           cell.setCellValue(title);
   
   
           //表头部分
           String[] titles = {"客户","订单号","货号","数量","工厂","工厂交期","船期","贸易条款"};
           row = sheet.createRow(1);
           for (int i = 0; i < titles.length; i++) {
               //创建单元格
               cell = row.createCell(i+1);
               //设置样式
               cell.setCellStyle(title(workbook));
               //设置内容
               cell.setCellValue(titles[i]);
           }
   
           //定义一个变量保存索行
           int index = 2;
   
           SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
   
           //获取到货物的数据
           List<ContractProductVo> contractProductList = contractProductService.findByShipTime(getLoginUserCompanyId(), inputDate);
           if(contractProductList!=null){
               //遍历所有的货物的数据
               for (ContractProductVo contractProductVo : contractProductList) {
                   //每一个货物的数据就是一行
                  row =  sheet.createRow(index++);
   
                  // 客户名称
                   String customerName = contractProductVo.getCustomName();
                   if(customerName!=null){
                      cell =  row.createCell(1);
                       //设置单元格样式
                       cell.setCellStyle(text(workbook));
                       //设置内容
                       cell.setCellValue(customerName);
                   }
   
   
                   // 订单号
                   String contractNo = contractProductVo.getContractNo();
                   if(contractNo!=null){
                       cell =  row.createCell(2);
                       //设置单元格样式
                       cell.setCellStyle(text(workbook));
                       //设置内容
                       cell.setCellValue(contractNo);
                   }
   
   
                   //货号
                   String productNo = contractProductVo.getProductNo();
                   if(productNo!=null){
                       cell =  row.createCell(3);
                       //设置单元格样式
                       cell.setCellStyle(text(workbook));
                       //设置内容
                       cell.setCellValue(productNo);
                   }
   
   
                   // 数量
                   Integer cnumber = contractProductVo.getCnumber();
                   if(cnumber!=null){
                       cell =  row.createCell(4);
                       //设置单元格样式
                       cell.setCellStyle(text(workbook));
                       //设置内容
                       cell.setCellValue(cnumber);
                   }
   
   
                   // 工厂
                   String factoryName = contractProductVo.getFactoryName();
                   if(factoryName!=null){
                       cell =  row.createCell(5);
                       //设置单元格样式
                       cell.setCellStyle(text(workbook));
                       //设置内容
                       cell.setCellValue(factoryName);
                   }
   
   
                   // 客户名称
                   Date deliveryPeriod = contractProductVo.getDeliveryPeriod();
                   if(deliveryPeriod!=null){
                       cell =  row.createCell(6);
                       //设置单元格样式
                       cell.setCellStyle(text(workbook));
                       //设置内容
                       cell.setCellValue(simpleDateFormat.format(deliveryPeriod));
                   }
   
   
                   // 客户名称
                   Date shipTime = contractProductVo.getShipTime();
                   if(shipTime!=null){
                       cell =  row.createCell(7);
                       //设置单元格样式
                       cell.setCellStyle(text(workbook));
                       //设置内容
                       cell.setCellValue(simpleDateFormat.format(shipTime));
                   }
   
                   //货号
                   String tradeTerms = contractProductVo.getTradeTerms();
                   if(tradeTerms!=null){
                       cell =  row.createCell(8);
                       //设置单元格样式
                       cell.setCellStyle(text(workbook));
                       //设置内容
                       cell.setCellValue(tradeTerms);
                   }
   
               }
           }
           /*
           //下载乱码的时候我们可以使用url编码
           String fileName = "出货表.xlsx";
           fileName = URLEncoder.encode(fileName,"UTF-8");
   
           //通知浏览器以附件下载的形式处理内容
           response.setHeader("content-disposition","attachment;filename="+fileName);
           //获取到浏览器的输出流
           ServletOutputStream outputStream = response.getOutputStream();
           workbook.write(outputStream);*/
   
           /*
               download(ByteArrayOutputStream byteArrayOutputStream, HttpServletResponse response, String returnName)
                       byteArrayOutputStream 将文件内容写入ByteArrayOutputStream
                       response：响应对象
                       returnName : 下载文件的名字
            */
           //字节数组输出流，该类的本质就是内部维护一个字节数组
           ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
           //把工作簿数据写入到ByteArrayOutputStream中
           workbook.write(byteArrayOutputStream);
           new DownloadUtil().download(byteArrayOutputStream,response,"出货表.xlsx");
   
       }
   
   
       //大标题的样式
       public CellStyle bigTitle(Workbook wb){
           CellStyle style = wb.createCellStyle();
           Font font = wb.createFont();
           font.setFontName("宋体");
           font.setFontHeightInPoints((short)16);
           font.setBold(true);//字体加粗
           style.setFont(font);
           style.setAlignment(HorizontalAlignment.CENTER);				//横向居中
           style.setVerticalAlignment(VerticalAlignment.CENTER);		//纵向居中
           return style;
       }
   
       //小标题的样式
       public CellStyle title(Workbook wb){
           CellStyle style = wb.createCellStyle();
           Font font = wb.createFont();
           font.setFontName("黑体");
           font.setFontHeightInPoints((short)12);
           style.setFont(font);
           style.setAlignment(HorizontalAlignment.CENTER);				//横向居中
           style.setVerticalAlignment(VerticalAlignment.CENTER);		//纵向居中
           style.setBorderTop(BorderStyle.THIN);						//上细线
           style.setBorderBottom(BorderStyle.THIN);					//下细线
           style.setBorderLeft(BorderStyle.THIN);						//左细线
           style.setBorderRight(BorderStyle.THIN);						//右细线
           return style;
       }
   
       //文字样式
       public CellStyle text(Workbook wb){
           CellStyle style = wb.createCellStyle();
           Font font = wb.createFont();
           font.setFontName("Times New Roman");
           font.setFontHeightInPoints((short)10);
   
           style.setFont(font);
   
           style.setAlignment(HorizontalAlignment.LEFT);				//横向居左
           style.setVerticalAlignment(VerticalAlignment.CENTER);		//纵向居中
           style.setBorderTop(BorderStyle.THIN);						//上细线
           style.setBorderBottom(BorderStyle.THIN);					//下细线
           style.setBorderLeft(BorderStyle.THIN);						//左细线
           style.setBorderRight(BorderStyle.THIN);						//右细线
   
           return style;
       }
   ```















