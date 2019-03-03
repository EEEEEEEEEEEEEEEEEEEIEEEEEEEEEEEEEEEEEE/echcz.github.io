---
layout: post
title: "Kotlin + Spring Boot 简明教程 - 1.第一个例子"
subtitle: "其实Java(JVM) Web开发的效率也可以很高"
date: 2019-03-03 14:36:00
author: "Echcz"
header-img: "img/in-post/2019-03-03-kotlin-springboot-tutorial.jpg"
catalog: true
tags:
  - "Web"
  - "Kotlin"
  - "SpringBoot"
---
{% include post/190303-kotlin-springboot-tutorial/header.md %}

{% raw %}
[本教程源码](https://github.com/echcz/kotlin-spring-boot-examples/tree/master/start)

## 构建一个简单的 Restful Web

构建一个提供两个 Restful GET 端点的 Web：

* /my/name：从配置文件中读取my.name，并返回
* /my/msg：从配置文件中读取my.msg，并返回

### 使用 Spring Initializr 快速初始化一个 Spring Boot 项目

1. 进入 https://start.spring.io
2. 选择 `Gradle Project` 和 `Kotlin`
3. 设置相关项目信息
4. 选择依赖：Web
5. 生成项目。这将生成项目结构目录、Gradle 配置文件、项目配置文件、启动配置类、测试类

生成的项目的`build.gradle`文件如下：

``` groovy
plugins {
    id 'org.springframework.boot' version '2.1.3.RELEASE'
    id 'org.jetbrains.kotlin.jvm' version '1.2.71'
    id 'org.jetbrains.kotlin.plugin.spring' version '1.2.71'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.github.echcz'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'com.fasterxml.jackson.module:jackson-module-kotlin'
    implementation 'org.jetbrains.kotlin:kotlin-reflect'
    implementation 'org.jetbrains.kotlin:kotlin-stdlib-jdk8'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

compileKotlin {
    kotlinOptions {
        freeCompilerArgs = ['-Xjsr305=strict']
        jvmTarget = '1.8'
    }
}

compileTestKotlin {
    kotlinOptions {
        freeCompilerArgs = ['-Xjsr305=strict']
        jvmTarget = '1.8'
    }
}
```

### 编辑配置文件

这里使用 ymal 作为配置文件类型，因为这相比 properties 更简结明了。编辑 resources/application.yml(删除生成的`resources/application.properties`)：

``` yml
my:
  name: 'echcz'
  msg: '${my.name} at spring boot' # ${} 可以引用定义的属性，这是由 Spring Boot 提供的
```

### 构建数据实体类

定义了配置文件后，我们需要将配置文件引入到程序，这里通过`@Value`注解引入。为了更清晰的项目结构，创建`domain`包，并在此包中定义`My`数据类，内容如下：*注意：这里使用 Kotlin，文件类型是 .kt，以后不作特殊说明都是这样*

``` kotlin
package com.github.echcz.start.domain

import org.springframework.beans.factory.annotation.Value
import org.springframework.stereotype.Component

@Component // @Component 将会在 Spring 容器中实例化一个此类的实例
// @Value 可以注入表达式中的值，使用“${}”可以引入配置文件中的定义的属性
data class My(@Value("\${my.name}") var name: String, @Value("\${my.msg}") var msg: String)
// 注意使用是"\${}" ，因为 kotlin 会将"${}"里的内容作为表达式执行，需要将"$"转义
```

### 创建控制器

控制器用于接收客户端请求并响应。为了更清晰的项目结构，创建`controller`包，并在此包中定义`MyController`类，内容如下：

``` kotlin
package com.github.echcz.start.controller

import com.github.echcz.start.domain.My
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RestController

@RestController // @RestController == @Component + @RequestBody(返回值将会序列化为 Json 响应给客户端，但此注解只能用在方法上)
@RequestMapping("my") // 用于映射路径 /my/**
// @Autowired 用于注入 Spring 容器中的 Bean，先匹配类型，再匹配名称，可以配合 @Qualifier 限定可以注入的 Bean (要求匹配到的 Bean 唯一)
class MyController @Autowired constructor(var my: My) {
    @GetMapping("name") // 用于映射路径 /my/name 且 http 方法为 GET
    fun getName(): String {
        return my.name
    }

    @GetMapping("msg")
    fun getMsg(): String {
        return my.msg
    }
}
```

## 运行

执行如下命令，运行程序：

``` shell
$ gradle bootRun
```

可以使用 [Postman](https://www.getpostman.com/) 查看程序对于不同请求的响应。在此例中：

GET http://localhost:8080/my/name -> echcz

GET http://localhost:8080/my/msg -> echcz at spring boot

[本教程源码](https://github.com/echcz/kotlin-spring-boot-examples/tree/master/start)
{% endraw %}
{% include post/190303-kotlin-springboot-tutorial/footer.md %}