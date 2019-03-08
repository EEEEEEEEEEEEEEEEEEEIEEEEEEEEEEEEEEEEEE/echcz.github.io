---
layout: post
title: "Kotlin + Spring Boot 简明教程 - 2.使用配置属性类与多环境配置"
subtitle: "其实Java(JVM) Web开发的效率也可以很高"
date: 2019-03-08 19:36:00
author: "Echcz"
header-img: "img/in-post/2019-03-03-kotlin-springboot-tutorial.jpg"
catalog: true
previous:
  url: "2019/03/03/kotlin-springboot-tutorial-1/"
  title: "Kotlin + Spring Boot 简明教程 - 1.第一个例子"
tags:
  - "Web"
  - "Kotlin"
  - "SpringBoot"
---
{% include post/190303-kotlin-springboot-tutorial/header.md %}

{% raw %}
[本教程源码](https://github.com/echcz/kotlin-spring-boot-examples/tree/master/configuration)

## 使用配置属性类

通过配置属性类可以很方便地将配置文件定义的信息封装到一个对象中。相比`@Value`注解，配置属性类更优雅，并且还可以定义默认值，这样即使配置文件没有定义相应的信息也可以运行，而不是报错。

使用配置属性类的方法很简单：

* 1\. 添加依赖。在 `build.gradle` 的 `dependencies` 块中加入 `annotationProcessor "org.springframework.boot:spring-boot-configuration-processor"` 依赖项
* 2\. 创建配置属性类，并添加 `@ConfigurationProperties` 和 `@Component` 注解
* 3\. 在需要用到相关配置信息的地方注入配置属性类实例

## 多环境配置

在实际开发中，通常同一套程序会被应用和安装到几个不同的环境，比如：开发、测试、生产等。其中每个环境的数据库地址、服务器端口等等配置都会不同，如果在为不同环境打包时都要频繁修改配置文件的话，那必将是个非常繁琐且容易发生错误的事。

为此，Spring Boot 通过配置多份不同环境的配置文件，并在运动时指定激活的环境，提供了多环境配置支持。

在 Spring Boot 中多环境配置文件名需要满足 `application-{profile}.properties` / `application-{profile}.yml` 的格式，其中{profile}对应你的环境标识，比如：

* application.yml: 默认(default)环境
* application-dev.yml：开发环境
* application-test.yml：测试环境
* application-prod.yml：生产环境

至于哪个具体的配置文件会被加载，需要在application.properties文件或环境变量中通过 `spring.profiles.active` 属性来设置，其值对应{profile}值。如：`spring.profiles.active=test` 就会加载 `application-test.properties` 配置文件内容。

## 构建案例

### 定义配置属性类

在 `domain` 包中定义 `My` 数据类，内容如下：

``` kotlin
package com.github.echcz.configuration.domain

import org.springframework.boot.context.properties.ConfigurationProperties
import org.springframework.stereotype.Component

@ConfigurationProperties(prefix = "my") // 标记类为配置属性类，并且注入配置中前缀为 my 的信息
@Component // 注册 Bean
data class My(var name: String = "echcz", var msg: String = "") // 在构造器中定义了默认值
```

### 定义控制器类

在 `controller` 包中定义 `MyController` 类，内容如下：

``` kotlin
package com.github.echcz.configuration.controller

import com.github.echcz.configuration.domain.My
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RestController

@RestController
@RequestMapping("my")
class MyController @Autowired constructor(val my: My) {
    @GetMapping("name")
    fun getName(): String {
        return my.name
    }

    @GetMapping("msg")
    fun getMsg(): String {
        return my.msg
    }
}
```

### 编辑配置文件

在 `application.yml` 配置文件中定义内容如下：

``` yml
spring:
  profiles:
    active: default

my:
  msg: "spring boot"
```

在 `application-test.yml` 配置文件中定义内容如下：

``` yml
my:
  name: "test"
  msg: "it is in test profile"
```

## 运行

执行如下命令，运行程序：

``` shell
$ gradle bootRun
```

可以使用 [Postman](https://www.getpostman.com/) 查看程序对于不同请求的响应。在此例中：

GET http://localhost:8080/my/name -> echcz

GET http://localhost:8080/my/msg -> spring boot

修改 `application.yml` 中的 `spring.profiles.active` 属性为 `test` 后运行程序，结果为：

GET http://localhost:8080/my/name -> test

GET http://localhost:8080/my/msg -> it is in test profile

[本教程源码](https://github.com/echcz/kotlin-spring-boot-examples/tree/master/configuration)
{% endraw %}
{% include post/190303-kotlin-springboot-tutorial/footer.md %}