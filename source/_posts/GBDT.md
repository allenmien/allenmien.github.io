---
title: GBDT
---
# GBDT

## GBDT回归和分类的相同

1. 根据样本，预测初始值$F_0(x)$
2. 根据损失函数，求一阶导数负梯度，得到残差的计算公式，并根据$F_0{x}$计算残差。
3. 建立决策树，对残差进行回归。
4. 根据建立好的回归树，对残差进行预测$\gamma_{jm}$。
5. $F_1(x) = F_0(x) + \gamma_{jm}$

## GBDT回归和分类的不同

GBDT中的回归和分类思路都是相似度，都是利用损失函数的一阶导数计算残差，并对残差进行回归，并不断的更新预测的残差，进而得到预测值。

只是由于回归和分类的损失函数不同，导致求残差的方式不同，预测残差的方式也不同。

而根据下面的四点不同，GBDT在代码实现的时候，实际上用到都是同一个回归树。只是有两个地方分别会调用自己的方法：

1. 初始化参数。
2. 回归树建立好之后，残差值的预测。

### 损失函数不同

#### 回归

$$
Loss = \frac{1}{2} * (y_i - F(x_i))^2
$$

#### 分类

$$
p_i = \frac{1}{1 + e^{(-F_m(x_i))}}
$$


$$
Loss = - \{y_i logp_i + (1 - y_i)log(1-p_i)\}= -\{y_iF_m(x_i) - log(1 + e^{F_m(x_i)})\}
$$

### 初始化方式不同

#### 回归

$$
F_0(x) = \bar{y}
$$

#### 分类

$$
F_0(x) = log(\frac{\sum y_i}{\sum(1 - y_i)})
$$

### 负梯度（残差）的求值不同

#### 回归

$$
y_i - F_{m-1}(x_i)
$$

#### 分类

$$
y_i - \frac{1}{1 + e^{(-F_{m-1}(x_i))}} = y_i - p_{i}^{m-1}
$$

### 残差的预测值不同

#### 回归

$\tilde{y_i}$决策树左边(右边)的残差(1..n)，但是不同的分类模型残差不同，也就是不同的分类模型，负梯度不同。
$$
\gamma_{jm} = average(\tilde{y_i})
$$

#### 分类

$$
\gamma_{jm} = \frac{\sum_{i=1}^{N}\tilde y_i}{\sum_{i=1}^{N}\tilde (y_i - y_i) *(1 - y_i + y_i)}
$$

## 举例

### 回归

10个样本，每个样本一个维度，例如样本1：(1, 5,56)...

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-05-15-100110.jpg)

#### STEP 1

初始化初始值

$F_0(x_m) = avrage(\tilde y_i) = 7.307$

#### STEP 2

计算负梯度
$$
\tilde y_i = - \frac{\partial Loss}{F(x_i)} = (y_i - F_0(xi))
$$
![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-05-15-093145.jpg)

#### STEP 3

建立回归树，拟合负梯度，数据为上面的数据集。

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-05-15-093358.jpg)

#### STEP 4

预测残差值

$\gamma_{jm} = average(\tilde{y_i})$

$\gamma_{11} = \frac{\tilde y_1 + \tilde y_2 + \tilde y_3 + \tilde y_4 + \tilde y_5 + \tilde y_6 }{6} = -1.0703$

$\gamma_{21} = \frac{\tilde y_7 + \tilde y_8 + \tilde y_9 + \tilde y_10 }{4} = 1.6055$

#### STEP 5

$F_1(x) = F_0(x) + \gamma_{jm}$

$F_1(x_1) = F_0(x_1) + (-1.0703) = 7.307 - 1.0703 = 6.2367$

$F_2(x_2) = ……..$

…..

由此得到$F_2(x_i)$

第一轮的迭代结束，开始第二轮的迭代。。。。

### 分类

10个样本，每个样本一维特征，分类为0或者1，举例样本1：(1，0)

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-05-15-094225.jpg)

#### STEP 1

初始化初始值

$F_0(x) =log(\frac{\sum y_i}{\sum(1 - y_i)})$

$F_0(x) = log(\frac{4}{6}) = −0.4054$

#### STEP 2

计算负梯度

$y_i - \frac{1}{1 + e^{(-F_{m-1}(x_i))}}$

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-05-15-094654.jpg)

#### STEP 3

建立回归树，拟合梯度，数据为上面的数据集。

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-05-15-094712.jpg)

#### STEP 4

预测残差值

$\gamma_{jm} = \frac{\sum_{i=1}^{N}\tilde y_i}{\sum_{i=1}^{N}\tilde (y_i - y_i) *(1 - y_i + y_i)}$

$\gamma_{11} = \frac{\tilde y_1 + \tilde y_2 + \tilde y_3 + \tilde y_4 + \tilde y_5 + \tilde y_6 + \tilde y_7 + \tilde y_8}{(y_1 - \tilde y_1)*(1 - y_1 - \tilde{y_1}) + (y_2 - \tilde y_2)*(1 - y_2 - \tilde{y_2}) + (y_3 - \tilde y_3)*(1 - y_3 - \tilde{y_3}) + (y_4 - \tilde y_4)*(1 - y_4 - \tilde{y_4}) + (y_5 - \tilde y_5)*(1 - y_5 - \tilde{y_5}) + (y_6 - \tilde y_6)*(1 - y_6 - \tilde{y_6}) + (y_7 - \tilde y_7)*(1 - y_7 - \tilde{y_7}) + (y_8 - \tilde y_8)*(1 - y_8 - \tilde{y_8})}  \\ = \frac{-1.2}{1.92}= −0.625$

$\gamma_{21} = \frac{1.2}{0.48} = 2.5$

#### STEP 5

$F_1(x) = F_0(x) + \gamma_{jm}$

$F_1(x_1) = F_0(x_1) + (−0.625) = −0.4054 −0.625 = -1.03$

$F_2(x_2) = ……..$

…..

由此得到$F_2(x_i)$

第一轮的迭代结束，开始第二轮的迭代。。。。