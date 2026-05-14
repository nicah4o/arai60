問題：https://leetcode.com/problems/group-anagrams/description/

## step1

問題は
1.strsという文字列の配列が与えられ
2.その中からアナグラムになっている文字列(eatとateとteaなど)を一つの配列にまとめ、strsの全ての文字列をアナグラムごとに配列に格納した配列を作れ
というもの。

最初に思いつく選択肢としては、文字列を文字コードの和として管理する、というもの。
ただ、同じ程度には違う文字列でも文字コードの和は同じになるだろうことが予想されやすく実装されない。

分からなかったのでアナグラムの"判定だけ"どうしたらいいか調べた（それが全てではある）。
結果、アナグラムの判定は文字列のソートをすることで判定できることが分かった。

ソートした文字列をキーとしてアナグラムにあたる文字列のインデックスの集合を要素とするハッシュマップを作ろうとするも断念。
sort()の使い方と(マップ)[キー].push_back(要素の配列への新要素)という書き方が分かっていなかった。

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<int>> word_to_indexes;
        for (int i = 0; i < strs.size(); i++) {
            word_to_indexes[sort(strs[i].begin(), strs[i].end())].insert(i);
        }
    }
};
```

## step2

leetcodeの解答見た。
一つのやり方としては
1.キー：ソートされた文字列　要素：元の文字列たちの格納された配列、のunordered_mapを作る。
2.返り値として変数を作り、そこにunordered_mapの.secondで与えられるアナグラムの成立する文字列たちを順に格納する。
というもの。

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> sort_to_same;
        for (string& s : strs) {
            string sorted = s;
            sort(sorted.begin(), sorted.end());
            sort_to_same[sorted].push_back(s);
        }
        vector<vector<string>> ans;
        for (auto& i : sort_to_same) {
            ans.push_back(i.second);
        }
        return ans;
    }
};
```

これがアナグラム→文字列ソートという情報を知っていた際の標準的な回答になると思う。
計算量は空間がO(NL)で時間がO(NLlogL)になる。ソートが入るから文字数L回分logLがある。
こちらをstep3で覚えた。

コメント集と他の方のコードを見た。
参考：https://github.com/Fuminiton/LeetCode/pull/12/changes

アルファベットの出現回数を数えてキーを生成するという方法もあるらしい。
"1#0#0#0#2#..." （aが1個、bが0個、...）というようなキーを作る。キーに対応する要素は引き続きアナグラム文字列の集合である。
こちらはソートをしないので時間計算量もO(NL)になる。

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> ans;

        for (string& s : strs) {
            array<int, 26> count = {0};

            // Count frequency of each letter in the string
            for (char c : s) {
                count[c - 'a']++;
            }

            string key;
            for (int num : count) {
                key += to_string(num) + "#";
            }

            ans[key].push_back(s);
        }

        vector<vector<string>> result;
        for (auto& entry : ans) {
            result.push_back(move(entry.second));
        }

        return result;        
    }
};
```
引用：https://leetcode.com/problems/group-anagrams/solutions/6113105/video-create-keys-for-group-of-strings-2-tqdx

この解法の論点として、pythonでは大文字が入力されたときに負の添え字（後ろから数えた時の添え字）への回数の追加が起こるし、cppでは単に境界外アクセスを起こすということがコメント集で指摘されている。
