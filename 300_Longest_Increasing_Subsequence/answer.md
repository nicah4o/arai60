問題： https://leetcode.com/problems/longest-increasing-subsequence/description/

## step1

問題は
　1.整数の配列numsを与えられ、
　2.numsから要素を削除して昇順の数字の配列を作った時に(subsequece)、そのような配列のなかで最長のものの要素数を返せ。
というもの。

考え
　1.例えば[5,6,2,3,4]のときは最初[5,6]を保持していても[2]が現れた時点で[2]から繋がる方向性も維持する必要がある。つまり配列を複数持つ必要がある。
　2.その上で要素が一つの配列と複数の配列の扱いは異なる。
　3.例えば[10,9,2,5,3]のときに[10]は要素が一つなので自分より小さい[9]が出た時点で保持する必要がない。[9]も[2]が出たら捨てることができる。
　4.しかし[5,6,2,3,4]の場合は[5,6]を捨てることができるのは[2,3]の要素数が2になる時点。
　5.一般的には要素数が同じ配列が現れた時点で最後の要素の大きさを比べて小さいほうのみ生かすということができる。
　6.また、push_backの条件も複数要素の配列と単独要素の配列では異なる。以下はnumsの要素への対応。
　7.単独要素配列なら、自分より小→それが新たな単独要素配列になる。自分より大→push_backできる。
　8.複数要素配列なら、最後の要素より小かつ最後から二番目の要素より大→最後の要素をそれにする。最後の要素より大→push_backする。

↓ただsizeの比較とかが上手く実装できず断念した。

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<vector<int>> subsequences;
        for (int num:nums) {
            for (vector<int> sub:subsequences) {
                int size = sub.size();
                int last = sub.back();
                int next_to_last = size >1 ? sub[size - 2] : numeric_limits<int>::min();
                if (last < num) {
                    sub.push_back(num);
                }
                else if (next_to_last < num && num < last) {
                    sub[size - 1] = num;
                }
            }
        }
        int max_size = 0;
        for (vector<int> sub:subsequences) {
            int size = sub.size();
            max_size = max(size, max_size);
        }
        return max_size;
    }
};
```

## step2

leetcodeのSolutionをみた。
- https://leetcode.com/problems/longest-increasing-subsequence/solutions/6092590/video-keep-elements-in-ascending-order-b-m5w0/
binary searchのdpの方法

時間計算量：O(NlogN)
```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> subsequence;
        for (int num:nums) {
            if (subsequence.empty() || subsequence.back() < num) {
                subsequence.push_back(num);
            }
            else {
                int idx = binarySearch(subsequence, num);
                subsequence[idx] = num;
            }
        }
        return subsequence.size();
    }
private:
    int binarySearch(const vector<int>& array, int target) {
        int left = 0;
        int right = array.size() - 1;
        while (left <= right) {
            int mid = (left + right) / 2;
            if (array[mid] == target) {
                return mid;
            }
            else if (array[mid] > target) {
                right = mid - 1;
            }
            else {
                left = mid + 1;
            }
        }
        return left;
    }
};
```

解き方
　・例えば[5,6,7,2,3]において、[5,6,7]という配列を保持する必要はない。5を2にすれば[2,6,7]で昇順のまま最大サイズも変更していない。そのまま配列を格納していくと、[2,3,7]でサイズは3でこれを返せばよい。
　・ただ、新しい値を格納できる条件は元の昇順を壊さないこと。新しい値をどこに格納するかは二分探索法を用いる。

copilotのdp
　・lower_boundという関数で指定した範囲でnum以上の数が初めて現れる場所を検索している。
 
```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> dp;
        for (int num : nums) {
            auto it = lower_bound(dp.begin(), dp.end(), num);
            if (it == dp.end()) {
                dp.push_back(num);
            } else {
                *it = num;
            }
        }
        return dp.size();
    }
};
```

- https://github.com/TORUS0818/leetcode/pull/33/changes
思い付きやすいdpとして紹介されていたのをやってみた。

時間計算量：O(N^2)
```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int n = nums.size();
        vector<int> dp_list(n, 1);
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[j] < nums[i]) {
                    dp_list[i] = max(dp_list[i], dp_list[j] + 1);
                }
            }
        }
        return *max_element(dp_list.begin(), dp_list.end());
    }
};
```

その他の解法は分からなかった。BIT, セグメントツリー, 座標圧縮という言葉があるようだ。
