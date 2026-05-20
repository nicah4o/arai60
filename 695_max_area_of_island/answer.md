問題：https://leetcode.com/problems/max-area-of-island/description/

## step1

問題は
　1.vector<vector<int>>のm行n列の配列に1,0が入っている。
　2.1を島とし、0を海とする。島は上下左右につながって一つの島として数える。斜めにつながる島は一つの島と数えない。
　3.最も大きい島に何個の1が含まれるか、面積を答えよ
というもの。

200.number of islandsで使用したのと同様のDFSの手段を使用したがデータの破壊を避けるために島を沈没させずにvisitedにて訪問を記録した。
　1.まずcount_island関数を作る。これは配列の(i,j)地点を与えられると、それが島なら一続きの島の面積を返し、海なら0を返す。
　2.これの中身は再帰関数になっており、訪問記録visitedと配列gridを(i,j)と共に引数にする。もし(i,j)地点が島なら、(i,j)をvisitedに追加し、(i,j)の四方一マスについて自分自身を呼び出して探索する。
　3.再帰の一回ごとに現在地点が島ならカウンターに+1をする。島の面積がわかるのであとはgridを走査する。

時間計算量は走査するのでO(MN)で、空間計算量はO(MN)である。スタック領域に再帰が積み重なることで最悪の場合はgridの要素一つ一つに対してメモリを使用する。

```cpp
class Solution {
private:
    int count_island(vector<vector<bool>>& visited,
                     const vector<vector<int>>& grid, int i, int j) {
        int count = 0;
        if (0 <= i && i < grid.size() && 0 <= j && j < grid[0].size() &&
            grid[i][j] == 1 && !visited[i][j]) {
            visited[i][j] = true;
            count = 1 + count_island(visited, grid, i - 1, j) +
                    count_island(visited, grid, i + 1, j) +
                    count_island(visited, grid, i, j - 1) +
                    count_island(visited, grid, i, j + 1);
        }
        return count;
    }

public:
    int maxAreaOfIsland(vector<vector<int>>& grid) {
        int m = grid.size();
        int n = grid[0].size();
        vector<vector<bool>> visited(m, vector<bool>(n, false));
        int count = 0;
        for (int i = 0; i < grid.size(); i++) {
            for (int j = 0; j < grid[0].size(); j++) {
                if (grid[i][j] == 1 && !visited[i][j]) {
                    int temp = count_island(visited, grid, i, j);
                    count = (count < temp) ? temp : count;
                }
            }
        }
        return count;
    }
};
```

## step2

他の方のコードとコメント集みた。
以下の方を参考にユニオンファインドやってみる。
参考：https://github.com/quinn-sasha/leetcode/pull/30/changes

```cpp
class UnionFind {
    vector<int> parent;
    vector<int> size;

public:
    UnionFind(int maxsize) : parent(maxsize), size(maxsize, 1) {
        for (int i = 0; i < maxsize; i++) {
            parent[i] = i;
        }
    }
    int find(int x) {
        if (parent[x] == x) {
            return x;
        }
        return parent[x] = find(parent[x]);
    }
    int unite(int x, int y) {
        int x_root = find(x);
        int y_root = find(y);
        if (x_root == y_root) {
            return x_root;
        }
        if (size[x_root] < size[y_root]) {
            swap(x_root, y_root);
        }
        parent[y_root] = x_root;
        size[x_root] += size[y_root];
        return x_root;
    }
    int get_size(int x) { return size[find(x)]; }
};
class Solution {
public:
    int maxAreaOfIsland(vector<vector<int>>& grid) {
        int num_rows = grid.size();
        int num_cols = grid[0].size();
        UnionFind uf(num_rows * num_cols);
        int max_area = 0;
        for (int row = 0; row < num_rows; row++) {
            for (int col = 0; col < num_cols; col++) {
                if (grid[row][col] == 0) {
                    continue;
                }
                max_area = max(max_area, 1);
                int id = row * num_cols + col;
                if (col + 1 < num_cols && grid[row][col + 1] == 1) {
                    int root = uf.unite(id, row * num_cols + col + 1);
                    max_area = max(max_area, uf.get_size(root));
                }
                if (row + 1 < num_rows && grid[row + 1][col] == 1) {
                    int root = uf.unite(id, (row + 1) * num_cols + col);
                    max_area = max(max_area, uf.get_size(root));
                }
            }
        }
        return max_area;
    }
};
```

ユニオンファインドは
　1.親をそれぞれの配列の要素が持ち、その親の親の親……と累進した最後の場所の親であるrootをある島の代表として扱う。
　2.これがどのように決められるかというと横の走査と縦の走査を行うことによる。隣の一つも島である場合、自分と隣のrootを比較してより多くの子供がいる方にuniteしてrootを一種類にする。
　3.島のサイズはrootに紐づいて管理される。
