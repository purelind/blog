---
date: 2017-08-30T19:44:11+08:00
title: Programming-Assignment-5-Kd-Trees
tags: []
categories: ["技术"]
---

[编程作业5：Kd-Trees](http://coursera.cs.princeton.edu/algs4/assignments/kdtree.html)

编写一个数据结构用于描绘单位正方形内的一系列点（所有的点都有x坐标和y坐标的值都在０－１之间），通过2d-tree来实现高效的范围查找（*range search*）（找出所有在查询的长方形区域中的点），同时实现最佳相邻点查询（*nearest-neighbor search*）（找出对于查询点距离最短的点）。2d-tree有相当多的应用，电脑动画天体物体分类，加速神经网络，数据挖掘，图像修复。

![](http://coursera.cs.princeton.edu/algs4/assignments/kdtree-ops.png)

**几何图元**　**Geometric primitives**　首先，使用以下的几何图元来表示平面上的点和轴对称的长方形。

![](http://coursera.cs.princeton.edu/algs4/assignments/RectHV.png)

* 数据结构[Point2D](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/Point2D.html)（algs4.ja中的一部分）不可修改，其用于表示平面的点。这是你可能需要使用的一部分API:

  ```java
  public class Point2D implements Comparable<Point2D> {
     public Point2D(double x, double y)              // construct the point (x, y)
     public  double x()                              // x-coordinate 
     public  double y()                              // y-coordinate 
     public  double distanceTo(Point2D that)         // Euclidean distance between two points 
     public  double distanceSquaredTo(Point2D that)  // square of Euclidean distance between                                                    // two points 
     public     int compareTo(Point2D that)          // for use in an ordered symbol table 
     public boolean equals(Object that)              // does this point equal that object? 
     public    void draw()                           // draw to standard draw 
     public  String toString()                       // string representation 
  }
  ```

* 数据结构[RectHV](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/RectHV.html)（algs4.jar中的一部分）不可修改，其用于表示轴对称长方形。这是你可能需要使用的一部分API:

  ```java
  public class RectHV {
     public    RectHV(double xmin, double ymin,      // construct the rectangle [xmin, xmax]                      double xmax, double ymax)     // x [ymin, ymax] 
     // throw a java.lang.IllegalArgumentException if (xmin > xmax) or (ymin > ymax)
     
     public  double xmin()                           // minimum x-coordinate of rectangle 
     public  double ymin()                           // minimum y-coordinate of rectangle 
     public  double xmax()                           // maximum x-coordinate of rectangle 
     public  double ymax()                           // maximum y-coordinate of rectangle 
     public boolean contains(Point2D p)              // does this rectangle contain the point                                                    // (either inside or on boundary)? 
     public boolean intersects(RectHV that)          // does this rectangle intersect that                                                      // rectangle (at one or more points)? 
     public  double distanceTo(Point2D p)            // Euclidean distance from point p to                                                      // closest point in rectangle 
     public  double distanceSquaredTo(Point2D p)     // square of Euclidean distance from                                                        // point p to closest point in rectangle 
     public boolean equals(Object that)              // does this rectangle equal that                                                          // object? 
     public    void draw()                           // draw to standard draw 
     public  String toString()                       // string representation 
  }
  ```

不要更改这些数据结构。

**暴力算法实现**　**Brute-force implementation**　编写一个不可修改的数据结构`PointSET.java`用于描绘单位正方形内一系列的点。通过红黑树（red–black BST）实现下列API：

```java
public class PointSET {
   public         PointSET()                               // construct an empty set of points 
   public           boolean isEmpty()                      // is the set empty? 
   public               int size()                         // number of points in the set 
   public              void insert(Point2D p)              // add the point to the set (if it 　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　// is not already in the set)
   public           boolean contains(Point2D p)            // does the set contain point p? 
   public              void draw()                         // draw all points to standard draw 
   public Iterable<Point2D> range(RectHV rect)             // all points that are inside the                                                              // rectangle (or on the boundary) 
   public           Point2D nearest(Point2D p)             // a nearest neighbor in the set to                                                            // point p; null if the set is empty 
   public static void main(String[] args)                  // unit testing of the methods                                                                  // (optional) 
}
```

*实现要求*　*Implementation requirements*　你必须使用[SET](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/SET.html)或者[java.util.TreeSet](https://docs.oracle.com/javase/8/docs/api/java/util/TreeSet.html)；不要自己实现红黑树。

*边缘情况*　*Corner cases*　如果任何参数为空，抛出``java.lang.IllegalArgumentException` 异常。

*性能要求*　*Performance requirements*　你的实现必须保证在最坏情况下，`insert()`和`contains()`方法用时正比与点集合中所有点的数量的对数；保证`nearest()`和`range()`方法用时正比与点集合中点的数量。

**2d-tree实现**　**2d-tree implementation**　编写一个可修改的数据结构`KdTree.java`，使用2d-tree实现相同二API（将`PointSET`用`KdTree`替代）。2d-tree是使用二为键值（keys）二叉查找树（BST），其思想就是建立一个指向节点的BST，以严格的交替顺序将点的x坐标值和y坐标值作为键值（keys）.

* *查找和插入*　*Search and insert*　用于查找和插入的算法实现类似BSTs算法中的查找和插入，但是在根部我们使用x坐标值（如果插入的点x坐标值小于根部点的x坐标值，将插入点放在左边子结点；否则，放入右边子结点）;之后在下一层级中，我们使用y坐标值（如果插入的点x坐标值小于根部点的x坐标值，将插入点放在左边子结点；否则，放入右边子结点）；之后下一层中，使用x坐标值，按照这样的规律进行下去。

  ![](http://oql549fm2.bkt.clouddn.com/algorithm-part-I-assi5-pic1.png)

* *绘图*　*Draw*　2d-tree通过简单的方式划分单元长方形：根部左侧的所有点放入左侧子树；根部右侧所有点放入右侧子树；递归的重复下去。`draw()`方法应该将所有的点依标准绘制绘制成黑色，垂直分割线为红色，水平分割线为蓝色。该方法不要求高效实现－其主要用于调试。

2d-tree相对于BST来说主要的优势在于2d-tree能够高效的实现范围查找和最近相邻查找。每一个结点对应着一个单元长方形中轴对称矩形，该矩形包含了所有其子树中的点。根部对应着单元矩形；根部左右两个子节点对应着根部点x坐标分开的两个矩形，这样重复下去。

* *范围查找*　*Range search*　为找到查询矩形区域内所有的点，使用下面的*精简规则*（*pruning rule*）从根部点开始不断递归的查找两个子树中的点：如果查询矩形与结点对应的矩形不相交，那就不必去检测那个结点（或者说它的子树）。只有在一个子树中可能包含查询矩形区域内中的点时候，才需要去检测该子树。
* *最近相邻查找*　*Nearest-neighbor search*　为了一个查询点相邻最近的点。使用下面的*精简规则*（*pruning rule*）从根部点开始不断递归的查找两个子树中的点：如果目前发现的最近点距离小于查询点和结点对应矩形见的距离，那就不必检测那个结点（或者说它的子树）。也就是说，只有当一个结点可能包含一个距离查询点距离小于目前已知最近的点的时候，该结点才需要去检测。精简原则的效率取决于快速发现一个临近点。为做到这样，需要组织下递归的方法艺保证：当有两个可能的子树需要进一步检测的时候，你需要总是选择查询点一侧的子树作为优先的子树进行检测－当检测第一个子树时发现的最近点会精简第二个子树检测查询。

**客户端**　**Clients**　你可能需要以下交互性的客户端程序用于检测和调试你的代码。

* [KdTreeVisualizer.java](http://coursera.cs.princeton.edu/algs4/testing/kdtree/KdTreeVisualizer.java)　计算并绘制出用户在标准绘制窗口点击一系列点所产生的2d-tree。
* [RangeSearchVisualizer.java](http://coursera.cs.princeton.edu/algs4/testing/kdtree/RangeSearchVisualizer.java)　从文件中（由命令行参数指定）读取一系列的点，将这些点插入一个2d-tree中。之后，用户在一个标准绘图窗口拖曳出一个轴对称矩形，程序在该矩形区域内进行范围查找。
* [NearestNeighborVisualizer.java](http://coursera.cs.princeton.edu/algs4/testing/kdtree/NearestNeighborVisualizer.java)　从文件中（由命令行参数指定）读取一系列的点，将这些点插入一个2d-tree。之后，在一个标准绘图窗口中鼠标指针位置进行最近相邻点查询。

**运行时间和内存使用的分析（可选，不计入成绩）**　**Analysis of running time and memory usage (optional and not graded)**

* 使用讲义和课本1.4节中的内存消耗模型，算出你的以点的数量n为函数的2d-tree数据结构的全部内存使用的字节数（bytes）。计算2d-tree使用的所有内存，包括：结点，点，矩形所使用的内存。
* 算出在单元矩形中创建一个有n个随机点的2d-tree需要花费的时间，以秒为单位计算。（不要将从标准输入中读取点的时间计入其中。）
* 当查询点随机分布在单元矩形中的末个位置，你的2d-tree实现对于[input100K.txt](http://coursera.cs.princeton.edu/algs4/testing/kdtree/input100K.txt)和[input1M.txt](http://coursera.cs.princeton.edu/algs4/testing/kdtree/input1M.txt)，一秒中可以进行多少次最近相邻计算？（不要将读取点和创建2d-tree的时间计入其中。）然后使用暴力算法重复这个问题试试。

**提交的作业**　**Submission**　只需要提交文件`PointSET.java`和`KdTree.java`。我们提供`algs4.jar`。你不得调用`java.lang`,`java.util`和`algs4.jar`之外的库函数。



*This assignment was developed by Kevin Wayne.*