---
title: JDK8新特性
tags: JDK8
categories: Java
abbrlink: 64019
date: 2018-10-07 22:02:53
---
# Java语言

## Lambda表达式与Functional接口
* 访问局部变量时，会将变量隐式转换成常量。
* 接口的静态方法和默认方法不影响函数式接口。
* `@FunctionalInterface`。

## 方法引用
* 方法引用用来支持Lambda简写。
* 构造器引用`T::new`，构造器本质上静态方法。
* 静态方法引用`T::staticMethod`。
* 类的任意对象的方法引用`T::Method`，第一个参数是对象。
* 对象的方法引用`O::Method`。

## 接口的静态方法与默认方法
* 接口的字段是*public static final*,接口的方法是*public*。
* 接口的方法新增*static*与*default*。

## 类型推测机制
* List<T> list=new List<>();

## 重复注解
* `@Repeatable`。

## 注解类型
* field、constructor、method、parameter、local_variable、package和type(类、接口和enum)。

# Java编译器

## 参数名字
* 参数名字保存在Java字节码中，`javac -parameters`。

## Java类库

## Stream
* 集合函数式编程

## 并行数组
* 数组并行处理。

## 并发
* ConcurrentHashMap。
    1. `取消段segments`，采用transient volatile hashEntry保存数据，实现`对每一行数据进行加锁`代替原来的段加锁，减少并发冲突。
    2. 数据结构改为`数组+单向链表+红黑树`。
* ForkJoinPool支持共有资源池。
* 容量锁StampedLock，代替ReadWriteLock。
* atomic增加：
    1. DoubleAccumulator
    2. DoubleAdder
    3. LongAccumulator
    4. LongAdder

## Optional

## Date/Time API
1. Instance Duration
1. LocaleDate LocalTime LocalDateTime
1. ZonedDateTIme

## JS引擎
```
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName( "JavaScript" );
engine.eval( "function f() { return 1; }; f() + 1;" );
```

## Base64
* 编码。

# JVM

## 元空间Metaspace代替永久代PermGen
* 字符串和静态变量转移到Java堆。
* 元空间使用本地内存而不在JVM中，元空间受本地内存限制，可以用-XX:MetaspaceSize -XX:MaxMetaspaceSize限制。

