---
date: 2017-08-26T19:44:11+08:00
title: Programming-Assignment3-Pattern-Recognition
tags: []
categories: ["技术"]
---

[编程作业3：模式识别](http://coursera.cs.princeton.edu/algs4/assignments/collinear.html)

编写程序识别点集合中的线性图案。

计算机视觉涉及分析视觉影像图形，重新构建生产它们的真实对象。该过程经常分成两个阶段进行：特征检测（*feature detection*）和图形识别（*pattern recognition*）。特征检测涉及选择重要图形的重要特征；图形识别涉及发现特征中的图形。我们将研究一个特定的涉及点和线段（points and line segments）的图形识别问题。这种类型的模式识别在许多其他的应用中也存在：比如统计数据分析。

**问题** **The problem**    在平面上给出n个不同的点，找出所有（最长的）连接4个或4个以上点的直线段。

![](http://coursera.cs.princeton.edu/algs4/assignments/lines2.png)

**点阵数据结构**  **Point data type**    创建一个不可修改的数据结构`Point`，通过实现以下API表示平面上的一个点：

```java
public class Point implements Comparable<Point> {
   public    Point(int x, int y)                // constructs the point (x, y)

   public    void draw()                        // draws this point
   public    void drawTo(Point that)            // draws the line segment from this point to that point
   public    String toString()                  // string representation

   public    int compareTo(Point that)          // compare two points by y-coordinates, breaking ties by x-coordinates
   public    double slopeTo(Point that)         // the slope between this point and that point
   public    Comparator<Point> slopeOrder()     // compare two points by slopes they make with this point
}
```

作为开始，数据结构`Point.java`实现了构造函数（constructor ）和`draw()`，`drawTo()`，和`toString()`方法。你的工作是实现以下成分：

- `compareTo()`方法需要通过各自的 y坐标比较，此时不考虑它们的 x坐标。确切来说，仅当y0 < y1 或者 y0 = y1 并且 x0 < x1，调用点（x0, y0）小于参数点（x1, y1）。
- `slopeTo()`方法应该返回调用点（x0, y0）和参数点（x1, y1）之间的斜率，通过公式（y1 - y0）/（x1 - x0）得出。将水平线段斜率视为 +0，将垂直线段视斜率视为正无穷，将退化的线段（一个点和它自身）斜率视为负无穷。
- `slopeOrder()`方法应返回一个比较器（comparator）用于比较两个参数点对于调用点（x0, y0）的斜率大小。确切来说，如果斜率（y1 - y0）/（x1 - x0）小于斜率（y2 - y0）/（x2 - x0），那么点（x1, y1）小于点（x2, y2）。对于水平线段，垂直线段和退化线段参照`slopeTo()`方法。
- 不要重写`equals()`和`hashCode()`方法。

*边缘情况*  *Corner cases*    为避免潜在的整形溢出和浮点精度带来的不便，你可以假设构造函数参数 x和 y都介于 0至32767之间。

**直线线段数据结构**  **Line segment data type**   为了表示平面的直线段，使用数据结构 [LineSegment.java](http://coursera.cs.princeton.edu/algs4/testing/collinear/LineSegment.java) ，它实现了以下API：

```java
public class LineSegment {
   public LineSegment(Point p, Point q)        // constructs the line segment between points p and q
   public   void draw()                        // draws this line segment
   public String toString()                    // string representation
}
```

**暴力算法**  **Brute force**     编写程序`BruteCollinearPoints.java`每次取4个点，检查它们是否在统一直线线段上，返回符合该要求的所有直线线段。为了检查4点：p, q, r , s是否共线，检查以下三个斜率是否相等：p与q之间的斜率，p和r之间的斜率，p和s之间的斜率。

```java
public class BruteCollinearPoints {
   public BruteCollinearPoints(Point[] points)    // finds all line segments containing 4 points
   public           int numberOfSegments()        // the number of line segments
   public LineSegment[] segments()                // the line segments
}
```

方法`segments()`只能包含一条涵盖了４个点的直线线段一次。如果４个点以*p*→*q*→*r*→*s*的顺序出现在一条直线线段上，那么你应该包含直线线段 *p*→*s* 或 *s*→*p*（不可同时包含俩）中的一条，此外不应该包含*p*→*r* 或 *q*→*r*这样的直线线段。对于`BruteCollinearPoints`不会出现５个及５个以上共线的点。

*边缘情况*　*Corner cases*   以下情况抛出``java.lang.IllegalArgumentException` `异常：如果构造函数的参数为空，如果数组指针为空，如果构造函数参数包含重复的点。

*性能指标*　*Performance requirement*　程序的时间复杂度最坏情况下需为 n4，空间使用需为正比与 n加直线线段返回的数量。

**基于排序，速度更快的解法**　**A faster, sorting-based solution**　　值得注意的是，寻找比上述暴力算法更快的解法是有可能的。给定一个点 p，以下方法可判定 p是否与４或多余４个的共线点共线。

- 将点 p看作原点
- 对于其余的点 q，确定点q和原点p之间的斜率
- 按照各点和原点p之间的斜率对这些点进行排序
- 检查是否存在3个（或以上）依序排列的相邻点各自对原点p的斜率都相等。如果存在，那么这些点再加上p则是共线点

将以上方法依次序应用再 n个点上产生了一个解决问题的高效算法。原因在于和原点p之间的斜率相等的所有点比如共线，而排序正好将这些点聚集在一起。该算法更高效的原因在于操作瓶颈是排序。

![](http://coursera.cs.princeton.edu/algs4/assignments/lines1.png)

编写程序`FastCollinearPoints.java`实现该算法。

```java
public class FastCollinearPoints {
   public FastCollinearPoints(Point[] points)     // finds all line segments containing 4 or more points
   public           int numberOfSegments()        // the number of line segments
   public LineSegment[] segments()                // the line segments
}
```

方法`segments()`应该包括最长的直线线段，包含4个（或更多）的共线点，只允许包含一次。比如说，如果一条直线线段中出现了五个共线点，依次序排列为： *p*→*q*→*r*→*s*→*t*，那么就不能包含其子集：*p*→*s* 或 *q*→*t*。

*边缘情况*　*Corner cases*　 以下情况抛出``java.lang.IllegalArgumentException` `异常：如果构造函数的参数为空，如果数组指针为空，如果构造函数参数包含重复的点。

*性能要求*  *Performance requirement*    程序的最坏情况下时间复杂度为n2logn，空间使用需为正比与 n加直线线段返回的数量。即使输入中包含5个及５个以上的共线点时`FastCollinearPoints`也需要可以正常工作。

**示例客户端**　**Sample client**　客户端程序将输入文件名作为命令行参数，读取输入文件（通过以下指定的格式）；一行行将程序发现的直线线段的打印成标准输出；同时将直线线段绘制成标准的绘制样式。

```java
public static void main(String[] args) {

    // read the n points from a file
    In in = new In(args[0]);
    int n = in.readInt();
    Point[] points = new Point[n];
    for (int i = 0; i < n; i++) {
        int x = in.readInt();
        int y = in.readInt();
        points[i] = new Point(x, y);
    }

    // draw the points
    StdDraw.enableDoubleBuffering();
    StdDraw.setXscale(0, 32768);
    StdDraw.setYscale(0, 32768);
    for (Point p : points) {
        p.draw();
    }
    StdDraw.show();

    // print and draw the line segments
    BruteCollinearPoints collinear = new BruteCollinearPoints(points);
    for (LineSegment segment : collinear.segments()) {
        StdOut.println(segment);
        segment.draw();
    }
    StdDraw.show();
}
```

**输入格式**　**Input format**　我们提供以下格式的几个而示例输入文件（对使用上述测试客户端适用）：整数 n，后面是 n对整数对（x, y），整数介于０和32767之间。以下是几个例子。

```java
% more input6.txt       % more input8.txt
6                       8
19000  10000             10000      0
18000  10000                 0  10000
32000  10000              3000   7000
21000  10000              7000   3000
 1234   5678             20000  21000
14000  10000              3000   4000
                         14000  15000
                          6000   7000
% java BruteCollinearPoints input8.txt
(10000, 0) -> (0, 10000) 
(3000, 4000) -> (20000, 21000) 

% java FastCollinearPoints input8.txt
(3000, 4000) -> (20000, 21000) 
(0, 10000) -> (10000, 0)

% java FastCollinearPoints input6.txt
(14000, 10000) -> (32000, 10000) 
```

**提交的作业**　**Deliverables**　只需提交`BruteCollinearPoints.java`, `FastCollinearPoints.java`和`Point.java`三个文件。我们提供了`LineSegment.java` 和 `algs4.jar`。只允许使用以下库函数：`java.lang`, `java.util`, and `algs4.jar`，不得调用其他库函数。需特别指明的是，你可能需要调用`Arrays.sort()`。

This assignment was developed by Kevin Wayne. 
Copyright © 2005.