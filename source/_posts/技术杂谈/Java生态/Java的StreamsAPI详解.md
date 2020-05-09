---
title: Java的StreamsAPI详解
toc: true
date: 2020-04-20 11:30:19
tags:
categories:
---

Stream 作为 Java 8 的一大亮点，它与 java.io 包里的 InputStream 和 OutputStream 是完全不同的概念。它也不同于 StAX对 XML 解析的 Stream，也不是 Amazon Kinesis 对大数据实时处理的 Stream。Java 8 中的 Stream是对集合（Collection）对象功能的增强，它专注于对集合对象进行各种非常便利、高效的聚合操作（aggregate operation），或者大批量数据操作(bulk data operation)。Stream API 借助于同样新出现的 Lambda表达式，极大的提高编程效率和程序可读性。同时它提供串行和并行两种模式进行汇聚操作，并发模式能够充分利用多核处理器的优势，使用 fork/join并行方式来拆分任务和加速处理过程。通常编写并行代码很难而且容易出错, 但使用 Stream API无需编写一行多线程的代码，就可以很方便地写出高性能的并发程序。所以说，Java 8 中首次出现的 java.util.stream是一个函数式语言+多核时代综合影响的产物。

Stream 不是集合元素，它不是数据结构并不保存数据，它是有关算法和计算的，它更像一个高级版本的 Iterator。原始版本的Iterator，用户只能显式地一个一个遍历元素并对其执行某些操作；高级版本的 Stream，用户只要给出需要对其包含的元素执行什么操作，比如 “过滤掉长度大于 10 的字符串”、“获取每个字符串的首字母”等，Stream 会隐式地在内部进行遍历，做出相应的数据转换。

Stream 就如同一个迭代器（Iterator），单向，不可往复，数据只能遍历一次，遍历过一次后即用尽了，就好比流水从面前流过，一去不复返。而和迭代器又不同的是，Stream 可以并行化操作，迭代器只能命令式地、串行化操作。顾名思义，当使用串行方式去遍历时，每个 item 读完后再读下一个                 item。而使用并行去遍历时，数据会被分成多个段，其中每一个都在不同的线程中处理，然后将结果一起输出。Stream 的并行操作依赖于 Java7 中引入的Fork/Join 框架（JSR166y）来拆分任务和加速处理过程。Java 的并行 API 演变历程基本如下：

- 1.0-1.4 中的 java.lang.Thread
- 5.0 中的 java.util.concurrent
- 6.0 中的 Phasers 等
- 7.0 中的 Fork/Join 框架
- 8.0 中的 Lambda

Stream 的另外一大特点是，数据源本身可以是无限的。



## 生成流的方式

从 Collection 和数组

-  Collection.stream()
-  Collection.parallelStream()
-  Arrays.stream(T array) or Stream.of()



从 BufferedReader

- java.io.BufferedReader.lines()

静态工厂

- java.util.stream.IntStream.range()
- java.nio.file.Files.walk()

自己构建

-  java.util.Spliterator

其它

  -  Random.ints()
  -  BitSet.stream()
  -  Pattern.splitAsStream(java.lang.CharSequence)
  -  JarFile.stream()

需要注意的是，对于基本数值型，目前有三种对应的包装类型 Stream：

IntStream、LongStream、DoubleStream。当然我们也可以用 Stream<Integer>、Stream<Long>>、Stream<Double>，但是 boxing 和 unboxing 会很耗时，所以特别为这三种基本数值型提供了对应的 Stream。

Java 8 中还没有提供其它数值型 Stream，因为这将导致扩增的内容较多。而常规的数值型聚合运算可以通过上面三种 Stream 进行。如下数值流的构造：

```
IntStream.of(new int[]{1, 2, 3}).forEach(System.out::println);
IntStream.range(1, 3).forEach(System.out::println);
IntStream.rangeClosed(1, 3).forEach(System.out::println);
```



## 函数类型

对函数式编程支持程度高低的一个重要特征是函数是否作为编程语言的一等公民出现，也就是编程语言是否有内置的结构来表示函数。作为面向对象的编程语言，Java 中使用接口来表示函数。直到
Java 8，Java 才提供了内置标准 API 来表示函数，也就是 java.util.function 包。Function<T,R> 表示接受一个参数的函数，输入类型为 T，输出类型为 R。Function 接口只包含一个抽象方法 R apply(T t)，也就是在类型为 T 的输入 t上应用该函数，得到类型为 R 的输出。除了接受一个参数的 Function 之外，还有接受两个参数的接口 BiFunction<T, U, R>，T 和 U分别是两个参数的类型，R 是输出类型。BiFunction 接口的抽象方法为 R apply(T t, U u)。超过 2 个参数的函数在 Java 标准库中并没有定义。如果函数需要3 个或更多的参数，可以使用第三方库，如 Vavr 中的 Function0 到 Function8。

除了 Function 和 BiFunction 之外，Java 标准库还提供了几种特殊类型的函数：

- Function<T,R> 
- BiFunction<T, U, R>
- Consumer<T>：接受一个输入，没有输出。抽象方法为 void accept(T t)。
- Supplier<T>：没有输入，一个输出。抽象方法为 T get()。
- Predicate<T>：接受一个输入，输出为 boolean 类型。抽象方法为 boolean test(T t)。
- UnaryOperator<T>：接受一个输入，输出的类型与输入相同，相当于 Function<T, T>。
- BinaryOperator<T>：接受两个类型相同的输入，输出的类型与输入相同，相当于 BiFunction<T,T,T>。
- BiPredicate<T, U>：接受两个输入，输出为 boolean 类型。抽象方法为 boolean test(T t, U u)。



## 流的操作类型

当把一个数据结构包装成 Stream 后，接下来，就要开始对里面的元素进行各类操作了。常见的操作可以归类如下两种：

- **Intermediate**：一个流可以后面跟随零个或多个 intermediate操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。
- **Terminal**：一个流只能有一个 terminal 操作，当这个操作执行后，流就被使用“光”了，无法再被操作。所以这必定是流的最后一个操作。Terminal 操作的执行，才会真正开始流的遍历，并且会生成一个结果，或者一个 side effect。
- **short-circuiting**:  当操作一个无限大的 Stream，而又希望在有限时间内完成操作，则在管道内拥有一个 short-circuiting 操作是必要非充分条件。对于一个 intermediate 操作，如果它接受的是一个无限大的 Stream，使用该操作将返回一个有限的新Stream。对于一个 terminal 操作，如果它接受的是一个无限大的 Stream，但能在有限的时间计算出结果。



在对于一个 Stream 进行多次转换操作 (Intermediate 操作)，每次都对 Stream 的每个元素进行转换，而且是执行多次，这样时间复杂度就是N（转换次数）个 for 循环里把所有操作都做掉的总和吗？其实不是这样的，转换操作都是 lazy 的，多个转换操作只会在 Terminal操作的时候融合起来，一次循环完成。我们可以这样简单的理解，Stream 里有个操作函数的集合，每次转换操作就是把转换函数放入这个集合中，在 Terminal操作的时候循环 Stream 对应的集合，然后对每个元素执行所有的函数。



 Intermediate操作

- map：参数是Function<T,R>类型，把一个元素类型为 T 的流转换成元素类型为 R 的流
- mapToInt
- flatMap：通过一个 Function 把一个元素类型为 T 的流中的每个元素转换成一个元素类型为 R 的流，再把这些转换之后的流合并
- filter：参数是Predicate ，过滤流中的元素，只保留满足所指定的条件的元素
- distinct：使用 equals 方法来删除流中的重复元素
- sorted：对流进行排序
- peek：返回的流与原始流相同。当原始流中的元素被消费时，会首先调用 peek 方法中指定的 Consumer 实现对元素进行处理
- limit：截断流使其最多只包含指定数量的元素
- skip：返回一个新的流，并跳过原始流中的前 N 个元素
- parallel
- equential
- unordered
- dropWhile（Java9新增）：从原始流起始位置开始删除满足指定 Predicate 的元素，直到遇到第一个不满足 Predicate 的元素。
- takeWhile（Java9新增）：从原始流起始位置开始保留满足指定 Predicate 的元素，直到遇到第一个不满足 Predicate 的元素。

Terminal操作

- forEach
- forEachOrdered
- toArray
- reduce
- collect：把结果收集到可变的容器中，如 List 或 Set。收集操作通过接口java.util.stream.Collector 来实现。Java 已经在类 Collectors 中提供了很多常用的 Collector 实现。
- min：流中元素的最小值
- max：流中元素的最大值
- count
- anyMatch
- allMatch
- noneMatch
- findFirst：查找流中的第一个
- findAny：查找流中的任意一个元素
- iterator

Short-circuiting操作：anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 limit

## 参考资料
> - [https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/]()
> - []()

