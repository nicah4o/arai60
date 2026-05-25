問題：https://leetcode.com/problems/maximum-depth-of-binary-tree/description/

## step1

問題は
　・binary treeのrootを与えられ、最も遠いleafまでのノード数を返せ
というもの。

再帰で走査するコードを書いた。計算量は時間計算量がO(N)、空間計算量もO(N)。
　1.nodeが空なら0を返す。
　2.左右の子が存在しない(=leaf)なら1を返す。
　3.左右の子のどちらかしか存在しないなら、存在するほうの深さに1を足したものを返す
　4.左右どちらもいるならmax関数で左右の子の深さのうちより深い方に1を足して返す

```cpp
class Solution {
private:
    int find_depth(TreeNode* node) {
        if (!node)
            return 0;
        if (!node->left && !node->right) {
            return 1;
        } else if (!node->left) {
            return find_depth(node->right) + 1;
        } else if (!node->right) {
            return find_depth(node->left) + 1;
        } else {
            return max(find_depth(node->left), find_depth(node->right)) + 1;
        }
    }

public:
    int maxDepth(TreeNode* root) { return find_depth(root); }
};
```

## stpe2

コメント集を読んだ。

https://github.com/goto-untrapped/Arai60/pull/45/changes/BASE..d0e1345617ff80f24a63afb97245c427aede9179
再帰のより洗練された記法。depthをグローバル変数で定義せずにrootからleafへ関数内で受け渡してもう一度rootへ戻ってこさせる。

```cpp
class Solution {
private:
    int find_depth(TreeNode* node, int depth) {
        if (!node)
            return depth;
        int left_depth = find_depth(node->left, depth);
        int right_depth = find_depth(node->right, depth);
        return max(left_depth, right_depth) + 1;
    }

public:
    int maxDepth(TreeNode* root) { return find_depth(root, 0); }
};
```

https://github.com/sakupan102/arai60-practice/pull/21/changes
BFSを用いた方法など。stack-loopのものは見れていない。スタックを用いることで再帰関数がスタック領域に積みあがる過程と等価なコードを実現しているようだ。

BFS

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root)
            return 0;
        queue<pair<TreeNode*, int>> node_depth;
        node_depth.push({root, 1});
        int max_depth = 0;
        while (!node_depth.empty()) {
            TreeNode* node = node_depth.front().first;
            int depth = node_depth.front().second;
            max_depth = max(max_depth, depth);
            node_depth.pop();
            if (node->left) {
                node_depth.push({node->left, depth + 1});
            }
            if (node->right) {
                node_depth.push({node->right, depth + 1});
            }
        }
        return max_depth;
    }
};
```
