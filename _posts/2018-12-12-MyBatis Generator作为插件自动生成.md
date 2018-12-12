---
layout:     post
title:      MyBatis Generator作为maven插件自动生成代码
subtitle:   最为方便的自动化生成方式
date:       2018-12-12
author:     KC
header-img: img/post-bg-debug.png
catalog: true
tags:
    - MyBatis
    - MyBatis Generator
---


>MyBatis Generator几种生成方式中的最方便的一种

## 背景

  众所周知，Mybatis是目前orm框架中最流行和最受欢迎的一款。使用的人多了，其不方便的地方也就暴露出来了，比如说针对业务中每一个需要用到的表都要新建对应的 dao,entity,Mapper及xml文件 这对于我们开发人员来说是很多没有意义的重复劳动，这个时候 MyBatis Generator 就应运而生了。

## 简介

  MyBatis Generator (MBG) 是一个Mybatis的代码生成器 MyBatis 和 iBATIS. 他可以生成Mybatis各个版本的代码，和iBATIS 2.2.0版本以后的代码。 他可以内省数据库的表（或多个表）然后生成可以用来访问（多个）表的基础对象。 这样和数据库表进行交互时不需要创建对象和配置文件。 MBG的解决了对数据库操作有最大影响的一些简单的CRUD（插入，查询，更新，删除）操作。 对于联合查询和存储过程手我们仍然需要手写SQL和对象。

## 运行方式

  MyBatis Generator (MBG) 可以通过以下方式运行:

- 从 命令提示符 使用 XML 配置文件 
- 作为 Ant 任务 使用 XML 配置文件 
- 作为 Maven Plugin
- 从另一个 Java 程序 使用 XML 配置文件

#### 通过Maven运行 MyBatis Generator



1. 指定maven项目pom中引入插件

```xml
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <!--建议使用最新版本-->
    <version>1.3.7</version>
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
    </dependencies>
    <configuration>
        <!-- 指定MBG配置文件位置-->
        <configurationFile>${project.basedir}/src/main/resources/generatorConfig.xml</configurationFile>
        <verbose>true</verbose>
        <!-- 是否覆盖上次生成的文件-->
        <overwrite>true</overwrite>
    </configuration>
</plugin>
```
注意： 这里直接在插件运行过程中动态引入了mysql数据库对应的驱动。如果不希望在这里引入的话，需要在MBG配置文件中指定**本地已存在的对应数据库驱动文件的位置**



2. 项目resources目录下引入MBG配置文件

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
               "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">


<generatorConfiguration>
    <!--导入属性配置-->
    <properties resource="datasource.properties"></properties>

    <!--指定特定数据库的jdbc驱动jar包的位置-->
    <classPathEntry location="E:/idea_workspace/mumall/mysql-connector-java-5.1.6-bin.jar"/>

    <context id="default" targetRuntime="MyBatis3">

        <!-- optional，旨在创建class时，对注释进行控制 -->
        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <!--jdbc的数据库连接 -->
        <jdbcConnection
                driverClass="${db.driverClassName}"
                connectionURL="${db.url}"
                userId="${db.username}"
                password="${db.password}">
        </jdbcConnection>


        <!-- 非必需，类型处理器，在数据库类型和java类型之间的转换控制-->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>


        <!-- Model模型生成器,用来生成含有主键key的类，记录类 以及查询Example类
            targetPackage     指定生成的model生成所在的包名
            targetProject     指定在该项目下所在的路径
        -->
        <javaModelGenerator targetPackage="com.mumall.pojo" targetProject="./src/main/java">
            <!-- 是否允许子包，即targetPackage.schemaName.tableName -->
            <property name="enableSubPackages" value="false"/>
            <!-- 是否对model添加 构造函数 -->
            <property name="constructorBased" value="true"/>
            <!-- 是否对类CHAR类型的列的数据进行trim操作 -->
            <property name="trimStrings" value="true"/>
            <!-- 建立的Model对象是否 不可改变  即生成的Model对象不会有 setter方法，只有构造方法 -->
            <property name="immutable" value="false"/>
        </javaModelGenerator>

        <!--mapper映射文件生成所在的目录 为每一个数据库的表生成对应的SqlMap文件 -->
        <sqlMapGenerator targetPackage="mappers" targetProject="./src/main/resources">
            <property name="enableSubPackages" value="false"/>
        </sqlMapGenerator>

        <!-- 客户端代码，生成易于使用的针对Model对象和XML配置文件 的代码
                type="ANNOTATEDMAPPER",生成Java Model 和基于注解的Mapper对象
                type="MIXEDMAPPER",生成基于注解的Java Model 和相应的Mapper对象
                type="XMLMAPPER",生成SQLMap XML文件和独立的Mapper接口
        -->

        <!-- targetPackage：mapper接口dao生成的位置 -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.mumall.dao" targetProject="./src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
        </javaClientGenerator>


        <table tableName="mumall_shipping" domainObjectName="Shipping" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
        <table tableName="mumall_cart" domainObjectName="Cart" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
        <table tableName="mumall_cart_item" domainObjectName="CartItem" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
        <table tableName="mumall_category" domainObjectName="Category" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
        <table tableName="mumall_order" domainObjectName="Order" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
        <table tableName="mumall_order_item" domainObjectName="OrderItem" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
        <table tableName="mumall_pay_info" domainObjectName="PayInfo" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
        <table tableName="mumall_product" domainObjectName="Product" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false">
            <columnOverride column="detail" jdbcType="VARCHAR" />
            <columnOverride column="sub_images" jdbcType="VARCHAR" />
        </table>
        <table tableName="mumall_user" domainObjectName="User" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>


        <!-- geelynote mybatis插件的搭建 -->
    </context>
</generatorConfiguration>
```

这里配置文件的名称要与plugin configurationFile配置中的名称保持一致，否则执行生成命令时是找不到的