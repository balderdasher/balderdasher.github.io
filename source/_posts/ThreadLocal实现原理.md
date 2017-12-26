---
title: ThreadLocal实现原理
date: 2017-12-26 15:48:56
tags: [java,threadlocal]
categories: java
permalink: how-thread-local-works
---
> `ThreadLocal`类提供了线程局部(thread-local)变量。通过`get()`或`set()`访问某个变量的每个线程都有自己的局部变量，它独立于变量的初始化副本，`ThreadLocal`实例通常是类中的`private static`字段，用于将某一状态和线程关联。

<!--more-->
## ThreadLocal是如何和线程关联起来的 ##
刚开始使用`ThreadLocal`的时候总会感觉很神奇，通过`set()`设置一下想用的变量，通过`get()`就可以取出来了，而且不同的线程之间相互不影响，线程局部变量，果真是名副其实。不过到底是怎么实现的呢，线程是线程，ThreadLocal是ThreadLocal，我们只使用了`ThreadLocal`就实现了不同的线程持有不同的变量，这些变量是如何与线程关联的呢，因为我们并没有去手动去关联它们。

我们通过查看`Thread`和`ThreadLocal`的源码，看看它们之间到底是通过什么建立起联系的。发现在`Thread`类中有如下定义：

```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```
由注释可知，它们之间的联系就是通过以上这个字段建立联系的。`threadLocals`字段负责保存与当前线程相关联的局部变量，它是`ThreadLocal`中的一个静态内部类`ThreadLocalMap`，ThreadLocalMap是支撑ThreadLocal实现的数据结构，类似于常用的`HashMap`，我们使用`ThreadLocal`其实就是通过设置这个map去维护线程局部变量的。此map中存储的键值对定义如下：

```java
static class Entry extends WeakReference<ThreadLocal> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal k, Object v) {
        super(k);
        value = v;
    }
}
```
由定义可知，这个键值对的键是`ThreadLocal`对象，值则是我们想要设置的具体变量值。

所以，线程与`ThreadLocal`的关系通俗来说就是：ThreadLocal就是线程局部变量，一个线程可以设置多个局部变量，也就是多个`ThreadLocal`，线程通过一个Map结构去维护我们设置的多个`ThreadLocal`，在这个map结构中，我们通过把先前设置的`ThreadLocal`对象用作键就可以取到具体的线程局部变量了。

## ThreadLocal是如何维护线程局部变量的 ##

通过之前的分析知道线程局部变量是由`ThreadLocal`类维护的，此类是如何维护当前线程的局部变量的呢，主要通过两个方法分析：`set()`、`get()`。

```java
public class ThreadLocalTest {
    public static ThreadLocal<String> userName = new ThreadLocal<>();
    public static ThreadLocal<Integer> userLevel = new ThreadLocal<>();

    static class Processor implements Runnable {
        private String id;

        public Processor(String id) {
            this.id = id;
        }

        @Override
        public void run() {
            userName.set("user_" + id);
            userLevel.set(new Random().nextInt(6));
            System.out.println(Thread.currentThread().getName() + " local vars:{" + "username:" + userName.get() + ",userlevel:" + userLevel.get() + "}");
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            String id = i + 1 + "";
            new Thread(new Processor(id), "thread_" + id).start();
        }
    }
}
```
输出：
> thread_1 local vars:{username:user_1,userlevel:0}
thread_2 local vars:{username:user_2,userlevel:3}
thread_3 local vars:{username:user_3,userlevel:4}
thread_4 local vars:{username:user_4,userlevel:5}
thread_5 local vars:{username:user_5,userlevel:3}

以上的代码演示了`ThreadLocal`的基本使用，通常我们最多的操作就是对变量的设置和获取，对应`set()`和`get()`方法，前者在设置变量时使用，代码如下：

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```
很简短的代码，联系上面`ThreadLocal`的使用demo，其逻辑为：当我们在某处设置ThreadLocal时，获取当前所在的线程，并从中取出当前线程的局部变量map，map存在则设置局部变量为当前值，否则创建新的map并设置变量值然后把此map设置为当前线程的局部变量表`threadLocals`(ThreadLocal和Thread建立联系)。

在此之后的`get()`操作就容易理解了:
```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null)
            return (T)e.value;
    }
    return setInitialValue();
}

private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}

protected T initialValue() {
    return null;
}
```

也是获取当前线程，然后从中取出局部变量表`threadLocals`，若不为空(设置过变量)则从变量表中通过用当前的`ThreadLocal`对象作为键取出对应的变量值。如果之前没有设置过局部变量，则返回初始化值（默认`null`）。这个初始化值是由`initialValue()`方法决定的，在初始化`ThreadLocal`时可以重写此方法以设置默认值。


