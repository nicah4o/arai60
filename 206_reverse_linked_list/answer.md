問題：https://leetcode.com/problems/reverse-linked-list/description/

## step1

まあ前回に引き続きスタックだからとりあえず全部の->valをpushして走査したら取り出してheadからのリストノードの値を入れ替えてみる。
```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        stack<int> valstack;
        ListNode* dummy = new ListNode(2147483647,head);
        while (head) { 
            valstack.push(head->val);
            head = head->next;
        }
        ListNode *temp = dummy->next;
        while (!valstack.empty()) {
            temp->val = valstack.top();
            valstack.pop();
            temp = temp->next;
        }
        return dummy->next;
    }
};
```
これでaccept。計算量はどちらもO(n)。

## step2

他の方のコード、コメント集を見た。再帰の方法もあるがO(n)。
参考：https://discord.com/channels/1084280443945353267/1231966485610758196/1239417493211320382

```cpp
class Solution {
private:
    ListNode* recursion(ListNode* head) {
        if (!head || !head->next) {
            return head;
        }
        else {
            ListNode* newhead = recursion(head->next);
            head->next->next = head;
            head->next = nullptr;
            return newhead;
        }
    }
```

特にこの方の空間計算量O(1)での方法は参考になった。
参考：https://github.com/attractal/leetcode/pull/7/changes#diff-4066faf067589a38d6fd384b7a41db69f3320f5699ad178f6a6df82904e85333R39

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* previous = nullptr;
        ListNode* node = head;
        while (node != nullptr) {
            ListNode* next_node = node->next;
            node->next = previous;
            previous = node;
            node = next_node;
        }
        return previous;
    }
};
```
再帰では始めに一番最後のノードが返されるところから最初のノードにまで遡っていくが、この方法ではprevious=nullとするイニシャルコストを払うことで最初から最後まで直線的に処理が行える。
