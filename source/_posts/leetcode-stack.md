---
title: Leetcode笔记7 - 栈结构总结
date: 2024-03-22 13:32:02
categories: 
- Leetcode算法笔记
tags: 
- Leetcode算法笔记
---

# Leetcode笔记7 - 栈结构总结

栈的特点是：先进后出，后进先出。

遇到需要先进后出的题，优先考虑栈结构。例如：匹配括号等。

## 1. 辅助栈

### [20. 有效的括号](https://leetcode.cn/problems/valid-parentheses/)

```python
# 输入：s = "()"
# 输出：true

# 输入：s = "(]"
# 输出：false

class Solution:
    def isValid(self, s: str) -> bool:
        ## ([{}()])
        stack = ['?']
        hashmap = {'(':')','[':']','{':'}','?':'?'}
        for c in s:
            if c in hashmap: stack.append(c)
            elif c != hashmap[stack.pop()]: return False
        
        return True if len(stack) == 1 else False
```



## 2. 单调栈

单调栈是用来解决“**下一个更大**”类型的题目。

一个数组，要求你找到每个元素对应的下一个更大的元素。

[To do]

