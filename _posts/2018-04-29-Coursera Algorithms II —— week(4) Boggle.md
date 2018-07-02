---
layout: post
title: "Coursera Algorithms II —— week(4) Boggle"
category: Algorithms
---

hi~~假期愉快

## Assignment

> 本周作业地址：
>
> http://coursera.cs.princeton.edu/algs4/assignments/boggle.html
>
> 现在你有一本字典，一个4x4的字母板，请你给出这块字母板上所有在字典中的单词，单词的构造需遵循如下几个条件（可以直接看图，会更直观些）：
>
> 1 每个单词上的相邻字母在板上也需要相邻（上下左右或斜对角均可）；
>
> 2 板上构成一个字母的路径不能有交叉；
>
> 3 一个单词至少需要有3个字母组成；
>
> 4 最后，单词必须在该字典中；
>
> ![52493442428](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfYItibuS0vK7G0UlXa1MziaDqsNNSCaSmVIPbG2AcvzS1PURsp1J2VGMV6iaVjQCgPebXQ6ZYhFXHu8g/0?wx_fmt=png)

## Introduction

看到题目的时候很开心，因为好久没做到意思这么简单明确的作业了，原先我题目都能读半天。

本周的主题是字符串，作业的内容实际上就是我们日常可见的文档内字符串搜索。在接下去的阅读中，你会发现即使如此简单的一个任务，在文档容量变得很大时，也需要用到简单而精巧的算法来保证实际的搜索效率。

我们首先给几个单词作为我们的字典内容，接下来我们需要思考一种数据结构来存储这个字典。

![52497812574](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfYItibuS0vK7G0UlXa1MziaDqiay5QDzZAKXpmyZzjkHCn1Ygl5JJafxqqwlnp5maAHYHJGjaHlTYVtA/0?wx_fmt=png)

说到搜索，首先会想到的就是之前说过的红黑树。作为平衡树的典范，它可以保证`log(N)^2`的搜索效率（这里N是单词数量，为什么有平方呢？因为每次比较还需要一一对比字符串里的每个字母啊，所以肯定要再乘一个系数，虽然我也不太明白为什么乘了log(N)），而我们仅仅只需要定义单词之间的大小关系即可。当然，哈希表会更快，但哈希表会缺失很多信息（比如，我们可能不单单想要精确搜索。举个栗子，我们可能会想知道字典中所有以*pre*开头的单词，哈希表显然无法满足我们的需求，当然了，红黑树应该也不行，或者说写起来很麻烦）。实际上，因为字符串的特殊性，我们完全可以不通过传统的比大小方式来实现搜索功能，甚至构造上也会更快。

为了简化情况，我们只考虑英文单词。由于单词的每个位置只可能有26种情况，所以我们完全可以构造一个如下图所示的26向单词查找树（26-Tries，tries从retrieval变换过来，大意就是取回的意思）：

![52497080659](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfYItibuS0vK7G0UlXa1MziaDqTW8Xm40U3PuojYS2NtdNtb4BjfJBOqO6KTdvG81ntHbW21utp0fFXA/0?wx_fmt=png)

稍微解释一下，首先有一个root节点，root节点下初始时连接26个空节点序列`next`，如果字典中有*a*开头的字母，则`next[0] = a`（在之后的解释中为了方便后文中遇到类似情况时我会写成`next[a] = a`），依此类推。举个栗子的话，对于*she*来说，如果`root.next[s] = null`，则首先另`root.next[s] == s`，同时创建关于*s*的`next`。如果已有`root.next[s] == s`的话，则不做任何改变。然后另`s.next[h] = h`，`h.next[e] = e`，最后在*e*处存一个与*she*关联的值即可（这个值只是用来表明从*e*处回溯一直到root的路径的逆能构成字典里的一个单词，图中用插入单词的顺序表示，如果没特殊要求的话完全可以用布尔值表示）。

单词查找树的搜索效率非常高，我们可以很轻易地证明其搜索效率仅与单词的长度*L*有关。但是其内存开销显然也很大，令单词数量为*N*，所有单词到root的路径长度的平均为*w*（意思就是最多总共有*Nw*个节点，但实际上不会有那么多的，因为肯定会有单词路径重复），那么总的连接数量就处在*26N~26Nw*之间。

为了避免26向单词查找树造成的过多空间消耗，我们可以使用一种结合了比大小及单词查找树两种思想的三向查找树（TST，Ternary Search Tree，其中Ternary是人名）：

![52497576928](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfYItibuS0vK7G0UlXa1MziaDqSXePKWmWPwzjADsf9ucTpticEz3jyWN6j2VjW2eO2V6lwj8X1PO0YeA/0?wx_fmt=png)

上图中左边是Tries，右边是对应的TST，这也也稍微简单解释一下。

首先TST没有root，这棵树的第一个节点就是*s*。*s*节点也有`next`数组，但`next`的长度只有3，我们对这3个children分别称为*left, mid, right*。参考红黑树的表示方法，*left*与*right*的连接用红色表示，意为这两个节点与*s*平级，*mid*的连接则用黑色表示，意为这个节点是*s*的下一级。

说完上面的表示方法，相信各位也大致看懂了，就是在原先Tries的基础上加上红线嘛。原先的Tries中，*a、b、s、t*是平级的，现在它们之间用红线连接了起来，*e、h、u*对于*s*来说是平级的，也用红线连起来。由于现在`next`数组只有3个元素了，所以总的连接数量处于*3N~3Nw*之间（采用与Tries中相同的符号表示，这里*w*会增大，见后文分析）。相比于Tries，TST搜索到一个单词所走的路径显然要增大，我们可以把路径分为红和黑两部分，黑的部分显然就是单词长度L，对于红的部分，我们可以认为单词上每个位置的字母都有一个平级搜索过程，该搜索过程的最坏情况是搜索26次（比如从Z一直平级搜索到A），最好情况就是正好对上，平均意义上可以认为是一个二分搜索，即需要*lg(26)*次，所以最终的路径长度可以认为是**Llg(26)**。

## Programming

有了上面的数据结构的一些知识就可以做这道题啦。

首先因为内存足够所以这里还是选用Tries。然后对于这道题来说，最大的难点在于在板上构造单词路径，很容易会想到图论中的BFS与DFS。BFS应该比较难实现（因为实际上每条路径都需要构造一个矩阵来存储该路径上的标记点），而DFS相对来说应该容易一些，路径回退的时候我们只需要重新把标记矩阵上的True改为False就可以了。

API：

```java
public class BoggleSolver
{
    // Initializes the data structure using the given array of strings as the dictionary.
    // (You can assume each word in the dictionary contains only the uppercase letters A through Z.)
    public BoggleSolver(String[] dictionary)

    // Returns the set of all valid words in the given Boggle board, as an Iterable.
    public Iterable<String> getAllValidWords(BoggleBoard board)

    // Returns the score of the given word if it is in the dictionary, zero otherwise.
    // (You can assume the word contains only the uppercase letters A through Z.)
    public int scoreOf(String word)
}
```

Tries的代码我就不放了，基本跟书上一致，但为了这道题做了一些小改变（踩了不少坑），我只放BoggleSolver中的getAllValidWords方法的代码：

```java
public class BoggleSolver {
    private myTrieST dict;      // Trie to store dictionary
    private boolean[] marked;   // if the alphabetas are marked in one search path
}
```

```java
public Iterable<String> getAllValidWords(BoggleBoard board) {
    SET<String> res = new SET<>();  // set to store all valid words
    int rows = board.rows();
    int cols = board.cols();
    marked = new boolean[rows * cols];  // initialize the marked
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            marked[i * cols + j] = true;
            char c = board.getLetter(i, j);
            StringBuffer begin = new StringBuffer("" + board.getLetter(i, j));
            depth_search(i, j, board, begin, res);  // see that in my github
        }
    }
    return res;
}
```

关键就是代码里的depth_search，有兴趣的可以看下我的GitHub（https://github.com/suyako-to-be-a-programmer/Algorithms-part2/tree/master/boggle）~~

## Summary

这道题大概是到现在为止唯一一道在3小时内做完然后拿到及格分的题（不过代码开始写得越来越随意了），但在最后的timing测试上我有一个一直过不去，而且效率差别相当大。我花了挺大力气将题目建议的优化方法基本实现了一遍（像是用循环代替递归，解决prefix等）但还是没有太大的起色，不过也实在没有时间再去查问题在哪里了（还是能力有限吧）。所以这个时候也挺想找个人一起做的，这样至少还能探讨一下(´；ω；`)

字符串虽然在日常中一直有用到而且很直观，但其中用到的算法并没有想象中那么简单，比如寻找子字符串用到的KMP算法几乎可以说是迄今为止碰到的最抽象的算法了，还有就是爬虫中经常用到的正则表达式。不过算法系列已经到倒数第二期啦，所以这些在本系列中应该是没有机会写了。下一期应该也会准时的，希望能以一个好的结尾结束吧~