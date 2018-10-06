# Java 类集 笔记

## 一.概述：

最初是用数组来实现的

   问题：数组是一个固定长度的，即使后面有了链表，可扩容，但链表的性能差，编写困难

​               保存的数据，是用Object，存在数据安全，和数据向下转型。

Jdk 1.2 后，出现了Vector（向量），但这个也不能很好的描述出数据结构。

jdk1.5 后，出现了集合框架，并且引入了泛型。

jdk1.8 后，针对大数据操作效率低下，又推出了Stream数据流，







### 核心接口

java.util 包中

 三大类：

- Collection、List（允许对象重复）、Set（不允许对象重复）后面2个接口都继承Collection接口

- Map

- Iterator、Enumeration

### 总结

**经常使用的就是ArrayList（单一值） 和 HashMap（Key，Value）**



## 二.Collection(java.util) 单个对象

整个类集中单值保存的最大父接口，即每次最多向集合中保存一个对象。

``` 
public interface Collection<E> extends Iterable<E> {
   boolean add(E e);
   boolean addAll(Collection<? extends E> c);
   void clear();
   boolean contains(Object o);
   boolean containsAll(Collection<?> c);
   boolean equals(Object o);
   boolean isEmpty();
   Iterator<E> iterator();
   //1.8
   default Stream<E> parallelStream()
   boolean remove(Object o);
   boolean removeAll(Collection<?> c);
   //1.8
   default boolean removeIf(Predicate<? super E> filter) 
   boolean retainAll(Collection<?> c); //保留
   default Spliterator<E> spliterator()；
   //1.8
   default Stream<E> stream()；
   Object[] toArray();
   <T> T[] toArray(T[] a);  
}
```

但我们现在一般都**不用Collection接口**了，都是使用的是它的**子接口** List和Set接口了。原因还是性能上的问题。子接口做了性能增强的处理。

### 1.List接口（java.util）

**List 是 Collection接口的最常用的子接口**

**List接口的常用2个子类（ArrayList、Vector）**

定义如下：

```java
public interface List<E> extends Collection<E>
//List 下 常用的2个子类
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
public class Vector<E>  extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable        
```



```java
//List接口重要扩展增强的方法
public interface List<E> extends Collection<E>{
    //取得索引位置的对象，从0 开始
    E get(int index);
    //修改制定索引位置的对象内容
    E set(int index, E element);
    //实例化ListIterator对象
    ListIterator<E> listIterator();
}

```

#### 1）ArrayList 子类（90%使用）

```java
ArrayList<String> list = new ArrayList<>();
System.out.println("list的size="+list.size()+" list 是否为空 = "+list.isEmpty());
list.add("hello");
list.add("hello");
list.add("world");
System.out.println("list的size="+list.size()+" list 是否为空 = "+list.isEmpty());
String s = list.get(2);
System.out.println("索引位置为2的值为:"+s);
list.set(2,"world haha!");
System.out.println("索引位置为2的值为:"+list.get(2));
```

#### 2）Vector子类（基本不使用了）

Vector类，在JDK1.0的时候就有了，当时还是基于数组，链表的方式来实现的，后来有了List集合框架后，为了向后兼容，让Vector去实现了List接口。

##### Stack类（Vector的子类）

**先进后出的一种数据结构**

虽然Stack extends Vector，但Stack**不使用Vector** 父类的方法，而是使用**自己**的方法

```java
//Stack
public class Stack<E> extends Vector<E> {

   public E push(E item) //压入栈
   //同步方法
   public synchronized E pop() //出栈
}
```



#### 3）Vector与ArrayList的区别

| 序号   | 区别点  | ArrayList（90%使用率）             | Vector（10%使用率）                          |
| ---- | ---- | ----------------------------- | --------------------------------------- |
| 1    | 推出时间 | JDK1.2                        | JDK1.0，属于旧的类                            |
| 2    | 性能   | 异步                            | 同步                                      |
| 3    | 安全   | 非线程安全                         | 线程安全                                    |
| 4    | 功能   | Iterator、ListIterator、foreach | Iterator、ListIterator、foreach、Enumerate |

#### 4）总结

- List中数据的保存顺序就是添加的顺序

- List接口中，可以保存重复的数据

- List接口 相比Collection增强了get(int idx)和set(int idx)方法

- List接口的实现类，一般都选择ArrayList



### 2.Set(java.util)子接口

使用率在20%，主要是集合**不允许重复**

**Set集合并不像List接口，对Collection接口做了功能增强，不支持get()方法**

Set接口下，有2个常用的子类**HashSet、TreeSet**

```java
public interface Set<E> extends Collection<E> 

public abstract class AbstractCollection<E> implements Collection<E>

public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E> 
//
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
//    
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
	
```



#### 1）TreeSet

TreeSet类是通过**Comparable**接口中**CompareTo()**方法判断是否有重复数据（返回0表示相等）。也就是说，需要加入到Set容器中的对象必须是实现了Comparable接口的类。

要实现CompareTo方法 ，必须是这个对象的所有属性要一起比较。好麻烦啊。

#### 2）HashSet

HashSet没有像TreeSet一样，实现了Comparable接口的CompareTo()方法.

**Comparable**接口中**CompareTo()**方法，只用于TreeSet类的判断重复。

#### 3）关于重复元素的判断

Comparable接口的CompareTo()方法，只是在排序的时候，要判断是否重复。

**除此之外，2个对象间判断重复，需要用到Object类的2个方法来判断**

1）public int **hashCode**()  判断这个值是否相等

2）public boolean **equals**（Object obj） 判断对象的内容是否一样

我们的开发工具idea和eclipse都有**自动生成hashCode和equals**的方法，直接生成后，就可以拿来使用判断

#### 4）总结

- Set 接口，不是开发中的首选

- Comparable接口，只用于TreeSet的比较，而TreeSet的使用很麻烦，现实开发应用中，几乎用不到这个类

  实在要用，使用HashSet

- Set 不管如何操作，数据要保证不能重复，就需要用到Object类的2个方法（hashCode和equals）

### 3.接口关系图

![](images\QQ截图20181001220837.png)

### 4.集合的输出

JDK1.8之前是 **Iterator(95%)**、**Enumeration(4.9%)**、ListIterator(0.05%)、for(0.05%) 四种输出及占用比例

#### 1）Iterator接口（迭代输出）核心

```java
//迭代器的核心操作
public interface Iterator<E> {
     boolean hasNext(); //获取是否还有下一个对象
     E next();          //取得下一个对象
}

实现类是哪个呢？
Collection接口中有一个方法 Iterator<E> iterator(); 可以获取到迭代器。

```

Collection所有的子类，都实现了iterator()方法。

```java
ArrayList<String> list = new ArrayList<>();
list.add("hello");
list.add("hello");
list.add("world");
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()){
    String sl = iterator.next();
    System.out.println(sl);
}
```

#### 2）ListIterator接口（双向迭代）

```java
public interface ListIterator<E> extends Iterator<E>
```

它是为List接口服务的。支持由后向前的顺序迭代输出，虽然用得不多。

```java
public static void main(String[] args) {
    List<String> list = new ArrayList<String>();
    list.add("a");
    list.add("b");
    list.add("c");
    ListIterator<String> listIterator = list.listIterator();
    //由前向后输出
    while (listIterator.hasNext()){
        String sl = listIterator.next();
        System.out.println(sl);
    }

    //由后向前输出
    while (listIterator.hasPrevious()) {
        String previous = listIterator.previous();
        System.out.println(previous);
    }

}
```

**注意：要想实现由后向前输出，必须先实现由后向前输出**

估计是数组指针的问题。添加完成后，指针在数据的最前面。

#### 3）for输出

```java
//for 语句
for (String s : list) {
    System.out.println(s);
}
//采用Lambda输出
list.forEach(x->{
    System.out.println(x);
});
```

好用，但不推荐使用。

#### 4）Enumeration（Vector专用）

这个是最老的输出接口，配合Vector来使用的。

```java
public interface Enumeration<E> {
    boolean hasMoreElements();
    E nextElement();
}
```

只能通过Vector子类，才能获取到Enumeration实例化对象。

```java
public Enumeration<E> elements()
```

#### 5）总结

推荐使用Iterator（**所有的Collection的子类都可以**） 和 Enumeration（**Vector类才支持**） 这2个接口来实现输出。

故，掌握了Iterator，就知道所有的Collection的子类集合元素的输出

## 三、Map接口

```java
public interface Map<K,V> {
    V put(K key, V value);
    V get(Object key);
    Set<Map.Entry<K,V>> entrySet() //将map集合转换为Set集合
    Set<K> keySet(); //取出全部的Key
}

public static interface Map.Entry<K,V>{
    static <K extends Comparable<? super K>,V> Comparator<Map.Entry<K,V>> comparingByKey()  
        
}
```

2个重要子类：HashMap、HashTable

- Hash名前缀的类都是无序的
- 数据Key如果重复，值将会被覆盖
- ​

#### 1）HashMap和HashTable的区别

| 序号   | 区别点    | HashMap（90%使用率） | HashTable（10%使用率） |
| ---- | ------ | --------------- | ----------------- |
| 1    | 推出时间   | JDK1.2          | JDK1.0，属于旧的类      |
| 2    | 性能     | 异步              | 同步                |
| 3    | 安全     | 非线程安全           | 线程安全              |
| 4    | 设置null | 可以为空            | Key 和 Value都不允许为空 |

#### 2）集合的输出

Map接口，没有定义Iterator接口的实例化方法，那我们怎么去输出集合元素呢。

Map接口里有一个内部接口**Map.Entry<K,V>**接口

```java
//内部的 static接口，就等同于外部的接口
public static interface Map.Entry<K,V>{
    K getKey();
    V getValue()； 
}
```

Map集合中，保存的对象是一个Map.Entry对象（这个对象里有2个属性，key 和 value）

![](H:\99.JavaDoc\4. Java8 笔记\images\QQ截图20181002000604.png)

利用Map接口中的    entrySet() //将map集合转换为Set<**Map.Entry<K,V>**> 集合（Map.Entry<K,V> 就当成一个普通泛型类即可），我们得到了一个Set<Entry>对象后，我们就可以利用Set集合的Iterator来迭代输出了。

范例：

```java
Map<String,Integer> map = new HashMap<String,Integer>();
map.put("小李",1);
map.put("小王",6);
map.put("小张",3);
Set<Map.Entry<String, Integer>> entries = map.entrySet();//得到对象的Set集合
Iterator<Map.Entry<String, Integer>> iterator = entries.iterator();//通过set集合得到Iterator
while (iterator.hasNext()){ //Iterator 迭代
    Map.Entry<String, Integer> entry = iterator.next(); //获取Iterator对象
    System.out.println(entry.getKey()+"---"+entry.getValue()); //输出entry对象的key和value
}
```

#### 3）注意事项

如果，我们的Key是一个自定义对象的话，我们要对自定义对象的**hashCode**和**equals**方法，**覆写**才行，因为通过key去查找，包括判断重复，都需要这2个方法的实现。



以后使用中，最好是用String来作为Key。因为String的hashCode和equals都是实现好了的。



#### 4）Properties子类（extends HashTable）

这个类，也基本不用了。

Properties 类，是用于保存Properties扩展名的资源配置文件的。key 和 value。

```java
//继承的是HashTable，并且Key值是Object
public class Properties extends Hashtable<Object,Object> {
    public synchronized Object setProperty(String key, String value)
    public String getProperty(String key) //当key值不存在，返回null
    public String getProperty(String key, String defaultValue)//当key不存在，返回默认值
    public void store(OutputStream out, String comments)throws IOException //输出到文件或内存，网络等
    public synchronized void load(InputStream inStream) throws IOException //读取输入流
    
}
```

```java
  public static void main(String[] args) throws Exception {

        Properties properties = new Properties();
        properties.setProperty("user","杨新强");
        properties.setProperty("psw","123456");
        properties.setProperty("driver","com.java.Mysql.Driver");
        properties.setProperty("url","java:mysql://localhost:3306/");
        System.out.println(properties.getProperty("url"));
        File file = new File("d:" + File.separator + "mysql.properties");
        //comments:注释，注意，不能有中文
        properties.store(new FileOutputStream(file),"mysql driver config");
        //读取输入流到配置类中
        properties.load(new FileInputStream(file));
    }
```

保存到资源文件的内容

```properties
#mysql driver config
#Tue Oct 02 11:04:07 CST 2018
user=\u6768\u65B0\u5F3A
url=java\:mysql\://localhost\:3306/
psw=123456
driver=com.java.Mysql.Driver
```

总结：资源文件的后缀名，必须为Properties,因为这是和语言的国际化配合的，而且Properties文件的读取，也可以采用ResourceBundle类来处理。

资源文件都是处理字符串。

#### 5）ResourceBundle类

以后的开发中，一般这个类来操作，因为它可以实现国际化

```java
public static void main(String[] args) {
    //区域：语言和地区（中国是zh_US)
    Locale locale = new Locale("en","US");
    //当前系统默认的地区
    Locale aDefault = Locale.getDefault();
    System.out.println(aDefault);
    //getBundle，是读取的文件名，不包含后缀，后缀默认就是Properties
    //资源文件，必须放到src的根目录下，否则查找不到
    //如果这里没有设置取那个地区的话，根据当前区域，在src下查找是否包含有本区域的配置文件，如果有就加载
    //故，ResourceBundle.getBundle(baseName) 后面可以不加上地区
    ResourceBundle messageProperites = ResourceBundle.getBundle("message",locale);
    String info = messageProperites.getString("info");
    String msg = messageProperites.getString("msg");
    //可以对占位符格式化，动态改变值
    msg = MessageFormat.format(msg,"小丽");
    System.out.println(info);
    System.out.println(msg);
}
```

## 四、Stream 数据流

### 1）概述

这个Stream和InputStream，OutputStream 不一样，它全路径在java.util.stream.Stream包中。

它需要配合Lambda一起来处理数据（Filter）

Collection接口的父接口Iterable<T>里有个方法

```java
public interface Iterable<T>{
    
    //这个方法只接收参数，没有返回
    default void forEach(Consumer<? super T> action)
}
```

```java
public static void main(String[] args) {
        ArrayList<String> arrayList = new ArrayList<>();
        arrayList.add("Hello");
        arrayList.add("World");
        arrayList.add("Good");
        arrayList.forEach(System.out::println); //::表示xx类.xxx方法的简写
        /* 输出
        Hello
        World
        Good
        */
    }
```

//上面的forEach，只能做输出操作，并不能过滤数据。

过滤数据，需要用到Stream。

```java
public interface Collection<E> extends Iterable<E> {
    //通过stream，可以得到Stream对象，来处理数据
    default Stream<E> stream()
}
```


### 2）Stream 接口定义

**Stream 是 Collection 处理数据的类。**

```java
public interface Stream<T> extends BaseStream<T, Stream<T>>{
    //排重
    Stream<T> distinct();
    //收集器，返回Collect，注意，不是Collection
    <R, A> R collect(Collector<? super T, A, R> collector);

    //-------------------------------
    //对集合中 每个元素判断是否全部匹配
    boolean allMatch(Predicate<? super T> predicate);
    //对集合中 每个元素判断是否部分匹配    
    boolean anyMatch(Predicate<? super T> predicate);
    //--------------------------------
    
    //map：数据处理，可以返回一个新的东西
    <R> Stream<R> map(Function<? super T, ? extends R> mapper);
 
    //-------------------------------------------------------------

    //reduce：数据分析(group by,sum,avg等数据统计用的)
    Optional<T> reduce(BinaryOperator<T> accumulator);
    T reduce(T identity, BinaryOperator<T> accumulator);
    <U> U reduce(U identity,BiFunction<U, ? super T, U> accumulator,
                 BinaryOperator<U> combiner);
    
    
    IntStream mapToInt(ToIntFunction<? super T> mapper);
    DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper);
    LongStream mapToLong(ToLongFunction<? super T> mapper);
    //------------------------------------------------------------
    
    //过滤器
    Stream<T> filter(Predicate<? super T> predicate);
    //迭代处理
    void forEach(Consumer<? super T> action);
    //设置分页，跳过的行数
    Stream<T> skip(long n);         
    Stream<T> limit(long maxSize); //页码最大行
    
    Optional<T> findFirst();

    
    public static<T> Stream<T> of(T... values)
    Stream<T> peek(Consumer<? super T> action);
    T reduce(T identity, BinaryOperator<T> accumulator);
    
    <A> A[] toArray(IntFunction<A[]> generator);
    Stream<T> sorted(Comparator<? super T> comparator);
    Stream<T> sorted();
    
}
```



### 3）收集器

我们需要用到Collectors类，这个类有很多静态方法

```java
public final class Collectors{
    public static <T> Collector<T,?,List<T>> toList()
    public static <T,C extends Collection<T>> Collector<T,?,C> toCollection(Supplier<C> collectionFactory)
    public static <T> Collector<T,?,Set<T>> toSet()
}
```

举例：

```java
public static void main(String[] args) {
        ArrayList<String> arrayList = new ArrayList<>();
        arrayList.add("Hello");
        arrayList.add("World");
        arrayList.add("Good");
        arrayList.add("Good");
        arrayList.add("World");
        //举例排重
        List<String> distinctList = arrayList.stream().distinct().collect(Collectors.toList());
        //distinctList.forEach(System.out::println);
       distinctList.forEach(x->System.out.println(x));

    }
```

### 4）过滤数据

```java
public static void main(String[] args) {
    ArrayList<String> arrayList = new ArrayList<>();
    arrayList.add("Hello");
    arrayList.add("World");
    arrayList.add("Good");
    arrayList.add("hi");
    arrayList.add("World");
    //举例排重
    List<String> distinctList = arrayList.stream().distinct().collect(Collectors.toList());
    //过滤包含字符为h的，要区分大小写
    List<String> d = arrayList.stream().filter(x -> x.contains("h")).collect(Collectors.toList());
    System.out.println(d);
}
```

### 5）处理数据

利用Map对元素本身处理

```java
public static void main(String[] args) {
    ArrayList<String> arrayList = new ArrayList<>();
    arrayList.add("Hello");
    arrayList.add("World");
    arrayList.add("Good");
    arrayList.add("hi");
    arrayList.add("World");
    //利用map 来处理数据
    List<String> list1 = arrayList.stream().map(x->{return x.toUpperCase();}).filter(x -> x.contains("H")).collect(Collectors.toList());
    //当Lambda中只有一句代码时，可以省略掉{}
    List<String> list1 = arrayList.stream().map(x-> x.toUpperCase()).filter(x -> x.contains("H")).collect(Collectors.toList());
    System.out.println(list1);
}
```

**利用Map返回一个新的对象**

```java
public class MyStream {

    public static void main(String[] args) {
        ArrayList<String> arrayList = new ArrayList<>();
        arrayList.add("Hello");
        arrayList.add("World");
        arrayList.add("Good");
        arrayList.add("hi");
        arrayList.add("World");
        //利用map 来处理数据
        List<String> list1 = arrayList.stream().map(x-> x.toUpperCase()).filter(x -> x.contains("H")).collect(Collectors.toList());
        List<Person> personList = arrayList.stream().map(x-> {
                return new Person(x,10);
        }).collect(Collectors.toList());

        System.out.println(personList);
    }
}

class Person{
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

### 6）分页

```java
public static void main(String[] args) {
    ArrayList<String> arrayList = new ArrayList<>();
    arrayList.add("1");
    arrayList.add("2");
    arrayList.add("3");
    arrayList.add("4");
    arrayList.add("5");
    arrayList.add("6");
    arrayList.add("7");
    arrayList.add("8");
    arrayList.add("9");
    arrayList.add("10");

    //分页 skip表示跳过多少行，这里是按每页显示3条数据的规则
    List<String> list = arrayList.stream().skip(3*2).limit(3).collect(Collectors.toList());
    System.out.println(list); //[7, 8, 9]

}
```

### 7）匹配判断

```java
public static void main(String[] args) {
    ArrayList<String> arrayList = new ArrayList<>();
    arrayList.add("11");
    arrayList.add("21");
    arrayList.add("31");
    arrayList.add("41");
    arrayList.add("51");
    arrayList.add("61");
    arrayList.add("71");
    arrayList.add("81");
    arrayList.add("91");
    //判断arrayList中的每个元素，是否都包含有1 字符
    boolean b = arrayList.stream().allMatch(x -> x.contains("1"));
    //判断arrayList中的任意一个元素，是否包含61
    boolean c = arrayList.stream().anyMatch(x -> x.contains("61"));
    System.out.println(b); //true
    System.out.println(c); //true
}
```

### 8）数据分析统计

#### 1.reduce

下面的例子，实现了最简单的Map（处理） 和 Reduce（分析统计)

```java
public class MyStream {

    public static void main(String[] args) {
        ArrayList<ShopCar> arrayList = new ArrayList<>();
        arrayList.add(new ShopCar("打火机",2,1.5));
        arrayList.add(new ShopCar("软玉溪",6,20));
        arrayList.add(new ShopCar("U盘",3,30));
        //计算总金额
        //1.map 将数量和单价计算出来，得到一个Double集合
        //2.对集合数据做分析(a, b) -> a+b  这是一个匿名方法，输入2个参数，得到一个值。第一个参数，就是第一行的值，第二个参数就是第二行的值
        //  a * b 就是代码体，做了一个相加的处理
        Double summaryMoney = arrayList.stream().map(x -> x.getAmount() * x.getPrice()).reduce((a, b) -> a + b).get();
        System.out.println(summaryMoney);
    }
}

/**
 * 购物车
 */
class ShopCar {
    private String productName; //商品名称
    private int amount;  //商品数量
    /**
     * 单价
     */
    private double price;

    public ShopCar(String productName, int amount, double price) {
        this.productName = productName;
        this.amount = amount;
        this.price = price;
    }

    public String getProductName() {
        return productName;
    }

    public int getAmount() {
        return amount;
    }

    public double getPrice() {
        return price;
    }
}
```

#### 2.summaryStatistics 统计摘要

DoubleStream有个方法summaryStatistics()

```java
IntStream mapToInt(ToIntFunction<? super T> mapper);
DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper);
LongStream mapToLong(ToLongFunction<? super T> mapper);
```

实例

```java
public static void main(String[] args) {
    ArrayList<ShopCar> arrayList = new ArrayList<>();
    arrayList.add(new ShopCar("打火机",2,1.5));
    arrayList.add(new ShopCar("软玉溪",6,20));
    arrayList.add(new ShopCar("U盘",3,30));

    //直接得到一个Sum值
    double sum = arrayList.stream().mapToDouble(x -> x.getAmount() * x.getPrice()).sum();
    
    //返回一个统计摘要
    DoubleSummaryStatistics doubleSummaryStatistics = arrayList.stream().mapToDouble(x -> x.getAmount() * x.getPrice()).summaryStatistics();
    System.out.println(doubleSummaryStatistics.getAverage());
    System.out.println(doubleSummaryStatistics.getMax());
    System.out.println(doubleSummaryStatistics.getSum());
    System.out.println(doubleSummaryStatistics.getMin());
    System.out.println(doubleSummaryStatistics.getCount());

}
```

