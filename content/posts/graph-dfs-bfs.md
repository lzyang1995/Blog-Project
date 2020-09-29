---
title: "图的深度优先及宽度优先遍历"
date: 2020-09-29T11:39:26+08:00
draft: false
tags: ["Algorithm"]
categories: ["算法"]
lightgallery: true
---

图的常用数据结构有两种：邻接表和邻接矩阵。而由于邻接矩阵占用的空间更多，在边数较少的情况下会成为稀疏矩阵，因此邻接表会用的更多一点。邻接表可以用一个Hash Map来表示，key是顶点（vertex），value是一个顶点的列表（对于带有权值的图，也会存储各个边的权值）。对于有向图中的一条边(u, v)，v会被存储在u对应的列表中。而对于无向图中的一条边(u, v)，v会被存储在u对应的列表中，并且u也会被存储在v对应的列表中。由于图中可能存在环，因此我们在遍历过程中需要标记访问过的节点，避免重复访问。需要注意的是，无向图能一次遍历完所有节点当且仅当该图是连通的，而有向图能一次遍历完所有节点当且仅当该图是强连通的。所以可能需要多次遍历才能访问到所有节点。

下面给出深度优先遍历（DFS）和宽度优先遍历（BFS）的伪代码：

DFS（参考自*Data Structures and Algorithm Analysis in C++, Fourth Edition*）：

```
function dfs(v) {
    v.visited = true;
    print(v);

    for each vertex w in v's adjacency list {
        if (!w.visited) {
            dfs(w);
        }
    }
}
```

BFS（参考自<https://www.geeksforgeeks.org/breadth-first-search-or-bfs-for-a-graph/>）:

```
function bfs(v) {
    let queue = [];
    v.visited = true;
    queue.push(v);

    while (queue.length !== 0) {
        const current = queue.shift();
        print(current);

        for each vertex w in current's adjacency list {
            if (!w.visited) {
                w.visited = true;
                queue.push(w);
            }
        }
    }
}
```

我们可以任取一个顶点v开始遍历，遍历完成后，如果仍有顶点未被标记，则从该顶点开始继续遍历，直到所有顶点都被标记为止。

## 参考资料：

1. *Data Structures and Algorithm Analysis in C++, Fourth Edition*
2. <https://www.geeksforgeeks.org/breadth-first-search-or-bfs-for-a-graph/>
3. <https://www.geeksforgeeks.org/depth-first-search-or-dfs-for-a-graph/>