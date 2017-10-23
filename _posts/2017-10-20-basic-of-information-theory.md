---
layout: post
title: 信息论基础知识
---

整理并总结信息论中信息熵的相关概念。

## 1. 写在前面

其实之前对于信息论的理解只是停留在一些公式上，对其中深层次的内涵还处于不求甚解的状态，正好借自己巩固机器学习基础知识的契机来认认真真整理下相关内容。

本文参考资料：
1. [wikipedia: Entrop (information theory)](https://en.wikipedia.org/wiki/Entropy_(information_theory))
2. [wikipedia: Shannon's source coding theorem](https://en.wikipedia.org/wiki/Shannon%27s_source_coding_theorem)
3. [PRML](http://www.springer.com/us/book/9780387310732)
4. [Deep Learning](http://www.deeplearningbook.org/)
5. [统计自然语言处理（第二版）](https://book.douban.com/subject/25746399/)

## 2. 信息论中的信息

现在假设存在一个随机变量 $x$，我们想知道当观测到该随机变量某个确定值的时候我们能够得到多少信息。这里所说的信息量指的是概率上的“惊讶程度”（degree of surprise），而不是事件本身内容的含义，例如，有人告诉你明天太阳还从东方升起，你知道这一事件的概率几乎为 $1$，因此一点都不惊讶，从信息论的角度这条消息的传递的信息量几乎为 $0$。换句话说，当一个高概率的事件发生时我们能得到的信息量非常少，相反的当一个低概率的事件发生时我们会得到较多的信息量。

现在我们可以给出信息量一个稍微正式点的定义了：信息量 $h(x)$ 是依赖于概率分布 $p(x)$ 的单调函数。此外 $h(\cdot )$ 还要求，对于两个不相关的事件 $x$，$y$，我们能从这两个不相关事件中获得的信息量应该等于他们各自信息量的加和，也就是说，$h(x,y)=h(x)+h(y)$。同时，从统计学角度上看，$x$, $y$ 应该是相互独立的，也就是，$p(x, y) = p(x)p(y)$。考虑这两个要求，我们很自然的想到 $h(x)$ 应该是 $p(x)$ 的对数函数：

$$
h(x) = -\log_2p(x)
$$

其中，$h(x)$ 是在区间$(0, 1 \rbrack$上的单调减函数，公式中的负号保证了信息量是非负的。关于对数函数中底的选择是任意的，但是按照信息论中的惯例通常是以 $2$ 为底，这时 $h(x)$ 的单位为 `bits`，此外，以欧拉常数 $e$ 为底时 $h(x)$ 单位为 `nats`（natural units），以 $10$ 为底时 $h(x)$ 的单位为 `dits`（decimal digits）。最后可以简单验证一下之前提到的条件：

$$
\begin{align*}
h(x, y)=& \log_2p(x,y)\\
=& \log_2p(x)p(y)\\
=& \log_2p(x) + \log_2p(y)\\
=& h(x) + h(y)
\end{align*}
$$

## 3. 信息熵

在数据传输的过程中，平均信息量就是依赖于概率分布 $p(x)$ 的 $h(x)$ 的期望：

$$
\mathrm{H}(x) = \mathbb{E}[h(x)] = -\sum_xp(x)\log_2p(x)
$$

这个 $\mathrm{H}(x)$ 常被称作随机变量 $x$ 的<u>熵</u>（entrop）。注意由于 $\lim_{p\to 0}p\log_2p = 0$，所以当 $p(x)=0$ 时，$p(x)\log_2p(x) = 0$。我们可以通过抛硬币的事例加强对信息熵的理解，下图（图片来自维基百科）是抛掷一枚硬币时硬币出现正面（$x=1$）的不同概率对应的熵值：


<center><img src="https://upload.wikimedia.org/wikipedia/commons/2/22/Binary_entropy_plot.svg" align="middle"></center>

其中横轴是硬币出现正面的概率 $p(x=1)$，纵轴是熵值 $\mathrm{H}(x)$ 的大小，当硬币是是一枚公平的硬币，也就是 $p(x=1)=p(x=0)=\frac{1}{2}$ 时，抛掷一枚硬币的平均信息量（惊讶程度，或者说不确定程度）是最大，当任意一面的概率提高时人们就更容易预测下一次会出现正面还是反面。我们还可以稍作拓展，假设离散随机变量 $x$ 的 $n$ 个值是等概率的，则 $\mathrm{H}(x) = -n*\frac{1}{n}\log_2 \frac{1}{n} = \log_2 n$，这是随机变量 $x$ 能获得的最大熵值。

### 3.1 无损压缩

压缩技术在我们的生活中经常会用到，无损压缩指的是压缩后的数据经过解压能够获得全部的原始数据，并且两者含有的信息量相同。根据 Shannon 的源编码理论（Shannon's source coding theorem），当独立同分布（i.i.d）的随机变量数据流长度趋于无穷时，不存在不损失信息的情况下平均编码长度小于源数据信息熵的编码方式，换句话说，源数据信息熵给出了无损压缩的一个上界。举例来说，假设原始数据源中有 $8$ 个值 $\lbrace a, b, c, d, e, f, g, h\rbrace$，并且他们对应的概率值分别为 $(\frac{1}{2},\frac{1}{4},\frac{1}{8},\frac{1}{16},\frac{1}{64},\frac{1}{64},\frac{1}{64},\frac{1}{64})$，对应的信息熵为：

$$
\begin{align*}
\mathrm{H}(x) &= -\frac{1}{2}\log_2\frac{1}{2}-\frac{1}{4}\log_2\frac{1}{4}-\frac{1}{8}\log_2\frac{1}{8}-\frac{1}{16}\log_2\frac{1}{16}-\frac{1}{64}\log_2\frac{1}{64}-\frac{1}{64}\log_2\frac{1}{64}-\frac{1}{64}\log_2\frac{1}{64}-\frac{1}{64}\log_2\frac{1}{64}\\
&= 2 \text{ bits}
\end{align*}
$$

由于随机随机变量的分布不是等概率的，我们可以利用这一不均匀的分布设计一种尽量接近但是不小于源数据中信息熵的编码方式。直觉上看，概率越高的值出现的几率更大，我们更希望它的编码长度较小，例如下面的这个编码方式：`0`，`10`，`110`，`1110`，`111100`，`111101`，`111110`，`111111`。它的平均编码长度为：

$$
\text{average code length } = \frac{1}{2}\times 1+\frac{1}{4}\times 2+\frac{1}{8}\times 3+\frac{1}{16}\times 4+4\times\frac{1}{64}\times 6 = 2 \text{ bits}
$$

这一编码方式的平均编码长度和源数据随机变量的信息熵的值相等，注意此时已经不存在更短的编码方式，当我们试图去缩短某些编码时就会造成歧义，例如，试图将 $c$ 的编码从 `110` 缩短为 `11` 则 `1110` 既可以是 $d$ 的编码，又可以是 $c$ 和 $b$ 连接构成的编码，其他情况类似。

## 4. 相对熵

相对熵（relative entrop）又叫做 Kullback-Leibler 差异（Kullback-Leibler divergence），通常简称为 KL 散度。当我们有两个针对同一随机变量的概率分布 $p(x)$ 和 $q(x)$ 时，我们可以用 KL 散度来衡量这两个概率分布之间的不同程度：

$$
D_{\mathrm{KL}}(P||Q) = \mathbb{E}\bigg[\log_2\frac{P(x)}{Q(x)} \bigg] = \mathbb{E}[\log_2P(x)-\log_2Q(x)]
$$

在离散随机变量的情况下，KL 散度代表了，发送从概率分布 $P$ 产生的数据时，使用概率分布 $Q$ 产生的能够最小化平均编码长度的编码方式进行编码所需要的额外信息量。这句话原文如下：

> In the case of discrete variables, it is the extra amount of information (measured in bits if we use the base 2 logarithm, but in machine learning we usually use nats and the natural logarithm) needed to send a message containing symbols drawn from probability distribution $P$, when we use a code that was designed to minimize the length of messages drawn from probability distribution $Q$.

其实可以从前文中我们所说的无损压缩（无损压缩和不丢失信息的信息传输都是编码解码的过程）来理解这句话，其中的道理是相通的。首先我们对 KL 散度的公式做个变形：

$$
\begin{align*}
D_{\mathrm{KL}}(P||Q) &= \mathbb{E}[\log_2P(x)-\log_2Q(x)]\\
&= \mathbb{E}[-(-\log_2P(x))-\log_2Q(x)]\\
&= \mathbb{E}[-\log_2Q(x)]-\mathbb{E}\lbrack-\log_2P(x)\rbrack\\
&= \mathrm{H}(x, Q) - \mathrm{H}(x)
\end{align*}
$$

其中，$\mathrm{H}(x)$ 是依赖概率分布 $P$ 的信息熵，在无损压缩中我们定义其为源数据的信息熵，$\mathrm{H}(x, Q) = -\sum_xP(x)\log_2Q(x)$ 称为<u>交叉熵</u>。此外我们知道，无损压缩中编码所需的额外信息量 $D_{\mathrm{KL}}(\cdot) = \text{average code length } - \mathrm{H}(x)$，并且最短平均编码长度是由概率分布 $Q$ 产生的：

$$
\begin{align*}
\text{average code length } &= \frac{1}{2}\times 1+\frac{1}{4}\times 2+\frac{1}{8}\times 3+\frac{1}{16}\times 4+4\times\frac{1}{64}\times 6 \\
&= -\frac{1}{2}\log_2\frac{1}{2}-\dots-\frac{1}{64}\log_2\frac{1}{2^6}\\
&= \mathrm{H}(x, Q)\\
\end{align*}
$$

因此可以求得 $Q(x)=(\frac{1}{2},\frac{1}{2^2},\frac{1}{2^3},\frac{1}{2^4},\frac{1}{2^6},\frac{1}{2^6},\frac{1}{2^6},\frac{1}{2^6})= (\frac{1}{2},\frac{1}{4},\frac{1}{8},\frac{1}{16},\frac{1}{64},\frac{1}{64},\frac{1}{64},\frac{1}{64}) = P(x)$，可以看到概率分布 $Q$ 和概率分布 $P$ 相同，KL 散度为 0。

在上述推理过程中，我们还可以得到一个重要性质——KL 散度是非负的，因为根据 Shannon 的源编码理论 $\mathrm{H}(x) \leq \text{average code length }=\mathrm{H}(x, Q)$，所以 $D_{\mathrm{KL}}(P\Arrowvert Q) = \mathrm{H}(x,Q)-\mathrm{H}(x)\geq 0$。KL 散度因为用来表示分布之间的差异，并且是非负的，所以也被称为 KL 距离，但是 KL 散度并不是真正意义上的距离因为它是非对称的，换句话说，$D_{\mathrm{KL}}(P\Arrowvert Q)\not\equiv D_{\mathrm{KL}}(Q\Arrowvert P)$。

## 5. 总结

这篇博客试图将信息论中的信息、信息熵、香农源编码理论、KL 散度以及交叉熵等知识串联成一个整体，内容涉及的知识难度都不高，属于信息论中比较基础和关键的部分。本文在参考一些权威著作的基础上也添加了一些自己的理解，如有不足之处欢迎指正。
