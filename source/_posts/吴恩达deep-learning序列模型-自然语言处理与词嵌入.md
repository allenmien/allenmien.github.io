---
title:      吴恩达deep-learning序列模型-自然语言处理与词嵌入
---
# 自然语言处理与词嵌入

## 词汇表征

### one-hot

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-22/67498506.jpg)

### word-embedding

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-22/135529.jpg)

### 可视化算法：t-SNE

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-22/90961.jpg)

## 使用词嵌入

## 词嵌入的特性

## 嵌入矩阵

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-23/17490042.jpg)

## 学习词嵌入

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-23/96693793.jpg)

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-23/44369892.jpg)

## Word2vec

### skip-gram

比方说，先选定一个词，orange。在orange的上下文为10的窗口内，随机选定一个词进行预测，假设这个词为“juice”。那么正确的答案就应该是juice。然后预测，在orange出现的概率下，除了orange之外的词出现的概率，并把这些结果输入到softmax函数，最后算这个softmax的损失函数。通过计算损失函数的SGD进行反向传播，进而影响softmax的参数，进而得到词向量。

计算慢，主要是softmax的分母要计算出所有的和选定的词之外的概率。

二分类可以缓解这个问题。

## 负采样

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-24/79303535.jpg)

负采样的方法

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-24/78291703.jpg)

## GloVe词向量

- 先找出一个上下文词C，和一个目标词T。
- 遍历训练集，得到T在上下文C的周围出现的次数。

  



![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-8-24/8973230.jpg)

## 情绪分类

情绪分类的一个很大的难点在于你没有非常多的标注样本。

使用词嵌入可以使用比较小的样本集也可以达到比较好的效果。

### 使用文本词向量平均值

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-26-034441.png)

#### 缺点

对于“Completely lacking in good taste, good service, and good ambience”  这种负面情绪的文本，但是包含很多正面词汇的情况，处理不佳。

### 使用RNN构建一个情绪分类器

many to one

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-26-034754.png)

## 词嵌入除偏

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-26-035530.png)

做法是把词分成像“grandmother”和“grandfather”这样的对性别没有偏见的词象限。

以及“babysister”和“doctor”这样的性别产生偏见的词象限。

由之前“he”、"she"、“male”、"female"等求差再求平均求出的性别的向量。

对于像“grandmother”和“grandfather”这样的词不用处理。

而对于“babysister”和"doctor"这样的会产生偏见的词，像性别这个纬度上投影，SVD，这样会缩减性别对他们的影响。

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-08-26-041210.png)

