問題： https://leetcode.com/problems/validate-binary-search-tree/description/

## step1

問題は
　・与えられたbinary treeがbinary search treeか判定せよ
というもの。

BSTの定義は
　1.左の部分木が子とその子孫に至るまで親のnodeの値未満である。
　2.右の部分木が子とその子孫に至るまで親のnodeの値より大きいこと。
　3.全ての部分木がBSTの条件を満たすこと。

考えたこと
　・今までの方法ではあるnodeとその子のみを見ることが多かったがこの問題ではrootから見る必要がある。
　・rootの左側はrootより必ず小さくなり、右側はrootより必ず大きくなる。
　・ここら辺を上手く分けるロジックが分からなかった。

コード
　・search_leftとsearch_rightを関数で作ってそれぞれmax値とmin値、nodeを引数として渡した。
　・自分の子を見るとこまではできるがその子が自分の親と比べてBSTを形成しているのかを判定できなかった。
　・例えば[32,26,47,19]と並んだ時に26から見て左の子が19なのはOKだが32のrootからみるとダメ。このような親の親と親の判定違いを正すロジックが組めなかった。
 
 ```cpp
class Solution {
public:
    bool search_left(TreeNode* node, int max) {
        if (!node)
            return true;
        if (!node->left || !node->right) {
            bool left_is_less = node->left ? node->left->val < node->val : true;
            bool right_is_great =
                node->right ? node->right->val > node->val : true;
            if (!left_is_less || !right_is_great)
                return false;
        } else if (node->left->val >= node->val ||
                   node->right->val <= node->val) {
            return false;
        }
        if (node->left) {
            return search_left(node->left, max);
        }
        if (node->right && node->right->val < max) {
            return search_right(node->right, max);
        } else if (node->right) {
            return false;
        }
        return true;
    }

    bool search_right(TreeNode* node, int min) {
        if (!node)
            return true;
        if (!node->left || !node->right) {
            bool left_is_less = node->left ? node->left->val < node->val : true;
            bool right_is_great =
                node->right ? node->right->val > node->val : true;
            if (!left_is_less || !right_is_great)
                return false;
        } else if (node->left->val >= node->val ||
                   node->right->val <= node->val) {
            return false;
        }
        if (node->left && node->left->val > min) {
            return search_left(node->left, min);
        } else if (node->left) {
            return false;
        }
        if (node->right) {
            return search_right(node->right, min);
        }
        return true;
    }

    bool isValidBST(TreeNode* root) {
        return search_left(root->left, root->val) &&
               search_right(root->right, root->val);
    }
};
```

時間かけると解ける時があるので損切りに時間がかかってしまった。解けることが目的ではないが快楽が大きい。

## step2

leetcodeの解説を見た。
- https://leetcode.com/problems/validate-binary-search-tree/solutions/5622933/video-check-range-of-each-node-by-niits-eofa/

解き方
　1.ある親nodeの左の子が取りうる値の範囲はその親nodeの取りうる値の範囲のうち、親nodeが取った値を最大値とする狭められた範囲である。
　2.ある親nodeの右の子が取りうる値の範囲はその親nodeの取りうる値の範囲のうち、親nodeが取った値を最小値とする狭められた範囲である。
　3.左のとき：min < node->left->val < node->val < max (minとmaxは親の値の範囲)
　4.右のとき：min < node->val < node->right->val < max
　5.つまり、左の子のときはnode->valを新しいmaxとして右の子のときはnode->valを新しいminとすればよい。

```cpp
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        return valid(root, min, max);        
    }

private:
    bool valid(TreeNode* node, long min, long max) {
        if (!node) return true;
        if (!(node->val > min && node->val < max)) return false;
        return valid(node->left, min, node->val) && valid(node->right, node->val, max);
    }    
};
```

2^-231 <= Node.val <= 2^231 - 1なのでint型だと判定できない場合のためにlong型を使う。

