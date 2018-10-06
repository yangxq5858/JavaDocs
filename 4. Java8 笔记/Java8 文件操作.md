# Java 文件操作 学习笔记

## 1.概述

在Java.IO 包中

五个核心类：File，InputStream, OutputStream, Reader,Writer

一个核心接口：Serializable

## 2.File类

File类只处理文件本身的操作（创建、删除等操作），**不涉及**到**文件的内容**。

要使用File类，首先要通过它的构造类，传入文件路径

``` java
public class File implements Serializable, Comparable<File>{
    //构造方法 全路径，这个时候，还没有执行创建，只是指定了一个文件路径而已
    public File(String pathname);
    //创建文件
    public boolean createNewFile() throws IOException;
    //获取当前文件的上级目录是否存在，注意，不是文件，而是目录
    public File getParentFile();
    //创建目录，根据file的路径创建,注意，这个只能创建一级目录
    public boolean mkdir();
    //***创建多级目录
    public boolean mkdirs();
    //获取文件的长度
    public long length()；
    //是否是文件
    public boolean isFile()；
    //是否为文件目录
    public boolean isDirectory()；
    //最后修改的日期 注意，这里是放回的long
    //Date构造方法接收long参数，new Date(long) 返回一个日期对象来处理
    public long lastModified()
    //列出目录下的所有文件名    
    public String[] list()；
    //列出目录下的所有文件
    public File[] listFiles()；
       
    
}
```

文件创建完成后，会出现延迟生成，是因为Java.io 是通过JVM调用底层系统的api来完成的

### 1）File.separator

避免不同的操作系统下的文件目录分隔符不一样

``` java
public static void main(String[] args) throws  Exception {
    //1. 设置文件的路径，现在文件是不存在的
    //windows和linux的路径分隔符是不一样的，用 File.separator 来代替（/或\)
    File file = new File("d:"+File.separator+"1.txt");
    if (file.exists()){
        file.delete();
    }
    file.createNewFile();
}
```

### 2）取得文件大小和最后修改日期

``` java
public static void main(String[] args) throws  Exception {
    File file = new File("d:"+File.separator+"1.jpg");
    if (file.exists()){
        long fileLong = file.length();
        double fileDouble = (double)fileLong / 1024 / 1024;
        //通过BigDecimal 对浮点型数据 精确转换为2位小数
        BigDecimal filedecimalecimal = new BigDecimal(fileDouble).divide(new BigDecimal(1), 2, BigDecimal.ROUND_UP);
        System.out.println(filedecimalecimal+" MB");
        //将long类型的日期，转换为格式化日期字符串
        long fileLastModifiedDate = file.lastModified();
        String sFileLastModifiedDate = new SimpleDateFormat("yyyy-MM-dd HH:dd:ss").format(new Date(fileLastModifiedDate));
        System.out.println(sFileLastModifiedDate);

    }
}
```

### 3）判断多级目录是否存在

```java
public static void main(String[] args) throws  Exception {
    File file = new File("d:"+File.separator+"aa"+File.separator+"fileDemo"+File.separator+"1.txt");

    //file.getParentFile() 这个返回的是一个目录 d:\aa\fileDemo
    if (!file.getParentFile().exists()){ //通过file.getParentFile() 这个是取得上级目录或文件是否存在
        file.getParentFile().mkdirs(); //这个命令可以创建多级目录
    }
    if (!file.exists()){ //如果文件不存在
        file.createNewFile();  //创建一个新的文件，但没有内容
    }

}
```

## 3.文件内容操作

操作文件只能通过**字节流和字符流**

### 1）操作步骤

要进行输入、输出操作，一般都会按照下面的步骤来进行（以文件内容操作为例）

- 通过File类定义一个要操作的文件路径（如果不是文件，就没有这一步）
- 通过字节流或字符流的**子类对象**为**父类**对象**实例化**
- 进行数据的读（输入）、写（输出）操作
- 数据流属于资源类，使用完毕后，要关闭

**流：**理解流的概念，流，就像输液的液体一样，它的输入是一个连续的过程。



### 2）字节流 JDK1.0（InputStream、OutputStream）

#### 1.OutputStream

```java
//java.io.OutputStream
//抽象类，不能直接使用，可以不用考虑接口Closeable, Flushable，因为这个类在JDK1.0就有了，
//而后面的接口都是JDK1.5才出来的，JDK 是想对接口进行细分而已。
public abstract class OutputStream implements Closeable, Flushable {
    //通过将任何缓冲的输出写入基础流来刷新此流。
    public void flush() throws IOException;
    public void close() throws IOException;
    //按单个字节输出,虽然是int型，但实际上是输出的byte类型
    public abstract void write(int b) throws IOException;
    //输出全部字节数组
    public void write(byte b[]) throws IOException; 
    //输出部分字节数组
    public void write(byte b[], int off, int len) throws IOException;
       
    
}
```

OutputStream的子类

- - All Implemented Interfaces:  

    [Closeable](../../java/io/Closeable.html)  ， [Flushable](../../java/io/Flushable.html) ， [AutoCloseable](../../java/lang/AutoCloseable.html) 

  - 已知直接子类：  

    [ByteArrayOutputStream](../../java/io/ByteArrayOutputStream.html) ， [FileOutputStream](../../java/io/FileOutputStream.html)  ， [FilterOutputStream](../../java/io/FilterOutputStream.html) ， [ObjectOutputStream](../../java/io/ObjectOutputStream.html) ， [OutputStream](../../org/omg/CORBA/portable/OutputStream.html) ， [PipedOutputStream](../../java/io/PipedOutputStream.html)  

FileOutputStream

```java
//创建或覆盖文件
public FileOutputStream(File file) throws FileNotFoundException
//是否采用追加模式
public FileOutputStream(File file, boolean append)
```

示例

```java
public static void main(String[] args) throws  Exception {
        //1. 实例化File对象
        File file = new File("d:"+File.separator+"aa"+File.separator+"filedemo"+File.separator+"1.txt");
        // 并创建目录，注意，这时，文件还没有被创建
        if (!file.getParentFile().exists()){ //通过file.getparentfile() 这个是取得上级目录或文件是否存在
            file.getParentFile().mkdirs(); //这个命令可以创建多级目录
        }
        //2. 用文件流子类FileOutputStream 实例化父类OutputStream
        OutputStream out = new FileOutputStream(file); //这个实例化流，会自动创建文件
        //3. 写入数据
        String content = "好好学习、天天向上、身体健康\r\n";
        byte[] bytes = content.getBytes(); //转为byte字节数组，才能输出
        //3.1 第一种方式，全部输出
        out.write(bytes);
        //3.2 第二种，按单个字节输出
//        for (int i = 0; i < bytes.length; i++) {
//            out.write(bytes[i]);
//        }
        //3.3 第三种，部分输出
//        out.write(bytes,0,6);

        //4. 关闭流
        out.close();
    }
```



#### 2.InputStream

public abstract int read() throws IOException; 读取单个的返回值的意义和后面2个不一样。

**返回值是int，是字节内容；未读取到数据时，返回-1。**

```java
public abstract class InputStream implements Closeable {
    //读取单个字节，返回值是int，是字节内容；未读取到数据时，返回-1。
    //这个方法，和后面的2个读取方式，很不一样哦。。。。。。。。。
    public abstract int read() throws IOException;
    
    //-------------------------------------------------
    //将读取的数据保存在字节数组里，返回读取的字节长度,读取到结尾时，返回-1
    public int read(byte b[]) throws IOException;
    //将读取的部分数据保存到字节数组里，返回读取到的部分数据的长度，读取到结尾时，返回-1
    //注意，这里inputStream在读取时，会自动偏移指针的，读完一个大小后，就会到下一个起点开始读取
    public int read(byte b[], int off, int len) throws IOException;
    //-------------------------------------------------
    
    public void close() throws IOException;
    public synchronized void mark(int readlimit);
    public synchronized void reset() throws IOException;
}
```

- - All Implemented Interfaces:  

    [Closeable](../../java/io/Closeable.html)  ， [AutoCloseable](../../java/lang/AutoCloseable.html) 

  - 已知直接子类：  

    [AudioInputStream](../../javax/sound/sampled/AudioInputStream.html) ， [ByteArrayInputStream](../../java/io/ByteArrayInputStream.html) ， [FileInputStream](../../java/io/FileInputStream.html) ，  [FilterInputStream](../../java/io/FilterInputStream.html) ， [InputStream](../../org/omg/CORBA/portable/InputStream.html) ， [ObjectInputStream](../../java/io/ObjectInputStream.html) ， [PipedInputStream](../../java/io/PipedInputStream.html)  ， [SequenceInputStream](../../java/io/SequenceInputStream.html) ， [StringBufferInputStream](../../java/io/StringBufferInputStream.html)  


```java
public class FileInputStream extends InputStream{
//构造方法
//通过打开与实际文件的连接创建一个 FileInputStream ，该文件由文件系统中的 File对象 file命名。
   public FileInputStream(File file);
//创建 FileInputStream通过使用文件描述符 fdObj ，其表示在文件系统中的现有连接到一个实际的文件。
   public FileInputStream(FileDescriptor fdObj) 
//通过打开与实际文件的连接来创建一个 FileInputStream ，该文件由文件系统中的路径名 name命名。  
   public FileInputStream(String name) 

} 
```

示例代码：

一次性读取到字节数组里

```java
public static void main(String[] args) throws  Exception {
    //1. 实例化File对象
    File file = new File("d:"+File.separator+"aa"+File.separator+"filedemo"+File.separator+"1.txt");
    if (file.exists()){
        //2. 读取数据
        InputStream input = new FileInputStream(file);
        byte data[] = new byte[1024];
        int len = input.read(data); //将数据保存到data中，并返回实际读取到的长度
        //3. 关闭
        input.close();
        //要显示字符串，new String(data),设置内容的长度，否则会在后面补齐1024的空白内容
        String content = new String(data,0,len);
        System.out.println("["+content+"]");
    }

}
```

读取单个字节的方式
```java
public static void main(String[] args) throws  Exception {
    //1. 实例化File对象
    File file = new File("d:"+File.separator+"aa"+File.separator+"filedemo"+File.separator+"1.txt");
    if (file.exists()){
        //2. 读取数据
        InputStream input = new FileInputStream(file);
        byte data[] = new byte[1024];
        int content = 0;
        int idx = 0; //数组的脚标

        //表示读取单个字节 content = input.read()
        while ((content = input.read()) != -1) {
            data[idx++] = (byte)content; //将读取到的内容int型，转化为字节
        }
        //3. 关闭
        input.close();
        //要显示字符串，new String(data),设置内容的长度，否则会在后面补齐1024的空白内容
        String contents = new String(data,0,idx);
        System.out.println("["+contents+"]");
    }

}
```



### 3）字符流 JDK1.1 (Reader、Writer)

这个类最大的不同就是，是按照字符输出，还有一个最大的不同，就是可以**输出字符串**，比字节流更加强大了



#### 1.Writer

```java
public abstract class Writer implements Appendable, Closeable, Flushable {
    public void write(String str)  
    public void write(String str, int off, int len)
    public void write(char[] cbuf)
    //将制定的字符序列写入
    public Writer append(CharSequence csq);
    public Writer append(char c)
    public abstract void flush()  
    public void write(int c)
    
}
```



- - All Implemented Interfaces:  

    [Closeable](../../java/io/Closeable.html)  ， [Flushable](../../java/io/Flushable.html) ， [Appendable](../../java/lang/Appendable.html) ， [AutoCloseable](../../java/lang/AutoCloseable.html) 

  - 已知直接子类：  

    [BufferedWriter](../../java/io/BufferedWriter.html) ， [CharArrayWriter](../../java/io/CharArrayWriter.html) ，  [FilterWriter](../../java/io/FilterWriter.html) ， [OutputStreamWriter](../../java/io/OutputStreamWriter.html) ， [PipedWriter](../../java/io/PipedWriter.html) ， [PrintWriter](../../java/io/PrintWriter.html) ， [StringWriter](../../java/io/StringWriter.html)  


示例：输出字符串

```java
public static void main(String[] args) throws  Exception {
    //1. 实例化File对象
    File file = new File("d:"+File.separator+"aa"+File.separator+"filedemo"+File.separator+"1.txt");
    //判断路径是否存在,创建目录
    if (!file.getParentFile().exists()){
        file.getParentFile().mkdirs();
    }
    //2.实例化Writer
    Writer writer = new FileWriter(file);
    //3.写入
    writer.write("你好，在吗？我有点开始想念你了哦");
    //关闭流
    writer.close();

}
```

#### 2.Reader

Reader 与 Writer 有一最大的不同，就是没有读取整个字符串的功能，因为我们不知道，这个读取的内容是否非常大，如果非常大，就有可能导致读取耗费很长时间，导致系统卡死。

```java
public abstract class Reader implements Readable, Closeable {
    //部分读取
    public abstract int read(char[] cbuf, int off, int len);
    //全部读取，返回读取到的字符的长度，读取到末尾时，返回-1,数据保存在字符数组中
    public int read(char[] cbuf); 
    public int read()；//返回单个字符，如果到了末尾返回-1
    
}
```

- - All Implemented Interfaces:  

    [Closeable](../../java/io/Closeable.html)  ， [AutoCloseable](../../java/lang/AutoCloseable.html) ， [Readable](../../java/lang/Readable.html)  

  - 已知直接子类：  

    [BufferedReader](../../java/io/BufferedReader.html) ， [CharArrayReader](../../java/io/CharArrayReader.html) ，  [FilterReader](../../java/io/FilterReader.html) ， [InputStreamReader](../../java/io/InputStreamReader.html) ， [PipedReader](../../java/io/PipedReader.html) ， [StringReader](../../java/io/StringReader.html) 

```java
public static void main(String[] args) throws  Exception {
    //1. 实例化File对象
    File file = new File("d:"+File.separator+"aa"+File.separator+"filedemo"+File.separator+"1.txt");
    if (file.exists()){
        Reader reader = new FileReader(file);
        char data[] = new char[1024];
        reader.read(data);
        reader.close();
        System.out.println(new String(data));
    }
}
```

## 4.字节流和字符流的区别

字节流：直接与终端进行操作

字符流：需要经过数据缓冲区处理后， 才可以输出。

示例：不关闭流，数据就还在缓冲区中，我们看1.txt的内容就是空的

```java
public static void main(String[] args) throws  Exception {
        //1. 实例化File对象
        File file = new File("d:"+File.separator+"aa"+File.separator+"filedemo"+File.separator+"1.txt");
        //判断路径是否存在,创建目录
        if (!file.getParentFile().exists()){
            file.getParentFile().mkdirs();
        }
        //2.实例化Writer
        Writer writer = new FileWriter(file);
        //3.写入
        writer.write("你好，在吗？我有点开始想念你了哦");
        //关闭流
//        writer.close();  //如果不关闭流，文件内容是不会写入的，数据还在缓冲区中
    }
```

### flush()

如果，我们不想关闭流，又想输出内容，就需要flush()函数。

flush()作用：将缓存区里的数据强制写入到流中，并清空缓存区。

### 总结

字节流的应用场景：图片，音乐，视频的处理

字符流的应用场景：有**中文需要处理**的字符，优先考虑，用字节流处理，容易导致读取数据异常。

## 5.转换流

因为字符流可以对字符串输出，而字节流不能。故而，需要流的转换，主要是为了处理**中文**

InputStreamReader 和 OutputStreamWriter 两个类来处理。

一般都不用它。



| 名称   | InputStreamReader                        | OutputStreamWriter                       |
| ---- | ---------------------------------------- | ---------------------------------------- |
| 定义结构 | public class InputStreamReader extends Reader | public class OutputStreamWriter extends Writer |
| 构造方法 | public  `InputStreamReader(InputStream in)` | public`OutputStreamWriter(OutputStream out)` |
|      |                                          |                                          |

通过构造方法，可以看出就将InputStream 和 Reader 联系起来了，故而可以进行转换了。



```java
public static void main(String[] args) throws  Exception {
    //1. 实例化File对象
    File file = new File("d:"+File.separator+"aa"+File.separator+"filedemo"+File.separator+"1.txt");
    //判断路径是否存在,创建目录
    if (!file.getParentFile().exists()){
        file.getParentFile().mkdirs();
    }
    OutputStream out = new FileOutputStream(file);
    //这里可以认为就是一个包裹对象
    OutputStreamWriter writer = new OutputStreamWriter(out);
    //下面就用Writer类的对象来输出了，就有了缓冲区概念
    writer.write("我是用的OutputStreamWriter转换流来操作的数据哦");
    writer.flush(); //或者用writer.close() 也行，也可以都用上
    writer.close();
}
```



## 6.文件Copy案例

方法一：一次性读取，一次性写入，需要开辟一个与源文件大小一样的空间来缓冲。但如果文件过大呢，就会出现卡死，而且一次性读取的时间，较长的话，也会影响速度。

方法二：一边读取，一边写入，这样我们需要的缓冲空间就不用太大。

```java
public static void main(String[] args) throws Exception {
        //定义输入、输出文件的路径
        File inputFile = new File("d:"+File.separator+"1.jpg");
        File outputFile = new File("d:"+File.separator+"2.jpg");
        if (!inputFile.exists()) return;
        long start = System.currentTimeMillis();

        //判断目标文件的目录是否存在
        if (!outputFile.getParentFile().exists()) outputFile.getParentFile().mkdirs();
        //定义输入流和输出流
        InputStream inputStream = new FileInputStream(inputFile);
        OutputStream outputStream = new FileOutputStream(outputFile);
        //采用 边读取，边写入的方式
        //采用单个字节读取的方式 开始读取
        int byteLen = 0;
        byte data[] = new byte[1024];
        //注意，这里的read(byte[]) 是每次读取1024字节的数据到data缓冲区，返回实际读取到的长度
        //注意，这里inputStream在读取时，会自动偏移指针的，读完一个1024，就会到下一个起点开始读取
        while((byteLen = inputStream.read(data))!=-1){
            System.out.println(byteLen); //返回1024,1024... 578(最后读取到的值）
            outputStream.write(data,0,byteLen);
        }

         //单个字节读取的方式
//        int tempByte = 0;
//        while((tempByte = inputStream.read())!=-1){
//            outputStream.write(tempByte);
//        }
        inputStream.close();
        outputStream.close();

        long end = System.currentTimeMillis();
        System.out.println("花费时间"+((end - start)/1000) + "s");
    }
```



## 7.字符编码

GBK,GB2312: 包含中文简体和中文繁体

ISO8859-1: 国际编码，可以表示任何国家的文字信息

UNICODE: 是十六进制编码，如果我们的文字信息中，英文字符很多，只有少数中文，就会导致编码很大，造成无用的空间占用与传输。

UTF-8:融合了ISO8859-1 和 UNICODE 的优点。

**以后，我们只会用到UTF-8**

示例：将ISO8859-6编码的String，转为UTF-8的字节输出

```java
 byte[] data = new byte[1024];
 while ((temp2=input2.read(data))!=-1){
            byte[] bytes = new String(data).getBytes("ISO8859-6");
            byte[] bytes1 = new String(bytes).getBytes("UTF-8");
   //这里就转码了
 }

```

```
new String(byte[],"UTF-8"); 也可以的

因为utf8可以用来表示/编码所有字符，所以new String( str.getBytes( "utf8" ), "utf8" ) === str，即完全可逆。 
```

## 8.内存流

### 1）概述

在ajax ，xml 对json的处理，会用到。

希望对IO有操作，但又不想生成文件，就需要用到内存流。

java.io 包里面提供了两组操作：

字节内存流：ByteArrayInputStream、ByteArrayOutputStream

字符内存流：CharArrayReader、CharArrayWriter。

| 名称   | ByteArrayInputStream                    | ByteArrayOutputStream           |
| ---- | --------------------------------------- | ------------------------------- |
| 继承结构 | java.lang.Object                        | java.lang.Object                |
|      | --java.io.InputStream                   | --java.io.OutputStream          |
|      | --java.io.ByteArrayInputStream          | --java.io.ByteArrayOutputStream |
| 构造方法 | public ByteArrayInputStream(byte[] buf) | public ByteArrayOutputStream()  |
|      | **输入流，可以将程序中的字节数组，传入到内存中**              | 输出流：没有构造参数                      |



### 2）操作流程（与文件操作是反的）

内存的操作（输入和输出）与文件的操作（输入和输出）的顺序是反的。**

以文件操作为例：**（以程序为参照物）**

输入：文件--->Inputstream--->程序

输出：程序--->OutputStream-->文件

内存的操作：**（以程序为参照物）**

输入：内存--->OutputStream-->程序

输出：程序--->InputStream-->内存



示例：将一个字符串转为大写，同时排除掉不需要，或不能转的字符，需要用到一个包装类Character类

```java
public final class Character implements java.io.Serializable, Comparable<Character>{
    public static int toUpperCase(int codePoint)；
    public static char toUpperCase(char ch)；
    public static char toLowerCase(char ch)；
    public static int toLowerCase(int codePoint)；
}
```



例子：

```java
public static void main(String[] args) throws IOException {
    String str = "Hello***World!!!";

    //将str的byte[] 保存到内存输入流中（从程序-->内存表示输出，使用的InputStream）
    InputStream input = new ByteArrayInputStream(str.getBytes());

    //将内存流中的数据取出（）
    OutputStream output = new ByteArrayOutputStream();
    int temp = 0;
    while((temp = input.read()) != -1){ //读取每一个数据
        output.write(Character.toUpperCase(temp)); //将每一个字节输出到内存
    }
    input.close();
    output.close();
    System.out.println(output);//HELLO***WORLD!!!
}
```



### 3）ByteArrayOutputStream



```java
//从内存输出到程序的流
//可以实现多个文件的同时读取
public class ByteArrayOutputStream extends OutputStream{
    //有一个自己的方法
    //转换为byte[]， 这样程序就可以得到这个字节流，我们就可以用字节流类，来处理其他事情了
    public synchronized byte toByteArray()[] 

}
```

toByteArray 这个方法，只有ByteArrayOutputSteam类才有。它可以实现多个文件的内容读取在一起。

```java
public static void main(String[] args) throws IOException {
        File file1 = new File("d:"+File.separator+"1.txt");
        File file2 = new File("d:"+File.separator+"2.txt");
        InputStream input1 = new FileInputStream(file1);
        InputStream input2 = new FileInputStream(file2);
        ByteArrayOutputStream output = new ByteArrayOutputStream();
        int temp1 = 0;
        int temp2 = 0;
        while ((temp1=input1.read())!=-1){
            output.write(temp1);  //将文件1的内容写入到 内存输出流
        }
        while ((temp2=input2.read())!=-1){
            output.write(temp2); //将文件2的内容写入到 内存输出流，相当于2个文件的内容进行追加了
        }
        byte[] bytes = output.toByteArray(); //将ByteArray 转换为byte[] 字节流
        output.close();
        input1.close();
        input2.close();

        System.out.println(new String(bytes));

    }
```

## 9.System对IO的操作（PrintStream）

在System类里面提供了三个静态方法

- 错误输出：public static final **PrintStream** err(); //基本不用

- 输出到标准输出设备（显示器）：public static final **PrintStream** out();

- 从标准输入设备读取（键盘）：public static final InputStream in();

### System.out

```java
public static void main(String[] args) throws IOException {
    OutputStream output = System.out;
    String str = "Hello world";
    byte[] bytes = str.getBytes();
    output.write(bytes);
}
```

### System.in

示例：获取键盘的输入数据，由于返回是InputStream，我们就可以用InputStream的read方法

```java
public static void main(String[] args) throws IOException {
    InputStream input = System.in;
    System.out.println("请输入数据：");
    byte[] data = new byte[1024];
    input.read(data);
    System.out.println("输入的数据为："+new String(data));

}
```

## 10.缓冲输入流（buffered）

一个工具类，主要用于**解决中文乱码**的问题。

字符缓冲区流：**BufferedReader**、BufferedWriter；

字节缓冲区流：BufferedInputStream、BufferedOutputStream；

最重要的是**BufferedReader**。因为它有一个方法可以读取一个**字符串**

```java
//读取一行数据，以换行分隔符为界
public String readLine() throws IOException
```

```java
//继承关系
|-java.lang.Object 
   |-java.io.Reader 
     |-java.io.BufferedReader 

//构造方法，接收字符流输入
public BufferedReader(Reader in) 
```

示例：

实现键盘输入，并用字符串的方式读取

```java
public static void main(String[] args) throws IOException {
    //System.in 是 inputStream类的对象
    //BufferedReader的构造方法，需要接收Reader类对象
    //故，需要一个InputStreamReader类把InputStream包裹起来，这样就实现了字节流向字符流的转换
    BufferedReader buf = new BufferedReader(new InputStreamReader(System.in));
    System.out.println("请输入数据");
    String readLine = buf.readLine();
    System.out.println("输入的内容为："+readLine);
}
```



有了从字节流-->字符流-->字符串的转换，我们就可以使用String类的正则表达式来做一些验证了。

其实String类同样支持 字节流--->字符串的转换

String str = new String(byte[]);



针对文本文件的读取，因为有了字符串的读取功能，就更加方便了。

```java
public static void main(String[] args) throws IOException {
    File file = new File("d:"+File.separator+"1.txt");
    BufferedReader buf = new BufferedReader(new FileReader(file));
    String data = null;
    StringBuffer strbuf = new StringBuffer();
    while ((data = buf.readLine())!=null){ //注意，这里判断是否到结尾，是用的null，不是-1
        strbuf.append(data+"\r\n");
    }
    System.out.println(strbuf);
    buf.close();
}
```

## 11.序列化

所谓的序列化，就是讲对象转换为二进制数据流进行传输，但并不是所有的类都可以进行序列化，如果要序列化，就要在类上实现java.io.Serializable接口。



```java
@SuppressWarnings("Serial") //这个一定要加上，否则反序列化，报错
class Book implements Serializable{
    private String title;
    private int age;

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

### 序列化类：

```java
//继承关系
  java.lang.Object 
    java.io.OutputStream 
      java.io.ObjectOutputStream 
 public Class ObjectOutputStream{
     //构造方法
     ObjectOutputStream(OutputStream out);       
     //序列化方法
     writeObject(Object obj)；
 }


```

```java
    public static void main(String[] args) throws IOException {
        File file = new File("d:"+File.separator+"book.txt");
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
        oos.writeObject(new Book("java开发",25));
        oos.close();
    }
```



### 反序列化类：

```java
java.io 
Class ObjectInputStream
  java.lang.Object 
    java.io.InputStream 
      java.io.ObjectInputStream 
```

```java
private static void des() throws IOException, ClassNotFoundException {
        File file = new File("d:"+File.separator+"book.txt");
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
        Object object = ois.readObject();
        Book book = (Book)object;
        System.out.println(book);
 }
```

### transient

```
transient 关键字：表示不序列化字段
private transient String title; //transient 表示不序列化
```


## 12.Collections 工具类（基本不用）

这个工具类可以对List、Set、Map进行操作。

- [java.lang.Object](../../java/lang/Object.html) 

- - java.util.Collections 


```java
public class Collections{
   //一次性添加多个元素
   addAll(Collection<? super T> c, T... elements)   
   //反转List中的数据
   reverse(List<?> list)；
}

```

示例：

```java
public static void main(String[] args) {
    List<String> list = new ArrayList<>();
    Collections.addAll(list,"a","b","C");
    System.out.println(list);
}
```

## 13.StringBuffer、StringBuilder、String的区别

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























