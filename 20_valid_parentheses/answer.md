問題：https://leetcode.com/problems/valid-parentheses/description/

## step1
stack思いつかなかった。
両端から括弧さがして見つけた方止まりもう片方も見つかったら進む。
それで最後出会ったときに括弧を見つけた回数で正誤判定。
↓間違い。

```cpp
class Solution {
public:
    bool isValid(string s) {
        int n = 0;
        int m = strlen(s);
        while (n != m){
            if (s[n] == 40 || s[m] == 41) {
                n = s[n] == 40 ? n : n+1;
                m = s[m] == 41 ? m : m-1;
            }
            else if (s[n] == 91 || s[m] == 93) {
                n = s[n] == 91 ? n : n+1;
                m = s[m] == 93 ? m : m-1;
                continue;
            }
            else if (s[n] == 123 || s[m] == 125) {
                n = s[n] == 123 ? n : n+1;
                m = s[m] == 125 ? m : m-1;
                continue;
            }
            n++;
            m--;
        }
    }
};
```
ここまで書いて断念。
あー、これ括弧しか出ないんですね。

## step2
他の方の回答見た。初見でわかってる方多くてビビる。

```cpp
class Solution {
public:
    bool isValid(string s) {
        unordered_map<char,char> map={{')','('},{'}','{'},{']','['}};
        stack<char>st;
        for(char c:s){
            if(map.find(c)==map.end()){
                st.push(c);
            }
            else if(!st.empty()&&map[c]==st.top()){
                st.pop();
            }
            else{
                return false;
            }
        }
        return st.empty();
    }
};```
余りpr見れず上の解答を覚えた。
unordered_mapを使うと閉じ括弧を鍵として開き括弧を消去できる。
予め'\0'をpushしておけば、else ifの!st.empty()は取り除ける。
参考：https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.ns0bie22a6m
