問題：https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/

## step1
一時間くらいやったがわからなかった。
現在地の値＝次の値の場合と現在地≠次かつ次＝次の次の値の場合はcurrentnodeを進める。
で、現在地≠次かつ次≠次の次の値の場合はcurrentnode進めるしpreviousnodeも進める。
例：3-4-4　currentnodeのみ3から4に行き更に4に行く。
例：3-4-5　previousnodeもcurrentnodeも3から4に行き5に行く。
ただnextの付け替えをする必要があるので、後者の例の場合は移動前にpreviousnode->nextにcurrentnodeを付け替える。
このシステムでは先頭のhead付け替えが対応できず諦めた。
headをwhileループに召喚するなら複数回のループのうち初回のみこれを扱う必要があるが難しい。
答えみた。番兵使用してheadも前にnodeを持つnodeとして扱うとのこと。
↓間違い
```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        if (head==nullptr) {
            return nullptr;
        }
        unordered_set<int> seen;
        ListNode *fore=head;
        ListNode *back=head;
        seen.insert(head->val);
        while (fore!=nullptr) {
            if(seen.find(fore->val)!=seen.end()) {
                fore=fore->next;
                continue;
            }
        }
    }
};
```

## step2
回答は以下のようになった。

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        if (head == nullptr || head->next == nullptr) {
            return head;
        }
        ListNode dummy(-1,head);
        ListNode *back = &dummy;
        ListNode *fore = head;
        while (fore != nullptr && fore->next != nullptr) {
            if (fore->val == fore->next->val) {
                while (fore->next != nullptr && fore->val == fore->next->val) {
                    fore = fore->next;
                }
                back->next = fore->next;/*back自体は動かさない*/
            }
            else {
                back = back->next;
            }
            fore = fore->next;
        }
        return dummy.next;
    }
};
```
要するに新しい値が出てきたときにその最初の地点にはback->nextとforeのみが来ててback本体はforeの同値走査と同時には動かないと。
典型コメント集からの分かりやすいコードを真似た。
参考：https://discord.com/channels/1084280443945353267/1195700948786491403/1197102971977211966

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode dummy(-1,head);
        ListNode *previous = &dummy;
        ListNode *current = head;
        bool is_deleting = false;
        int deleting = 0;
        while (current) {
            if (is_deleting && deleting == current->val) {
                previous->next = current->next;
                current = current->next;
                continue;
            }
            is_deleting = false;
            if (!current->next) {
                previous = current;
                current = current->next;
                continue;
            }
            if (current->val != current->next->val) {
                previous = current;
                current = current->next;
                continue;
            }
            is_deleting = true;
            deleting = current->val;
            previous->next = current->next;
            current = current->next;
        }
        return dummy.next;
    }
};
```
continueでif文が無限ループにできることを学んだ。
一つ目のifで削除作業を行い、二つ目のifはcurrent次のノードがないなら終わろうというもの。
三つ目のifは1-2-3-4-5-6-7みたいな理想状態のときにpreviousを6、currentを7まで進める。
それ以外の場合は削除する機能をオンにする。
