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

方法としては
　1.m行n列のbool配列visitedを作る。falseで初期化して上陸したらtrueにする。
　2.ラムダ式である配列の位置を引数にして、その位置から繋がる島をvisitedにより記録する。
　3.ラムダ式の中身はBFSつまり最初の位置から上下左右に島がありそれがvisitedでないならキューに積んでゆく。キューがなくなる、つまり島のどの位置の上下左右にも島の続きがない場合にループが終わる（＝島の個数が一つ増える）。

 ```cpp
class Solution {
public:
    int numIslands(vector<vector<char>>& grid) {
        int island_count = 0;
        int rows = grid.size();
        int columns = grid[0].size();
        vector<vector<bool>> visited(rows, vector<bool>(columns, false));
        int directions[4][2] = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
        auto visit = [&](int row, int column) {
            queue<pair<int, int>> islands;
            visited[row][column] = true;
            islands.push({row, column});
            while (!islands.empty()) {
                auto [row, column] = islands.front();
                islands.pop();
                for (int i = 0; i < 4; i++) {
                    int y = row + directions[i][0];
                    int x = column + directions[i][1];
                    if (0 <= y && y < grid.size() && 0 <= x &&
                        x < grid[0].size() && grid[y][x] == '1' &&
                        !visited[y][x]) {
                        islands.push({y, x});
                        visited[y][x] = true;
                    }
                }
            }
        };
        for (int m = 0; m < grid.size(); m++) {
            for (int n = 0; n < grid[0].size(); n++) {
                if (grid[m][n] == '1' && !visited[m][n]) {
                    island_count++;
                    visit(m, n);
                }
            }
        }
        return island_count;
    }
};
```

こちらは空間計算量もO(NM)になる。DFSはO(1)になるから問題の性質上走査するのでDFSが望ましいと思う。
