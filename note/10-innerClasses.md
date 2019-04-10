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