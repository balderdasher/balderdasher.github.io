---
title: LinkedList那点事
date: 2017-12-29 14:59:33
tags: [java,集合]
categories: java
permalink: things-about-linkedlist
---
![LinkedList继承结构](uploads/notes/LinkedList.png)

<!--more-->

LinkedList是`List`接口的双向链表实现，允许容纳包括null在内的所有引用类型对象的元素，对元素的获取始终需要从链表头开始遍历直到链表尾部，所以`LinkedList`是顺序访问类型的容器，基于数组的`ArrayList`则是通过下标就可以直接定位访问元素的随机访问，这点从类的继承结构中也可以看出：

- ArrayList实现了`RandomAccess`接口，提供随机访问的能力
- LinkedList则继承了`AbstractSequentialList`,提供顺序访问的能力
- LinkedList同时还实现了`Deque`接口，可以当做双端队列使用

## LinkedList底层数据结构是什么？ ##

如同其名字一样，LinkedList的实现基于**双向链表**，所谓的双向链表是针对单向链表而言，单向链表中的一个节点只有指向下一个节点的链，而双向链表中的每个节点都有两条链，一条指向它的前驱节点，一条指向它的后继节点。
![双向链表](/uploads/notes/double-link.png)

如上图所示，添加元素时其实就是新增了链表的一个结点，结点中的`item`存储了实际的元素，然后通过向前和向后的链接使容器中的多个元素形成一个链表。结点定义如下：

```java
private static class Node<E> {
    E item;		// 实际存储的数据
    Node<E> next;	// 前一个结点引用
    Node<E> prev;	// 后一个结点引用

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

##  ##