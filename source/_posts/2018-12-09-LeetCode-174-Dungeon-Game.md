---
title: LeetCode 174. Dungeon Game
tags: C++
categories: LeetCode
mathjax: true
date: 2018-12-09 18:12:53
---

# 一、题目要求
The demons had captured the princess (P) and imprisoned her in the bottom-right corner of a dungeon. The dungeon consists of M x N rooms laid out in a 2D grid. Our valiant knight (K) was initially positioned in the top-left room and must fight his way through the dungeon to rescue the princess.

The knight has an initial health point represented by a positive integer. If at any point his health point drops to 0 or below, he dies immediately.

Some of the rooms are guarded by demons, so the knight loses health (negative integers) upon entering these rooms; other rooms are either empty (0's) or contain magic orbs that increase the knight's health (positive integers).

In order to reach the princess as quickly as possible, the knight decides to move only rightward or downward in each step.

Write a function to determine the knight's minimum initial health so that he is able to rescue the princess.

For example, given the dungeon below, the initial health of the knight must be at least 7 if he follows the optimal path RIGHT-> RIGHT -> DOWN -> DOWN.

-2 (K)	-3	3
-5	-10	1
10	30	-5 (P)
 

Note:

- The knight's health has no upper bound.
- Any room can contain threats or power-ups, even the first room the knight enters and the bottom-right room where the princess is imprisoned.

<!--more-->
# 二、题目分析
这题很显然可以用动态规划来求解，可以将走到一格的问题分解为走到其上方一格和走到其左边一格两个子问题，那么状态就是走到这格的最小所需HP，再减去当前格的HP损耗得到一个中间值，再和之前格的所需HP一起取一个最大值就能找到能走出这格的最小所需HP，也就是最终答案了。
这样子想想似乎很简单，但是实际上状态转移就变得十分复杂，最主要的是考虑这样一种情况，不能活着走出之前的格子，这样的话之前计算得到的状态就完全没有用处了，还不如直接把走出格子所需的HP作为状态。
所以试着换个思路，把状态定义为走出这格的最小所需HP试试。那么接下来考虑继续状态转移，走出当前格的最小所需HP就是`min(走出上格的最小HP, 走出左格的最小HP) - 当前格HP损耗`，但是实际这么实现以后，发现碰到为正值的HP补充的格子之后就会出现问题，在碰到比较大的HP补充之后，计算出来的最小所需HP就会变为负值，而如果单纯地把它变为1，则导致无法走过先前的格子，整个思路到这里陷入了僵局。
经过一番漫长的思考，意识到应该把从某个位置走到终点所需的最小HP作为状态，然后从终点处倒着计算出在(0, 0)的所需HP，因为走到终点是一个不可割裂的过程，能够成功走过之前的路所需的最小HP并不能确保走过之后的路，但是反过来向前计算就可以确保。

# 三、算法实现
```cpp
int calculateMinimumHP(vector<vector<int>> &dungeon) {
  int M = dungeon.size();
  int N = dungeon[0].size();
  vector<vector<int>> localMinimumHP(M + 1, vector<int>(N + 1, INT_MAX));
  localMinimumHP[M - 1][N - 1] = 1 - dungeon[M - 1][N - 1];
  if (localMinimumHP[M - 1][N - 1] <= 0) {
    localMinimumHP[M - 1][N - 1] = 1;
  }
  for (int i = M - 1; i >= 0; i--) {
    for (int j = N - 1; j >= 0; j--) {
      if (i == M - 1 && j == N - 1) continue;
      int enterMinHP = min(localMinimumHP[i + 1][j], localMinimumHP[i][j + 1]) - dungeon[i][j];
      localMinimumHP[i][j] = enterMinHP <= 0 ? 1 : enterMinHP;
    }
  }

  return localMinimumHP[0][0];
}
```