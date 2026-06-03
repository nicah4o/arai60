問題： https://leetcode.com/problems/binary-tree-level-order-traversal/description/

## step1

問題は
　・binary treeが与えられ、同じ深さのnodeの値を配列にして、深さの浅い順から並べた配列の配列を返す
というもの。

step1での解き方は
　1.bfsで走査していく。キューにはTreeNode*と深さを入れ、それとは別に用意したmapで深さをkeyとして同じ深さのnodeの値をまとめた配列を作る。
　2.返り値の配列にmapから要素を移す。
というもの。

vector<vector<int>>を初期値を指定せずに構築すると、iが深さでjがnodeの値のときvector[i].push_back(j)として答えを直接書き込めないのが困った。
mapの操作に最悪logNがかかる。定数値では深さをKとしてlogKが正しい。

時間計算量：O(NlogN)
空間計算量：O(N)
```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        map<int, vector<int>> level_to_nums;
        vector<vector<int>> ans;
        if (!root) return ans;
        queue<pair<TreeNode*, int>> node_and_level;
        node_and_level.push({root, 0});
        while (!node_and_level.empty()) {
            auto [node, level] = node_and_level.front();
            node_and_level.pop();
            level_to_nums[level].push_back(node->val);
            if (node->left) {
                node_and_level.push({node->left, level + 1});
            }
            if (node->right) {
                node_and_level.push({node->right, level + 1});
            }
        }
        for (auto [key, vec] : level_to_nums) {
            ans.push_back(vec);
        }
        return ans;
    }
};
```

copilotにmapを使わないものを作ってもらった。
```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> ans;
        if (!root) return ans;
        queue<pair<TreeNode*, int>> q;
        q.push({root, 0});
        while (!q.empty()) {
            auto [node, level] = q.front();
            q.pop();
            if (level == ans.size()) {
                ans.push_back({});
            }
            ans[level].push_back(node->val);
            if (node->left)
                q.push({node->left, level + 1});
            if (node->right)
                q.push({node->right, level + 1});
        }
        return ans;
    }
};
```

当初のmap使用したコードより時間計算量はO(N)に減る。
深さは0から始まるが、このlevelが0の要素を格納するにはans.sizeが1以上である必要がある。
level == ans.size()となるとき、ansの配列数が足りていないので空の配列を挿入する。

## step2

コメント集をみた。
- https://github.com/hayashi-ay/leetcode/pull/32/changes
bfsのより洗練された書き方。レベルごとに配列を作る。親nodeの個数を予め記録しておき、その回数分のみqueueからpopさせて配列に入れる。

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> ans;
        if (!root) return ans;
        queue<TreeNode*> nodeq;
        nodeq.push(root);
        while (!nodeq.empty()) {
            int size = nodeq.size();
            vector<int> level;
            for (int i = 0; i < size; i++) {
                TreeNode* node = nodeq.front();
                nodeq.pop();
                level.push_back(node->val);
                if (node->left)
                    nodeq.push(node->left);
                if (node->right)
                    nodeq.push(node->right);
            }
            ans.push_back(level);
        }
        return ans;
    }
};
```

この場合は時間計算量O(N)空間計算量O(N)。

また、dfs再帰の方法も参考にさせてもらった。

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> ans;
        dfs(root, 0, ans);
        return ans;
    }
    void dfs(TreeNode* node, int level, vector<vector<int>>& ans) {
        if (!node)
            return;
        if (level == ans.size()) {
            ans.push_back({});
        }
        ans[level].push_back(node->val);
        dfs(node->left, level + 1, ans);
        dfs(node->right, level + 1, ans);
    }
};
```

再帰で返り値を持たないという方法が目新しかった。step3ではこのコードを練習した。
