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

# 背包 DP

#### [Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/)

#### 1. 如何轉換成背包問題

這個問題可以轉換成一個01背包問題。給定一個集合，可以選擇是否將集合中的元素放入背包中，使得背包中的元素總和等於所有元素總和的一半。具體來說，如果集合中元素的總和是奇數，那麼無法平分成兩部分，直接返回false。如果是偶數，問題就轉化為能否從集合中選擇一部分元素，使得這些元素的總和等於所有元素總和的一半。

#### 2. 解法

`dp[i][j]`表示從前`i`個數字中選擇一些數字，是否可以使得這些數字的總和恰好為`j`。

狀態轉移方程為：

* 如果不選擇當前數字`nums[i]`，則`dp[i][j] = dp[i-1][j]`
* 如果選擇當前數字`nums[i]`，則`dp[i][j] = dp[i-1][j-nums[i]]`
* 初始條件為`dp[0][0] = true`，表示沒有選擇任何數字時，總和為0是可能的。

```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int sum = 0;
        for (int num : nums) sum += num;
        if (sum % 2 != 0) return false;

        int target = sum / 2;
        vector<vector<bool>> dp(nums.size() + 1, vector<bool>(target + 1, false));
        dp[0][0] = true;

        for (int i = 1; i <= nums.size(); i++) {
            for (int j = 0; j <= target; j++) {
                dp[i][j] = dp[i-1][j];
                if (j >= nums[i-1]) {
                    dp[i][j] = dp[i][j] || dp[i-1][j-nums[i-1]];
                }
            }
        }

        return dp[nums.size()][target];
    }
};
```

#### [Ones and Zeroes](https://leetcode.com/problems/ones-and-zeroes/)

#### 1. 如何轉換成背包問題

給定一組二進制字串和兩個整數m和n，問題要求選出最多的字符串，使得其中0的數量不超過m個，1的數量不超過n個。這個問題可以轉化為二維背包問題，其中每個字符串可以看作是一件物品，「0的數量」和「1的數量」分別對應到二維背包中的兩個維度的「重量限制」。

#### 2. 解法

`dp[i][j][k]`的值表示在前`i`個字串中，使用不超過`j`個0和`k`個1時，可以選出的最大字符串數量。

狀態轉移方程如下：

* 如果不選當前的字符串，則`dp[i][j][k] = dp[i-1][j][k]`
* 如果選擇當前的字符串，假設當前字符串包含`zeros`個0和`ones`個1，則`dp[i][j][k] = max(dp[i-1][j][k], 1 + dp[i-1][j-zeros][k-ones])`

觀察到每個狀態`dp[i][j][k]`只依賴於`dp[i-1][...]`的前一狀態。這代表在計算`dp[i][...]`時，只需要知道`dp[i-1][...]`的值。因此，我們不需要保留一個維度來表示考慮到的字串索引，只需使用兩個維度來追踪0和1的數量限制即可。

#### 優化後的設計：

* **二維陣列**：`dp[j][k]`，現在表示對於當前考慮的字符串，最多可以使用`j`個0和`k`個1時的最大字符串數量。
* **狀態轉移**：依然是考慮是否選擇當前字符串，但更新方式變為`dp[j][k] = max(dp[j][k], 1 + dp[j-zeros][k-ones])`，其中`zeros`和`ones`是當前字符串中0和1的數量。

```
class Solution {
public:
    int findMaxForm(vector<string>& strs, int m, int n) {
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
        for (string& s : strs) {
            int zeros = count(s.begin(), s.end(), '0');
            int ones = s.length() - zeros;
            for (int i = m; i >= zeros; i--) {
                for (int j = n; j >= ones; j--) {
                    dp[i][j] = max(dp[i][j], 1 + dp[i - zeros][j - ones]);
                }
            }
        }
        return dp[m][n];
    }
};
```

#### [Coin Change](https://leetcode.com/problems/coin-change)

#### 1. 如何轉換成背包問題

這個問題可以看作是一種完全背包問題的變形。在完全背包問題中，每種物品（這裡指硬幣）可以無限次使用，目標是填滿一個固定容量的背包（這裡指達到總金額）。與完全背包問題的不同之處在於，這個問題的目標是最小化使用到的物品數量（即硬幣數量），而不是最大化背包的價值。

#### 2. 解法

&#x20;`dp[i]` 表示組成金額 `i` 需要的最小硬幣數量。初始時，`dp[0] = 0` 因為金額 0 不需要任何硬幣，其餘的 `dp[i]` 初始化為一個大數，表示無法達到該金額。

對於每個硬幣面額 `coin` 和每個可能的金額 `i`，更新 `dp[i]` 為 `min(dp[i], dp[i - coin] + 1)`。這表示如果當前硬幣可以被用來組成金額 `i`，則可以嘗試使用這個硬幣，看看是否能得到更少的硬幣數量。

```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        vector<int> dp(amount + 1, amount + 1);
        dp[0] = 0;

        for (int i = 1; i <= amount; i++) {
            for (int coin : coins) {
                if (i - coin >= 0) {
                    dp[i] = min(dp[i], dp[i - coin] + 1);
                }
            }
        }

        return dp[amount] > amount ? -1 : dp[amount];
    }
};
```

#### [Coin Change II](https://leetcode.com/problems/coin-change-ii)

#### 1. 如何轉換成背包問題

這個問題可以轉換成一個經典的完全背包問題，其中每種硬幣可以無限次使用，目標是找到所有可能的方式來填滿總金額的背包。

#### 2. 解法

`dp[i]` 表示達到金額 `i` 的組合方式總數。初始時，`dp[0] = 1`，表示金額為 0 有一種方式（不使用任何硬幣）。

對於每種硬幣，我們遍歷所有可能的金額，更新 `dp[i]` 的值為所有包含當前硬幣面額下，能達到該金額的組合總數。對於每個金額 `i` 和每個硬幣 `coin`，更新 `dp[i]` 為 `dp[i] + dp[i - coin]`。

```cpp
class Solution {
public:
    int change(int amount, vector<int>& coins) {
        vector<int> dp(amount + 1, 0);
        dp[0] = 1;

        for (int coin : coins) {
            for (int i = coin; i <= amount; i++) {
                dp[i] += dp[i - coin];
            }
        }

        return dp[amount];
    }
};
```

#### [Perfect Squares](https://leetcode.com/problems/perfect-squares/)

#### 1. 如何轉換成背包問題

這個問題可以被看作是一個變形的完全背包問題。在這裡，我們可以無限次選擇任何一個完全平方數（相當於背包問題中的物品），目標是用這些完全平方數的和剛好填滿給定的數字 `n`（相當於背包的容量）。與傳統的背包問題不同的是，這裡我們關心的是最小化使用的完全平方數的數量，而不是最大化價值。

#### 2. 解法

`dp[i]` 表示組成數字 `i` 所需的最少完全平方數的數量。對於每個數字 `i`，我們遍歷所有小於等於 `i` 的完全平方數 `j*j`，並更新 `dp[i] = min(dp[i], dp[i - j*j] + 1)`。

```cpp
class Solution {
public:
    int numSquares(int n) {
        vector<int> dp(n + 1, INT_MAX);
        dp[0] = 0;

        for (int i = 1; i <= n; i++) {
            for (int j = 1; j * j <= i; j++) {
                dp[i] = min(dp[i], dp[i - j * j] + 1);
            }
        }

        return dp[n];
    }
};
```

#### [Number of Dice Rolls With Target Sum](https://leetcode.com/problems/number-of-dice-rolls-with-target-sum)

#### 1. 如何轉換成背包問題

這個問題可以視為一種無序組合的背包問題，其中每次骰子的投擲可以看作是選擇一種「物品」，而目標和則是「背包容量」。不同之處在於，這裡不是從固定的物品集合中選擇，而是每次都從1到骰子的面數（`f`）中選擇一個值。目標是找到所有可能的選擇方式，使得總和達到目標值 `target`。

#### 2. 解法

&#x20;`dp[i][j]` 來表示使用前 `i` 次投擲（考慮前 `i` 個骰子）能夠使得點數之和為 `j` 的方法數。初始時，`dp[0][0] = 1` 表示沒有投擲任何骰子時，達到點數和為0的方法數為1（不投擲）。

對於每次投擲，我們需要考慮所有可能的點數（1到 `f`），並更新 `dp` 表。對於每個點數 `k`，如果它小於或等於目前的目標點數 `j`，則 `dp[i][j] += dp[i-1][j-k]`。

```cpp
class Solution {
public:
    int numRollsToTarget(int d, int f, int target) {
        const int MOD = 1e9 + 7;
        vector<vector<int>> dp(d + 1, vector<int>(target + 1, 0));
        dp[0][0] = 1;

        for (int i = 1; i <= d; i++) {
            for (int j = 1; j <= target; j++) {
                for (int k = 1; k <= f && k <= j; k++) {
                    dp[i][j] = (dp[i][j] + dp[i-1][j-k]) % MOD;
                }
            }
        }

        return dp[d][target];
    }
};
```

#### [Maximum Profit in Job Scheduling](https://leetcode.com/problems/maximum-profit-in-job-scheduling)

#### 1. 如何轉換成背包問題

這個問題可以視為一種特殊的背包問題，其中每項工作可以看作是一件物品，工作的持續時間可以視為物品的重量，工作的收益則對應物品的價值。不同於傳統背包問題的是，這裡的「背包容量」不是簡單的重量限制，而是時間的連續性和不重疊的約束。目標是最大化收益，同時確保選擇的工作不會在時間上重疊。

#### 2. 解法

首先，按照工作的結束時間對所有工作進行排序。然後`dp[i]` 表示考慮到第 `i` 個工作時能獲得的最大收益。對於每項工作，有兩種選擇：做這項工作或不做。如果選擇做這項工作，需要找到最近的、不與當前工作時間重疊的之前的工作 `j`，這樣 `dp[i]` 就等於 `dp[j] + profit[i]` 和 `dp[i-1]` 中的較大者。

```cpp
class Solution {
public:
    int jobScheduling(vector<int>& startTime, vector<int>& endTime, vector<int>& profit) {
        int n = startTime.size();
        vector<vector<int>> jobs;
        for (int i = 0; i < n; ++i) {
            jobs.push_back({endTime[i], startTime[i], profit[i]});
        }
        sort(jobs.begin(), jobs.end());
        
        map<int, int> dp = {{0, 0}}; // {時間,收益}
        for (auto& job : jobs) {
            int curEndTime = job[0], curStartTime = job[1], curProfit = job[2];
            int lastProfit = prev(dp.upper_bound(curStartTime))->second;
            int newProfit = lastProfit + curProfit;
            if (newProfit > dp.rbegin()->second) {
                dp[curEndTime] = newProfit;
            }
        }
        return dp.rbegin()->second;
    }
};
```
