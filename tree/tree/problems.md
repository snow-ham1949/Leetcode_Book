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

### Preorder/Inorder

#### [Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/)

這題主要只是熟悉 inorder 的順序，知道順序就可以做了。

```cpp
class Solution {
public:
    vector<int> ans;
    
    void inorder(TreeNode* root) {
        if (root == NULL) {
            return;
        }
        
        inorder(root->left);
        ans.push_back(root->val);
        inorder(root->right);
    }
    
    vector<int> inorderTraversal(TreeNode* root) {
        inorder(root);
        return ans;
    }
};
```

#### [Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

#### 因為 preorder 是從根節點開始，所以每一次我們都可以從 preorder 當前沒看過最前面的元素來切開 inorder，然後去分別遞迴左邊和右邊。

```cpp
class Solution {
public:
    TreeNode* build(vector<int>& preorder, vector<int>& inorder, int l, int r, int& ind) {
        if (l > r) return nullptr;
        TreeNode* root = new TreeNode(preorder[ind++]);
        int pos = distance(inorder.begin(), find(inorder.begin(), inorder.end(), root->val));
        root->left = build(preorder, inorder, l, pos - 1, ind);
        root->right = build(preorder, inorder, pos + 1, r, ind);
        return root;
    }
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        int n = preorder.size(), ind = 0;
        return build(preorder, inorder, 0, n - 1, ind);
    }
};
```

### BST

#### [Range Sum of BST](https://leetcode.com/problems/range-sum-of-bst/)

根據 BST 的定義，左子樹一定小於等於根節點，右子樹一定大於等於根節點，所以我們可以直接從每個子樹的根節點去判斷哪邊的子樹是包含的，用遞迴的方式把他們加起來就可以了。

```cpp
class Solution {
public:
    int rangeSumBST(TreeNode* root, int low, int high) {
        if (!root) return 0;
        if (root->val < low) return rangeSumBST(root->right, low, high);
        if (root->val > high) return rangeSumBST(root->left, low, high);
        return root->val + rangeSumBST(root->left, low, high) + rangeSumBST(root->right, low, high);
    }
};
```

#### [Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/)

一樣利用 BST 的定義，我們可以先假設從最左鏈的葉子節點（最小的）一步一步往上找，但要注意的是，當前節點的右子樹比父親還要小，所以要先遍歷。為了模擬 Last In First Out 的過程，我們就可以用 stack。

```cpp
class Solution {
public:
    int kthSmallest(TreeNode* root, int k) {
        stack<TreeNode*> stk;
        TreeNode* node = root;
        int cnt = 0;
        while (node || !stk.empty()) {
            while (node) {
                stk.push(node);
                node = node->left;
            }
            node = stk.top();
            stk.pop();
            ++cnt;
            if (cnt == k) return node->val;
            node = node->right;
        }
        return -1;
    }
};
```

### 其他

#### [Populating Next Right Pointers in Each Node](https://leetcode.com/problems/populating-next-right-pointers-in-each-node/)

一樣用遞迴做，每次就處理好當前的節點就好，然後遞迴去做左子樹和右子樹。當前節點就把左邊牽到右邊，然後如果他有隔壁的話，那麼他右子節點的隔壁就應該要是當前節點的 next 的左邊子節點。

```cpp
class Solution {
public:
    void ConnectNext(Node* root) {
        if (root && root->left && root->right) {
            root->left->next = root->right;
            if (root->next) {
                root->right->next = root->next->left;
            }
            ConnectNext(root->left);
            ConnectNext(root->right);
        }
    }
    Node* connect(Node* root) {
        if (!root) return NULL;
        ConnectNext(root);
        return root;
    }
};

```
