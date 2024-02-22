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

### 基礎數學

#### [Pow(x, n)](https://leetcode.com/problems/powx-n/)

利用分治每次切兩半的概念轉成二進位來算，所謂的快速冪

```cpp
class Solution {
public:
    double myPow(double x, int n) {
        long long int _n = abs(n);
        double res = 1.0;
        while (_n) {
            if (_n & 1) res = res * x;
            x = x * x;
            _n >>= 1;
        }
        if (n < 0) return 1.0 / res;
        else return res;
    }
};
```

#### [Arranging Coins](https://leetcode.com/problems/arranging-coins/)

可以發現從第一個開始放是最好的，所以利用 sigma 公式，對答案二分搜算出 $$\frac{x(x + 1)}{2} \leq n$$ 的 $$x$$

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

### 質因數

#### [Count Primes](https://leetcode.com/problems/count-primes/)

埃拉托斯特尼篩法（Sieve of Eratosthenes）是用來找出一定範圍內所有的質數的演算法。基本思想是從2開始，逐步篩選掉非質數，留下的就是質數。

執行步驟如下：

1. **創建一個陣列**：首先，創建一個長度為n+1的陣列（假設我們要查找的範圍是從1到n），並將2到n的每個數字標記為未篩選（即可能是質數）。
2. **篩選質數**：從列表中找到第一個標記為未篩選的數字（在第一次迭代中，這將是2）。這個數字是質數。然後，將這個質數的所有倍數（不包括質數本身）標記為已篩選（即非質數）。
3. **重複篩選過程**：繼續進行，每次都找到下一個未被篩選的數字，並將其所有倍數標記為已篩選。重複這個過程，直到達到列表的末尾。
4. **輸出質數**：列表中所有仍然標記為未篩選的數字就是1到n範圍內的所有質數。

但可能會發現下面的寫法不一樣，因為再做了一些優化。

1. 只算奇數`i = 3, i += 2`
2. `j`從`i * i` 開始，因為小於`i * i`的變數在之前就被處理過了
3. `j`以 `2 * i` 增加，因為 `i` 跟 `i + i` (偶數倍數）之間的任何數也都已經被處理過了

```cpp
class Solution {
public:
    int countPrimes(int n) {
        if (n <= 2) return 0; 

        vector<bool> prime(n + 1);
        for (int i = 3; i < n; i += 2) prime[i] = 1;

        prime[2] = 1;
        int cnt = 0;

        for (int i = 3; i * i < n; i += 2) {
            if (prime[i]) {
                for (int j = i * i; j <= n; j += 2 * i) {
                    prime[j] = 0;
                }
            }
        }

        for (int i = 2; i < n; i++) if (prime[i]) cnt++;

        return cnt;
    }
};
```

#### [Ugly Number](https://leetcode.com/problems/ugly-number/)

有 2 3 5 的話就一直除下去，直到等於 1，就代表他是由 2 3 5 組成的。

```cpp
class Solution {
public:
    bool isUgly(int n) {
        if (n == 0) return false;
        while (n % 2 == 0) n /= 2;
        while (n % 3 == 0) n /= 3;
        while (n % 5 == 0) n /= 5;
        return n == 1;
    }
};
```

#### [Ugly Number II](https://leetcode.com/problems/ugly-number-ii/)

每一次從  set 裡面刪掉最小的，然後加入他的 2 3 5 倍，就可以好好的排序跟插入了。

```cpp
class Solution {
public:
    #define ll long long int
    int nthUglyNumber(int n) {
        set<ll> s;
        s.insert(1);
        ll ans;
        for (int i = 0; i < n; i++) {
            ans = *s.begin();
            s.erase(ans);
            s.insert(ans * 2);
            s.insert(ans * 3);
            s.insert(ans * 5);
        }
        return (int)ans;
    }
};
```

### 需要想一下的數學

#### [Bulb Switcher](https://leetcode.com/problems/bulb-switcher/description/)

這個問題的直覺思路是找出經過 $$n$$ 輪後仍然亮著的燈泡數量。每一輪，我們會切換一些燈泡的狀態。

最開始所有的燈泡都是關閉的，最終只有被奇數次切換的燈泡會保持開啟。每當我們在第 $$i$$輪，就會切換所有因數包含i的燈泡。因此，我們需要找到那些有奇數個因數的燈泡，這些燈泡會被奇數次切換（每一個因數切換一次）。

可能一開始不容易理解，但通過一些例子，我們可以輕易看出完全平方數有奇數個因數，因為任何數的因數通常是成對出現的，但完全平方數的平方根與自身配對。

總結來說，我們只需找出1到n之間有多少個完全平方數。我們可以逐一檢查每個數是否為完全平方數，或者直接找出n的平方根的下整數，這個值等於該範圍內存在的完全平方數的數量。

```cpp
class Solution {
public:
    int bulbSwitch(int n) {
        return sqrt(n);
    }
};
```
