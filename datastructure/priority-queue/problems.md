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

### Dijkstra

#### [Network Delay Time](https://leetcode.com/problems/network-delay-time/)

這是一題 Dijkstra 的裸題，只要按照前面演算法講得好好實作就可以了。

```cpp
class Solution {
public:
  const int inf = 0x3f3f3f3f;
  typedef pair<int, int> pii;
  vector<vector<pii>> g;
  int networkDelayTime(vector<vector<int>>& times, int n, int k) {
    g.resize(n + 1, vector<pair<int, int>>());
    for (auto &e : times) {
      g[e[0]].push_back({e[1], e[2]});
    }

    vector<int> dist(n + 1, inf);
    priority_queue<pii, vector<pii>, greater<pii>> pq;
    dist[k] = 0;
    pq.push({0, k});
    while (!pq.empty()) {
      auto [dis, u] = pq.top();
      pq.pop();
      if (dis != dist[u]) continue;
      for (auto [v, w] : g[u]) {
        if (dist[u] + w < dist[v]) {
          dist[v] = dist[u] + w;
          pq.push({dist[v], v});
        }
      }
    }
    int mx = 0;
    for (int i = 1; i <= n; i++) {
      if (dist[i] == inf) return -1;
      mx = max(mx, dist[i]);
    }
    return mx;
  }
};
```

#### [Path with Maximum Probability](https://leetcode.com/problems/path-with-maximum-probability/)

在這個問題中，我們不是尋找最短路徑，而是尋找最大機率的路徑。使用優先隊列可以幫助我們每次都選擇當前機率最高的節點來進行下一步的探索。這樣可以保證我們總是在嘗試最有可能達到目的地的路徑。

從起點開始，不斷更新與當前節點相鄰節點的成功機率。如果通過當前節點到達相鄰節點的機率高於已知的機率，則更新這個機率，並將相鄰節點加入優先隊列。當優先隊列為空，或者已經找到終點的最大機率時，就代表結束了。

```cpp
class Solution {
public:
    double maxProbability(int n, vector<vector<int>>& edges, vector<double>& succProb, int start, int end) {
        vector<vector<pair<int, double>>> g(n);
        for (int i = 0; i < edges.size(); ++i) {
            g[edges[i][0]].push_back({edges[i][1], succProb[i]});
            g[edges[i][1]].push_back({edges[i][0], succProb[i]});
        }

        vector<double> prob(n, 0.0);
        priority_queue<pair<double, int>> pq;
        pq.push({1.0, start});
        prob[start] = 1.0;

        while (!pq.empty()) {
            auto [currProb, u] = pq.top();
            pq.pop();

            if (u == end) return currProb;
            if (currProb < prob[u]) continue;

            for (auto &[v, p] : g[u]) {
                double nextProb = currProb * p;
                if (nextProb > prob[v]) {
                    prob[v] = nextProb;
                    pq.push({nextProb, v});
                }
            }
        }

        return 0.0;
    }
};
```

#### [Find the City With the Smallest Number of Neighbors at a Threshold Distance](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/)

這個問題可以用 Floyd-Warshall 或者 Dijkstra 解決，但是這邊在講 priority queue 所以先用 Dijkstra 解。

1. **對每個城市跑 Dijkstra：**計算從每個城市出發到達所有其他城市的最短路徑。
2. **計算符合閾值的鄰居數量：**對於每個城市，計算在給定閾值距離內可以到達的其他城市數量。
3. **選擇最少鄰居的城市：**找出具有最少鄰居數量的城市。如果有多個這樣的城市，選擇編號最大的一個。

```cpp
class Solution {
public:
    int findTheCity(int n, vector<vector<int>>& edges, int distanceThreshold) {
        const int inf = 1e9;
        vector<vector<pair<int, int>>> g(n);

        for (auto& e : edges) {
            g[e[0]].push_back({e[1], e[2]});
            g[e[1]].push_back({e[0], e[2]});
        }

        auto dijkstra = [&](int start) {
            vector<int> dist(n, inf);
            priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
            dist[start] = 0;
            pq.push({0, start});

            while (!pq.empty()) {
                auto [d, u] = pq.top();
                pq.pop();

                if (d > dist[u]) continue;

                for (auto [v, w] : g[u]) {
                    if (dist[u] + w < dist[v]) {
                        dist[v] = dist[u] + w;
                        pq.push({dist[v], v});
                    }
                }
            }
            return dist;
        };

        int city = 0, minNeighbour = n;
        for (int i = 0; i < n; ++i) {
            auto dist = dijkstra(i);
            int count = count_if(dist.begin(), dist.end(), [&](int d) { return d <= distanceThreshold; });
            if (count <= minNeighbour) {
                city = i;
                minNeighbour = count;
            }
        }

        return city;
    }
};
```

這樣的時間複雜度是 $$O(n^2\log(n))$$，比起 Floyd Warshall $$O(n^3)$$ 快。

### 其他

#### [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)

這題可以先單純地計算出每個數字的頻率，再放進 priority queue 看最大的前 k 個是誰就可以了。

```cpp
class Solution {
public:
    typedef pair<int, int> pii;
    vector<int> topKFrequent(vector<int>& nums, int k) {
        priority_queue<pii> pq;
        map<int, int> mp;
        for (int x : nums) {
            mp[x]++;
        }
        for (auto [x, f] : mp) {
            pq.push({f, x});
        }
        vector<int> res;
        while (k--) {
            pii p = pq.top();
            pq.pop();
            res.push_back(p.second);
        }
        return res;
    }
};
```

#### [Last Stone Weight](https://leetcode.com/problems/last-stone-weight/)

因為需要在每一輪中快速找到兩塊最重的石頭，使用最大堆的優先隊列 priority queue 可以非常高效地進行這一操作，因為它可以在 O(1) 的時間複雜度內返回最大元素，並且在 O(log n) 的時間複雜度內插入和刪除元素。

```cpp
class Solution {
public:
    int lastStoneWeight(vector<int>& stones) {
        priority_queue<int> pq(stones.begin(), stones.end());

        while (pq.size() > 1) {
            int y = pq.top();
            pq.pop();
            int x = pq.top();
            pq.pop();

            if (x != y) {
                pq.push(y - x);
            }
        }

        return pq.empty() ? 0 : pq.top();
    }
};
```

#### [The Skyline Problem](https://leetcode.com/problems/the-skyline-problem/)

當我們掃描每個建築物時，我們需要知道當前最高的建築物是什麼。優先隊列能夠讓我們快速得知當前的最大高度，並且當建築物的起點或終點被遇到時，能夠迅速加入或移除建築物。

大致的步驟如下：

1. **處理建築物**: 將每個建築物分成兩部分 - 開始和結束，並將它們存入一個列表中。
2. **排序**: 將這個列表按照 x 值排序。如果 x 值相同，則高的建築物優先。
3. **使用優先隊列**: 遍歷排序後的列表，並使用優先隊列跟蹤當前的最大高度。當遇到建築物開始時，將其高度加入優先隊列；當遇到建築物結束時，將其高度從優先隊列中移除。
4. **生成天際線**: 每次當最大高度改變時，將當前位置和高度加入到天際線中。

```cpp
class Solution {
public:
    vector<vector<int>> getSkyline(vector<vector<int>>& buildings) {
        vector<pair<int, int>> heights;
        for (const auto& b : buildings) {
            heights.emplace_back(b[0], -b[2]); // start
            heights.emplace_back(b[1], b[2]); // end
        }

        sort(heights.begin(), heights.end());
        multiset<int> pq;
        pq.insert(0);
        vector<vector<int>> skyline;
        int prev = 0;

        for (const auto& h : heights) {
            if (h.second < 0) {
                pq.insert(-h.second);
            } else {
                pq.erase(pq.find(h.second));
            }
            int curr = *pq.rbegin();
            if (curr != prev) {
                skyline.push_back({h.first, curr});
                prev = curr;
            }
        }

        return skyline;
    }
};
```
