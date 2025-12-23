---
layout: post
title: "Matrix area sum using prefix sum"
date: 2025-11-24
description: ""
tag: Algorithms
prime: false
---

## Matrix Prefix Sum

```java
int[][] pre;
public void build(int[][] grid) {
    int n = grid.length;
    int m = grid[0].length;
    pre = new int[n + 1][m + 1];
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            pre[i + 1][j + 1] = pre[i][j + 1] + pre[i + 1][j] - pre[i][j] + grid[i][j];
        }
    }
}
```

## Calculate Matrix Area

```java
public int calculate(int a, int b, int c, int d) {
    return pre[c + 1][d + 1] - pre[a][d + 1] - pre[c + 1][b] + pre[a][b];
}
```