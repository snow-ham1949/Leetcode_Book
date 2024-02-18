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

# Tree DP

#### [House Robber III](https://leetcode.com/problems/house-robber-iii)

1. **最優子結構**：每個節點選取或不選取，都會影響其子樹的最優解。
2. **重疊子問題**：同一個節點在不同的選取狀態下，其子樹的最優解會被重複計算多次。

對於每個節點，我們考慮兩種情況：選取這個節點和不選取這個節點。

* 如果選取當前節點，則不能選取其直接子節點，但可以選取子節點的子節點。
* 如果不選取當前節點，則可以選取其直接子節點。

因此，對於每個節點，我們需要記錄兩個值：

1. 選取當前節點時的最大值。
2. 不選取當前節點時的最大值。

這樣，我們可以用一個遞迴函數，返回一個包含這兩個值的數組，來實現這個DP。

```cpp
class Solution {
public:
    vector<int> robSub(TreeNode* root) {
        if (!root) return {0, 0};
        
        vector<int> left = robSub(root->left);
        vector<int> right = robSub(root->right);
        int val1 = root->val + left[1] + right[1];
        int val2 = max(left[0], left[1]) + max(right[0], right[1]);
        
        return {val1, val2};
    }

    int rob(TreeNode* root) {
        vector<int> result = robSub(root);
        return max(result[0], result[1]);
    }
};
```

#### [Binary Tree Cameras](https://leetcode.com/problems/binary-tree-cameras)

1. **最優子結構**：一個節點是否安裝攝像頭，取決於其子節點的狀態，且子節點的最優解可以獨立於父節點之外求解。
2. **重疊子問題**：對於每個節點，我們需要根據其子節點是否被監控來決定當前節點的狀態，這一過程在每個節點處都會重複進行。\


* 使用遞歸函數`installCamera`來遍歷二元樹，並為每個節點定義三種狀態：
  * `1`：節點已被覆蓋但不安裝攝像頭（由其他節點的攝像頭覆蓋）。
  * `0`：節點安裝了攝像頭。
  * `-1`：節點未被覆蓋，需要攝像頭。
* 當遞歸訪問到空節點時，返回`1`表示該節點已被覆蓋（虛擬覆蓋，實際上是因為沒有該節點）。
* 當一個節點的左子節點或右子節點返回`-1`時，表示該子節點需要攝像頭，因此當前節點安裝攝像頭，並返回`0`。
* 如果任一子節點安裝了攝像頭（即返回`0`），則當前節點被覆蓋，返回`1`。
* 如果左右子節點都被覆蓋但沒有攝像頭，則當前節點需要攝像頭，返回`-1`。
* 最後，檢查根節點的狀態。如果根節點需要攝像頭（返回`-1`），則總攝像頭數量加一。

```cpp
class Solution {
private:
    int cameraCount;
    int installCamera(TreeNode* node) {
        if (!node) return 1;

        int leftStatus = installCamera(node->left);
        int rightStatus = installCamera(node->right);

        if (leftStatus == -1 || rightStatus == -1) {
            cameraCount++;
            return 0;
        }

        if (leftStatus == 0 || rightStatus == 0) {
            return 1;
        }
        return -1;
    }

public:
    int minCameraCover(TreeNode* root) {
        cameraCount = 0;
        if (installCamera(root) == -1) {
            cameraCount++;
        }
        return cameraCount;
    }
};
```

#### [Distribute Coins in Binary Tree](https://leetcode.com/problems/distribute-coins-in-binary-tree)

* 對於每個節點，計算其子樹中硬幣的過剩或不足量，這由當前節點的硬幣數加上左右子節點的過剩或不足量，再減去1（每個節點需要保留一枚硬幣）得到。
* 然後，計算從當前節點到其父節點或從父節點到當前節點移動硬幣的次數。這等於當前節點和其子樹中的硬幣過剩或不足量的絕對值。
* 通過遞歸，我們從葉子節點開始向上計算，直到達到根節點。這樣確保了在滿足每個節點最終都有一枚硬幣的條件下，移動的總次數最小。

```cpp
class Solution {
public:
    int distributeCoins(TreeNode* root) {
        int moves = 0;
        calculateMoves(root, moves);
        return moves;
    }

private:
    int calculateMoves(TreeNode* node, int& moves) {
        if (!node) return 0;
        int leftExcess = calculateMoves(node->left, moves);
        int rightExcess = calculateMoves(node->right, moves);
        int excess = node->val + leftExcess + rightExcess - 1;
        moves += abs(excess);
        return excess;
    }
};
```

#### [Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum)

用 DFS 來遍歷二元樹，同時計算兩個重要的值：

* **局部最大路徑和**（對於當前節點，不考慮回溯到父節點的情況下，從當前節點出發的最大路徑和）。
* **全局最大路徑和**（遍歷過程中遇到的所有節點路徑和的最大值）。

**解法詳解**

1. 對於每個節點`p`，計算從其左子節點`left`和右子節點`right`出發的最大路徑和，如果這些值為負，則不考慮這些路徑，即通過`max(dfs(p->left), 0)`和`max(dfs(p->right), 0)`來確保只考慮正貢獻值。
2. 然後，對於節點`p`，其最大貢獻值`res`被更新為`max(res, p->val + l + r)`，這表示節點`p`作為轉折點（即路徑的最高點）時的最大路徑和，包括從`p`到其左右子節點的路徑。
3. `dfs`函數返回當前節點`p`能提供給其父節點的最大路徑和，即`p->val + max(l, r)`，這意味著如果要經過節點`p`向上回溯，應該選擇左子節點和右子節點中較大的一個路徑和來進行。
4. 最終，`maxPathSum`函數啟動DFS遍歷並返回全局最大路徑和`res`。

```cpp
class Solution {
public:
    int res = INT_MIN;
    int dfs(TreeNode* p) {
      if (!p) return 0;
      int l = max(dfs(p->left), 0), r = max(dfs(p->right), 0);
      res = max({res, p->val + l + r});
      return p->val + max(l, r);
    }
    int maxPathSum(TreeNode* root) {
      dfs(root);
      return res;
    }
};
```

#### [Maximum Sum BST in Binary Tree](https://leetcode.com/problems/maximum-sum-bst-in-binary-tree)

這個問題可以通過後序遍歷（左子樹 -> 右子樹 -> 節點自身）來解決，因為我們需要先處理子節點，然後根據子節點的訊息判斷當前節點所形成的樹是否是BST。在遍歷的過程中，對於每個節點，我們需要知道四個訊息：

1. 當前子樹是否是BST。
2. 當前子樹中所有節點的值之和。
3. 當前子樹中的最小值。
4. 當前子樹中的最大值。

這樣，當我們遍歷到一個節點時，我們可以根據其左右子樹的信息來判斷以當前節點為根的子樹是否為BST，並更新全局最大BST子樹的值之和。

```cpp
class Solution {
public:
    struct Info {
        bool isBST;
        int sum, minVal, maxVal;
        Info(bool isBST, int sum, int minVal, int maxVal) : isBST(isBST), sum(sum), minVal(minVal), maxVal(maxVal) {}
    };
    
    int maxSumBST(TreeNode* root) {
        int maxSum = 0;
        dfs(root, maxSum);
        return maxSum;
    }
    
    Info dfs(TreeNode* node, int& maxSum) {
        if (!node) return {true, 0, INT_MAX, INT_MIN};
        
        auto left = dfs(node->left, maxSum);
        auto right = dfs(node->right, maxSum);
        
        if (left.isBST && right.isBST && node->val > left.maxVal && node->val < right.minVal) {
            int sum = node->val + left.sum + right.sum;
            maxSum = max(maxSum, sum);
            return {true, sum, min(node->val, left.minVal), max(node->val, right.maxVal)};
        }
        
        return {false, 0, 0, 0};
    }
};
```
