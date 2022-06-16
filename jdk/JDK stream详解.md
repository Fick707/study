# JDK stream详解

jdk的stream是对集合对象功能的增强，使集合对象的聚合操作、批量操作更便利和高效，借助于lambda表达式，有极好的编程效率及很高的可读性。

## stream（流）总览
* stream不是集合元素，不是数据结构，不保存数据。它是关于算法和计算的。
* stream就是Iterator，单向，不可往复，数据只遍历一次。
* 不同于Iterator，stream可以实现并行操作，提升性能。

## 流的构成
我们使用流，一般包含三个过程：获取流-->转换流（0次或者多次）-->操作，形成如下图所示的管道：

![](https://fick707.com/pics/study/java/stream/stream_pipeline.png)

### 流的获取或生成

* 从集合或数组
	* Collection.stream()
	* Collection.parallelStream()
	* Arrays.stream(T array) or Stream.of()
* 从 BufferedReader
	* java.io.BufferedReader.lines()
* 静态工厂
	* java.util.stream.IntStream.range()
	* java.nio.file.Files.walk()
* 自己构建
	* java.util.Spliterator
	* 其它
		* Random.ints()
		* BitSet.stream()
		* Pattern.splitAsStream(java.lang.CharSequence)
		* JarFile.stream()

### 流的操作（转换、操作）

流的操作分为两类：

* Intermediate：一个流可以有0个或者多个Intermediate操作，用于打开流、映射、过滤等，返回一个新的流。这些操作都是Lazy的，即调用这些方法时，并没有真正开始遍历流；
* Terminal: 一个流只能有一个Terminal操作，调用这些方法，即开始遍历流并执行操作，生成结果；

如果一个流有多个Intermediate操作，真正执行时，并不会遍历多次，而将多种操作在一次遍历中执行了。

还有一种操作被称为：short-circuiting，用于：
* 对于一个intermediate 操作，如果它接受的是一个无限大（infinite/unbounded）的Stream，但返回一个有限的新Stream。
* 对于一个 terminal 操作，如果它接受的是一个无限大的 Stream，但能在有限的时间计算出结果。

当操作一个无限大的 Stream，而又希望在有限时间内完成操作，则在管道内拥有一个 short-circuiting 操作是必要非充分条件。

#### 操作分类
* Intermediate 
	* map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered
* Terminal 
	* forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator
* Short-circuiting 
	* anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 limit

## 流的常见用法
对 Stream 的使用就是实现一个 filter-map-reduce 过程，产生一个最终结果，或者导致一个副作用（side effect）。

#### 构建流的几种常见方法示例
```java
// 一个 Stream 只可以使用一次，代码为了简洁而重复使用了数次。
// 1. Individual values
Stream stream = Stream.of("a", "b", "c");
// 2. Arrays
String [] strArray = new String[] {"a", "b", "c"};
stream = Stream.of(strArray);
stream = Arrays.stream(strArray);
// 3. Collections
List<String> list = Arrays.asList(strArray);
stream = list.stream();

// 数值流的构造
IntStream.of(new int[]{1, 2, 3}).forEach(System.out::println);
IntStream.range(1, 3).forEach(System.out::println);
IntStream.rangeClosed(1, 3).forEach(System.out::println);
```
对于基本数值型，目前有三种对应的包装类型 Stream：  
IntStream、LongStream、DoubleStream。当然我们也可以用 Stream<Integer>、Stream<Long> >、Stream<Double>，但是 boxing 和 unboxing 会很耗时，所以特别为这三种基本数值型提供了对应的Stream。 
Java 8 中还没有提供其它数值型 Stream，因为这将导致扩增的内容较多。而常规的数值型聚合运算可以通过上面三种 Stream 进行

#### 流转换数据结构
```java

// 流转换成其他数据结构
// 1. Array
String[] strArray1 = stream.toArray(String[]::new);
// 2. Collection
List<String> list1 = stream.collect(Collectors.toList());
List<String> list2 = stream.collect(Collectors.toCollection(ArrayList::new));
Set set1 = stream.collect(Collectors.toSet());
Stack stack1 = stream.collect(Collectors.toCollection(Stack::new));
// 3. String
String str = stream.collect(Collectors.joining()).toString();

// List to Map,假设accounts是一个Account对象的List
return accounts.stream().collect(Collectors.toMap(Account::getId, Account::getUsername));
```

#### map
```java
// 转成大写
List<String> output = wordList.stream().map(String::toUpperCase).collect(Collectors.toList());

// 求平方
List<Integer> nums = Arrays.asList(1, 2, 3, 4);
List<Integer> squareNums = nums.stream().map(n -> n * n).collect(Collectors.toList());
```
map 生成的是个 1:1 映射，每个输入元素，都按照规则转换成为另外一个元素

#### flatMap
```java
Stream<List<Integer>> inputStream = Stream.of(
 Arrays.asList(1),
 Arrays.asList(2, 3),
 Arrays.asList(4, 5, 6)
 );
Stream<Integer> outputStream = inputStream.flatMap((childList) -> childList.stream());
// outputSteam结果是[1,2,3,4,5,6]
```
flatMap 把 input Stream 中的层级结构扁平化，就是将最底层元素抽出来放到一起，最终 output 的新 Stream 里面已经没有 List 了，都是直接的数字。

#### filter
```java
// 只留下偶数
Integer[] sixNums = {1, 2, 3, 4, 5, 6};
Integer[] evens = Stream.of(sixNums).filter(n -> n%2 == 0).toArray(Integer[]::new);

// 把单词挑出来
List<String> output = reader.lines().flatMap(line -> Stream.of(line.split(REGEXP))).filter(word -> word.length() > 0).collect(Collectors.toList());
```
filter 对原始 Stream 进行条件匹配，通过匹配的元素被留下来生成一个新Stream。

#### forEach
```java
roster.stream().filter(p -> p.getGender() == Person.Sex.MALE).forEach(p -> System.out.println(p.getName()));
```
forEach 方法接收一个 Lambda 表达式，然后在 Stream 的每一个元素上执行该表达式。  
forEach 不能修改自己包含的本地变量值，也不能用 break/return 之类的关键字提前结束循环。  
forEach 是Terminal操作，如果想对元素做多次操作，可以使用peek

#### peek
```java
Stream.of("one", "two", "three", "four")
 .filter(e -> e.length() > 3)
 .peek(e -> System.out.println("Filtered value: " + e))
 .map(String::toUpperCase)
 .peek(e -> System.out.println("Mapped value: " + e))
 .collect(Collectors.toList());
```

#### findFirst
```java
Optional<String> item = stream.findFirst();
```
findFirst是一个 termimal 兼 short-circuiting 操作，它总是返回 Stream 的第一个元素，或者空。  
它的返回值类型：Optional。
```java
// Optional 的简单示例
Optional.ofNullable(item).ifPresent(System.out::println);
int len = Optional.ofNullable(item).map(String::length).orElse(-1);
```
Stream 中的 findAny、max/min、reduce 等方法等返回 Optional 值。还有例如 IntStream.average() 返回 OptionalDouble 等等。

#### reduce
```java
// 字符串连接，concat = "ABCD"
String concat = Stream.of("A", "B", "C", "D").reduce("", String::concat);
// 求最小值，minValue = -3.0
double minValue = Stream.of(-1.5, 1.0, -3.0, -2.0).reduce(Double.MAX_VALUE, Double::min);
// 求和，sumValue = 10, 有起始值
int sumValue = Stream.of(1, 2, 3, 4).reduce(0, Integer::sum);
// 求和，sumValue = 10, 无起始值
sumValue = Stream.of(1, 2, 3, 4).reduce(Integer::sum).get();
// 过滤，字符串连接，concat = "ace"
concat = Stream.of("a", "B", "c", "D", "e", "F").
 filter(x -> x.compareTo("Z") > 0).
 reduce("", String::concat);
```
这个方法的主要作用是把 Stream 元素组合起来。它提供一个起始值（种子），然后依照运算规则（BinaryOperator），和前面 Stream 的第一个、第二个、第 n 个元素组合。从这个意义上说，字符串拼接、数值的 sum、min、max、average 都是特殊的 reduce。例如 Stream 的 sum 就相当于Integer sum = integers.reduce(0, (a, b) -> a+b);或Integer sum = integers.reduce(0, Integer::sum);

也有没有起始值的情况，这时会把 Stream 的前面两个元素组合起来，返回的是 Optional。

#### limit/skip
```java
// persons是有100000个元素的列表
List<String> personList2 = persons.stream().
map(Person::getName).limit(10).skip(3).collect(Collectors.toList());

// 注意sorted 操作后的limit/skip对执行次数的影响
List<Person> personList2 = persons.stream().sorted((p1, p2) -> p1.getName().compareTo(p2.getName())).limit(2).collect(Collectors.toList());
```
limit 返回 Stream 的前面 n 个元素；skip 则是扔掉前 n 个元素（它是由一个叫 subStream 的方法改名而来）。  
请注意limit/skip对执行次数的影响：  
这是一个有 100000 个元素的 Stream，但在 short-circuiting 操作 limit 和 skip 的作用下，管道中 map 操作指定的 getName() 方法的执行次数为 limit 所限定的 10 次，而最终返回结果在跳过前 3 个元素后只有后面 7 个返回。   
有一种情况，limit/skip 无法达到 short-circuiting的目的：把它们放在 Stream 的排序操作后，原因是 sorted 这个 intermediate 操作：此时系统并不知道 Stream 排序后的次序如何，所以 sorted 中的操作看上去就像完全没有被 limit 或者 skip 一样。

#### sorted
```java
List<Person> personList2 = persons.stream().limit(2).sorted((p1, p2) -> p1.getName().compareTo(p2.getName())).collect(Collectors.toList());
```
对 Stream 的排序通过 sorted 进行，它比数组的排序更强之处在于你可以首先对 Stream 进行各类 map、filter、limit、skip 甚至 distinct 来减少元素数量后，再排序，这能帮助程序明显缩短执行时间。

#### min/max/distinct
```java
int longest = br.lines().mapToInt(String::length).max().getAsInt();

List<String> words = br.lines().flatMap(line -> Stream.of(line.split(" "))).filter(word -> word.length() > 0).map(String::toLowerCase).distinct().sorted().collect(Collectors.toList());
```
min 和 max 的功能也可以通过对 Stream 元素先排序，再 findFirst 来实现，但前者的性能会更好，为 O(n)，而 sorted 的成本是 O(n log n)。  
distinct 来找出不重复的元素

#### Match
Stream 有三个 match 方法，从语义上说：

* allMatch：Stream 中全部元素符合传入的 predicate，返回 true
* anyMatch：Stream 中只要有一个元素符合传入的 predicate，返回 true
* noneMatch：Stream 中没有一个元素符合传入的 predicate，返回 true 

它们都不是要遍历全部元素才能返回结果。例如 allMatch 只要一个元素不满足条件，就 skip 剩下的所有元素，返回 false。

## 进阶（自已构造流）

### group by

[stream group by](https://blog.csdn.net/frankenjoy123/article/details/70739800)  
[stream group by 2](https://blog.csdn.net/sugelachao/article/details/117736155)

### Stream.generate

通过实现 Supplier 接口，你可以自己来控制流的生成。这种情形通常用于随机数、常量的 Stream，或者需要前后元素间维持着某种状态信息的 Stream。把 Supplier 实例传递给 Stream.generate() 生成的 Stream，默认是串行（相对 parallel 而言）但无序的（相对 ordered 而言）。由于它是无限的，在管道中，必须利用 limit 之类的操作限制 Stream 大小。

```java
Random seed = new Random();
Supplier<Integer> random = seed::nextInt;
Stream.generate(random).limit(10).forEach(System.out::println);
//Another way
IntStream.generate(() -> (int) (System.nanoTime() % 100)).
limit(10).forEach(System.out::println);
```

#### 自实现 Supplier
```java
Stream.generate(new PersonSupplier()).
limit(10).
forEach(p -> System.out.println(p.getName() + ", " + p.getAge()));

private class PersonSupplier implements Supplier<Person> {
 private int index = 0;
 private Random random = new Random();
 @Override
 public Person get() {
 	return new Person(index++, "StormTestUser" + index, random.nextInt(100));
 }
}
```

## 结束语
Stream 的特性可以归纳为：

* 不是数据结构
* 它没有内部存储，它只是用操作管道从 source（数据结构、数组、generator function、IO channel）抓取数据。
* 它也绝不修改自己所封装的底层数据结构的数据。例如 Stream 的 filter 操作会产生一个不包含被过滤元素的新 Stream，而不是从 source 删除那些元素。
* 所有 Stream 的操作必须以 lambda 表达式为参数
* 不支持索引访问
* 你可以请求第一个元素，但无法请求第二个，第三个，或最后一个。不过请参阅下一项。
* 很容易生成数组或者 List
* 惰性化
* 很多 Stream 操作是向后延迟的，一直到它弄清楚了最后需要多少数据才会开始。
* Intermediate 操作永远是惰性化的。
* 并行能力
* 当一个 Stream 是并行化的，就不需要再写多线程代码，所有对它的操作会自动并行进行的。
* 可以是无限的
* 集合有固定大小，Stream 则不必。limit(n) 和 findFirst() 这类的 short-circuiting 操作可以对无限的 Stream 进行运算并很快完成。