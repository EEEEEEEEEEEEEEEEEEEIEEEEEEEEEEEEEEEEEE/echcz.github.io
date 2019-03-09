---
layout: post
title: "Kotlin + Spring Boot 简明教程 - 3.使用 Swagger2 构建 RESTful API 文档"
subtitle: "其实Java(JVM) Web开发的效率也可以很高"
date: 2019-03-09 19:08:00
author: "Echcz"
header-img: "img/in-post/2019-03-03-kotlin-springboot-tutorial.jpg"
catalog: true
previous:
  url: "2019/03/08/kotlin-springboot-tutorial-2/"
  title: "Kotlin + Spring Boot 简明教程 - 2.使用配置属性类与多环境配置"
tags:
  - "Web"
  - "Kotlin"
  - "SpringBoot"
---
{% include post/190303-kotlin-springboot-tutorial/header.md %}

{% raw %}
[本教程源码](https://github.com/echcz/kotlin-spring-boot-examples/tree/master/swagger)

## Swagger2 介绍

往往我们构建的 RESTful API 需要要面对多个开发人员或多个开发团队：IOS 开发、Android 开发或是 Web 开发等。为了减少与其他团队平时开发期间的频繁沟通成本，传统上我们会创建一份 RESTful API 文档来记录所有接口细节，然而这样的做法有以下几个问题：

* 由于接口众多，并且细节复杂（需要考虑不同的 HTTP 请求类型、HTTP 头部信息、HTTP 请求内容等），高质量地创建这份文档本身就非常吃力
* 随着时间推移，不断修改接口实现的时候都必须同步修改接口文档，而文档与代码又处于两个不同的媒介，除非有严格的管理机制，不然很容易导致不一致现象

为了解决上述问题，本文将介绍 RESTful API 的重磅好伙伴 Swagger2，它可以轻松的整合到Spring Boot中，并与 Spring MVC 程序配合组织出强大 RESTful API 文档。它既可以减少我们创建文档的工作量，同时也将说明内容整合到代码中，让维护文档和修改代码整合为一体，可以让我们在修改代码逻辑的同时方便的修改文档说明。另外 Swagger2 也提供了强大的页面测试功能来调试每个 RESTful API。

## 使用 Swagger2

### 添加 Swagger2 依赖

在 `build.gradle` 的 `dependencies` 块中加入 `implementation 'io.springfox:springfox-swagger2:2.9.2'` 和 `implementation 'io.springfox:springfox-swagger-ui:2.9.2'` 依赖项

### 配置 Swagger2

1. 首先在配置类上添加 `@EnableSwagger2` 注解
2. 注册 Swagger2 配置 Bean: `springfox.documentation.spring.web.plugins.Docket`

示例如下：

``` kotlin
package com.github.echcz.swagger

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication
import org.springframework.context.annotation.Bean
import springfox.documentation.builders.ApiInfoBuilder
import springfox.documentation.builders.PathSelectors
import springfox.documentation.builders.RequestHandlerSelectors
import springfox.documentation.service.ApiInfo
import springfox.documentation.service.Contact
import springfox.documentation.spi.DocumentationType
import springfox.documentation.spring.web.plugins.Docket
import springfox.documentation.swagger2.annotations.EnableSwagger2

@SpringBootApplication
@EnableSwagger2
class TestApplication{
    @Bean
    fun docket(): Docket {
        return Docket(DocumentationType.SWAGGER_2) // 设置使用swagger2文档类型
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.github.echcz.swagger.controller")) // 基础扫描包
                .paths(PathSelectors.any()) // 扫描所有路径
                .build()
    }

    private fun apiInfo(): ApiInfo {
        return ApiInfoBuilder()
                .title("Spring Boot Demo") // 标题
                .description("Spring Boot 整合 Swagger2 构建 Restful API doc") // 详情
                .termsOfServiceUrl("https://echcz.github.io/") // 服务条款
                .contact(Contact("Echcz", "https://github.com/echcz", "echcz@outlook.com")) // 联系人
                .version("0.0.1") // 版本
                .build()
    }
}
```

### 使用 Swagger2

在控制器类的路径映射器方法上添加 `@ApiOperation`、`@ApiImplicitParams`、`@ApiImplicitParam` 注解以对 API 构建文档说明。示例如下：

``` kotlin
package com.github.echcz.swagger.controller

import com.github.echcz.swagger.domain.User
import com.github.echcz.swagger.service.UserService
import io.swagger.annotations.ApiImplicitParam
import io.swagger.annotations.ApiImplicitParams
import io.swagger.annotations.ApiOperation
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.web.bind.annotation.*

@RestController
@RequestMapping("user")
class UserController @Autowired constructor(val userService: UserService) {
    @ApiOperation("获取所有用户")
    @GetMapping
    fun getAll(): List<User> {
        return userService.findAll()
    }

    @ApiOperation(value = "获取某个用户", notes = "根据url的{username}来获取某个用户")
    @ApiImplicitParams(
            ApiImplicitParam(name = "username", value = "用户名", required = true, dataType = "String")
    )
    @GetMapping("{username}")
    fun getOne(@PathVariable("username") username: String): User? {
        return userService.findOne(username)
    }

    @ApiOperation(value = "添加/修改用户", notes = "根据{user}的username添加/修改用户")
    @ApiImplicitParam(name = "user", value = "用户实体", required = true, dataType = "User")
    @PostMapping
    fun post(@RequestBody user: User) {
        userService.save(user)
    }

    @ApiOperation(value = "删除某个用户", notes = "根据url的{username}来删除某个用户")
    @ApiImplicitParam(name = "username", value = "用户名", required = true, dataType = "String")
    @DeleteMapping("{username}")
    fun delete(@PathVariable("username") username: String) {
        userService.delete(username)
    }
}
```

## 运行

执行如下命令，运行程序：

``` shell
$ gradle bootRun
```

进入 http://localhost:8080/swagger-ui.html 即可查看 Swagger2 文档。要对某个 API 进入测试，点击其对应的 `Try it out` 按钮。 

[本教程源码](https://github.com/echcz/kotlin-spring-boot-examples/tree/master/swagger)
{% endraw %}
{% include post/190303-kotlin-springboot-tutorial/footer.md %}