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

### Floyd-Warshall

#### [Find the City With the Smallest Number of Neighbors at a Threshold Distance](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/)

1. 初始化距離矩陣，如果存在邊`from_i`到`to_i`，則`dist[from_i][to_i] = weight_i`，否則，如果`from_i`不等於`to_i`，則`dist[from_i][to_i] = 無窮大`，對於所有`i`，`dist[i][i] = 0`。
2. 使用Floyd-Warshall算法更新距離矩陣，具體為：對於每個節點`k`，遍歷矩陣中的每個元素`dist[i][j]`，如果`dist[i][k] + dist[k][j] < dist[i][j]`，則更新`dist[i][j] = dist[i][k] + dist[k][j]`。
3. 當所有節點被考慮作為中間節點後，遍歷每個城市`i`，計算有多少個城市`j`（`j != i`）滿足`dist[i][j] <= distanceThreshold`。找到擁有最小鄰居數量且編號最大的城市。

(這題也可以用 dijkstra 做）

```cpp
class Solution {
public:
    int findTheCity(int n, vector<vector<int>>& edges, int distanceThreshold) {
        vector<vector<int>> dist(n, vector<int>(n, 1e9));
        for (const auto& e : edges) {
            dist[e[0]][e[1]] = dist[e[1]][e[0]] = e[2];
        }
        for (int i = 0; i < n; ++i) dist[i][i] = 0;

        // Floyd-Warshall算法
        for (int k = 0; k < n; ++k) {
            for (int i = 0; i < n; ++i) {
                for (int j = 0; j < n; ++j) {
                    dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
                }
            }
        }

        int minCount = n, ans = 0;
        for (int i = 0; i < n; ++i) {
            int count = 0;
            for (int j = 0; j < n; ++j) {
                if (i != j && dist[i][j] <= distanceThreshold) ++count;
            }
            if (count <= minCount) {
                minCount = count;
                ans = i; 
            }
        }

        return ans;
    }
};
```

#### [Network Delay Time](https://leetcode.com/problems/network-delay-time/)

這題一樣可以用 dijkstra 做，但我們試著用 floyd-warshall 做。算從節點`k`到所有其他節點的最短路徑，然後找出這些路徑中的最長時間。首先初始化一個距離矩陣，然後使用Floyd-Warshall算法更新這個矩陣，最後計算從節點`k`到所有其他節點的最大距離，即信號到達所有節點所需的最長時間。如果有節點無法從`k`接收到信號，則返回`-1`。

```cpp
class Solution {
public:
    int networkDelayTime(vector<vector<int>>& times, int n, int k) {
        const int INF = 1e9;
        vector<vector<int>> dist(n, vector<int>(n, INF));
        
        for (int i = 0; i < n; ++i) dist[i][i] = 0;
        for (const auto& time : times) {
            dist[time[0] - 1][time[1] - 1] = time[2];
        }
        
        // Floyd-Warshall算法
        for (int i = 0; i < n; ++i)
            for (int j = 0; j < n; ++j)
                for (int k = 0; k < n; ++k)
                    dist[j][k] = min(dist[j][k], dist[j][i] + dist[i][k]);
        
        int maxDist = 0;
        for (int i = 0; i < n; ++i) {
            if (dist[k - 1][i] == INF) return -1;
            maxDist = max(maxDist, dist[k - 1][i]);
        }
        
        return maxDist;
    }
};
```

### Bellman-Ford

#### [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/)

我們需要稍微修改傳統的Bellman-Ford算法，以限制最多通過`k`站的條件。首先初始化一個最小成本陣列`cost`，對於每一次從`0`到`K`的迭代，它都會根據航班信息更新到達每個城市的最小成本。這裡使用了一個臨時數組`tempCost`來儲存每輪迭代的結果，以防止當前輪次的更新影響到其他邊的松弛操作。最終，如果`cost[dst]`不是`INT_MAX`，則返回`cost[dst]`；否則，返回`-1`表示無法達到目的地。

```cpp
class Solution {
public:
    int findCheapestPrice(int n, vector<vector<int>>& flights, int src, int dst, int K) {
        vector<int> cost(n, INT_MAX);
        cost[src] = 0;
        for (int i = 0; i <= K; ++i) {
            vector<int> tempCost(cost); 
            for (const auto& flight : flights) {
                int u = flight[0], v = flight[1], w = flight[2];
                if (cost[u] == INT_MAX) continue; 
                tempCost[v] = min(tempCost[v], cost[u] + w);
            }
            cost = tempCost;
        }
        
        return cost[dst] == INT_MAX ? -1 : cost[dst];
    }
};
```

#### [Network Delay Time](https://leetcode.com/problems/network-delay-time/)

首先初始化一個距離陣列`dist`來記錄從源節點`k`到每個其他節點的最短路徑，並將所有距離初始化為無窮大（除了源節點到自己的距離為0）。然後，通過重複遍歷所有邊並更新距離陣列來進行松弛操作。最後，遍歷距離陣列以找到最長的最短路徑時間。如果所有節點都可以到達，返回這個時間；否則返回`-1`。

```cpp
class Solution {
public:
    int networkDelayTime(vector<vector<int>>& times, int n, int k) {
        const int INF = 1e9;
        vector<int> dist(n + 1, INF); 
        dist[k] = 0;
        
        // Bellman-Ford算法
        for (int i = 0; i < n; ++i) {
            bool anyUpdate = false;
            for (const auto& t : times) {
                int u = t[0], v = t[1], w = t[2];
                if (dist[u] < INF && dist[v] > dist[u] + w) {
                    dist[v] = dist[u] + w;
                    anyUpdate = true;
                }
            }
            if (!anyUpdate) break;
        }
        
        int maxTime = 0;
        for (int i = 1; i <= n; ++i) {
            if (dist[i] == INF) return -1; 
            maxTime = max(maxTime, dist[i]);
        }
        return maxTime;
    }
};
```
