---
title: LeetCode 76. Minimum Window Substring
tags: C++
categories: LeetCode
mathjax: true
date: 2018-10-28 23:05:35
---

# 一、题目要求
Given a string S and a string T, find the minimum window in S which will contain all the characters in T in complexity O(n).

Example:
```
Input: S = "ADOBECODEBANC", T = "ABC"
Output: "BANC"
```
Note:

- If there is no such window in S that covers all characters in T, return the empty string "".
- If there is such window, you are guaranteed that there will always be only one unique minimum window in S.
<!--more-->
# 二、题目分析
题目要求在$O(n)$的复杂度下完成算法，也就意味着要在一次或常数次遍历中把答案找出来。而题目只需要子串包含T中的字母而不涉及到顺序，所以这种情况下计数就很合理。
那么暴力解法就出现了，先对T中出现的字母进行计数，然后从S中遍历子串比较字母的计数，但这样算法复杂度仍然有$O(n^2)$，需要更进一步优化算法。
采用变区间的方式，从S的最左侧开始，建立一个子区间，不断增大右边界，并维护子区间内字母数量的信息，如果满足T中所有字母在子区间中出现，那么就增大左边界缩小区间直到不满足条件，接着增大右边界，重复上述过程直到遍历完整个S。这样就能在$O(n)$的时间复杂度内解决问题。

# 三、算法实现
```cpp
string minWindow(string &s, string &t) {
  if (s.empty() || t.empty()) {
    return "";
  }
  int target[255]{};
  int required = 0;
  for (char c : t) {
    if (target[c] == 0) {
      required++;
    }
    target[c]++;
  }

  int counter[255]{};
  int left = 0, right = 0, fulfilled = 0;
  int minSize = INT32_MAX, minLeft = -1;
  while (right < s.length()) {
    char current_right = s[right];
    counter[current_right]++;
    if (counter[current_right] == target[current_right]) {
      fulfilled++;
    }

    while (left <= right && fulfilled == required) {
      char current_left = s[left];
      int current_size = right - left + 1;
      if (current_size < minSize) {
        minLeft = left;
        minSize = current_size;
      }
      counter[current_left]--;
      if (counter[current_left] < target[current_left]) {
        fulfilled--;
      }
      left++;
    }
    right++;
  }
  if (minLeft == -1) {
    return "";
  }
  return s.substr(minLeft, minSize);
}
```