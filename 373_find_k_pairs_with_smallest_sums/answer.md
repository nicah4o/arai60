問題：https://leetcode.com/problems/find-k-pairs-with-smallest-sums/description/

## step1

問題は、
1.nums1とnums2という二つの昇順に並べられたvector<int>型配列と整数kが与えられ、
2.それら二つの配列から値を一つずつ取得してその和が小さい順にk個、値のペアをreturnせよ。
というもの。

最初思いついたのはpriority_queueに{和, [値1, 値2]}というpairの形でnum1とnum2のすべての組み合わせを与えて小さい順にk個呼び出すというもの。
計算量は酷く、要素数nums1がN個でnum2がM個だと空間計算量がO(NM)とかになる。
実装したが、やはり計算量で無理だった。memory limit exceeded。

次に思いついたのはpriority_queueに和が小さいk個を入れるというもの。ただし、樹形図をnums1からnums2に描いたときのk個とnums2からnums1に描いたときのk個の2k個を集計した。
二重for文を異なる順序で二回書いたということですね。これなら空間計算量はO(2k)になるから大丈夫だと思ったがやはり同じ理由で無理だった。

```cpp
class Solution {
public:
    vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2,
                                       int k) {
        using pi = pair<int, vector<int>>;
        priority_queue<pi, vector<pi>, greater<pi>> sum_to_pair;
        int i = 1;
        for (int j : nums1) {
            for (int k : nums2) {
                sum_to_pair.push({j + k, {j, k}});
                i++;
                if (i == k) {
                    break;
                } else {
                    continue;
                }
                break;
            }
        }
        i = 1;
        for (int j : nums2) {
            for (int k : nums1) {
                sum_to_pair.push({j + k, {k, j}});
                i++;
                if (i == k) {
                    break;
                } else {
                    continue;
                }
                break;
            }
        }
        i = 1;
        vector<vector<int>> result;
        pair<int, vector<int>> duplicate;
        while (i <= k) {
            if (duplicate == sum_to_pair.top()) {
                sum_to_pair.pop();
            }
            result.push_back(sum_to_pair.top().second);
            i++;
            duplicate = sum_to_pair.top();
            sum_to_pair.pop();
        }
        return result;
    }
};
```

## step2
leetcodeの解答みた。自分が考えるのを逃げてた部分が分かった。
具体的に言うと昇順に並べられてるという条件について。これってどういうことかと言えば、nums1[n] + nums2[n]は常にnums1[n+1] + nums2[n+1]よりも小さいか同じということです。
自分としてはk個必要なら最低k回はループ回す必要がある、というような思考で空間O(2k)の方法をしていた。
ただ不安になるのはnums1[n]+nums2[n+2]がnums1[n+1]+nums2[n+1]より小さいというような場合を見落としてるんじゃないかということ。
もしこれがn=k-1回目のとこに位置していたら
