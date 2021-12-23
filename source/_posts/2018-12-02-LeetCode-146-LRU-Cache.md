---
title: LeetCode 146. LRU Cache
tags: C++
categories: LeetCode
mathjax: true
date: 2018-12-02 18:40:40
---
# 一、题目要求
Design and implement a data structure for Least Recently Used (LRU) cache. It should support the following operations: `get` and `put`.

`get(key)` - Get the value (will always be positive) of the key if the key exists in the cache, otherwise return -1.
`put(key, value)` - Set or insert the value if the key is not already present. When the cache reached its capacity, it should invalidate the least recently used item before inserting a new item.

Follow up:
Could you do both operations in $O(1)$ time complexity?

Example:
```
LRUCache cache = new LRUCache( 2 /* capacity */ );

cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // returns 1
cache.put(3, 3);    // evicts key 2
cache.get(2);       // returns -1 (not found)
cache.put(4, 4);    // evicts key 1
cache.get(1);       // returns -1 (not found)
cache.get(3);       // returns 3
cache.get(4);       // returns 4
```
<!--more-->
# 二、题目分析
题目要求提到可以在$O(1)$的时间复杂度里完成缓存的写和读的操作，那就是在明示用哈希表来处理值的读取，然后对于LRU的维护就采用队列就好了。
但是问题来了，如果缓存被访问，要更新LRU队列，要如何在$O(1)$的时间复杂度里完成呢？
为了不需要遍历整个队列就找到目标，在哈希表中就要同时保存键对应的值和它在LRU队列中的位置，方便更新的时候直接访问。
至于队列的实现，用数组来实现的话，难免产生昂贵的删除元素的开销。因此采用链表实现，而又有仅通过一个节点指针来删除该节点的需求，所以应当采用双向链表。
本来是想省事直接用STL里的轮子来做，不过好久没有用迭代器，跌跌碰碰写了比较长的时间，完全自己来实现估计也差不多了。
得益于`list`的迭代器可以双向移动，所以它就相当于一个双向链表，哈希表就用`unordered_map`来替代。
# 三、算法实现
```cpp
class LRUCache {
 public:
  LRUCache(int capacity) : _capacity(capacity) {}

  int get(int key) {
    auto it = _lookup.find(key);
    if (it != _lookup.end()) {
      update(it);
      return it->second.first;
    } else {
      return -1;
    }
  }

  void put(int key, int value) {
    auto it = _lookup.find(key);
    if (it != _lookup.end()) {
      update(it);
    } else {
      if (_activeCache.size() == _capacity) {
        _lookup.erase(_activeCache.back());
        _activeCache.pop_back();
      }
      _activeCache.push_front(key);
    }
    _lookup[key] = make_pair(value, _activeCache.begin());
  }

 private:
  typedef pair<int, list<int>::iterator> ValAndPos;
  typedef unordered_map<int, ValAndPos> LookUp;

  int _capacity;
  list<int> _activeCache;
  LookUp _lookup;

  void update(LookUp::iterator it) {
    int key = it->first;
    auto pos = it->second.second;
    _activeCache.erase(pos);
    _activeCache.push_front(key);
    it->second.second = _activeCache.begin();
  }
};

```