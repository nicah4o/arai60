問題：https://leetcode.com/problems/subarray-sum-equals-k/description/

## step1

問題は、
　1.int型配列numsと整数kが与えられ、
　2.numsの部分配列（連続した空でない部分。インデックス1,2はよいが1,3は連続してないのでダメ）の合計がkになるものの数を求めよ
というもの。ちなみにkは負の値もとる。

思いつかなかったのでO(N^2)であるブルートフォース（ヒント1を見た）を実行。
> 1 <= nums.length <= 2 * 10^4

なので、O(N^2)は最大でも4億ステップに収まり、cppだと1億～10億ステップ/秒なので、4～0.4秒に収まる。
参考：https://github.com/Yuto729/leetcode/pull/16#discussion_r2602118324

N個の要素一つ一つにつきその位置iの次の位置jを足す。jはループで走査しつつsumに足していく。kになった時にのみカウントを増やす。

```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int count = 0;
        for (int i = 0; i < nums.size(); i++) {
            int sum = 0;
            sum += nums[i];
            if (sum == k) {
                count++;
            }
            for (int j = i + 1; j && j < nums.size(); j++) {
                sum += nums[j];
                if (sum == k) {
                    count++;
                }
            }
        }
        return count;
    }
};
```

結果として1937msでアクセプト。

## step2

解答とコメント集みた。
なるほど、記録しておくのは部分配列の中でもnumsの先頭からの要素が0個からN個に至るまでのN+1個でいいのか。
で、それらの差集合を取ると[1,1,2,5]でk=7の時は[1,1,2,5]-[1,1]=[2,5]などを回収できるという。
実際にはunordered_mapを使うので和の差をとり9-2=7とする。つまり今までの部分配列に2が存在したらカウントする。

```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        unordered_map<int, int> sum_to_count;
        sum_to_count[0] = 1;
        int ans = 0;
        int sum = 0;
        for (int num : nums) {
            sum += num;
            if (sum_to_count.contains(sum - k)) {
                ans += sum_to_count[sum - k];
            }
            sum_to_count[sum]++;
        }
        return ans;
    }
};
```

これだと計算量は時間O(N)でいける。一回の走査で済むから。
あと、cppだとint ans,sum;となげやりに書くとスタック領域を使いまわしている為に0を代入してくれず前に使われたデータの残骸が代入されるとのこと（gemini）。
