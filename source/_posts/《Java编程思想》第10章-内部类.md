---
title: 《Java编程思想》第10章-内部类
date: 2017-05-18 18:38:54
tags: [java,读书笔记]
categories: java
permalink: thinking-in-java-inner-class
---
## 10.1 定义 ##

**定义在一个类内部的类叫做内部类**

创建内部类的方式就是简单地把类的定义放在外围类的里面。

如果想从外部类的非静态方法之外的任意位置创建某个内部类的对象，必须具体指明这个对象的类型，如：**OuterClassName.InnerClassName**。
<!--more-->

## 10.2 链接到外部类 ##

生成内部类的对象时，这个对象和生成它的**外围对象**有一种联系，它能访问外围对象的所有成员而不需要**任何特殊条件**。此外，内部类还拥有其外部类的所有元素的访问权。

```java
public interface Selector {
    boolean end();
    Object current();
    void next();
}

public class Sequence {
    private Object[] items;
    private int next = 0;

    public Sequence(int size) {
        items = new Object[size];
    }

    public void add(Object x) {
        if (next < items.length) {
            items[next++] = x;
        }
    }

    private class SequenceSelector implements Selector {
        private int i = 0;

        @Override
        public boolean end() {
            return i == items.length;
        }

        @Override
        public Object current() {
            return items[i];
        }

        @Override
        public void next() {
            if (i < items.length) {
                i++;
            }
        }
    }

    public Selector selector() {
        return new SequenceSelector();
    }

    public static void main(String[] args) {
        Sequence sequence = new Sequence(10);
        for (int i = 0; i < 10; i++) {
            sequence.add(Integer.toString(i));
        }
        Selector selector = sequence.selector();
        while (!selector.end()) {
            System.out.println(selector.current() + " ");
            selector.next();
        }
    }
}
```

输出：0 1 2 3 4 5 6 7 8 9 

在以上程序中，`SequenceSelector`是提供`Selector`功能的`private`类，在它的`end()`、`current()`和`next()`方法中都用到了`items`，这是一个引用，它并不属于`SequenceSelector`的一部分，而是外围类`Sequence`的一个`private`字段，但是`SequenceSelector`可以访问并且使用它们，就像使用它自己拥有的东西一样，非常方便。

所以，内部类自动拥有对外部类所有成员的访问权，但是这是如何做到的呢？

当一个外围类对象创建了一个内部类对象时，此内部类对象会秘密地捕获一个指向那个外围类对象的引用，当内部类对象在访问此外围类的成员时，就是用这个秘密的外围类对象引用来选择外围类的成员。原理就是这样，具体的细节编译器会帮我们去处理。

内部类的对象只能在于外围类对象相关联的情况下才能被创建(内部类是**非static**时)。构建内部类对象时，需要一个指向外围类对象的引用，如果编译器访问不到这个引用就会报错。

## 10.3 使用.this和.new ##

如果需要生成对外部类对象的引用，可以使用**外部类名字.this**。这样产生的引用自动具有正确的类型，这在编译器已经知晓并且检查，所有没有任何运行时开销。

```java
public class DotThis {
    public void f(){
        System.out.println("DotThis.f()");
    }

    public class Inner{
        public DotThis outer(){
            return DotThis.this;
        }
    }

    public Inner inner(){
        return new Inner();
    }

    public static void main(String[] args) {
        DotThis dt = new DotThis();
        DotThis.Inner dti = dt.inner();
        dti.outer().f();
    }
}
```
如果需要创建某个内部类的对象，需要使用**.new**语法：

```java
public class DotNew {
    public class Inner{}

    public static void main(String[] args) {
        DotNew dn = new DotNew();
        DotNew.Inner dni = dn.new Inner();
    }
}
```

如以上程序所示：**必须使用外部类的对象来创建内部类对象**。这是因为在拥有外部类对象之前是不可能创建内部类对象的，内部类对象会暗自连接到创建它的外部类对象上。

但是，如果创建的是**嵌套类（静态内部类）**，就不需要对外部类对象的引用。

## 10.4 内部类与向上转型 ##

```java
public interface Contents {
    int value();
}

public interface Destination {
    String readLabel();
}

public class Parcel {
    private class PContents implements Contents{
        private int i = 11;
        @Override
        public int value() {
            return i;
        }
    }

    protected class PDestination implements Destination{
        private String label;
        private PDestination(String whereto){
            label = whereto;
        }
        @Override
        public String readLabel() {
            return label;
        }
    }

    public Destination destination(String s){
        return new PDestination(s);
    }

    public Contents contents(){
        return new PContents();
    }
}

public class TestParcel {
    public static void main(String[] args) {
        Parcel p = new Parcel();
        Contents c = p.contents();
        Destination d = p.destination("beijing");
    }
}
```
在以上程序中，内部类`PContents`和`PDestination`分别是`private`和`protected`的，意味着客户端程序员想了解这些成员是会受到限制的，实际上我们不能向下转型成`private内部类`，因为不能访问其名字，于是，private内部类给我们提供了一种途径，通过这种方式能够完全阻止任何依赖于类型的编码，并且完全隐藏了实现的细节，此外，从客户端程序员的角度看，由于不能访问新增加的、原本不属于公共接口的方法，所以扩展接口是没有价值的，这也能让编译器是代码变得更高效。

## 10.5 在方法和作用域内的内部类 ##

内部类除了能在类里面定义以外，还能在其他一些地方定义，如方法内部和作用域。通常这样做有两个理由：

1. 实现了某类型的接口，于是可以创建并返回对其的引用(如以上的`PDestination`可改为`Destination`方法中的一个内部类)
2. 想创建一个类来辅助解决一个复杂的问题，但是又不希望这个类是公共可用的。

## 10.6 匿名内部类 ##

匿名内部类看起来像下面这样：

```java
public class AnonymousInnerclass {

    public Contents contents(){
        return new Contents() {
            private int i = 11;
            @Override
            public int value() {
                return i;
            }
        };
    }

    public static void main(String[] args) {
        AnonymousInnerclass aic = new AnonymousInnerclass();
        aic.contents();
    }
}
```
contents()方法将返回值的生成和表示这个返回值的类定义结合在了一起，这个类没有名字，是匿名的，这种奇怪的语法意思是：“创建一个继承自`Contents`的匿名类的对象。”，通过new表达式返回的引用自动向上转型为`Contents`的引用，如果不使用匿名内部类，那么代码看起来就是下面这样

```java
public class AnonymousInnerclass {

    // 不使用匿名内部类时要定义的内部类
    class MyContents implements Contents{
        private int i = 11;
        @Override
        public int value() {
            return i;
        }
    }

    public Contents contents(){
        return new MyContents();
    }

    public static void main(String[] args) {
        AnonymousInnerclass aic = new AnonymousInnerclass();
        aic.contents();
    }
}
```
如果在一个方法中定义一个内部类，并且这个内部类需要用到这个方法的参数对象，这时候参数引用必须是**final**的，[java为什么匿名内部类的参数引用必须用final？](http://cuipengfei.me/blog/2013/06/22/why-does-it-have-to-be-final/ "java为什么匿名内部类的参数引用必须用final？")

匿名内部类只能实现一个接口。

### 10.6.1 优化的工厂方法 ###

```java
public interface DbService {
    void connect();
    void query();
}

interface DbServiceFactory {
    DbService getDbService();
}

public class MysqlDb implements DbService {
    private MysqlDb(){}
    @Override
    public void connect() {
        System.out.println("Mysql connect...");
    }
    @Override
    public void query() {
        System.out.println("Mysql query...");
    }
    public static DbServiceFactory factory = new DbServiceFactory() {
        @Override
        public DbService getDbService() {
            return new MysqlDb();
        }
    };
}

public class OracleDb implements DbService {
    private OracleDb(){}
    @Override
    public void connect() {
        System.out.println("Oracle connect...");
    }
    @Override
    public void query() {
        System.out.println("Oracle query...");
    }
    public static DbServiceFactory factory = new DbServiceFactory() {
        @Override
        public DbService getDbService() {
            return new OracleDb();
        }
    };
}

public class DbFactories {
    public static void serviceConsumer(DbServiceFactory fact) {
        DbService dbService = fact.getDbService();
        dbService.connect();
        dbService.query();
    }

    public static void main(String[] args) {
        serviceConsumer(MysqlDb.factory);
        serviceConsumer(OracleDb.factory);
    }
}
```
如上所示，使用匿名内部类能够更简洁又更优雅地实现工厂方法。避免了每个数据库服务类都创建自己的服务工厂类。

## 10.7 嵌套类 ##

声明为**static**的内部类成为**嵌套类**。嵌套类对象与外围类对象之间不需要有联系。

嵌套类意味着：

- 要创建嵌套类的对象，并不需要依赖外围类的对象
- 不能从嵌套类的对象中访问非静态的外围类对象

嵌套类与普通的内部类还有一个区别：普通内部类的字段与方法只能放在类的外部层次，所以**普通内部类不能有static数据和static字段，也不能包含嵌套类**，但是嵌套类可以包含这些东西。

### 10.7.1 接口内部的类 ###

正常情况下，不能在接口内部放置任何代码，但**嵌套类可以作为接口的一部分**，位于接口中任何类都自动是`public`和`static`的，所以只是将嵌套类置于接口的命名空间而已，甚至可以在内部类中实现外围接口：

```java
public interface MyInterface{
	void func();
	class Test implements MyInterface{
		public void func(){
			System.out.println("func!");
		}
		public static void main(String[] args){
			new Test().func();
		}
	}
}
```

以上技术在我们想要创建公共代码，使它们能够被某个接口的所有实现共用时非常有用。

### 10.7.2 从多层嵌套类中访问外部类成员 ###

**一个内部类不管被嵌套了多少层，它都能透明地访问所有它所嵌入的外围类的所有成员**

LKZS：

```java
class Test
	private void f(){}
	class A{
		private void g(){}
		public class B{
			void h(){
				f();
				g();
			}
		}
	}
}

public class MutilAccess{
	public static void main(String[] args){
		Test t = new Test();
		Test.A a = t.new A();
		Test.A.B b = a.new B();
		b.h();
	}
}
```
## 10.8 为什么需要内部类 ##
使用内部类最大的原因是：每个内部类都能独立地继承自一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类没有影响。

内部类使多重继承的解决方案变得完整，使用内部类可以获得以下特性：

- 内部类可以有多个实例，每个实例都有自己的状态信息，并且与其外围类的信息相互独立
- 在单个外围类中，可以让多个内部类以不同的方式实现同一个接口，或继承同一个类
- 创建内部类的时刻并不依赖于外围类的创建
- 内部类并没有令人迷惑的“is-a”关系，它就是一个独立的实体

### 10.8.1 闭包与回调 ###

> 闭包是一个可调用的对象，它记录了一些信息，这些信息来源于创建它的作用域

通过闭包的定义，可以看出内部类是面向对象的闭包，因为它不仅包含外围类对象（创建内部类的作用域）的信息，还自动拥有一个指向此外围类对象的引用，在此作用域内，内部类有权操作所有的成员，包括private成员。

通过内部类提供闭包的功能是优良的解决方案，它比指针更灵活、更安全，见下例：

```java
public interface Incrementable {
    void increment();
}

/**
 * 外围类实现一个接口
 */
public class Callee1 implements Incrementable {
    private int i = 0;
    @Override
    public void increment() {
        i++;
        System.out.println(i);
    }
}

public class MyIncrement {
    public void increment(){
        System.out.println("Other operation");
    }
    public static void f(MyIncrement mi){
        mi.increment();
    }
}

/**
 * 内部类实现接口
 */
public class Callee2 extends MyIncrement {
    private int i = 0;

    @Override
    public void increment() {
        super.increment();
        i++;
        System.out.println(i);
    }

    private class Closure implements Incrementable{

        @Override
        public void increment() {
            Callee2.this.increment();
        }
    }
    Incrementable getCallbackRefrence(){
        return new Closure();
    }
}

public class Caller {
    private Incrementable callbackRefrence;

    Caller(Incrementable cbh) {
        callbackRefrence = cbh;
    }

    void go() {callbackRefrence.increment();
    }
}

public class Callbacks {
    public static void main(String[] args) {
        Callee1 c1 = new Callee1();
        Callee2 c2 = new Callee2();
        MyIncrement.f(c2);
        Caller caller1 = new Caller(c1);//普通方式
        Caller caller2 = new Caller(c2.getCallbackRefrence());//闭包
        caller1.go();
        caller1.go();
        caller2.go();
        caller2.go();
    }
}
```

以上的例子进一步展示了外围类实现一个接口与内部类实现此接口之间的区别，`Callee1`是最简单的方式，`Callee2`继承自`MyIncrement`,后者已经有一个不同的`increment()`方法并且与`Incrementable`接口期望的`increment()`方法完全不相关，如果`Callee2`继承了`MyIncrement`，就不能为了Incrementable的用途而覆盖increment（）方法，所以只能使用内部类独立地实现Incrementable。

内部类`Closure`实现了`Incrementable`,以提供一个返回`Callee2`的**“钩子”**（hook），而且是一个安全的钩子。无论谁获得此`Incrementable`的引用，都只能调用`increment()`,除此之外没有其他功能。

Caller的构造器需要一个`Incrementable`的引用作为参数，然后在某个时刻，Caller对象可以使用此引用回调Callee类

回调的价值在于它的灵活性——可以在运行时动态地决定需要调用什么方法。

### 10.8.2 内部类与控制框架 ###

**应用程序框架**就是被设计用来解决某类特定问题的一个类或者一组类。用来响应事件的系统成为**事件驱动系统**

一个抽象的事件类`Event`类来描述要控制的事件

```java
/**
 * 抽象的Event用来描述要控制的事件
 */
public abstract class Event {
    /** 事件执行时间 */
    private long eventTime;
    /** 事件执行的延迟时间 */
    protected final long delayTime;

    public Event(long delayTime) {
        this.delayTime = delayTime;
        start();
    }

    public void start(){
        eventTime = System.nanoTime() + delayTime;
    }

    public boolean ready(){
        return System.nanoTime() >= eventTime;
    }

    public abstract void action();
}
```

下面是一个用来管理并触发时间的实际控制框架：

```java
/**
 * 用来管理并触发时间的实际控制框架
 */
public class Controller {
    /** 事件列表 */
    private List<Event> eventList = new ArrayList<>();

    public void addEvent(Event e){
        eventList.add(e);
    }

    public void run(){
        while(eventList.size() > 0){
            // 拷贝事件列表以便读取的元素的时候去不更新事件列表
            for (Event e:new ArrayList<>(eventList)) {
                if (e.ready()){
                    System.out.println(e);
                    e.action();
                    eventList.remove(e);
                }
            }
        }
    }
}
```

目前为止我们并不知道Event做了什么,这就是设计的关键所在：**使变化的事物与不变的事物相互分离**，这里的变化事物就是指不同的Event对象，它们拥有不同的行为，我们只要创建不同的Event子类来表现不同的行为就可以了。这正是内部类要做的事情，内部类允许：

- 控制框架的完整实现是由单个的类创建的，从而使得实现的细节被封装起来，内部类用来表示解决问题所必须的各种不同的action();
- 内部类能够很容易地访问外围类的任意成员，所以可以避免这种实现变得笨拙。

框架的特定实现：**温室控制**：

```java
/**
 * 框架特定实现：温室控制
 */
public class GreenhouseControls extends Controller {
    private boolean light = false;

    /**
     * 事件：开灯
     */
    public class LightOn extends Event {
        public LightOn(long delayTime) {
            super(delayTime);
        }

        @Override
        public void action() {
            light = true;
        }

        @Override
        public String toString() {
            return "Light is on";
        }
    }

    /**
     * 事件：关灯
     */
    public class LightOff extends Event {
        public LightOff(long delayTime) {
            super(delayTime);
        }

        @Override
        public void action() {
            light = false;
        }

        @Override
        public String toString() {
            return "Light is off";
        }
    }

    private boolean water = false;

    /**
     * 事件：浇水
     */
    public class WaterOn extends Event {
        public WaterOn(long delayTime) {
            super(delayTime);
        }

        @Override
        public void action() {
            water = true;
        }

        @Override
        public String toString() {
            return "Water is on";
        }
    }

    /**
     * 事件：关水
     */
    public class WaterOff extends Event {
        public WaterOff(long delayTime) {
            super(delayTime);
        }

        @Override
        public void action() {
            water = false;
        }

        @Override
        public String toString() {
            return "Water is off";
        }
    }

    /**
     * 事件：响铃
     */
    public class Bell extends Event {
        public Bell(long delayTime) {
            super(delayTime);
        }

        @Override
        public void action() {
            addEvent(new Bell(delayTime));
        }

        @Override
        public String toString() {
            return "Bing!";
        }
    }

    /**
     * 事件：重启
     */
    public class Restart extends Event {
        private Event[] eventList;

        public Restart(long delayTime, Event[] eventList) {
            super(delayTime);
            this.eventList = eventList;
            for (Event e : eventList) {
                addEvent(e);
            }
        }

        @Override
        public void action() {
            for (Event e : eventList) {
                e.start();
                addEvent(e);// 返回每个事件
            }
            start(); // 返回当前事件
            addEvent(this);
        }

        @Override
        public String toString() {
            return "Restarting system";
        }
    }

    /**
     * 事件：暂停
     */
    public class Terminate extends Event {
        public Terminate(long delayTime) {
            super(delayTime);
        }

        @Override
        public void action() {
            System.exit(0);
        }

        @Override
        public String toString() {
            return "System terminating";
        }
    }
}
```
以上的温室控制器通过添加不同的Event对象来配置该系统，这是**命令设计模式**的一个例子，在eventlist中存放着没一个被封装成对象的命令请求：

```java
public class GreenhouseController {
    public static void main(String[] args) {
        GreenhouseControls controls = new GreenhouseControls();
        controls.addEvent(controls.new Bell(900));
        Event[] events = {
                controls.new LightOn(200),
                controls.new LightOff(400),
        };
        controls.addEvent(controls.new Restart(200,events));
        if (args.length==1){
            controls.addEvent(new GreenhouseControls.Terminate(new Integer(args[0])));
        }
        controls.run();
    }
}
```

输出：
```
Bing!
Light is on
Light is off
Restarting system
System terminating
```

## 10.9 内部类的继承 ##

> 因为内部类的构造器必须连接到指向其外围类对象的引用，所以在继承内部类时，那个指向外围类的“秘密的”引用必须被初始化，但是在导出类中不再存在可连接的默认外围类对象，要解决这个问题，就必须使用像下面这样的语法：

```java
class WithInner {
    class Inner{}
}

public class InheritInner extends WithInner.Inner {
    public InheritInner(WithInner wi) {
        wi.super();
    }

    public static void main(String[] args) {
        WithInner wi = new WithInner();
        InheritInner ii = new InheritInner(wi);
    }
}
```
要理解以上代码就得先理解内部类，Inner是WithInner的内部类，那么一般的用法WithInner.Inner inner = new WithInner().new Inner()，我们可以看出要想创建Inner的对象必须先创建WithInner的对象之后才能创建Inner对象，那么现在要用一个类InheritInner继承Inner类，在继承过程中构造方法会被调用，即使不写也会调用默认构造方法，但问题出现了，在调用父类Inner构造方法时找不到WithInner的对象，所以就必须给InheritInner类的构造方法传入WithInner对象再通过wi.super()方法调用Inner的默认构造方法，因为这是创建对象的基本流程，所以这句话wi.super()是必须的。


