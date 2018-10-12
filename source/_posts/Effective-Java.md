---
title: Effective Java
categories: Java
date: 2018-10-09 08:45:37
tags:
---

# 创建和销毁对象

## 考虑用静态工厂方法代替构造器

**优点**
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

**缺点**
1. 类如果不含public或protected的构造器，就不能子类化。
1. 静态工厂方法和其他静态方法没有区别，在API文档中无法明确标志出来。常用名称：
    * valueOf/of
    * getInstance/newInstance
    * getType/newType

## 遇到多个构造器参数时要考虑用构建器

**重叠构造器模式**
1. 当参数过多时，代码会变得难编写，并且较难阅读。

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

**JavaBeansm模式**
1. 通过setter设置参数，阅读容易，但构造过程被分割导致JavaBean在构造过程中可能处于不一致的状态。另外，JavaBeans模式也阻止了把类变成不可变的可能。

**Builder模式**
1. 既保证了安全性，也有很好的可读性。

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

## 用私有构造器或者枚举类型强化Singleton属性

**饿汉模式**
1. 没有限制的话，可以通过反射调用私有构造器。
1. 由于反序列化时会创建一个新的实例，所以可序列化的Singleton类的实例字段必须都是transient。

```
public class Student{
    private static final Student INSTANCE=new Student();

    private Student(){}
    public static Student getInstance(){return INSTANCE;}
}
```

**单个元素的枚举类型**
1. 无偿地提供序列化机制，绝对防止多次实例化。
1. 单元数的枚举类型是实现Singleton的最佳方法。

## 通过私有构造器强化不可实例化的能力
1. 私有构造器不仅不能实例化，还不可以继承，可以防止子类实例化。

## 避免创建不必要的对象
1. 要优先使用基本类型而不是装箱基本类型，要当心无意识的自动装箱。
1. 适配器除了后被对象之外，没有其他状态信息，所以不需要创建多个适配器实例。

## 消除过期的对象引用
1. 内存泄漏极端情况下会导致磁盘交换，甚至会导致OutOfMemoryError。
1. 清空对象引用应该是一种例外，而不是一种规范行为。
1. 只要类是自己管理内存，就应该警惕内存泄漏问题。
1. 内存泄漏的另一个常见来源是缓存。如果回调没有显式取消注册，它们就会积聚。
1. 内存泄漏的第三个常见来源是监听器和其他回调。

## 避免使用终结方法
1. 终结方法finalizer通常是不可预测的，也是危险的，一般情况下是不必要的。终结方法的缺点在于不能保证会被及时地执行，而且根本那就补保证它们会被执行。
1. 如果未被捕获的异常在终结过程中被抛出来，那么异常会被忽略，终结过程也会终止。
1. 使用终结方法会有非常严重的性能损失。
1. 普通对象通过本地方法委托给本地对象，当普通对象回收时，GC不会发现本地对等体，如果本地对等体不拥有关键资源，则可以用终结方法回收。
1. 子类的终结方法不会自动调用超类的终结方法。

# 对于所有对象都通用的方法
1. Object的非final方法：equals、hashCode、toString、clone和finalize。

## 覆盖equals时请遵守通用约定

**满足以下任何一个条件可以不覆盖equals方法**
1. 类的每个实例本质上都是唯一的。
1. 不关心类是否提供了“逻辑相等”的测试功能。
1. 超类已经覆盖了equals，从超类继承过来的行为对于子类也是合适的。
1. 类的私有的或是包级私有的，可以确定它的equals方法永远不会被调用。

**equals方法实现了等价关系**
1. 自反性。对于任何非null的引用值x，x.quals(x)必须返回true。
1. 对称性。对于任何非null的引用值x和y，当且仅当y.equals(x)返回true时，x.equals(y)必须返回true。
1. 传递性。对于任何非null的引用值x、y和z，如果x.equals(y)返回true，并且y.equals(z)也返回true，那么x.equals(z)也必须返回true。
1. 一致性。对于任何非null的引用值x和y，只要equals的比较操作在对象中所用的信息没有被修改，多次调用x.equals(y)就会一致地返回true，或者一致地返回false。
1. 对于任何非null的引用值x，x.equals(null)必须返回false。

**等价与继承不可兼得**
1. 无法在扩展可实例化的类的同时，既增加新的值组件，同时又保留equals约定。
1. getClass可以完全隔离超类和子类，可以解决等价关系，但会导致超类的一些方法不能用于子类，破坏了继承关系。instanceof反之。
1. 可以用复合代替继承，需要扩展值组件的类可以包含原类，并提供一个获得原类的视图方法。还有就是不混用超类和子类也可以避免问题，这和getClass速途同归。
1. 抽象类的子类中增加新的值组件，不会违反equals约定。

**高质量equals**
1. 使用==操作符检查“参数是否是对象引用”。性能优化。
1. 使用isntanceof操作符检查“参数是否为对象的类型或子类”。同时也检查了参数非空。
1. 把参数转换成正确的类型。
1. 检查参数中的关键字段是否与该对象中对应的相匹配。float和double需要对NaN和-0.0等进行特殊处理。
1. 检查等价关系。
    * 覆盖equals时总要覆盖hashCode。
    * 不要企图让equals方法过于智能。
    * 不要将equals声明中的Object对象替换成其他的类型。重写时参数类型必须是一样的，返回类型可以是子类型。

```
public class Student{
    private String id;
    private String name;
    private int age;

    @Override
    public boolean equals(Object o){
        if(o==this) return true;
        if(!(o instanceof Student)) return false;
        Student oStudent=(Student)o;
        return Objects.equals(id,oStudent.id)
            && Objects.equals(name,oStudent.name)
            && age==oStudent.age;
    }
}
```

## 覆盖equals时总要覆盖hashCode

**Object规范**
1. 默认情况下，Object.hashCoe返回对象的内存地址。
1. 如果2个对象的根据equals(Object)比较是相等的，那么hashCode方法返回的整数结果必须相等。
1. 如果2个对象的根据equals(Object)比较是不相等的，那么hashCode方法返回的整数结果是不确定的。
    * 散列函数会倾向于为不相等的对象产生不相等的散列码。
    * HashMap有一项优化，将散列码缓存起来，如果散列码不匹配，则对象不相等。对于HashMap来说，没有覆盖hashCode的类的所有键都是不相等的。

**散列码的计算**
1. 简单的散列函数实现：
    * hash=非零常数值，比如17。
    * boolean，f?1:0。
    * byte char short int，(int)f。
    * long，(int)(f^(f>>>32))。
    * float，Float.floatToIntBits(f)。
    * double，Double.doubleToLongBits(f)，再从long->int。
    * Object，f==null?0:f.hashCode()。
    * Array，把每一个元素当做单独的字段。
    * 循环计算，hash=31*hash+每一个字段的计算值。
    * return hash。
    * 测试。

```
public class Studnet{
    @Override
    public int hashCode(){
        int hash=17;
        hash=31*hash+id.hashCode();
        hash=31*hash+name.hashCode();
        hash=31*hash+age;
        return hash;
    }
}
```

2. 散列码的计算必须排除equals比较计算中没有用到的任何域。之所以选择31奇素数，如果是偶数的话，乘法溢出会导致信息丢失，因为与2相乘等价于移位运算。而使用素数只是习惯使然，并且31可以很容易用移位和减法代替乘法，31*i==(i<<5)-i。
1. 如果是不可变类，计算散列码的开销也比较大，可以考虑把散列码缓存在对象内部。如果类的大多数对象会被用做散列键，可以在实例的时候计算散列码，否则可以延迟初始化散列码。

```
//缓存，延迟初始化。
private volatile int hashCode;

@Override
public int hashCode(){
    int hash=hashCode;
    if(hash==0){
        ...
    }
    return hash;
}
```

4. 不要试图从散列码计算中排除掉一个对象的关键部分来提高性能。

## 始终要覆盖toString
1. 默认显示“类名@hashCode”。
1. 如果指定了格式，最好提供一个相匹配的静态工厂或者构造器，以便对象和字符串之间来回转换。Java类库的很多值类都采用了这种做法，包括BigInteger和BigDecimal和绝大多数的基本类型包装类。
1. 指定了格式的缺点是必须始终如一地坚持。

## 谨慎地覆盖clone
1. Object.clone是protected的，如果子类没有实现Cloneable，那么调用clone会抛出CloneNotSuportedException。
1. Object.clone是特殊的方法，它会通过反射将当前对象的所有字段进行浅拷贝。

```
class Student implements Cloneable{
    @Override
    public Student clone() throws CloneNotSupportedException {
        return (Student)super.clone();
    }
}
```

3. 永远不要让客户去做任何类库能够替客户完成的事情。
1. 如果想要实现深拷贝，可以递归地调用clone方法。
1. clone架构与final对象不兼容，final对象无法深拷贝。

```
class Student implements Cloneable{
    private String id="id";
    private String name;
    private int age;
    private int[] scores;

    @Override
    public Student clone() throws CloneNotSupportedException {
        Student clone = (Student) super.clone();
        clone.scores=scores.clone();
        return clone;
    }
}
```

**拷贝构造器或拷贝工厂代替Cloneeable/clone**
1.  另一个实现对象拷贝地好办法是提供一个拷贝构造器或拷贝工厂，拷贝构造器或拷贝工厂唯一的参数类型和返回的类型相同。这2种方法都比Cloneable/clone方法更具有优势。
    * 不用实现Cloneable，不用修改clone为public和它的返回类型。
    * 不会抛出受检异常。
    * 支持final对象。
    * 不用类型转换。
    * 返回类型有更多选择。

## 考虑实现Comparable接口

**约定**
1. x.compareTo(y)==-y.compareTo(x)。
1. 比较关系是可传递的。
1. 强烈建议(x.compareTo(y)==0)==(x.equals(y))。

**compareTo与equals**
1. 依赖比较关系的类包括有序集合类PriorityQueue、TreeSet和TreeMap，以及工具类Collections和Arrays。
1. 与equals不同，在跨越不同类的时候，compareTo可以不做比较。
1. 除却例外，x.compareTo(y)==0和x.equals(y)是一致的。例外如，BigDecimal("1.0")和BigDecimal("1.00")，通过equals比较是不相等的，而通过compareTo比较是相等的。
1. Comparable接口是参数化的，因此compareTo方法不必进行类型检查和类型转换。
1. 浮点的比较用Float.compare和Double.compare。

```
public static int compare(double d1, double d2) {
        if (d1 < d2) return -1; 
        if (d1 > d2) return 1; 

        long thisBits    = Double.doubleToLongBits(d1);
        long anotherBits = Double.doubleToLongBits(d2);

        return thisBits == anotherBits ?  0 : (thisBits < anotherBits ? -1 : 1);
}
```

```
class Student implements Comparable<Student>{
    private String id="id";
    private String name;
    private int age;

    @Override
    public int compareTo(Student o) {
        int result=id.compareTo(o.id);
        if (result!=0) return result;
        
        result=name.compareTo(o.name);
        if (result!=0) return result;

        return age-o.age;
    }
}
```

# 类和接口










