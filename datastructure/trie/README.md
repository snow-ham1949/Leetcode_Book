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

# Trie

Trie（又稱前綴樹或字典樹）是一種樹形資料結構，用於高效地儲存和搜尋字符串集合中的key。每個節點代表一個字元，從根到某一節點的路徑會形成一個字串。

插入、刪除、尋找等等都跟字串的長度成正比。

以下是模板。

```cpp
class Trie {
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
    Trie() {
        root = new Node();
    }
    
    void insert(string word) {
        Node *node = root;
        for (char c : word) {
            int ind = c - 'a';
            if (node->child[ind] == NULL) node->child[ind] = new Node();
            node = node->child[ind];
        }
        node->end = true;
    }
    
    bool search(string word) {
        Node *node = root;
        for (char c : word) {
            int ind = c - 'a';
            if (node->child[ind] == NULL) return false;
            node = node->child[ind];
        }
        return node->end;
    }
    
    bool startsWith(string prefix) {
        Node *node = root;
        for (char c : prefix) {
            int ind = c - 'a';
            if (node->child[ind] == NULL) return false;
            node = node->child[ind];
        }
        return true;
    }
};
```
