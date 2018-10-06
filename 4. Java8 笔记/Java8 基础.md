# Java 基础 学习笔记

## 1.StringBuffer、StringBuilder、String的区别

String类的特点：

  String类对象有2种实例化方式：

​       直接赋值：只开辟一块堆内存空间，可以自动入池

​       构造方法：开辟2块堆内存空间，不会自动入池，需要使用intern()手工入池

 任何一个字符串都是String类的匿名对象；

 字符串一旦声明则不可改变，可以改变的只是String类对象的引用。

********************

**String类的内容不可改变**

********************

StringBuffer的内容是可以改变的，它的创建，必须实例化。

| 名称   | String                                   | StringBuffer                             |                   |
| ---- | ---------------------------------------- | ---------------------------------------- | ----------------- |
| 出现时间 | JDK1.0                                   | JDK1.0                                   | JDK1.5            |
| 继承关系 | 实现了CharSequence 接口                       | 实现了CharSequence 接口                       |                   |
|      | public final class String     implements java.io.Serializable, Comparable<String>, CharSequence | public final class StringBuffer    extends AbstractStringBuilder    implements java.io.Serializable, CharSequence | 和StringBuffer基本一样 |
| 是否同步 |                                          | 同步方法                                     | 异步方法              |



从上面的继承关系，可以看到，只要我们看到CharSequence类型的，我们都可以用String类型。因为String类型是实现了CharSequence接口

```java
public static void main(String[] args) {
    //"Hello CharSequence!" 是String的匿名实例化对象
    //charSequence = "Hello CharSequence!" 表示向上转型
    CharSequence charSequence = "Hello CharSequence!";
    System.out.println(charSequence);
}
```



String 和 StringBuffer 转换

方法一：利用StringBuffer的构造方法。

```java
public final class StringBuffer extends AbstractStringBuilder  implements java.io.Serializable, CharSequence{
    public StringBuffer(String str); 
    StringBuffer append(String str);
    public synchronized String toString();
      
}
```



```java
public static void main(String[] args) {
    StringBuffer stringBuffer = new StringBuffer("Hello"); //将String 变为StringBuffer
    System.out.println(stringBuffer);

}

```



方法二：利用append()方法将String变为StringBuffer。

```java
public static void main(String[] args) {
    StringBuffer stringBuffer = new StringBuffer();
    stringBuffer.append("hello");
    System.out.println(stringBuffer);

}

```



将StringBuffer变为String。

方法一：用toString()方法转换

方法二：利用String的构造方法，我们看到这里，也可以将StringBuilder转换为String。

```java
public String(StringBuffer buffer) 
public String(StringBuilder builder)
```

StringBuffer类中，特殊的方法

```java
//字符串反转
public synchronized StringBuffer reverse()
//在指定索引位置，插入数据
public  StringBuffer insert(int offset, xxx类型的数据)
//删除部分数据
public synchronized StringBuffer delete(int start, int end);  

```



## 2.Runtime类

### 1）Runtime类的作用

在每一个JVM进程里面都会有一个Runtime类的对象，这个对象的主要功能是取得与运行时相关的环境或者创建新的进程等操作。

****************************************

**Runtime类定义的时候，它的构造方法被私有化了。**

**单例设计模式**

**可以手工调用GC**

**********************

```java
public class Runtime {
    private static Runtime currentRuntime = new Runtime();
    //取得Runtime的实例化对象
    public static Runtime getRuntime() {
        return currentRuntime;
    }

    /** Don't let anyone else instantiate this class */
    private Runtime() {} //单例模式设计
  
    public native long totalMemory();
    public native long totalMemory();
    public native long freeMemory();
    //释放垃圾
    public native void gc();
    //执行 可执行程序，一般是不用的
    public Process exec(String command) throws IOException
    //与之对应的，可以调用Process的销废进程
    Process类中的 public abstract void destroy();
}
```



```java
public static void main(String[] args) throws InterruptedException, IOException {
    Runtime runtime = Runtime.getRuntime();
    Process process = runtime.exec("C:\\Windows\\system32\\mspaint.exe");
    sleep(3000);
    process.destroy();
}
```

```java
public static void main(String[] args) {
    Runtime runtime = Runtime.getRuntime();
    System.out.println("1.max Memory:"+runtime.maxMemory()); //  /1024/1024 得到MB
    System.out.println("1.total Memory:"+runtime.totalMemory());
    System.out.println("1.free Memory:"+runtime.freeMemory());

    String str="";
    for (int i = 0; i < 1000; i++) {
        str += i;
    }

    System.out.println("2.max Memory:"+runtime.maxMemory()); //  /1024/1024 得到MB
    System.out.println("2.total Memory:"+runtime.totalMemory());
    System.out.println("2.free Memory:"+runtime.freeMemory());
    //调用gc方法，释放内存
    runtime.gc();
    System.out.println("3.max Memory:"+runtime.maxMemory()); //  /1024/1024 得到MB
    System.out.println("3.total Memory:"+runtime.totalMemory());
    System.out.println("3.free Memory:"+runtime.freeMemory());

}
```

## 3. System类

 ```java
public final class System {
     //取得当前的系统时间，单位：毫秒
     public static native long currentTimeMillis(); 
  
     //重复定义了一个gc方法
     public static void gc() {
        Runtime.getRuntime().gc();
     }
  
}
 ```

```java
public static void main(String[] args) {
    long start = System.currentTimeMillis();
    String str = "";
    for (int i = 0; i < 30000 ; i++) {
        str += i;
    }
    //Runtime.getRuntime().gc();
    long end = System.currentTimeMillis();
    System.out.println("花费了: "+(end - start) + " 毫秒");
}
```

##4.对象克隆

```java
public class Object {
  //如果，我们要想对 对象的回收做一定的处理，需要复写这个方法
  //Throwable类是最大的错误顶级类（Error和Exception 都是从这个继承下来的，Error类是我们无法处理的，只有Exception类，才能处理）
  //在对象回收时，就算出现了任何的异常，也不会中断执行，不会影响对象的消亡这个结果的。
   protected void finalize() throws Throwable { }
   //对象克隆 ，要求被克隆的类，必须实现Cloneable接口 
   protected native Object clone() throws CloneNotSupportedException;
}
```



**final、finally、finalize的区别：**

final：**关键字**，定义不能被继承的类、不能被覆写的方法、常量。

finally：**关键字**，异常的统一出口；

finalize：**方法**，Object类提供的方法 protected void finalize() throws Throwable { }



示例：

1. 必须覆写clone方法
2. 必须实现Cloneable接口

```java
//1. 必须实现 Cloneable 接口
class Book implements Cloneable{

    private String title;
    private Double price;

    public Book(String title, Double price) {
        this.title = title;
        this.price = price;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public Double getPrice() {
        return price;
    }

    public void setPrice(Double price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return "Book{" +
                "title='" + title + '\'' +
                ", price=" + price +
                '}';
    }

    //2. 对象要想具有Clone，必须覆写，因为这是一个protected方法
    //我们改为public方法
    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

public class MyClone {

    public static void main(String[] args) throws CloneNotSupportedException {
        Book book1 = new Book("Java开发",68.5);
        Book book2 = (Book)book1.clone();
        book2.setTitle("JSP开发");
        System.out.println(book1);
        System.out.println(book2);
    }
}

```



## 5.标识接口（空接口）

标识接口就是说，一个接口中没有方法定义，也没有成员定义，它表示一种能力而已。又叫空接口。



## 6.数字操作类

### 1）Math类

Math类里面的方法，都是static的，因为，这个类是没有属性的。

```
public final class Math {
   //这个类不能满足我们的大于0.5 就进位，它是一种科学的进位方式
   public static long round(double a);
   public static int round(float a);
}
```

### 2）Random类

这个类的主要功能，是取得随机数

```
protected int next(int bits) ; //bits 表示一个边界值
```



```java
public class MyString {

    public static void main(String[] args) {
        //生成一个小于36的6位不重复的数，彩票
        Random random = new Random();
        int foot = 0;
        int data[] = new int[6];
        //当不知道要循环几次时，要用到while条件
        while (foot < 6){
            int i = random.nextInt(37);
            if (!isRepeat(data,i)){
                data[foot++] = i;
            }
        }
        Arrays.sort(data); //用工具类，排序数组
        for (int i = 0; i < data.length; i++) {
            System.out.print(data[i]+"、");
        }
    }

    public static boolean isRepeat(int data[],int temp){
        if (temp == 0){
            return true;
        }
        for (int i = 0; i < data.length; i++) {
            if (data[i] == temp){
                return true;
            }
        }
        return false;
    }

}
```

### 3）大数字操作类

BigInteger

BigDecimal

他们的构造方法，是可以接收**String类型**的。

我们用

  Double d = Double.MAX_VALUE;

  System.out.println(d * d);

就会输出一个**Infinity** **无限**大的一个数。

但大数据有可能会出现，怎么办，我们可以用String类型来做中间过渡处理，就需要可以接收String类型的。

*******************

Number 类是所有数字型类型的顶级

***************************

```java


public class BigInteger extends Number implements Comparable<BigInteger> {
    public BigInteger(String val) 
    public BigInteger(int signum, byte[] magnitude) {      
}
```

```java
public static void main(String[] args) {
    BigInteger bigIntegerA = new BigInteger("45654646546546465321387");
    BigInteger bigIntegerB = new BigInteger("4565464655321387");
    System.out.println("加法："+bigIntegerA.add(bigIntegerB));
    System.out.println("减法："+bigIntegerA.subtract(bigIntegerB));
    System.out.println("乘法："+bigIntegerA.multiply(bigIntegerB));
    System.out.println("除法："+bigIntegerA.divide(bigIntegerB));
    //返回1个2位的数组，0 位商数，1为余数
    BigInteger[] bigIntegers = bigIntegerA.divideAndRemainder(bigIntegerB);
    System.out.println("除法带余数的，商数为："+ bigIntegers[0] + "余数："+bigIntegers[1]);
}
```

但Java提供的计算，只是简单的 + - *  / ,如果要进行其它运算，就需要借助第三方的工具包。



















