問題：https://leetcode.com/problems/add-two-numbers/
## step1
割と簡単そうだと思った。
値を取り出す順に10^nをかけることで元の整数を復元できる。
桁数nと合計値を記録しておき、合計値のほうを高い桁から割っていく。
桁を下げつつノードに格納。
↓間違い

```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        int n1, n2, n3;
        int ten1 = 0;
        while (l1 != nullptr) {
            if (l1->next == nullptr) {
                n1 += l1->val;
                break;
            }
            n1 += (pow(10, ten1) * l1->val);
            l1 = l1->next;
            ten1++;
        }
        int ten2 = 0;
        while (l2 != nullptr) {
            if (l2->next == nullptr) {
                n2 += l2->val;
                break;
            }
            n2 += (pow(10, ten2) * l2->val);
            l2 = l2->next;
            ten2++;
        }
        if (ten1 == ten2) {
        } else if (ten1 > ten2) {
            ten2 = ten1;
        } else {
            ten1 = ten2;
        }
        ListNode* head;
        head = nullptr;
        n3 = n1 + n2;
        while (ten1 != -1) {
            ListNode* p = new ListNode(n3 / pow(10, ten1));
            p->next = head;
            n3 %= (long long)pow(10, ten1);
            ten1--;
            head = p;
        }
        return head;
    }
};
```
スタックオーバーフローで上手くいかない。intは10桁、long long型で19桁。
答え見た。
10^nかけるという発想にとらわれ、元の数を復元しなくてもよいと気づかなかった。
筆算のこと、全然わかってなかった。

##step2
他の方の回答見た。
参考：https://github.com/takao-Tokunaga/leetcode/pull/5/changes
再帰で書くものもあるらしいが見れていない。

```cpp
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        int sum, carry = 0;
        ListNode dummy(-1);
        ListNode* node = &dummy;
        while (l1 || l2 || carry) {
            int v1 = l1 ? l1->val : 0;
            int v2 = l2 ? l2->val : 0;
            sum = v1 + v2 + carry;
            carry = sum / 10;
            node->next = new ListNode(sum % 10);
            node = node->next;
            if (l1)
                l1 = l1->next;
            if (l2)
                l2 = l2->next;
        }
        return dummy.next;
    }
};
```
