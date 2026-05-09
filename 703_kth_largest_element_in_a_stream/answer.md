問題：https://leetcode.com/problems/kth-largest-element-in-a-stream/description/

## step1
二つ前のスタックの問題の時にそもそもスタックの使い方よくわかっていなかった反省からpriority_queueについても調べた。
geminiに聞きながら解いたので初見で解くというstep1の意味がなくなってしまった（反省）。
普通に書くと降順に処理する（.top()で一番大きいものがでてくる）。
k番目を知りたいので、.top()がk番目であるとよい。つまり、昇順で要素数がk個のpriority_queueを作ればよい。

```cpp
class KthLargest {
public:
        priority_queue<int,vector <int>,greater<int>> q;
        int k_val;
    KthLargest(int k, vector<int>& nums) {
        k_val = k;
        for (int i:nums) {
            q.push(i);
        }
        int add(int val);
    }
    
    int add(int val) {
        q.push(val);
        while (q.size() != k_val) {
            q.pop();
        }
        return q.top();
    }
};
```
これがacceptされた。時間計算量はinitがO(NlogN)でaddがO(logk)となる。空間計算量はO(n)。

## step2
コメント集ではcppならばpriority_queueではなくmapでの実装をまず考えるとの指摘があった。
参考：https://github.com/Ryotaro25/leetcode_first60/pull/9#discussion_r1619710596

```cpp
class KthLargest {
    map<int,int> numbers;
    int k_val;
public:
    KthLargest(int k, vector<int>& nums) {
        k_val = k;
        for (int i : nums) {
            if (numbers.contains(i)) {
                numbers[i]++;
            }
            else {
                numbers[i] = 1;
            }
        }
        int add(int val);
    }
    
    int add(int val) {
        if (numbers.contains(val)) {
            numbers[val]++;
        }
        else {
            numbers[val] = 1;
        }
        std::map<int, int>:: reverse_iterator j;
        j = numbers.rbegin();
        int i = 1;
        while (i+j->second <= k_val) {
            i += j->second;
            j++;
        }
        return j->first;
    }
};
```
一応こんな感じでmapを使用して書いた。この場合は計算量はinitが時間O(nlogn)でaddがO(n)。空間計算量もO(n)。
