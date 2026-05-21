問題：https://leetcode.com/problems/word-ladder/description/

## step1

問題は
　1.wordListというstringの配列とbeginWord,endWordという文字列が与えられ、
　2.beginWordを一文字ずつ変更してendWordにするのだが、その途中の文字はwordListに含まれる文字である必要がある。
　3.beginWordからwordListの文字列を経由して最短でendWordになる連続する文字列の個数をbeginWord,endWord含めて返り値として渡せ
というもの。

最短の問題なのでとりあえずBFSで解くことにした。ただ、走査はできてもそこから最短の長さを求めることができなかった。
以下は間違っているコード。unordered_mapで前の文字列を保存しようとしている。

```cpp
class Solution {
public:
    int ladderLength(string beginWord, string endWord,
                     vector<string>& wordList) {
        auto it = find(wordList.begin(), wordList.end(), endWord);
        if (it == wordList.end())
            return 0;
        unordered_map<string, string> visited;
        queue<string> wordque;
        wordque.push(beginWord);
        while (!wordque.empty()) {
            string word = wordque.front();
            wordque.pop();
            for (string s : wordList) {
                int difcount = 0;
                for (int i = 0; i < s.size(); i++) {
                    if (!visited[s].empty())
                        break;
                    if (s[i] != word[i]) {
                        difcount++;
                    }
                }
                if (difcount == 1) {
                    wordque.push(s);
                    visited[s] = word;
                }
            }
        }
    }
};
```

## step2

他の方のコードを見た。
自分のコードでは文字列が一文字違いのものか検証するためにwordListを全件走査しているが、unordered_setにwordListをコピーすることで手元の文字列の方を(文字列の長さ)×(アルファベット26文字)で変化させたもので検索できるようになる。
例えば文字列の個数Nに対して文字列の長さMとすると、自分のコードでは検証に最悪O(NM)かかるが、unordered_setならO(M)で済む。

あとは、wordとそれがbeginWordから何個目の文字列なのかを同時にpairで扱うという方法も思いつかなかった。
BFSでは前方から走査するとbegin="hit",end="cog"でwordList=["hot","dot","dog","lot","log","cog"]なら、キューには
hit-hot, hot-dot-lot, dot-lot-dog, lot-dog-log, dog-log-cog, log-cog, cogと積み重なっていき、cogが出るまでにどのように複数の継承線(hot-dot-dog-cogとhot-lot-log-cogの二つの方向のこと)を維持するのか分からなかった。

step1のコードを使いまわした。

```cpp
class Solution {
public:
    int ladderLength(string beginWord, string endWord,
                     vector<string>& wordList) {
        auto it = find(wordList.begin(), wordList.end(), endWord);
        unordered_set<string> visited;
        queue<pair<string, int>> wordque;
        wordque.push({beginWord, 1});
        visited.insert(beginWord);
        while (!wordque.empty()) {
            string word = wordque.front().first;
            int current_length = wordque.front().second;
            wordque.pop();
            if (word == endWord) {
                return current_length;
            }
            for (string s : wordList) {
                if (visited.contains(s)) {
                    continue;
                }
                int difcount = 0;
                for (int i = 0; i < s.size(); i++) {
                    if (s[i] != word[i]) {
                        difcount++;
                    }
                }
                if (difcount == 1) {
                    wordque.push({s, current_length + 1});
                    visited.insert(s);
                }
            }
        }
        return 0;
    }
};
```

2627msかかった。時間計算量はO(NM)の検証がN回なのでO(N^2×M)で空間計算量も最悪O(NM)。

## step3

```cpp
class Solution {
public:
    int ladderLength(string beginWord, string endWord,
                     vector<string>& wordList) {
        unordered_set<string> words(wordList.begin(), wordList.end());
        queue<pair<string, int>> wordque;
        unordered_set<string> visited;
        wordque.push({beginWord, 1});
        visited.insert(beginWord);
        while (!wordque.empty()) {
            string word = wordque.front().first;
            int current_num = wordque.front().second;
            wordque.pop();
            if (word == endWord) {
                return current_num;
            }
            for (int i = 0; i < word.size(); i++) {
                string changed_word = word;
                for (char c = 'a'; c <= 'z'; c++) {
                    changed_word[i] = c;
                    if (words.contains(changed_word) &&
                        !visited.contains(changed_word)) {
                        wordque.push({changed_word, current_num + 1});
                        visited.insert(changed_word);
                    }
                }
            }
        }
        return 0;
    }
};
```
こちらは計算量はO(NM)。Mの定数倍の変更を文字列N個に対して行う。

参考；https://github.com/Hurukawa2121/leetcode/pull/20
visitedを使わずともeraseで辞書のハッシュセットから消すこともできる。
