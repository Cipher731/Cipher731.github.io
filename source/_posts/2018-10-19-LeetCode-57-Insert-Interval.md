---
title: LeetCode 57. Insert Interval
tags: C++
categories: LeetCode
mathjax: true
date: 2018-10-19 20:36:31
---

# 一、题目要求
Given a set of non-overlapping intervals, insert a new interval into the intervals (merge if necessary).

You may assume that the intervals were initially sorted according to their start times.

Example 1:
```
Input: intervals = [[1,3],[6,9]], newInterval = [2,5]
Output: [[1,5],[6,9]]
```
Example 2:
```
Input: intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval = [4,8]
Output: [[1,2],[3,10],[12,16]]
Explanation: Because the new interval [4,8] overlaps with [3,5],[6,7],[8,10].
```
<!--more-->

# 二、解题
如果往已有区间中插入新的区间，会存在以下三种情况：
1. 没有与现有区间发生任何重叠，直接插入
1. 与一个区间发生重叠，将两个区间合并在一起 
1. 与两个区间发生重叠，将三个区间合并在一起

考虑两个区间发生重叠时合并区间后的情况，将会以两个区间中较小的下界作为新区间的下界，较大的上界作为新区间的上界。
继续注意到第三种情况下可以视为两次第二种情况，将第一次合并的区间作为待插入区间即可。
因此只需要在遍历的时候找到需要插入的位置，再进行相应操作即可。
算法如下：
```cpp
class Solution {
 public:
  vector<Interval> insert(vector<Interval> &intervals, Interval newInterval) {
    vector<Interval> ret;
    auto iter = intervals.begin();
    while (iter != intervals.end()) {
      if (iter->end < newInterval.start) {
        ret.push_back(*iter);
      } else if (iter->start > newInterval.end) {
        break;
      } else {
        newInterval.start = min(newInterval.start, iter->start);
        newInterval.end = max(newInterval.end, iter->end);
      }
      iter++;
    }
    ret.push_back(newInterval);
    while (iter != intervals.end()) {
      ret.push_back(*iter);
      iter++;
    }
    return ret;
  }
};
```
