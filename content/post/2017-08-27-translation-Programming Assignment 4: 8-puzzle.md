---
date: 2017-08-27T19:44:11+08:00
title: Programming-Assignment4-8Puzzle
tags: []
categories: ["技术"]
---

[编程作业4：８puzzle问题](http://coursera.cs.princeton.edu/algs4/assignments/8puzzle.html)

编写程序利用 A* 搜寻算法解决8-puzzle问题（和该问题自然的推广）。

**问题** **The problem**    [8-puzzle 问题](https://en.wikipedia.org/wiki/15_puzzle)是Noyes Palmer Chapman在1870年发明并普及的一个难题。游戏该怎么玩？在一个3*3的的网格中，有８个方块分别被标注为1至8，同时还有一个空的方块。你的目标就是尽量使用最少的移动步数来重新组织这些方块使得它们依次序排列。你可以水平方向和垂直方向上滑动一个方块到空的方块区域中。下图展示了从一个初始的棋盘（board）(左)变化为目标棋盘（右）的一系列符合规则的移动。

```
    1  3        1     3        1  2  3        1  2  3        1  2  3
 4  2  5   =>   4  2  5   =>   4     5   =>   4  5      =>   4  5  6
 7  8  6        7  8  6        7  8  6        7  8  6        7  8 

 initial        1 left          2 up          5 left          goal
```

**最佳优先搜索**　**Best-first search**　现在，我们介绍对于该问题的一种解法，该解法阐述了一种名为 [A* search algorithm](https://en.wikipedia.org/wiki/A*_search_algorithm)的常规人工智能方法。那么该如何定义策略的搜寻节点，一个棋盘，达到目标棋盘需要的移动步数，前一步的搜寻节点。第一步，将初始的搜寻节点（初始棋盘，0步移动，前一步搜寻节点为空）插入到一个优先队列中。然后，从优先队列中删除优先级最小的搜寻节点，同时将向优先队列中插入所有相邻搜寻节点（可通过刚刚出列的搜寻节点移动一步得到的节点称为相邻搜寻节点）。重复这个步骤直到出列的搜寻队列符合目标棋盘的样式。该方法的成功与否取决于搜寻节点优先级函数（ *priority function*）的选择。 考虑以下两个优先级函数：

* *Hamming priority function*    不在指定位置的区块数量＋目前为止达到搜寻节点锁移动步数。直觉告诉我们，搜寻节点如果不在指定为止的区块数量较小，那么它离需要达成的目标样式就很相近了，同时我们更喜欢使用尽量少的移动步数来达到目前的搜寻节点。
* *Manhattan priority function*　当前所有区块距离它们目标位置的Manhattan距离的和（水平方向和垂直方向上的距离之和）＋目前为止达到搜寻节点移动步数。

举个例子，下面的初始搜寻节点的Hanming优先级和Manhattan优先级分别是5，10。



```
 8  1  3        1  2  3     1  2  3  4  5  6  7  8    1  2  3  4  5  6  7  8
 4     2        4  5  6     ----------------------    ----------------------
 7  6  5        7  8        1  1  0  0  1  1  0  1    1  2  0  0  2  2  0  3

 initial          goal         Hamming = 5 + 0          Manhattan = 10 + 0

```

做一个关键性的观察：为了解决来自优先队列中给出的搜寻节点的难题，我们至少需要其优先级大小的总移动步数，不论是使用Hanming还是Manhattan优先级函数。（对于Hamming priority，每一个不在指定位置的区块必须移动一次以达到其指定位置。对于Manhattan priority，每一个区块必须移动对应其目标位置的Manhattan距离大小的步数。需要注意的是，当计算Hamming或Manhattan优先级的时候，我们不会计算空区块。）最后，当目标棋盘出列的时候，我们不仅可以得到的从初始棋盘变化至目标棋盘期间的一系列移动步骤，而且这也是移动步数最小的。（挑战：从数学上证明该结论的正确性。）

**关键性的最优化**　**A critical optimization**　最佳优先选择有一个烦人的特性：和一样的棋盘样式对应的搜寻节点会多次进入优先队列。为了减少不必要的无用搜寻节点的探测，涉及搜寻节点的相邻节点时，如果该相邻节点的棋盘样式和先前搜寻节点的棋盘样式相同，那就不要将该相邻节点入列。

```
8  1  3       8  1  3       8  1       8  1  3     8  1  3
 4     2       4  2          4  2  3    4     2     4  2  5
 7  6  5       7  6  5       7  6  5    7  6  5     7  6

 previous    search node    neighbor   neighbor    neighbor
                                      (disallow)
                                      
```

**第二个最优化**　**A second optimization**　为避免诸多的优先队列操作中每次都重新计算搜寻节点的Manhattan priority，在你构建该搜寻节点的时候提前计算它的Manhattan priority值；并将该值保存到一个实例变量中；在需要该值的时候返回（return）该值。类似这样的缓存技术应用广泛：当你需要重复计算相同的数同时计算这样的大量数据是一个时间瓶颈的操作之时，考虑使用缓存。

**游戏树** **Game tree** 　观测计算过程的一种方式就是*游戏树*（*game tree*），每一个搜寻节点是游戏树中的一个节点，该节点的子节点对应的是它的相邻搜寻节点。游戏树的根节点是初始搜寻节点；内部节点则是已经进行了移动的节点；枝叶节点通过优先队列来维持；每进行一步，A*算法从优先队列中移除具有最小优先级的节点并对其进行处理（添加它的子节点到游戏树中，同时添加子节点到优先队列中）。



![](http://coursera.cs.princeton.edu/algs4/assignments/8puzzle-game-tree.png)

**检测是否有解的难题**　**Detecting unsolvable puzzles**　不是所有的初始棋盘样式都可以通过一系列合理的移动成为目标棋盘样式，比如下面两个：

```
1  2  3         1  2  3  4
 4  5  6         5  6  7  8
 8  7            9 10 11 12
                13 15 14 
unsolvable
                unsolvable
```

为检测这种情况，我们利用这样的一个事实：棋盘样式依据是否能够成为目标样式可以分为两种相当的类型：（i）最终能够成为目标棋盘样式（ii）如果我们通过交换一对区块的位置（不包括空区块）修改初始棋盘样式后，该棋盘可以变成目标棋盘样式。（相当艰巨的挑战：数学上证明这个事实。）利用这个事实，在两个问题实例上同时运行A*算法－－一个是初始棋盘样式，一个是交换初始棋盘样式上的一对区块的位置，保持步调一致（两个方法在各种对应的游戏树中交替的不断探测搜寻节点。）二者中，必然有一个可以变成目标棋盘样式。

**棋盘和解法数据类型**　**Board and Solver data types**　通过创建一个不可修改的数据结构`Board`来创建你的程序，实现以下API:

```
public class Board {
    public Board(int[][] blocks)           // construct a board from an n-by-n array of blocks
                                           // (where blocks[i][j] = block in row i, column j)
    public int dimension()                 // board dimension n
    public int hamming()                   // number of blocks out of place
    public int manhattan()                 // sum of Manhattan distances between blocks and                                                // goal
    public boolean isGoal()                // is this board the goal board?
    public Board twin()                    // a board that is obtained by exchanging any pair                                              // of blocks
    public boolean equals(Object y)        // does this board equal y?
    public Iterable<Board> neighbors()     // all neighboring boards
    public String toString()               // string representation of this board (in the                                                  // output format specified below)

    public static void main(String[] args) // unit tests (not graded)
}
```

*边缘情况*　*Corner cases*　你可以假设构造函数接收一个n*n的数组，该数组包含n2个介于０至n2-1之间的整数，其中０代表空区块。

*性能要求*　*Performance requirements*　你的实现应该满足在最坏情况下所有的`Board`方法时间复杂度为n2（或更好）。

此外，创建一个不可修改的数据结构`Solver`，实现以下API：

```
public class Solver {
    public Solver(Board initial)           // find a solution to the initial board (using the                                              // A* algorithm)
    public boolean isSolvable()            // is the initial board solvable?
    public int moves()                     // min number of moves to solve initial board; -1 if                                            // unsolvable
    public Iterable<Board> solution()      // sequence of boards in a shortest solution; null                                              // if unsolvable
    public static void main(String[] args) // solve a slider puzzle (given below)
}
```

为了实现A*算法，你必须要使用[MinPQ](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/MinPQ.html) ，algs4.jar中用于优先队列的类。

*边缘情况*　*Corner cases*　如果传递一个空的参数，构造函数应抛出一个`java.lang.IllegalArgumentException`异常。

**解法检测客户端**　**Solver test client**　通过以下测试客户端从文件（以命令行参数指定）中读取数位块，打印出解法的标准输出。

```
public static void main(String[] args) {

    // create initial board from file
    In in = new In(args[0]);
    int n = in.readInt();
    int[][] blocks = new int[n][n];
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            blocks[i][j] = in.readInt();
    Board initial = new Board(blocks);

    // solve the puzzle
    Solver solver = new Solver(initial);

    // print solution to standard output
    if (!solver.isSolvable())
        StdOut.println("No solution possible");
    else {
        StdOut.println("Minimum number of moves = " + solver.moves());
        for (Board board : solver.solution())
            StdOut.println(board);
    }
}
```

**输入和输出格式**　**Input and output formats**　棋盘样式的输入和输出格式：首先是棋盘维度n，然后是n*n的棋盘样式，使用0代表空区块。举例如下：

```
% more puzzle04.txt
3
 0  1  3
 4  2  5
 7  8  6

% java Solver puzzle04.txt
Minimum number of moves = 4

3
 0  1  3 
 4  2  5 
 7  8  6 

3
 1  0  3 
 4  2  5 
 7  8  6 

3
 1  2  3 
 4  0  5 
 7  8  6 

3
 1  2  3 
 4  5  0   
 7  8  6 

3
 1  2  3 
 4  5  6 
 7  8  0
% more puzzle3x3-unsolvable.txt
3
 1  2  3
 4  5  6
 8  7  0

% java Solver puzzle3x3-unsolvable.txt
No solution possible
```

你的程序对于任意的n*n棋盘样式（对于任何2<128）都应该可正常运行，即使对于其中的一些例子它们运行时间很长，只要时间在一个合理的范围就行。

**提交的作业** **Deliverables** 　只需要提交文件`Board.java`和`Solver.java`（使用Manhattan priority）。我们提供algs4.jar库。除了`java.lang`，`java.util`，`algs4.jar`中的库函数，不得调用其他库函数。你必须使用为优先队列准备的[MinPQ](http://algs4.cs.princeton.edu/code/javadoc/edu/princeton/cs/algs4/MinPQ.html)函数。