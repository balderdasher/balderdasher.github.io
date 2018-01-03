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
## 影响HashMap性能的参数是什么？ ##
`HashMap` 的实例有两个参数影响其性能：*初始容量* 和*加载因子*。**容量** 是哈希表中桶的数量(数组的长度)，初始容量只是哈希表在创建时的容量。*加载因子* 是哈希表在其容量自动增加（扩容）之前可以达到多满的一种尺度。当哈希表中的条目数超出了**加载因子与当前容量的乘积**时，则要对该哈希表进行 `rehash`(扩容) 操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数。

## HashMap的容量为什么一定要是2的次幂？ ##

## 如何处理大量的hash碰撞攻击？ ##



