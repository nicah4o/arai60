問題：https://leetcode.com/problems/number-of-islands/

## step1

問題は
　1.1と0が要素のm行n列の行列が与えられ（二次元配列ではなくvector<vector<char>>）、
　2.1を島として0を海とする。
　3.1が上下左右につながるとき、それを一つの島と数える（斜めに繋がっているだけでは一つの島でなく二つの島）
　4.島の個数を返せ
というもの。

分からなかったのでleetcodeのdiscussionを読みながらやった。これが参考になった。
参考：https://leetcode.com/problems/number-of-islands/description/comments/1570785/

やり方としては
　1.visit関数をラムダ式でnumIslands関数の中に書く
　2.この関数はある島の'1'を発見したら島を全て'0'に書き換えるという沈没関数。
　3.行列を走査するときにこの関数を呼び出した回数を返り値とする
というDFSの方法。

時間計算量は二重ループでO(NM)になる。

```cpp
class Solution {
public:
    int numIslands(vector<vector<char>>& grid) {
        auto visit = [&](auto& self, int x, int y) -> void {
            if (0 <= x && x < grid[0].size() && 0 <= y && y < grid.size() &&
                grid[y][x] == '1') {
                grid[y][x] = '0';
                self(self, x + 1, y);
                self(self, x - 1, y);
                self(self, x, y + 1);
                self(self, x, y - 1);
            }
        };
        int count = 0;
        for (int i = 0; i < grid.size(); i++) {
            for (int j = 0; j < grid[0].size(); j++) {
                if (grid[i][j] == '1') {
                    visit(visit, j, i);
                    count++;
                }
            }
        }
        return count;
    }
};
```

## step2

コメント集と他の方のコードを見た。
この方のを参考にBFSでもやってみる。
参考：https://github.com/attractal/leetcode/pull/8/changes

