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

# Disjoint Set Union (DSU)

並查集是一種有效管理元素分組的資料結構。它支持兩種操作：搜尋（Find）和合併（Union）。

* 搜尋（Find）可以找到元素屬於哪個集合
* 合併（Union）可以把兩個集合合併在一起。

DSU可以應用在最小生成樹 (MST) 算法（例如Kruskal算法）和圖的連通性分析。可以透過路徑壓縮跟按照 rank 或者 size 合併來達到近乎常數的時間複雜度。

這裡是模板。

```cpp
struct DSU {
    int n;
    vector<int> p, sz;
    DSU(int _n) : n(_n), sz(n, 1) {
        p.resize(_n);
        iota(p.begin(), p.end(), 0);
    }
    int find(int x) {
        return x == p[x] ? x : p[x] = find(p[x]);
    }
    void _union(int x, int y) {
        x = find(x), y = find(y);
        if (x == y) return;
        if (sz[x] > sz[y]) swap(x, y);
        sz[y] += sz[x];
        p[x] = y;
    }
};
```
