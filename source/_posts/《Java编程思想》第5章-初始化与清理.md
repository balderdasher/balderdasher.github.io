---
title: 《Java编程思想》第5章-初始化与清理
date: 2017-05-12 13:26:45
tags: [java,读书笔记]
categories: java
permalink: thinking-in-java-init-and-clear
---
## 重载（overload）

> 重载，简单说，就是函数或者方法有相同的名称，但是参数列表不相同的情形，这样的同名不同参数的函数或者方法之间，互相称之为重载函数或者方法。

<!-- more -->
其中，参数列表不相同分为以下几种情况：
1. 参数个数
2. 参数类型
3. 参数顺序(`一般情况下别这么做，因为这回使代码难以维护`)
为什么我们不能根据函数或者方法的`返回值`来区分是否重载呢，这是因为有时候我们并不关心方法的返回值，就是单纯地调用一下函数或者方法而已，就像这样：
`fn();`
这时我们忽略了方法的返回值，所以java无法判断到底应该调用哪个`fn()`,别人也无法理解这样的代码，因此，根据方法的返回值来区分重载方法是行不通的。

## this关键字

> `this`关键字只能在`方法内部`使用,表示对`"调用方法的那个对象"`的引用

*注意*：在方法内部调用同一个类的另一个方法时没有必要使用`this`，直接调用即可，当前方法中的`this`引用会自动应用于同一个类中的其它方法，所以可以这样写代码：

```java
public class Apricot {
	void pick(){/* ... */}
	void pit(){pick();/* ... */}//此处不必写：this.pick();
}
```

只有当需要明确指出对当前对象的引用时，才需要使用`this`关键字，例如，当需要返回当前对象的引用时常常在`return`语句里这样写：

```java
public class Leaf {
	int i = 0;
	Leaf increment() {
		i++;
		return this;
	}
}
```
这种方式在我们需要在一条语句里对同一个对象执行多次操作时非常有用，类似于这样：

```java
student.getUp().dressUp().getWashed().makeUp().gotoSchool();
```

### 在构造器中调用构造器

> 一个类有多个构造器时，有时我们需要在一个构造器中调用另一个构造器，因为这样可以避免重复的代码

```java
public class Student{
	private String name;
	private int age;
	//构造器1
	public Student(String name){
		this.name = name;
	}
	//构造器2
	public Student(String name,int age){
		this(name);//不用this则为this.name=name
		this.age = age;
	}
}
```

### 静态数据的初始化

> - 无论创建多少个对象，静态数据都只占用一分存储区域，`static`关键字不能用于局部变量，只能作用于域，基本类型的静态变量在没有对它进行初始化时，它们会获得基本类型的默认值，如果是一个引用对象，那么它的默认初始化值就是`null`
> - 静态初始化只有在必要时刻才会进行，意味着只有当有引用指向一个类时，这个类中的静态变量才会被初始化，此后，静态对象不会再被初始化


