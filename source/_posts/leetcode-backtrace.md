---
title: 回溯类问题（DFS）解题模板
date: 2024-03-14 19:32:27
tags:
---

# 回溯类问题-DFS

每次刷题刷完了不久就会忘记DFS的很多细节。。直到看到了东哥的[回溯算法详解](https://mp.weixin.qq.com/s/nMUHqvwzG2LmWA9jMIHwQQ)，才从根上理解了这一类算法的底层原理。

对于我这种看到算法就头疼的选手，还真得靠这种大佬帮忙梳理总结底层原理才行。。

**核心框架 && 解题模板**：

```python
## 定义：
#1、路径：当前已经做过的所有选择
#2、选择列表：当前结点可以做的选择
#3、结束条件：即到达决策树底层，无法再做选择的条件

## 核心框架
def backtrace(路径，选择列表)：
	if 满足结束条件：
    	return
    for 选择 in 选择列表：
        #做选择
        backtrace()
        #撤销选择

for 选择 in 选择列表：
	# 做选择
    选择列表.remove(当前选择)
    路径.add(当前选择)
    # 回溯
    backtrace(路径，选择列表)
    # 撤销选择
    路径.remove(当前选择)
    选择列表.add(当前选择)

```



## [49. 子集](https://leetcode.cn/problems/subsets/)

```python
# 输入：nums = [1,2,3]
# 输出：[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]

## 按照回溯的思想，有前序和后序两种方式
## 1、如果是前序执行，则路径的记录顺序应该为:
## [],[1],[1,2],[1,2,3],(从这里开始回撤)[1,3],(再次回撤)[2],[2,3],(再次回撤)[3]
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        res = [] # 记录最终结果
        track = [] # 维护当前路径
        def backtrace(nums, start):
            res.append(track[:]) # 把当前路径记录到最终结果里
            ## 模板
            ## 注意，这里的nums就是我们可以做出选择的选择列表;
            ## 且这里的隐藏结束条件是for循环到头
            for i in range(start, len(nums)): 
                ## 做选择
                track.append(nums[i])
                ## 回溯
                backtrace(nums, i+1)
                ## 回撤
                track.pop()
        backtrace(nums, 0)
        return res
    
## 2、也可以后序执行，则路径的记录顺序为：
## [1,2,3],[1,2],(这里开始有个回撤)[1,3],(再次回撤)[1],(再次回撤)[2,3],(再次回撤)[2],(再次回撤)[3],(自己补一个)[]
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        res = [] # 记录最终结果
        track = [] # 维护当前路径
        def backtrace(nums, start):
            ## 模板
            for i in range(start, len(nums)):
                ## 做选择
                track.append(nums[i])
                ## 回溯
                backtrace(nums, i+1)
                ## 撤回选择
                track.pop()
            res.append(track[:])
        backtrace(nums, 0)
        res.append([])
        return res

```



## [46. 全排列](https://leetcode.cn/problems/permutations/description/)

```python
# 输入：nums = [1,2,3]
# 输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]

class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        res = []
        track = []
        def backtrace(nums):
            if len(track) == len(nums):
                res.append(track[:])
            for i in range(len(nums)):
                if nums[i] in track: continue
                ## 做选择
                track.append(nums[i])
                ## 回溯
                backtrace(nums)
                ## 撤销选择
                track.pop()
                
        backtrace(nums)
        return res

```



## [226. 翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/)

```python
# 输入：root = [4,2,7,1,3,6,9]
# 输出：[4,7,2,9,6,3,1]

class Solution:
    def invertTree(self, root: Optional[TreeNode]) -> Optional[TreeNode]:
        def dfs(root):
            if not root:
                return
            root.left, root.right = root.right, root.left
            dfs(root.left)
            dfs(root.right)
        dfs(root)
        return root
```



## [543. 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/)

```python
# 输入：root = [1,2,3,4,5]
# 输出：3
# 解释：3 ，取路径 [4,2,1,3] 或 [5,2,1,3] 的长度

class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        self.d = 0  # 记录最终结果
        def dfs(root):
            if not root:
                return 0
            left = dfs(root.left)
            right = dfs(root.right)
            self.d = max(self.d, left + right)
            return max(left, right) + 1   # 要维护的是当前结点的最长子路径，回撤操作+1
        dfs(root)
        return self.d
```

