---
title: 构建工具——Maven_Draft
date: 2018-02-06
tags: [构建工具, maven, 过程记录, 备忘, 草稿]
categories: [tech]
urlname: buildtools-maven-draft
---
***

前言：主要是记录在Maven入门过程中的草稿与备忘。

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-06-maven-icon-1.jpg)

<!--more-->

## Maven基本构建命令

-   clean
    清除`Target`目录

-   compiler
    编译项目
    
-   test
    测试项目代码

-   package
    打包

-   install
    安装jar包到本地仓库

## Maven标准目录
     

```
|-- pom.xml
|-- src
|   |-- main
|   |   `-- java
|   |   `-- resources
|   |   `-- webapp
|   |   `-- filters
|   `-- test
|   |   `-- java
|   |   `-- resources
|   |   `-- filters
|   `-- it
|   `-- assembly
|   `-- site
`-- LICENSE.txt
`-- NOTICE.txt
`-- README.txt
```

## 自动生成项目目录

- archetype:genetype

## 生命周期

>   三个大过程不会相互依赖，但每一个过程中的后面的小步骤会依赖前面的步骤，如只执行`package`命令的话也会进行`compiler->test->package`的过程

1.  Clean（清理）
2.  Default（**构建**）
    执行后面的会依次执行前面的
    
    - compiler
    - test
    - package
    - install
3.  Site
    生成、发布站点

## setting.xml文件
    
如修改镜像站、本地仓库地址、JDK默认的版本等

```xml
<profile>
    <id>jdk-1.8</id>
    <activation>
        <jdk>1.8</jdk>
    </activation>
    <properties>
        <maven.complier.source>1.8</maven.complier.source>
        <maven.complier.target>1.8</maven.complier.target>
        <maven.complier.complierVersion>1.8</maven.complier.complierVersion>
    </properties>
</profile>
```
    
## 依赖冲突、原则及解决

分别依赖于不同版本的Jar包所导致的冲突问题

Maven在有不同Jar版本时所采用的原则：

-   短路径优先
-   路径相同时谁先被解析到用谁的（即在POM.xml中的位置）

排除引入方法，如下图：

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-06-041044.jpg)

## 聚合与继承

-   聚合
    将多个Maven项目同时进行操作
    `<packing>pom</packing>` + `<module>`
    
-   继承
    - `<packing>pom</packing>`
    - `<dependencyManagement>`
    - 子类写`<parent>`，无需写版本号及Scope，由引入的父POM控制

## Tomcat7 Maven插件


```xml
<plugin>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <version>2.2</version>
</plugin>
```
>   遇到的一个问题

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-06-070036.jpg)

>   主要是因为Tomcat Maven 插件中提供的Servlet Jar 与Spring中的Servlet Jar 冲突，将手动引入的Servlet Jar作用域标为`provided`即可。

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-06-070734.jpg)   

## 其它备忘

-   设置Maven工程的编码
    
    ```xml
    project.build.sourceencoding
    ```

-   在插件中设置JDK版本
    
    ```xml
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
    ```


