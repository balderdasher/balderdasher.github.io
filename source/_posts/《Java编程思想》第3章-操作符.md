---
title: 《Java编程思想》第3章-操作符
date: 2017-05-09 12:13:06
tags: [java,读书笔记]
categories: java
permalink: thinking-in-java-operator
---

## 赋值 ##

　　基本类型存储了实际的值，而并非指向一个对象的引用，所以在为其赋值的时候，是直接将内容从一个地方复制到另一个地方，如对基本类型使用a=b,那么b的内容就复制给了a，如果接着修改了b，则a根本就不会受到影响，但是在为对象"赋值"的时候，真正赋予的是对象的引用，若将一个对象赋值给另一个对象，实际是将引用从一个地方复制到另一个地方，意味着假如使用c=d,那么c和d都指向原本只有d指向的对象

<!-- more -->

## Java中的参数传递 ##

　　java中的参数传递分两种情况：基本类型的参数传递和引用类型的参数传递，基本类型的参数传递是传递这个值的拷贝，无论怎么改变这个值，原值不受影响；应用类型的参数传递其实传递的是内存地址，传递之前的值是否改变取决于传递之后所指向的内存地址中的对象是否改变。

　　**总结**：java在执行参数传递时，从内存模型上分析：基本类型会开辟一块新的堆内存地址并赋值为所传递基本类型值的拷贝，因此参数所指向的堆内存地址和之前不是指向同一内存地址，所以无论怎么改变传递之后的值原值都不会改变，而对象类型的参数传递不会开辟新的内存地址，参数的引用和原值的引用指向同一地址，所以修改传递之后的值会影响传递之前的值，基本类型传递的是原值的一个副本值，而引用类型传递的是原值的内存地址，因此说java的参数传递只有值传递，基本类型和对象的参数传递根本区别就在于是否开辟新的堆内存地址，对象类型的参数传递之后原值是否改变取决于传递参数之后，参数变量的引用地址中的对象是否被修改（原值改变）或者是否指向了新的对象（因为是两个不同的引用地址，所以原值不变）。

>ps：String类型虽然是对象类型，但是作为参数传递时，表现和基本类型一致。

- [参考一](http://www.360doc.com/content/15/0507/20/7673502_468817265.shtml)

- [参考二](http://blog.sina.com.cn/s/blog_59ca2c2a0100qhjx.html)

----------

## 移位操作符 ##

> 符号

- 左移：<<
- 右移：>>

> 公式

- `m<<n=m*2ⁿ`
- `m>>n=m/2ⁿ`

> 描述

- 左移相当于`左边的数`*`2`的`右边数`次方
- 右移相当于`左边的数`/`2`的`右边数`的次方

> 备注

- 右移时由于是取整运算，所以如`3>>2`=0、`5>>2`=1

### 示例

```java
public class MoveBitOpt {
    public static void main(String[] args) {
        int i = 4;
        int pos = 2;
        System.out.println("before move: " + Integer.toBinaryString(i));
        moveLeft(i, pos);
        moveRight(i, pos);
    }

    /**
     * 左移
     *
     * @param i
     * @param pos
     */
    public static void moveLeft(int i, int pos) {
        i <<= pos;
        System.out.println("after left move: " + Integer.toBinaryString(i));
        System.out.println(i);
    }

    /**
     * 右移
     *
     * @param i
     * @param pos
     */
    public static void moveRight(int i, int pos) {
        i >>= pos;
        System.out.println("after right move: " + Integer.toBinaryString(i));
        System.out.println(i);
    }
}
```

输出:

```java
before move: 100
after left move: 10000
16
after right move: 1
1
```
