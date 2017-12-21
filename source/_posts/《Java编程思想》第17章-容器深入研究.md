---
title: 《Java编程思想》第17章-容器深入研究
date: 2017-05-25 11:29:50
tags: [java,容器,读书笔记]
categories: java
permalink: thinking-in-java-containers
---
![list-set-queue](/uploads/thinking-in-java/java-containers-list-set-queue.png)
<!--more-->
![java-map](/uploads/thinking-in-java/java-containers-map.png)
## 深入研究容器的意义 ##
* 学习散列机制是如何工作的
* 在使用散列容器时怎样编写`hashCode()`和`equals()`方法
* 为什么某些容器有不同版本的实现以及如何选择它们

<!--more-->
## 17.1 完整的容器分类法 ##
完整的容器图
![full](http://i.imgur.com/bY03QpW.png)

## 17.2 填充容器 ##
Collections容器工具类中的填充方法
* `Collections.nCopies()`
* `Collections.fill()`

以上两种填充方式都只是复制一个对象引用来填充整个容器，并且只对`List`对象有用。

## 17.3 散列与散列码 ##
正确的`equals()`方法必须满足一下5个条件：
1. 自反性：对任意`x`,`x.equals(x)`一定返回true
2. 对称性：对任意`x`和`y`，如果`y.equals(x)`返回true，则`x.equals(y)`也返回true
3. 传递性：对任意`x`、`y`、`z`，如果有`x.equals(y)`返回true，`y.equals(z)`返回true，则`x.equals(z)`一定返回true
4. 一致性：对任意`x`和`y`，如果对象中用于等价比较的信息没有改变，那么无论调用`x.equals(y)`多少次，返回的结果应该保持一致，要么一直是true，要么一直是false
5. 对任何不是`null`的`x`，`x.equals(null)`一定返回false

**默认的`Object.equals()`只是比较对象的地址**，所以一个`Stu`并不等于另一个`Stu`。

### 17.3.1 理解hashCode() ###
首先，使用散列的目的在于：想要使用一个对象来查找另一个对象，下面是一个使用一对`ArrayList`实现的`Map`，包含了Map接口的完整实现，因此提供了`entrySet()`方法：

```java

public class SlowMap<K,V> extends AbstractMap<K,V> {
    private List<K> keys = new ArrayList<>();
    private List<V> values = new ArrayList<>();

    @Override
    public V put(K key, V value) {
        V oldValue = get(key);
        if (!keys.contains(key)){
            keys.add(key);
            values.add(value);
        }else{
            values.set(keys.indexOf(key),value);
        }
        return oldValue;
    }

    @Override
    public V get(Object key) {
        if (!keys.contains(key)){
            return null;
        }
        return values.get(keys.indexOf(key));
    }

    @Override
    public Set<Map.Entry<K, V>> entrySet() {
        Set<Map.Entry<K,V>> set = new HashSet<>();
        Iterator<K> ki = keys.iterator();
        Iterator<V> vi = values.iterator();
        while (ki.hasNext()){
            set.add(new MapEntry<K, V>(ki.next(),vi.next()));
        }
        return set;
    }

    class MapEntry<K,V> implements Map.Entry<K,V>{
        private K key;
        private V value;

        public MapEntry(K key, V value) {
            this.key = key;
            this.value = value;
        }

        @Override
        public K getKey() {
            return key;
        }

        @Override
        public V getValue() {
            return value;
        }

        @Override
        public V setValue(V v) {
            V result = v;
            value = v;
            return result;
        }

        @Override
        public int hashCode() {
            return (key == null ? 0 : key.hashCode())^
                    (value == null ? 0 : value.hashCode());
        }

        @Override
        public boolean equals(Object obj) {
            if (!(obj instanceof MapEntry)){
                return false;
            }
            MapEntry me = (MapEntry) obj;
            return (key == null?me.getKey() == null:key.equals(me.getKey()))
                    && (value == null?me.getValue() == null:value.equals(me.getValue()));
        }

        @Override
        public String toString() {
            return key + "=" + value;
        }
    }

    public static void main(String[] args) {
        SlowMap<String,String> map = new SlowMap<>();
        map.put("a","a");
        map.put("b","b");
        System.out.println(map);
    }
}
```

SlowMap说明创建一个新的Map并不难，但是正如其名字所示，它并不快，问题在于对键的查询，键没有按照任何特定顺序保存，所以只能使用简单的线性查询，而线性查询是最慢的查询方式。

散列的价值在于速度：散列使得查询得以快速进行，由于瓶颈位于键的查询速度，因此解决方案之一就是保持键的顺序状态，然后使用`Collection.binarySearch()`进行查询。

散列则更进一步，它将键保存在某处，以便能够快速找到。存储一组元素最快的数据结构是数组，所以使用数组来表示键的信息（键的信息，不是键本身），但是因为数组不能够扩容，因此就有一个问题：我们希望在Map中保存数量不确定的值，但是如果键的数量被数组容量限制了又该怎么办呢？

答案就是：数组并不保存键本身。而是通过键对象生成一个数字，将其作为数组的下标。这个数字就是**散列码**，由散列函数`hashCode()`生成。

所以可以说：**散列码就是在Map中的键对象在其保存键信息的数组中的下标数字**。

为解决数组容量被固定的问题，不同的键可以产生相同的下标，也就是说，散列码可能会有*冲突*。因此，数组多大已经不重要了，任何键总能在数组中找到它的位置。

于是，在map中查询一个值的过程首先就是计算散列码，然后使用散列码查询数组，如果能够保证没有冲突（只有值的数量是固定时才有可能），那就可以有一个*完美的散列函数*，但是这种情况只是特例（EnumMap和EnumSet中得到了实现）。通常，冲突由外部连接处理：数组并不直接保存值，而是保存值的`list`。然后对list中的值使用`equals()`方法进行线性的查询。这部分的查询会比较慢，但是如果散列函数好的话，数组的每个位置就只有较少的值，因此不是查询整个list，而是快速地跳到数组的某个位置，只对很少的元素进行比较。这就是`HashMap`会如此快的原因。

根据以上关于散列的原理，就可以实现如下一个简单的散列Map了：

```java
public class SimpleHashMap<K,V> extends AbstractMap<K,V> {
    // 指定一个质数作为哈希表的长度，用于获得均匀分布
    static final int SIZE = 997;

    // 用于存放值list的数组，即散列表
    LinkedList<MapEntry<K,V>>[] buckets = new LinkedList[SIZE];

    @Override
    public V put(K key, V value) {
        V oldValue = null;
        // 计算该值要存放在数组中的哪个位置
        int index = Math.abs(key.hashCode()) % SIZE;
        // 该位置可用则在此位置初始化存储空间
        if (buckets[index] == null){
            buckets[index] = new LinkedList<MapEntry<K,V>>();
        }
        // 待放入的bucket
        LinkedList<MapEntry<K,V>> bucket = buckets[index];
        // 构造键值对
        MapEntry<K,V> pair = new MapEntry<>(key,value);

        boolean found = false;
        // 在数组中对应下标的list中查找key，若存在相同的key则替换旧值
        ListIterator<MapEntry<K,V>> it = bucket.listIterator();
        while (it.hasNext()){
            MapEntry<K,V> iPair = it.next();
            if (iPair.getKey().equals(key)){
                oldValue = iPair.getValue();
                it.set(pair); // 替换旧值
                found = true;
                break;
            }
        }
        // 未查到key则在数组相应位置中的list中放入此键值对
        if (!found){
            buckets[index].add(pair);
        }
        return oldValue;
    }

    @Override
    public V get(Object key) {
        // 先根据key确定在数组中定位查询范围
        int index = Math.abs(key.hashCode()) % SIZE;
        if (buckets[index] == null){
            return null;
        }
        // 在确定的查询范围中根据key查询查到则返回值
        for (MapEntry<K,V> iPair:buckets[index]){
            if (iPair.getKey().equals(key)){
                return iPair.getValue();
            }
        }
        return null;
    }

    @Override
    public Set<Entry<K, V>> entrySet() {
        Set<Entry<K,V>> set = new HashSet<>();
        for (LinkedList<MapEntry<K,V>> bucket : buckets){
            if (bucket == null){
                continue;
            }
            for (MapEntry<K,V> mpair : bucket){
                set.add(mpair);
            }
        }
        return set;
    }

    public static void main(String[] args) {
        SimpleHashMap<String,String> m = new SimpleHashMap<>();
        m.put("a","a");
        m.put("b","b");
        System.out.println(m);
        System.out.println(m.get("a"));
        System.out.println(m.entrySet());
    }
}
```
输出：
```
{a=a, b=b}
a
[a=a, b=b]
```

### 17.3.2 覆盖hashCode() ###
理解了散列的原理之后，编写自己的hashCode()就更有意义了，

设计hashCode()时最重要的因素就是：无论何时，对同一个对象调用hashCode()都应该生成同样的值。如果put时和get时产生的hashCode()值不一样，那就无法重新取得对象了，所以如果hashCode()方法依赖于对象中易变的数据，那么当此数据发生变化时，使用hashCode()将会产生不一样的散列码，相当于产生了一个不同的键。

另外，也不应该使hashCode()依赖于具有唯一性的对象信息，尤其是使用`this`的值，这只能产生很糟糕的hashCode(),因为这样做无法生成一个新的键，使之与put时原始的键值相同（因为每个对象的地址虽然是唯一的但是它们也不同所以不会产生相同的键）。

以`String`类为例，`String`有个特点：如果程序中有多个String对象，都包含相同的字符串序列，那么这些String对象都映射到同一块内存区域。所以`new String（“hello”）`生成的两个实例虽然是相互独立的，但是对它们使用hashCode()应该生成同样的结果，如下所示：

```java
public class StringHashCode {
    public static void main(String[] args) {
        String[] hellos = "Hello Hello".split(" ");
        System.out.println(hellos[0].hashCode());
        System.out.println(hellos[1].hashCode());
    }
}
```

输出：
```
69609650
69609650
```
所以对于String而言，hashCode()明显是基于内容的，因此，想要hashCode()实用，它必须速度快而且有意义，也就是说它必须基于对象的内容生成散列码，要记住散列码不必是独一无二的（因为散列码只决定该对象放于哪个桶中，而该桶中可以放多个对象，所以散列码相同无所谓），应该更关注速度而不是唯一性，但前提是通过hashCode()和equals(),必须能够完全确定对象的身份。

另一个因素就是：好的`hashCode()`应该产生分布均匀的散列码，不难想象，如果散列码都集中在一块，那么`HashMap`在某些区域的负载会很重，这样就不如分不均与的散列码快（负载重的话put和get操作时对key的线性查询范围增大所以速度就慢了）。

Joshua Bloch在他的`Effective Java`丛书中给出了如何写出一个好的`hashCode()`的指导：

1) 给`int`变量`result`赋予某个非零值常量，如`17`。
2) 为对象内部每个有意义的域`f`（即每个可以做`equals()`操作的域）计算出一个`int散列码`**c**。

域类型|计算
:--|:--
boolean|c=(f?0:1)
byte、char、short、int|c=(int)f
long|c=(int)(f^(f>>>32))
float|c=(int)Float.floatToIntBits(f)
double|long l = Double.doubleToLongBits(f);c=(int)(l^(l>>>32))
Object，其equals()调用这个域的equals()|c=f.hashCode()
数组|对每个元素应用上述规则

3) 合并计算得到的散列码：`result = 37 * result + c`;
4) 返回`result`
5) 检查hashCode()最后生成的结果，确保相同的对象有相同的散列码。

以下是一个以上述规则为指导的例子：

```java
/**
 * 遵循指导的一个hashCode()实现
 * Created by balderdasher on 2016/7/13.
 */
public class CountedString {
    private static List<String> created = new ArrayList<>();
    private String s;
    private int id = 0;

    public CountedString(String s) {
        this.s = s;
        created.add(s);
        // id 为此String在CountedString中使用的实例总数
        for (String s1 : created) {
            if (s1.equals(s)) {
                id++;
            }
        }
    }

    @Override
    public String toString() {
        return "String：" + s + " id：" + id + " hashCode()：" + hashCode();
    }

    @Override
    public int hashCode() {
        int result = 17;
        result = 37 * result + s.hashCode();
        result = 37 * result + id;
        return result;
    }

    @Override
    public boolean equals(Object obj) {
        return obj instanceof CountedString &&
                s.equals(((CountedString) obj).s) &&
                id == ((CountedString) obj).id;
    }

    public static void main(String[] args) {
        Map<CountedString,Integer> map = new HashMap<>();
        CountedString[] cs = new CountedString[5];
        for (int i = 0; i < cs.length; i++) {
            cs[i] = new CountedString("hi");
            map.put(cs[i],i);
        }
        System.out.println(map);
        for (CountedString cstring:cs){
            System.out.println("Looking up " + cstring);
            System.out.println(map.get(cstring));
        }
    }
}
```

输出：
```
{String：hi id：4 hashCode()：146450=3, String：hi id：5 hashCode()：146451=4, String：hi id：2 hashCode()：146448=1, String：hi id：3 hashCode()：146449=2, String：hi id：1 hashCode()：146447=0}
Looking up String：hi id：1 hashCode()：146447
0
Looking up String：hi id：2 hashCode()：146448
1
Looking up String：hi id：3 hashCode()：146449
2
Looking up String：hi id：4 hashCode()：146450
3
Looking up String：hi id：5 hashCode()：146451
4
```

## 17.4 选择接口的不同实现 ##
尽管实际上只有四中容器：`Map`、`List`、`Set`和`Queue`，但是每种接口都有各种实现版本，我们在实际使用时应该如何选择呢？

容器之间的区别可以归结为在它们背后有什么样的数据结构支持，即所用的接口是由什么数据结构实现的，如`ArrayList`底层由数组支持，而`LinkedList`是由双向链表实现，每个对象包含数据的同时还包含指向链表中前一个和后一个元素的引用，因此如果要经常插入或删除元素，LinkedList就比较合适，否则应该用速度更快的ArrayList。

### 17.4.1 对List的选择 ###
通过性能测试可以得出结论：

- **查询：**对于背后有数组支持的List和`ArrayList`，无论列表的大小如何，访问数据都是快速和一致的；而对于`LinkedList`，访问元素的时间对于较大的列表将明显增加，所以如果需要大量的随机访问，应该用`ArrayList`。
- **插入和删除：**在列表中执行插入和删除操作时，对于`ArrayList`,当列表变大时，其开销将变得很大，但是对于`LinkedList`来说开销相对低廉，并且不随列表尺寸而发生变化，这是因为ArrayList在插入或删除元素时，必须创建空间并将它的所有引用向前或后移动，所以ArrayList的尺寸越大，开销就越多。而LinkedList只需要链接新的元素不必修改列表中剩余的元素，所以无论LinkedList的尺寸如何变化，插入的开销都大致相同。

### 17.4.2 对Set的选择 ###
- `HashSet`的性能基本上总是比`TreeSet`好，特别是添加和查询元素时
- `TreeSet`存在的唯一原因是因为它可以维持元素的排序状态，所以只有当需要一个排好序的Set时，才应该使用`TreeSet`
- 因为`TreeSet`的内部结构支持排序，并且因为迭代是更有可能的操作，所以用`TreeSet`迭代一般要比用`HashSet`快。
- 对于插入操作，`LinkedHashSet`比`HashSet`的代价更高，这是由维护链表所带来的额外开销

### 17.4.3 对Map的选择 ###
- 除了`IdentityHashMap`，所有的Map实现的插入操作都会随着Map尺寸的变大而明显变慢
- HashMap 是Hashtable 的轻量级实现（非线程安全的实现），他们都完成了Map 接口，主要区别在于HashMap 允许空（null）键值（key）,由于非线程安全，效率上可能高于Hashtable。HashMap 允许将null 作为一个entry 的key 或者value，而Hashtable 不允许。HashMap 把Hashtable 的contains 方法去掉了，改containsvalue 和containsKey。因为contains方法容易让人引起误解。Hashtable 继承自Dictionary 类，而HashMap Java1.2 引进的Map interface 的一个实现。最大的不同是，Hashtable 的方法是Synchronize 的，而HashMap 不是，在多个线程访问Hashtable 时，不需要自己为它的方法实现同步，而HashMap 就必须为之提供外同步。Hashtable 和HashMap 采用的hash/rehash 算法都大概一样，所以性能不会有很大的差异。
- `TreeMap`通常比`HashMap`要慢
- `LinkedHashMap`在插入时比`HashMap`慢一点，因为它维护散列数据结构的同时还要维护链表（以保持插入顺序），但正因为有链表，所以迭代速度更快
- `IdentityHashMap`具有完全不同的性能，因为它使用`==`而不是`equals()`来比较元素

**HashMap的性能因子**

hashMap中的几个术语解释：
* 容量：表中的桶位数
* 初始容量：表在创建时所拥有的桶位数，HashMap和HashSet允许指定初始容量的构造器。
* 尺寸：表中当前存储的项数
* 负载因子：尺寸/容量，空表的负载因子是0，半满表的则是0.5,以此类推，负载轻的表产生冲突的可能性小，因此对于插入和查找都是最理想的（但是迭代过程会慢），`HashMap`和`HashSet`都具有允许指定负载因子的构造器，表示当负载情况达到该负载因子的水平时，容器将自动增加其容量（桶位数），实现方式是使容量大致加倍，并将对象重新散列分布。

## 快速报错 ##
Java容器有一种保护机制，能够防止多个进程同事修改同一个容器的内容，如果在迭代遍历某个容器的过程中，另一个过程介入并且插入、删除或修改此容器内的某个对象，那么就会出现问题：*也许迭代过程已经处理过容器中的该元素了，也许还没处理，也许在调用`size()`之后容器的尺寸收缩了等等灾难情景*。Java容器类类库采用**快速报错(fail-fast)**机制。它会探查容器上的任何除了你的进程所进行的操作以外的所有变化，一旦发现其他进程修改了容器，就会立刻抛出`ConcurrentModificationException`异常。这就是“快速报错”的意思。

快速报错机制的工作原理简单来讲就是：只需创建一个迭代器，然后向迭代器所指向的`Collection`添加点什么，这时候快速报错机制就起作用了：

```java
/**
 * Java容器的快速失败机制
 * Created by mrdios on 2016/7/19.
 */
public class FailFast {
    public static void main(String[] args) {
        Collection<String> c = new ArrayList<>();
        Iterator<String> it = c.iterator();
        c.add("bitch");
        try {
            String s = it.next();
        } catch (ConcurrentModificationException e) {
            System.out.println(e);
        }
    }
}
```
output:
```
java.util.ConcurrentModificationException
```

以上代码演示了快速报错机制的效果，这也提醒我们：**<font color="red">在使用Java容器时，应该在添加完所有元素之后，再获取迭代器</font>**。
