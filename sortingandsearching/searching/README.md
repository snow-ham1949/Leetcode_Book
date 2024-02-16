---
description: 搜尋
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

# 搜尋

### 整數二分搜

如果我們知道要搜尋的東西有單調性，意思是整個序列的變化是單調的，那我們就可以使用二分搜。

這邊直接給一個可以套用到任何情境的模板

```cpp
while (r - l > 1) {
    int m = l + (r - l) / 2;
    if (chk(m)) l = m; // or r = m;
    else r = m; // or l = m;
}
```

這個模板是什麼意思呢？

我們可以想像對於要搜尋的東西，把包含他的部分稱為左邊，剩下的稱為右邊，那我們可以發現，左邊都會是符合搜尋條件的，右邊則不是。這樣的話就可以套用第一種沒有註解的模板，如果符合條件，我們就試著把界往上壓，因為停止條件是 r - l > 1，所以最後 l 會是符合條件最大的， r 則是不符合條件最小的。

反之的話可以想想看，結論是 l 會是不符合條件最大的，r 會是符合條件最小的。

要注意自己的定義是上面的哪一種，根據自己的定義來設定 l, r 的初始值。

當然也可以利用 STL 中的 lower\_bound 跟 upper\_bound，這邊就不細講用法。

#### 對序列二分搜

假設序列存在單調性，我們就可以對他二分搜，舉一個簡單的例子

> 給一個序列 arr，求小於 x 中最大的數字，若無則回傳 -1。

首先我們需要序列有單調性，所以我們可以使用 sort 使他有單調性。接下來就套用上面的模板，我們先假設有解，所以需要「l 會是符合條件最大的」，因此 l 一開始就要是一個符合條件的值，而 r 則是不符合條件的最小值。為什麼要先假設有解呢，因為這樣在二分搜的時候才不會 index out of bound. 至於有沒有解，最後看 l 是不是 < x 就可以了。

```cpp
int l = 0, r = arr.length();
while (r - l > 1) {
    int m = l + (r - l) / 2;
    if (chk(m)) l = m;
    else r = m;
}
if (arr[l] < x) return -1;
else return x;
```

#### 對答案二分搜

直接給例題比較方便講解。

> Given an array of integers `nums` containing `n + 1` integers where each integer is in the range `[1, n]` inclusive.
>
> There is only **one repeated number** in `nums`, return _this repeated number_.
>
> You must solve the problem **without** modifying the array `nums` and uses only constant extra space.

可以修改序列或者用 O(n) 的 space 就不需要使用二分搜，考慮在題目的限制下，我們試著進行二分搜。

可以知道答案一定落在 \[1, n] 之間，那麽答案有沒有單調性呢？答案是有的，假設今天有一個數字 x ，序列中 <= x 的數字有 cnt 個，那個 cnt > x 的話就代表答案存在 \[1, x] 之間，因為一定有 <= x 的人重複了，才會讓 cnt > x。所以繼續套用上面的模板，假設 r 是最小符合的答案（為什麼不用 l ？因為 >= 重複數字的數字才會符合上面的條件，因此我們希望越小越好）。

```cpp
class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        int l = 0, r = nums.size() - 1;
        auto chk = [&nums](int x) {
            int cnt = 0;
            for (int &xx : nums) {
                if (xx <= x) cnt++;
            }
            if (cnt > x) return 1;
            else return 0;
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

### 浮點數二分搜

二分搜一個很重要的概念就是會把數字切成兩半，如果今天要搜的東西是浮點數的話，那要怎麼確認精度呢？可以想像 2^30 大概會是 1e6，所以做 30 次對半切精度就可以達到 1e-6，大概是這種概念，一樣附上模板。

```cpp
for (int i = 0; i < 30; i++) { 
    double m = (l + r) / 2.0;
    if (chk(m)) l = m;
    else r = m;
}
```
