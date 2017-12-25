---
title: 《Java编程思想》第20章-注解
date: 2017-06-02 11:11:05
tags: [java,读书笔记]
categories: java
permalink: thinking-in-java-annotation
---
> **注解(也被称为*元数据*)为我们在代码中添加信息提供了一种形式化的方法，使我们可以在稍后某个时刻非常方便地使用这些数据**

<!--more-->
## 20.1 基本语法 ##

下面的写法很少见吧:

```java
public class Testable {
    public void execute() {
        System.out.println("Executing...");
    }

    public @Test void testExecute() {
        execute();
    }
}
```

这个`@Test`的位置有点奇怪，平常不都是放在方法上面的吗，其实注解`@Test`可以与任何修饰符共同作用于方法，例如`public`、`static`或`void`，被注解的方法与其他的方法没有区别，从语法角度看，注解的使用方式几乎与修饰符的使用一模一样。

### 20.1.1 定义注解 ###

下面是前面的程序用到的`@Test`注解的定义，注解的定义看起来很像接口的定义。事实上，与其他任何Java接口一样，注解也将会编译成class文件。

```java
/**
 * 注解定义
 * @author mrdios
 * @date 2016-07-26 14:41
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
}
```

除了@符号，`@Test`的定义很像一个空的接口，定义注解时，会需要一些`元注解`（meta-annotation）,如`@Target`和`@Retention`。它们定含义分别是：

- @Target用来定义注解将用于什么地方（例如是一个方法或者一个属性）。
- @Retention用来定义该注解在哪一个级别可用，级别有三个：源代码（SOURCE）、类文件中（CLASS）和运行时（RUNTIME）。

在注解中一般都会包含一些元素表示某些值，当分析处理注解时，程序或工具可以利用这些值，注解的元素看起来就像接口的方法，不同的是注解中可以为其指定默认值。

没有元素的注解称为**标记注解**，上面的`@Test`就是一个标记注解。

下面是一个简单的注解，可以用来跟踪一个项目中的用例：

```java
/**
 * 一个简单的注解
 *
 * @author mrdios
 * @date 2016-07-26 14:51
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UseCase {
    public int id();
    public String description() default "no description";
}
```
在以下类中，有三个方法使用了此注解：

```java
/**
 * "@UseCase"注解的使用
 *
 * @author mrdios
 * @date 2016-07-26 14:57
 */
public class PasswordUtil {
    @UseCase(id = 47, description = "密码必须包含至少一位数字")
    public boolean validatePassword(String password) {
        return (password.matches("\\w*\\d\\w*"));
    }

    @UseCase(id = 48)
    public String encryptPassword(String password) {
        return new StringBuilder(password).reverse().toString();
    }

    public boolean checkForNewPassword(List<String> prevPasswords, String password) {
        return !prevPasswords.contains(password);
    }
}
```

可以看出，注解的元素在使用时表现为`名-值对`的形式，并需要置于`@UseCase`声明之后的括号内，在使用时如果没有给出某个元素的值，则使用这个元素的默认值。

### 20.1.2 元注解 ###

Java目前只内置了三种标准注解（`@OverRide`、`@Deprecated`、`@SuppressWarning`），以及四种元注解。元注解专门负责注解其他注解：

元注解|描述
:---|:---
@Target|表示该注解可以用于什么地方。可能的`ElementType`参数包括：</br>CONSTRUCTOR：构造器的声明<br>FIELD：域声明（包括enum实例）<br>LOCAL_VARIABLE：局部变量声明<br>METHOD：方法声明<br>PACKAGE：包声明<br>PARAMETER：参数声明<br>TYPE：类、接口（包括注解类型）或enum声明
@Retention|表示需要在什么级别保存该注解信息。可选的`RetentionPolicy`参数包括：<br>SOURCE：注解将被编译器丢弃<br>CLASS：注解在class文件中可用，但会被VM丢弃<br>RUNTIME：VM将在运行期也保留注解，因此可以通过反射机制读取注解信息
@Documented|将此注解包含在Javadoc中
@Inherited|允许子类继承父类中的注解

## 20.2 编写注解处理器 ##

一下是一个简单的注解注解处理器，用来查找@UseCase标记，提供了一组id值，它会列出在`PasswrodUtil`中找到的用例，以及缺失的用例：

```java
package com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter20;

import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

/**
 * 注解处理器
 *
 * @author mrdios
 * @date 2016-07-26 15:39
 */
public class UseCaseTracker {
    public static void trackUseCases(List<Integer> useCases,Class<?> c1){
        for (Method m:c1.getDeclaredMethods()){
            UseCase uc = m.getAnnotation(UseCase.class);
            if (uc != null){
                System.out.println("Found Use Case：" + uc.id() + " " + uc.description());
                useCases.remove(new Integer(uc.id()));
            }
        }
        for (int i : useCases){
            System.out.println("Warning：Missing use case-" + i);
        }
    }

    public static void main(String[] args) {
        List<Integer> useCases = new ArrayList<>();
        Collections.addAll(useCases,47,48,49,50);
        trackUseCases(useCases,PasswordUtil.class);
    }
}
```

输出：

```
Found Use Case：49 检查是不是新的密码
Found Use Case：48 no description
Found Use Case：47 密码必须包含至少一位数字
Warning：Missing use case-50
```

### 20.2.1 注解元素 ###

注解元素可用的类型如下所示：

- 所有基本类型(int、float、boolean等)
- String
- enum
- Annotation
- 以上类型的数组

如果使用了以上类型之外的其他类型，编译器就会报错，包括任何包装类型，注解可以嵌套。

### 20.2.2 默认值限制 ###

编译器对注解中元素的默认值有严格的限制：元素不能有不确定的值，要么具有默认值，要么使用注解时提供元素的值。

另外，对于非基本类型的元素，无论是声明还是在注解接口中定义默认值时，都不能以null作为其值，这个约束使得处理器很难表现一个元素的存在或缺失的状态，为了绕开这个约束，可以定义一些特殊的值，例如`空字符串`或者`负数`，以此来表示某个元素不存在

### 20.2.3 生成外部文件 ###

有些框架需要一些额外的信息才能与源代码协同工作，比如hibernate的hbm映射文件，这些描述文件脱离了源代码之外，所以在我们编写了Javabean之后还要忍受编写xml文件的烦恼，于此同时，同一个类拥有了两个单独的信息源，经常也会导致代码同步问题。

每当上述情况出现的时候，注解的价值就体现出来了，比如我们希望提供一些基本的对象/关系映射功能，能够自动生成数据库表，我们可以抛弃xml描述文件的方式，使用注解，将所有的信息保留在JavaBean源文件中，以下就是用注解的实现方式：

首先，需要一个新的注解，用来定义与Bean关联的数据库表的名字，以下的注解告诉处理器，你需要为我生成一个数据库表：

```java
/**
 * 数据库表的注解，用于实体bean类上，说明该实体生成的数据库表信息
 * @author mrdios
 * @date 2016-07-26 16:17
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface DBTable {
    public String name() default "";
}
```

接下来就是javaBean的注解，用于描述JavaBean中的属性与生成数据库表中的字段的一些联系：

```java
/**
 * 表字段约束注解
 *
 * @author mrdios
 * @date 2016-07-26 16:21
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Constraints {
    /* 是否主键 */
    boolean primaryKey() default false;

    /* 是否允许为空 */
    boolean allowNull() default true;

    /* 是否唯一 */
    boolean unique() default false;
}

/**
 * sql类型
 * @author mrdios
 * @date 2016-07-26 16:26
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SQLString {
    int value() default 0;
    String name() default "";
    Constraints constraints() default @Constraints;
}

/**
 * sql类型
 * @author mrdios
 * @date 2016-07-26 16:28
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SQLInteger {
    String name() default "";
    Constraints constraints() default @Constraints;
}
```

接下来是一个简单的Bean定义，使用了以上这些注解：

```java
/**
 * JavaBean
 *
 * @author mrdios
 * @date 2016-07-26 16:33
 */
@DBTable(name = "MEMBER")
public class Member {
    @SQLString(30)
    private String firstName;
    @SQLString(50)
    private String lastName;
    @SQLInteger
    private Integer age;
    @SQLString(value = 30, constraints = @Constraints(primaryKey = true))
    private String handle;
    static int memberCount;

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public Integer getAge() {
        return age;
    }

    public String getHandle() {
        return handle;
    }

    @Override
    public String toString() {
        return handle;
    }
}
```
### 20.2.4 注解不支持继承 ###

不能使用`extends`关键字来继承某个@interface。也就是说：Java在定义注解时，不能继承自某个其他的注解。

### 20.2.5 实现处理器 ###

以下是一个针对上面对象/关系映射注解处理器的例子，将读取一个类文件，检查其中的数据库注解，并生成用来创建数据库的SQL语句：

```java
package com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter20.framework;

import java.lang.annotation.Annotation;
import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

/**
 * 注解处理器
 *
 * @author mrdios
 * @date 2016-07-26 17:01
 */
public class TableCreator {
    public static void main(String[] args) throws Exception {
        List<Class<?>> beanClasses = new ArrayList<>();
        Collections.addAll(beanClasses, Member.class);
        process(beanClasses);
    }

    public static void process(List<Class<?>> beanClasses) throws Exception {
        for (Class c1 : beanClasses) {
            Class<?> c = Class.forName(c1.getName());
            DBTable dbTable = c.getAnnotation(DBTable.class);
            if (dbTable == null) {
                System.out.println("No DBTable annotations in class " + c.getName());
                continue;
            }
            String tableName = dbTable.name();
            if (tableName.length() < 1) {
                tableName = c.getSimpleName().toUpperCase();
            }
            List<String> columnDefs = new ArrayList<>();
            for (Field field : c.getDeclaredFields()) {
                String columnName;
                Annotation[] anns = field.getDeclaredAnnotations();
                if (anns.length < 1) {
                    continue;
                }
                if (field.isAnnotationPresent(SQLInteger.class)) {
                    SQLInteger sInt = field.getAnnotation(SQLInteger.class);
                    if (sInt.name().length() < 1) {
                        columnName = field.getName().toUpperCase();
                    } else {
                        columnName = sInt.name();
                    }
                    columnDefs.add(columnName + " INT" + getConstraints(sInt.constraints()));
                }
                if (field.isAnnotationPresent(SQLString.class)) {
                    SQLString sString = field.getAnnotation(SQLString.class);
                    if (sString.name().length() < 1) {
                        columnName = field.getName().toUpperCase();
                    } else {
                        columnName = sString.name();
                    }
                    columnDefs.add(columnName + " VARCHAR(" + sString.value() + ")" + getConstraints(sString.constraints()));
                }
                StringBuilder createCommand = new StringBuilder("CREATE TABLE " + tableName + "(");
                for (String columnDef : columnDefs) {
                    createCommand.append("\n   ").append(columnDef).append(",");
                }
                String tableCreate = createCommand.substring(0, createCommand.length() - 1) + ");";
                System.out.println("Table Creation SQL for " + c.getName() + " is : \n" + tableCreate);
            }
        }
    }

    private static String getConstraints(Constraints con) {
        String constraints = "";
        if (!con.allowNull()) {
            constraints += " NOT NULL";
        }
        if (con.primaryKey()) {
            constraints += " PRIMARY KEY";
        }
        if (con.unique()) {
            constraints += " UNIQUE";
        }
        return constraints;
    }
}
```

输出：

```sql
Table Creation SQL for com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter20.framework.Member is : 
CREATE TABLE MEMBER(
   FIRSTNAME VARCHAR(30));
Table Creation SQL for com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter20.framework.Member is : 
CREATE TABLE MEMBER(
   FIRSTNAME VARCHAR(30),
   LASTNAME VARCHAR(50));
Table Creation SQL for com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter20.framework.Member is : 
CREATE TABLE MEMBER(
   FIRSTNAME VARCHAR(30),
   LASTNAME VARCHAR(50),
   AGE INT);
Table Creation SQL for com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter20.framework.Member is : 
CREATE TABLE MEMBER(
   FIRSTNAME VARCHAR(30),
   LASTNAME VARCHAR(50),
   AGE INT,
   HANDLE VARCHAR(30) PRIMARY KEY);
```

