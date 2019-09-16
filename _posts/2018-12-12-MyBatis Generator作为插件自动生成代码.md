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



1.指定maven项目pom中引入插件

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



2.项目resources目录下引入MBG配置文件

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

    <!--可多个 最主要用法是指定特定数据库的jdbc驱动jar包的位置 插件中如果依赖的话无需配置 也可引入其他需要的jar包 比如打包好的通用集成实体类-->
    <!--<classPathEntry location="xxx/xxx/mysql-connector-java-5.1.6-bin.jar"/>-->


    <!--context:生成一组对象的环境
    id:必选，上下文id，用于在生成错误时提示
    targetRuntime:
        1，MyBatis3：默认的值，生成基于MyBatis3.x以上版本的内容，包括XXXBySample；
        2，MyBatis3Simple：类似MyBatis3，只是不生成XXXBySample；
    introspectedColumnImpl：类全限定名，用于扩展MBG -->
    <context id="mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">

        <!-- 生成的Java文件的编码 -->
        <property name="javaFileEncoding" value="UTF-8"/>

        <property name="mergeable" value="false"/>

        <!-- <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
             <property name="mappers" value="com.kc.mybatis.mapper.BaseMapper" />
         </plugin>-->

        <!-- 格式化java代码 -->
        <property name="javaFormatter" value="org.mybatis.generator.api.dom.DefaultJavaFormatter"/>
        <!-- 格式化XML代码 -->
        <property name="xmlFormatter" value="org.mybatis.generator.api.dom.DefaultXmlFormatter"/>

        <!-- 创建class时，是否关闭自动生成的默认注释及时间戳 -->
        <commentGenerator>
            <property name="suppressAllComments" value="true"/>
            <property name="suppressDate" value="true"/>
        </commentGenerator>

        <!--jdbc的数据库连接 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3307/kc?useSSL=false&amp;characterEncoding=utf8&amp;serverTimezone=Asia/Shanghai&amp;nullCatalogMeansCurrent=true"
                        userId="root"
                        password="123456"/>

        <!-- 非必需，类型处理器，在数据库类型和java类型之间的转换控制-->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>


        <!-- Model模型生成器,用来生成含有主键key的类，记录类 以及查询Example类
            targetPackage     指定生成的model生成所在的包名
            targetProject     指定在该项目下所在的路径
        -->
        <javaModelGenerator targetPackage="com.kc.demo.webmagic.model" targetProject="src/main/java">
            <!-- 在targetPackage的基础上，根据数据库的schema再生成一层package，最终生成的类放在这个package下，默认为false -->
            <property name="enableSubPackages" value="false"/>
            <!-- 是否对model添加包含所有字段的构造函数 -->
            <property name="constructorBased" value="true"/>
            <!-- 是否对类CHAR类型的列的数据进行trim操作 -->
            <property name="trimStrings" value="true"/>
            <!-- 建立的Model对象是否 不可改变  即生成的Model对象不会有 setter方法，只有构造方法 -->
            <property name="immutable" value="false"/>
            <!-- 设置一个根对象，所有生成的java对象会继承这个类-->
            <property name="rootClass" value="com.kc.demo.webmagic.model.RecordEntity"/>
        </javaModelGenerator>

        <!--mapper接口对应的xml文件生成配置 -->
        <sqlMapGenerator targetPackage="com.kc.demo.webmagic.mapper" targetProject="src/main/java">
            <property name="enableSubPackages" value="false"/>
        </sqlMapGenerator>

        <!-- 生成Mapper接口，如果没有配置该元素，那么默认不会生成Mapper接口
         type：选择怎么生成mapper接口（在MyBatis3/MyBatis3Simple下）：
         1，ANNOTATEDMAPPER：会生成使用Mapper接口+Annotation的方式创建（SQL生成在annotation中），不会生成对应的XML；
         2，MIXEDMAPPER：使用混合配置，会生成Mapper接口，并适当添加合适的Annotation，但是XML会生成在XML中；
         3，XMLMAPPER：会生成Mapper接口，接口完全依赖XML；
         注意，如果context是MyBatis3Simple：只支持ANNOTATEDMAPPER和XMLMAPPER
        -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.kc.demo.webmagic.mapper"
                             targetProject="src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false"/>
        </javaClientGenerator>

        <!--指定目标表名 可以配置多个
        可选属性：
        1，schema：数据库的schema；
        2，catalog：数据库的catalog 如果不指定的话 同样的表名存在多个库中的情况下 connectionURL必须要加上参数；
        3，alias：为数据表设置的别名，如果设置了alias，那么生成的所有的SELECT SQL语句中，列名会变成：alias_actualColumnName
        4，domainObjectName：生成的domain类的名字，如果不设置，直接使用表名作为domain类的名字；可以设置为somepck.domainName，那么会自动把domainName类再放到somepck包里面；
        5，enableInsert（默认true）：指定是否生成insert语句；
        6，enableSelectByPrimaryKey（默认true）：指定是否生成按照主键查询对象的语句（就是getById或get）；
        7，enableSelectByExample（默认true）：MyBatis3Simple为false，指定是否生成动态查询语句；
        8，enableUpdateByPrimaryKey（默认true）：指定是否生成按照主键修改对象的语句（即update)；
        9，enableDeleteByPrimaryKey（默认true）：指定是否生成按照主键删除对象的语句（即delete）；
        10，enableDeleteByExample（默认true）：MyBatis3Simple为false，指定是否生成动态删除语句；
        11，enableCountByExample（默认true）：MyBatis3Simple为false，指定是否生成动态查询总条数语句（用于分页的总条数查询）；
        12，enableUpdateByExample（默认true）：MyBatis3Simple为false，指定是否生成动态修改语句（只修改对象中不为空的属性）；
        13，modelType：参考context元素的defaultModelType，相当于覆盖；
        14，delimitIdentifiers：参考tableName的解释，注意，默认的delimitIdentifiers是双引号，如果类似MYSQL这样的数据库，使用的是`（反引号，那么还需要设置context的beginningDelimiter和endingDelimiter属性）
        15，delimitAllColumns：设置是否所有生成的SQL中的列名都使用标识符引起来。默认为false，delimitIdentifiers参考context的属性

        注意，table里面很多参数都是对javaModelGenerator，context等元素的默认属性的一个复写；
        -->
        <table tableName="proxy_ip" domainObjectName="ProxyIp"
               enableInsert="false" enableSelectByPrimaryKey="false"
               enableUpdateByPrimaryKey="false" enableDeleteByPrimaryKey="false">

            <!-- 与实体类配置的rootClass同理 -->
            <property name="rootClass" value="com.kc.demo.webmagic.model.RecordEntity"/>

            <!--配置生成的实体类及xml想要忽略的字段-->
            <!--<ignoreColumn column="created_by"/>
            <ignoreColumn column="created_time"/>
            <ignoreColumn column="last_modified_by"/>
            <ignoreColumn column="last_modified_time"/>-->
        </table>

    </context>
</generatorConfiguration>

```

这里配置文件的名称要与plugin configurationFile配置中的名称保持一致，否则执行生成命令时是找不到配置的

**注意:** `jdbcConnection` 标签的 `connectionURL` 属性值 在多个数据库中存在相同名称的表的情况下 要加上`nullCatalogMeansCurrent=true` 参数  否则xml文件中生成的resultmap会有重复



3.执行generate命令

1. 项目根目录下执行 mvn mybatis-generator:generate 命令

   ![](http://www.kcblog.cn/img/2018-12-12/1.jpg)

2. idea中执行插件命令

   ![](http://www.kcblog.cn/img/2018-12-12/2.jpg)

#### mapper接口继承通用maper

对于使用通用mapper的小伙伴 如果想一劳永逸的话可以直接在生成mapper接口文件时继承你想继承的通用mapper类  只需要再插件中增加通用mapper的依赖

![](http://www.kcblog.cn/img/2018-12-12/3.jpg)

然后把MBG配置文件中 plugin 标签的那段代码打开即可  value 值是你想要继承的通用mapper的类的全路径

