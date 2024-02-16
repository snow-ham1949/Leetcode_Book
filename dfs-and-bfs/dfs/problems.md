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

### 枚舉子集 or 狀態

#### [Combination Sum](https://leetcode.com/problems/combination-sum/description/)

我們可以每一次去看要選當前 id 的元素加上去還是繼續往前選，如果和已經大於 target 或者是已經沒有元素可以選的時候就結束這一種枚舉。

```cpp
class Solution {
public:
    vector<vector<int>> res;
    void dfs(vector<int>& candidates, int id, int sum, vector<int>&v, int target) {
        if (sum > target || id == (int)candidates.size()) return;
        if (sum == target) {
            res.push_back(v);
            return;
        } else {
            v.push_back(candidates[id]);
            sum += candidates[id];
            dfs(candidates, id, sum, v, target);
            v.pop_back();
            sum -= candidates[id];
            dfs(candidates, id + 1, sum, v, target);
        }
    }
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        sort(candidates.begin(), candidates.end());
        vector<int> v;
        dfs(candidates, 0, 0, v, target);
        return res;
    }
};
```

#### [Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)

首先可以開一個 map 紀錄一個電話按鈕對應到哪些字母，接下來就好好枚舉子集就可以了，作法跟上面那題是差不多的，就是對於一個元素先選起來，然後看後面可以怎麼接，再把他清除看同一個電話按鈕還有沒有其他選擇。

```cpp
class Solution {
public:
    map<int, string> mp;
    vector<string> letterCombinations(string digits) {
        if (digits == "") return vector<string>();
        char c = 'a';
        for (int i = 2; i <= 9; i++) {
            for (int j = 0; j < (i == 7 || i == 9 ? 4 : 3); j++) {
                mp[i].push_back(c);
                c++;
            }
        }
        vector<string> res;
        auto dfs = [&](auto dfs, int ind, string digits, string cur) -> void {
            if (ind == digits.size()) {
                res.push_back(cur);
                return;
            }
            for (char c : mp[digits[ind] - '0']) {
                cur.push_back(c);
                dfs(dfs, ind + 1, digits, cur);
                cur.pop_back();
            }
        };
        dfs(dfs, 0, digits, "");
        return res;
    }
};
```

### 圖中 dfs

#### [Word Search](https://leetcode.com/problems/word-search/description/)

可以想到一個基本的做法，O(mn) 去判斷每一格是否等於 word 的第一個，如果可以我們就 4-dir 去 dfs 看能不能組成 word，注意在每一次 dfs 的時候不能重複走一樣的格子，然後這個紀錄在下一次 dfs 的時候就要清掉。

```cpp
class Solution {
public:
    int n, m;
    int d[5] = {0, 1, 0, -1, 0};
    int vis[7][7];
    bool chk(int x, int y) {
        return x >= 0 && x < n && y >= 0 && y < m;
    }
    bool dfs(int x, int y, vector<vector<char>>& board, string word, int id) {
        if (id == word.size()) return true;
        for (int k = 0; k < 4; k++) {
            int dx = x + d[k], dy = y + d[k + 1];
            if (chk(dx, dy) && !vis[dx][dy] && board[dx][dy] == word[id]) {
                vis[dx][dy] = 1;
                if (dfs(dx, dy, board, word, id + 1)) return true;
                vis[dx][dy] = 0;
            }
        }
        return false;
    }
    bool exist(vector<vector<char>>& board, string word) {
        n = board.size(), m = board[0].size();
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (board[i][j] == word[0]) {
                    vis[i][j] = 1;
                    if (dfs(i, j, board, word, 1)) return true;
                    vis[i][j] = 0;
                }
            }
        }
        return false;
    }
};
```

### 樹上 dfs

#### [Graph Valid Tree](https://leetcode.com/problems/graph-valid-tree/description/)

樹的基本性質是：n 個點 n - 1 條邊，沒有環且連通。所以我們就去 dfs 看有沒有環，最後再檢查是否所有點都有走過（連通）就可以了。

```cpp
class Solution {
public:
    bool validTree(int n, vector<vector<int>>& edges) {
        vector<vector<int>> g(n, vector<int>());
        for (auto &v : edges) {
            g[v[0]].push_back(v[1]);
            g[v[1]].push_back(v[0]);
        }
        vector<bool> vis(n, false);
        auto dfs = [&](auto dfs, int u, int p) {
            vis[u] = 1;
            for (int v : g[u]) {
                if (v == p) continue;
                else if (vis[v]) {
                    return false;
                } else {
                    if (!dfs(dfs, v, u)) return false;
                }
            }
            return true;
        };
        if (!dfs(dfs, 0, -1)) return false;
        for (int i = 0; i < n; i++) {
            if (!vis[i]) {
                return false;
            }
        }
        return true;
    }
};
```

#### [Path Sum II](https://leetcode.com/problems/path-sum-ii/description/)

基本上跟枚舉子集是同一個概念，先選選看然後遞迴下去，如果走到葉子和是對的就加進答案裡面，不是的話就返回的時候把它清掉。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    vector<vector<int>> pathSum(TreeNode* root, int targetSum) {
        vector<vector<int>> res;
        if (!root) return res;
        auto dfs = [&](auto dfs, TreeNode* t, int rem, vector<int> &v) {
            if (!t) return;
            v.push_back(t->val);
            if (!t->left && !t->right && rem == t->val) {
                res.push_back(v);
            }
            dfs(dfs, t->left, rem - t->val, v);
            dfs(dfs, t->right, rem - t->val, v);
            v.pop_back();
        };
        vector<int> v;
        dfs(dfs, root, targetSum, v);
        return res;
    }
};
```

#### [Path Sum III](https://leetcode.com/problems/path-sum-iii/description/)

這題進階一點，因為需要的是區段和，所以我們會需要前綴和來看某一個區段是不是好的，意思是說，如果我們現在全選某一條 path 的和是 sum，但是前面已經有  sum - target 了，那 sum - (sum - target) = target，就代表有部分區段和是符合的。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int dfs(TreeNode *t, long long int tar, long long int cur, map<long long int, int> &pre) {
        if (!t) return 0;
        cur += t->val;
        int cnt = pre[cur - tar];
        pre[cur]++;
        int lcnt = dfs(t->left, tar, cur, pre), rcnt = dfs(t->right, tar, cur, pre);
        pre[cur]--;
        cur -= t->val;
        return cnt + lcnt + rcnt;
    }
    int pathSum(TreeNode* root, int targetSum) {
        map<long long int, int> pre;
        pre[0] = 1;
        return dfs(root, (long long int)targetSum, 0, pre);
    }
};
```

#### [All Nodes Distance K in Binary Tree](https://leetcode.com/problems/all-nodes-distance-k-in-binary-tree/)

這邊要算小孩的話應該比較簡單，就遞迴 k 次就行了，那如果是祖先或者兄弟呢？我們可以想像，如果把圖建出來（不用單純 treenode 的話），那麼我們可以把要求的點當作根，直接 dfs 就好。所以利用 map 來建圖，接下來 dfs。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    map<TreeNode*, TreeNode*> par;
    set<TreeNode*> vis;
    vector<int> res;
    void build(TreeNode* t) {
        if (!t) return;
        if (t->left) par[t->left] = t;
        if (t->right) par[t->right] = t;
        build(t->left);
        build(t->right);
    }
    void dfs(TreeNode *t, int k) {
        if (vis.find(t) != vis.end()) return;
        vis.insert(t);
        if (k == 0) {
            res.push_back(t->val);
            return;
        }
        if (t->left) dfs(t->left, k - 1);
        if (t->right) dfs(t->right, k - 1);
        if (par[t]) dfs(par[t], k - 1);
    }
    vector<int> distanceK(TreeNode* root, TreeNode* target, int k) {
        build(root);
        dfs(target, k);
        return res;
    }
};
```

#### [Height of Binary Tree After Subtree Removal Queries](https://leetcode.com/problems/height-of-binary-tree-after-subtree-removal-queries/)

這題的話需要前一頁提到的 dfs 序，想像我們用 dfs 序去看一棵樹，選部分的子樹就變成了選一個區間，是一種蠻常見關於子樹的問題的技巧。那麼這題要求我們輸出移除區間後的最大值，可以想成取 max(前綴的最大值, 後綴的最大值)，所以預先跑出 dfs 序和高度，計算出前綴後綴最大值，就可以每次詢問的時候問 max(前綴的最大值, 後綴的最大值) 了。

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    map<int, int> dep, in, out;
    vector<int> ord;
    void dfs(TreeNode *t, int level) {
        dep[t->val] = level;
        ord.push_back(t->val);
        in[t->val] = (int)ord.size() - 1;
        if (t->left) dfs(t->left, level + 1);
        if (t->right) dfs(t->right, level + 1);
        out[t->val] = (int)ord.size() - 1;
    }
    vector<int> treeQueries(TreeNode* root, vector<int>& queries) {
        dfs(root, 0);
        int ord_sz = (int)ord.size();
        vector<int> pre = {dep[ord[0]]};
        for (int i = 1; i < ord_sz; ++i) {
            pre.push_back(max(pre.back(), dep[ord[i]]));
        }
        vector<int> suf = {dep[ord.back()]};
        for (int i = ord_sz - 2; i >= 0; --i) {
            suf.push_back(max(suf.back(), dep[ord[i]]));
        }
        reverse(suf.begin(), suf.end());
        vector<int> res;
        for (int q : queries) {
            int st = in[q], ed = out[q];
            res.push_back(0);
            if (st > 0) {
                res.back() = max(res.back(), pre[st - 1]);
            }
            if (ed < ord_sz - 1) {
                res.back() = max(res.back(), suf[ed + 1]);
            }
        }
        return res;
    }
};
```

### 搭配資料結構

[Word Search II](https://leetcode.com/problems/word-search-ii/description/)

這題首先要知道 Trie 這個資料結構，可以學完再回來看。有了 Trie 之後我們可以先把所有的 word 加進 Trie 裡面，接下來 O(mn) 對於棋盤上每一個字母都去看他在 Trie 裏面有沒有節點，如果有節點代表他可能可以是某個字母的開頭，接下來一樣 4-dir 去看剛剛前一個節點是否有一個小孩是他鄰近的格子。

```cpp
class Solution {
public:
    struct Node {
        bool end;
        Node* child[26];
        Node() {
            end = false;
            for (int i = 0; i < 26; i++) child[i] = NULL;
        }
    };
    vector<string> res;
    void add(Node* node, string word) {
        for (char c : word) {
            int ind = c - 'a';
            if (node->child[ind] == NULL) node->child[ind] = new Node();
            node = node->child[ind];
        }
        node->end = true;
    }
    int n, m;
    bool chk(int x, int y) {
        return x >= 0 && x < n && y >= 0 && y < m;
    }
    int d[5] = {0, -1, 0, 1, 0};
    void dfs(Node *node, int x, int y, vector<vector<char>>& board, vector<vector<int>>& vis, string &word) {
        vis[x][y] = 1;
        word.push_back(board[x][y]);
        if (node->end) {
            res.push_back(word);
            node->end = false;
        }
        for (int i = 0; i < 4; i++) {
            int dx = x + d[i], dy = y + d[i + 1];
            if (chk(dx, dy) && !vis[dx][dy]) {
                int ind = board[dx][dy] - 'a';
                if (node->child[ind]) {
                    dfs(node->child[ind], dx, dy, board, vis, word);
                }
            }
        }
        word.pop_back();
        vis[x][y] = 0;
    }
    vector<string> findWords(vector<vector<char>>& board, vector<string>& words) {
        Node *root = new Node();
        for (auto s : words) {
            add(root, s);
        }
        n = board.size(), m = board[0].size();
        vector<vector<int>> vis(n, vector<int>(m, 0));
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                int ind = board[i][j] - 'a';
                if (root->child[ind]) {
                    string s = "";
                    dfs(root->child[ind], i, j, board, vis, s);
                }
            }
        }
        return res;
    }
};
```
