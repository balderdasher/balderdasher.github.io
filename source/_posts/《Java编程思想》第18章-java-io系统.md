---
title: 《Java编程思想》第18章-java io系统
date: 2017-05-31 11:11:05
tags: [java,io,读书笔记]
categories: java
permalink: thinking-in-java-io
---
## 18.1 File类 ##
File类是一个实用类库工具，可以帮助我们处理文件目录问题。File（文件）类既能代表一个特定文件的*名称*，又能代表一个目录下的一组文件的名称。

<!--more-->
### 18.1.1 目录列表器 ###
查看一个目录列表有两种方式：

1. 调用不带参数的`list()`方法，可以获得File对象包含的全部列表。
2. 使用“目录过滤器”来获取一个受限列表，如得到所有扩展名为`.java`的文件。

```java
/**
 * Created by mrdios on 2016/7/19.
 */
public class DirList {
    public static void listView(String[] args) {
        File path = new File(".");
        String[] list;
        if (args.length == 0) {
            list = path.list();
        } else {
            list = path.list(filter(args[0]));
        }
        Arrays.sort(list, String.CASE_INSENSITIVE_ORDER);
        for (String dirItem : list) {
            System.out.println(dirItem);
        }
    }

    public static FilenameFilter filter(final String regex){
        return new FilenameFilter() {
            private Pattern pattern = Pattern.compile(regex);
            @Override
            public boolean accept(File dir, String name) {
                return pattern.matcher(name).matches();
            }
        };
    }

    public static void main(String[] args) {
        listView(new String[]{".java"});
    }
}
```

以上代码中，把`filter`传递给`list()`使用，使`list()`可以回调filter中的`accept()`方法，这种结构称为**回调**，这也是**策略模式**的一种形式：list()实现了基本的功能，按照`FilenameFilter`的形式提供了这个**策略**，意味着可以传递实现了`FilenameFilter`接口的任何类的对象，用于选择`list()`方法的行为方式，策略的目的就是提供了代码行为的灵活性。

list()方法会为此目录下的每个文件名调用`accept()`来判断该文件是否包含在内。

## 18.2 输入和输出 ##

> 编程语言的I/O类库中的流是一个抽象的概念，它代表任何有能力产出数据的数据源对象或是有能力接受数据的接收端对象。“流”屏蔽了实际的I/O设备中处理数据的细节。
> Java的I/O类库分为输入和输出两部分，按照类的功能进行分类，与输入有关的所有类都应该从`InputStream`继承，而与输出有关的所有类都应该从`OutputStream`继承。

### 18.2.1 InputStream类型 ###

InputStream的作用是用来表示那些从不同数据源产生输入的类，这些数据源包括：

1. 字节数组
2. String对象
3. 文件
4. “管道”，工作方式与实际管道相似（从一端输入，从另一端输出）
5. 一个由其他种类的流组成的序列，以便我们可以将它们随机合并到一个流内
6. 其他数据源，如Internet连接等

每一种数据源都有相应的`InputStream`子类，`FilterInputStream`也属于一种InputStream，为“装饰器”(decorator)类提供基类，其中，“装饰器”类可以把属性或有用的借口与输入流连接在一起。

**表18-1 InputStream类型**

类|功能|构造器参数/如何使用
:--|:--|:--
ByteArrayInputStream|允许将内存的缓冲区当做InputStream使用|缓冲区，字节将从中取出作为一种数据源：将其与FilterInputStream对象相连以提供有用借口
StringBufferInputStream|将String转换成InputStream|字符串。底层实现实际使用StringBuffer作为一种数据源：将其与FilterInputStream对象相连以提供有用接口
FileInputStream|用于从文件中读取信息|字符串，表示文件名、文件或FileDescriptor对象，作为一种数据源：将其与FilterInputStream对象相连以提供有用接口
PipedInputStream|产生用于写入相关PipedOutputStream的数据，实现“管道化”概念|PipedOutputStream，作为多线程中数据源：将其与FilterInputStream对象相连以提供有用接口
SequenceInputStream|将两个或多个InputStream对象转换成单一InputStream|两个InputStream对象或一个容纳InputStream对象的容器Enumeration，作为一种数据源：将其与FilterInputStream对象相连以提供有用接口
FilterInputStream|抽象类，作为“装饰器”的接口，其中“装饰器”为其他的InputStream类提供有用功能|见表18-3

### 18.2.2 OutputStream类型 ###

该类别的类决定了输出所要去往的目标：字节数组、文件或管道。`FilterOutputStream`为“装饰器”类提供了一个基类。

**表 18-2 OutputStream类型**

类|功能|构造器参数/如何使用
:--|:--|:--
ByteArrayOutputStream|在内存中创建缓冲区，所有送往“流”的数据都要放在此缓冲区|缓冲区初始化尺寸（可选）；用于指定数据的目的地：将其与FilterOutPutStream对象相连以提供有用接口
FileOutputStream|用于将信息写至文件|字符串，表示文件名、文件或FileDescriptor对象；指定数据的目的地：将其与FilterOutPutStream对象相连以提供有用接口
PipedOutputStream|任何写入其中的信息都会自动作为相关PipedInputStream的输出，实现“管道”概念|PipedInputStream；指定用于多线程的数据的目的地：将其与FilterOutPutStream对象相连以提供有用接口
FilterOutputStream|抽象类，作为“装饰器”的接口，其中“装饰器”为其他OutputStream提供有用功能|见表18-4

## 18.3 添加属性和有用的接口 ##

Java的I/O类库需要多种不同功能的组合，所以用到了装饰器模式，这也是io类库里存在`filter`(过滤器)
的原因，抽象类`filter`是所有装饰器类的基类。但是装饰器有个缺点，它在给我们提供灵活性的同时也增加了代码的复杂性，Java I/O类库操作不便的原因就是我们必须创建许多类——“核心” I/O类型加上所有的装饰器，才能得到我们希望的单个I/O对象。

FilterInputStream和FilterOutputStream是用来提供装饰器类接口以控制特定输入流（InputStream）和输出流（OutputStream）的两个类。它们分别继承自Java I/O类库中的基类`InputStream`和`OutputStream`，这两个类是装饰器的必要条件（以便能为所有正在被装饰的对象提供通用接口）。

### 18.3.1 通过FilterInputStream从InputStream读取数据 ###

FilterInputStream类能够完成两件完全不同的事情。其中`DataInputStream`允许我们读取不同的基本类型数据以及`String`对象（所有方法都以“read”开头，如`readByte()、readFloat()`）。配合相应的`DataOutputStream`，就可以通过数据“流”将基本数据类型的数据从一个地方迁移到另一个地方。

其他FilterInputStream类则在内部修改`InputStream`的行为方式：是否缓冲，是否保留读过的行（允许查询或设置行数），以及是否把单一字符推回输入流等

**表18-3 FilterInputStream类型**

类|功能|构造器参数/如何使用
:--|:--|:--
DataInputStream|与DataOutputStream搭配使用，因此可以按照可移植方式从流读取基本数据类型|InputStream；包含用于读取基本类型数据的全部接口
BufferedInputStream|使用它可以防止每次读取时都进行实际写操作。代表“使用缓冲区”|InputStream，可以指定缓冲区大小（可选）；本质上不提供接口，只不过是向进程中添加缓冲区所必需的，与接口对象搭配
LineNumberInputStream|跟踪输入流中的行号，可调用`getLineNumber()`和`setLineNumber(int)`|InputStream；仅增加了行号，因此可能要与接口对象搭配使用
PushBackInputStream|具有“能弹出一个字节的缓冲区”。因此可以将读到的最后一个字符回退|InputStream；通常作为编译器的扫描器，因为Java编译器的需要所以出现此类，我们永远不会用到

### 18.3.2 通过FilterOutputStream向OutPutStream写入 ###

与`DataInputStream`对应的`DataOutputStream`可以将各种基本数据类型以及String对象格式化输出到“流”中，这样一来，任何机器上的任何`DataInputStream`都能够读取它们。所有方法都以“write”开头，如`writeByte()`、`writeFloat()`等。

**表18-4 FilterOutputStream类型**

类|功能|构造器参数/如何使用
:--|:--|:--
DataOutputStream|与`DataInputStream`搭配使用，因此可以按照可移植方式向流中写入基本类型数据|OutputStream；包含用于写入基本类型数据的全部接口
PrintStream|用于产生格式化输出。其中`DataOutputStream`处理数据的`存储`，`PrintStream`处理显示|OutputStream，可以用Boolean值指示是否在每次换行时清空缓存（可选）应该是对`OutputStream`对象的“final”封装，可能会经常使用
BufferedOutputStream|使用它以避免每次发送数据是都要进行实际的写操作。代表“使用缓冲区”可以使用`flush()`清空缓冲区|OutputStream，可以指定缓冲区大小（可选）；本质上并不提供接口，只不过是向进程中添加缓冲区所必需的。与接口对象搭配

## 18.4 Reader和Writer ##

虽然一些原始的流类库不再被使用，但是`InputStream`和`OutputStream`在以面向字节形式的I/O中仍可以提供极有价值的功能，`Reader`和`Writer`则提供兼容`Unicode`与面向字符的I/O功能，另外：

1. Java 1.1向InputStream和OutputStream继承结构中添加了一些新类，所以这俩类是不会被取代的
2. 有时我们必须把来自“字节”层次结构中的类和“字符”层次结构中的类结合起来使用。为实现这个目的，要用到“适配器”（adapter）类：`InputStreamReader`可以把`InputStream`转换为`Reader`，而`OutPutStreamWriter`可以把`OutputStream`转换为`Writer`。

设计`Reader`和`Writer`主要是为了国际化。老的I/O继承层次结构仅支持8位字节流，并且不能很好地处理16位的Unicode字符。由于Unicode用于字符国际化，所以添加Reader和Writer继承层次结构就是为了在所有的I/O操作中都支持Unicode。

### 18.4.1 数据的来源和去处 ###

我们应该尽量尝试用Reader和Writer，一旦代码无法成功编译，再考虑用面向字节的类库。

下表展示了两个继承结构中数据的来源和去处（即数据物理上来自哪里去向哪里）之间的对应关系：


来源与去处：Java 1.0类|相应的Java1.1类
:--|:--
InputStream|reader；适配器：InputStreamReader
OutputStream|Writer；适配器：OutPutStreamWriter
FileInputStream|FileReader
FileOutputStream|FileWriter
StringBufferInputStream(已弃用)|StringReader
无|StringWriter
ByteArrayInputStream|CharArrayReader
ByteArrayOutputStream|CharArrayWriter
PipedInputStream|PipedReader
PipedOutputStream|PipedWriter

## 18.5 I/O流的典型使用方式 ##

### 18.5.1 缓冲输入文件 ###

```java
/**
 * 缓冲输入文件
 * Created by mrdios on 2016/7/19.
 */
public class BufferedInputFile {
    public static String read(String fileName) throws IOException{
        BufferedReader in = new BufferedReader(new FileReader(new File(fileName)));
        String s;
        StringBuffer sb = new StringBuffer();
        while ((s = in.readLine()) != null){
            sb.append(s + "\n");
        }
        in.close();
        return sb.toString();
    }

    public static void main(String[] args) throws IOException {
        System.out.println(read("C:\\think\\test.txt"));
    }
}
```

输出：
```
hello everybody
Today,the rain in Beijing is really big.
```
### 18.5.2 从内存输入 ###

```java
/**
 * 从内存输入
 * Created by mrdios on 2016/7/20.
 */
public class MemoryInput {
    public static void main(String[] args) throws IOException {
        StringReader in = new StringReader(BufferedInputFile.read("C:\\think\\test.txt"));
        int c;
        while ((c = in.read()) != -1) {
            System.out.print((char) c);//in.read()以int方式返回下一下一字节，所以要转换为char
        }
    }
}
```
输出:
```
hello everybody
Today,the rain in Beijing is really big.
```

### 18.5.3 格式化的内存输入 ###

要读取格式化数据，可以使用`DataInputStream`，它是面向字节的类（不是面向字符），因此要使用`InputStream`类而不是`Reader`类。

```java
/**
 * 格式化的内存输入
 * Created by mrdios on 2016/7/20.
 */
public class FormattedMemoryInput {
    public static void main(String[] args) throws IOException {
        try {
            DataInputStream in = new DataInputStream(new ByteArrayInputStream(BufferedInputFile.read("C:\\think\\test.txt").getBytes()));

            while (true){
                System.out.print((char)in.readByte());
            }
        } catch (IOException e) {
            System.err.println("End of stream");
        }
    }
}
```

### 18.5.4 基本的文件输出 ###

```java
/**
 * 基本的文件输出
 * Created by mrdios on 2016/7/20.
 */
public class BasicFileOutput {
    static String fileIn = "C:\\think\\test.txt";
    static String fileOut = "C:\\think\\BasicFileOutput.out";
    public static void main(String[] args) throws IOException {
        BufferedReader in = new BufferedReader(new StringReader(BufferedInputFile.read(fileIn)));
        PrintWriter out = new PrintWriter(new FileWriter(fileOut));
        int lineCount = 1;
        String s;
        while ((s = in.readLine()) != null){
            // 写入
            out.println(lineCount++ + "：" + s);
        }
        out.close();
        // 显示存储的文件
        System.out.println(BufferedInputFile.read(fileOut));
    }
}
```
输出：
```
1：hello everybody
2：Today,the rain in Beijing is really big.
```

**文本文件输出的快捷方式**

Java SE5在`PrintWriter`中添加了一个辅助构造器，使得不必再每次希望创建文件文件并向其中写入时，都去执行所有的装饰工作：

```java
/**
 * 文本文件输出的快捷方式
 * Created by mrdios on 2016/7/20.
 */
public class FileOutputShortcut {
    static String fileIn = "C:\\think\\test.txt";
    static String fileOut = "C:\\think\\FileOutputShortcut.out";

    public static void main(String[] args) throws IOException {
        BufferedReader in = new BufferedReader(new StringReader(BufferedInputFile.read(fileIn)));
        // 快捷方式
        PrintWriter out = new PrintWriter(fileOut);
        int lineCount = 1;
        String s;
        while ((s = in.readLine()) != null){
            out.println(lineCount++ + ":" + s);
        }
        out.close();
        System.out.println(BufferedInputFile.read(fileOut));
    }
}
```

### 18.5.5 存储和恢复数据 ###

为了输出可供另一个“流”恢复的数据，我们需要用`DataOutputStream`写入数据，并用`DataInputStream`恢复数据，注意`DataOutputStream`和`DataInputStream`是面向字节的，因此要使用`InputStream`和`OutPutStream`。

```java
/**
 * 存储和恢复数据
 * Created by mrdios on 2016/7/20.
 */
public class StoringAndRecoveringData {
    public static void main(String[] args) throws IOException {
        // store
        DataOutputStream out = new DataOutputStream(new BufferedOutputStream(new FileOutputStream("C:\\think\\Data.txt")));
        out.writeDouble(3.1415926);
        out.writeUTF("That was pi");
        out.writeDouble(1.41413);
        out.writeUTF("Square root of 2");
        out.close();
        // recover
        DataInputStream in = new DataInputStream(new BufferedInputStream(new FileInputStream("C:\\think\\Data.txt")));
        System.out.println(in.readDouble());
        System.out.println(in.readUTF());
        System.out.println(in.readDouble());
        System.out.println(in.readUTF());
    }
}
```
输出：
```
3.1415926
That was pi
1.41413
Square root of 2
```
如果使用`DataOutputStream`写入数据，Java保证我们可以使用`DataInputStream`准确地读取数据——无论读和写数据的平台多么不同。

使用`DataOutputStream`时，写字符串并且让`DataInputStream`能够恢复的唯一可靠做法就是使用`UTF-8`编码。

### 18.5.6 读写随机访问文件 ###

使用`RandomAccessFile`，利用`seek()`可以在文件中到处移动，并修改文件中的某个值。使用`RandomAccessFile`时必须要知道文件排版才能正确地操作，它拥有读取基本类型和UTF-8字符串的各种具体方法：

```java
/**
 * 读取随机访问文件
 * Created by mrdios on 2016/7/20.
 */
public class UsingRandomAccessFile {
    static String file = "rtest.dat";
    static void display() throws IOException{
        RandomAccessFile rf = new RandomAccessFile(file,"r");
        for (int i = 0; i<7; i++){
            System.out.println("Value " + i + "：" + rf.readDouble());
        }
        System.out.println(rf.readUTF());
        rf.close();
    }

    public static void main(String[] args) throws IOException {
        RandomAccessFile rf = new RandomAccessFile(file,"rw");
        for (int i = 0; i<7;i++){
            rf.writeDouble(i*1.414);
        }
        rf.writeUTF("The end of the file");
        rf.close();
        display();
        rf = new RandomAccessFile(file,"rw");
        rf.seek(5*8);
        rf.writeDouble(47.0001);
        rf.close();
        display();
    }
}
```
输出：
```
Value 0：0.0
Value 1：1.414
Value 2：2.828
Value 3：4.242
Value 4：5.656
Value 5：7.069999999999999
Value 6：8.484
The end of the file
Value 0：0.0
Value 1：1.414
Value 2：2.828
Value 3：4.242
Value 4：5.656
Value 5：47.0001
Value 6：8.484
The end of the file
```

## 18.6 压缩 ##

压缩类|功能|
:--|:--
CheckedInputStream|GetCheckSum()为任何InputStream产生校验和(不仅是解压缩)
CheckedOutputStream|GetCheckedSum()为任何OutputStream产生校验和（不仅是压缩）
DeflaterOutputStream|压缩类的基类
ZipOutputStream|一个DeflaterOutputStream，用于将数据压缩成Zip文件格式
GZIPOutputStream|一个DeflaterOutputStream，用于将数据压缩成GZIP文件格式
InflaterInputStream|解压缩类的基类
ZipInputStream|一个DeflaterInputStream，用于解压缩Zip文件格式的数据
GZIPInputStream|一个DeflaterOutputStream，用于解压缩GZIP文件格式的数据

Zip和GZIP是最常用的压缩算法。

### 18.6.1 用GZIP进行简单压缩 ###

如果想对单个数据流（不是一系列互异数据）进行压缩，GZIP是比较合适的选择，下面是对单个文件进行压缩的例子：

```java
/**
 * 单个文件的GZIP压缩
 * Created by mrdios on 2016/7/20.
 */
public class GZIPcompress {
    static String file = "C:\\think\\test.txt";
    public static void main(String[] args) throws IOException {
        BufferedReader in = new BufferedReader(new FileReader(file));
        BufferedOutputStream out = new BufferedOutputStream(new GZIPOutputStream(new FileOutputStream("C:\\think\\test.gz")));
        System.out.println("Writing file...");
        int c;
        while ((c = in.read()) != -1){
            out.write(c);
        }
        in.close();
        out.close();
        System.out.println("Reading file...");
        BufferedReader in2 = new BufferedReader(new InputStreamReader(new GZIPInputStream(new FileInputStream("C:\\think\\test.gz"))));
        String s;
        while ((s = in2.readLine()) != null){
            System.out.println(s);
        }
    }
}
```
输出：
```
Writing file...
Reading file...
hello everybody
Today,the rain in Beijing is really big.
```

### 18.6.2 用Zip进行多文件保存 ###

使用`Checksum()`类来计算和校验文件的检验和，有两种`Checksum`类型：

1. Adler32（比较快）
2. CRC32（慢一些，但更准确）

```java
/**
 * 用zip进行多文件保存
 * Created by mrdios on 2016/7/20.
 */
public class Zipcompress {
    static String[] files = new String[]{"C:\\think\\BasicFileOutput.out","C:\\think\\test.txt"};
    public static void main(String[] args) throws IOException {
        FileOutputStream f = new FileOutputStream("test.zip");
        CheckedOutputStream csum = new CheckedOutputStream(f, new Adler32());
        ZipOutputStream zos = new ZipOutputStream(csum);
        BufferedOutputStream out = new BufferedOutputStream(zos);
        zos.setComment("A test of java Zipping");
        for (String file: files){
            System.out.println("Writing file " + file);
            BufferedReader in = new BufferedReader(new FileReader(file));
            zos.putNextEntry(new ZipEntry(file));
            int c;
            while ((c = in.read()) != -1){
                out.write(c);
            }
            in.close();
            out.flush();
        }
        out.close();
        // 关闭文件流之后才能校验
        System.out.println("Checksum: " + csum.getChecksum().getValue());
        // 解压文件
        System.out.println("Reading file...");
        FileInputStream fi = new FileInputStream("test.zip");
        CheckedInputStream csumi = new CheckedInputStream(fi,new Adler32());
        ZipInputStream in2 = new ZipInputStream(csumi);
        BufferedInputStream bis = new BufferedInputStream(in2);
        ZipEntry ze;
        while ((ze = in2.getNextEntry()) != null){
            System.out.println("Reading file " + ze);
            int x;
            while ((x = bis.read()) !=-1){
                System.out.print((char)x);
            }
        }
        System.out.println("\nShow entries...");
        if (files.length == 1){
            System.out.println("Checksum: " + csumi.getChecksum().getValue());
        }
        bis.close();
        ZipFile zf = new ZipFile("test.zip");
        Enumeration e = zf.entries();
        while (e.hasMoreElements()){
            ZipEntry ze2 = (ZipEntry) e.nextElement();
            System.out.println("File: " + ze2);
        }
    }
}
```
输出：
```
Writing file C:\think\BasicFileOutput.out
Writing file C:\think\test.txt
Checksum: 3774907338
Reading file...
Reading file C:\think\BasicFileOutput.out
1hello everybody
2Today,the rain in Beijing is really big.
Reading file C:\think\test.txt
hello everybody
Today,the rain in Beijing is really big.
Show entries...
File: C:\think\BasicFileOutput.out
File: C:\think\test.txt
```
### 18.6.3 Java档案文件（.jar） ###

Sun的JDK自带的jar程序可以根据我们的选择自动压缩文件，可以用命令的形式调用它，如下所示：

jar [options] destination [manifest] inputfile(s)

其中`options`只是一个字母集合（不用输入任何“-”或其他任何标识符）。以下选项字符在Unix和Linux系统中的tar文件中也具有相同的意义，如下所示：

option|desc
:--|:--
c|创建一个新的或空的压缩文档
t|列出目录表
x|解压所有文件
x file|解压该文件
f|"我打算指定一个文件名。"如果没用这个选项，jar假设所有的输入都来自于标准输入；或者在创建一个文件时，输出对象也是假设为标准输出
m|表示第一个参数将是用户自建的清单文件的名字
v|产生详细输出，描述jar所做的工作
O|只存储文件，不压缩文件（用来创建一个可放在类路径中的JAR文件）
M|不自动创建文件清单

## 18.7 XML ##

对象的序列化的一个限制是它只是java的解决方案：只有java程序才能反序列化这种对象。将数据转换为XML格式是另一种更具操作性的解决方案，可以使其被各种各样的平台和语言使用。

LKZS：

```java
package com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter18;

import nu.xom.Document;
import nu.xom.Element;
import nu.xom.Serializer;

import java.io.BufferedOutputStream;
import java.io.FileOutputStream;
import java.io.OutputStream;
import java.util.Arrays;
import java.util.List;

/**
 * xml的使用:对象转换为xml
 * Created by balderdasher on 2016/7/20.
 */
public class Person {
    private String first, last;

    public Person(String first, String last) {
        this.first = first;
        this.last = last;
    }

    // 从person对象产生xml元素
    public Element getXML() {
        Element person = new Element("person");
        Element firstName = new Element("first");
        firstName.appendChild(first);
        Element lastName = new Element("last");
        lastName.appendChild(last);
        person.appendChild(firstName);
        person.appendChild(lastName);
        return person;
    }

    public Person(Element person) {
        first = person.getFirstChildElement("first").getValue();
        last = person.getFirstChildElement("last").getValue();
    }

    @Override
    public String toString() {
        return first + " " + last;
    }

    public static void format(OutputStream os, Document doc) throws Exception {
        Serializer serializer = new Serializer(os, "ISO-8859-1");
        serializer.setIndent(4);
        serializer.setMaxLength(60);
        serializer.write(doc);
        serializer.flush();
    }

    public static void main(String[] args) throws Exception {
        List<Person> person = Arrays.asList(
                new Person("Dr.Bunsen", "Honeydew"),
                new Person("Gonzo", "The Great"),
                new Person("Phillip", "Fry")
        );
        System.out.println(person);
        Element root = new Element("people");
        for (Person p : person) {
            root.appendChild(p.getXML());
        }
        Document doc = new Document(root);
        format(System.out, doc);
        format(new BufferedOutputStream(new FileOutputStream("People.xml")), doc);
    }
}
```

输出：

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<people>
    <person>
        <first>Dr.Bunsen</first>
        <last>Honeydew</last>
    </person>
    <person>
        <first>Gonzo</first>
        <last>The Great</last>
    </person>
    <person>
        <first>Phillip</first>
        <last>Fry</last>
    </person>
</people>
```

从xml中反序列化对象：

```java
package com.mrdios.competencymatrix.java.readingnotes.ThinkingInJava.chapter18;

import nu.xom.Builder;
import nu.xom.Document;
import nu.xom.Elements;

import java.util.ArrayList;

/**
 * 从xml中反序列化对象
 * Created by balderdasher on 2016/7/20.
 */
public class People extends ArrayList<Person> {
    public People(String fileName) throws Exception {
        Document doc = new Builder().build(fileName);
        Elements elements = doc.getRootElement().getChildElements();
        for (int i = 0;i<elements.size();i++){
            add(new Person(elements.get(i)));
        }
    }

    public static void main(String[] args) throws Exception {
        People p = new People("People.xml");
        System.out.println(p);
    }
}
```
输出：
```
[Dr.Bunsen Honeydew, Gonzo The Great, Phillip Fry]
```
