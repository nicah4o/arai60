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
            TreeNode* root = new TreeNode(preorder.front());
            preorder.pop_front();
            auto it = find(inorder.begin(), inorder.end(), root->val);
            auto idx = it - inorder.begin();
            vector<int> left_inorder(inorder.begin(), inorder.begin() + idx);
            vector<int> right_inorder(inorder.begin() + idx +1, inorder.end());
            root->left = build(preorder, left_inorder);
            root->right = build(preorder, right_inorder);
            return root;
        }
        return nullptr;
    }
};
```

```cpp
class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        deque<int> preorderdq(preorder.begin(), preorder.end());
        return build(preorderdq, inorder);
    }
private:
    TreeNode* build(deque<int>& preorder,vector<int>& inorder) {
        if (!inorder.empty()) { 
        TreeNode* node = new TreeNode(preorder.front());
        preorder.pop_front();
        auto it = find(inorder.begin(), inorder.end(), node->val);
        auto idx = it - inorder.begin();
        vector<int> left_inorder(inorder.begin(), inorder.begin() + idx);
        vector<int> right_inorder(inorder.begin() + idx + 1, inorder.end());
        node->left = build(preorder, left_inorder);
        node->right = build(preorder, right_inorder);
        return node;
        }
        return nullptr;
    }
};
```
left_inorderとright_inorderはコピーの手間が無駄なので関数で引き回してもよい。
その場合、関数に引数にleft_limitとright_limitが出る。下のコード。
findは探索にO(N)かかるのでmapのO(logN)のほうがよい。下のコード。
dequeもインデックス管理でよい。

```cpp
class Solution {
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        map<int, int> inorder_position;
        for (int i = 0;i < inorder.size();i++) {
            inorder_position[inorder[i]] = i;
        }
        vector<tuple<TreeNode*, int, int>> nodes_and_range;
        TreeNode dummy;
        nodes_and_range.emplace_back(&dummy, numeric_limits<int>::max(), numeric_limits<int>::max());
        for (int p:preorder) {
            TreeNode* node = new TreeNode(p);
            int node_position = inorder_position[node->val];
            auto[parent, left_limit, right_limit] = nodes_and_range.back();
            if (node_position < left_limit) {
                parent->left = node;
                nodes_and_range.emplace_back(node, node_position, left_limit);
                continue;
            }
            while (1) {
                auto[parent, left_limit, right_limit] = nodes_and_range.back();
                if (node_position < right_limit) {
                    parent->right = node;
                    nodes_and_range.pop_back();
                    nodes_and_range.emplace_back(node, node_position, right_limit);
                    break;
                }
                nodes_and_range.pop_back();
            }
        }
        return dummy.left;
    }
};
```
