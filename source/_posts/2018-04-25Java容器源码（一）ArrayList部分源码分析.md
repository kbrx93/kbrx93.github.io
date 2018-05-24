---
title: Java容器源码（一）ArrayList部分源码分析
date: 2018-05-03
tags: [ArrayList, 容器, 源码]
categories: [tech]
urlname: Java-SourceCode-ArrayList
---
***

前言：ArrayList 是 Java 中非常重要的集合类，本质上就是一个数据存储的数据结构。底层是用数组来实现的。

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-05-24-nick-hillier-339049-unsplash%20-1-.jpg)

<!--more-->


### 继承关系

![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-04-25-052634.jpg)


### 常用方法、内部类及变量

-   构造方法

    -   ArrayList(): void
 
        ```java
            /**
             * Constructs an empty list with an initial capacity of ten.
             */
            public ArrayList() {
                // DEFAULTCAPACITY_EMPTY_ELEMENTDATA 是一个默认空的 Object 数组
                this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
            }
        ```
    
    -   ArrayList(Collection<? extends E> c): void
    
        ```java
            /**
             * Constructs a list containing the elements of the specified
             * collection, in the order they are returned by the collection's
             * iterator.
             *
             * @param c the collection whose elements are to be placed into this list
             * @throws NullPointerException if the specified collection is null
             */
            public ArrayList(Collection<? extends E> c) {
                // 提供一个与 c 对应的对象数组副本
                // 可用作 new ArrayList(Arrays.asList(Object[] os))，获取 "真正" 的 ArrayList。
                elementData = c.toArray();
                if ((size = elementData.length) != 0) {
                    // c.toArray might (incorrectly) not return Object[] (see 6260652)
                    /**
                     * 如果有元素存在的话，再进行类型的判断，如果得到的运行期的 class 类型不为 Object，
                     * 则再进行复制转型。
                     * 需要这一步的原因是因为 toArray 方法只能保证返回的数组类型是 Object[]，但是实际
                     * 的类型却是由 new 来决定的。举个例子 Father类
                     * Object[] os = new Father[5]
                     * os[0] = new Other()  这一句就会报错，因为 os 实际的类型是 class [LArrayTest$Father;
                     */
                    if (elementData.getClass() != Object[].class)
                        elementData = Arrays.copyOf(elementData, size, Object[].class);
                } else {
                    // replace with empty array.
                    this.elementData = EMPTY_ELEMENTDATA;
                }
            }
        ```

-   add 方法

    ![](https://image-1251774567.cosgz.myqcloud.com/blog/2018-04-25-104641.jpg)
    
    -   ensureCapacityInternal 方法
          
        ```java
        private void ensureCapacityInternal(int minCapacity) {
                ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
            }
        ```
    
    -   ensureExplicitCapacity 方法
    
        ```java
        private void ensureExplicitCapacity(int minCapacity) {
                // 记录对内部结构的修改次数，增加删除元素及扩容操作都需要增加 1 
                modCount++;
        
                // overflow-conscious code
                // 如果容量不够这一次 add 操作，即 elementData 数组是全满的状态，则进入扩容操作
                if (minCapacity - elementData.length > 0)
                    grow(minCapacity);
            }
        ```

-   扩容

    ```java
        /**
         * Increases the capacity to ensure that it can hold at least the
         * number of elements specified by the minimum capacity argument.
         *
         * @param minCapacity the desired minimum capacity
         */
        private void grow(int minCapacity) {
            // overflow-conscious code
            // 原来数组的大小
            int oldCapacity = elementData.length;
            // 新数组大小 = 旧数组大小 * 1.5，并对新数组大小进行一些验证
            int newCapacity = oldCapacity + (oldCapacity >> 1);
            // 新数组大小不够这一次 add 操作的时候，将新数组大小改为最小需要的数组大小，当数组原来大小是 0 时会需要这一判断
            if (newCapacity - minCapacity < 0)
                newCapacity = minCapacity;
            
            //当新数组大小超过规定的最大值（Integer.MAX_VALUE - 8）时，
            if (newCapacity - MAX_ARRAY_SIZE > 0)
                // 如果 minCapacity 小于 0，则说明溢出了，抛出 OOM
                // 否则 return (minCapacity > MAX_ARRAY_SIZE) ?
                //             Integer.MAX_VALUE :
                //             MAX_ARRAY_SIZE;
                newCapacity = hugeCapacity(minCapacity);
            // minCapacity is usually close to size, so this is a win:
            //复制操作
            elementData = Arrays.copyOf(elementData, newCapacity);
        }
    ```
    
    -   Arrays.copyOf 方法

        ```java
        public static <T> T[] copyOf(T[] original, int newLength) {
                return (T[]) copyOf(original, newLength, original.getClass());
            }
        ```
        
    -   上面调用的 copyOf 方法
    
        ```java
         /**
             * Copies the specified array, truncating or padding with nulls (if necessary)
             * so the copy has the specified length.  For all indices that are
             * valid in both the original array and the copy, the two arrays will
             * contain identical values.  For any indices that are valid in the
             * copy but not the original, the copy will contain <tt>null</tt>.
             * Such indices will exist if and only if the specified length
             * is greater than that of the original array.
             * The resulting array is of the class <tt>newType</tt>.
             *
             * @param <U> the class of the objects in the original array
             * @param <T> the class of the objects in the returned array
             * @param original the array to be copied
             * @param newLength the length of the copy to be returned
             * @param newType the class of the copy to be returned
             * @return a copy of the original array, truncated or padded with nulls
             *     to obtain the specified length
             * @throws NegativeArraySizeException if <tt>newLength</tt> is negative
             * @throws NullPointerException if <tt>original</tt> is null
             * @throws ArrayStoreException if an element copied from
             *     <tt>original</tt> is not of a runtime type that can be stored in
             *     an array of class <tt>newType</tt>
             * @since 1.6
             */
            public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        
                // 判断原来数组的类型是不是 Object，如果是，直接生成一个新 Object 数组，
                // 如果不是，则 Array.newInstance 方法（底层也是一个 native 方法）生成一个对应类对象类型的数组
               T[] copy = ((Object)newType == (Object)Object[].class)
                    ? (T[]) new Object[newLength]
                    : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        
                // 这个方法是一个 native 方法，
                // 表示从 original 数组的第 0 个位置开始复制到 copy 数组的 第 0 个位置，
                // 复制长度为Math.min(original.length, newLength)
                System.arraycopy(original, 0, copy, 0,
                                 Math.min(original.length, newLength));
                return copy;
            }
        ```

-   remove 方法

    -   remove(Object o): boolean
    
        ```java
         /**
             * Removes the first occurrence of the specified element from this list,
             * if it is present.  If the list does not contain the element, it is
             * unchanged.  More formally, removes the element with the lowest index
             * <tt>i</tt> such that
             * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>
             * (if such an element exists).  Returns <tt>true</tt> if this list
             * contained the specified element (or equivalently, if this list
             * changed as a result of the call).
             *
             * @param o element to be removed from this list, if present
             * @return <tt>true</tt> if this list contained the specified element
             */
            public boolean remove(Object o) {
                // 如果传入的对象为空，则进行遍历，当遇到第一个为空的元素时，
                // 则进入 fastRemove，而这个方法本质上就是将后面的数组组元素直接向前移动一位
                // 因为不用返回对象（null）
                if (o == null) {
                    for (int index = 0; index < size; index++)
                        if (elementData[index] == null) {
                            fastRemove(index);
                            return true;
                        }
                } else {
                    //传入的对象不空，还是一个遍历，用 equals 方法逐一进行比较，如果相等则进入 fastRemove，并返回 true
                    for (int index = 0; index < size; index++)
                        if (o.equals(elementData[index])) {
                            fastRemove(index);
                            return true;
                        }
                }
                return false;
            }
        ```

    -   remove(int index): E
    

        ```java
            /**
             * Removes the element at the specified position in this list.
             * Shifts any subsequent elements to the left (subtracts one from their
             * indices).
             *
             * @param index the index of the element to be removed
             * @return the element that was removed from the list
             * @throws IndexOutOfBoundsException {@inheritDoc}
             */
            public E remove(int index) {
                // 检查传入的索引是否大于数组元素的个数
                // 如果大的话，则抛 IndexOutOfBoundsException
                // 如果传入的负数的话，则在运行 elementData[index] 的时候就会报错，所以不用关心，
                // 而这里只因为数组的大小在大多数情况下是大于元素个数的，后面的一部分为 null
                rangeCheck(index);
        
                // 修改次数 + 1
                modCount++;
                
                // 查询旧值
                E oldValue = elementData(index);
        
                // 元素移动的个数
                int numMoved = size - index - 1;
                // 移动数组后面的元素，即后面的元素向前移动一个单元
                if (numMoved > 0)
                    System.arraycopy(elementData, index+1, elementData, index,
                                     numMoved);
                // 将最后的元素置 null，帮助 GC
                elementData[--size] = null; // clear to let GC do its work
        
                return oldValue;
            }
        ```
    
-   toArray(T[] a): T[]

    ```java
        /**
         * Returns an array containing all of the elements in this list in proper
         * sequence (from first to last element); the runtime type of the returned
         * array is that of the specified array.  If the list fits in the
         * specified array, it is returned therein.  Otherwise, a new array is
         * allocated with the runtime type of the specified array and the size of
         * this list.
         *
         * <p>If the list fits in the specified array with room to spare
         * (i.e., the array has more elements than the list), the element in
         * the array immediately following the end of the collection is set to
         * <tt>null</tt>.  (This is useful in determining the length of the
         * list <i>only</i> if the caller knows that the list does not contain
         * any null elements.)
         *
         * @param a the array into which the elements of the list are to
         *          be stored, if it is big enough; otherwise, a new array of the
         *          same runtime type is allocated for this purpose.
         * @return an array containing the elements of the list
         * @throws ArrayStoreException if the runtime type of the specified array
         *         is not a supertype of the runtime type of every element in
         *         this list
         * @throws NullPointerException if the specified array is null
         */
        @SuppressWarnings("unchecked")
        public <T> T[] toArray(T[] a) {
            // 如果传入的数组长度不够，则返回一个刚好大小的的新数组
            if (a.length < size)
                // Make a new array of a's runtime type, but my contents:
                return (T[]) Arrays.copyOf(elementData, size, a.getClass());
            
            // 长度够，则进行深度拷贝
            System.arraycopy(elementData, 0, a, 0, size);
            
            // 如果传入的数组空间有剩余的话，则有用元素的后一位置为 null，
            // 这样做是可以帮助使用者判断数组的大小，可以看作是提供一个完结标识
            if (a.length > size)
                a[size] = null;
            return a;
        }
    ```

