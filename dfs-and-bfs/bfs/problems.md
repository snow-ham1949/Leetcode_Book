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

### 狀態搜尋

#### [Word Ladder](https://leetcode.com/problems/word-ladder/)

BFS 從起始單詞開始，探索所有可能的一步變換，然後所有可能的兩步變換，依此類推，直到找到終止單詞或者探索完所有可能的變換。每一步探索都試圖變換當前單詞的每一個字母，將其替換為其他 25 個可能的字母，並檢查結果是否在 wordList 中。

1. 將 wordList 放入一個集合中，以便快速查找。
2. 使用一個隊列來進行 BFS，隊列中的每個元素包含當前的單詞以及到達該單詞的轉換步數。
3. 從起始單詞開始，對每個字母進行 26 次嘗試（a 到 z），生成新的單詞。如果新單詞在 wordList 中且未被訪問過，則將其加入隊列。
4. 如果在任何時候生成的新單詞等於終止單詞，則立即返回當前步數 。
5. 如果隊列變空，表示無法從起始單詞變換到終止單詞，返回 0。

```cpp
class Solution {
public:
    int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
        set<string> wordSet(wordList.begin(), wordList.end());
        if (wordSet.find(endWord) == wordSet.end()) {
            return 0;
        }
        
        queue<pair<string, int>> q;
        q.push({beginWord, 1});
        
        while (!q.empty()) {
            auto [word, level] = q.front();
            q.pop();
            
            if (word == endWord) {
                return level;
            }
            
            wordSet.erase(word); 
            
            for (int i = 0; i < word.size(); ++i) {
                char originalChar = word[i];
                for (char c = 'a'; c <= 'z'; ++c) {
                    word[i] = c;
                    if (wordSet.find(word) != wordSet.end()) {
                        q.push({word, level + 1});
                        wordSet.erase(word); 
                    }
                }
                word[i] = originalChar; 
            }
        }
        
        return 0; 
    }
};
```

#### [Word Ladder II](https://leetcode.com/problems/word-ladder-ii/)

這題還要求了轉換序列，所以我們要想辦法回朔過程，因此在做 BFS 的時候，加上 parent 來記錄是從哪一個單詞走過來的。BFS 做完之後，再用 DFS 去回朔過程就可以了。

```cpp
class Solution {
public:
    vector<vector<string>> findLadders(string beginWord, string endWord, vector<string>& wordList) {
        unordered_set<string> wordSet(wordList.begin(), wordList.end());
        vector<vector<string>> ladders;
        if (wordSet.find(endWord) == wordSet.end()) return ladders;

        unordered_map<string, vector<string>> parents;
        unordered_set<string> level_now, level_next;
        level_now.insert(beginWord);
        bool found = false;

        wordSet.erase(beginWord);
        wordSet.erase(endWord);

        while (!level_now.empty() && !found) {
            for (const auto& word : level_now) {
                wordSet.erase(word); 
            }

            for (const auto& word : level_now) {
                string w = word;
                for (char& ch : w) {
                    char ch_old = ch;
                    for (ch = 'a'; ch <= 'z'; ++ch) {
                        if (ch == ch_old) continue;
                        if (endWord == w) {
                            found = true;
                            parents[w].push_back(word);
                        } else if (wordSet.find(w) != wordSet.end() && !found) {
                            if (level_next.find(w) == level_next.end()) {
                                level_next.insert(w);
                                parents[w].push_back(word);
                            } else {
                                parents[w].push_back(word);
                            }
                        }
                    }
                    ch = ch_old;
                }
            }

            if (found) break;

            level_now.swap(level_next);
            level_next.clear();
        }

        if (found) {
            vector<string> path;
            dfs(endWord, beginWord, parents, path, ladders);
        }

        return ladders;
    }

private:
    void dfs(const string& word, const string& beginWord, 
             unordered_map<string, vector<string>>& parents, 
             vector<string>& path, vector<vector<string>>& ladders) {
        if (word == beginWord) {
            path.push_back(word);
            vector<string> currentPath = path;
            reverse(currentPath.begin(), currentPath.end());
            ladders.push_back(currentPath);
            path.pop_back();
            return;
        }

        const vector<string>& preds = parents[word];
        path.push_back(word);
        for (const string& pred : preds) {
            dfs(pred, beginWord, parents, path, ladders);
        }
        path.pop_back();
    }
};
```

#### [Sliding Puzzle](https://leetcode.com/problems/sliding-puzzle/)

目標是通過滑動數字，將板從初始狀態轉換為目標狀態 123450。如果可以解決，返回所需的最小移動次數；如果無解，則返回 -1。因為 BFS 可以在最短的時間內找到從初始狀態到目標狀態的最短路徑，所以可以用 BFS 來解。在這個問題中，每一步移動都相當於在圖的概念中從一個節點（即一個板的狀態）移動到另一個節點（即經過一次數字滑動後的板狀態）。

使用隊列進行層序遍歷。對於每個狀態，尋找空塊（0）可以移動的方向，生成新的板狀態，並檢查是否達到目標狀態。

* 如果達到目標狀態，則返回當前的移動次數
* 如果未達到，則將新的板狀態加入隊列繼續搜索

```cpp
class Solution {
public:
    int slidingPuzzle(vector<vector<int>>& board) {
        string target = "123450";
        string start = "";
        for (auto &row : board) {
            for (int num : row) {
                start += num + '0';
            }
        }
        
        vector<vector<int>> neighbors = {{1, 3}, {0, 2, 4}, {1, 5}, {0, 4}, {1, 3, 5}, {2, 4}}; // 每個位置可以交換的位置
        
        unordered_set<string> visited;
        queue<pair<string, int>> q; // {當前狀態，移動次數}
        q.push({start, 0});
        visited.insert(start);
        
        while (!q.empty()) {
            auto [curr, moves] = q.front(); q.pop();
            if (curr == target) return moves;
            
            int zero = curr.find('0');
            for (int neighbor : neighbors[zero]) {
                string next = curr;
                swap(next[zero], next[neighbor]);
                if (!visited.count(next)) {
                    visited.insert(next);
                    q.push({next, moves + 1});
                }
            }
        }
        
        return -1;
    }
};
```

### 圖上 BFS

#### [As Far from Land as Possible](https://leetcode.com/problems/as-far-from-land-as-possible/)

這題可以用 Multi-source BFS解決。意思是我們可以從所有的 1（陸地）同時開始搜索，逐步向外擴展到最遠的 0（海洋），這樣當搜索停止時，我們找到的就是距離陸地最遠的海洋。

```cpp
class Solution {
public:
    int maxDistance(vector<vector<int>>& grid) {
        int n = grid.size(), maxDist = -1;
        queue<pair<int, int>> q;
        
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                if (grid[i][j] == 1) {
                    q.push({i, j});
                }
            }
        }

        vector<vector<int>> dirs = {{0, 1}, {1, 0}, {-1, 0}, {0, -1}};
        while (!q.empty()) {
            int size = q.size();
            ++maxDist;
            while (size--) {
                auto [x, y] = q.front(); q.pop();
                for (auto &dir : dirs) {
                    int nx = x + dir[0], ny = y + dir[1];
                    if (nx >= 0 && nx < n && ny >= 0 && ny < n && grid[nx][ny] == 0) {
                        grid[nx][ny] = 1;
                        q.push({nx, ny});
                    }
                }
            }
        }
        
        return maxDist <= 0 ? -1 : maxDist;
    }
};
```

#### [Max Area of Island](https://leetcode.com/problems/max-area-of-island/)

對於網格中的每一個點，如果該點是土地（1），則對其進行 BFS 以計算島嶼的面積。然後對於每次 BFS，計算並更新島嶼的最大面積。

```cpp
class Solution {
public:
    int n, m;
    int dx[4] = {-1, 0, 1, 0};
    int dy[4] = {0, -1, 0, 1};
    bool check(pair<int, int> &p) {
        return p.first >= 0 && p.first < n && p.second >= 0 && p.second < m;
    }
    int bfs(vector<vector<int>>& grid, vector<vector<bool>>& vis, int sx, int sy) {
        pair<int, int> src = {sx, sy};
        queue<pair<int, int>> q;
        q.push(src);
        int area = 0;
        while (!q.empty()) {
            pair<int, int> u = q.front();
            q.pop();
            area++;
            for (int i = 0; i < 4; i++) {
                pair<int, int> p = {u.first + dx[i], u.second + dy[i]};
                if (check(p) && grid[p.first][p.second] == 1 && !vis[p.first][p.second]) {
                    q.push(p);
                    vis[p.first][p.second] = true;
                }
            }
        }
        return area;
    }
    int maxAreaOfIsland(vector<vector<int>>& grid) {
        n = grid.size(), m = grid[0].size();
        vector<vector<bool>> vis(n, vector<bool>(m, false));
        int res = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (grid[i][j] == 1 && !vis[i][j]) {
                    vis[i][j] = true;
                    res = max(res, bfs(grid, vis, i, j));
                }
            }
        }
        return res;
    }
};
```

#### [Surrounded Regions](https://leetcode.com/problems/surrounded-regions/)

這一題的關鍵是觀察到有跟存在邊界的 'O' 連起來的 'O' 就不會變成 'X'，所以我們可以從所有邊界 'O' 做 BFS ，沒有被遍歷到的就會變成 'X'。

```cpp
class Solution {
public:
    // the point is that 'O' around the board will not change to 'X'
    int d[5] = {0, -1, 0, 1, 0};
    void solve(vector<vector<char>>& board) {
        int n = board.size(), m = board[0].size();
        queue<pair<int, int>> q;
        vector<vector<bool>> vis(n, vector<bool>(m, false));

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (board[i][j] == 'O' && (i == 0 || j == 0 || i == n - 1 || j == m - 1)) {
                    vis[i][j] = 1;
                    q.push({i, j});
                }
            }
        }
        
        auto chk = [&](int x, int y) {
            return x >= 0 && x < n && y >= 0 && y < m;
        };

        auto bfs = [&]() {
            while (!q.empty()) {
                auto [x, y] = q.front();
                q.pop();
                for (int i = 0; i < 4; i++) {
                    int nx = x + d[i], ny = y + d[i + 1];
                    if (chk(nx, ny) && board[nx][ny] == 'O' && !vis[nx][ny]) {
                        vis[nx][ny] = 1;
                        q.push({nx, ny});
                    }
                }
            }
        };

        bfs();

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (!vis[i][j]) board[i][j] = 'X';
            }
        }
    }
};
```

#### [Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/)

這個問題可以透過反向思維來解決，不是考慮水如何從一個點流到兩個大洋，我們考慮從兩個大洋的邊界開始，看水能夠逆流到達哪些點。這樣，我們只需要從太平洋和大西洋的邊界開始，使用 BFS 來標記所有能夠到達的點。最後，任何同時被兩次標記（即既可以從太平洋到達又可以從大西洋到達）的點就是我們要找的答案。

```cpp
class Solution {
public:
    int n, m;
    typedef pair<int, int> pii;
    #define X first
    #define Y second
    bool chk(pii p) {
        return p.X >= 0 && p.X < n && p.Y >= 0 && p.Y < m;
    }
    int d[5] = {0, -1, 0, 1, 0};
    void bfs(int sx, int sy, vector<vector<bool>>& vis, vector<vector<int>>& h) {
        queue<pii> q;
        q.push({sx, sy});
        while (!q.empty()) {
            pii p = q.front(); q.pop();
            if (vis[p.X][p.Y]) continue;
            vis[p.X][p.Y] = 1;
            for (int i = 0; i < 4; i++) {
                pii pp(p.X + d[i], p.Y + d[i + 1]);
                if (chk(pp) && h[pp.X][pp.Y] >= h[p.X][p.Y]) {
                    q.push(pp);
                }
            }
        }
    }
    vector<vector<int>> pacificAtlantic(vector<vector<int>>& heights) {
        n = heights.size(), m = heights[0].size();
        vector<vector<bool>> pac(n, vector<bool>(m, false)), alt(n, vector<bool>(m, false));
        for (int i = 0; i < n; i++) {
            bfs(i, 0, pac, heights);
            bfs(i, m - 1, alt, heights);
        }
        for (int i = 0; i < m; i++) {
            bfs(0, i, pac, heights);
            bfs(n - 1, i, alt, heights);
        }
        vector<vector<int>> res;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (pac[i][j] && alt[i][j]) {
                    res.push_back(vector<int>{i, j});
                }
            }
        }
        return res;
    }
};
```

#### [Shortest Path with Alternating Colors](https://leetcode.com/problems/shortest-path-with-alternating-colors/)

這題如果邊只有一個顏色就很好解決，兩個顏色我們就開兩張圖，要記錄的東西也都開兩個，每次看上一次的邊是什麼顏色，就根據那個顏色決定現在要更新哪一個紀錄的距離。

```cpp
class Solution {
public:
    vector<int> shortestAlternatingPaths(int n, vector<vector<int>>& redEdges, vector<vector<int>>& blueEdges) {
        vector<vector<int>> graphRed(n), graphBlue(n);
        for (auto& edge : redEdges) graphRed[edge[0]].push_back(edge[1]);
        for (auto& edge : blueEdges) graphBlue[edge[0]].push_back(edge[1]);

        vector<int> distRed(n, INT_MAX), distBlue(n, INT_MAX);
        distRed[0] = distBlue[0] = 0;
        
        queue<pair<int, int>> q; // {node, color}, color: 0 for red, 1 for blue
        q.push({0, 0}); // start with red
        q.push({0, 1}); // start with blue

        while (!q.empty()) {
            auto [node, color] = q.front(); q.pop();
            auto& graph = (color == 0) ? graphBlue : graphRed;
            auto& distCurrent = (color == 0) ? distBlue : distRed;
            auto& distNext = (color == 0) ? distRed : distBlue;
            
            for (int nextNode : graph[node]) {
                if (distCurrent[node] + 1 < distNext[nextNode]) {
                    distNext[nextNode] = distCurrent[node] + 1;
                    q.push({nextNode, 1 - color}); // switch color for next level
                }
            }
        }

        vector<int> result(n);
        for (int i = 0; i < n; ++i) {
            int minDist = min(distRed[i], distBlue[i]);
            result[i] = (minDist == INT_MAX) ? -1 : minDist;
        }

        return result;
    }
};
```

#### [Shortest Path in a Grid with Obstacles Elimination](https://leetcode.com/problems/shortest-path-in-a-grid-with-obstacles-elimination/)

因為存在障礙物消除的選項，每個格子的訪問狀態不僅取決於其位置，還取決於到達該位置時剩餘的障礙物消除次數。因此，我們需要在 BFS 過程中追蹤每個位置的剩餘障礙物消除次數。

1. **狀態表示**：使用三元組 `(x, y, k)` 表示當前位置為 `(x, y)` 且剩餘可消除的障礙物數量為 `k` 的狀態。
2. **初始化隊列**：將起始狀態 `(0, 0, k)` 放入隊列，其中 `k` 是允許消除的最大障礙物數量。
3. **BFS 搜索**：從隊列中取出狀態進行擴展，對於每個方向的移動，如果遇到障礙物且剩餘的障礙物消除次數大於 0，則可以選擇消除它；否則，如果是空地或已經消除過障礙物，則可以直接通過。
4. **訪問記錄**：使用一個三維數組 `visited[x][y][k]` 來記錄 `(x, y)` 位置在還剩 `k` 次消除障礙物機會時是否被訪問過，以避免重複訪問。
5. **結束條件**：當 BFS 到達終點 `(n-1, m-1)` 時，返回當前的步數。

```cpp
using namespace std;

class Solution {
public:
    int shortestPath(vector<vector<int>>& grid, int k) {
        int n = grid.size(), m = grid[0].size();
        vector<vector<vector<bool>>> visited(n, vector<vector<bool>>(m, vector<bool>(k + 1, false)));
        queue<vector<int>> q;
        q.push({0, 0, k, 0});
        visited[0][0][k] = true;
        
        vector<int> dirs = {-1, 0, 1, 0, -1}; 
        
        while (!q.empty()) {
            auto cur = q.front(); q.pop();
            int x = cur[0], y = cur[1], remainingK = cur[2], steps = cur[3];
            
            if (x == n - 1 && y == m - 1) return steps;
            
            for (int i = 0; i < 4; ++i) {
                int nx = x + dirs[i], ny = y + dirs[i + 1];
                if (nx >= 0 && nx < n && ny >= 0 && ny < m) {
                    int nextK = remainingK - grid[nx][ny];
                    if (nextK >= 0 && !visited[nx][ny][nextK]) {
                        visited[nx][ny][nextK] = true;
                        q.push({nx, ny, nextK, steps + 1});
                    }
                }
            }
        }
        
        return -1; 
    }
};
```

#### [Cut Off Trees for Golf Event](https://leetcode.com/problems/cut-off-trees-for-golf-event/)

這個問題可以分解為兩個子問題：

1. **找出按高度排序的樹的位置**：首先遍歷整個網格，找到所有的樹，並按照它們的高度進行排序。
2. **計算從一棵樹移動到另一棵樹的最短步數**：對於排序後的樹的列表，從第一棵樹開始，計算從當前樹到下一棵樹的最短路徑，並累加這些步數。

```cpp
class Solution {
public:
    int cutOffTree(vector<vector<int>>& forest) {
        int m = forest.size(), n = forest[0].size();
        vector<vector<int>> trees;
       
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (forest[i][j] > 1) trees.push_back({forest[i][j], i, j});
            }
        }

        sort(trees.begin(), trees.end());
        
        int totalSteps = 0, x = 0, y = 0;

        for (int i = 0; i < trees.size(); ++i) {
            int steps = bfs(forest, x, y, trees[i][1], trees[i][2]);
            if (steps == -1) return -1;
            totalSteps += steps;
            x = trees[i][1];
            y = trees[i][2];
        }
        return totalSteps;
    }
    
    int bfs(vector<vector<int>>& forest, int sx, int sy, int tx, int ty) {
        if (sx == tx && sy == ty) return 0;
        int m = forest.size(), n = forest[0].size();
        vector<vector<int>> dirs = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
        vector<vector<bool>> visited(m, vector<bool>(n, false));
        queue<pair<int, int>> q;
        q.push({sx, sy});
        visited[sx][sy] = true;
        int steps = 0;
        
        while (!q.empty()) {
            ++steps;
            int size = q.size();
            for (int i = 0; i < size; ++i) {
                auto [x, y] = q.front(); q.pop();
                for (auto& dir : dirs) {
                    int nx = x + dir[0], ny = y + dir[1];
                    if (nx >= 0 && nx < m && ny >= 0 && ny < n && !visited[nx][ny] && forest[nx][ny] > 0) {
                        if (nx == tx && ny == ty) return steps;
                        q.push({nx, ny});
                        visited[nx][ny] = true;
                    }
                }
            }
        }
        return -1;
    }
};
```

### 樹上 BFS

#### [Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)

單純的 BFS，把每一層遍歷的節點存到 vector 裏面就好。

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> res;
        if (!root) return res;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty()) {
            int sz = q.size();
            vector<int> v;
            for (int i = 0; i < sz; i++) {
                TreeNode* node = q.front(); q.pop();
                v.push_back(node->val);
                if (node->left) q.push(node->left);
                if (node->right) q.push(node->right);
            }
            res.push_back(v);
        }
        return res;
    }
};
```

#### [Maximum Level Sum of a Binary Tree](https://leetcode.com/problems/maximum-level-sum-of-a-binary-tree/)

跟上一題差不多，只是這次對於每層我們要記錄他的總和，然後每次取 max 就好。

```cpp
class Solution {
public:
    int maxLevelSum(TreeNode* root) {
        queue<TreeNode*> q;
        q.push(root);
        int mx = -1e9, res = -1, level = 1;
        while (!q.empty()) {
            int sz = q.size();
            int sum = 0;
            for (int i = 0; i < sz; i++) {
                TreeNode* node = q.front(); q.pop();
                sum += node->val;
                if (node->left) q.push(node->left);
                if (node->right) q.push(node->right);
            }
            if (sum > mx) {
                mx = sum;
                res = level;
            }
            level++;
        }
        return res;

    }
};
```

#### [Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/)

就是一般的 BFS，只是要判斷一下是當前層的最後一個的時候要推進答案裡面。

```cpp
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        vector<int> res;
        if (!root) {
            return res;
        }
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty()) {
            int sz = q.size();
            vector<int> v;
            for (int i = 0; i < sz; i++) {
                auto u = q.front(); q.pop();
                if (u->left) q.push(u->left);
                if (u->right) q.push(u->right);
                if (i == sz - 1) res.push_back(u->val);
            }
        }
        return res;
    }
};
```

#### [Maximum Width of Binary Tree](https://leetcode.com/problems/maximum-width-of-binary-tree/)

1. 在每一層的開始，記錄當前層首個節點的索引和最後一個節點的索引。使用這兩個索引計算當前層的寬度（`ed - st + 1`），並更新最大寬度。
2. **處理子節點**：對於當前節點的每個非空子節點，計算它的索引（對於左子節點是 `2 * ind + 1`，對於右子節點是 `2 * ind + 2`），並將子節點及其索引作為一對加入隊列中。

這邊的關鍵是，每一層都重新編號（ `tmp_ind - st` ），這樣才不會算索引值算到溢位。

```cpp
class Solution {
public:
    typedef pair<TreeNode*, long long> info;
    int widthOfBinaryTree(TreeNode* root) {
        if (!root) return 0;
        long long width = 1;
        queue<info> q;
        q.push({root, 0ll});
        while (!q.empty()) {
            int sz = q.size();
            long long st = q.front().second, ed = q.back().second;
            width = max(width, ed - st + 1);
            for (int i = 0; i < sz; i++) {
                auto [u, tmp_ind] = q.front();
                q.pop();
                long long ind = tmp_ind - st;
                if (u->left) {
                    q.push({u->left, 2 * ind + 1});
                }
                if (u->right) {
                    q.push({u->right, 2 * ind + 2});
                }
            }
        }
        return width;
    }
};
```
