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











































































































































































































































