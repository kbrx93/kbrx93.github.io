---
title: Java杂记（七）关于Java中访问修饰符protected的一个小疑问
date: 2018-04-08
tags: [Java, 杂记, protected, 访问修饰符]
categories: [tech]
urlname: Java-Notes-Protected-Question
---
***

前言：在查询关于 Java 数据域绑定问题的时候，意外看到一篇关于访问修饰符 `protected` 的文章，感觉受益颇多，纠正了自己以前对于 `protected` 的"误解"，特此记录一下自己的验证过程及自己的理解。

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-04-08-075056.jpg)

<!--more-->

## 主要内容

-   强烈建议阅读[此文章](https://blog.csdn.net/justloveyou_/article/details/61672133)

-   下面说说我自己的理解，首先有两个要素要确定。

    -   调用的方法（假设为 sayHello 方法，并且是 protected 的）来自哪里（我们姑且称之为源类）

    -   由源类确定sayHello方法的可见性（同包 + 子类）
    
        >   `注意`：
        >   -   这里在我看来同包的可见性与子类的可见性是`不一样`的，下面会结合例子进行说明，力求表述清楚。
        >   -   Java 中成员变量是静态绑定的，在继承的时候没有动态效果，为更好理解与说明，所以下面的例子全部以方法为主。

## 例子

-   以最普通的一个例子开头，在包 `access.access1` 下有如下 `Person` 类
    
    ```java
    package access.access1;
    
    public class Person {
    
        protected void sayHello() {
            System.out.println("Person.sayHello");
        }
    }
    ```
    
    -   由上可知：sayHello 方法的可见范围为 access1 包及Person的子类

-   先试试同包的情况：


    ```java
    package access.access1;
    
    public class CallObject {
    
        public static void main(String[] args) {
    
            Person p = new Person();
            p.sayHello();
        }
    }
    ```

    -   结果：不出意外，一切正常，编译通过。

-   再试子类（不同包）：在包 `access.access2` 下有如下 `Man` 类

    ```java
    package access.access2;
    
    import access.access1.Person;
    
    public class Man extends Person {
    
        public static void main(String[] args) {
            Person p = new Person();
            p.sayHello(); // Error
        }
    }
    ```
    
    -   结果：嗯。。。出错了，说好的可见呢?

-   再试试？

    ```java
    package access.access2;
    
    import access.access1.Person;
    
    public class Man extends Person {
    
        public static void main(String[] args) {
            Man man = new Man();
            man.sayHello();
        }
    }
    ```

    -   结果：成功运行。证明了一点，protected 对于子类与同包的可见性是不一样的。

        -   包：可调用
        -   子类：可获得

## + static？

-   在把 Person 类的 sayHello 方法标成 static 后，上面的三个例子均通过，具体原因以后再探索吧。

## 小结

-   在面对 protected 方法时，首先看它来自哪里，再看它在哪里调用（子类还是同包）。
-   子类实例无法调用父类实例的 protected 方法。（父类：我可以搞一份给你，但你不能拿我的）


