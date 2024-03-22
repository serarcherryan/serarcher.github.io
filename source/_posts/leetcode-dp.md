---
title: Leetcode笔记6 - 动态规划解题模板
date: 2024-03-20 23:12:38
categories: 
- Leetcode算法笔记
tags: 
- Leetcode算法笔记
---



## Leetcode笔记6 - 动态规划

**核心思路**

- 确定状态，即问题的变量
- 确定dp方程
- 采用自底向上 / 自顶向下的解法

**自底向上**

- 从dp(0)开始举几个例子即可
- 确定好base case，假设base case确定到dp(n)，那么for循环的变量一定要从n+1的情况开始
- 例：for循环的变量从2开始，那么一定要确定的base case是dp(0), dp(1)

## [322. 零钱兑换](https://leetcode.cn/problems/coin-change/)

```python
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        ## dp(n) = min{d(n-coin)} + 1, coin=1,2,5, n > 0
        ## dp(0) == 0
        ## dp(1) == min{dp(1-1), dp(1-2), dp(1-5)} + 1 == dp(1-1) + 1 == 1
        ## dp(2) == min{dp(2-1), dp(2-2), dp(2-5)} + 1 == dp(2-2) + 1 == 1
        ## dp(3) == min{dp(3-1), dp(3-2), dp(3-5)} + 1 == dp(2) + 1 == 2
        ## dp(4) == min{dp(4-1), dp(4-2), dp(4-5)} + 1 == dp(2) + 1 == 2
        dp = [float('inf')] * (amount + 1)
        dp[0] = 0
        for coin in coins:
            for x in range(coin, amount + 1):
                dp[x] = min(dp[x], dp[x - coin] + 1)
        return dp[amount] if dp[amount] != float('inf') else -1
```

