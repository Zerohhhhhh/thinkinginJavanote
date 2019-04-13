# 第11章 持有对象

## 基本概念

java容器类类库的用途是“保存对象”，并将其划分为两个不同的概念：

1. `Collection`。
   - 一个独立元素的序列，这些元素都服从一条或多条规则。List必须按照插入的顺序保存元素，而`Set`不能有重复元素。`Queue`按照排队规则来确定对象产生的顺序。
   - Collection在每个槽中只能保存一个元素。此类容器包括：List，它以特定的顺序保存一组元素；Set，元素不能重复；Queue，只允许在容器的以“端”插入对象，并从另外一“端”移除对象。
2. `Map`。
   - 一组成对的“键值对”对象，允许你使用键来查找值。`ArrayList`允许你使用数字来查找值，因此在某种意义上讲，它将数字与对象关联在一起。映射表允许我们使用另一个对象来查找某个对象，它也被称为“关联数组”，或者被称为“字典”。
   - Map在每个槽内保存了两个对象，即键和与之相关联的值。

## List

`ArrayList`和`LinkedList`都是List类型。从输出可以看出，它们都按照被插入的顺序保存元素。

- `ArrayList`，擅长于随机访问元素，但是在List的中间插入和移除元素较慢。
- `LinkedList`，通过代价较低的List中间进行的插入和删除操作，提供了优化的顺序访问。`LinkedList`在随机访问方面相对比较慢。

## 迭代器

任何容器类，都必须有某种方式可以插人元素并将它们再次取回。毕竟，持有事物是容器最基本的工作。对于List，`add()`是插人元素的方法之一，而`get()`是取出元素的方法之一。

如果从更高层的角度思考，会发现这里有个缺点：要使用容器，必须对容器的确切类型编程。初看起来这没什么不好，但是考虑下面的情况：如果原本是对着List编码的，但是后来发现如果能够把相同的代码应用于set，将会显得非常方便，此时应该怎么做？或者打算从头开始编写通用的代码，它们只是使用容器，不知道或者说不关心容器的类型，那么如何才能不重写代码就可以应用于不同类型的容器？

迭代器（也是一种设计模式）的概念可以用于达成此目的。**迭代器是一个对象，它的工作是遍历并选择序列中的对象，而客户端程序员不必知道或关心该序列底层的结构**。此外，迭代器通常被称为轻量级对象：创建它的代价小。因此，经常可以见到对迭代器有些奇怪的限制；例如，Java的Iterator只能单向移动，这个Iterator只能用来：

- 使用方法`iterator()`要求容器返回一个`Iterator`。`Iterator`将准备好返回序列的第一个元素。
- 使用`next()`获得序列中的下一个元素。
- 使用`hasNext()`检查序列中是否还有元素。
- 使用`remove()`将迭代器新近返回的元素删除。

```java
public class SimpleIteration {
  public static void main(String[] args) {
    List<Pet> pets = Pets.arrayList(12);
    Iterator<Pet> it = pets.iterator();
    while(it.hasNext()) {
      Pet p = it.next();
      System.out.print(p.id() + ":" + p + " ");
    }
    System.out.println();
    // A simpler approach, when possible:
    for(Pet p : pets)
      System.out.print(p.id() + ":" + p + " ");
    System.out.println();	
    // An Iterator can also remove elements:
    it = pets.iterator();
    for(int i = 0; i < 6; i++) {
      it.next();
      it.remove();
    }
    System.out.println(pets);
  }
} /* Output:
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx 8:Cymric 9:Rat 10:EgyptianMau 11:Hamster
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx 8:Cymric 9:Rat 10:EgyptianMau 11:Hamster
[Pug, Manx, Cymric, Rat, EgyptianMau, Hamster]
*///:~
```

**接收对象容器并传递它，从而在每个对象上都执行操作，这种思想十分强大，并且贯穿于本书。**

```java
public class CrossContainerIteration {
  public static void display(Iterator<Pet> it) {
    while(it.hasNext()) {
      Pet p = it.next();
      System.out.print(p.id() + ":" + p + " ");
    }
    System.out.println();
  }	
  public static void main(String[] args) {
    ArrayList<Pet> pets = Pets.arrayList(8);
    LinkedList<Pet> petsLL = new LinkedList<Pet>(pets);
    HashSet<Pet> petsHS = new HashSet<Pet>(pets);
    TreeSet<Pet> petsTS = new TreeSet<Pet>(pets);
    display(pets.iterator());
    display(petsLL.iterator());
    display(petsHS.iterator());
    display(petsTS.iterator());
  }
} /* Output:
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx
4:Pug 6:Pug 3:Mutt 1:Manx 5:Cymric 7:Manx 2:Cymric 0:Rat
5:Cymric 2:Cymric 7:Manx 1:Manx 3:Mutt 6:Pug 4:Pug 0:Rat
*///:~
```

`display()`方法不包含任何有关它所遍历的序列的类型信息，而这也展示了Iterator的真正威力：**能够将遍历序列的操作与序列底层的结构分离**。迭代器统一了对容器的访问方式。

### ListIterator

`ListIterator`是一个更加强大的Iterator的子类型，它只能用于各种List类的访问。Iterator只能向前访问，但是`ListIterator`可以双向移动。它还可以产生相对于迭代器在列表中指向的当前位置的前一个和后一个元素的索引，并且可以使用`set()`方法替换它访问过的最后一个元素。你可以通过调用`listlterator()`方法产生一个指向List开始处的`Listlterator`，并且还可以通过调用`listlterator(n)`方法创建一个一开始就指向列表索引为n的元素处的`Listlterator`。下面的示例演示了所有这些能力：

```java
public class ListIteration {
  public static void main(String[] args) {
    List<Pet> pets = Pets.arrayList(8);
    ListIterator<Pet> it = pets.listIterator();
    while(it.hasNext())
      System.out.print(it.next() + ", " + it.nextIndex() +
        ", " + it.previousIndex() + "; ");
    System.out.println();
    // Backwards:
    while(it.hasPrevious())
      System.out.print(it.previous().id() + " ");
    System.out.println();
    System.out.println(pets);	
    it = pets.listIterator(3);
    while(it.hasNext()) {
      it.next();
      it.set(Pets.randomPet());
    }
    System.out.println(pets);
  }
} /* Output:
Rat, 1, 0; Manx, 2, 1; Cymric, 3, 2; Mutt, 4, 3; Pug, 5, 4; Cymric, 6, 5; Pug, 7, 6; Manx, 8, 7;
7 6 5 4 3 2 1 0
[Rat, Manx, Cymric, Mutt, Pug, Cymric, Pug, Manx]
[Rat, Manx, Cymric, Cymric, Rat, EgyptianMau, Hamster, EgyptianMau]
*///:~
```



## LinkedList

## Stack

## Set

`HashSet`、`TreeSet`和`LinkedHashSet`都是Set类型。`HashSet`的存储顺序无实际意义，但获取元素方式的速度最快。`TreeSet`按照比较结果的升序保存对象；`LinkedHashSet`按照被添加的顺序保存对象。

## Map

`HashMap`、`TreeMap`和`LinkedHashMap`。`HashMap`提供最快的查找技术，没有任何明显的顺序来保存元素。`TreeMap`按照比较结果的升序保存键，而`LinkedHashMap`则按照插入顺序保存键，同时还保留了`HashMap`的查询速度。

## Queue

## Collection和Iterator

## Foreach与迭代器