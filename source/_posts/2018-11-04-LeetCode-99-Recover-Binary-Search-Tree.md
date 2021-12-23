---
title: LeetCode 99. Recover Binary Search Tree
tags: C++
categories: LeetCode
mathjax: true
date: 2018-11-04 21:22:34
updated: 2018-11-04 21:22:34
---

# 一、题目要求
Two elements of a binary search tree (BST) are swapped by mistake.

Recover the tree without changing its structure.

Example 1:
```
Input: [1,3,null,null,2]

   1
  /
 3
  \
   2
Output: [3,1,null,null,2]

   3
  /
 1
  \
   2
```

Example 2:
```
Input: [3,1,4,null,null,2]

  3
 / \
1   4
   /
  2

Output: [2,1,4,null,null,3]

  2
 / \
1   4
   /
  3
```
<!--more-->

# 二、题目分析
这是一个二叉树的题，题目说有两个元素的位置错位了，需要把它们找出来并恢复。
一开始花了好多时间还是没有想到应该用什么算法去解决这个问题，主要是无法确定错误发生的位置。
从例二就可以看出简单地判断和儿子节点的大小并不能确定当前节点有没有错误。
但是想到这里就想到了每个节点不仅仅要大于左儿子小于右儿子，而是应该大于左子树小于右子树，那么进而发觉有一种遍历方式就是按左子树-本身-右子树的方式来遍历整棵树，那就是中序遍历，所以这题其实并不复杂，只需要在中序遍历的时候进行一点额外的工作就行了。
具体来说就是确定左子树中最大的节点，而这实际上就是中序遍历时当前节点的前一个节点，只要当前节点大于这个"前一个节点"就是有效的节点。
那么接下来算法就很容易了。

# 三、算法实现
```cpp
class Solution {
 public:
  TreeNode *firstWrong = nullptr;
  TreeNode *secondWrong = nullptr;
  TreeNode *prevNode = new TreeNode(INT32_MIN);

  void recoverTree(TreeNode *root) {
    inOrder(root);
    cout << firstWrong->val << endl << secondWrong->val;
    swap(firstWrong->val, secondWrong->val);
  }

  void inOrder(TreeNode *current) {
    if (!current) {
      return;
    }
    inOrder(current->left);

    if (prevNode->val >= current->val) {
      if (!firstWrong) {
        firstWrong = prevNode;
      }
      secondWrong = current;
    }
    
    prevNode = current;
    inOrder(current->right);
  }
};
```