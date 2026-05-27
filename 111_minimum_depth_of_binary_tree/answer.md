問題：https://leetcode.com/problems/minimum-depth-of-binary-tree/description/

## step1

問題は
　・binary treeのrootを与えられ、leafに至る経路の中で最も短いものを返せ
というもの。

昨日の問題はこの逆で経路の中で最長のものを返す問題だった。
昨日はmax()を使ったのでmin()に置き換えればよいと思ったが、そう単純ではなかった。
発生する問題として、
　・nullのnodeを深さ0と数えると、nullでない側に子を持つ親であっても深さ1と数えてしまう(min()関数により)。
というものがあった。

結局分岐を大量に作ることで解決した。
1.nullの場合　2.leafの場合　3.左がない場合　4.右がない場合　5.左右両方ともある場合
時間計算量は全てのnodeを訪問するのでO(N)、空間計算量もスタック領域に積むのでO(N)。

```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        if (!root) {
            return 0;
        }
        if (!root->left && !root->right) {
            return 1;
        }
        if (!root->left) {
            return minDepth(root->right) + 1;
        }
        if (!root->right) {
            return minDepth(root->left) + 1;
        }
        return min(minDepth(root->left), minDepth(root->right)) + 1;
    }
};
```

## stpe2

コメント集を見た。
maxDepthとminDepthは早期終了できない/できるの違いがあり、それが下から集める/上から集めるの違いになっている。
BFSは上から集めるアルゴリズムにあたる。勿論全ての場合でmaxDepthとminDepthを解くことができる。

BFS
時間計算量；O(N)
空間計算量：O(N)
```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        if (!root)
            return 0;
        queue<pair<TreeNode*, int>> node_depth;
        node_depth.push({root, 1});
        while (!node_depth.empty()) {
            auto [node, depth] = node_depth.front();
            node_depth.pop();
            if (!node->left && !node->right) {
                return depth;
            }
            if (node->left) {
                node_depth.push({node->left, depth + 1});
            }
            if (node->right) {
                node_depth.push({node->right, depth + 1});
            }
        }
        return 0;
    }
};
```

https://github.com/attractal/leetcode/pull/31/changes
トップダウン/ボトムアップ × 再帰/ループの四通りある。

ボトムアップ&再帰をstep3でやった。
```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        if (!root) return 0;

        int l = minDepth(root->left);
        int r = minDepth(root->right);

        if (!l || !r) return l + r + 1;
        return min(l, r) + 1;
    }
};
```

最初に発生した問題(max()をmin()に置き換えるだけだとnullが最短経路として数えられる)は
```cpp
if (!l || !r) return l + r + 1;
```
と書くことで解決する。片方のみ子がある場合は和を返すというのは思いつかなかった。
