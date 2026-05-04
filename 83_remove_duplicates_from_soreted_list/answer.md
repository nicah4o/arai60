問題：https://leetcode.com/problems/remove-duplicates-from-sorted-list/description/

##step1
  Given the head of a **sorted** linked list, delete all duplicates such that each element appears only once.
予めソートされてることに気づかずにやり始めてしまった。
つまり[1,1,2,1]とかが出るのかと勘違いした。
この場合は二つのポインタとハッシュセットが必要だと思った。
前に出るものと後ろに出るもの。それでハッシュセットを前の値で検索してあったら後ろから書き換える。
↓間違い

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        unordered_set<ListNode*> seen;
        ListNode *back = head;
        ListNode *fore = head->next;
        
        while (back != nullptr && fore != nullptr) {
            seen.insert(back);
            if (seen.find(fore) != seen.end()) {
                back->next = back->next->next;
            }
            else {
                back = back->next;
                fore = back->next;
            }
        }
        return head;
    }
};
```
141.linked list cycle(https://leetcode.com/problems/linked-list-cycle/description/) のようにポインタを二個使う必要はない。
むしろnullptrを読むので良くない。あとunordered_setをint型にしないと要素が見れない。
リストはポインタを格納してるのであってこの状態からfindメソッドの引数としてint型の要素は探せない。
こういう基本仕様を141やったときに理解しておくような勉強をする必要がある。

## step2
他の方の回答みた。普通の解き方は以下のようになる。

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode *temp = head;
        while (temp != nullptr && temp->next != nullptr) {
            ListNode *fore = temp->next;
            if (temp->val == fore->val) {
                temp->next = temp->next->next;
            }
            else {
                temp = temp->next;
            }
            
        }
        return head;
    }
};
```
特にここでいうtempをcurrentにしてcurrentが右往左往するのはよくないとの指摘があった。
参考：https://github.com/aiueoriku/LeetCode/pull/9#discussion_r2638341640
あとはcontinueを使用してelseを使わないもの
参考：https://github.com/tamagoyaki-chiu/LeetCode-Practice/pull/6#discussion_r3128441852
50行目から

```cpp
            if(temp->val == fore->val){
                temp->next = temp->next->next;
                continue;
            }
            temp = temp->next;
```
番兵を使うもの
参考：https://github.com/takao-Tokunaga/leetcode/pull/3#discussion_r3067895722

再帰も使えるらしい。
参考：https://github.com/tamagoyaki-chiu/LeetCode-Practice/pull/6#discussion_r3141263821
書いた。

```cpp
private:
    ListNode* deleteloop(ListNode* head) {
        ListNode* temp = head;
        if (temp != nullptr && temp->next != nullptr) {
            if (temp->val == temp->next->val){
                temp->next = temp->next->next;
                return deleteloop(temp);
            }
            else {
                temp->next = deleteloop(temp->next);
            }
        }
        return head;
    }
```
89行目のreturnとelse以下の代入がわからなかったからgemini使用した。
で、returnは同じ場所で二度以上deleteする機能であり、else以下の代入はもしtemp->nextが存在しなくなってた場合に繋げる機能がある。
[1,3,3,3,5]とかで3を全消しする場合1->nextは存在しないからきちんと5につなぐ必要がある。
当初のハッシュを使うものも書いた。
空間計算量がO(n)ではある。

```cpp
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        unordered_set<int> seen;
        ListNode *temp = head;
        while (temp != nullptr && temp->next != nullptr) {
            seen.insert(temp->val);
            if (seen.find(temp->next->val) != seen.end()) {
                temp->next = temp->next->next;
                continue;
            }
            temp = temp->next;
        }
        return head;
    }
};
```
