---
layout: post
title: "Gradle 快速入门"
subtitle: "JVM 世界的富有突破性的构建工具"
date: 2019-02-06 22:05:00
author: "Echcz"
header-img: "img/in-post/2019-01-31-manjaro-install-use.jpg"
catalog: true
tags:
  - "Gradle"
  - "构建工具"
  - "JVM"
---
{% raw %}
## 前言

Gradle 是 JVM 世界的富有突破性的构建工具，正迅速成为许多开源项目和前沿企业构建系统的选择，同时也在挑战遗留的自动化构建项目。Gradle 具有如下特性:

* 像 Ant 一样的通用的灵活的构建工具
* 像 Maven 一样的基于`约定优于配置`的构建框架
* 强大的多工程构建支持
* 基于 Apache Ivy 的强大的依赖管理
* 对已有的 Maven 和 Ivy 仓库的全面支持
* 支持传递性依赖管理，而不需要远程仓库或者 pom.xml 或者 ivy 配置文件
* Ant 式的任务和构建是 Gradle 的第一公民
* 基于Groovy的领域特定语言(DSL)来声明项目设置，抛弃了基于XML的各种繁琐配置
* 具有广泛的领域模型

本文章只是快速入门，针对的是初学者。目的是让初学者对这个优秀的工具有一个大致的了解，并能看懂一些常用的 Gradle 项目，也能自己构建一些简单的 Gradle 项目。

## 基本概念

Gradle 本身的领域对象主要有 Project 和 Task。Project 为 Task 提供了执行上下文，所有的 Plugin 要么向Project中添加用于配置的 Property，要么向 Project 中添加不同的 Task。一个Task 表示一个逻辑上较为独立的执行过程，比如编译 Java 源代码，拷贝文件，打包 Jar 文件，甚至可以是执行一个系统命令或者调用 Ant 。另外，一个 Task 可以读取和设置 Project 的 Property 以完成特定的操作。

首先让我们创建一个最简单的 Gradle 的项目，在 Project 中创建一个名为 `helloWorld` 的 DefaultTask 类型的 Task。在项目目录里创建一个名为 `build.gradle` 的文本文件，内容如下：

``` groovy
task helloWorld {
    doLast {
        println "Hello World!!!"
    }
}
```

默认情况下，Gradle 将当前目录下的 build.gradle 文件作为项目的构建文件。文件内容其实就是 Groovy 代码，Gradle 提供了 DSL 方便我们的构建。`task` 是一个 Groovy 方法，`task` 后的大括号里的内容是传递给 `task` 方法的一个闭包，用于配置这个 task。`doLast` 也是一个 Grovvy 方法，表示在执行该 task 时的最终执行传递进来的闭包。

在项目目录下执行：

``` shell
project $ gradle helloWorld
```

gradle 将执行 helloWorld Task 以进行构建。命令行将输出如下：

``` shell
> Task :helloWorld
Hello World!!!

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
```

## Task 的创建与配置

### 创建 Task

* 创建 Task 最方便的就是使用 Project 的 task 方法了：

``` groovy
task taskA {
    doLast {
        println "this is taskA"
    }
}
```

这将在 TaskContainer 类型对象 tasks 中创建一个名为 `taskA` 的 Task，用于打印 `it is taskA`。

* 使用 TaskContainer 的 create 方法创建 Task：

``` groovy
tasks.create(name: "taskB") {
    doLast {
        println "this is taskB"
    }
}
```

### 声明 Task 的类型

Task 是有类型的，不同类型的 Task 定义了不同的行为。默认情况下，task 定义的 Task 类型是 DefaultTask。我们可以显示地声明 Task 类型（当然可以自定义 Task 类型，这个下面说）：

``` groovy
task taskCopy(type: Copy) {
    from 'source'
    into 'target'
}
```

`type: Copy` 将taskCopy 声明为 Copy 类型。执行 taskCopy 将把 `source` 目录里的内容复制到 `target` 目录里。*注意：文件路径是相对当前 Project 而言的，也就是 build.gradle 文件所在目录*

### 声明 Task 之间的依赖关系

Task 之间是可以有依赖关系的，这样 Task 之间就会组织成一条条执行链。例如 taskC 依赖 taskB，这样在执行 taskC 时，taskB 会先执行。

* 在定义 Task 时声明它的依赖关系(被依赖的 Task 应先定义)：

``` groovy
task taskC(dependsOn: taskB) {
    doLast {
        println "this is taskC"
    }
}
```

* 在定义 Task 后，再声明依赖关系：

``` groovy
task taskD {
    doLast {
        println "this is taskD"
    }
}

taskD.dependsOn taskB
```

### 配置 Task 的 Property

Task 除了执行操作外，还可以有多个 Property，这些 Property 用于配置 Task 的属性，以方便配置 Task 的行为。Gradle 为每个 Task 默认定义了一些 Property，比如 description,logger等。此外，每个特定类型的 Task 也有特定的 Property，比如 Copy 类型的 from 和 to 等。当然，我们也可以动态地向 Task 中添加 Property。通常在执行一个 Task 之前，我们需要先设定 Property。Gradle 提供了多种方法设置 Task 的 Property。

* 在定义 Task 时设置 Property：

``` groovy
task taskE {
    description = "this is taskE"

    doLast {
        println description
    }
}
```

* 通过调用 Task 的 configure 方法设置 Property：

``` groovy
task taskF {
    doLast {
        println description
    }
}

taskF.configure {
    description = "this is taskF"
}
```

Gradle 在执行 Task 时分为两个阶段，首先是配置阶段，然后才是实际的执行阶段。所以在 taskF 时，依然会打印 `this is taskF`。

* 通过闭包的方式设置 Property:

``` groovy
task taskG {
    doLast {
        println description
    }
}

taskG {
    description = "this is taskG"
}
```

实际上，通过闭包的方式在内部也是通过调用 Task 的 configure 方法完成的。

## 增量式构建

每个 Task 都拥有 inputs(输入) 和 outputs(输出) 属性，他们的类型分别是 TaskInputs 和 TaskOutputs。在执行一个 Task 时，如果它的输入与输出与前一次执行时没有发生变化，那么 Gradle 便认为该 Task 是最新的(UP-TO-DATE)，这时，Gradle 将不予执行。这就是增量式构建，通过增量式构建可以节约构建时间。Task 的输入与输出可以是文件或文件夹，也可以是 Project 的 Property，还可以是某个闭包所定义的条件。

下面我们定义一个 Task，用于将 sourceDir 文件夹里的文件的内容合并到 combine.txt 文件里：

``` groovy
task taskCombine {
    def sources = fileTree('sourceDir')
    def target = file('combine.txt')

    doLast {
        target.withPrintWriter { writer ->
            sources.each { source ->
                writer.println source.text
            }
        }
    }
}
```

多次执行 taskCombine Task，Task 都会反复执行，即使上一次执行已经得到了所需的结果。如果 taskCombine 是一个耗时的任务，这样势必会没必要地浪费大量时间。如果将 sources 声明为 taskCombine 的 inputs，将 target 声明为 taskCombine 的 outputs，将会执行增量式构建：

``` groovy
task taskCombine2 {
    def sources = fileTree('sourceDir')
    def target = file('combine.txt')

    inputs.dir sources
    outputs.file target

    doLast {
        target.withPrintWriter { writer ->
            sources.each { source ->
                writer.println source.text
            }
        }
    }
}
```

可以看到后一个 Task 只比前一个 Task 多了两行用于声明 inputs 和 outputs 的代码。当首次执行 taskCombine2 时，Gradle 会执行该 Task，但如果再执行一次，命令行将显示：

``` shell
> Task :taskCombine2 UP-TO-DATE

BUILD SUCCESSFUL in 0s
1 actionable task: 1 up-to-date
```

可以看到 taskCombine2 被标记为 `UP-TO-DATE`，表示该 Task 是最新的，Gradle 不予执行。

如果我们修改了 inputs（对于 taskCombine2 来说就是 sourceDir 目录里的内容）或修改/删除了 outputs（对于 taskCombine2 来说就是 combine.txt 文件），该 Task 就不是最新的了，Gradle 就会重新执行该 Task。我们可以使用 upToDateWhen 方法来决定一个 Task 是否是最新的，该方法接受一个闭包作为检查条件，感兴趣的同学可以自行了解，这里就不详细说明了。

## 多 Project 构建

在多 Project 的项目中，我们会操作多个 Project 领域对象。Gradle 提供了强大的多 Project 构建支持。要创建多 Project 的 Gradle 项目，我们首先需要在根 Project 中加入名为 `settings.gradle` 的配置文件，该文件应该包含各个子 Project 的名称。比如，我们有一个根 Project 名为 `root-project`，它包含有两个子 Project，名字分别为 `sub-project1` 和 `sub-project2`，此时对应的文件目录结构如下：

``` shell
root-project
├── build.gradle
├── settings.gradle
├── sub-project1
│   └── build.gradle
└── sub-project2
    └── build.gradle
```

root-project/settings.gradle 文件的内容如下：

``` groovy
include 'sub-project1', 'sub-project2'
```

### 在根 Project 中配置所有 Project

在 root-project/build.gradle 文件中添加如下内容：

``` groovy
allprojects {
    task taskAll {
        doLast {
            println project.name
        }
    }
}
```

执行如下命令：

``` shell
root-project $ gradle taskAll
# 注：gradle :taskAll 将只执行 root-project 里的 taskAll
# 注：gradle :sub-project1:taskAll 将只执行 sub-project1 里的 taskAll
```

命令行将输出：

``` shell
> Task :taskAll
root-project

> Task :sub-project1:taskAll
sub-project1

> Task :sub-project2:taskAll
sub-project2

BUILD SUCCESSFUL in 0s
3 actionable tasks: 3 executed
```

### 在根 Project 中配置所有子 Project

在 root-project/build.gradle 文件中添加如下内容：

``` groovy
subprojects {
    task taskSub {
        doLast {
            println project.name
        }
    }
}
```

执行如下命令：

``` shell
root-project $ gradle taskSub
```

命令行将输出：

``` shell
> Task :sub-project1:taskSub
sub-project1

> Task :sub-project2:taskSub
sub-project2

BUILD SUCCESSFUL in 0s
2 actionable tasks: 2 executed
```

### 在根 Project 中配置某个 Project

在 root-project/build.gradle 文件中添加如下内容：

``` groovy
project(':sub-project1') {
   task taskPro1 {
       doLast {
           println project.name
       }
   }
}
```

执行如下命令：

``` shell
root-project $ gradle taskPro1
```

命令行将输出：

``` shell
> Task :sub-project1:taskPro1
sub-project1

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
```

### 以一种更灵活的方式配置 Project

Gradle 其本质就是用 Groovy 语言操作一些领域对象，所以我们可以将 Groovy 的语言特性用在 Gradle 领域对象上，比如我们可以对 Project 进行过滤后再配置：

``` groovy
configure(allprojects.findAll { it.name.startsWith('sub-') }) {
   task taskSub2 {
       doLast {
            println "I am ${project.name}"
       }
   }
}
```

执行如下命令：

``` shell
root-project $ gradle taskSub2
```

命令行将输出：

``` shell
> Task :sub-project1:taskSub2
I am sub-project1

> Task :sub-project2:taskSub2
I am sub-project2

BUILD SUCCESSFUL in 0s
2 actionable tasks: 2 executed
```

### 多 Project 的 Task 之间的依赖配置

多 Project 项目的不同项目的 Task 之间也是可以相互依赖的。在 root-project/build.gradle 文件中添加如下内容：

``` groovy
allprojects {
    task taskA {
        doLast {
            println "this is ${project.name}"
        }
    }

    task taskB {
        doLast {
            println "I am ${project.name}"
        }
    }
}
```

在 root-project/sub-project1/build.gradle 文件中添加如下内容：

``` groovy
taskA.dependsOn ':sub-project2:taskB'
```

执行如下命令：

``` shell
root-project $ gradle :sub-project1:taskA
```

命令行将输出：

``` shell
> Task :sub-project2:taskB
I am sub-project2

> Task :sub-project1:taskA
this is sub-project1

BUILD SUCCESSFUL in 0s
2 actionable tasks: 2 executed
```

## 自定义 Property

Gradle 在默认情况下已经为 Project 定义了很多 Property，其中比较常见的有：

* project : Project 本身
* name : Project 的名字
* path : Project 的绝对路径
* description : Project 的描述信息
* buildDir : Project 构建结果存入目录
* version : Project 的版本号

我们可以 project.PROPERTY_NAME 的方式使用 Project 的 Property。如果直接使用 PROPERTY_NAME，Gradle 则会设置 delegate 所指对象的 Property：默认情况下，如果所处的 Task 有则使用 Task 的，如果没有则会使用 Project 的。

我们也可以自定义 Project 或 Task 的 Property：

* 在 build.gradle 文件中定义 Property：

``` groovy
ext.propertyA = "this is propertyA" // 定义 Project 的 Property
ext {
    propertyB = "this is propertyB" // 以闭包的方式定义 Property
}

task taskH {
    ext.propertyA = "this is taskH.propertyA" // 定义 taskH 的 Property

    doLast {
        println propertyA // 打印 taskH 的 propertyA
        println project.propertyA // 打印 Project 的 propertyA
        println propertyB // 打印 Project 的 propertyB
    }
}
```

* 通过命令行参数定义 Property：

``` groovy
task taskI {
    doLast {
        println propertyC
    }
}
```

执行以下命令：

``` shell
project $ gradle -PpropertyC="this is propertyC 1" taskI
```

将打印 `this is propertyC 1`。

* 通过 JVM 系统参数定义 Property：

在 Java 中，我们可以通过 `-D` 参数定义 JVM 系统参数。在 Gradle 中，也可以通过 `-D` 参数向 Project 中传入 Property，只是在传入时要加上 `org.gradle.project.` 作为前缀。对于上面的 taskI，执行以下命令：

``` shell
project $ gradle -Dorg.gradle.project.propertyC="this is propertyC 2" taskI
```

将打印 `this is propertyC 2`。

* 通过系统环境变量设置 Property：

系统环境变量需要以 `ORG_GRADLE_PROJECT_` 作为前缀。对于上面的 taskI，执行以下命令：

``` shell
project $ export ORG_GRADLE_PROJECT_propertyC="this is propertyC 3"
project $ gradle taskI
```

将打印 `this is propertyC 3`。

## 自定义 Task 类型

Gradle 中的 Task 要么是由不同的 Plugin 引入的，要么是我们自己在 build.gradle 文件中直接创建的。在默认情况下，我们所创建的 Task 是 DefaultTask 类型，该类型是一个非常通用的 Task 类型，而在有些时候，我们希望创建一些具有特定功能的 Task ，比如 Copy 和 Jar 等。还有些时候，我们希望定义自己创建的 Task 类型。接下来我们以定义一个简单的 MyTask 为例，讲解如何自定义 Task 类型：

* 在 build.gradle 文件中定义 Task 类型：

``` groovy
class MyTask extends DefaultTask { // 自定义 Task 类型
    @Optional // 表示属性是可选的，即在创建该 Task 时，可以不指定 msg
    String msg = 'I am MyTask'

    @TaskAction // 表示 Task 根执行的动作，即在执行该 Task 时，run 方法将会执行
    def run() {
        println "run: $msg"
    }
}

task taskJ(type: MyTask) // 执行 taskJ 将打印 `run: I am myTask`

task taskK(type: MyTask) { // 执行 taskK 将打印 `run: I am taskK`
    msg = "I am taskK"
}
```

* 在当前工程中定义 Task 类型：

Gradle 在执行时，会自动地查找 buildSrc 目录下所定义的Task类型，并首先编译该目录下的 groovy 代码以供 build.gradle 文件使用。在当前项目的 buildSrc/src/main/groovy/mytask 目录下创建 MyTask2.groovy 文件，内容如下：

``` groovy
package mytask
import org.gradle.api.*
import org.gradle.api.tasks.*

class MyTask2 extends DefaultTask {
    @Optional
    String msg = 'I am MyTask2'

    @TaskAction
    def todo() {
        println "todo: $msg"
    }
}
```

在 build.gradle 文件中加入如下内容：

``` groovy
// 因为 MyTask2 是定义在 mytask 包下，所以要加  `mytask.` 作为前缀
task taskL(type: mytask.MyTask2) // 执行 taskL 将打印 `todo: I am myTask2`

task taskM(type: mytask.MyTask2) { // 执行 taskM 将打印 `todo: I am taskM`
    msg = "I am taskL"
}
```

* 在单独的项目中定义 Task 类型：

我们可以创建一个 Groovy 项目，定义 Task 类型，并在客户端项目中引入该项目定义的 Task 类型。

首先创建服务项目如下：

在 src/main/groovy/mytask2 目录下创建 Mytask3.groovy 文件，内容如下：

``` groovy
package mytask2
import org.gradle.api.*
import org.gradle.api.tasks.*

class MyTask3 extends DefaultTask {
    @Optional
    String msg = 'I am MyTask3'

    @TaskAction
    def play() {
        println "play: $msg"
    }
}
```

创建 build.gradle 文件，内容如下：

``` groovy
plugins {
    id 'groovy'
    id 'maven'
}

version = '1.0'
group = 'mytasks'
archivesBaseName = 'mytask2'

repositories.mavenCentral()

dependencies {
    implementation gradleApi()
    implementation localGroovy()
}

uploadArchives {
    repositories.mavenDeployer {
        repository(url: 'file:../lib')
    }
}
```

执行以下命令，将生成的构建成品上传到 ../lib 目录中，以供接下来客户端项目使用：

``` shell
mytask2 $ gradle uploadArchives
```

然后在客户端项目的 build.gradle 文件中做如下配置：

``` groovy
buildscript {
    repositories {
        maven {
            url 'file:../lib'
        }
    }

    dependencies {
        classpath group: 'mytasks', name: 'mytask2', version: '1.0'
    }
}

task taskN(type: mytask2.MyTask3) // 执行 taskN 将打印 `play: I am MyTask3`

task taskO(type: mytask2.MyTask3) { // 执行 taskN 将打印 `play: I am taskO`
    msg = "I am taskO"
}
```

## 自定义 Plugin

Gradle 通过使用 Plugin 向 Project 添加 Task，定义 Configurations 和 Property 等。自定义 Plugin 与 自定义 Task 类型的方式类似，都可以：

* 在 build.gradle 文件中定义
* 在当前工程中定义
* 在单独的项目中定义

接下来我只演示在 build.gradle 文件中定义 Plugin，相信大家能根据`自定义 Task 类型` 举一反三：

在 build.gradle 文件中添加如下内容：

``` groovy
plugins {
    id 'ShowInfo' // 应用自定义插件：ShowInfo
}

class ShowInfo implements Plugin<Project> { // 自定义 Plugin
    void apply(Project project) {
        project.ext.myName = "echcz" // 为 Project 添加 Property：myName
        project.task("showInfo") { // 为 Project 添加 Task：showInfo
            doLast {
                println project.myName
                println project.name
                println project.path
                println project.description
                println project.buildDir
                println project.version
            }
        }
    }
}
```

执行以下命令：

``` shell
project $ gradle showInfo
```

将输出：

``` shell
> Task :showInfo
echcz
project
:
null
/home/echcz/project/build
unspecified

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
```

## 实战演练：快速构建一个 Java 项目

想使用 Gradle 快速构建一个 Java 项目，最简单的方法就是使用 java Plugin。java Plugin 通过向项目中添加一些相互依赖的 Task，构建了生命周期的概念，就像 Maven 一样。

java Plugin 主要引入的 Task 及说明如下：

| Task | 说明 |
|:--:|:-- |
| compileJava | 利用 javac 编译 Java 源文件 |
| processResource | 将项目的资源文件复制到 classes 目录中 |
| classes | 组装 classes 目录 |
| compileTestJava | 利用 javac 编译 Java 测试源文件 |
| processTestResource | 将项目的测试资源文件复制到 classes 目录中 |
| testClasses | 组装测试用的 classes 目录 |
| jar | 打包成 jar 文件 |
| javadoc | 生成 java 帮助文档 |
| test | 执行测试用例 |
| assemble | 组合分析所有的档案文件 |
| check | 执行所有的验证类任务 |
| build | 执行构建 |
| uploadArchives | 上传存档文件 |
| clean | 删除 build 文件，例项目回归到原始状态 |
| cleanTASK_NAME | 删除由任务产生的文件，比如 cleanJar 就是删除任务 jar 产生的文件 |

下面演示创建一个名为 `java-demo` 的 Java 项目：

### 配置 build.gradle 与 settings.gradle 文件

在 build.gradle 文件中添加如下内容：

``` groovy
plugins {
    id 'java' // 使用 java Plugin
}

group 'com.github.echcz.gradle' // 设置项目组
version '1.0' // 设置项目版本

sourceCompatibility = '1.8' // 设置源码编译版本为 JDK 1.8

repositories {
    mavenCentral() // 使用 maven 中央仓库
}
```

在 settings.gradle 文件中添加如下内容：

``` groovy
rootProject.name = 'java-demo'
```

### 创建项目目录结构

在默认情况下，java Plugin 采用了与 Maven 相同的 Java 项目目录结构：

``` shell
java-demo
├── build.gradle
├── settings.gradle
└── src
    ├── main
    │   ├── java
    │   └── resources
    └── test
        ├── java
        └── resources
```

实际上 Gradle 使用了 SourceSet 这个概念用于映射目录结构，用于指定哪些文件要被编译，哪些文件根被排除。java Plugin 默认实现了 两个 SourceSet：`main`，`test`。如果我们想创建一个名为 `api` 的 SourceSet 来存放接口，可以如下声明：

``` groovy
sourceSets {
    api
    main {
        compileClasspath = compileClasspath + files(api.output.classesDir)
    } // 1
   test {
        runtimeClasspath = runtimeClasspath + files(api.output.classesDir)
    } // 2
}

classes.dependsOn apiClasses // 3
```

java Plugin 会每个 SourceSet 创建相应的 Task：对于名为 `api` 的 SourceSet 会为其创建 compileApiJava（对 main 来说是 compileJava），processApiResource（对 main 来说是 processResource） 和 apiClasses（对 main 来说是 classes） 这三个 Task。因为编译 main 中的源码依赖 api 中的类文件，所以需要添加 `3` 处的内容；因为我们需要将 api 编译生成的类文件放在 main 的 classpath 下，所以需要添加 `1` 处的内容；因为运行测试时需要加载 api 中的类文件，所以需要添加 `2` 处的内容。

### 依赖管理

通常一个 Java 项目总会依赖其它项目，可能是第三方类库，也可能是你自己开发的其它 Java 项目。所以依赖管理是使用 Gradle 绕不开的话题。

#### 配置 Repository

在声明对其它项目的依赖时，我们需要配置 Gradle 的 Repository。在配置好依赖后，Gradle 会自动下载这些依赖到本地缓存（默认在 ~/.gradle/caches/modules-2/files-2.1 目录）。Gradle 可以使用 Maven 和 Ivy 的 Repository，也可以使用本地文件系统作为 Repository。正如前文的 build.gradle 文件里的 `repositories` 配置项，就是使用 Maven 的 Repository。

使用过 Maven 的同学会发现 Maven 的中央仓库的下载速度很慢而会使用 阿里云的 Maven 仓库镜像。Gradle 也可以使用阿里云的镜像，其方法是在 build.gradle（对当前项目有效） 或 ~/.gradle/init.gradle 或 ~/.gradle/init.d/\*.gradle（对当前用户有效）或 $GRADLE_HOME/init.d/\*.gradle（对当前系统有效）添加如下内容：

``` groovy
allprojects {
    repositories {
        maven { url 'https://maven.aliyun.com/repository/public/' }
        mavenLocal()
        mavenCentral()
    }
}
```

#### 配置依赖

Gradle 会对依赖进行分组管理，以方便在不同的 Task 使用不同的依赖。每一组依赖称为一个 Configuration，我们先将依赖加入到 Configuration 中，再在配置 Task 时通过configurations.CONFIGURATION_NAME.DEPENDENCY_NAME 使用依赖。例子如下：

``` groovy
configurations {
    myDependency
}

dependencies {
   myDependency 'org.apache.commons:commons-lang3:3.0'
}

task showMyDependency {
    doLast {
        println configurations.myDependency.asPath
    }
}
```

不过 java Plugin 已经为我们定义好 configurations 了，我们只需要给不同的 Configuration 配置相应的依赖就行了。例子如下：

``` groovy
dependencies {
   implementation 'org.apache.commons:commons-lang3:3.0'
   testCompile 'junit:junit:4.12'
}
```

java Plugin 定义的 Configuration 及说明：

| Configuration | 说明 |
|:--:|:-- |
| implementation | 仅实施依赖项，在编译期隐藏自身使用的依赖 |
| compileOnly | 仅编译期依赖项，运行时不提供 |
| compileClasspath | 编译类路径，在编译源码（执行 compileJava 任务）时提供 |
| annotationProcessor | 编译期使用的注释处理器 |
| runtimeOnly | 仅运行期依赖项 |
| runtimeClasspath | 运行时依赖项，包括 implementation 和 runtimeOnly 的元素 |
| testImplementation | 仅测试实施依赖项 |
| testCompileOnly | 仅测试编译期依赖项 |
| testCompileClasspath | 测试编译类路径 |
| testRuntimeOnly | 仅测试运行期依赖项 |
| testRuntimeClasspath | 测试运行时依赖项 |
| archives | 生成制品依赖项，在执行 uploadArchives 任务时提供 |
| default | 项目在运行时所需的工件和依赖项 |

## 参考链接

* 关于 Gradle 的详细信息可以访问 [Gradle 官网](https://gradle.org/)
* Gradle 的教程可以访问 [W3CSchool-珍珍阿姨-Gradle 教程](https://www.w3cschool.cn/gradle/)
* Gradle 的快速入门教程可以访问 [CNBlogs-无知者云-Gradle 快速入门](http://www.cnblogs.com/davenkin/p/gradle-learning-1.html)
* 关于 Gradle 的一些配置说明可以访问 [简书-19snow93-寄Android开发Gradle你需要知道的知识](https://www.jianshu.com/p/8b8a550246bd)
* java Plugin 的使用文档可以访问 [Gradle Docs-The Java Plugin](https://docs.gradle.org/current/userguide/java_plugin.html)

{% endraw %}