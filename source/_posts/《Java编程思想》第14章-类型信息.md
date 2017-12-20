---
title: 《Java编程思想》第14章-类型信息
date: 2017-05-23 14:20:29
tags: [java,读书笔记]
categories: java
permalink: thinking-in-java-rtti
---
## 14.1 RTTI是什么 ##
RTTI：Run-Time Type Information，运行时，识别一个对象的类型。
java主要以两种方式在运行时识别对象和类型信息：
- “传统的”RTTI，假定我们在编译时已经知道了所有的类型
- “反射”机制，在运行时发现和使用类的信息

<!--more-->
## 14.2 class对象 ##
类型信息在运行时是如何表示的呢，这项工作是由称为Class对象的特殊对象完成的，它包含了与类有关的信息，事实上，class对象就是用来创建所有“常规”对象的。Java使用Class对象来执行其RTTI。

类是程序的一部分，每一个类都有一个class对象，或者说每当编写了一个新类，就会产生一个class对象（恰当地说是被保存在一个同名的.class文件中），JVM将使用“类加载器”子系统来生成类对象。

类加载器子系统可以包含一条`类加载器链`，但是只有一个`原生类加载器`，它是JVM实现的一部分，原生类加载器加载的是所谓的`可信类`，包括Java API类，它们通常从本地盘加载，在这条加载器链中，通常不需要添加额外的类加载器，除非有特殊的需求（特殊的加载方式以支持web应用，或者从网络上下载类），那么可以使用额外的类加载器。

所有的类都是在第一次使用时，动态地加载到JVM的，当程序创建第一个对类的静态成员的引用时，就会加载这个类。所以构造器也是类的静态方法，即使在构造器之前没有使用static关键字，因此，使用`new`操作符创建类的新对象也会被当做对类的静态成员的引用。

因此，java程序在它运行之前并非被完全加载，其各个部分是在必需时才加载的。

类加载器首先检查这个类的class对象是否已经加载，如果尚未加载，默认的类加载器就会根据类名查找.class文件，在这个类的字节码被加载时，它会接收验证，以确保其没有被破坏，并不包含不良java代码。

一旦某个类的Class对象被载入内存，它就被用来创建这个类的所有对象。如下：

```java
class Candy {
    static{
        System.out.println("Loading Candy");
    }
}

class Cookie {
    static{
        System.out.println("Loading Cookie");
    }
}

class Gum {
    static{
        System.out.println("Loading Gum");
    }
}

public class SweetShop {
    public static void main(String[] args) {
        System.out.println("inside main");
        new Candy();
        System.out.println("After creating Candy");
        try {
            Class.forName("Gum");
        } catch (ClassNotFoundException e) {
            System.out.println("Could not find Gum");
        }
        System.out.println("After Class.forName(\"Gum\")");
        new Cookie();
        System.out.println("After creating Cookie");
    }
}
```
输出：
```
inside main
Loading Candy
After creating Candy
Loading Gum
After Class.forName("Gum")
Loading Cookie
After creating Cookie
```

程序中的Candy等类中的static子句在类第一次被加载时执行，从输出中可以看到，Class对象仅在需要的时候才被加载，static初始化是在类加载时进行的，对于`Class.forName("Gum")`,这个方法是Class类（所有的Class对象都属于这个类）的一个static成员，`forName`是取得对象的引用的一种方法，返回的是**一个class对象的引用**。对`forName()`的调用是为了产生副作用：如果类Gum还没有被加载就加载它。

无论何时，只要你想在运行时使用类型信息，就必须先获得对恰当的Class对象的引用。Class.forName()就是实现此功能的便捷途径，因为我们不需要为了获得Class引用而去新建一个该类型的对象，如果我们已经拥有一个类型的对象，直接可以调用其`getClass()`方法来获取Class引用。

Class的`newInstance()`方法是实现“虚拟构造器”的一种途径，虚拟构造器意味着：“我不知道你的确切类型，但无论如何要正确地创建你自己。”，使用`newInstance()`来创建的类，必须带有默认构造器。

### 14.2.1 类字面常量 ###
Java还提供了另一种方法来生成对Class对象的引用：`类字面常量`。比如`Candy.class`,这样做既简单又安全，因为它在编译器就受到检查（不用try），并且它根除了`forName()`方法的调用，所以也更高效。

类字面常量不仅可以用于普通类，还可用于`接口`、`数组`以及`基本数据类型`，对于基本数据类型的包装类，其拥有的标准字段`TYPE`是一个指向对应基本数据类型Class对象的引用，如下所示：

.class|等价于
:--|:--
boolean.class|Boolean.TYPE
char.class|Character.TYPE
byte.class|Byte.TYPE
short.class|Short.TYPE
int.class|Integer.TYPE
long.class|Long.TYPE
float.class|Float.TYPE
double.class|Double.TYPE
void.class|Void.TYPE

为了使用类而做的准备工作实际包含三个步骤：
1. **加载**。由类加载器执行，该步骤将查找字节码，并从这些字节码中创**建一个Class对象**
2. **连接**。在链接阶段将验证类中的字节码，为静态域分配存储空间，如果必需的话，将解析这个类创建的对其他类的所有引用。
3. **初始化**。若该类有父类，则对其初始化，执行静态初始化器和静态初始化块。

初始化有效地实现了尽可能的“惰性”，即延迟到了对静态方法（构造器隐式静态）调用或对非常量静态域首次引用时才执行。

## 14.3 反射：运行时的类信息 ##
RTTI和反射的真正区别在于：对RTTI来说，编译器在编译时打开和检查.class文件。（换句话说，我们可以用“普通”方式调用对象的所有方法。）而对于反射机制来说，.class文件在编译时是不可获取的，所以是在运行时打开和检查.class文件。

## 14.4 动态代理 ##
在之前学到，java中服用代码的方式有组合、继承和代理，一个简单的代理模式是这样的：

```java
interface Interface {
    void doSomething();

    void somethingEsle(String arg);
}

class RealObject implements Interface {

    @Override
    public void doSomething() {
        System.out.println("doSomething");
    }

    @Override
    public void somethingEsle(String arg) {
        System.out.println("somethingElse " + arg);
    }
}

class SimpleProxy implements Interface{

    private Interface proxied;

    public SimpleProxy(Interface proxied) {
        this.proxied = proxied;
    }

    @Override
    public void doSomething() {
        System.out.println("SimpleProxy doSomething");
        proxied.doSomething();
    }

    @Override
    public void somethingEsle(String arg) {
        System.out.println("SimpleProxy somethingElse " + arg);
        proxied.somethingEsle(arg);
    }
}

class SimpleProxyDemo{
    public static void consumer(Interface face){
        face.doSomething();
        face.somethingEsle("billie");
    }

    public static void main(String[] args) {
        consumer(new RealObject());
        consumer(new SimpleProxy(new RealObject()));
    }
}
```
输出：
```
doSomething
somethingElse billie
SimpleProxy doSomething
doSomething
SimpleProxy somethingElse billie
somethingElse billie
```

Java的`动态代理`比代理的思想更前卫一些，因为它可以动态地创建代理并动态地处理对所代理方法的调用。在动态代理上所做的所有调用都会被重定向到单一的`调用处理器`上，它的工作是揭示调用的类型并确定相应的对策，下面是用动态代理重写的`SimpleProxyDemo`

```java
/**
 * 动态代理
 *
 * @author huxiong
 * @date 2016/07/05 16:17
 */
class DynamicProxyHandler implements InvocationHandler {
    private Object proxied;

    public DynamicProxyHandler(Object proxied) {
        this.proxied = proxied;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("**** proxy: " + proxy.getClass() + ". method：" + method + ". args：" + args);
        if (args != null) {
            for (Object arg : args) {
                System.out.println("  " + arg);
            }
        }
        return method.invoke(proxied, args);
    }
}

class DynamicProxyDemo {
    public static void consumer(Interface face) {
        face.doSomething();
        face.somethingEsle("billie");
    }

    public static void main(String[] args) {
        RealObject real = new RealObject();
        consumer(real);
        // 创建动态代理
        Interface proxy = (Interface) Proxy.newProxyInstance(Interface.class.getClassLoader(), new Class[]{Interface.class}, new DynamicProxyHandler(real));
        consumer(proxy);
    }
}
```
输出：
```
doSomething
somethingElse billie
**** proxy: class com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter14.proxy.$Proxy0. method：public abstract void com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter14.proxy.Interface.doSomething(). args：null
doSomething
**** proxy: class com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter14.proxy.$Proxy0. method：public abstract void com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter14.proxy.Interface.somethingEsle(java.lang.String). args：[Ljava.lang.Object;@277889e9
  billie
somethingElse billie
```
