---
title: Java中的引用Reference
tags: [java]
categories: java
permalink: references-in-java
---

> 在`jdk 1.2`之前的Java中，若一个对象不被任何变量所引用，那么程序就无法在使用这个对象。也就是说只有对象只有在`可触及状态（reachable）`下才能被程序所使用。从jdk 1.2开始增加了`java.lang.ref`包，此包提供了引用对象类，支持在某种程度上与垃圾回收器之间的交互。程序可以使用一个引用对象来维持对另外某一对象的引用，所采用的方式是使后者仍然可以被回收器回收。程序还可以安排在回收器确定某一给定对象的可到达性已经更改之后的某个时间得到通知。
> 在原来只有强引用的基础上新增了`软引用(SoftReference)`、`弱引用(WeakReference)`、`虚引用(PhantomReference)`3种对象引用类型。
<!-- more -->
## 引用包规范 ##

*引用对象*`Reference`封装了对另一个对象`T`的引用，这样就可以像其他任何对象一样检查和操作引用自身。有三种类型的引用对象，按从弱到强依次为： *软* 引用、 *弱* 引用和 *虚* 引用。正如下面定义的那样，每种类型对应于一个不同的可到达性级别。软引用适用于实现内存敏感的缓存，弱引用适用于实现无法防止其键（或值）被回收的规范化映射，而虚引用则适用于以某种比 Java 终结机制更灵活的方式调度 `pre-mortem` 清除操作。

每种引用对象类型都是通过抽象的基本 `Reference` 类的一个子类实现的。其中一个子类的实例封装了对特定对象的引用，该对象名为指示对象。每个引用对象都提供了获取和清除该引用的方法。引用对象是不可变的，因此，除了清除操作之外，没有提供 set 操作。通过添加任何所需的字段和方法，程序可以为这些子类进一步创建子类，或者可以不加更改地使用这些子类。

### 通知 ###

在创建引用对象时，通过向 *引用队列* 注册 一个适当的引用对象，程序可以请求在对象可到达性更改时获得通知。在垃圾回收器确定引用的可到达性已经更改为对应于引用类型的值之后的某一时间，它会将引用添加到相关的队列中。此时，该引用被认为是 *已加入队列的*（`Enqueued`）。通过轮询或阻塞，直到获得了引用，程序才可以从队列中移除引用。引用队列是通过 `ReferenceQueue` 类实现的。

已注册的引用对象及其队列之间的关系是单向的。也就是说，队列不会追踪那些向它注册的引用。如果一个已注册的引用本身变得不可到达，则永远不会将它加入到队列中。使用引用对象的程序的责任是，只要程序对其指示对象感兴趣，就要确保对象是可达到的。

虽然某些程序会选择专门使用一个线程从一个或多个队列中移除引用对象并处理它们，但这是绝对没有必要的。一种通常很有用的策略是：在执行另一个相当频繁的操作期间检查引用队列。例如，使用弱引用来实现弱键的哈希表能在每次访问表时轮询其引用队列。这就是 `WeakHashMap` 类的工作方式。因为 `ReferenceQueue.poll()` 方法仅仅检查内部数据结构，此检查只为哈希表访问方法增加了很小的系统开销。

### 自动清除引用

在将*软引用*和*弱引用*添加到向其注册的队列（如果有）之前，回收器将自动清除这些引用。所以，软引用和弱引用不需要向队列注册即可使用，而虚引用则需要这样做。通过虚引用可到达的对象将仍然保持原状，直到清除所有这类引用或者它们本身变得不可到达。

### 可到达性

从最强到最弱，不同的可到达性级别反映了对象的生命周期。在操作上，可将它们定义如下：

- 如果某一线程可以不必遍历所有引用对象而直接到达一个对象，则该对象是*强可到达* 对象。新创建的对象对于创建它的线程而言是强可到达对象。
- 如果一个对象不是强可到达对象，但通过遍历某一软引用可以到达它，则该对象是*软可到达* 对象。
- 如果一个对象既不是强可到达对象，也不是软可到达对象，但通过遍历弱引用可以到达它，则该对象是弱可到达 对象。当清除对某一弱可到达对象的弱引用时，便可以终止此对象了。
- 如果一个对象既不是强可到达对象，也不是软可到达对象或弱可到达对象，它已经终止，并且某个虚引用在引用它，则该对象是*虚可到达* 对象。
- 最后，当不能以上述任何方法到达某一对象时，该对象是*不可到达* 对象，因此可以回收此对象。

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

### 辅助类ReferenceQueue ###

引用队列`ReferenceQueue`类是使用几种引用时的辅助类，它的作用是在使用几种引用时方便监控我们所创建的引用的状态变化，如果构造一个指向`T`对象的引用时指定了一个引用队列，那么此队列将监控所有`T`类型对象的可达性状态变化，当gc决定此对象可达性改变时，相应的引用将会进入队列，我们可以从队列中取出此引用以便在对象回收之前做出处理。

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
        lock.notifyAll();//发布通知可以从中取出引用处理了如remove方法
        return true;
    }
}
```
当一个引用进入队列之后，说明这个引用所指向的对象将要被回收了，这时候我们可以从队列中取出引用以便进行回收前的一些处理，从队列中取出引用的操作如下：

```java
public Reference<? extends T> poll() {
    if (head == null)
        return null;//没有将被回收的对象，返回null
    synchronized (lock) {
        return reallyPoll();//否则链表操作加锁并从中取出引用
    }
}

private Reference<? extends T> reallyPoll() {       /* Must hold lock */
    Reference<? extends T> r = head;
    if (r != null) {// 和放入队列的逻辑相同：取出头引用处理并重新设置头
        head = (r.next == r) ?
            null :
            r.next; // Unchecked due to the next field having a raw type in Reference
        r.queue = NULL;//确保取出的引用不会再被放入队列
        r.next = r;
        queueLength--;
        if (r instanceof FinalReference) {
            sun.misc.VM.addFinalRefCount(-1);
        }
        return r;
    }
    return null;
}
```
除了`poll()`方法可以从引用队列中取出将要被回收的对象引用外，还有`remove()`、`remove(long timeout)`方法具有相同的功能，它们相当于轮询版的`poll()`，这个两个方法会一直阻塞直到有引用被放入队列中或者超时时间已过。

remove方法在`Finalizer`中有相应的应用场景，此类是jvm专用用来处理对象在被回收时执行对象中`finalize()`方法的，自动启动一个线程不断从注册的队列中取出引用并在回收指向对象时执行其`finalize()`方法。

## 几种引用 ##

1. `SoftReference`：软引用，这种引用会根据内存需求任凭被垃圾回收器清除，即内存充足时不会被回收，反之被回收，因此软引用常被用来实现内存敏感的缓存。jvm会保证所有指向软可达对象的软引用在抛出`OOM`之前被清除。
2. `WeakReference`：弱引用，

