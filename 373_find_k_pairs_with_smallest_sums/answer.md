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
不安になったのはnums1[n+1]+nums2[n]がnums1[n]+nums2[n+1]より小さいというような場合を見落としてるんじゃないかということ。
だからfor文を二回回した。

正解の一つは、nums1[n]+nums2[0]をk個集めたpriority_queueからtop()を取得していくたびにnums1[n]+nums2[1],nums1[n]+nums2[2],というようにnums2の添え字を増やしたものをpriority_queueに追加していく。
この方のRPを参考にした。
参考：https://github.com/attractal/leetcode/pull/13/changes

```cpp
class Solution {
public:
    vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2,
                                       int k) {
        using tu = tuple<int, int, int>;
        std::priority_queue<tu, vector<tu>, greater<tu>> sum_to_pair;
        vector<vector<int>> ans;
        int j;
        k < nums1.size() ? j = k : j = nums1.size();
        for (int i = 1; i <= j; i++) {
            sum_to_pair.push({nums1[i - 1] + nums2[0], i - 1, 0});
        }
        while (!sum_to_pair.empty() && ans.size() < k) {
            auto [_, idx1, idx2] = sum_to_pair.top();
            sum_to_pair.pop();
            ans.push_back({nums1[idx1], nums2[idx2]});
            if (idx2 + 1 < nums2.size()) {
                sum_to_pair.push(
                    {nums1[idx1] + nums2[idx2 + 1], idx1, idx2 + 1});
            }
        }
        return ans;
    }
};
```
時間計算量がO(klog(min(n, k)))で空間計算量がO(min(n, k))。

追記：5/13　最初の試みの改善案

```cpp
class Solution {
public:
    vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2,
                                       int k) {
        using pi = pair<int, vector<int>>;
        bool flag = false;
        priority_queue<pi, vector<pi>, greater<pi>> sum_to_pair;
        int i = 1;
        for (int j : nums1) {
            for (int k : nums2) {
                sum_to_pair.push({j + k, {j, k}});
                i++;
                if (i == k) {
                    flag = true;
                    break;
                }
            }
            if (flag) {
                break;
            }
        }

        flag = false;
        i = 1;
        for (int j : nums2) {
            for (int k : nums1) {
                sum_to_pair.push({j + k, {k, j}});
                i++;
                if (i == k) {
                    flag = true;
                    break;
                }
            }
            if (flag) {
                break;
            }
        }

        i = 1;
        vector<vector<int>> result;
        while (!sum_to_pair.empty() && i <= k) {
            result.push_back(sum_to_pair.top().second);
            i++;
            sum_to_pair.pop();
        }
        return result;
    }
};
```
