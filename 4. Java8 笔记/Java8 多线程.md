# 多线程 学习 笔记

##  一、多线程基础


### 1.多线程实现

#### 1）继承java.lang.Thread类

```java 
public class MyThread extends Thread {
    private int count;
    public MyThread(int count){
        this.count = count;
    }
    @Override
    public void run() {
        while (count > 0){
            System.out.println(Thread.currentThread().getName()+"-->"+count);
            count --;
        }
    }
}
```

调用

注意，MyThread是一个类，通常我们New 实例化后，是直接调用对象的方法运行（这里的方法就是run方法，但我们如果直接调用run方法，则无法实现多线程的效果，**必须使用start()方法**，才能起到资源竞争的效果，多个线程基本同时运行。

```java 
@Test
public void test1(){
    MyThread myThread1 = new MyThread(10);
    MyThread myThread2 = new MyThread(10);
    MyThread myThread3 = new MyThread(10);

    //注意，这里不能用myThread1.run()方法调用，否则无法实现多线程的效果（即线程交替执行）
    //调用start()方法，也是执行run方法中的代码。
    myThread1.start();
    myThread2.start();
    myThread3.start();
}	
```

```java 
//Thread的源代码
/**
  IllegalThreadStateException 异常是一个运行时异常

  //这里是调用的本级操作系统的JNI函数（Java native Interface)
  //这里是一个接口，具体的实现是由JVM来实现的不同操作系统的具体实现
   private native void start0();

*/


public synchronized void start() {
    /**
     * This method is not invoked for the main method thread or "system"
     * group threads created/set up by the VM. Any new functionality added
     * to this method in the future may have to also be added to the VM.
     *
     * A zero status value corresponds to state "NEW".
     */
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    /* Notify the group that this thread is about to be started
     * so that it can be added to the group's list of threads
     * and the group's unstarted count can be decremented. */
    group.add(this);

    boolean started = false;
    try {
        start0(); //这里才是最关键的
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
              it will be passed up the call stack */
        }
    }
}
```

#### 2）实现java.lang.Runnable接口（扩展了Callable接口）推荐使用

优势：解决单继承的限制。

**java的继承都是单继承，不能继承多个类，有一定的局限性，所以，用接口实现是最好的方式。**

我们来看这个Runnable接口

```java
@FunctionalInterface //这是一个函数式接口注解，这个注解就限制了，该接口只能有一个方法，静态方法和默认实现除外
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
     * @see     java.lang.Thread##run()
     */
    public abstract void run();
}
```

**在接口里的任何方法，都是public的，即使没有写public**

**Thread类也是实现的Runnable接口**

Runnable接口与继承Thread相比的区别：

​     Runnable 只有run方法了，没有start()方法，那我们怎么启动呢？

​    **只要是多线程，我们都要用Thread来启动。**

Thread类的提供的构造方法看下。

```java
public Thread(Runnable target) {
    init(null, target, "Thread-" + nextThreadNum(), 0);
}
```

构造方法里的参数是一个Runnable的参数

```java
 public static void main(String[] args) {
     
        //都可以用同一种方式（Thread 也是实现的Runnable接口），
        //即调用构造方法返回一个Thread对象，再调用Thread对象的start()启动
        new Thread(new MyThread(20),"a1").start();
        new Thread(new MyThread(20)).start();
        new Thread(new MyThread(20)).start();

//        new Thread(new MyRunnable(20)).start();
//        new Thread(new MyRunnable(20)).start();
//        new Thread(new MyRunnable(20)).start();
    }
```

**都可以用同一种方式（Thread 也是实现的Runnable接口），即调用构造方法返回一个Thread对象，再调用Thread对象的start()启动**



#### 3）java.util.concurrent.Callable接口

可以返回结果。

注意包路径不一样了，也是一个函数式的注解，调用是用call()，不是run(),并且有返回值了

```java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

这个Callable接口实现类，怎么来运行，或者说被调用呢，Thread类是无法直接调用的，它必须采用一个包装类FutureTask 将 Callable 对象包进去，而FutureTask类是实现了RunnableFuture接口的。



public class FutureTask<V> implements RunnableFuture<V>

``` 
public class FutureTask<V> implements RunnableFuture<V>

//它的构造方法，接收Callable对象
public FutureTask(Callable<V> callable) {
```

它是实现了RunnableFuture接口。RunnableFuture又是继承了Runnable接口

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    /**
     * Sets this Future to the result of its computation
     * unless it has been cancelled.
     */
    void run();
}
```

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

由于RunnableFuture接口继承了Runnable，所以，我们可以用Thread来启动

例子：

```java
public class MyCallable implements Callable<String> {
    private int ticket = 100;

    @Override
    public String call() throws Exception {
        for (int i = 0; i < 100 ; i++) {
            if (ticket > 0){
                System.out.println(Thread.currentThread().getName()+"-->卖票" + ticket);
                ticket -- ;
            }
        }
        return "票已卖光";
    }
}

class TestCallable{
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        MyCallable myCallable = new MyCallable();
        //必须用FutureTask来包装Callable对象，使其具有Runnable特性
        FutureTask<String> futureTask = new FutureTask<String>(myCallable);
        //具有Runnable特性的futureTask，就可以用Thread来启动了
        new Thread(futureTask).start();
        String taskResult = futureTask.get();
        System.out.println(taskResult);

    }
}
```













#### 4）Thread 和 Runnable 两种实现方式的区别

1）**Runnable接口 解决了  单继承的限制。**

2）Runnable 接口不能直接启动，必须用Thread类中的Start()方法启动，而继承Thread的实现类，本身就可以直接启动

他们的关系：

![](H:\99.JavaDoc\4. Java8 笔记\images\搜狗截图20181001121737.png)

客户端调用的是Thread类里的start()方法，来执行的，真正执行的又是从Thread的构造方法参数里的Runnable对象（MyThread implement Runnable）的run方法。

看起来，有点像代理模式（但，代理模式应该是执行run方法才对，这是因为Thread类是在设计模式出来之前，jdk1.0 就有了）

**3）Runnable能够更好的实现数据共享（供其他线程使用）**

```java
class MyThread extends Thread {
    private int ticket = 10; //票数
    @Override
    public void run() {
        while (ticket > 0){
            System.out.println(Thread.currentThread().getName()+"-->"+ticket);
            ticket --;
        }
    }
}

class test{
    public static void main(String[] args) {
        new Thread(new MyThread()).start();
        new Thread(new MyThread()).start();
        new Thread(new MyThread()).start();
    }
}
```

输出

Thread-1-->10
Thread-5-->10
Thread-3-->10
Thread-5-->9
Thread-1-->9
Thread-5-->8
Thread-3-->9

.....

可以看出，现在的情况，是票数10，是每个线程对象都在卖这10张票

![](H:\99.JavaDoc\4. Java8 笔记\images\搜狗截图20181001125948.png)

这是因为，我们没有用到共享对象来处理。

代码变形：

```java
class MyThread implements Runnable {
    private int ticket = 100; //票数
    @Override

    public void run() {
        while (ticket > 0){
            System.out.println(Thread.currentThread().getName()+"-->"+ticket);
            try {
                Thread.sleep(10);
                ticket --;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
    }
}

class test{
    public static void main(String[] args) {
        MyThread mt = new MyThread();
        //3个线程对象的Target Runnable对象，是同一个mt对象，就要用到数据共享了。
        new Thread(mt).start();
        new Thread(mt).start();
        new Thread(mt).start();
    }
}
```

mt对象，就是同一块数据资源。

![](H:\99.JavaDoc\4. Java8 笔记\images\搜狗截图20181001125543.png)



### 2.线程的命名与取得

1）通过构造方法

```
public Thread(Runnable target, String name)
```

2）通过setName方法

```
public final synchronized void setName(String name) 
```

3）通过getName获取线程的名字

```java
public final String getName()
//注意，这里获取当前线程，是用的native方法，而且是静态的，是类方法
public static native Thread currentThread()
```

```java
public class MyRunnable implements Runnable {
    private int count;
    public MyRunnable(int count){
        this.count = count;
    }
    @Override
    public void run() {
        while (count > 0){
            //获取线程名字
            System.out.println(Thread.currentThread().getName()+"-->"+count);
            count --;
        }
    }
}


class testRunnable {
    public static void main(String[] args) {
        MyThread mt = new MyThread();
        //设置线程名字
        new Thread(mt,"t1").start();
        new Thread(mt,"t2").start();
        new Thread(mt,"t3").start();
    }
}
```

下面例子说明，主方法也是一个线程，线程名字叫做main

```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}

class testRunnable {
    public static void main(String[] args) {
        MyRunnable mt = new MyRunnable();
        new Thread(mt, "t1").start(); //输出的线程名字为t1
        mt.run(); //输出的线程名字为 main
    }
}
```

**进程去哪儿了呢？**

我们运行程序是通过 java MyRunnable.class 文件。每运行一次，生成一个java的进程，而我们在控制台上看到的main，只是进程中的一个子线程，名字叫main（注意，不是main方法名哈）

**JVM启动时，要启动几个线程？**

一个main线程

一个gc线程，用于垃圾回收的线程

### 3.线程的休眠

休眠就是让线程执行变慢一点

Thread.Sleep()

### 4.线程的优先级

优先级越高，越可能先执行。这个的作用不大，执行效果不明显。

``` 
//设置优先级
public final void setPriority(int newPriority)
//获取优先级
public final int getPriority() {

//优先级是Thread中已经定义了的

    /**
     * 最低优先级
     */
    public final static int MIN_PRIORITY = 1;

   /**
     * 正常优先级
     */
    public final static int NORM_PRIORITY = 5;

    /**
     * 最高优先级
     */
    public final static int MAX_PRIORITY = 10;
```

### 5.线程同步问题

#### 1)发现问题

```java
public class MyRunnable implements Runnable {
    private int ticket = 5;
    @Override
    public void run() {
        for (int i = 0; i < 20 ; i++) {
            if (ticket > 0){
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+" 卖票--> "+ ticket--);
            }
        }
    }
}

class testRunnable {
    public static void main(String[] args) {
        MyRunnable mt = new MyRunnable();
        new Thread(mt, "t1").start();
        new Thread(mt, "t2").start();
        new Thread(mt, "t3").start();
        new Thread(mt, "t4").start();

    }
}
```

输出结果：

```java
t1 卖票--> 5
t2 卖票--> 4
t3 卖票--> 3
t4 卖票--> 2
t1 卖票--> 1
t3 卖票--> 0
t4 卖票--> -1
t2 卖票--> 0
```

出现了线程同步问题了。

#### 2)解决问题

使用同步方法synchronized，2种用法

#### 1.同步代码块

```java
public class MyRunnable implements Runnable {
    private  int ticket = 5;
    @Override
    public void run() {
        for (int i = 0; i < 20 ; i++) {
            synchronized (this){ //同步代码块
                if (ticket > 0){
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName()+" 卖票--> "+ ticket--);
                }
            }

        }
    }
}
```

同步代码块，看起来不优雅，我们可以把代码块抽取出来，在方法名上加上synchronized关键字，成为一个同步方法。

#### 2.同步方法

```java
public class MyRunnable implements Runnable {
    private  int ticket = 5;
    @Override
    public void run() {
        for (int i = 0; i < 20 ; i++) {
            sale();
        }
    }

    //这个就是同步方法
    private synchronized void sale() {
        if (ticket > 0){
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+" 卖票--> "+ ticket--);
        }
    }
}
```

#### 3.死锁

知道  过多的使用synchronized同步方法（代码块），可能会带来死循环，即死锁的情况，就OK了。

### 6.生产者和消费者

生产者和消费者是两个不同的线程，操作统一资源的情况。

操作模式：

-    生产者负责数据的生产，消费者负责消费生产出来的数据

-    生产者生产出来的一组数据后，消费者就要取走一组数据

现在有这样一个例子：

**前提：生产者和消费者之间没有用一个容器来过渡管理产品的生成时。**

生产者 和 消费者是2个线程，消费者要按顺序消费生产者生产出来的产品，但由于各自执行的速度不一致（即使一致，由于抢占资源的几率不是一样的），导致容易出现消费者多消费，或者少消费（这里消费者要慢一些，就会出现少消费）

**解决问题：**

为了让生产和消费的步调一致，就需要引入下图中的资源池来过渡。并且在资源池上有一个信号灯

**红灯时**，表示可以把生产出来的产品放到资源池里了，放入后，红灯变为绿灯。

**绿灯时**，表示资源池有产品了，通知消费者来消费，消费者取走后，绿灯变为红灯。

这样，我们就能让生产和消费保持步调一致了。

![](H:\99.JavaDoc\4. Java8 笔记\images\搜狗截图20181001164401.png)

#### Object的wait()和notify()实现线程间通讯

Object对象里有3个方法：

```java
class Object{

    //等待
    public final void wait() throws InterruptedException

    //通知 或者叫 唤醒第一个等待的线程，原生方法
    public final native void notify();
    //唤醒全部等待线程，那个优先级高，哪个就先执行    
    public final native void notifyAll();
    
}
```

下面的例子，还没有用上红绿灯机制。肯定会出现生产和消费不一致的情况

```java
//产品类
class Product {
    private String name;
    private String model;

    //同步方法，避免多线程给各个属性分开赋值时，容易错位
    public synchronized void setProduct(String name, String model) throws InterruptedException {
        Thread.sleep(10);
        this.name = name;
        this.model = model;
    }


    public synchronized Product getProduct() throws InterruptedException {
        Thread.sleep(10);
        return this;
    }

    @Override
    public String toString() {
        return "Product{" +
                "name='" + name + '\'' +
                ", model='" + model + '\'' +
                '}';
    }
}

//生产者--> 负责给product 赋值，类似于一个生产动作
class Productor implements Runnable {
    private Product product;

    public Productor(Product product) {
        this.product = product;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            try {
                if (i % 2 == 0) {
                    product.setProduct("changhong", "虹1号");
                } else {

                    product.setProduct("tcl", "t1号");
                }
            } catch (Exception e) {}

        }
    }
}

//消费者  负责消费，通过取值来模拟消费
class Customer implements Runnable {

    private Product product;

    public Customer(Product product) {
        this.product = product;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            try {
                System.out.println(product.getProduct().toString());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
}

//客户端
public class ProductorAndCustom {
    public static void main(String[] args) {
        Product product = new Product();
        Productor productor = new Productor(product);
        Customer customer = new Customer(product);
        new Thread(productor).start(); //开启生产者 生产
        new Thread(customer).start();  //开启消费者 消费

    }

}
```

输出

```java
Product{name='changhong', model='虹1号'}
Product{name='tcl', model='t1号'}
Product{name='changhong', model='虹1号'}
Product{name='tcl', model='t1号'}
Product{name='changhong', model='虹1号'}
Product{name='changhong', model='虹1号'}
Product{name='tcl', model='t1号'}
Product{name='changhong', model='虹1号'}
Product{name='changhong', model='虹1号'}
Product{name='changhong', model='虹1号'}
Product{name='tcl', model='t1号'}
```

可以看到不是生产1个，紧接着就消费一个。

要实现生产一个，消费一个，必须加上红绿灯条件来处理。



```java
//产品类
class Product {
    private String name;
    private String model;

    //是否为绿灯
    //绿灯时，可消费，不能生产，反之亦然
    private Boolean isGreenLight = false;

    //同步方法，避免多线程给各个属性分开赋值时，容易错位
    public synchronized void setProduct(String name, String model) throws InterruptedException {
        if (isGreenLight) {
            wait(); //如果为绿灯，不能生产，等待，线程挂起，当消费完成时，会通知直接执行后面的方法。
        }
        Thread.sleep(10);
        this.name = name;
        this.model = model;
        isGreenLight = true;
        super.notify();
    }


    public synchronized Product getProduct() throws InterruptedException {
        if (!isGreenLight) {
            super.wait();
        }
        Thread.sleep(10);
        isGreenLight = false;
        super.notify();
        return this;
    }

    @Override
    public String toString() {
        return "Product{" +
                "name='" + name + '\'' +
                ", model='" + model + '\'' +
                '}';
    }
}

//生产者--> 负责给product 赋值，类似于一个生产动作
class Productor implements Runnable {
    private Product product;

    public Productor(Product product) {
        this.product = product;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            try {

                if (i % 2 == 0) {
                    product.setProduct("changhong", "虹1号");
                } else {

                    product.setProduct("tcl", "t1号");
                }
            } catch (Exception e) {
            }

        }
    }
}

//消费者  负责消费，通过取值来模拟消费
class Customer implements Runnable {

    private Product product;

    public Customer(Product product) {
        this.product = product;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            try {
                System.out.println(product.getProduct().toString());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
}

//客户端
public class ProductorAndCustom {
    public static void main(String[] args) {
        Product product = new Product();
        Productor productor = new Productor(product);
        Customer customer = new Customer(product);
        new Thread(productor).start(); //开启生产者 生产
        new Thread(customer).start();  //开启消费者 消费

    }

}
```

上面的例子，用到了**Object.wait() 和 Object.nodify()**方法来实现 **线程间通讯**

输出结果：

```java
Product{name='changhong', model='虹1号'}
Product{name='tcl', model='t1号'}
Product{name='changhong', model='虹1号'}
Product{name='tcl', model='t1号'}
Product{name='changhong', model='虹1号'}
Product{name='tcl', model='t1号'}
```

按照顺序生产和消费了。

#### Thread.sleep()和Object.wait() 区别

2个方法是不同的类提供的

sleep，时间一到，自动唤醒

wait，需要等待notify 来唤醒

## 二、多线程高级篇

### 1.Executors框架

java.util.concurrent.Executors 框架里有很多线程池的处理

![](H:\99.JavaDoc\4. Java8 笔记\images\搜狗截图20181002181807.png)

```java
//Executors 工厂类下，调用的是ThreadPoolExecutor的构造器，来创建不一样的线程池
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
}
```









