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

# 序列 DP

#### [Fibonacci Number](https://leetcode.com/problems/fibonacci-number/)

按照題目的規定列 dp 式跑迴圈就好了。

```cpp
class Solution {
public:
    int fib(int n) {
      if (n == 0) return 0;
      if (n == 1) return 1;
      vector<int> dp(n + 1, 0);
      dp[1] = 1;
      for (int i = 2; i <= n; i++) dp[i] = dp[i - 1] + dp[i - 2];
      return dp.back();
    }
};
```

#### [Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)

因為可以從前一階或前兩階走過來，所以這題跟上一題是一樣的遞迴式，只是要注意 base case 不一樣。

```
class Solution {
public:
    int climbStairs(int n) {
      if (n <= 2) return n;
      vector<int> dp(n + 1);
      dp[1] = 1, dp[2] = 2;
      for (int i = 3; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
      }
      return dp[n];
    }
};
```

#### [Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)

$$O(n^2)$$ 的解法

* **子問題的重疊性**：在尋找最長遞增子序列的過程中，我們會發現需要反復解決相同的子問題，即對於序列中的每個元素，我們需要知道以該元素結尾的最長遞增子序列的長度。
* **最優子結構**：一個序列的最長遞增子序列可以由其前面的序列的最長遞增子序列加上當前元素（如果當前元素大於前面序列中的最後一個元素）組成。

`dp[i]` 表示以第 `i` 個元素結尾的最長遞增子序列的長度。對於每個元素，我們遍歷之前的所有元素，更新 `dp[i]` 的值為之前元素的 `dp[j] + 1`（如果 `nums[i] > nums[j]`）中的最大值。最後，`dp` 數組中的最大值就是整個序列的最長遞增子序列的長度。

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        if (nums.empty()) return 0;
        vector<int> dp(nums.size(), 1);
        int maxLen = 1;
        for (int i = 1; i < nums.size(); ++i) {
            for (int j = 0; j < i; ++j) {
                if (nums[i] > nums[j]) {
                    dp[i] = max(dp[i], dp[j] + 1);
                }
            }
            maxLen = max(maxLen, dp[i]);
        }
        return maxLen;
    }
};
```

$$O(n\log{n})$$ 的解法

維護一個陣列 `tails`，其中 `tails[i]` 表示長度為 $$i+1$$ 的所有遞增子序列中結尾元素的最小值。這樣做的目的是用最小的結尾元素來保持遞增子序列的擴展性，從而使得後續的元素有更大的機會成為某個遞增子序列的一部分。

#### 解法原理

1. **初始化**：`tails` 陣列初始化為空，表示目前沒有遞增子序列。
2. **遍歷**：對於數組 `nums` 中的每個元素，用二分搜在 `tails` 中找到第一個大於或等於當前元素的位置，如果找到，則替換該位置的元素（這表示我們找到了一個更小的元素作為結尾，來優化現有的遞增子序列）；如果沒有找到，則將當前元素添加到 `tails` 的末尾（這表示我們找到了一個更長的遞增子序列）。
3. **結果**：遍歷結束後，`tails` 陣列的大小即為最長遞增子序列的長度。

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> tails;
        for (int x : nums) {
            auto it = lower_bound(tails.begin(), tails.end(), x);
            if (it == tails.end()) tails.push_back(x);
            else *it = x;
        }
        return tails.size();
    }
};
```

#### [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/)

`dp[i][j]` 表示第一個字串的前 `i` 個字元和第二個字串的前 `j` 個字元的最長公共子序列的長度。轉移方程如下：

* 如果 `text1[i-1] == text2[j-1]`，則 `dp[i][j] = dp[i-1][j-1] + 1`；
* 否則，`dp[i][j] = max(dp[i-1][j], dp[i][j-1])`。

如果當前字元匹配，則最長公共子序列的長度增加1；如果不匹配，則從前一步的解中選擇較大的一個。

```cpp
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int n = text1.size(), m = text2.size();
        vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                if (text1[i - 1] == text2[j - 1]) dp[i][j] = dp[i - 1][j - 1] + 1;
                else dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
        return dp[n][m];
    }
};
```

#### [Edit Distance](https://leetcode.com/problems/edit-distance/)

`dp[i][j]` 表示從第一個字串的前 `i` 個字元轉換到第二個字串的前 `j` 個字元所需的最少操作次數。轉移方程如下：

* 如果 `word1[i-1] == word2[j-1]`，則 `dp[i][j] = dp[i-1][j-1]`，因為當前字元相同，不需要額外操作。
* 否則，`dp[i][j] = 1 + min(dp[i-1][j-1], dp[i-1][j], dp[i][j-1])`，分別對應替換、刪除和插入操作。

```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        int m = word1.size(), n = word2.size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1));
        for (int i = 0; i <= m; i++) {
            for (int j = 0; j <= n; j++) {
                if (i == 0) dp[i][j] = j; 
                else if (j == 0) dp[i][j] = i; 
                else if (word1[i-1] == word2[j-1]) dp[i][j] = dp[i-1][j-1];
                else dp[i][j] = 1 + min({dp[i-1][j-1], dp[i-1][j], dp[i][j-1]});
            }
        }
        return dp[m][n];
    }

};
```

#### [Interleaving String](https://leetcode.com/problems/interleaving-string/)

* **子問題的重疊性**：判斷一個較長字串是否為兩個較短字串交錯組成時，可以將問題分解成判斷較短字串的前綴是否能組成較長字串的前綴，這些子問題之間存在重疊。
* **最優子結構**：如果 `s3` 的前 `i+j` 個字元可以由 `s1` 的前 `i` 個字元和 `s2` 的前 `j` 個字元交錯組成，則該問題的解可以通過解決子問題來構造。

`dp[i][j]` 表示 `s1` 的前 `i` 個字元和 `s2` 的前 `j` 個字元是否能交錯組合成 `s3` 的前 `i+j` 個字元。轉移方程如下：

* 如果 `s1[i-1] == s3[i+j-1]` 且 `dp[i-1][j]` 為真，則 `dp[i][j] = true`。
* 如果 `s2[j-1] == s3[i+j-1]` 且 `dp[i][j-1]` 為真，則 `dp[i][j] = true`。
* 否則，`dp[i][j] = false`。

```cpp
class Solution {
public:
    bool isInterleave(string s1, string s2, string s3) {
        int n1 = s1.size(), n2 = s2.size(), n3 = s3.size();
        if (n1 + n2 != n3) return false;

        vector<vector<bool>> dp(n1 + 1, vector<bool>(n2 + 1, false));
        dp[0][0] = true;
        for (int i = 0; i <= n1; ++i) {
            for (int j = 0; j <= n2; ++j) {
                if (i > 0) {
                    dp[i][j] = dp[i][j] || (dp[i-1][j] && s1[i-1] == s3[i+j-1]);
                }
                if (j > 0) {
                    dp[i][j] = dp[i][j] || (dp[i][j-1] && s2[j-1] == s3[i+j-1]);
                }
            }
        }

        return dp[n1][n2];
    }

};
```

#### [Palindrome Partitioning II](https://leetcode.com/problems/palindrome-partitioning-ii/)

* **子問題的重疊性**：對於任何一個子串，如果它是一個回文，那麼在這個子串的基礎上進行分割的最優解可以用來構造整個字符串的最優解。
* **最優子結構**：整個問題的最優解可以通過組合子問題的最優解來獲得，即對於每一個位置，考慮到這個位置為止的所有回文子串分割方案，選擇分割次數最少的一個。

1. 首先，進行一次預處理來確定任意子字串是否為回文。這可以通過一個二維 boolean 陣列 `isPalindrome` 來實現，其中 `isPalindrome[i][j]` 表示從 `i` 到 `j`（含）的子字串是否為回文。
2. **計算最小分割次數**：使用一個一維數組 `dp`，其中 `dp[i]` 表示從字符串開始到第 `i` 個字符的最小分割次數。對於每個 `j`，如果子串 `s[j...i]` 是回文，則 `dp[i] = min(dp[i], dp[j-1]+1)`。

```cpp
class Solution {
public:
    int minCut(string s) {
        int n = s.length();
        vector<vector<bool>> isPalindrome(n, vector<bool>(n, false));
        vector<int> dp(n);
        
        for (int i = 0; i < n; ++i) {
            dp[i] = i;
            for (int j = 0; j <= i; ++j) {
                if (s[i] == s[j] && (i - j <= 2 || isPalindrome[j+1][i-1])) {
                    isPalindrome[j][i] = true;
                    dp[i] = j == 0 ? 0 : min(dp[i], dp[j-1] + 1);
                }
            }
        }
        
        return dp[n-1];
    }

};
```

#### [Minimum Cost Tree From Leaf Values](https://leetcode.com/problems/minimum-cost-tree-from-leaf-values)

這題看起來是樹 DP，但其實可以轉化為：給定一個陣列`A`，每次選擇陣列中相鄰的兩個元素`a`和`b`，移除較小的一個`min(a, b)`，這次操作的代價是`a * b`。問題求解的目標是找出移除整個陣列直到只剩一個元素的最小代價。

使用 stack 來追踪可能的移除操作，以保證每次操作都能達到局部最優，從而達到全局最優。具體做法是：

* 初始化一個棧`stk`，並將`INT_MAX`放入 stack ，以方便處理邊界情況。
* 遍歷陣列`arr`中的每個元素`x`：
  * 當當前元素`x`大於棧頂元素時，表示找到了一個局部的最小代價移除操作的機會。此時，將棧頂元素彈出，計算移除的代價（即棧頂元素與其下一個元素和`x`中較小者的乘積），並加到總代價`res`上。
  * 將當前元素`x`推入棧中。
* 最後，棧中除了`INT_MAX` 外可能還剩下一些元素，這些元素是按照從小到大的順序排列的。因此，從第二個元素開始，將每個元素與其前一個元素的乘積加到總代價`res`上就是答案。

```cpp
class Solution {
public:
    int mctFromLeafValues(vector<int>& arr) {
        int res = 0;
        vector<int> stk = {INT_MAX};
        for (int x : arr) {
            while (stk.back() <= x) {
                int mid = stk.back();
                stk.pop_back();
                res += mid * min(stk.back(), x);
            }
            stk.push_back(x);
        }
        for (int i = 2; i < stk.size(); i++) {
            res += stk[i] * stk[i - 1];
        }
        return res;
    }
};
```
