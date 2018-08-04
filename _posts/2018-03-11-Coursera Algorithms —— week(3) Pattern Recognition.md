---
layout: post
title: "Coursera Algorithms —— week(3) Pattern Recognition"
category: Algorithms
---

## Assignment

本周作业地址：http://coursera.cs.princeton.edu/algs4/checklists/collinear.html

> 以下图为例，分别采用暴力循环及排序的方法，找出图中所有由四个或四个以上点构成的直线。
>
> ![](http://p4mu7u37x.bkt.clouddn.com/lines2.png)

## Introduction

计算机的广泛使用使得数据无处不在，而整理数据的第一步往往是排序。排序的重要性无需多言，面对各种各样的实际问题也催生了各类排序方法，这里简单概括一下几种常见的**基于比较**的排序方式（以对大小为n的数组`a[n]`进行从小到大的排序为例）。

* 选择排序

  遍历数组`a[0:n-1]`，找到最小值并记录其位置`i`，交换`a[0]`与`a[i]`，之后遍历数组`a[1:n-1]`，找到最小值并记录位置`j`，交换`a[1]`与`a[j]`......依此类推，直到将整个数组排序。

  易知不论数组原先的实际情况如何，都需要经过约**n^2**次比较及**n**次交换，因此时间复杂度为**O(n^2)**。

* 插入排序

  插入排序在数组中维护一个指针`i`，`a[:i]`是有序的，`a[i:]`是尚未进行排序的。

  现在我们观察`a[i]`，另`pivot = a[i]`，向左移动`i`直至`a[i] < pivot`，同时在该过程中另`a[i + 1] = a[i]`。整体过程相当于找到指针`j`，其中`a[j:i] > a[i]`，我们对`a[j:i]`整体右移一位，然后另`a[j] = pivot`即可。

  插入排序的时间复杂度与数组情况有关，如果为有序数组，则只需经过**n-1**次比较即可，时间复杂度为**O(n)**，如果为逆序数组，需要经过**n^2**次比较及**n^2**次交换，时间复杂度为**O(n^2)**。


* 归并排序

  自底向上的归并排序类似下图（这也是Java排序所采用的方法）

  ![1533286527372](http://p4mu7u37x.bkt.clouddn.com/mergesort.png)

  可以看到每次merge的两个子数组`a, b`都是排好序的，现在想让两个数组合并得到一个新的有序数组`c`，只要逐一进行比较然后将小的放入即可。

  关于时间复杂度，每一轮merge都要进行约n次比较，总共有lgn轮，因此最终时间复杂度为**O(nlgn)**。

* 快速排序

  另`pivot = a[0]`，快速排序在数组中维护了两个指针`i,j`，其中`a[1:i] < pivot`，`a[i:j] >= pivot`，`a[j:]`则是尚未排序的数组。观察`a[j]`，如果`a[j] < pivot`，我们交换`a[j], a[i++]`；反之`j++`即可。循环结束后交换`a[0], a[i - 1]`，此时可以确定pivot所在的索引即为`i - 1`。同时我们就得到了两个子数组`a[:i-1], a[i:]`，依次对这两个子数组重复快排即可。

  在数组倒序的最坏情况下，快排要比较$n^2/2$次，时间复杂度为O(n^2)，一般情况下可以参照这张图：

  ![1533285650130](http://p4mu7u37x.bkt.clouddn.com/quicksort.png)

  在每次左右两数组长度分为1:9的情况下，快排构成的树高最高为C*lg(n)，而树的每一层比较次数均小于等于n，因此时间复杂度为**O(nlgn)**

总结一下各个排序的基本情况。

| 方法     | 最优情况时间复杂度 | 最坏情况时间复杂度 | 平均时间复杂度 |
| -------- | :----------------: | :----------------: | :------------: |
| 选择排序 |       O(n^2)       |       O(n^2)       |     O(n^2)     |
| 插入排序 |        O(n)        |       O(n^2)       |     O(n^2)     |
| 归并排序 |      O(nlgn)       |      O(nlgn)       |    O(nlgn)     |
| 快速排序 |      O(nlgn)       |       O(n^2)       |    O(nlgn)     |

可以证明一个最理想的排序算法最坏情况下也至少需要经过**lg(n!)**次的比较，归并和快排已经相当接近这个下界了。

那么，以Java为例，在我们调用Arrays.sort()的过程中，其选择的是何种排序算法呢？这里我们来做一些源码分析。

```java
public static void sort(int[] a) {
    DualPivotQuicksort.sort(a, 0, a.length - 1, null, 0, 0);
}
```

可以看到Java选择的排序方法是DualPivotQuicksort，我们进一步看这个排序方法的源码。

```java
final class DualPivotQuicksort {
    
    private static final MAX_RUN_COUNT = 67;
    
    private static final int QUICKSORT_THRESHOLD = 286;
    
    // 处于简化考虑省掉了work，workBase, workLen三个参数，因为我们用到的快排是
    static void sort(int[] a, int left, int right) {
        // 如果选择排序的数组长度小于286，则采用快排（这里的快排与上述快排不同）
        if (right - left < QUICKSORT_THRESHOLD) {
            sort(a, left, right, true);
            return;
        }

        /*
         * Jdk9在这里首先判断原数组是否已较为有序，在这里有序的定义是
         * 原数组中连续上升或下降的子数组（这些子数组被称为run）的个数，
         * 如果run的个数大于等于67则认为原数组无序。
         *
         * run[]数列存储了每个run的头部索引，count记录了run的个数
         */
        int[] run = new int[MAX_RUN_COUNT + 1];
        int count = 0; run[0] = left;

        // 检查数组是否已较为有序
        for (int k = left; k < right; run[count] = k) {
            // 跳过头部相等的部分
            while (k < right && a[k] == a[k + 1])
                k++;
            if (k == right) break;
            if (a[k] < a[k + 1]) { // 上升子数组
                while (++k <= right && a[k - 1] <= a[k]);
            } else if (a[k] > a[k + 1]) { // 下降子数组
                while (++k <= right && a[k - 1] >= a[k]);
                // 对下降子数组变换转为上升子数组
                for (int lo = run[count] - 1, hi = k; ++lo < --hi; ) {
                    int t = a[lo]; a[lo] = a[hi]; a[hi] = t;
                }
            }

            // 对于转换后的下降子数组，如果其最小值大于等于其前的上升子数组，则这两个
            // 数组合并为一个上升子数组，同时count-1
            if (run[count] > left && a[run[count]] >= a[run[count] - 1]) {
                count--;
            }

            /*
             * run数量达到了67，认为原数组无序程度较大，采用快排
             */
            if (++count == MAX_RUN_COUNT) {
                sort(a, left, right, true);
                return;
            }
        }

        // run数组应该满足以下条件:
        //    run[0] = 0
        //    run[<last>] = right + 1; (terminator)

        if (count == 0) {
            // 数列整体相等
            return;
        } else if (count == 1 && run[count] > right) {
            // 单一的上升数组或者经过转换的单一的下降数组，注意run[count] > right这个条件
            return;
        }
        right++;
        if (run[count] < right) {
            // 边界条件处理，发生在最后一个run全部相等或者只有单独一个数的情况
            run[++count] = right;
        }

        // 之后的整体思想是归并，但是设计比较复杂，我在这里作了简化，不过不保证是对的
        // 创建一个临时数组b用来辅助归并
        int[] b = new int[right - left];

        // 归并
        for (int last; count > 1; count = last) {
            for (int k = (last = 0) + 2; k <= count; k += 2) {
                int hi = run[k], mi = run[k - 1];
                for (int i = run[k - 2], p = i, q = mi; i < hi; ++i) {
                    if (q >= hi || p < mi && a[p] <= a[q]) {  // 注意&&优先级高于||
                        b[i] = a[p++];
                    } else {
                        b[i] = a[q++];
                    }
                }
                run[++last] = hi;
            }
            // 当count为奇数时复制最后一个run到辅助数组b
            if ((count & 1) != 0) {
                for (int i = right, lo = run[count - 1]; --i >= lo;
                    b[i] = a[i]
                );
                run[++last] = right;
            }
            int[] t = a; a = b; b = t;  // 将数组a和辅助数组b交换
        }
    }
}
```

大致概括一下整体思路：

1. 如果数组长度小于286，直接快排，但是这里的快排与常规快排不同，用的是[双轴快排](http://www.codeblab.com/wp-content/uploads/2009/09/DualPivotQuicksort.pdf)，即在常规快排的基础上新增了一个pivot。由于要与两个pivot进行比较，双轴快排较单轴快排的比较总数会更多，但是由于物理上的一些原因，双轴快排会更快，具体分析可以见[这里](https://arxiv.org/pdf/1511.01138.pdf)；
2. 如果数组长度大于286，首先判断数组是否较为有序，如果较为有序则采用归并排序，反之则采用双轴快排，这部分具体见代码；

## Programming

没有特别要说明的地方，具体代码可参考[我的github](https://github.com/suyako-to-be-a-programmer/Algorithms-part1/tree/master/Pattarn%20Recognition)

## Summary

这里翻译下算法导论上列出的对排序重要性的几点说明：

1. 直接应用层面：如银行按支票号码对支票进行排序；
2. 程序的关键子程序：如绘制三维图形时需要根据我们的视角方向对画布上的物体进行排序，以此决定绘制顺序；
3. 对排序的研究催生了很多其它重要技术；
4. 可以通过对排序复杂度的分析来推导其它问题的复杂度；
5. 排序算法的研究暴露了很多工程问题（如双轴快排为什么比常规快排更快）；

