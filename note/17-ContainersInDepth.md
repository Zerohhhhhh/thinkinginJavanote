# 第17章 容器深入研究

## Collection的部分功能方法

`boolean addAll(Conlection<? extends T>)` ：添加参数中的所有元素。只要添加了意元素就返回true（可选的）。

`Object[] toArray()`：返回一个数组，该数组包含容器中的所有元素。

`<T> T[]  toArray(T[] a)`：返回一个数组，该数组包含容器中的所有元素。返回结果的运行时类型与参数数组a的类型相同，而不是单纯的Object。

## 可选操作

“未获支持的操作" 这种方式可以实现Java容器类库的一个重要目标 ： 容器应该易学易用 。 未获支持的操作是一种特例，可以延迟到需要时再实现。 但是，为了让这种方式能够工作 ：

1. `UnsupportedOperationException`必须是一种罕见事件。 即，对于大多数类来说 ， 所有操作都应该可以工作 ， 只有在特例中才会有未获支持的操作。 这种设计留下了一个“后门”，如果你想创建新的Collection，但是没有为Collection接口中的所有方法都提供有意义的定义，那么它仍旧适合现有的类库。

2. 如果一个操作是未获支持的，那么在实现接口的时候可能就会导致`UnsupportedOperationException`异常，而不是将产品程序交给客户以后才出现此异常，这种情况是有道理的。毕竟，它表示编程上有错误：使用了不正确的接口实现。

### 未获支持的操作

最常见的未获支持的操作，都来源于背后由固定尺寸的数据结构支持的容器。当你用`Arrays.asList()`将数组转换为List时，就会得到这样的容器。

因为`Arrays.asList()`会生成一个List, 它基于一个固定大小的数组， 仅支持那些不会改变数组大小的操作。 任何会引起对底层数据结构的尺寸进行修改的方法都会产生一个`UnsupportedOperationException`异常，以表示对未获支持操作的调用（一个编程错误）。

注意 ， 应该把`Arrays.asList()`的结果作为构造器的参数传递给任何Collection（或者使用`addAll()`方法 ， 或`Collections.addAll(`)静态方法），这样可以生成允许使用所有的方法的普通容器。

## Set和存储顺序

- 加入Set的元素必须定义`equals()`方法以确保对象的唯一性。
- `hashCode()`只有这个类被置于`HashSet`或者`LinkedHashSet`中时才是必需的。但是对于良好的编程风格而言，你应该在覆盖equals()方法时，总是同时覆盖`hashCode(`)方法。
- 如果一个对象被用于任何种类的排序容器中，例如SortedSet（TreeSet是其唯一实现），那么它必须实现Comparable接口。
- 注意，`SortedSet`的意思是“按对象的比较函数对元素排序”，而不是指“元素插入的次序”。插入顺序用`LinkedHashSet`来保存。

## 队列

- 队了并发应用，Queue在Java SE5中仅有的两个实现是`LinkiedList`和`PriorityQueue`，它们仅有**排序行为**的差异，性能上没有差异。
- 优先级队列`PriorityQueue`的排列顺序也是通过实现`Comparable`而进行控制的。

## Map(映射表)

**映射表**（也称为**关联数组**Associative Array）。

### 性能

HashMap使用了特殊的值，称作散列码（hash code），来取代对键的缓慢搜索。散列码是“相对唯一”的、用以代表对象的int值，它是通过将该对象的某些信息进行转换而生成的。 
hashCode()是根类Object中的方法，因此所有对象都能产生散列码。

对Map中使用的键的要求与对Set中的元素的要求一样：

- 任何键都必须具有一个equals()方法；
- 如果键被用于散列Map，那么它必须还具有恰当的hashCode()方法；
- 如果键被用于`TreeMap`，那么它必须实现Comparable。

1. `HashMap`： Map基于散列表的实现（它取代了`Hashtable`)。插人和查询“键值对"的开销是固定的。可以通过构造器设置容量和负载因子，以调整容器的性能。
2. `LinkedHashMap`：类似于`HashMap`，但是迭代遍历它时，取得“键值对”的顺序是其插人次序，或者是最近最少使用(LRU)的次序。只比`HashMap`慢—点，而在迭代访问时反而更快，因为它使用链表维护内部次序。
3. `TreeMap`：基于红黑树的实现。查看“键"或“键值对'时，它们会被排序（次序由Comparable或Comparator决定)。`TreeMap`的特点在于，所得到的结果是经过排序的。`TreeMap`是唯一的带有`subMap()`的方法的Map，它可以返回一个子树。
4. `WeakHashMap`：弱键(weak key)映射，允许释放映射所指向的对象，这是为解决某类特殊问题而设计的。如果映射之外没有引用指向某个“键”，则此“键"可以被垃圾收集器回收。
5. `ConcurrentHashMap`：一种线程安全的Map，它不涉及同步加锁。我们将在“并发”一章中讨论。
6. `IdentityHashMap`： 使用==代替`equals()`对“键”进行比较的散列映射。专为解决特殊问题而设计的。

## 散列与散列码

1. `HashMap`使用`equals()`判断当前的键是否与表中存在的键相同。
2. 默认的`Object.equals()`只是比较对象的地址。如果要使用自己的类作为`HashMap`的键，必须同时重写`hashCode()`和`equals()`。 
3. 正确的`equals()`方法必须满足下列5个条件：

- 自反性。对任意x，`x.equals(x)`—定返回true.
- 对称性。对任意x和y，如果`y.equals(x`)返回true，则`x.equals(y)`也返回true。
- 传递性。对任意x、y、z，如果有`x.equals(y)`返回true，`y.equals(z)`返回true，则`x.equals(z)`一定返回true.
- 一致性。对任意x和y，如果对象中用于等价比较的信息没有改变，那么无论调用`x.equals(y)`多少次，返回的结果应该保持一致，要么一直是true，要么一直是false。
- 对任何不是null的x，`x.equals(null)`一定返回false。

### 散列概念

使用散列的目的在于：**想要使用一个对象来查找另一个对象**。 Map的实现类使用散列是为了**提高查询速度**。

```java
	散列的价值在于速度：散列使得查询得以快速进行。由于瓶颈位于查询速度，因此解决方案之一就是保持键的排序状态，然后使用Collections.binarySearch()进行查询。
	散列则更进一步，它将键保存在某处，以便能够很快找到。存储一组元素最快的数据结构是数组，所以用它来表示键的信息（请小心留意，我是说键的信息，而不是键本身）。但是因为数组不能调整容量，因此就有一个问题：我们希望在Map中保存数量不确定的值，但是如果键的数量被数组的容量限制了，该怎么办？
	答案就是：数组并不保存键本身。而是通过键对象生成一个数字，将其作为数组的下标。这个数字就是散列码，由定义在Object中的、且可能由你的类覆盖的hashCode()方法（在计算机科学的术语中称为散列函数）生成。
	为解决数组容量固定的问题，不同的键可以产生相同的下标。也就是说，可能会有冲突，即散列码不必是独一无二的。因此，数组多大就不重要了，任何键总能在数组中找到它的位置。
```

### HashMap查询过程（快速原因）

因此，`HashMap`中查询一个key的过程就是：

1. 首先计算散列码。
2. 然后使用散列码查询数组（散列码作变数组下标）。
3. 如果没有冲突，即生成这个散列码的对象只有一个，则散列码对应的数组下标的位置就是这个要查找的元素。
4. 如果有冲突，则散列码对应的下标所在数组元素保存的是一个list，然后对list中的值使用equals()方法进行线性查询。（jdk1.8之后，不是使用单纯的链表，而是当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。

因此，不是查询整个list，而是快速地跳到数组的某个位置，只对很少的元素进行比较。这便是`HashMap`会如此快速的原因。

###  简单散列Map的实现

1. 散列表中的槽位（slot）通常称为桶位（bucket），因此将表示实际散列表的数组命名我bucket。
2. 为使散列均匀，桶的数量通常使用质数（JDK5中是质数，JDK7中已经是2的整数次方了）。 

```java
	事实证明，质数实际上并不是散列桶的理想容量。近来，（通过广泛的测试）Java的散列函数都使用2的整数次方。对现代处理器来说，除法与求余数是最慢的操作。使用2的整数次方长度的散列表，可用掩码代替除法。因为get()是使用最多的操作，求余数的%操作是其开销最大的部分，而使用2的整数次方可以消除此开销（也可能对hashCode()有些影响）。
```

3. get()方法按照与put()方法相同的方式计算在buckets数组中的索引，这很重要，因为这样可以保证两个方法可以计算出相同的位置。

### 覆盖`hashCode()`

设计`hashCode()`时要考虑的因素：

- 最重要的因素：无论何时，对同一相对象调用`hashCode()`都应该生成同样的值。
- 此外，不应该使`hashCode()`依赖于具有唯一性的对象信息，尤其是使用this的值，这只能产生很糟糕的`hashCode()`。因为这样做无法生成一个新的键，使之与`put()`中原始的键值对中的键相同。即应该使用对象内有意义的识别信息。也就是说，它必须基于对象的内容生成散列码。
- 但是，通过`hashCode() equals()`必须能够完全确定对象的身份。
- 因为在生成桶的下标前，`hashCode()`还需要进一步处理，所以散列码的生成范围并不重要，只要是`int`即可。
- 好的`hashCode()`应该产生分布均匀的散列码。

### `HashMap`的性能因子

`HashMap`中的一些术语：

- 容量（Capacity）：表中的桶位数（The number of buckets in the table）。
- 初始容量（Initial capacity）：表在创建时所拥有的桶位数。HashMap和HashSet都具有允许你指定初始容量的构造器。
- 尺寸（Size）：表中当前存储的项数。
- 负载因子（`Loadfactor`）：尺寸/容量。空表的负载因子是0，而半满表的负载因子是0.5，依此类推。负载轻的表产生冲突的可能性小，因此对于插入和查找都是最理想的（但是会减慢使用迭代器进行遍历的过程）。`HashMap`和`HashSet`都具有允许你指定负载因子的构造器，表示当负载情况达到该负载的水平时，容器将自动增加其容量（桶位数），实现方式是使容量大致加倍，并重新将现有对象分布到新的桶位集中（这被称为再散列）。

```java
	HashMap使用的默认负载因子是0.75（只有当表达到四分之三满时，才进行再散列），这个因子在时间和空间代之间达到了平衡。更高的负载因子可以降低表所需的空间，但会增加查找代价，这很重要，因为查找是我们在大多数时间里所做的操作（包括get()和put()）。
```

如果你知道将要在`HashMap`中存储多少项，那么创建一个具有恰当大小的初始容量将可以避免自动再散列的开销。

## 设定Collection或Map为不可修改

创建一个只读的Collection或Map，有时可以带来某些方便。Collection类可以帮助达成此目的，它有一个方法，参数为原本的容器，返回值是容器的只读版本。

```java
public class ReadOnly {
  static Collection<String> data =
    new ArrayList<String>(Countries.names(6));
  public static void main(String[] args) {
    Collection<String> c =
      Collections.unmodifiableCollection(
        new ArrayList<String>(data));
    print(c); // Reading is OK
    //! c.add("one"); // Can't change it

    List<String> a = Collections.unmodifiableList(
        new ArrayList<String>(data));
    ListIterator<String> lit = a.listIterator();
    print(lit.next()); // Reading is OK
    //! lit.add("one"); // Can't change it

    Set<String> s = Collections.unmodifiableSet(
      new HashSet<String>(data));
    print(s); // Reading is OK
    //! s.add("one"); // Can't change it

    // For a SortedSet:
    Set<String> ss = Collections.unmodifiableSortedSet(
      new TreeSet<String>(data));

    Map<String,String> m = Collections.unmodifiableMap(
      new HashMap<String,String>(Countries.capitals(6)));
    print(m); // Reading is OK
    //! m.put("Ralph", "Howdy!");

    // For a SortedMap:
    Map<String,String> sm =
      Collections.unmodifiableSortedMap(
        new TreeMap<String,String>(Countries.capitals(6)));
  }
} /* Output:
[ALGERIA, ANGOLA, BENIN, BOTSWANA, BULGARIA, BURKINA FASO]
ALGERIA
[BULGARIA, BURKINA FASO, BOTSWANA, BENIN, ANGOLA, ALGERIA]
{BULGARIA=Sofia, BURKINA FASO=Ouagadougou, BOTSWANA=Gaberone, BENIN=Porto-Novo, ANGOLA=Luanda, ALGERIA=Algiers}
*///:~

```

## Collection或Map的同步控制

Collections类有办法能够自动同步整个容器。其语法与“不可修改的”方法相似：

```java
package net.mrliuli.containers;

import java.util.*;

public class Synchronization {
    public static void main(String[] args){
        Collection<String> c = Collections.synchronizedCollection(new ArrayList<String>());
        List<String> list = Collections.synchronizedList(new ArrayList<String>());
        Set<String> s = Collections.synchronizedSet(new HashSet<String>());
        Set<String> ss = Collections.synchronizedSortedSet(new TreeSet<String>());
        Map<String, String> m = Collections.synchronizedMap(new HashMap<String, String>());
        Map<String, String> sm = Collections.synchronizedSortedMap(new TreeMap<String, String>());
    }
}
```

