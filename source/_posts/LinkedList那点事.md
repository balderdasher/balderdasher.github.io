---
title: LinkedList那点事
date: 2017-12-29 14:59:33
tags: [java,集合]
categories: java
permalink: things-about-linkedlist
---
![LinkedList继承结构](/uploads/notes/LinkedList.png)

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

## LinkedList没用数组结构，它的根据下标获取元素是如何实现的 ##
LinkedList是`List`接口的一个实现，List接口中规定了按照元素在列表中的位置获取元素的方法`E get(int index)`,对于使用数组实现的`ArrayList`来说这是很容易的，因为元素在列表中的位置就是它在数组中的下标。

对于使用链表结构的`LinkedList`来说，每个结点并没有像数组那样存储元素在列表中的位置，所以按位置查找元素时只能按一定的顺序遍历链表直到到达指定位置然后才能取出元素，其实现如下：

```java
public E get(int index) {
    checkElementIndex(index);	//index >= 0 && index < size;
    return node(index).item;
}
/**
 * Returns the (non-null) Node at the specified element index.
 */
Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```
node(int index)实现了按位置查找链表结点，而且经过了一次二分优化，从而把最坏查找时间减少一半：
- 当给定位置位于链表前半部分时从前往后遍历链表
- 当给定位置位于链表后半部分时从后往前遍历链表

## 正确地遍历删除列表中的元素 ##
List集合的相应实现一般都会有`fail-fast`机制，以防止遍历列表时修改列表，List接口的相应实现通过记录列表的修改次数`modCount`来实现，添加，删除元素元素时修改次数递增。

expectedModCount则是表示迭代器对集合进行修改的次数。
设置expectedModCount的目的就是要保证在使用迭代器期间，LinkedList对象的修改只能通过迭代器且只能这一个迭代器进行。

在迭代器中修改集合时，同时修改`modCount`和`expectedModCount`的值，并且在修改之前检查二者是否一致，如果不一致这说明集合已经在其他地方被修改过，此时将抛出`ConcurrentModificationException`异常。所以在使用List时应该注意：

- 确保所有元素都被添加完毕时才使用其迭代器
- 需要遍历修改集合时确保使用迭代器操作集合

针对第二点有个有趣的事情，假如我想删除集合中的所有元素，可不可以用迭代器进行呢？
```java
@Test
public void testIteDel() {
    Iterator<Integer> iterator = list.iterator();
    while (iterator.hasNext()) {
//            Integer item = iterator.next();
            iterator.remove();
    }
    System.out.println(list);
}
```
以上代码运行将抛出`java.lang.IllegalStateException`异常，而放开代码中的注释则可以正常运行，原因是`move()`方法中会检查当前迭代返回值`lastReturned`是否为null，是则抛出此异常，而`lastReturned`只在`next()`和`previous()`方法中赋值，所以在删除前应该调用一下它们，否则会认为当前迭代没有值返回，既然没有值的时候去删除就应该是不合法的状态。

**清空List中的元素还是老老实实地调用现成的`clear()`方法吧**。