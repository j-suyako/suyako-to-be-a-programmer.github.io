---
layout: post
title: "Coursera Algorithms —— week(3) Pattern Recognition"
category: Algorithms
---

![](https://mmbiz.qpic.cn/mmbiz_gif/ickltPAryrfbhlWWtP2ibNickVKjtG8ics2dp5Ns8jBYBRliaYtxIOE8rQSHbPIURPV7ffxQjwgNGJ0nK6eje9QHqTQ/0?wx_fmt=gif)

## Assignment

本周作业地址：http://coursera.cs.princeton.edu/algs4/checklists/collinear.html

> 以下图为例，分别采用暴力循环及排序的方法，找出图中所有由四个或四个以上点构成的直线。
>
> ![](https://mmbiz.qpic.cn/mmbiz_png/ickltPAryrfbhlWWtP2ibNickVKjtG8ics2dTs0dOvpN97w1PfDuMyd8cawoPwmx5NfozarEuUlFjoia6w6WAj0LfTw/0?wx_fmt=png)

## Introduction

计算机的广泛使用使得数据无处不在，而整理数据的第一步往往是排序。排序的重要性无需多言，面对各种各样的实际问题也催生了各类排序方法，这里简单概括一下几种常见的排序方式（以对大小为n的数组`a[n]`进行从小到大的排序为例）。

* 选择排序

  遍历数组`a[0:n-1]`，找到最小值并记录其位置`i`，交换`a[0]`与`a[i]`，之后遍历数组`a[1:n-1]`，找到最小值并记录位置`j`，交换`a[1]`与`a[j]`......依此类推，直到将整个数组排序。

  易知不论数组原先的实际情况如何，都需要经过约**n^2**次比较及**n**次交换，因此时间复杂度为**O(n^2)**。

* 插入排序

  我初看时以为就是冒泡，因为形式上有点相似......两者最大的不同就是要不要重复遍历数组。

  想象有人在发牌，等他发完牌后你面前有17张牌，你拿起第一张是8，拿起第二张是5，这时你一般会把5放到8的前面。再拿起一张看是3，你会再把3放到最前面......不断重复，在你拿起最后一张牌并插入的时候你的牌堆就整理好了。

  插入排序的原理就是上述的整牌过程，现在`a[0] = 8`，移到下一位发现`a[1] = 5`，所以我们交换`a[0]`与`a[1]`，即`a[0] = 5, a[1] = 8`，再移到下一位发现`a[2] = 3`，先与`a[1]`比较得出`a[2] < a[1]`，交换`a[1]`与`a[2]`得`a[1] = 3, a[2] = 8`。再进行`a[1]`与`a[0]`的比较得出`a[1] < a[0] `，交换`a[0]`与`a[1]`，不断重复上述步骤直到整个数组排序。

   插入排序的时间复杂度与数组情况有关，如果为有序数组，则只需经过**n-1**次比较即可，时间复杂度为**O(n)**，如果为逆序数组，需要经过**n^2**次比较及**n^2**次交换，时间复杂度为**O(n^2)**。

* 希尔排序

  由上述复杂度分析可知，插入排序在数组部分有序的情况下极为高效，这个性质催生了希尔排序。

  还是拿牌来说事，假设17张牌最后一张（即`a[16]`）为3的话，这个时候一个个回去比显然效率太低了，有什么办法可以提高效率呢？最快的肯定是拿`a[16]`与`a[0]`比较，但我们并不知道最后会拿到什么，所以折中一下，让`a[16]`与`a[12]`比，新的`a[12]`与`a[8]`比较......依此类推，直到`a[4]`与`a[0]`比较，在这个过程中我们就得到了一个`a[0]、a[4]、a[8]、a[12]、a[16]`局部有序的新数组。同理，可以让`a[1]、a[5]、a[9]、a[13]`，`a[2]...a[14]`，`a[3]...a[15]`局部有序。之后我们再进行插入排序使整个数组排序就快多啦。

  希尔排序的复杂度难以准确分析，因为这不仅与数组的初始状态有关，也与在你进行局部排序的过程使用的步长大小有关，一般情况下步长选择为1、4、13等（3的倍数+1）。该步长序列下，最坏情况的时间复杂度为**O(n^3/2)**。


* 归并排序

  假设两个已经排序好的数组`a[n], b[m]`，我们现在想让两个数组合并得到一个新的数组`c[n+m]`，怎么排最快相信大家都知道......就是逐一比较嘛，小的那个就放到新的数组里，最坏情况下比较n+m次，时间复杂度为**O(n)**。

  现在问题来了，怎么让`a[n]`和`b[m]`变得有序？以`a[n]`为例，我们同样将其分为两部分`a1[n1]`和`a2[n2]`（一般情况下都是等分的），如果这两个部分是有序的我们就直接归并，如果不是有序的我们就继续往下分......一直分到最后，对某个大小为2的数组`ap[np]`可以分出两个大小都为1的子数组，这时候就可以直接调用归并了。归并完后`ap[np]`也变得有序了，就可以与`aq[nq]`进行归并......如此往复就可以得到有序的`a[n]`。

  关于时间复杂度，原始数组`a[n]`分成两个数组，在两个数组有序后归并最坏情况下比较n次；两个数组分成四个数组，每个比较n/2次，总共2个，加起来同样需要n次比较。所以每一轮的比较次数都是**O(n)**的时间复杂度，总共需要分割**lg(n)**轮（以2为底），因此最终的时间复杂度为**O(nlgn)**。

除了以上几种排序，还有最经典的快排，这里略过不表。总结一下各个排序的基本情况。

| 方法     | 最优情况时间复杂度 | 最坏情况时间复杂度 | 平均时间复杂度 |
| -------- | :----------------: | :----------------: | :------------: |
| 选择排序 |       O(n^2)       |       O(n^2)       |     O(n^2)     |
| 插入排序 |        O(n)        |       O(n^2)       |     O(n^2)     |
| 希尔排序 |        O(n)        |      O(n^3/2)      |     \>O(n)     |
| 归并排序 |        O(n)        |      O(nlgn)       |    O(nlgn)     |
| 快速排序 |      O(nlgn)       |      O(nlgn)       |    O(nlgn)     |

可以证明一个最理想的排序算法最坏情况下也至少需要经过**lg(n!)**次的比较，归并和快排已经相当接近这个下界了。

## Programming

作业的推荐时间是5个钟头，就我个人经验来看这5个钟头让你达到80分没问题，但要克服其中的难题要学习的东西远超5个钟头。

这次作业提供了一个LineSegments类，并给了三个JAVA类要求编写，先来看下每个类的API接口。

**Point**

```java
public class Point implements Comparable<Point> {
   /*Create an immutable data type Point that represents a point in the plane by implementing the following API*/
   public  Point(int x, int y)              // constructs the point (x, y)
   public  void draw()                      // draws this point
   public  void drawTo(Point that)          // draws the line segment from this point to that point
   public  String toString()                // string representation
   public  int compareTo(Point that)        // compare two points by y-coordinates, breaking ties by x-coordinates
   public  double slopeTo(Point that)       // the slope between this point and that point
   public  Comparator<Point> slopeOrder()   // compare two points by slopes they make with this point
}
```

**LineSegments**

```java
public class LineSegment {
   /*represent line segments in the plane*/
   public LineSegment(Point p, Point q)        // constructs the line segment between points p and q
   public void draw()                          // draws this line segment
   public String toString()                    // string representation
}
```

**BruteCollinearPoints**

```java
public class BruteCollinearPoints {
   /*examines 4 points at a time and checks whether they all lie on the same line segment*/
   public BruteCollinearPoints(Point[] points)    // finds all line segments containing 4 points
   public int numberOfSegments()                  // the number of line segments
   public LineSegment[] segments()                // the line segments
}
```

**FastCollinearPoints**

```java
public class FastCollinearPoints {
   /*A faster, sorting-based solution*/
   public FastCollinearPoints(Point[] points)     // finds all line segments containing 4 or more points
   public int numberOfSegments()                  // the number of line segments
   public LineSegment[] segments()                // the line segments
}
```

输入输出形式应该如下：

```java
% input.txt
[[10000,0], [0,10000], [3000,7000], [7000,3000], [20000,21000], [3000,4000], [14000,15000], [6000,7000]]
```

```java
% java FastCollinearPoints input.txt
(3000, 4000) -> (20000, 21000) 
(0, 10000) -> (10000, 0)
```

本次作业的最大难点在于如果是一条由5个点构成的直线，其输出形式类似于`point[1]->point[5]`，而不应该出现`point[2]->point[5]`等子集。我一开始的想法是给每个`point`类增加一个端点容器，对所有`points`按纵坐标值排序，之后遍历`points`，在`point1`对其之后的所有`points`进行slope排序，在对`point2->point5`形成线段m1时`point2->point5`的端点容器中push point1。之后在遍历到`point2`时，在对其之后的`point3->point5`形成线段m2时从其端点容器中读取`point1`，比较`point1`与`point2`两点的斜率与m2的斜率，两者相等即排除m2。

该方法可行是可行，就是要改`point`的API，我在`FastCollinearPoints`类中重写了一个新的`point`，通过后就放弃了这个方法。后来逛课程论坛的时候看到一个方法，每个`point1`对其它`points`进行slope排序，在数组中会存在一段相对`point1`其slope值相等`point[2]-point[5]`的随机连续排列，将`point1`与`point[2]-point[5]`中的最小值比较（纵坐标比较），如果`point1`比较小就找到`point[2]-point[5]`中的最大值`point[max]`，最后形成线段`point1->point[max]`。

说得比较绕......总之这种方法不需要改API，而且写起来更容易点。

以下限于篇幅原因只放每个类重点的API

**Point**

```java
public class Point implements Comparable<Point> {

    private final int x;     // x-coordinate of this point
    private final int y;     // y-coordinate of this point
    
    public double slopeTo(Point that) {
        /*Returns the slope between this point and the specified point.*/
        if (this.x == that.x && this.y == that.y)
            return Double.NEGATIVE_INFINITY;
        if (this.x == that.x)
            return Double.POSITIVE_INFINITY;
        if (this.y == that.y)
            return +0.0;
        return (double)(that.y - this.y) / (that.x - this.x);
    }
    
    public int compareTo(Point that) {
        /*Compares two points by y-coordinate, breaking ties by x-coordinate.*/
        if (this.y < that.y)
            return -1;
        else if (this.y > that.y)
            return 1;
        else
            return Integer.compare(this.x, that.x);
    }
    
    public Comparator<Point> slopeOrder() {
        /*Compares two points by the slope they make with this point.*/
        return new BySlope();
    }
    
    private class BySlope implements Comparator<Point>{
        public int compare(Point that1,Point that2){
            double slope1 = Point.this.slopeTo(that1);
            double slope2 = Point.this.slopeTo(that2);
            if (slope1 < slope2)
                return -1;
            else if (slope1 == slope2)
                return 0;
            else
                return 1;
        }
    }
```

**BruteCollinearPoints**

```java
public class BruteCollinearPoints {

    private final LineSegment[] final_res;
    private int count;
    
    public BruteCollinearPoints(Point[] points) {
        /* find all line segments containing 4 points */
        if (points == null) throw new IllegalArgumentException();
        Point[] points_copy = check(points);  // due to points is referenced in this class, we could mutate it accicedently, so here we get a copy of points if points is valid.
        LineSegment[] temp_res = new LineSegment[2];
        count = 0;
        int m = points_copy.length;
        for (int i = 0; i < m - 3; i++) {
            for (int j = i + 1; j < m - 2; j++) {

                double slope1 = points_copy[i].slopeTo(points_copy[j]);  // get slope of point i and point j

                for (int k = j + 1; k < m - 1; k++) {
                    double slope2 = points_copy[i].slopeTo(points_copy[k]);  // get slope of point i and point k
                    // reduce loop of slope3
                    if (slope1 == slope2) {
                        for (int l = k + 1; l < m; l++) {
                            double slope3 = points_copy[i].slopeTo(points_copy[l]);  // get slope of point i and point l
                            if (slope1 == slope3) {
                                // increase the capacity of temp_res as array
                                if (count == temp_res.length) {
                                    LineSegment[] temp = temp_res;
                                    temp_res = Arrays.copyOf(temp, 2*count);
                                }
                                temp_res[count++] = new LineSegment(points_copy[i], points_copy[l]);
                                break;  // no more than 5 collinear points in input
                            }
                        }
                    }
                }
            }
        }
        final_res = Arrays.copyOf(temp_res, count);
    }
    
    private Point[] check(Point[] points){
        /* throw IllegalArgumentException if any point in points is null or any repeated points and return a copy of points if input is valid*/
        Point[] points_copy = Arrays.copyOf(points, points.length);
        for (Point each : points_copy)
            if (each == null) throw new IllegalArgumentException();
        Arrays.sort(points_copy);
        for (int i = 0; i < points_copy.length - 1; i++)
            if (points_copy[i].compareTo(points_copy[i+1]) == 0) throw new IllegalArgumentException();
        return points_copy;
    }
    
    public LineSegment[] segments(){
        /* the line segments, due to the same reason as points, it should return a copy of final_res */
        return Arrays.copyOf(final_res, final_res.length);
    }
```

**FastCollinearPoints**

```java
public class FastCollinearPoints {

    private final LineSegment[] final_res;
    private int count;
    
    public FastCollinearPoints(Point[] points) {
        if (points == null) throw new IllegalArgumentException();
        check(points);  // check here doesn't return a copy of point because it's not useful next
        LineSegment[] temp_res = new LineSegment[2];
        count = 0;
        int m = points.length;
        Point[] points_copy = Arrays.copyOf(points, m);
        for (Point each : points) {
            Comparator<Point> c = each.slopeOrder();
            Arrays.sort(points_copy, c);
            // j represents the index of second point in a segment
            int j = 1;
            while (j < m - 2) {
                double slope1 = points_copy[0].slopeTo(points_copy[j]);
                // k represents the index of last point in a segment then plus 1
                int k = j + 2;
                while (k < m && slope1 == points_copy[0].slopeTo(points_copy[k]))
                    k = k + 1;
                if (k == j + 2) j++;
                else {
                    if (count == temp_res.length) {
                        LineSegment[] temp2 = temp_res;
                        temp_res = Arrays.copyOf(temp2, 2*count);
                    }
                    Point[] tuple = Arrays.copyOfRange(points_copy, j, k);
                    Point[] min_max = min_max(tuple);  // find the min and max in tuple
                    if (points_copy[0].compareTo(min_max[0]) < 0) temp_res[count++] = new LineSegment(points_copy[0], min_max[1]);
                    j = k;
                }
            }
        }
        final_res = Arrays.copyOf(temp_res, count);
    }
    
    private Point[] min_max(Point[] points) {
        Point[] res = {points[0], points[0]};
        for (Point each : points) {
            if (res[0].compareTo(each) > 0)
                res[0] = each;
            if (res[1].compareTo(each) < 0)
                res[1] = each;
        }
        return res;
    }
```

这次的JAVA代码也出现了一些新的东西，即Comparable和Comparator接口，具体的不多说，看代码就能明白，这里只介绍其用途。Comparable接口表明该类可排序，类中必须要有一个comparTo方法，表明大小关系。当类可以有不止一种排序方法时就可以在类中构造一个新的Comparator类，可以通过调用Arrays.sort(a, comparator)的方法来对a数组进行排序。

另一个要注意的就是在数组作为形参传入时，由于数组其实是作为引用传入的，所以一定要注意代码是否无意改变了该数组（这个问题我一开始并没有在意，后来反应到了分数上，我困扰了很久查了论坛才找到原因）。这也是为什么在代码中先对数组进行一个拷贝的原因。同样的情况还反映在最后的返回结果上，由于final_res同样为数组，直接返回的也是一个引用，所以需要先进行一次拷贝再返回。

## Summary

这次作业写了我很久，主要是改了又改，也收获了一些东西。一个是排序的应用，我觉得可以归纳为当你要查找数组中某几项特定条件的值时，先通过该特定条件对数组进行排序总比你暴力for循环要快的多；其次是数组的引用，我花了很久的时间在解决这个引用问题上，算是对写代码中一定要注意的几个问题有了一个深刻印象。

下周作业：优先队列的应用

