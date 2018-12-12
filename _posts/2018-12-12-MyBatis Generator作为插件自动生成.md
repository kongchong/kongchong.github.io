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

> 通过Maven运行 MyBatis Generator

1. 指定maven项目pom中引入插件

```
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
这里配置文件的名称要与plugin configurationFile配置中的名称保持一致，否则是找不到的

