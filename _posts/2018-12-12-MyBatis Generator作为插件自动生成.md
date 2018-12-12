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