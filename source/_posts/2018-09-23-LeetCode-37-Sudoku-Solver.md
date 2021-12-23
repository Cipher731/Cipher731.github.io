---
title: LeetCode 37. Sudoku Solver
date: 2018-09-23 13:54:37
tags: C++
categories: LeetCode
mathjax: false
---

# 一、题目要求
Write a program to solve a Sudoku puzzle by filling the empty cells.

A sudoku solution must satisfy all of the following rules:

Each of the digits 1-9 must occur exactly once in each row.
Each of the digits 1-9 must occur exactly once in each column.
Each of the the digits 1-9 must occur exactly once in each of the 9 3x3 sub-boxes of the grid.
Empty cells are indicated by the character '.'.

Note:

The given board contain only digits 1-9 and the character '.'.
You may assume that the given Sudoku puzzle will have a single unique solution.
The given board size is always 9x9.
<!--more-->

# 二、一次错误的尝试
由于Cipher我对数独并不是很熟悉，一开始解这题的时候只考虑到从确定的值一个接一个推导出所有的值的方法，但是经历了一次WA之后发现，实际上会有这样一个情况，无法通过给定的信息确定下一个填上的数字，所以需要试探性地对可能的情况进行搜索，这个时候自然想到需要用到回溯。
下面是一根筋的错误解法
```cpp
void solveSudoku(vector<vector<char>> &board) {
  vector<bitset<9>> rowFlags(9, bitset<9>());
  vector<bitset<9>> colFlags(9, bitset<9>());
  vector<vector<bitset<9>>> blockFlags(3, vector<bitset<9>>(3, bitset<9>()));

  for (int i = 0; i < 9; i++) {
    for (int j = 0; j < 9; j++) {
      if (board[i][j] == '.') {
        continue;
      }
      int val = board[i][j] - '1';
      rowFlags[i].set(val);
      colFlags[j].set(val);
      blockFlags[i / 3][j / 3].set(val);
    }
  }

  queue<pair<int, int>> q;

  for (int i = 0; i < 9; i++) {
    for (int j = 0; j < 9; j++) {
      auto totalFlags = rowFlags[i] | colFlags[j] | blockFlags[i / 3][j / 3];
      if (totalFlags.count() == 8 && board[i][j] == '.') {
        q.push(make_pair(i, j));
      }
    }
  }

  while (!q.empty()) {
    int i = q.front().first;
    int j = q.front().second;
    if (board[i][j] != '.') {
      q.pop();
      continue;
    }
    int block_i = i / 3;
    int block_j = j / 3;
    auto totalFlags = rowFlags[i] | colFlags[j] | blockFlags[block_i][block_j];
    if (totalFlags.count() == 8) {
      auto answer = (unsigned) log2(totalFlags.flip().to_ulong());
      board[i][j] = answer + '1';
      rowFlags[i].set(answer);
      colFlags[j].set(answer);
      blockFlags[block_i][block_j].set(answer);
      for (int k = 0; k < 9; k++) {
        q.push(make_pair(i, k));
        q.push(make_pair(k, j));
      }
      for (int m = 0; m < 3; m++) {
        for (int n = 0; n < 3; n++) {
          q.push(make_pair(block_i + m, block_j + n));
        }
      }
    }
    q.pop();
  }
}
```

# 三、进行修正
从之前的尝试中已经得出了结论：需要在无法确定时对所有可能的填法进行尝试。而尝试失败后需要进行回溯，所以使用递归会很清晰。
修正后的算法如下:
```cpp
class Solution {
 public:
  vector<bitset<9>> rowShown;
  vector<bitset<9>> colShown;
  vector<vector<bitset<9>>> blockShown;

  Solution() : rowShown(vector<bitset<9>>(9, bitset<9>())),
               colShown(vector<bitset<9>>(9, bitset<9>())),
               blockShown(vector<vector<bitset<9>>>(3, vector<bitset<9>>(3, bitset<9>()))) {}

  void solveSudoku(vector<vector<char>> &board) {
    for (int i = 0; i < 9; i++) {
      for (int j = 0; j < 9; j++) {
        if (board[i][j] == '.') {
          continue;
        }
        int val = board[i][j] - '1';
        rowShown[i].set(val);
        colShown[j].set(val);
        blockShown[i / 3][j / 3].set(val);
      }
    }
    solve(board);
  }

  bool solve(vector<vector<char>> &board) {
    for (int i = 0; i < 9; i++) {
      for (int j = 0; j < 9; j++) {
        if (board[i][j] == '.') {
          for (char guess = '1'; guess <= '9'; guess++) {
            if (isValid(board, i, j, guess)) {
              setDigit(board, i, j, guess, true);
              if (solve(board)) {
                return true;
              } else {
                setDigit(board, i, j, guess, false);
              }
            }
          }
          return false;
        }
      }
    }
    return true;
  }

  void setDigit(vector<vector<char>> &board, int row, int col, char guess, bool value) {
    board[row][col] = value ? guess : '.';
    rowShown[row].set(guess - '1', value);
    colShown[col].set(guess - '1', value);
    blockShown[row / 3][col / 3].set(guess - '1', value);
  }

  bool isValid(vector<vector<char>> &board, int row, int col, char guess) {
    int pos = guess - '1';
    return !(rowShown[row].test(pos) || colShown[col].test(pos) || blockShown[row / 3][col / 3].test(pos));
  }
};
```
![](/images/LeetCode-37.png)