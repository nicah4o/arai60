問題：https://leetcode.com/problems/first-unique-character-in-a-string/description/

## step1

問題は
　1.文字列を与えられ、その内で重複していない最初の要素があるインデックスを返す
　2.例えば「leetcode」なら重複しないのは'l','t','c','o','d'だが、最初の'l'があるインデックスの0を返す
というもの。

最初に考えたのは、mapで{文字,インデックス}とした早見表を作ること。
探索にO(NlogN)かかってしまうし、どちらにせよ全探索のO(N)かかりそうだから（最後の一文字まで見ないと単独が確定できないため）、問題の分類でもあるhashtableでいこうとした。

unordered_setですでに現れた文字を記録することを考えたが、返り値インデックスの要求に対してその情報を保持できない。

よくよく考えるとstring型には元々find()メソッドがあることを思い出した。昨日の問題で重厚なcpp版split関数を書いていた方のコードから学んだことだ。
じゃあそもそもhashtableを使う必要すらない。後ろからの探索と前からの探索がぶつかればそれが単独だから、そのインデックスを返せばよい。

```cpp
class Solution {
public:
    int firstUniqChar(string s) {
        for (int i = 0; i < s.size(); i++) {
            if (s.rfind(s[i]) == s.find(s[i])) {
                return i;
            }
        }
        return -1;
    }
};
```

空間計算量はO(1)で時間計算量はO(N^2)だと思う。時間のほうは自信がない。
文字列最後の文字が最初の単独文字の場合、N-1回の両側からの走査（真ん中同士で最悪O(N)）でO(N^2)だと思う。

## step2

コメント集と他の方のコードをみた。
まず{文字,頻度}を計測して、その上でもう一度走査する時に頻度1のインデックスを返すというもの。これはO(N)で済む。
参考：https://leetcode.com/problems/first-unique-character-in-a-string/solutions/6975043/video-2-simple-solutions-by-niits-53r9

```cpp
class Solution {
public:
    int firstUniqChar(string s) {
        unordered_map<char, int> char_to_count;
        for (char c : s) {
            char_to_count[c]++;
        }
        for (int i = 0; i < s.size(); i++) {
            if (char_to_count[s[i]] == 1) {
                return i;
            }
        }
        return -1;
    }
};
```

これ、前半はアルファベットで26文字の配列を作るだけでもよいとのこと。

pythonなら一度のループでいけるようだ。
参考：https://discord.com/channels/1084280443945353267/1233603535862628432/1237973103670198292
> gotoさんの解法とも被りますが、

>最初に文字の登場回数をカウントして、2回目のループで登場回数が1回だけの文字を探すというのが最初に思いつきます。
登場回数のカウントは、問題の制約でアルファベット小文字26文字とあるので配列（list）で持ってもいいですし、ハッシュテーブル（dict）で持っても良いかなと思います。個人的には特段パフォーマンス要件などなければハッシュテーブルでいいかなと思います。

>その次に、ループを実は1周にできるのではというのが思いつきます。1回登場した文字のインデックスを保持するハッシュテーブルと2回以上登場した文字を管理するハッシュセットを用意して、最後にハッシュテーブルの先頭要素を返す感じになるかなと思います。

```cpp
class Solution {
public:
    int firstUniqChar(string s) {
        unordered_map<char, int> char_to_index;
        unordered_set<char> overraps;
        for (int i = 0; i < s.size(); i++) {
            if (overraps.contains(s[i])) {
                continue;
            }
            if (!char_to_index.contains(s[i])) {
                char_to_index[s[i]] = i;
            } else {
                char_to_index.erase(s[i]);
                overraps.insert(s[i]);
            }
        }
        int min_index = INT_MAX;
        for (const auto& [key, val] : char_to_index) {
            if (val < min_index) {
                min_index = val;
            }
        }
        return (min_index == INT_MAX) ? -1 : min_index;
    }
};
```

linked list mapというjavaにしかないmapでは入力順（insertした順）にデータを辿れるとのこと。
以下、geminiの実装。

```cpp
#include <iostream>
#include <unordered_map>
#include <list>
#include <string>

class SimpleLinkedHashMap {
private:
    // 1. 挿入順を記憶するための連結リスト（キーと値のペアを保持）
    std::list<std::pair<std::string, int>> order_list;
    
    // 2. 高速検索のためのハッシュマップ
    // 【ポイント】値の代わりに、リスト内の「住所（イテレータ）」を記憶する！
    std::unordered_map<std::string, std::list<std::pair<std::string, int>>::iterator> hash_map;

public:
    // データの挿入 (put)
    void put(const std::string& key, int value) {
        // すでに同じキーがある場合は、古い住所のデータをリストから消す（重複防止）
        if (hash_map.find(key) != hash_map.end()) {
            order_list.erase(hash_map[key]);
        }
        
        // A. リストの末尾（一番後ろ）に最新のデータを追加する
        order_list.push_back({key, value});
        
        // B. ハッシュマップに「今リストの末尾に追加したデータの住所」をメモする
        // (--order_list.end() で、今入れた最後の要素を指すポインタが取れます)
        hash_map[key] = --order_list.end();
    }

    // データの検索 (get)
    int get(const std::string& key) {
        if (hash_map.find(key) == hash_map.end()) {
            return -1; // 見つからない場合
        }
        // ハッシュマップから「リストの住所」を引き、そこから中身（secondのデータ）を返す
        return hash_map[key]->second;
    }

    // 挿入順での走査 (Print)
    void printAll() {
        // リストの鎖を先頭から順番にたどるだけで、挿入順に出力できる！
        // 以前やった C++17の構造化束縛 [key, val] がここで活きます
        for (const auto& [key, val] : order_list) {
            std::cout << key << " : " << val << std::endl;
        }
    }
};

int main() {
    SimpleLinkedHashMap lhm;
    lhm.put("Alice", 90);
    lhm.put("Bob", 80);
    lhm.put("Charlie", 95);
    
    // ハッシュマップだけど、挿入した順（Alice -> Bob -> Charlie）に出る！
    lhm.printAll(); 
}
```
