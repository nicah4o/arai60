問題： https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/description/

## step1

問題は
　1.前順で走査された二分木の値と中順で走査された二分木の値が配列で与えられる
　2.それらから元の二分木を作って返す
というもの。

ちなみに
　・前順はrootから左>右の優先順位でDFSをしていく方法
　・中順はまずrootからleafになるまで左に行った場所からスタート。右があるなら右→ないなら上に上がる、を繰り返す。走査はleafになるまで左に行く方法。

考え
　・前順はrootの位置がわかるが（先頭）、中順は分からない。
　・どのような順番で構築するのか想像もつかない

何も思いつかなかったので答えをみた。

## step2

- https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/solutions/6744135/video-2-solutions-with-on2-and-on-time-b-xdpe/
leetcodeのsoulutionの一つ

解き方は
　1.preorderで最初にあたるrootがinorderでどの位置にあるかを見る。rootのTreeNodeをつくる。
　2.その位置でinorderを二分割する。rootを削除したpreorderと分割したinorderを関数に入れ、root->leftとroot->rightにする。
　3.分割されたinorderが0個になるとき、処理は行わずnullptrを返す（再帰の底はleafの子node）。

```cpp
class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        deque<int> preorderdq(preorder.begin(), preorder.end());
        return build(preorderdq, inorder);
    }
private:
    TreeNode* build(deque<int>& preorder, vector<int>& inorder) {
        if (!inorder.empty()) {
            int val = preorder.front();
            preorder.pop_front();
            auto it = find(inorder.begin(), inorder.end(), val);
            int idx = it - inorder.begin();
            TreeNode* root = new TreeNode(val);
            vector<int> leftInorder(inorder.begin(), inorder.begin() + idx);
            vector<int> rightInorder(inorder.begin() + idx + 1, inorder.end());
            root->left = build(preorder, leftInorder);
            root->right = build(preorder, rightInorder);
            return root;
        }
        return nullptr;
    }
};
```

コメント集見た。
- https://github.com/kazukiii/leetcode/pull/30
preorderとinorderでの頭からの実装
写させてもらった。

preorderの頭からの実装
```cpp
class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        map<int,int> inorder_position;
        for (int i = 0;i < inorder.size();i++) {
            inorder_position[inorder[i]] = i;
        }
        TreeNode dummy;
        vector<tuple<TreeNode*, int, int>> stack;
        stack.emplace_back(&dummy, numeric_limits<int>::max(), numeric_limits<int>::max());
        for (int p:preorder) {
            TreeNode* node = new TreeNode(p);
            int node_position = inorder_position[node->val];
            auto [parent, left_limit, right_limit] = stack.back();
            if (node_position < left_limit) {
                parent->left = node;
                stack.emplace_back(node, node_position, left_limit);
                continue;
            }
            while (1) {
                auto[parent, left_limit, right_limit] = stack.back();
                if (node_position < right_limit) {
                    parent->right = node;
                    stack.pop_back();
                    stack.emplace_back(node, node_position, right_limit);
                    break;
                }
                stack.pop_back();
            }
        }
        return dummy.left;
    }
};
```

inorderの頭からの実装
```cpp
class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        map<int, int> preorder_position;
        for (int i = 0;i < preorder.size();i++) {
            preorder_position[preorder[i]] = i;
        }
        vector<TreeNode*> stack;
        auto gather_descendants = [&](int node_position) {
            TreeNode *child = nullptr;
            while (!stack.empty()) {
                TreeNode* back = stack.back();
                if (preorder_position[back->val] < node_position) {
                    break;
                }
                stack.pop_back();
                back->right = child;
                child = back;
            }
            return child;
        };
        for (int i:inorder) {
            TreeNode* node = new TreeNode(i);
            int node_position = preorder_position[node->val];
            node->left = gather_descendants(node_position);
            stack.emplace_back(node);
        }
        return gather_descendants(numeric_limits<int>::min());
    }
};
```
