---
title: K-Sum类问题解题模板
date: 2024-03-13 23:04:05
categories: 
- Leetcode算法笔记
tags: 
- Leetcode算法笔记
---

# K-Sum

K-Sum类问题是指给你一个数组，让你从中挑选出K个数字满足和为N。

K-Sum类的核心思想是 2Sum，然后套娃就行了。因此此类问题的关键点在于**2Sum**的实现。

**核心思路**：

​	1、**数组排序**

​	2、**双指针**：头尾相向移动，遇到相同的数字跳过****

​	3、**防止重复值**：每个最外层的遍历都需要防止重复值，以及对于遍历数组的边界值的处理

**套路模板：**

实现(K-1)-Sum，以3Sum为例，首先要实现2Sum：

```python
# 2-Sum可直接默写
def twoSum(nums, start, target):
    res = []
    low, high = start, len(nums)-1
    while low < high:
        sum_ = nums[low] + nums[high]
        if sum_ < target:
            low += 1
            # 指针一定要注意，因为前面已经low += 1了
            # 所以现在的nums[low]一定要和nums[low-1]去比较
            while low < high and nums[low] == nums[low-1]: low += 1 
        elif sum_ > target:
            high -= 1
            # nums[high]同理
            while low < high and nums[high] == nums[high+1]: high -= 1
        else:
            res.append([nums[low], nums[high]])
            low += 1
            high -= 1
            while low < high and nums[low] == nums[low-1]: low += 1
            while low < high and nums[high] == nums[high+1]: high -= 1
    return res

# 3-Sum 主要是要注意边界值的问题 和 要防止有重复值
def threeSum(nums, target):
    nums.sort()
    res = []
    for i in range(nums[i]-2): ## 注意边界值为K-1
        if nums[i] > 0: break ## 因为是升序排列，可提高效率
        if low > 0 and nums[low] == nums[low-1]: continue ## 防止有重复值
        ## 调用2Sum
        target_ = target - nums[i]
        tuples = twoSum(nums, i+1, target_)
        for tup in tuples:
            tup.append(nums[i])
            res.append(tip)
    return res

        
```



## [15. 三数之和](https://leetcode.cn/problems/3sum/)

```python
class Solution:
    def twoSum(self, nums, start, target):
        res = []
        low = start
        high = len(nums) - 1
        while low < high:
            sum_ = nums[low] + nums[high]
            if sum_ > target:
                high -= 1
                while low < high and nums[high] == nums[high+1]: high = high -1
            elif sum_ < target:
                low += 1
                while low < high and nums[low] == nums[low-1]: low += 1
            else:
                res.append([nums[low], nums[high]])
                high -= 1
                low += 1
                while low < high and nums[high] == nums[high+1]: high -= 1
                while low < high and nums[low] == nums[low-1]: low += 1
        return res

    def threeSum(self, nums: List[int]) -> List[List[int]]:
        nums.sort()
        res = []
        for i in range(len(nums)-2):
            if nums[i] > 0: break
            if i > 0 and nums[i] == nums[i-1]: continue
            tuples = self.twoSum(nums, i+1, -nums[i])
            for tup in tuples:
                tup.append(nums[i])
                res.append(tup)
        return res
```

 

## [1. 两数之和](https://leetcode.cn/problems/two-sum/description/)

这里有个小技巧：**给定一个数组，当我们每次需要移动一个指针，然后遍历扫描该指针后面的数组元素的时候，其实可以用哈希表来降低算法复杂度 —— 仅需要扫一遍即可**

```python
# 输入：nums = [2,7,11,15], target = 9
# 输出：[0,1]
# 解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 

class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        hashtable = {}
        for i, num in enumerate(nums):
            if target - num in hashtable:
                return [hashtable[target-num], i]
            hashtable[nums[i]] = i # 每次在最后往哈希表里更新即可
        return []
```

