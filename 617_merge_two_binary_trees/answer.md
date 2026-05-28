問題： https://leetcode.com/problems/merge-two-binary-trees/

## step1

問題は
　1.二つのbinary treeを与えられ、対応する同じ場所のvalueを合計した新しい木を返せ
　2.ただし、片方が値を持たない場合は0として扱う
というもの。

解き方は
　1.両方の木でnodeが存在する場合、そのnodeの左右の子を再帰で新しいnodeの左右に代入する。そのnodeの値はval同士の和。
　2.片方の木にしかnodeが存在しない場合、存在する方のnodeを新しいnodeとして挿げ替える(そのまま返す)。
というDFS。

時間計算量：O(N)
空間計算量：O(N)

```cpp
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
        if (!root1 && !root2)
            return nullptr;
        TreeNode* newtree = new (TreeNode);
        if (root1 && root2) {
            newtree->left = mergeTrees(root1->left, root2->left);
            newtree->right = mergeTrees(root1->right, root2->right);
        }
        if (!root1) {
            return root2;
        }
        if (!root2) {
            return root1;
        }
        int val1 = root1 ? root1->val : 0;
        int val2 = root2 ? root2->val : 0;
        newtree->val = val1 + val2;
        return newtree;
    }
};
```

copilotによる短縮版

```cpp
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
        if (!root1) return root2;
        if (!root2) return root1;
        
        TreeNode* node = new TreeNode(root1->val + root2->val);
        node->left = mergeTrees(root1->left, root2->left);
        node->right = mergeTrees(root1->right, root2->right);
        
        return node;
    }
};
```

nullptrが入ったときの早期リターンと代入のための変数宣言はいらなかった。

## step2

コメント集みた。
- https://github.com/irohafternoon/LeetCode/pull/26/changes
DFS,BFS,番兵の方法など

BFSのこの方法を写させてもらった。
番兵を使うことでnullptrのleftなどへのアクセスを可能にしているほか、TreeNodeのポインタのポインタを使用することで参照渡しを可能にしている。
(キューだとFIFOなので上から新しい木を作ることになるが、単純にTreeNode *nodeをnode->val=と代入するだけでは毎回の代入はwhileの一回分のスタック領域で変更されるだけで親に反映されない。)

```cpp
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
        queue<tuple<const TreeNode*, const TreeNode*, TreeNode**>>
            nodes_to_merge;
        TreeNode* new_head = new TreeNode();
        nodes_to_merge.emplace(root1, root2, &new_head);
        while (!nodes_to_merge.empty()) {
            auto [node1, node2, ptr_to_merged_node] = nodes_to_merge.front();
            nodes_to_merge.pop();
            if (!node1 && !node2) {
                *ptr_to_merged_node = nullptr;
                continue;
            }
            if (!node1) {
                node1 = kSentinel;
            }
            if (!node2) {
                node2 = kSentinel;
            }
            (*ptr_to_merged_node)->val = node1->val + node2->val;
            (*ptr_to_merged_node)->left = new TreeNode();
            nodes_to_merge.emplace(node1->left, node2->left,
                                   &((*ptr_to_merged_node)->left));
            (*ptr_to_merged_node)->right = new TreeNode();
            nodes_to_merge.emplace(node1->right, node2->right,
                                   &((*ptr_to_merged_node)->right));
        }
        return new_head;
    }

private:
    static const TreeNode* const kSentinel;
};
const TreeNode* const Solution::kSentinel = new TreeNode(0);
```

step3ではこのコードを練習した。正直二重参照よく分かっていないのでまた考えたい。
