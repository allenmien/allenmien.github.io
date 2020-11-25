---
title: Simhash
---
# Simhash

Simhash算法是为了解决文本相似性的。

## Simhash流程实现

- **1、分词**，把需要判断文本分词形成这个文章的特征单词。最后形成去掉噪音词的单词序列并为每个词加上权重，我们假设权重分为5个级别（1~5）。比如：“ 美国“51区”雇员称内部有9架飞碟，曾看见灰色外星人 ” ==> 分词后为 “ 美国（4） 51区（5） 雇员（3） 称（1） 内部（2） 有（1） 9架（3） 飞碟（5） 曾（1） 看见（3） 灰色（4） 外星人（5）”，括号里是代表单词在整个句子里重要程度，数字越大越重要。
- **2、hash**，通过hash算法把每个词变成hash值，比如“美国”通过hash算法计算为 100101,“51区”通过hash算法计算为 101011。这样我们的字符串就变成了一串串数字。
- **3、加权**，通过 2步骤的hash生成结果，需要按照单词的权重形成加权数字串，比如“美国”的hash值为“100101”，通过加权计算为“4 -4 -4 4 -4 4”；“51区”的hash值为“101011”，通过加权计算为 “ 5 -5 5 -5 5 5”。
- **4、合并**，把上面各个单词算出来的序列值累加，变成只有一个序列串。比如 “美国”的 “4 -4 -4 4 -4 4”，“51区”的 “ 5 -5 5 -5 5 5”， 把每一位进行累加， “4+5 -4+-5 -4+5 4+-5 -4+5 4+5” ==》 “9 -9 1 -1 1 9”。这里作为示例只算了两个单词的，真实计算需要把所有单词的序列串累加。
- **5、降维**，把4步算出来的 “9 -9 1 -1 1 9” 变成 0 1 串，形成我们最终的simhash签名。 如果每一位大于0 记为 1，小于0 记为 0。最后算出结果为：“1 0 1 0 1 1”。

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-04-15-091527.jpg)

## Simhash距离计算

比较两个simhash的01有多少个不同。

通过海明距离（Hamming distance）就可以计算出两个simhash到底相似不相似。举例如下： **1**01**01** 和 **0**01**10** 从第一位开始依次有第一位、第四、第五位不同，则海明距离为3。

## Simhash存储和索引

因为Simhash计算的是海量的数据，如果每个文章都是按照0101的去存储和比较的话，就要存很多完整长度的行，然后新来一条，都要和已经存储下来的每一行，去做位运算或者乘法运算之类的。

但是Simhash采用了一种很巧妙的方式。借鉴了HashMap中，对key做hash，新来的key去hash的table中寻址的方法。

**存储**：

1. 将一个64位的simhash签名拆分成4个16位的二进制码。（图上红色的16位）
2. 分别拿着4个16位二进制码查找当前对应位置上是否有元素。（放大后的16位）
3. 对应位置没有元素，直接追加到链表上；对应位置有则直接追加到链表尾端。（图上的 S1 — SN）

**查找**：

1. 将需要比较的simhash签名拆分成4个16位的二进制码。
2. 分别拿着4个16位二进制码每一个去查找simhash集合对应位置上是否有元素。
3. 如果有元素，则把链表拿出来顺序查找比较，直到simhash小于一定大小的值，整个过程完成。

**原理**：
借鉴hashmap算法找出可以hash的key值，因为我们使用的simhash是局部敏感哈希，这个算法的特点是只要相似的字符串只有个别的位数是有差别变化。那这样我们可以推断两个相似的文本，至少有16位的simhash是一样的。具体选择16位、8位、4位，大家根据自己的数据测试选择，虽然比较的位数越小越精准，但是空间会变大。分为4个16位段的存储空间是单独simhash存储空间的4倍。之前算出5000w数据是 382 Mb，扩大4倍1.5G左右，还可以接受。

假设我们库中有5000万(2^26)的文本， 每来一个新文本我们都需要做5000万次的汉明距离计算， 这个计算量还是比较大的，假设签名为64位。

考虑目标文档3位以内变化的签名有 C(64, 3) 4万多可能， 然后 4万 * 5000万的hash查询， 时间复杂度太大了

我们可以将库中已有的5000万文章的签名表切成4个表， 分别存储将64byte切开的ABCD四段中一段， 每进来一篇新文章，同样的分割为ABCD字段， 去与对应的表精确匹配(这个过程可以设置4个并发比较)， 取每个表join后剩下的文档ID， 大概每个表产出备选文档ID 有5000万/2^16 = 1000左右的量级，要找到5000万文档中和目标文档汉明距离小于3的文档集， 那么库中的文档至少ABCD有一段和目标文档完全匹配，这样我们最多会有 4 * 2 ^(26 -16) 篇备选文档， 大概4000多次汉明距离的计算就可以了。

simhash用于比较大的长文本, 可以取汉明距离为3， 对于短文本的效果比较差， 汉明距离可以取大一些进行相似过滤

![](https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-04-15-093842.jpg)