# Java8 新特性 笔记

## 1.Lambda

### 前世今生

以前只能用匿名内部类的方式来实现，缺点：

1）语法相对复杂

2）在调用内部类的上下文中，指引和this的指代容易混淆

3）类加载和实例创建语法不可避免，lambda可以简化写法

4）不能引用外部的非final对象

5）不能抽象化控制流程，可以部分解决

**6）只能运行在创建它的线程种，不能运行在多线程中**

###  构成

1）参数列表

2）->

3) {代码块 }

### 案例

#### 用lambda简化Runnable接口的实现方式。

```java
 //测试Lambda用法
    @Test
    public void test1(){

        int j = 2;
        new Runnable() {
            @Override
            public void run() {
                System.out.println("用匿名内部类的方式实现Runnable接口");
                System.out.println(j);
            }
        }.run();

        int i = 1;
        Runnable r = ()->{
            System.out.println("用Lambda的方式实现Runnable接口");
            System.out.println(i);
//            i++; //可以访问外部普通变量了，但还是无法操作
        };
        r.run();

    }
```

#### 用Lambda实现自定义接口

```java
interface MyAction{
        void execute(String content,Integer seq);
    }

    @Test
    public void test2(){
        new MyAction() {
            @Override
            public void execute(String content,Integer seq) {
                System.out.println(content);
            }
        }.execute("用匿名内部类的方式实现MyAction接口",2);

        //参数列表，也可以不写参数的类型
        MyAction myAction = ( content, seq) -> {
            System.out.println(content+"seq:"+seq);
        };
        myAction.execute("用Lambda的方式实现MyAction接口",4);

}
```

## 2.Stream 泛型接口 代表数据流

### 概述

**Stream 作为 Java 8 的一大亮点，它与 java.io 包里的 InputStream 和 OutputStream 是完全不同的概念**，Java 8 中的 Stream 是对集合（Collection）对象功能的增强， Stream API 借助于同样新出现的 Lambda 表达式，极大的提高编程效率和程序可读性。

使用 Stream API 无需编写一行多线程的代码，就可以很方便地写出高性能的**并发**程序

```java
Stream<Person> stream = people.parallelStream();
```

Stream 就如同一个**迭代器（Iterator）**，单向，不可往复，数据只能遍历一次，遍历过一次后即用尽了，就好比流水从面前流过，一去不复返

而和迭代器又不同的是，Stream 可以并行化操作，迭代器只能命令式地、串行化操作。顾名思义，当使用串行方式去遍历时，每个 item 读完后再读下一个 item。而使用并行去遍历时，数据会被分成多个段，其中每一个都在不同的线程中处理，然后将结果一起输出。Stream 的并行操作依赖于 Java7 中引入的 Fork/Join 框架（JSR166y）来拆分任务和加速处理过程。



### Java 的并行 API 演变历程

如下几个阶段

1. 1.0-1.4 中的 java.lang.Thread
2. 5.0 中的 java.util.concurrent
3. 6.0 中的 Phasers 等
4. 7.0 中的 Fork/Join 框架
5. 8.0 中的 Lambda

Stream 的另外一大特点是，数据源本身可以是无限的。



作用：类似于C# 中的List.Select(   过滤器，即用lambda  )

**java中没有C#中的委托Delegate **，就采用Stream.Filter( lambda ) 来实现的。

Steam的api

https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/



### 特性：

  1）不是一个数据结构，不能存储数据，可以认为是一个数据源

  2）通过管道（Filter）操作数据

  3）stream() :创建一个Stream接口实现类的对象

如：Stream<Person> stream = people.stream(); //pepole 为 ArrayList<Person> 集合对象

### 过滤器：

如：stream = stream.Filter(p->{p.Sex == 1});

### 使用例程

```java
public class Person {
    public enum Sex  {
            MALE,FLMALE //男 和 女
    }

    private String name;
    private int age;
    private Sex sex;

    public Person(String name, int age, Sex sex) {
        this.name = name;
        this.age = age;
        this.sex = sex;
    }

    public Person() {
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

    public Sex getSex() {
        return sex;
    }

    public void setSex(Sex sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", sex=" + sex +
                '}';
    }
}
```

```java
private static List<Person> createPeople() {
    List<Person> peoples = new ArrayList<>();
    Person person = new Person("a1", 18, Person.Sex.MALE);
    peoples.add(person);
    person = new Person("a2", 19, Person.Sex.MALE);
    peoples.add(person);
    person = new Person("a3", 20, Person.Sex.FLMALE);
    peoples.add(person);
    person = new Person("a4", 21, Person.Sex.FLMALE);
    peoples.add(person);
    return peoples;
}
```

```java
public void test3(){
    List<Person> people = createPeople();
    Stream<Person> stream = people.stream();
    //使用过滤器
    Stream<Person> personStream = stream.filter(p -> p.getSex() == Person.Sex.MALE);
    personStream.forEach(
            p->{
                System.out.println(p);
            }
         //还可以流式的操作
        stream.filter(p -> p.getSex() == Person.Sex.MALE).forEach(p->{
            System.out.println(p);
        });
    );
}
```

流操作示例

```java
int sum = widgets.stream()
.filter(w -> w.getColor() == RED)
 .mapToInt(w -> w.getWeight())
 .sum(); 
```

```java
List<Person> personList = stream.collect(Collectors.toList());
```

### 构造流的几种方法

// 1. Individual values
Stream stream = Stream.of("a", "b", "c");
// 2. Arrays
String [] strArray = new String[] {"a", "b", "c"};
stream = Stream.of(strArray);
stream = Arrays.stream(strArray);
// 3. Collections
List<String> list = Arrays.asList(strArray);
stream = list.stream();

需要注意的是，对于基本数值型，目前有三种对应的包装类型 Stream：

**IntStream、LongStream、DoubleStream**。当然我们也可以用 Stream<Integer>、Stream<Long> >、Stream<Double>，但是 装箱 和 拆箱 会很耗时，所以特别为这三种基本数值型提供了对应的 Stream。

Java 8 中还没有提供其它数值型 Stream



### 流的三种操作类型

- **Intermediate**：一个流可以后面跟随零个或多个 intermediate 操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。
- **Terminal**：一个流只能有一个 terminal 操作，当这个操作执行后，流就被使用“光”了，无法再被操作。所以这必定是流的最后一个操作。Terminal 操作的执行，才会真正开始流的遍历，并且会生成一个结果，或者一个 side effect。
- **short-circuiting**。用以指：
  - 对于一个 intermediate 操作，如果它接受的是一个无限大（infinite/unbounded）的 Stream，但返回一个有限的新 Stream。
  - 对于一个 terminal 操作，如果它接受的是一个无限大的 Stream，但能在有限的时间计算出结果。

### 流的操作

接下来，当把一个数据结构包装成 Stream 后，就要开始对里面的元素进行各类操作了。常见的操作可以归类如下。

- Intermediate：

map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered

- Terminal：

forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator

- Short-circuiting：

anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 limit



DoubleStream的例子

```java
public void test4(){
    List<Person> peoples = createPeople();
    Stream<Person> stream = peoples.stream();
    double avgHeight = stream.filter(p -> p.getName().indexOf("a") >= 0)
            .mapToDouble(p -> p.getHeigth()) //转换为DoubleStream
            .average()
            .getAsDouble();
    System.out.println(avgHeight);

}
```



## 3.LocalDate、LocalTime、LocalDateTime

### LocalDate

```java
import java.time.*;

public class jdk8Date {
    /**
     * LocalDate是一个不可变的类，它表示默认格式(yyyy-MM-dd)的日期，
     * @param args
     */
    public static void main(String[] args) {
        jdk8Date java8tester = new jdk8Date();
        java8tester.testLocalDateTime();
    }
    //local
    public void testLocalDateTime(){
        // 当前日期yyyy-MM-dd
        LocalDate localDate1 = LocalDate.now();
        System.out.println(localDate1);
        // 创建一个日期
        LocalDate localDate2 = LocalDate.of(2017, 10, 17);
        System.out.println(localDate2);
        // 获取指定时区的当前时间
        LocalDate localDate3 = LocalDate.now(ZoneId.of("Asia/Kolkata"));
        System.out.println(localDate3);
        // 格林威治时间+天数
        //默认获取的是以UTC时区，世界协调时间，为基础
        LocalDate localDate4 = LocalDate.ofEpochDay(365);
        System.out.println(localDate4);
        // 某年的第几天的日期
        LocalDate localDate5 = LocalDate.ofYearDay(2017, 200);
        System.out.println(localDate5);
    }
}
```

### LocalTime

```java
import java.time.LocalTime;
import java.time.ZoneId;

public class LocalTimes {
    /**
     * LocalTime是一个不可变的类，它的实例代表一个符合人类可读格式的时间，默认格式是hh:mm:ss.zzz。
     * @param args
     */
    public static void main(String[] args) {
        //当前时间
        LocalTime time = LocalTime.now();
        System.out.println("Current Time="+time);
        // 创建当前时间
        LocalTime specificTime = LocalTime.of(12,20,25,40);
        System.out.println("Specific Time of Day="+specificTime);

        // 获取指定时区当前时间
        LocalTime timeKolkata = LocalTime.now(ZoneId.of("Asia/Kolkata"));
        System.out.println("Current Time in IST="+timeKolkata);

        // 当天多少秒的时间
        LocalTime specificSecondTime = LocalTime.ofSecondOfDay(10000);
        System.out.println("10000th second time= "+specificSecondTime);
    }
}
```

### LocalDateTime

```java
import java.time.*;

public class LocalDateTimes {
    /**
     * LocalDateTime是一个不可变的日期-时间对象，它表示一组日期-时间，默认格式是yyyy-MM-dd-HH-mm-ss.zzz。
     * 它提供了一个工厂方法，接收LocalDate和LocalTime输入参数，创建LocalDateTime实例。
     * @param args
     */
    public static void main(String[] args) {
        // 当前日期时间
        LocalDateTime today = LocalDateTime.now();
        System.out.println("Current DateTime="+today);

        // 当前日期时间
        today = LocalDateTime.of(LocalDate.now(), LocalTime.now());
        System.out.println("Current DateTime="+today);

        // 指定时间日期时间
        LocalDateTime specificDate = LocalDateTime.of(2014, Month.JANUARY, 1, 10, 10, 30);
        System.out.println("Specific Date="+specificDate);

        // 当前指定时区日期时间
        LocalDateTime todayKolkata = LocalDateTime.now(ZoneId.of("Asia/Kolkata"));
        System.out.println("Current Date in IST="+todayKolkata);

        // 格林威治后多少分钟的日期时间
        LocalDateTime dateFromBase = LocalDateTime.ofEpochSecond(10000, 0, ZoneOffset.UTC);
        System.out.println("10000th second time from 01/01/1970= "+dateFromBase);
    }

}
```

### Instant

```java
import java.time.Duration;
import java.time.Instant;

public class instants {
    /**
     * Instant类是用在机器可读的时间格式上的，它以Unix时间戳的形式存储日期时间。
     * @param args
     */
    public static void main(String[] args) {
        // 当前时间戳
        Instant timestamp = Instant.now();
        System.out.println("Current Timestamp = "+timestamp); // Current Timestamp = 2018-06-26T15:54:21.310Z

        Instant specificTime = Instant.ofEpochMilli(timestamp.toEpochMilli()); // Specific Time = 2018-06-26T15:54:21.310Z
        System.out.println("Specific Time = "+specificTime);

        Duration thirtyDay = Duration.ofDays(30);
        System.out.println(thirtyDay); // PT720H
    }
}
```

### DateUtils 

```java
import java.time.*;

public class DateUtils {
    /**
     * 大多数日期/时间API类都实现了一系列工具方法，如：加/减天数、周数、月份数，等等。
     * 还有其他的工具方法能够使用TemporalAdjuster调整日期，并计算两个日期间的周期。
     * @param args
     */
    public static void main(String[] args) {
        // 当前日期时间,UTC(世界协调时间格式)
        LocalDateTime today = LocalDateTime.now();
        System.out.println("Current DateTime="+today);

        // 当前日期时间
        today = LocalDateTime.of(LocalDate.now(), LocalTime.now());
        System.out.println("Current DateTime="+today);

        // 指定时间日期时间
        LocalDateTime specificDate = LocalDateTime.of(2014, Month.JANUARY, 1, 10, 10, 30);
        System.out.println("Specific Date="+specificDate);

        // 当前指定时区日期时间
        LocalDateTime todayKolkata = LocalDateTime.now(ZoneId.of("Asia/Kolkata"));
        System.out.println("Current Date in IST="+todayKolkata);

        // 格林威治后多少分钟的日期时间
        LocalDateTime dateFromBase = LocalDateTime.ofEpochSecond(10000, 0, ZoneOffset.UTC);
        System.out.println("10000th second time from 01/01/1970= "+dateFromBase);
    }
}
```



### DateTimeFormatter

将字符串解析为日期对象

```java
public static void main(String[] args) {
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy.MM.dd HH:mm:ss");
    LocalDateTime localDateTime = LocalDateTime.parse("2018.09.30 17:44:25", formatter);
    System.out.println(localDateTime);


}

```

## 4.Optional类

Optional类，是一个容器类，用于包裹一个对象，更容易对这个对象进行空值的判断和处理