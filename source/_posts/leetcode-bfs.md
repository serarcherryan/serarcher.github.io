---
title: Leetcode笔记5 - BFS解题模板
date: 2024-03-20 23:03:16
categories: 
- Leetcode算法笔记
tags: 
- Leetcode算法笔记
---

# Leetcode笔记5 - BFS

**核心思路**

- 维护一个队列，队列里存放的是同一行的所有节点。
- 通过while循环每次把队列里的结点全部读掉，并且每读（pop）一个，就要将该结点的左右子结点存放进去
- 循环直到队列为空



## [102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/)

```python
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
        res = []
        queue = [root]

        while queue:
            ## 通过队列长度来遍历掉当前层的所有结点
            n = len(queue)
            level = []
            for _ in range(n):
	            node = queue.pop(0)
                level.append(node.val)
                if node.left: level.append(node.left)
                if node.right: level.append(node.right)
            res.append(level)
            
        return res
            
```



## [199. 二叉树的右视图](https://leetcode.cn/problems/binary-tree-right-side-view/)

跟上面的唯一不同就是每层只取最右边的值作为结果返回

```python
class Solution:
    def rightSideView(self, root: Optional[TreeNode]) -> List[int]:
        if not root:
            return []
        res = []
        queue = [root]
        
        while queue:
            n = len(queue)
            level = []
            for _ in range(n):
                node = queue.pop(0)
                level.append(node.val)
                if node.left: queue.append(node.left)
                if node.right: queue.append(node.right)
            res.append(level[-1])
        return res
```

