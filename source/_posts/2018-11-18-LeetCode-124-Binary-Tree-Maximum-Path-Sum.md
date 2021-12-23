---
title: LeetCode 124. Binary Tree Maximum Path Sum
tags: C++
categories: LeetCode
mathjax: true
date: 2018-11-18 18:33:00
---

# 一、题目要求
Given a non-empty binary tree, find the maximum path sum.

For this problem, a path is defined as any sequence of nodes from some starting node to any node in the tree along the parent-child connections. The path must contain at least one node and does not need to go through the root.

Example 1:
```
Input: [1,2,3]

       1
      / \
     2   3

Output: 6
```
Example 2:
```
Input: [-10,9,20,null,null,15,7]

   -10
   / \
  9  20
    /  \
   15   7

Output: 42
```
<!--more-->

# 二、题目分析
这道题目的要求是找到和最大的子树，和找最大子序列和有相似之处。比较之后可以发现差别并不是很大，同样可以用动态规划的思想进行求解。
状态转移方程：`n->maxSum = n->val + max(0, n->left->maxSum) + max(0, n->right->maxSum)`
但是这个问题的对象是二叉树，对状态的保存并不方便，因为子树不好表示，自底向上的建表法不容易实现。
那么就试着采用自顶向下的记忆化递归的思路，但是随之稍微一看就可以发现，需要求解的子问题是子树的子树最大和，对树进行遍历并不会有重复的子问题产生，所以记忆化并没有帮助，所以解决整个问题的算法就简化到了简单的递归遍历的使用。
那么接着就根据状态转移方程来实现算法就行了，并不难。结果...出错了。没有读清楚题目，并不是最大的子树，而是一条路径，好在对状态转移方程只需要做微小的变动。
新的状态转移方程：`n->maxSum = n->val + max(0, n->left->maxOneWaySum) + max(0, n->right->maxOneWaySum)`
因为要形成一条路径，所以对于一个特定的节点n而言，它的儿子节点就不能再继续分头求和了，因为那样就无法形成一条路径了。

# 三、算法实现
```cpp
class Solution {
 private:
  int answer = INT32_MIN;
 public:
  int maxPathSum(TreeNode *root) {
    maxOneWaySum(root);
    return answer;
  }

  int maxOneWaySum(TreeNode *node) {
    if (!node) {
      return 0;
    }
    int left = max(0, maxOneWaySum(node->left));
    int right = max(0, maxOneWaySum(node->right));
    int currentMaxSum = left + right + node->val;
    if (currentMaxSum > answer) {
      answer = currentMaxSum;
    }
    return max(left, right) + node->val;
  }
};
```