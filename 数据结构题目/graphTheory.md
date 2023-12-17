# graphTheory

## 一、深度优先搜索 理论基础

这种算法从起始节点开始，尽可能深地探索每个分支，直到无法再深入为止，然后回溯到上一层，继续探索其他分支。

在深度优先搜索中，重要的是维护一个记录已经访问过的节点的列表，以避免陷入无限循环。这种算法的一个优点是它对于解决许多问题都很简单直观，但需要注意的是，在某些情况下可能会陷入无限循环，因此需要额外的措施来处理这种情况。

dfs用的是回溯：

```java
void dfs(参数) {
    if (终止条件) {
        存放结果;
        return;
    }

    for (选择：本节点所连接的其他节点) {
        处理节点;
        dfs(图，选择的节点); // 递归
        回溯，撤销处理结果
    }
}
```



深搜三部曲：

**①确定递归函数 参数**

一般情况，深搜需要 二维数组数组结构保存所有路径，需要一维数组保存单一路径，这种保存结果的数组，我们可以定义一个全局变量，避免让我们的函数参数过多。

②确定 终止条件

③处理当前节点的出发路径



## 二、所有可能的路径

### 题目

![](https://raw.githubusercontent.com/vankykoo/javaStudy/main/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%AD%A6%E4%B9%A0%E9%A2%98%E7%9B%AE/graph/093.png)



### 思路

使用递归：

* 确定递归参数
  * 首先是整个图
  * 然后是小图
* 终止条件
  * 如果最后一个节点已经是n - 1 了，就把当前路径添加到结果集中。
* 递归逻辑
  * 遍历小图中的所有元素，先把当前元素添加到当前路径中
  * 递归调用
  * 删除当前路径的最后一个元素



### 代码

我的代码

```java
class Solution {
    List<List<Integer>> result = new ArrayList<>();
    LinkedList<Integer> path = new LinkedList<>();

    public List<List<Integer>> allPathsSourceTarget(int[][] graph) {
        path.add(0);
        path(graph,graph[0]);
        return result;
    }

    public void path(int[][] graph, int[] arr){
        if(path.getLast() == graph.length - 1){
            List<Integer> list = new LinkedList<>(path);
            result.add(list);
        }

        for(int i = 0; i < arr.length; i++){
            path.add(arr[i]);
            path(graph,graph[arr[i]]);
            path.removeLast();
        }
    }

}
```



优化，提升执行速度

```java
class Solution {
    public List<List<Integer>> allPathsSourceTarget(int[][] graph) {
        List<List<Integer>> result = new ArrayList<>();
        LinkedList<Integer> path = new LinkedList<>();
        dfs(graph, 0, path, result);
        return result;
    }

    private void dfs(int[][] graph, int node, LinkedList<Integer> path, List<List<Integer>> result) {
        //先加到当前路径中，然后马上进行判断是不是 n - 1，这样做可以减少递归层数
        path.add(node);	
        
        if (node == graph.length - 1) {
            result.add(new ArrayList<>(path));
            path.removeLast(); // 注意这里要回溯
            return;
        }

        for (int nextNode : graph[node]) {
            dfs(graph, nextNode, path, result);
        }

        path.removeLast(); // 回溯到上一个节点
    }
}
```





## 三、广度优先搜索 理论基础

广度优先搜索（BFS）是一种图遍历算法，用于从图的起始节点开始，逐层访问其邻近节点。BFS的思路是以一种“层次”的方式逐级遍历图中的节点，首先访问起始节点，然后访问与起始节点直接相邻的节点，接着是与这些相邻节点相邻的节点，以此类推。

以下是广度优先搜索的基本思路：

1. **选择起始节点：** 选择图中的一个起始节点作为搜索的起点。

2. **标记起始节点：** 将起始节点标记为已访问，并将其加入队列（或其他数据结构）中。

3. **遍历队列：** 进入循环，不断从队列中取出节点，访问该节点，并将其所有未访问的相邻节点加入队列。这确保了先访问当前层级的所有节点，再进入下一层级。

4. **标记已访问节点：** 在将节点加入队列之前，标记该节点为已访问，以防止重复访问。

5. **重复步骤3和步骤4，直到队列为空：** 不断重复上述步骤，直到队列为空。这意味着图中的所有可达节点都已经被访问。

BFS通常使用队列来实现，因为队列遵循先进先出（FIFO）的原则，符合BFS的层次遍历性质。这确保了在同一层级上的节点先被访问，然后才是下一层级的节点。

BFS常用于解决图的最短路径问题、连通性问题，以及其他需要按层次遍历的场景。其时间复杂度通常是O(V + E)，其中V是图中的顶点数，E是边数。



### 代码

```java
public class BFS {

    static int[][] dir = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}}; // 方向数组，表示上下左右四个方向

    static void bfs(char[][] grid, boolean[][] visited, int x, int y) {
        int rows = grid.length;
        int cols = grid[0].length;

        Queue<int[]> queue = new LinkedList<>(); // 定义队列
        queue.add(new int[]{x, y}); // 起始节点加入队列
        visited[x][y] = true; // 只要加入队列，立刻标记为访问过的节点

        while (!queue.isEmpty()) { // 开始遍历队列里的元素
            int[] cur = queue.poll(); // 从队列取元素
            int curx = cur[0];
            int cury = cur[1]; // 当前节点坐标

            for (int i = 0; i < 4; i++) { // 开始向当前节点的四个方向左右上下遍历
                int nextx = curx + dir[i][0];
                int nexty = cury + dir[i][1]; // 获取周边四个方向的坐标

                if (nextx < 0 || nextx >= rows || nexty < 0 || nexty >= cols) continue;  // 坐标越界了，直接跳过

                if (!visited[nextx][nexty] && grid[nextx][nexty] == '1') { // 如果节点没被访问过且是 '1'
                    queue.add(new int[]{nextx, nexty});  // 队列添加该节点为下一轮要遍历的节点
                    visited[nextx][nexty] = true; // 只要加入队列立刻标记，避免重复访问
                }
            }
        }
    }

    public static void main(String[] args) {
        char[][] grid = {
                {'1', '1', '0', '0', '0'},
                {'1', '1', '0', '0', '0'},
                {'0', '0', '1', '0', '0'},
                {'0', '0', '0', '1', '1'}
        };

        int rows = grid.length;
        int cols = grid[0].length;

        boolean[][] visited = new boolean[rows][cols];
      
        int count = 0;

        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (!visited[i][j] && grid[i][j] == '1') {
                    bfs(grid, visited, i, j);
                    count++;
                }
            }
        }

        System.out.println("Number of Islands: " + count);
    }
}
```



## 四、岛屿数量----深搜版

### 题目

![111.png (757×1056) (raw.githubusercontent.com)](https://raw.githubusercontent.com/vankykoo/image/main/111.png)

### 思路

1. `numIslands` 方法是主要的入口点，用于遍历整个二维字符数组，当发现未访问的陆地（'1'）时，调用 `dfs` 方法，并将岛屿数量增加。
2. `dfs` 方法是深度优先搜索的实现，用于从给定的坐标 `(x, y)` 开始，递归地遍历与之相连的陆地，并将其标记为已访问。
3. `checked` 数组用于记录某个位置是否被访问过，避免重复访问。
4. `dir` 数组定义了四个方向的偏移量，用于在DFS中移动到相邻的位置。
5. 在 `dfs` 方法中，首先检查终止条件，如果当前位置已经被访问过或者是水域（'0'），则直接返回。否则，将当前位置标记为已访问，然后遍历当前位置的四个相邻位置，递归调用 `dfs`。



### 代码

```java
class Solution {
    // 记录已访问过的位置
    boolean[][] checked;
    // 定义四个方向的偏移量
    int[][] dir = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};

    public int numIslands(char[][] grid) {
        // 初始化已访问数组
        checked = new boolean[grid.length][grid[0].length];
        // 记录岛屿数量
        int result = 0;

        // 遍历整个二维数组
        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[0].length; j++) {
                // 如果当前位置未访问且是陆地，进行搜索
                if (!checked[i][j] && grid[i][j] == '1') {
                    dfs(grid, i, j);
                    result++;
                }
            }
        }

        return result;
    }

    // 深度优先搜索
    public void dfs(char[][] grid, int x, int y) {
        // 终止条件
        if (checked[x][y] || grid[x][y] == '0') {
            return;
        }

        // 标记当前位置为已访问
        checked[x][y] = true;

        // 遍历当前位置的四个相邻位置
        for (int i = 0; i < 4; i++) {
            int nextX = x + dir[i][0];
            int nextY = y + dir[i][1];

            // 检查相邻位置是否合法，如果越界则继续下一次循环
            if (nextX < 0 || nextX >= grid.length || nextY < 0 || nextY >= grid[0].length) {
                continue;
            }

            // 递归调用深度优先搜索
            dfs(grid, nextX, nextY);
        }
    }
}
```



## 五、岛屿数量----广搜版

### 思路

遍历grid数组，如果有陆地没有被走过（visited == false)  的，进行广搜，然后岛屿数量加一。



### 代码

```java
class Solution {
    // 用于标记是否访问过某个位置
    boolean[][] visited;
    
    // 方向数组，表示上、右、下、左四个方向
    int[][] dir = {{-1, 0}, {0, 1}, {1, 0}, {0, -1}};

    // 主函数，计算岛屿数量
    public int numIslands(char[][] grid) {
        // 初始化岛屿数量为0
        int result = 0;

        // 初始化visited数组，标记是否访问过每个位置
        visited = new boolean[grid.length][grid[0].length];

        // 遍历整个二维数组
        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[0].length; j++) {
                // 如果当前位置未被访问且为岛屿（'1'表示岛屿）
                if (!visited[i][j] && grid[i][j] == '1') {
                    // 进行广度优先搜索，并将岛屿数量增加
                    bfs(grid, i, j);
                    result++;
                }
            }
        }

        // 返回最终的岛屿数量
        return result;
    }

    // 辅助函数，实现广度优先搜索
    public void bfs(char[][] grid, int x, int y) {
        // 使用队列存储待访问的位置
        Queue<int[]> queue = new LinkedList<>();
        
        // 将当前位置加入队列，并标记为已访问
        queue.add(new int[]{x, y});
        visited[x][y] = true;

        // 遍历队列中的位置
        while (!queue.isEmpty()) {
            // 取出队列头部的位置
            int[] cur = queue.poll();
            int curX = cur[0];
            int curY = cur[1];

            // 遍历当前位置的上、右、下、左四个方向
            for (int i = 0; i < 4; i++) {
                int nextX = curX + dir[i][0];
                int nextY = curY + dir[i][1];

                // 检查下一个位置是否越界
                if (nextX < 0 || nextX >= grid.length || nextY < 0 || nextY >= grid[0].length) {
                    continue;
                }

                // 如果下一个位置未被访问且为岛屿，加入队列并标记为已访问
                if (!visited[nextX][nextY] && grid[nextX][nextY] == '1') {
                    queue.add(new int[]{nextX, nextY});
                    visited[nextX][nextY] = true;
                }
            }
        }
    }
}

```





## 六、岛屿的最大面积

### 题目

![114.png (850×1443) (raw.githubusercontent.com)](https://raw.githubusercontent.com/vankykoo/image/main/114.png)



### 思路

使用深搜广搜都可以，在每次搜索的时候记录岛屿数量，取最大数量即可。



### 代码

```java
class Solution {
    // 记录已访问过的位置
    boolean[][] checked;
    // 定义四个方向的偏移量
    int[][] dir = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    //记录每次搜索的岛屿的面积
    int path = 0;

    public int maxAreaOfIsland(int[][] grid) {
        // 初始化已访问数组
        checked = new boolean[grid.length][grid[0].length];
        // 记录最大岛屿面积
        int result = 0;

        // 遍历整个二维数组
        for (int i = 0; i < grid.length; i++) {
            for (int j = 0; j < grid[0].length; j++) {
                path = 0;
                // 如果当前位置未访问且是陆地，进行搜索
                if (!checked[i][j] && grid[i][j] == 1) {
                    dfs(grid,i,j);
                    result = Math.max(result, path);
                }
            }
        }

        return result;
    }

    // 深度优先搜索
    public void dfs(int[][] grid, int x, int y) {
        // 终止条件
        if (checked[x][y] || grid[x][y] == 0) {
            return;
        }

        // 标记当前位置为已访问
        checked[x][y] = true;
        path++;
        // 遍历当前位置的四个相邻位置
        for (int i = 0; i < 4; i++) {
            int nextX = x + dir[i][0];
            int nextY = y + dir[i][1];

            // 检查相邻位置是否合法，如果越界则继续下一次循环
            if (nextX < 0 || nextX >= grid.length || nextY < 0 || nextY >= grid[0].length) {
                continue;
            }

            // 递归调用深度优先搜索
            dfs(grid, nextX, nextY);
        }
    }
}
```





## 七、飞地的数量

### 题目

![001.png (839×1445) (raw.githubusercontent.com)](https://raw.githubusercontent.com/vankykoo/image/main/pic/001.png)

### 思路

* 我的思路：
  * 深搜逻辑
  * 如果是边界岛屿，就进行标记，path 赋值为 -1，且要记录已访问过的位置
  * 如果不是边界岛屿，就一直增加岛屿面积
  * 最后把飞地面积加起来
* 答案思路，深搜是把边界岛屿都覆盖，都变成水地，最后在遍历一遍看不是水地的面积

### 代码

我的代码

```java
class Solution {
    // 记录已访问过的位置
    boolean[][] checked;
    // 定义四个方向的偏移量
    int[][] dir = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    //记录每次搜索的岛屿的面积
    int path = 0;

    public int numEnclaves(int[][] grid) {
        checked = new boolean[grid.length][grid[0].length];
        int res = 0;

        for(int i = 0; i < grid.length; i++){
            for(int j = 0; j < grid[0].length; j++){
                if (!checked[i][j] && grid[i][j] == 1){
                    path = 0;
                    dfs(grid,i,j);
                    if(path != -1){
                        res += path;
                    }
                }
            }
        }

        return res;
    }

    // 深度优先搜索
    public void dfs(int[][] grid, int x, int y) {
        // 终止条件
        if (checked[x][y] || grid[x][y] == 0) {
            return;
        }
      
		//如果是边界岛屿，标记
        if(x == 0 || y == 0 || x == grid.length - 1 || y == grid[0].length - 1){
            path = -1;
        }

        // 标记当前位置为已访问
        checked[x][y] = true;
        //如果不是边界岛屿，增加面积
        if(path != -1){
            path++;
        }
        // 遍历当前位置的四个相邻位置
        for (int i = 0; i < 4; i++) {
            int nextX = x + dir[i][0];
            int nextY = y + dir[i][1];

            // 检查相邻位置是否合法，如果越界则继续下一次循环
            if (nextX < 0 || nextX >= grid.length || nextY < 0 || nextY >= grid[0].length) {
                continue;
            }

            // 递归调用深度优先搜索
            dfs(grid, nextX, nextY);
        }
    }
}
```



答案代码

```java
class Solution {
    public int numEnclaves(int[][] grid) {
        int rows = grid.length;
        int cols = grid[0].length;

        // 将与边界相连的陆地进行 DFS，并标记为 0
        for (int i = 0; i < rows; i++) {
            if (grid[i][0] == 1) {
                dfs(grid, i, 0);
            }
            if (grid[i][cols - 1] == 1) {
                dfs(grid, i, cols - 1);
            }
        }

        for (int j = 0; j < cols; j++) {
            if (grid[0][j] == 1) {
                dfs(grid, 0, j);
            }
            if (grid[rows - 1][j] == 1) {
                dfs(grid, rows - 1, j);
            }
        }

        // 统计剩余的陆地数量
        int result = 0;
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (grid[i][j] == 1) {
                    result++;
                }
            }
        }

        return result;
    }

    // DFS 遍历相连的陆地
    private void dfs(int[][] grid, int x, int y) {
        if (x < 0 || x >= grid.length || y < 0 || y >= grid[0].length || grid[x][y] == 0) {
            return;
        }

        grid[x][y] = 0; // 标记为已访问

        // 遍历相邻的陆地
        dfs(grid, x - 1, y);
        dfs(grid, x + 1, y);
        dfs(grid, x, y - 1);
        dfs(grid, x, y + 1);
    }
}
```



## 八、被围绕的区域

### 题目

![002.png (843×1143) (raw.githubusercontent.com)](https://raw.githubusercontent.com/vankykoo/image/main/pic/002.png)

### 思路

和上一题的思路几乎一致，先把边界岛屿赋值为‘Y’，最后把不是‘Y’的赋值为‘X’，把'Y'赋值为‘O'即可。

###代码

```java
class Solution {
    int m,n;  // m为行数，n为列数
    boolean[][] checked;  // 判断该位置是否访问过，避免重复访问
    int[][] dir = {{-1,0},{1,0},{0,1},{0,-1}};  // 一个方向数组，表示向上、向下、向左、向右四个方向

    public void solve(char[][] board) {
        m = board.length;  // 获得矩阵行数
        n = board[0].length;  // 获得矩阵列数
        checked = new boolean[m][n];  // 初设每个位置都没有访问过

        // 遍历每一行，将边缘的'O'和与其连通的'O'都变为'Y'
        for(int i = 0; i < m; i++){
            if(board[i][0] == 'O' && checked[i][0] == false){  // 如果最左边是'O'且没有访问过
                dfs(board, i, 0);  // 用DFS将与其连通的'O'都变为'Y'
            }

            if(board[i][n - 1] == 'O' && checked[i][n-1] == false){  // 如果最右边是'O'且没有访问过
                dfs(board, i, n-1);  // 用DFS将与其连通的'O'都变为'Y'
            }
        }

        // 遍历每一列，将边缘的'O'和与其连通的'O'都变为'Y'
        for(int i = 0; i < n; i++){
            if(board[0][i] == 'O' && checked[0][i] == false){  // 如果最上边是'O'且没有访问过
                dfs(board, 0, i);  // 用DFS将与其连通的'O'都变为'Y'
            }

            if(board[m-1][i] == 'O' && checked[m-1][i] == false){  // 如果最下边是'O'且没有访问过
                dfs(board, m-1,i);  // 用DFS将与其连通的'O'都变为'Y'
            }
        }

        // 遍历整个矩阵，把'O'变为'X'，把'Y'变为'O'
        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++){
                if(board[i][j] == 'Y'){  // 如果是'Y'，则变回'O'
                    board[i][j] = 'O';
                }else{  // 其他情况，即'O' 和 'X'，都变为'X'
                    board[i][j] = 'X';
                }
            }
        }

    }

    // 使用深度优先搜索将与边缘'O'连通的'O'变为'Y'
    public void dfs(char[][] board,int x, int y){
        if(board[x][y] == 'X' || checked[x][y] == true){  // 如果是'X'或者已经访问过，则直接返回
            return;
        }

        board[x][y] = 'Y';  // 将'O'变为'Y'
        checked[x][y] = true;  // 标记该位置已经访问过

        // 对该位置的四个方向进行深度优先搜索
        for(int i = 0; i < 4; i++){
            int nextX = x + dir[i][0];
            int nextY = y + dir[i][1];

            // 如果新的位置在矩阵范围内，则继续搜索
            if(nextX >= 0 && nextX < m && nextY >= 0 && nextY < n){
                dfs(board, nextX, nextY);
            }
        }
    }
}
```





## 九、太平洋大西洋水流问题

### 题目

![004.png (852×1767) (raw.githubusercontent.com)](https://raw.githubusercontent.com/vankykoo/image/main/pic/004.png)

### 思路

首先通过两个DFS从海岸线流入陆地，标记可以流到哪些格子（DFS是正向思维，从海岸线流入陆地）。

然后在遍历地图的时候，只有都被标记的格子才放入res。



### 代码

```java
class Solution {
    int[][] dir = {{-1,0},{1,0},{0,1},{0,-1}}; // 这是四个方向的向量，表示向上、向下、向左和向右移动
    boolean[][][] checked; // 这是一个三维布尔数组，记录每个位置是否可以流向太平洋（index 0）和大西洋（index 1）
    List<List<Integer>> res = new ArrayList<>(); // 这是最后的结果列表，记录可以同时流向太平洋和大西洋的位置
    int m,n; // m和n分别表示矩阵的行数和列数

    public List<List<Integer>> pacificAtlantic(int[][] heights) {
        m = heights.length; // 初始化行数
        n = heights[0].length; // 初始化列数
        checked = new boolean[m][n][2]; // 初始化记录数组

        for (int i = 0; i < m; i++) {
            // 假设所有边界的位置都可以流向相应海洋，对于左右边界，从大西洋边界开始向陆地做DFS，从太平洋边界开始向陆地做DFS。
            checked[i][n - 1][0] = true;
            checked[i][0][1] = true;
            dfs(heights, i, n - 1, 0); 
            dfs(heights, i, 0, 1);
        }

        for (int i = 0; i < n; i++) {
            // 假设所有边界的位置都可以流向相应海洋，对于上下边界，从大西洋边界开始向陆地做DFS，从太平洋边界开始向陆地做DFS。
            checked[m - 1][i][0] = true;
            checked[0][i][1] = true;
            dfs(heights, m - 1, i, 0);
            dfs(heights, 0, i, 1);
        }

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                // 如果该位置即可以到太平洋又可以到大西洋，就放入答案数组
                if (checked[i][j][0] && checked[i][j][1])
                    res.add(List.of(i, j));
            }
        }

        return res;
    }

    public void dfs(int[][] heights, int x, int y, int ocean){
        for (int i = 0; i < 4; i++) {
            int nextX = x + dir[i][0];
            int nextY = y + dir[i][1];

            // 如果新位置越界或者高度低于当前位置，或者新位置已经被访问过，就跳过。否则就对新位置做DFS。
            if (nextX < 0 || nextX >= m || nextY < 0 || nextY >= n)
                continue;
            if (heights[nextX][nextY] < heights[x][y] || checked[nextX][nextY][ocean]) continue;
            checked[nextX][nextY][ocean] = true;
            dfs(heights, nextX, nextY, ocean);
        }
    }
}
```





##十、最大人工岛

###题目

![005.png (860×1270) (raw.githubusercontent.com)](https://raw.githubusercontent.com/vankykoo/image/main/pic/005.png)



### 思路

* 暴力解法
  * 每次遇到零就要计算旁边的岛的面积。这样会有很多重复的计算，超时！
* 给每个岛标上号，防止重复计算。
  * 第一次遍历时先找岛，然后给每个岛表上号，并记录每个岛的面积，存在map中。
  * 第二次遍历就是找零，让零变成一，然后计算可以连起来的最大的岛屿是多少，且可以直接根据遍历直接获得岛屿的面积，不用重复计算。



###代码

```java
class Solution {
    //定义一个二维数组，表示四个方向的偏移量
    private static final int[][] position = {{-1, 0}, {0, 1}, {1, 0}, {0, -1}};

    //使用深度优先搜索算法，计算一个岛屿的面积，并将其标记为mark
    public int dfs(int[][] grid, int row, int col, int mark) {
        int ans = 0; //初始化面积为0
        grid[row][col] = mark; //将当前位置标记为mark
        //遍历四个方向
        for (int[] current: position) {
            int curRow = row + current[0], curCol = col + current[1]; //计算相邻位置的坐标
            //如果坐标越界或者不是陆地，跳过
            if (curRow < 0 || curRow >= grid.length || curCol < 0 || curCol >= grid.length) continue;
            //如果是陆地，递归调用dfs，并累加面积
            if (grid[curRow][curCol] == 1)
                ans += 1 + dfs(grid, curRow, curCol, mark);
        }
        return ans; //返回面积
    }

    //给定一个二维网格，每个格子是陆地（值为1）或水域（值为0），返回最大的岛屿面积
    public int largestIsland(int[][] grid) {
        int ans = Integer.MIN_VALUE, size = grid.length, mark = 2; //初始化答案为最小值，网格大小，和岛屿标记
        Map<Integer, Integer> getSize = new HashMap<>(); //创建一个哈希表，存储每个标记对应的岛屿面积
        //遍历网格中的每个格子
        for (int row = 0; row < size; row++) {
            for (int col = 0; col < size; col++) {
                //如果是陆地，调用dfs计算岛屿面积，并将其存入哈希表
                if (grid[row][col] == 1) {
                    int areaSize = 1 + dfs(grid, row, col, mark);
                    getSize.put(mark++, areaSize);
                }
            }
        }
        //再次遍历网格中的每个格子
        for (int row = 0; row < size; row++) {
            for (int col = 0; col < size; col++) {
                //如果是水域，尝试将其变为陆地，并计算新的岛屿面积
                if (grid[row][col] == 0) {
                    Set<Integer> hashSet = new HashSet<>(); //创建一个哈希集合，存储相邻的岛屿标记
                    int curSize = 1; //初始化当前面积为1
                    //遍历四个方向
                    for (int[] current: position) {
                        int curRow = row + current[0], curCol = col + current[1]; //计算相邻位置的坐标
                        //如果坐标越界，跳过
                        if (curRow < 0 || curRow >= grid.length || curCol < 0 || curCol >= grid.length) continue;
                        int curMark = grid[curRow][curCol]; //获取相邻位置的标记
                        //如果标记已经在哈希集合中，或者不在哈希表中，跳过
                        if (hashSet.contains(curMark) || !getSize.containsKey(curMark)) continue;
                        hashSet.add(curMark); //将标记加入哈希集合
                        curSize += getSize.get(curMark); //累加相邻岛屿的面积
                    }
                    ans = Math.max(ans, curSize); //更新最大面积
                }
            }
        }
        //如果最大面积仍为最小值，说明网格中没有水域，返回网格的总大小；否则返回最大面积
        return ans == Integer.MIN_VALUE ? size * size : ans;
    }
}

```





## 十一、单词接龙

###题目

![006.png (850×1194) (raw.githubusercontent.com)](https://raw.githubusercontent.com/vankykoo/image/main/pic/006.png)



### 思路

* 步骤
  * 把可以相接的单词连接起来
  * 通过广搜找到最短路径

![](https://code-thinking-1253855093.file.myqcloud.com/pics/20210827175432.png)



### 代码

```java
public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    HashSet<String> wordSet = new HashSet<>(wordList); //转换为hashset 加快速度
    if (wordSet.size() == 0 || !wordSet.contains(endWord)) {  //特殊情况判断
        return 0;
    }
    Queue<String> queue = new LinkedList<>(); //bfs 队列
    queue.offer(beginWord);
    Map<String, Integer> map = new HashMap<>(); //记录单词对应路径长度
    map.put(beginWord, 1);

    while (!queue.isEmpty()) {
        String word = queue.poll(); //取出队头单词
        int path  = map.get(word); //获取到该单词的路径长度
        for (int i = 0; i < word.length(); i++) { //遍历单词的每个字符
            char[] chars = word.toCharArray(); //将单词转换为char array，方便替换
            for (char k = 'a'; k <= 'z'; k++) { //从'a' 到 'z' 遍历替换
                chars[i] = k; //替换第i个字符
                String newWord = String.valueOf(chars); //得到新的字符串
                if (newWord.equals(endWord)) {  //如果新的字符串值与endWord一致，返回当前长度+1
                    return path + 1;
                }
                if (wordSet.contains(newWord) && !map.containsKey(newWord)) { //如果新单词在set中，但是没有访问过
                    map.put(newWord, path + 1); //记录单词对应的路径长度
                    queue.offer(newWord);//加入队尾
                }
            }
        }
    }
    return 0; //未找到
}
```





## 十二、钥匙和房间

### 题目

![007.png (849×1159) (raw.githubusercontent.com)](https://raw.githubusercontent.com/vankykoo/image/main/pic/007.png)

### 思路

* 通过**深度优先搜索**遍历房间，通过递归调用`dfs`函数。
* 在`dfs`函数中，首先将当前房间标记为已访问，然后遍历当前房间的钥匙列表，如果相邻的房间未访问，则递归调用`dfs`函数进行访问。
* 在主函数中，遍历标记数组，如果存在未访问的房间，则返回false，否则返回true。



### 代码

```java
class Solution {
    // 用于标记房间是否被访问过
    boolean[] checked;

    public boolean canVisitAllRooms(List<List<Integer>> rooms) {
        // 初始化标记数组
        checked = new boolean[rooms.size()];

        // 从0号房间开始进行深度优先搜索
        dfs(rooms, rooms.get(0), 0);

        // 检查所有房间是否都被访问过
        for (boolean isChecked : checked) {
            if (!isChecked) {
                return false; // 存在未访问的房间，返回false
            }
        }
        return true; // 所有房间都被访问过，返回true
    }

    // 深度优先搜索函数
    public void dfs(List<List<Integer>> rooms, List<Integer> keys, int index) {
        // 将当前房间标记为已访问
        checked[index] = true;

        // 遍历当前房间的钥匙列表
        for (Integer key : keys) {
            if (!checked[key]) {
                // 如果相邻的房间未访问，则递归调用DFS进行访问
                dfs(rooms, rooms.get(key), key);
            }
        }
    }
}
```



##十三、岛屿的周长

### 题目

![009.png (856×1336) (raw.githubusercontent.com)](https://raw.githubusercontent.com/vankykoo/image/main/pic/009.png)



### 思路

1. 初始化变量：m 和 n 分别为网格的行数和列数，checked 数组用于标记已访问过的节点，dir 数组表示四个方向（上、下、左、右），sum 记录岛屿的周长。
2. 遍历整个网格，找到所有未被访问过的陆地节点，并从这些节点开始进行深度优先搜索（DFS）。
3. 在 DFS 过程中，遇到边界或水时，增加 sum 的值；遇到陆地时，继续递归搜索其相邻节点。
4. 最后返回 sum 的值作为岛屿的周长。



### 代码

```java
class Solution {
    // 用于标记已访问的格子
    boolean[][] checked;
    // 方向数组，表示上下左右四个方向
    int[][] dir = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    // 行数和列数
    int m, n;
    // 记录岛屿的周长
    int sum;

    // 主函数，计算岛屿的周长
    public int islandPerimeter(int[][] grid) {
        m = grid.length;
        n = grid[0].length;
        checked = new boolean[m][n];

        sum = 0;

        // 遍历整个二维数组
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                // 如果当前格子未被访问且是岛屿的一部分
                if (!checked[i][j] && grid[i][j] == 1) {
                    // 深度优先搜索
                    dfs(grid, i, j);
                }
            }
        }

        // 返回岛屿的周长
        return sum;
    }

    // 深度优先搜索函数
    public void dfs(int[][] grid, int x, int y) {
        // 如果当前格子已经访问过或者是海洋（值为0）
        if (checked[x][y] || grid[x][y] == 0) {
            return;
        }

        // 标记当前格子已访问
        checked[x][y] = true;

        // 遍历四个方向
        for (int i = 0; i < 4; i++) {
            int nextX = x + dir[i][0];
            int nextY = y + dir[i][1];

            // 如果越界，周长加一
            if (nextX < 0 || nextX >= m || nextY < 0 || nextY >= n) {
                sum++;
                continue;
            }

            // 如果相邻格子是海洋，周长加一
            if (grid[nextX][nextY] == 0) {
                sum++;
            }

            // 递归搜索相邻的岛屿格子
            dfs(grid, nextX, nextY);
        }
    }
}

```





## 十四、并查集基础

### 1）介绍

并查集（Disjoint-Set Data Structure 或者 Union-Find Algorithm）是一种数据结构，它用来表示一组对象的部分划分情况，并且支持两种操作：**Find** 和 **Union**。这个名字很好地概括了它的功能：

- **Find**：查询元素所在集合的标识符。
- **Union**：合并两个已知所属集合的元素。

这个数据结构的关键在于能够有效地维护这些操作，并在面对大量插入、删除和查找的情况下保持高效。

####	1.并查集的基本原理

并查集的核心是一个森林，即一组树的集合，每个树代表一个独立的集合。每个节点包含一个指向其父节点的指针，这使得我们可以跟踪元素之间的层级关系。如果两个节点在同一棵树中，则它们属于同一个集合。如果不在同一棵树中，则它们分别属于不同的集合。

####	2.查找（Find）操作

`Find` 操作的目标是找到某个元素所处集合的“根”节点。我们通过沿着从当前节点到根节点的路径上的指针逐步向上移动来实现这一点。为了提高效率，在每次查找过程中都会更新沿途节点的父节点为找到的根节点，这样下一次查找就可以更快地到达根节点，这就是路径压缩（Path Compression）技术。

#### 	3.合并（Union）操作

当需要将两个元素加入到同一个集合中时，就需要执行 `Union` 操作。这是通过简单地将一棵树的根节点链接到另一棵树的根节点来完成的。为了保证将来查找操作的有效性，通常会选择较矮的那棵树作为子树连接到较高的那棵树上，这样可以尽量保持树的平衡，减少查找的时间复杂度。

####	4.并查集的应用

并查集广泛应用于各种需要动态维护集合关系的场景，例如社交网络中的朋友关系、图论中的连通性问题、网络路由等。在实际应用中，由于采用了路径压缩和按秩合并的优化策略，使得并查集即使在处理大规模数据时也能保持极高的效率。



### 2）示例

以下是一个简单的Java代码示例，演示了并查集的基本实现。该示例使用路径压缩和按秩合并（rank）优化，以提高性能。

```java
class UnionFind {
    private int[] parent;
    private int[] rank;

    public UnionFind(int size) {
        parent = new int[size];
        rank = new int[size];

        // 初始化，每个元素自成一组
        for (int i = 0; i < size; i++) {
            parent[i] = i;
            rank[i] = 0;
        }
    }

    public int find(int x) {
        // 查找元素x所属的集合的代表元素，并进行路径压缩
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    public void union(int x, int y) {
        // 将元素x所属的集合和元素y所属的集合合并，使用按秩合并进行优化
        int rootX = find(x);
        int rootY = find(y);

        if (rootX != rootY) {
            if (rank[rootX] < rank[rootY]) {
                parent[rootX] = rootY;
            } else if (rank[rootX] > rank[rootY]) {
                parent[rootY] = rootX;
            } else {
                parent[rootX] = rootY;
                rank[rootY]++;
            }
        }
    }
}

public class Main {
    public static void main(String[] args) {
        int n = 6; // 元素个数

        UnionFind uf = new UnionFind(n);

        // 合并一些集合
        uf.union(0, 1);
        uf.union(1, 2);
        uf.union(2, 3);

        // 查找集合的代表元素
        System.out.println(uf.find(3)); // 输出 0，因为元素 0 是集合 {0, 1, 2, 3} 的代表元素
        System.out.println(uf.find(4)); // 输出 4，因为元素 4 自成一组

        // 合并新的集合
        uf.union(4, 5);

        // 查找集合的代表元素
        System.out.println(uf.find(5)); // 输出 4，因为元素 4 是集合 {4, 5} 的代表元素
    }
}
```

这个Java代码示例展示了并查集的基本操作，包括初始化、查找和合并。这是一个通用的实现，可以根据具体问题进行适当的调整和扩展。



## 十五、寻找图中是否存在路径

### 题目

![012.png (848×1550) (raw.githubusercontent.com)](https://raw.githubusercontent.com/vankykoo/image/main/pic/012.png)



### 思路

1. **初始化并查集：** 在构造函数中，你首先初始化了每个节点的父节点和秩。每个节点最初都是其自身的父节点，秩初始化为 0。

2. **边的合并：** 通过遍历给定的边数组 `edges`，对每一条边进行合并操作，调用 `union` 方法。在这个方法中，你找到每个节点所在的集合的根节点，并根据秩的比较决定如何合并。

3. **查找根节点：** 通过 `find` 方法，你实现了路径压缩，即将节点到根节点的路径上的所有节点的父节点直接设为根节点。这有助于加速后续的查找操作。

4. **合并操作：** 在 `union` 方法中，你通过比较节点所在树的秩，将秩较小的树合并到秩较大的树上。如果秩相等，随意选择一个树作为父节点，并增加其秩。

5. **最终判断路径是否连通：** 在 `validPath` 方法中，你查找源节点和目标节点的父节点，然后判断它们是否相同，以确定是否存在一条有效路径。




### 代码

```java
class Solution {
    // 用于存储每个节点的父节点
    private int[] parent;   
    // 用于记录每个节点的秩（rank），在合并时用于优化
    private int[] rank;

    // 判断是否存在一条有效路径
    public boolean validPath(int n, int[][] edges, int source, int destination) {
        // 初始化并查集
        parent = new int[n];
        rank = new int[n];

        // 初始化每个节点的父节点和秩
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            rank[i] = 0;
        }

        // 遍历边，进行合并操作
        for(int i = 0; i < edges.length; i++){
            union(edges[i]);
        }
        
        // 查找源节点和目标节点的父节点
        int sourceFather = find(source);
        int destinationFather = find(destination);

        // 判断源节点和目标节点是否在同一个集合中
        return sourceFather == destinationFather;
    }

    // 查找节点 x 的根节点（代表）
    public int find(int x) {
        // 路径压缩：将节点 x 到根节点的路径上的所有节点的父节点直接设为根节点，减少后续查找的时间
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    // 合并两个节点所在的集合
    public void union(int[] edge) {
        int rootX = find(edge[0]);
        int rootY = find(edge[1]);

        if (rootX != rootY) {
            // 按秩合并：将秩较小的树合并到秩较大的树上，以保持树的平衡
            if (rank[rootX] < rank[rootY]) {
                parent[rootX] = rootY;
            } else if (rank[rootX] > rank[rootY]) {
                parent[rootY] = rootX;
            } else {
                // 秩相等时，随便合并一个并增加其秩
                parent[rootX] = rootY;
                rank[rootY]++;
            }
        }
    }
}
```





## 十六、冗余连接

### 题目

![013.png (825×1438) (raw.githubusercontent.com)](https://raw.githubusercontent.com/vankykoo/image/main/pic/013.png)



### 思路

并查集会将遍历过的结点生成为一棵树，这道题中，如果树中已经有当前路径的两个结点，那么就返回这个路径即可。



### 代码

```java
class Solution {
    // 用于存储每个节点的父节点
    private int[] parent;   
    // 用于记录每个节点的秩（rank），在合并时用于优化
    private int[] rank;

    // 主函数，解决力扣第684题
    public int[] findRedundantConnection(int[][] edges) {
        int n = edges.length * 2;
        // 初始化并查集
        parent = new int[n];
        rank = new int[n];

        // 初始化每个节点的父节点和秩
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            rank[i] = 0;
        }

        // 遍历边，进行合并操作
        for (int i = 0; i < edges.length; i++) {
            // 判断是否已经添加过，且在同一棵树上
            if (find(edges[i][0]) == find(edges[i][1])) {
                return new int[]{edges[i][0], edges[i][1]};
            } else {
                union(edges[i]);
            }
        }

        return null;
    }

    // 查找节点 x 的根节点（代表）
    public int find(int x) {
        // 路径压缩：将节点 x 到根节点的路径上的所有节点的父节点直接设为根节点，减少后续查找的时间
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    // 合并两个节点所在的集合
    public void union(int[] edge) {
        int rootX = find(edge[0]);
        int rootY = find(edge[1]);

        if (rootX != rootY) {
            // 按秩合并：将秩较小的树合并到秩较大的树上，以保持树的平衡
            if (rank[rootX] < rank[rootY]) {
                parent[rootX] = rootY;
            } else if (rank[rootX] > rank[rootY]) {
                parent[rootY] = rootX;
            } else {
                // 秩相等时，随便合并一个并增加其秩
                parent[rootX] = rootY;
                rank[rootY]++;
            }
        }
    }
}
```





## 十七、冗余连接Ⅱ

### 题目

![015.png (877×1787) (raw.githubusercontent.com)](https://raw.githubusercontent.com/vankykoo/image/main/pic/015.png)

### 思路

前面的步骤和之前一样【构建并查集】，这里多了几个方法：

1. **找到图中的多余边：**
   - `getRemoveEdge(int[][] edges)` 方法用于找到图中的多余边。它遍历边数组，如果两个节点已经在同一连通分量中，则返回这条边；否则，将这两个节点合并到同一连通分量中。
2. **判断删除一条边后是否形成树：**
   - `isTreeAfterRemoveEdge(int[][] edges, int deleteEdge)` 方法用于判断删除一条边后是否形成树。它先初始化父节点数组，然后遍历边数组，如果两个节点已经在同一连通分量中，则说明形成环，返回 `false`；否则，将这两个节点合并到同一连通分量中。最终，如果没有形成环，则返回 `true`。
   - 这个方法和上一题的类似
3. **找到入度为2的节点：**
   - 在 `findRedundantDirectedConnection(int[][] edges)` 方法中，维护了一个入度数组 `inDegree`，用于统计每个节点的入度。
   - 遍历边数组，找到入度为2的节点，并将其记录在 `twoDegree` 列表中。
4. **判断入度为2的节点删除一条边后是否形成树：**
   - 如果 `twoDegree` 列表不为空，分别判断删除其中一条边后是否形成树，如果是，则返回该边；否则，返回另一条边。
5. **如果没有入度为2的节点，直接找到多余的边：**
   - 如果 `twoDegree` 列表为空，则调用 `getRemoveEdge(edges)` 方法找到图中的多余边。



总结：找到入度为2的节点，并记录这两条边，把这些边放到一个数组/集合里，按原来edges的顺序放，然后从后向前遍历这些边，如果删除这个边能构成树，那么就是结果了。

### 代码

```java
public class Solution {

    private static final int N = 1010;
    private int[] father;

    // 构造函数，初始化并查集的父节点数组
    public Solution() {
        father = new int[N];

        for (int i = 0; i < N; ++i) {
            father[i] = i;
        }
    }

    // 查找节点的根节点，实现路径压缩
    private int find(int u) {
        if (u == father[u]) {
            return u;
        }
        father[u] = find(father[u]);
        return father[u];
    }

    // 合并两个节点所在的连通分量
    private void join(int u, int v) {
        u = find(u);
        v = find(v);
        if (u == v) return;
        father[v] = u;
    }

    // 判断两个节点是否在同一连通分量
    private Boolean same(int u, int v) {
        u = find(u);
        v = find(v);
        return u == v;
    }

    // 初始化并查集的父节点数组
    private void initFather() {
        for (int i = 0; i < N; ++i) {
            father[i] = i;
        }
    }

    // 找到图中的多余边
    private int[] getRemoveEdge(int[][] edges) {
        initFather();
        for (int i = 0; i < edges.length; i++) {
            if (same(edges[i][0], edges[i][1])) {
                return edges[i];
            }
            join(edges[i][0], edges[i][1]);
        }
        return null;
    }

    // 判断删除一条边后是否形成树
    private Boolean isTreeAfterRemoveEdge(int[][] edges, int deleteEdge) {
        initFather();
        for (int i = 0; i < edges.length; i++) {
            if (i == deleteEdge) continue;
            if (same(edges[i][0], edges[i][1])) {
                return false;
            }
            join(edges[i][0], edges[i][1]);
        }
        return true;
    }

    // 找到图中的多余有向边
    public int[] findRedundantDirectedConnection(int[][] edges) {
        int[] inDegree = new int[N];
        for (int i = 0; i < edges.length; i++) {
            // 统计每个节点的入度
            inDegree[edges[i][1]] += 1;
        }

        // 存储入度为2的节点
        ArrayList<Integer> twoDegree = new ArrayList<Integer>();
        for (int i = edges.length - 1; i >= 0; i--) {
            // 找到入度为2的节点
            if (inDegree[edges[i][1]] == 2) {
                twoDegree.add(i);
            }
        }

        if (!twoDegree.isEmpty()) {
            // 如果删除其中一条边后形成树，则返回这条边
            if (isTreeAfterRemoveEdge(edges, twoDegree.get(0))) {
                return edges[twoDegree.get(0)];
            }
            // 否则返回另一条边
            return edges[twoDegree.get(1)];
        }

        // 否则，返回多余的边
        return getRemoveEdge(edges);
    }
}
```





























































































































































































