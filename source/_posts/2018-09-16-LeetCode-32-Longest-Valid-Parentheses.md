---
title: LeetCode 32. Longest Valid Parentheses
date: 2018-09-16 09:41:42
tags: C++
categories: LeetCode
mathjax: true
---

# 一、题目要求
Given a string containing just the characters '(' and ')', find the length of the longest valid (well-formed) parentheses substring.

Example 1:
```
Input: "(()"
Output: 2
Explanation: The longest valid parentheses substring is "()"
```
Example 2:
```
Input: ")()())"
Output: 4
Explanation: The longest valid parentheses substring is "()()"
```
<!-- more -->

# 二、暴力求解
先考虑暴力求解的复杂度，对给定字符串的每个子串进行是否为有效括号序列的校验，枚举子串的时间复杂度为$O(N^2)$，校验是否为有效括号序列的时间复杂度为$O(N)$，总的时间复杂度为$O(N^3)$。

# 三、利用栈求解
接下来更进一步地思考时间复杂度更低的算法，在判断括号序列是否有效时我们通常会使用栈，我们是否可以在一趟遍历中检验有效性的同时记录下序列长度呢。

在检验括号序列是否有效的时候，我们只需要记录先前无效序列的结束位置来作为当前有效序列的起始位置，借此在当前序列变为无效的时候计算当前序列的长度，再记录整个遍历过程中最长的有效序列长度就行了。

但是如果当前序列结束的时候，栈内还有左括号存留，就需要利用最后一个匹配到的左括号作为序列的起始来计算当前序列的长度。因此我们在栈中需要保存匹配到的左括号的索引，在结束匹配的时候用栈顶的索引来计算长度，这个时候看到如果将当前序列之前的右括号索引推入栈中，计算的时候也能直接用这个索引，并在初始化时在栈中推入-1作为之前序列的结束位置，这样的话处理不同状态的代码具有高度一致性，十分优雅。

这个算法时间复杂度为$O(N)$，完整的代码如下：
```cpp
int longestValidParentheses(const string &str) {
  int ret = 0;
  size_t len = str.length();
  stack<int> s;
  s.push(-1);
  for (int i = 0; i < len; i++) {
    if (str[i] == '(') {
      s.push(i);
    } else {
      s.pop();
      if (s.empty()) {
        s.push(i);
      } else {
        ret = max(ret, i - s.top());
      }
    }
  }
  return ret;
}
```

# 四、利用动态规划求解
这类求最长的题目通常能用动态规划进行求解，依据动态规划的解题步骤，先确定状态和状态方程。

将当前索引的最长有效括号序列长度作为状态，分析状态转换条件，得出如下状态方程
$$ dp[i]= \begin{cases} dp[i-2]+2, & \text {if $s[i] == \textrm')\textrm'$ and $s[i-1] == \textrm'(\textrm'$} \\\\ dp[i-1]+dp[i-dp[i-1]-2]+2, & \text {if $s[i] == \textrm')\textrm'$ and $s[i-1] == \textrm')\textrm'$ and $s[i−dp[i−1]−1] == \textrm'(\textrm'$} \end{cases} $$

这个算法时间复杂度为$O(N)$，代码就不贴出来了。
