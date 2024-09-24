---
layout: post
title: "强大的Stream流"
date: 2024-06-19 
description: "Java Stream流详解"
tag: Java
---  
  如果你要问我Java中最常用最高效最爽的数据处理方式是什么，那我一定会回答——Stream流！只能说年少不知Stream好，错把循环当成宝，当然这是玩笑，要在合适的场景用合适的方法。
  
  言归正传，最近接了一个需求，其中有一段代码需要特殊的数据处理，开始的方案是用了几个极其丑陋的for循环占了十几行代码，导致我心情不佳，后面发现其实这也是可以靠Stream流处理的，只是我之前没有用过，最后精简下来的代码如下：

![image-20240913153943668](/images/posts/Java/javaStream/image-20240913153943668.png)

​	先不说好不好读的问题，不好读咱可以加注释，就单说这简洁的程度很难不让人心情愉悦。所以我就发现Stream比我想象中还要强大，但其实平时常用到的就只有map、filter等方法，大大的限制了自己对流的想象力。于是就想着写一篇博客来收集和介绍关于Stream流的各种用法，也算是一种以终为始的学习。

## Stream流简介

​	Stream流是Java 8引入的一种新的抽象概念，用于处理集合数据。它提供了一种高效且易于使用的方式来对集合进行各种操作，如过滤、映射、排序、归约等。Stream流可以看作是对集合数据进行操作的一种管道，它允许开发者以一种声明式的方式来处理数据，而不是通过传统的命令式编程方式。

Stream流有以下主要特点：
1. **声明式编程**：Stream流允许开发者以一种更接近自然语言的方式来描述数据处理逻辑，而不是通过一系列的循环和条件判断来实现。
2. **惰性求值**：Stream 流的操作分为中间操作和终端操作。中间操作（如filter、map）不会立即执行，而是等到终端操作（如collect、forEach）被调用时才会执行。这种惰性求值的特性可以提高性能，尤其是在处理大数据集时。
3. **并行处理**：Stream 流支持并行处理，通过调用 parallelStream()方法，可以将数据处理任务分配到多个线程中执行，从而提高处理速度。
4. **丰富的操作方法**：Stream 流提供了丰富的操作方法，如 filter、map、reduce、collect、sorted、distinct 等，可以满足各种数据处理需求。

## 获取流的方式

在 Java 中，获取 Stream 流的方式有多种，具体取决于你要处理的源数据类型。以下是一些常见的获取 Stream 流的方式

### 从集合中获取流

Java中的集合类（如 List、Set、Map等）提供了stream() 方法，可以直接获取Stream流。

~~~ java
public class StreamExample {
    public static void main(String[] args) {
        // 从 List 获取 Stream 流
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        Stream<String> nameStream = names.stream();

        // 从 Set 获取 Stream 流
        Set<Integer> numbers = new HashSet<>(Arrays.asList(1, 2, 3, 4, 5));
        Stream<Integer> numberStream = numbers.stream();

        // 从 Map 获取 Stream 流（通过 entrySet）
        Map<String, Integer> map = new HashMap<>();
        map.put("Alice", 25);
        map.put("Bob", 30);
        Stream<Map.Entry<String, Integer>> mapStream = map.entrySet().stream();
    }
}
~~~

### 从数组中获取流

Java提供了 Arrays.stream() 方法，可以从数组获取Stream流。

~~~ java
public class StreamExample {
    public static void main(String[] args) {
        // 从数组获取 Stream 流
        String[] names = {"Alice", "Bob", "Charlie"};
        Stream<String> nameStream = Arrays.stream(names);

        // 从数组获取 Stream 流（指定范围）
        int[] numbers = {1, 2, 3, 4, 5};
        IntStream numberStream = Arrays.stream(numbers, 1, 4); // 获取索引 1 到 3 的元素
    }
}
~~~

### 使用Stream的静态工厂方法

Java 提供了一些静态工厂方法来创建Stream流，如 Stream.of()、Stream.iterate()、Stream.generate() 等

~~~ java
public class StreamExample {
    public static void main(String[] args) {
        // 使用 Stream.of() 创建 Stream 流
        Stream<String> nameStream = Stream.of("Alice", "Bob", "Charlie");

        // 使用 Stream.iterate() 创建无限流
        Stream<Integer> evenNumbers = Stream.iterate(0, n -> n + 2);

        // 使用 Stream.generate() 创建无限流
        Stream<Double> randomNumbers = Stream.generate(Math::random);
    }
}
~~~

其中，无限流是指那些没有固定大小的流，它们可以无限地生成元素。在Java中，无限流通常通过 Stream.iterate() 或 Stream.generate()方法创建。无限流的特点是它们不会自动终止，因此在使用时需要特别注意，通常需要结合 limit()方法来限制生成的元素数量，以避免无限循环或内存溢出。

### 从文件中获取流

可以使用 Files.lines()方法从文件中获取 Stream流，逐行读取文件内容

~~~ java
public class StreamExample {
    public static void main(String[] args) {
        try (Stream<String> lines = Files.lines(Paths.get("example.txt"))) {
            lines.forEach(System.out::println);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
~~~

###  从字符串获取流

可以使用 Pattern.splitAsStream() 方法从字符串中获取Stream流，将字符串按正则表达式分割

~~~ java
public class StreamExample {
    public static void main(String[] args) {
        String text = "Hello, World! This is a test.";
        Stream<String> words = Pattern.compile("\\W+").splitAsStream(text);
        words.forEach(System.out::println);
    }
}
~~~

## Stream流的操作方法

在 Java中，Stream流提供了丰富的操作方法，这些方法可以分为两大类：**中间操作**（Intermediate Operations）和**终端操作**（Terminal Operations）。中间操作返回一个新的Stream，而终端操作则触发流的处理并返回最终结果。

### 中间操作

- **filter(Predicate\<T>)**:根据给定的条件过滤流中的元素

  ~~~ java
  Stream<Integer> filteredStream = Stream.of(1, 2, 3, 4, 5).filter(n -> n % 2 == 0);
  ~~~

- **map(Function\<T, R>)**: 将流中的每个元素映射为另一种类型

  ~~~ java
  Stream<String> nameLengths = Stream.of("Alice", "Bob", "Charlie").map(String::length);
  ~~~

- **flatMap(Function\<T, Stream\<R>>)**:将流中的每个元素映射为一个流，并将所有流合并为一个流

  ~~~ java
  Stream<String> words = Stream.of("Hello World", "Java Stream")
                               .flatMap(line -> Arrays.stream(line.split(" ")));
  ~~~

- **distinct()**:去除流中的重复元素

  ~~~ java
  Stream<Integer> distinctNumbers = Stream.of(1, 2, 3, 2, 1).distinct();
  ~~~

- **sorted()**：对流中的元素进行排序

  ~~~ java
  Stream<Integer> sortedNumbers = Stream.of(5, 3, 1, 4, 2).sorted();
  ~~~

- **peek(Consumer\<T>)**：对流中的每个元素执行给定的操作，主要用于调试。

  ~~~ java
  Stream<Integer> peekedStream = Stream.of(1, 2, 3).peek(System.out::println);
  ~~~

- **limit(long maxSize)**：截取流中的前maxSize个元素

  ~~~ java
  Stream<Integer> limitedStream = Stream.iterate(0, n -> n + 1).limit(10);
  ~~~

- **skip(long n)**：跳过流中的前n个元素

  ~~~ java
  Stream<Integer> skippedStream = Stream.of(1, 2, 3, 4, 5).skip(2)
  ~~~

### 终端操作

- **forEach(Consumer\<T>)**：对流中的每个元素执行给定的操作

  ~~~ java
  Stream.of("Alice", "Bob", "Charlie").forEach(System.out::println);
  ~~~

- **collect(Collector<T, A, R>)**：将流中的元素收集到一个集合中

  ~~~ java
  List<String> names = Stream.of("Alice", "Bob", "Charlie").collect(Collectors.toList());
  ~~~

- **toArray()**：将流中的元素转换为数组

  ~~~ java
  String[] namesArray = Stream.of("Alice", "Bob", "Charlie").toArray(String[]::new);
  ~~~

- **reduce(BinaryOperator\<T>)**：将流中的元素归约为一个值

  ~~~ java
  Optional<Integer> sum = Stream.of(1, 2, 3, 4, 5).reduce((a, b) -> a + b);
  ~~~

- **min(Comparator\<T>)**：返回流中的最小元素

  ~~~ java
  Optional<Integer> min = Stream.of(5, 3, 1, 4, 2).min(Integer::compareTo);
  ~~~

- **max(Comparator\<T>)**：返回流中的最大元素

  ~~~ java
  Optional<Integer> max = Stream.of(5, 3, 1, 4, 2).max(Integer::compareTo);
  ~~~

- **count()**：返回流中的元素数量

  ~~~ java
  long count = Stream.of("Alice", "Bob", "Charlie").count();
  ~~~

- **anyMatch(Predicate\<T>)**：判断流中是否存在满足给定条件的元素

  ~~~ java
  boolean hasEven = Stream.of(1, 2, 3, 4, 5).anyMatch(n -> n % 2 == 0);
  ~~~

- **allMatch(Predicate\<T>)**：判断流中的所有元素是否都满足给定条件

  ~~~ java
  boolean allPositive = Stream.of(1, 2, 3, 4, 5).allMatch(n -> n > 0);
  ~~~

- **noneMatch(Predicate\<T>)**：判断流中是否不存在满足给定条件的元素

  ~~~ java
  boolean noneNegative = Stream.of(1, 2, 3, 4, 5).noneMatch(n -> n < 0);
  ~~~

- **findFirst()**：返回流中的第一个元素

  ~~~ java
  Optional<String> firstName = Stream.of("Alice", "Bob", "Charlie").findFirst();
  ~~~

- **findAny()**：返回流中的任意一个元素

  ~~~ java
  Optional<String> anyName = Stream.of("Alice", "Bob", "Charlie").findAny();
  ~~~

- **concat()**：接受两个Stream作为参数，并返回一个新的Stream，其中包含两个输入Stream的所有元素

  ~~~ java
  Stream<String> combinedStream = Stream.concat(stream1, stream2);
  ~~~

### flatMap详解

​	上述基本上就是Stream接口中定义的所有方法，我认为大部分方法都是可以根据描述来判断其用法的，可能最为抽象的就属flatMap这个方法了，其方法签名如下：
~~~ java
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)
~~~

- mapper 是一个函数，它接受一个元素并返回一个 Stream
- flatMap方法将每个元素应用 mapper 函数，然后将所有生成的 Stream 合并成一个单一的 Stream

假设我们有一个包含多个列表的列表，我们希望将这些列表合并成一个单一的列表，就可以使用flatMap来实现这一点：

~~~ java
public class FlatMapExample {
    public static void main(String[] args) {
        // 创建一个包含多个列表的列表
        List<List<String>> listOfLists = Arrays.asList(
            Arrays.asList("Apple", "Banana"),
            Arrays.asList("Cherry", "Date"),
            Arrays.asList("Elderberry", "Fig")
        );

        // 使用 flatMap 将嵌套的列表扁平化
        List<String> flatList = listOfLists.stream()
            .flatMap(List::stream)  // 将每个列表转换为 Stream，然后合并
            .collect(Collectors.toList());
    }
}
~~~

![image-20240913204203112](/images/posts/Java/javaStream/image-20240913204203112.png)

简而言之，如果以后你的数据结构中有多个嵌套的结构，那你可以尝试用flatMap把它推平来处理。
