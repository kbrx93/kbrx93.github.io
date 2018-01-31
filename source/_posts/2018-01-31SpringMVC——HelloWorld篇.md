---
title: SpringMVC——HelloWorld篇
date: 2018-01-31
tags: [框架, SpringMVC, MVC, HelloWorld]
categories: [tech]
urlname: springmvc-helloworld
---
***

前言：这篇笔记主要是记录SpringMVC的HelloWorld(*初学习*)

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-01-31-061754.png)

<!--more-->

## 准备

-   Intellij IDEA 2017.3.3
-   Tomcat：8.5
-   Spring：4.0 +
-   SpringMVC：4.0+
-   JSTL：1.2
-   Servlet：4.0

>   web.xml的版本：`2.3`(这里的版本对之后的配置有一点小影响)

## 主要过程

1.  用IDEA搭建Maven工程，并完善相关目录结构
2.  pom.xml中引入相关依赖
3.  编写核心类（在这里是`Controller类 `）
4.  配置Spring配置文件`application.xml`及`web.xml`的相关配置项，分两种：

    *   全注解（**最常用**）
    *   在配置文件中配置`bean`的方式(了解即可)

5.  编写JSP并运行Tomcat进行相关测试
6.  过程中遇到的问题

## 详细过程

记录详细的过程

### 环境搭建
用IDEA搭建Maven工程，并完善相关目录结构

1.  选择搭建Maven工程，并使用maven-webapp模板
![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-01-31-024301.png)
2. 填写相关的groupId和ArtifactId
![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-01-31-024501.png)
3. 最后Finish即可
![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-01-31-024712.png)
4. 因为默认的模板的目录结构是不完整的，所以还需要手动补全，如下图
![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-01-31-025058.png)

### 引入依赖
pom.xml中引入相关依赖

```xml
<dependencies>
        <!-- ============Spring相关============ -->
        <!-- Spring-Context -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.3.7.RELEASE</version>
        </dependency>

        <!-- Spring-AOP -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>4.3.7.RELEASE</version>
        </dependency>

        <!-- ============Spring MVC相关============ -->
        <!-- Spring-WebMVC -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>4.3.7.RELEASE</version>
        </dependency>

        <!-- Spring-Web -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>4.3.7.RELEASE</version>
        </dependency>

        <!-- ============Servlet相关============ -->
        <!-- JSTL -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>

        <!-- Servlet -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.0</version>
        </dependency>
    </dependencies>
```

### 编写核心类

1.  注解方式
    
    ```java
    /**
     * @author: kbrx93
     */
    @Controller
    public class AnnotationController {
    
        @RequestMapping("/annotation_hello")
        public void sayHello() {
            System.out.println("AnnotationController.sayHello");
        }
    }
    ```
    
2.  继承接口方式
    
    ```java
    /**
     * SpringMVC HelloWrold
     *
     * @Author: kbrx93
     */
    public class HelloWorldController implements Controller{
    
        @Override
        public ModelAndView handleRequest(
                javax.servlet.http.HttpServletRequest httpServletRequest, 
                javax.servlet.http.HttpServletResponse httpServletResponse
        ) {
            System.out.println("This is the SpringMVC Hello World example!");
            ModelAndView mv = new ModelAndView();
            mv.addObject("user", "kbrx93");
            mv.setViewName("index.jsp");
            return mv;
        }
    }
    ```

### 编写配置文件

1.  对应注解方式（`“扫”、“驱”、“静”`）
    > 注意要在文件头引入`xmlns:mvc="http://www.springframework.org/schema/mvc"` 和 `http://www.springframework.org/schema/mvc`
    
    ```java
     <!-- 全注解方式配置(扫驱静) -->

    <!-- 1. 开启扫描（biubiubiu～～） -->
    <context:component-scan base-package="com.kbrx93.controller"/>

    <!-- 2. 引入注解驱动 -->
    <!--

        不是必要条件，但最好加上。
        一句话：你的手机通信加上之后可能就由移动变成联通了，提供的服务相同，但组件不同。
        并且相对来说更加可控

     -->
    <mvc:annotation-driven/>

    <!-- 3. 静态资源处理Servlet（因为拦截“/”覆盖掉原本的Tomcat中处理静态资源的Servlet） -->
    <!-- 静态资源：你把老子相好拦下来了却不打算干点什么？？？ -->
    <mvc:default-servlet-handler/>
    ```
     
2.  对应接口方式
    
    ```xml
    <!-- name属性即为访问的路径 -->
    <bean name="/hello" class="com.kbrx93.controller.hello.HelloWorldController"></bean>
    ```

### 运行测试

-   `/webapp/show.jsp`文件如下：

    ```html
    <%--
      Author: kbrx93
    --%>
    <%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
    <html>
    <head>
        <title>Title</title>
    </head>
    <body>
    ${user}
    </body>
    </html>
    
    ```

-   运行Tomcat测试结果：
    ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-01-31-055459.jpg)
    
    ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-01-31-055501.jpg)

### 问题

记录过程中遇到的问题   

-   `${user}`EL表达式在JSP中不生效？？？

    主要是由于`web.xml`文件中声明xsd版本问题（*前面提到的*），一般有以下几种：
    
    -   web-app_2_2.xsd
        
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>  
    <!DOCTYPE web-app PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.2//EN" "http://java.sun.com/dtd/web-app_2_2.dtd"> 
        ```
    -   web-app_2_3.xsd
        
        ```xml
    <?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE web-app PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" "http://java.sun.com/dtd/web-app_2_3.dtd">
        ```
    -   `web-app_2_4.xsd` 

        ```xml
        <?xml version="1.0" encoding="UTF-8"?>  
<web-app xmlns="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" version="2.4" xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee   http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd"> 
        ```

    -   web-app_2_5.xsd 

        ```xml
        <?xml version="1.0" encoding="UTF-8"?>  
<web-app xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" version="2.5" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee   http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">  
        ```

    其中只有2.4版本的EL解析开关是默认`打开`的，其它都是关闭的，所以自然而然解决方法就有两种：
    
    1.  更换声明为`2.4`版本
    2.  在每一个使用到EL表达式的JSP页面的开头`<%@ page %>`标签中加上`isELIgnored="false"`属性，即
        
        ```xml
        <%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
        ``` 

-   使用注解方式调用返回值为`null`的方法发生`404`？？？

    由于使用了注解，所以对于方法的返回值、方法名、参数等没有了限制，但是要注意，如果在参数中没有注入一个`HttpServletResponse`参数，那么就一定要返回一个模型视图对象。**可以理解为一定要有一个响应对象。**
    
-   在项目的`web.xml`文件中配置了拦截`/`请求（*即所有*）的`DispatcherServlet`，之后直接访问静态资源时`如*.html`等报`404`错误？？？

    这是因为在`Tomcat`的`web.xml`中也配置了一个拦截`/`的默认`Servlet`，默认的静态资源本来也会由它处理，但是由于后加载的项目的`web.xml`也有一个拦截`/`的`Servlet`，故被覆盖了，然而静态资源在SpringMVC项目中却没有进行相关的配置，故`404`。      
    解决方法即在配置文件`application.xml`中引入`<mvc:default-servlet-handler/>`，将静态资原交由SpringMVC的默认Servlet处理即可。

## 写在最后

这篇主要是对SpringMVC做一个初步的认识，后面对其执行流程及基本组件会加以补充。    



