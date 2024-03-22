---
title: Leetcode笔记4 - 排序算法总结
date: 2024-03-19 22:29:55
categories: 
- Leetcode算法笔记
tags: 
- Leetcode算法笔记
---

# Leetcode笔记4 - 排序算法总结

总结一把最可能面试面到的排序：**快速排序 **和 **归并排序**

## 快速排序

**核心思路**

 - 快速排序的思路是每次确定一个值，该值的左边都是比它小的，右边都是比它大的

### [215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)

```python
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        ## 快速排序的思路是每次确定一个值，该值的左边都是比它小的，右边都是比它大的
        def partition(nums, low, high):
            mid = (low + high) // 2
            nums[high], nums[mid] = nums[mid], nums[high]
            
            i = low
            for j in range(low, high):
                if nums[j] <= nums[high]:
                    nums[i], nums[j] = nums[j], nums[i]
                    i += 1
            nums[i], nums[high] = nums[high], nums[i]
            return i

        def quickSort(nums, low, high):
            if low >= high:
                return
            pi = partition(nums, low, high)
            quickSort(nums, low, pi-1)
            quickSort(nums, pi+1, high)
        
        quickSort(nums, 0, len(nums)-1)
        return nums[-k]
```



## 归并排序

**核心思路**

- 分成左右两个**有序数组**，然后要将两个数组按顺序合并好（双指针合并）
- 涉及到数组边界不清晰的时候，**最好用while循环遍历数组**而不用nums[low:high]的形式

### [215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)

```python
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        ## 归并排序的思路是，分成左右两个有序数组，然后要将两个数组按顺序合并好
        def merge(nums, low, mid, high): # 职能是将两个有序数组按顺序合并好——双指针
            i, j = low, mid+1
            res = []
            while i <= mid and j <= high:
                if nums[i] < nums[j]: 
                    res.append(nums[i])
                    i += 1
                else: 
                    res.append(nums[j])
                    j += 1
            while i <= mid:
                res.append(nums[i])
                i += 1
            while j <= high:
                res.append(nums[j])
                j += 1
            nums[low:high+1] = res[:]
            

        def sort(nums, low, high):
            if low >= high:
                return
            mid = (low + high) // 2
            sort(nums, low, mid)
            sort(nums, mid+1, high)
            merge(nums, low, mid, high)
        
        sort(nums, 0, len(nums)-1)
        return nums[-k]
```

