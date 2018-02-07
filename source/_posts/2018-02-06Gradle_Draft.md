---
title: 构建工具——Gradle_Draft
date: 2018-02-06
tags: [构建工具, gradle, 过程记录, 备忘, 草稿]
categories: [tech]
urlname: buildtools-gradle-draft
---
***

前言：记录了在Gradle入门的过程的草稿及备忘。

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-07-gradle-logo.png)

<!-- more -->

## Gradle的安装

见[官网](https://gradle.org/)

## Groovy

介绍Groovy语言的语法特点

-   闭包
-   高效性

## 第一个Gradle项目

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-06-144449.jpg)

>   有些目录及文件（如gradle.properties及subDemo模块）是后来加上去，src目录下的目录结构请参考[Maven的标准目录结构](https://kbrx93.com/tech/buildtools-maven-draft/)

-   理解Project及Task
-   大概的书写过程及形式
-   自定义一个任务
    如下：
    
```groovy
//闭包
def createOther = {
    path - >
    File dir = new File(path)
    if (!dir.exists()) {
        dir.mkdirs()
    }
}
// 生成other目录及子目录
task makeOther() {
    def paths = ['src/main/other', 'src/main/other/WEB-INF']
    doFirst {
        paths.forEach(createOther)
    }
}
/**********************************/

//闭包
def createDir = {
    path - >
    File dir = new File(path)
    if (!dir.exists()) {
        dir.mkdirs()
    }
}
// 生成webapp目录及子目录
// 依赖于makeOther
task makeWeb() {
    dependsOn 'makeOther'
    def paths = ['src/main/webapp', 'src/main/webapp/WEB-INF']
    doFirst {
        paths.forEach(createDir)
    }
}
```

## 生命周期

主要是有三个阶段：

-   初始化
    进行项目的初始化（如Project类的创建）
    
-   配置
    执行配置语句，确定依赖关系
    
-   执行
    执行动作语句，进行构建

## 依赖管理

主要有两点：仓库管理及依赖引入

-   仓库管理

```groovy
repositories {
//    本地仓库
    mavenLocal()

//    中央仓库
    mavenCentral()

//    私服
    maven {
        url "xxxxx"
    }
}
```

-   依赖引入

```groovy
dependencies {
//    两种坐标形式都可以
    testCompile group: 'junit', name: 'junit', version: '4.12'
    compile "junit:junit:4.12"
}
```

## 版本冲突及解决方法

有两种方法来手动解决版本冲突(***Groovy默认选取最高版本***)：

-   强制指定全局版本
    
```groovy
configurations.all {
    resolutionStrategy {
//      关闭Gradle对版本冲突的默认处理（全部更改为最高版本）
        failOnVersionConflict()

//        指定版本
        force "org.hamcrest:hamcrest-core:1.3"
    }
}
```
    
-   针对特定的Jar排除传递依赖
    
```groovy
dependencies {
    compile ("junit:junit:4.12") {
        exclude group:"org.hamcrest", module:"hamcrest-core"
    }
}
```


## 多项目构建

1.  公共属性
    公共的属性（如*group 'com.kbrx93'
version '1.0-SNAPSHOT'*）可以写在整个项目的根下的`gradle.properties`配置文件

2.  子模块相互依赖
    可以用`compile project(":subDemo")`的方式引用
    
3.  allprojects及subprojects

## 自动化测试

Gradle会自动发现test测试用例并执行相关操作

## 发布

使用maven-publish插件进行发布


```groovy
apply plugin: 'maven-publish'

//进行源码的打包任务
task sourceJar(type: Jar) {
    from sourceSets.main.allJava
}

//插件相关的配置
publishing {
    publications {
        maven(MavenPublication) {
            //指定group/artifact/version信息，可以不填。
            //默认使用项目group/name/version作为groupId/artifactId/version
            groupId project.group
            artifactId project.name
            version project.version
            //如果是war包填写components.web，如果是jar包填写components.java
            from components.java

            //配置上传源码
            artifact sourceJar {
                classifier "sources"
            }

        }
    }
    repositories {
        maven {
            //上传地址
            url = "xxx"
            //用户
            credentials {
                username 'kbrx93'
                password '123456'
            }
        }
    }
}
```

