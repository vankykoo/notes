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







































































































































































































































































