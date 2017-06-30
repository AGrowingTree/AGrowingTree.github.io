---
layout: post
title: 中国剩余定理与扩展欧几里得算法
categories: Algorithms
---

## 前言

零零碎碎的消耗了不少时间，终于把**中国剩余定理**以及**扩展欧几里得**算法弄明白了。这篇博客主要整理学习过程中的遇到的证明问题。


## 中国剩余定理

> 本节主要参考[中文维基百科](https://zh.wikipedia.org/wiki/%E4%B8%AD%E5%9B%BD%E5%89%A9%E4%BD%99%E5%AE%9A%E7%90%86)
### 简要介绍

中国剩余定理主要用来求解一元线性同余方程组。一元线性同余方程组可以表示为：

\begin{equation}
S = 
\begin{cases}
&x \equiv a_1 (mod m_1) \\\\
&x \equiv a_2 (mod m_2) \\\\
&\vdots \\\\
&x \equiv a_n (mod	m_n)
\end{cases}
\end{equation}

公式$(1)$引用



