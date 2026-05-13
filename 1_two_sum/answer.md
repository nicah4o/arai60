問題：https://leetcode.com/problems/two-sum/description/

## step1
問題は
1.numsという配列と整数targetを与えられる。
2.numsの中から二つの値の和がtargetになる組を見つけてreturnする。この組は一つしかない。
というもの。

arai60の分類だとhashmapに該当するのだが、どこでmapを使うべきかわからなかった。
今までの問題だとsum_to_pairのmapを作ることが多かった。
前回の問題でも(https://leetcode.com/problems/find-k-pairs-with-smallest-sums/) 、そのようなsum_to_pairで全走査する方式を使って上手くいかなかったが思いつかなかったので今回も全走査した。

ただ、返り値が一意に定まるという条件があるので、mapは作らずに合計値がtargetになる組を二重のforで探すのみにした。

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        int i, j;
        for (i = 0; i <= (nums.size() - 1); i++) {
            for (j = i + 1; j <= (nums.size() - 1); j++) {
                if (nums[i] + nums[j] == target) {
                    vector<int> ans = {i, j};
                    return ans;
                }
            }
        }
        return {};
    }
};
```

時間計算量がO(N^2)で空間計算量はO(1)。acceptされたが43msで最頻値が0ms,2msなので20倍以上遅い解法である。

## step2
leetcodeの解法見た。unordered_mapをsum_to_pairでなくnum_to_indexで作る。
targer-nums[i] をnum_to_indexで検索すると一瞬で答えがわかる。
この問題で真に求めたいのはペアの数字の値じゃなくてインデックスであるということを分からないと解けない。

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> num_to_index;
        for (int i = 0; i < nums.size(); i++) {
            int complement = target - nums[i];
            if (num_to_index.contains(complement)) {
                vector<int> ans = {i, num_to_index[complement]};
                return ans;
            }
            else {
                num_to_index.insert({nums[i], i});
            }
        }
        return {};
    }
};
```

時間計算量はO(N)で空間計算量もO(N)。
実装では特に先にmapにメモして探すのではなく探してからメモするという仕組みが思いつかなかった。
