---
title: SpringMVC执行流程初识
date: 2018-02-04
tags: [框架, 源码, SpringMVC, 执行流程]
categories: [tech]
urlname: SpringMVC-execution
---
***

前言：针对SpringMVC的源码来初步分析其执行流程

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-Spring.jpg)

<!--more-->

## 总体流程

如下图：

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-Image%202018-02-04%2012-35-40.png)

执行过程：

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-Untitled%20Diagram.xml%20-%20draw.io%202018-02-04%2012-00-20%20copy.png)

## 源码分析

1.  从DispatcherServlet入手.  
    请求进来都会先去找`service方法`

    ![service方法](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-spring-webmvc%E2%80%A6%202018-02-04%2012-07-29.png)
    
    > 此方法在父类`FrameworkServlet`中
    
2.  发现执行processRequest方法，点击进入

    ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-052654.jpg)

    其中关键调用了`doService方法`，此方法在`FrameworkServlet`定义为抽象，即由`DispatcherServlet`实现
    
    ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-spring-webmvc%E2%80%A6%202018-02-04%2012-09-28.png)

    >即实际上执行Dispatcher中的`doService方法`
    
3.  查看doService方法
    
    ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-052735.jpg)

4.  查看**doDispatch**方法（**重点**）
    整个方法体：
    
    ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-052842.jpg)
    
    按步骤来：
    1.  第一部分
        
        ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-053027.jpg)

        -   先判断是否二进制请求，并进行相关处理
        -   然后调用`getHandler方法`获得处理执行链对象
            具体方法体：
            
            ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-053345.jpg)
            
    2.  第二部分
        获得真正Handler的执行者`HandlerAdapter` 
        
        ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-053503.jpg)
        
    3.  第三部分
        开始执行拦截器及Handler

        ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-053641.jpg)
        
        其中的具体方法体如下：
        -   applyPreHandle方法
            
            ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-053920.jpg)
            
        -   applyPostHandle方法

            ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-053942.jpg)

        -   processDispatcherResult方法
            
            ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-054002.jpg)
            
            其中render方法体：
            
            ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-054104.jpg)
            
            triggerAfterCompletion方法体：
            
            ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-02-04-054143.jpg)

## 写在最后

主要是从源码上分析并记录初步的执行流程，话说一个好的截图工具真的能节省很多的时间。。。


