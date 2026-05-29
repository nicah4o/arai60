問題： https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/description/

## step1

問題は
　・昇順に並べられたintの配列の各要素をnodeの値とするAVL木を作れ
というもの。

AVL木とは
　・全てのnodeについて、そのnodeを親とする左右の子がrootである二つの部分木の高さの差が1以下になるような二分探索木

AVL木を作ることはしたことがなかったので分からないと思い、答えを見た。
(copilotによると高さバランス木を作ると結果的にAVL木になるだけ。で、AVL木を作ることはふつう任意のnode挿入列に対して回転や挿入、削除を行うことで整数配列でAVL木を作るとは言わない)

## step2

- https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/solutions/6892739/video-find-middle-of-tree-or-subtree-by-94thd/

leetcodeのsolution。解法としては
　1.配列のある範囲を添え字で指定すると、その範囲の真ん中midにあたる要素を取り出してnodeにする関数を作る。
　2.そのnodeの->left,->rightにはさっきの範囲をmidから分割して渡す。
　3.left<rightとなったらそこが再帰の底。最後のnodeはleft=rightとなる。

 時間計算量：O(N)
 空間計算量：O(N)

 ```cpp
class Solution {
private:
    TreeNode* makeAVL(vector<int>& nums, int left, int right) {
        if (left > right)
            return nullptr;
        int mid = left + (right - left) / 2;
        TreeNode* node = new TreeNode(nums[mid]);
        node->left = makeAVL(nums, left, mid - 1);
        node->right = makeAVL(nums, mid + 1, right);
        return node;
    }

public:
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        return makeAVL(nums, 0, nums.size() - 1);
    }
};
```

nodeを挿入してAVL木を作る問題ではないので回転とかをする必要はない。

- https://github.com/colorbox/leetcode/pull/38/changes

二重ポインタを用いたBFSの方法

```cpp
class Solution {
public:
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        queue<tuple<int, int, TreeNode**>> nodes;
        TreeNode* root = new TreeNode();
        nodes.emplace(0, nums.size() - 1, &root);
        while (!nodes.empty()) {
            auto [left_index, right_index, ptr_to_node] = nodes.front();
            nodes.pop();
            if (left_index > right_index) {
                continue;
            }
            int mid_index = (left_index + right_index) / 2;
            *ptr_to_node = new TreeNode(nums[mid_index]);
            TreeNode* node = *ptr_to_node;
            nodes.emplace(left_index, mid_index - 1, &(node->left));
            nodes.emplace(mid_index + 1, right_index, &(node->right));
        }
        return root;
    }
};
```

(left+right)/2の方が left + (right - left) / 2より簡潔でわかりやすい。
ここでの二重ポインタは
　ptr_to_node=&rootを指し、次に*ptr_to_node = new TreeNode(……)でroot = new TreeNode(……)を意味している。ただ、rootもそもそもTreeNode *型である。
