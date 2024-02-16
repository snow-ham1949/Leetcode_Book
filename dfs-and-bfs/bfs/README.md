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

# BFS（廣度優先搜尋）

廣度優先搜尋則可以想成是一層一層去擴展可能性，達到特定的目標（例如遍歷完整張圖、或者找到目標等等）之後停止。

可以參考 Wikipedia 上面的 gif 來區分 DFS 跟 BFS

<figure><img src="../../.gitbook/assets/Depth-First-Search.gif" alt=""><figcaption><p>DFS</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/Breadth-First-Search-Algorithm.gif" alt=""><figcaption><p>BFS</p></figcaption></figure>

我自己覺得 BFS 可以有一種特定的模板，BFS 通常在 C++ 裡面就是搭配 queue，queue 裡面會存當前節點的訊息。不熟 queue 的可以先看[這裡](../../datastructure/queue/)

```cpp
queue<Info> q;
while (!q.empty()) {
    Info info = q.front();
    q.pop();
    for (Info nei : info.neighbors) {
        // do something    
    }
}
```

主要可以用在

* 拓展狀態
* 樹/圖上搜尋
