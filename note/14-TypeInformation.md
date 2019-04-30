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

----

## 注册工厂

## `instanceof` 与 Class 的等价性

- `instanceof` 和 `isInstance()` 保持了类型的概念，它指的是“**你是这个**类吗，或者你是这个类的派生类吗？”
- 使用`==` 和 `equals()` 比较实际的Class对象，没有考虑继承——它要么是这个**确切的类型**，要么不是。

---

## 反射：运行时的类信息（Reflection: runtime class information）

### 工作原理

Class类与`java.lang.reflect`类库一起对反射的概念进行了支持，该类库包含了Field、Method以Constructor类 （每个类都实现了Member接口 ) O。这些类型的对象是由JVM在运行时创建的 ， 用以表示未知类里对应的成员。 这样你就可以使用Constructor创建新的对象， 用 `get()`和`set()`方法读取和修改与Field对象关联的字段 ， 用`invoke()`方法调用与Method对象关联的方法。 另外，还可以调用`getField()`、`getMethods()`和`getConstructors()`等很便利的方法，以返回表示字段、 方法以及构造器的对象的数组 （在JDK文档中 ， 通过查找Class类可了解更多相关资料）。这样， 匿名对象的类信息就能在运行时被完全确定下来 ， 而在编译时不需要知道任何事情。

### RTTI与反射的真正区别在于：

- 对于RTTI来说，是**编译时**打开和检查.class文件。（换句话说，我们可以用“普通”方式调用对象的所有方法。）
- 对于反射机制来说，.class文件在编译时是不可获取的，所以是在**运行时**打开和检查.class文件。

### 类方法提取器

类方法提取器能够动态地提取某个类的信息。

```java
public class ShowMethods {
  private static String usage =
    "usage:\n" +
    "ShowMethods qualified.class.name\n" +
    "To show all methods in class or:\n" +
    "ShowMethods qualified.class.name word\n" +
    "To search for methods involving 'word'";
  private static Pattern p = Pattern.compile("\\w+\\.");
  public static void main(String[] args) {
    if(args.length < 1) {
      print(usage);
      System.exit(0);
    }
    int lines = 0;
    try {
      Class<?> c = Class.forName(args[0]);
      Method[] methods = c.getMethods();
      Constructor[] ctors = c.getConstructors();
      if(args.length == 1) {
        for(Method method : methods)
          print(
            p.matcher(method.toString()).replaceAll(""));
        for(Constructor ctor : ctors)
          print(p.matcher(ctor.toString()).replaceAll(""));
        lines = methods.length + ctors.length;
      } else {
        for(Method method : methods)
          if(method.toString().indexOf(args[1]) != -1) {
            print(
              p.matcher(method.toString()).replaceAll(""));
            lines++;
          }
        for(Constructor ctor : ctors)
          if(ctor.toString().indexOf(args[1]) != -1) {
            print(p.matcher(
              ctor.toString()).replaceAll(""));
            lines++;
          }
      }
    } catch(ClassNotFoundException e) {
      print("No such class: " + e);
    }
  }
} /* Output:
public static void main(String[])
public native int hashCode()
public final native Class getClass()
public final void wait(long,int) throws InterruptedException
public final void wait() throws InterruptedException
public final native void wait(long) throws InterruptedException
public boolean equals(Object)
public String toString()
public final native void notify()
public final native void notifyAll()
public ShowMethods()
*///:~
```

#### 要点

- Class的`getMethod()`和`getConstructors()`方法分别返回Method对象的数组和Constructor对象的数组。
- `Class.forName()`生产的结果在编译时是不可知的，因此所有的方法特征签名信息都是在执行时被提取出来的。
- 如果将`showMethods`作为一个非public的类（也就是拥有包访问权限），输出中就不会在显示出这个自动合成的默认构造器了。该自动合成的默认构造器会自动被赋予与类一样的访问权限。

---

## 动态代理

### 静态代理

代理是基本的设计模式之一，它为了能够提供额外的或不同的操作，而插入的用来代替“实际”对象的对象。这些操作通常涉及与“实际”对象的通信，因此代理通常充当着中间人的角色。

```java
interface Interface {
  void doSomething();
  void somethingElse(String arg);
}

class RealObject implements Interface {
  public void doSomething() { print("doSomething"); }
  public void somethingElse(String arg) {
    print("somethingElse " + arg);
  }
}	

class SimpleProxy implements Interface {
  private Interface proxied;
  public SimpleProxy(Interface proxied) {
    this.proxied = proxied;
  }
  public void doSomething() {
    print("SimpleProxy doSomething");
    proxied.doSomething();
  }
  public void somethingElse(String arg) {
    print("SimpleProxy somethingElse " + arg);
    proxied.somethingElse(arg);
  }
}	

class SimpleProxyDemo {
  public static void consumer(Interface iface) {
    iface.doSomething();
    iface.somethingElse("bonobo");
  }
  public static void main(String[] args) {
    consumer(new RealObject());
    consumer(new SimpleProxy(new RealObject()));
  }
} /* Output:
doSomething
somethingElse bonobo
SimpleProxy doSomething
doSomething
SimpleProxy somethingElse bonobo
somethingElse bonobo
*///:~
```

因为`consumer()`接受的Interface,所以它无法知道正在获得的到底是`RealObject`还是`SimpleProxy`，因为这二者都实现了Interface。但是`Simpleproxye`经被插人到了客户端和`RealObject`之间，因此它会执行操作，然后调用`RealObject`上相同的方法。

### 动态代理

Java的动态代理比代理的思想更向前迈进了一步，因为它可以动态地创建代理并动态地处理对所代理方法的调用。在动态代理上所做的所有调用都会被重定向到单一的调用处理器上，它的工作是揭示调用的类型并确定相应的对策。下面是用动态代理重写的Demo：

```java
class DynamicProxyHandler implements InvocationHandler {
  private Object proxied;
  public DynamicProxyHandler(Object proxied) {
    this.proxied = proxied;
  }
  public Object
  invoke(Object proxy, Method method, Object[] args)
  throws Throwable {
    System.out.println("**** proxy: " + proxy.getClass() +
      ", method: " + method + ", args: " + args);
    if(args != null)
      for(Object arg : args)
        System.out.println("  " + arg);
    return method.invoke(proxied, args);
  }
}	

class SimpleDynamicProxy {
  public static void consumer(Interface iface) {
    iface.doSomething();
    iface.somethingElse("bonobo");
  }
  public static void main(String[] args) {
    RealObject real = new RealObject();
    consumer(real);
    // Insert a proxy and call again:
    Interface proxy = (Interface)Proxy.newProxyInstance(
      Interface.class.getClassLoader(),
      new Class[]{ Interface.class },
      new DynamicProxyHandler(real));
    consumer(proxy);
  }
} /* Output: (95% match)	
doSomething
somethingElse bonobo
**** proxy: class $Proxy0, method: public abstract void Interface.doSomething(), args: null
doSomething
**** proxy: class $Proxy0, method: public abstract void Interface.somethingElse(java.lang.String), args: [Ljava.lang.Object;@42e816
  bonobo
somethingElse bonobo
*///:~
```

#### 工作原理

通过调用静态方法`Proxy.newProxyInstance()`可以创建动态代理，这个方法需要得到一个类加载器（你通常可以从已经被加载的对象中获取其类加载器，然后传递给它),一个你希望该代理实现的接口列表（不是类或抽象类），以及`InvocationHandler`接口的一个实现。动态代理可以将所有调用重定向到调用处理器，因此通常会向调用处理器的构造器传递给一个“实际”对象的引用，从而使得调用处理器在执行其中介任务时，可以将请求转发。

`invoke()`方法中传递进来了代理对象，以防你需要区分请求的来源，但是在许多情况下，你并不关心这一点。然而，在`invoke()`内部，在代理上调用方法时需要格外当心，因为对接口的调用将被重定向为对代理的调用。  

#### 常用方式

通常，你会执行被代理的操作，然后使用`Method.invoke()`将请求转发给被代理对象，并传人必需的参数。这初看起来可能有些受限，就像你只能执行泛化操作一样。但是，你可以通过传递其他的参数，来过滤某些方法调用。

### 动态代理的优点及美中不足

- 优点：动态代理与静态代理相较，最大的好处是接口中声明的所有方法都被转移到调用处理器一个集中的方法（`InvocationHandler.invoke`）中处理。这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样每一个方法进行中转。

- 美中不足：它始终无法摆脱仅支持interface代理的桎梏，因为它的设计注定了这个遗憾。

---

## 空对象

为了减少使用内置的null表示缺少的对象时，有时引入空对象的思想将会很有用，它可以接受传递给它的所代表的对象的消息，但是将返回表示为实际上并不存在任何“真实”对象的值。

空对象最有用之处在于它更靠近数据，因为对象表示的是空间内的实体。

下例为空对象和动态代理相结合的例子：

 ```java
public interface Operation {
  String description();
  void command();
} ///:~
 ```

```java
public interface Robot {
  String name();
  String model();
  List<Operation> operations();
  class Test {
    public static void test(Robot r) {
      if(r instanceof Null)
        System.out.println("[Null Robot]");
      System.out.println("Robot name: " + r.name());
      System.out.println("Robot model: " + r.model());
      for(Operation operation : r.operations()) {
        System.out.println(operation.description());
        operation.command();
      }
    }
  }
} ///:~
```

```java
public class SnowRemovalRobot implements Robot {
  private String name;
  public SnowRemovalRobot(String name) {this.name = name;}
  public String name() { return name; }
  public String model() { return "SnowBot Series 11"; }
  public List<Operation> operations() {
    return Arrays.asList(
      new Operation() {
        public String description() {
          return name + " can shovel snow";
        }
        public void command() {
          System.out.println(name + " shoveling snow");
        }
      },	
      new Operation() {
        public String description() {
          return name + " can chip ice";
        }
        public void command() {
          System.out.println(name + " chipping ice");
        }
      },
      new Operation() {
        public String description() {
          return name + " can clear the roof";
        }
        public void command() {
          System.out.println(name + " clearing roof");
        }
      }
    );
  }	
  public static void main(String[] args) {
    Robot.Test.test(new SnowRemovalRobot("Slusher"));
  }
} /* Output:
Robot name: Slusher
Robot model: SnowBot Series 11
Slusher can shovel snow
Slusher shoveling snow
Slusher can chip ice
Slusher chipping ice
Slusher can clear the roof
Slusher clearing roof
*///:~
```

```java
// Using a dynamic proxy to create a Null Object.
class NullRobotProxyHandler implements InvocationHandler {
  private String nullName;
  private Robot proxied = new NRobot();
  NullRobotProxyHandler(Class<? extends Robot> type) {
    nullName = type.getSimpleName() + " NullRobot";
  }
  private class NRobot implements Null, Robot {
    public String name() { return nullName; }
    public String model() { return nullName; }
    public List<Operation> operations() {
      return Collections.emptyList();
    }
  }	
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    return method.invoke(proxied, args);
  }
}

public class NullRobot {
  public static Robot newNullRobot(Class<? extends Robot> type) {
    return (Robot)Proxy.newProxyInstance(NullRobot.class.getClassLoader(), new Class[]{ Null.class, Robot.class },
     new NullRobotProxyHandler(type));
  }	
  public static void main(String[] args) {
    Robot[] bots = {
      new SnowRemovalRobot("SnowBee"),
      newNullRobot(SnowRemovalRobot.class)
    };
    for(Robot bot : bots)
      Robot.Test.test(bot);
  }
} /* Output:
Robot name: SnowBee
Robot model: SnowBot Series 11
SnowBee can shovel snow
SnowBee shoveling snow
SnowBee can chip ice
SnowBee chipping ice
SnowBee can clear the roof
SnowBee clearing roof
[Null Robot]
Robot name: SnowRemovalRobot NullRobot
Robot model: SnowRemovalRobot NullRobot
*///:~
```

### YAGNI

极限编程（XP）的原则之一，YAGNI（You Aren’t Going to Need It，你永不需要它），即“做可以工作的最简单的事情”。

###  模拟对象与桩（Mock Objects & Stubs）

空对象的逻辑变体是*模拟对象*和*桩*。空对象的逻辑变体是模拟对象和桩。与空对象一样，它们都表示在最终的程序中所使用的“实际”对象。但是，模拟对象和桩都只是假扮可以传实际信息的存活对象，而不是像空对象那样可以成为null的一种更加智能化的替代物。

模拟对象和桩之间的差异在于程度不同。模拟对象往往是轻量级和自测试的，通常很多模拟对象被创建出来是为了处理各种不同的测试情况。桩只是返回桩数据，它通常是重量级的，并且经常在测试之间被复用。桩可以根据它们被调用的方式，通过配置进行修改，因此桩是一种复杂对象，它要做很多事。然而对于模拟对象，如果你需要做很多事情，通常会创建大量小而简单的模拟对象。

## 接口与类型信息

- 通过使用反射，仍旧可以到达并调用所有方法，甚至是private方法。因此，任何人都可以获取你最私有的方法的名字和签名，然后调用它们。
- 即使将接口实现为一个私有内部类或匿名类，通过使用反射，依旧可以到达并调用所有方法。因此，没有任何方式可以阻止反射到达并调用那些非公共访问权限的方法。对于域来说，的确如此，即便是private域。
- final域实际上在遭遇使用反射修改时是安全的。运行时系统会在不抛异常的情况下接受任何修改尝试，但是实际上不会发生任何修改。

```java
public class ModifyingPrivateFields {
  public static void main(String[] args) throws Exception {
    WithPrivateFinalField pf = new WithPrivateFinalField();
    System.out.println(pf);
    Field f = pf.getClass().getDeclaredField("i");
    f.setAccessible(true);
    System.out.println("f.getInt(pf): " + f.getInt(pf));
    f.setInt(pf, 47);
    System.out.println(pf);
    f = pf.getClass().getDeclaredField("s");
    f.setAccessible(true);
    System.out.println("f.get(pf): " + f.get(pf));
    f.set(pf, "No, you're not!");
    System.out.println(pf);
    f = pf.getClass().getDeclaredField("s2");
    f.setAccessible(true);
    System.out.println("f.get(pf): " + f.get(pf));
    f.set(pf, "No, you're not!");
    System.out.println(pf);
  }
} /* Output:
i = 1, I'm totally safe, Am I safe?
f.getInt(pf): 1
i = 47, I'm totally safe, Am I safe?
f.get(pf): I'm totally safe
i = 47, I'm totally safe, Am I safe?
f.get(pf): Am I safe?
i = 47, I'm totally safe, No, you're not!
*///:~

```

