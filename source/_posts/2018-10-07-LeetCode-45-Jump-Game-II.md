---
title: LeetCode 45. Jump Game II
tags: C++
categories: LeetCode
date: 2018-10-07 19:56:12
updated: 2018-10-07 19:56:12
---

# 一、题目要求
Given an array of non-negative integers, you are initially positioned at the first index of the array.

Each element in the array represents your maximum jump length at that position.

Your goal is to reach the last index in the minimum number of jumps.

Example:
```
Input: [2,3,1,1,4]
Output: 2
Explanation: The minimum number of jumps to reach the last index is 2.
    Jump 1 step from index 0 to 1, then 3 steps to the last index.
```
Note:
You can assume that you can always reach the last index.
<!--more-->

# 二、题目分析
这道题可以像前两篇那样使用搜索来做，但是连续三周都用搜索就很没劲。
试着寻找另外的方法，发现可以使用贪心算法，也就是说每一跳的时候都选取下一次能跳得最远的跳法，那么只需要遍历每个位置上的步数即可

# 三、实现算法
```cpp
class Solution {
 public:
  int jump(vector<int> &nums) {
    int jumps = 0;
    for (int i = 0, last_fartherest = 0, current_fartherest = 0; current_fartherest < nums.size() - 1; jumps++) {
      for (; i <= last_fartherest; i++) {
        if (i + nums[i] > current_fartherest) {
          current_fartherest = i + nums[i];
        }
      }
      last_fartherest = current_fartherest;
    }
    return jumps;
  }
};
```