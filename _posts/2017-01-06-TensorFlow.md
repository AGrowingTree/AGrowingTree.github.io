---
layout: post
title: TensorFlow 基本工作原理
categories: Deep-Learning
---



> 详情请参考[此处](https://www.oreilly.com/learning/hello-tensorflow)

## 前言

因为项目需要，我必须在短时间内学习 TensorFlow 然后训练处一个文本分类的卷积神经网络（CNN）。我给自己安排了这三个步骤：

1. 学习 TensorFlow 基本原理，不会的再查 [API](https://www.tensorflow.org/api_docs/)
2. 理解 CNN 的基本 内涵
3. 实现

有个官方的 [TensorFlow Turorials](https://www.tensorflow.org/tutorials/) 主要讲深度学习（可以配合 [Google 深度学习公开课](https://classroom.udacity.com/courses/ud730/lessons/6370362152/concepts/63798118150923#) 视频一起学），很值得一看。但是理解 TensorFlow 的基本工作原理应该再学习官方教程之前。

本文意义：
关于本文所讲的概念，在 API 中其实都有，但是他们是交织在一起的，本文希望将他们分开来，按照流程逐个来讲，希望能够更容易理解。

本文环境：

+ Python 2.7（Python 3 不影响）
+ Ipython 交互终端
+ TensorFlow 0.12 版本

确保自己安装了 [TensorFlow](https://www.tensorflow.org/get_started/os_setup)，推荐选择[清华镜像](https://mirrors.tuna.tsinghua.edu.cn/help/tensorflow/) 。

## 如何理解 TensorFlow

先说点题外话，卷积神经网络中的 “Pooling” 翻译成 “池化”，我不知道是谁翻译的，但是真的不敢恭维，就是这个翻译严重影响了我对 Pooling 的理解！！！人家叫 Pooling 就是水池啊？人家加上了 “ing” 你也知道这是动词化啊，改成池化？？？我一查翻译才发现 Pool 有动词形式，含义是“汇总，凑钱，凑，凑巴”，这才是 Pooling 的本意：压缩降维！翻译成紧凑化也行啊，对于卷积神经网络的理解请看 Google 深度学习公开课。

现在再来说 TensorFlow，`Tensor` 翻译过来就是张量的意思，[知乎的解释](https://www.zhihu.com/question/20695804)涉及物理也没太看明白，但是这句话吸引了我

> 什么是张量? 在数学家眼中, 张量已经被抽象成了线性变换. 

`Tensor` 是一种线性变换，再笼统一点，就是变换，Flow 就是流动的意思，不妨大胆猜测，TensorFlow 就是从输入层开始，到输出层的不断的变换（并不准确）。再来看关于 [Tensor 的 API](https://www.tensorflow.org/api_docs/python/framework/core_graph_data_structures#Tensor)

> A Tensor is a symbolic handle to one of the outputs of an Operation. 

`Tensor` 是经过一种操作（就是变换的意思，具体含义会在下文讲）后输出数据的抽象。那么在 TensorFlow 中，这种流动是怎样组织起来的呢？用 `Graph` 来组织。

## Graph

### 节点和操作

```powershell
In [1]: import tensorflow as tf
```
TensorFlow 非常棒的地方就在于，它会自动帮我们管理各种状态。导入之后，TensorFlow 会产生一个默认的 `Graph` ，但是是空的。

```powershell
In [2]: graph = tf.get_default_graph()

In [4]: graph.get_operations()
Out[4]: []
```

 `Graph` 中的节点也可以叫做**操作**（operations or ops），用来进行对输入数据的加工。上文提到的 `Tensor` 的第一个基本目的就是

+ 一个 `Tensor`（某操作的输出）可以被传递给另一个操作。在操作之间构建数据流的的链接。

其实就是对 `Graph` 中节点通过有向边进行数据流动的抽象。

现在我们就可以向 `Graph` 中添加一个简单的操作

```powershell
In [5]: input_value = tf.constant(1.0)

In [6]: graph.get_operations()[0].node_def
Out[6]:
name: "Const"
op: "Const"
attr {
  key: "dtype"
  value {
    type: DT_FLOAT
  }
}
attr {
  key: "value"
  value {
    tensor {
      dtype: DT_FLOAT
      tensor_shape {
      }
      float_val: 1.0
    }
  }
}
```

其中 `tf.constant(1.0)` 向 `Graph` 中添加了一个值为 `1.0` 的常数。通过`graph.get_operations()[0].node_def` 可以看到，实际上 `Graph` 中的节点并不是存储的 Python 的普通变量，而是一个看起来很复杂的对象。实际上这里 TensorFlow 内部使用的是 **Google Protocol Buffers**，

> Protocol Buffers 是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，很适合做数据存储或 RPC 数据交换格式。它可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。目前提供了 C++、Java、Python 三种语言的 API。

总的来说就是来优化数据传输的。这里输出的节点信息就是用 protocol buffers 来表示的。下面就来讲为什么要用这种方式表示。

### 为什么用 protocol buffers

这是官方教程中的[一段话](https://www.tensorflow.org/tutorials/mnist/pros/#deep-mnist-for-experts#computation_graph) 大意就是，在进行矩阵运算这种操作的时候，Python 的性能还是太差，所以实际做法都是用其他更加高效的语言进行外部计算然后返回结果。但是在 Python 调用其他语言，其他语言返回时候的数据传输同样是一笔巨大的开销。所以在 TensorFlow 中，用户先对 `Graph` 中的节点进行表示，TensorFlow 会调用外部的其他语言对整个 `Graph` 中节点进行统一的计算，这样来减少数据传输的巨大开销。

之前定义的变量名 `input_value` 是对该节点的一个引用

```powershell
In [7]: input_value
Out[7]: <tf.Tensor 'Const:0' shape=() dtype=float32>
```

我们可以看到这是一个32位的浮点常数，没有维度，名字是 `Const:0`。那么该如何让 `Graph` 中操作执行呢（虽然听起来执行一个常数操作有点奇怪）？就要用到我们接下来介绍的`tf.Session()`

## Session

首先给出 `Session` 的定义

> A class for running TensorFlow operations.

有点像 `main` 函数，对之前 `Graph` 中定义的所有操作执行。

```powershell
In [8]: sess = tf.Session()

In [9]: sess.run(input_value)
Out[9]: 1.0
```
关于 `Seesion.run()` 的参数详情请看 [API](https://www.tensorflow.org/api_docs/python/client/session_management#Session.run)。我们首先声明了一个 `Session` 类的对象，**然后运行整个 `Graph` **，返回 `input_value` 指向节点的结果。这是 Tensor 的第二个基本目的

+ 当一个 `Graph` 在 `Session` 中加载完之后，`Tensor` 的值可以被计算，通过把它传递给 `Session.run()`

## 总结

明白了 `Graph`，`Session`，`Tensor` 的含义之后再去学[TensorFlow Turorials](https://www.tensorflow.org/tutorials/)， 代码的基本框架就明白了。我认为有必要回头再读一下他们的 API，明白三者是如何联系的。

API 大放送

+ [Graph](https://www.tensorflow.org/versions/r0.11/api_docs/python/framework/core_graph_data_structures#Graph)
+ [Session](https://www.tensorflow.org/api_docs/python/client/session_management#Session)
+ [Tensor](https://www.tensorflow.org/api_docs/python/framework/core_graph_data_structures#Tensor)










