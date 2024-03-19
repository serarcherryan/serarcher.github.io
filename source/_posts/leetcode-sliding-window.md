---
title: 滑动窗口解题模板
date: 2024-03-12 20:21:53
tags: Leetcode算法笔记
---

因为工作关系，coding不常用到算法，每次面试/工作中遇到都要回忆好久。因此特意总结一些常见leetcode算法的解题模板，面试/工作中遇到一些类似的问题可以节省很多回忆的时间。

**模板并不是所有问题的最优解，对于不同类型的问题，时间/空间复杂度都要分别再行思考，往往会有更加精巧的解法。**

# 滑动窗口


**核心思路：**

​	1、**扩大窗口**：移动右指针，直至达到特定要求（初始最优值）的时候

​	2、**收缩窗口**：记录当前最优值，并移动左指针，直至满足扩大窗口的要求

**套路模板**：

​	双重循环：**外层循环移动右指针**，**里层循环移动左指针**

```python
left, right = 0, 0
for right in range(n): ## 扩大窗口，右移右指针
    ## 处理一些窗口相关的数据
    while window needs shirnk: ## 当达到当前最优值时，收缩窗口，右移左指针 
    	## 处理一些窗口相关的数据
        keep.remove(s[left])
        left += 1
    
    ## 一轮结束，开始新一轮扩大窗口
    ## 处理一些窗口相关的数据
    keep.add(s[right])
    
```



## [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        if not s:
            return 0
        keep = set()
        left, cur_len, max_len, n = 0, 0, 0, len(s)
        for right in range(n): ## 扩大窗口，右移右指针
            cur_len += 1	## 处理一些窗口相关的数据：记录当前字符串长度
            while s[right] in keep: ## 当遇到重复字符时，收缩窗口，右移左指针
                ## 处理一些窗口相关的数据：
                ## 右移左指针（之前要先删掉窗口里的元素）
                ## 当前长度 - 1
                keep.remove(s[left]) 
                left += 1
                cur_len -= 1
            ## 一轮结束，需要维护最大长度值，并且增加窗口元素
            max_len = max(max_len, cur_len)
            keep.add(s[right])
        return max_len

```



## [209. 长度的最小子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)

```python
class Solution:
    def minSubArrayLen(self, target: int, nums: List[int]) -> int:
        left, n = 0, len(nums)
        min_len = n + 1
        sum_ = 0
        for right in range(n): ## 扩大窗口，右移右指针
            sum_ += nums[right] ## 求窗口的和
            while sum_ >= target: ## 当和大于等于target时，收缩窗口，右移左指针
                min_len = min(min_len, right + 1 - left) ## 记录最小的窗口长度
                ## 右移左指针，当前长度-1
                sum_ -= nums[left] 
                left += 1
        if min_len == n + 1:
            return 0
        return min_len
```

