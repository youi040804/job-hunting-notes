### 一、识别特征

看到：

- 二维网格；
- 上下左右相邻；
- 连通区域；
- 岛屿数量；
- 某个区域面积；
- 判断某块区域是否连通。

优先想到：

```text
DFS / BFS + visited
```

### 二、核心思想

遍历整个网格。

每遇到一个没有访问过的目标格子，就说明发现了一个新的连通块。

然后从这个位置进行 DFS，把整个连通块访问完。

```text
找到未访问节点
        ↓
记录一个新的连通块
        ↓
DFS 访问所有相邻节点
        ↓
继续扫描网格
```

### 三、DFS 基本模板

```cpp
void dfs(vector<vector<char>>& grid, int i, int j) {
    if (i < 0 || i >= grid.size() ||
        j < 0 || j >= grid[0].size()) {
        return;
    }

    if (grid[i][j] == '0') {
        return;
    }

    grid[i][j] = '0';

    dfs(grid, i - 1, j);
    dfs(grid, i + 1, j);
    dfs(grid, i, j - 1);
    dfs(grid, i, j + 1);
}
```

主循环：

```cpp
for (int i = 0; i < grid.size(); i++) {
    for (int j = 0; j < grid[0].size(); j++) {
        if (grid[i][j] == '1') {
            count++;
            dfs(grid, i, j);
        }
    }
}
```

### 四、关键理解

访问过的节点必须立刻标记，否则可能重复访问甚至无限递归。

如果题目允许修改原数组，可以直接修改：

```cpp
grid[i][j] = '0';
```

否则通常使用：

```cpp
visited[i][j] = true;
```

### 五、经典例题

- LeetCode 200 岛屿数量
- 岛屿最大面积
- 被围绕的区域