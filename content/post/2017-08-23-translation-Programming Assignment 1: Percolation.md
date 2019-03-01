---
date: 2017-08-23T19:44:11+08:00
title: Programming-Assignment-1-Percolation
tags: []
categories: ["技术"]
---

[编程作业１：渗透法（联通问题）](http://coursera.cs.princeton.edu/algs4/assignments/percolation.html)

通过蒙特卡罗仿真（Monte Carlo simulation）编写一程序用于估计渗透阈值（*percolation threshold*）

**安装 Java编程环境**。通过安装指南在你电脑上按步骤安装好 Java编程环境（[Mac Os X](http://algs4.cs.princeton.edu/mac/) - [Windows](http://algs4.cs.princeton.edu/windows/) - [Linux](http://algs4.cs.princeton.edu/linux/)）。在遵循这些指导步骤之后，javac-algs4和 java-algs4命令将 algs4.jar库作为类路径，而 algs4.jar包含用于 I/O的 Java类和所有书中的算法。

为了获取 algs4.jar库中的类，你需要使用`import`语句，示例如下：

```java
import edu.princeton.cs.algs4.StdRandom;
import edu.princeton.cs.algs4.StdStats;
import edu.princeton.cs.algs4.WeightedQuickUnionUF;
```

需要注意的是你的代码必须在默认的包（*default package*）里面，如果你使用的是`package`语句，自动评分器将不能评估你的作业。

**渗透**（**Percolation**），假设存在一随机分布着绝缘金属材料的复合系统，为了使得该复合系统成为电导体，其哪一部分材料需要是金属（即非绝缘）。假设存在一上表面的是水（或者底部是油）的多孔模型，在什么情况下上方的水能够流入底部（或油从底部上浮到上表面）？科学家们通过定义名为联通（percolation）的抽象过程来模拟这些情形。

**模型**（**The model**）我们通过n*n的网格区域来模拟一个渗透系统。每个网格区域不是打开就是闭合状态。如果一网格块通过一系列相邻的状态为打开的网格块连接到顶部一个打开的网格块，那么该网格块则为饱和区域（**A full site**）。如果在系统最底部一行存在一个填满区域的网格块，那么我们就说该系统是可联通的。换言之，如果我们填满所有与顶部一行相连的网格块然后渗透过程中填满了部分底部一行中打开的网格块，一个系统就是可联通的（可联通）。（对于绝缘金属材料例子而言，打开的网格块对应着金属材料，所以一个可渗透的系统在填满网格块（full sites）的的传导下，拥有一条从顶部到底部的金属路径。对于多孔物体例子而言，打开的网格块对应着水流可流经的空的区域，所以当水流填满空区域时，若水流从顶部流过底部，那该系统即是可联通的。）

![](http://coursera.cs.princeton.edu/algs4/assignments/percolates-yes.png)

![](http://coursera.cs.princeton.edu/algs4/assignments/percolates-no.png)

**问题**（**The problem**）在一个著名的科学问题中，研究人员对以下问题非常感兴趣，如果网格块各自独立的以概率p随机设定为打开状态（反之闭合的概率则为１－p），那么该系统可联通的概率是多少？当 p等于０，系统一段不可联通；当 p等于１，系统必然可联通。下面的图分别展示了对于20\*20随机网格（左）和100\*100随机网格（右），网格块空闲概率p与可联通概率之间的关系。

![](http://coursera.cs.princeton.edu/algs4/assignments/percolation-threshold20.png)

![](http://coursera.cs.princeton.edu/algs4/assignments/percolation-threshold100.png)

当n的值足够大的时候，存在一个阈值p\*，当 p < p\*的时候随机的n\*n网格块几乎毫无可能联通，而当 p > p\*的时候，随机的n*n网格块几乎一直可联通。目前为止对于确定联通阈值p\*没有数学解法。你的任务就是编写程序用于估计阈值p\*的值。

**联通问题数据结构**（**Percolation data type**）为了模拟一个联通系统，创建拥有如下API名为`Percolation`的数据结构：

```java
public class Percolation {
   public Percolation(int n)                // create n-by-n grid, with all sites blocked
   public    void open(int row, int col)    // open site (row, col) if it is not open already
   public boolean isOpen(int row, int col)  // is site (row, col) open?
   public boolean isFull(int row, int col)  // is site (row, col) full?
   public     int numberOfOpenSites()       // number of open sites
   public boolean percolates()              // does the system percolate?

   public static void main(String[] args)   // test client (optional)
}
```

*边界问题*（*Corner cases*　）通常，行和列的索引介于１和n之间，（１，１）代表左上角的网格块：如果以下方法的参数越界则抛出`java.lang.IllegalArgumentException` 异常：`open()`, `isOpen()`, 或`isFull()`。如果*n* ≤ 0，构造函数则抛出``java.lang.IllegalArgumentException` `异常。

*性能要求*（*Performance requirements*）构造函数时间复杂读O( *n*^2)；所有方法（mehtods）用时为：常数时间加上常数次调用 `union-find` 方法： `union()`, `find()`, `connected()`, 和 `count()`。　　

**蒙特卡罗仿真**　**Monte Carlo simulation**　为了估计联通阈值，考虑以下计算实验

- 初始化所有网格块为闭合状态　　
- 重复下列步骤直至系统联通: 
  - 在所有闭合的网格块中均匀随机选择一个网格块
  - 打开该网格块
- 当系统可联通的时候，打开的网格块数量提供了一个联通阈值的估计值

举个例子，如果按照以下快照那样在一个20\*20栅格中打开网格块，那我们的估计的联通阈值为：204/400 = 0.51，因为系统在第204块网格块打开的时候，变为可联通状态。

![](http://coursera.cs.princeton.edu/algs4/assignments/percolation-50.png)

![](http://coursera.cs.princeton.edu/algs4/assignments/percolation-100.png)

![](http://coursera.cs.princeton.edu/algs4/assignments/percolation-150.png)

![](http://coursera.cs.princeton.edu/algs4/assignments/percolation-204.png)

通过重复计算实验T次，并取结果的平均值，我们可以获得更为精确的联通阈值。假设计算实验t中打开的网格块数量为xt，则样本均值提供了联通阈值的估计值；样本标准差s则衡量了阈值的锐度。
$$
\overline x  = \frac{x_1 \, + \, x_2 \, + \, \cdots \, + \, x_{T}}{T},
\quad s^2  = \frac{(x_1 - \overline x )^2 \, + \, (x_2 - \overline x )^2 \,+\, \cdots \,+\, (x_{T} - \overline x )^2}{T-1}
$$
假设 T足够大（不少于30），以下式子提供了95%的联通阈值置信区间：
$$
\left [ \; \overline x  -  \frac {1.96 s}{\sqrt{T}}, \;\;
           \overline x  +  \frac {1.96 s}{\sqrt{T}} \; \right]
$$
为了完成一系列的计算实验，创建一名为`PercolationStats`的数据类型，并实现以下API：

```java
public class PercolationStats {
   public PercolationStats(int n, int trials)    // perform trials independent experiments on an n-by-n grid
   public double mean()                          // sample mean of percolation threshold
   public double stddev()                        // sample standard deviation of percolation threshold
   public double confidenceLo()                  // low  endpoint of 95% confidence interval
   public double confidenceHi()                  // high endpoint of 95% confidence interval

   public static void main(String[] args)        // test client (described below)
}
```

如果 *n* ≤ 0 或 *trials* ≤ 0，构造函数应该抛出`java.lang.IllegalArgumentException` 异常。

此外，`main()`方法接收命令行参数n和T，在n*n网格上进行T次独立的计算实验（如上），同时打印出样本均值，样本标准差，和联通阈值的95%置信区间。使用[StdRandom](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/StdRandom.html) 产生随机数，使用[StdStas](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/StdStats.html) 用于计算样本均值和样本平均差。

```
% java PercolationStats 200 100
mean                    = 0.5929934999999997
stddev                  = 0.00876990421552567
95% confidence interval = [0.5912745987737567, 0.5947124012262428]

% java PercolationStats 200 100
mean                    = 0.592877
stddev                  = 0.009990523717073799
95% confidence interval = [0.5909188573514536, 0.5948351426485464]


% java PercolationStats 2 10000
mean                    = 0.666925
stddev                  = 0.11776536521033558
95% confidence interval = [0.6646167988418774, 0.6692332011581226]

% java PercolationStats 2 100000
mean                    = 0.6669475
stddev                  = 0.11775205263262094
95% confidence interval = [0.666217665216461, 0.6676773347835391]
```

**分析运行时间和内存使用（可选不计入成绩）** **Analysis of running time and memory usage (optional and not graded)　** 通过[QuickFindUF](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/QuickFindUF.html) 的快速查找算法实现`Percolation`数据类型。

- 通过[Stopwatch](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/Stopwatch.html) 来估计对于不同的n和T`PercolationStats`的运行总时间。当n加倍的时候，总运行时间如何变化？当T加倍的时候，总运行时间如何变化？给出你电脑总运行时间关于n或T的函数关系式。


- 使用课堂中的64位内存消耗模型，计算出用于模拟n*n可联通系统的`Percolation`对象使用的总内存大小。统计包含`union–find`数据结构在内的所有内存使用量。

现在是时候去利用[WeightedQuickUnionUF](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/WeightedQuickUnionUF.html)中 *weighted quick union* 实现`Percolation`数据结构。回答前文的问题。

**用于提交的东西**　**Deliverables**　只需提交`Percolation.java`（使用来自[WeightedQuickUnionUF](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/WeightedQuickUnionUF.html)的weighted quick-[union](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/StdRandom.html) 算法）和`PercolationStats.java`。我们提供`algs4.jar`。你的提交作业中只允许调用[StdIn](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/StdIn.html)，[Stdout](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/StdOut.html)，[StdRandom](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/StdRandom.html)，[StdStats](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/StdStats.html)，[WeightedQuickUnionUF](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/WeightedQuickUnionUF.html)和 java.lang，不得调用其他库函数。

**娱乐** **For fun** 创建你自己的percolation输入文件并在论坛中分享它们。为了获得一些灵感，可以搜寻一些已经解决的拼图来构造一个图形。



*This assignment was developed by Bob Sedgewick and Kevin Wayne.* 
*Copyright © 2008.*

