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

### 基礎題目選解

#### [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/)

這題可以用 priority\_queue 、vector、set 等等作為 sort 的工具，主要就是把距離算好然後拿出前 k 小就可以了。

```cpp
class Solution {
public:
    typedef pair<int, pair<int, int>> info;
    vector<vector<int>> kClosest(vector<vector<int>>& points, int k) {
        priority_queue<info, vector<info>, greater<info>> pq;
        for (auto &v : points) {
            pq.push({v[0] * v[0] + v[1] * v[1], {v[0], v[1]}});
        }
        vector<vector<int>> res;
        for (int i = 0; i < k; i++) {
            info p = pq.top();
            pq.pop();
            res.push_back(vector<int>{p.second.first, p.second.second});
        }
        return res;
    }
};
```

#### [Sort Character By Frequency](https://leetcode.com/problems/sort-characters-by-frequency/description/)

開 map 紀錄每一個 character 出現過幾次，在使用內建的排序好後，從大到小構成 string 就可以了

```cpp
class Solution {
public:
    string frequencySort(string s) {
      map<char, int> mp;
      for (char c : s) mp[c]++;
      vector<pair<int, char>> v;
      for (auto [c, cnt]: mp) {
        v.push_back({cnt, c});
      }
      sort(v.begin(), v.end(), greater<pair<int, char>>());
      string ans = "";
      for (auto [freq, ch]: v) {
        ans.append(freq, ch);
      }
      return ans;
    }
};
```

#### [Sort Array by Parity](https://leetcode.com/problems/sort-array-by-parity/)

實作一個 lambda function，如果前者是偶數則 return true，反之 return false。我的實作是透過 & 1 來比較，因為如果是奇數的話 & 1 就會是 1，反之則是 0

```cpp
class Solution {
public:
    vector<int> sortArrayByParity(vector<int>& nums) {
        sort(nums.begin(), nums.end(), [](int a, int b){
            return (a & 1) < (b & 1);
        });
        return nums;
    }
};
```

#### [Custom Sort String](https://leetcode.com/problems/custom-sort-string/)

可以先開一個 hash table 紀錄每一個 letter 的 order 大小，接著把 s 裡面所有的 character 和他的 order 一起 sort，最後再拼回去就好。

```cpp
class Solution {
public:
    string customSortString(string order, string s) {
        vector<int> ord(26, 28);
        int ind = 0;
        for (char c : order) {
            ord[c - 'a'] = ind++;
        }
        vector<pair<int, char>> v;
        for (char c : s) {
            v.push_back({ord[c - 'a'], c});
        }
        sort(v.begin(), v.end(), [](auto a, auto b){
            return a.first < b.first;
        });
        string res = "";
        for (auto [_, c] : v) {
            res.push_back(c);
        }
        return res;
    }
};
```

### 進階題目選解

#### [Sort the Matrix Diagonally](https://leetcode.com/problems/sort-the-matrix-diagonally/)

這題可以利用 priority\_queue 作為 sort 的工具，注意到 diagonal 的 row index - column index 都是相同的，因此可以開一個 map 去記錄同一個 row index - column index 的數字有哪些，並且用 priority\_queue，之後再按照順序把數字放回去。

```cpp
class Solution {
public:
    vector<vector<int>> diagonalSort(vector<vector<int>>& mat) {
        int n = mat.size(), m = mat[0].size();
        map<int, priority_queue<int, vector<int>, greater<int>>> mp;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                mp[i - j].push(mat[i][j]);
            }
        }
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                mat[i][j] = mp[i - j].top();
                mp[i - j].pop();
            }
        }
        return mat;
    }
};
```

#### [Largest Values From Labels](https://leetcode.com/problems/largest-values-from-labels/)

可以把所有同樣 label 的人放進同一個 priority\_queue 裡面做排序，再根據 useLimit 去把所有可以的都放進一個大 priority\_queue 去看最後可以取哪些，最後再根據大 priority\_queue 的 size 跟 numWanted 去選擇要選誰。

```cpp
class Solution {
public:
    int largestValsFromLabels(vector<int>& values, vector<int>& labels, int numWanted, int useLimit) {
        map<int, priority_queue<int>> mp;
        int n = values.size();
        for (int i = 0; i < n; i++) {
            mp[labels[i]].push(values[i]);
        }
        priority_queue<int> pq;
        for (auto [_, tmp_pq] : mp) {
            for (int i = 0; i < useLimit && !tmp_pq.empty(); i++) {
                pq.push(tmp_pq.top());
                tmp_pq.pop();
            }
        }
        int res = 0;
        while (!pq.empty() && numWanted--) {
            res += pq.top();
            pq.pop();
        }
        return res;
    }
};
```

### 更進階一點（搭配其他演算法）

這裡只會舉例一些比較簡單的題目，重點在於 sort 而非其他演算法。

#### [3Sum Closest](https://leetcode.com/problems/3sum-closest/) （搭配雙指針）

我們可以先將 nums 排序，接著對於 nums\[i]，我們希望找到 l, r 使得 nums\[i] + nums\[l] + nums\[r] 最接近 target。根據排序，我們知道 nums 有單調性，因此我們可以先假裝 l = i + 1, r = n - 1，如果這樣加起來大於 target 的話，那我就把 r--，讓大的部分小一點；反之則讓 l++，讓小的部分大一點。

因為他要求的是最靠近的 sum，所以在算現在的 nums\[i] + nums\[l] + nums\[r] 時，要記得更新答案！

```cpp
class Solution {
public:
    int threeSumClosest(vector<int>& nums, int target) {
        int n = nums.size(), res = 1000000000;
        sort(nums.begin(), nums.end());
        for (int i = 0; i < n; i++) {
            int l = i + 1, r = n - 1;
            while (l < r) {
               int sum = nums[i] + nums[l] + nums[r];
               if (abs(sum - target) < abs(res - target)) {
                   res = sum;
               } 
               sum > target ? r-- : l++;
            }
        }
        return res;
    }
};
```
