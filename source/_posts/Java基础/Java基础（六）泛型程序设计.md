---
title: Java基础（六） 泛型程序设计
date: 2019-10-11 8:04:20
tags:
	- 泛型
	- 通配符
categories: Java
---

## 为什么使用泛型

重用，是Java核心思想，泛型很好的解决了不同类型，相同操作的数据处理。

在Java中增加泛型类之前，泛型程序设计是用继承实现的，ArrayList类只维护一个Object引用的数组：

```java
public class ArrayList {
    private Object[] array;
    public void remove(int index) {...}
    public Object get(int index) {...}
}
```

这种方法有两个问题，当获取一个值时必须进行强制转换类型。

```java
ArrayList list = new ArrayList();
// 获取到Object，必须强制转型为String:
String first = (String) list.get(0);
```

这里没有错误检查，可以向数组列表中条件任何类的对象。

```java
files.add(new File("...."));
```

这个调用在编译的时候不会报错，但是在get的时候会强制转换成String类型，就会产生编译的错误；

泛型提供了一个更好的解决方案，类型参数（type parameters）,ArrayList类有一个类型参数用来指示元素的类型： Array List<String> files = new ArrayList<String> ();

具有更好的可读性。

同时编译器在编译的时候，可以更好的利用这个信息，不需要强行转换，编译器直接返回String而不是Object

## 定义简单的泛型类

类型变量    __T__    用尖括号括起来 <T> 放在类名的后面，泛型类可以有多个类型变量，

__Java库中的表示__

+  E  表示集合元素类型
+ K 和 V 分别表示关键字与值的类型
+ T 表示”任意值“    有时也还可以用”U“或者”S“来表示

```java
public class PairTest1
{
   public static void main(String[] args)
   {
      String[] words = { "Mary", "had", "a", "little", "lamb" };
      Pair<String> mm = ArrayAlg.minmax(words);
      System.out.println("min = " + mm.getFirst());
      System.out.println("max = " + mm.getSecond());
   }
}

_______________________________________________________________
class ArrayAlg
{
   /**
    * Gets the minimum and maximum of an array of strings.
    * @param a an array of strings
    * @return a pair with the min and max value, or null if a is null or empty
    */
   public static Pair<String> minmax(String[] a)
   {
      if (a == null || a.length == 0) return null;
      String min = a[0];
      String max = a[0];
      for (int i = 1; i < a.length; i++)
      {
         if (min.compareTo(a[i]) > 0) min = a[i];
         if (max.compareTo(a[i]) < 0) max = a[i];
      }
      return new Pair<>(min, max);
   }
}

```

## 泛型方法

```java
class Arrayalg{
    publci static<T> T get Middle(T a){
        return a[a.length/2];
    }
}
```

列（3.14，154，0）

在匹配类型的时候，首先会自动打包，打包成一个Double和两个Integer对象，而后找这些类共同的类型，事实上，找到两个这样的超类型，Number和Comparable接口。其本身也是一个泛型类型，在这时，采取补救措施写为double值。

## 类型变量的限定

有时候，在类和方法对类型变量加以约束，下面时一个计算元素中的最小元素

```java
class ArrayAlg{
    public static <T> T min(Tp[] a){
        if(a==null || a.length ==0)
            return null;
        T smallest = a[0];
        for(int i =1; i<a.length ; i++)
            if(smallest.compareTo(a[i])>0) smallest =a[i];
        return smallest;
    }
}
```

在这里面会有一问题，变量smallest类型为T,这意味着他可以为任意一个类的对象，怎么才能确信T所属类型中有compareTo方法呢？

这里需要这个设定

```java
public static <T extends Comparable> T min( T [] a)
```



## 泛型代码和虚拟机

虚拟机没有泛型类型对象--所有的对象都是普通类，

### 类型擦除

无论何时定义一个泛型类型，都自动提供了一个相对应的原始类型（raw type）.原始类型的名字就是删去类型参数后的泛型名称，擦除（erased）类型变量，并替换为限定类型

Java语言的泛型实现方式是擦拭法（Type Erasure）。

所谓擦拭法是指，虚拟机对泛型其实一无所知，所有的工作都是编译器做的。

例如，我们编写了一个泛型类`Pair<T>`，这是编译器看到的代码：

```
public class Pair<T> {
    private T first;
    private T last;
    public Pair(T first, T last) {
        this.first = first;
        this.last = last;
    }
    public T getFirst() {
        return first;
    }
    public T getLast() {
        return last;
    }
}
```

而虚拟机根本不知道泛型。这是虚拟机执行的代码：

```
public class Pair {
    private Object first;
    private Object last;
    public Pair(Object first, Object last) {
        this.first = first;
        this.last = last;
    }
    public Object getFirst() {
        return first;
    }
    public Object getLast() {
        return last;
    }
}
```

因此，Java使用擦拭法实现泛型，导致了：

- 编译器把类型`<T>`视为`Object`；
- 编译器根据`<T>`实现安全的强制转型。

使用泛型的时候，我们编写的代码也是编译器看到的代码：

```
Pair<String> p = new Pair<>("Hello", "world");
String first = p.getFirst();
String last = p.getLast();
```

而虚拟机执行的代码并没有泛型：

```
Pair p = new Pair("Hello", "world");
String first = (String) p.getFirst();
String last = (String) p.getLast();
```

所以，Java的泛型是由编译器在编译时实行的，编译器内部永远把所有类型`T`视为`Object`处理，但是，在需要转型的时候，编译器会根据`T`的类型自动为我们实行安全地强制转型。

### Java泛型的局限性

#### 不能用基本类型实例化类型参数

即`<T>`不能是基本类型，例如`int`，因为实际类型是`Object`，`Object`类型无法持有基本类型：

```java
Pair<int> p = new Pair<>(1, 2); // compile error!

  原始类型           封装类  
  boolean             Boolean  
  char                   Character  
  byte                   Byte  
  short                 Short  
  int                     Integer  
  long                   Long  
  float                 Float  
  double               Double  
  
  引用类型和原始类型的区别:

1.两者的初始化方式不同

1
2
int i = 5;                       // 原始类型
Integer j = new Integer(10);     // 对象引用  java 1.5以后支持自动装箱所以   Integer j = 10; 也可以
```



#### 运行时类型查询只适用与原始类型

虚拟机中的对象总有一个特定的非泛型类型，因此，所有的泛型查询只产生原始类型。

如  if(a instanceof Pair<String>)  这个语句是错误的

如果强制转换会出现警告

```java
pair<String> p = (pair<String>) a;  // Warning--can only test that a is a Pair
```

#### 不能创建参数化类型的数组

```java
pair<String> [] table = new pair<String>[10]   //是错误的
```

擦除后 table 的类型灰色Object 数组会记着这个类型，如果试图存储其他的数据类型就会报错

#### Varargs警告

向参数个数可变的方法中传递一个泛型类型的实例，和上面的这个相似，在多惨传递的到时候类似与可变的数组，但是在这里系统报的但是一个警告，不是一个错误。

我们可以用一下的操作来抑制警告

```java
@SafeVarargs   //压制警告 
public static <T> void addAll(Colletion<T> coll,T....)//多参 可变   方法
    
```

#### 不能实例化类型变量

类型擦除本身是将T改成Object 当然不希望调用new Object()

```java
publci Pair(){
    first= new T ();
}//这个是错误的

//我们应该这样处理
pai<String> p = Pair.makPair(String::new);
```

#### 不能构建泛型数组

#### 泛型类的静态上下文中类型变量无效

#### 不能抛出或者捕获泛型类的实例

#### 可以消除对受查异常的检查

#### 不恰当的覆写方法

有些时候，一个看似正确定义的方法会无法通过编译。例如：

```
public class Pair<T> {
    public boolean equals(T t) {
        return this == t;
    }
}
```

这是因为，定义的`equals(T t)`方法实际上会被擦拭成`equals(Object t)`，而这个方法是继承自`Object`的，编译器会阻止一个实际上会变成覆写的泛型方法定义。

换个方法名，避开与`Object.equals(Object)`的冲突就可以成功编译：

```
public class Pair<T> {
    public boolean same(T t) {
        return this == t;
    }
}
```

## 泛型类型的继承规则

一个类可以继承自一个泛型类。例如：父类的类型是`Pair<Integer>`，子类的类型是`IntPair`，可以这么继承：

```
public class IntPair extends Pair<Integer> {
}
```

使用的时候，因为子类`IntPair`并没有泛型类型，所以，正常使用即可：

```
IntPair ip = new IntPair(1, 2);
```

个人理解： 当子类和父类同时有泛型的时候，他们两个引用的对象一定要相同不然就会出现错误，即使子类和父类引用的泛型对象继承与同一个对象；

![1570675117130](Java基础（六）泛型程序设计/1570675117130.png)

### 小结

Java的泛型是采用擦拭法实现的；

擦拭法决定了泛型`<T>`：

- 不能是基本类型，例如：`int`；
- 不能获取带泛型类型的`Class`，例如：`Pair<String>.class`；
- 不能判断带泛型类型的类型，例如：`x instanceof Pair<String>`；
- 不能实例化`T`类型，例如：`new T()`。

泛型方法要防止重复定义方法，例如：`public boolean equals(T obj)`；

子类可以获取父类的泛型类型`<T>`。

## 通配符类型

固定的泛型类型系统使用起来并不爽，解决方案：通配符类型。

```java
Pair<? extends Employess>
```

我们在前面讲到，不饿能将Pair<Manager>传递这个方法，这样很受现在，解决方案就是使用通配符

```java
public static void pritBuddies(Pair<? extends Empo>)
```

![IMG_5246(jpg)](Java基础（六）泛型程序设计/IMG_5246(20191010-105456).jpg)

从方法内部代码看，传入`List<? extends Integer>`或者`List<Integer>`是完全一样的，但是，注意到`List<? extends Integer>`的限制：

- 允许调用`get()`方法获取`Integer`的引用；
- 不允许调用`set(? extends Integer)`方法并传入任何`Integer`的引用（`null`除外）。

因此，方法参数类型`List<? extends Integer>`表明了该方法内部只会读取`List`的元素，不会修改`List`的元素（因为无法调用`add(? extends Integer)`、`remove(? extends Integer)`这些方法。换句话说，这是一个对参数`List<? extends Integer>`进行只读的方法（恶意调用`set(null)`除外）

### 使用extends限定T类型

在定义泛型类型`Pair<T>`的时候，也可以使用`extends`通配符来限定`T`的类型：

```
public class Pair<T extends Number> { ... }
```

现在，我们只能定义：

```
Pair<Number> p1 = null;
Pair<Integer> p2 = new Pair<>(1, 2);
Pair<Double> p3 = null;
```

因为`Number`、`Integer`和`Double`都符合`<T extends Number>`。

非`Number`类型将无法通过编译：

```
Pair<String> p1 = null; // compile error!
Pair<Object> p2 = null; // compile error!
```

因为`String`、`Object`都不符合`<T extends Number>`，因为它们不是`Number`类型或`Number`的子类。

+ 使用`extends`通配符表示可以读，不能写。

### 通配符的超类限定 super

通配符的限定和类型变量的限定相似，但是有一个附加功能，能够知道一个超类型限定（supertyoe bound）格式如下：

```java
？ super Manager
```

因此，使用`<? super Integer>`通配符表示：

- 允许调用`set(? super Integer)`方法传入`Integer`的引用；
- 不允许调用`get()`方法获得`Integer`的引用。

唯一例外是可以获取`Object`的引用：`Object o = p.getFirst()`。

换句话说，使用`<? super Integer>`通配符作为方法参数，表示方法内部代码对于参数只能写，不能读。

### 对比extends和super通配符

我们再回顾一下`extends`通配符。作为方法参数，`<? extends T>`类型和`<? super T>`类型的区别在于：

- `<? extends T>`允许调用读方法`T get()`获取`T`的引用，但不允许调用写方法`set(T)`传入`T`的引用（传入`null`除外）；
- `<? super T>`允许调用写方法`set(T)`传入`T`的引用，但不允许调用读方法`T get()`获取`T`的引用（获取`Object`除外）。

一个是允许读不允许写，另一个是允许写不允许读。

### PECS原则

何时使用`extends`，何时使用`super`？为了便于记忆，我们可以用PECS原则：Producer Extends Consumer Super。

即：如果需要返回`T`，它是生产者（Producer），要使用`extends`通配符；如果需要写入`T`，它是消费者（Consumer），要使用`super`通配符。

还是以`Collections`的`copy()`方法为例：

```
public class Collections {
    public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        for (int i=0; i<src.size(); i++) {
            T t = src.get(i); // src是producer
            dest.add(t); // dest是consumer
        }
    }
}
```

需要返回`T`的`src`是生产者，因此声明为`List<? extends T>`，需要写入`T`的`dest`是消费者，因此声明为`List<? super T>`。

### 无限定通配符

我们已经讨论了`<? extends T>`和`<? super T>`作为方法参数的作用。实际上，Java的泛型还允许使用无限定通配符（Unbounded Wildcard Type），即只定义一个`?`：

```java
void sample(Pair<?> p) {
}
```

因为`<?>`通配符既没有`extends`，也没有`super`，因此：

- 不允许调用`set(T)`方法并传入引用（`null`除外）；
- 不允许调用`T get()`方法并获取`T`引用（只能获取`Object`引用）。

换句话说，既不能读，也不能写，那只能做一些`null`判断：

```java
static boolean isNull(Pair<?> p) {
    return p.getFirst() == null || p.getLast() == null;
}
```

大多数情况下，可以引入泛型参数`<T>`消除`<?>`通配符：

```java
static <T> boolean isNull(Pair<T> p) {
    return p.getFirst() == null || p.getLast() == null;
}
```

## 泛型和反射

Java的部分反射API也是泛型。例如：`Class<T>`就是泛型：

```java
// compile warning:
Class clazz = String.class;
String str = (String) clazz.newInstance();

// no warning:
Class<String> clazz = String.class;
String str = clazz.newInstance();
```

调用`Class`的`getSuperclass()`方法返回的`Class`类型是`Class<? super T>`：

```
Class<? super String> sup = String.class.getSuperclass();
```

构造方法`Constructor<T>`也是泛型：

```
Class<Integer> clazz = Integer.class;
Constructor<Integer> cons = clazz.getConstructor(int.class);
Integer i = cons.newInstance(123);
```

我们可以声明带泛型的数组，但不能用`new`操作符创建带泛型的数组：

```
Pair<String>[] ps = null; // ok
Pair<String>[] ps = new Pair<String>[2]; // compile error!
```

必须通过强制转型实现带泛型的数组：

```
@SuppressWarnings("unchecked")
Pair<String>[] ps = (Pair<String>[]) new Pair[2];
```

使用泛型数组要特别小心，因为数组实际上在运行期没有泛型，编译器可以强制检查变量`ps`，因为它的类型是泛型数组。但是，编译器不会检查变量`arr`，因为它不是泛型数组。因为这两个变量实际上指向同一个数组，所以，操作`arr`可能导致从`ps`获取元素时报错，例如，以下代码演示了不安全地使用带泛型的数组：

```
Pair[] arr = new Pair[2];
Pair<String>[] ps = (Pair<String>[]) arr;

ps[0] = new Pair<String>("a", "b");
arr[1] = new Pair<Integer>(1, 2);

// ClassCastException:
Pair<String> p = ps[1];
String s = p.getFirst();
```

要安全地使用泛型数组，必须扔掉`arr`的引用：

```
@SuppressWarnings("unchecked")
Pair<String>[] ps = (Pair<String>[]) new Pair[2];
```

上面的代码中，由于拿不到原始数组的引用，就只能对泛型数组`ps`进行操作，这种操作就是安全的。

带泛型的数组实际上是编译器的类型擦除：

```
Pair[] arr = new Pair[2];
Pair<String>[] ps = (Pair<String>[]) arr;

System.out.println(ps.getClass() == Pair[].class); // true

String s1 = (String) arr[0].getFirst();
String s2 = ps[0].getFirst();
```

所以我们不能直接创建泛型数组`T[]`，因为擦拭后代码变为`Object[]`：

```
// compile error:
public class Abc<T> {
    T[] createArray() {
        return new T[5];
    }
}
```

必须借助`Class<T>`来创建泛型数组：

```
T[] createArray(Class<T> cls) {
    return (T[]) Array.newInstance(cls, 5);
}
```

我们还可以利用可变参数创建泛型数组`T[]`：

```
public class ArrayHelper {
    @SafeVarargs
    static <T> T[] asArray(T... objs) {
        return objs;
    }
}

String[] ss = ArrayHelper.asArray("a", "b", "c");
Integer[] ns = ArrayHelper.asArray(1, 2, 3);
```

谨慎使用泛型可变参数

__Java.lang,Class<T>__

+ T newInstance() 返回无参狗再去构造的一个实例
+ T cast(Object obj) 如果obj为null或者有可能转化类型为T,则返回obj,否则抛出，BadCasException异常

__小结__

+ 部分反射API是泛型，例如：`Class<T>`，`Constructor<T>`；

+ 可以声明带泛型的数组，但不能直接创建带泛型的数组，必须强制转型；

+ 可以通过`Array.newInstance(Class<T>, int)`创建`T[]`数组，需要强制转型；

+ 同时使用泛型和可变参数时需要特别小心。