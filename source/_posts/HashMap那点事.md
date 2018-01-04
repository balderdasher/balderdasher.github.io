---
title: HashMap那点事
date: 2018-01-03 15:52:57
tags: [java,集合]
categories: java
permalink: things-about-hashmap
---
![HashMap继承机构](/uploads/notes/HashMap.png)
<!--more-->

HashMap是基于哈希表的 `Map` 接口的实现。此实现提供所有可选的映射操作，并允许使用 null 值和 null 键。（除了非同步和允许使用 null 之外，HashMap 类与 Hashtable 大致相同。）此类不保证映射的顺序，特别是它不保证该顺序恒久不变。
## HashMap的数据结构是什么？ ##
HashMap底层是*链表散列*的数据结构，即数组和链表的结合体，底层是一个存放`Entry`的数组，`Entry`又可能是链表中的一个结点，其定义如下：
```java
/** 
 * The table, resized as necessary. Length MUST Always be a power of two. 
 */  
transient Entry[] table;  
static class Entry<K,V> implements Map.Entry<K,V> {  
    final K key;  
    V value;  
    Entry<K,V> next;  
    final int hash;  
    ……  
}  
```
![HashMap数据结构](/uploads/notes/HashMap-data-structure.jpg)
## 影响HashMap性能的参数是什么？ ##
`HashMap` 的实例有两个参数影响其性能：*初始容量* 和*加载因子*。**容量** 是哈希表中桶的数量(数组的长度)，初始容量只是哈希表在创建时的容量。*加载因子* 是哈希表在其容量自动增加（扩容）之前可以达到多满的一种尺度。当哈希表中的条目数超出了**加载因子与当前容量的乘积**时，则要对该哈希表进行 `rehash`(扩容) 操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数。

## HashMap的容量为什么一定要是2的次幂？ ##
HashMap的容量即底层数组的长度`length`,`length`将被用于计算元素将被放入数组的位置，计算方法如下：
```java
static int indexFor(int h, int length) {  
    return h & (length-1);  
} 
```
取模运算`h % length`确保元素的分布相对是比较均匀的，当`length`是2的次幂时，`h & (length-1)`运算等价于取模运算`h % length`，但是`&`（与）运算比`%`（取模）运算具有更高的效率。同时，当length为2的次幂时，通过计算可以是元素获得更均匀的分布。

假设数组长度为15和16，重新计算后的hash分别为8和9，那么通过`indexFor`散列之后的元素分布结果如下：

table.length|h & (table.length-1)|hash|table.length-1|index
---|---|---|---|---
15|8 & (15-1)|1000|1110|1000(8)
15|9 & (15-1)|1001|1110|1000(8)
16|8 & (16-1)|1000|1111|1000(8)
16|9 & (16-1)|1001|1111|1001(9)

从上表中可以看出，当length等于15时，不同的hash值8和9都被定位到了数组中的同一个位置8，产生了碰撞，而且此时的运算结果最后一位永远是0，所以0001，,011,1001,1011，0111,1101等位置将永远都不能存放元素了，空间浪费很大，也进一步增加了碰撞的几率，降低了查询效率。

相反，当数组长度为16时，参与运算的length-1的二进制数每个位的值都为1，使得在低位与时，得到的值和原hash的低位相同，因此不同的hash值8和9被定位到了不同的位置上。因此当数组长度为2的次幂时，不同的key被定位到相同index的几率较小，数据在数组上的分布比较均匀，使得HashMap的查询效率得以提高。

## 如何处理大量的hash碰撞攻击？ ##
此种攻击最常见的是POST数据攻击，构造一个含有大量碰撞key的post请求达到攻击的目的。
- 控制post数据的大小
- 限制http请求body的大小和参数的数量
- 限制每个桶链表的最大长度
- 使用其它数据结构如红黑树取代链表组织碰撞哈希（并不解决哈希碰撞，只是减轻攻击影响，将N个数据的操作时间从O(N^2)降至O(NlogN)，代价是普通情况下接近O(1)的操作均变为O(logN)）

## jdk8中HashMap实现有什么变化 ##

jdk8中`HashMap`实现的底层结构有所变化，当桶上的链表长度大于8时，链表将转变为红黑树结构，一定程度上减轻了hash碰撞攻击的影响，优化了元素的查找效率。





