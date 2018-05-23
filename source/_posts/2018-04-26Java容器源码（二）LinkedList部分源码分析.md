---
title: Java容器源码（二）LinkedList部分源码分析
date: 2018-05-22
tags: [LinkedList, 容器, 源码]
categories: [tech]
urlname: Java-SourceCode-LinkedList
---
***

<p><img class="img-logo" src="https://image-1251774567.cosgz.myqcloud.com/blog/2018-05-23-aaditya-kalia-332803-unsplash%20-1-.jpg" alt=""/></p>


前言：LinkedList 是 Java 中非常重要的集合类，本质上就是一个数据存储的数据结构。底层是用链表来实现的。这个类注意与 ArrayList 对比

<!-- more -->

### 继承关系

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-04-26-072352.jpg)

### 常用方法、内部类及变量

-   维护的变量

    ```java
        transient int size = 0;
    
        /**
         * Pointer to first node.
         * Invariant: (first == null && last == null) ||
         *            (first.prev == null && first.item != null)
         */
        transient Node<E> first;
    
        /**
         * Pointer to last node.
         * Invariant: (first == null && last == null) ||
         *            (last.next == null && last.item != null)
         */
        transient Node<E> last;
    ```

-   Node 内部类（三元组）

    ```java
        private static class Node<E> {
            E item;
            Node<E> next;
            Node<E> prev;
    
            Node(Node<E> prev, E element, Node<E> next) {
                this.item = element;
                this.next = next;
                this.prev = prev;
            }
        }
    ```

-   构造方法

    -   LinkedList()

        ```java
            /**
             * Constructs an empty list.
             */
            public LinkedList() {
            }
        ```

    -   LinkedList(Collection<? extends E> c)
    
        ```java
            /**
             * Constructs a list containing the elements of the specified
             * collection, in the order they are returned by the collection's
             * iterator.
             *
             * @param  c the collection whose elements are to be placed into this list
             * @throws NullPointerException if the specified collection is null
             */
            public LinkedList(Collection<? extends E> c) {
                this();
                addAll(c);
            }
        ```
    
    
-   add 方法

    -   add(E e): boolean
    
        ```java
            public boolean add(E e) {
                // 将节点 e 增加到链表的最后
                linkLast(e);
                return true;
            }
        ```

    -   add(int index, E element): void
    

        ```java
            /**
             * Inserts the specified element at the specified position in this list.
             * Shifts the element currently at that position (if any) and any
             * subsequent elements to the right (adds one to their indices).
             *
             * @param index index at which the specified element is to be inserted
             * @param element element to be inserted
             * @throws IndexOutOfBoundsException {@inheritDoc}
             */
            public void add(int index, E element) {
                //在指定的位置增加节点
        
                // index >= 0 && index <= size 就报错 IndexOutOfBoundsException
                checkPositionIndex(index);
        
                // 索引值正好等于节点个数，则直接调用 linkLast 增加在节点的最尾处
                // 如果不相等，则调用 node 方法返回对应索引的结点，然后 调用 linkBefore 方法，
                // 将新元素加到 原结点之前
                // 注意这果 node 方法返回的节点一点非空，因为索引值在最开始就检查过了
                if (index == size)
                    linkLast(element);
                else
                    linkBefore(element, node(index));
            }
        ```

    -   linkLast(E e): void

    ```java
        /**
         * Links e as last element.
         */
        void linkLast(E e) {
            // 保存一下尾指点
            final Node<E> l = last;
    
            // 生成新的结点，新结点的前置指针指向链表的最后，后置为空
            final Node<E> newNode = new Node<>(l, e, null);
    
            // 将 last 指向新生成的结点
            last = newNode;
    
            // 如果 l 为空，换言之，这是添加的第一个节点，则将 first 指向新结点
            // 如空不为空，则将链表的最后结点的后置指针指向新结点
            if (l == null)
                first = newNode;
            else
                l.next = newNode;
    
    
            size++;
            modCount++;
        }
    ```

    -   linkBefore(E e, Node<E> succ): void

    ```java
        /**
         * Inserts element e before non-null Node succ.
         */
        void linkBefore(E e, Node<E> succ) {
            // assert succ != null;
            // 跟 linkLast 一样的套路
            /**
             * 1. 先保存前一个节点指针
             * 2. 生成新结点，新结点前置为前一个节点，后置为传入节点
             * 3. 传入节点的前置修改为新结点
             * 4. 判断前上个节点是否存在，如果不存在，则说明新生成的节点应该在最开始，所以 first = newNode
             *      如果存在，则前一个节点的后置指针指向新生成的节点
             */
            final Node<E> pred = succ.prev;
            final Node<E> newNode = new Node<>(pred, e, succ);
            succ.prev = newNode;
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            size++;
            modCount++;
        }
    ```

-   remove 方法

    -   remove(): E

        ```java
            /**
             * Retrieves and removes the head (first element) of this list.
             *
             * @return the head of this list
             * @throws NoSuchElementException if this list is empty
             * @since 1.5
             */
            public E remove() {
                // 默认移除第一个
                return removeFirst();
            }
        ```

    -   remove(int index): E

        ```java
            /**
             * Removes the element at the specified position in this list.  Shifts any
             * subsequent elements to the left (subtracts one from their indices).
             * Returns the element that was removed from the list.
             *
             * @param index the index of the element to be removed
             * @return the element previously at the specified position
             * @throws IndexOutOfBoundsException {@inheritDoc}
             */
            public E remove(int index) {
                checkElementIndex(index);
                // node 方法找到对应结点，unlink 方法除掉它
                return unlink(node(index));
            }
        ```

    -   unlinkLast 及 unlink 方法 
    
        ```java
            /**
             * Unlinks non-null last node l.
             */
            private E unlinkLast(Node<E> l) {
                // assert l == last && l != null;
                final E element = l.item;
                final Node<E> prev = l.prev;
                l.item = null;
                l.prev = null; // help GC
                last = prev;
                if (prev == null)
                    first = null;
                else
                    prev.next = null;
                size--;
                modCount++;
                return element;
            }
        
            /**
             * Unlinks non-null node x.
             */
            E unlink(Node<E> x) {
                // assert x != null;
                // 保存节点对应信息
                final E element = x.item;
                final Node<E> next = x.next;
                final Node<E> prev = x.prev;
        
                if (prev == null) {
                    first = next;
                } else {
                    prev.next = next;
                    x.prev = null;
                }
        
                if (next == null) {
                    last = prev;
                } else {
                    next.prev = prev;
                    x.next = null;
                }
        
                x.item = null;
                size--;
                modCount++;
                return element;
            }
        ```

-   indexOf(Object o): int

    
    ```java
        /**
         * Returns the index of the first occurrence of the specified element
         * in this list, or -1 if this list does not contain the element.
         * More formally, returns the lowest index {@code i} such that
         * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
         * or -1 if there is no such index.
         *
         * @param o element to search for
         * @return the index of the first occurrence of the specified element in
         *         this list, or -1 if this list does not contain the element
         */
        public int indexOf(Object o) {
            // 寻找索引，没有什么意外，遍历
            int index = 0;
            if (o == null) {
                for (Node<E> x = first; x != null; x = x.next) {
                    if (x.item == null)
                        return index;
                    index++;
                }
            } else {
                for (Node<E> x = first; x != null; x = x.next) {
                    if (o.equals(x.item))
                        return index;
                    index++;
                }
            }
            return -1;
        }
    ```


