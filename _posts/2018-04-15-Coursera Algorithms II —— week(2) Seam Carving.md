---
layout: post
title: "Coursera Algorithms II —— week(2) Seam Carving"
category: Algorithms
---

放一张最近看到的动图，快笑死了:smile:

![](https://mmbiz.qpic.cn/mmbiz_gif/ickltPAryrfbAMvVbJibHUHbicnF3z2PiaK5vA6oYNFcVhUx54PhTa8OfQPVAibaaiabY33Cz01XdtG4u5ibc7DhXOsbg/0?wx_fmt=gif)

## Assignment

> 对下图左的图片进行缩放，要求尽可能保留图片信息，缩放后的效果如下图右所示。
>
> <img src="https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbAMvVbJibHUHbicnF3z2PiaK5SshN1fdut9vJeYwJWTw92SjmW9CpGicHt3U9ED7EjAcphPIkDOnbhhA/0?wx_fmt=png" style="zoom:85%"> <img src="https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbAMvVbJibHUHbicnF3z2PiaK5dNZculcLgeKsqRlzINgia0icMAJt9yJDeDMbNs1Z9ZF4BA4Rxf3hENCQ/0?wx_fmt=png" style="zoom:85%">

## Introduction

首先观察一下图片，看着挺像就是把原图右边部分裁掉而已，但仔细观察可以发现，相比于左图，右图中人与人之间的距离明显变近。缩放后的图与原图相比并没有什么违和感，说明缩放过程中只是把原图中不重要的信息（在这张图中就是水面和天空）给裁掉了。

除了图片的缩放之外，还有很多地方也会有这方面功能的要求，比如桌面应用界面的缩放等。

<img src="https://mmbiz.qpic.cn/mmbiz_gif/ickltPAryrfbAMvVbJibHUHbicnF3z2PiaK5icdeTBwDu8TxxheqrHQUMD8aT3AVCVDF6NnrkkEJs9b9dB6FhJZ0nDw/0?wx_fmt=gif" height="400px">

本文将介绍一种基于图论中最短路径的缩放算法——seam carver，直译过来的话可以这么理解：假设我们现在需要对图片按照宽度方向进行缩放，那么首先我们需要在竖直方向确定一条裂缝（seam），然后我们照着这条裂缝把图片撕成两半，最后把裂缝左右两边的图片拼回去（不包含裂缝）即可得到一张宽度比原图小一号的新图。不断重复该步骤直到得到的图片宽度达到要求，具体过程就像下方gif展示的一样（这里旁边的边其实是上一幅图留下的，因为这个gif是我自己写的，我不太清楚怎么清除上一幅图所以留下来了，不过这样也更直观一些）。

![](https://mmbiz.qpic.cn/mmbiz_gif/ickltPAryrfbAMvVbJibHUHbicnF3z2PiaK5YzViaYjGjicjs4y2LIw4xccLGhhAVV32ic21OicEf5pgJraFzvml65ib2Tg/0?wx_fmt=gif)

所以剩下的任务就是找到那条裂缝了，寻找裂缝的依据只有一条——这条裂缝相比于其它裂缝包含的信息更少。

通常的，我们会认为在一张图上，边界部分相比其它地方包含更多的信息（对于上面这张图，显然我们在缩放的时候不会想删除人与水交界的那些像素点，但是水面和天空的部分就显得可有可无了）。对于某个边界像素点，它的一个重要特征就是其左右或者上下的像素点在颜色上往往有着显著的差异，因此，我们可以利用这个性质来定义一个像素点所包含的信息。裂缝的信息则定义为裂缝途径的所有像素点的信息和。

一个像素点的具体信息计算方式可以有很多种，这里简单介绍其中一个。下图展示了`width x height = 3 x 4`的像素图，定义左上角的像素点坐标为`(0, 0)`，右下角的坐标定义为`(3, 4)`，其余像素点的坐标用`(w, h)`来表示，其中`w`表示其所在宽度，`h`表示其所在高度。对于像素点`(1, 2)`，其信息定义为$\sqrt{\Delta_x^2(1, 2) + \Delta_y^2(1, 2)}$，其中$\Delta_x^2(1, 2) = R_x(1, 2)^2 + G_x(1, 2)^2 + B_x(1, 2)^2$，这里$R_x, G_x, B_x$分别表示像素点`(0, 2)`与`(2, 2)`RGB值的差分，即$R_x(1, 2) = 255 - 255 = 0, G_x(1, 2) = 205 - 203 = 2, B_x(1, 2) = 255 - 51 = 204$。对于边界点，则默认其信息为1000。

![](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbAMvVbJibHUHbicnF3z2PiaK5flm3IshGoN12kpMNed9Wmib2yEWZ8ibeiaz3dTjYricvf9L4WPF1YURfag/0?wx_fmt=png)

现在每个像素点的信息也定义完毕啦，正式开始介绍今天的重点——寻找图上信息和最小的裂缝。

首先再明确一下裂缝的几何定义，一条纵向裂缝应该满足两个条件：

1 图的每一行有且仅有一个像素点在裂缝路径上；

2 裂缝路径上相邻像素点位置的宽度信息其差分绝对值不应大于1（意思就是上一个像素点如果是`(5, 2)`的话，下一个像素点只能在`(4, 3), (5, 3), (6, 3)`之中选一个）。

虽然目标很明确，但寻找**信息和最小的裂缝**这个问题在算法意义上看起来还不是那么直观。现在我们再进一步，把整张图改成**有向无环图结构**（这里的图结构是算法上的图结构，和上文说的图不太一样）。

<img src="https://mmbiz.qpic.cn/mmbiz_jpg/ickltPAryrfbAMvVbJibHUHbicnF3z2PiaK5AfgzdiciaFFnVeKvdicNM3jlxIoFspMibAP67UlUsxAVLZ4ickKorxzZn8g/0?wx_fmt=jpeg" height="400px">



为了区别于上文提到的图，我们把该有向无环图结构称为`G`，`G`上的每一个节点代表原图中的像素点，像素点`p->q`的路径长度为`q`像素点的信息值。这里我们新增了两个虚拟点`A`与`B`，`B`点的信息值设为0，`A`点与最顶端的所有像素点相连（以`A`为起点），`B`点与最底端的所有像素点相连（以`B`为终点）。这样寻找**信息和最小的裂缝**这个问题在算法上就转为直观的寻找**`A`与`B`点之间的最短路径**问题。

关于最短路径，对不同的图结构有不同的做法。一般的有向无环图是在对节点进行拓扑排序后依次放松各节点来求最短路径，不过本题比较特殊，因为节点的拓扑排序实际上已经给你了（拓扑排序大致可以理解为：如果在图结构中`p->q`，则在拓扑排序后得到的节点序列中`p`在`q`之前），所以我们只要实现放松（Relax）这个步骤就好。

具体过程如下：

1 在图结构中指定一个起点`o`，终点`t`，对任意节点`i`，另`dis[i]`为`i`到`o`的最短路径距离，其中`dis[o] = 0`，其余所有节点初始化为$+\infin$，`edge(i, j)`为`i`到`j`的距离，`path[i]`为`o`到`i`这条路径上`i`的上一个节点，初始化为`i`本身；

2 对图结构中的所有节点作拓扑排序，得到一个节点序列`topological`，对任意节点`i`，另`index[i]`为`i`在`topological`中的位置，若有`i->j`，则必然有`index[i] < index[j]`;

3 从`index[o]`开始遍历`topological`，假设`p->[q1, q2, ..., qn]`，则有`dis[q1] = min(dis[q1], dis[p] + edge(p, q1))`，如果`min`取后者，则另`path[q1] = p `，依次类推；

4 当`topological`遍历到`index[t]`时停止循环，此时`dis[t]`即为`o`到`t`的最短路径距离，递归`path[t]`即可得到这条路径上的所有点；

## Programming

老规矩，先看作业提供的接口：

```java
public class SeamCarver {
   public SeamCarver(Picture picture)                // create a seam carver object based on the given picture
   public Picture picture()                          // current picture
   public     int width()                            // width of current picture
   public     int height()                           // height of current picture
   public  double energy(int x, int y)               // energy of pixel at column x and row y
   public   int[] findHorizontalSeam()               // sequence of indices for horizontal seam
   public   int[] findVerticalSeam()                 // sequence of indices for vertical seam
   public    void removeHorizontalSeam(int[] seam)   // remove horizontal seam from current picture
   public    void removeVerticalSeam(int[] seam)     // remove vertical seam from current picture
}
```

构造、求像素点信息值这几个函数比较简单，水平和竖直其实就一个转置的问题，所以我这里只放找到水平裂缝（findHorizontalSeam）的代码：

```java
public class SeamCarver {
    private Picture currPic;           // current picture after reduce width or height
    private int width;                 // width of current picture
    private int height;                // height of current picture
    private double[][] currEnergy;     // matrix to restore each energy of pixel in current picture
}
```

```java
public int[] findHorizontalSeam() {
    int[] res = new int[width];
    double[] distTo = new double[width * height + 1];
    int[] edgeTo = new int[width * height + 1];
    for (int i = 0; i < distTo.length; i++) {
        if (i < height) {
            distTo[i] = 1000;
            edgeTo[i] = -1;
        } else {
            distTo[i] = Double.POSITIVE_INFINITY;
            edgeTo[i] = i;
        }
    }
    for (int x = 0; x < width; x++) {
        for (int y = 0; y < height; y++)
            horizontalRelax(x, y, distTo, edgeTo);  // private function to calculate distance
    }
    int curr = width * height;
    int currWidth = width;
    while (edgeTo[curr] != -1) {
        curr = edgeTo[curr];
        res[--currWidth] = curr % height;
    }
    return res;
}
```

其中的horizontalRelax就是我们的求距离函数了，这个怎么写都行，只是记得注意边界条件，我就不放啦（其实只是懒）。

## Summary

图的几个算法感觉都挺重要的，下星期还有一篇（然而我代码都没开始写）。上次携程机试的最后一题就是求一幅图中的最长路径（然而我那时候并没有学到），最长路径和最短路径基本是一致的，只要我们把距离改成负的就行。另外图还有几个很有趣的性质，课程中提到了一个根据货币汇率变化套现过程的实现，感觉很有趣，我准备结合爬虫实践一下。