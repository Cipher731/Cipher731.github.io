---
title: LeetCode 23. Merge k Sorted Lists
date: 2018-09-08 11:08:55
tags: C++
categories: LeetCode
mathjax: true
---

# 一、题目要求
Merge k sorted linked lists and return it as one sorted list.
```
Input:
[
  1->4->5,
  1->3->4,
  2->6
]
Output: 1->1->2->3->4->4->5->6
```
<!-- more -->

# 二、分析题目
最简单的解法是取出所有节点的值，对所有数据进行排序，因此这种方法的时间复杂度取决于排序算法的时间复杂度，最好为{% mathjax %}O(NlogN){% endmathjax %}

其次比较容易想到的解法就是对每个链表维护一个指针，每轮对每个链表的当前值进行比较。
这种方法下每输出一个节点比较k个数据，设总计N个节点，因此时间复杂度为$O(kN)$

做题时受到`mergeKLists`函数命名的启发，想到可以一次合并两个链表，再用分治的方法可以以{% mathjax %}O(Nlogk){% endmathjax %}的时间复杂度解决问题。

# 三、具体实现
实现`merge2Lists`的函数用于合并两个链表
```cpp
ListNode *merge2Lists(ListNode *list1, ListNode *list2) {
  auto head = new ListNode(0);
  auto iter = head;
  while (list1 && list2) {
    if (list1->val <= list2->val) {
      iter->next = list1;
      list1 = list1->next;
    } else {
      iter->next = list2;
      list2 = list1;
      list1 = iter->next->next;
    }
    iter = iter->next;
  }
  if (list1) {
    iter->next = list1;
  } else {
    iter->next = list2;
  }
  return head->next;
}
```

接着实现`mergeKLists`的函数用于合并所有链表。
不过一开始实现这个函数的时候脑子一抽，写成了下面这种形式，成功地让时间复杂度达到了$O(kN^2)$，比直接暴力还慢，取得了反向登顶的成就。  
![](/images/LeetCode-23-1.png)
```cpp
ListNode *mergeKLists(vector<ListNode *> &lists) {
  if (lists.empty()) {
    return nullptr;
  }
  for (int i = 0; i + 1 < lists.size(); i++) {
    lists[i + 1] = merge2Lists(lists[i], lists[i + 1]);
  }
  return lists[lists.size() - 1];
}
```

那么正确的的实现方法如下

```cpp
ListNode *mergeKLists(vector<ListNode *> &lists) {
  if (lists.empty()) {
    return nullptr;
  }
  int level = 1;
  while (level < lists.size()) {
    for (int i = 0; i + level < lists.size(); i += 2 * level) {
      lists[i] = merge2Lists(lists[i], lists[i + level]);
    }
    level *= 2;
  }
  return lists[0];
}
```

花费的时间
![](/images/LeetCode-23-2.png)
