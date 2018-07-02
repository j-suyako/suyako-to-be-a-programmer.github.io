---
layout: post
title: "Coursera Algorithms —— week(5) kd tree"
category: Algorithms
---

![52190743989](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbc9ictBsVm3KHccyr9pgCsZZIQiaYxq0ZvQgevgELDia9gJiaGD0fH2I5O8icVAKHz2yiat5VpricdUlXVw/0?wx_fmt=png)

纪念一下第一阶段结题。

这一周东西比较多，很多细节就跳过了，不过我写下来主要还是给自己一个输出的过程，防止*自以为自己懂却不能很好地描述出来*。不过kdtree在几何里面应该经常会用到，最常见的用途就是用来搜索距离给定点的最近点，有兴趣的可以百度一下。

## Assignment

> 本周作业地址：
>
> http://coursera.cs.princeton.edu/algs4/assignments/kdtree.html
>
> 在一个单位正方形内有数量不等，随机生成的点集合*points*，现要求用红黑树及kd树分别实现以下两种功能：
>
> 1. range search方法，给出一个矩形*rec*，寻找*points*中所有在*rec*中的点（包含边界）；
> 2. nearest neighbor方法，给出一个点*q*，寻找*points*中距点*q*最近的点；
>
> ![](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbc9ictBsVm3KHccyr9pgCsZneibbaNiavp0Dc7NpAT20OKlhx3CYibk5bVtZm0uZhxLBApmNxchcIscA/0?wx_fmt=png)

## Introduction

暴力循环是不可能暴力循环的，这辈子都不会暴力循环的。最基础的想法就是将单位正方形划分为一个个小格子，这样最后只需要根据*rec*和*q*的情况对特定的几个小格子进行分析即可。这种方法被称为*grid implementation*，具体实现如下图所示：

![52187453770](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbc9ictBsVm3KHccyr9pgCsZicaKqsu5HBBZibhSh3HEDGUV86fYRrYOmnLa0fuVZ7Fzl1Wskr0mt4Ng/0?wx_fmt=png)

*grid implementation*的主要问题在于如果点的分布恰好都集中在有限的几个小格子中，那其效率与暴力搜索其实并没有什么两样（不要以为这种情况极端，在实际问题中非常常见）；其次*grid implementation*对于*nearest neighbors*方法支持并不怎么好，不是说不能写，就是写起来真得非常笨重；另外这里虽然把*range search*及*nearest neighbor*方法具体化了，但实际上还有很多抽象的运用，这些抽象的应用并不容易弄成grid这种形式。

在这里我们降维打击一下，假设现在有一条绳子，绳子上随机分布着一些点，要求同样实现*range search*及*nearest neighbor*方法，这时候应该怎么做呢？

这个时候情况就很简单了，对所有点进行排序然后用二分法不断迭代搜索就可以了。

由一维情况提供的思路，很容易想到面对二维点时，我们可以先根据点的x坐标进行排序，之后不断二分（当然会有一些细节要处理，这些等会儿会在代码中体现），也必然能像一维一样实现类似效果，而且这种方法并不受点的分布情况的影响（这里说下，这种方法实际上是我脑补的，我实际上是从kdtree反推回来觉得可行的）。

这里面还有一个问题就是在实际情况中，我们的点可能需要不断增加或者删除。如果是要维持一个有序序列，增加或者删除操作相当耗时。因为这个原因，专门用来解决这个问题的，从二叉树演化过来的红黑树就应运而生了。

经典的kd树方法具体展示如下方gif：

![](https://mmbiz.qpic.cn/mmbiz_gif/ickltPAryrfbc9ictBsVm3KHccyr9pgCsZiaLCqAULjibmmMic3vLpDMylc9SzJgGibdN7Ha8nlN1Ad2Ru0KAuZGFTaQ/0?wx_fmt=gif)

下面我分4步来说明：

1. 插入第一个点*p1*后，对单位正方形关于*p1.x*划分成*rec1*与*rec2*，划分的线用红色表示；
2. 对第二个点*p2*，将其与*p1*关于x值进行比较，若`p2.x < p1.x`，则对*rec1*关于*p2.y*进行划分，反之对*rec2*进行划分，划分线用蓝色表示；
3. 对第三个点*p3*，首先与*p1*关于x值进行比较，若在*p1*左边则再与*p2*关于y值进行比较，放入相应的*rec*并进行划分；若在*p1*右边则直接关于*rec2*进行划分；
4. 其余点进行同样操作；

如果用树结构来表示上述10个点的话应如下图所示（这里顺序和我自己画的略有不同......不过意思是一样的）：

![52196116643](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbc9ictBsVm3KHccyr9pgCsZcnDicwx2aW8miaKfCIEmoePOdHMbk9yAJqYjr4dK7tlaVVZdCrmNeEZg/0?wx_fmt=png)

所以说kd树只是一种变相的二分法，只是它二分的顺序是先x后y，而且它是根据插入顺序二分的。至于为什么第二个点要关于y进行划分，我个人觉得是为了防止有x相同的两个点的情况，因为所有点必须对应一条线。如果真的是那种完全随机生成几乎不可能有重复x情况的点，我觉得是无所谓的（不知道我理解对不对）。

kd树的思想借鉴于二叉树，所以接下来会讲一下二叉树及其演化红黑树。

**红黑树**是一种**平衡二叉树**，所以需要先从**二叉树**讲起，二叉树的抽象形式如下图所示：

![52187555514](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbc9ictBsVm3KHccyr9pgCsZmVeR5e0vYK7fbOWwRaayzskepXaxNyukhB04DEY63ubkGErIptNe8w/0?wx_fmt=png)

二叉树是由链表实现的（数组也可以，但会有很多浪费）。这里我们把母节点称为*root*，每个根节点称为*node*，每个*node*存放一个*value*，一个*node*既可以为*parent node*，也可以为*children node*。每个*parent node*下面又分为*left children node*与*right children node*。一个无重复节点的二叉树应具有如下几个性质：

1. `parent_node.value > left_children_node.value`；
2. `parent_node.value < right_children_node.value`;

实际上大部分数据的定义都很简单......总之只要一个数据结构满足上述两个性质，我们就可以称其为无重复节点的二叉树（这里加上无重复节点只是为了方便讨论，有重复应该也是可以的）。

对于一个现有的二叉树，记其最大高度为*h*，我们可以很轻易地实现插入以及查找节点等功能，删除节点会稍微麻烦点，看到的不妨思考下具体应该怎么做。这三个功能的时间复杂度均为*O(h)*。

除了上述三个基本功能外，还有一个很重要的，和今天作业息息相关的功能就是范围查找，记该功能为*range*。回到二叉树那张图，如果我们想要找出包含在**[F..T]**之中的所有节点，我们可以先构造一个队列*queue*，然后`range(S,F,T)`，具体实现就是比较*F、S*与*T、S*，由于`F < S && T > S`，所以在*queue*中新增节点*S*，接着对*S*节点下的左子节点和右子节点重复*range*（再举`range(E,F,T)`的例子，由于`F > E`，所以*queue*不做变化，同时只`range(R,F,T)`），一旦碰到空的子节点就停止迭代。

但二叉树有一个很明显的缺点，就是关于树的高度*h*我们并不能保证。最坏的情况，也就是顺序输入的情况下，*h*就是输入的长度*n*，最好的情况下，也就是左右子节点全部填充的情况下，才能达到*lg(n)*，因此为了能够形成一个完美平衡树，二叉树不断演化形成了多种平衡树结构，其中应用最广泛的就是**红黑树**。

（红黑树不太好讲，我这里尽量写一下。）

了解**红黑树**要从先2-3树开始说起......（我也很绝望）。2-3树与二叉树形式上是类似，二叉树的两个性质2-3树同样满足，不同的地方主要在于：

1. 二叉树里每个*node*只存放一个*value*，2-3树的每个*node*可以存放一至两个*value*，当一个*parent node*存放两个*value*时（我们记小的为*value1*，大的为*value2*），还会新增一个`middle children node`，具体关系如下：

   * `parent_node.value1 > left_children_node.values`；
   * `parent_node.value2 < right_children_node.values`;
   * `parent_node.value1 < middle_children_node.values < parent_node.value2`;

2. 2-3树高度的增加并不是像二叉树那样自顶向下的，而是自底向上的。当需要插入某个*value*时，该*value*首先会如二叉树那样沉淀到某一个*node*中，若该*node*之前只存放了一个*value*，则不修改树结构，若该*node*之前已存放了*value1*与*value2*，则需要在*value1、value2、value*中选取中间值回传到其*parent node*，记该方法为*bubble*。对*parent node*同样进行*bubble*......直到传回*root*，若*root*中之前已有两个*value*，则树结构的*root*更改为中间值，树高加1。具体如下图所示：

   ![52190283639](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbc9ictBsVm3KHccyr9pgCsZfeRb1H1a30iaUhdsXs45SUYpINYkl32IEXW0DCibtHTTI9lwv7mYmHHw/0?wx_fmt=png)

上述的*bubble*操作还有些细节要处理，比如*parent node*为一个*3-node*时子节点的处理，这里不赘述。2-3树是完美平衡的，但对其的很多操作并不容易通过程序实现，**红黑树**相当于一种弱化的但是更容易被支持的2-3树结构。

![52190327080](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbc9ictBsVm3KHccyr9pgCsZhKS95gbZYLic7SX64iaNWdY0xYjsuZnYyCLRztudvCr86XAV7ul4Viavg/0?wx_fmt=png)

基本一图顶千言了......如果把红黑树里红边连接的两个*node*认为处在同一个高度的话（实际上并不是，我们只是把这条边抽象地标红了而已），那它就是一个2-3树，具体的细节我实在写不动了，这里只说红黑树的三个性质：

1. 红链接均为左链接；
2. 没有任何一个节点同时和两条红链接相连；
3. **红黑树是完美黑色平衡的**；

最后放一个复杂度对比表：

| 数据结构 |  插入  |  查找  |  删除  |
| :------: | :----: | :----: | :----: |
| 有序数组 |  O(n)  |  O(1)  |  O(n)  |
|  二叉树  |  O(h)  |  O(h)  |  O(h)  |
|  红黑树  | O(lgn) | O(lgn) | O(lgn) |

## Programming

题目要求用红黑树和二叉树分别实现功能，且已预先给定了一个Point2D类以及RectHV（矩形）类，红黑树比较简单，我这里只写二叉树的代码。我们来看要提交的接口。

```java
// kdtree implementation
public class KdTree {
   public KdTree()                               // construct an empty set of points 
   public boolean isEmpty()                      // is the set empty? 
   public int size()                             // number of points in the set 
   public void insert(Point2D p)                 // add the point to the set (if it is not already in the set)
   public boolean contains(Point2D p)            // does the set contain point p? 
   public void draw()                            // draw all points to standard draw 
   public Iterable<Point2D> range(RectHV rect)   // all points that are inside the rectangle (or on the boundary) 
   public Point2D nearest(Point2D p)             // a nearest neighbor in the set to point p; null if the set is empty 

   public static void main(String[] args)        // unit testing of the methods (optional) 
}
```

上面的几个接口中我只放一些比较重要的。首先，原始的Point2D类给的api是不够的，需要在KdTree中新增一个Node类：

```java
private class Node implements Comparable<Node>{
    private Point2D point;  // corresponding point
    private Node left;      // left children node
    private Node right;     // right children node
    private RectHV rect;    // corresponding rect
    private int height;     // height of this node in kdtree

    public Node(Point2D p) {
        point = p;
        left = null;
        right = null;
        height = 0;
        rect = new RectHV(0, 0, 1, 1);
    }

    @Override
    public int compareTo(Node that) {
        if (that.height % 2 != 0)
            return Double.compare(this.point.x(), that.point.x());
        else
            return Double.compare(this.point.y(), that.point.y());
    }
}
```

上述代码中比较关键的是这个height，按照kd树的定义，对于偶数的height，两个节点我们比较y，对于奇数height，我们比较x。

然后是构造方法：

```java
public class KdTree {
    private Node root;  // root of this kdtree
    private int count;  // size of this kdtree

    public KdTree() {
        root = null;
        count = 0;
    }
}
```

Insert方法：

```java
public void insert(Point2D p) {
    if (p == null) throw new IllegalArgumentException();
    Node toInsert = new Node(p);
    root = insert(root, root, 0, toInsert);  // construct a private insert to recursive
}

/**
 * @param curr current node
 * @param pre previous node
 * @param flag vertical or horizontal
 * @param toInsert the node to be inserted
 * @return a new root that represent the whole kdtree
 */
private Node insert(Node curr, Node pre, int flag, Node toInsert) {
    ++toInsert.height;
    if (pre == null) {
        // if pre is null, means kdtree is empty, then root = toInsert
        ++count;
        return toInsert;
    }
    if (toInsert.point.equals(curr.point))
        // if toInsert has been in this kdtree, maintains root
        return curr;
    if (curr == null) {
        // if toInsert doesn't in this kdtree, apply it to its corresponding position and get its rect by spliting the rect of previous node
        ++count;
        if (toInsert.height % 2 != 0) {
            if (flag == 1) toInsert.rect = split(pre.rect, pre.point, 'h')[1];
            else toInsert.rect = split(pre.rect, pre.point, 'h')[0];
        } else {
            if (flag == 1) toInsert.rect = split(pre.rect, pre.point, 'v')[1];
            else toInsert.rect = split(pre.rect, pre.point, 'v')[0];
        }
        return toInsert;
    }
    if (toInsert.compareTo(curr) >= 0)
        curr.right = insert(curr.right, curr, 1, toInsert);
    else
        curr.left = insert(curr.left, curr, 0, toInsert);
    return curr;
}
```

其实写到这的时候我很怀疑自己的智商，因为我真的不太会递归......上面的insert方法肯定可以更精简，这里先这么放着了。其中代码里用到的split方法时用来切割对应的矩形的，这里就不贴了。

range方法：

```java
public Iterable<Point2D> range(RectHV rect) {
    if (rect == null) throw new IllegalArgumentException();
    Stack<Point2D> res = new Stack<>();
    range(root, res, rect);
    return res;
}

private void range(Node x, Stack<Point2D> res, RectHV rect) {
    if (x == null) return;
    boolean isIntersect = x.rect.intersects(rect);
    if (isIntersect) {
        if (rect.contains(x.point))
            res.push(x.point);
        range(x.left, res, rect);
        range(x.right, res, rect);
    }
}
```

nearest方法：

```java
public Point2D nearest(Point2D p) {
    if (p == null) throw new IllegalArgumentException();
    if (root == null) return  null;
    Node target = new Node(p);
    Node[] res = { root };  // use array to save nearest point
    double[] min_dis = { root.point.distanceSquaredTo(target.point)};  // use array to save minimum distance
    nearest(root, target, min_dis, res);
    return res[0].point;
}

private void nearest(Node x, Node target, double[] min_dis, Node[] res) {
    if (x == null) return;
    if (x.rect.distanceSquaredTo(target.point) < min_dis[0]) {
        double temp = x.point.distanceSquaredTo(target.point);
        if (temp < min_dis[0]) {
                min_dis[0] = temp;
                res[0] = x;
        }
        if (target.compareTo(x) < 0) {
        nearest(x.left, target, min_dis, res);
        nearest(x.right, target, min_dis, res);
        } else {
            nearest(x.right, target, min_dis, res);
            nearest(x.left, target, min_dis, res);
        }
    }
}
```

nearst里面一个比较关键的点在于求距离的时候用的是未开根号的数值（开不开根号速度会差很多），同时这里用了数组来保存当前搜索得到的最近的点，不过我也不确定有没有更好的方法。

## Summary

这周的递归写得我怀疑人生，虽然最后还是完成了不过总是迷迷糊糊的感觉，也算是一种训练吧。

coursera的算法课分上下两部分，第1部分说是有六周，但是最后一周是没有作业的。当然了我现在已经进入第2部分了，不过因为进度问题所以下周应该不会更新。下周应该会更一篇如何用markdown写微信公众号，希望对个人账号有用。