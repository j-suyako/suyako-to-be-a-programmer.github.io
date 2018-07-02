---
layout: post
title: "Coursera Algorithms —— week(1) union-find"
category: Algorithms
---

### 写在前面

说了好久的想开个公众号来放我自己的学习日志，今天总算是放出来了。申明一下开公众号的目的，上学期经过一番思想斗争之后决定还是转计算机，之后在咨询经验的时候有人建议我记录自己的学习过程（感谢鸟哥）。其实之前也有类似的想法，不过也是这番话才让我下定决心去干这件事。至于为什么博客能做的事要放到公众号上来，主要还是抱着一丝侥幸看能不能传播得远一些，这样一来可以吸引一些和我有共同想法共同经历的人交流经验互相学习（有哪位大佬愿意加我我也是很开心的\_(:3」∠)\_，我的微信ID：BEAFINEGIRLKISSME，也可以在公众号里直接留下id(⌒▽⌒)）；二来公众号~~有人看的话~~能稍微督促一下自己；最后是应该会记录今年自己的秋招经历，希望对后来的学弟学妹能有所帮助。

其实这篇已经是我写的第二篇了......第一篇是另一个系列，感觉不一定能写下去，所以先拿coursera平台的Algorithms课程作为开篇系列吧。课程分为上下两部分，总共应该有12期，分为排序->搜索->图->字符串等几大块，我自己现在的进度大概在搜索的红黑树那里。本课程每周在讲完一个算法后都有一个编程作业，不会要你自己撸一遍算法（当然自己写一遍最好），基本都在考察你对算法的运用，大体来说作业都挺有趣的。在本地完成作业调试通过后即可上传，系统会对你的程序进行正确性、时间及内存的测试，总分100，得到80分即可通过。本系列主要记录的就是我针对这些作业写的程序以及过程中碰到的问题及思考。

### 所以第一周讲了什么？

第一周其实基本没讲啥（围笑脸）。

好吧，第一周作为介绍性课程，讲了java入门，栈、队列、链表等数据结构，这些就跳过了，反正网上一堆资料讲得比我好。最后为了说明设计和分析算法的基本方法，给了一个具体的引导性的例子，如下图所示：

![](http://p4mu7u37x.bkt.clouddn.com/pqconnected.png)

可以把连接*p*和*q*的路径看成迷宫的通关路线，不过作业没要我们求*pq*之间的路径这么复杂，我们只要求*p*和*q*之间是否存在这么一条路径，也即下文所说的连通性，就可以了（其实问题这么一转换就完全变成另一个问题了）。

连通性的问题有很多运用，具体见下图：

![](http://p4mu7u37x.bkt.clouddn.com/连通模型的运用.png)

很多我也不太确定是什么意思，一些大概迷迷糊糊有点感觉，但不太明白具体该咋操作（像是集合那个），如果有翻译错的地方还请告知。

接下来我们从一个小例子来看一个连通性判断问题：

![](http://p4mu7u37x.bkt.clouddn.com/connection_procedure.png)

如上图所示，共有编号0-9的10个点，开始处于相互独立的状态。第一步用红线连接3、4，第二步连接3、8......第五步连接2、1，到这里都是基本操作（滑稽）。第六步连接8、9的时候，因为之前的连接8、9在这时其实已经处于连接的状态，所以我们实际应该**跳过这一步**，依次类推到最后一步连接6、7。

至此，用一句话总结我们的问题，就是：如何在第六步连接8、9的时候判定8和9已经连通了？

### 算法介绍

这里不贴代码，只通过文字和图片来讲思路（下文所有的`log(n)`均已2为底数）。

#### **quick-find**

还是放图说明

![](http://p4mu7u37x.bkt.clouddn.com/quick-find.jpg)

这里为简单起序列里只放3、4、8、9四个点，初始状态```id[i] = i```，表示各个点处于相互独立的状态，这里`i`对应0-9。在经过第一次的4、3连接后（注意连接顺序），我们让```id[3] = 4```，这样当我们想要查询3、4两个点是否连通时只要判断```id[3] == id[4]```就可以了，返回True表示连通，False表示不连通。接着我们尝试连接8、3，由于```id[8] != id[3]```，我们将id[8]的值改为4。接着再连接9、4，像前两步一样我们将id[4]改为9，但除了修改```id[4]```之外，我们还要修改```id[3]```和```id[8]```，因为3、8两个点是和4连通的，所以我们令```id[3] = 9， id[4] = 9```，对于程序来说就是循环一遍列表，判断```id[i] == 4```，返回True就修改```id[i] = 9```。这里就是quick-find主要的开销所在了，因为你要找到所有id为4的`i`只能对列表做一次循环（这里可能会问为什么不修改`id[9]`，这样就只修改一次了，但对于程序来说你要判断哪个修改的次数更少也一样要做一次循环）。最后我们连接8、9，由于两个id相同，就可以不做任何修改了。

整理一下上述步骤：

* 连接*p*,*q*，若```id[q] != id[p]```，则另```id[q] = id[p]```
* 对列表进行循环，对所有```id[i] == id[q]```，令```id[i] = id[p]```

我们来计算每一步连接的开销：

* 访问`id[p], id[q]`，**2**次访问数组开销
* 对列表循环，**n**次访问数组开销，这里**n**是数组的大小
* 对所有`id[i] == id[q]`，修改`id[i] = id[p]`，由于修改次数不定，其修改数组开销也不定

综上，连接开销至少是O(n)级别的。

#### quick-union

quick-union同样采用了id记录的方法，但有明确的抽象结构，这里我们换一种形式来表达。

![](http://p4mu7u37x.bkt.clouddn.com/quick-union-aid.jpg)

如上图所示，我们首先增加一个**root**状态，最开始的时候3、4、8、9的root为它本身，四者相互独立。在经过第一次的4、3连接后，我们让3连向4，这时修改3的root为4。之后连接3、8，由于3的root是4，8的root为它本身，所以判断两者不连通，将8连向3的root，也就是4。再连接9、4，过程同上。最后连接8、9时，8的root为9，9的root为它自身，所以判断两者连通，跳过此连接。最终形成的抽象结构就像树一样。

那么我们如何求得root呢？这时就要通过quick-find中用到的id了，可以把求root的过程看成一个上溯的过程。对最后连接8、9前的状态来说，我们令`id[8] = 4, id[3] = 4, id[4] = 9, id[9] = 9`，上溯8的root时不断判断`id[i] == i`，一旦返回True就说明上溯到了源头，这时候的`i`就是8的root了。

id的修改如下：

![](http://p4mu7u37x.bkt.clouddn.com/quick-union.jpg)

简单吧？其实就是在两者的root不相等时让后者的root连向前者的root就可以了。

整理一下步骤：

* 连接*p*,*q*时，各自上溯root
* 判断`root[p] == root[q]`，如果两者不等认为两者不连通，修改`root[root[q]] = root[p]`

接着计算开销：

* 上溯`root[p]`，最好的情况下上溯一次（返回*p*自身），最坏的情况**n**次，这里**n**是数组的大小
* 上溯`root[q]`，开销与`root[p]`类似
* 判断`root[p] == root[q]`，返回`False`则修改`id[root[q]]`，最多有修改开销一次


显然quick-union算法的开销取决于树的高度，而这个高度在算法中并不能保证。综上，quick-union的开销不定，最好的情况下复杂度为O(1)，最坏情况下为O(n)。

#### weighted-quick-union

如果要对上述quick-union的这棵树做什么改进的话，那显然是这样的：

![](http://p4mu7u37x.bkt.clouddn.com/weight-quick-union-aid.jpg)

在连接9、4时不是另`root[4]`指向`root[9]`，而是另`root[9]`指向`root[4]`，树的高度就可以降低，连接所需的开销也得以减少，这种进阶的quick-union被称为weighted-quick-union。

具体做法就是新建一个size数组，初始化为1。在连接*pq*的过程中，假设*p*、*q*分属于两棵树，`root[p] = i, root[q] = j`，`size[i], size[j]`分别记录了两棵树的大小（这里记录高度也是可以的），比较`size[i], size[j]`就可以自己决定合并的顺序到底是`i->j`抑或`j->i`。不妨设`size[i] <= size[j]`，这时除了另`id[i] = j`外，还需要让`size[j] += size[i]`，意思就是两棵树合并的时候这棵树的root所在的size记录了这棵树的大小。

可以证明weight-quick-union方法构造的树的高度不会超过**log(n)**，具体证明可以自己去看这课。

计算开销：

- 上溯`root[p]`，最多上溯log(n)次
- 上溯`root[q]`，开销与`root[p]`类似
- 判断`root[p] == root[q]`，返回`False`则有修改一次id的开销与一次size的开销

综上，weighted-quick-union算法的复杂度为O(log(n))级别。

### 题目概述

讲完了算法来看题目，整个题目的完整文件在[这里](http://coursera.cs.princeton.edu/algs4/assignments/percolation.html)

第一次看的时候还是很懵逼的，放个gif感受下

![](http://p4mu7u37x.bkt.clouddn.com/题目概述.gif)



emmm...

![欲言又止](http://p4mu7u37x.bkt.clouddn.com/欲言又止.jpg)

稍微总结一下题目的意思

如下图所示是一个**8*8**方格的渗透模型，可以想象成像素游戏（比如泰拉瑞拉）中占了64个体积的土块，其中白色和蓝色的方块表示已经被挖空了。现在我们从上方倒足量的水，这时蓝色的即表示水在往下走时会经过的方块，白色表示虽然挖空但是不会有水经过的方块。我们要求的就是在一个对全实心土块**随机**挖方块的过程中，经过**多少次**挖取之后，水可以顺利地从土块中流出。

![渗透模型](http://p4mu7u37x.bkt.clouddn.com/渗透模型.png)

根据上述的题目要求，可以认为土块有两种状态：闭合及连通。乍看之下从闭合到连通需要的挖取次数似乎是不定的，不过大量的计算告诉我们这其中存在一个阈值*p*(~= 0.6 \* 方块数量)，当方块数量小于*p*时，土块不能连通，而一旦到达*p*值附近，则不论挖取过程如何，土块基本都能连通。所以，最后的任务即是写一个程序以期求出这个阈值*p*。

![阈值](http://p4mu7u37x.bkt.clouddn.com/阈值.png)

### Percolation类的构造

因为之后要将代码提交给网站打分，所以这里一个Percolation(渗透)类的接口已经给你定死了（但是还是可以写private方法），要做的就是一个个填充。

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

题目本身和最开始讲的union-find问题非常相似，一个土块有64个方块，一开始的时候相互不连通，之后在挖取方块的过程中，如果被挖取的方块周围有其它已经为空的方块，则将该方块与空方块连在一起，对应于Union-find类里的union方法。所以Percolation类中首先可以确定一个大小为64的Weighted-Quick-Union-Find类实例的变量。由于每个方块有空和非空两种状态，可以定义一个大小为64的数组存放状态。由于最后要求打开的方块数量，再定义一个记录打开方块数量的变量。本程序的主要难点在于如何定义土块的连通这一状态，这里暂时采取在上下方各加一个状态为空的虚拟方块的做法，上方的虚拟方块被设定为与第一排所有的方块相邻，类似的，下方的虚拟方块被设定为与最后一排所有的方块相邻，这样，土块是否连通转换为这两个虚拟方块是否连通的问题。

```java
/******************************************************************************
 *  Author:         suyako
 *  Written:        9/22/2017
 *  Last updated:   2/13/2018
 *
 *  Compilation:    javac Percolation.java
 *  Execution:      java Percolation
 *  Dependencies:   WeightedQuickUnionUF.java
 *
 *  Build a percolation class
 *
 *  % java Percolation
 *  Build a 10-by-10 Percolation class, and open site(1, 5), site(2, 6), then
 *  return the number of open sites.
 ******************************************************************************/

import java.lang.IllegalArgumentException;

import edu.princeton.cs.algs4.StdOut;
import edu.princeton.cs.algs4.WeightedQuickUnionUF;

public class Percolation {

    private WeightedQuickUnionUF UF;  //  weighted quick-union find data structure
    private int opened;               //  number of opened sites
    private int size;                 //  size of grid
    private int[] condition;          //  matrix store the status of all sites, where 0 is block, 1 is open

    /**
     * Initializes an percolation model which has {@code N}-by-{@code N} blocked grid and one opened virtual site
     * @param N the number of sites
     * @throws IllegalArgumentException if {@code N} <= 0
     */
    public Percolation(int N) {
        if (N <= 0)
            throw new IllegalArgumentException("N should be more than 0.");  // at least one site required
        UF = new WeightedQuickUnionUF(N*N+2);  // additional two blocks represent virtual site at top and at bottom
        condition = new int[N*N+2];
        opened = 0;
        size = N;
        for (int i = 1; i <= N; i++) {
            for (int j = 1; j <= N; j++)
                condition[xyTo1D(i, j)] = 0;  // any site in grid is closed
        }
        condition[N*N] = 1;  //  the virtual site is opened
        condition[N*N+1] = 1;
    }

    // expand to one-dimensional vector
    private int xyTo1D(int i, int j) {
        if (i < 1 || i > size || j < 1 || j > size)
            throw new IllegalArgumentException("Argument out bound.");
        return (i - 1) * size + (j - 1);
    }

    // find the site near by the target site
    private int[] nearby(int i, int j) {
        int[] res = new int[4];

        // res[0] is the left of target site, duplicate itself if it doesn't exist
        if (j > 1)
            res[0] = xyTo1D(i, j-1);
        else
            res[0] = xyTo1D(i, j);

        // res[1] is the right of target site, duplicate itself if it doesn't exist
        if (j < size)
            res[1] = xyTo1D(i, j+1);
        else
            res[1] = xyTo1D(i, j);

        // res[2] is the top of target site, duplicate itself if it doesn't exist
        if (i > 1)
            res[2] = xyTo1D(i-1, j);
        else
            res[2] = size * size;

        // res[3] is the bottom of target site, duplicate itself if it doesn't exist
        if (i < size)
            res[3] = xyTo1D(i+1, j);
        else
            res[3] = size * size + 1;
        return res;
    }

    // open site (row, col) if it is not opened
    public void open(int i, int j) {
        if (isOpen(i, j)) return;
        int location = xyTo1D(i, j);
        condition[location] = 1;
        opened++;
        int[] nearby = nearby(i, j);
        for (int k : nearby) {
            if (condition[k]==0) continue;                 //  if nearby site is closed, continue
            else if (UF.connected(location, k)) continue;  //  if has connected, continue
            else UF.union(location, k);                    //  if nearby site is opened, then connect them
        }
    }

    // is site (row, col) full?
    public boolean isFull(int i, int j) {
        return UF.connected(xyTo1D(i, j), size*size);  //full site is an open site that can be connected to the top virtual site
    }

    // is site (row, col) open?
    public boolean isOpen(int i, int j) {
        int location = xyTo1D(i, j);
        return condition[location]==1;
    }

    // does the system percolate?
    public boolean percolates() {
        return UF.connected(size*size, size*size+1)
    }

    // number of open sites
    public int numberOfOpenSites() {
        return opened;
    }

    public static void main(String[] args) {
        Percolation perc = new Percolation(10);
        perc.open(1, 5);
        perc.open(2, 6);
        StdOut.println(perc.numberOfOpenSites());
    }
}
```

### PercolationStats类的构造

这个没啥好说的，就是对Percolation类进行测试，接口也已经给了，这里就直接放代码了。

```java
/******************************************************************************
 *  Author:         suyako
 *  Written:        9/22/2017
 *  Last updated:   2/13/2018
 *
 *  Compilation:    javac PercolationStats.java
 *  Execution:      java PercolationStats
 *  Dependencies:   StdOut.java StdRandom.java
 *
 *  Build a series of percolation calculation experiments
 *
 *  % java PercolationStats
 *  Build 100 200-by-200 Percolation models, print the average percolation
 *  threshold, the standard deviation of percolation threshold, low endpoint,
 *  high endpoint.
 ******************************************************************************/

import edu.princeton.cs.algs4.StdOut;
import edu.princeton.cs.algs4.StdRandom;
import edu.princeton.cs.algs4.StdStats;


public class PercolationStats {

//    private Percolation perc;  // percolation model
    private double[] Times;    // matrix to store the outcome of each experiment
    private int size;          // number of trials
    private double mean;
    private double dev;

    /**
     * Initializes a series of computational percolation experiments
     * @param n size of percolation model
     * @param trials number of trials
     * @throws IllegalArgumentException if {@code n} or {@code trials} less than or equal to 0
     */
    public PercolationStats(int n, int trials) {
        if (n <= 0 || trials <= 0)
            throw new IllegalArgumentException("Row index n or trials less than or equal to 0.");
        size = trials;
        Times = new double[trials];

        for (int i = 0; i < trials; i++) {
            Percolation perc = new Percolation(n);  // create a new Percolation object each trial
            // open site randomly if perc is not percolated
            while (!perc.percolates()) {
                int row = StdRandom.uniform(n) + 1;
                int col = StdRandom.uniform(n) + 1;
                if (perc.isOpen(row, col))
                    continue;
                else
                    perc.open(row, col);
            }
            double temp = perc.numberOfOpenSites();
            Times[i] = temp / (n * n);
        }
        mean = StdStats.mean(Times);
        dev = StdStats.stddev(Times);
    }

    // calculate the average
    public double mean() {
        return mean;
    }

    // calculate the standard deviation
    public double stddev() {
        return dev;
    }

    // low endpoint of 95% confidence interval
    public double confidenceLo() {
        return mean - 1.96 * dev / Math.sqrt(size);
    }

    // high endpoint of 95% confidence interval
    public double confidenceHi() {
        return mean + 1.96 * dev / Math.sqrt(size);
    }

    public static void main(String[] args) {
        PercolationStats percstate = new PercolationStats(200, 100);
        StdOut.println("mean                   " + " = "+ percstate.mean());
        StdOut.println("stddev                 " + " = " + percstate.stddev());
        StdOut.println("95% confidence interval" + " = " + "[" + percstate.confidenceLo() + ", " + percstate.confidenceHi() + "]");
    }
}
```

### 测试

在上传到网站之前先进行一些本地测试

#### *p*

对**200*200**的土块进行约100次测试取平均值，得出的*p*值结果为0.592，标准差为0.01，与课程给的结果还是挺接近的。

![结果对比](http://p4mu7u37x.bkt.clouddn.com/结果对比.png)

#### 过程可视化

课程本身给了一些测试样本和可视化测试程序，我们来看一下输出动画。

![percolation可视化](http://p4mu7u37x.bkt.clouddn.com/可视化.gif)

emmm...

大问题是没有，但是左下角是什么鬼？？？

![](http://p4mu7u37x.bkt.clouddn.com/左下角.png)

问题还是出在percolates()和isFull()方法的编写上，程序中蓝色方块是由isFull方法决定的，而isFull的判定如下：

```java
public boolean isFull(int i, int j) {
    return UF.connected(xyTo1D(i, j), size*size);  //full site is an open site that can be connected to the top virtual site
}

public boolean percolates() {
    return UF.connected(size*size, size*size+1);
}
```

左下角的方块虽然并没有与最上方的虚拟方块直接连通，但是在土块连通后由于最下方虚拟方块的关系，左下角是和最上方虚拟方块间接连通的，这才导致它变成蓝色。

虽然这个并不影响最终的*p*值，但课程测试还是将其作为一个bug来处理的（题干中将这个现象称为**backwash**，实际上这也是这道题最难的地方），我们只能另找办法来解决这个问题。

最直接的一个思路就是去掉最下方的虚拟方块，这样保证了isFull的正确性，至于连通判定改为对最后一行方块进行循环判定其与最上方的虚拟方块是否连接。写了一遍上传后发现通不过Timing测试（对**8\*8**土块来说，原来的判定一次的时间变成了判定八次），想了很久才想到可以写两个UF的方法，第一个UF中没有最下方的虚拟方块，用于对isFull的判定，第二个UF中存在最下方的虚拟方块，用于对percolates的判定，修改代码如下：

```java
private int[] nearby(int flag, int i, int j) {
    int[] res = new int[4];
    // res[0] is the left of target site, duplicate itself if it doesn't exist
    if (j > 1)
        res[0] = xyTo1D(i, j-1);
    else
        res[0] = xyTo1D(i, j);
    // res[1] is the right of target site, duplicate itself if it doesn't exist
    if (j < size)
        res[1] = xyTo1D(i, j+1);
    else
        res[1] = xyTo1D(i, j);
    // res[2] is the top of target site, duplicate itself if it doesn't exist
    if (i > 1)
        res[2] = xyTo1D(i-1, j);
    else
        res[2] = size * size;
    // res[3] is the bottom of target site, duplicate itself if it doesn't exist
    if (i < size)
        res[3] = xyTo1D(i+1, j);
    else {
        if (flag == 1)       // UF1
            res[3] = xyTo1D(i, j);
        else if (flag == 2)  // UF2
            res[3] = size * size + 1;
    }
    return res;
}

public void open(int i, int j) {
    if (isOpen(i, j)) return;
    int location = xyTo1D(i, j);
    condition[location] = 1;
    opened++;
    int[] nearby1 = nearby(1, i, j);
    int[] nearby2 = nearby(2, i, j);
    for (int k : nearby1) {
        if (condition[k]==0) continue;                  //  if nearby site is closed, continue
        else if (UF1.connected(location, k)) continue;  //  if has connected, continue
        else UF1.union(location, k);                    //  if nearby site is opened, then connect them
    }
    for (int k : nearby2) {
        if (condition[k]==0) continue;
        else if (UF2.connected(location, k)) continue;
        else UF2.union(location, k);
    }
}

public boolean isFull(int i, int j) {
    return UF1.connected(xyTo1D(i, j), size*size);  //full site is an open site that can be connected to the top virtual site
}

public boolean percolates() {
    return UF2.connected(size*size, size*size+1);
}
```

最后的输出动画如下：

![](http://p4mu7u37x.bkt.clouddn.com/修改后可视化.gif)

#### 课程测试

暂时好像找不到其它问题了，来试试代码上传。

![](http://p4mu7u37x.bkt.clouddn.com/课程测试.png)

​

主要的扣分项还是在内存方面，因为用了两个UF的原因导致大了一倍的内存，不过暂时也想不到其它方法啦，如果有哪位大佬有一个UF就能解决问题的方法还望告知。

### 总结

这道题是我去年9月做的，当时没有解决backwash的问题，这两天翻出来又做了一遍，同时稍微规范了下自己写的java代码（一开始的nearby函数写得很丑陋...），当然我不太会java，所以可能还是有很多代码写得不好的地方。这道题作为算法的第一课教会了我一些东西，一个是java初步；一个是规范的接口设计要尽可能简洁，一个函数只做一件事；最后是算法与数据结构往往存在着千丝万缕的关系，选取合适的数据结构至关重要。

最后再说一下，希望有相同转行想法的小伙伴能联系我一起交流，微信ID：BEAFINEGIRLKISSME。然后，coursera algorithm这个系列希望能做到周更吧：）