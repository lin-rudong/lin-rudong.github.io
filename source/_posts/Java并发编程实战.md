---
title: Java并发编程实战
date: 2018-10-25 09:26:45
tags:
categories: Java
---

# 简介

## 并发的原因
1. 资源利用率。在某些情况下，程序必须等待某个外部操作执行完成，例如IO，而在等待时程序无法执行其他任何工作。如果在等待的同时可以运行另一个程序，可以提高资源的利用率。
1. 公平性。不同的用户和程序对于计算机上的资源用着同等的使用权，可以通过时间分片实现。
1. 便利性。在执行多个任务时，应该编写多个程序，每个程序执行一个任务并在必须时互相通信，这比编写一个程序来执行所有任务更容易实现。

## 线程的优势
1. 发挥多处理器的强大能力。
1. 建模的简单性。
1. 异步事件的简化处理。
1. 响应更灵敏的用户界面。

## 线程带来的风险
1. 安全性问题。数据共享简化线程间通信的同时，也带来了巨大的风险。
1. 活跃性问题。活跃性是指某个操作无法继续执行下去，比如死锁。
1. 性能问题。

# 基础知识

## 线程安全性

**多线程数据访问出错修复方法**
1. 不共享变量。
1. 将变量修改为不可变。
1. 访问变量时使用同步。

### 线程安全性
1. 无状态对象一定是线程安全的。

```
public class MyServlet implements Servlet{
    public void service(ServletRequest request,ServletResponse response){
        ...
    }
}
```

### 原子性
1. 竞态条件，由于执行时序而出现不正确的结果。最常见的竞态条件是，先检查后执行。
1. 复合操作，java.util.concurrent.atomic包含的原子变量类的访问操作都是原子的。

### 加锁机制
1. 内置锁，Java提供同步代码块来支持原子性，同步代码块包括2部分，1个作为锁的对象引用，1个作为由这个锁保护的代码块。synchronized修饰的方法就是一种横跨整个方法体的同步代码块，该同步代码块的锁就是方法调用所在的对象，静态的则是以Class对象作为锁。
1. Java的内置锁相当于互斥锁。由于每次只能有一个线程执行同步代码块，所以会以原子方式执行，即一组语句作为一个不可分割的单元被执行。
1. Java的内置锁是可以重入的，重入意味着获取锁的操作粒度是线程而不是调用。子类的覆盖方法也会获得父类锁，重入可以避免这种死锁发生。

### 用锁来保护状态
1. 如果用同步来协调对某个变量的访问，那么在访问这个变量地所有位置上都需要使用同步。
1. 只有被多个线程同时访问的可变数据才需要通过锁来保护。

### 活跃性与性能
1. 当执行时间较长的计算或者可能无法快速完成的操作时，一定不要持有锁。

## 对象的共享

### 可见性
1. 通常，我们无法确保执行读操作的线程能适时地看到其他线程写入的值。
1. 编译器可能会对操作的执行顺序进行重排序优化，这种重排序只能保证在单线程中有正确的结果。
1. 非volatile类型的64位数值变量double和long的读写操作会分解成2个32位操作，不是原子操作，这可能读取到某个值的高32位和另一个值的低32位。
1. 加锁的含义不仅仅局限于互斥行为，还包括内存可见性。
1. volatile变量操作不会和其他内存操作一起重排序，volatile变量会同步到主内存，所以读取volatile变量时总会返回最新写入的值。

**使用volatile变量必须满足以下所有条件**
1. 访问变量时不需要加锁。
1. 对变量的写入操作不依赖变量的当前值，或者你能确保只有单个线程更新变量的值。
1. 该变量不会与其他状态变量一起纳入不变性条件中。

### 发布与逸出
1. 发布一个对象是指对象能够在当前作用域之外的代码中使用。多线程发布会破坏封装性，使得程序难以维持不变性条件，比如，对象构造完成之前就发布该对象。逸出是指当某个不应该发布的对象被发布。
1. 使用工厂方法来防止this引用在构造过程中逸出。

### 线程封闭
1. 线程封闭是指仅在单线程内访问数据，不共享数据就不需要同步。
1. Swing的对象都不是线程安全的，应保证只有事件线程才能访问对象，使用单线程子系统的另一个原因是为了避免死锁。

**线程封闭的3种方式**
1. Ad-hoc线程封闭是指维护线程封闭性的职责完全由程序实现来承担，Ad-hoc线程封闭是非常脆弱的，因为没有任何一种语言特性能将对象封闭到目标线程上，所以在程序中尽量少用它。在可能的情况下，应该使用更强的线程封闭技术，比如栈封闭或者ThreadLocal类。当决定使用线程封闭技术时，通常是因为要将某个特定的子系统实现为一个单线程子系统。
1. 栈封闭是指只能通过局部变量才能访问对象。
1. 维持线程封闭性的一种更规范方法是使用ThreadLocal，通常用于防止对可变的单实例变量或全局变量进行共享。概念上看，可以将Thread Local<T>视为包含Map<Thread,T>，其中保存了特定与该线程的值，不过实现上是将特定于线程的值保存在Thread对象中，当线程结束后这些值会被GC回收。

### 不变性
1. 对象创建以后其状态就不能修改。
1. 对象的所有字段都是final类型。
1. 对象被正确创建。在对象的创建期间，this引用没有逸出。

### 安全发布

**初始化安全性**
1. 要安全地发布一个对象，对象的引用以及对象的状态必须同时对其他线程可见。即使某个对象的引用对其他线程是可见的，也并不意味着对象状态对于其他线程一定是可见的。
1. 不可变对象有一种特殊的初始化安全性保证，发布不可变对象的引用时即使没有同步，也仍然可以安全地访问对象。这种保证还将延申到被正确创建对象中的所有final类型的字段，在没有额外同步的情况下，也可以安全地访问final类型的字段，但不能保证可以访问final类型字段的状态。
1. 最简单和最安全的方式是使用静态初始化器，静态初始化器由JVM在类的初始化阶段执行，由于JVM内部存在着同步机制，所以可以被安全地发布。
1. 将一个键或者值放入并发安全容器中，可以安全地将它发布给任何从这些容器中访问它的线程。

**安全发布方式**
1. 在静态初始化函数中初始化一个对象引用。
1. 将对象的引用保存到volatile类型的字段或者AtomicReferance对象中。
1. 将对象的引用保存到某个正确构造对象的final类型字段中。
1. 将对象的引用保存到一个由锁保护的字段中。

**发布需求取决于可变性**
1. 不可变对象可以通过任意机制来发布。
1. 事实不可变对象必须通过安全方式来发布。
1. 可变对象必须通过安全方式来发布，并且必须是线程安全的或者由某个锁保护起来。

**安全地共享对象**
1. 线程封闭。
1. 只读共享。
1. 线程安全共享。
1. 保护对象。

## 对象的组合

### 设计线程安全的类

**3个基本要素**
1. 找出构成对象状态的所有变量。
1. 找出约束状态变量的不变性条件。
1. 建立对象状态的并发访问管理策略。




