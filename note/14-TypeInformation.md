# 第14章 类型信息

**运行时类型信息（RTTI）使得你可以在程序运行时发现和使用类型信息。**

它使你从只能在编译期执行面向类型的操作的禁锢中解脱了出来，并且可以使用某些非常强大的程序。对RTTI的需要，揭示了面向对象设计中许多有趣（并且复杂）的问题，同时也提出了如何组织程序的问题。

**Java在运行时识别对象和类的信息的**。主要有两种方式：一种是 “传统的” RTTI，它假定我们在编译时已经知道了所有的类型；另一种是“反射”机制，它允许我们在运行时发现和使用类的信息。

----

## RTTI（Runtime Type Identification）运行阶段类型识别

### 用途

为了确定基类指针实际指向的子类的具体类型。

### 工作原理

通过类型转换运算符回答“是否可以安全地将对象的地址赋给特定类型的指针”这样的问题。

### Java中

在Java中，**所有的类型转换都是在运行时进行正确性检查的**。这也是RTTI的**含义**：在运行时，识别一个对象的类型。

#### 丢失具体类型信息的问题

1. 多态中表现的类型转换是RTTI最基本的使用形式，但这种转换并不彻底。如数组容器实际上将所有元素当作Object持有，取用时再自动将结果转型回声明类型。而数组在填充（持有）对象时，具体类型可能是声明类型的子类，这样放到数组里就会向上转型为声明类型，持有的对象就丢失了具体类型。而取用时将由Object只转型回声明类型，并不是具体的子类类型，所以这种转型并不彻底。
2. 多态中表现了具体类型的行为，但那只是“多态机制”的事情，是由引用所指向的具体对象而决定的，并不等价于在运行时识别具体类型。 

  以上揭示了一个问题就是具体类型信息的丢失！有了问题，就要解决问题，这就是RTTI的需要，即在运行时确定对象的具体类型。

#### 证实具体类型信息的丢失

```java
abstract class Shape{
    void draw(){
        System.out.println(this + ".draw()");
    }
    abstract public String toString();  //要求子类需要实现 toString()
}

class Circle extends Shape{
    @Override
    public String toString() {
        return "Circle";
    }
    public void drawCircle(){}
}

class Square extends Shape{
    @Override
    public String toString() {
        return "Square";
    }
}

class Triangle extends Shape{
    @Override
    public String toString() {
        return "Triangle";
    }
}
public class Shapes {
    public static  void main(String[] args){
        List<Shape> shapeList = Arrays.asList(
                new Circle(), new Square(), new Triangle()  // 向上转型为 Shape，此处会丢失原来的具体类型信息！！对于数组而言，它们只是Shape类对象！
        );
        for(Shape shape : shapeList){
            shape.draw();   // 数组实际上将所有事物都当作Object持有，在取用时会自动将结果转型回声明类型即Shape。
        }
        //shapeList.get(0).drawCircle(); //这里会编译错误：在Shape类中找不到符号drawCircle()，证实了具体类型信息的丢失!!
    }
}

```

##  Class对象

要理解RTTI在Java中的工作原理， 首先必须知道类型信息在运行时是如何表示的。 这项工作是由称为Class对象的特殊对象完成的 ， 它包含了与类有关的信息。 事实上 ， Class对象就是用来创建类的所有的 “常规" 对象的。 Java使用Class对象来执行其RTTI， 即使你正在执行的是类似转型这样的操作。Class类还拥有大量的使用RTTI的其他方式。

类是程序的一部分，每个类都有一个Class对象。 换言之，每当编写并且编译了一个新类，就会产生一个Class对象 （更恰当地说， 是被保存在一个同名的.class文件中 ）。为了生成这个类的对象，运行这个程序的Java虚拟机(JVM)将使用被称为“类加载器”的子系统。

类加载器子系统实际上可以包含一条类加载器链，但是只有一个原生类加载器，它是JVM 实现的一部分。原生类加载器加载的是所谓的可信类，包括Java API类，它们通常是从本地盘加载的。在这条链中，通常不需要添加额外的类加载器，但是如果你有特殊需求（例如以某种特殊的方式加载类，以支持Web服务器应用，或者在网络中下载类) ,那么你有一种方式可以挂接额外的类加载器。

所有的类都是在对其第一次使用时，动态加载到JVM中的。当程序创建第一个对类的静态成员的引用时，就会加载这个类。这个证明构造器也是类的静态方法，即使在构造器之前并没有使用static关键字。因此，使用new操作符创建类的新对象也会被当作对类的静态成员的引用。

因此，Java程序在它开始运行之前并非被完全加载，其各个部分是在必需时才加载的。动态加载使能的行为，在诸如c++这样的静态加载语言中是很难或者根本不可能复制的。

### 类加载器的工作过程

1. 类加载器首先检查这个类的Class对象是否已经加载。

2. 如果尚未加载，默认的类加载器就会根据类名查找.class文件（例如，某个附加类加载器可能会在数据库中查找字节码）。

3. 在这个类的字节码被加载时，它们会接受验证，以确保其没有被破坏，并且不包含不良Java代码（这是Java 中用于安全防范目的的措施之一)。

4. 一旦某个类的Class对象被载人内存，它就被用来创建这个类的所有对象。

   下面的示范程序可以证明这一点：

```java
class Candy {
  static { print("Loading Candy"); }
}

class Gum {
  static { print("Loading Gum"); }
}

class Cookie {
  static { print("Loading Cookie"); }
}

public class SweetShop {
  public static void main(String[] args) {	
    print("inside main");
    new Candy();
    print("After creating Candy");
    try {
      Class.forName("Gum");
    } catch(ClassNotFoundException e) {
      print("Couldn't find Gum");
    }
    print("After Class.forName(\"Gum\")");
    new Cookie();
    print("After creating Cookie");
  }
} /* Output:
inside main
Loading Candy
After creating Candy
Loading Gum
After Class.forName("Gum")
Loading Cookie
After creating Cookie
*///:~
```

特别有趣的一行是：`Class.forName("Gum");`

这个方法是Class类（所有Class对象都属于这个类）的一个static成员。Class对象就和其他对象一样，我们可以获取并操作它的引用（这也就是类加载器的工作）。`forName()`是取得Class 对象的引用的一种方法。它是用一个包含目标类的文本名（注意拼写和大小写）的String作输人参数，返回的是一个Class对象的引用，上面的代码忽略了返回值。对`forName()`的调用是为了它产生的“副作用"：如果类Gum还没有被加载就加载它。在加载的过程中，Gum的static子句被执行。

无论何时，只要你想在运行时使用类型信息，就必须首先获得对恰当的Class对象的引用。`Class.forName()`就是实现此功能的便捷途径，因为你不需要为了获得Class引用而持有该类型的对象。但是，如果你已经拥有了一个感兴趣的类型的对象，那就可以通过调用`getClass()`方法来获取Class引用了，这个方法属于根类Object的一部分，它将返回表示该对象的实际类型的Class引用。

Class包含很多有用的方法，下面是其中的一部分：

- `getName()`：产生全限定名。
- `getSimpleName()`：产生不含包名的类名。
- `getCanonicalName()`：产生全限定的类名。
- `isInterface()`
- `getInterface()`：返回Class对象，表示在感兴趣的Class对象中所包含的接口。
- `getSuperclass()`：查询其直接基类。
- `newInstance()`：是实现“虚拟构造器”的一种途径，虚拟构造器允许声明：“我不知道你的确切类型，但是无论如何要正确地创建你自己。”在前面的示例中，up仅仅只是一个Class引用，但是这个引用指向的是Toy对象。当然，在你可以发送Object能够接受的消息之外的任何消息之前，你必须更多地了解它，并执行某种转型。另外，使用newInstance()来创建的类，必须带有默认的构造器。

### 类字面常量

#### 使用类字面常量.class是获取Class对象引用的另一种方法。

如 `FancyToy.class`。建议使用这种方法。

- 编译时就会受到检查（因此不需要放到try语句块中），所以既简单又安全。根除了对`forName()`的调用，所以也更高效。
- 类字面常量`.class`不仅适用于普通的类，也适用于接口、数组和基本类型。
- 基本类型的包装器类有一个标准字段TYPE，它是一个引用，指向对应的基本数据类型的Class引用，即有`boolean.class` 等价于 `Boolean.TYPE`，`int.class` 等价于 `Integer.TYPE`…
- 注意，使用`.class`来创建Class对象的引用时，不会自动地初始化该Class对象。

#### 为了使用类而做的准备工作实际包含三个步骤

1. **加载**。这是由类加载器执行的。该步骤将查找字节码（通常在CLASSPATH所指定的路径中查找），并从这些字节码中创建一个Class对象。
2. **链接**。在链接阶段将验证类中的字节码，为静态域分配存储空间，并且如果必需的话，将解析这个类创建的对其他类的所有引用。
3. **初始化**。如果该类具有超类，则对其初始化，执行静态初始化器和静态初始块。

初始化被延迟到了对静态方法（构造器隐式地是静态的）或者非常数静态域进行初次引用时才执行。即初始化有效地实现了尽可能 的“**惰性**”。 

```java
package net.mrliuli.rtti;

import java.util.Random;

class Initable{
    static final int staticFinal = 47;      // 常数静态域
    static final int staticFinal2 = ClassInitialization.rand.nextInt(1000);     // 非常数静态域（不是编译期常量）
    static{
        System.out.println("Initializing Initable");
    }
}

class Initable2{
    static int staticNonFinal = 147;    // 非常数静态域
    static {
        System.out.println("Initializing Initable2");
    }
}

class Initable3{
    static int staticNonFinal = 74;     // 非常数静态域
    static {
        System.out.println("Initializing Initable3");
    }
}

public class ClassInitialization {

    public static Random rand = new Random(47);
    public static void main(String[] args) throws Exception {
        Class initalbe = Initable.class;                // 使用类字面常量.class获取Class对象引用，不会初始化
        System.out.println("After creating Initable ref");
        System.out.println(Initable.staticFinal);       // 常数静态域首次引用，不会初始化
        System.out.println(Initable.staticFinal2);      // 非常数静态域首次引用，会初始化
        System.out.println(Initable2.staticNonFinal);   // 非常数静态域首次引用，会初始化
        Class initable3 = Class.forName("net.mrliuli.rtti.Initable3");      // 使用Class.forName()获取Class对象引用，会初始化
        System.out.println("After creating Initable3 ref");
        System.out.println(Initable3.staticNonFinal);   // 已初始化过
    }

}
 /* Output:
After creating Initable ref
47
Initializing Initable
258
Initializing Initable2
147
Initializing Initable3
After creating Initable3 ref
74
*///:~
```

---

## 泛化的Class引用

**Class**引用总是指向某个**Class**对象，此时，这个**Class**对象可以是**各种类型**的，当使用**泛型语法**对Class引用所指向的Class对象的类型进行**限定**时，这就使得Class对象的类型变得**具体**，这样编译器**编译时**也会做一些额外的**类型检查**工作。

```java
public class GenericClassReferences {
    public static void main(String[] args){
        Class intClass = int.class;
        Class<Integer> genericIntClass = int.class;
        genericIntClass = Integer.class;    // Same thing
        intClass = double.class;
        // genericIntClass = double.class;  // Illegal, genericIntClass 限制为Integer 的Class对象
    }
}
```

---

### 使用通配符`?`放松一些限制

将通配符与extends关键字相结合如`Class<? extends Number>`，就创建了一个范围，使得这个Class引用被限定为Number类型或其子类型。

```java
public class BoundedClassReferences {
    public static void main(String[] args){
        Class<? extends Number> bounded = int.class;
        bounded = double.class;
        bounded = Number.class;
        // Or anything derived from Number
    }
}
```

下面的实例使用了泛型类语法。它存储了一个类引用，稍候又产生了一个List，填充这个List的对象是使用`newInstance()`方法，通过该引用生产的。

```java
class CountedInteger {
  private static long counter;
  private final long id = counter++;
  public String toString() { return Long.toString(id); }
}

public class FilledList<T> {
  private Class<T> type;
  public FilledList(Class<T> type) { this.type = type; }	
  public List<T> create(int nElements) {
    List<T> result = new ArrayList<T>();
    try {
      for(int i = 0; i < nElements; i++)
        result.add(type.newInstance());
    } catch(Exception e) {
      throw new RuntimeException(e);
    }
    return result;
  }
  public static void main(String[] args) {
    FilledList<CountedInteger> fl =
      new FilledList<CountedInteger>(CountedInteger.class);
    System.out.println(fl.create(15));
  }
} /* Output:
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14]
*///:~
```

### 总结，使用泛型类后

- 使得编译期进行类型检查，因此如果你操作有误，稍后立即就会发现这一点。
- `.newInstance()`将返回确切类型的对象，而不是`Object`对象

### 注意

```java
public class GenericToyTest {
  public static void main(String[] args) throws Exception {
    Class<FancyToy> ftClass = FancyToy.class;
    // Produces exact type:
    FancyToy fancyToy = ftClass.newInstance();
    Class<? super FancyToy> up = ftClass.getSuperclass();
    // This won't compile:
    // Class<Toy> up2 = ftClass.getSuperclass();
    // Only produces Object:
    Object obj = up.newInstance();
  }
}
```

`getSuperclass()`返回的是基类，因此是表达式`Class<? super FancyToy> up`接受`ftClass.getSuperclass()`，而不是`Class<Toy>`接受这样的声明。

### 较少使用的转型语法

`cast()`和`asSubclass()`方法。

---

## 类型转换前先做检查

RTTI形式包括：

1. 传统类型转换，如(Shape)
2. 代表对象的类型的Class对象.
3. 第三种形式，就是关键字 `instanceof`。它返回一个布尔值，告诉我们对象是不是某个特定类型或其子类。如`if(x instanceof Dog)`语句会检查对象x是否从属于Dog类。
4. 还一种形式是动态的`instanceof：Class.isInstance()`方法提供了一种动态地测试对象的途径。`Class.isInstance()`方法使我们不再需要`instanceof`表达式。

### `Class.isAssignableFrom()`

**`Class.isAssignableFrom()`** ：调用类型可以被参数类型赋值，即判断传递进来的参数是否属于调用类型继承结构（是调用类型或调用类型的子类）。