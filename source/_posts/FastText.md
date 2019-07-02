---
title: FastText
---
# FastText

## 神经网络结构

本质上还是一个三层的DNN，但是和word2vec哪里不同呢？

一层层的看：

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-04-04-085720.jpg)

### 输入层

输入层

#### 字符

一个序列的字符Bag of words。

例如："我爱天安门"。

输入就是："我"，"爱"，"天"，"安"，"门"，五个词的one-hot向量。

#### N-Gram

N-Gram特征向量的原因是为了弥补Bag of Words词语顺序信息丢失的弊端。一般取2-Gram。

例如："我爱天安门"

输入："我爱"，"爱天"，"天安"，"安门"，四个词对的one-hot向量。

#### 加平均

也就是说，fasttext中词表不再仅仅是，字符的个数，还包涵所有的2-Gram，并一起取one-hot。当然2-Gram有可能非常大，我们会在第二层做hash分桶。

得到字符和N-Gram的向量一直，加在一起取平均就可以了。

这样对于一个文本来讲，输入层的向量表示为：1\*20000

### 隐藏层

隐藏层还是作为映射层。权重矩阵的大小为：20000\*300。

所以经过隐藏层得到的是一个：1\*300的向量。

但是隐藏层这里fasttext为了加速运算做了一个trick。对于2-gram 相似的Hash，使用同一个wordvec。

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-04-04-090634.jpg)

### 分类层

分类层还是使用了Huffman Tree。具体结构如下：

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-04-04-090725.jpg)

对于从隐藏层得到了任何一个向量：1\*300。与Huffman Tree中的正确的(比方说"cat")路径中经过的每一个内接点相乘，取softmax，并叠层。

那么对于任何一个内结点，都存在一个300\*1的向量。

拿1结点来讲，在给定context的条件下，向右走的概率为(这里每一个内结点的softmax，其实就是左右两种情况)：

即：
$$
P(1_{right}|context) = softmax((1*300)*(300*1) + b1)
$$
那么向左走的概率为:
$$
1 - P(1_{right}|context)
$$
如果我们的目标是cat ，那么概率为:
$$
P('cat'|context) = (1 - P(1_{right}|context)) \cdot P(2_{right}|context) \cdot P(5_{right}|context)
$$

## Why Fast

我们看$(1*300)*(300*1)$，其实$(1*300)$的意思是对于Tree中的每一个内接点，第二层中的300个神经元都与他是连接的。而$(300*1)$只是权重的个数而已。

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-04-02-100320.jpg)

对于传统的softmax来讲，隐藏层到分类层到参数个数为300*20000。这些参数都需要更新的话，简直没有办法计算了。

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-04-04-090725.jpg)

但是对于Huffman Tree来讲，拿cat举例，我只需要更新。

$(1-P(1_{right}|context)) \cdot P(2_{right}|context)  \cdot P(5_{right}|context) ——>1$  

就好了。

也就是参数为：300\*300\*3。

说到底是训练每一个二叉树的分类能力，而与具体的每个词不相关了。