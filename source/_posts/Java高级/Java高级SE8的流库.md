---
title: Java高级 （一）SE8的流库
date: 2019-10-23 15:31:02
tags: 
	- stream
	
categories: 
	- Java
---

##  Stream

对数据的操作，能在Java中像操作SQL一样操作数据，首先对数据源进行一系列的流水式的操作，不对原来的数据源进行改变。

__集合讲的是数据，流讲的是计算__

注意

+ Stream 自己不会存储元素
+ Stream 不会对源数据产生改变，
+ Stream 是有延迟执性的

## 流的创建

__java.util.stream.Stream8__

+ static <T> Stream <T> of (T  values) 产生一个元素给定值的流
+ static <T> Stream <T> empty()    产生一个不包含任何元素的流
+ static <T> Strame <T> generate (Supplier<T> s)  产生一个无限流，他的值时通过反复调用 函数s  而构建的
+ static <T> Strame <T>iterate （T seed ,UnaryOperator<T> f） 产生一个无限流，他的元素包含，种子，在种子上调用f产生的值，在前一个元素上调用f产生的值

__java.util.Arrays__

+ static <T >Stream<T> stream (T[]  array,  int   startInclusive , int endExclusive)  产生一个流，他的元素时由数组种指定范围内的元素构成的。

__java.util.regex.Pattern __

+ Stream<String> splitAsStream (CharSequence input) 产生一个流，他的元素时输入种由该模式界定的部分

__java.nio.file.Files__

+ static Stream <String> lines(Path path)
+ static Stream <String> lines(Path path, Charset cs)  产生一个流，他的元素时指定文件中的行，该文件的字符集位UTF-8，或者指定的字符集

__java.util.function.Supplier<T>__

+ T get()   提供一个值

```java
public class CreatingStreams
{
   public static <T> void show(String title, Stream<T> stream)
   {
      final int SIZE = 10;
      List<T> firstElements = stream
            .limit(SIZE + 1)
            .collect(Collectors.toList());
      System.out.print(title + ": ");
      for (int i = 0; i < firstElements.size(); i++)
      {
         if (i > 0) System.out.print(", ");
         if (i < SIZE) System.out.print(firstElements.get(i));
         else System.out.print("...");
      }
      System.out.println();
   }

   public static void main(String[] args) throws IOException
   {
      Path path = Paths.get("../gutenberg/alice30.txt");
      String contents = new String(Files.readAllBytes(path),
            StandardCharsets.UTF_8);

      Stream<String> words = Stream.of(contents.split("\\PL+"));
      show("words", words);
      Stream<String> song = Stream.of("gently", "down", "the", "stream");
      show("song", song);
      Stream<String> silence = Stream.empty();
      show("silence", silence);

      Stream<String> echos = Stream.generate(() -> "Echo");
      show("echos", echos);

      Stream<Double> randoms = Stream.generate(Math::random);
      show("randoms", randoms);

      Stream<BigInteger> integers = Stream.iterate(BigInteger.ONE,
            n -> n.add(BigInteger.ONE));
      show("integers", integers);

      Stream<String> wordsAnotherWay = Pattern.compile("\\PL+").splitAsStream(
            contents);
      show("wordsAnotherWay", wordsAnotherWay);

      try (Stream<String> lines = Files.lines(path, StandardCharsets.UTF_8))
      {
         show("lines", lines);
      }
   }
}
```

## 流的一些过滤操作

### filter , map ,和 flatMap（组合） 方法

流的转换回产生一个新的流，他的元素派生自另一个流中的元素。

__java.util.stream.Stream__

+ Stream<T> fillter(Predicate<?  super   T>  predicate)  产生一个流，他包含当前流中所有满足predicate的元素
+ <R> Stream<R> map (Function<? suoper T, ? extends  Stream <? extends R>> mapper)产生一个流，他是将mapper应用与当前流中所有元素所产生的结果连接在一起而获得的。（注意，这里的每个结果都是一个流）  

+ <R> Stream <R> faltMap (Function<? super T , ? extends Stream <? extegnd R>> mapper) 产生一个流，他是通过将mapper应用与当前流中所有元素所产生的解决连接到一起。

### 抽取子流和连接流

__java.util.stream .Stream 8__

+ Stream <T> limit ( long maxSize)   产生一个流，其中包含了当前流中最初的maxSize个元素   取前面的
+ Stream<T> skip (long n)  产生一个流，他的元素是当前流中除了前面的n个元素之外的所有元素。  取后面的
+ static <T> Stream <T> concat<Stream <? extends T > a ,Stream <? extends T>b> 产生一个流，将a流和b流连接起来，a在前b在后。

### 其他转换

__java.util.stream .Stream 8__

+ Stream<T> distinct()  产生一个流，去除当前流中不同的元素，      去重
+ Stream<T>  sorted()
+ Stream<T> sorted(Comparator<? super T> comparator) 产生一个流，他的元素是当前流中的所有的元素按照顺序排序的，第一个方法要求元素是实现了Comparable类的实例
+ Stream<T > peek(Consunmer <?  super  T>action) 产生一个流，他与当前流总的元素相同，在获取期中每个元素时，会将其传递给action.

## 流的结束（重点）

流的结束，我可以从流中获得怎么样的数据

### 简单的约简

简约会将流简约为可以在程序中使用的非流值，

__java.util.stream .Stream 8__

__一下操作都是终结操作__

+ Optional<T> max (Comparator<? super T> comparator)
+ Optional<T> min (Comparator<? super T> comparator) 分别产生这个流的最大元素，和最小元素，使用由给定比较器定义的排序规则，如果这个流为空，会产生，一个空的Optional对象，这些操作都是终结操作。
+ Optional<T> findfirst()
+ Optional<T> findAny() 分别产生这个流的第一个和任意一个元素，如果这个流为空，会产生一个空的Optional对象
+ boolean anyMatch (Predicate<? super T> predicate)
+ boolean allMatcth(Predicate<? super T> predicate)
+ boolean noneMatch(Predicate<? super T> predicate)  分别在这个流中任意元素，所有元素和没有任何元素匹配给定断言时返回true,这些操作都是终结操作。

### 收集结果

__java.util.stream.BaseStream 8__

+ Iterator<T> iterator()  产生一个用于获取当前流中的各各元素的迭代器，时终结操作

__java.util.stream.Stream 8__

+ void forEach (Consumer <? super T > action) 在流的每个元素上调用action,属于一种终结操作
+ Object[]  toArry()
+ Object[]  toArry( IntFunction<A[]> generator) 产生一个对象数组，或者将引用A[]::new 传递给构造器，返回一个A类型的数组，这些操作都是终结操作
+ <R ,A > R collect(Collector<? super T , A, R  > collector)  使用给定的收集器来收集当前流中的元素，Collectors类有用与多个收集器的工厂方法。

__java.util.stream.Stream.Collectors 8__

+ statci <T> Collector <T ,? ,List<T>> toList()   

+ statci <T> Collector <T ,? ,List<T>> toSet()   产生一个将元素收集到列表汇总集中的收集器

+ statci <T > Collector <T ,C extends Collection <T> > Collector<T ,?, C> toCollection(Supplier <C> collectionFactory)  产生一个将元素收集到任意的集合中的收集器，可以传递一个诸如，TreeSet:: new 的构造引用

+ static Collector<CharSequence ,?,String > joining()

+ static Collector<CharSequence ,?,String > joining(CharSequence delimiter)

+ static Collector<CharSequence ,?,String > joining(CharSequence delimiter ,CharSequence prefix ,CharSeQuence suffix) 产生一个连接字符的手机响，分割符会至于字符串之间，而第一个字符串之前可以有前缀，最后一个字符串之后可以有后缀，如果没有指定，他们都为空

+ static <T> Collector<T ,? ,IntSummaryStatistics> summarizingInt(ToIntFunction<? super T > mapper)

+ static <T> Collector<T ,? ,LongSummaryStatistics> summarizingInt(ToIntFunction<? super T > mapper)

+ static <T> Collector<T ,? ,DoubleSummaryStatistics> summarizingInt(ToIntFunction<? super T > mapper)

  能够产生一个（Int, Long, Double ）SummaryStatistics 对象的收集器，通过她可以获得将mapper应用与每个元素后所产生结果得个数，总和 ，平均值，最大最小值

  > + long getCount()   产生元素个数
  > + （Int, Long, Double ） getsum () 
  > + double getAvuerage() 产生汇总后得元素得总和或者平均值，或者没有任何元素得时候返回0
  > + （Int, Long, Double ）get Max
  > + （Int, Long, Double ）get Min 返回最大，最小值，

### 收集到映射表中

Collectors.toMap方法有两个函数引元，他们用来产生映射表的键和值。

```java
Map<Integer,String> idToName = people().collect(
			    Collectors.toMap(Person::getId, Person::getName));
```

通常情况下，值应该是实际的元素，因此第二个函数可以使用Function.identity()。

```shell
java.util.function.Function是函数式接口，它的特点是有且只有一个抽象方法。
对于函数式接口，形如这样的表达式（x -> x + 1）能够实例化一个对象。
并且该表达式正是那个唯一的抽象方法的实现。

Map<Integer, Person> idToPerson = people().collect(
				Collectors.toMap(Person::getId, Function.identity()));
```

如果多个元素映射同一个键，那么就会存在冲突，收集器将会抛出一个IllegalStateException对象，可以提供第三个引元来覆盖这种行为。

```java
Map<String,String> languageNames = locales.collect(
				Collectors.toMap(Locale::getDisplayLanguage, 
				l->l.getDisplayLanguage(),
				(existingValue,newValue)->existingValue));
```

如果一个键对应多个值时，需要一个Map<String,<Set < String > >。将已有集和新集做并操作。

```java
Map<String,Set<String>> countryLanguageSets = locales.collect(
				Collectors.toMap(Locale::getDisplayCountry, 
				l->Collections.singleton(l.getDisplayLanguage()),
				(a,b)->{//union of a and b 
					Set<String> union = new HashSet<>(a);
					union.addAll(b);
					return union;
				}));
```

如果想要得到TreeMap，可以将构造器作为第四个引元来提供，需要提供一种合并函数。

```java
idToPerson = people().collect(
				Collectors.toMap(Person::getId, Function.identity(),(
						existingValue,newValue)->{throw new IllegalStateException();//????
						},TreeMap::new));
```

### 群组和分区

 在上一节中，我们使每个映射表的值都生成单列集，然后指定将现有集与新集合并，这种处理显得有些冗长。
groupingBy支持将具有相同特性的值群聚成组。 

在上一节中，我们使每个映射表的值都生成单列集，然后指定将现有集与新集合并，这种处理显得有些冗长。
groupingBy支持将具有相同特性的值群聚成组。

```java
Map<String, List<Locale>> countryToLocales = locales.collect(
				Collectors.groupingBy(Locale::getCountry));

```

函数Locale::getCountry是群组的分类函数，现在可以查找给定国家代码对应的所有地点了。

```java
List<Locale> swissLocales = countryToLocales.get("CH");

```

- 注意：每个Locale都有一个语言代码（如英语的en）和一个国家代码（如美国的US）。Locale en_US描述的是美国英语，en_IE是爱尔兰英语。某个国家有多个Locale。（一个国家可以有多个语言）

当分类函数是断言函数（即返回boolean值的函数）时，流的元素可以区分为两个列表：该函数返回true的元素和其他元素。在这种情况下，使用partitioningBy更加高效。

```java
Map<Boolean,List<Locale>> englishAndOtherLocales = locales.collect(
				Collectors.partitioningBy(l->l.getLanguage().equals("en")));
		List<Locale> englishLocales = englishAndOtherLocales.get(true);

```

- 注意：如果调用groupingByConcurrent方法，就会在使用并行流时获得一个被并行组装的并行映射表。这与toConcurrentMap方法完全类似。
- 注意：
  `locales = Stream.of(Locale.getAvailableLocales());` 每次使用流时都要写一遍，因为当进行终止操作时，流就已经关闭了。不写则会报错`stream has already been operated upon or closed`

### 下游收集器

下游收集器用来处理groupingBy产生的映射表中每一个值的列表。
列如，如果想要获得集而不是列表，那么可以使用上一节中看到的Collector.toSet收集器：

- 注意 需导入 import static java.util.stream.Collectors.*;

```java
Map<String,Set<Locale>> countryToLocaleSet = locales.collect(
				groupingBy(Locale::getCountry,toSet()));

```

1. counting会产生收集到的元素的个数。

```java
Map<String, Long> countryToLocaleCounts = locales.collect(
				groupingBy(Locale::getCountry,counting()));

```

1. summing(Int|Long|Double)会接受一个函数作为引元，将函数运用到下游元素中，并产生他们的和。

```java
Map<String,Integer> stateToCityPopulation = cities.collect(
				groupingBy(City::getState,summingInt(City::getPopulation)));

```

1. maxBy和minBy会接受一个比较器，并产生下游元素中的最大值和最小值。

```java
Map<String, Optional<String>> stateToLongestCityName = cities
				.collect(groupingBy(City::getState,mapping(City::getName,java
						maxBy(Comparator.comparing(String::length)))));

```

1. mapping方法会产生将函数引用到下游结果上的收集器，并将函数值传递给另一个收集器。

```java
Map<String,Set<String>> countryToLanguages = locales.collect(
				groupingBy(Locale::getDisplayCountry,mapping(
						Locale::getDisplayLanguage,toSet())));
```

### 简约操作

reduce方法是一种用于从流中计算某个值得通用机制。

__java.utli.Stream 8__

+ Optional<T>  reduce (BinaryOperatir<T> accumulator)
+ T reduce (T identity , BinaryOperatir<T> accumulator))
+ <U>  U reduce ( U identity , BiFunction<U , ? super T ,U ) accumulator , BinaryOperator<U> combiner)用给定得accumulator函数产生流中元素得累计总和，如果提供了幺元，那么第一个被累计得元素就时该幺元，如果通过了组合器，那么她可以用来将分别累计得各个部分整合成总和
+ 

例：接受一个二元函数，并从前两个元素开始持续应用它。

```java
List<Integer> values = Arrays.asList(0,1,2,3,4,5,6,7,8,9);
Optional <Integer> sum = values.stream().reduce((x,y)->x+y);//result = 45
```

在上面的情况中reduce方法会计算v0+v1+v2+…，其中vi是流中元素。如果流为空则返回一个Optional.empty。

通常，如果reduce方法有一项约简操作op，那么该约简就会产生v0 op v1 op v2 op…，这项操作应该是可结合的（（x op y）op z = x op （y op z））：机组和元素使使用的顺序不应该成为问题。这在使用并行流时，可以执行高效的约简。

注意：减法是一个不可结合操作的例子。==减法可用reduce，但是不能并行操作（parallelStream）。上述代码，stream结果为-45，parallelStream结果为5（出错）。

通常会有一个幺元值e使得 e op x = x，可以使用这个元素作为计算的起点。
例如：0是加法的幺元，如果流为空，则返回幺元值，也就不需要处理Optional类了。

```java
List<Integer>values = Arrays.asList();
Integer sum = values.stream().reduce(0,(x,y)->x-y);//注意返回值为Integer， 而非Optional
```

如果想要对对象流的某些属性求和（例如求字符串流中所有字符串的长度），那么就不能使用简单形式的reduce，而是需要（T,T）->T这样的函数，==即引元和结果类型相同的函数。==但是在这种情况下，流的元素具有String类型，而累计结果是整数。

首先，需要提供一种“累积器”函数（total，word）-> total + word.length()。这个函数会被反复调用，产生累积的总和。
但是，当计算被并行化时，会有多个这种类型的计算，需要将他们的结果合并。因此需要第二个函数来执行此处理。

```java
words = Stream.of(contents.split(" "));
int result = words.reduce(0,
				(total, word)->total+word.length(),//引元和结果类型相同的函数
				(total1,total2)->total1+total2);
				//这里有一个小疑问：为什么total是int类型，而word是String类型
```

注意：通常，映射为数字流并使用其方法来计算总和、最大值和最小值会更容易。

### 基本类型操作

 将每个整数都包装到包装器对象中是很低效的 ， 对于一些基本类型：double、float、long、short、char、byte、和boolean，流库中有直接存储基本类型值的类型IntStream、LongStream和DoubleStream。若果想要存储short、char、byte、和boolean，可以使用IntStream，对于float，可以使用DoubleStream。 

基本类型得处理有

__java.util.stream.IntStream 8__

+ static IntStream range( int startInclusive , int endExclusive)
+ static IntStream rangeClose( int startInclusive , int endExclusive)  产生一个指定范围得构造IntStream
+ Static IntStream of( int values)  产生一个给定元素构造得InsStream 
+ int []  toArray[] 产生一个由当前流得元素构成得数据中
+ int sum () 求和
+ OptionalDuble average()  求和
+ Optionalint max ()
+ OptionalInt min()
+ summaryStatistics() 产生当前流中元素的额总和，平均值，最大值，最小值，或者从中获取这些解惑得所有四种值的对象
+ Stream<Integer> boxed()  产生用于当前流中得元素得包装器对象流

__java.util.stream.LongStream 8__



__java.util.stream.DoubleStream 8__



__java.util.stream.CharStream 8__

+ IntStream codePoints()   产生当前字符串所有得Unicode码点构成得流

__java.util.Random 1.0__   产生随机数流，给定值是有限 



通常，基本类型流上的方法与对象流上的方法类似。 下面是主要差异：

- toArray方法会返回基本类型数组。
- 产生可选结果的方法会返回一个OptionalInt、OptionalLong、OptionalDouble、这些类与Optional类类似，但是具有getAsInt、getAsLong和getAsDouble方法，而不是get方法。
- 具有返回总和、平均值、最大值和最小值的sum、average、max和min方法。对象流没有定义这些方法。
- summaryStatistics方法会产生一个类型为IntSummaryStatistics、LongSummaryStatistics或DoubleSummaryStatistics的对象，他们可以同时报告流的总和，平均值、最大值和最小值。

### 并行流

流使得并行处理块操作变得很容易。这个过程几乎是自动的，但是需要遵守一些规则。
首先，必须有一个并行流。可以用Collection.parallelStream()方法从任何集合中获取一个并行流：

```java
Stream<String> parallelWords = words.parallelStream();

```

而且，parallel方法可以将任意顺序流转换为并行流。

```java
Stream<String> parallelWords =Stream.of(wordArra).parallel();

```

只要在终结方法执行时，流处于并行模式，那么所有的中间流操作都将被并行化。
当流操作并行运行时，其目标是要让其返回结果与顺序执行时返回的结果相同。重要的是，这些操作可以以任意顺序执行。

假设想要对字符串流中的所有短单词计数：

```java
int[] shortWords = new int[10];
		wordList.parallelStream().forEach(s->
		{
			if(s.length()<10) shortWords[s.length()]++;
		});

```

这是一段非常非常糟糕的代码。传递给forEach的函数会在多个并发线程中运行，每个都会更新共享的数组。这是一种经典的竞争情况。如果多次运行这个程序，就会发现每次运行都会产生不同的结果，而且每个都是错的。

如果想确保传递给并行流操作的任何函数都可以安全地并行执行，那么需要远离易变状态。
例如：如果用长度将字符串群组，然后分别对他们计数，那么就可以安全地并行化这项计算。

```java
Map<Integer, Long> shortWordCounts = wordList.parallelStream()
				.filter(s->s.length()<10)
				.collect(groupingBy(String::length,counting()));
		System.out.println(shortWordCounts);
```

- 警告：传递给并行流操作的函数不应该被堵塞。并行流使用fork-join池来操作流的各个部分。如果多个流操作被堵塞，那么池可能就无法做任何事情了。

默认情况下，从有序集合（数组和列表）、范围、生成器和迭代产生的流，或者通过调用Stream.sorted产生的流，都是有序的 。他们的结果是按照原来元素的顺序累积的，因此是完全可预知的。若果运行相拥的操作两次，将会得到完全相同的结果。

排序并不排斥高效的并行处理。例如：当计算stream.map(fun)时，流可以被划分为n的部分，它们会被并行地处理。然后结果将会按照顺序重新组装起来。
当放弃排序需求时，有些操作可以被更有效地并行化。可以通过在流上调用unordered方法。
在有序流中，Stream.distinct方法会保留所有相同元素中的第一个，但这对并行化是一种阻碍，因为处理每个部分的线程 在其之前的所有部分都被处理完之前，并不知道应该丢弃哪些元素。如果可以接受保留唯一元素中任意一个的做法那么所有部分都可以并行的处理（实用共享的集来跟踪重复元素）。

还可以调用通过放弃排序来提高limit方法的速度。如果只想从流中取出任意n个元素，而并不在意到底要获取哪些，那么可以调用：

```java
Stream<String> sample = words.parallelStream().unordered().limit(n);
```

合并映射表的代价很高昂，正因为这个原因，Collectors.groupByConcurrent方法使用了共享的并发映射表。为了从并行化中获益，映射表中值的顺序不会与流中的顺序相同。

```java
Map<Integer, List<String>> result = wordList.parallelStream()
				.collect(Collectors.groupingByConcurrent(String::length));
```

当然，如果使用独立于排序的下游收集器，那么就不必在意了。

```java
Map<Integer, Long> wordCounts = wordList.parallelStream()
				.collect(Collectors.groupingByConcurrent(String::length,counting()));
```

-警告：不要修改执行某项流操作后会将元素返回到流中的集合没搞懂，希望大佬私聊我教教我Thanks♪(･ω･)ﾉ（即使这种修改是线程安全的）。记住，流不会收集他们的数据，流总是在单独的集合中。如果修改了这样的集合，那么流操作的结果就是未定义的。JDK文档对这项需求并未做出任何约束，并且对顺序流和并行流都采取了这种处理方式。
更准确地讲，因为中间流操作都是惰性的，所以直到执行终结操作时才对集合进行修改仍是可行的。
例如，下面的操作尽管并不推荐，但是仍旧可以工作：

```java
	Stream<String> wordss = wordList.stream();
	wordList.add("END");
	long n = wordss.distinct().count();
```

但是，下面的代码是错误的：

```java
Stream<String> words = wordList.stream();
words.forEach(s->{if(s.length()<12) wordList.remove(s);});
```

为了让并行流操作正常工作，需要满足大量的条件：

- 数据应该在内存中。必须等到数据到达时非常低效的。
- 流应该可以被高效的分成若干个子部分。由数组或平衡二叉树支撑的流都可以工作的很好，但是Stream.iterate返回的结果不行。
- 流操作的工作量应该具有较大的规模。如果工作负载并不是很大，那么搭建并行计算式所付出的代价就没有什么意义。
- 流操作不应该被阻塞。

所以说，不要将所有的流都转换为并行流。只有在对已经位于内存中的数据执行大量计算操作时，才应该使用并行流





## Optional 类型

Optional<T> 对象是一种包装类，要么包装了类型T对象，要么没有包装任何对象，用来解决NullPointerExceptions,问题。虽然不能有效的消除空指针，但是这个类有助于创建简单，可读性更强，对比应程序错误更少得到程序。

### 如何使用Optional

创建Optional

__java.util.Optional 8__

+ static <T> Optional<T> of (T value)
+ static <T> Optional<T> ofNullable (T value) 产生一个具有给定值的Optional,如果value为null，那么第一个方法会抛出NullpointerExcepting对象，第二方法返回一个空的Optional.
+ static <T> Optional<T> empty()  产生一个空的Optional.



使用Optional

__java.util.Optional 8 __

+ T orElse(T other) 产生这个Optional的值，或者在改Optional为空时，产生other.
+ T orElseGet(Supplier <? extends T> other)  产生这个Optional 的值，或者在改Optional为空时，抛出exceptionSupplier的结果
+ <X extends Throwable> T orElseThrow<Supplier <? extends X> exceptionSupplier) 产生个optional的值，或者在该Optional为空时，抛出调用exceptionSupplier的结果
+ void   ifPresent(Consumer <? super T> consumer)   如果该Optional不为空，那么就将他的值传递给consumer.
+ <U> Optional<U> map (Function<? suoer T ,? extends U> mapper) 产生将该Optional的值传递给mapper后的结果，只要这个Optional不为空且结果不为null，否则产生一个空的Optional

### 不合适使用Optional值的方式

get方法会在Optional值存在的情况下获得其中包装的元素，或者不存在的情况抛出一个NoSuchElementException对象，

__java.util.Optional 8__

+ T get()   产生这个Optional的值，或者在该Optional为空时，就抛出一个NoSuchElementException对象
+ boolean isPresent() 如果该Optional不为空，则返回ture

### 用flatMap来构建Optional值的函数

 map返回的是结果集，flatmap返回的是包含结果集的Observable（返回结果不同） 

__java.util.Optional 8__

+ <U> Optional<U> flatMap (Function <? super T , Optional<U>> mapper) 产生将mapper应用余当前Optional值所产生的结果，或者在当前Optional为空时，返回一个空Optional







[参考文章]( https://blog.csdn.net/z036548 )

