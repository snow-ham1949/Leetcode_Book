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

# 題目選解

### 基礎位元操作

#### [Find the Difference](https://leetcode.com/problems/find-the-difference/)

這題當然可以一位一位枚舉做，但是我們這邊要說的是位元操作，所以用位元操作解。`^` (xor) 的特性是 `a ^ a == 0`，利用這件事情，我們就可以用一個 char 去 xor 所有字元，有一樣的就會被配對掉，剩下來的就會是 char xor 的結果。

```cpp
class Solution {
public:
    char findTheDifference(string s, string t) {
        char res = '\0';
        for (auto &c : s) res ^= c;
        for (auto &c : t) res ^= c;
        return res;
    }
};
```

#### [Single Number](https://leetcode.com/problems/single-number/)

跟上一題一樣的道理。

```cpp
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int res = 0;
        for (int &v : nums) res ^= v;
        return res;
    }
};
```

### 枚舉子集合

#### [Subsets](https://leetcode.com/problems/subsets/)

用前面枚舉子集合的模板，看現在選到哪些元素就放進一個 vector 裏面，把這些 vector 搜集起來就好。

```
class Solution {
public:
    vector<vector<int>> subsets(vector<int>& nums) {
        int n = nums.size();
        vector<vector<int>> res;
        for (int s = 0; s < (1 << n); s++) {
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

#### [Subsets II](https://leetcode.com/problems/subsets-ii/)

差不多的題目，可是這邊注意元素重複的問題，我們可以把元素 hash 成 string，放進 set 裡看剛剛有沒有出現過，如果有出現我們就不要他。

```
class Solution {
public:
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> res;
        set<string> st;
        int n = nums.size();
        for (int subset = 0; subset < (1 << n); subset++) {
            vector<int> v;
            string s = "";
            for (int i = 0; i < n; i++) {
                if (subset & (1 << i)) {
                    v.push_back(nums[i]);
                    s = s + to_string(nums[i]) + ",";
                }
            }
            if (st.find(s) == st.end()){
                res.push_back(v);
                st.insert(s);
            }
        }
        
        return res;
    }
};
```

### Bitmask

#### [Maximum Product of Word Lengths](https://leetcode.com/problems/maximum-product-of-word-lengths/)

因為他要求兩個字串不能有相同的字母，我們可以考慮把字串 hash 成二進位，這樣就可以用 `&` 來看是不是等於 0，如果是的話代表他們沒有相同的字母，就可以跟答案取 max。

```cpp
class Solution {
public:
    int maxProduct(vector<string>& words) {    
        int n = words.size(), ans = 0;      
        vector<int> mask(n);              
        for (int i = 0; i < n; i++) {     
            for (auto& ch : words[i]) mask[i] |= 1 << (ch - 'a');
            for (int j = 0; j < i; j++) {
                if((mask[i] & mask[j]) == 0) {
                    ans = max(ans, int(size(words[i]) * size(words[j]))); 
                }
            }
        }    
        return ans;
    }
};
```
