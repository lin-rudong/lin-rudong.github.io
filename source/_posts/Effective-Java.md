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
