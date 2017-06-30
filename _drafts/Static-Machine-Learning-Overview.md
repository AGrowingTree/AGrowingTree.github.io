# 统计机器学习的策略

统计学习方法有三大要素：

​					方法=模型+策略+算法	

监督学习中，模型主要有两类：

+ 决策函数表示的非概率模型：$$\mathcal{F}=\{f\lvert Y=f(X)\}$$
+ 条件概率表示的概率模型：$$\mathcal{F}=\{P|P(Y|X)\}$$

其中 $\mathcal{F}$ 表示模型的假设空间。算法就是像 梯度下降， EM 算法等更新参数的方法。这些属于以后的酷炫内容，这里不再展开，本文主要总结策略方面的内容。

策略用来评估模型的好坏，我们可以用一个损失函数（loss function）来预测模型的犯错程度，记做 $L(Y, f(x))$。其中 $y_i$ 是 $x_i$ 的真实输出，$\hat{y}_i$ 是 $x_i$ 的预测输出。换句话说，损失函数提供了一种衡量真实输出与预测输出之间差异的方法。常用的损失函数有：

+ 0-1 损失函数（0-1 loss function）：$$L(Y,f(X))=\begin{cases}1,&Y=f(x)\\0,&Y\neq f(x)  \end{cases}$$
+ 平方差损失函数（quadratic loss function）：$$L(Y,f(X))==\left( Y-f(X)\right)^2$$
+ 绝对损失函数（absolute loss function）：$$L(Y,f(X))==| Y-f(x)|$$
+ 对数损失函数（logarithmic loss function）：$$L(Y,P(Y|X))=-\log{P(Y|X)}$$
+ 交叉熵损失函数（cross entropy function）：$$L(Y,f(X))=-Y\log f(X)$$

损失函数越小说明模型的效果越好，学习出来的分布越逼近随机变量 $(X,Y)$ 的真实分布 $P(X,Y)$ ，所以损失函数的期望就是：
$$
R_{exp}(f)=E_p[L(Y,f(x)]=\int_{\mathcal{X}\times\mathcal{Y}}L(y,f(x))P(y|x)\mathrm{d}x\mathrm{d}y
$$
其中$\mathcal{X}$ 表示输入空间 $\mathcal{Y}$ 表示输出空间 。$R_{exp}$ 称为风险函数（risk function），也就是模型 $f(x)$ 关于理论联合分布 $P(Y|X)$ 的平均损失。由于理论联合分布是未知的（实际上模型学习的目的就是来逼近理论联合分布），实际当中我们通常采用经验风险（empirical risk）衡量模型效果：
$$
R_{emp}(f)=\frac{1}{N}\sum^{N}_{i=1}L(y_i,f(x_i))
$$
$R_{exp}$ 是模型关于联合分布的风险函数，经验风险 $R_{emp}$ 是模型关于训练样本的评价损失。根据大数定律，当 $ N\rightarrow \infty$ 时 $R_{emp}$ 趋于 $R_{exp}$。经验风险最小化（empirical risk minimiztion， ERM）策略认为，经验风险最小的模型就是最优模型：
$$
\min_{f\in\mathcal{F}}\ \ \frac{1}{N}\sum^{N}_{i=1}L(y_i,f(x_i))
$$
在样本容量足够大时这一策略可以保证比较好的学习效果，但是样本量较小的时候就容易产生过拟合（over-fitting）现象。结构风险最小化（structure risk minimization，SRM）是为了防止过拟合提出的。结构风险记做 $R_{srm}$：
$$
R_{srm}(f)=\frac{1}{N}\sum^{N}_{i=1}L(y_i,f(x_i))+\lambda J(f)
$$
其中 $\lambda$ 是权衡系数， $J(f)$ 表示模型的复杂度。$J(f)$ 和模型的复杂程度成正比，模型越复杂 $J(f)$ 越大，反之 $J(f)$ 越小。结构风险最小化策略认为结构风险最小的模型：
$$
\min_{f\in\mathcal{F}}\ \ \frac{1}{N}\sum^{N}_{i=1}L(y_i,f(x_i))+\lambda J(f)
$$
监督学习问题变成了以经验风险或者结构风险为目标函数的最优化问题。正则化项可以选取参数向量的 $L_2$ 范数（或者 $L_1$ 范数）：
$$
L(w) =\frac{1}{N}\sum_{i=1}^{N}L(y_i,f(x_i))+\frac{\lambda}{2}\lVert w\lVert^2
$$
经验风险和模型复杂度都小的模型泛化能力较强，在未知数据的预测上效果更好。正则化符合奥卡姆剃刀（Occam's razor）原理。奥卡姆剃刀原理在模型选择问题上认为：在所有选择的模型中，能够很好地解释已知数据并且十分简单才是好的模型。