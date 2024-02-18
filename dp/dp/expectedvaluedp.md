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

# 期望 DP

#### [New 21 Game](https://leetcode.com/problems/new-21-game)

1. **機率與期望的關係**：這個問題自然地與機率和期望相關聯，因為我們要計算的是在給定遊戲規則下，達成某種結果（即總分數落在特定範圍內）的機率。
2. **過程的無後效性**：無論 Alice 目前的分數是多少，她接下來的決策和勝率只依賴於當前的狀態，而不是她是如何達到這個狀態的。這是動態規劃適用的一個重要特性。
3. **子問題的重複性**：在計算達到某一分數點的勝率時，我們需要依賴於達到其他分數點的勝率，這些子問題會被重複計算多次，適合用 DP 來優化。

#### 解法

* 定義一個一維 DP 數組 `dp[i]` 來表示達到分數 `i` 時的勝率。
* 對於每一個分數 `i`，其勝率可以由前 `W` 個分數的勝率加權平均得到，因為從任何一個較小的分數抽到一張牌達到 `i` 的機率是相等的。
* 計算時需要注意邊界條件，即當分數超過 `K` 時，應停止抽牌，因此超過 `K` 分後的勝率需要特別處理。

```cpp
class Solution {
public:
    double new21Game(int N, int K, int W) {
        if (K == 0 || N >= K + W) return 1.0;
        vector<double> dp(N + 1);
        dp[0] = 1.0;
        double Wsum = 1.0, res = 0.0;
        for (int i = 1; i <= N; ++i) {
            dp[i] = Wsum / W;
            if (i < K) Wsum += dp[i];
            else res += dp[i];
            if (i - W >= 0) Wsum -= dp[i - W];
        }
        return res;
    }
};

```

#### [Soup Servings](https://leetcode.com/problems/soup-servings)

1. **問題的機率性質**：由於問題求解的是 A 贏的機率，自然聯想到期望和機率的計算。
2. **狀態的轉移**：每次操作後湯的剩餘量變化，可以看作是一個狀態轉移過程，而每個狀態的勝利機率取決於前一狀態，適合用動態規劃來解。

* 定義 DP 數組 `dp[i][j]` 表示當 A 剩餘 `i` 毫升，B 剩餘 `j` 毫升時，A 贏的機率。
* 遞推關係基於四種操作，每種操作後，A 贏的機率是四種操作導致的新狀態下 A 贏機率的加權平均。
* 特別注意 N 很大時的優化，因為當 N 非常大時，A 贏的機率會趨近於一定值，可以設定一個閾值，當 N 超過這個閾值時，直接返回這個趨近值，以提高計算效率。

```cpp
class Solution {
public:
    double soupServings(int N) {
        if (N > 4800) return 1.0; 
        int n = (N + 24) / 25;
        vector<vector<double>> dp(n + 1, vector<double>(n + 1, 0.0));
        for (int i = 0; i <= n; ++i) {
            for (int j = 0; j <= n; ++j) {
                if (i == 0) dp[i][j] = 1;
                if (i == 0 && j == 0) dp[i][j] = 0.5;
                if (i > 0 && j > 0) {
                    dp[i][j] = 0.25 * (dp[max(i - 4, 0)][j] + dp[max(i - 3, 0)][max(j - 1, 0)] +
                                       dp[max(i - 2, 0)][max(j - 2, 0)] + dp[max(i - 1, 0)][max(j - 3, 0)]);
                }
            }
        }
        return dp[n][n];
    }
};
```

#### [Knight Probability in Chessboard](https://leetcode.com/problems/knight-probability-in-chessboard)

1. **機率計算**：問題本質上是求給定條件下的一個機率問題，涉及到的是在不同狀態間轉移的機率計算。
2. **狀態轉移**：騎士的每一步移動都可以看作是從當前狀態（即其位置）到下一個狀態的轉移，這些狀態間的轉移具有無後效性，即後續狀態的機率僅依賴於當前狀態，不依賴於到達當前狀態的路徑。

#### 解法

* 使用一個三維 DP 數組 `dp[k][i][j]`，表示騎士從 `(i, j)` 出發，還剩下 `k` 步時留在棋盤上的機率。
* 初始化 `dp[0][i][j] = 1`，因為在沒有移動前，騎士肯定是在棋盤上的。
* 對於每個 `k` 從 1 到 `K`，更新 `dp[k][i][j]`，根據騎士能移動的八個方向來計算留在棋盤上的機率。

```cpp
class Solution {
public:
    double knightProbability(int N, int K, int r, int c) {
        vector<vector<vector<double>>> dp(K+1, vector<vector<double>>(N, vector<double>(N, 0)));
        int dr[] = {-2, -1, 1, 2, 2, 1, -1, -2};
        int dc[] = {1, 2, 2, 1, -1, -2, -2, -1};
        
        dp[0][r][c] = 1; 
        for (int k = 1; k <= K; ++k) {
            for (int i = 0; i < N; ++i) {
                for (int j = 0; j < N; ++j) {
                    for (int d = 0; d < 8; ++d) {
                        int ni = i + dr[d], nj = j + dc[d];
                        if (ni >= 0 && ni < N && nj >= 0 && nj < N) { 
                            dp[k][ni][nj] += dp[k-1][i][j] / 8.0; 
                        }
                    }
                }
            }
        }
        
        double ans = 0;
        for (int i = 0; i < N; ++i) {
            for (int j = 0; j < N; ++j) {
                ans += dp[K][i][j]; 
            }
        }
        return ans;
    }
};
```

#### [Stone Game IV](https://leetcode.com/problems/stone-game-iv)

**狀態轉移的特性**：問題可以分解成較小的子問題，即考慮在剩下某個數量的石頭時，當前玩家能否勝利。

* 定義 DP 數組 `dp[i]` 表示當剩下 `i` 塊石頭時，當前玩家是否能贏得遊戲（`true` 為能贏，`false` 為不能贏）。
* 初始化 `dp[0] = false`，因為當沒有石頭時，當前玩家直接輸。
* 對於每個狀態 `i`，遍歷所有可能移除的石頭數量，如果存在一種移除方式使得對手處於輸的狀態（即 `dp[i - j*j] = false`），則當前玩家能贏（`dp[i] = true`）。

```cpp
class Solution {
public:
    bool winnerSquareGame(int n) {
        vector<bool> dp(n + 1, false);
        for (int i = 1; i <= n; ++i) {
            for (int j = 1; j * j <= i; ++j) {
                if (!dp[i - j * j]) { 
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[n];
    }
};
```
