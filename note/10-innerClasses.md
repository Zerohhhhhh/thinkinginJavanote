# 第10章 内部类

## 创建内部类

如果想从外部类的非静态方法之外的任意位置创建某个内部类的对象，具体地指明这个对象的类型：

```java
OuterClassName.InnerClassName
```

```java
public class Outer {
    //内部类
    class Inner{
        private String str;
        Inner(String str){
            this.str = str;
        }
        String getStr(){
            return str;
        }
    }

    //外部类方法，返回内部类对象
    private Inner toInner(String str){
        return new Inner(str);
    }

    public static void main(String[] args) {
        Outer outer = new Outer();
        //创建内部类对象：OuterClassName.InnerClassName
        Outer.Inner inner = outer.toInner("it's inner");
        Print.print(inner.getStr());
    }
}
```

## 链接到外部类

当生成一个内部类的对象时，此对象与制造它的外围对象（enclosing object）之间就有了一种联系，所以**内部类的对象能访问其外围对象的所有成员，而不需要任何特殊条件**。此外，内部类还拥有外围类的所有元素的访问权。以下例子是“迭代器”设计模式的一个例子。

```java
interface Selector{
    boolean end();
    Object current();
    void next();
}

public class Sequence {
    private Object[] items;
    private int next = 0;
    private Sequence(int size){ items = new Object[size]; }
    private void add(Object x){
        if (next < items.length)
            items[next++] = x;
    }
    private class SequenceSelector implements Selector{
        //内部类直接调用其外围类的方法和字段，就像自己拥有它们似的。
        private int i = 0;
        @Override
        public boolean end() { return i == items.length; }
        @Override
        public Object current() { return items[i]; }
        @Override
        public void next() {
            if (i < items.length)
                i++;
        }
    }
    private Selector selector(){
        return new SequenceSelector();
    }

    public static void main(String[] args) {
        Sequence sequence = new Sequence(10);
        for (int i = 0;i<10;i++){
            sequence.add(Integer.toString(i));
        }
        Selector selector = sequence.selector();
        while (!selector.end()){
            System.out.print(selector.current() + " ");
            selector.next();
        }
    }
}


/**
 * Output
 * 0 1 2 3 4 5 6 7 8 9
 */
```

内部类如何做到自动拥有对其外围类所有成员的访问权？当某个外围类的对象创建了一个内部类对象时，此内部类对象必定会秘密捕获一个指向那个外围类对象的引用。然后，在访问此外围类的成员时，就是用那个引用来选择外围类的成员。构建内部类对象时，需要一个指向其外围类对象的引用，如果编译器访问不到这个引用就会报错。

## 使用.this与.new

如果你需要生成对外部类对象的引用，可以使用外部类的名字后面紧跟圆点和this。**这样产生的引用自动地具有正确地类型，这一点在编译期就被知晓并受到检查，因此没有任何运行时开销**。

```java
public class DotThis {
    void f(){
        Print.print("DotThis.f()");
    }
    public class Inner{
        public DotThis outer(){
            //生成对外部类对象的引用，可以使用外部类的名字后面紧跟圆点和this
            return DotThis.this;
        }
    }
    public Inner inner(){
        return new Inner();
    }
    public static void main(String[] args) {
        DotThis dotThis = new DotThis();
        DotThis.Inner dotThisInner = dotThis.inner();
        dotThisInner.outer().f();
    }
/**
 * output
 * DotThis.f()
 */
}
```

有时想要告知某些其他对象，去创建其某个内部类的对象。要实现此目的，你必须在new表达式中提供对其他外部类对象的引用，这是需要使用new语法。在以上程序的main方法中添加下面代码：

```java
//使用.new创建内部类对象
DotThis.Inner dotThisInner1 = dotThis.new Inner();
dotThisInner1.outer().f();
```

其运行结果与上述相同。

要想直接创建内部类的对象，不能去引用外部类的名字，而是必须使用外部类的对象来创建内部类对象。

在拥有外部类对象之前是不可能创建内部类对象的。这是因为内部类对象会暗暗地连接到创建它的外部类对象上。如果创建的是嵌套类（静态内部类），那么它就不需要对外部类对象的引用。

## 内部类与向上转型

当将内部类向上转型为其基类，尤其是转型为一个接口的时候，内部类就有了用武之地。（从实现了某个接口的对象，得到对此接口的引用，与向上转型为这个对象的基类，实质上效果是一样的）。

## 在方法和作用域内的内部类

1. 内部类在方法内时，在该方法外无法被访问。
2. 内部类在作用域内时，在该作用域外无法被访问。
3. 内部类无论在哪里都会和其他类一起被编译。

----

## 匿名内部类

1. **无参数构造器**

```java
public class Parcel7 {
    public Contents contents(){
        return new Contents() { //匿名内部类
            private int i = 11;
            @Override
            public int value() {
                return i;
            }
        };
    }

    public static void main(String[] args){
        Parcel7 p = new Parcel7();
        Contents contents = p.contents();
        Print.print(contents.value());
    }
}
```

contents方法将返回值的生成与表示这个返回值的类的定义结合在一起。同时，这个类是匿名的。这种语法指的是：创建一个继承自Contents的匿名类的对象。通过new表达式返回的引用被自动向上转型为对Contents的引用。

2. **有参数构造器**

```java
public class Parcel8 {
    public Wrapping wrapping(int x){
       return new Wrapping(x){
           @Override
           public int value() {
               return super.value() * 47;
           }
       };
    }
    public static void main(String[] args) {
        Parcel8 p = new Parcel8();
        Wrapping w = p.wrapping(10);
        Print.print(w.value());
    }
}
```

```java
public class Wrapping {
    private int i;

    public Wrapping(int i) {
        this.i = i;
    }
    public int value(){
        return i;
    }
}
```

在匿名内部类末尾的分号，并不是用来标记此内部类结束的。实际上，它标记的是表达式的结束，只不过这个表达式正巧包含了匿名内部类。

3. **在匿名内部类中定义字段时，能够对其执行初始化操作**

```java
interface Destination{
    String readLabel();
}
public class Parcel9 {

    public Destination destination(final String dest){
        return new Destination() {
            private String label = dest;
            @Override
            public String readLabel() {
                return label;
            }
        };
    }

    public static void main(String[] args) {
        Parcel9 parcel9 = new Parcel9();
        Destination destination = parcel9.destination("Tasmania");
        Print.print(destination.readLabel());
    }
}

```

如果定义一个匿名内部类，并且希望它使用一个在其外部定义的对象，那么编译器会要求其参数引用是final的**（使用jdk8时不用final也不会出错，可能jdk8改进了，有待考证）**。

4. **为匿名内部类创建一个构造器**

```java
abstract class Base{
    public Base(int i){
        Print.print("Base constructor,i="+ i);
    }
    public abstract void f();
}

public class AnonymousConstructor {
    public static Base getBase(int i){
        return new Base(i) {
            {
                Print.print("Inside instance initializer");
            }
            @Override
            public void f() {
                Print.print("in anonymous f()");
            }
        };
    }

    public static void main(String[] args) {
        Base base = getBase(47);
        base.f();
    }
    /**
     * output
     * Base constructor,i=47
     * Inside instance initializer
     * in anonymous f()
     */
}

```

在匿名类中不可能有命名构造器，但通过**实例初始化**，能够达到为匿名内部类创建一个构造器的效果。

在此例中，不要求变量i一定是final的。因为i被传递给匿名类的基类的构造器，它并不会在匿名类内部被直接使用。

```java
public class Parcel10 {
    public Destination destination(final String dest,final float price){
        return new Destination() {
            private int cost;
            {
                cost = Math.round(price);
                if (cost > 100){
                    Print.print("Over budget");
                }
            }
            private String label = dest;
            @Override
            public String readLabel() {
                return label;
            }
        };
    }

    public static void main(String[] args) {
        Parcel10 parcel10 = new Parcel10();
        Destination destination = parcel10.destination("Tasmania",101.2143242f);
        Print.print(destination.readLabel());
    }
}

```

在实例初始化操作的内部，if语句不能作为字段初始化动作的一部分来执行。所以对于匿名类而言，实例初始化的实际效果就是构造器。当然它受到了限制——不能重载实例初始化方法，所以你仅有一个这样的构造器。

匿名内部类与正规的继承相比有些限制，因为匿名内部类既可以扩展类，也可以实现接口，但是不能两者兼备。而且如果是实现接口，也只能实现一个接口。

-----

### 再访工厂方法

```java
interface Service {
  void method1();
  void method2();
}

interface ServiceFactory {
  Service getService();
}	

class Implementation1 implements Service {
  private Implementation1() {}
  public void method1() {print("Implementation1 method1");}
  public void method2() {print("Implementation1 method2");}
  public static ServiceFactory factory =
    new ServiceFactory() {
      public Service getService() {
        return new Implementation1();
      }
    };
}	

class Implementation2 implements Service {
  private Implementation2() {}
  public void method1() {print("Implementation2 method1");}
  public void method2() {print("Implementation2 method2");}
  public static ServiceFactory factory =
    new ServiceFactory() {
      public Service getService() {
        return new Implementation2();
      }
    };
}	

public class Factories {
  public static void serviceConsumer(ServiceFactory fact) {
    Service s = fact.getService();
    s.method1();
    s.method2();
  }
  public static void main(String[] args) {
    serviceConsumer(Implementation1.factory);
    // Implementations are completely interchangeable:
    serviceConsumer(Implementation2.factory);
  }
} /* Output:
Implementation1 method1
Implementation1 method2
Implementation2 method1
Implementation2 method2
*///:~
```

---

## 嵌套类

如果不需要内部类对象与其外围类对象之间有联系 ， 那么可以将内部类声明为static。 这通常称为嵌套类 。  普通的内部类对象隐式地保存了一个引用， 指向创建它的外围类对象。 然而， 当内部类是static的时 ， 就不是这样了。嵌套类意味着：

- 要创建嵌套类的对象，并不需要其外围类的对象。
- 不能从嵌套类的对象中访问非静态的外围类对象。

**嵌套类与普通的内部类还有一个区别。** 普通内部类的字段与方法 ， 只能放在类的外部层次上， 所以普通的内部类不能有static数据和static字段 ， 也不能包含嵌套类。 但是嵌套类可以包含所有这些东西。

在一个普通的（非static）内部类中，通过一个特殊的this引用可以链接到其外围类对象。嵌套类就没有这个特殊的this引用，这使得它类似于一个static方法。

---

### 接口内部的类

正常情况下，不能再接口内部放置任何代码，但嵌套类可以作为接口的一部分。放到接口中的任何类都自动地是public和static的。因为类是static的，只是将嵌套类置于接口的命名空间内，这并不违反接口的规则。甚至可以在内部类中实现其外围接口。

```java
public interface ClassInInterface {
    void howdy();
    class Test implements ClassInInterface{
        @Override
        public void howdy() {
            Print.print("howdy");
        }

        public static void main(String[] args) {
            ClassInInterface classInInterface = new Test();
            classInInterface.howdy();
            new Test().howdy();
        }
    }
}
```

如果想要创建某些公共代码，使得他们可以被某个接口的所有不同实现所共用，那么使用接口内部的嵌套类会显得很方便。

---

### 使用嵌套类来放置测试代码

在每个类中都写一个main方法，用来测试这个类，这样做有一个缺点，那就是必须带着那些已编译过的额外代码。使用嵌套类来放置测试代码可以解决这个问题。

```java
public class TestBed {
    public void f(){
        Print.print("f()");
    }
    public static class Tester{
        public static void main(String[] args) {
            TestBed t = new TestBed();
            t.f();
        }
    }
}
```

---

### 从多层嵌套类中访问外部类的成员

一个内部类被嵌套多少层并不重要，它能透明地访问所有它所嵌入的外围类的所有成员。

---

## 为什么需要内部类

一般说来，内部类继承自某个类或实现某个接口，内部类的代码操作创建它的外围类的对象。所以可以认为**内部类提供了某种进人其外围类的窗口**。

内部类必须要回答的一个问题是：如果只是需要一个对接口的引用，为什么不通过外围类实现那个接口呢？答案是：“如果这能满足需求，那么就应该这样做。”那么内部类实现一个接口与外围类实现这个接口有什么区别呢？答案是：后者不是总能享用到接口带来的方便，有时需要用到接口的实现。所以，**使用内部类最吸引人的原因是**：

**<u>每个内部类都能独立地继承自一个（接口的）实现，所以无论外围类是否已经承了某个（接口的）实现，对于内部类都没有影响</u>**。

如果没有内部类提供的、可以继承多个具体的或抽象的类的能力，一些设计与编程问题就很难解决。从这个角度看，**<u>内部类使得多重继承的解决方案变得完整</u>**。接口解决了部分问题，而内部类有效地实现了“多重继承”。**也就是说，内部类允许继承多个非接口类型**（类或抽象类)。

为了看到更多的细节，让我们考虑这样两种情形：

1. 即必须在一个类中以某种方式实现两个接口。由于接口的灵活性，你有两种选择：使用单一类，或者使用内部类。

```java
interface A {}
interface B {}

class X implements A, B {}

class Y implements A {
  B makeB() {
    // Anonymous inner class:
    return new B() {};
  }
}

public class MultiInterfaces {
  static void takesA(A a) {}
  static void takesB(B b) {}
  public static void main(String[] args) {
    X x = new X();
    Y y = new Y();
    takesA(x);
    takesA(y);
    takesB(x);
    takesB(y.makeB());
  }
}
```

2. 如果拥有的是抽象的类或具体的类，而不是接口，那就只能使用内部类才能实现多重继承。

```java
class D {}
abstract class E {}

class Z extends D {
  E makeE() { return new E() {}; }
}

public class MultiImplementation {
  static void takesD(D d) {}
  static void takesE(E e) {}
  public static void main(String[] args) {
    Z z = new Z();
    takesD(z);
    takesE(z.makeE());
  }
} 
```

---

如果不需要解决 “多重继承' 的问题 ， 那么自然可以用别的方式编码 ， 而不需要使用内部类。但如果使用内部类，还可以获得其他一些特性：

1. 内部类可以有多个实例 ， 每个实例都有自己的状态信息， 并且与其外围类对象的信息相互独立。
2. 在单个外围类中 ， 可以让多个内部类以不同的方式实现向一个接口 ， 或继承同一个类。稍后就会展示一个这样的例子。
3.  创建内部类对象的时刻并不依赖于外围类对象的创建。
4. 内部类并没有令人迷惑的 "is-a" 关系 ； 它就是一个独立的实体。

举个例子， 如果Sequence.java不使用内部类 ， 就必须声明 "Sequence是一个Selector", 对于某个特定的Sequence只能有一个Selector。然而使用内部类很容易就能拥有另一个方法 reverseSelector()，用它来生成一个反方向遍历序列的Selector“。只有内部类才有这种灵活性。

---

### 闭包与回调

**闭包 (closure)** 是一个可调用的对象 ， 它记录了一些信息 ， 这些信息来自于创建它的作用域。 通过这个定义 ， 可以看出内部类是面向对象的闭包， 因为它不仅包含外围类对象 （创建内部类的作用域） 的信息 ， 还自动拥有一个指向此外围类对象的引用 ， 在此作用域内 ， 内部类有权操作所有的成员 ， 包括private成员。

**回调  (callback)**，通过回调，对象能够携带一些信息，这些信息允许它在稍后的某个时刻调用初始的对象。Java中没有指针，通过内部类提供的闭包功能可以实现回调。

```java
// Callbacks.java
// using inner classes for callbacks

interface Incrementable{
    void increment();
}

// Very simple to just implement the interface:
class Callee1 implements Incrementable{
    private int i = 0;
    public void increment(){
        System.out.println(++i);
    }
}

class MyIncrement{
    public void increment(){ System.out.println("Other operation"); }
    static void f(MyIncrement mi) { mi.increment(); }
}

// If your class must implement increment() in some other way, you must use an inner class:
class Callee2 extends MyIncrement{
    private int i = 0;
    public void increment(){
        super.increment();
        System.out.println(++i);
    }
    private class Closure implements Incrementable{
        public void increment(){
            // Specify outer-class method, otherwise you'd get an infinite recursion:
            Callee2.this.increment();
        }
    }
    Incrementable getCallbackReference(){
        return new Closure();
    }
}

class Caller{
    private Incrementable callbackReference;
    Caller(Incrementable cbh){ callbackReference = cbh; }
    void go(){ callbackReference.increment(); }
}

public class Callbacks {
    public static void main(String[] args){
        Callee1 c1 = new Callee1();
        Callee2 c2 = new Callee2();
        MyIncrement.f(c2);
        Caller caller1 = new Caller(c1);
        Caller caller2 = new Caller(c2.getCallbackReference());
        caller1.go();
        caller1.go();
        caller2.go();
        caller2.go();
    }
}
/**Output:
* Other operation
* 1
* 1
* 2
* Other operation
* 2
* Other operation
* 3
**/
```

`回调的价值在于它的灵活性，可以在运行时动态地决定需要调用什么方法。`

