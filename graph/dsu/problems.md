---
description: 題目選解
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

# 題目選解

### DSU

#### [Graph Valid Tree](https://leetcode.com/problems/graph-valid-tree/)

有效的樹需要滿足兩個條件：首先，圖中不能有環；其次，圖中的所有節點必須是相連的，即從任意一點出發，都能到達圖中的任意其他點。

1. 初始化 DSU ，節點數量為圖中節點的數量。
2. 遍歷圖中的每一條邊。對於每條邊的兩個節點，使用 DSU 中的 `_union` 方法來合併它們所在的集合。
   * 如果在合併之前發現這兩個節點已經在同一個集合中，這代表著添加這條邊會產生一個環，因此圖不可能是一棵樹。
3. 在遍歷完所有邊後，檢查是否只有一個集合。如果有多於一個集合，則代表著圖中的節點不是全部相連的，因此也不是一棵樹。

<pre class="language-cpp"><code class="lang-cpp">// struct DSU ...
<strong>class Solution {
</strong>public:
    bool validTree(int n, vector&#x3C;vector&#x3C;int>>&#x26; edges) {
        DSU dsu(n);
        for (auto&#x26; edge : edges) {
            int u = edge[0], v = edge[1];
            if (dsu.find(u) == dsu.find(v)) {
                return false;
            }
            dsu._union(u, v);
        }
        int root = dsu.find(0);
        for (int i = 1; i &#x3C; n; ++i) {
            if (dsu.find(i) != root) return false;
        }
        return true;
    }
};
</code></pre>

#### [Redundant Connection](https://leetcode.com/problems/redundant-connection/)

1. 初始化一個 DSU ，節點數量設為圖中節點的數量。
2. 遍歷給定的每條邊。對於每條邊的兩個節點，使用 DSU 的 `find` 方法檢查這兩個節點是否已經在同一個集合中。
   * 如果它們已經在同一個集合中，這條邊就是一條冗餘連接，因為在它們之間添加這條邊不會改變節點的連通性。
   * 如果它們不在同一個集合中，則使用 DSU 的 `_union` 方法將它們所在的集合合併。
3. 最後一條檢測到的使兩個節點處於同一集合中的邊就是答案。

```cpp
// struct DSU ...
class Solution {
public:
    vector<int> findRedundantConnection(vector<vector<int>>& edges) {
        int n = edges.size();
        DSU dsu(n + 1);
        for (auto& edge : edges) {
            if (!dsu._union(edge[0], edge[1])) {
                return edge;
            }
        }
        return {};
    }
};
```

#### [Bricks Falling When Hit](https://leetcode.com/problems/bricks-falling-when-hit/)

我們可以反向思考這個問題：從最後一次敲擊開始逆序重建磚塊，看看每次添加磚塊時能讓多少磚塊因為這次添加而與頂部連接。這樣，我們就可以用 DSU 來維護磚塊間的連接關係，特別是它們是否與頂部連接。

1. **初始化 DSU**：為了方便管理磚塊是否與頂部相連，我們可以將網格的頂部看作一個虛擬的節點，所有頂部的磚塊都與這個虛擬節點連接。
2. **反向添加磚塊**：從最後一次敲擊開始，逆序地將磚塊添加回去。對於每次添加，檢查這塊磚四周的其他磚塊，如果已經有與頂部連接的磚塊，則這次添加操作將使得這塊磚以及與它連接的磚塊都與頂部連接。
3. **計算掉落的磚塊數量**：對於每次添加，計算因這次添加而新增加的與頂部連接的磚塊數量（排除這塊剛剛添加的磚塊本身），這就是這次操作對應的掉落磚塊數。

```cpp
struct DSU {
    vector<int> p, sz;
    DSU(int _n) : p(_n), sz(_n, 1) {
        iota(p.begin(), p.end(), 0);
    }
    int find(int x) {
        return x == p[x] ? x : p[x] = find(p[x]);
    }
    bool _union(int x, int y) {
        x = find(x), y = find(y);
        if (x == y) return false;
        if (sz[x] < sz[y]) swap(x, y);
        sz[x] += sz[y];
        p[y] = x;
        return true;
    }
    bool isConnected(int x, int y) {
        return find(x) == find(y);
    }
};

class Solution {
public:
    vector<int> hitBricks(vector<vector<int>>& grid, vector<vector<int>>& hits) {
        int m = grid.size(), n = grid[0].size();
        DSU dsu(m * n + 1); 
        for (auto& hit : hits) {
            if (grid[hit[0]][hit[1]] == 1) {
                grid[hit[0]][hit[1]] = 2; 
            }
        }

        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (grid[i][j] == 1) {
                    int id = i * n + j;
                    if (i == 0) {
                        dsu._union(id, m * n); 
                    }
                    if (i > 0 && grid[i-1][j] == 1) {
                        dsu._union(id, (i-1) * n + j);
                    }
                    if (j > 0 && grid[i][j-1] == 1) {
                        dsu._union(id, i * n + j - 1);
                    }
                }
            }
        }

        vector<vector<int>> dirs = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
        vector<int> res(hits.size(), 0);
        for (int k = hits.size() - 1; k >= 0; --k) {
            int x = hits[k][0], y = hits[k][1];
            if (grid[x][y] == 2) {
                int id = x * n + y;
                int origin = dsu.sz[dsu.find(m * n)]; 
                if (x == 0) {
                    dsu._union(id, m * n);
                }
                for (auto& dir : dirs) {
                    int nx = x + dir[0], ny = y + dir[1];
                    if (nx >= 0 && nx < m && ny >= 0 && ny < n && grid[nx][ny] == 1) {
                        dsu._union(id, nx * n + ny);
                    }
                }
                int newSize = dsu.sz[dsu.find(m * n)]; 
                res[k] = max(0, newSize - origin - 1); 
                grid[x][y] = 1; 
            }
        }

        return res;
    }
};
```

#### [Making A Large Island](https://leetcode.com/problems/making-a-large-island/)

1. **網格掃描**：首先，掃描整個網格，使用 DSU 將相鄰的 1（即相鄰的島嶼部分）合併成一個集合，同時記錄每個集合（即每個島嶼）的大小。
2. **尋找最大島嶼**：通過上一步，我們可以知道將哪一個 0 轉變為 1 可以使得島嶼面積最大化。對於網格中的每個 0，檢查它相鄰的四個方向，看看這些方向上的島嶼屬於哪些不同的集合，然後將這些集合的大小加起來，加上 1（轉變的這個 0 本身），就是將這個 0 轉變為 1 後能得到的島嶼大小。
3. **特殊情況處理**：如果網格中沒有 0，即整個網格都是島嶼，則直接返回整個網格的大小。

```cpp
struct DSU {
    vector<int> p, sz;
    DSU(int n) : p(n), sz(n, 1) {
        for (int i = 0; i < n; ++i) p[i] = i;
    }
    int find(int x) {
        return x == p[x] ? x : p[x] = find(p[x]);
    }
    void _union(int x, int y) {
        x = find(x), y = find(y);
        if (x != y) {
            if (sz[x] < sz[y]) swap(x, y);
            sz[x] += sz[y];
            p[y] = x;
        }
    }
    int size(int x) {
        return sz[find(x)];
    }
};

class Solution {
public:
    int largestIsland(vector<vector<int>>& grid) {
        int n = grid.size(), res = 0;
        DSU dsu(n * n);
        vector<vector<int>> directions = {{0, 1}, {1, 0}, {-1, 0}, {0, -1}};
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                if (grid[i][j] == 1) {
                    for (auto& dir : directions) {
                        int x = i + dir[0], y = j + dir[1];
                        if (x >= 0 && x < n && y >= 0 && y < n && grid[x][y] == 1) {
                            dsu._union(i * n + j, x * n + y);
                        }
                    }
                }
            }
        }
        // 更新最大島嶼面積
        for (int i = 0; i < n * n; ++i) res = max(res, dsu.size(i));
        // 檢查每個 0，看看將其轉變為 1 後的最大島嶼面積
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                if (grid[i][j] == 0) {
                    int area = 1;
                    set<int> connected;
                    for (auto& dir : directions) {
                        int x = i + dir[0], y = j + dir[1];
                        if (x >= 0 && x < n && y >= 0 && y < n && grid[x][y] == 1) {
                            connected.insert(dsu.find(x * n + y));
                        }
                    }
                    for (int id : connected) area += dsu.size(id);
                    res = max(res, area);
                }
            }
        }
        return res;
    }
};
```

#### [Graph Connectivity With Threshold](https://leetcode.com/problems/graph-connectivity-with-threshold/)

對於所有大於`threshold`的整數`i`，將`i`的所有倍數連接起來。這樣，如果兩個數能通過它們的倍數連接到同一個集合，則這兩個數就是連通的。

1. **初始化 DSU**：初始化一個大小為`n + 1`的DSU
2. **連接倍數**：對於每一個整數`i`，從`threshold + 1`開始，將`i`的所有倍數連接起來。即，對於每個`j`從`2 * i`開始，每次增加`i`，直到`n`，使用`_union(i, j)`將它們合併到同一集合。
3. **處理查詢**：對於每一對查詢`(a, b)`，如果`find(a) == find(b)`，則返回`true`表示它們是連通的，否則返回`false`。

```cpp
// struct DSU ...
class Solution {
public:
    vector<bool> areConnected(int n, int threshold, vector<vector<int>>& queries) {
        DSU dsu(n + 1);
        for (int i = threshold + 1; i <= n; ++i) {
            for (int j = 2 * i; j <= n; j += i) {
                dsu._union(i, j);
            }
        }
        
        vector<bool> ans;
        for (auto& query : queries) {
            ans.push_back(dsu.find(query[0]) == dsu.find(query[1]));
        }
        return ans;
    }
};
```

#### [Checking Existence of Edge Length Limited Paths](https://leetcode.com/problems/checking-existence-of-edge-length-limited-paths/)

主要想法是先將邊按照長度從小到大排序，然後逐步將這些邊加入到圖中，同時利用 DSU 維護圖中節點的連通性。對於每個查詢，只考慮長度小於限制的邊，以確保如果兩個節點可以連通，則它們之間就存在符合條件的路徑。

1. **排序邊**：首先，將圖中的所有邊按照長度進行排序。
2. **初始化 DSU**：初始化一個 DSU 結構，用於維護節點間的連通性。
3. **處理查詢**：將查詢按照邊長限制從小到大排序，並逐個處理。
4. **逐步添加邊**：從長度最小的邊開始，逐步將邊加入圖中，同時利用 DSU 合併邊的兩個端點。每當加入一條邊時，檢查所有未處理的查詢，對於每個查詢，如果兩個查詢節點已經通過 DSU 連通，且當前邊的長度小於查詢的限制，則標記該查詢為成功。

```cpp
// struct DSU ...
class Solution {
public:
    vector<bool> distanceLimitedPathsExist(int n, vector<vector<int>>& edgeList, vector<vector<int>>& queries) {
        vector<bool> ans(queries.size(), false);
        vector<pair<int, pair<int, int>>> edges;
        for (auto& e : edgeList) {
            edges.emplace_back(e[2], make_pair(e[0], e[1])); 
        }
        sort(edges.begin(), edges.end());
        
        vector<pair<int, int>> sortedQueries;
        for (int i = 0; i < queries.size(); ++i) {
            sortedQueries.emplace_back(i, queries[i][2]);
        }
        sort(sortedQueries.begin(), sortedQueries.end(), [&](const pair<int, int>& a, const pair<int, int>& b) {
            return a.second < b.second;
        });
        
        DSU dsu(n);
        int j = 0; // edges index
        for (auto& [i, limit] : sortedQueries) {
            while (j < edges.size() && edges[j].first < limit) {
                dsu._union(edges[j].second.first, edges[j].second.second);
                ++j;
            }
            ans[i] = dsu.find(queries[i][0]) == dsu.find(queries[i][1]);
        }
        
        return ans;
    }
};
```

#### [Path With Minimum Effort](https://leetcode.com/problems/path-with-minimum-effort/)

首先計算並排序所有相鄰格子之間的高度差，然後逐步嘗試合併格子，每次合併後檢查網格的左上角和右下角是否已經連通。當找到使得這兩個格子連通的最小高度差時，返回該高度差作為答案。

```cpp
// struct DSU ...
class Solution {
public:
    int minimumEffortPath(vector<vector<int>>& heights) {
        int m = heights.size(), n = heights[0].size();
        vector<tuple<int, int, int>> edges; 
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (j < n - 1) {
                    int diff = abs(heights[i][j] - heights[i][j+1]);
                    edges.emplace_back(diff, i * n + j, i * n + j + 1);
                }
                if (i < m - 1) {
                    int diff = abs(heights[i][j] - heights[i+1][j]);
                    edges.emplace_back(diff, i * n + j, (i + 1) * n + j);
                }
            }
        }
        sort(edges.begin(), edges.end());

        DSU dsu(m * n);
        for (auto& [diff, x, y] : edges) {
            dsu._union(x, y);
            if (dsu.find(0) == dsu.find(m * n - 1)) {
                return diff; 
            }
        }
        return 0; 
    }
};
```

### MST

#### [Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/)

MST 基礎題，這邊 Kruskal 跟 Prim 都寫一次。

Kruskal

1. **初始化 DSU**
2. **處理邊**：計算所有點對之間的曼哈頓距離，並將這些距離作為邊，邊的權重就是距離。
3. **排序邊**：按照邊的權重（即點對之間的距離）從小到大排序。
4. **構建 MST**：遍歷排序後的邊列表，使用 DSU 檢查當前邊的兩個端點是否已連通。如果未連通，則將這兩個端點連接起來，並將邊的權重加到總成本上。
5. **返回總成本**：當所有點都連接後，返回累積的總成本作為答案。

```cpp
// struct DSU ...
class Solution {
public:
    int minCostConnectPoints(vector<vector<int>>& points) {
        int n = points.size();
        DSU dsu(n);
        vector<tuple<int, int, int>> edges;
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < n; ++j) {
                int cost = abs(points[i][0] - points[j][0]) + abs(points[i][1] - points[j][1]);
                edges.emplace_back(cost, i, j);
            }
        }
        
        sort(edges.begin(), edges.end());
        
        int totalCost = 0;
        for (auto& [cost, i, j] : edges) {
            if (dsu.find(i) != dsu.find(j)) {
                dsu._union(i, j);
                totalCost += cost;
            }
        }
        
        return totalCost;
    }
};
```

Prim

1. **使用最小堆**：將所有點以其到已連接部分的距離作為鍵值加入最小堆中。起始時，選擇一個點的距離設為0（作為算法的起始點），其餘點的距離設為無窮大。
2. **更新並選擇點**：每次從最小堆中取出距離最小的點（即堆頂元素），將其加入到已連接部分。然後，遍歷所有未連接的點，更新它們到已連接部分的距離，並調整最小堆。

```cpp
class Solution {
public:
    int minCostConnectPoints(vector<vector<int>>& points) {
        int n = points.size();
        auto dist = [&](int i, int j) {
            return abs(points[i][0] - points[j][0]) + abs(points[i][1] - points[j][1]);
        };

        vector<bool> visited(n, false);
        priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
        pq.push({0, 0}); 
        int totalCost = 0;
        int edges = 0;

        while (!pq.empty() && edges < n) {
            auto [cost, i] = pq.top();
            pq.pop();
            if (visited[i]) continue;
            visited[i] = true;
            totalCost += cost;
            edges++;

            for (int j = 0; j < n; ++j) {
                if (!visited[j]) {
                    pq.push({dist(i, j), j});
                }
            }
        }

        return totalCost;
    }
};
```

#### [Find Critical and Pseudo-Critical Edges in Minimum Spanning Tree](https://leetcode.com/problems/find-critical-and-pseudo-critical-edges-in-minimum-spanning-tree/) 

我們可以使用 Kruskal 算法來尋找最小生成樹，並通過以下步驟來識別關鍵邊和偽關鍵邊：

1. **找出最小生成樹的權重**：首先，使用 Kruskal 算法找出不考慮任何特定邊時的最小生成樹的總權重。
2. **識別關鍵邊**：對於每條邊，嘗試不包括這條邊，然後再次使用 Kruskal 算法計算最小生成樹的總權重。如果這時的總權重增加，或者無法形成最小生成樹，則這條邊是關鍵邊。
3. **識別偽關鍵邊**：對於不是關鍵邊的邊，將該邊先加入最小生成樹中，然後使用 Kruskal 算法計算剩餘邊構建最小生成樹的總權重。如果加入這條邊後仍能構建出原最小生成樹的總權重，則這條邊是偽關鍵邊。

```cpp
// struct DSU ...
class Solution {
public:
    vector<vector<int>> findCriticalAndPseudoCriticalEdges(int n, vector<vector<int>>& edges) {
        int m = edges.size();
        for (int i = 0; i < m; ++i) {
            edges[i].push_back(i); 
        }
        sort(edges.begin(), edges.end(), [](const vector<int>& a, const vector<int>& b) {
            return a[2] < b[2];
        });

        auto kruskal = [&](int banned, int must) {
            DSU dsu(n);
            int weight = 0, count = 0;
            if (must >= 0) {
                weight += edges[must][2];
                dsu._union(edges[must][0], edges[must][1]);
                count++;
            }
            for (int i = 0; i < m; ++i) {
                if (i == banned) continue;
                if (dsu._union(edges[i][0], edges[i][1])) {
                    weight += edges[i][2];
                    count++;
                }
            }
            return count == n - 1 ? weight : INT_MAX;
        };

        int base_weight = kruskal(-1, -1);
        vector<int> critical, pseudo_critical;
        for (int i = 0; i < m; ++i) {
            if (kruskal(i, -1) > base_weight) { 
                critical.push_back(edges[i][3]);
            } else if (kruskal(-1, i) == base_weight) { 
                pseudo_critical.push_back(edges[i][3]);
            }
        }

        return {critical, pseudo_critical};
    }
};
```

