---
title: Effective Java
categories: Java
abbrlink: 60662
date: 2018-10-09 08:45:37
tags:
---
# 创建和销毁对象
## 考虑用静态工厂方法代替构造器
### 优点
1. 有名称,能确切地描述正被返回的对象。
1. 不必在每次调用的时候都创建一个新对象,可以控制实例实现Singleton或者使得不可变类能使用==操作符代替equals方法，从而提升性能。
1. 可以返回原返回类型的任何子类型对象,可以隐藏底层的实现使API变得简洁。  
    * 静态工厂方法是服务提供者框架的基础，服务提供者框架包含3个组件：服务接口、提供者注册API和服务访问API,除此之外还有第4个可选的组件，服务提供者接口，默认是通过反射按照服务类名实例化。
    * 对于JDBC来说，Connection是服务接口，DriverManager.registerDriver是提供者注册API，DriverManager.getConnection是服务访问API以及Driver就是服务提供者接口。
1. 创建参数化类型实例的时候，可以实现类型推导。
```
public class HashMap{
    public static <K,V> HashMap<K,V> newInstance(){
        return new HashMap<K,V>();
    }
}

Map<String,List<String>> map=HashMap.newIntstance();
```
### 缺点
1. 类如果不含public或protected的构造器，就不能子类化。
1. 静态工厂方法和其他静态方法没有区别，在API文档中无法明确标志出来。常用名称：
    * valueOf/of
    * getInstance/newInstance
    * getType/newType
## 遇到多个构造器参数时要考虑用构建器
1. 重叠构造器模式
```
public class Student{
    private String id;
    private String name;
    private int age;

    public Student(String id){
        this(id,null,0);
    }
    public Student(String id,String name){
        this(id,name,0);
    }
    public Student(String id,String name,int age){
        this.id=id;
        this.name=name;
        this.age=age;
    }
}
```
    * 当参数过多时，代码会变得难编写，并且较难阅读。
2. JavaBeansm模式
    * 通过setter设置参数，阅读容易，但构造过程被分割导致JavaBean在构造过程中可能处于不一致的状态。另外，JavaBeans模式也阻止了把类变成不可变的可能。
3. Builder模式
    * 既保证了安全性，也有很好的可读性。
```
public class Student{
    private final String id;
    private final String name;
    private final int age;

    private Student(Builder builder){
        id=builder.id;
        name=builder.name;
        age=builder.age;
    }

    public static class Builder{
        private String id;
        private String name;
        private int age;

        public Builder id(String id){
            this.id=id;return this;
        }
        public Builder name(String name){
            this.name=name;return this;
        }
        public Builder age(int age){
            this.age=age;return this;
        }

        public Student build(){
            return new Student(this);
        }
    }
}

Student student=new Student.Builder().id("id").name("name").age(1).build();
```



