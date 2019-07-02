---
title:      吴恩达deep-learning序列模型-循环序列模型
---
# 循环序列模型

## 为什么选择序列模型

## 数学符号

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-19-074302.jpg)

## 循环神经网络模型

如果采用传统的神经网络模型。

- 输入和输出在不同的样本中，可能不同
- 他的结果，不能共享
  - 比方在一篇文章中，识别出来“Harry”是一个人名，他并不能共享到其他的样本中，标准另一个样本中那个的“Harry”也是一个人名。

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-19-074844.png)

### 那么循环神经网络是怎么做的呢？

第一个样本输入神经网络，他会预测出，这是否是一个实体的结果。

循环神经网络做的是，当他读到第二个词的时候，他不仅仅是利用第二个词去预测结果。

他还会输入来自第一个词（时间步1）的信息。

具体来来讲，时间步1的激活值，会传递到时间步2。

### 吴恩达的循环神经网络的表示方法：

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-19-082133.png)

### 通常论文的循环神经网络的表示方法：

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-19-082251.jpg)

### 循环神经网络中的参数表示方法

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-19-091138.jpg)

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-19-091228.jpg)

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-19-091244.jpg)

## 通过时间的反向传播

###  损失函数

交叉熵

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-19-123442.jpg)

## 不同类型的循环神经网络

### many to many

#### 输入和输出相同

命名实体识别

上面介绍的RNN为many to many 类型，输入和输出的数量的相同的。但是还有其他的类型。

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-19-130136.png)

#### 输入和输出不同

机器翻译

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-19-125942.png)

### many to one

情感分析，输入一篇文本，但是输出是，这片文章是消极还是积极。

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-19-125314.jpg)

### one to one

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-19-125333.jpg)

### one to many

music generation

第一个层输出的结果也会喂给下一层，作为输入。

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-19-125402.jpg)

## 语言模型和序列生成

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-20/54017445.jpg)

## 对新序列采样

### 基于词汇

如果模型已经训练好了，那么能够从模型中得到什么呢？

一种非正式的方法就是进行一次新序列采样。

如下图，已经得到一个基于语料的RNN模型：

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-20/50406473.jpg)

我们的目的是生成一个新的单词序列，也就是一句话。

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-20/76449817.jpg)

### 基于字符

“Cat”词表会是[C,  a, t]

#### 优点

不必担心未出现的字符

#### 缺点

- 序列会非常长
- 句子之间的关系捕捉不是很好
- 计算起来成本比较高

趋势是，大多数的情况都是使用基于词汇的，但是特殊情况下会使用基于字符的。

基于字符会用在处理大量的未知词汇的应用。或者专有词汇的应用。

## 带有神经网络的梯度消失

### 梯度消失

例子：

1. The cat , which already ate ...... , was full.
2. The cats , which already ate ...... , were full.

上面的两个例子，cat和 was，cats和were  之间有长期的依赖关系。但是使用RNN很难建立这种长时间的联系。

因为训练很深的神经网络，会出现梯度消失的问题。

如果这是 一个非常深的神经网络，这个网络前向传播，再反向传播，从后面的输出误差得到的梯度，是很难传到前面的，很难影响到前面的权重的计算。

也就是说，一个y^3,其实是受他附件的影响比较大，但是对比较远的地方，影响就比较小。

最根本的原因是，一个输出的误差很难通过梯度下降的方法传播到 距离 他非常远的地方，并对他产生影响。

RNN的一个缺点就是不擅长处理长期依赖问题。

### 梯度爆炸

在反向传播的时候，不仅仅会出现梯度指数型消失的问题，还会出现梯度指数型爆炸的问题。

梯度爆炸会使得参数变得非常大，以至于神经网络参数崩溃。

#### 梯度修剪

设置一个阈值，如果梯度大于阈值，缩放梯度向量，保证不会特别大。

## GRU单元

门控循环单元 

由于细胞单元C的存在，使得GRU能够缓解反向传播过程中的梯度消失现象。

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-21/63555368.jpg)

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-21/39320879.jpg)

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-21/16930615.jpg)

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-21/38238640.jpg)

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-21/76496374.jpg)

## 长短期记忆（LSTM）

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-22/13942477.jpg)

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-22/9426287.jpg)

## 双向RNN

双向RNN不仅可以获取之前的信息，还可以获取未来的信息。

但是双向RNN的缺点在于，需要一个完整的序列。

对于语音识别，需要等到这句话说完，才可以进行BRNN。

但是对于自然语言处理，文本是非常适合用BRNN的。

### BRNN

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-22/18986231.jpg)

## 深层循环神经网络

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-22/66076549.jpg)

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-22/15083812.jpg)