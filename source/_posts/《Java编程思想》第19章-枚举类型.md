---
title: 《Java编程思想》第19章-枚举类型
date: 2017-06-01 11:11:05
tags: [java,读书笔记]
categories: java
permalink: thinking-in-java-enum
---
> 关键字enum可以将一组具名的值的有限集合创建为一种新的类型，这些具名的值可以作为常规的程序组件使用，这就是枚举类型的本质。

<!--more-->
## 19.1 enum基本特性 ##

```java
package com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter19;

/**
 * 枚举类功能演示
 * Created by mrdios on 2016/7/21.
 */
public class EnumClass {
    public static void main(String[] args) {
        for (Shrubbery s:Shrubbery.values()){
            // 值在定义时候的顺序,从0开始
            System.out.println(s + " ordinal: " + s.ordinal());
            // enum类实现了Comparable接口，还实现了Serializable接口
            System.out.println(s.compareTo(Shrubbery.CRAWLING));
            // 使用==来比较enum实例，equals()和hashCode()方法由编译器自动提供
            System.out.println(s == Shrubbery.CRAWLING);
            System.out.println(s.equals(Shrubbery.CRAWLING));
            // 通过调用getDeclaringClass得到实例所在的enum类
            System.out.println(s.getDeclaringClass().getSimpleName());
            System.out.println(s.name());
            System.out.println("----------------------");
        }
        // 从一个字符串构造一个枚举
        for (String s:"HANGING CRAWLING GROUND".split(" ")){
            Shrubbery shrub = Enum.valueOf(Shrubbery.class,s);
            System.out.println(shrub);
        }
    }
}

enum Shrubbery{GROUND,CRAWLING,HANGING}
```
输出：
```
GROUND ordinal: 0
-1
false
false
Shrubbery
GROUND
----------------------
CRAWLING ordinal: 1
0
true
true
Shrubbery
CRAWLING
----------------------
HANGING ordinal: 2
1
false
false
Shrubbery
HANGING
----------------------
HANGING
CRAWLING
GROUND
```

## 19.2 向enum中添加新方法 ##

> 除了不能继承另一个`enum`外，我们可以将`enum`看做一个常规的类，可以向其中添加新方法，甚至可以有`main()`方法。

```java
/**
 * enum中添加新方法
 * Created by mrdios on 2016/7/21.
 */
public enum OfferType {
    // 实例的定义必须在最方法或者属性之前，否则会编译错误
    FOOD("美食"),
    HAPPY("娱乐"),
    STAY("住宿"),
    SHOPPING("购物"),
    TRIP("出行"),
    LIFE("生活服务"),
    OTHER("其它");

    private String desc;

    // 构造器必须是private的或者是包访问权限(ide已建议为包访问权限)的
    OfferType(String desc) {
        this.desc = desc;
    }

    public String getDesc() {
        return desc;
    }

    // enum中可以有main()方法
    public static void main(String[] args) {
        for (OfferType o : OfferType.values()) {
            System.out.println(o + ": " + o.getDesc());
        }
    }
}
```
输出：
```
FOOD: 美食
HAPPY: 娱乐
STAY: 住宿
SHOPPING: 购物
TRIP: 出行
LIFE: 生活服务
OTHER: 其它
```

从程序中可以看出，当为enum添加属性或者方法时，正确的姿势是在定义完实例之后加上分号，然后再添加属性或者方法，如果在实例之前出现任何属性和方法的定义，将会得到编译错误，另外，enum的构造器建议为private或者default的，因为即使我们将之声明为public的，我们也只能在enum定义的内部使用其构造器创建enum实例。一旦enum的定义结束。编译器就不允许再使用其构造器来创建任何实例了，所以所以声明为public是毫无意义的。

## 19.3 switch语句中的enum ##

为什么switch语句中可以使用enum呢？

一般来说，在switch中只能使用整数值，而枚举类型本身就具备整数值的次序，并且可以通过`ordinal()`方法取得次序，所以肯定是编译器帮我们做了某些工作让switch语句使用了enum中的整数值次序，所以我们才可以在switch语句中使用enum。

一个红绿灯（小型状态机）的例子：

```java
/**
 * switch语句中的enum
 * Created by mrdios on 2016/7/21.
 */
enum Signal {
    GREEN, YELLOW, RED
}

public class TrafficLight {
    Signal color = Signal.RED;
    public void change() {
        switch (color) {
            // 在case语句中不必使用enum类型来修饰一个enum实例如此处不必用Signal.RED
            case RED:
                color = Signal.GREEN;
                break;
            case GREEN:
                color = Signal.YELLOW;
                break;
            case YELLOW:
                color = Signal.RED;
                break;
        }
    }

    @Override
    public String toString() {
        return "The traffic light is " + color;
    }

    public static void main(String[] args) {
        TrafficLight t = new TrafficLight();
        for (int i = 0; i < 7; i++) {
            System.out.println(t);
            t.change();
        }
    }
}
```
输出：
```
The traffic light is RED
The traffic light is GREEN
The traffic light is YELLOW
The traffic light is RED
The traffic light is GREEN
The traffic light is YELLOW
The traffic light is RED
```

## 19.4 神秘的values() ##

编译器为我们创建的enum类都继承自Enum类，但从Enum类的源码来看，它并没有`values()`方法，但是为什么我们又可以使用这个方法呢？难道存在某种“隐藏的”方法吗？

利用反射机制已经证明：`values()`方法是由编译器添加的`static`方法，而且编译器还添加了`valueOf()`方法，这个valueOf()与Enum中原有的valueOf()不同，这个方法只需一个参数，而enum原有的需要两个参数。

## 19.5 实现，而非继承 ##

因为所有的enum都继承自`java.lang.Enum`类，因为Java不支持多重继承，所以我们新建enum时不能再继承其他的类，但是我们可以同时实现一个或多个接口：

```java
import org.springframework.ui.Model;
import javax.servlet.http.HttpServletRequest;
/**
 * 通知接口
 * @author mrdios
 */
public interface NotifyInterface {
    void doNotify(Model model);
	void doNotify(HttpServletRequest request);
}

/**
 * 通知类型
 * @author mrdios
 */
public enum NoticeType {
    SUCCESS("成功","成功"),
    ERROR("错误","错误"),
    WARN("警告","警告"),
    INFO("提示","提示");

    private String name;
    private String desc;

    NoticeType(String name, String desc) {
        this.name = name;
        this.desc = desc;
    }

    /** getters and setters **/
}

import org.springframework.ui.Model;
import javax.servlet.http.HttpServletRequest;

/**
 * 单例通知，用于提示
 * 0:弹出框和页面提示均需设置"infoTip"参数用于展示提示信息
 * 1:弹出框提示须设置"type"参数用于artDialog展示不同的样式、"redirectUrl"参数可选决定关闭弹出框时的跳转链接(x)
 * 2:页面提示时设置"title"参数设置页面的title属性
 *
 * @author mrdios
 */
public enum Notification implements NotifyInterface {
	NOTICE {
		@Override
        public void doNotify(Model model) {
            model.addAttribute("infoTip", this);
        }
	    @Override
	    public void doNotify(HttpServletRequest request) {
		    request.setAttribute("infoTip", this);
	    }
    };

	// 后台通知页面
	public static final String BACK_NOTIFY_VIEW = "info";
    private NoticeType type;
    private String title;
    private String infoTip;
    private String redirectUrl;
    private String callBack;//附加回调参数
	private int historyGo = -1;//页面中history.go(int)的参数，仅当redirectUrl为空时此字段才有用，默认-1，即回退1个历史记录

    /**
     * 通知后擦黑板，防止当前通知影响下一个通知
     *
     * @return this
     */
    public void reset() {
        this.type = null;
        this.callBack = null;
        this.title = null;
        this.infoTip = null;
        this.redirectUrl = null;
	    this.historyGo = -1;
    }


    public String getCallBack() {
        return callBack;
    }

    public Notification setCallBack(String callBack) {
        this.callBack = callBack;
        return this;
    }

    public String getInfoTip() {
        return infoTip;
    }

    public Notification setInfoTip(String infoTip) {
        this.infoTip = infoTip;
        return this;
    }

    public String getRedirectUrl() {
        return redirectUrl;
    }

    public Notification setRedirectUrl(String redirectUrl) {
        this.redirectUrl = redirectUrl;
        return this;
    }

    public String getTitle() {
        return title;
    }

    public Notification setTitle(String title) {
        this.title = title;
        return this;
    }

    public NoticeType getType() {
        return type;
    }

    public Notification setType(NoticeType type) {
        this.type = type;
        return this;
    }

	public int getHistoryGo() {
		return historyGo;
	}

	public Notification setHistoryGo(int historyGo) {
		this.historyGo = historyGo;
		return this;
	}
}
```

以上程序是我为Java web后台设计的一个通知程序，例如后台成功添加一条数据以弹出框形式提示保存成功，我们在程序中保存完数据之后携带一个通知跳转到设置的通知页面`info.html`，此页面上只有一段用于弹出框的js，弹出框的各种参数来源于从通知中取出来的参数，比如弹出框显示的图标取决于通知的类型等。下面是使用示例：

```java
if (null == activity) {
    Notification.NOTICE.setType(NoticeType.ERROR).setInfoTip("活动不存在！").doNotify(model);
    return Notification.BACK_NOTIFY_VIEW;
}
```

从程序中可以看到，enum可以实现一个或多个接口，如果实现了某个接口，那么所有enum的实例都必须实现接口中的所有方法（上面是单例）。

## 19.6 使用接口组织枚举 ##

有时候我们希望使用子类将一个enum中的元素进行分组，在一个接口内部，创建实现该接口的枚举，以此将元素进行分组，可以达到将枚举元素分类组织的目的，比如想用enum来表示不同类别的商品，同时还希望每个enum元素仍然保持`Goods`类型，可以这样实现：

```java
/**
 * 接口组织枚举，实现接口是enum子类化的唯一办法
 * Created by mrdios on 2016/7/21.
 */
interface Goods {
    enum Office implements Goods {COMPUTER, GAME;}
    enum Food implements Goods {FRUIT, SEAFOOD, MEAT, WINE}
    enum Book implements Goods {LIFE, EDUCATION, TECHNOLOGY}
}

public class GoodsType {
    public static void main(String[] args) {
        Goods goods = Goods.Office.COMPUTER;
        goods = Goods.Food.FRUIT;
        goods = Goods.Book.TECHNOLOGY;
    }
}
```

## 19.7 使用EnumSet替代标志 ##

> Set是一种包含不重复对象的集合，enum也要求其中的成员是唯一的，所以enum看起来也具有集合的行为。但由于不能从enum中删除或添加元素，所以它只能算是用处不大的集合，Java引入`EnumSet`是为了通过enum创建一种替代传统基于`int`“标志位”的替代品，这种标志可以用来表示某种“开/关”信息，不过使用这种标志最终操作的只是一些`bit`，而不是这些bit想要表达的概念，所以也很容易写出令人难以理解的代码。

EnumSet因为必须与非常高效的bit标志竞争，所以它非常快。它的优点在于：在说明一个二进制位是否存在时，具有更好的表达能力，并且无需担心性能。

EnumSet中的元素必须来自于一个enum，下面的例子表示大楼中警报器的安放位置,然后用`EnumSet`来跟踪报警器的状态：

```java
/**
 * 大楼中警报传感器的安放位置
 * Created by mrdios on 2016/7/21.
 */
public enum AlarmPoints {
    STAIR1, STAIR2, LOBBY, OFFICE1, OFFICE2, OFFICE3, OFFICE4, BATHROOM, UTILITY, KITCHEN;
}

/**
 * EnumSet跟踪警报器状态
 * Created by mrdios on 2016/7/21.
 */
public class EnumSets {
    public static void main(String[] args) {
        EnumSet<AlarmPoints> points = EnumSet.noneOf(AlarmPoints.class); // empty set
        points.add(AlarmPoints.BATHROOM);
        System.out.println(points);
        points.addAll(EnumSet.of(AlarmPoints.STAIR1, AlarmPoints.STAIR2, AlarmPoints.KITCHEN));
        System.out.println(points);
        points = EnumSet.allOf(AlarmPoints.class);
        points.removeAll(EnumSet.of(AlarmPoints.STAIR1, AlarmPoints.STAIR2, AlarmPoints.KITCHEN));
        System.out.println(points);
        points.removeAll(EnumSet.range(AlarmPoints.OFFICE1, AlarmPoints.OFFICE4));
        System.out.println(points);
        points = EnumSet.complementOf(points);//取补集
        System.out.println(points);
    }
}
```
输出：
```
[BATHROOM]
[STAIR1, STAIR2, BATHROOM, KITCHEN]
[LOBBY, OFFICE1, OFFICE2, OFFICE3, OFFICE4, BATHROOM, UTILITY]
[LOBBY, BATHROOM, UTILITY]
[STAIR1, STAIR2, OFFICE1, OFFICE2, OFFICE3, OFFICE4, KITCHEN]
```

观察以上程序的输出也可以看出一个规律：无论以什么样的顺序从EnumSet中添加或删除元素，最终元素的顺序与enum中的保持一致性。

## 19.8 使用EnumMap ##

> EnumMap是一种特殊的Map，它要求其中的键(key)必须来自一个enum。由于enum本身的限制，所以EnumMap在内部可由数组实现，所以EnumMap的速度很快，我们可以使用enum实例在EnumMap中进行查找操作，但也只能将enum的实例作为键来调用put()方法，其他的操作跟一样的Map差不多。

以下是一个**命令设计模式**的用法，命令设计模式首先需要一个只有单一方法的接口，然后从该接口实现具有各自不同行为的多个子类，然后我们就可以构造命令对象，并在需要的时候使用它们：

```java
/**
 * 使用EnumMap
 * Created by mrdios on 2016/7/22.
 */
interface Command{void action();}

public class EnumMaps {
    public static void main(String[] args) {
        EnumMap<AlarmPoints,Command> em = new EnumMap<>(AlarmPoints.class);
        em.put(AlarmPoints.KITCHEN, new Command() {
            @Override
            public void action() {
                System.out.println("厨房着火了");
            }
        });
        em.put(AlarmPoints.BATHROOM, new Command() {
            @Override
            public void action() {
                System.out.println("浴室起火了");
            }
        });
        for (Map.Entry<AlarmPoints,Command> e:em.entrySet()){
            System.out.println(e.getKey());
            e.getValue().action();
        }
        try {
            em.get(AlarmPoints.UTILITY).action();
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
```
输出：
```
BATHROOM
浴室起火了
KITCHEN
厨房着火了
java.lang.NullPointerException
```
## 19.9 常量相关的方法 ##

Java的enum允许我们为enum实例编写方法，从而为每个enum实例赋予各自不同的行为，实现常量相关的方法有两种方案：

1. 为enum定义一个或多个`abstract`方法，然后为每个enum实例实现该抽象方法。
2. 使enum实现一个或多个接口，然后在每个enum实例中实现接口中的方法(10.5节已演示)。

综合来看，第一种方案比较灵活，因为它不需要新建接口，直接在enum中定义每个实例需要实现的抽象方法就可以了，下面是一个例子：

```java
/**
 * 常量相关的方法
 * Created by mrdios on 2016/7/22.
 */
public enum SystemConfig {
    DATE_TIME {
        @Override
        String getInfo() {
            return DateFormat.getDateInstance().format(new Date());
        }
    },
    CLASSPATH {
        @Override
        String getInfo() {
            return System.getenv("CLASSPATH");
        }
    },
    VERSION {
        @Override
        String getInfo() {
            return System.getProperty("java.version");
        }
    };

    abstract String getInfo();

    public static void main(String[] args) {
        for (SystemConfig config : values()) {
            System.out.println(config.getInfo());
        }

    }
}
```
输出：
```
2016-7-22
.;C:\Program Files\Java\jdk1.7.0_65\lib\dt.jar;C:\Program Files\Java\jdk1.7.0_65\lib\tools.jar;
1.7.0_65
```

在以上代码中，通过相应的enum实例调用其上的方法，这也成为**表驱动的代码**，与上面的命令设计模式有相似之处。

每个enum实例具备自己独特的行为，似乎说明每个enum实例就像一个独特的类，以上例子中enum实例似乎被当做其“超类”`SystemConfig`来使用，在调用`getInfo()`方法时，体现出多态行为。但是enum实例与类的相似之处也仅限于此了，事实证明我们并不能真的将enum实例作为一个类型来使用。虽然如此，还是可以将其看做是类。



