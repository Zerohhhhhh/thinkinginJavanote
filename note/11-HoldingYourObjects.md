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



## LinkedList

## Stack

## Set

`HashSet`、`TreeSet`和`LinkedHashSet`都是Set类型。`HashSet`的存储顺序无实际意义，但获取元素方式的速度最快。`TreeSet`按照比较结果的升序保存对象；`LinkedHashSet`按照被添加的顺序保存对象。

## Map

`HashMap`、`TreeMap`和`LinkedHashMap`。`HashMap`提供最快的查找技术，没有任何明显的顺序来保存元素。`TreeMap`按照比较结果的升序保存键，而`LinkedHashMap`则按照插入顺序保存键，同时还保留了`HashMap`的查询速度。

## Queue

## Collection和Iterator

## Foreach与迭代器