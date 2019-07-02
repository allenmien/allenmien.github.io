---
title: PCA主成分分析法
---
# PCA主成分分析法

## 实践

其实主成分分析法的实现方法非常明了，一步步的来就可以。但是想了解的更加透彻，知道为什么这样做是可以的，或许需要一步步的推导公式开始，先说实现方法。

### 数据

 假设我们得到的2维数据如下：

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-19/60861608.jpg)

### 第一步

分别求x和y的平均值，然后对于所有的样例，都减去对应的均值。这里x的均值是1.81，y的均值是1.91，那么一个样例减去均值后即为（0.69,0.49），得到

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-19/63568222.jpg)

### 第二步

求特征协方差矩阵

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-19/23469031.jpg)

 对角线上分别是x和y的方差，非对角线上是协方差。协方差是衡量两个变量同时变化的变化程度。

### 第三步

求协方差的特征值和特征向量

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-19/78770624.jpg)

### 第四步

将特征值按照从大到小的顺序排序，我们选择其中最大的那个，这里是1.28402771，对应的特征向量是![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-19/960789.jpg)

### 第五步

将样本点投影到选取的特征向量上。

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-19/85891351.jpg)

结束

### 对比

#### PCA之前：

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-19/55175392.jpg)

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-19/39768521.jpg)

#### PCA之后

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-19/94723949.jpg)

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-19/46775265.jpg)

其实PCA的本质就是减少坐标轴、旋转坐标轴，使得减少坐标轴之后、旋转坐标轴之后的数据特征的离散程度，尽可能的损失的少，尽可能的代表原来的数据的特征。

## 公式推导

### 放在前面

作为一些知识科普

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-07-19-081534.png)

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-07-19-082158.png)

### 开始推导

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-07-19-082302.png)

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-07-19-082322.png)

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-07-19-082412.png)

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-07-19-082507.png)

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2018-07-19-082528.png)

## 写在后面

### 引用

1.[主成分分析法详解](https://blog.csdn.net/wtq1993/article/details/51305236)

2.[【机器学习】主成分分析详解](https://blog.csdn.net/lyl771857509/article/details/79435402)