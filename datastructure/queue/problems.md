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

# 題目選解

### Topological Sort （拓墣排序）

#### [Course Schedule](https://leetcode.com/problems/course-schedule/)

```cpp
class Solution {
public:
    bool canFinish(int numCourses, std::vector<std::vector<int>>& prerequisites) {
        vector<vector<int>> g(numCourses);
        vector<int> in(numCourses, 0);
        for (const auto& p : prerequisites) {
            g[p[1]].push_back(p[0]);
            ++in[p[0]];
        }
        queue<int> q;
        for (int i = 0; i < numCourses; ++i) {
            if (in[i] == 0) q.push(i);
        }
        while (!q.empty()) {
            int u = q.front();
            q.pop();
            for (int v : g[u]) {
                if (--in[v] == 0) q.push(v);
            }
        }
        for (int i = 0; i < numCourses; ++i) {
            if (in[i] != 0) return false;
        }
        return true;
    }
};

```

#### [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/)

這題是 Course Schedule 的延伸，不僅要判斷是否能完成所有課程，還要提供一個完成課程的順序。這就直接使用了拓撲排序的結果。

```cpp
class Solution {
public:
    vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
        vector<vector<int>> g(numCourses);
        vector<int> in(numCourses, 0);
        for (auto &v : prerequisites) {
            g[v[1]].push_back(v[0]);
            in[v[0]]++;
        }
        vector<int> que;
        for (int i = 0; i < numCourses; i++) {
            if (in[i] == 0) que.push_back(i);
        }
        for (int i = 0; i < (int)que.size(); i++) {
            int u = que[i];
            for (int v : g[u]) {
                if (--in[v] == 0) que.push_back(v);
            }
        }
        if ((int)que.size() == numCourses) return que;
        else return {};
    }
};
```

#### [Alien Dictionary](https://leetcode.com/problems/alien-dictionary/)

在這個問題中，給定一組外星語單詞，要根據這些單詞推斷字母的順序。這可以透過比較相鄰單詞找出字符間的相對順序，從而建立圖中的邊，並運用拓撲排序來確定字母的正確順序。

這邊我使用 map 來存圖以及紀錄入度點數，需要特別注意的是，如果後者是前者的 prefix，那代表不能找出一個合理的順序。

```cpp
class Solution {
public:
    string alienOrder(vector<string>& words) {
        map<char, set<char>> g;
        map<char, int> deg;
        for (auto s : words) {
            for (auto c : s) {
                deg[c] = 0;
            }
        }
        int n = words.size();
        for (int i = 0; i < n - 1; i++) {
            string s = words[i], t = words[i + 1];
            if (s.size() > t.size()) {
                int sz = t.size(), flag = 0;
                for (int j = 0; j < sz; j++) {
                    if (s[j] != t[j]) {
                        flag = 1;
                        break;
                    }
                }
                if (flag == 0) return "";
            }
            int sz = min(s.size(), t.size());
            for (int j = 0; j < sz; j++) {
                if (s[j] != t[j]) {
                    if (g[s[j]].find(t[j]) == g[s[j]].end()) {
                        g[s[j]].insert(t[j]);
                        deg[t[j]]++;
                    }
                    break;
                }
            }
        }
        queue<char> q;
        for (auto &c : deg) {
            if (c.second == 0) q.push(c.first);
        }
        string res = "";
        while (!q.empty()) {
            char u = q.front(); q.pop();
            res.push_back(u);
            if (g[u].size() != 0) {
                for (auto v : g[u]) {
                    --deg[v];
                    if (deg[v] == 0) q.push(v);
                }
            }
        }
        return res.size() == deg.size() ? res : "";
    }
};
```

### Deque

#### [Longest Absolute File Path](https://leetcode.com/problems/longest-absolute-file-path/)

len 是用來追蹤目錄或文件名的長度，level 將用來追蹤深度。

每當我們遇到一個 '\n' 字符時，我們知道目錄的名字已經結束，所以我們可以將目錄的長度推入 deque，它只會儲存我們正在處理中的層次結構，所以如果 level 小於 deque 的大小，代表 deque 現在包含同一層曾經處理的目錄或檔案，所以必須 pop out。

最後，當遇到一個 . 字元時，這是一個文件，所以我們可以透過將 deque 中所有目錄名的長度相加來嘗試將其作為結果，我們將在最終結果中加上 level 作為 '/'，以及加上當前文件名的 len 來取 max。

```cpp
class Solution {
public:
    int lengthLongestPath(string input) {
        deque<int> dq;
        int len = 0, res = 0, level = 0, isFile = 0;
        for (char c : input) {
            if(c == '\n') {
                dq.push_back(len);
                level = len = isFile = 0;
            }
            else if (c == '\t') ++level;
            else if (c == '.') { 
                ++len;
                isFile = 1;
            }
            else {
                ++len;
                if(isFile) { 
                    res = max(res, len + accumulate(dq.begin(), dq.end(), 0) + level);
                }
                while (level < dq.size()) {
                    dq.pop_back();
                }
            }
        }        
        return res;
    }
};

```

### 其他

#### [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/)

這是一個經典的圖搜索問題，可以使用 BFS 來解決。在這個問題中，每次擴展節點時，都使用 queue 來追踪下一個可能的飛行路徑，並保持追踪目前為止的花費以及經過的停靠次數，然後在達到 K 次停靠或找到目的地時終止搜索。

```cpp
class Solution {
public:
    using T = array<int, 3>;
    int findCheapestPrice(int n, vector<vector<int>>& flights, int src, int dst, int k) {
        vector<vector<pair<int, int>>> g(n, vector<pair<int, int>>());
        for (auto &v : flights) {
            g[v[0]].push_back({v[1], v[2]});
        }
        vector<int> dist(n, 1e9);
        queue<pair<int, int>> q;
        q.push({src, 0});
        int stop = 0;
        while (stop <= k && !q.empty()) {
            int sz = q.size();
            for (int i = 0; i < sz; i++) {
                auto [u, dis] = q.front();
                q.pop();
                for (auto [v, w] : g[u]) {
                    if (dist[v] > dis + w) {
                        dist[v] = dis + w;
                        q.push({v, dist[v]});
                    }
                }
            }
            stop++;
        }
        return dist[dst] == 1e9 ? -1 : dist[dst];
    }
};

```
