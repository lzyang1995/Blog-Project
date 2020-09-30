---
title: "LeetCode 743: Network Delay Time"
date: 2020-09-30T16:58:57+08:00
draft: false
tags: ["LeetCode", "Algorithm"]
categories: ["算法"]
lightgallery: true

math:
  enable: true
---

原题链接： <https://leetcode.com/problems/network-delay-time/>

这道题是一个图的问题。思路如下：首先，图中某个顶点接收到信号的时间就是从K到该顶点的最短路径的长度。所以该题的答案就是从K到所有顶点的最短路径长度的最大值。因此我们可以用Dijkstra算法先求出K到各顶点的最短路径长度，然后返回其中的最大值。如果最大值是Infinity，则表示有的顶点是从K出发无法到达的，那么就按题目的要求返回-1。

Dijkstra算法也是一个非常著名的贪心算法了，用于求某个顶点s到其他各个顶点的最短路径。它的思路就是把所有顶点分为两类：known和unknown，known表示s到该顶点的最短路径已经确定了，unknown则表示还没有确定。每一个顶点v都对应着一个距离dist，表示从s开始只经过known集合中的顶点到达v的最短路径长度。在每一次循环中，该算法都会从所有unknown的顶点中取出dist最小的顶点v并加入到known中，表示这个长度就是从s到该顶点的最短路径长度了，贪心也是体现在这个地方。然后会更新剩下的unknown顶点的路径长度，即如果存在某个unknown顶点w，使得从s经过v到达w的路径长度比当前从s到w的路径长度更短的话，则会将w的路径长度进行更新。Dijkstra算法的伪代码如下（摘自*Data Structures and Algorithm Analysis in C++, Fourth Edition*）：

```
void Graph::dijkstra( Vertex s )
{
    for each Vertex v
    {
        v.dist = INFINITY;
        v.known = false;
    }
    s.dist = 0;

    while( there is an unknown distance vertex )
    {
        Vertex v = smallest unknown distance vertex; // 1
        v.known = true;
        
        // 2
        for each Vertex w adjacent to v 
            if( !w.known )
            {
                DistType cvw = cost of edge from v to w;
                
                if( v.dist + cvw < w.dist )
                {
                    // Update w
                    decrease( w.dist to v.dist + cvw );
                    w.path = v;
                }
            }
    }
}
```

其中v.path存储的是从s到v的最短路径中，v前面的顶点，可以用于得到具体的路径。值得注意的是，**如果图中存在权值为负数的边，那么Dijkstra算法得到的结果可能是错误的**。

Dijkstra算法的时间复杂度与伪代码中1部分的算法有关。如果我们采用一个比较简单的方法，即把当前所有unknown的顶点遍历一遍去寻找dist最小的顶点，那么这一部分总的时间复杂度是$O(|V|^2)$。而后续2部分更新unknown顶点的dist及path的操作对于每一条边最多执行一次，因此总的时间复杂度是$O(|E|)$。这样Dijkstra算法的时间复杂度就是$O(|E| + |V|^2) = O(|V| ^ 2)$。如果图中的边数较多，那么这个复杂度已经接近最优了。但如果图中的边数较少的话，实际上对于1部分有一个基于priority queue的效率更高的方法。我们可以使用priority queue来存储unknown中的顶点的dist值，这样寻找unknown中dist最小的顶点只需要执行deleteMin操作即可。对于后续dist及path的更新，我们可以不去更新priority queue中的已有节点，而是直接插入一个新结点来代表新的路径长度。这意味着priority queue中一个顶点可能出现多次，因此我们在deleteMin时需要判断当前顶点是否已经在known集合中了，如果是则需要继续执行deleteMin。对于这个算法，priority queue中的节点数为$O(|V| + |E|) = O(|E|)$，因此1部分的总的时间复杂度为$O(|V|log|E|)$，2部分总的时间复杂度为$O(|E|log|E|)$，Dijkstra算法的时间复杂度为$O((|V| + |E|)log|E|) = O(|E|log|V|)$（因为$|E| = O(|V| ^ 2)$，$log|E| = O(2log|V|)$）。

回到LeetCode的问题，下面是我使用比较简单的$O(|V| ^ 2)$的算法实现的代码（JS中没有priority queue，得自己写，因此没有用第二种）：

```js
// Author: Zhiyang Li
// Date: 2020.09.30

/**
 * @param {number[][]} times
 * @param {number} N
 * @param {number} K
 * @return {number}
 */
var networkDelayTime = function(times, N, K) {
    const graph = new Map();
    
    for (let i = 1;i <= N;i++) {
        graph.set(i, []);
    }
    for (const [u, v, w] of times) {
        graph.get(u).push(new Node(v, w));
    }
    
    const known = new Array(N + 1);
    const dist = new Array(N + 1);
    
    for (let i = 1;i <= N;i++) {
        known[i] = false;
        dist[i] = Infinity;
    }
    
    dist[K] = 0;
    for (let i = 0;i < N;i++) {
        const curId = findMin(known, dist);
        known[curId] = true;
        
        for (const node of graph.get(curId)) {
            if (!known[node.id] && dist[curId] + node.cost < dist[node.id]) {
                dist[node.id] = dist[curId] + node.cost;
            }
        }
    }
    
    let result = Math.max(...(dist.slice(1)));
    return result === Infinity ? -1 : result;
};

function Node(id, cost) {
    this.id = id;
    this.cost = cost;
}

function findMin(known, dist) {
    let id = -1;
    let minDist = Infinity;
    for (let i = 1;i < known.length;i++) {
        if (!known[i] && (id === -1 || dist[i] < minDist)) {
            id = i;
            minDist = dist[i];
        }
    }
    
    return id;
}
```

## 参考资料

1. *Data Structures and Algorithm Analysis in C++, Fourth Edition*
2. <https://www.geeksforgeeks.org/dijkstras-shortest-path-algorithm-greedy-algo-7/>
