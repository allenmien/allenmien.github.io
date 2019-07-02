---
title:      Two Sum
---
# Two Sum

## 题目

> 给定一个整数数组和一个目标值，找出数组中和为目标值的两个数。
>
> 你可以假设每个输入只对应一种答案，且同样的元素不能被重复利用。
>
> 示例:
>
> 给定 nums = [2, 7, 11, 15], target = 9
>
> 因为 nums[0] + nums[1] = 2 + 7 = 9  
> 所以返回 [0, 1]

## 解法

```python
from typing import List

class Solution(object):
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        record = dict()
        for i in range(len(nums)):
            n = target - nums[i]
            if n in record.keys():
                return[record[n], i]
            record[nums[i]] = i


nums = [2, 7, 11, 15]
target = 9
print(Solution().twoSum(nums, target))
```

## 解析

这道题的思路是用空间换时间。

第一轮对nums的循环是不可避免的。

这个算法高效的原因在于对第一轮对每一次循环的结果，进行了保存，并没有浪费掉消耗的时间。

假设我们有一个record dict , key为循环的该值，value为该值的索引。

每一次的循环，我们先用target - 当前值，去record里找有没有key，找到就返回。

模拟一下：

第一轮：record = {}, i=0, n = 7, 在record中不存在，则record[2]= 0

第二轮：record = {2:0}, i=1, n= 2, 在record中存在，则 return [record[2], 2] = [0, 1]

**图解如下**

<iframe height=500 width=500 src="https://markdocpicture.oss-cn-hangzhou.aliyuncs.com/iPic/2019-05-11-Two%20Sum.gif">



