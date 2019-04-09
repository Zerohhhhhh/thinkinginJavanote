# 第9章 接口

***接口和内部类为我们提供了一种将接口与实现分离的更加结构化的方法。***

## 抽象类和抽象方法

创建抽象类的目的：希望通过这个通用接口操纵一系列类。

java提供一个叫做抽象方法的机制，这种方法是不完整的；仅有声明而没有方法体。抽象方法声明的语句：

```java
abstract void f();
```

包含抽象方法的类叫做抽象类。如果一个类包含一个或多个抽象方法，该类必须被限定为抽象的（即abstract class）。抽象类也可以有具体方法。

如果从一个抽象类继承，并想创建该新类的对象，那么就必须为基类中的所有抽象方法提供方法定义。如果不这样做（可以不做），那么导出类也是抽象类，且编译器将会强制用abstract关键字来限定这个类。

### 抽象类和抽象方法的作用

1. 创建抽象类和抽象方法非常有用，因为它们可以使类的抽象性明确起来，并告诉用户和编译器打算怎样来使用它们。
2. 抽象类还是很有用的重构工具，因为它们使得我们可以很容易地将公共方法沿着继承层次结构向上移动。

## 接口

abstract关键字允许人们在类中创建一个或多个没有任何定义的方法一一提供了接口部分，但是没有提供任何相应的具体实现，这些实现是由此类的继承者创建的。

interface这个关键字产生一个完仝抽象的类，它根本就没有提供任何具体实现。它允许创建者确定方法名、参数列表和返回类型，但是没有任何方法体。接口只提供了形式，而未提供任何具体实现。

**任何使用某特定接口的代码都知道可以调用该接口的哪些方法，而且仅需知道这些。因此，接口被用来建立类与类之间的协议**。

要想创建一个接口，需要用interface关键字来替代class关键字。**就像类一样，可以在interface关键字前面添加public关键字（但仅限于该接口在与其同名的文件中被定义)。如果不添加public关键字，则它只具有包访问权限，这样它就只能在同一个包内可用**。

**接口也可以包含域，但是这些域隐式地是static和final的**。

要让一个类遵循某个特定接口（或者是一组接口），需要使用implements关键字。

可以选择在接口中显式地将方法声明为public的，但即使不这么做，它们也是public的。当要实现一个接口时，在接口中被定义的方法必须被定义为是public的；否则，它们将只能得到默认的包访问权限，这样在方法被继承的过程中，其可访问权限就被降低了。

## 完全解耦P174

只要一个方法操作的是类而非接口，那么你就只能使用这个类及其子类。如果你想要将这个方法应用于不再此继承结构中的某个类，那么你就会触霉头了。接口可以在很大程度上放宽这种限制，因此，它使得我们可以编写可复用性更好的代码。

将接口从具体实现中解耦使得接口可以应用于多种不同的具体实现，因此代码也就更具可复用性。

###  策略设计模式：

创建一个能够根据所传递的参数对象的不同而具有不同行为的方法。这类方法包含所要执行的算法中固定不变的部分，而“策略”包含变化的部分。

### 适配器设计模式

适配器中的代码将接受你所拥有的接口，并产生你所需要的接口。

## Java中的多重继承

java中，只能继承一个类，可以实现多个接口。所有接口名都置于implements关键字之后，用逗号将他们一一隔开。实现多个接口，意味着可以向上转型为每个接口，每一个接口都是一个独立类型。

### 使用多重继承的核心原因：

为里能够向上转型为多个基类型（以及由此而带来的灵活性）。同时防止客户端程序员创建该类的对象。

## 通过继承来扩展接口

一般情况下，只可以将extends用于单一类，但是可以引用多个基类接口。如：有接口A,B,C，则以下语法正确，仅限于extends接口。

```java
interface C extends A,B{}
```

## 适配接口

接口最吸引人的原因之一就是允许同一个接口具有多个不同的具体实现。它的体现形式通常是一个接受接口类型的方法，而该接口的实现和向该方法传递的对象则取决于方法的使用者。

因此，接口的一种常见用法就是前面提到的策略设计模式，此时你编写一个执行某些操作的方法，而该方法将接受一个同样是你指定的接口。你主要就是要声明：“你可以用任何你想要的对象来调用我的方法，只要你的对象遵循我的接口”。

## 接口中的域

1. 放入接口中的任何域都自动是static和final的，所以接口就成为了一种很便捷的用来创建常量组的工具。
2. 既然域是static的，它们就可以在类第一次被加载是初始化，这发生在任何域首次被访问时。
3. **这些域不是接口的一部分**，它们的值被存储在该接口的静态存储区域内。

## 接口和工厂

**生成遵循某个接口的对象的典型方式就是工厂方法设计模式**。这与直接调用构造器不同，我们在工厂对象上调用的是创建方法，而该工厂对象将生成接口的某个实现的对象。理论上，通过这种方式，我们的代码将完全与接口的实现分离，这就使得我们可以透明地将某个实现替换为另一个实现。

```java
interface Service{
    void method1();
    void method2();
}

interface ServiceFactory{
    Service getService();
}

class Implementation1 implements Service{
    Implementation1(){ }

    @Override
    public void method1() {
        Print.print("Implementation1 method1");
    }

    @Override
    public void method2() {
        Print.print("Implementation1 method2");
    }
}

class Implementation1Factory implements ServiceFactory{
    @Override
    public Service getService() {
        return new Implementation1();
    }
}

class Implementation2 implements Service{
    Implementation2(){ }

    @Override
    public void method1() {
        Print.print("Implementation2 method1");
    }

    @Override
    public void method2() {
        Print.print("Implementation2 method2");
    }
}

class Implementation2Factory implements ServiceFactory{
    @Override
    public Service getService() {
        return new Implementation2();
    }
}

public class Factories {
    public static void serviceConsumer(ServiceFactory factory){
        Service service = factory.getService();
        service.method1();
        service.method2();
    }

    public static void main(String[] args){
        serviceConsumer(new Implementation1Factory());
        serviceConsumer(new Implementation2Factory());
    }
}


/** output
 * Implementation1 method1
 * Implementation1 method2
 * Implementation2 method1
 * Implementation2 method2
 */
```



