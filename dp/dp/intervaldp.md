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

# 區間 DP

#### [Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence)

考慮到回文的特性，我們可以這樣思考：

* 如果子串的首尾字符相同，那麼這兩個字符可以成為回文子序列的一部分，並且我們需要查看這兩個字符之間的子串的最長回文子序列長度。
* 如果首尾字符不同，則最長回文子序列不包含這兩個字符中的至少一個，因此我們需要比較去掉一個字符後子串的最長回文子序列長度。

我們可以定義 DP 陣列 `dp[i][j]` 為字串從位置 `i` 到位置 `j` 的子串中最長回文子序列的長度。然後我們按子串的長度進行遍歷，從長度為 1 的子串開始，直到整個字符串的長度。對於每一個子串，我們根據上述規則更新 `dp[i][j]` 的值。

```cpp
class Solution {
public:
    int longestPalindromeSubseq(string s) {
        int n = s.size();
        vector<vector<int>> dp(n, vector<int>(n, 0));

        for (int i = 0; i < n; ++i) dp[i][i] = 1;

        for (int len = 2; len <= n; ++len) {
            for (int i = 0; i <= n - len; ++i) {
                int j = i + len - 1;
                if (s[i] == s[j]) {
                    dp[i][j] = dp[i + 1][j - 1] + 2;
                } else {
                    dp[i][j] = max(dp[i + 1][j], dp[i][j - 1]);
                }
            }
        }

        return dp[0][n - 1];
    }
};
```

#### [Maximum Length of Pair Chain](https://leetcode.com/problems/maximum-length-of-pair-chain)

問題要求找到最長的鏈，其中每對的第二個元素小於下一對的第一個元素。這與「最長遞增子序列」（LIS）有相似之處，其中我們尋找的是一系列逐步增加的元素。通過將配對按照第一個元素（或第二個元素）進行排序，我們可以將問題轉化為類似於尋找一系列「遞增」配對的問題，這可以用動態規劃來解決。

首先，將所有的配對按照第一個元素的大小進行排序。然後設 `dp[i]` 為包含第 `i` 個配對作為結尾的最長配對鏈的長度。對於每一個配對 `i`，我們查找之前所有 `j < i` 的配對，並檢查是否可以將配對 `i` 附加到配對 `j` 上，即 `pairs[j][1] < pairs[i][0]`。如果可以，我們更新 `dp[i]` 為 `dp[j] + 1` 中的最大值。

```cpp
class Solution {
public:
    int findLongestChain(vector<vector<int>>& pairs) {
        sort(pairs.begin(), pairs.end(), [](const vector<int> &a, const vector<int> &b) {
            return a[0] < b[0];
        });
        int n = pairs.size();
        vector<int> dp(n, 1); 
        for (int i = 1; i < n; ++i) {
            for (int j = 0; j < i; ++j) {
                if (pairs[j][1] < pairs[i][0]) {
                    dp[i] = max(dp[i], dp[j] + 1);
                }
            }
        }
        return *max_element(dp.begin(), dp.end());
    }
};
```

#### [Guess Number Higher or Lower II](https://leetcode.com/problems/guess-number-higher-or-lower-ii)

#### 1. 怎麼想到用區間 DP 來解

這個問題之所以適合使用區間 DP 來解決，是因為它要求我們考慮範圍內的所有可能數字作為猜測的起點，並對每個可能的起點，計算如果猜測錯誤時（即數字更高或更低）需要繼續在剩餘範圍內猜測的最小代價。問題的解決方案依賴於子問題的解：即不同範圍（或區間）內的最小代價，這些子問題的解被用來構建更大範圍的最小代價解答。

#### 2. 解法

&#x20;`dp[i][j]` 表示在範圍 `[i, j]` 內猜數字，並且保證猜中數字所需支付的最小金額。對於任何一個範圍 `[i, j]`，我們選擇一個猜測點 `k`，然後基於 `k` 將範圍分為左右兩部分，最壞情況下的花費將是 `k + max(dp[i][k-1], dp[k+1][j])`。我們需要在所有可能的 `k` 中找到花費最小的一個。

```cpp
class Solution {
public:
    int getMoneyAmount(int n) {
        vector<vector<int>> dp(n+1, vector<int>(n+1, 0));
        for (int len = 2; len <= n; len++) { 
            for (int start = 1; start <= n - len + 1; start++) {
                int minCost = INT_MAX;
                int end = start + len - 1;
                for (int k = start; k < end; k++) {
                    int cost = k + max(dp[start][k - 1], dp[k + 1][end]);
                    minCost = min(minCost, cost);
                }
                dp[start][end] = minCost;
            }
        }
        return dp[1][n];
    }
};
```

#### [Burst Balloons](https://leetcode.com/problems/burst-balloons)

戳破一個氣球的行為會影響到其左右兩側氣球的戳破策略，形成一個區間問題。當我們戳破一個氣球時，得到的硬幣數是該氣球左側的最後一個未被戳破的氣球和右側的第一個未被戳破的氣球的值的乘積，再加上該氣球自身的值。因此，我們需要考慮每個可能的區間，並在這個區間內找到一個最佳的戳破順序以獲得最大的硬幣數量。

定義 `dp[i][j]` 為戳破區間 `(i, j)` 內所有氣球（不包括 `i` 和 `j`）所能獲得的最大硬幣數量。為了方便處理邊界情況，我們在氣球序列的開頭和結尾各添加一個虛擬的氣球，其值均為 1。

對於每個區間 `(i, j)`，我們嘗試戳破其中的每一個氣球 `k`，並將問題分解為戳破 `(i, k)` 和 `(k, j)` 兩個子區間的問題。戳破氣球 `k` 所獲得的硬幣數量為 `nums[i] * nums[k] * nums[j]`，加上戳破 `(i, k)` 和 `(k, j)` 所獲得的硬幣數量。

```cpp
class Solution {
public:
    int maxCoins(vector<int>& nums) {
        int n = nums.size();
        nums.insert(nums.begin(), 1);
        nums.push_back(1);
        vector<vector<int>> dp(n + 2, vector<int>(n + 2, 0));
        
        for (int len = 1; len <= n; ++len) {
            for (int start = 1; start <= n - len + 1; ++start) {
                int end = start + len - 1;
                for (int k = start; k <= end; ++k) {
                    int coins = nums[start - 1] * nums[k] * nums[end + 1];
                    coins += dp[start][k - 1] + dp[k + 1][end];
                    dp[start][end] = max(dp[start][end], coins);
                }
            }
        }
        
        return dp[1][n];
    }
};
```

#### [Palindrome Removal](https://leetcode.com/problems/palindrome-removal)

我們可以將整個陣列分割成多個子區間，並且對於每一個子區間，我們都試圖找出將其完全移除所需的最少操作次數。一個子區間如果本身是一個回文，那麼它可以一次被完全移除；否則，我們需要將它分割成更小的子區間，並且對每個子區間進行操作。這種分割的過程自然形成了一種由底向上求解的動態規劃方法。

定義 `dp[i][j]` 為從索引 `i` 到 `j` 的子陣列完全移除所需的最少操作次數。對於任意一個子區間 `(i, j)`，我們有兩種基本情況：

* 如果 `arr[i] == arr[j]`，那麼我們可以同時移除這兩個元素以及它們之間的所有元素（如果它們形成一個回文）。
* 如果 `arr[i] != arr[j]`，那麼我們需要分別考慮移除從 `i` 到 `j-1` 和從 `i+1` 到 `j` 的子數組所需的最少操作次數，並從中選擇最優的方案。

此外，我們還需要考慮將區間 `(i, j)` 分割成兩部分的情況，並計算將這兩部分分別移除所需的操作次數之和。

```cpp
class Solution {
public:
    int minimumMoves(vector<int>& arr) {
        int n = arr.size();
        vector<vector<int>> dp(n, vector<int>(n, INT_MAX));
        for (int i = 0; i < n; ++i) {
            dp[i][i] = 1; 
            if (i < n - 1) {
                dp[i][i + 1] = (arr[i] == arr[i + 1] ? 1 : 2); 
            }
        }

        for (int len = 3; len <= n; ++len) {
            for (int i = 0; i <= n - len; ++i) {
                int j = i + len - 1;
                if (arr[i] == arr[j]) {
                    dp[i][j] = dp[i + 1][j - 1];
                }
                for (int k = i; k < j; ++k) {
                    dp[i][j] = min(dp[i][j], dp[i][k] + dp[k + 1][j]);
                }
            }
        }

        return dp[0][n - 1];
    }

};
```

#### [Palindrome Partitioning III](https://leetcode.com/problems/palindrome-partitioning-iii)

這個問題可以分解成兩個部分：一是如何將字串分割成 `k` 個子串；二是如何最小化將每個子字串轉換成回文串所需的修改次數。

首先，我們需要一個函數來計算將任意子字串 `s[i...j]` 轉換成回文串所需的最小修改次數。這可以通過雙指針方法在線性時間內完成。

接著，我們定義 `dp[i][k]` 為將前 `i` 個字符分割成 `k` 個回文子串所需的最小修改次數。最後答案是 `dp[n][k]`，其中 `n` 是字串的長度。

```cpp
class Solution {
public:
    int palindromePartition(string s, int k) {
        int n = s.length();
        vector<vector<int>> cost(n, vector<int>(n, 0));
        for (int len = 2; len <= n; ++len) {
            for (int i = 0; i <= n - len; ++i) {
                int j = i + len - 1;
                cost[i][j] = cost[i + 1][j - 1] + (s[i] != s[j] ? 1 : 0);
            }
        }

        vector<vector<int>> dp(n + 1, vector<int>(k + 1, INT_MAX));
        dp[0][0] = 0; 
        for (int i = 1; i <= n; ++i) {
            for (int j = 1; j <= min(i, k); ++j) {
                if (j == 1) {
                    dp[i][j] = cost[0][i - 1]; 
                } else {
                    for (int m = j - 1; m < i; ++m) {
                        dp[i][j] = min(dp[i][j], dp[m][j - 1] + cost[m][i - 1]);
                    }
                }
            }
        }

        return dp[n][k];
    }

};
```

#### [Minimum Score Triangulation of Polygon](https://leetcode.com/problems/minimum-score-triangulation-of-polygon/)

對於一個多邊形，我們可以選擇三角形（由多邊形的邊和對角線構成）來進行剖分。每次選擇都會將多邊形分割成兩個部分，每一部分都可以獨立地進行最優剖分。這種選擇的方式自然形成了一個區間問題，我們需要考慮所有可能的分割方式來找到全局最優的解。

定義 `dp[i][j]` 表示將多邊形的頂點 `i` 到頂點 `j`（按順時針或逆時針順序）構成的子多邊形進行最優三角剖分所得到的最小分數。對於任意一個子多邊形，我們可以選擇一條對角線（`i` 到 `k`，`k` 到 `j`），將其分割成兩個子多邊形，然後計算這兩個子多邊形的最優解和當前三角形的分數，更新 `dp[i][j]`。

```cpp
class Solution {
public:
    int minScoreTriangulation(vector<int>& values) {
        int n = values.size();
        vector<vector<int>> dp(n, vector<int>(n, 0));
        for (int len = 3; len <= n; ++len) { 
            for (int i = 0; i <= n - len; ++i) {
                int j = i + len - 1;
                dp[i][j] = INT_MAX;
                for (int k = i + 1; k < j; ++k) {
                    int score = dp[i][k] + dp[k][j] + values[i] * values[k] * values[j];
                    dp[i][j] = min(dp[i][j], score);
                }
            }
        }
        return dp[0][n - 1];
    }
};
```

#### [Remove Boxes](https://leetcode.com/problems/remove-boxes)

核心思想是將原始的盒子序列壓縮成多個組，每組包含相同數字的盒子及其計數。對於每一組，我們試圖找到最佳的移除策略：要麼直接移除這組盒子來獲得得分，要麼尋找機會與其他相同數字的盒子組合以獲得更高的得分。

&#x20;`dp[l][r][extra]` 被用來儲存從左邊界到右邊界的組，並且右邊界右側有 `extra` 個相同數字的盒子時，可以獲得的最大得分。這樣，我們就可以從底層子問題開始，逐步構建出整個問題的解。

```cpp
class Solution {
public:
    int dp[105][105][105];
    int removeBoxes(vector<int>& boxes) {
        int n = boxes.size(), count = 1;
        vector<vector<int>> groups; 
        memset(dp, -1, sizeof(dp));
        for (int i = 1; i < n; i++) {
            if (boxes[i] != boxes[i - 1]) {
                groups.push_back({boxes[i - 1], count});
                count = 0;
            }
            count++;
        }
        groups.push_back({boxes[n - 1], count});

        function<int(int, int, int)> dfs = [&](int l, int r, int extra) {
            if (l > r) return 0;
            if (dp[l][r][extra] != -1) return dp[l][r][extra];

            int size = groups[l][1], number = groups[l][0];
            dp[l][r][extra] = dfs(l + 1, r, 0) + (extra + size) * (extra + size);
            for (int i = l + 1; i <= r; i++) {
                if (number == groups[i][0]) {
                    dp[l][r][extra] = max(dp[l][r][extra], dfs(l + 1, i - 1, 0) + dfs(i, r, extra + size));
                }
            }
            return dp[l][r][extra];
        };

        return dfs(0, groups.size() - 1, 0);
    }
};
```

