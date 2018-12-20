```
---
layout:     post
title:      SpringBoot中web项目全局异常处理
subtitle:   自定义异常处理器
date:       2018-12-20
author:     KC
header-img: img/post-bg-debug.png
catalog: true
tags:
    - 全局异常处理
    - SpringMVC
    - ControllerAdvice
    - ExceptionHandler
---
```

> ControllerAdvice + ExceptionHandler + ResponseBody全局异常处理

### 背景

传统的spring web项目中，业务逻辑里发生异常时往往需要一个一个的进行处理，业务方法多了那对应的处理代码就会变得臃肿和重复，所以需要一个全局的统一异常处理，解耦业务代码，返回给前端友好的提示信息。本文就以SpringBoot项目为例介绍如何对项目中的异常进行统一处理。



#### 新建springboot项目，引入相关依赖

```java
  <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```

本文为演示所用，只引入了web依赖



#### 创建自定义业务异常类

![](http://pk0rgr66v.bkt.clouddn.com/1545283030732.jpg)

#### 自定义全局异常处理

![](http://pk0rgr66v.bkt.clouddn.com/1545283336375.jpg)

上图代码中共使用了 @ControllerAdvice  @ExceptionHandler  @ResponseBody 三个注解

- `@ControllerAdvice` 是spring 3.2版本中新增的注解，从名字的定义可以理解为控制增强。用于定义`@ExceptionHandler`，`@InitBinder`和`@ModelAttribute`方法，使这些配置适用于所有使用`@RequestMapping`的方法。 除了`@ControllerAdvice` 还有`@RestControllerAdvice` 他们的区别和`@Controller` 与 `@RestController` 是一个道理
- `@ExceptionHandler` 拦截处理某些异常（一个或多个，不加参数默认拦截所有异常），从而能够减少代码重复率和复杂度
- `@ResponseBody` 将返回的对象结果转换为json格式，如果使用 `@RestControllerAdvice` 的话无需使用

注 本文由于演示的原因没有在具体的处理逻辑中增加日志打印，如果在项目中一定要记得打印相关错误日志

#### 编写测试controller

![](http://pk0rgr66v.bkt.clouddn.com/1545284354378.jpg)

方法中我们手动抛出了自定义的异常，这时候访问对应的地址结果如下所示

![](http://pk0rgr66v.bkt.clouddn.com/1545284484635.jpg)

至此一个简易的全局异常处理器就完成了