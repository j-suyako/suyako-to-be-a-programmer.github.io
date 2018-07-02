---
layout: post
title: "Coursera Algorithms —— week(4) 8 puzzle"
category: Algorithms
---

![](https://mmbiz.qpic.cn/mmbiz_gif/ickltPAryrfbhlWWtP2ibNickVKjtG8ics2dp5Ns8jBYBRliaYtxIOE8rQSHbPIURPV7ffxQjwgNGJ0nK6eje9QHqTQ/0?wx_fmt=gif)

## Assignment

> 本周作业地址：http://coursera.cs.princeton.edu/algs4/assignments/8puzzle.html
>
> 用A\*算法求解8puzzle问题（类似于华容道）的最短路径，如下图所示
>
> ![](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfZn7CxuCOIQGzOPy1571vH88KAagvqf8xia3AeSnEPhmqrU28g3TGNkS34nMthqAE24SNuZLgetoRA/0?wx_fmt=png)

## Introduction

先来看下wiki上对A\*算法的介绍

![](https://mmbiz.qpic.cn/mmbiz_gif/ickltPAryrfZn7CxuCOIQGzOPy1571vH8TPd4rqbuxvxtllfXQTCBa7PnY4WeFTxojV6JKG0yAAONVKKX4Ye6kA/0?wx_fmt=gif)

如上图所示，目标是求出起始点（红点A）到目标点（绿点B）之间的最短路径，途中有墙相隔。在算法求得的路径经过其中某一点时，会用一个填充的小圆表示该点，在该点的相邻点中（即从A点开始走一步就可以到的点），如果有算法未曾经过的点，则用一个未填充的小圆来表示。每个点的填充颜色代表与B点的距离远近，红色的较远，原谅色的较近。

具体的实现原理如下：

1. 从A点开始，首先计算其与B点的距离L，然后另A点的优先级等于L，即`A.priority = L`；
2. 接着以A点的第一个相邻点A‘为例，计算其与点B的距离为L’，另`A'.priority = 1 + L'`，这里的1代表第一步，然后将A‘存入数组`P`；
3. 对A点的所有相邻点进行同样操作后一并放入数组`P`；
4. 从数组`P`中找到priority属性最小的点M（同时在数组中删除该点），另`M.predecessor = A`，表示M点的上一步是A，若`M != B`，重复2~4；
5. 最后从B开始，提取其predecessor属性，直到得到A位置，每次取出的点即为最终A点到B点的路径。

上述算法中有一个值得注意的地方，就是在priority属性的计算中，我们是将移动步数也计算进去的，回到开始的gif中，其表现形式就是在路径碰壁后，接下来从数组`P`中提取priority属性最小的点时，会从A点的临近点重新开始计算路径（可以看到一开始的路径碰壁后路径计算会从A点附近重新开始）。

~~感觉这个算法应该能解迷宫（￣▽￣）~~

A\*算法本身应该比较容易理解，但在具体实现过程中有一个难点：在第四步时我们需要从数组`p`中找到priority属性最小的点M然后删除该点。对于一个长度为n的数组，找到其中的最小值需要的时间复杂度是O(n)，删除该点的时间复杂度也为O(n)。然而在这个问题中，我们并不知道中间会循环2-4几次，每次都需要O(n)的开销去找最小值并删掉显然不划算，况且这个数组长度可能还在不断增加（这里再提醒一下，不是对本轮循环中新增的点进行比较，而是要对整个数组进行比较，具体原因就是之前说的，在路径碰壁后很可能需要从源头开始重新计算）。

为了解决**找到动态变化的数组中最小值**的问题，就要介绍下今天要讲的新的数据结构——二叉堆(heap)。

因为课程给的demo是用来找到最大值的，自己画图又太麻烦，我这里也就以找到数组中的最大值为例来写了。一个有序的max-heap应该如下图所示：

![52130358530](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfZn7CxuCOIQGzOPy1571vH81SDBBVotvKZLkE2L0frM5nribCAjibMo7Vd7oqDAH20fnw3mIia6vmyag/0?wx_fmt=png)

如果用数组表示就是`[,T,S,R,P,N,O,A,E,I,H,G]`，怎么排的应该都看的出吧......这个数组`a`具有以下两个特点：

1. `a[0]`为空；
2. 对任意k，有`a[k] <= a[k/2]`，这里的除法为向下整除，也就是3/2=1（这个特性把0放空了，因为写起来会省事一点）；

是的，就是这么简单，只要一个数组满足以上两个条件，我们就称其为堆有序的，数组的第一个值就是该数组的最大值。

再回想一下问题，我们有两个操作需要完成：新增一个元素，删除数组的最小值（这里替换成最大值，原理是一样的）。

现在我们尝试着在这个数组中新增一个元素：

![](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfZn7CxuCOIQGzOPy1571vH8oLKynh8yrUFK4SjeFxibvicVKrE2eFSVqNLFNpLk77UsFtKZCrgGcofg/0?wx_fmt=png)

步骤如下：

1. 将待插入值放在`a[end]`，该例中`end = 11`，`a[11] = S`；
2. 比较`a[k]`与`a[k/2]`，如果`a[k] > a[k/2]`，交换两者的值，否则认为堆有序，循环结束，该例中交换了S和H；
3. 令`k = k/2`，重复2直至堆有序

接下来再看下删除最大值：

![](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfZn7CxuCOIQGzOPy1571vH8O4zKfDkrcE0l2Job2MLS59IvQGth1AIofKCwTTqjye53cfOgiadzyXw/0?wx_fmt=png)

步骤如下：

1. 交换`a[1],a[end]`，然后删除`a[end]`，该例中交换T和H后交换删除了T；
2. 选取`a[2],a[3]`中较大的一个`a[k]`，如果`a[k] > a[1]`，将其与`a[1]`交换，该例中交换了S和H；
3. 选取`a[2*k],a[2*k+1]`中的较大值，如果其大于`a[k]`，与`a[k]`交换，重复该步骤直到`a[k] > a[2*k] && a[k] > a[2*k+1]`，该例中最后交换到P和H就停止了

至此，一个二叉堆及其所需要的在A*算法中所需要的操作就构造完成了，可以很容易证明插入与删除的复杂度都是O(lgn)的（以2为底）。

写完了这些终于可以写本周作业了，这里再来回顾一下作业题。

![](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfZn7CxuCOIQGzOPy1571vH88KAagvqf8xia3AeSnEPhmqrU28g3TGNkS34nMthqAE24SNuZLgetoRA/0?wx_fmt=png)

我们对每种排布情况称为一个board，board中的每一个数字称为一个block，现在需要解决的最后一个问题就是每个board与goal之间的距离该如何定义？

作业给出了两种距离定义方式，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfZn7CxuCOIQGzOPy1571vH8OfC76XfdUTVVCpzCg2taXrPUgHOy0PbOdKxIODhXy1ib3TfI6MoOlcg/0?wx_fmt=png)

Hamming就是不对应的位置的总数，Manhattan就是每个block与其对应位置的Manhattan距离（垂直+水平）相加。

最后的最后，由于并不是每个初始board都是有解的，我们还需要构造一个初始board的twinBoard（通过交换非空的任意两个block得到），可以证明两者中必有一个有解（作业这么说的，然而我并不会证明），所以对这两个board依次进行A*算法就可以啦~

## Programming

写得累死了，一想到下周的KdTree我就不由得脑阔疼Σ(ﾟдﾟ;)

一样，先看这次作业提供的两个API。

```java
public class Board {
    public Board(int[][] blocks)           // construct a board from an n-by-n array of blocks
                                           // (where blocks[i][j] = block in row i, column j)
    public int dimension()                 // board dimension n
    public int hamming()                   // number of blocks out of place
    public int manhattan()                 // sum of Manhattan distances between blocks and goal
    public boolean isGoal()                // is this board the goal board?
    public Board twin()                    // a board that is obtained by exchanging any pair of blocks
    public boolean equals(Object y)        // does this board equal y?
    public Iterable<Board> neighbors()     // all neighboring boards
    public String toString()               // string representation of this board (in the output format specified below)

    public static void main(String[] args) // unit tests (not graded)
}
```

```java
public class Solver {
    public Solver(Board initial)           // find a solution to the initial board (using the A* algorithm)
    public boolean isSolvable()            // is the initial board solvable?
    public int moves()                     // min number of moves to solve initial board; -1 if unsolvable
    public Iterable<Board> solution()      // sequence of boards in a shortest solution; null if unsolvable
    public static void main(String[] args) // solve a slider puzzle (given below)
}
```

不知道有没有看的人（不存在的）也在学这门课的，我这里就放一些比较关键的代码了。Board比较简单，我直接写Solver。

这里说下，二叉堆的数据结构作业会提供，就是下面代码中的MinPQ。

由于Board的method提供得并不全，需要在Solver中新建一个新的searchNode类才能解决问题，searchNode类代码如下：

```java
private class searchNode implements Comparable<searchNode> {
        private Board board;               // corresponding board to this searchNode
        private searchNode predecessor;    // predecessor searchNode of this searchNode
        private int moves;                 // corresponding moves to this searchNode
        private int priority;              // corresponding priority to this searchNode

        public searchNode(Board bd, searchNode pre, int m) {
            this.board = bd;
            this.predecessor = pre;
            this.moves = m;
            this.priority = m + bd.manhattan();  // in this code use manhattan distance to calculate priority
        }

        public boolean isGoal() {
            return this.board.isGoal();
        }

        public Iterable<searchNode> neighbors() {
            Stack<searchNode> neighbors = new Stack<>();
            for (Board neighbor: board.neighbors()) {
                // should not push a neighbor if its board is the same as the board of the predecessor search node
                if (moves == 0 || !predecessor.board.equals(neighbor)) {
                    searchNode e = new searchNode(neighbor, this, moves+1);
                    neighbors.push(e);
                }
            }
            return neighbors;
        }

        public int compareTo(searchNode that) {
            if (this.priority < that.priority) return -1;
            if (this.priority > that.priority) return 1;
            // breaking tie
            else return Integer.compare(that.moves, this.moves);
        }
    }
```

这里有两个需要非常注意的地方（整个算法效率的关键所在），第一个是我们在求一个board的相邻board时，可能会把board的predecessor重复计入，这个是一定要避免的；第二个是我上述代码中最后注释breaking tie的地方（并不知道中文怎么翻译），作用是当两个board的priority相等时，需要再增加一个约束条件。这个新的约束条件并不是固定的，因为对不同的情况应该有不同的最优解。我这里根据最后的测试情况，认为priority相同时，所用步数较小的应该排在顶端（想一下最开始说明A\*算法时用到的gif，路径碰壁后应该从A点附近重新开始计算路径）。

```java
public class Solver {
    private Stack<Board> solution;
    private boolean solvable;
    private int moves;

    public Solver(Board initial) {
        /* find a solution to the initial board */
        if (initial == null) throw new IllegalArgumentException();
        searchNode currNode = new searchNode(initial, null, 0);
        searchNode currTwinNode = new searchNode(initial.twin(), null, 0);
        solution = new Stack<>();
        solvable = false;
        moves = -1;
        // operate heap of initialBoard and twinInitialBoard alternately
        MinPQ<searchNode> PQ1 = new MinPQ<>();
        MinPQ<searchNode> PQ2 = new MinPQ<>();
        PQ1.insert(currNode);
        PQ2.insert(currTwinNode);
        while (!currNode.isGoal() && !currTwinNode.isGoal()) {
            for(searchNode each : currNode.neighbors())
                PQ1.insert(each);
            currNode = PQ1.delMin();
            //StdOut.println(initial.predecessor());
            for(searchNode each : currTwinNode.neighbors())
                PQ2.insert(each);
            currTwinNode = PQ2.delMin();
        }
        if (currNode.isGoal()) {
            solvable = true;
            moves = currNode.moves;
            while (currNode != null) {
                solution.push(currNode.board);
                currNode = currNode.predecessor;
            }
        }
        else {
            solution = null;
        }
    }

    public Iterable<Board> solution() {
        // Queue here was to FIFO
        Queue<Board> copyOfSolution = new Queue<>();
        if (solution != null) {
            for (Board each : solution)
                copyOfSolution.enqueue(each);
            return copyOfSolution;
        }
        else return null;
    }
}
```

剩余的其实没啥好说的......该讲的应该也都讲了，一个是board与twinBaord的交替操作，两者中一个达成goal时终止循环；另一个是solution这边要求返回Immutable(可以参见上章)类型，所以返回了一个原solution的copy，然后这里用的是Queue是因为原solution为从goal board到initial board的栈，栈是后进先出的，所以for循环时首先弹出的是initial board，这个没问题。但如果copy也是用stack的话在对copy结果进行输出时结果会倒过来，变成goal到initial（说得有点绕，意会就好）。

## Summary

1. 树的概念；
2. 二叉堆的实际应用；
3. java使用熟练了一些(･∀･)

下周作业：kdtree