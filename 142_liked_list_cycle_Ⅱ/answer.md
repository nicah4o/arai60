問題：https://leetcode.com/problems/linked-list-cycle-ii/
## step1

問題は141番の循環リストの循環開始ノードを特定するというもの。
leetcodeの仕様しらずprintfまで書くものと勘違いした。
それをやると配列の添え字が必要になるがそれができなかった。
↓特に意味ない

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        unordered_set<ListNode*> seen;
        ListNode *p = head;
        int n,m;
        while(p! = NULL){
            if(seen.contains(p)){
                if(seen.size >= m){
                    printf("tail connects to node index %d",n-m);
                }
                m = n;
            }
            n++;
            seen.insert(p);
            p = p->next;
        }
        printf("no cycle");
    }
};
```
nで今までの要素数を数えつつループ起きたらその時の要素数をmに保存しとく、みたいな。
解答みて一番はうさかめの方、パズル的解法がよくわからないと思った。
あとポインタを戻り値にすればいいんだっていう。

## step2

leetcodeの解説みた。
https://leetcode.com/problems/linked-list-cycle-ii/solutions/6750409/video-3-solutions-two-pointer-set-and-re-7ldf/
要は、スタートとループ開始点、衝突点の間をの距離をそれぞれA,B,Cとする。Cは衝突してからループ開始点までにあたる。
うさぎが衝突までにA+B+C+Bいくのに対しかめはA+Bのみ。かめの距離二倍がうさぎでその式を解くとA=Cとなる。
衝突状態から一人だけスタート地点に持ってきて今度はかめの速度で歩くとA=Cだけ歩きループ開始点に来れる。

ハッシュを使う方法も引き続きある。記述はコンパクトだが空間計算量がO(n)でうさかめはO(1)。
特にseen.end()を右辺値にしないとイテレータ同士の等式にならないのにひっかかった。

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        unordered_set<ListNode*> seen;
        while(head != nullptr){
            if(seen.find(head) != seen.end()){
                return(head);
            }
            seen.insert(head);
            head = head->next;
        }
        return(nullptr);
    }
};
```

## step3

うさかめを練習した。

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode *fast = head;
        ListNode *slow = head;
        while(fast != nullptr && fast->next != nullptr){
            fast = fast->next->next;
            slow = slow->next;
            if(fast == slow){
                break;
            }
        }
        if(fast == nullptr || fast->next == nullptr){
            return(nullptr);
        }
        fast = head;
        while(fast != slow){
            fast = fast->next;
            slow = slow->next;
        }
        return(fast);
    }
};
```
この問題の意図としては二通りの解法でハッシュの方が標準的だが、パズルの方は知らないとできない。
そのため知識をその場で与えられたときの反応・考えを見るといった意味があるらしい。
参考：https://discord.com/channels/1084280443945353267/1262761766887358557/1316089240680923257
