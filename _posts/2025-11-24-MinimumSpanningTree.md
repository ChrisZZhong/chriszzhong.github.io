---
layout: post
title: "Minimum Spanning Tree"
date: 2025-11-24
description: ""
tag: Algorithms
prime: false
---

## Note that MST only exists in undirected graph

## Kruskal algorithm (most common case) O(n + m) + O(mlogm)

1. **sort all edges in ascending order by weight**, and start considering edges from the **smallest weight**.

2. If adding the current edge does **not form a cycle**, **pick this edge** else skip. 

3. repeat to all edges or pick edge equals to (n - 1)

### use **Union Find**

1. **Sort edge by weight, start by smallest edge**

2. **find(a) != find(b) --> pick, else skip**

3. **repeat to end**

```Java
class Demo {

    int[] father;

    public void build(int n) {
        father = new int[n];
        for (int i = 0; i < n; i++) {
            father[i] = i;
        }
    }

    public int find(int x) {
        int a = x;
        while (father[a] != a) {
            a = father[a];
        }
        while (x != father[x]) {
            father[x] = a;
            x = father[x];
        }
        return a;
    }

    public boolean union(int a, int b) {
        int fa = find(a);
        int fb = find(b);
        if (fa != fb) {
            father[fa] = fb;
            return true;
        }
        return false;
    }

    public int MST(int n, int[][] edges) {
        build(n);
        Arrays.sort(edges, 0, n, (a, b) -> a[2] - b[2]);
        int ans = 0;
        int edgeCnt = 0;
        for (int[] edge : edges) {
            if (union(edge[0], edge[1])) {
                edgeCnt++;
                ans += edge[2];
            }
        }
        return edgeCnt == n - 1? ans : -1;
    }
}
```


## Prim algorithm (need improment) O(n + m) + O(m + logm), can be improved to O((n + m) * logn)

1. Initialize set and heap:
    - empty --> set
    - empty --> heap (min heap with weight)
2. start from any vertices
    - add node to set
    - add all edges linked with that node to heap
3. Pop the edge from heap, check vertex x:
    - A: if x in set, do nothing
    - B: if x not in set, add x to set and add its edges tp heap
    - repeat step 3 until heap is empty

```Java
public int minimumSpanningTree(int n, int[][] edges) {
    // 用 List<int[]>[] 存图，每个 int[] = {neighbor, weight}
    List<int[]>[] graph = new ArrayList[n];
    Arrays.setAll(graph, i -> new ArrayList<>()); // 初始化

    for (int[] e : edges) {
        int u = e[0], v = e[1], w = e[2];
        graph[u].add(new int[]{v, w});
        graph[v].add(new int[]{u, w}); // 无向图
    }

    Set<Integer> visited = new HashSet<>();
    PriorityQueue<int[]> heap = new PriorityQueue<>((a, b) -> a[1] - b[1]);

    // 从 0 开始
    visited.add(0);
    for (int[] e : graph[0]) {
        heap.offer(e);
    }

    int mstWeight = 0;
    while (!heap.isEmpty()) {
        int[] cur = heap.poll();
        int node = cur[0], weight = cur[1];
        if (visited.contains(node)) continue;

        visited.add(node);
        mstWeight += weight;

        for (int[] e : graph[node]) {
            if (!visited.contains(e[0])) {
                heap.offer(e);
            }
        }
    }

    return visited.size() == n ? mstWeight : -1;
}
```

## Practice:

[1168. Optimize Water Distribution in a Village](https://leetcode.cn/problems/optimize-water-distribution-in-a-village/description/)

[1697. Checking Existence of Edge Length Limited Paths](https://leetcode.cn/problems/checking-existence-of-edge-length-limited-paths/description/)

## Minimum Bottleneck Spanning Tree (MBST)

A Minimum Bottleneck Spanning Tree (MBST) is a spanning tree of a weighted undirected graph in which the largest edge weight (the bottleneck edge) is minimized among all possible spanning trees.

Every MST is also a MBST, But not every MBST is an MST