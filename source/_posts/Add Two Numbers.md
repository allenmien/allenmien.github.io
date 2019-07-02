---
layout:     post
title:      Add Two Numbers
subtitle:   Add Two Numbers
date:       2019-05-11
author:     Mark
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Add Two Numbers
---
# Add Two Numbers

## 题目

> 给出两个 **非空** 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 **逆序** 的方式存储的，并且它们的每个节点只能存储 **一位** 数字。
>
> 如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。
>
> 您可以假设除了数字 0 之外，这两个数都不会以 0 开头。
>
> **示例：**
>
> ```
> 输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
> 输出：7 -> 0 -> 8
> 原因：342 + 465 = 807
> ```

## 解法

```python
from typing import List

# Definition for singly-linked list.


class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None


class Solution:
    def addTwoNumbers(self, l1: ListNode, l2: ListNode) -> ListNode:
        carry = 0
        root = n = ListNode(0)
        while l1 or l2 or carry:
            v1 = v2 = 0
            if l1:
                v1 = l1.val
                l1 = l1.next
            if l2:
                v2 = l2.val
                l2 = l2.next
            carry, val = divmod(v1+v2+carry, 10)
            n.next = ListNode(val)
            n = n.next
        return root.next
```

## 释义

这个题目有一下几个点：

1. divmod函数是对参数x/y 取整和求余。代表的是数字想加中的逢10进1位。而carry中存的其实下一位有可能进的数，可以是0，也可以是1。
2. root和n的关系，其实是一个变量地址和内存地址的引用问题，下面有引用的示意图。
3. v1和v2初始化是为了防止，比方说下一轮的l1和l2都已经是None了，但是carry还不是0，这样就会陷入死循环。
4. if l1，if l2的判断是为了防止carry= 1, 而 l1 和 l2 = None，导致的l1.val 报错。 

## 图解



<iframe height=500 width=500 src="http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-05-11-add%20two%20Numbers.gif">

![](http://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-05-11-112638.jpg)