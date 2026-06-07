```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> dp(nums.size(), 1);
        for (int i = 0;i < nums.size();i++) {
            for (int j = 0;j < i;j++) {
                if (nums[j] < nums[i]) {
                    dp[i] = max(dp[i], dp[j]+1);
                }
            }
        }
        return *max_element(dp.begin(), dp.end());
    }
};
```

copilot指摘
　・const size_t n = nums.size()としてもよかった。毎回sizeを読む必要はないから。
　・dpでなくlist_lengthsのほうが良いかもしれない。ただ、この問題の命名は難しいとの議論があった。
　・std::使用を検討。

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
    int binarySearch(vector<int> array, int target) {
        int left = 0;
        int right = array.size() - 1;
        while (left <= right) {
            int mid = (left + right) / 2;
            if (array[mid] == target) {
                return mid;
            }
            else if (target < array[mid]) {
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

copilot指摘
　・int BinarySearch(const std::vector<int>& array, int target)として参照渡ししないと毎回コピーしている。
　・メソッド名は lowerCamelCase か PascalCaseで統一
　・int mid = left + (right - left) / 2;とすることでオーバーフロー回避できる。なぜこうするのかわからなかったがいきなり和はオーバーフローの可能性がある。

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        vector<int> subsequence;
        for (int num:nums) {
            auto it = lower_bound(subsequence.begin(), subsequence.end(), num);
            if (subsequence.empty() || subsequence.back() < num) {
                subsequence.push_back(num);
            }
            else {
                *it = num;
            }
        }
        return subsequence.size();
    }
};
```

copilot指摘
　・itは(subsequence.empty() || subsequence.back() < num)のときsubsequence.end()になるからそれが条件でよい。
