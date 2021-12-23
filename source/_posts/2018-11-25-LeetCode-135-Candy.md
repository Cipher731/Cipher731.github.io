---
title: LeetCode 135. Candy
tags: C++
categories: LeetCode
mathjax: true
date: 2018-11-25 18:48:16
---

# 一、题目要求

There are N children standing in a line. Each child is assigned a rating value.

You are giving candies to these children subjected to the following requirements:

Each child must have at least one candy.
Children with a higher rating get more candies than their neighbors.
What is the minimum candies you must give?

Example 1:

```
Input: [1,0,2]
Output: 5
Explanation: You can allocate to the first, second and third child with 2, 1, 2 candies respectively.
```

Example 2:

```
Input: [1,2,2]
Output: 4
Explanation: You can allocate to the first, second and third child with 1, 2, 1 candies respectively.
             The third child gets 1 candy because it satisfies the above two conditions.
```

<!--more-->

# 二、题目分析

## 1、暴力算法

这是一道比较有意思的应用题。
目标是给一群具有分数的儿童分糖果，要求是（1）人人至少分到一颗糖果，（2）如果分数比相邻儿童高，则需要分到更多的糖果。
先提出一种最直观的算法，我们可以根据第一条规则，先给所有儿童分一颗糖，接下来根据第二条规则，按每个儿童与相邻儿童的分数关系，不断迭代增加所有儿童的糖果数直到分的糖果数不再发生变化，最后对所有儿童的糖果进行求和即可。
接下来分析这种暴力算法需要进行几次迭代，从最终拥有最少糖果数的儿童看，每一轮迭代只能向相邻一位传递信息，使之增加糖果数，考虑最差情况，最少糖果在内循环的末尾，最多糖果在内循环的开始，就需要n次迭代（加上没有产生变化的那次循环）才能够稳定下来退出循环。
因此最终得到暴力算法时间复杂度为$O(N^2)$，空间复杂度为$O(N)$，一个大小为N的数组。

## 2、改进

现在来思考一种改进的方法，暴力算法主要存在的问题可以从最差情况一眼看出来，那就是如果糖果增加的顺序和内循环的顺序相反时，效率就极为低下。
因此如果从两端各自进行遍历的话，就能一举解决这个问题。那么新的问题是如何处理两轮遍历得到的糖果数，应该以哪个为最终结果呢？
由于规则只限制下界，因此只要取两者最大值，就能够得到符合任何一边的糖果数了。
那么这种算法的时间复杂度为$O(N)$，只需要进行两轮迭代就可以了，空间复杂度也为$O(N)$，两个大小为N的数组分别用于左右遍历。

# 三、算法实现

```cpp
int candy(vector<int>& ratings) {
  int sum = 0;
  int left[ratings.size()]{};
  int right[ratings.size()]{};
  for (int i = 1; i < ratings.size(); i++) {
    if (ratings[i] > ratings[i - 1]) {
      left[i] = left[i - 1] + 1;
    }
  }
  for (int i = ratings.size() - 2; i >= 0; i--) {
    if (ratings[i] > ratings[i + 1]) {
      right[i] = right[i + 1] + 1;
    }
  }
  for (int i = 0; i < ratings.size(); i++) {
    sum += max(left[i], right[i]);
  }
  return sum + ratings.size(); //不用填充1的trick
}
```