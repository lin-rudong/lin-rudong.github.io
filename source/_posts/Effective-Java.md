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

## 使类和成员的可访问性最小化

**封装**
1. 设计良好的模块会隐藏所有的实现细节，把API与实现清晰地隔离开来，这个概念称为信息隐藏或封装，是软件设计的基本原则之一。
1. 封装可以有效地解除组成系统的各模块之间的耦合关系，使得这些模块可以独立地开发、测试、优化、使用、理解和修改。
1. 顶层的类和接口只有2种可能的访问级别：包级私有的和公有的。默认是包级私有，作为包的实现的一部分，而公有则作为该包导出的API的一部分。
1. 成员（字段、方法、嵌套类和嵌套接口）有4种可能的访问级别，按照可访问性递增顺序如下。
    * 私有的
    * 包级私有的
    * 受保护的（类导出的API）
    * 公有的（类导出的API）
1. 子类重写的方法的访问级别不允许低于超类中访问级别，确保任何可使用超类的实例的地方也都可以使用子类的实例。

**封装规则**
1. 尽可能地使每个类或者成员不被外界访问。
1. 如果包级私有的顶层类或接口只是在某一个类的内部被用到，就应该考虑使它成为唯一使用它的那个类的私有嵌套类。
1. 实例字段决不能是公有的。包含公有可变字段的类不是线程安全的。
1. 静态字段也不能是公有的，有一种例外情况，假设常量是类抽象的一部分，可以通过public static final来暴露这些常量。很重要的一点是，这些常量要么包含基本类型的值，要么包含指向不可变对象的引用。
1. 长度非0的数组总是可变的，所有，类具有公有的静态final数组字段，或者返回这种字段的访问方法，这几乎总是错误的，因为客户端能够修改数组中的内容。
    * 可以将数组变成私有的，并增加一个公有的不可变列表视图。
    * 也可以将数组变成私有的，并添加一个公有方法访问数组的一个备份。

```
private static final Student[] PRIVATE_STUDENTS={...};
public static final List<Student> STUDENTS=Collections.unmodifiableList(Arrays.asList(PRIVATE_STUDENTS));
```
 
## 在公有类中使用访问方法而非公有域
1. 公有类永远都不应该暴露可变的字段。
1. 如果类是包级私有的，或者是私有的嵌套类，因为只是实现的一部分而没有导出API，直接暴露它的字段并没有本质错误。

## 使可变性最小化
1. Java类库的不可变类：String、基本类型的包装类，BigInteger和BigDecimal。

**不可变类的5条规则**
1. 不要提供任何会修改对象状态的方法。
1. 保证类不会被扩展。防止子类破坏类的不可变行为，一般做法是使这个类成为final。
1. 使所有的字段都是final的。
1. 使所有的字段都成功私有的。
1. 确保对于任何可变组件的互斥访问。如果类具有指向可变对象的字段，必须确保类的客户端无法获得这些对象的引用。

**不可变**
1. 不可变类的如果需要进行运算，会创建并返回新的实例，这种模式是函数式的做法。与之相对应的是更常见的过程式的或者命令式的做法。
1. 不可变对象本质上是线程安全的，它们不要求同步。不可变对象可以被自由地共享，影应该鼓励客户端尽可能地重用现有的实例，最简单的办法就是，对于频繁用到的值，为它们提供public static final常量。静态工厂可以进一步把频繁被请求的实例缓存起来。

```
public static final Complex ZERO=new Complex(0,0);
```

3. 不仅可以共享不可变对象，也可以共享它们的内部信息。
1. 不可变类真正唯一的缺点是，对于每个不同的值都需要一个单独的对象。解决办法是提供一个公有的可变套配类，比如String的StringBuilder和基本上已经废弃的StringBuffer。
1. 让不可变的类变成final的另一种做法就是，让类的所有构造器都变成私有的或者包级私有的（在包内同样可以继承），并添加公有的静态工厂。
1. 除非有令人信服的理由要使字段变成是非final的，否则要使每个字段都是final的，限制它的可变性。不要再构造器或者静态工厂之外再提供公有的初始化方法，除非有令人信服的理由必须这么做。不应该提供重新初始化方法，重用和共享不可兼得。

## 复合优先于继承
1. 在包的内部使用继承是非常安全的，因为子类和超类的实现都处在同一个程序员的控制之下。对于专门为了继承而设计并且具有很好的文档说明的类来说，也是非常安全的。然而，对普通的具体类进行跨越包边界的继承，则是非常危险的。
1. 继承打破了封装性，子类依赖于其超类中特定功能的实现细节，超类的实现可能会随着发行版本的不同而有所变化，子类可能会遭到破坏因而，子类必须跟着其超类的更新而演变，除非超类是专门为了扩展而设计的，并且具有很好的文档说明。

**继承可能破坏依赖**
1. 超类的方法之间可能存在依赖，如果重写了其中的方法，可能破坏依赖关系。
1. 超类在后续的发行版本中可能获得新的方法，子类可能无意识地重写了这个方法，这也将破坏依赖关系。

**复合**
1. 复合将现有类作为私有字段实现扩展。新类的实例方法都可以调用被包含的现有类实例的对应的方法，并返回它的结果，这称为转发，新类的方法称为转发方法。
1. 修饰模式是指每一个包装类实例都把另一个被包装类实例包装起来以增加特性。
1. 只有当子类真正是超类的子类型时，存在is-a关系时才适合用继承。如果在适合使用复合的地方使用了继承，则会不必要地暴露实现细节和限制在原始的实现上。继承还会把缺陷传播到子类。

## 要么为继承为设计，并提供文档说明，要么就禁止继承。
1. 类的文档必须精确地描述重写每个方法所带来的影响。
1. 对于为了继承而设计的类，唯一的测试方法就是编写子类。
1. 构造器决不能调用可被重写的方法。这样会导致子类中重写版本的方法将会在子类的构造器运行之前就先被调用。

```
class People{
    public People(){
        f();
    }
    public void f(){}
}

class Student extends People{
    private String name="name";

    @Override
    public void f(){
        System.out.println(name);
    }
}

public class Main {

    public static void main(String[] args) {
        Student student = new Student(); //打印 “null”
        student.f(); //打印 “name";
    }

}
```

4. 如果决定在一个为了继承而设计的类中实现Cloneable或Serializeble接口，应该意识到，clone和readObject方法在行为上非常类似于构造器，不能调用可重写的方法。

## 接口优于抽象类
1. JDK8开始，接口增加了默认方法和静态方法。
1. Java只允许单继承，而接口没有限制。
1. 现有的类可以很容易被更新，以实现新的接口。
1. 接口是定义mixin（混合类型）的理想选择。mixin类型表明提供了可供选择的行为，允许任选的功能可被混合到类型的主要功能中。比如Comparable允许类表明它的实例可以与其他的可互相比较的对象进行排序。
1. 接口允许我们构造非层次结构的类型框架。

**骨架实现与简单实现**
1. 骨架实现类比较简单，在接口中确定哪些方法是最为基本的，其他的方法可以根据它们来实现。这些基本方法将称为骨架实现类的抽象方法，而其他的方法提供具体的实现。骨架实现类是为了继承的目的而设计的。
1. 简单实现，比如额AbstractMap和SimpleEntry，很像骨架实现，这是因为它实现了接口，并且是为了继承而设计的。但是区别在于它不是抽线的：它是最简单的可能的有效实现。你可以原封不动地使用，也可以看情况将它子类化。

## 接口只用于定义类型
1. 常量接口没有任何方法，只包含静态的final字段。常量接口模式是对接口的不良使用。类在内部使用某些常量，属于实现细节。
1. 当类或接口的常量能看作枚举类型的成员，就应该用枚举类型。否则，应该使用不可实例化的工具类。

## 类层次优于标签类
1. 标签类，实例有多种风格。缺点，多种风格实现混在一起，破坏了可读性，也让实例承担着属于其他风格不相关的字段。标签类过于冗长、容易出错，并且效率低下。

## 用函数对象表示策略
1. 函数对象可以作为参数，允许函数的调用者通过传入函数对象，来指定自己的行为。这称为策略模式。
1. Java中常用函数接口来实现函数对象，

## 优先考虑静态成员类
1. 嵌套类存在的目的应该只是为它的外围类提供服务。嵌套类有4种：静态成员类、非静态成员类、匿名类和局部类。除了第1种外，其他3种都称为内部类,内部类不能有静态成员。
1. 静态成员类是最简单的一种嵌套类，最好把它看作是普通的类，它可以访问外围类的所有成员，包括私有成员。静态成员类是外围类的一个静态成员，遵守同样的可访问性规则。
1. 非静态成员类的每个实例都隐含着与外围类的一个外围实例相关联。在非静态成员类的实例方法内部，可以调用外围实例上的方法，或者利用修饰过的this获得外围实例的引用。在没有外围实例的情况下，是不可以创建非静态成员类的实例的。当非静态成员类的实例被创建时，它与外围类实例之间的关联关系就被建立起来，这会增加非静态成员类实例的开销。
1. 如果声明成员类不要求访问外围实例，就要作为静态成员类。私有静态成员类的一种常见用法是用来代表外围类所代表的对象的组件。
1. 匿名类在使用的时候同时声明和实例化。当且仅当匿名类出现在非静态的环境种，它才有外围实例。匿名类只有在声明的时候才可以实例化。
    * 匿名类的一种常见用法是动态地创建函数对象。
    * 另一种常见用法是创建过程对象，比如Runnable、Thread或者TimerTask实例。
    * 第三种常见用法是在静态工厂方法的内部。
1. 局部类是4种嵌套类中用得最少的类，和匿名类一样，只有当局部类是在非静态环境中定义的时候，才有外围实例。

# 泛型

## 请不要在新代码中使用原生态类型
1. 数组的协变，数组之间和其元素类型有着一致的继承关系。如下，由于编译时只能确定引用的静态类型，fruits的静态类型是Fruit[]，所以可以编译通过。但是运行时可以知道数组的实际类型，所以会出现ArrayStoreException。

```
Fruit[] fruits=new Apple[10];
fruits[0]=new Fruit();
```

2. 所有泛型都是原生态类型的一个子类型，List<Object>和List<String>之间没有继承关系，都是继承于List。
1. 可以使用无限制的通配符类型`?`来代替原生态类型。原生态类型不安全，因为它可以包装所有的泛型子类型，然后擅自帮泛型添加元素，导致泛型在运行时可能会出现类型转换异常。List<?>用来代替List,表示不知道具体类型，所以不能限制存，只能存null，但不影响get。

```
List<Apple> apples=new ArrayList<Apple>();
List list=apples;

//正常，因为泛型擦除，运行时只有List，但是泛型apples被破坏了。
list.add(new Orange()); 
list.get(0); //正常，返回Object。
apples.get(0); //运行错误，泛型在运行时会自动进行类型转换。
```

4. 由于类型擦除，在类字面量中只能用原生态类型，而且instanceof操作只能用原生类类型或者?通配符类型可以进行，

## 消除非受检警告
1. 如果无法消除警告，同时可以证明引起警告的代码是类型安全的，可以用@SuppressWarnings("unchecked")注解来禁止这条警告。每当使用注解时，都要添加一条注解。

## 列表优先于数组

**创建泛型数组是非法的**
1. 数组是协变的，就是数组和元素保持一致的继承性，因为继承性只能将类型检验延迟到运行时。相反泛型是不可变的，在编译时便可以进行类型检验，从而保证运行时的自动类型转换顺利进行。数组协变存在缺陷，因为类型检查应该在编译时进行的，防止运行时出现异常。
1. 数组是具体化的，运行时有实际类型，而泛型通过擦除，运行时没有元素类型信息。
1. 泛型数组不是类型安全的，如果创建泛型数组，泛型的类型擦除会让数组不能保存泛型的元素类型，从而可能导致运行时数组的类型检查失效，另外数组协变可以避过泛型的编译时类型检查，导致泛型在运行时的自动类型转换异常。

```
List<String>[] stringLists=new List<String>[1];
Object[] objects=stringLists;
objects[0]=new List<Integer>();
stringLists[0].get(0);
```

4. List<E>和List<String>类似的类型是不可具体化的类型，在运行时不包含元素的类型信息，唯一可具体化的参数化类型是无限制的通配符类型。

## 优先考虑泛型
1. 可以绕过创建泛型数组的禁令，先创建Object[]，然后转换成泛型数组。这种用法是合法的，但不是类型安全的。
1. 将字段的类型改为Object[]，而获取元素时转换类型。更安全但是可能很繁琐，所以第1条更常用。

## 优先考虑泛型方法
1. 泛型方法无需明确指定类型参数的值，可以通过类型推导确定。
2. 类型限制，在声明定义时使用，作用是限制参数的类型。

```
public static <T extends Comparable<T>> void sort(List<T> list){}
```

## 利用有限制通配符来提升API的灵活性
1. 为了获得最大限度的灵活性，要在表示生产者或者消费者的输入参数上使用通配符类型。如果某个输入参数既是生产者又是消费者，就不能用通配符。PECS表示producer-extends，consumer-super。
1. 不要用通配符类型作为返回类型，因为它会强制用户在客户端代码使用通配符类型，通配符对于类的用户应该是无形的。

## 优先考虑类型安全的异构容器
1. Map<Class<?>,Object>

# 枚举和注解

## 用enum代替int常量
1. Java枚举本质上是int值，基本思想是通过public static final字段为每个枚举常量导出实例的类。因为没有可以访问的构造器，所以枚举类型是真正的final，并且不能创建枚举类型的实例。
1. 策略枚举，把策略选择作为枚举的一个私有的嵌套枚举，可以为上层的枚举选择策略，然后根据策略选择所做的操作委托给策略枚举。

## 用实例字段代替序数
1. 永远不要根据枚举的序数导出与它关联的值，而是要将它保存在一个实例字段中。

## 用EnumSet代替位字段
1. 位字段，能用OR位运算将多个常量合并表示集合的字段。

```
public class Text{
    public static final int BOLD=1 << 0;
    public static final int ITALIC=1 << 1;
    ... 
}
```

2. EnumSet底层的枚举类型元素小于或等于64，那么整个EnumSet是用1个long来表示的。EnumSet的方法是采用位算法实现的。

```
public class Text{
    public enum Style{
        BOLD,ITATIC,...
    }

    public void applyStyles(Set<Style> styles)
}


text.applyStyles(EnumSet.of(Style.BOLD,style.ITALIC));
```

## 用EnumMap代替序数索引
1. 最好不要用序数来索引数组，而要使用EnumMap。EnumMap比较简短、清楚，也更加安全。

## 用接口模拟可伸缩的枚举
1. 枚举不可以继承类，但是可以实现接口。枚举的构造函数不可以是public，因为枚举不可以通过构造函数实例化。

```
public interface Operation{
    double apply(double x,double y);
}

public enum BasicOperation implements Operation{
    PLUS("+"){
        public double apply(double x,double y){
            return x+y;
        }
    }

    private final String symbol;
    BasicOperation(String symbol){
        this.symbol=symbol;
    }
}
```

2. 虽然无法编写可扩展的枚举类型，却可以通过编写接口以及实现该接口的基础枚举类型，对它进行模拟。

## 注解优先于命名模式
1. 以前是用命名模式表明程序元素需要通过某种工具或框架进行特殊处理，比如测试方法名称要以test作为开头。
    * 如果文字拼错没有任何提示。
    * 无法确保它们只用于相应的程序元素上。
    * 没有提供将参数值与程序元素关联起来的好方法。
1. 有了注解，就完全没有理由再使用命名模式。大多数程序员都不必定义注解类型，但是所有的程序员都应该使用Java平台所提供的预定义的注解类型。

## 坚持使用Override注解

## 用标记接口定义类型
1. 标记接口是没有包含方法声明的接口。
1. 如果没有实现Serializable标记接口，ObjectOutputStream.write(Object)方法将会失败。
1. 如果标记是应用到任何程序元素而不是类或者接口，就必须使用注解。如果需要作为相关方法的参数，那么优先使用接口，它可以为你提供编译时类型检查。

# 方法

## 检查参数的有效性
1. 在文档中指明参数的限制，用Javadoc的@throws说明会抛出的异常，这些异常通常是IllegalArgumentException、IndexOutOfBoundsException或NullPointerException，并且在方法体的开头处检查参数。
1. 对于未被导出的方法，即private和包级私有的方法，作为包的创建者，你应该确保只将有效的参数值传递进来。因此，这些方法应该用断言来检查它们的参数。如果没有将-ea标记传递给Java解释器，那么它们不会起作用，本质上也不会有成本开销。
1. 有的参数，方法本身用到，却被保存起来供以后使用，检验这类参数的有效性尤为重要，比如构造器。

## 必要时进行保护性拷贝
1. Java是安全的语言，它对于缓冲区溢出、数组越界、非法指针以及其他的内存破坏错误都自动免疫。
1. 对于构造器的每个可变参数进行保护性拷贝是必要的，使用备份作为实例字段，而不使用原始的对象。这时不应该检查原始参数的有效性，应该检查备份的有效性。
    * 如果参数是非final的，clone方法可能返回有专门处于恶意而设计的不可信子类。
1. 返回可变内部字段时考虑保护性拷贝。
1. 如果不能容忍对象进入数据结构后发生变化，就要考虑保护性拷贝。
1. 世事无绝对，如果信任客户端的话，也可以不用保护性拷贝。

## 谨慎设计方法签名
1. 谨慎地选择方法的名称。
1. 不要过于追求提供便利的方法。对于类和接口所支持的每个动作，都提供一个功能齐全的方法，只有当一项操作被经常用到的时候，才考虑为它提供快捷方式。
1. 避免过长的参数列表。参数小于等于4个最好。
    * 正交分解方法。
    * 创建分解类，一般位静态成员类，用来保存参数的分组。
    * 结合前2种方法的特征，对象构建和方法调用都可以采用Builder方法，可以考虑定义一个对象来表示方法的所有参数。
1. 对于参数类型，要优先使用接口而不是类。解耦，容易扩展，支持更多实现。
1. 对于boolean参数，要优先使用2个元素的枚举类型。它使代码更易阅读和编写，也使以后更易扩展。

## 慎用重载
1. 重载方法是在编译时做出决定的，由参数的静态类型决定，即静态分派，并且由所有参数决定，即静态多分派。
1. 覆盖方法是在运行时做出决定的，由调用者的实际类型决定的，即动态分配，并且只由调用者单独决定，即动态单分派。
1. 安全而保守的策略是，永远不要导出2个具有相同参数数目的重载方法。次之就是保证相同数目的参数的类型有明显的不同，即不能互相转换,这时需要注意基本类型和对应的包装类型。

```
List<Integer> list=new ArrayList<>(Arrays.asList(4,3,2,1,0));
list.remove(0); //index
list.remove((Integer) 0); //Object

Set<Integer> set=new HashSet<>(Arrays.asList(4,3,2,1,0));
set.remove(0); //Object
```

4. 如果2个重载方法在同样的参数上被调用时，它们执行相同的功能，返回相同的结果，重载就不会带来危害。这时候，更具体化的重载方法会把调用转发给更一般化的重载方法。

## 慎用可变参数
1. 可变参数方法接受0个或者多个指定类型的参数，会将这些参数转变成数组，0个参数时也会转变成数组，而不会是null。
1. 没有参数时的检查只能在运行时，可以通过2个参数，1个是指定类型的参数，1个是可变参数，限制没有参数的发生。

```
static int min(int firstArg,int... remainingArgs){
    ...
}
```

3. 提供常用的重载方法可以提高可变参数的性能。

```
public void f(int a1){}
public void f(int a1,int a2){}
public void f(int a1,int a2,int a3){}
public void f(int a1,int a2,int a3,int ...rest){}
```

## 返回0长度的数组或者集合，而不是null。
1. 0长度数组是不可变的，所以很可能被自由共享。
1. toArray数组参数的长度为0，指明所期望的返回类型，因为不能创建泛型数组，而可以通过参数创建具体的数组。如果toArray的数组参数的长度够长，则不会创建数组，采用数组参数并返回数组参数。

```
List<Integer> list= Arrays.asList(1,2,3);
Integer[] array=list.toArray(new Integer[0]);
```

## 为所有导出的API元素编写文档注释
1. Javadoc利用特殊格式的文档注释，可以根据源代码自动产生API文档。
1. 在每个被导出的类、接口、构造器、方法和字段声明之前增加一个文档注释。如果类是可序列化的，也应该对它的序列化形式编写文档。为了维护代码，还应该为没有导出的元素编写文档注释。
1. 文档应该列举处方法的所有前提条件和后置条件。除此之外，还需要描述副作用，最后，也应该描述线程安全性。
    * 前提条件是指为了能够调用方法，必须满足的条件。由@param和@throws标签描述。
    * 后置条件是指调用成功完成了后，哪些条件必须要满足。
1. 方法的每个参数都有1个@param标签，和1个@return标签，除非返回类型是void，对于抛出的异常，无论是受检还是非受检异常，都要1个@throws。
1. 当为泛型类型编写文档时，要为所有的类型参数编写类型参数。当为枚举类型编写文档时，要确保在文档中说明常量，以及类型，还有任何公有的方法。为注解类型编写文档时，要确保在文档中说明所欲成员，以及类型本身。
1. 包级私有的文档注释放在package-info.java的文件中。
1. Javadoc具有继承方法注释的能力，接口的文档注释优先于超类的。
1. 在文档注释内部出现任何HTML标签都是允许的，但是HTML元字符必须要经过转义。

# 通用程序设计

## 将局部变量的作用域最小化
1. 要使局部变量的作用域最小化，最有力的办法就是在第一次使用它的地方声明。
1. 几乎每个局部变量的声明都应该包含一个初始化表达式。如果还没有足够的信息对一个变量进行有意义的初始化，就应该推迟这个声明。try例外。
1. 循环中提供了特殊的机会来将变量的作用域最小化。如果循环终止之后不再需要循环变量的内容，for循环就优先于while循环。
1. 最后一种将局部变量的作用域最小化的方法是使方法小而集中。

## for-each循环优先于传统的for循环

## 了解和使用类库
1. 必须熟悉java.lang、java.util，一定程度上java.io也需要。java.util包种的集合框架和并发工具更是重要。
1. 不要重复造轮子。

## 如果需要精确的答案，请避免使用float和double。
1. float和double不适合货币计算，应该用BigDecimal，也可以用int和long用来表示以分做单位的货币。

## 基本类型优先于装箱基本类型
1. 装箱基本类型默认值为null，如果null拆箱会抛出NullPointerException。
1. ==运算符对2个装箱基本类型不会自动拆箱。

## 如果其他类型更适合，则尽量避免使用字符串。
1. 字符串不适合代替其他的值类型。
1. 字符串不适合代替枚举类型，枚举类型比字符串更加适合用来表示枚举类型的常量。
1. 字符串不适合代替聚集类型。如果有多个组件，用类更合适。

## 当心字符串连接的性能
1. 用StringBuilder或者字符数组可以提高性能。

## 通过接口引用对象

## 接口优先于反射机制
1. java.lang.reflect反射机制提供了通过程序来访问已加载的类的信息的能力。给定一个Class实例，可以获得Constructor、Method和Field实例，通过它们可以访问底层对等体。
1. 反射机制允许一个类使用另一个在编译时不存在的类。
    * 丧失了编译时类型检查的好处，在运行时可能会失败。
    * 执行反射访问所需要的代码非常笨拙和冗长。
    * 性能损失。
1. 普通应用程序在运行时不应该以反射方式访问对象。
1. 对于有些程序，它们必须用到在编译时无法获取的类，但是在编译时存在适当的接口或者超类，就可以以反射的方式创建实例，然后通过它们的接口或者超类，以正常的方式访问这些实例。如果构造器不带参数，甚至不需要使用java.lang.reflect包，Class.newInstance方法就提供了所需的功能。

## 谨慎地使用本地方法
1. 本地方法是指本地程序设计语言，比如C或C++。
    * 提供访问特定于平台的机制，比如访问注册表和文件锁。
    * 提供访问遗留代码库的能力，从而可以访问遗留数据。
    * 可以提高系统性能。
1. 使用本地方法来提高性能的做法不值得提倡。因为JVM变得越来越快。而且很多本来本地方法才有的特性，Java也在不断完善。

## 谨慎地进行优化
1. 很多计算上的过失都被错误地归咎于效率。
1. 不要去计较效率上的一些小小的得失。
1. 在没有绝对清晰的优化方案之前，不要进行优化。
1. 不要因为性能而牺牲合理的结构，要努力编写好的程序而不是快的程序，好的结构可以优化程序。
1. 努力避免那些限制性能的设计决策。当一个系统设计完成之后，其中最难以更改的组件是那些指定了模块之间交互关系以及模块和外界交互关系的组件。主要是API、线路层协议以及永久数据格式。

## 遵守普遍接受的命名惯例
1. 包的名称，可以包括小写字母和数字，很少使用数字。包的组成部分应该比较简短，通常不超过8个字符，鼓励使用有意义的缩写形式。虽然Java语言并没有支持包层次，但是包在设计的时候还是要按照层次结构划分。
1. 类、接口、枚举和注解的名称，用驼峰形式，应该尽量避免用缩写。
1. 方法和字段的名称，用小驼峰形式，也应该尽量避免用缩写。
1. 常量的名称，用大写字母和下划线。
1. 局部变量名称，和字段相似，只不过它允许缩写。
1. 类型参数，T表示任意的类型，E表示集合的元素类型，K和V表示映射的键和值类型，X表示异常。任意类型的序列可以是T、U、V或者T1、T2、T3。
1. 转换类型toType，返回视图asType，返回跟对象同值得基本类型的方法typeValue。

# 异常

## 只针对异常的情况才使用异常
1. 异常应该只用于异常的情况下，它们永远不应该用于正常的控制流。

## 对可恢复的情况使用受检异常，对编程错误使用运行时异常。
1. 3种抛出结构：受检异常，运行时异常和错误。受检异常必须捕获处理。
1. 运行时异常和错误，属于未受检异常，一般是不可恢复的情况，继续执行下去也是有害无益，所以不需要也不应该捕获。
1. 运行时异常用来表明编程错误。自定义的未受检异常都应该继承RuntimeException。

## 避免不必要地使用受检的异常

## 优先使用标准的异常
1. 最经常被重用的异常是IllegalArgumentException。另一个经常被重用的异常是IllegalStateException，因为接收对象的状态而使调用非法，比如对象被正确初始化之前就被使用。
1. 所有的方法调用错误都可以归结于非法参数和非法状态，但还有一些更具体的异常，比如NullPointerException和IndexOutOfBoundsException。
1. ConcurrentModificationException，如果对象被设计成专门用于单线程或者于外部同步机制配合使用，一旦发现它正在或已经被并发地修改，抛出这个异常。
1. UnsupportedOperationException，如果对象不支持所请求的操作，就会抛出这个异常。
1. ArithmeticException和NumberFormatException。

## 抛出与抽象相对应的异常
1. 异常转译，高层的实现应该捕获底层的异常，同时抛出可以按照高层抽象进行解释的异常。
1. 异常链，底层的异常作为原因被传到高层的异常，高层的异常提高访问方法来获得底层的异常。

## 每个方法抛出的异常都要有文档

## 在细节消息中包含能捕获失败的信息

## 努力使失败保持原子性
1. 失败原子性，失败的方法调用应该使对象保持在被调用之前的状态。

## 不要忽略异常

# 并发

## 同步访问共享的可变数据
1. 同步不仅可以阻止一个线程看到对象处于不一致的状态之中，它还可以保证进入同步方法或者同步代码块的每个线程，都看到由同一个锁保护的之前所有的修改效果。
1. 语言规范保证读写变量是原子的，除非这个变量的类型为long或double，但是不保证一个线程写入的值对于另一个线程将是可见的。

```
//没有同步，done没有线程可见性。
while(!done)
    i++;

//JVM优化。
if(!done)
    while(true)
        i++;
```

3. 当多个线程共享可变数据的时候，每个读或者写数据的线程都必须执行同步，保证原子性和可见性。
1. 活性失败，程序无法前进。安全性失败，程序会计算出错误的结果。

## 避免过度同步
1. 同步区域内部，不要调用设计成要被覆盖的方法，或者是由客户端以函数对象的形式提供的方法。这些方法是外来的，调用它可能会导致死锁，异常或者数据损坏。比如，foreach遍历集合的时候，不可以同时修改集合，否则会发生ConcurrentModificationException。
1. Java的锁是可重入锁，可能会将活性失败变成安全性失败。
1. CopyOnWriteArrayList可以在foreach遍历集合的同时修改。通过重新拷贝整个底层数组来实现所有的写操作，由于内部数组永远不改动，因此迭代不需要锁定。
1. 开放调用，在同步区域之外调用外来方法。这样可以避免死锁，也可以减少同步时间提高并发性。
1. 通常，你应该在同步区域内做尽可能少的工作。

## executor和task优先于线程
1. 小程序或者轻载服务器使用Executors.newCachedThreadPool，大负载服务器最好使用Executors.newFixedThreadPool，如果想最大限度地控制线程池使用ThreadPoolExecutor。
1. ScheduledThreadPoolExecutor用来代替Timer。

## 并发工具优先于wait和notify
1. java.util.concurrent中的高级工具分成3类：执行器，并发集合和同步器。
1. 优先使用ConcurrentHashMap，而不是使用Collections.synchronizedMap或者Hashtable。只要用并发Map替换老式的同步Map，就可以提升并发应用程序的性能。
1. 大多数ExecurotService实现包括ThreadPoolExecutor都使用BlockingQueue的生产者-消费者队列。
1. 同步器的作用是使线程能够等待另一个线程，允许它们协调动作。最常用的同步器是CountDownLatch和Semaphone，较不常用的是CyclicBarrier和Exchanger。
1. 不应该是使用wait和notify，如若不可避免，也应该在条件循环中调用wait方法，nofity和notifyAll可以唤醒等待线程，保守的建议是你总是应该使用notifyAll。

```
while(!condition)
    wait(); //释放锁，加入等待队列。继续执行需要唤醒和锁，而且还需要再判断条件，因为很可能被nofifyAll过度唤醒。

```

## 线程安全性的文档化
1. 线程安全性的级别：
    * 不可变的immutable，实例是不变的，不需要外部的同步，例如String、Long和BigInteger。
    * 无条件的线程安全unconditionally thread-safe，实例是可变的，但是类有着足够的内部通古比，不需要外部的同步，比如Random、ConcurrentHashMap。
    * 有条件的线程安全conditionally thread-safe，与无条件的区别是有些方法为进行安全的并发使用而需要外部同步，例如Collections.synchronized包装的集合，它们的迭代器要求外部同步。
    * 非线程安全not thread-safe，例如ArrayList和HashMap。
    * 线程对立thread-hostile，即使有外部同步，也不能并发使用，线程对立的根源通常在于，没有同步地修改静态数据。
1. 公有可访问锁对象允许客户端同步地执行一个方法调用序列，但可能导致拒绝服务攻击。

## 慎用延迟初始化
1. 大多数的字段应该正常地进行初始化，除非了为了达到性能目标，或者为了破坏有害地初始化循环。
1. 如果出于性能的考虑而需要对静态字段使用延迟初始化，就使用lazy initialization holder class模式，可以保证类要到被用到的时候才会被初始化。

```
private static class FieldHolder{
    static final FieldType field=new FieldType();
}

//不需要同步
static FieldType getField(){
    return FieldHolder.field;
}
```

3. 如果出于性能的考虑而需要对实例字段使用延迟初始化，就使用双重检查模式。静态字段也可以用这个模式，但不如上面介绍的模式好。如果可以接受重复初始化的实例字段，也可以考虑使用单重检查模式，而不需要同步。

```
private volatile FieldType field;

FiledType getField(){
    if(field==null){ //同步性能提升
        synchronized(this){ //同步
            if(field==null){ //延迟初始化
                field=new FieldType();
            }
        }
    }
    return field;
}
```

## 不要依赖于线程调度器
1. 线程优先级是Java平台上最不可移植的特征了。
1. 对于大多数程序员来说，Thread.yield的唯一用途是在测试期间人为地增加程序的并发性。

# 避免使用线程组
1. 线程组并没有提供太多有用的功能，而且它们提供的许多功能还都是有缺陷的。

# 序列化

## 谨慎地实现Serializable接口

**实现Serializable的代价**
1. 实现Serializable接口而付出地最大代价是，一旦一个类被发布，就大大降低了“改变这个类的实现”的灵活性。
1. 实现Serializable的第二个代价是，它增加了出现Bug和安全漏洞的可能性。反序列化机制是一个隐藏的构造器，具备与其他构造器相同的特点。
1. 实现Serializable的第三个代价是，随着类发行新的版本，相关的测试负担也增加了。

**注意**
1. 为了继承而设计的类应该尽可能少地去实现Serializable接口，用户的接口应该尽可能少地继承Serializable接口。在为了继承而设计的类中，真正实现了Serializable接口的有Throwable、Component和HttpServlet类，这样RMI的异常才可以从服务器端传到客户端，GUI才可以被发送、保存和恢复，会话状态可以被缓存。
1. 内部类不应该实现Serializable，它们使用编译器产生的合成字段来保存只想外围实例的引用，以及保存来自外围作用域的局部变量的值。这些字段如何对应到类定义中没有明确的规定，因此，内部类的默认序列化形式是定义不清楚的。然而，静态成员类可以实现Serializable接口。

## 考虑使用自定义的序列化形式
1. 接受默认的序列化形式是一个非常重要的决定，你需要从正确性、性能和灵活性多个角度对这种编码形式进行考察。
1. 即使你确定了默认的序列化形式是合适的，通常还必须提供一个readObject方法以保证约束关系和安全性。

**对象的物理表示法与它的逻辑数据内容有区别时**
1. 它将这个类的导出API永远地束缚在该类的内部表示法上。私有的StringList.Entry类变成了公有API的一部分，使得类永远也摆脱不掉维护链表项所需要的所有代码，即使它不再使用链表作为内部数据结构。
1. 它会消耗过多的空间。序列化形式保存了实现细节的链接关系。
1. 它会消耗过多的时间。序列化逻辑不了解对象图的拓扑关系，必须经过一个昂贵的突变里过程。
1. 它会引起栈溢出。默认的序列化过程要对对象图执行一次递归遍历。

```
public class StringList implements Serializable{
    private int size;
    private Entry head;

    private static class Entry implements Serializable{
        String data;
        Entry next;
        Entry previous;
    }
}
```

**默认的序列化不能描述对象的逻辑状态时自定义序列化**
1. transient修饰符表明这个实例字段将从一个类的默认序列化形式中省略掉。在反序列化的时候，这些字段将被初始化为默认值。
1. 私有的writeObjet和readObject通过反射机制可以自定义序列化形式。defaultWriteObject和defaultReadObject可以方便以后的发行版本中增加非transient的实例字段。
1. 序列化形式需要文档注释，哪怕是私有的也将作为API的一部分。

```
public class StringList implements Serializable{
    private transient int size;
    private transient Entry head;

    //不再需要实现序列化接口，因为不用默认的序列化形式。
    private static class Entry{
        String data;
        Entry next;
        Entry previous;
    }

    private void writeObject(ObjectOutputStream s){
        s.defaultWriteObject(); 
        s.wirteInt(size);
        for(Entry e=head,;e!=null;e=e.next){
            s.writeObject(e.data);
        }
    }

    private void readObject(ObjectInputStram s){
        s.defaultReadObject();
        size=s.readInt();
        for(int i=0;i<size;i++){
            entry.add((String)s.readObject());
        }
    }
}
```

4. 不管你选择了哪种序列化形式，都要为你自己编写的每个可序列化的类声明一个显示的序列版本UID。

## 保护性地编写readObject
1. readObject相当于另一个公有的构造器。构造器必须检查参数的有效性，必要的时候对参数进行保护性拷贝。
1. 当一个对象被反序列化的时候，对于客户端不应该拥有的对象引用，如果哪个字段包含了这样的对象引用，就必须要做保护性拷贝。
1. readObject方法不可以调用可被覆盖的方法，不然被覆盖的方法将在子类的状态被反序列化之前先运行，程序很可能会失败。

**健壮的readObject**
1. 对于对象引用域必须保持为私有的类，要保护性地拷贝这些域中的每个对象。不可变类的可变组件就属于这一类别。
1. 对于任何约束条件，如果检查失败，则抛出一个InvalidObjectException异常。这些检查动作应该跟在所有的保护性拷贝之后。
1. 如果整个对象图在被反序列化之后必须进行验证，就应该使用ObjectInputValidation接口。
1. 无论是直接方式还是间接方式，都不要调用类中任何可被覆盖的方法。

## 对于实例控制，枚举类型优先于readResolve。
1. readObject不管是显示还是默认的，都会返回一个新建的实例，这个实例不同于该类初始化时创建的实例。
1. readResolve特性允许你用另一个实例代替readObject创建的实例。对于一个正在被反序列化的对象，如果它的类定义了一个readResolve方法，并且具备正确的声明，那么在反序列化之后，新建对象上的readResolve方法就会被调用。然后，该方法返回的对象引用将被返回，取代新建的对象，新建对象的引用不需要再被保留。
1. 如果依赖readResolve进行实例控制，那么所有实例字段要么是基本类型，要么是transient。因为该方法忽略了被反序列化的对象，所以序列化形式并不需要包含任何是的数据。如果Singleton包含一个非transient对象引用字段，这个字段的内容就可以在Singleton的readResolve方法运行之前被反序列化，当对象引用字段的内容被反序列化时，它将允许一个精心制作的流盗用指向最初被反序列化的Singleton的引用。

```
private Object readResovle(){
    return INSTANCE;
}
```

4. 如果readResolve是私有的，则不能继承，如果是保护的或者公有的，如果子类没有覆盖，那么反序列化后将会产生一个超类，导致ClassCastException。

## 考虑用序列化代理代替序列化实例
1. 为可序列化的类设计一个私有的静态嵌套类，精确地表示外围类的实例的逻辑状态，这个嵌套类被称作序列化代理，它应该有一个单独的构造器，参数类型是外围类。
1. 外围类添加writeReplace方法，在序列化之前，产生代理类实例代替外围类的实例。这样将永远不会产生外围类的序列化实例，为了防止伪造攻击，可以禁用readObject。

```
private static class SerializationProxy implements Serializable{
    private final Date start;
    private final Date end;

    SerializationProxy(Period p){
        this.start=p.start;
        this.end=p.end;
    }

    private Object readResolve(){
        return new Period(start,end);
    }

    private static final long serialVersionUID=1;
}

private Object writeReplace(){
    return new  SerializationProxy(this);
}

private void readObject(ObjectInputStream stream) ObjectInputStream stream{
    throw new InvalidObjectException();
}
```

3. 序列化代理方法的缺点是增加了开销，但是可以阻止为字节流的攻击以及内部字段的盗用攻击。这种方法允许Period的字段为final的。