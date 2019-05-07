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

