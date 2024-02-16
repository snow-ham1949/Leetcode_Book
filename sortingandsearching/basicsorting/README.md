---
description: 排序
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

# 基礎排序

### 什麽是排序

排序就是按照一定的規則把序列排好。最常遇到的就是將數字由小排成大，或者給你一個 structure，請你找出在某種規則下的第一名，這些都是排序。

很多演算法都會搭配排序使用，例如

1. 二分搜
2. DP (Dynamic Programming)
3. sliding window
4. Greedy

等等，因此排序非常重要！

### 基礎排序

可以使用 C++ 內建的 sort，例如

```cpp
vector<int> v;
sort(v.begin(), v.end())
```

這樣都是從小排到大，如果想要反著排序的話可以

```cpp
sort(v.begin(), v.end(), greater<>());
```

### 進階一點點的排序（搭配 lambda function ）

例如[這題](https://leetcode.com/problems/largest-number/description/)。

> 給定 n 個數字，求怎麼拼接可以讓組出的數字最大。

先從 n = 2 開始想，如果給兩個字串 s, t 我要怎麼知道怎樣比較大？那我可以把兩種都拼拼看，如果 s + t > t + s，那我就回傳 s + t，不然就回傳 t + s。

所以有了個概念，如果我可以把所有的東西按照這個規則排序，那再把他們拼接起來就是最大的數字了。

```cpp
sort(v.begin(), v.end(), [&](string s, string t){
    return s + t > t + s;
});
```

簡單說一下 lambda  function 的組成，分成 capture clause 跟 parameter list，capture clause 是在排序的時候如果想吃外面的變數，就要放到 \[&] 這裡，parameter list 則是後面的小括號，讓使用者傳入參數用的。

### 這個那個都是排序

如果在使用 STL 的時候，map、set、priority\_queue 等等都是可以在 insert 完之後就做好排序的資料結構！因此不一定要都用 vector，也可以使用這些 STL 去排序！
