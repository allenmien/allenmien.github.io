---
title:      自然语言处理中的TD-IDF、PageRank、TextRank
---
# TF-IDF、PageRank、TextRank

## TF-IDF

### 什么是TF-IDF

> 字词的重要性随着它在文件中出现的次数成正比增加，但同时会随着它在语料库中出现的频率成反比下降。

**词频 (term frequency, TF)** 指的是某一个给定的词语在该文件中出现的次数。

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-25/63738428.jpg)

**逆向文件频率 (inverse document frequency, IDF)** 

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-25/77978618.jpg)

**TF-IDF**

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-25/38449628.jpg)

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-25/91855146.jpg)

## PageRank

### 算法来源

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-25/40788749.jpg)

### 算法原理

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-25/71459610.jpg)

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-25/99304278.jpg)

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-25/30640781.jpg)

### 算法实现

```python
# -*-coding:utf-8-*-
"""
@Time   : 2018/7/25 9:23
@Author : Mark
@File   : page_rank.py
"""
# -*- coding: utf-8 -*-

from pygraph.classes.digraph import digraph


class PRIterator:
    __doc__ = '''计算一张图中的PR值'''

    def __init__(self, dg):
        self.damping_factor = 0.85  # 阻尼系数,即α
        self.max_iterations = 100  # 最大迭代次数
        self.min_delta = 0.00001  # 确定迭代是否结束的参数,即ϵ
        self.graph = dg

    def page_rank(self):
        #  先将图中没有出链的节点改为对所有节点都有出链
        for node in self.graph.nodes():
            if len(self.graph.neighbors(node)) == 0:
                for node2 in self.graph.nodes():
                    digraph.add_edge(self.graph, (node, node2))

        nodes = self.graph.nodes()
        graph_size = len(nodes)

        if graph_size == 0:
            return {}
        page_rank = dict.fromkeys(nodes, 1.0 / graph_size)  # 给每个节点赋予初始的PR值
        damping_value = (
                                1.0 - self.damping_factor) / graph_size  # 公式中的(1−α)/N部分

        flag = False
        for i in range(self.max_iterations):
            change = 0
            for node in nodes:
                rank = 0
                for incident_page in self.graph.incidents(node):  # 遍历所有“入射”的页面
                    rank += self.damping_factor * (
                            page_rank[incident_page] / len(
                        self.graph.neighbors(incident_page)))
                rank += damping_value
                change += abs(page_rank[node] - rank)  # 绝对值
                page_rank[node] = rank

            print("This is NO.%s iteration" % (i + 1))
            print(page_rank)

            if change < self.min_delta:
                flag = True
                break
        if flag:
            print("finished in %s iterations!" % node)
        else:
            print("finished out of 100 iterations!")
        return page_rank


if __name__ == '__main__':
    dg = digraph()

    dg.add_nodes(["A", "B", "C", "D", "E"])

    dg.add_edge(("A", "B"))
    dg.add_edge(("A", "C"))
    dg.add_edge(("A", "D"))
    dg.add_edge(("B", "D"))
    dg.add_edge(("C", "E"))
    dg.add_edge(("D", "E"))
    dg.add_edge(("B", "E"))
    dg.add_edge(("E", "A"))

    pr = PRIterator(dg)
    page_ranks = pr.page_rank()

    print("The final page rank is\n", page_ranks)
```

```python
This is NO.1 iteration
{'A': 0.2, 'C': 0.08666666666666667, 'B': 0.08666666666666667, 'E': 0.31050000000000005, 'D': 0.1235}
This is NO.2 iteration
{'A': 0.29392500000000005, 'C': 0.11327875000000001, 'B': 0.11327875000000001, 'E': 0.27940540625, 'D': 0.16142221875}
This is NO.3 iteration
{'A': 0.26749459531249997, 'C': 0.10579013533854167, 'B': 0.10579013533854167, 'E': 0.3020913084941407, 'D': 0.15075094285742188}
This is NO.4 iteration
{'A': 0.2867776122200196, 'C': 0.1112536567956722, 'B': 0.1112536567956722, 'E': 0.2999867138432907, 'D': 0.1585364609338329}
This is NO.5 iteration
{'A': 0.2849887067667971, 'C': 0.11074680025059253, 'B': 0.11074680025059253, 'E': 0.30595816211326343, 'D': 0.15781419035709435}
This is NO.6 iteration
{'A': 0.29006443779627394, 'C': 0.11218492404227762, 'B': 0.11218492404227762, 'E': 0.3071778399574342, 'D': 0.1598635167602456}
This is NO.7 iteration
{'A': 0.2911011639638191, 'C': 0.11247866312308208, 'B': 0.11247866312308208, 'E': 0.3092942847281384, 'D': 0.16028209495039195}
This is NO.8 iteration
{'A': 0.2929001420189177, 'C': 0.11298837357202668, 'B': 0.11298837357202668, 'E': 0.31029995701216717, 'D': 0.161008432340138}
This is NO.9 iteration
{'A': 0.29375496346034213, 'C': 0.11323057298043027, 'B': 0.11323057298043027, 'E': 0.31122614803916593, 'D': 0.16135356649711313}
This is NO.10 iteration
{'A': 0.29454222583329104, 'C': 0.11345363065276579, 'B': 0.11345363065276579, 'E': 0.3118039106048226, 'D': 0.16167142368019125}
This is NO.11 iteration
{'A': 0.2950333240140992, 'C': 0.1135927751373281, 'B': 0.1135927751373281, 'E': 0.3122514984282559, 'D': 0.16186970457069255}
This is NO.12 iteration
{'A': 0.29541377366401755, 'C': 0.11370056920480498, 'B': 0.11370056920480498, 'E': 0.312557474621215, 'D': 0.1620233111168471}
This is NO.13 iteration
{'A': 0.2956738534280328, 'C': 0.11377425847127598, 'B': 0.11377425847127598, 'E': 0.3127819940001969, 'D': 0.16212831832156827}
This is NO.14 iteration
{'A': 0.29586469490016737, 'C': 0.1138283302217141, 'B': 0.1138283302217141, 'E': 0.3129401916060185, 'D': 0.16220537056594256}
This is NO.15 iteration
{'A': 0.29599916286511574, 'C': 0.11386642947844947, 'B': 0.11386642947844947, 'E': 0.31305426256607427, 'D': 0.16225966200679048}
This is NO.16 iteration
{'A': 0.29609612318116313, 'C': 0.11389390156799623, 'B': 0.11389390156799623, 'E': 0.3131354372049671, 'D': 0.16229880973439462}
This is NO.17 iteration
{'A': 0.2961651216242221, 'C': 0.11391345112686294, 'B': 0.11391345112686294, 'E': 0.3131936384609857, 'D': 0.16232666785577968}
This is NO.18 iteration
{'A': 0.29621459269183786, 'C': 0.11392746792935407, 'B': 0.11392746792935407, 'E': 0.3132351892873392, 'D': 0.16234664179932953}
This is NO.19 iteration
{'A': 0.2962499108942383, 'C': 0.11393747475336752, 'B': 0.11393747475336752, 'E': 0.31326492583997373, 'D': 0.1623609015235487}
This is NO.20 iteration
{'A': 0.2962751869639777, 'C': 0.11394463630646035, 'B': 0.11394463630646035, 'E': 0.3132861775857534, 'D': 0.162371106736706}
This is NO.21 iteration
{'A': 0.2962932509478904, 'C': 0.11394975443523561, 'B': 0.11394975443523561, 'E': 0.31330137763112553, 'D': 0.16237840007021073}
This is NO.22 iteration
{'A': 0.29630617098645673, 'C': 0.11395341511282941, 'B': 0.11395341511282941, 'E': 0.31331224432853666, 'D': 0.1623836165357819}
This is NO.23 iteration
{'A': 0.2963154076792562, 'C': 0.11395603217578926, 'B': 0.11395603217578926, 'E': 0.31332001507954593, 'D': 0.16238734585049966}
This is NO.24 iteration
{'A': 0.29632201281761406, 'C': 0.11395790363165731, 'B': 0.11395790363165731, 'E': 0.3133255711032878, 'D': 0.16239001267511166}
This is NO.25 iteration
{'A': 0.29632673543779464, 'C': 0.11395924170737515, 'B': 0.11395924170737515, 'E': 0.31332954395074825, 'D': 0.1623919194330096}
This is NO.26 iteration
{'A': 0.29633011235813606, 'C': 0.11396019850147188, 'B': 0.11396019850147188, 'E': 0.31333238460743484, 'D': 0.16239328286459742}
finished in D iterations!
('The final page rank is\n', {'A': 0.29633011235813606, 'C': 0.11396019850147188, 'B': 0.11396019850147188, 'E': 0.31333238460743484, 'D': 0.16239328286459742})
```

## TextRank

### 原理

进行关键词提取时，TextRank算法思想和PageRank算法类似，不同的是，TextRank中时以词为节点，以共现关系建立起节点之间的链接。

- 分词，并去除停用词等噪音
- 对于每一个词，前后5个词，共10个作为和它link的词。但是比方说，“程序员”在文章中出现了两次，那就是和程序员这个词link的此为这10个词的并集
- 算法就类似PageRank，进行迭代，直至收敛

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/18-7-25/45854060.jpg)

### 算法实现

```python
# -*-coding:utf-8-*-
"""
@Time   : 2018/7/25 12:03
@Author : Mark
@File   : text_rank.py
"""
import sys

from pygraph.classes.digraph import digraph

reload(sys)
sys.setdefaultencoding('utf-8')


class PRIterator:
    __doc__ = '''计算一张图中的PR值'''

    def __init__(self, dg):
        self.damping_factor = 0.85  # 阻尼系数,即α
        self.max_iterations = 100  # 最大迭代次数
        self.min_delta = 0.00001  # 确定迭代是否结束的参数,即ϵ
        self.graph = dg

    def page_rank(self):
        #  先将图中没有出链的节点改为对所有节点都有出链
        for node in self.graph.nodes():
            if len(self.graph.neighbors(node)) == 0:
                for node2 in self.graph.nodes():
                    digraph.add_edge(self.graph, (node, node2))

        nodes = self.graph.nodes()
        graph_size = len(nodes)

        if graph_size == 0:
            return {}
        page_rank = dict.fromkeys(nodes, 1.0 / graph_size)  # 给每个节点赋予初始的PR值
        damping_value = (
                                1.0 - self.damping_factor) / graph_size  # 公式中的(1−α)/N部分

        flag = False
        for i in range(self.max_iterations):
            change = 0
            for node in nodes:
                rank = 0
                for incident_page in self.graph.incidents(node):  # 遍历所有“入射”的页面
                    rank += self.damping_factor * (
                            page_rank[incident_page] / len(
                        self.graph.neighbors(incident_page)))
                rank += damping_value
                change += abs(page_rank[node] - rank)  # 绝对值
                page_rank[node] = rank

            print("This is NO.%s iteration" % (i + 1))

            if change < self.min_delta:
                flag = True
                break
        if flag:
            print("finished in %s iterations!" % node)
        else:
            print("finished out of 100 iterations!")
        return page_rank


if __name__ == '__main__':
    dg = digraph()
    words_dict = {
        u"开发": [u"专业", u"程序员", u"维护", u"英文", u"程序", u"人员"],
        u"软件": [u"程序员", u"分为", u"界限", u"高级", u"中国", u"特别", u"人员"],
        u"程序员": [u"开发", u"软件", u"分析员", u"维护", u"系统", u"项目", u"经理", u"分为", u"英文", u"程序", u"专业", u"设计", u"高级", u"人员",
                 u"中国"],
        u"分析员": [u"程序员", u"系统", u"项目", u"经理", u"高级"],
        u"维护": [u"专业", u"开发", u"程序员", u"分为", u"英文", u"程序", u"人员"],
        u"系统": [u"程序员", u"分析员", u"项目", u"经理", u"分为", u"高级"],
        u"项目": [u"程序员", u"分析员", u"系统", u"经理", u"高级"],
        u"经理": [u"程序员", u"分析员", u"系统", u"项目"],
        u"分为": [u"专业", u"软件", u"设计", u"程序员", u"维护", u"系统", u"高级", u"程序", u"中国", u"特别", u"人员"],
        u"英文": [u"专业", u"开发", u"程序员", u"维护", u"程序"],
        u"程序": [u"专业", u"开发", u"设计", u"程序员", u"编码", u"维护", u"界限", u"分为", u"英文", u"特别", u"人员"],
        u"特别": [u"软件", u"编码", u"分为", u"界限", u"程序", u"中国", u"人员"],
        u"专业": [u"开发", u"程序员", u"维护", u"分为", u"英文", u"程序", u"人员"],
        u"设计": [u"程序员", u"编码", u"分为", u"程序", u"人员"],
        u"编码": [u"设计", u"界限", u"程序", u"中国", u"特别", u"人员"],
        u"界限": [u"软件", u"编码", u"程序", u"中国", u"特别", u"人员"],
        u"高级": [u"程序员", u"软件", u"分析员", u"系统", u"项目", u"分为", u"人员"],
        u"中国": [u"程序员", u"软件", u"编码", u"分为", u"界限", u"特别", u"人员"],
        u"人员": [u"开发", u"程序员", u"软件", u"维护", u"分为", u"程序", u"特别", u"专业", u"设计", u"编码", u"界限", u"高级", u"中国"]
    }

    words_key_list = list()
    for k, v in words_dict.iteritems():
        words_key_list.append(k)
    dg.add_nodes(words_key_list)

    for k, v in words_dict.iteritems():
        for i in v:
            dg.add_edge((k, i))

    pr = PRIterator(dg)
    page_ranks = pr.page_rank()

    sort_text_rank = sorted(page_ranks.items(), key=lambda d: d[1], reverse=True)
    for k, v in sort_text_rank:
        print str(k) + u":" + str(v)
    print sort_text_rank
```

```python
This is NO.1 iteration
This is NO.2 iteration
This is NO.3 iteration
This is NO.4 iteration
This is NO.5 iteration
This is NO.6 iteration
This is NO.7 iteration
This is NO.8 iteration
This is NO.9 iteration
This is NO.10 iteration
This is NO.11 iteration
This is NO.12 iteration
This is NO.13 iteration
This is NO.14 iteration
This is NO.15 iteration
This is NO.16 iteration
This is NO.17 iteration
This is NO.18 iteration
This is NO.19 iteration
This is NO.20 iteration
This is NO.21 iteration
finished in 编码 iterations!
程序员:0.101580333895
人员:0.0859679826211
分为:0.0740249633386
程序:0.0740144395754
高级:0.0514255757372
软件:0.0493472878223
中国:0.0492889214488
特别:0.049256278494
专业:0.0491852387529
维护:0.0491848961483
系统:0.0466872602095
编码:0.0436170552749
界限:0.0433726145516
开发:0.043302777825
项目:0.0406679129266
分析员:0.040667762853
英文:0.037449970767
设计:0.0368904751286
经理:0.034092209713
```



## 引用声明

[TF-IDF原理及使用](https://blog.csdn.net/zrc199021/article/details/53728499)

[PageRank算法--从原理到实现](https://blog.csdn.net/rubinorth/article/details/52215036)

[基于TextRank的关键词提取算法](https://blog.csdn.net/zhangf666/article/details/77841845)

