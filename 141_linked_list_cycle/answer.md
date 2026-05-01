## step1

普通の線形リストしか知らなかったので見当がつかなかった。
適当な長さの配列にvalueの無限ループが収まるか否かみたいな無限/有限を利用するのかと考えた。
解答見た。
速度の違うポインタで追いかける。最初が一番遠いのであとは速度の差の速さで縮んでいくというもの。

## step2

他の方の回答も見た。どうやらハッシュを用いた方法がこの問題の本意であるらしい。
面接官目線での講師の方コメント
参考：https://github.com/hiro111208/leetcode/pull/1/changes
”うさぎとかめ”のほうがノード二つなので空間計算量は優秀（O(n)とO(1)）。
直近の方を参考にした。
参考：https://github.com/enari-k/LeetCode/pull/65/changes

```cpp
class Solution {
public:
bool hasCycle(struct ListNode* head) {
    struct ListNode* fast = head;
    struct ListNode* slow = head;
    while (fast != NULL && fast->next != NULL) {
        fast = fast->next->next;
        slow = slow->next;
        if (fast == slow) {
            return true;
        }
    }
    return false;
    }
};
```

## step3

こちらのハッシュの方法が標準的なのでこちらを覚えた。
変数seenをポインタ型にしてしまうミスが多発した。

```cpp
class Solution {
public:
    bool hasCycle(ListNode *head) {
        unordered_set<ListNode*> seen;
        ListNode *p = head;
        while(p != NULL){
            if(seen.contains(p)){
                return true;
            }
            seen.insert(p);
            p = p->next;
        }
        return false;
    }
};
```
