---
title: Java杂记（三）ArrayList自动扩容的一道面试题
date: 2018-04-04
tags: [Java, 杂记, ArrayList, 自动扩容, 面试题]
categories: [tech]
urlname: Java-ArrayList-Automatic-expansion
---
***

前言：在一个微信群中看到一道关于阿里ArrayList自动扩容的题，感觉自己以前的认知出现了偏差，特此记录下。

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-04-04-031155.png)

<!--more-->

## 主要过程

-   题目：总内存为150M，已有ArrayList占100M，此时ArrayList要自动扩容，怎么办？

-   答案：要扩就扩呗。

-   解析

    -   这道本身就是一个陷阱，我们由源码可以知道，ArrayList要扩容时要先新建一个现在容量的1.5倍的新对象数组
        
        ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-04-04-012004.jpg)


    -   然后再调用Arrays.copy（本质上是调用System.arraycopy）复制过去

        ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-04-04-011836.jpg)

    >   好了，我的疑问出现了，要新建一个150M的对象数组，但是只有50M了，怎么办？？，又不能序列化到磁盘，太慢了

    -   在调试过程中我发现，上面这个疑问是多么地**愚蠢**。。

    -   关键在于对`T[] copy = new Object[newLength]`这句理解不够透彻，这句话干了什么，生成了==若干个null的引用==啊。
    
    -   也就是说，对象数组只有在`实际增加`（也就是put的时候）的时候才会占用内存（就是新买的钱包里没有钱一个道理）

    -   打完，收工。





