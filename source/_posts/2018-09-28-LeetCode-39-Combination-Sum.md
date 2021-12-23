---
title: LeetCode 39. Combination Sum
tags: 
  - C++ 
  - Python
categories: LeetCode
mathjax: true
date: 2018-09-28 19:35:09
---

# 一、题目要求
Given a set of candidate numbers (candidates) (without duplicates) and a target number (target), find all unique combinations in candidates where the candidate numbers sums to target.

The same repeated number may be chosen from candidates unlimited number of times.

Note:

All numbers (including target) will be positive integers.
The solution set must not contain duplicate combinations.

Example 1:
```
Input: candidates = [2,3,6,7], target = 7,
A solution set is:
[
  [7],
  [2,2,3]
]
```
Example 2:

```
Input: candidates = [2,3,5], target = 8,
A solution set is:
[
  [2,2,2,2],
  [2,3,3],
  [3,5]
]
```
<!--more-->

# 二、回溯算法
这个问题和之前碰到的Sudoku Solver一样，可以利用带回溯的dfs来得到所有解。
C++代码如下：
```cpp
class Solution {
 private:
  void solve(vector<int> &candidates, int target, vector<vector<int>> &answer, vector<int> &current, int begin) {
    if (target == 0) {
      answer.push_back(current);
    } else {
      for (int i = begin; i < candidates.size() && candidates[i] <= target; i++) {
        current.push_back(candidates[i]);
        solve(candidates, target - candidates[i], answer, current, i);
        current.pop_back();
      }
    }
  }

 public:
  vector<vector<int>> combinationSum(vector<int> &candidates, int target) {
    sort(candidates.begin(), candidates.end());
    vector<vector<int>> answer{};
    vector<int> current{};
    solve(candidates, target, answer, current, 0);
    return answer;
  }
};
```

# 三、动态规划
这个问题也可以用动态规划来解决，那么选定当前答案作为状态的参数，所以dp数组有`target + 1`个元素。
接着分析状态转移条件，`dp[n]`的值对于每个candidate`c`由`dp[n-c]`中每个数组附加上`c`来决定，这样就可以得出结果了。
由于用Python来实现状态转移十分方便，所以就用Python作答了
Python3代码如下：
```python
class Solution:
    def combinationSum(self, candidates: [int], target: int) -> [[int]]:
        dp = [[] for _ in range(target + 1)]
        dp[0].append([])
        for c in candidates:
            for comb_sum in range(target + 1):
                if dp[comb_sum] and comb_sum + c <= target:
                    dp[comb_sum + c].extend([comb + [c] for comb in dp[comb_sum]])
        return dp[target]
```
