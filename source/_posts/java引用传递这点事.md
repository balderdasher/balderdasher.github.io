---
title: java引用传递这点事
date: 2017-12-27 19:09:23
tags: [java]
categories: java
permalink: pass-value-in-java
---
![pass-value-in-java](/uploads/notes/pass-value-in-java-example.png)
<!--more-->
今天在看`java.io`包时看到一个类`DeleteOnExitHook`，这个类中有如下方法：

```java
static void runHooks() {
    LinkedHashSet<String> theFiles;

    synchronized (DeleteOnExitHook.class) {
        theFiles = files;
        files = null;
    }

    ArrayList<String> toBeDeleted = new ArrayList<>(theFiles);

    // reverse the list to maintain previous jdk deletion order.
    // Last in first deleted.
    Collections.reverse(toBeDeleted);
    for (String filename : toBeDeleted) {
        (new File(filename)).delete();
    }
}
```
这个类的作用是注册一个`shutdown hook`(关闭钩子)，jvm退出时会自动删除指定的文件列表。以上方法在类的静态块代码中注册的关闭钩子中的线程类执行。要删除的文件列表则可以通过`File`类中的`deleteOnExit()`方法添加，此方法会调用`DeleteOnExitHook`类中的静态方法`add()`将文件注册到待删除文件列表。

以上不是重点，重点是以上代码中同步块中的代码，乍一看会产生一种错觉：`theFiles`为`null`了，想想也是啊，A=B,B=0,A岂不就是等于0了？

其实不然，至少在java中不是这样的。以下的程序将会验证
```java
@Test
public void testF(){
    List<String> strs = new ArrayList<>();
    strs.addAll(Arrays.asList("1", "2", "3"));
    List<String> temp = strs;
    strs = null;
    System.out.println(temp);
}
```
以上代码的输出结果会是什么呢？我在群聊中贴出了以上代码，有不少同学的答案是`null`。

实际上的输出是：

> [1, 2, 3]

看来还真的是看起来越简单的东西越容易犯迷糊也越容易忘记，之所以会是以上输出归根结底就是一句话：**Java只有值传递**。
来一张图看得更清楚些：
![pass-value-in-java](/uploads/notes/pass-value-in-java.png)

从图中可以看出，`temp=strs`看起来是把`strs`的引用传递给了`temp`，实际上传递的是在堆中对象的地址，strs的引用传递给temp，意味着temp引用指向的地址与strs的一样，当`strs=null`时，意味着strs不指向任何堆中的对象了。

**在java中，`ref=null`意味着引用ref不指向任何对象，而不是把ref指向的对象设置为null**。如果ref=null就能把它所指对象设为null的话，gc就没有存在的意义了。

对于java中的引用传递，一个更容易理解的例子就是两个贼与一个储物柜的情景，贼`A`(引用a)抢了珠宝店后把珠宝（对象）藏在一个号码为`110`(对象的实际地址)的储物柜(堆)里，他持有这个储物柜的钥匙`key1`，同时贼`B`来接头了，贼`A`配了一把钥匙`key2`给贼`B`，这时候老大来了，他说小A啊，你那把钥匙还留着有点不地道吧，贼A只能把`key1`销毁了(设为null)，这时候相当于贼A与珠宝失去了联系，而贼B可以拿着钥匙打开储物柜。见图：
![pass-value-in-java](/uploads/notes/pass-value-in-java-example.png)


