---
title: SVM推导
---
# SVM推导

## 定义

SVM的定义是在特征空间上间隔最大的线性分类器。

核函数，使它成为实质上的非线性分类器。

SVM的学习策略就是间隔最大化，形式化为求解一个凸二次规划问题，也等价于正则化的合页损失函数是最小化问题。

## 线性可分支持向量机

假设给定一个特征空间上的训练数据集：
$$
T = {(x_1, y_1),(x_2,y_2),...,(x_N,y_N)}
$$
其中，$x\epsilon R^n, y_i \epsilon \{+1, -1\}, i=1,2,3…,N$ ，假设数据集是线性可分的。

目标是在特征空间上，找到一个分隔超平面，能够将所有的点正确的分类。

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-04-02-070849.jpg)

假设超平面为：
$$
\omega \cdot x + b = 0
$$

### 几何间隔

对于给定的训练数据集$T$中任意一个点$x_i$，到超平面$(w, b)$的几何间隔为：

> 几何距离 = 点到超平面的距离*$y_i$

$$
\gamma_i= y_i(\frac{w \cdot x_i + b}{||w||})
$$

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-04-02-071157.jpg)

对于数据集$T$中所有的点中，到超平面的几何间隔的最小的值为：
$$
\gamma = \mathop{\min}_{i=1,….,N} \gamma_i = \min_{i=1,..N} y_i\frac{w \cdot x_i + b}{||w||}
$$

#### 约束1

如果该平面能够把所有的数据点正确划分，那么：
$$
w \cdot x_i + b > 0, y_i > 0 \\
w \cdot x_i + b < 0, y_i < 0
$$
也就是：
$$
y_i(w \cdot x_i + b) > 0
$$


#### 约束2

超平面参数$w, b$成比例的更改，不会影响几何间隔。此处为了推导方便，可以在**距离超平面几何距离最近**的这个点$x_i$上（**Support Vector**），让$w,  b$成比例的更改，使得$|y_i(w \cdot x_i + b) =1|$, 那么对于$r$就是在$x_i$这点上：

> 可以理解：无论$w,b$如何成比例的更改，该超平面$w \cdot x + b =0$还是该超平面，所以点到超平面的几何距离不会变化

$$
\gamma = \mathop{\min}_{i=1,….,N} \gamma_i = y_i\frac{w \cdot x_i + b}{||w||}
$$

如果数据集被正确划分，那么$y_i(w \cdot x_i +b) > 0$， 而且几何距离最小的点$x_i$，距离超平面的距离为$|y_i(w \cdot x_i + b) =1|$，也就是$y_i(w \cdot x_i +b) =1$。那么对于所有的点有$y_i(w \cdot x_i +b) >=1$。那么：
$$
r = \min_{i=1,..N} \gamma_i = \frac{|y_i(w \cdot x_i + b) |}{||w||} =  \frac{1}{||w||}, \ \ \ \ \ \ \ y_i(w \cdot x_i + b) >= 1
$$

### 间隔最大化

我们需要最大化 该点到超平面的几何间隔$\gamma​$，即：
$$
\max r = \max \frac{1}{||w||} = \min \frac{1}{2} {||w||}^2
$$

$$
y_i(w \cdot x_i + b) >= 1
$$

### 拉格朗日求解

#### 构造拉格朗日函数

SVM的目标函数为，有多个不等式约束条件的优化为题，其拉格朗日函数可以写为：
{% raw %}$$
L(\omega,b,\alpha) = \frac{{||w||^2}}{2} + \sum_{i=1}^{m}\alpha_i(1-y_i(\omega^Tx_i + b))
$${% endraw %}
其中$\omega, b$为SVM待求的目标参数，$\alpha$为构建拉格朗日的参数。

#### 对偶问题

现在的目标是：
$$
\min_{\omega,b}[\max_{\alpha:\alpha_j \ge 0}L(\omega,b,\alpha)]
$$
也就是保证每一个$\alpha \ge 0$的情况下，最小的$L(\omega, b, \alpha)$的$\omega, b$；

转化为对偶问题：
$$
\max_{\alpha:\alpha \ge 0}[\min_{\omega,b}L(\omega,b,\alpha)]
$$

#### 求解

1. 先求解$\min_{w, b}L(\omega,b,\alpha)$：

$$
\min_{\omega,b}L(\omega,b,\alpha) = \min_{\omega, b}[\frac{1}{2}||w||^2 + \sum_{i=1}^{m}\alpha_i(1- y_i(\omega^Tx_i + b))]
$$

分别对$\omega, b$求导：
$$
\frac{\partial L}{\partial \omega} = \omega - \sum_{i=1}^{m}\alpha_iy_ix_i = 0 \rightarrow \omega = \sum_{i=1}^{m}\alpha_iy_ix_i
$$

$$
\frac{\partial L}{\partial b} = - \sum_{i=1}^{m}\alpha_iy_i = 0 \rightarrow \sum_{i=1}^{m}\alpha_iy_i = 0
$$

代入拉格朗日函数：
$$
\min_{\omega,b} L(\omega, b, \alpha) = \frac{1}{2}\Big [\sum_{i=1}^{m}\alpha_iy_ix_i \Big]^T \Big[\sum_{i=1}^{m}\alpha_iy_ix_i\Big] + \sum_{i=1}^m \alpha_i - (\sum_{i=1}^{m}\alpha_ix_iy_i)(\sum_{j=1}^{m}\alpha_jy_jx_j^T) -\sum_{i=1}^{m}\alpha_i y_i b \\ = \frac{1}{2}\sum_{i=1}^{m}\sum_{j=1}^{m}\alpha_ix_iy_i\alpha_jy_jx_j^T+ \sum_{i=1}^m \alpha_i - \sum_{i=1}^{m}\sum_{j=1}^{m}\alpha_ix_iy_i\alpha_jy_jx_j^T \\= \sum_{i=1}^m \alpha_i - \frac{1}{2}\sum_{i=1}^{m}\sum_{j=1}^{m}\alpha_i\alpha_jy_iy_jx_i^Tx_j
$$

2. 求解$\max _{\alpha : \alpha \ge 0}\sum_{i=1}^m \alpha_i - \frac{1}{2}\sum_{i=1}^{m}\sum_{j=1}^{m}\alpha_i\alpha_jy_iy_jx_i^Tx_j​$

且：$\sum_{i=1}^{m}\alpha_iy_i = 0$
$$
N(\alpha_i, \alpha_j) = \sum_{i=1}^m \alpha_i - \frac{1}{2}\sum_{i=1}^{m}\sum_{j=1}^{m}\alpha_i\alpha_jy_iy_jx_i^Tx_j
$$
对$\alpha_i, \alpha_j$分别求导：
$$
\frac{\partial N}{\partial \alpha_j} = \sum_{i=1}^{m}\alpha_i-\frac{1}{2}\sum_{i=1}^{m}\alpha_iy_iy_jx_i^Tx_j = 0 
$$

$$
\frac{\partial N}{\partial \alpha_i} = -\frac{1}{2}\sum_{j=1}^{m}\alpha_jy_iy_jx_i^Tx_j = 0
$$

可以分别求得$\alpha_i, \alpha_j$ ，把求得的$\alpha_i, \alpha_j$，带入$\omega = \sum_{i=1}^{m}\alpha_iy_ix_i$ ，就可以求得最大几何间隔的$\omega$了。

#### 拉格朗日函数求解示例

优化目标函数：
$$
\min_x f(x) = x_i^2 + x_j^2
$$
约束条件：
$$
h(x) = x_1 - x_2 -2 = 0
$$

$$
g(x) = (x_1 -2)^2 + x_2^2 -1 \ge 0
$$

构建拉格朗日函数：
$$
L(x,\alpha, \beta) = (x_1^2 + x_2^2) + \alpha(x_1 - x_2 -2) + \beta \Big[(x_1 -2)^2 + x_2^2 -1 \Big]
$$
先对$x_1, x_2$求导：
$$
\frac{\partial L}{\partial x_1} = 2x_1 + \alpha + 2 \beta (x_1-2) = 0 \rightarrow x_1 = \frac{4\beta -\alpha}{2\beta + 2}
$$

$$
\frac{\partial L}{\partial x_2} = 2x_2 -  \alpha + 2\beta x_2 = 0 \rightarrow x_2 = \frac{\alpha}{2\beta +2}
$$

带入$L(x,\alpha, \beta)$：
$$
L(x, \alpha, \beta) = - \frac{\alpha^2 + 4\alpha + 2\beta^2 -6\beta}{2\beta + 1}
$$
再分别对$\alpha, \beta$求导：
$$
\frac{\partial L}{\partial\alpha}
$$

$$
\frac{\partial L}{\partial \beta}
$$

得到：
$$
\alpha = -2, \beta = \sqrt{2} -1
$$
带入$x_1, x_2$得：
$$
x_1 = 2 - \frac{\sqrt{2}}{2} , x_2 = - \frac{\sqrt{2}}{2}
$$

## 软间隔和松弛变量

前面的推导是基于数据是线性可分的，但是实际情况是，数据可能并不是线性可分到。

但是这也分两种：第一种，数据点分布确实很复杂，需要通过核函数映射到高维空间，才能在高维空间可分；第二种，有一些噪点导致本来线性可分的数据，现在不可分了。

解决第二种问题点办法就是引入松弛变量。

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-04-08-070205.jpg)

图中蓝色的点就是噪音，导致本可以分开的数据点，现在不可分了。

我们之前线性可分的约束条件为：
$$
y_i(w^T x_i + b) \ge 1
$$
现在我们给他加一个松弛变量：
$$
y_i(w^T x_i + b) \ge 1 - \xi_i
$$
同样，目标函数变为:
$$
\min \frac{1}{2}||w||^2 + C\sum_{i=1}^{n}\xi_i
$$
拉格朗日函数为:
$$
L(\omega, b, \xi, \alpha, r) = \frac{1}{2}||w||^2 + C\sum_{i=1}^{n}\xi_i - \sum_{i=1}^{n}\alpha_i(y_i(\omega^Tx_i + b) - 1 + \xi_i) - \sum_{i=1}{n}r_i\xi_i
$$
同样对$\omega, b, \xi$求导：
$$
\frac{\partial L}{\partial \omega} = 0 \to \omega = \sum_i^{n}\alpha_ix_iy_i
$$

$$
\frac{\partial L}{\partial b} = 0 \to \sum_{i=1}^{n}\alpha_iy_i = 0
$$

$$
\frac{\partial L}{\partial \xi_i} = 0  \to  C-\alpha_i -r_i = 0
$$

带入L得：
$$
L(\omega, b, \xi, \alpha, r) = \sum_{i=1}^{n}\alpha_i -\frac{1}{2}\alpha_i\alpha_jy_iy_j<x_i,x_j>
$$

$$
0 \le \alpha_i \le C, i =1,...n
$$

$$
\sum_{i=1}^{n}\alpha_iy_i = 0
$$

其中C为控制"寻找margin最大的超平面"和"保证数据点偏差最小"之间的权重，自己定义。

