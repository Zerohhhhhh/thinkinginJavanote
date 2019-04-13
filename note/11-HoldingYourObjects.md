# 第11章 持有对象

## 基本概念

java容器类类库的用途是“保存对象”，并将其划分为两个不同的概念：

1. `Collection`。
   - 一个独立元素的序列，这些元素都服从一条或多条规则。List必须按照插入的顺序保存元素，而`Set`不能有重复元素。`Queue`按照排队规则来确定对象产生的顺序。
   - Collection在每个槽中只能保存一个元素。此类容器包括：List，它以特定的顺序保存一组元素；Set，元素不能重复；Queue，只允许在容器的以“端”插入对象，并从另外一“端”移除对象。
2. `Map`。
   - 一组成对的“键值对”对象，允许你使用键来查找值。`ArrayList`允许你使用数字来查找值，因此在某种意义上讲，它将数字与对象关联在一起。映射表允许我们使用另一个对象来查找某个对象，它也被称为“关联数组”，或者被称为“字典”。
   - Map在每个槽内保存了两个对象，即键和与之相关联的值。

---

## List

`ArrayList`和`LinkedList`都是List类型。从输出可以看出，它们都按照被插入的顺序保存元素。

- `ArrayList`，擅长于随机访问元素，但是在List的中间插入和移除元素较慢。
- `LinkedList`，通过代价较低的List中间进行的插入和删除操作，提供了优化的顺序访问。`LinkedList`在随机访问方面相对比较慢。

---

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

---

### `ListIterator`

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

---

## `LinkedList`

`LinkedList`也像`ArrayList`一样实现了基本的List接口，但是它执行某些操作（在List的中间插入和移除）时比`ArrayList`更高效，但在随机访问操作方面却要逊色一些。

`LinkedList`还添加了可以使其用作栈、队列和双向队列的方法。

---

## Stack

“栈”通常是指“后进先出”（LIFO）的容器。最后“压入”栈的元素，第一个“弹出”栈。

```java
public class StackTest {
  public static void main(String[] args) {
    Stack<String> stack = new Stack<String>();
    for(String s : "My dog has fleas".split(" "))
      stack.push(s);
    while(!stack.empty())
      System.out.print(stack.pop() + " ");
  }
} /* Output:
fleas has dog My
*///:~
```



---

## Set

`HashSet`、`TreeSet`和`LinkedHashSet`都是Set类型。`HashSet`的存储顺序无实际意义，但获取元素方式的速度最快。`TreeSet`按照比较结果的升序保存对象；`LinkedHashSet`按照被添加的顺序保存对象。

Set中最常被使用的是测试归属性，可以很容易地询问某个对象是否在某个Set中的。

```java
public class SetOfInteger {
    public static void main(String[] args) {
        Random random = new Random(47);
        Set<Integer> integerSet = new HashSet<>();
        for (int i = 0;i < 10000;i++){
            integerSet.add(random.nextInt(30));
        }
        System.out.print(integerSet);
    }
}
```

`HashSet`所维护的顺序与`TreeSet`或`LinkedHashSet`都不同，因为它们的实现具有不同的元素存储方式。`TreeSet`将元素存储在红-黑树数据结构中，而`HashSet`使用的是散列函数。`LinkedHashList`因为查询速度的原因也使用了散列，但是看起来它使用了链表来维护元素的插入顺序。

关于Set容器最常见的操作之一，就是使用`contains()`测试Set的归属性。

```java
public class SetOperations {
  public static void main(String[] args) {
    Set<String> set1 = new HashSet<String>();
    Collections.addAll(set1,
      "A B C D E F G H I J K L".split(" "));
    set1.add("M");
    print("H: " + set1.contains("H"));
    print("N: " + set1.contains("N"));
    Set<String> set2 = new HashSet<String>();
    Collections.addAll(set2, "H I J K L".split(" "));
    print("set2 in set1: " + set1.containsAll(set2));
    set1.remove("H");
    print("set1: " + set1);
    print("set2 in set1: " + set1.containsAll(set2));
    set1.removeAll(set2);
    print("set2 removed from set1: " + set1);
    Collections.addAll(set1, "X Y Z".split(" "));
    print("'X Y Z' added to set1: " + set1);
  }
} /* Output:
H: true
N: false
set2 in set1: true
set1: [D, K, C, B, L, G, I, M, A, F, J, E]
set2 in set1: false
set2 removed from set1: [D, C, B, G, M, A, F, E]
'X Y Z' added to set1: [Z, D, C, B, G, M, A, F, Y, X, E]
*///:~
```

当`TreeSet`对字母进行排序时，默认排序是按`字典序`进行的，因此大写或小写字母被划分到了不同组中。如果想要按照`字母序`排序，那么可以向`TreeSet`的构造器传入`String.CASE_INSENTIVE_ORDER`比较器（比较器就是建立排序顺序的对象）。

---

## Map

`HashMap`、`TreeMap`和`LinkedHashMap`。`HashMap`提供最快的查找技术，没有任何明显的顺序来保存元素。`TreeMap`按照比较结果的升序保存键，而`LinkedHashMap`则按照插入顺序保存键，同时还保留了`HashMap`的查询速度。

将对象映射到其他对象的能力是一种解决编程问题的杀手锏。例如，考虑一个程序，它将用来检查java的Random类的随机性。

```java
public class Statistics {
  public static void main(String[] args) {
    Random rand = new Random(47);
    Map<Integer,Integer> m =
      new HashMap<Integer,Integer>();
    for(int i = 0; i < 10000; i++) {
      // Produce a number between 0 and 20:
      int r = rand.nextInt(20);
      Integer freq = m.get(r);
      m.put(r, freq == null ? 1 : freq + 1);
    }
    System.out.println(m);
  }
} /* Output:
{15=497, 4=481, 19=464, 8=468, 11=531, 16=533, 18=478, 3=508, 7=471, 12=521, 17=509, 2=489, 13=506, 9=549, 6=519, 1=502, 14=477, 10=513, 5=503, 0=481}
*///:~
```

---

## Queue

队列是一个典型的先进先出（FIFO）容器。即从容器的一端放入事物，从另一端取出，并且事物放入容器的顺序与取出的顺序是相同的。队列常被当做一张可靠的将对象从程序的某个区域传输到另一个区域的途径。队列在并发编程中特别重要。

`offer()`方法是与Queue相关的方法之一，它在允许的情况下，将一个元素插人到队尾，或者返回false。`peek()`和`element()`都将在不移除的情况下返回队头，但是peek()方法在队列为空时返回null，而`element()`会抛出`NoSuchElementException`异常。`poll()`和`remove()`方法将移除并返回队头，但是`poll()`在队列为空时返回null，而`remove()`会抛出`NoSuchElementException`异常。

Queue接口窄化了对`LinkedList`的方法的访问权限，以使得只有恰当的方法才可以使用。因此，你能够访问的`LinkedList`的方法会变少（这里你实际上可以将queue转型回，但是至少我们不鼓励这么做）。

注意，与Queue相关的方法提供了完整而独立的功能。即，对于Queue所继承的Collection,在不需要使用它的任何方法的情况下，就可以拥有一个可用的Queue。

---

### `PriorityQueue`

先进先出声明的是下一个元素应该是等待时间最长的元素。

优先级队列声明下一个弹出元素是最需要的元素。

```java
public class PriorityQueueDemo {
  public static void main(String[] args) {
    PriorityQueue<Integer> priorityQueue = new PriorityQueue<Integer>();
    Random rand = new Random(47);
    for(int i = 0; i < 10; i++)
      priorityQueue.offer(rand.nextInt(i + 10));
    QueueDemo.printQ(priorityQueue);

    List<Integer> ints = Arrays.asList(25, 22, 20,
      18, 14, 9, 3, 1, 1, 2, 3, 9, 14, 18, 21, 23, 25);
    priorityQueue = new PriorityQueue<Integer>(ints);
    QueueDemo.printQ(priorityQueue);
    priorityQueue = new PriorityQueue<Integer>(
        ints.size(), Collections.reverseOrder());
    priorityQueue.addAll(ints);
    QueueDemo.printQ(priorityQueue);

    String fact = "EDUCATION SHOULD ESCHEW OBFUSCATION";
    List<String> strings = Arrays.asList(fact.split(""));
    PriorityQueue<String> stringPQ = new PriorityQueue<String>(strings);
    QueueDemo.printQ(stringPQ);
    stringPQ = new PriorityQueue<String>(strings.size(), Collections.reverseOrder());
    stringPQ.addAll(strings);
    QueueDemo.printQ(stringPQ);

    Set<Character> charSet = new HashSet<Character>();
    for(char c : fact.toCharArray())
      charSet.add(c); // Autoboxing
    PriorityQueue<Character> characterPQ = new PriorityQueue<Character>(charSet);
    QueueDemo.printQ(characterPQ);
  }
} /* Output:
0 1 1 1 1 1 3 5 8 14
1 1 2 3 3 9 9 14 14 18 18 20 21 22 23 25 25
25 25 23 22 21 20 18 18 14 14 9 9 3 3 2 1 1
       A A B C C C D D E E E F H H I I L N N O O O O S S S T T U U U W
W U U U T T S S S O O O O N N L I I H H F E E E D D C C C B A A
  A B C D E F H I L N O S T U W
*///:~
```



---

## Collection和Iterator

Collection是描述所有序列容器的共性的根接口，它可能会被认为是一个“附属接口”，即因为要表示其他若干个接口的共性而出现的接口。

使用接口描述的一个理由是它可以使我们能够创建更通用的代码。通过针对接口而非具体实现来编写代码，我们的代码可以应用于更多的对象类型。因此，如果我编写的方法将接受一个Collection，那么该方法就可以应用于任何实现了Collection的类——这也就使得一个新类可以选择去实现Collection接口，以便我的方法可以使用它。

在java中，用迭代器而不是Collection来表示容器之间的共性。但是，这两种方法绑定到了一起，因为实现Collection就意味着需要提供iterator()方法。

```java
public class InterfaceVsIterator {
  public static void display(Iterator<Pet> it) {
    while(it.hasNext()) {
      Pet p = it.next();
      System.out.print(p.id() + ":" + p + " ");
    }
    System.out.println();
  }
  public static void display(Collection<Pet> pets) {
    for(Pet p : pets)
      System.out.print(p.id() + ":" + p + " ");
    System.out.println();
  }	
  public static void main(String[] args) {
    List<Pet> petList = Pets.arrayList(8);
    Set<Pet> petSet = new HashSet<Pet>(petList);
    Map<String,Pet> petMap =
      new LinkedHashMap<String,Pet>();
    String[] names = ("Ralph, Eric, Robin, Lacey, " +
      "Britney, Sam, Spot, Fluffy").split(", ");
    for(int i = 0; i < names.length; i++)
      petMap.put(names[i], petList.get(i));
    display(petList);
    display(petSet);
    display(petList.iterator());
    display(petSet.iterator());
    System.out.println(petMap);
    System.out.println(petMap.keySet());
    display(petMap.values());
    display(petMap.values().iterator());
  }	
} /* Output:
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx
4:Pug 6:Pug 3:Mutt 1:Manx 5:Cymric 7:Manx 2:Cymric 0:Rat
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx
4:Pug 6:Pug 3:Mutt 1:Manx 5:Cymric 7:Manx 2:Cymric 0:Rat
{Ralph=Rat, Eric=Manx, Robin=Cymric, Lacey=Mutt, Britney=Pug, Sam=Cymric, Spot=Pug, Fluffy=Manx}
[Ralph, Eric, Robin, Lacey, Britney, Sam, Spot, Fluffy]
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx
*///:~
```

两个版本的`display()`方法都可以使用Map或Collection的子类型来工作，而且Collection接口和Iterator都可以将`display()`方法与底层容器的特定实现解耦。

在本例中，这两种方式都可以凑效。事实上，Collection要更方便一点，因为它是Iterator类型，因此，在`display(Collection)`实现中，可以使用`foreach`结构，从而使代码更加清晰。

当你要实现一个不是Collection的外部类时，由于让它去实现Collection接口可能非常困难或麻烦，因此使用Iterator就会变得非常吸引人。例如，如果我们通过继承一个持有Pet对象的类来创建一个Collection的实现， 那么我们必须实现所有的Collection方法， 即使我们在`display()`方法中不必使用它们 ， 也必须如此。 尽管这可以通过继承`AbstractCollection`而很容易地实现 ， 但是你无论如何还是要被强制去实现`iterator()`和`size()`， 以便提供`AbstractCollection`没有实现 ， 但是`AbstractCollection`中的其他方法会使用到的方法：

```java
public class CollectionSequence
extends AbstractCollection<Pet> {
  private Pet[] pets = Pets.createArray(8);
  public int size() { return pets.length; }
  public Iterator<Pet> iterator() {
    return new Iterator<Pet>() {
      private int index = 0;
      public boolean hasNext() {
        return index < pets.length;
      }
      public Pet next() { return pets[index++]; }
      public void remove() { // Not implemented
        throw new UnsupportedOperationException();
      }
    };
  }	
  public static void main(String[] args) {
    CollectionSequence c = new CollectionSequence();
    InterfaceVsIterator.display(c);
    InterfaceVsIterator.display(c.iterator());
  }
} /* Output:
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx
*///:~

```

从本例中，你可以看到，如果你实现Collection，就必须实现`iterator()`，并且只能拿实现`iterator()`与继承`AbstractCollection`相比，花费的代价只有略微减少。但是，如果你的类已经继承了其他的类，那么你就不能再继承`AbstractCollection`了。在这种情况下，要实现Collection，就必须实现该接口中的所有方法。此时，继承并提供创建迭代器的能力就会显得容易得多了：

```java
class PetSequence {
  protected Pet[] pets = Pets.createArray(8);
}

public class NonCollectionSequence extends PetSequence {
  public Iterator<Pet> iterator() {
    return new Iterator<Pet>() {
      private int index = 0;
      public boolean hasNext() {
        return index < pets.length;
      }
      public Pet next() { return pets[index++]; }
      public void remove() { // Not implemented
        throw new UnsupportedOperationException();
      }
    };
  }
  public static void main(String[] args) {
    NonCollectionSequence nc = new NonCollectionSequence();
    InterfaceVsIterator.display(nc.iterator());
  }
} /* Output:
0:Rat 1:Manx 2:Cymric 3:Mutt 4:Pug 5:Cymric 6:Pug 7:Manx
*///:~

```

生成Iterator是将队列与消费队列的方法连接在一起耦合度最小的方式，并且与实现Collection相比，它在序列类上所施加的约束也少得多。

---

## `Foreach`与迭代器

到目前为止，`foreach`语法主要用于数组，但是它也可以应用于任何Collection对象。能够与`foreach`一起工作时所有Collection对象的特征。

之所以能够工作，是因为被称为`Iterable`的接口，该接口包含一个能够产生Iterator的iterator()方法，并且`Iterable`接口被`foreach`用来在序列中移动。因此任何实现`Iterable`的类，都可以将它用于`foreach`语句中：

```java
public class IterableClass implements Iterable<String> {
  protected String[] words = ("And that is how " +
    "we know the Earth to be banana-shaped.").split(" ");
  public Iterator<String> iterator() {
    return new Iterator<String>() {
      private int index = 0;
      public boolean hasNext() {
        return index < words.length;
      }
      public String next() { return words[index++]; }
      public void remove() { // Not implemented
        throw new UnsupportedOperationException();
      }
    };
  }	
  public static void main(String[] args) {
    for(String s : new IterableClass())
      System.out.print(s + " ");
  }
} /* Output:
And that is how we know the Earth to be banana-shaped.
*///:~

```

在java SE5，大量的类都是`Iterable`类型，主要包括所有的Collection类（但是不包括各种Map）。

`foreach`语句可以用于数组或其他任何`Iterable`，但是这并不意味着数组肯定也是一个`Iterable`，而任何自动包装也不会自动发生。尝试吧数组当作一个`Iterable`参数传递会导致失败。这说明不存在任何从数组到`Iterable`的自动转换，必须手工执行这种转换。

---

### 适配器方法惯用法

