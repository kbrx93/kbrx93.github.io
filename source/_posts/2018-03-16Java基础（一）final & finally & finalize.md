---
title: Java基础（一）final & finally & finalize
date: 2018-03-17
tags: [Java, 基础, 区别, final, finally, finalize]
categories: [tech]
urlname: Java-Basis-final-finally-finalize
---
***

前言：对final、finally、及finalize三个Java概念进行区别。（由于网上的资料非常多，可能只是列一些关键及在查找过程中发现的相关联的知识点，并提供一些值得参考的文章）


<!--more-->

## final

-   Java关键字、"不可改变"
-   作用
    -   在类上：不能被继承或重写
    -   在方法上：不能被重写
    -   在成员变量上：在使用中不能被改变

>   final变量初始化时机：
>   1.  final变量定义时
>   2.  构造函数

### 文章

-   final -> 内联 -> 类层次分析 -> 虚方法 -> 接口变量与final -> 匿名内部类与final -> private 与final

-   [深入理解Java中的final关键字](http://www.importnew.com/7553.html)
-   [Final of Java，这一篇差不多了](https://www.jianshu.com/p/f68d6ef2dcf0) 
    这篇更底层，值得一看

>   上面的文章中有几点我个人觉得需要注意或补充，如下

1.  文中提到在方法上用final关键字可以使编译器编译的时候转入`内联或内嵌`模式，会提高效率，这一点也在《Think in Java》一书中提及。
    
    但是，请注意是在==早期==的时候，在现代的Java编译器中，可以通过CHA（Class Hierarchy Analysis 类层次分析）来智能地判断一个方法是否可以进行内联，从而进行接下来的操作，并不一定需要final的"提示"，==也就是final在现代的编译器中和提高效率没有直接联系==

2.  上面提到了内联及类层次分析的概念，先说内联，其本质就是`copy`

 
    ```Java
    public void A {
        B();
    }
    
    public final void B {
        balabala
    }
    ```
    
    上面的代码编译器在调用A的时候，发现调用了B，所以本来是要进行一系列压栈、保存现场、弹栈等一系列的操作（因为Java中的动态绑定，只有到了运行期，才知道实际调用的方法），但是它发现B是个final的[手动滑稽]，所以它知道这个家伙是个倔脾气，谁也不能改变它，所以它就直接把B的代码copy到了A中调用B的地方（代码展开，`静态绑定`），接着顺序运行代码了，这样的效率要比正常的方法调用高。

    >   关于内联内容，可以看看[final修饰递归方法会提高效率吗？](https://www.zhihu.com/question/66083114) 
 
    
3.  `类层次分析`，一种对类的关系进行分析的方法（废话），前面说了，通过类层次分析可以在编译时对方法进行智能的分析，从而决定是否能进行内联，与final无关。具体过程如下（引用自[java中final的作用](http://www.mamicode.com/info-detail-2206376.html)）：
    
    -   编译器在进行内联时, 如果是非虚方法, 那么直接进行内联

    -   如果遇到虚方法(使用invokevirtual进行调用), 则java通过引入"类型继承关系分析"(Class Hierarchy Analysis, CHA), CHA查询此方法在当前程序下是否有多个目标版本可供选择

        -   如果查询结果只有一个版本, 则可以进行内联, 称为`守护内联`

        -   如果CHA查询得到多个版本的目标方法, 则编译器会使用内联缓存来完成方法内联: 在未发生方法调用之前, 内联缓存状态为空, 当第一次调用发生后, 缓存记录下方法接收者的版本信息, 并且每次进行方法调用的时候都比较接收者版本, 如果以后进来的每次调用的方法接收者版本都一样则继续使用内联, 如果发生变化, 则说明程序使用了虚方法的多态特性, 则会取消内联, 查找虚方法进行方法的分派。


    
1.  `接口变量与final`
    
    ```java
    interface A{ 
    int i = 0; //i 默认是public static final
    }   
    ```
    这里可理解成接口本身就相当于一个模板，所以模板内的东西一定是可见并且不可变的
    
1.  `在匿名类中所有变量都必须是final变量`

    这句更准确的表达是匿名内部类来自外部闭包环境的自由变量必须是final的，关于这个问题，记住一个点：`变量同步`，即为了局部变量与在内部类建立的拷贝副本保持一致。

    >   具体参考：
    >   -   [java为什么匿名内部类的参数引用时final？](https://www.zhihu.com/question/21395848)

1. `private 与final`
    两者功能上有重叠部分（不可被子类见自然不可重写），所以写在一起没有问题，但是一般不这么写
    
## finally

-   异常处理
-   finally其实就是执行时机及各式各样的return这两个问题

### 文章

1.  [关于 Java 中 finally 语句块的深度辨析](https://www.ibm.com/developerworks/cn/java/j-lo-finally/)

    这篇文章深度是有的，但是Demo举的不是很好
    
2.  [Java finally语句到底是在return之前还是之后执行？](http://www.cnblogs.com/lanxuezaipiao/p/3440471.html)

    对比上一篇，这篇文章更好懂，同时，针对值传递与引用传递，文中说得不是很清楚，在下面的评论有十分易懂的解释。截图如下：
    
    ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-03-16-153954.jpg)
    
    ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-03-16-154113.jpg)
    
    ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-03-16-154145.jpg)
    
    可以理解成，你在try中的返回之前，会有一个变量（不管是值还是引用）的==备份==，你在finally中的所有操作都只是操作原来的变量，当操作完成后返回try中的return时会被备份覆盖，而区别只是引用类型你在finally可以操作它指向的对象，并没有改动引用本身（还是指向原来的那个对象），所以你对它指向的对象的操作自然不会被覆盖掉。
    
>   基本上搞定上面两篇，finally的基本问题就搞懂了（当然还可以再深入）    

## finalize

1.  [java finalize方法](https://basebase.github.io/2016/08/27/java-finalize%E6%96%B9%E6%B3%95/)

    -   finalize方法==不一定==会执行，只能保证如果重写了finalize，则GC时一定会执行该方法（JVM没有意外退出）
    
    -   System.gc()只是加快gc调用，也并一定会执行GC
    
    -   关键是==对象销毁过程==要理解

1. [java finalize方法总结、GC执行finalize的过程](http://bijian1013.iteye.com/blog/2288223)

    可以结合第一篇的对象销毁过程一起理解

1.  [JVM的垃圾回收机制详解和调优](http://developer.51cto.com/art/200607/29278_all.htm)

    -   finalize抛出的未捕获异常只会导致该对象的finalize执行退出

    -   用户可以自己调用对象的finalize方法，但是这种调用是正常的方法调用，和对象的销毁过程无关

    -   JVM在GC的时候如果发现对象有finalize方法，一定会去执行，但如果finalize方法是空的话，它会"视而不见"，也就是说直接Reclaimed，不会放入F-Queue

1.  [详解java垃圾回收机制(转)及finalize方法（转）](https://my.oschina.net/u/2297250/blog/383407)
    
    这篇更多的是GC的知识，可以配合《深入理解Java虚拟机》来Enjoy

1.  [Java 对象释放与 finalize 方法](http://mazhuang.org/2015/12/15/java-object-finalize/)

    -   发生 GC 时一个对象的内存是否释放取决于是否存在该对象的引用，如果该对象包含对象成员，那对象成员也遵循本条
    
    -   对象里包含的对象成员按声明顺序进行释放

