# 第13章 字符串

## 不可变String

String对象是不可变的。String类中每一个看起来会修改String值的方法，实际上都是创建了一个全新的String对象，以包含修改后的字符串内容。而最初的String对象则丝毫未动。

```java
public class Immutable {
  public static String upcase(String s) {
    return s.toUpperCase();
  }
  public static void main(String[] args) {
    String q = "howdy";
    print(q); // howdy
    String qq = upcase(q);
    print(qq); // HOWDY
    print(q); // howdy
  }
}
/* Output:
howdy
HOWDY
howdy
*///:~
```

当把q传给`upcase()`方法时，实际传递的是引用的一个拷贝。其实，每当把String对象作为方法的参数时，都会复制一份引用，而该引用所指的对象其实一直待在单一的物理位置上，从未动过。

回到`upcase()`的定义，传入其中的引用有了名字s，只有`upcase()`运行的时候，局部引用s才存在。一旦`upcase()`运行结束，s就消失了。当然了，`upcase()`的返回值，其实只是最终结果的引用。这足以说明，`upcase()`返回的引用已经指向了一个新的对象，而原本的q则还在原地。

---

## StringBuilder

字符串拼接采用`StringBuilder`的效率要好于“+”和`StringBuffer`。

`StringBuilder`提供了丰富而全面的方法，包括delete()方法、replace()方法、substring()甚至reverse()，但是最常用的还是append()和toString()，还有delete()。

`StringBuilder`是java SE5引入的，在这之前java用的是`StringBuffer`。后者是线程安全的，因此开销也会大些。

---

## 无意识的递归

```java
public class InfiniteRecursion {
  public String toString() {
    return " InfiniteRecursion address: " + this + "\n";
  }
}
```

此时调用toString时，由于编译器看到一个String对象`”InfiniteRecursion address: “`后面跟着一个“+”，而再后面的对象不是String，于是编译器试着将this转换成一个String。通过调用this上的toString()方法，于是就发生了递归调用。

