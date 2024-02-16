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

### 基礎二分搜

#### [Intersection of Two Arrays](https://leetcode.com/problems/intersection-of-two-arrays/)

如果一個序列是有序的，那我們要找他在不在裡面就可以使用二分搜。這題的概念就是這樣，如果我可以先把其中一個序列排好（並且把重複的去掉），那我就可以遍歷另一個序列，然後使用二分搜找他是不是在排好的那個序列裡面。

```cpp
class Solution {
public:
    void pre(vector<int> &v) {
        sort(v.begin(), v.end());
        v.erase(unique(v.begin(), v.end()), v.end());
    }
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        pre(nums1); 
        pre(nums2);
        vector<int> v;
        for (int &x : nums1) {
            auto it = lower_bound(nums2.begin(), nums2.end(), x);
            if (it != nums2.end() && *it == x) {
                v.push_back(x);
            }
        }
        return v;
    }
};
```

#### [Arranging Coins](https://leetcode.com/problems/arranging-coins/)

如果想要排最多的 row，那肯定是從最少的開始排。再加上發現如果現在的錢幣可以滿足 x 個 row，那麼他一定可以滿足 x - 1 個 row。有了這個單調性我們就可以對答案二分搜。驗證現在的 row 可不可行用國中數學教過的 1 + 2 + ... + n = n(n + 1) / 2 就可以了。

```cpp
class Solution {
public:
    int arrangeCoins(int n) {
        long long int l = 0, r = (long long int)n + 5;
        auto chk = [&](long long int x) {
            return x * (x + 1) / 2 <= (long long int)n;
        };
        while (r - l > 1) {
            long long int m = (l + r) / 2;
            if (chk(m)) l = m;
            else r = m;
        }
        return l;
    }
};
```

#### [Valid Perfect Square](https://leetcode.com/problems/valid-perfect-square/)

這題規定不能用 sqrt。那換一個想法，我們知道如果他是一個平方數的話，他肯定可以開根號，且開根號後的數值一定小於自己。再加上數字的平方大小是有單調性的。所以我們就可以使用二分搜來找跟它開根號之後最接近的值，最後再驗平方是不是它就可以了。

```cpp
class Solution {
public:
    bool isPerfectSquare(int num) {
        long long int l = 0, r = (long long int)num + 5;
        while (r - l > 1) {
            long long int m = (l + r) / 2;
            if (m * m <= num) l = m;
            else r = m;
        }
        return l * l == (long long int)num;
    }
};
```

#### [Guess Number Higher or Lower](https://leetcode.com/problems/guess-number-higher-or-lower/)

想要講這題只是因為他在算 mid 的時候會溢位所以要特別注意，要使用模板中比較特別的寫法。

```cpp
/** 
 * Forward declaration of guess API.
 * @param  num   your guess
 * @return 	     -1 if num is higher than the picked number
 *			      1 if num is lower than the picked number
 *               otherwise return 0
 * int guess(int num);
 */

class Solution {
public:
    int guessNumber(int n) {
        int l = 0, r = n;
        auto chk = [](int x) {
            int res = guess(x);
            return res <= 0;
        };
        while (r - l > 1) {
            int m = l + (r - l) / 2;
            if (chk(m)) r = m;
            else l = m;
        }
        return r;
    }
};
```

#### [Nth Magical Number](https://leetcode.com/problems/nth-magical-number/)

這題的話其實也是國中數學而已，知道要怎麼算 x 是第幾個 divisible by a, b 的就可以二分搜了，因為 divisibe by a, b 的數字也是具有單調性的。

```cpp
class Solution {
public:
    const int mod = 1e9 + 7;
    int nthMagicalNumber(int n, int a, int b) {
        long long int l = 0, r = 1e18;
        long long int gcd = std::gcd(a, b), lcm = a * b / gcd;
        auto chk = [&](long long int x) {
            return x / a + x / b - x / lcm >= (long long int)n;
        };
        while (r - l > 1) {
            long long int m = l + (r - l) / 2;
            if (chk(m)) r = m;
            else l = m;
        }
        return r % mod;
    }
};
```

### 模擬 + 二分搜

#### [Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/)

先想想看答案有沒有單調性，答案是有的，因爲如果我們假設最大的總和是 x，且他是被切成 k 段中的最大值，那麽 x + 1 就是一個更鬆的界，一定更能夠切成 k 段，且每段都不超過 x + 1。

那接下來我們要怎麼驗 x 能不能是最大值呢？這部分可以考慮模擬，如果有任何一個數字大於 x，那麼代表 x 絕對不會是最大值，再來我們可以考慮現在的總和，如果加上現在模擬到的數字不超過 x，那麼就加上去，否則就代表我們現在要在這裡切一段。最後看切出來的段數有沒有 <= k 即可。

```cpp
class Solution {
public:
    int splitArray(vector<int>& nums, int k) {
        int l = -1, r = accumulate(nums.begin(), nums.end(), 0);
        auto chk = [&](int lim) {
            int cnt = 1, cur = 0;
            for (int &x : nums) {
                if (x > lim) return false;
                if (cur + x <= lim) cur += x;
                else {
                    cnt++;
                    cur = x;
                }
            }
            return cnt <= k;
        };
        while (r - l > 1) {
            int m = (l + r) / 2;
            if (chk(m)) r = m;
            else l = m;
        }
        return r;
    }
};
```

#### [Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/)

一樣我們可以發現有單調性，如果在 speed k 內可以吃完，那代表 k + 1 一定也可以吃完，所以嘗試對答案二分搜。那要怎麼驗證 speed k 在 h 內吃不吃得完呢？一樣可以模擬，如果一堆的數量 <= k 那麼一次就可以吃完，不然我們就算一下這堆要吃多久，最後統計一下看能不能在 h 內吃完就可以了。

```cpp
class Solution {
public:
    int minEatingSpeed(vector<int>& piles, int h) {
        sort(piles.begin(), piles.end());
        long long int l = 0, r = 1e9 + 1;
        auto chk = [&](long long int k) {
            long long int cnt = 0;
            for (int &x : piles) {
                if (x <= k) cnt++;
                else cnt += ((long long int)x + k - 1) / k;
            }
            return cnt <= h;
        };
        while (r - l > 1) {
            long long int m = l + (r - l) / 2;
            if (chk(m)) r = m;
            else l = m;
        }
        return r;
    }
};
```

### 雙指針＋二分搜

#### [Kth Smallest Number in Multiplication Table](https://leetcode.com/problems/kth-smallest-number-in-multiplication-table/)

一樣可以先想到，如果數字 x 在乘法表裡面有至少 k 個數字 <= 它，那麽 x + 1 也一定至少有 k 個數字 <= 它，所以可以二分搜。

那要怎麼驗正數字 x 在乘法表裡面有多少數字 <= 它呢？觀察一下乘法表，我們可以發現，如果從最左下角開始走，每次看當前的格子是否 <= x ，如果是，就代表這整個 column 都 <=，否則就往上走一格再看，這樣就可以在 O(m + n) 的時間內完成。

```cpp
class Solution {
public:
    int findKthNumber(int m, int n, int k) {
        int l = 0, r = m * n;
        auto chk = [&](int num) {
            int x = m, y = 1, cnt = 0;
            while (x >= 1 && y <= n) {
                if (x * y <= num) {
                    cnt += x;
                    y++;
                } else {
                    x--;
                }
            }
            return cnt >= k;
        };
        while (r - l > 1) {
            int m = (l + r) / 2;
            if (chk(m)) r = m;
            else l = m;
        }
        return r;
    }
};
```

#### [Find K-th Smallest Pair Distance](https://leetcode.com/problems/find-k-th-smallest-pair-distance/)

這題的單調性跟上一題很像所以略過，我們來想要如何驗證一個 x 是不是有 >= k 個 difference <= 它。

假設我們先 sort 好，那麼對於每一個 i (0 <= i < n) ，我們都希望找到一個 j 使得 nums\[i] - nums\[j] <= x ，因為這樣我們就知道有 i - j + 1 個數量的 difference <= x，這樣就可以線性做完。又因為他是 sort 好的，所以 j 只會遞增不會遞減，所以我們可以使用雙指針來 O(n) 驗證。

```cpp
class Solution {
public:
    int smallestDistancePair(vector<int>& nums, int k) {
        sort(nums.begin(), nums.end());
        int l = -1, r = nums.back() - nums.front();
        int n = nums.size();
        auto chk = [&](int x) {
            int cnt = 0;
            for (int i = 0, j = 0; i < n; i++) {
                while (j < i && nums[i] - nums[j] > x) j++;
                cnt += (i - j);
            }
            return cnt >= k;
        };
        while (r - l > 1) {
            int m = (l + r) / 2;
            if (chk(m)) r = m;
            else l = m;
        }
        return r;
    }
};
```

#### [Heaters](https://leetcode.com/problems/heaters/)

一樣先看答案有沒有單調性，如果半徑為 x 可以蓋到所有的 houses，那代表 x + 1 肯定也可以。所以具有單調性可以二分搜。

那驗證是不是可以用一些或所有的 heaters 蓋到所有的 houses，我們可以考慮先把用 heaters 跟 houses sort 好之後用兩個指針來檢查，對於一個 heaters ，我們可以爬行去看他可以蓋到哪些 house，如果蓋不到了，那就換下一個 heaters；反之則檢查下一棟房子，如果這時候房子都蓋到了，那代表這個半徑可以。如果 heaters 都用完了但還有房子還沒蓋到，那就是不行。

```cpp
class Solution {
public:
    int findRadius(vector<int>& houses, vector<int>& heaters) {
        sort(houses.begin(), houses.end());
        sort(heaters.begin(), heaters.end());
        int n = houses.size(), m = heaters.size();
        auto chk = [&](long long int rad) {
            int ind_heater = 0, ind_house = 0;
            while (ind_heater < m) {
                while (ind_heater < m && ind_house < n && houses[ind_house] >= heaters[ind_heater] - rad && houses[ind_house] <= heaters[ind_heater] + rad) {
                    ind_house++;
                } 
                if (ind_house == n) return true;
                else ind_heater++;
            }
            return false;
        };
        long long int l = -1, r = 1e10;
        while (r - l > 1) {
            long long int m = (l + r) / 2;
            if (chk(m)) r = m;
            else l = m;
        }
        return r;
    }
};
```

### dp + 二分搜

這部分比較難，可以等看完後面的 dp 之後再回來看，只是想說有類似的技巧就先整理再一起。

#### [Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)

一般的 LIS 有 O(n^2) 的做法，不過我們可以做到 O(nlgn)，對於每一個數字，我們可以去看可以不可以插入 LIS，假設 LIS 的數都比他小，那他可以插在最後面，否則的話我們可以考慮替換掉原本存在 LIS 的數字，因為一樣的長度的話，數字當然越小越好，這樣才能組成更長的 LIS。

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> dp;
        for (int &x : nums) {
            auto id = lower_bound(dp.begin(), dp.end(), x) - dp.begin();
            if (id == dp.size()) dp.push_back(x);
            else dp[id] = x;
        }
        return dp.size();
    }
};
```

#### [Russian Doll Envelopes](https://leetcode.com/problems/russian-doll-envelopes/)

觀察一下就會發現，這根本就是 sort 完之後做 LIS，那 sort 的概念前面也說過了，不過這邊我們要想一下怎麼 sort 才可以好好的建 LIS。我們需要長和寬都比前面的大，所以 sort 的時候可以考慮同一個長的 envelope，寬較大的放在前面，不同長的話，小的放在前面。

這樣的話我們就可以用寬來做 LIS，並且因為長已經排序過，所以在做 LIS 的時候只要考慮寬就可以了，因為後面的長度總是 >= 前面的長度。

```cpp
class Solution {
public:
    static bool cmp(vector<int> &a, vector<int> &b) {
        if (a[0] == b[0]) return a[1] > b[1];
        return a[0] < b[0];
    }
    int maxEnvelopes(vector<vector<int>>& envelopes) {
        sort(envelopes.begin(), envelopes.end(), cmp);
        int n = envelopes.size();
        vector<int> dp;
        for (int i = 0; i < n; i++) {
            auto it = lower_bound(dp.begin(), dp.end(), envelopes[i][1]);
            if (it == dp.end()) dp.push_back(envelopes[i][1]);
            else *it = envelopes[i][1];
        }
        return dp.size();
    }
};
```

### bfs + 二分搜

bfs 一樣後面才會提到，所以可以等看完後面再來看這裡。

#### [Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/)

一樣我們可以對答案二分搜，如果在某個時間 t 可以到達終點的話，代表 t + 1 一定也可以到達終點。那要怎麼驗證時間 t 能不能達到終點呢？如果會 bfs 的話就可以想到，那我就直接 O(n + n) 去看從 (0, 0) 開始試著走，如果可以到就 return true，不行就 return false。

```cpp
class Solution {
public:
    int d[5] = {0, -1, 0, 1, 0};
    int swimInWater(vector<vector<int>>& grid) {
        int n = grid.size();
        int l = -1, r = n * n;
        auto bound = [&](int x, int y) {
            return x >= 0 && x < n && y >= 0 && y < n;
        };
        auto chk = [&](int time) {
            if (grid[0][0] > time) return false;
            queue<pair<int, int>> q;
            q.push({0, 0});
            vector<vector<bool>> vis(n, vector<bool>(n, false));
            vis[0][0] = 1;
            while (!q.empty()) {
                auto [x, y] = q.front();
                q.pop();
                if (x == n - 1 && y == n - 1) return true;
                for (int i = 0; i < 4; i++) {
                    int dx = x + d[i], dy = y + d[i + 1];
                    if (bound(dx, dy) && grid[dx][dy] <= time && !vis[dx][dy]) {
                        vis[dx][dy] = 1;
                        q.push({dx, dy});
                    }
                }
            }
            return false;
        };
        while (r - l > 1) {
            int m = (l + r) / 2;
            if (chk(m)) r = m;
            else l = m;
        }
        return r;
    }
};
```
