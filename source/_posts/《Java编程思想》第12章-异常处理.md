---
title: 《Java编程思想》第12章-异常处理
date: 2017-05-20 10:48:33
tags: [java,读书笔记,异常]
categories: java
permalink: thinking-in-java-exceptions
---
**java的基本理念是：结构不佳的代码不能运行**，
异常处理能够保持程序及系统的健壮性。
<!--more-->
## 12.1 基本异常 ##

**异常情形**是指阻止当前方法或域继续执行的问题。异常情形和普通问题的区别就是普通问题在当前环境下能够获得足够的信息去处理错误，而异常情形在当前环境下无法获得必要的信息来解决问题，所以不能继续下去。所能做的只是从当前环境跳出，并把问题交给上一级环境，这就是**抛出异常时发生的事情**。

### 抛出异常后发生了什么？ ###

- 首先，java将使用`new`在堆上创建异常对象。
- 然后，当前的执行路径被终止，并且从当前环境中弹出对异常对象的引用
- 与此同时，异常处理机制接管程序，并开始寻找一个恰当的地方来继续执行程序。

以上的*恰当的地方*指的就是`异常处理程序`，它的任务是将程序从错误状态中恢复，以使程序要么换一种方式运行，要么继续运行下去。

一个简单的抛出异常的例子：对象引用`t`,进行传递的时候可能尚未被初始化，这时候调使用它调用其方法之前，会先进行检查，这时候可以创建一个代表错误信息的对象，并且从当前环境中“抛出”，这样就把错误信息传播到了“更大”的环境中，这个过程被称为“***抛出一个异常***”，像下面这样：

```java
if(t == null){
 throw new NullPointerException();
}
```
这样就抛出了异常，于是在当前环境下就不必操心这个问题了，它将在别的地方得到处理。

### 12.1.2 异常参数 ###

与java中的其他对象一样，我们用`new`在堆上创建异常对象，这伴随着存储空间的分配和构造器的调用。**所有标准异常类都有两个构造器**：一个是默认构造；另一个是接收字符串作为参数，以便能把相关信息放入异常对象的构造器：`throw new NullPointerException("t = null")`。

可以抛出任意类型的`Throwable`对象，它是异常类型的根类。

## 12.2 捕获异常 ##

### 12.2.1 try块 ###
处于`try`块内的代码称为**监控区域**

### 12.2.2 异常处理程序 ###

处理抛出异常的地点称为“异常处理程序”，针对每个捕获的异常都应该有相应的处理程序，异常处理程序紧跟try块之后，以关键字`catch`表示：

## 12.3 创建自定义异常 ##

自定义异常的方式通常是继承java已有的异常类

只有默认构造器的简单自定义异常：
```java
public class SimpleException extends Exception {
}

public class InheritingException {
    public void f() throws SimpleException{
        System.out.println("Throw SimpleException from f()");
        throw new SimpleException();
    }

    public static void main(String[] args) {
        InheritingException sed = new InheritingException();
        try {
            sed.f();
        } catch (SimpleException e) {
            System.out.println("Caught it!");
        }
    }
}
```
输出：
```
Throw SimpleException from f()
Caught it!
```

带字符串参数构造器的自定义异常：

```java
/**
 * 带字符串参数构造器的自定义异常
 */
public class MyException extends Exception {
    public MyException() {
    }

    public MyException(String message) {
        super(message);
    }
}

public class FullConstructorsExcp {
    public static void f() throws MyException {
        System.out.println("Throwing MyException from f()");
        throw new MyException();
    }

    public static void g() throws MyException {
        System.out.println("Throwing MyException from g()");
        throw new MyException("Originated in g()");
    }

    public static void main(String[] args) {
        try {
            f();
        } catch (MyException e) {
            e.printStackTrace(System.out);
        }
        try {
            g();
        } catch (MyException e) {
            e.printStackTrace(System.out);
        }
    }
}
```
输出：
```
Throwing MyException from f()
com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter12.MyException
	at com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter12.FullConstructorsExcp.f(FullConstructorsExcp.java:10)
	at com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter12.FullConstructorsExcp.main(FullConstructorsExcp.java:20)
Throwing MyException from g()
com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter12.MyException: Originated in g()
	at com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter12.FullConstructorsExcp.g(FullConstructorsExcp.java:15)
	at com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter12.FullConstructorsExcp.main(FullConstructorsExcp.java:25)
```

## 12.4 受检异常和未受检异常 ##

java的异常分为两大类：

1. checkedexception（受检异常）
2. unchecked exception（未受检异常）

未受检异常也叫运行时异常（`RuntimeException`），java编译器不要求一定要捕获或一定要抛出未受检异常，但是对于受检异常，java要求必须在方法里捕获或者继续抛出

对于未受检异常，有以下几种处理方式：
- 捕获
- 继续抛出
- 不处理

对于受检异常，处理方式有：
- 继续抛出（消极的做法）
- 用`try...catch`捕获

### 12.4.1 常见异常 ###

未受检异常：
- Exception
- FileNotFoundException
- IOException
- SQLException

受检异常：
- NullPointerException
- ClassCastException
- ArrayIndexsOutOfBoundsException
- ArithmeticException(算数异常)
