---
typora-root-url: D:\suyako-to-be-coder\wechat public account\Algorithms\week(2)
layout: post
title: "Coursera Algorithms——week(2) Deques and Randomized Queues"
category: Algorithms
---

## Assignment

1. 一个双向栈，实现栈和队列功能的集合（即既可以从栈首增加或删除数据，也可以从栈尾增加或删除数据），时间复杂度要求O(1)；
2. 一个随机栈，在pop时随机弹出数据，时间复杂度要求O(1);
3. 一个客户端检测随机栈;

## Introduction

本周的作业比较简单，也就简单写一下。队列和栈是常见的数据类型，描述了一组对象的集合，在实际程序编写中均有着广泛的运用。两者的不同之处在于访问对象的先后顺序：队列为先进先出，对比生活中的实际例子就是先排队的能先办业务；栈为后进先出，就像整理桌面的时候需要先处理摞得最高的书。

对于队列与栈，我们希望它们至少有两个功能：在对象集合的尾端新增(**push**)一个对象&按照先进先出或者后进先出的顺序弹出(**pop**)对象。我们可以采用数组或者链表这两种数据结构来完成这两项功能，数组这里不赘述，简单介绍下链表（看到链表有想到区块链吗？其实现原理是类似的哦（￣▽￣））。

如下图所示，有三个节点，每一个节点包含了如下两个信息：节点内容本身&下一个节点的位置。当我们读取完该节点的内容后，就可以通过读取下一个节点的位置来跳转到下一个节点，然后就可以进行喜闻乐见的循环啦。所以，当我们想要描述一条链表的时候，我们只需要知道这条链表的第一个节点的位置就够了。

![](链表.png)

这里随便扯几句区块链，顾名思义，就是一堆区块组成的链表，不过这条链表的实现更像下面这张图。

![](区块链.png)

这是一条很长很长的链表，然后每过10分钟都还要生成一个新的节点，为了让节点不脱离原先的链表，所以在生成这个新的节点时需要让这个节点指向这条链表的末节点。为什么说区块链的内容没法篡改呢，因为节点是靠记录上一个节点的位置来指向上一个节点的，然而区块链里每个节点的位置都是靠**该节点**和**上一个节点**内容决定的......这就导致了，一旦**节点A**的内容被篡改，那么A的位置信息也变化了，A的下一个节点**B**没法再连上A，链表也就在A点断掉了。所以如果要篡改的就要把**B**也给改了，然而B一改C又连不上了......总之，要改的话就要把A之后的所有节点都给修改一遍，由于某种原因，这个地址的计算很耗时，所以区块链作为分布式记账系统虽然没有管理员，但是任何人都无法轻易修改其中的数据。

更具体的内容可以看[这里]()，我这里写的还是有不少硬伤的(比如这里的地址其实是hash值)，我就是想凑个乐子(⌒▽⌒)

回到队列和栈，我们现在用一条链表来表示这两种数据类型。对于栈来说，我们在每次**push**一个新的节点时，让这个新的节点指向原链表的第一个节点，在**pop**时，我们删除链表的第一个节点并返回这个节点的内容，新的链表将会从原链表的第二个节点开始算起。对于队列来说稍显麻烦，因为先进去的要先出来，所以我们还需要记录这条链表最后一个节点的位置。**push**的操作与栈相同，**pop**则是删除最后一个节点并返回该节点内容，新的链表将会在原链表的倒数第二个节点结束。

## Programming

来看题目，第一个双向栈很简单，刚讲队列的时候基本就已经实现了，把栈的功能再加上就好。

第二个随机栈麻烦一点，链表应该无法实现（因为pop要求效率O(1)，对于链表来说pop掉非头非尾的数据只能从第一个节点开始一步步循环直到找到该节点），所以这里用数组来实现。具体思路就是pop掉一个随机地址的数据后，用该数组最末尾的数据填充该地址，这样就能实现O(1)的要求了。

那个客户端就是让你来熟悉管道操作的， 这里跳过了。

双向栈代码如下：

```java
import java.util.NoSuchElementException;
import java.util.Iterator;
import edu.princeton.cs.algs4.StdOut;

/**
 * A generalization of a stack and a queue that supports adding and removing items
 * from either the front or the back of the data structure
 *
 * @author suyako
 */
public class Deque<Item> implements Iterable<Item> {
    private Node first;  // the first node
    private Node last;   // the last node
    private int N;       // size of nodes
    // construct class Node
    private class Node{
        Item item;       // current node
        Node next;       // point to next node
        Node pre;        // point to previous node
        public Node(Item e) {
            item = e;
        }
    }
    
    public Deque(){
        /* construct an empty deque */
        first = null;
        last = first;
        N = 0;
    }

    public boolean isEmpty(){
        /* is the deque empty */
        return first == null;
    }

    public int size(){
        /* return the number of items on the deque */
        return N;
    }

    public void addFirst(Item item){
        /* add the item to the front */
        if(item == null) {throw new IllegalArgumentException("Null is not supported in this structure.");}
        if(N == 0) {first =new Node(item); last = first; N++;}
        else{
            Node old_first = first;
            first = new Node(item);
            first.next = old_first;
            old_first.pre = first;
            N++;
        }
    }

    public void addLast(Item item){
        /* add the item to the end */
        if(item == null){throw new IllegalArgumentException("Null is not supported in this structure.");}
        if(N == 0){first =new Node(item); last = first; N++;}
        else{
            Node new_last = new Node(item);
            new_last.pre = last;
            last.next = new_last;
            last = new_last;
            N++;
        }
    }

    public Item removeFirst(){
        /* remove and return the item from the front*/
        if(isEmpty()) {throw new NoSuchElementException("deque is empty.");}
        Node res = first;
        if(N == 1) {first = null; last = null;}
        else {first = first.next; first.pre = null;}
        N--;
        return res.item;
    }

    public Item removeLast(){
        /* remove and return the item from the end */
        if(isEmpty()){throw new NoSuchElementException("deque is empty.");}
        Node res = last;
        if(N == 1){first = null; last = null;}
        else {last = last.pre; last.next = null;}
        N--;
        return res.item;
    }

    public Iterator<Item> iterator(){
        /* return an iterator over items in order from front to end */
        return new ListIterator();
    }

    private class ListIterator implements Iterator<Item>{
        private Node current = first;
        public boolean hasNext() { return current != null;}
        public void remove() {throw new UnsupportedOperationException("This operation is not supported.");}
        public Item next(){
            if(current == null){throw new NoSuchElementException("No more items to return");}
            Item item = current.item;
            current = current.next; 
            return item;
        }
    }
}
```

随机栈代码如下：

```java
import java.util.NoSuchElementException;
import java.util.Iterator;
import edu.princeton.cs.algs4.StdOut;
import edu.princeton.cs.algs4.StdRandom;

/**
 * A randomized queue is similar to data stack or queue, except that the item removed
 * is chosen uniformly at random from items in the data structure.
 *
 * @author suyako
 */
public class RandomizedQueue<Item> implements Iterable<Item>{
    private Item[] data;
    private int N;
    
    public RandomizedQueue(){ 
        /* construct an empty randomized queue */
        data = (Item[]) new Object[2];
        //data = Item[2];
        N = 0;
    }

    public boolean isEmpty(){
        /* is the randomized queue empty */
        return N == 0;
    }

    public int size(){
        /*return the number of items on the randomized queue */
        return N;
    }

    private void resize(int capacity){
        /* adjust the size of list when it reaches its capacity or 3/4 of its data has been removed */
        Item[] temp = (Item[]) new Object[capacity];
        for (int i = 0; i < N; i++)
            temp[i] = data[i];
        data = temp;
    }

    public void enqueue(Item item){
        /* add the item */
        if(item == null) throw new IllegalArgumentException();
        if(N == data.length) resize(2 * data.length);
        data[N++] = item;
    }

    public Item dequeue(){
        /* remove and return a random item */
        if(isEmpty()) throw new NoSuchElementException();
        int m = StdRandom.uniform(N);
        Item res = data[m];
        data[m] = data[--N];  // assign the last item to data[m]
        if (N > 0 && N == data.length / 4) resize(data.length / 2);
        return res;
    }

    public Item sample(){
        /* return a random item but do not remove it */
        if(isEmpty()) throw new NoSuchElementException();
        int m = StdRandom.uniform(N);
        return data[m];
    }

    public Iterator<Item> iterator(){
        /* return an independent iterator over items in random order*/
        return new RandomListIterator();
    }

    private class RandomListIterator implements Iterator<Item>{
        public boolean hasNext(){return N > 0;}
        public void remove(){throw new UnsupportedOperationException();}
        public Item next(){
            if(!hasNext()) throw new NoSuchElementException();
            return dequeue();
        }
    }

    public static void main(String[] args) { 
        RandomizedQueue<String> randomizedqueue = new RandomizedQueue<String>();
        String[] collection = {"A","B","C","D","E","F","G","H"};
        for(String s : collection)
            randomizedqueue.enqueue(s);
        for(String s : randomizedqueue)
            StdOut.println(s);
    }
}
```

这周的Java代码里出现了一些新特性，一个是泛型，就是`public class Deque<Item> implements Iterable<Item>`里尖括号中出现的`Item`，这个`Item`可以是`Integer, String`或者是自己创建的类等任何引用类型，泛型的作用在于代码复用，即不需要再对每个类型写相似功能的实现。

由于Java的一些遗留问题，使用泛型创建数组的时候不可以直接`new Item[N]`，只能用`(Item[] new Object[N])`代替，而且即使这样写了编译时也会报warning，但是不影响使用。

其次是数组的扩容问题，由于栈和队列不是固定长度的，需要自动调整容量。容量调整包括在数组满时扩容（这里调整为2倍），在数组里出现一定程度的null时减容（这里取的是3/4，为什么不取1/2？因为扩容与减容实际上都要复制一遍数组，非常耗时，取1/2的话如果在这个临界点上重复push与pop，就会造成数组重复扩容与减容）。

最后是接口，为了实现循环，在类声明后面都加上了`implements Iterable<Item>`，主要作用在于调用类内的`iterator`方法并返回一个迭代器（具体实现机理还需要更多实践来了解）。

## Score

把代码上传看一下这次的得分：

![](score.png)

![](error.png)

两分扣的地方我没看懂啥意思......跟迭代器有关，感觉应该是小问题，就是真心没看出问题在哪儿。

## Summary

这次作业挺简单的，但也收获了不少。首先是栈和队列的简单实现，还有就是Java里泛型和接口机制的一些运用。前两周以熟悉课程为主，下一周就开始正经的算法课啦(=・ω・=)