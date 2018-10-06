# Java8 反射 笔记

## 1. Class 类对象的实例化

### 概述

java.lang.Class :

​     是一个类

​     是所有反射的源头

### 实例化方式

1）调用Object类的getClass() 方法获得

通过getClass() 获得一个Class类实例化对象，如：

Date date = new Date();

Class clazz =  date.getClass(); //这个就是获得一个Class **类** 的实例化对象

区别：这是通过一个类的实例化对象，调用getClass()取得的

2) 类的Class属性(**使用最多的一种**，Spring，Mybatis等第三方使用较多)

如：Class clazz = Date.class ;

3）Class类提供的forName()方法

```java
public void test2() throws ClassNotFoundException {

        //第一种方式 使用对象Object.getClass() 来获得
        Date date = new Date();
        Class clazz =  date.getClass();
        System.out.println(clazz);


        //第二种方式 Spring使用最多的一种
//        Class<?> clazz = Date.class;
//        System.out.println(clazz);

       //第三种方式，官方使用最多的
//        Class<?> clazz = Class.forName("java.util.Date");
//        System.out.println(clazz);

    }
```

都输出的是

```java
class java.util.Date
```

第三种方式的好处：不需要Import java.util.Date 包

### 实例化对象

```java
public T newInstance()
    throws InstantiationException, IllegalAccessException
    
    
```

## 2.为什么要用反射

下面这个例子，是一个工厂模式

```java
interface Fruit {
    public void eat();
}

class Apple implements Fruit {
    @Override
    public void eat() {
        System.out.println(" 吃苹果");
    }
}

class Orange implements Fruit {
    @Override
    public void eat() {
        System.out.println(" 吃橘子");
    }
}

//bean工厂，在没有用反射前，要用很多个if来判断，导致，我们如果增加了一个Fruit实现，就要改造下面这段代码，利用反射，我们可以不再修改了，也就做到了变化隔离。
class BeanFactory {
    public static Fruit getInstance(String className) {
        //修改前的，直接New一个对象，就会导致我们的工厂类，要不断的变化

//        if (className.equals("Apple")) {
//            return new Apple();
//        } else if (className.equals("Orange")) {
//            return new Orange();
//        } else {
//            return null;
//        }

        //采用反射，解耦
        Fruit fruit = null;
        try {
            fruit = (Fruit) Class.forName(className).newInstance();
        } catch (Exception e){}
        return fruit;
    }
}

public class TestBeanFactory {
    public static void main(String[] args) {
//        Fruit fruit = BeanFactory.getInstance("Apple");
//        fruit.eat();

        Fruit fruit = BeanFactory.getInstance("com.hx.bean.Orange");
        fruit.eat();


    }
}
```

## 3.使用反射调用构造方法

上面的例子都是调用的构造方法，都是无参数的。**newInstance()**

```java
public class Book {
    private String title;
    private Double price;

    public Book(String title,Double price){
        this.title = title;
        this.price = price;
    }

    @Override
    public String toString() {
        return "Book{" +
                "title='" + title + '\'' +
                ", price=" + price +
                '}';
    }
}
```

测试

```java
@Test
public void test4() throws Exception {
    Class<?> clazz = Class.forName("com.hx.bean.Book");
    Object instance = clazz.newInstance();
    System.out.println(instance);
}
```

结果出错，Book类没有提供无参数

```java
Caused by: java.lang.NoSuchMethodException: com.hx.bean.Book.<init>()
```

### 分析Class类

获取所有的构造方法

```java
public Constructor<?>[] getConstructors() throws SecurityException
```

获取指定参数顺序的构造

```java
public Constructor<T> getConstructor(Class<?>... parameterTypes) {
    checkMemberAccess(Member.PUBLIC, Reflection.getCallerClass(), true);
    return getConstructor0(parameterTypes, Member.PUBLIC);
}
```

上面2个方法，都是返回的一个java.lang.reflect.Constructor，这个类就在反射包路径下了，正式进入到反射主题了。

我们来看下这个Constructor类中，有实例化方法

```java
public T newInstance(Object ... initargs)
    throws InstantiationException, IllegalAccessException,
           IllegalArgumentException, InvocationTargetException
```

### 带参数构造器的实例化对象

```java
    @Test
    public void test4() throws Exception {
        Class<?> clazz = Class.forName("com.hx.bean.Book");
        //通过有参数的获取到构造器
        Constructor<?> constructor = clazz.getConstructor(String.class, Double.class);
        //用构造器来实例化
        Object instance = constructor.newInstance("Tomcat",88.50);
        System.out.println(instance);
//        Object instance = clazz.newInstance();
//        System.out.println(instance);
    }
```

虽然可以实现，调用有参构造器来实例化对象了，还是比较麻烦的，也不利于统一代码编写，故，**我们每个类都要保留一个无参构造方法。**

ps：只要在类中，没有定义一个有参数的构造方法，编译器默认就会自动创建一个无参构造方法的。但如果有了一个带参数的，编译器就不会自动生成无参构造方法。

## 4.反射调用方法

### 对象实例化3种方式

new，反射，克隆（Clone）

Class类中 获取到的方法是反射包中的  java.lang.reflect.Method

```java
//取得所有的方法
public Method[] getMethods() throws SecurityException
//获取指定名字的方法
public Method getMethod(String name, Class<?>... parameterTypes)
        throws NoSuchMethodException, SecurityException
```

```java
@Test
public void test5() throws Exception {
    Class<?> clazz = Class.forName("com.hx.bean.Book");
    Object instance = clazz.newInstance();
    String fieldName = "title";
    String methodName = getSetterMethodName(fieldName);
    Method method = clazz.getMethod(methodName, String.class);
    method.invoke(instance,"小莉");
    System.out.println(instance);

}
//调用有参构造器，实例化后，再调用set方法
    public void test6() throws Exception {
        Class<?> clazz = Class.forName("com.hx.bean.Book");
        Constructor<?> constructor = clazz.getConstructor(String.class, Double.class);
        Object instance = constructor.newInstance("Tomcat",88.50);
        System.out.println("实例化后的值："+ instance);

        String fieldName = "title";
        String methodName = getSetterMethodName(fieldName);
        Method method = clazz.getMethod(methodName, String.class);
        method.invoke(instance,"小莉");
        System.out.println("调用方法后的值："+instance);

    }

   //获取Setter方法的名称
    private String getSetterMethodName(String fieldName){
        String methodName = "set" + fieldName.substring(0,1).toUpperCase() +                                     fieldName.substring(1);
        return methodName;
    }
```

Book类做了变化

```java
public class Book {
    private String title;
    private Double price;

    //加入了无参的构造器
    public Book() {
    }

    public Book(String title, Double price){
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
}
```

## 5.反射调用成员

```
java.lang.reflect.Field
```

```java
//获取全部属性
public Field[] getDeclaredFields() throws SecurityException
//获取指定属性
public Field getDeclaredField(String name) throws NoSuchFieldException, SecurityException 

//Field下的关键方法
//get方法
public Object get(Object obj) throws IllegalArgumentException, IllegalAccessException
//set方法
public void set(Object obj, Object value)
        throws IllegalArgumentException, IllegalAccessException
        
//注意，如果Field是private，是无法执行set方法的，但如果我们打开它的封装性，即可操作性
//就是下面这个方法，这个方法位于Field类的父类 AccessibleObject
public void setAccessible(boolean flag) throws SecurityException

```

下面一个例子，调用一个Private的属性成员，并赋值，和取值

```java
//测试调用成员
@Test
public void test7() throws Exception {
    Class<?> clazz = Class.forName("com.hx.bean.Book");
    Constructor<?> constructor = clazz.getConstructor(String.class, Double.class);
    Object instance = constructor.newInstance("Tomcat",88.50);
    Field nameField = clazz.getDeclaredField("name");
    nameField.setAccessible(true); //取消属性的封装，private就可以访问了
    nameField.set(instance,"Java开发指南");
    System.out.println(nameField.get(instance));
    System.out.println(instance);
}
```

Book类，增加了一个name的private的成员属性，**按理在外部是无法访问的，但我们利用反射，是可以做到修改和读取的**

```java
public class Book {
    private String title;
    private Double price;
    private String name; // 该属性没有设置getter 和 setter

    public Book() {
    }

    public Book(String title, Double price){
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
}
```

