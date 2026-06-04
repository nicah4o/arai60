問題： https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/description/

## step1

問題は
　1.binary treeが与えられ、同じ深さのnodeの値で配列を作るのだが、その配列に格納する順番を深さのレベルごとに左からと右からで変更する。
　2.rootでは左から始まり、その次の深さでは右からnodeの値を格納、その次の深さでは左からという風に左右のどちらを始めにして格納するかを交互に変える。
というもの。

前回の102.binary tree level order traversalのbfsにreverse関数を加えることで解いた。
reverseの判定のためにis_leftというフラグを使用した。

時間計算量：O(N)
空間計算量：O(N)
```cpp
class Solution {
public:
    vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
        vector<vector<int>> values_by_level;
        if (!root) return values_by_level;
        queue<TreeNode*> nodeq;
        nodeq.push(root);
        bool is_left = true;
        while (!nodeq.empty()) {
            int level = nodeq.size();
            vector<int> values;
            for (int i = 0; i < level; i++) {
                TreeNode* node = nodeq.front();
                nodeq.pop();
                values.push_back(node->val);
                if(node->left){
                    nodeq.push(node->left);
                }
                if(node->right){
                    nodeq.push(node->right);
                }
            }
            if(!is_left){
                reverse(values.begin(),values.end());
            }
            is_left = !is_left;
            values_by_level.push_back(values);
        }
        return values_by_level;
    }
};
```

## step2

他の方のコードをみた。
- https://github.com/dorxyxki/arai60/pull/27/changes
stack使用したbfsとdeque使用した積み方のbfs

stack&bfs
```cpp
class Solution {
public:
    vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
        vector<vector<int>> values_by_level;
        bool is_reversed = false;
        stack<TreeNode*> nodest;
        nodest.push(root);
        while (!nodest.empty()) {
            stack<TreeNode*> nextst;
            vector<int> values;
            while (!nodest.empty()) {
                TreeNode* node = nodest.top();
                nodest.pop();
                if (!node) continue;
                values.push_back(node->val);
                if (is_reversed) {
                    nextst.push(node->right);
                    nextst.push(node->left);
                } else {
                    nextst.push(node->left);
                    nextst.push(node->right);
                }
            }
            if (!values.empty())
                values_by_level.push_back(values);
            nodest = nextst;
            is_reversed = !is_reversed;
        }
        return values_by_level;
    }
};
```

・stackなので次のレベルをstackに積むときに左から積むと次のレベルは右から取り出される。つまり現在の左右順を参照して次のレベルを積む動作が見やすい。
・queueのようにsizeを参照してfor文を回すことはできない。例えばroot->rightとroot->leftは同じレベルに存在するが、2回の処理と決めるとroot->rightの左右の子をroot->leftより先に処理してしまう。

deque
```cpp
class Solution {
public:
    vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
        vector<vector<int>> values_by_level;
        if (!root)
            return values_by_level;
        queue<TreeNode*> nodeq;
        nodeq.push(root);
        bool left_to_right = true;
        while (!nodeq.empty()) {
            int size = nodeq.size();
            deque<int> level;
            for (int i = 0; i < size; i++) {
                TreeNode* node = nodeq.front();
                nodeq.pop();
                if (left_to_right) {
                    level.push_back(node->val);
                } else {
                    level.push_front(node->val);
                }
                if (node->left) {
                    nodeq.push(node->left);
                }
                if (node->right) {
                    nodeq.push(node->right);
                }
            }
            values_by_level.emplace_back(level.begin(), level.end());
            left_to_right = !left_to_right;
        }
        return values_by_level;
    }
};
```

・dequeではpush_frontもpush_backもO(1)で使用できる。

dfs再帰
```cpp
class Solution {
public:
    vector<deque<int>> values_deque;
    void dfs(TreeNode* node, int level) {
        if (!node) return;
        if (values_deque.size() == level) {
            values_deque.push_back(deque<int>());
        }
        if (level % 2 == 0) {
            values_deque[level].push_back(node->val);
        } else {
            values_deque[level].push_front(node->val);
        }
        dfs(node->left, level + 1);
        dfs(node->right, level + 1);
    }
    vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
        dfs(root, 0);
        vector<vector<int>> values_by_level;
        for (auto& dq : values_deque) {
            values_by_level.emplace_back(dq.begin(), dq.end());
        }
        return values_by_level;
    }
};
```

メンバ変数を再帰で変更する方法。最後にvectorに変換する必要がある。step3ではこれらを一度ずつ練習した。
