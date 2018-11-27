---
title: Java核心技术2
date: 2018-11-26 15:07:45
tags:
categories: Java
---

# Java SE8的流库

## 从迭代到流的操作
1. 流并不存储其元素。这些集合可能存储在底层的集合中，或者是按需生成的。
1. 流的操作不会修改其数据源。
1. 流的操作是尽可能惰性执行的。这意味着直至需要其结果时，操作才会执行。

## 流的创建
1. 集合：stream、parallelStream。
1. 数组：Arrays.stream、Stream.of。
1. 无限流：Stream.generate、Stream.iterate。
1. 空流：Stream.empty。
1. 正则：splitAsStream。
1. 文件：Files.lines。

## filter、map和flatMap方法
1. filter：的参数是Predicate<T>，从T到boolean的函数。
1. map：一对一映射。
1. flatMap：是一对多映射，需要定义映射成多个流的方式。

## 抽取子流和连接流
1. limit：抽取前面的子流，常用于裁剪无限流。
1. skip：丢弃前面的子流，抽取后面的子流。
1. concat：连接流。

## 其他的流转换
1. distinct：剔除重复元素。
1. sorted：排序。

```Java
Stream<String> s;
s.sorted(Comparator.comparing(String::length).reversed())
```

3. peek：复制流，并通过函数参数遍历，常用于调试。

## 简单约简
1. 约简是一种终结操作。
1. count：计数。
1. max、min：最大值和最小值。
1. findFirst、findAny：第一个值和任意匹配的值，findAny常用于并行处理。
1. anyMatch、allMatch和noneMatch：断言匹配。

## Optional类型
1. get：如果不存在，抛出异常。
1. orElse、orElseGet、orElseThrow：如果不存在，使用默认操作。
1. ifPresent：如果存在，执行操作。
1. map：类似ifPresent，有返回值。
1. flatMap：类似map，不会多重可选包装。

**创建Optional值**
1. Optional.empty：空。
1. Optional.of：如果为null，抛出异常。
1. Optional.ofNullable：可以是null。

## 收集结果
1. iterator、forEach、forEachOrdered：遍历。
1. toArray：获得由流元素组成的数组。
1. collect：收集到另一个目标中，Collectors提供了大量收集方法。

```Java
List<String> result=stream.collect(Collectors.toList());
Set<String> result=stream.collect(Collectors.toSet());
TreeSet<String> result=stream.collect(Collectors.toCollection(TreeSet::new));

String result=stream.collect(Collectors.joinning());
String result=stream.collect(Collectors.joining(","));

//约简
IntSummaryStatistics summary=stream.collect(Collectors.summarizingInt(String::length));
double average=summary.getAverage();
double max=summary.getMax();
```

## 收集到映射表中
1. Collectors.toMap(key,value)，将元素收集到映射表中。
1. Function.identity()，x->x。

## 群组和分区
1. Collectors.groupingBy：分组，生成映射表，值为列表。
1. Collectors.partitioningBy：分区，类似分组，键值是true或false。

## 下游收集器
1. 用于处理groupingBy生成的映射表的列表值的Collectors，和collect的收集器本质一样。
1. counting、summing、maxBy、minBy。
1. mapping：先映射，再执行下游收集器。

## 约简操作
1. reduce。
1. 提供者、累积器、组合器模式：提供者创建目标类型的新实例，累积器添加元素到实例，组合器合并实例。

## 基本类型流
1. IntStream、LongStream、DoubleStream。
1. range、rangeClosed：生成步长为1的整数范围。
1. codePoints和chars会生成IntStream。
1. Random类具有ints、longs和doubles方法，会返回基本类型流。
1. boxed可以包装基本类型流。

**差异**
1. toArray方法会直接返回基本类型数组。
1. 具有max、min、sum、average方法，方法会直接返回结果。
1. summaryStatistics也会直接返回结果，而不需要定义参数。

## 并行流
1. 只要在终结方法执行时，流处于并行模式，那么所有的中间流操作都将被并行化。
1. 并行处理可以导致竞争情况，要避免线程不安全时使用并行模式。
1. 默认情况下，有序集合、范围、生成器和迭代产生的流，或者通过Stream.sorted产生的流，都是有序的，它们的结果是按照原来元素的顺序累积的，如果运行相同的操作两次，将会得到相同的结果。
1. 当在流上调用unordered方法时，有些操作可以被更有效地并行化。在有序的流中，distinct会保留所有相同元素中的第一个，这对并行化是一种阻碍。放弃排序还可以提高limit的速度，从流中取出任意n个元素，而不在意要获取哪些。
1. 流操作都是惰性的，在终结操作前对集合进行修改是可行的，但是并不推荐。

**并行流正常工作的条件**
1. 数据应该在内存中。必须等到数据到达是非常低效的。
1. 流应该可以被高效地分成若干个子部分。
1. 流操作的工作量应该具有较大的规模。
1. 流操作不应该被阻塞。并行流使用fork-join池来操作池的各个部分，如果多个流操作被阻塞，那么池可能就无法做任何事情了。