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

内部类如何做到自动拥有对其外围类所有成员的访问权？当某个外围类的对象创建了一个内部类对象时，此内部类对象必定会秘密捕获一个指向那个外围类对象的引用。然后，在访问此外围类的成员时，就是用那个引用来选择外围类的成员。构建内部类对象时，需要一个指向其外围类对象的引用，如果编译器访问不到这个引用就会报错。**内部类不能为static类。**

## 使用.this与.new