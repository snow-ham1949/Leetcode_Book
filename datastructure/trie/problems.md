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

#### [Design Add and Search Words Data Structure](https://leetcode.com/problems/design-add-and-search-words-data-structure/)

這題只要加上特殊的 search 就好，那我們可以在 Trie 上 dfs，遇到 . 的時候就當作所有字元都可以下去找就好了。

```cpp
class WordDictionary {
public:
    struct Node {
        bool end;
        Node *child[26];
        Node() {
            end = false;
            for (int i = 0; i < 26; i++) child[i] = NULL;
        }
    };
    Node *root;

    WordDictionary() {
        root = new Node();    
    }
    
    void addWord(string word) {
        Node *node = root;
        for (char c : word) {
            int ind = c - 'a';
            if (node->child[ind] == NULL) node->child[ind] = new Node();
            node = node->child[ind];
        }
        node->end = true;
    }

    bool dfs(Node* node, string word, int ind) {
        if (ind == word.size()) return node->end;
        int idx = word[ind] - 'a';
        if (word[ind] == '.') { 
            for (int i = 0; i < 26; i++) {
                if (node->child[i]) {
                    bool flag = dfs(node->child[i], word, ind + 1);
                    if (flag) return true;
                }
            }
            return false;
        } else {
            if (!node->child[idx]) return false;
            else return dfs(node->child[idx], word, ind + 1);
        }
    }
    
    bool search(string word) {
        return dfs(root, word, 0);
    }
};

/**
 * Your WordDictionary object will be instantiated and called as such:
 * WordDictionary* obj = new WordDictionary();
 * obj->addWord(word);
 * bool param_2 = obj->search(word);
 */
```

#### [Maximum XOR of Two Numbers in an Array](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/)

需要對 Trie 類進行一些修改才能夠處理整數的二進制。

首先，整數的二進制表示最多包含 32 位（對於 32 位整數）。因此，Trie 中的每個節點應該只有兩個子節點，對應於 0 和 1。此外，我們不需要 `end` 標誌，因為我們將處理完整的 32 位整數。

1. 對於數組中的每個數，將其二進制表示插入 Trie。
2. 對於數組中的每個數，使用 Trie 查找與其異或值最大的數。

那為什麼 Trie 可以很好的處理這個問題呢？當我們想要找出與給定整數異或值最大的另一個整數時，我們可以從 Trie 的根節點開始，對給定數字的每一位進行比較。對於給定數字的每一位，如果當前 Trie 節點有一個相反位的子節點（即如果當前位是 0，則選擇子節點 1；如果當前位是 1，則選擇子節點 0），則選擇該子節點繼續向下搜索。這樣做可以確保每一步都嘗試最大化異或結果的值。

```cpp
class Solution {
public:
    class Trie {
    public:
        struct Node {
            Node* child[2];
            Node() {
                child[0] = child[1] = nullptr;
            }
        };

        Node* root;

        Trie() {
            root = new Node();
        }

        void insert(int num) {
            Node* node = root;
            for (int i = 31; i >= 0; i--) {
                int bit = (num >> i) & 1;
                if (!node->child[bit]) {
                    node->child[bit] = new Node();
                }
                node = node->child[bit];
            }
        }

        int maxXor(int num) {
            Node* node = root;
            int maxNum = 0;
            for (int i = 31; i >= 0; i--) {
                int bit = (num >> i) & 1;
                if (node->child[!bit]) {
                    maxNum |= (1 << i);
                    node = node->child[!bit];
                } else {
                    node = node->child[bit];
                }
            }
            return maxNum;
        }
    };

    int findMaximumXOR(vector<int>& nums) {
        Trie trie;
        for (int num : nums) {
            trie.insert(num);
        }

        int maxXor = 0;
        for (int num : nums) {
            maxXor = max(maxXor, trie.maxXor(num));
        }
        return maxXor;
    }

};
```

#### [Prefix and Suffix Search](https://leetcode.com/problems/prefix-and-suffix-search/)

這題要稍微改一下模板，因為題目要求的是回傳符合字串的 index ，所以 Trie node 必須記錄是哪個 index。

首先在 constructor 的時候，可以先對所有字串生成前綴後綴組合，例如 "abc" 就生成 "abc{", "bc{", "c{" 以及 "{abc", "{bc", "{c"，然後就會在 Trie 插入以下字串

* `"abc{abc"`
* `"bc{abc"`
* `"c{abc"`
* `"a{abc"`
* `"ab{abc"`

當我們想要找到以 "a" 為前綴和 "c" 為後綴的單詞時，我們實際上在 Trie 中查找字符串 "c{a"。因為我們已經將所有可能的前綴和後綴組合插入到 Trie 中了，所以 "c{a" 會導向 "abc" 這個單詞對應的路徑。

```cpp
class WordFilter {
public:
    class TrieNode {
    public:
        vector<TrieNode*> children;
        int ind;
        
        TrieNode() : ind(0), children(27, nullptr) {}
    };
    TrieNode* trie;
    WordFilter(vector<string>& words) {
        trie = new TrieNode();
        for (int ind = 0; ind < (int)words.size(); ++ind) {
            string word = words[ind] + "{";
            for (int i = 0; i < word.length(); ++i) {
                TrieNode* cur = trie;
                cur->ind = ind;
                for (int j = i; j < 2 * (int)word.length() - 1; ++j) {
                    int k = word[j % (int)word.length()] - 'a';
                    if (cur->children[k] == nullptr) {
                        cur->children[k] = new TrieNode();
                    }
                    cur = cur->children[k];
                    cur->ind = ind;
                }
            }
        }
    }

    int f(string prefix, string suffix) {
        TrieNode* cur = trie;
        string searchStr = suffix + "{" + prefix;
        for (char c : searchStr) {
            if (cur->children[c - 'a'] == nullptr) {
                return -1;
            }
            cur = cur->children[c - 'a'];
        }
        return cur->ind;
    }
};
```
