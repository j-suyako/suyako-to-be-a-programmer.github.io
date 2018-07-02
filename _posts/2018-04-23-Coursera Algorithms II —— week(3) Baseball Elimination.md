---
layout: post
title: "Coursera Algorithms II —— week(3) Baseball Elimination"
category: Algorithms
---

听说jazz hiphop与写代码更配哦::smile:（墙裂推荐西原健一郎）

## Assignment

> 本周作业地址：
>
> http://coursera.cs.princeton.edu/algs4/assignments/baseball.html
>
> 下图是美国棒球联盟东区（大概这么翻译？）1996年8月30日部分战队的战绩表，试证明Detroit队的确已经出局了（这里出局的意思是拿不到第一或并列第一）。另外请记住这里w、l、r、g等数组的含义，Introduction部分会用到。
>
> ![52432711708](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfYH0iaZKru8P7282yL8sMj7jTwxicsXibtt7yRWia8zkoMq7rM0a2QtXPoCyIMSAMYB8XFcjjwqP2TibTw/0?wx_fmt=png)

## Introduction

如果有关注NBA的话，想必一定会对前端时间西部最后一个比赛日的3-8名争夺战印象深刻。

![](https://mmbiz.qpic.cn/mmbiz_jpg/ickltPAryrfYH0iaZKru8P7282yL8sMj7jHlpsVFj3BibIZhR8Zkq7cNKoEKp7Xcia5m7PHkblC7svdEwHu09pmKlw/0?wx_fmt=jpeg)

实际上，除了这种特殊情况外，在很多情况下判断一个球队是否出局的困难度可能比我们想象得要复杂得多。比如对于assignment里的那张赛程图，为了确定Det是否出局，我们首先假设其在剩余的比赛中全胜，则其最终会获得76场胜利。此时NY最多只能赢一场胜利，假设其28场全输的话则Bos会获得77场胜利，Det出局，所以我们这里假设NY只赢了对Bos的一场球。此时Bos获得76场胜利，对Bal与Tor则全输，此时Bal已累计获得76场胜利，所以再假设Bal对Tor全输，然后我们就可以发现Tor获得77胜—>经过了如此长的步骤我们才能确定Det出局！

如果你对上面过程看得眼花的话，那么其实还有一种更简便的方法：抛开Det，剩余4个队已获得278场胜利，这四个队之间的比赛还有27（3 + 8 + 7 + 2 + 7）场，所以拢共会获得305场胜利。305/4=76.25 > 76，所以Det已经被确定淘汰了。

所以似乎只要把剩余几个队的已胜场和未胜场加起来除一下就好了？当然不是啦，不然这么简单的话也就不会特意拿出来当一期作业了\_(:з」∠)\_实际上如果我们把剩余4个队看作一个子集的话，这个子集才是我们要求的东西。如果不理解的话可以在上述赛程图中自行再加一个队，将原先的4个队与该新加的队重新构成一个子集，要另这个子集的已胜场和未胜场的平均小于76是很容易的事。

本周算法课就是讲述如何利用图结构和FordFulkerson(FF)算法解决这个求子集的问题。

首先，我们来看这么一个问题:

![52440909369](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfYH0iaZKru8P7282yL8sMj7j4815DTpM5RCmESrCppVggkZs7NTSzklefQbbEyiaJgtO6fh0AjqK2fQ/0?wx_fmt=png)

在上面这张流量图中，假设s是出水厂，t是家，其余的节点随意吧，当啥都行。如果节点与节点之间存在管道供水流通的话会用一条带箭头的边表示，同时，每条边还具有两个属性—>流量、容量。流量代表当前的水流通过量，容量代表该管道的最大水流通过量。问s->t的最大水流量是多少？

这道题挺难的，反正我自己想的话大概要想好久吧╮(￣▽￣)╭

FF算法就是用来专门解决这个问题的，对这道题来说FF的求解思路大概是这样的：

1 找到一条从s->t的路径，该路径可随意选取，无任何要求，计算该路径的流量，见下图；

![52440983801](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfYH0iaZKru8P7282yL8sMj7jy7LiaicObM5zMkCyu0xoEUicI3xx9icHWRjIPUtTbTqiaafuWn2VehjMibbA/0?wx_fmt=png)

2 找到第二条路径，这条路径与第一条路径不重复，同样计算该路径流量；

![52440994295](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfYH0iaZKru8P7282yL8sMj7jOhFEAYIsS7RG3OCc8XNpdS5epqhZf3rTJzGwEEFqUBSNicScu6Lyqvg/0?wx_fmt=png)

3 这个时候似乎已经无法再找一条不重复路径使t点流量增加了，那么t点流量就确定是20了吗？FF算法这时会找到第三条路径，看图片前要不要再想想看，我这里留两行空(｀・ω・´)





![52441019816](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfYH0iaZKru8P7282yL8sMj7jj1TG75bzOjzZzBIMPdctByDLTIxGgSoNIYfic6zxOFJaANp8DpsfIgA/0?wx_fmt=png)

FF会找到一条包含反向边的路径，该反向边只要求此时流量大于0即可，同上面已找到的两条路径一起，这些路径被称为扩充路径(augmenting path)，扩充路径要求(1)路径上的正向边流量未达到容量；(2)路径上的反向边流量不为空。具体不再多说，看图自行理解一下~

4 找到最后一条扩充路径

![52441063241](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfYH0iaZKru8P7282yL8sMj7j60wQe3rBK3mwJGb9GicgbD4icekJnJibZmspicPIcjyCiaBicaadibawCdIQw/0?wx_fmt=png)

所以s->t的最大流量就是28 啦~



那么讲这些跟球队淘汰问题又有什么联系呢？实际上，我们的确可以把确定一个球队是否淘汰的问题转化为流量图。

如果要确定Det是否淘汰，我们首先假设Det在剩下比赛中全胜，则其最终胜场为`w[4] + r[4]`，然后我们可以对剩余四个队建立如下流量图：

![](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfYH0iaZKru8P7282yL8sMj7jUFKXaVDaVEJ0D48TicqIqvlIvib5ucKDD8nuQHmTxjgY3gBd9WrKH9xA/0?wx_fmt=png)

其中s、t节点为虚设节点，0、1、2、3这四个节点分别代表NY，Bal，Bos，Tor，0-1节点即代表NY-Bal，s-(0-1)的容量设为NY-Bal的剩余场数`g[0][1]`，此处为3，流量初始化为0，其余节点同。(0-1)-0的流量即为NY对Bal的胜利场数，初始化为0，其容量只要设置成`>=g[0][1]`即可，为了方便我们设置成$+\infin$。

最关键的就是0-t的容量了，我们这里设置成`w[4] + r[4] - w[0]`，意思就是NY在于其它队比赛的过程中其最后胜场不能超出Det的胜场数（这里已经假设Det剩下比赛全胜）。至此，我们就构建了一个流量图G，如果我们求G中s->t的最大流量会得出什么结论呢？显然，如果在达到最大流量后与s相邻的边中一旦有一条（假设为s-(A-B)）其流量未达到其容量，就表示A-B两队之间的比赛尚未打完，再打下去必然有一个队胜场数超过Det。而所有这些边对应的球队就是我们最终要求的子集啦。

FF算法的具体实现方式这里不表（实际上作业也没用到，只是调用了API而已），有兴趣的找维基(｀・ω・´)

## Programming

首先先看下我们要求编写的接口

```java
public BaseballElimination(String filename)                    // create a baseball division from given filename in format specified below
public              int numberOfTeams()                        // number of teams
public Iterable<String> teams()                                // all teams
public              int wins(String team)                      // number of wins for given team
public              int losses(String team)                    // number of losses for given team
public              int remaining(String team)                 // number of remaining games for given team
public              int against(String team1, String team2)    // number of remaining games between team1 and team2
public          boolean isEliminated(String team)              // is given team eliminated?
public Iterable<String> certificateOfElimination(String team)  // subset R of teams that eliminates given team; null if not eliminated
```

其实这道题的最主要难点是在构造这个流量图上，毕竟其它东西内置的算法已经提供了。不过构造流量图这个任务是在isElinimated方法和certificateOfElimination方法中实现的（因为你必须提供team的名字来构造流量图来确定该team是否被淘汰），没有明确公共接口。我这里只放构造流量图的代码。

```java
private FlowNetwork generate(int ID) {  
        // ID is the id number of team to be certained if is eliminated
        int numOfi2j = (V - 1) * (V - 2) / 2;  // number of i-j node
        int numOfVert = V - 1;
        FlowNetwork G = new FlowNetwork(numOfi2j + numOfVert + 2);  // don't forget s and t
        int count = 1;
        for (int i = 0; i < V; i++) {
            for (int j = i + 1; j < V; j++) {
                if (i == ID || j == ID) continue;
                int p = i < ID ? i : i - 1;
                int q = j < ID ? j : j - 1;
                FlowEdge edge1 = new FlowEdge(0, count, play_against[i][j]);
                FlowEdge edge2 = new FlowEdge(count, numOfi2j + 1 + p, Double.POSITIVE_INFINITY);
                FlowEdge edge3 = new FlowEdge(count, numOfi2j + 1 + q, Double.POSITIVE_INFINITY);
                G.addEdge(edge1);
                G.addEdge(edge2);
                G.addEdge(edge3);
                count++;
            }
        }
        for (int i = numOfi2j + 1; i < G.V() - 1; i++) {
            int p = i - numOfi2j - 1;
            if (p >= ID)
                p += 1;
            double capacity = wins[ID] + remaining[ID] - wins[p];
            FlowEdge edge = new FlowEdge(i, G.V() - 1, capacity);
            G.addEdge(edge);
        }
        return G;
    }
```

很直观地写了一下......不过的确也想不到更好的构造方法。

## Summary

最近又回南京做试验了，所以这周的日志拖到了现在。其实这周作业也是周四才完成的，作业本身不难，但这周的内容是视频独有的，配套教材并没有提及FF，FF本身又有点绕，看着一堆洋文第一次有点有心无力的感觉，不过总算也是完成了吧。想来这个系列写到现在也已经两个月了，算法课强逼着自己完成了大半，但对于自己究竟掌握多少还是没什么把握。机器学习方面，进度卡在SVM也已经好久了，对偶函数和KKT条件秀得我头晕。四月好像不知不觉就在走出国流程和来回南京当中度过，暑期实习找到现在除了几次机试也没接到过面试或笔试通知，多半怕是也凉凉了（看了下头条简历还在筛选，原来还想修改下简历，不过不知道为啥我投的岗位已经下架了Σ(ﾟдﾟ;)）。这是一篇满腹牢骚的个人四月总结，但总还是要心怀希望的。路艰且长吧，希望每个迷茫的同学都有个好的结果（有看到这的同学留个言吧，哈哈哈）。