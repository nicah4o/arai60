問題：https://leetcode.com/problems/top-k-frequent-elements/description/

## step1

問題は
1.vector<int>型の配列numsと整数kを与えられる。
（ただし、kは最低1から最大でも「配列に一回のみ出現する値」の個数までしか取らない。例：[1,2,3,4,4]ならばkは1から3までしか取らない。）
2.配列の中で最も多く出現する値を上からk個数えてvector<int>型の配列として返す。配列内の順番は問わない。
というもの。

前回の問題で用いたpriority_queueを単純に適用することはできない。求めたいのは値の大小ではなく値の出現頻度だ。
値が現れた回数を値とのセットで扱う必要がある。キーとそれに対応する値をpairとして格納するmapを使用する。
（前回：https://leetcode.com/problems/kth-largest-element-in-a-stream/description/）

まずmapに<キー, 要素> = <値, 値の出現回数>として値と出現回数を記録する。
mapではキーから要素にアクセスできるが要素からキーを探すことはできない。今回、出現回数の大小から必要な値を求めたい。つまり要素からキーにアクセスしたい。
だから、二つ目のmapにキーと要素を逆転して格納する。つまり、<キー, 要素> = <値の出現回数, 値>としたmapをつくる。
しかし、mapは要素の重複は認めてもキーの重複は認めない。逆関数をmapで作るには全単射である必要があるが、同じ出現回数の異なる値がある以上不可能である。
解決策として、二つ目のmapにはmultimapというキーの重複を許すmapを使うことでキーの重複する異なる値を格納できる。

最後に、vector<int>を作ってmultimapの後ろから走査を行う。mapは昇順、つまり最初のイテレータに最小のキー、最後のイテレータに最大のキーが並ぶ。
出現回数の最も多い要素（値）をk個入れたvectorを返す。

計算量はgeminiによると空間計算量がO(M)で時間計算量がO(NlogM)になるらしい。Mはnumsのうちのユニークな要素数。

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        map<int, int> mp;
        for (int i:nums) {
            if (mp.contains(i)) {
                mp[i]++;
            }
            else {
                mp[i] = 1;
            }
        }
        multimap<int, int> multi;
        for (const auto& i:mp) {
            multi.emplace(i.second,i.first);
        }
        vector<int>v;
        std::multimap<int ,int>:: reverse_iterator it;
        it = multi.rbegin();
        while (v.size() != k) {
            v.push_back(it->second);
            it++;
        }
        return v;
    }    
};
```
acceptされたが7msで最頻値0msより遅い。別の方法がある。

## step2

他の方のコードとコメント集みた。priority_queue使う方法とクイックセレクトという方法があるらしい。
参考：https://discord.com/channels/1084280443945353267/1235829049511903273/1245555256360697949

頻度の数え上げはmapでなくunordered_mapでおこなう。unordered_mapはハッシュマップなので時間計算量がO(N)で済む。対してmapは木構造でO(NlogM)。
あと、priority_queueでgreater（昇順）にしたもの最小ヒープとよぶらしい。最小ヒープの要素数を減らしていくと自動的にk番目に行き着く。
priority_queueにはpairも入れることができる。firstでソートした後にsecondでソートされるようだ。

```cpp
class Solution {
public:
    std::vector<int> topKFrequent(std::vector<int>& nums, int k) {
        // 1. unordered_map で数え上げ (O(N))
        std::unordered_map<int, int> counts;
        for (int n : nums)
            counts[n]++;

        // 2. 最小ヒープを作成。中身は pair<頻度, 数字>
        // 頻度が低いものが top() に来るように std::greater を使う
        using pi = std::pair<int, int>;
        std::priority_queue<pi, std::vector<pi>, std::greater<pi>> min_heap;

        for (auto const& [num, freq] : counts) {
            min_heap.push({freq, num});
            if (min_heap.size() > k) {
                min_heap.pop(); // 頻度が一番低いものを追い出す
            }
        }

        // 3. ヒープに残った K 個を取り出す
        std::vector<int> result;
        while (!min_heap.empty()) {
            result.push_back(min_heap.top().second);
            min_heap.pop();
        }
        return result;
    }
};
```
geminiの書いたこの理想的なコードをstep3で練習。計算量は時間O(N+Mlogk)で空間O(M)。

```cpp
class Solution {
public:
    std::vector<int> topKFrequent(std::vector<int>& nums, int k) {
        std::unordered_map<int, int> counts;
        for (int n : nums)
            counts[n]++;

        // (数字, 頻度) のペアを vector に移す
        std::vector<std::pair<int, int>> unique;
        for (auto const& pair : counts) {
            unique.push_back(pair);
        }

        // std::nth_element を使い、頻度で並び替える
        // 第2引数の位置（M-k番目）に、正しい要素が来るように配置される
        std::nth_element(
            unique.begin(), unique.begin() + unique.size() - k, unique.end(),
            [](const std::pair<int, int>& a, const std::pair<int, int>& b) {
                return a.second < b.second; // 頻度の昇順
            });

        // 境界線（M-k）から後ろが「上位 k 個」
        std::vector<int> result;
        for (int i = unique.size() - k; i < unique.size(); ++i) {
            result.push_back(unique[i].first);
        }
        return result;
    }
};
```
これもgeminiのクイックセレクト。{数字, 頻度}のペアでM-K番目の頻度を基準としてそれより小さいものを前に移転するnth_element関数。
この関数の引数は(fist, nth, last, policy, comp)で、上のコードではcompにラムダ式を用いている。compは引数1が引数2が小さいならtrueを返すような関数を受けるほか、greater{}とかも受ける。
参考：https://en.cppreference.com/cpp/algorithm/nth_element
計算量は時間平均O(M)で空間O(M)。最悪だと時間O(M^2)になるらしい。[1,2,3,4,5]でピボットが5だと1~4と5比較して4回。4がピボットになり3回……。
これを繰り返すと初項(N-1)の等差数列でN(N-1)/2回の探索が必要。でN^2がでる。
