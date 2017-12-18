---
title: 《Java编程思想》第11章-持有对象
date: 2017-05-18 10:42:51
tags: [java,读书笔记]
categories: java
permalink: thinking-in-java-hold-object
---
本章介绍了对象的持有技术，其实就是java中的容器，不作详细记录，只记录一些平时忽略掉的重点。
<!--more-->
java中其实只有四种容器：**Map、List、Set**和**Queue**。

- `Iterator`只能前向移动，`ListIterator`只能用于各种List的访问，但是它能双向移动。可以通过调用`listIterator()`方法产生一个指向List开始处的ListIterator，也可以通过`listIterator(n)`方法创建一个一开始就指向列表索引为n的元素处的ListIterator。
- `LinkedList在`执行插入和移除时比`ArrayList`更高效，但是随机访问操作的效率低于`ArrayList`，`LinkedList`还添加了可以使其用作栈、队列或双端队列的方法。
- “栈”是“后进先出”（LIFO）的容器，最后一个“压入”栈的元素第一个“弹出”栈。LinkedList具有直接实现栈功能的所有方法，所以可以将LinkedList作为栈使用。
- HashSet的输出没有规律可循是因为它出于速度原因的考虑使用了`散列`。HashSet与LinkedHashSet或TreeSet都不同，TreeSet将元素存储在`红-黑树`数据接口中，LinkedHashSet也是用了散列，但是看起来它使用了链表来维护元素的插入顺序。
- 队列是一个典型的`先进先出`（FIFO）的容器，从容器一端放入，从另一端取出，并且放入容器的顺序与从容器中取出的顺序是相同的
- LinkedList提供了支持队列的方法，并且实现了`Queue`接口，所以可以用作Queue的一种实现

## 总结 ##
java提供了大量持有对象的方式：

1. 数组将数字和对象联系起来，它保存类型明确的对象，查询对象时，不需要对结果做类型转换，数组时可以多维的，可以保存基本类型的数据。但是，**数组一旦生成，其容量就不能改变**。
2. Collection保存单一的元素，而Map保存相关联的键值对。java的泛型使得可以指定容器中存放的对象类型，防止我们将错误的类型放到容器中，在容器中获取元素时不必进行类型转换。各种Collection和Map都可以在向容器中添加更多容器时自动调整尺寸（`自动扩容`）。容器不能持有基本类型的数据。
3. 像数组一样，List也建立数字索引与对象的联系，因此数组和List都是排好序的容器。List能够自动扩容。
4. 如果要进行大量的随机访问，应该使用`ArrayList`；如果要经常从表中间插入和删除元素，则应该使用`LinkedList`。
5. 各种Queue及栈的行为，由LinkedList提供支持。
6. Map是一种将对象（而非数字）与对象相关联的设计。HashMap设计用来快速访问；而TreeMap保持“键”始终处于排序状态，所以它没有HashMap快。LinkedHashMap保持元素插入的顺序，但是也通过散列提供了快速访问能力。
7. Set不接受重复元素。HashSet提供最快的查询速度，而TreeSet保持元素处于排序状态，LinkedHashSet以插入顺序保存元素。
8. 新程序中不应该使用Vector、Hashtable和Stack。

## java容器简图 ##

![java容器简图](http://i.imgur.com/OZVUh9h.png)
