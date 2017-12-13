---
title: Java中的几种引用
tags: [java]
categories: java
permalink: references-in-java
---

> 在`jdk 1.2`之前的Java中，若一个对象不被任何变量所引用，那么程序就无法在使用这个对象。也就是说只有对象只有在`可触及状态（reachable）`下才能被程序所使用。从jdk 1.2开始增加了`java.lang.ref`包，在原来只有强引用的基础上新增了`软引用(SoftReference)`、`弱引用(WeakReference)`、`虚引用(PhantomReference)`、`final引用(FinalReference)`4种对象引用类型。

## Reference类 ##

### 概览 ###
> 泛型类`Reference`是几种新增引用类型的抽象基类，因为引用对象是通过与垃圾回收器的密切合作来实现的，所以不能直接为此类创建子类。

以上是此类的API文档上说的，乍一看很是费解，怎么就不能直接为此类创建子类了，上面提到的几种引用不都是直接继承的它吗，这话到底是什么意思呢？

其实`Reference`类不能直接创建子类的意思可以理解为我们在写代码的时候不能直接继承它，因为`Reference`的直接子类(软、弱、虚、final)都是由jvm定制化处理的，在代码中直接继承于它没有任何作用，只能继承或使用它的子类。

### 构造函数 ###

```java
Reference(T referent) {
    this(referent, null);
}

Reference(T referent, ReferenceQueue<? super T> queue) {
    this.referent = referent;
    this.queue = (queue == null) ? ReferenceQueue.NULL : queue;
}
```
Reference类共有两个构造函数，都表示构造一个指向`T`类型对象的引用。其区别在于带不带`ReferenceQueue`类型的`queue`参数，带queue的意义在于我们可以在外部对这个queue进行监控，如果有对象即将被回收，那么相应的`Reference`对象就会被放入这个queue中，然后我们可以拿到引用对象做进一步的处理。

构造函数中`T`类型的参数`referent`表示这个引用所指向的对象，这个对象即将被回收意味着此对象除了被指向它的`reference`引用之外没有其它引用了（并非真没有，而是gcRoot可达性分析之后确定为不可达，以避免循环引用问题）。

构造函数中`ReferenceQueue`类型的`queue`参数即是`T`对象即被回收时所要通知的队列,当对象即被回收时,整个reference对象(而不是被回收的对象)会被放到queue里面,然后外部程序即可通过监控这个queue拿到相应的数据了。

如果没有queue参数，就只有不断轮询Reference对象，通过判断里面的`get()`是否返回`null`（虚引用不能这么做，因为它的get始终返回null，因此它只有带queue的构造函数）来判断它所指向的对象是否被回收。这两种方法都有相应的使用场景，如`WeakHashMap`选择去查询queue来判断是否有对象将被回收，而`ThreadLocalMap`则采用判断`get()`是否为null来做处理。

### ReferenceQueue ###

从名字上来看，`ReferenceQueue`似乎是一个队列，但其内部并没有实际的存储结构，它的存储是依赖`Reference`节点之间关系，因为Reference中的`next`字段存储了它的下一个节点，所以多个Reference形成了一个链表，ReferenceQueue可以看做是这个链表的容器，不过它只存储链表的头结点(`head`)。

### Reference的状态 ###

每个引用对象都有自己相应的状态值，即描述自身及引用所指示对象`T`当前处于什么样的状态，方便进行查询、定位或处理。

- `Active`：活动状态，即引用所指示的对象处于强引用状态，还没有被回收，此状态下引用对象不会被放入引用队列中（如果定义时注册了引用队列）,新创建的引用对象处于活动状态
- `Pending`：等待状态，准备放入引用队列中，在此状态下要处理的引用对象将排队等到放入引用队列中，在这个时间窗口，相应的引用对象为pending状态，进入到此状态的任何reference可认为`next`为自身（jvm设置），创建时未注册引用队列的引用对象将永远不会是等待状态
- `Enqueued`：进入状态，引用对象所指向的对象已经为等待回收，并且相应的引用对象已经放到引用队列中了，准备由外部线程来从queue中获取相应数据，此状态下，next为下一个要处理的对象，queue为特殊标识对象`ENQUEUED`，创建时未注册引用队列的引用对象将永远不可能是进入状态
- `Inactive`：不活动状态，即此引用对象已经由外部从引用队列中获取到并且已经处理掉了，意味着此引用对象指向的对象T可以被回收，并且引用对象本身reference也可以被回收了，所以进入此状态的引用对象肯定是应该被回收掉的。一旦一个引用对象进入此状态，那么它的状态将不会再改变了。

jvm不需要定义相应的状态值来判断引用处于哪个状态，而是通过计算`next`和`queue`进行判断。

## Reference实现原理 ##

当使用几种类型的Reference时，只要初始化了一个`Reference`对象`ref`，Reference内部的静态块代码就会创建并启动处理引用的线程`ReferenceHandler`就会自动运行，这是一个最高优先级的守护线程，此线程做的事就是一直运行处理`Pending`状态的引用，如果发现合适的引用则将其放入注册的引用队列。其实现如下：

```java
static boolean tryHandlePending(boolean waitForNotify) {
    Reference<Object> r;
    Cleaner c;
    try {
        synchronized (lock) {
            if (pending != null) {// 如果有pending状态的引用
                r = pending;// 拿到pending状态的引用
                // 'instanceof' might throw OutOfMemoryError sometimes
                // so do this before un-linking 'r' from the 'pending' chain...
                c = r instanceof Cleaner ? (Cleaner) r : null;
                // 将此引用从pending链断开
                pending = r.discovered;
                r.discovered = null;
            } else {
                // The waiting on the lock may cause an OutOfMemoryError
                // because it may try to allocate exception objects.
                if (waitForNotify) {
                    lock.wait();
                }
                // retry if waited
                return waitForNotify;
            }
        }
    } catch (OutOfMemoryError x) {
        // Give other threads CPU time so they hopefully drop some live references
        // and GC reclaims some space.
        // Also prevent CPU intensive spinning in case 'r instanceof Cleaner' above
        // persistently throws OOME for some time...
        Thread.yield();
        // retry
        return true;
    } catch (InterruptedException x) {
        // retry
        return true;
    }

    // Fast path for cleaners
    if (c != null) {
        c.clean();
        return true;
    }
	// 若注册了引用队列则放入其中，进入enqueued状态
    ReferenceQueue<? super Object> q = r.queue;
    if (q != ReferenceQueue.NULL) q.enqueue(r);
    return true;
}
```
其中的`discovered`,表示要处理的对象的下一个对象.即可以理解要处理的对象也是一个链表,通过discovered进行排队,这边只需要不停地拿到pending,然后再通过discovered不断地拿到下一个对象即可.因为这个pending对象,两个线程都可能访问,因此需要加锁处理.

引用从pending状态到enqueued是通过`queue.enqueue(r)`操作，这是引用队列中的一个方法，只由Reference类调用，来看看里面都做了什么：

```java
boolean enqueue(Reference<? extends T> r) { /* Called only by Reference class */
    synchronized (lock) {
        // 获得锁之后检查引用是否已经被加入过或者已经被移除
        ReferenceQueue<?> queue = r.queue;
        if ((queue == NULL) || (queue == ENQUEUED)) {
            return false;
        }
        assert queue == this;
        r.queue = ENQUEUED;// 设置已入队状态
        r.next = (head == null) ? r : head;//后进先出的队列，后进来的引用先被取出处理
        head = r;
        queueLength++;
        if (r instanceof FinalReference) {
            sun.misc.VM.addFinalRefCount(1);
        }
        lock.notifyAll();//发布通知可以从中取出引用处理了
        return true;
    }
}
```
