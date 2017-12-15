---
title: 《Java编程思想》第9章-接口
date: 2017-05-17 22:06:10
tags: [java,读书笔记]
categories: java
permalink: thinking-in-java-interface
---
**接口和内部类为我们提供了一种将接口和实现分离的更加结构化的方法**

我们应该首先了解抽象类，因为它是普通类和接口之间的中庸之道，我们不可能总是使用接口，有时候也要使用抽象类。
<!--more-->
## 9.1 抽象类和抽象方法 ##

抽象方法的语法：`abstract void f();`,被`abstract`关键字修饰或者包含抽象方法的类叫**抽象类**。

抽象类中可以没有抽象方法，但是一个类包含一个或多个抽象方法时，这个类必须被限定为抽象类。（否则编译器会报错）

## 9.2 接口 ##

一个接口表示：“所有实现了该特定接口的类看起来都像这样”。

接口可以包含域，接口中的域隐式地是**static**和**final**的。

接口中的方法都是**public**的，即使我们声明方法时不写public。

## 9.3 完全解耦 ##

### 9.3.1 策略模式 ###

创建一个能够根据所传递参数对象的不同而具有不同行为的方法，被称为**策略**设计模式，这类方法包含所要执行的算法中固定不变的部分，而“策略”包含变化的部分，策略就是传递进去的参数对象，包含要执行的代码。如以下代码所示：

```java
/**
 * 处理器类
 *
 * @author huxiong
 * @date 2016/06/27 15:43
 */
public class Processor {
    /**
     * 用于返回处理器名字
     *
     * @return
     */
    public String name() {
        return getClass().getSimpleName();
    }

    /**
     * 处理方法，接收参数并处理然后返回，返回类型可以是协变类型
     * @param input
     * @return
     */
    Object process(Object input) {
        return input;
    }
}

/**
 * 大写转换处理类
 * @author huxiong
 * @date 2016/06/27 15:48
 */
public class Upcase extends Processor {
    @Override
    String process(Object input) {
        return ((String)input).toUpperCase();
    }
}

/**
 * 小写转换处理类
 * @author huxiong
 * @date 2016/06/27 15:53
 */
public class Downcase extends Processor{
    @Override
    String process(Object input) {
        return ((String)input).toLowerCase();
    }
}

public class Apply {

    public static String s = "you can you up no can no bibi";

    /**
     * 策略设计模式：根据传入不同的处理器采取不同的处理方式
     * @param p 处理器（策略）
     * @param s 待处理字符串
     */
    public static void process(Processor p, Object s){
        System.out.print("Using Processor " + p.name() + "：");
        System.out.println(p.process(s));
    }

    public static void main(String[] args) {
        process(new Upcase(),s);
        process(new Downcase(),s);
    }
}
```
输出：

```
Using Processor Upcase：YOU CAN YOU UP NO CAN NO BIBI
Using Processor Downcase：you can you up no can no bibi
```
假如此时我们定义了另一个跟`Processor`拥有相同接口的类“Filter”，此时我们不能将`Filter`用于`Apply.process()`方法，即便这样可以正常运行，这里主要是因为`Apply.process()`方法与`Processor`之间的耦合过紧。

但是，如果`Processor`是一个接口，以上的限制就会变得松动，使我们可以复用`Apply.process()`。

使用接口复用代码的一种方式是遵循该接口来编写自己的类，但是，我们经常碰到无法修改想要使用的类的情况，**适配器模式**可以解决这种情况。

### 9.3.2 适配器模式 ###

适配器中的代码将接受我们所拥有的接口，并产生我们需要的接口，如下所示：

```java
public interface Processor {
    String name();
    Object process(Object input);
}

public class Apply {
    public static void process(Processor p,Object s){
        System.out.print("Using Processor " + p.name() + "：");
        System.out.println(p.process(s));
    }
}

public class Waveform {
    private static long counter;
    private final long id = counter++;

    @Override
    public String toString() {
        return "waveform " + id;
    }
}

public class Filter {
    public String name(){
        return getClass().getSimpleName();
    }

    public Waveform process(Waveform input){
        return input;
    }
}

public class LowPass extends Filter{
    double cutoff;
    public LowPass(double cutoff){
        this.cutoff = cutoff;
    }

    @Override
    public Waveform process(Waveform input) {
        return input;
    }
}

public class HighPass extends Filter{
    double cutoff;
    public HighPass(double cutoff){
        this.cutoff = cutoff;
    }

    @Override
    public Waveform process(Waveform input) {
        return input;
    }
}

/**
 * 滤波器适配器
 * @author huxiong
 * @date 2016/06/27 16:49
 */
public class FilterAdapter implements Processor{
    Filter filter;

    public FilterAdapter(Filter filter) {
        this.filter = filter;
    }

    @Override
    public String name() {
        return filter.name();
    }

    @Override
    public Object process(Object input) {
        return filter.process((Waveform) input);
    }
}

public class FilterProcessor {
    public static void main(String[] args) {
        Waveform w = new Waveform();
        Apply.process(new FilterAdapter(new LowPass(1.0)),w);
        Apply.process(new FilterAdapter(new HighPass(2.0)),w);
    }
}
```
输出：

```
Using Processor LowPass：waveform 0
Using Processor HighPass：waveform 0
```

在以上程序中，FilterAdapter的构造器接受我们拥有的接口`Filter`，然后生成需要的`Processor`接口的对象用于在`Apply.process()`方法。

我们注意到：在`FilterAdapter`中使用了代理，`FilterAdapter`相当于`Filter`的一个代理类，只不过这个代理类还实现了`Processor`接口

代理模式和适配器模式看起来很像，区别在于适配器还实现了接口，在这个接口中，有着跟被代理对象中的一些方法。

## 9.4 java中的多继承 ##

> java支持多继承，其方式是通过实现多个接口，如`class app implements InterfaceA,InterfaceB,InterfaceC` 。
> 使用接口的核心原因：为了能够向上转型为多个基类型，而使用接口的第二个原因则与使用抽象基类相同：防止客户端程序员创建该类的对象，并确保这仅仅是建立一个接口，因此，我们该使用抽象类还是接口呢？

如果要创建不带任何方法定义和成员变量的基类，我们应该选择接口而不是抽象类，如果知道某事物应该成为一个基类，第一选择应该是使用接口。

## 9.5 通过继承来扩展接口 ##

接口可以多继承接口，形式如：`interfaceA extends InterfaceB,InterfaceC`。在组合的不同接口中使用相同的方法名通常会造成代码可读性的混乱，应该尽量避免。

## 9.6 工厂方法设计模式 ##

> 接口是实现多重继承的途径，而生成遵循某个接口的对象的典型方式是就是**工厂方法**设计模式，与直接调用构造器不同，我们在工厂对象上调用的是创建方法，而该工厂对象将生成接口的某个具体实现的对象，这种方法可以将代码和接口的实现分离，使我们能够透明地将某个实现替换为另一个实现。

一个例子：

```java
/**
 * 数据库服务接口
 * @date 2016/06/27 20:15
 */
public interface DbService {
    void connect();
    void query();
}

/**
 * mysql 数据库服务
 * @date 2016/06/27 20:20
 */
public class MysqlDb implements DbService {

    @Override
    public void connect() {
        System.out.println("Mysql connect...");
    }

    @Override
    public void query() {
        System.out.println("Mysql query...");
    }
}

/**
 * oracle 数据库服务
 * @date 2016/06/27 20:22
 */
public class OracleDb implements DbService {
    @Override
    public void connect() {
        System.out.println("Oracle connect...");
    }

    @Override
    public void query() {
        System.out.println("Oracle query...");
    }
}

/**
 * 数据库服务工厂接口
 * @date 2016/06/27 20:18
 */
interface DbServiceFactory {
    DbService getDbService();
}

/**
 * mysql 数据库服务工厂
 * @date 2016/06/27 20:28
 */
public class MysqlDbFactory implements DbServiceFactory {
    @Override
    public DbService getDbService() {
        return new MysqlDb();
    }
}

/**
 * oracle 数据库服务工厂
 * @date 2016/06/27 20:30
 */
public class OracleDbFactory implements DbServiceFactory {
    @Override
    public DbService getDbService() {
        return new OracleDb();
    }
}

/**
 * 工厂方法模式
 * @date 2016/06/27 20:25
 */
public class DbFactories {
    public static void serviceConsumer(DbServiceFactory fact) {
        DbService dbService = fact.getDbService();
        dbService.connect();
        dbService.query();
    }

    public static void main(String[] args) {
        serviceConsumer(new MysqlDbFactory());
        serviceConsumer(new OracleDbFactory());
    }
}
```

输出：
```
Mysql connect...
Mysql query...
Oracle connect...
Oracle query...
```
