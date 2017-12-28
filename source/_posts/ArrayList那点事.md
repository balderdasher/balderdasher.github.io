---
title: ArrayList那点事
date: 2017-12-28 18:16:07
tags: [java,集合]
categories: [java]
permalink: things-about-arraylist
---
![ArrayList继承结构](/uploads/notes/ArrayList.png)

<!--more-->

## 简单概述一下ArrayList这种数据结构 ##
> `ArrayList`是Java集合框架中`List`分支的一个具体实现类，属于元素**有序**、**可重复**、**可为null**的序列。它有用List接口的通用功能：
> - 可以精确控制元素插入列表中的位置
> - 可以通过元素在列表中的下标访问元素
> - 可以在列表中查找元素

## 如何理解ArrayList的有序性？ ##
ArrayList的有序性基于其底层的数据结构*数组*，数组是内存中一块连续的空间，list中的元素就存储在这个数组之上，数组本身是连续的，而且ArrayList插入或获取元素时可以指定元素所在的位置，这个位置映射到数组的下标，因此我们可以控制元素在列表中的顺序，所以说ArrayList是有序的

至于列表中元素的可重复性，是因为数组的数据结构决定了其中的元素之间是没有相互依赖关系的，所以不同位置的元素可以重复。

## ArrayList的初始容量是多少？ ##
假如有定义`ArrayList<String> list = new ArrayList<>();`
> 问：此时list的容量是多少？

正确的解答应该是`0`而不是类中定义的`DEFAULT_CAPACITY = 10`，请看如下源码：
```java
/**
 * Shared empty array instance used for empty instances.
 */
private static final Object[] EMPTY_ELEMENTDATA = {};
/**
 * The array buffer into which the elements of the ArrayList are stored.
 * The capacity of the ArrayList is the length of this array buffer. Any
 * empty ArrayList with elementData == EMPTY_ELEMENTDATA will be expanded to
 * DEFAULT_CAPACITY when the first element is added.
 */
private transient Object[] elementData;
/**
 * Constructs an empty list with an initial capacity of ten.
 */
public ArrayList() {
    super();
    this.elementData = EMPTY_ELEMENTDATA;
}
```
可以看出，通过无参构造器初始化一个`ArrayList`时，实际存储元素的`elementData`是一个空数组，并没有分配空间，所以此时list容量为0.

其实`elementData`的注释已经说明：**刚通过无参构造器初始化时的`ArrayList`在第一个元素被添加时才扩充其容量为默认的10**。

这种设计无疑是合理的，否则当有大量的ArrayList被初始化而没有被使用时，将会浪费一些没有被实际利用的内存空间。这种设计将内存的分配延迟到了第一次向列表中添加元素的时机。

由此可见，无参构造器`ArrayList()`上的注释是不准确的。

## ArrayList的扩容机制是什么样的？ ##
当ArrayList达到容量上限，也就是底层的数组没有多余的空间存储新元素时，ArrayList将会扩容，扩容操作一般发生在向列表中添加元素时，扩充的容量为原来的一半，即扩充后的容量为原来容量的`1.5`倍。相当于重新分配了一个较长的数组，然后把之前数组上的元素都复制到新的数组中去。
```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);//1.5倍
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```
