---
layout: post
title: "Coursera Algorithms II —— week(5) Burrows-Wheeler"
category: Algorithms
---

## Assignment

> 本周作业地址：
>
> http://coursera.cs.princeton.edu/algs4/assignments/burrows.html
>
> 在Huffman压缩的基础上，对输入字符串“ABRACADABRA!”进行BW变换变为更有益于压缩的形式。

## Introduction

<p color='red'>信息熵</p>

在写压缩之前我想首先写下我们经常看到的信息熵到底是怎么回事，这个概念与本次用到的Huffman压缩息息相关。

我习得的关于信息熵的知识来源于这篇博客（http://colah.github.io/posts/2015-09-Visual-Information/），有兴趣的可以直接看原文，以下介绍只是简化版本。

假设我们现在有一个文本，文本很长，但其实只包含四个单词：dog, cat, fish, bird。如果现在要求你发送一封电报给A，A要求电报上只准出现0或者1（二进制世界的真实传输形态），所以你决定和A商讨一套只有你们两个人懂的编码技术来对这个文本编码。

最简单的，我们可以这样做。

![52557624153](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbyJ4kFeSOkxtVvUsrFEbSk5MfE9RHEURdlfvb2ZAKnxqYahOkp1452dMJjVH5czrqyObq5yiaeicWQ/0?wx_fmt=png)

每两个bit（一个bit为0或者1）代表一个单词，如果你的电报信息是“dog cat fish bird”，那么最终你的电报发送版本就是“00011011”。因为A也知道这套编码技术（称为`C0`吧），所以交流对你们来说不成问题。

现在我们升级一下情景，假设电报中每个单词出现的频率不一样，比如总共8个单词，dog出现了4次，cat出现2次，fish和bird各出现一次。很不幸的，你为发电报付的钱与你的电报长度息息相关，这时`C0`似乎就显得不那么合理了。

为了说明为什么不合理，`C0`总共需要用到8x2=16个bit。我这里另举一套编码技术`C1`，dog对应0，cat对应10，fish对应110，bird对应111，那么这时只需用到1x4+2*2+3+3=14个bit，显然为了钱考虑，虽然麻烦了点我们也应该用`C1`。

有同学可能问为什么cat不能对应1，然后fish用10，bird用01。这种编码（称为`C2`吧）不可取的地方在于我们很容易在各单词之间造成混淆。举例来说，还是“dog cat fish bird”，采用`C1`后我们的电报为“010110111”。A收到电报后开始解读，0直接读为dog就好（因为没有一个单词的前缀是01开头的，所以我们读到0就可以直接输出了），1没有对应的单词，那么我们继续看10，读到了cat，依此类推读110，111。也就是说我们的分割方式只可能是“0 10 110 111”这么一种。

然而如果采用`C2`，我们的电报为“011001”，这个时候分割方式可能就多啦，除了“0 1 10 01”，还有“01 10 01”，“01 10 0 1”等等，这个时候就造成了语义混淆啦（为什么这里不对空格编码？因为空格也要占用bit啊^_^）。

我们可以用单词树来表示上述结构。如下图所示，每次从root开始搜索，每个节点碰到0往左，碰到1往右，如果某个节点对应一个单词（我们称为叶节点，就像树一样，直到叶子才停止分叉），那么我们输出这个单词并返回root即可。你和A只要共同确定了这么一棵单词树，那么信息沟通就无障碍。

![52562238290](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbyJ4kFeSOkxtVvUsrFEbSkO5mhMVia1tgMcgBAYbe40xd9FwSYAicWrjH9ACrX5RFbEias7wyIHBtvQ/0?wx_fmt=png)

最后的问题就是我们可以确定上述的`C1`编码是使电报长度最小的方式吗？这个问题是Huffman压缩具体要解决的问题，限于篇幅这篇文章我不谈这个，但我这里会给出求电报最小长度的公式。与原博采用图解的方式不同，我觉得数学公式会更明确一些。

首先是一个cost的概念。回到`C1`编码技术，我们现在假设有一个由3个bit组成的账本，其总容量为1，因为每个bit由0或者1构成，所以总共有2^3=8个编码。对于dog，其对应的编码为0，这时候因为我们之前讨论的避免歧义的关系，账本上所有第一个bit是0的编码都无法继续使用，也就是dog相当于占用了4个编码，对应下图中dog所在红框的四个叶节点，即总容量的1/2。对于cat，其对应的编码为10，占用了所有以10开头的编码，所以其占用容量为1/4。相应的，fish和bird占用的容量为1/8。

![52562287922](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbyJ4kFeSOkxtVvUsrFEbSkq8XeQFrXs35Eu4ic786gunuFmSqLzs0PC2EHUY696TpHN9propfjibeQ/0?wx_fmt=png)

因此，我们可以得出结论，对于一个长度为L的编码，其占用的容量为1/(2^L)，同时所有编码占到的总容量为1。

求电报的最小长度相当于求每个单词的平均最小长度，设对于单词x其频率为p(x)，编码长度为L(x)，再加上cost的概念我们可以得到下面的公式及其求解：

![52558531448](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbyJ4kFeSOkxtVvUsrFEbSkmQCKbdr3mrd0rUDeT02fSLfcE0dTlbKrH8gqzFu5qzXqR3O7njJ2Wg/0?wx_fmt=png)

上式中P是已知值，但P有个隐含条件就是所有频率和为1。求解可以采用拉格朗日乘子，很简单可以得到对应L的乘子为-1/ln(2)，而**L(x)=-log2(P(x))**，有兴趣的自己推一下（因为打公式实在太麻烦了\_(:з」∠)_）。

所以最后，对于一封电报，其每个单词的平均最小长度就是sum(-p*log2(p))啦，这个公式衍生出的交叉信息熵会在各个机器学习算法的损失函数中见到哦。

<p color='red'>Huffman压缩</p>

Huffman压缩的原理就是上面讲的发电报，不同的是它是对每个字母赋予一个编码（包括空格），也就是说可能A赋予为0，B赋予为10等等。这里面需要注意的一点是在计算机中，A与B可能是以ASCII码的形式存储的，占用7个或8个字节（取决于是否为ASCII扩展集），变为0和1的形式后其编码长度就是实际bit长度了，这样才得以实现压缩。同时如上面分析的，每个字母的编码与字母出现的频率相关，保证了总的电报长度最小。具体的Huffman压缩实现方法有兴趣可以百度。

<p color='red'>Burrows-Wheeler变换</p>

Burrows-Wheeler变换（以下简称BW变换）是本次作业的主要内容（没错，我也不知道为什么要扯上面这么多）。关于BW，“dog cat”的例子讲起来有点复杂，我们换成简单的字符串`S`“AABCAABC”来讲解。

如果直接对`S`进行Huffman压缩的话，显然A对应0，B对应10，C对应11，所以`S`经过压缩后其编码为001011001011，总共12个bit，这是`S`用01进行电报传输的最优长度。

现在我们首先对`S`进行排序，得到`S1`’AAAABBCC‘，这时我们可以看到有很多连续的重复字符，我们对所有这些重复字符全部设为A，此时`S1`转换为`S2`'AAAABACA'。`S2`要还原为`S1`是非常容易的（不该出现A的地方就是重复字符~），也就是说信息没有丢失，我们再对这个字符串进行Huffman压缩，显然因为一个B和一个C变为A的原因电报长度相比`S`会减少2。

以上就是Burrows-Wheeler变换的主要思想，首先用一种特定方式对`S`进行重排（并不只是简单的排序，这里我没有明白为什么不能直接排序）得到一个重复字符连续出现的新字符串`S1`，再然后对这些重复字符全部用同一个字符表示即可（MoveToFront处理或者游标处理）。具体的方式这里不写，有兴趣百度~

## Programming

最后一次作业我就不放代码啦，我想了想我自己也应该不想在公众号上阅读别人的代码，哈哈。

如果想看代码的话可以去我GitHub:

https://github.com/suyako-to-be-a-programmer/Algorithms-part2/tree/master/burrows

## Summary

算法课的10期作业到此结束，也没想到自己能一周一期地写完吧。但也只是完成了作业而已，不谈网络和操作系统这些东西，即使是算法本身也还有茫茫的东西需要我去补。想过之后写一些机器学习的东西，不过自己也才刚入门，还是不来误导人了。但这个公众号应该还是会时不时更新到我秋招结束为止，转行艰难，希望能给后面的学弟学妹踩出一条路。

最后放张成绩图作为总结吧~最后一次作业分数有点低，不过实在不想改了（手动捂脸）。

![52562301234](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbyJ4kFeSOkxtVvUsrFEbSklmnBdK7mAeLI1XJXiaFL45TaUkdWYHuCvOicPSwjQySIs81PSH1FIqJA/0?wx_fmt=png)

