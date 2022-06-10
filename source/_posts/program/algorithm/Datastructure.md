---
title: Datastructure
toc: true
date: 2022-05-13 08:59:23
tags:
---

## 数据结构

### 数组

### 链表

### 栈

### 队列

### 二叉树

- 二叉搜索树BST
- 二叉平衡AVL
- 红黑树

- 前序遍历

```java
void preOrder(TreeNode root) {
    if(root == null) return;
    System.out.println(root.val);
    preOrder(root.left);
    preOrder(root.right);
}
```

- 中序遍历

```java
void inOrder(TreeNode root) {
    if(root == null) return;
    preOrder(root.left);
    System.out.println(root.val);
    preOrder(root.right);
}
```

- 后序遍历

```java
void postOrder(TreeNode root) {
    if(root == null) return;
    preOrder(root.left);
    preOrder(root.right);
    System.out.println(root.val);
}
```

### 多叉树

- B树
- B+树

### 图

- 构成:
  - 节点
  - 边
- 表示:
  - 邻接表 `List<Integer>[]`
  - 邻接矩阵 `boolean[][] matrix = new boolean[from][to]`
- 度
  - 无向图: 每个节点相连的边的条数
  - 有向图: 入度(指向当前节点) 和 出度(从当前节点执行别的节点)
- 加权: 每条边有一定的权重
- 有向图环检测
  - DFS记录遍历path
  - BFS结合入度(保证不走回头路)数量遍历图, 遍历次数等于节点数则表示无环
- 拓扑排序: 有向无环图
  - DFS后序遍历结果反转后即为排序结果
  - BFS结合入度遍历图, 遍历顺序即为排序结果
- 二分图

- DFS遍历

```java
// 记录从起点到当前节点的路径
List<Integer> path;
// 记录被遍历过的节点
Set<Integer> visited;

/* 图遍历框架 */
void traverse(int[][] graph, int n){
    for(int i = 0; i < n; i++){ 
        // 图中并不是所有节点都相连，所以要用一个 for 循环将所有节点都作为起点调用一次 DFS 搜索算法。
        traverse(graph, i);
    }
}
void traverse(int[][] graph, int start) {
    if(visited.contains(start)) return;
    // 做选择：标记节点start在路径上
    path.add(start);
    // 经过节点start，标记为已遍历
    visited.add(start);
    for(int n : graph[start]) {
        traverse(graph, n);
    }
    // 撤销选择：节点start离开路径
    path.remove(path.size() - 1);
}
```

- BFS遍历

```java
void traverse(int[][] graph, int n){
    // 构建入度
    int[] indegree = new int[n];
    for(int i = 0; i < n; i++){
        int[] next = graph[i];
        if(next != null){
            indegree[i] = next.length;
        }
    }
    Queue<Integer> q = new LinkedList<>();
    // 入度为0的作为起点
    for(int i = 0; i < n; i++){
        if(indegree[i] == 0) q.add(i);
    }
    // 记录遍历节点数
    int count = 0;
    // 遍历路径即为拓扑排序结果
    int[] path = new int[n];
    // 遍历图
    while(!q.isEmpty()){
        int curr = q.poll();
        path[count] = curr;
        count++;
        for(int next : graph[curr]){
            indegree[next]--;
            // 入度节点都被遍历完, 当前节点可以加入队列
            if(indegree[next] == 0) q.add(next);
        }
    }
    // 遍历节点等于全量节点, 说明图无环
    // return count == n
}
```

- Dijkstra最短路径算法

```java

// 记录到达当前节点的距离
class State{
    // 节点id
    int node;
    // 从起点到当前节点的距离
    int dis;
    State(int node, int dis){
        this.node = node;
        this.dis = dis;
    }
}

/**
 * 1.build graph with adjacent list
 * 2.use Dijkstra find shortest path
 * TC=O(ElogE), SC=O(E), E=edges.length
 */
int[] shortestPath(int n, int[][] edges, int start){
    Map<Integer, Integer>[] graph = new HashMap[n];
    // 1.build graph with adjacent list
    for(int[] edge : edges){
        int from = edge[0], to = edge[1], weight = edge[2];
        if(graph[from] == null) graph[from] = new HashMap<>();
        graph[from].put(to, weight);
    }
    int[] disTo = new int[n];
    Arrays.fill(disTo, Integer.MAX_VALUE);
    // base case
    disTo[start] = 0;
    Queue<State> pq = new PriorityQueue<>();
    pq.add(new State(start, 0));
    // 2.use Dijkstra find shortest path
    while(!pq.isEmpty()){
        State curr = pq.poll();
        int currNode = curr.node;
        int currDis = curr.dis;
        if(currDis > disTo[currNode]){
            // already had shorter path
            continue;
        }
        if(graph[currNode] == null) continue;
        // traverse adjacent node
        for(int next : graph[currNode].keySet()){
            int nextDis = currDis + graph[currNode].get(next);
            // update disTo[next] if from currNode to nextNode is shorter
            if(nextDis < disTo[next]){
                disTo[next] = nextDis;
                pq.add(new State(next, nextDis));
            }
        }
    }
    return disTo;
}


```

### 常见

- LRU
- LFU



## 常见问题
