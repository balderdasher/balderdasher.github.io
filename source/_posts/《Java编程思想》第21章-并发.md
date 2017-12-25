---
title: 《Java编程思想》第21章-并发
date: 2017-06-05 11:11:05
tags: [java,读书笔记]
categories: java
permalink: thinking-in-java-concurrent
---
## 关于并发 ##

**为什么需要并发**

编程问题中的很多部分都可以通过使用顺序编程来解决，顺序编程，就是程序中的所有事物在任意时刻都只能执行一个步骤。然而对于某些问题，如果能够并行地执行程序中的多个部分，会非常高效而且方便，这些部分要么看起来在并发地执行，要么在多处理器的环境下可以同时执行。并行编程可以是程序执行速度得到极大提高。

<!--more-->
**什么是并发**

并发简单地讲，就是并行地执行程序中的多个部分。

## 21.1 基本的线程机制 ##

> 并发编程使我们可以将程序划分为多个分离的、独立运行的任务。通过使用多线程机制，这些独立任务中的每一个都将由**执行线程**来驱动。一个线程就是在进程中的一个单一的顺序控制流，因此，单个进程可以拥有多个并发执行的任务，但是程序使得每个任务都好像有自己的CPU一样，其底层机制是切分CPU时间，我们通常不需要考虑它。
> 线程模型为编程带来了便利，它简化了在单一程序中同事交织在一起的多个操作的处理。在使用线程时，CPU将轮流给每个任务非配其占用时间。每个任务都觉得自己在一直占用CPU，但事实上CPU时间是划分成片段分配给了所有的任务（除非确实运行在多处理器机器上），线程的一大好处就是可以使我们从这个层次抽身而出，代码不必知道它是运行在具有一个还是多个CPU的机器上，所以，使用线程机制是一种建立透明的、可扩展的程序的方法。为什么呢，因为如果程序运行缓慢的话，多加几个CPU就可以加快程序的运行速度。

### 21.1.1 定义任务 ###

线程可以驱动任务，`Runnable`接口提供了一种描述任务的方式，想要定义任务，只需实现`Runnable`接口并编写其中的`run()`方法即可，使得该任务可以执行你的命令，以下是一个显示发射倒计时的任务：

```java
/**
 * 使用Runnable定义任务
 *
 * @author mrdios
 * @date 2016-07-27 14:27
 */
public class LiftOff implements Runnable {
    protected int countDown = 10;
    private static int taskCount = 0;
    private final int id = taskCount++;

    public LiftOff() {
    }

    public LiftOff(int countDown) {
        this.countDown = countDown;
    }

    public String status() {
        return "#" + id + "(" + (countDown > 0 ? countDown : "LiftOff!") + "), ";
    }

    @Override
    public void run() {
        while (countDown-- > 0) {
            System.out.println(status());
            Thread.yield();
        }
    }

    public static void main(String[] args) {
        LiftOff launch = new LiftOff();
        launch.run();
    }
}
```
输出：
```
#0(9), #0(8), #0(7), #0(6), #0(5), #0(4), #0(3), #0(2), #0(1), #0(LiftOff!),
```

任务的`run()`方法通常总会有某种形式的循环，使得任务一直运行下去直到不再需要，所以要设定跳出循环的条件（有种选择是直接从run（）返回）。通常`run()`被写成无限循环的形式，意味着除非某个条件使得`run()`停止，否则它将永远运行下去。

在`run`中对静态方法`Thread.yield()`的调用时对**线程调度器**（Java线程机制的一部分，可以将CPU从一个线程转移给另一个线程）的一种建议，它意思是说：“我已经执行完生命周期中最重要的部分了，此刻正式切换给其他任务执行一段时间的大好时机。”

当实现`Runnable`接口定义一个任务时，必须具有`run()`方法，但这个方法并没有什么特殊之处——它不会产生任何内在的线程能力，要实现线程的行为，必须要显式地将任务附着到线程上。

### 21.1.2 Thread类 ###

正如上面所说，将`Runnable`对象转变为工作任务的传统方式就是把它提交给一个`Thread`构造器：

```java
public class BasicThread {
    public static void main(String[] args) {
        Thread t = new Thread(new LiftOff());
        t.start();
        System.out.println("Waiting for LiftOff");
    }
}
```
输出：
```
#0(9), #0(8), #0(7), #0(6), #0(5), #0(4), #0(3), #0(2), #0(1), #0(LiftOff!),
```

通过程序得知，`Thread`构造器只需要一个`Runnable`对象。调用`Thread`对象的`start()`方法为该线程执行必须的初始化操作，然后调用`Runnable`的run()方法（从输出中可以看出），以便在这个新线程中启动该任务。

观察程序我们会觉得`start()`应该是产生了一个对长期运行方法的调用，`Waiting for LiftOff`这一句应该输出在最后，但输出告诉我们并不是这样的，输出可以看出，start()迅速返回了，因为`Waiting for LiftOff`出现在了第一句。实际上，start（）产生的是对`LiftOff.run()`的方法调用，并且这个方法还没有完成，因为`LiftOff.run()`是在新线程中执行的，所以我们可以执行位于`main()`线程中的其他操作（不局限与main（），任何线程都可以启动另一个线程），因此，程序会同时运行两个方法，`main()`和`LiftOff.run()`是程序中与其他线程“同时”执行的代码。

当我们添加更多地线程去驱动更多的任务时，任务之间的彼此呼应会看得更清楚一些：

```java
/**
 * 更多的线程驱动更多的任务
 *
 * @author mrdios
 * @date 2016-07-27 15:18
 */
public class MoreBasicThreads {
    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            new Thread(new LiftOff()).start();
        }
        System.out.println("Waiting for LiftOff");
    }
}
```
输出：
```
Waiting for LiftOff
 #0(9), #3(9), #1(9), #2(9), #4(9), #0(8), #3(8), #1(8), #2(8), #4(8), #0(7),
 #3(7), #2(7),#4(7), #1(7), #0(6), #3(6), #4(6), #1(6), #2(6), #0(5), #3(5), 
 #4(5), #1(5),#2(5), #0(4),#3(4), #4(4), #1(4), #2(4), #0(3), #3(3), #4(3), 
 #2(3), #1(3), #0(2), #3(2),#4(2), #2(2),#1(2), #0(1), #3(1), #4(1), #2(1), #1(1), 
 #0(LiftOff!), #3(LiftOff!), #2(LiftOff!), #4(LiftOff!), #1(LiftOff!),
```

以上程序的输出可能会每次都不一样，另外也会随着JDK版本的不同而表现出差异，较早的JDK不会频繁对时间切片，因此线程1可能会先执行完然后线程2再执行，较晚的JDK看起来会产生更好的时间切片行为。

### 21.1.3 使用Executor ###

Executor是`java.util.concurrent`包中为我们管理`Thread`对象的**执行器**，Executor在客户端和任务执行之间提供了一个间接层；与客户端直接执行任务不同，这个中介对象将执行任务。Executor允许我们管理异步任务的执行而无须显式地管理线程的生命周期。Executor是Java SE5/6中启动任务的优选方法。

```java
public class CachedThreadPool {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            exec.execute(new LiftOff());
        }
        exec.shutdown();
    }
}
```

以上例子使用Executor实现了发射倒计时，`ExecutorService`是具有服务生命周期的Executor，例如关闭等，它知道如何构建恰当的上下文来执行Runnable对象，`CachedThreadPool`将为每个任务都创建一个线程，`ExecutorService`对象是使用静态的Executor方法创建的，这个方法可以确定Executor的类型。

在最后对`shutdown()`方法的调用目的是为了防止新任务被提交给这个Executor，当前线程将继续运行在`shutdown()`被调用之前所提交的所有任务，这个程序将在Executor中的所有任务完成之后尽快退出。

上述例子中我们明确启动了5个线程来执行不同的任务，这时候我们可以将CachedThreadPool替换为不同类型的Executor，正如上例所示，当我们需要使用有限的线程集来执行所提交任务的时候，通常使用`FixedThreadPool`

使用FixedThreadPool，就可以一次性地预先执行代价高昂的线程分配，所以也就可以限制线程的数量了，这样可以节省时间，因为不用为每个任务都固定地付出创建线程的开销。在事件驱动的系统中，需要线程的事件处理器，通过直接从池中获取线程，也可以得到相应的服务，而且我们不会滥用可获得的资源，因为FixedThreadPool使用的Thread对象的数量是有界的。

注意，在任何线程池中，现有线程在有可能的情况下，都会被自动复用。

`CachedThreadPool`在程序执行过程中通常会创建与所需数量相同的线程，然后在它回收旧线程时停止创建新线程，因此它是合理的Executor的首选，只有当这种方式会引发问题时，才需要考虑切换到FixedThreadPool。

SingleThreadExecutor就好比线程数量为1的FixedThreadPool，这对于希望在另一个线程中连续运行的任何事物（长期存活的任务）来说，都是很有用的。

### 21.1.4 从任务中产生返回值 ###

Runnable是执行工作的独立任务，但是它不产生任何返回值，如果希望任务完成时能够返回一个值，应该实现`Callable`接口而不是`Runnable`接口。`Callable`是一种具有类型参数的泛型，它的类型参数表示的是从方法`call()`(而不是run())中返回的值，并且必须使用`ExecutorService.submit()`方法调用它，下面是一个简单的例子：

```java
/**
 * 需要从任务返回一个值时使用Callable
 * @author mrdios
 * @date 2016-07-28 14:15
 */
public class TaskWithResult implements Callable<String> {
    private int id;

    public TaskWithResult(int id) {
        this.id = id;
    }

    @Override
    public String call() throws Exception {
        return "result of TaskWithResult " + id;
    }

    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        ArrayList<Future<String>> results = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            results.add(exec.submit(new TaskWithResult(i)));
        }
        for (Future<String> fs:results){
            try {
                System.out.println(fs.get());
                //java7以上特性，可以对一个或多个异常做相同的处理
            } catch (InterruptedException | ExecutionException e) {
                System.out.println(e);
            } finally {
                exec.shutdown();
            }
        }
    }
}
```
输出：
```
result of TaskWithResult 0
result of TaskWithResult 1
result of TaskWithResult 2
result of TaskWithResult 3
result of TaskWithResult 4
result of TaskWithResult 5
result of TaskWithResult 6
result of TaskWithResult 7
result of TaskWithResult 8
result of TaskWithResult 9
```

如上所示，`submit()`方法会产生`Future`对象，它用`Callable`返回结果的特定类型进行了参数化。可以用`isDone()`方法来查询`Future`是否已经完成。当任务完成时，可以调用`get()`方法来获取结果，事实上也可以不用调用`isDone()`检查就直接调用`get()`，这时候get()将阻塞直到结果准备就绪。

### 21.1.5 休眠 ###

影响任务行为的一种简单方法是调用`sleep()`，这将使任务终止执行给定的时间。在`LiftOff`中把`yield()`换成`sleep()`会怎样呢？

```java
/**
 * 使用sleep
 * @author mrdios
 * @date 2016-07-28 14:32
 */
public class SleepingTask extends LiftOff {
    @Override
    public void run() {
        try {
            while (countDown-- > 0){
                System.out.println(status());
                TimeUnit.MILLISECONDS.sleep(100);
            }
        } catch (InterruptedException e) {
            System.err.println("Interrupted");
        }
    }

    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            exec.execute(new SleepingTask());
        }
        exec.shutdown();
    }
}
```
输出：
```
 #0(9), #2(9), #1(9), #3(9), #4(9), #4(8), #3(8), #1(8), #2(8), #0(8), #2(7), #1(7), #3(7),
 #4(7), #0(7), #4(6), #3(6), #0(6), #2(6), #1(6), #1(5), #2(5), #0(5), #3(5), #4(5), #1(4),
 #0(4), #2(4), #3(4), #4(4), #4(3), #3(3), #0(3), #2(3), #1(3), #1(2), #3(2), #4(2), #0(2),
 #2(2), #2(1), #0(1), #4(1), #3(1), #1(1), #3(LiftOff!), #4(LiftOff!), #0(LiftOff!), #2(LiftOff!), #1(LiftOff!),
```

### 21.1.6 优先级 ###

线程的**优先级**将该线程的重要性传递给了调度器。调度器将倾向于让优先级最高的线程先执行，然而并不意味着优先级较低的线程得不到执行（优先级不会导致死锁），仅仅只是执行顺序较低而已。

**大多数情况下，所有线程都应该以默认的优先级运行，试图操纵线程的优先级通常会带来问题。**

下面是演示优先级的示例，可以用`getPriority()`来读取现有线程的优先级，并且在任何时刻都可以通过`setPriority()`来修改它：

```java
/**
 * 线程优先级演示
 *
 * @author mrdios
 * @date 2016-07-28 15:27
 */
public class SimplePriority implements Runnable {
    private int countDown = 5;
    private volatile double d;
    private int priority;

    public SimplePriority(int priority) {
        this.priority = priority;
    }

    @Override
    public String toString() {
        return Thread.currentThread() + ": " + countDown;
    }

    @Override
    public void run() {
        Thread.currentThread().setPriority(priority);
        while (true) {
            for (int i = 0; i < 10000; i++) {
                d += (Math.PI + Math.E) / (double) i;
                if (i % 1000 == 0) {
                    Thread.yield();
                }
                System.out.println(this);
                if (--countDown == 0) {
                    return;
                }
            }
        }
    }

    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            exec.execute(new SimplePriority(Thread.MIN_PRIORITY));
        }
        exec.execute(new SimplePriority(Thread.MAX_PRIORITY));
        exec.shutdown();
    }
}
```
输出：
```
Thread[pool-1-thread-6,10,main]: 5
Thread[pool-1-thread-6,10,main]: 4
Thread[pool-1-thread-6,10,main]: 3
Thread[pool-1-thread-6,10,main]: 2
Thread[pool-1-thread-6,10,main]: 1
Thread[pool-1-thread-1,1,main]: 5
Thread[pool-1-thread-1,1,main]: 4
Thread[pool-1-thread-1,1,main]: 3
Thread[pool-1-thread-1,1,main]: 2
Thread[pool-1-thread-1,1,main]: 1
Thread[pool-1-thread-2,1,main]: 5
Thread[pool-1-thread-2,1,main]: 4
Thread[pool-1-thread-2,1,main]: 3
Thread[pool-1-thread-2,1,main]: 2
Thread[pool-1-thread-2,1,main]: 1
Thread[pool-1-thread-3,1,main]: 5
Thread[pool-1-thread-3,1,main]: 4
Thread[pool-1-thread-3,1,main]: 3
Thread[pool-1-thread-3,1,main]: 2
Thread[pool-1-thread-3,1,main]: 1
Thread[pool-1-thread-4,1,main]: 5
Thread[pool-1-thread-4,1,main]: 4
Thread[pool-1-thread-4,1,main]: 3
Thread[pool-1-thread-4,1,main]: 2
Thread[pool-1-thread-4,1,main]: 1
Thread[pool-1-thread-5,1,main]: 5
Thread[pool-1-thread-5,1,main]: 4
Thread[pool-1-thread-5,1,main]: 3
Thread[pool-1-thread-5,1,main]: 2
Thread[pool-1-thread-5,1,main]: 1
```

以上程序中，在`run()`里执行了10000次开销很大的浮点运算，如果没有加入这些运算的话，就看不到设置优先级的效果，有了这些运算，可以看到最高优先级的线程被线程调度器优先选择执行。

尽管JDK有10个优先级，但它与多数操作系统都不能良好地映射，比如Windows有7个优先级是不固定的，所以这种映射关系也是不确定的，唯一可移植的方法是当调整优先级的时候，只使用`MAX_PRIORITY`、`NORM_PRIORITY`和`MIN_PRIORITY`三种级别。

### 21.1.7 让步 ###

如果我们已经确认已经完成了`run()`中一次循环迭代过程所需的工作，就可以给线程调度机制一个提示：你的工作已经做得差不多了，可以让别的线程使用CPU了。这个提示通过调用`yield()`方法来做出，不过这只是一个提示，没有任何机制能保证它将会被采纳。当调用`yield()`时，也意味着我们在建议具有**相同优先级**的其他线程可以运行。

一般情况下，对于任何重要的控制或在调整应用时，都不能依赖于`yield()`，实际上`yield()`经常被误用。

### 21.1.8 后台线程 ###

所谓后台（daemon）线程，是指在程序运行的时候在后台提供一种通用服务的线程，并且这种线程并不属于程序中不可或缺的部分。因此当所有的非后台线程结束时，程序也就终止了，同时会杀死进程中所有后台线程。反过来说，只要有任何非后台线程还在运行，程序就不会终止，比如，执行`main()`的就是一个非后台线程。

```java
/**
 * 后台线程
 *
 * @author mrdios
 * @date 2016-07-28 17:07
 */
public class SimpleDaemons implements Runnable {
    @Override
    public void run() {
        try {
            while (true) {
                TimeUnit.MILLISECONDS.sleep(100);
                System.out.println(Thread.currentThread() + ": " + this);
            }
        } catch (InterruptedException e) {
            System.out.println("sleep() interrupted");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            Thread daemon = new Thread(new SimpleDaemons());
            daemon.setDaemon(true);//必须在start()之前设置
            daemon.start();
        }
        System.out.println("All daemons started");
        TimeUnit.MILLISECONDS.sleep(175);
    }
}
```
输出：
```
All daemons started
Thread[Thread-7,5,main]: com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter21.SimpleDaemons@5826d8e6
Thread[Thread-6,5,main]: com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter21.SimpleDaemons@5e6a1140
Thread[Thread-8,5,main]: com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter21.SimpleDaemons@1ad60225
Thread[Thread-1,5,main]: com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter21.SimpleDaemons@61ae0436
Thread[Thread-9,5,main]: com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter21.SimpleDaemons@1ad60225
Thread[Thread-2,5,main]: com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter21.SimpleDaemons@6796a753
Thread[Thread-0,5,main]: com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter21.SimpleDaemons@592b12d
Thread[Thread-4,5,main]: com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter21.SimpleDaemons@43be87a0
Thread[Thread-5,5,main]: com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter21.SimpleDaemons@53c36f46
Thread[Thread-3,5,main]: com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter21.SimpleDaemons@11ba3c1f
```

必须在线程启动之前调用`setDaemon()`方法，才能把此线程设置为后台线程。

在以上程序中，如果注释掉`main()`方法里的最后一句代码或者main()线程的睡眠时间改为100以下或者大于100但是很接近100时会怎样呢？我们会看不到线程启动后的输出结果，因为一旦`main()`一旦完成它的工作，就没有什么能阻止程序终止了，因为除了后台线程以外，已经没有线程在运行了。

### 21.1.9 加入一个线程 ###

一个线程可以在另一个线程上调用`join()`方法，效果就是等待一段时间直到第二个线程结束才继续执行。如果某个线程在另一个线程`t`上调用`t.join`，此线程将被挂起，直到目标线程`t`结束才恢复(即t.isAlive()返回false)。

也可以在调用`join()`时带上一个超时参数，这样如果目标线程在这段时间到期时还没有结束的话，join()方法总能返回。

对`join()`方法的调用可以被中断，做法是在调用线程上调用`interrupt()`方法，这时候需要处理异常：

```java
/**
 * 线程加入（join）
 *
 * @author mrdios
 * @date 2016-07-28 17:42
 */
class Hide extends Thread {
    private int duration;

    public Hide(String name, int sleepTime) {
        super(name);
        duration = sleepTime;
        start();
    }

    @Override
    public void run() {
        try {
            sleep(duration);
        } catch (InterruptedException e) {
            System.out.println(getName() + " was interrupted " + "isInterrupted(): " + isInterrupted());
            return;
        }
        System.out.println(getName() + " has hided");
    }
}

class Seek extends Thread {
    private Hide child;

    public Seek(String name, Hide child) {
        super(name);
        this.child = child;
        start();
    }

    @Override
    public void run() {
        try {
            child.join();
        } catch (InterruptedException e) {
            System.out.println("Interrupted");
        }
        System.out.println(child.getName() + " has hided and " + getName() + " can go get him");
    }
}

public class HideAndSeek {
    public static void main(String[] args) {
        Hide
                tom = new Hide("tom", 1500),
                jack = new Hide("jack", 1500);
        Seek
                dad = new Seek("dad", tom),
                mom = new Seek("mom", jack);
//        jack.interrupt();
    }
}
```
输出：
```
tom has hided
tom has hided and dad can go get him
jack has hided
jack has hided and mom can go get him
```
以上是一个演示捉迷藏游戏的程序，父母和两个孩子捉迷藏，孩子藏起来让父母找，孩子藏的时间是构造函数的参数决定的，孩子在规定的时间内藏好了，父母才能开始寻找，也就是说父母只能等待孩子藏好之后才能开始找，所以在`Seek`线程中的加入了`Hide`线程，hide好了，才能seek。

上述代码中注释掉的最后一句模拟了中断，被中断的线程调用`isInterrupted()`方法返回的将是ture，因为调用`.interrupt()`方法的时候会为线程的此标志设置值，然而，如果放开注释，将会看到此值是false，这是因为在异常被捕获的时候这个标志总是为false。

## 21.2 共享受限资源 ##

### 21.2.1 解决共享资源竞争 ###

对于应该什么时候同步的问题，应该运用Brain的同步规则：**如果你正在写一个变量，它可能接下来将被另一个线程读取，或者正在读取一个上一次被另一个线程写过的变量，那么你必须使用同步，并且，读写线程都必须使用相同的监视器同步。**

### 21.2.2 原子性与易变性 ###

在线程中，“原子操作不需要进行同步控制”是一个不正确的认识，原子性可以应用于除`long`和`double`之外的所有基本类型之上的“简单操作”。对于读写除long和double之外的基本类型变量这样的操作，可以保证它们会被当作不可分(原子)的操作来操作内存，但是JVM可以将64位(long和double)的读写当作两个分离的32位操作来执行，这就产生了一个在读取和写入操作中间发生上下文切换，从而导致不同的任务可以看到不正确结果的可能性，但是当定义long和double变量时，如果使用`volatile`关键字，就会获得原子性。

volatile关键字确保了应用中的可视性，如果将几个域声明为volatile，那么只要对这个域产生了写操作，那么所有的读操作就都可以看到这个修改，即便用了本地缓存，情况也如此，volatile域会立即被写入到主存中，而读取操作就发生在主存中。

### 21.2.3 线程本地存储 ###

防止任务在共享资源上产生冲突的一种方式是根除对变量的共享，线程本地存储是一种自动化机制，可以为使用相同变量的每个不同的线程都创建不同的存储。因此，如果有5个线程都要使用变量`x`所表示的对象，那线程本地存储就会生成5个用于`x`的不同的存储块，更重要的是，它们使得我们可以将状态与线程关联起来。

#### 21.2.4 在阻塞时终结 ####

**线程状态**

一个线程可以处于以下四种状态：
1. 新建(new)：当线程被创建时，它只会短暂处于这个状态。此时它已经分配了必须的系统资源，并执行了初始化，此刻线程已经有资格获得CPU时间了，之后调度器将把这个线程转变为可运行状态或阻塞状态。
2. 就绪(Runnable)：在这种状态下，只要调度器把时间片分配给线程，线程就可以运行。也就是说，在任意时刻，线程可以运行也可以不运行。只要调度器能分配时间片给线程。它就可以运行，这不同于死亡和阻塞状态。
3. 阻塞(Block)：线程能够运行，但有某个条件阻止它运行。当线程处于阻塞状态时，调度器将忽略线程，不会分配给线程任何CPU时间，直到线程重新进入就绪状态，它才有可能执行操作。
4. 死亡(Dead)：处于死亡或终止状态的线程将不再是可调度的。并且再也不会得到CPU时间，它的任务已结束，或不再是可运行的。任务死亡的通常方式是从`run()`方法返回。但是任务的线程还可以被中断。

**进入阻塞状态**

一个任务进入阻塞状态，可能有以下原因：
1. 通过调用`sleep(milliseconds)`使任务进入休眠状态，在这种情况下，任务在指定的时间内不会运行。
2. 通过调用`wait()`使线程挂起。直到线程得到了`notify()`或`notifyAll()`消息（或SE5的`signal()`或`signalAll()`消息），线程才会进入就绪状态
3. 任务在等待某个输入/输出完成
4. 任务试图在某个对象上调用其同步控制方法，但是对象锁不可用，因为另一个任务已经获取了这个锁



