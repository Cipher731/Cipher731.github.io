---
title: LeetCode 42. Trapping Rain Water
tags: C++
categories: LeetCode
mathjax: true
date: 2018-10-14 23:01:33
---

# 一、题目要求
Given n non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.

![](http://www.leetcode.com/static/images/problemset/rainwatertrap.png)  
The above elevation map is represented by array [0,1,0,2,1,0,1,3,2,1,2,1]. In this case, 6 units of rain water (blue section) are being trapped.  **Thanks Marcos** for contributing this image!

**Example:**
```
Input: [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6
```
<!--more-->
# 二、利用暴力算法求解
考虑暴力算法的复杂度，对于每个位置要分别找到它左边和右边最高的高地，那么时间复杂度应当是$O(n^2)$，无需额外的空间，所以空间复杂度为$O(1)$。

# 三、利用栈求解
可以从左向右遍历，依次将索引推入栈中，但碰到比栈顶所保存的凸起更高的凸起时，意味着存在着出现蓄水池的可能性，此时弹出栈顶元素作为水池的底，当前遍历到的凸起作为水池的右边界，新栈顶作为水池的左边界，这样就能计算出水池的体积。其实到这里可以发现，这个栈是一个单调栈，栈顶始终是最小值，和单调栈的经典利用“解决木板倒水问题”实际上有着异曲同工之妙。
这个方法下需要进行一次遍历，还要维护一个栈，所以时间复杂度是$O(n)$，空间复杂度为$O(n)$。
代码如下：
```cpp
int trap(vector<int> &height) {
  stack<int> s;
  int ret = 0;
  for (int i = 0; i < height.size(); i++) {
    while (!s.empty() && height[i] > height[s.top()]) {
      int bottom = s.top();
      s.pop();
      if (s.empty()) {
        break;
      }
      int poolWidth = i - s.top() - 1;
      int poolHeight = min(height[i], height[s.top()]) - height[bottom];
      ret += poolHeight * poolWidth;
    }
    s.push(i);
  }
  return ret;
}
```