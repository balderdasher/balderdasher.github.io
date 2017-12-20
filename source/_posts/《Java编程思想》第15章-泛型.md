---
title: 《Java编程思想》第15章-泛型
date: 2017-05-23 14:41:18
tags: [java,读书笔记]
categories: java
permalink: thinking-in-java-generic
---
## 15.1 泛型的出现 ##
### 15.1.1 为什么要用泛型 ###
一般的类和方法，只能使用具体的类型：要么是基本类型，要么是自定义的类。如果需要编写可以应用于多种类型的代码。这种刻板的限制对代码的束缚就很大。所以从Java SE5开始，泛型出现了，泛型实现了**参数化类型**的概念，使代码可以应用于多种类型。

### 15.1.2 什么是泛型 ###
泛型这个术语的意思是：“适用于许多许多的类型”，就是一种宽泛的类型，适用于很多的类型。

泛型可以用于类、接口、方法等

<!--more-->
## 15.2 泛型类 ##
有时候，我们希望一个类可以用于不同类型的对象，通常而言，在使用时用来表示一种类型的对象，泛型就可以用来指定这个类用于什么类型的对象，而且由编译器来保证类型的正确性。

遇到上面的需求时，通常我们会暂时不指定类型，而是稍后使用的时候再决定使用什么类型，要达到这个目的就需要使用类型参数，用尖括号括住，放在类名后面。然后在使用这个类的时候，再用实际的类型替换此参数类型，下面的例子中，T就是类型参数：

```java
/**
 * 泛型类
 *
 * @author huxiong
 * @date 2016/07/07 15:17
 */
public class GenericClass<T> {
    private T field;

    public T getField() {
        return field;
    }

    public void setField(T field) {
        this.field = field;
    }

    @Override
    public String toString() {
        return "GenericClass{" +
                "field=" + field +
                '}';
    }

    public static void main(String[] args) {
        GenericClass<String> s = new GenericClass<>();
        s.setField("gaga");
        System.out.println("s：" + s);
        GenericClass<Integer> i = new GenericClass<>();
        i.setField(100);
        System.out.println("i：" + i);
    }
}
```
输出：
```
s：GenericClass{field=gaga}
i：GenericClass{field=100}
```
GenericClass类可以用于任意类型的对象，它是一个泛型类，我们只要在使用的时候指定其具体类型就可以了。

## 15.3 泛型接口 ##
同样，泛型也可以用于接口，例如`生成器`，这是一种专门负责创建对象的类，这也是`工厂方法设计模式`的一种应用，不过在使用时，不需要任何参数，下面是一个制造玩具的程序：

```java
/**
 * 生产接口
 *
 * @param <T>
 */
interface Builder<T> {
    T next();
}

/**
 * 玩具
 */
class Toy {
    private static long counter = 0;
    private final long id = counter++;

    @Override
    public String toString() {
        return getClass().getSimpleName() + id;
    }
}

/**
 * 玩具狗
 */
class DogToy extends Toy {
}

/**
 * 玩具猫
 */
class CatToy extends Toy {
}

/**
 * 玩具熊
 */
class BearToy extends Toy {
}

/**
 * 玩具制造器
 */
class ToyBuilder implements Builder<Toy>, Iterable<Toy> {
    private Class[] types = {DogToy.class, CatToy.class, BearToy.class};
    private int size = 0;//for iteration
    private final Random rand = new Random(47);

    public ToyBuilder() {
    }

    public ToyBuilder(int size) {
        this.size = size;
    }

    @Override
    public Toy next() {
        try {
            // make a toy randomly
            return (Toy) types[rand.nextInt(types.length)].newInstance();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * 自定义迭代器
     */
    class ToyIterator implements Iterator<Toy> {
        int count = size;

        @Override
        public boolean hasNext() {
            return count > 0;
        }

        @Override
        public Toy next() {
            count--;
            return ToyBuilder.this.next();
        }

        @Override
        public void remove() {
            throw new UnsupportedOperationException();
        }
    }

    @Override
    public Iterator<Toy> iterator() {
        return new ToyIterator();
    }

    public static void main(String[] args) {
        ToyBuilder tb = new ToyBuilder();
        for (int i = 0; i < 3; i++) {
            System.out.println(tb.next());
        }
        System.out.println("------------------------");
        for (Toy t : new ToyBuilder(3)) {
            System.out.println(t);
        }
    }
}
```
输出：
```
BearToy0
BearToy1
CatToy2
------------------------
BearToy3
BearToy4
CatToy5
```

## 15.4 泛型方法 ##
泛型方法所在的类可以是泛型类也可以不是泛型类，也就是说是否拥有泛型方法，与其所在的类是否是泛型没有关系。

泛型方法的定义就是将泛型参数列表置于返回值之前，像这样：

```java
/**
 * 泛型方法
 * @author huxiong
 * @date 2016/07/07 15:00
 */
public class GenericMethod {
    public <T> void f(T x){
        System.out.println(x.getClass().getName());
    }

    public static void main(String[] args) {
        GenericMethod gm = new GenericMethod();
        gm.f("baby");
        gm.f(1);
        gm.f(3.0f);
        gm.f(5.0d);
        gm.f(gm);
    }
}
```
输出：
```
java.lang.String
java.lang.Integer
java.lang.Float
java.lang.Double
com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter15.GenericMethod
```

通过程序我们看到，当使用泛型类时，必须在创建对象的时候指定类型参数的值，而使用泛型方法时通常不需要，因为编译器会为我们找出正确的类型。这成为**类型参数推断**

## 15.5 泛型擦除 ##
**在泛型代码中，无法获得任何有关泛型类型参数的信息。**

因此，我们可以知道如类型参数标识符和泛型类型边界这类的信息，但是却无法知道用来创建某个特定实例的实际的类型参数。之所以会这样，是因为java泛型是使用擦除来实现的，当使用泛型时，任何具体的类型信息都被擦除了，唯一知道的就是在使用一个对象，因此`List<String>`和`List<Integer>`在运行时事实上是相同的类型，这两种形式都被擦除成它们的“原生”类型List

```java
class SuperMan {
    public void fly() {
        System.out.println("超人在飞...");
    }
}

class Hero<T> {
    private T someone;

    public Hero(T someone) {
        this.someone = someone;
    }

    public void savePeople() {
//        someone.fly(); // 此句不能通过编译
    }
}

class Hero2<T extends SuperMan> {
    private T someone;

    public Hero2(T someone) {
        this.someone = someone;
    }

    public void savePeople() {
        someone.fly(); // 此句通过编译
    }
}

public class GenericWipe {
    public static void main(String[] args) {
        SuperMan sm = new SuperMan();
        Hero2<SuperMan> hero = new Hero2<>(sm);
        hero.savePeople();
    }
}
```
就以上程序而言，由于有了擦除，java编译器无法将`savePeople()`必须能够在`someone`上调用`fly()`这一需求映射到`SuperMan`拥有`fly()`这一事实上。所以`Hero`中的`savePeople()`将不能通过编译，为了调用`fly()`，我们必须协助泛型类，给定泛型的边界，以告知编译器只能接收遵循这个边界的类型，所以`Hero2`中相应的代码编译通过，

编译器实际上会把类型参数替换为它的擦除，就像以上程序中一样，`T`擦除到了`SuperMan`，就好像在类的声明中用`SuperMan`替换了`T`一样。

### 15.5.1 擦除与迁移兼容性 ###
擦除并不是一个语言特性，它是java的泛型实现中的一种折中，因为java刚出现时并没有这个组成部分，所以这种折中也是必须的。

在基于擦除的实现中，泛型类型被当做第二类类型处理，就是不能再某些重要的上下文环境中使用的类型。泛型类型只有在静态类型检查期间才出现，在此之后程序中所有的泛型类型都将被擦除，替换为他们的泛型上界，如`List<T>`被替换成`List`

擦除也是java语言的一种“迁移兼容性”，如果某个类库要迁移到泛型上的话，为了实现迁移兼容性，这个类库和应用程序都必须与其他所有的部分是否使用了泛型无关。它们必须不具备探测其它类库是否使用了泛型的能力，所以必须将这些证据“擦除”。

## 15.6 使用泛型的问题 ##
* 任何基本类型都不能作为泛型参数，诸如`List<int>`,可以使用包装类如`List<Integer>`。
* 一个类不能实现同一个泛型接口的两种变体，由于擦除的原因，这两个变体会成为相同的接口，如`interface A<T>`,`class B implements A<B>`,`class C extends B implements A<C>`，此时`C`将不能编译，因为擦除会将`A<B>`和`A<C>`简化为相同的类`A`。
* 在泛型中，拥有方法`void f(List<T> v)`和`void f(List<W> v)`的一个类`UserList<T,W>`是不能编译通过的，因为由于擦除的原因，这个两个重载方法将产生相同的类型签名，所以这时候必须提供有明显区别的方法名。
* 不能同时`catch`同一个泛型异常的多个实例，如果定义了一个泛型异常类`GenericException<T>`，此时不能同时捕获异常`GenericException<Integer>`和`GenericException<String>`，因为在泛型擦除后他们都是相同的类型。
* **泛型类的静态变量是共享的**，如以下程序，由于经过类型擦除，所有的泛型类实例都关联到同一份字节码上，泛型类的所有静态变量是共享的
```java
public class StaticTest{
    public static void main(String[] args){
        GT<Integer> gti = new GT<Integer>();
        gti.var=1;
        GT<String> gts = new GT<String>();
        gts.var=2;
        System.out.println(gti.var);
    }
}
class GT<T>{
    public static int var=0;
    public void nothing(T x){}
}
```

输出：2

## PS ##
- 虚拟机中没有泛型，只有普通类和普通方法
- 所有泛型类的类型参数在编译时都会被擦除
- 创建泛型对象时请指明类型，让编译器尽早地做参数检查
- 不要忽略编译器的警告信息，否则会有潜在的`ClassCastException`异常
