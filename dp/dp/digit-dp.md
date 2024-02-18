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

# Digit DP

[Numbers At Most N Given Digit Set](https://leetcode.com/problems/numbers-at-most-n-given-digit-set)

1. **位數限制與選擇**：考慮到每一位上的數字需要從集合 D 中選擇，且整數的總數受 N 的位數限制，這種問題自然引導我們考慮每一位數字的選擇如何影響最終的計數。
2. **重複子問題**：從最高位到最低位逐位考慮時，每一位上的選擇依賴於更高位的數字選擇以及這些選擇如何影響剩餘位的可能性。例如，一旦某一位選的數字小於 N 的對應位數字，那麼其後的位可以自由選擇 D 中的任何數字。
3. **狀態定義與轉移**：可以定義狀態來表示當前考慮到的位數、之前的選擇是否已經確保小於 N 以及是否已經開始選擇（非前導零的情況）。

**狀態表示**：`dp[i][less][start]` 表示當前考慮到 N 的第 i 位，`less` 表示之前的選擇是否已經確保小於 N，`start` 表示是否已經開始選擇數字（處理前導零）。

1. **初始化**：最開始沒有選擇任何數字，因此 `dp[0][0][0] = 1`。
2. **狀態轉移**：對於每一位，考慮所有在 D 中的數字，更新狀態。如果選擇的數字小於 N 的當前位，則後續選擇可以任意；如果等於當前位，需要進一步考慮後續位。
3. **計數**：累加所有滿足條件的狀態。

```cpp
class Solution {
public:
    long long int mypw(long long int x, long long int k) {
        long long int res = 1;
        while (k) {
            if (k & 1) res *= x;
            x *= x;
            k >>= 1;
        }
        return res;
    }
    int atMostNGivenDigitSet(vector<string>& D, int N) {
        string S = to_string(N);
        int K = S.length();
        vector<int> dp(K+1, 0);
        dp[K] = 1; 
        for (int i = K - 1; i >= 0; --i) {
            int Si = S[i] - '0';
            for (string d : D) {
                if (stoi(d) < Si) 
                    dp[i] += pow(D.size(), K - i - 1);
                else if (stoi(d) == Si)
                    dp[i] += dp[i + 1];
            }
        }

        for (int i = 1; i < K; ++i)
            dp[0] += (int)mypw(D.size(), i);
        return dp[0];
    }
};
```

#### [Numbers With Repeated Digits](https://leetcode.com/problems/numbers-with-repeated-digits)

1. **狀態表示**：定義一個五維的動態規劃數組 `dp[pos][tight][mask][repeated][started]`，其中：
   * `pos` 表示當前處理到 N 的第幾位（從左到右）。
   * `tight` 表示當前的數位是否受到 N 的上界限制。
   * `mask`用來表示哪些數位已經在之前的數位中使用過。
   * `repeated` 表示當前已選擇的數位中是否已經有重複的數位。
   * `started` 表示是否已經開始選擇數位（即已經不再是前導零的狀態）。
2. **轉移過程**：從最高位開始，逐位枚舉可能選擇的數位。對於每一個數位，根據是否觸及 N 的上界、是否已經有數位重複、以及是否已經開始來更新狀態，並向下一位進行遞歸搜索。

```cpp
class Solution {
public:
    string N_str;
    int len;
    int dp[10][2][1024][2][2];

    int solve(int pos, int tight, int mask, int repeated, int started) {
        if (pos == len) {
            return repeated && started;
        }

        if (dp[pos][tight][mask][repeated][started] != -1) {
            return dp[pos][tight][mask][repeated][started];
        }

        int upperBound = tight ? N_str[pos] - '0' : 9;
        int ans = 0;

        for (int digit = 0; digit <= upperBound; ++digit) {
            int newTight = tight && (digit == upperBound);
            int newStarted = started || (digit != 0);
            int newMask = newStarted ? (mask | (1 << digit)) : mask;
            int newRepeated = newStarted && ((mask & (1 << digit)) || repeated);

            ans += solve(pos + 1, newTight, newMask, newRepeated, newStarted);
        }

        return dp[pos][tight][mask][repeated][started] = ans;
    }

    int numDupDigitsAtMostN(int N) {
        N_str = to_string(N);
        len = N_str.size();
        memset(dp, -1, sizeof(dp));
        return solve(0, 1, 0, 0, 0);
    }
};
```

