---
title: Java技术相关
date: 2020-6-12 08:01:42
tags:
	
categories: Java 集训
---

## 知识点

### 多态

多态： 一种类的多种形式（同一个行为具有多个不同表现形式或形态的能力）

多态分为：编译时多态，运行时多塔

编译时多态：同名方法不同的参数。

运行时多态：在Java中，每个对象的状态是多态的，每个子类的对象同时也是超类的对象。

多态存在的三个必要条件

- 继承
- 重写
- 父类引用指向子类对象

### 抽象类和接口

抽象类是用来捕捉子类的通用特性的。接口是抽象方法的集合。

从设计层面来说，抽象类是对类的抽象，是一种模板设计，接口是行为的抽象，是一种行为的规范。

**相同点**

- 接口和抽象类都不能实例化
- 都位于继承的顶端，用于被其他实现或继承
- 都包含抽象方法，其子类都必须覆写这些抽象方法

**不同点**

| 参数       | 抽象类                                                       | 接口                                                         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 声明       | 抽象类使用abstract关键字声明                                 | 接口使用interface关键字声明                                  |
| 实现       | 子类使用extends关键字来继承抽象类。如果子类不是抽象类的话，它需要提供抽象类中所有声明的方法的实现 | 子类使用implements关键字来实现接口。它需要提供接口中所有声明的方法的实现 |
| 构造器     | 抽象类可以有构造器                                           | 接口不能有构造器                                             |
| 访问修饰符 | 抽象类中的方法可以是任意访问修饰符                           | 接口方法默认修饰符是public。并且不允许定义为 private 或者 protected |
| 多继承     | 一个类最多只能继承一个抽象类                                 | 一个类可以实现多个接口                                       |
| 字段声明   | 抽象类的字段声明可以是任意的                                 | 接口的字段默认都是 static 和 final 的                        |

###  可选操作

​	简单说就是抽象类的的某些派生copy类实现里，或者接口的某个实现类里面，某个方法可能是无意义的，调用该百方法会抛出一个异常。例如在collection的某些实现度类里，里面的元素可能都是只读的，那么add这个接口是无意义的，调用会抛出UnspportedOperationException异常知。
从设计的角度说，如果一个接口的方法设计为optional，表示这个方法不是为所有的实道现而设定的，而只是为某一类的实现而设定的。

​	定义可选操作的原因：“这样做可以防止在设计中出现接口爆炸的情况”。通过抽象类来推迟某些方法的实现，直到需要实现的时候。

### 基本类型与包装类型

一个是基本类型（primitive），如int、double等八种基本数据类型；

另一个是引用类型（reference type），如String、List等。而每一个基本类型又各自对应了一个引用类型，称为包装类型（或装箱类型，boxed primitive）。

*基本类型与包装类型的主要区别在于以下三个方面：*

+ 基本类型只有值，而包装类型则具有与它们的值不同的同一性（identity）。这个同一性是指，两个引用是否指向同一个对象，如果指向同一个对象，则说明具有同一性。（与此类似的还有等同性。）
+ 基本类型只有功能完备的值，而包装类型除了其对应的基本类型所有的功能之外，还有一个非功能值：null。
+ 本类型通常比包装类型更节省时间与空间。

```java
 Integer g = new Integer(0);
        Integer h = new Integer(0);
        System.out.println(g==h);//false
        //以上代码，两个对象虽然具有相同的值，但其引用指向的对象却不相同，因此会输出false。
       // 再来看一段代码：
        Integer a = 0;
        Integer b = 0;
        System.out.println(a==b);//true
        //当我们直接给一个Integer赋予一个int值的时候，它会调用一个valueOf()的方法  直接装箱
        Integer d = 128;
        Integer f = 128;
        System.out.println(f==d);//false
       // Integer的常量池是由-128至127组成。当我们给一个Integer赋的值在这个范围之类时就直接会从缓存返回一个相同的引用，所以a==b会输出true。而超过这个范围时，就会重新new一个对象。因此，f==d就会输出一个false。
    
```



总结

+ 基本类型是要优先于包装类型。基本类型更加简单、更加快速。

+ 一下情况包装类型的使用会更合理：

+ > 1、作为集合中的元素、键和值。
  > 2、在参数化类型中。比如：你不能这样写——ArryList<int>,你只能写ArrayList<Integer>.
  > 3、在进行反射方法的调用时。
  > 总之，当可以选择时候，基本类型是要优先于包装类型。基本类型更加简单、更加快速。

### String  Stringbuffer  StringBuilder

#### String 

String类中使用字符数组保存字符串，如下就是，因为有“final”修饰符，所以可以知道string对象是不可变的。

```java
　　　　private final char value[];
```

String的值是不可变的，这就导致每次对String的操作都会生成新的String对象，不仅效率低下，而且大量浪费有限的内存空间。

#### StringBuilder与StringBuffer

__源码__

构建StringBuilder

```java
//构建StringBuiler对象
public StringBuilder() {
    //继承父类
        super(16);
    }
//初始化一个数组大小为16
AbstractStringBuilder(int capacity) {
        value = new char[capacity];
    }
//放入元素
public StringBuilder append(int i) {
        super.append(i);
        return this;
    }
//父类添加元素
public AbstractStringBuilder append(String str) {
    //校验添加的元素是否为空    
    if (str == null)
            return appendNull();
    //求出需添加元素的长度
        int len = str.length();
    //校验长度（下述）
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
//校验长度
private void ensureCapacityInternal(int minimumCapacity) {
        // overflow-conscious code
    //计算长度在添加后长度是否超出
        if (minimumCapacity - value.length > 0) {
            //若超出 创建新容量的数组 并拷贝到新的数组上 
            value = Arrays.copyOf(value,newCapacity(minimumCapacity));
        }
             
    }
//数组扩容 二倍+2
private int newCapacity(int minCapacity) {
        // overflow-conscious code
        int newCapacity = (value.length << 1) + 2;
        if (newCapacity - minCapacity < 0) {
            newCapacity = minCapacity;
        }
        return (newCapacity <= 0 || MAX_ARRAY_SIZE - newCapacity < 0)
            ? hugeCapacity(minCapacity)
            : newCapacity;
    }
//数组复制
public static char[] copyOf(char[] original, int newLength) {
        char[] copy = new char[newLength];
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```

StringBuffer与Stringbuilder区别

在构建的时候上锁了

```java
public synchronized StringBuffer append(String str) {
        toStringCache = null;
        super.append(str);
        return this;
    }
```



StringBuilder与StringBuffer都继承自AbstractStringBuilder类，在AbstractStringBuilder中也是使用字符数组保存字符串，如下就是，可知这两种对象都是可变的。

```java
　　char[] value;
```

StringBuffer是可变类，和线程安全的字符串操作类，任何对它指向的字符串的操作都不会产生新的对象。 每个StringBuffer对象都有一定的缓冲区容量，当字符串大小没有超过容量时，不会分配新的容量，当字符串大小超过容量时，会自动增加容量。

StringBuffer和StringBuilder类功能基本相似，主要区别在于StringBuffer类的方法是多线程、安全的，而StringBuilder不是线程安全的，相比而言，StringBuilder类会略微快一点。对于经常要改变值的字符串应该使用StringBuffer和StringBuilder类。



StringBuilder与StringBuffer有公共父类AbstractStringBuilder(抽象类)。

StringBuilder、StringBuffer的方法都会调用AbstractStringBuilder中的公共方法，如super.append(...)。只是StringBuffer会在方法上加synchronized关键字，进行同步。

它们的主要方法

| 方法                                                    | 说明                                                         |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| StringBuffer append(参数)                               | 追加内容到当前StringBuffer对象的末尾，类似于字符串的连接     |
| StringBuffer deleteCharAt(int index)                    | 删除指定位置的字符，然后将剩余的内容形成新的字符串           |
| StringBuffer insert(位置, 参数)                         | 在StringBuffer对象中插入内容，然后形成新的字符串             |
| StringBuffer reverse()                                  | 将StringBuffer对象中的内容反转，然后形成新的字符串           |
| void setCharAt(int index, char ch)                      | 修改对象中索引值为index位置的字符为新的字符ch                |
| void trimToSize()                                       | 将StringBuffer对象的中存储空间缩小到和字符串长度一样的长度，减少空间的浪费，和String的trim()是一样的作用 |
| StringBuffer delete(int start, int end)                 | 删除指定区域的字符串                                         |
| StringBuffer replace(int start, int end, String s)      | 用新的字符串替换指定区域的字符串                             |
| void setlength（int n）                                 | 设置字符串缓冲区大小                                         |
| int capacity()                                          | 获取字符串的容量                                             |
| void ensureCapacity(int n)                              | 确保容量至少等于指定的最小值。如果当前容量小于该参数，然后分配一个新的内部数组容量更大。新的容量是较大的. |
| getChars(int start,int end,char chars[],int charStart); | 将字符串的子字符串复制给数组                                 |

#### 线程安全

String中的对象是不可变的，也就可以理解为常量，显然线程安全。

AbstractStringBuilder是StringBuilder与StringBuffer的公共父类，定义了一些字符串的基本操作，如expandCapacity、append、insert、indexOf等公共方法。

StringBuffer对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。

#### 总结

（1）.如果要操作少量的数据用 = String 
（2）.单线程操作字符串缓冲区 下操作大量数据 = StringBuilder 
（3）.多线程操作字符串缓冲区 下操作大量数据 = StringBuffer

### final 和 Static

#### final

1. 修饰类时

> 修饰类当用final去修饰一个类的时候，表示这个类不能被继承
>
> 注意：
>
> a. 被final修饰的类，final类中的成员变量可以根据自己的实际需要设计为fianl。
>
> b. final类中的成员方法都会被隐式的指定为final方法。

2. 修饰方法时

> 被final修饰的方法不能被重写。
>
> **注意：**
>
> a. 一个类的private方法会隐式的被指定为final方法。
>
> b. 如果父类中有final修饰的方法，那么子类不能去重写。

3. 修饰成员变量

> 这里有两种情况
>
> + 修饰的是**基本类型**
>
> 必须要赋初始值，而且是只能初始化一次。
>
> +  修饰引用类型
>
> ```java
> final EntityClass entityClassFinal = new EntityClass();
> entityClassFinal.setAge(1);
> entityClassFinal.setUser("xuxu");
> ```
>
> 如果修饰的成员变量是一个引用类型，则是说这个引用的地址的值不能修改，但是这个引用所指向的对象里面的内容还是可以改变的。

用final生命的变量一旦赋予了值，就不能改变。在运行期来看就是一个内存地址，也就是说不能用它指向其他地址，至于地址里存放的内容是能改变的

#### static

static的主要作用在于创建独立于具体对象的域变量或者方法

__static产生的原因__

> 我们知道，当我们通过new关键字去创建对象的时候，那么数据的存储空间才会被分配，类中的成员方法才能被对象所使用。但是呢有两种特殊的情况：1、我们通过new关键字创建的对象共享同一个资源，而不是说每个对象都拥有自己的数据，或者说根本就不需要去创建对象，这个资源和对象之间是没有关系的。2、希望某个方法不与包含它的类的任何对象联系在一起。总结下来就是说：**即使没有创建对象，也能使用属性和调用方法**，static目的就是在于解决这个问题。

**格式**

> 修饰变量：static 数据类型 变量名
>
> 修饰方法：【访问权限修饰符】 static 方法返回值 方法名(参数列表)

**特点**

> 1. static可以修饰变量，方法被
> 2. static修饰的变量或者方法是独立于该类的任何对象，也就是说，这些变量和方法不属于任何一个实例对象，而是被类的实例对象所共享。
> 3. 在类被加载的时候，就会去加载被static修饰的部分。
> 4. 被static修饰的变量或者方法是优先于对象存在的，也就是说当一个类加载完毕之后，即便没有创建对象，也可以去访问。

**static静态变量**

> 被static修饰的成员变量叫做静态变量，也叫做类变量，说明这个变量是属于这个类的，而不是属于是对象，没有被static修饰的成员变量叫做实例变量，说明这个变量是属于某个具体的对象的。
>
> 

**静态变量和实例变量的区别**

> 实例变量：每次创建对象，都会为每个对象分配成员变量内存空间，实例变量是属于实例对象的，**在内存中，创建几次对象，就有几份成员变量**。
>
> 静态变量：静态变量由于不属于任何实例对象，是属于类的，所以在内存中只会有一份，在类的加载过程中，JVM为静态变量分配一次内存空间。

**static静态方法**

> 被static修饰的方法也叫做静态方法，因为对于静态方法来说是不属于任何实例对象的，那么就是说在静态方法内部是不能使用this的，因为既然不属于任何对象，那么就更谈不上this了。

**static应用场景**

> 如果某个成员变量是被所有对象所共享的，那么这个成员变量就应该定义为静态变量。

**static如何去访问**

> 静态变量：
>
> 类名.静态变量
>
> 对象.静态变量(不推荐的)
>
> 静态方法：
>
> 类名.静态方法
>
> 对象.静态方法(不推荐)
>
> 这里呢就啰嗦一句，由于被static修饰的变量和方法是不属于任何实例对象的，所以在这里，强烈建议不要通过对象的方式去访问静态的变量或者方法。

**static使用注意事项**

> 1. 在静态方法中没有this关键字因为静态是随着类的加载而加载，而this是随着对象的创建而存在的。静态比对象优先存在。
> 2. 静态可以访问静态的，但是静态不能访问非静态的。
> 3. 非静态的可以去访问静态的。



**总结**

> 1. 静态只能访问静态。
> 2. 非静态既可以访问非静态的，也可以访问静态的。

### PO VO

首先，java有几种对象(PO,VO,DAO,BO,POJO) 

- PO: persistant object 持久对象,可以看成是与数据库中的表相映射的java对象。使用Hibernate来生成PO是不错的选择。
- VO: value object值对象。通常用于业务层之间的数据传递，和PO一样也是仅仅包含数据而已。但应是抽象出的业务对象,可以和表对应,也可以不,这根据业务的需要.有一种观点就是：PO只能用在数据层，VO用在[商业逻辑](https://www.baidu.com/s?wd=商业逻辑&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)层和表示层。各层操作属于该层自己的数据对象，这样就可以降低各层之间的耦合，便于以后系统的维护和扩展。如果将PO用在各个层中就相当于我们使用全局变量，在OO设计非常不赞成使用全局变量。

１．VO是用new关键字创建，由GC回收的。

PO则是向数据库中添加新数据时创建，删除数据库中数据时削除的。并且它只能存活在一个数据库连接中，断开连接即被销毁。

２．VO是值对象，精确点讲它是业务对象，是存活在业务层的，是业务逻辑使用的，它存活的目的就是为数据提供一个生存的地方。

PO则是有状态的，每个属性代表其当前的状态。它是物理数据的对象表示。使用它，可以使我们的程序与物理数据解耦，并且可以简化对象数据与物理数据之间的转换。

３．VO的属性是根据当前业务的不同而不同的，也就是说，它的每一个属性都一一对应当前业务逻辑所需要的数据的名称。

PO的属性是跟数据库表的字段一一对应的。

PO对象需要实现序列化接口。

VO是独立的Java Object。

__总结__：    PO可以看成是与数据库中的表相映射的java对象,VO用在[商业逻辑](https://www.baidu.com/s?wd=商业逻辑&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)层和表示层。各层操作属于该层自己的数据对象.



## 数组

### 打印数组

（1）传统的for循环方式

```java
for(int i=0;i<array.length;i++)
{
      System.out.println(array[i]);
}
```



（2）for each循环

```java
for(int a:array)
    System.out.println(a);
```



（3）利用Array类中的toString方法

调用Array.toString(a)，返回一个包含数组元素的字符串，这些元素被放置在括号内，并用逗号分开

```java
int[] array = {1,2,3,4,5};
System.out.println(Arrays.toString(array));
```

## 数相关

### 生成随机数

 直接使用Math.random()方法生成随机数的方法。

```java
//定义一个double类型的变量来接收Math.random()函数产生的[0,1.0)区间的随机数
	double b1 = Math.random();
//随机生成1~100之间的一个整数
    int randomNumber = (int)(Math.random() * 100) + 1;

```

 使用Random类的方法。

Random()：创建一个新的随机数生成器。

Random(long seed)：使用单个 long 种子创建一个新的随机数生成器。

 第一种构造方法是使用默认当前系统时间的毫秒数作为种子数:Random r1 = new Random();  

 第二种方法是使用自己指定的种子数  Random random1 = new Random(100); 

 发现只要种子数和nextInt()中的参数一致的话，每次生成的随机数都是一样的（所以这是伪随机数）。 

下面是Java.util.Random()方法摘要：

1. protected int next(int bits)：生成下一个伪随机数。
2. boolean nextBoolean()：返回下一个伪随机数，它是取自此随机数生成器序列的均匀分布的boolean值。
3. void nextBytes(byte[] bytes)：生成随机字节并将其置于用户提供的 byte 数组中。
4. double nextDouble()：返回下一个伪随机数，它是取自此随机数生成器序列的、在0.0和1.0之间均匀分布的 double值。
5. float nextFloat()：返回下一个伪随机数，它是取自此随机数生成器序列的、在0.0和1.0之间均匀分布float值。
6. double nextGaussian()：返回下一个伪随机数，它是取自此随机数生成器序列的、呈高斯（“正态”）分布的double值，其平均值是0.0标准差是1.0。
7. int nextInt()：返回下一个伪随机数，它是此随机数生成器的序列中均匀分布的 int 值。
8. int nextInt(int n)：返回一个伪随机数，它是取自此随机数生成器序列的、在（包括和指定值（不包括）之间均匀分布的int值。
9. long nextLong()：返回下一个伪随机数，它是取自此随机数生成器序列的均匀分布的 long 值。
10. void setSeed(long seed)：使用单个 long 种子设置此随机数生成器的种子。



### Java位运算

位运算符用来对二进制位进行操作，包括按位取反（~）、按位与（&）、按位或（|）、异或（^）、右移（>>）、左移（<<）和无符号右移（>>>）。位运算符只能对整数型和字符型数据进行操作。

**1. 取反（~）**

参加运算的一个数据，按二进制位进行“取反”运算。

运算规则：~1=0； ~0=1；

即：对一个二进制数按位取反，即将0变1，1变0。

**2. 按位与（&）**

参加运算的两个数据，按二进制位进行“与”运算。

运算规则：0&0=0; 0&1=0; 1&0=0; 1&1=1；即：两位同时为“1，结果才为“1，否则为0。

例如：3&5 即 0000 0011 & 0000 0101 = 0000 0001 因此，3 & 5的值得1。

**3. 按位或（|）**

参加运算的两个对象，按二进制位进行“或”运算。

运算规则：0 | 0=0； 0 | 1=1； 1 | 0=1； 1 | 1=1；

即 ：参加运算的两个对象只要有一个为1，其值为1。

例如：3 | 5，即 0000 0011 | 0000 0101 = 0000 0111 因此，3 | 5的值得7。

**4. 异或（^）**

参加运算的两个数据，按二进制位进行“异或”运算。

运算规则：0^0=0； 0^1=1； 1^0=1； 1^1=0；

即：参加运算的两个对象，如果两个相应位为“异”（值不同），则该位结果为1，否则为0。

**5. \**左移（<<）\****

运算规则：按二进制形式把所有的数字向左移动对应的位数，高位移出（舍弃），低位的空位补零。例如： 12345 << 1，则是将数字12345左移1位：

## JSON

### json 解析

引入 json-lib-2.4.jar这个jar包，并且引入的函数包是net.sf.json.JSONObject;而不是org.json.JSONObject; 

POM.XML引入

```xml
        <dependency>
            <groupId>net.sf.json-lib</groupId>
            <artifactId>json-lib</artifactId>
            <version>2.4</version>
            <classifier>jdk15</classifier>
        </dependency>
```

#### **将json串转换成数组Array**

```java
package com.yinxin.server;
import java.util.Arrays;
import net.sf.json.JSONArray;
 
public class Test {
		public static void main(String[] args) {
			int[] num=new int[]{89,90,99};
		       JSONArray jsonArray = JSONArray.fromObject(num);
		       Object array = JSONArray.toArray(jsonArray);
		       System.out.println(array);
		       System.out.println(Arrays.asList((Object[]) array));
		   }
 
}
```

#### 将 Json 串转成 JavaBean/Map

```java
public class Test {
		public static void main(String[] args) {
			   /**
		        * 将 Json 形式的字符串转换为 Map
		        */
		       String str = "{\"name\":\"cy\",\"age\":22}";
		       JSONObject jsonObject = JSONObject.fromObject(str);
		       Map<String, Object> map = (Map<String, Object>) JSONObject.toBean(jsonObject, Map.class);
		       System.out.println(map);
 
		       /**
		        * 将 Json 形式的字符串转换为 JavaBean
		        */
		       str = "{\"id\":\"100101\",\"name\":\"zhangsan\"}";
		       jsonObject = JSONObject.fromObject(str);
		       System.out.println(jsonObject);
		       Person person = (Person) JSONObject.toBean(jsonObject, Person.class);
		       System.out.println(person);
		  }
}
```

#### 获取JSON Map形式的值

```java
[    
	{
        "id": 1540629,
        "name": "我的小鱼你睡着了",
        "packageName": "com.ct.client",
        "iconUrl": "app/com.ct.client/icon.jpg",
        "stars": 2,
        "size": 4794202,
        "des": "是他的手，还是我的手，愿夜幕永不开启"
    }
]

JSONArray jsonArray=new JSONArray(json);
            for(int i=0;i<jsonArray.length();i++){
                JSONObject object=jsonArray.getJSONObject(i);
                String id=object.getString("id");
                String name=object.getString("name");
                String packageName=object.getString("packageName");
                String iconUrl = object.getString("iconUrl");
                double stars=Double.parseDouble(object.getString("stars"));
                String size=object.getString("size");
                String downloadUrl = object.getString("downloadUrl");
                String des = object.getString("des");
               //其他操作
               ....
            }

```

## 其他

### get/post选择

**GET**

“读取“一个资源。比如Get到一个html文件。反复读取不应该对访问的数据有副作用。比如”GET一下，用户就下单了，返回订单已受理“，这是不可接受的。没有副作用被称为“幂等“（Idempotent)。

因为GET因为是读取，就可以对GET请求的数据做缓存。这个缓存可以做到浏览器本身上（彻底避免浏览器发请求），也可以做到代理上（如nginx），或者做到server端（用Etag，至少可以减少带宽消耗）

**POST**

在页面里<form> 标签会定义一个表单。点击其中的submit元素会发出一个POST请求让服务器做一件事。这件事往往是有副作用的，不幂等的。

每次提交，表单的数据被浏览器用编码到HTTP请求的body里。浏览器发出的POST请求的body主要有有两种格式，一种是application/x-www-form-urlencoded用来传输简单的数据，大概就是"key1=value1&key2=value2"这样的格式。另外一种是传文件，会采用multipart/form-data格式。采用后者是因为application/x-www-form-urlencoded的编码方式对于文件这种二进制的数据非常低效。

浏览器在POST一个表单时，url上也可以带参数，只要<form action="url" >里的url带querystring就行。只不过表单里面的那些用<input> 等标签经过用户操作产生的数据都在会在body里。

因此我们一般会**泛泛的说**“GET请求没有body，只有url，请求数据放在url的querystring中；POST请求的数据在body中“。但这种情况仅限于浏览器发请求的场景。

__幂等定义__

HTTP方法的幂等性是指一次和多次请求某一个资源应该具有同样的副作用。

可以简单的理解为，一次请求和多次请求获得的结果是一样的，产生的影响也是一样的，则为符合幂等性。

### HTTP请求幂等性

为什么需要幂等性呢？我们先从一个例子说起，假设有一个从账户取钱的远程API（可以是HTTP的，也可以不是），我们暂时用类函数的方式记为：

```
bool withdraw(account_id, amount)
```

withdraw的语义是从account_id对应的账户中扣除amount数额的钱；如果扣除成功则返回true，账户余额减少amount；如果扣除失败则返回false，账户余额不变。值得注意的是：和本地环境相比，我们不能轻易假设分布式环境的可靠性。一种典型的情况是withdraw请求已经被服务器端正确处理，但服务器端的返回结果由于网络等原因被掉丢了，导致客户端无法得知处理结果。如果是在网页上，一些不恰当的设计可能会使用户认为上一次操作失败了，然后刷新页面，这就导致了withdraw被调用两次，账户也被多扣了一次钱。

这个问题的解决方案一是采用分布式事务，通过引入支持分布式事务的中间件来保证withdraw功能的事务性。分布式事务的优点是对于调用者很简单，复杂性都交给了中间件来管理。缺点则是一方面架构太重量级，容易被绑在特定的中间件上，不利于异构系统的集成；另一方面分布式事务虽然能保证事务的ACID性质，而但却无法提供性能和可用性的保证。

另一种更轻量级的解决方案是幂等设计。我们可以通过一些技巧把withdraw变成幂等的，比如：

```
int create_ticket() bool idempotent_withdraw(ticket_id, account_id, amount)
```

create_ticket的语义是获取一个服务器端生成的唯一的处理号ticket_id，它将用于标识后续的操作。idempotent_withdraw和withdraw的区别在于关联了一个ticket_id，一个ticket_id表示的操作至多只会被处理一次，每次调用都将返回第一次调用时的处理结果。这样，idempotent_withdraw就符合幂等性了，客户端就可以放心地多次调用。

基于幂等性的解决方案中一个完整的取钱流程被分解成了两个步骤：1.调用create_ticket()获取ticket_id；2.调用idempotent_withdraw(ticket_id, account_id, amount)。虽然create_ticket不是幂等的，但在这种设计下，它对系统状态的影响可以忽略，加上idempotent_withdraw是幂等的，所以任何一步由于网络等原因失败或超时，客户端都可以重试，直到获得结果。

和分布式事务相比，幂等设计的优势在于它的轻量级，容易适应异构环境，以及性能和可用性方面。在某些性能要求比较高的应用，幂等设计往往是唯一的选择。

__HTTP的幂等性__

HTTP协议本身是一种面向资源的应用层协议，但对HTTP协议的使用实际上存在着两种不同的方式：一种是RESTful的，它把HTTP当成应用层协议，比较忠实地遵守了HTTP协议的各种规定；另一种是SOA的，它并没有完全把HTTP当成应用层协议，而是把HTTP协议作为了传输层协议，然后在HTTP之上建立了自己的应用层协议。本文所讨论的HTTP幂等性主要针对RESTful风格的，不过正如上一节所看到的那样，幂等性并不属于特定的协议，它是分布式系统的一种特性；所以，不论是SOA还是RESTful的Web API设计都应该考虑幂等性。下面将介绍HTTP GET、DELETE、PUT、POST四种主要方法的语义和幂等性。

+ HTTP GET方法用于获取资源，不应有副作用，所以是幂等的

+ HTTP DELETE方法用于删除资源，有副作用，但它应该满足幂等性
+ POST所对应的URI并非创建的资源本身，而是资源的接收者,HTTP响应中应包含帖子的创建状态以及帖子的URI。两次相同的POST请求会在服务器端创建两份资源，它们具有不同的URI；所以，POST方法不具备幂等性。

### HTTP常见错误返回代码

1×× 　保留
2×× 　表示请求成功地接收
3×× 　为完成请求客户需进一步细化请求
4×× 　客户错误
5×× 　服务器错误 

**100 Continue**

指示客户端应该继续请求。回送用于通知客户端此次请求已经收到，并且没有被服务器拒绝。
客户端应该继续发送剩下的请求数据或者请求已经完成，或者忽略回送数据。服务器必须发送
最后的回送在请求之后。

**101 Switching Protocols**
服务器依照客服端请求，通过Upgrade头信息，改变当前连接的应用协议。服务器将根据Upgrade头立刻改变协议
在101回送以空行结束的时候。

**Successful**
———————————————-
**200 OK**
指示客服端的请求已经成功收到，解析，接受。

**201 Created**
请求已经完成并一个新的返回资源被创建。被创建的资源可能是一个URI资源，通常URI资源在Location头指定。回送应该包含一个实体数据
并且包含资源特性以及location通过用户或者用户代理来选择合适的方法。实体数据格式通过煤体类型来指定即content-type头。最开始服务 器
必须创建指定的资源在返回201状态码之前。如果行为没有被立刻执行，服务器应该返回202。

**202 Accepted**
请求已经被接受用来处理。但是处理并没有完成。请求可能或者根本没有遵照执行，因为处理实际执行过程中可能被拒绝。

**203 Non-Authoritative Information**
不是权威性信息。
**204 No Content**
服务器已经接受请求并且没必要返回实体数据，可能需要返回更新信息。回送可能包含新的或更新信息由entity-headers呈现。

**205 Reset Content**
服务器已经接受请求并且用户代理应该重新设置文档视图。

**206 Partial Content**
服务器已经接受请求GET请求资源的部分。请求必须包含一个Range头信息以指示获取范围可能必须包含If-Range头信息以成立请求条件。

**Redirection**
—————————————————
**300 Multiple Choices

**请求资源符合任何一个呈现方式。

**301 Moved Permanently**
请求的资源已经被赋予一个新的URI。

**302 Found**
通过不同的URI请求资源的临时文件。
**303 See Other**

**304 Not Modified**
如果客服端已经完成一个有条件的请求并且请求是允许的，但是这个文档并没有改变，服务器应该返回304状态码。304
状态码一定不能包含信息主体，从而通常通过一个头字段后的第一个空行结束。

**305 Use Proxy

**请求的资源必须通过代理（由Location字段指定）来访问。Location资源给出了代理的URI。

**306 Unused**

**307 Temporary Redirect**
临时重定向。
**Client Error**
———————————————–
**400 Bad Request**
因为错误的语法导致服务器无法理解请求信息。

**401 Unauthorized**
如果请求需要用户验证。回送应该包含一个WWW-Authenticate头字段用来指明请求资源的权限。

**402 Payment Required**
保留状态码。

**403 Forbidden**
服务器接受请求，但是被拒绝处理。

**404 Not Found**
服务器已经找到任何匹配Request-URI的资源。

**405 Menthod Not Allowed**
Request-Line 请求的方法不被允许通过指定的URI。

**406 Not Acceptable**
客户端浏览器不接受所请求页面的 MIME 类型。
**407 Proxy Authentication Required**
要求进行代理身份验证。
**408 Reqeust Timeout**
客服端没有提交任何请求在服务器等待处理时间内。

**409 Conflict**

**410 Gone**

**411 Length Required**
服务器拒绝接受请求在没有定义Content-Length字段的情况下。

**412 Precondition Failed**
前提条件失败。
**413 Request Entity Too Large**
服务器拒绝处理请求因为请求数据超过服务器能够处理的范围。服务器可能关闭当前连接来阻止客服端继续请求。

**414 Request-URI Too Long**
服务器拒绝服务当前请求因为URI的长度超过了服务器的解析范围。

**415 Unsupported Media Type**
服务器拒绝服务当前请求因为请求数据格式并不被请求的资源支持。

**416 Request Range Not Satisfialbe**
所请求的范围无法满足。
**417 Expectation Failed**
执行失败。

**Server Error**
————————————————-
**500 Internal Server Error**
服务器遭遇异常阻止了当前请求的执行

**501 Not Implemented**
服务器没有相应的执行动作来完成当前请求。

**502 Bad Gateway**
Web 服务器用作网关或代理服务器时收到了无效响应。
**503 Service Unavailable**
因为临时文件超载导致服务器不能处理当前请求。

**504 Gateway Timeout**
网关访问超时。

### Java中instanceof和isInstance的具体区别

- obj.instanceof(class)

表示对象obj是否是class类或其子类的对象

1. 一个对象是自身类的一个对象
2. 一个对象是自身类父类和接口的一个对象
3. 所有对象都是Object类的对象
4. 凡是null有关的都是false

- class.isInstance(obj)

即对象obj能否转化为类class的对象，动态等价于instanceof

1. 一个对象能转化为自身类的对象
2. 一个对象能被转化为自身类的父类和实现的接口的对象
3. 所有对象都能转化为Object类的对象
4. 凡是null有关的都是false

可见与instanceof用法相同，关键在于动态等价

对obj.instanceof(class)，在编译时编译器需要知道类的具体类型

对class.isInstance(obj)，编译器在运行时才进行类型检查，故可用于反射，泛型中

>参考：
>
>https://stackoverflow.com/questions/8692214/when-to-use-class-isinstance-when-to-use-instanceof-operator
>
>https://stackoverflow.com/questions/15757014/isinstance-instanceof-why-theres-no-generic-way
>
>http://blog.csdn.net/u010002184/article/details/79306195

## 12常见类

+ [Object类](https://blog.csdn.net/ytasdfg/article/details/81044206)
+ [Scanner类](https://blog.csdn.net/ytasdfg/article/details/81044624)
+ [String类](https://blog.csdn.net/ytasdfg/article/details/81046102)
+ [StringBuffer类](https://blog.csdn.net/ytasdfg/article/details/81046725)
+ [Integer类](https://blog.csdn.net/ytasdfg/article/details/81047315)
+ [Character类](https://blog.csdn.net/ytasdfg/article/details/81080488 )
+ [ Math类](https://blog.csdn.net/ytasdfg/article/details/81081554)
+ [Random类](https://blog.csdn.net/ytasdfg/article/details/81081735)
+ [System类](https://blog.csdn.net/ytasdfg/article/details/81082435)
+ [BigInteger类与BigDecimal类](https://blog.csdn.net/ytasdfg/article/details/81082792)
+ [Date类](https://blog.csdn.net/ytasdfg/article/details/81083260)
+ [Calendar类](https://blog.csdn.net/ytasdfg/article/details/81086118)

