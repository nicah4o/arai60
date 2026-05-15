問題：https://leetcode.com/problems/intersection-of-two-arrays/description/

## step1

問題は
1.nums1とnums2という二つの並べられていないint型配列が与えられ、
2.二つに共通する値を配列にして返す
というもの。

最初に考えるのは全件走査することで、二重のforループでO(NM)の時間計算量になるが、constrainsより
> 1 <= nums1.length, nums2.length <= 1000

なので最大でもNM=1000000でたぶん行ける。前に10000×10000の全件走査はMLEが先に失敗したので、TLEはもう少しいけるという考え。
ただいい方法が思いついたのでそちらを実装。

手でやる場合、要素の重複でなく値の重複をみるので（[1,1],[1,1]でも返すのは[1]のみ）、最初にnums1,nums2の数字の重複をなくすべきだと考えた。
unordered_setは要素の重複を認めないので今回の趣旨に合っている。
その上でnums1,nums2を昇順のpriority_queueに入れてtop()を比較して同じなら通過、違うなら大きい方を残して小さい方をpop()することも考えた。
しかし、1.two sumのようにnums1のみunordered_setに入れてnums2の要素を含むか検索すればよいのだと分かった。

```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        unordered_set<int> num1s;
        for (int num : nums1) {
            num1s.insert(num);
        }
        vector<int> ans;
        for (int num : nums2) {
            if (num1s.contains(num)) {
                ans.push_back(num);
                num1s.erase(num);
            }
        }
        return ans;
    }
};
```

空間計算量はO(k)で時間計算量はO(N+M)。kはnums1にある数字の種類数。

## step2

他の方のコードとコメント集を見た。この問題の題意はここからnums1とnums2の条件が変わったときにどのように解くかという点にあるとのこと。
参考：https://github.com/attractal/leetcode/pull/17/changes
nums1が大きく、ソートされていてnums2が小さいとき。

```cpp
class Solution {
public:
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        sort(nums1.begin(), nums1.end());
        sort(nums2.begin(), nums2.end());
        vector<int> intersections;
        for (int i = 0; i < nums2.size(); ++i) {
            if (i > 0 && nums2[i] == nums2[i - 1]) {
                continue;
            }
            int target = nums2[i];
            int left = -1;
            int right = nums1.size();
            bool found = false;
            while (right - left > 1) {
                int middle = left + (right - left) / 2;
                if (nums1[middle] == nums2[i]) {
                    intersections.push_back(nums2[i]);
                    break;
                }
                if (nums1[middle] < nums2[i]) {
                    left = middle;
                } else {
                    right = middle;
                }
            }
        }
        return intersections;
    }
};
```
うーん結局どちらもソートしてしまうことになった。
計算量がいくつになるのか分からない。最初のソートでO(NlogN+MlogM)で以降のバイナリサーチでO(MlogN)だと思う。
