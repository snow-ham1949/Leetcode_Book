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

# Linked List

### Singly Linked List

由一堆節點組成，每一個節點都包含資訊以及下一個節點的指標。Leetcode 用的 Linked List 多數是這個。

插入節點 $$O(1)$$

刪除節點 $$O(1)$$

### Doubly Linked List

不一樣的是每一個節點包含資訊、下一個節點的指標、上一個節點的指標。可以用 pointer 或者 xor 實作。

xor 運算具有幾個關鍵特性非常有用：

1. **逆運算性質**：對於任何數字`a`和`b`， $$a$$ xor $$b$$ xor $$b$$ 的結果總是`a`。這代表著如果我們有一個節點的前後節點地址的XOR值，我們就可以還原出其中一個節點的地址。
2. **相同數字XOR的結果為零**：任何數字與自身進行XOR運算的結果都是零。這在 xor linked list 中用於標識 linked list 的末端，因為最後一個節點的 next pointer 是 null，即`0`。

#### pointer 版本

```cpp
struct Node {
    int data;
    Node *prev, *next;
    Node(int data) : data(data), prev(nullptr), next(nullptr) {}
};
struct DoublyLinkedList {
    Node *head, *tail;
    DoublyLinkedList() : head(nullptr), tail(nullptr) {}
}
```

#### xor 版本

```cpp
struct XORNode {
    int data;
    XORNode* npx;
    XORNode(int data) : data(data), npx(nullptr) {}
};
XORNode* XOR(XORNode* a, XORNode* b) {
    return reinterpret_cast<XORNode*>(
        reinterpret_cast<uintptr_t>(a) ^ reinterpret_cast<uintptr_t>(b));
}
struct XORLinkedList {
    XORNode *head;
    XORLinkedList() : head(nullptr) {}
    void add(int data) {
        XORNode* newNode = new XORNode(data);
        newNode->npx = XOR(head, nullptr);
        if (head != nullptr) {
            XORNode* next = XOR(head->npx, nullptr);
            head->npx = XOR(newNode, next);
        }
        head = newNode;
    }
}
```

插入節點 $$O(1)$$

刪除節點 $$O(1)$$

### 快慢指標

在 Linked List 題目中，快慢指標是一種常見的技巧。這個方法使用兩個指標以不同的速度遍歷鏈表：一個是快指標（每次移動兩個節點），另一個是慢指標（每次移動一個節點）。常用於解決以下幾種類型的 Linked List 問題：

1. **檢測環**：在檢測 Linked List 是否有環的問題中，快指標最終會追上慢指標，如果確實存在環。如果快指標到達 Linked List 尾部（即 `null`），則表明 Linked List中無環。
2. **尋找中點**：使用快慢指標來尋找 Linked List 的中點。當快指標到達鏈表末尾時，慢指標則位於鏈表的中間位置。這在很多問題中都是一個非常有用的技巧。
3. **尋找倒數第 k 個節點**：可以設定兩個指標，其中一個指標先向前移動 $$k$$ 步，然後兩個指標同時前進，當先行的指標到達鏈表末尾時，另一個指標所在的位置就是倒數第 $$k$$ 個節點。

這種方法的優點在於它能夠在不需要額外儲存空間的情況下，有效地對 Linked List 進行遍歷和檢測。
