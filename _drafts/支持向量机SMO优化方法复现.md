---
layout: post
title: SMO in SVM
---

## ISSUES

> 支持向量机的损失函数的对偶形式如下所示：
> $$
> \max_{\alpha}\sum_{i = 1}^m\alpha_i-\frac{1}{2}\sum_{i=1}^m\sum_{j=1}^m\alpha_i\alpha_jy_iy_jx_i^Tx_j\\[1.2cm]
> \begin{align}
> s.t. 
> &\sum_{i=1}^m\alpha_iy_i=0\\
> &0\le \alpha_i \le C, \hspace{1cm} i=1,2,\dots,m.
> \end{align}
> $$
> 现要求对alpha进行优化。

## DESCRIPTION

这篇文章主要就是对Platt论文的翻译，可以直接看原文，简直不能更通俗易懂了。

首先简要回顾一下支持向量机的概念。如下图左所示，在一个线性二元可分的模型中，存在无数个超平面将两类样本分开，在所有这些超平面中，红色的那个在直观上处于“正中间”，看起来分割效果也最好，事实上这种直觉也是正确的。因为样本不可能包含所有的实际情况，而处于正中的超平面对噪声的容忍度是最好的，如下图右所示。



超平面在数学上可以表示为$w^Tx+b=0​$，其中w为法向量，b为位移项。在这里，因为w与b可以成比例放大或缩小，对于正样本，我们另$w^Tx+b\ge1​$，负样本则为$w^Tx+b\leq-1​$。同时因为正负样本自身的标记$y\in\lbrace-1,1\rbrace​$，所以正负样本最终表示为$(w^Tx_i+b)y_i\ge1​$。对于取到等号的样本我们称为支持向量（support vector），这也是支持向量机名称的由来。优化的最终目标就是要使两个异类支持向量到超平面的距离之和r最大，由点到平面的距离公式可得$r=\frac{2}{\|w\|}​$，因此最终优化目标转为:
$$
\begin{align}
&min\hspace{0.5cm} \frac{1}{2}\|w\|^2\\
&s.t.\hspace{0.5cm}y_i(w^Tx_i+b)\ge1,i=1,2,\dots,m.
\end{align}
$$




在这里想稍微花点笔墨写一下对偶公式及KKT条件，毕竟虐了我一周（而且现在也不敢说自己真懂了）。

首先由拉格朗日乘子法，原优化目标函数可以写成如下形式：
$$
min\hspace{0.5cm} L(w,b,\alpha)=\frac{1}{2}\|w\|^2+\sum_{i=1}^m\alpha_i(1-y_i(w^Tx_i+b_i))\\
\begin{align}
s.t. \hspace{0.5cm}
&1-y_i(w^Tx_i+b_i)\leq0,i=1,2,\dots,m\\
&\alpha_i\ge0\\
&\alpha_i(1-y_i(w^Tx_i+b_i))=0
\end{align}
$$
在这里后面的s.t.所附的条件就是KKT条件了，尽自己所能解释一下它的意思。

对于任意$i$，$y_i(w^Tx_i+b)\ge1$这个限制条件，我们可以分解为$y_i(w^Tx_i+b)>1$和$y_i(w^Tx_i+b)=1$两个约束条件。第一个约束条件对优化目标不起控制作用，相当于求$\frac{1}{2}\|w\|^2-0\cdot(1-y_i(w^Tx_i+b))$的最小值。而对于第二个约束条件，由拉格朗日乘子法我们可以构造无约束优化目标$\frac{1}{2}\|w\|^2-\alpha_i\cdot(1-y_i(w^Tx_i+b))$，