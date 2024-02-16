---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Stack

一種後進先出（LIFO, Last In First Out）的資料結構，只允許在一端（通常是頂部）進行添加和移除元素。它的基本操作包括 push（添加元素到頂部）和 pop（移除頂部元素）。

堆疊常用於演算法中實現遞迴處理（DFS）、反轉資料、括號匹配檢查等等。

C++ 中有 STL 可以用，用法如下。

```cpp
stack<int> stk;
stk.push(9); // O(1)
stk.pop(); // O(1)
int tp = stk.top(); // O(1)
```
