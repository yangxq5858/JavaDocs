# Java 注解 笔记



**注解是一个特殊的类**，采用@interface 关键字标注，如：

```java
public @interface MyComponentScan {
}
```

## 1.四个标注的元注解类型

Java5.0定义了4个标准的meta-annotation类型，它们被用来提供对其它 annotation类型作说明。Java5.0定义的元注解：
　1.@Target,
　2.@Retention,
　3.@Documented,
　4.@Inherited

### @Target：

　　　@Target说明了Annotation所修饰的对象范围：Annotation可被用于 packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数）。在Annotation类型的声明中使用了target可更加明晰其修饰的目标。

　　**作用：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）**

　　**取值(ElementType)有：**

　　　　1.CONSTRUCTOR:用于描述构造器
　　　　2.FIELD:用于描述域
　　　　3.LOCAL_VARIABLE:用于描述局部变量
　　　　4.METHOD:用于描述方法
　　　　5.PACKAGE:用于描述包
　　　　6.PARAMETER:用于描述参数
　　　　7.TYPE:用于描述类、接口(包括注解类型) 或enum声明

### **@Retention：**

　　**@Retention**定义了该Annotation被保留的时间长短：某些Annotation仅出现在源代码中，而被编译器丢弃；而另一些却被编译在class文件中；编译在class文件中的Annotation可能会被虚拟机忽略，而另一些在class被装载时将被读取（请注意并不影响class的执行，因为Annotation与class在使用上是被分离的）。使用这个meta-Annotation可以对 Annotation的“生命周期”限制。

　　**作用：表示需要在什么级别保存该注释信息，用于描述注解的生命周期（即：被描述的注解在什么范围内有效）**

　　**取值（RetentionPoicy）有：**

　　　　1.SOURCE:在源文件中有效（即源文件保留）
　　　　2.CLASS:在class文件中有效（即class保留）
　　　　3.RUNTIME:在运行时有效（即运行时保留）

　　Retention meta-annotation类型有唯一的value作为成员，它的取值来自java.lang.annotation.RetentionPolicy的枚举类型值

### **@Documented:**

　　**@**Documented用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。Documented是一个标记注解，没有成员

### **@Inherited：**

　　@Inherited 元注解是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。表示标注的类的子类也同样具有这个注解。

## 2.注解如何使用的

1）定义注解类 A

2）在目标类B上（或者方法、参数等）标注 A注解

3）定义一个C类，利用反射读取B类的注解信息，进行具体的逻辑处理



## 3.给注解添加属性

注解的属性写法和普通类的方法类似。**属性的返回值，就是属性的参数类型**

属性的类型：基本类型、环境变量、注解类型、Class类型，前面这些类型的数组

### 基本类型属性

```java
@Target(value = {ElementType.TYPE})
@Retention(value = RetentionPolicy.RUNTIME)  //必须设置为Runtime，否则，用反射是无法读取到数据的
public @interface MyComponent {
    String value(); //这个其实是一个属性，只是形式上和方法类似（没有public，private等标识）
    String color() default ""; //设置默认值
}
```

当**只有**一个属性**必须赋值**时，并且属性的名称为value时，在注解上可以不用加 “**value =** ” 

ps：仅仅限于属性的名称为value时

```java
@MyComponent("MyBean01")
public class MyBean {

    //@MyComponent
    public void init(){

    }
}
```

上面的代码，可以使用简写的方式，是因为，color 有了默认值，如果color没有默认值，就不能用简写方式了，因为不止一个属性需要赋值了。

------

### 数组属性

```java
@Target(value = {ElementType.TYPE})
@Retention(value = RetentionPolicy.RUNTIME)  //必须设置为Runtime，否则，用反射是无法读取到数据的
public @interface MyComponent {

    String value(); //写成value时，如果只有它这个属性需要赋值时，可以省略value=

    MyColorType color() default MyColorType.Red; //设置默认值

    int[] lengths() default {}; //数组

}
```

当调用的时候，只有一个值时，可以省略掉{}，如下的lengths = 12

```java
@MyComponent(value = "MyBean01",color = MyColorType.Green,lengths = 12)

@MyComponent(value = "MyBean01",color = MyColorType.Green,lengths = {12,13})
public class MyBean {
    public void init(){

    }
}
```

### 注解类型属性

```java
@Target(value = {ElementType.TYPE})
@Retention(value = RetentionPolicy.RUNTIME)  //必须设置为Runtime，否则，用反射是无法读取到数据的
public @interface MyComponent {

    String value(); //写成value时，如果只有它这个属性需要赋值时，可以省略value=

    MyColorType color() default MyColorType.Red; //设置默认值

    int[] lengths() default {}; //数组

//    long idx();

//    char sChar();

    MyAnnotation myAnnotation();
    //MyAnnotation myAnnotation() default @MyAnnotation("123"); //带默认值的
}
```

MyAnnotation 的定义如下：

```java
public @interface MyAnnotation {
    String value();
}
```

调用

```java
@MyComponent(value = "MyBean01",color = MyColorType.Green,lengths = {12,13},myAnnotation = @MyAnnotation("abc"))
public class MyBean {

    //@MyComponent
    public void init(){

    }
}
```

测试

```java
public class MyAnnotationTest {

    @Test
    public void test1(){
        boolean b = MyBean.class.isAnnotationPresent(MyComponent.class);
        if (b) {
            MyComponent myComponentAnnotation = MyBean.class.getAnnotation(MyComponent.class);
            System.out.println(myComponentAnnotation.value());
            System.out.println(myComponentAnnotation.color());
            System.out.println(myComponentAnnotation.color().getDeclaringClass());
            System.out.println(myComponentAnnotation.myAnnotation().value());
        }
    }
}
```

### Class类型属性

```java
@Target(value = {ElementType.TYPE})
@Retention(value = RetentionPolicy.RUNTIME)  //必须设置为Runtime，否则，用反射是无法读取到数据的
public @interface MyComponent {

    String value(); //写成value时，如果只有它这个属性需要赋值时，可以省略value=

    MyColorType color() default MyColorType.Red; //设置默认值

    int[] lengths() default {}; //数组

//    long idx();

//    char sChar();

    MyAnnotation myAnnotation() default @MyAnnotation("123");

    Class<?> clazz();
}

```

```java
@MyComponent(clazz = Green.class,value = "MyBean01",color = MyColorType.Green,lengths = {12,13},myAnnotation = @MyAnnotation("abc"))
```



##4. 函数式接口

```java 
   //这是一个函数式接口注解，这个注解就限制了，该接口只能有一个方法，静态方法和默认实现除外
   @FunctionalInterface
   public interface Runnable {
       /**
        * When an object implementing interface <code>Runnable</code> is used
        * to create a thread, starting the thread causes the object's
        * <code>run</code> method to be called in that separately executing
        * thread.
        * <p>
        * The general contract of the method <code>run</code> is that it may
        * take any action whatsoever.
        *
        * @see     java.lang.Thread#run()
        */
       public abstract void run();
   }
   
```



