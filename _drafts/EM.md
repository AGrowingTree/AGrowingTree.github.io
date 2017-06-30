# 从 k-means 到 EM 算法

## 写在前面



## 高斯混合分布与 EM 算法

假设我们对下图数据进行聚类，由于 k-means 采用的欧氏距离，如果使用 k-means 模型进行聚类效果会非常差。实际上图中的样本点大致上服从多个高斯分布。如果能够求解出所有高斯分布概率密度函数的参数，$\mu$（均值）和 $\Sigma$（协方差），那么就能确定样本属于某一高斯分布的概率，然后选取使样本概率最大的高斯分布作为样本类别。所以该聚类问题转化为，如何利用已知数据学习高斯分布参数的问题。

![](/Users/Demon/Resume/GitRepo/hychn.github.io/img/em-gaussian.png)

我们用 $\{x^{(1)},\dots x^{(m)}\}$ 表示数据集。另外再引入一个变量 $z^{(i)}$，令$z^{(i)}\backsim \mathrm{Multinomial(\phi)}$（其中 $\phi_j \geq 0, \sum_{j=1}^{k}\phi_j=1$，参数$\phi_j$ 给出$p(z^{(i)}=j)，$），$x^{(i)}|z^{(i)}=j\backsim\mathcal{N}(\mu_j,\Sigma_j)$。也就是说，我们假设 $x^{(i)}$ 是根据 $z^{(i)}$ 从某个高斯分布中生成的（高斯分布的数量为 $k$），$p(z^{(i)}=j)$ 表示 $x^{(i)}$ 属于第 $j$ 个高斯分布的概率。这就是高斯混合模型，变量 $z^{(i)}$ 就是隐含变量。

为了求解问题，我们首先写出似然函数：
$$
\begin{align}
\ell(\phi,\mu,\Sigma)=\log&\sum_{i=1}^mp(x^{(i)};\phi,\mu,\Sigma)\\
=&\sum_{i=1}^m\log\sum_{j=1}^kp(x^{(i)}|z^{(i)};\mu,\Sigma)p(z^{(i)};\phi)
\end{align}
$$
我们对这个似然函数直接求导会很困难，因为 $\log()$ 中包含的是多项和而不是多项的乘积。但是如果我们确定 $z^{(i)}$ 以后：
$$
\ell(\phi,\mu,\Sigma)=\sum_{i=1}^m\log p(x^{(i)}|z^{(i)};\mu,\Sigma)+\log p(z^{(i)};\phi)
$$
这个似然函数就很容易求导了
$$
\begin{align}
\phi_j&=\frac{1}{m}\sum^{m}_{i=1}1\{z^{(i)}=j\},\\
\mu_j&=\frac{\sum^m_{i=1}1\{z^{(i)}=j\}x^{(i)}}{\sum^m_{i=1}1\{z^{(i)}=j\}}\\
\Sigma_j&=\frac{\sum^m_{i=1}1\{z^{(i)}=j\}(x^{(i)}-\mu_j)(x^{(i)}-\mu_j)^T}{\sum^m_{i=1}1\{z^{(i)}=j\}}\\
\end{align}
$$
当确定 $z^{(i)}$ 时，多项式分布可以看做是伯努利分布。这里只对 $\mu_j$ 的求导进行证明，其他证明同理：
$$

$$

我们假定已知 $z^{(i)}$ 来简化求导，但是实际上 $z^{(i)}$ 是不知道的。要解决这个问题需要分两步：E-step 和 M-step。E-step 用来“猜测” $z^{(i)}$ 的取值，M-step 更新参数。

>Repeat until convergence: {
>
>>E-step, for each i, j:
>>
>>> $w^{(i)}_j$
>
>
>
>}

很像 k-means 对不对？k-means 算法的第一步，根据样本到不同中心点的距离，选取最近的中心点作为样本类别；第二部，中心点根据被它标记过的样本更新自己的位置。高斯混合分布的 EM 算法中的 E-step 实际上是分别计算出样本  $x^{(1)}$ 从 $k$ 个不同的高斯分布中产的概率。