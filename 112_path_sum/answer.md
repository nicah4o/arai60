問題： https://leetcode.com/problems/path-sum/description/

## step1

問題は
　1.あるTreeNodeとtargetSumという整数が渡される。
　2.TreeNodeのrootからleafへの経路のvalの和でtargetSumになるものがあればtrue、なければfalseを返せ。
というもの。

targetSumからroot->valを引いたものをroot->leftとroot->rightと共に引数にして再帰で関数を回す方法で解いた。

時間計算量：O(N)
空間計算量：O(N)

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root) {
            return false;
        }
        targetSum -= root->val;
        if (targetSum == 0 && !root->left && !root->right) {
            return true;
        }
        bool left_value = hasPathSum(root->left, targetSum);
        bool right_value = hasPathSum(root->right, targetSum);
        if (left_value || right_value) {
            return true;
        } else {
            return false;
        }
    }
};
```

書き方が場当たり的になってしまった(trueを返す時の条件など)。copilotに短縮してもらった。

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root) return false;

        targetSum -= root->val;

        if (!root->left && !root->right)
            return targetSum == 0;

        return hasPathSum(root->left, targetSum) ||
               hasPathSum(root->right, targetSum);
    }
};
```

## step2

- https://github.com/attractal/leetcode/pull/37/changes
bfsなどの方法

自分でもstack bfsをやってみた。

時間計算量：O(N)
空間計算量：O(N)

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root)
            return false;
        stack<pair<TreeNode*, int>> nodes_and_sums;
        nodes_and_sums.push({root, 0});
        bool result = false;
        while (!nodes_and_sums.empty()) {
            auto [node, sum] = nodes_and_sums.top();
            nodes_and_sums.pop();
            sum += node->val;
            if (!node->left && !node->right && !result) {
                result = (sum == targetSum);
            }
            if (node->left) {
                nodes_and_sums.push({node->left, sum});
            }
            if (node->right) {
                nodes_and_sums.push({node->right, sum});
            }
        }
        return result;
    }
};
```

resultにboolで代入せずともtrue条件にsum == targetSumをいれればよかった。

## step3

stackを練習した。

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (!root)
            return false;
        stack<pair<TreeNode*, int>> nodes_and_sums;
        nodes_and_sums.push({root, root->val});
        while (!nodes_and_sums.empty()) {
            auto [node, sum] = nodes_and_sums.top();
            nodes_and_sums.pop();
            if (!node->left && !node->right) {
                if (sum == targetSum)
                    return true;
            }
            if (node->left) {
                nodes_and_sums.push({node->left, sum + node->left->val});
            }
            if (node->right) {
                nodes_and_sums.push({node->right, sum + node->right->val});
            }
        }
        return false;
    }
};
```
