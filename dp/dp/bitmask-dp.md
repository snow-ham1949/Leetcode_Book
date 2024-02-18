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

# Bitmask DP

#### [Subsets](https://leetcode.com/problems/subsets)

這題其實不是 DP，只是枚舉子集合的技巧，但是後面會用到 bitmask 的部分，所以放在這裡。

```
class Solution {
public:
    vector<vector<int>> subsets(vector<int>& nums) {
        int n = nums.size();
        vector<vector<int>> res;
        for (int s = 0; s < (1 << n); s++) { // subset, 二進位
            vector<int> v;
            for (int i = 0; i < n; i++) {
                if (s & (1 << i)) {
                    v.push_back(nums[i]);
                }
            }
            res.push_back(v);
        }
        return res;
    }
};
```

#### [Partition to K Equal Sum Subsets](https://leetcode.com/problems/partition-to-k-equal-sum-subsets)

1. **子問題重疊**：試圖將原問題分解為較小的問題時，我們發現對於給定的子集合，判斷其是否能分割成具有特定和的較小子集是一個重複出現的問題。
2. **最優子結構**：原問題的解可以由子問題的解直接得出。如果我們能夠找到一種方式，將數組分割成 k-1 個和相等的子集，那麼剩下的元素自然形成第 k 個和相等的子集。

使用 bitmask 作為狀態表示，其中每個位的 0 或 1 表示對應位置的數字是否被選中。動態規劃的狀態可以表示為 dp\[mask]，表示選取的元素集合為 mask 時，是否可以滿足條件。

1. **初始化**：dp\[0] = true，表示沒有選擇任何元素時，條件被滿足（沒有元素的情況被認為是合法的）。
2. **狀態轉移**：對於每個狀態 mask，遍歷所有元素，嘗試加入未被選中的元素，並檢查是否可以形成一個和為目標值（總和/k）的子集。如果可以，則更新 dp 狀態為 true。
3. **目標**：檢查 dp\[(1<\<n)-1] 是否為 true，其中 n 是數組 nums 的長度，(1<\<n)-1 表示所有元素都被選中的狀態。

```cpp
class Solution {
public:
    bool canPartitionKSubsets(vector<int>& nums, int k) {
        int sum = accumulate(nums.begin(), nums.end(), 0);
        if (sum % k != 0) return false;
        int target = sum / k;
        int n = nums.size();
        vector<bool> dp(1 << n, false);
        vector<int> total(1 << n, 0);
        dp[0] = true;

        for (int mask = 0; mask < (1 << n); ++mask) {
            if (!dp[mask]) continue;
            for (int i = 0; i < n; ++i) {
                if (!(mask & (1 << i)) && total[mask] + nums[i] <= target) {
                    dp[mask | (1 << i)] = true;
                    total[mask | (1 << i)] = (total[mask] + nums[i]) % target;
                }
            }
        }

        return dp[(1 << n) - 1];
    }
};
```

#### [Number of Ways to Wear Different Hats to Each Other](https://leetcode.com/problems/number-of-ways-to-wear-different-hats-to-each-other/)

1. **狀態壓縮和位運算**：由於每個帽子只能被一個人戴，我們可以使用位來表示帽子的分配狀態，這是一個典型的狀態壓縮問題，適合用動態規劃來解決。
2. **重複子問題**：如果我們已經知道了前 i-1 個帽子分配的方法數，當我們考慮第 i 個帽子分配給不同人的情況時，可以重用之前的計算結果。
3. **最優子結構**：每一步的答案依賴於之前步驟的答案，這代表可以通過組合之前的解來構建當前的解。

**初始化**：`dp[0][0] = 1`，沒有帽子分配給任何人時，只有一種方法。

1. **狀態轉移**：對於每一頂帽子 i，遍歷所有人 j，如果人 j 可以戴上帽子 i，並且在當前狀態 mask 中人 j 沒有帽子（`mask & (1 << j) == 0`），則我們可以更新狀態為 `dp[mask | (1 << j)][i + 1] += dp[mask][i]`。
2. **結果**：計算所有人都有帽子的狀態下，考慮完所有帽子後的方法總數，即 `dp[(1<<n)-1][m]`。

定義 `dp[mask][i]` 為當前已分配帽子的狀態為 mask（每個位表示一個人是否已經有帽子），且只考慮前 i 個帽子時，的分配方法總數。我們需要計算的終態是 `dp[(1<<n)-1][m]`，其中 n 是人數，m 是帽子數量。

```cpp
class Solution {
public:
    int dp[1025][45];
    int numberWays(vector<vector<int>>& hats) {
        const int mod = 1e9+7;
        int n = hats.size();
        dp[0][0] = 1; 
        vector<vector<int>> people(41); 

        for (int i = 0; i < n; ++i) {
            for (int hat : hats[i]) {
                people[hat].push_back(i);
            }
        }

        for (int i = 0; i <= 40; ++i) { 
            for (int mask = 0; mask < (1 << n); ++mask) {
                dp[mask][i+1] = (dp[mask][i+1] + dp[mask][i]) % mod; 
                for (int person : people[i]) {
                    if (mask & (1 << person)) continue; 
                    dp[mask | (1 << person)][i+1] = (dp[mask | (1 << person)][i+1] + dp[mask][i]) % mod; // 戴上這頂帽子的情況
                }
            }
        }

        return dp[(1<<n)-1][41];
    }
};
```
