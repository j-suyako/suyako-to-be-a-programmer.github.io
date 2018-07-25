---
layout: post
title: "Coursera Algorithms —— week(1) union-find"
category: Algorithms
---

### Assignments

如下图所示是一个**8*8**方格的渗透模型，可以想象成像素游戏（比如泰拉瑞拉）中占了64个体积的土块，其中白色和蓝色的方块表示已经被挖空了。现在我们从上方倒足量的水，这时蓝色的即表示水在往下走时会经过的方块，白色表示虽然挖空但是不会有水经过的方块。我们要求的就是在一个对全实心土块**随机**挖方块的过程中，经过**多少次**挖取之后，水可以顺利地从土块中流出?

<img src="http://p4mu7u37x.bkt.clouddn.com/渗透模型.png" style="margin: 0 auto">



### Introduction

根据上述的题目要求，可以认为土块有两种状态：闭合及连通。乍看之下从闭合到连通需要的挖取次数似乎是不定的，不过大量的计算告诉我们这其中存在一个阈值*p*(~= 0.6 \* 方块数量)，当方块数量小于*p*时，土块不能连通，而一旦到达*p*值附近，则不论挖取过程如何，土块基本都能连通。所以，最后的任务即是写一个程序以期求出这个阈值*p*。

<img src="http://p4mu7u37x.bkt.clouddn.com/阈值.png" style="margin: 0 auto">

现在的问题还是比较复杂的，我们需要对它进一步简化以明确连通的意义。

<img src="http://p4mu7u37x.bkt.clouddn.com/virtual_block.png" style="margin: 0 auto; width: 50%">

如图所示，在上下方各加一个状态为空的虚拟方块，上方的虚拟方块被设定为与第一排所有的方块相邻，类似的，下方的虚拟方块被设定为与最后一排所有的方块相邻，这样，土块是否连通转换为这两个虚拟方块是否连通的问题。

有了连通的明确定义，我们需要对题目中的挖孔过程再作一点深入的分析。对于处于`[i, j]`位置的方格来说，当它被挖开时，如果`[i + 1, j]`位置的孔已经被挖开了，则我们可以认为`[i, j]`与`[i + 1, j]`相连。更简化点，将方格当作一个个无体积的点的话，我们可以画出一条线连接`[i, j]`与`[i + 1, j]`对应的点。随着我们画的线越来越多，点与点之间或直接或间接地连接了起来，这些点构成了一个集合。直到挖开某个方格为止，我们的两个虚拟方块所在的集合被合并，挖取过程结束，简化的具体过程见下图。

<img src="http://p4mu7u37x.bkt.clouddn.com/connection_procedure.png" style="margin: 0 auto">

一般的，对于检查两个子元素是否在同一个集合中及合并两个集合的问题，我们通常可以用并查集（课程的叫法为*weighted-quick-union-find*）来解决，这里对该算法从*quick-find*->*quick-union-find*->*weighted-quick-union-find*的演化作一个简单介绍。

#### quick-find

![](http://p4mu7u37x.bkt.clouddn.com/quick-find.jpg)

这里为简单起序列里只放3、4、8、9四个点，初始状态```id[i] = i```，表示各个点处于相互独立的状态，这里`i`对应0-9。在经过第一次的4、3连接后（注意连接顺序），我们让```id[3] = 4```，这样当我们想要查询3、4两个点是否连通时只要判断```id[3] == id[4]```就可以了，返回True表示连通，False表示不连通。接着我们尝试连接8、3，由于```id[8] != id[3]```，我们将id[8]的值改为4。接着再连接9、4，像前两步一样我们将id[4]改为9，但除了修改```id[4]```之外，我们还要修改```id[3]```和```id[8]```，因为3、8两个点是和4连通的，所以我们令```id[3] = 9， id[4] = 9```，对于程序来说就是循环一遍列表，判断```id[i] == 4```，返回True就修改```id[i] = 9```。这里就是quick-find主要的开销所在了，因为你要找到所有id为4的`i`只能对列表做一次循环（这里可能会问为什么不修改`id[9]`，这样就只修改一次了，但对于程序来说你要判断哪个修改的次数更少也一样要做一次循环）。最后我们连接8、9，由于两个id相同，就可以不做任何修改了。

整理一下上述步骤：

- 连接*p*,*q*，若```id[q] != id[p]```，则另```id[q] = id[p]```
- 对列表进行循环，对所有```id[i] == id[q]```，令```id[i] = id[p]```

我们来计算每一步连接的开销：

- 访问`id[p], id[q]`，**2**次访问数组开销
- 对列表循环，**n**次访问数组开销，这里**n**是数组的大小
- 对所有`id[i] == id[q]`，修改`id[i] = id[p]`，由于修改次数不定，其修改数组开销也不定，但平均下来必然与**n**正相关

综上，连接开销是O(n)级别的。

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

在连接9、4时不是另`root[4]`指向`root[9]`，而是另`root[9]`指向`root[4]`，树的高度就可以降低，连接所需的开销也得以减少，这种进阶的*quick-union*被称为*weighted-quick-union*。

具体做法就是新建一个size数组，初始化为1。在连接*pq*的过程中，假设*p*、*q*分属于两棵树，`root[p] = i, root[q] = j`，`size[i], size[j]`分别记录了两棵树的大小（这里记录高度也是可以的），比较`size[i], size[j]`就可以自己决定合并的顺序到底是`i->j`抑或`j->i`。不妨设`size[i] <= size[j]`，这时除了另`id[i] = j`外，还需要让`size[j] += size[i]`，意思就是两棵树合并的时候这棵树的root所在的size记录了这棵树的大小。

可以证明weight-quick-union方法构造的树的高度不会超过**log(n)**，具体证明可以自己去看这课。

计算开销：

- 上溯`root[p]`，最多上溯log(n)次
- 上溯`root[q]`，开销与`root[p]`类似
- 判断`root[p] == root[q]`，返回`False`则有修改一次id的开销与一次size的开销

综上，weighted-quick-union算法的复杂度为O(log(n))级别。



### Programming

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

因为实际上写起来还是比较容易的，这里就不放具体的代码了，具体可去我[github](https://github.com/suyako-to-be-a-programmer)查看。



### Testing

在上传到网站之前先进行一些本地测试

#### *p*

对**200*200**的土块进行约100次测试取平均值，得出的*p*值结果为0.592，标准差为0.01，与课程给的结果还是挺接近的。

![结果对比](http://p4mu7u37x.bkt.clouddn.com/结果对比.png)

#### 过程可视化

课程本身给了一些测试样本和可视化测试程序，我们来看一下输出动画。

<img src="http://p4mu7u37x.bkt.clouddn.com/可视化.gif" style="margin: 0 auto; width: 50%">

可以看到左下角有奇怪的地方

<img src="http://p4mu7u37x.bkt.clouddn.com/左下角.png" style="margin: 0 auto; width: 50%">

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

最直接的一个思路就是去掉最下方的虚拟方块，这样保证了isFull的正确性，至于连通判定改为对最后一行方块进行循环判定其与最上方的虚拟方块是否连接。写了一遍上传后发现通不过Timing测试（对**8\*8**土块来说，原来的判定一次的时间变成了判定八次），后来想到可以写两个UF的方法，第一个UF中没有最下方的虚拟方块，用于对isFull的判定，第二个UF中存在最下方的虚拟方块，用于对percolates的判定。

最后的输出动画如下：

<img src="http://p4mu7u37x.bkt.clouddn.com/修改后可视化.gif" style="margin: 0 auto; width: 50%">

#### 平台测试

暂时好像找不到其它问题了，来试试代码上传。

![](http://p4mu7u37x.bkt.clouddn.com/课程测试.png)

​

主要的扣分项还是在内存方面，因为用了两个UF的原因导致大了一倍的内存，不过暂时也想不到其它方法啦。
