---
date: 2017-08-25T19:44:11+08:00
title: Programming-Assignment2-Deques-and-Randomized-Queues
tags: []
categories: ["技术"]
---



### Programming Assignment 2: Deques and Randomized Queues

---

[编程作业2 ：双端队列与随机队列](http://coursera.cs.princeton.edu/algs4/assignments/queues.html)

编写一个泛型数据结构用于双端队列（deque）和随机队列（randomized queue）。这次作业的目的在于通过数组和链表来实现基本的数据结构，同时向你介绍泛型（generics）和迭代器（iterators）。

**双端队列 **  **Dequeue**  *double-ended queue* 或者说 *deque*（即双端队列） （发言"deck"）是对可以支持从数据结构的首末两端添加和删除元素的堆栈和队列的概括。创建一个泛型双端队列实现以下API：

```
public class Deque<Item> implements Iterable<Item> {
   public Deque()                           // construct an empty deque
   public boolean isEmpty()                 // is the deque empty?
   public int size()                        // return the number of items on the deque
   public void addFirst(Item item)          // add the item to the front
   public void addLast(Item item)           // add the item to the end
   public Item removeFirst()                // remove and return the item from the front
   public Item removeLast()                 // remove and return the item from the end
   public Iterator<Item> iterator()         // return an iterator over items in order from front to end
   public static void main(String[] args)   // unit testing (optional)
}
```

*边缘情况* *Corner cases* 。 如果客户端尝试添加空元素抛出`java.lang.IllegalArgumentException`异常；如果客户端尝试从空双端队列删除元素抛出`java.util.NoSuchElementException`队列；如果客户端从迭代器中调用`remove()`方法抛出`java.lang.UnsupportedOperationException`异常；如果客户端在迭代器中调用`next()`方法但是没有元素可以返回的时候， 抛出`java.util.NoSuchElementException`异常。

*性能要求*  *Performance requirements*。 双端队列的实现必须支持每一个双端队列的操作（包括构造）在常数时间内完成。包含n个元素的双端队列至多可以使用的内存为：48n + 192，空间使用量正比与目前双端队列的元素的数量。此外，迭代器的实现需保证每一个操作（包括构造）常数时间内完成

**随机队列**  **Randomized queue**    随机队列类似与堆栈和队列，区别在于随机队列删除元素是在其数据结构的元素中随机性的删除。创建一个`RandomizedQueue`泛型数据结构并实现以下API：

```
public class RandomizedQueue<Item> implements Iterable<Item> {
   public RandomizedQueue()                 // construct an empty randomized queue
   public boolean isEmpty()                 // is the queue empty?
   public int size()                        // return the number of items on the queue
   public void enqueue(Item item)           // add the item
   public Item dequeue()                    // remove and return a random item
   public Item sample()                     // return (but do not remove) a random item
   public Iterator<Item> iterator()         // return an independent iterator over items in random order
   public static void main(String[] args)   // unit testing (optional)
}
```

*边缘情况*  *Corner cases*      对于同一个随机队列的两个或两个以上的迭代器的顺序必须彼此独立。每一个迭代器迭代器必须维持自己的随机顺序。如果客户端尝试添加空元素抛出`java.lang.IllegalArgumentException`异常，如果尝试从一个空的随机队列中取出一个元素则抛出`java.util.NoSuchElementException`异常。如果尝试在迭代器中调用`remove()`方法则抛出`java.lang.UnsupportedOperationException`异常。如果客户端在迭代器中调用`next()`方法但是没有元素可以返回的时候， 抛出`java.util.NoSuchElementException`异常。

*性能要求* *Performance requirements*    实现的随机队列需要保证随机队列的操作（不包括创建迭代器）常数时间内完成。也就是说，任意顺序执行的m个队列操作（开始是一个空队列）在最坏情况下至多需要 cm 个步骤，c为常数。包含n个元素的随机队列至多使用 48n + 192字节的内存。此外，迭代器的实现需保证 `next()`和`hasNext()`在最坏情况下运行时间为常数；构造在线性时间内完成，每一次迭代你可能（需要）使用线性比例的额外内存。

**排列客户端** **Permutation client**   编写`Permutation.java`客户端程序接收命令行参数k；通过`StdIn.readString()`读取标准输入字符串；随机的打印出它们中的 k个。至多能打印字符串序列中的字符一次。

  

```
% more distinct.txt                        % more duplicates.txt
A B C D E F G H I                          AA BB BB BB BB BB CC CC

% java Permutation 3 < distinct.txt       % java Permutation 8 < duplicates.txt
C                                          BB
G                                          AA
A                                          BB
                                           CC
% java Permutation 3 < distinct.txt        BB
E                                          BB
F                                          CC
G                                          BB
```

程序需要实现以下API：

```
public class Permutation {
   public static void main(String[] args)
}
```

可以假设 0 ≤ k ≤ n，n是标准输入字符串的字符数量。

*性能要求*   *Performance requirements*    程序`Permutation`的运行时间需要与输入大小成线性比。你也许只需要常数大小的内存加上一个`Deque`或`RandonizedQueue`对象大小（至多为n）的内存。（作为额外的挑战，做到一个`Deque`或`RandomizedQueue`对象最大内存使用仅为 k。）

**提交的作业**  **Deliverables**    只需提交`Deque.java`，`RandomizedQueue.java`，和`Permutation.java`。我们提供了`algs4.jar`。允许调用的库函数如下：[StdIn](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/StdIn.html) ，[StdOut](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/StdOut.html) ，[StdRandom](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/StdRandom.html) , `java.land`，`java.util.Iterator`和`java.util.NoSuchElementException`，不得调用其他的库函数。特别需指明的是，不用使用`java.util.LinkedList`与`java.util.ArrayList`。

