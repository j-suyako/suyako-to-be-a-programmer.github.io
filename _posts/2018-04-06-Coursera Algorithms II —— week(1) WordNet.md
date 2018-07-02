---
layout: post
title: "Coursera Algorithms II —— week(1) WordNet"
category: Algorithms
---

![](https://mmbiz.qpic.cn/mmbiz_gif/ickltPAryrfbhlWWtP2ibNickVKjtG8ics2dp5Ns8jBYBRliaYtxIOE8rQSHbPIURPV7ffxQjwgNGJ0nK6eje9QHqTQ/0?wx_fmt=gif)

中间隔了两周，算法课的第二部分终于开工了:smile_cat:

## Assignment

> 现给出两个txt文件，第一个文件列出了一定数量的同义词集及其对应的ID和解释，每一行的形式如下：
>
> ```
> 36,AND_circuit AND_gate,a circuit in a computer that fires only when all of its inputs fire
> ```
>
> 在上述语句中，AND_circuit和AND_gate构成一个同义词集，两个词用空格隔开。36是该同义词集的编号，最后的”a circuit ... fire“则是其对应的解释，三者之间用逗号隔开。
>
> 第二个txt文件则列出了不同同义词集之间的从属关系，每一行形式如下：
>
> ```
> 164,21012,56099
> ```
>
> 表示编号为164的同义词集（"Actifed"，安维汀，一种药物）从属于编号为21012（“antihistamine”，抗组胺药）与编号为56099（“nasal_decongestant”，鼻腔减充血剂）的同义词集。
>
> 现要求通过这两个文件构造一个同义词集网络，如下图所示，每个同义词集构成该网络的一个节点，节点之间的指向关系则代表同义词集之间的从属关系，如"miracle"从属于"event"与"happening...natural_event"。同时网络应实现如下接口：1. 给出任意两节点，求与这两个节点距离之和最小的共同父节点，如"jump leap"与"group action"的共同父节点应该是"event"； 2. 给出任意两节点，通过接口1得到的父节点可构造一条这两个节点之间的最短父路径（shortest ancestor path, SAP），求该路径的长度。
>
> ![](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfYOibaiaX7CCfu4HmfF1Gric0DMobcPqlYLGsSPiahMiaHcuz2DVPaLqC2QOZU8BczKz6jgzeJNib5pLt1A/0?wx_fmt=png)

## Introduction

图是一种非常直观的结构，就这道题来说，每个词集认为是一个节点，代表从属关系的箭头相当于边。在把每个同义词集转化为ID后，就可以构建一个数组`graph`，`graph[i]`用来存放第`i`个节点对应的边。由于`i`作为这条边第一个节点的编号已经确定，所以这条边可以直接用第二个节点的编号`j`表示，其中`j`是ID为`i`的同义词集所属的同义词集的编号。同时，因为一个词集可能从属于多个词集，所以`graph[i]`的类型应当为栈或者队列，即`graph[i] = Stack([j1, j2, ..., jn])`。

在图论中，寻找两点之间的路径是一个非常常用的功能（比如移动端的导航地图），对于本题，很直观的，通过计算两个同义词集之间的距离可以判断这两个同义词集的相关性。显然距离越近相关性越强。寻找两节点之间路径的方法主要有深度优先搜索（Depth First Search, DFS）与宽度优先搜索（Breadth First Search, BFS）。下面将逐一进行介绍。

<p>深度优先搜索</p>

给定两个节点`v`与`w`，拟搜索一条`v->w`的路径。一个很自然的想法是从`v`开始，然后搜索`v`的第一个临近节点`v0 = adj(v)[0]`，如果`v0 != w`，则继续搜索`v0`的临近节点`v1 = adj(v0)[0]`......同时，我们新建一个标记数组`marked`与距离数组`dis`，在搜索到某个节点`vt`时，若其在`marked`的对应位置为`false`，则将其改为`true`并在距离数组中的对应位置令`dis[t] = dis[t-1] + 1`；若其在`marked`的对应位置为`true`，则跳过该节点并搜索`adj(v(t-1))`的下一个节点（这里的t-1代表的意思是从属于ID为t的节点的ID，并不是实际的t-1）。具体实现过程可使用栈来实现：将所有`v`的临近节点压入一个栈，姑且称为`toRetrive`，然后弹出栈中最后被压入的一个节点，压入该节点的临近节点......直到搜索到`w`为止。

<p>宽度优先搜索</p>

搜索过程与DFS几乎是一致的，唯一不同的地方在于搜索的顺序，而且很有意思的地方在于导致这个搜索顺序不同的原因只是因为我们将用来存放临近节点的`toRetrive`的数据结构从栈换成了队列。在给定节点`v`后，与DFS一样，我们将所有`v`的临近节点`v1,v2,...,vn`存入`toRetrive`，同时该标记的标记，该记录距离的记录距离。紧接着，我们从`toRetrive`提取节点时，因为队列是先进先出的（FIFO），所以我们会在`v1,v2,...,vn`全部搜索完毕后才进入第二步的搜索，直到搜索到节点`w`为止。

显然，BFS得到的路径在不考虑路径权重的影响下是步数最少的，也是本题应当采用的方法。

对于本题来说还有几个特殊的地方：

1 路径是单向的，单纯地对WordNet使用BFS显然不能找到路径。解决方法应该有多种，我个人采用的是lockstep方法（即同步对`v`及`w`进行BFS），若某点在`v`与`w`的BFS搜索过程中均出现过，则显然该点是`v`与`w`的一个共同父节点；

2 一个同义词集可能会有好几个编号（），解决办法是一开始就在`toRetrive`中存入所有的`v`的编号；

3 虽然WordNet中不会出现环，但我们在构造SAP方法时其实也应该要考虑到环的存在的。环会造成一些问题，这里暂且不表；

## Programming

题目总共要求编写三个类，分别是WordNet，SAP，Outcast。WordNet和Outcast的主要作用就是调用SAP实现功能，所以最关键的还是SAP，我这里也只放SAP的关键代码。

首先来看看其提供的API。

```java
public class SAP {

   // constructor takes a digraph (not necessarily a DAG)
   public SAP(Digraph G)

   // length of shortest ancestral path between v and w; -1 if no such path
   public int length(int v, int w)

   // a common ancestor of v and w that participates in a shortest ancestral path; -1 if no such path
   public int ancestor(int v, int w)

   // length of shortest ancestral path between any vertex in v and any vertex in w; -1 if no such path
   public int length(Iterable<Integer> v, Iterable<Integer> w)

   // a common ancestor that participates in shortest ancestral path; -1 if no such path
   public int ancestor(Iterable<Integer> v, Iterable<Integer> w)

   // do unit testing of this class
   public static void main(String[] args)
}
```

length和ancestor因为上述提到的一个同义词集可能有多个从属词集的原因所以进行了重载，不过实际上因为都要存入队列里，基本没有什么区别。

首先定义SAP类的几个私有变量：

```java
public class SAP {
    private int V;                        // number of vertices
    private Digraph original;             // the digraph to construct SAP
    private int[] v_res;                  // the length of any vertices to v
    private int[] w_res;                  // the length of any vertices to w
    private boolean[] v_marked;           // is one vertice marked in BFS of v
    private boolean[] w_marked;           // is one vertice marked in BFS of w
}
```

因为用了lockstep的方法，所以这里定义了两个距离记录数组及标记数组。

然后写ancestor方法，length方法与ancestor方法类似，就不放了：

```java
public int ancestor(int v, int w) {
    if (!check(v) || !check(w)) throw new IllegalArgumentException();  // check corner case
    Queue<Integer> from_v = new Queue<>();
    from_v.enqueue(v);
    Queue<Integer> from_w = new Queue<>();
    from_w.enqueue(w);
    int ancestor = searchAncestor(from_v, from_w);  // private method to search the common ancestor
    return ancestor;
}
```

上述代码中的searchAncestor方法就是我们主要要实现的部分了，具体实现如下：

```java
private int searchAncestor(Queue<Integer> from_v, Queue<Integer> from_w) {
    int ancestor = -1;
    while (!from_v.isEmpty() || !from_w.isEmpty()) {
        if (!from_v.isEmpty())
            ancestor = myBFS(from_v, "v", ancestor);  // customize BFS method
        if (!from_w.isEmpty())
            ancestor = myBFS(from_w, "w", ancestor);
    }
    return ancestor;
}
```

上述代码中myBFS方法根据第二个参数是为代码复用准备的，实现如下：

```java
private int myBFS(Queue<Integer> from, String vORw, int ancestor) {
    boolean[] curr_marked; boolean[] oppo_marked;
    int[] curr_res; int[] oppo_res;
    if (vORw == "v") {
        curr_marked = v_marked; oppo_marked = w_marked;
        curr_res = v_res; oppo_res = w_res;
    } else {
        curr_marked = w_marked; oppo_marked = v_marked;
        curr_res = w_res; oppo_res = v_res;
    }
    int curr = from.dequeue();
    for (int e : original.adj(curr)) {
        if (!curr_marked[e]) {
            curr_res[e] = curr_res[curr] + 1;
            curr_marked[e] = true;
            from.enqueue(e);
        }
        if (oppo_marked[e]) {
            int min_dis = ancestor == -1 ? Integer.MAX_VALUE : curr_res[ancestor] + oppo_res[ancestor];
            ancestor = curr_res[e] + oppo_res[e] < min_dis ? e : ancestor;
        }
    }
    return ancestor;
}
```

## Improving

上面代码的主要开销在于每次调用SAP方法时都要重新初始化一遍，但事实上对于一个同义词集网络来说，由于该网络往往存在一个root（root从属于自身），所以像SAP中标记和距离数组的改动往往只有极小的一部分（对于拥有上万个节点及上万条边的同义词集网络，最后的改动数往往只有几十）。所以，如果能记录每次改动的数据，在下次计算时只重新初始化这些改动的部分，则效率将大大提高。

```java
public class SAP {
    ......  // as before
    private Queue<Integer> all_marked;  // recorde all marked vertices
}
```

```java
public int ancestor(int v, int w) {
    ......  // as before
    int ancestor = searchAncestor(from_v, from_w);
    reinitialize();  // add a new method that reinitialize all the variables of SAP
    return ancestor;
}
```

```java
private void reinitialize() {
    while (!all_marked.isEmpty()) {
        int e = all_marked.dequeue();
        v_res[e] = 0;
        w_res[e] = 0;
        v_marked[e] = false;
        w_marked[e] = false;
    }
}
```

除了上面的改动之外myBFS和searchAncesotor也需要一点小小的改动，这里不表。

## Summary

BFS和DFS的思想都很简单，但也是之后各类图论算法的基础。另外书上是通过递归来实现BFS的，不过我不太喜欢递归，所以这里就用栈来实现了，事实上应该任何递归都可以通过栈来实现，但是栈会多出额外的存储开销。

下周作业：图结构中边带权重时两点之间的最短路径，不过这几天事比较多，估计可能又得拖一周╮(￣▽￣)╭

