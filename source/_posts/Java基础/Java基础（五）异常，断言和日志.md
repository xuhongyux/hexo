---
title: Java基础（五）异常，断言和日志
date: 2019-10-10 15:04:20
tags: 
	- 异常
	- 日志
	- 断言
	- 调试技巧
	- Java基础
categories: Java
---

## 异常

异常示意图

所有的异常都是由Throwable继承下来的，但是在下一层理解分解为两个Error和Exception

![IMG_5240(20191009-142902)](Java基础（五）异常，断言和日志/IMG_5240(20191009-142902).jpg)

![1570603295876](Java基础（五）异常，断言和日志/1570603295876.png)

![1570607831039](Java基础（五）异常，断言和日志/1570607831039.png)

### 声明受查异常并抛出异常

如果无法处理的异常，Java可以抛出一个异常。

抛出一个内存溢出的异常

```java
public FileInputStream(String name) throws FileNotFoundException 
```

__声明时需要注意__

+ 需要抛出异常的情形

> 1. 调用了一个抛出异常的方法
> 2. 程序运行过程中发现错误
> 3. 程序出现错误
> 4. Java虚拟机和运行时库出现的内部错误
>
> 前两种错误必须需要抛出，如果不抛出，则会出现死亡陷阱

+ 我们这里主要是将错误抛出并不处理错误，其中的根源是错误的根源，在我们能够排除错误的时候我们尽量优先自己处理错误，尽量不要将错误抛给别人。
+ 当然我们也可以自己捕获异常，自己来处理异常

在需要抛出多个异常的时候，异常之间用 ， 隔开。

__如何抛出异常__

我们要依据可能出现的异常的类型来抛出异常

比如在处理文件的时候出现文件大小和描述不同的异常

```java
throw new EoFEception();
//或者
EOFECEprion e = new EoFEception();
throw e;
public FileInputStream(String name) throws e{
    
}
//还可以携带自己对异常的描述
String gripe = "描述的内容"；
throw new EoFEception(gripe);
```

### 创建异常类

在实际的编程过程中，我们由很多异常都没办法被充分的描述清楚。这个时候我们就需要自己来定义个异常类了。

我们只需要定义一个派生于Exception的类，或者派生与Exception的子类。

__Throwable 1.0__

+ Throwable()    构造一个新的Throwable对象，这个第项没有详细的描述信息
+ Throwable(String  message)  构造一个新的throwable对象，这个对象带有特定的详细描述信息。
+ String getMessage()   获得Throwable 对象的详细描述信息 

### 捕获异常

异常能自己处理的就自己处理尽量不抛出去，本着这个原则，我们就需要自己能够捕获到异常。

语法

```java
try{
    代码块
}catch(Exception e){//可以有多个 这样可以拦截多个不同的异常
    异常处理
}finally{//可选 一定会运行
    //注意   在使用finally的时候 假如 我们有return语句 在rty模块中出现了这个语句，在finallly也出现这个语句 那么 finally将会把前面的try模块的return值
}

//
e.getMessage()   得到详细的错误信息
e.getClass().getName()   得到异常对象的世界类型

```

#### 异常的二次处理

有时候我们仅仅是将异常的信息拦截下来，并且再次抛出。

```java
try{
    
}catch (exception e){
    throw new exception(e.getMessage());
}
```

#### 带资源的try

当try模块中带有资源的访问,当try推出的时候，就会自动调用 str.close(),就好像使用了finally模块一样都会调用。

#### 分析堆栈轨迹元素

堆栈轨迹是一个方法调用过程的列表，他包含了程序执行过程中方法调用的特定位置

```java
public class StackTraceTest
{
    /**
     * Computes the factorial of a number
     * @param n a non-negative integer
     * @return n! = 1 * 2 * . . . * n
     */
    public static int factorial(int n)
    {
        System.out.println("factorial(" + n + "):");
        Throwable t = new Throwable();
        //获取堆栈的轨迹信息
        StackTraceElement[] frames = t.getStackTrace();
        for (StackTraceElement f : frames)
            System.out.println(f);
        int r;
        if (n <= 1) r = 1;
        else r = n * factorial(n - 1);
        System.out.println("return " + r);
        return r;
    }

    public static void main(String[] args)
    {
        Scanner in = new Scanner(System.in);
        System.out.print("Enter n: ");
        int n = in.nextInt();
        factorial(n);
    }
}

```

getStackTrace()方法来获取堆栈的信息

__java.lang.Throwable 1.0__

+ Throwable (Throwable cause)
+ Throwable(String message, Throwable cause)   用给定的“原因”构造一个Throwable对象
+ Throwable initCause(Throwable cause) 将这个对象设置为“原因”，如果这个对象已经被设置为“原因”，则抛出一个异常，返回this引用。

### 异常机制的使用技巧

#### 异常处理不能代替简单的测试   

只在异常的情况下使用异常机制。

#### 不要过分的细化异常

#### 利用异常层次结构

不要只抛出RuntimeException异常，应该寻找更加适当的子类或者创建自己的异常类

不要只捕获Throwble异常，否则程序代码更加的难读

#### 不要压制异常

#### 在检测错误时，“苛刻”要比放任更好

## 断言

断言机制允许在测试期间向代码中插入一些检测语句，当代码发布的时候，这个插入的检测代码将会被自动移除。

### 断言的语法

Java语言引入关键字assert,这个关键字有两种形式：

+ assert 条件；

+ assert 条件：表达式；

这两种形式都会对条件进行检查，如果结构为false则抛出一个AssertionError的异常。第二种形式中，表达式将被传入AssertionError的构造器，并转成一个小学字符串

例如： 将x的值传递给AssertionError对象，从而以后可以在后面显示出来；

```java
assert x >=0 : x ;
```

启用断言

java -enableassertions Myapp

禁用断言

-enableassertions 或者 -da

在启用或者禁用的时候不用重新编译

### 断言的参数

在Java中我们有三种方式错误处理机制：

+ 抛出一个异常
+ 日志
+ 使用断言

什么时候使用断言

+ 断言的失败是致命的，不可恢复的
+ 断言检测只适用与开发和测试阶段

__java.lang.ClassLoader 1.0__

+ void setDefaultAssertionStatus(boolean b) 对于通过类加载器加载所有的类来说，如果没有明显的说明，就用来启用或者禁用断言
+ void setClassAssertionStatus(String className , boolean b) 对于给定的类和他的内部类，启用或者禁用断言
+ void setPackageAssertionStatus(String className , boolean b) 对于给定的包和他的内部类，启用或者禁用断言
+ void clearAssertionStatus() 移除所有的类和包，的显示断言状态设置，并禁用所有听过这个类加载器加载的类的断言

## 日志

日志就是用来代替，System.out.println，并对之进行规划整理

__优点__

+ 可以轻易的取消全部的日志输出记录，或者仅仅取消莫格日志的级别

+ 日志记录可以被定向到不同的处理器，用于显示或者存储

+ 日志方便过滤，和格式控制

+ 日志配置比较容易更换

  

生成简单的日志记录，使用全局日志记录器（global logger）调用器info方法：

```java  
Logger.getGlobal().info("File ->显示的信息")；
```

在适当的地方（比如main开始）调用

```java
Logger.getGlobal().setLevel(Level.OFF);   //取消所有日志
```

```java
public class Hello {
    public static void main(String[] args) {
        Logger logger = Logger.getGlobal();
        logger.info("start process...");
        logger.warning("memory is running out...");
        logger.fine("ignored.");
        logger.severe("process will be terminated...");
    }
}
```

输出

```java
Mar 02, 2019 6:32:13 PM Hello main
INFO: start process...
Mar 02, 2019 6:32:13 PM Hello main
WARNING: memory is running out...
Mar 02, 2019 6:32:13 PM Hello main
SEVERE: process will be terminated...
```

使用日志最大的好处是，它自动打印了时间、调用类、调用方法等很多有用的信息。

再仔细观察发现，4条日志，只打印了3条，`logger.fine()`没有打印。这是因为，日志的输出可以设定级别。JDK的Logging定义了7个日志级别，从严重到普通：

- SEVERE
- WARNING
- INFO
- CONFIG
- FINE
- FINER
- FINEST

因为默认级别是INFO，因此，INFO级别以下的日志，不会被打印出来。使用日志级别的好处在于，调整级别，就可以屏蔽掉很多调试相关的日志输出.

在不同日志框架下面不同的配置方法。

这里就不过多的叙述。在Java核心技术10 卷1      289P

## 调试技巧

+ 可以用下面的方法打印出你想测试的任何信息

```java
System.out.println("x = "+ x);
//或者
Logger.getGlobal().info("x = "+ x)
//大多数的Java类库都覆盖了toString方法，
```

+ 在每个类中单独放一个min方法可以进行单元测试
+ 在[Junit5](http://junit.org)中可以看相关的文档
+ 日志代理是一个子类的对象，他可以截获方法的调用，并进行日志记录，然后调用超类的对象。

如果在调用Random类的nextDouble方法时出现了问题，就可以按照下面的的方式，一匿名子类的实例的形式创建一个代理对象

```JAVA
Random generator = new Random()
{
    double result =super.nextDouble();
    Logger.getGlobal().info("nextDouble"+ result);
    return rusult;
}
```

当调用nextDouble方法时，就会产生一个日志消息，要想知道，谁调用了这个方法，就要生成一个堆栈轨迹。

+ 利用Throwable类提供一个printStackTrace方法，可以从任何一个移除对象中获取堆栈的情况

```java
try{
    
}catch (Throwable t){
    t.prinStacTrace();
    throw t;
}
```

不一定需要通过捕获异常来生成堆栈记录，在代码的任何位置插入下面的这条语句就可以获得堆栈轨迹；

