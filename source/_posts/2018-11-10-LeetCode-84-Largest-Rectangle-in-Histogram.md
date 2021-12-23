---
title: LeetCode 84. Largest Rectangle in Histogram
tags: C++
categories: LeetCode
mathjax: true
date: 2018-11-10 17:18:19
---

# 一、题目要求
Given n non-negative integers representing the histogram's bar height where the width of each bar is 1, find the area of largest rectangle in the histogram.
![](https://assets.leetcode.com/uploads/2018/10/12/histogram.png)
Above is a histogram where width of each bar is 1, given height = [2,1,5,6,2,3].
![](https://assets.leetcode.com/uploads/2018/10/12/histogram_area.png)
The largest rectangle is shown in the shaded area, which has area = 10 unit.

Example:
```
Input: [2,1,5,6,2,3]
Output: 10
```
<!--more-->

# 二、题目分析
这题让我想起了之前做到的一道接雨水的题，只不过方向不太一样。
仔细思考了一番之后，发现这题也可以用与那题类似的方法——利用单调栈来解决。
从左向右进行遍历，单调栈中栈顶索引对应柱形高度为栈中最大值，就是最高的那一根柱子，遇到比栈顶对应高度小的索引时就到了计算的环节了。
由于始终维持着栈的单调性，所以从**栈顶元素所在柱形**（记其索引为p）到**当前遍历到的柱形**（记其索引为q），其间的柱子高度一定大于两者，也就是说能够以必然能以p的高度作为矩形的一边，以p、q之间的距离作为矩形的另一边，这样来形成一个矩形。
再注意到一个细节，如果当遍历结束之后，栈中仍然存在元素，那么意味着自那个元素之后的所有柱形都比它高，那么就能形成一个新的矩形。
因为上述流程相当于给每根柱子找到最长的宽度来形成矩形，所以肯定能找到最大的矩形。
如果进一步引入一个高度为0的哨兵元素放入直方图中，这样就无需考虑遍历结束后栈中仍然有元素（除去哨兵）存在了，算法能进一步简洁。

# 三、算法实现
```cpp
int largestRectangleArea(vector<int>& heights) {
  int maxArea = 0;
  stack<int> s;
  heights.push_back(0);

  for (int i = 0; i < heights.size(); i++) {
    while (!s.empty() && heights[s.top()] > heights[i]) {
      int height = heights[s.top()];
      s.pop();
      int width = s.empty() ? i : i - s.top() - 1;
      maxArea = max(maxArea, height * width);
    }
    s.push(i);
  }
  return maxArea;
}
```