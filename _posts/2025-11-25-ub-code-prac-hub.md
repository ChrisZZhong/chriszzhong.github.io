---
layout: post
title: "UB coding prac hub & other"
date: 2025-12-19
description: "UB coding prac hub"
tag: Algorithms
prime: false
---

## [programhelp](https://programhelp.net/?s=uber)

## 找到二叉搜索树中的第k个大元素

给定一个二叉搜索树（BST）和一个正整数k，找到二叉搜索树中的第k个大元素。要求理解并实现这一算法。

思路: 先从根节点（10）开始，移动到右子节点（20）。然后继续向右移动到节点（40）。由于节点 (40) 没有子节点，因此将计数器增加到 1。再移回节点 (20) 并将计数器增加到 2。最后移动到节点 (20) 的左子节点，即节点 (15)，并将计数器增加到 3，找到第三大元素 (15)。

反向中序遍历（DFS）

```java
class Solution {
    int k;
    int ans;

    public int kthLargest(TreeNode root, int k) {
        this.k = k;
        dfs(root);
        return ans;
    }

    private void dfs(TreeNode node) {
        if (node == null) return;

        dfs(node.right);

        if (--k == 0) {
            ans = node.val;
            return;
        }

        dfs(node.left);
    }
}
TC: O(n)
SC: O(h)
```

Follow up 面试官询问如何处理多个 k 值的问题。将k值存储在一个数据库中，并查找到 max_k 。在进行逆向中序遍历时，用一个节点记录已访问的节点数。当节点值在 k 值存储中时，将节点对应值存储在结果字典中。返回最终结果字典，其中包含所有 k 值对应的节点值。

```java
class Solution {
    int count = 0;
    int maxK;
    Set<Integer> ks;
    Map<Integer, Integer> result = new HashMap<>();

    public Map<Integer, Integer> kthLargestMultiple(TreeNode root, List<Integer> queries) {
        ks = new HashSet<>(queries);
        maxK = Collections.max(queries);
        dfs(root);
        return result;
    }

    private void dfs(TreeNode node) {
        if (node == null || count >= maxK) return;

        dfs(node.right);

        count++;
        if (ks.contains(count)) {
            result.put(count, node.val);
        }

        dfs(node.left);
    }
}

```

## LC 815 – 多源 BFS

这类题 Uber 非常喜欢 BFS/graph 方向。
我们在进入 coding 之前就先提醒过学员：Uber 更关注你的 problem reasoning，而不是直接敲代码。

于是学员结构化地把这几个点讲得很清楚：

图如何建（bus → stops or stops → bus）

BFS 为什么适合

是否需要访问去重

时间复杂度 O(N + M) 的解释

在他讲到 third step 的时候，我通过语音提示补了一句让他问面试官：“这里按 bus-level 还是 stop-level build graph，你有没有偏好？”
这句话非常关键，让整个交流变得更自然，也让面试官觉得你是“与面试官合作解决问题”的那种工程师。

代码部分写得干净，测试用例过得很稳。


## LC 269 – Hard 拓扑

这题难度不低，非常考察：

如何从字符串关系构建有向图
如何确定 invalid 情况（prefix edge case）
Topological sort 的口述稳定性
学员当时进房间时有点紧张，我通过语音轻轻提醒：“先别想代码，先画示例，再说图怎么建。”
这句话非常关键，把他的节奏稳住了。

整轮下来，他的思路讲得非常像“成熟 backend 工程师”的版本：

建图 → 入度数组 → BFS
prefix invalid 的检查
多答案时是否需要处理
面试官基本没有质疑他的逻辑。

## 员工层级树 + 找共同上级

面试官是一位白人小哥，开场非常友好，寒暄完直接进入技术环节。题目核心其实就是 Lowest Common Ancestor (LCA) 的变体，只不过是以 Uber 的员工体系为背景包装。

```
题目描述：
公司层级结构是一棵树，从 CEO 到普通员工，每个员工节点包含：

id
name
指向其直接经理（mgr）的引用
以及所有直接下属（directReports）的引用

实现一个 whoIsYourBoss(emp1, emp2) 方法，返回他们的最近共同上级的 ID。
```

我先问清楚了几个关键点，这一步其实很重要：

每个人只有一个直属经理吗？

CEO 没有经理？

输入同一个员工怎么办？➡️ 返回自身。

如果节点不存在树中？➡️ 返回 null。

这类树题最容易出问题的地方就是这些“边界条件”，问清楚反而能给面试官留下良好的沟通印象。

实现部分我采用常规 LCA 思路：

自底向上回溯（记录父节点路径）。
若公司规模大，也可提前存 manager 映射 + visited 集合判断。

Follow-up 比较有意思：

“如果节点不包含指向经理的引用，只能从 CEO 出发搜索，该如何实现？”

这时不能用哈希路径法，需要用 DFS 从上往下搜。我写完两个版本（自底向上 + 自顶向下）后，面试官表示很满意。

## 找出只出现一次的用户 ID

题目描述：
给定一个数组，表示一段时间内访问 Uber 系统的用户 ID，找出只出现过一次的 ID。

关键点是“出现次数不固定”，我没直接问“是不是每个元素都出现两次”，而是用举例法确认：

“例如，这样的输入合理吗？ [1,1,2,2,2,3,4,4]”

面试官点头确认。

我先写了一个最直观的哈希表解法：

遍历统计频次
输出频次为 1 的 ID
Follow-up 是进阶版：

如果每个重复 ID 只出现两次呢？

我立刻切换到 XOR 解法，O(n) 时间 + O(1) 空间，完美回答收尾。

## [prachub](https://prachub.com/companies/uber/positions/software-engineer/categories/coding-and-algorithms)

## Detect cycle in directed dependency graph

You are given a directed dependency graph representing services in a system.

*   There are `n` services, labeled from `0` to `n - 1` .
*   You are given a list of directed edges `dependencies` , where each element is a pair `[a, b]` meaning **service `a` depends on service `b`** (i.e., there is a directed edge `a -> b` ).

Assume:

*   `1 <= n <= 10^5`
*   `0 <= a, b < n`
*   There may be zero or more edges.

**Task:**

1.  Implement a function that determines whether there is at least one cycle in this directed graph.
    *   Return `true` if there is a cycle.
    *   Return `false` otherwise.
2.  Follow-up: Now you are given **multiple dependency arrays** instead of just one. Each array represents an additional list of edges on the same set of services. For example:
    
    *   `dependencies1` , `dependencies2` , ..., `dependenciesk`
    
    Each `dependenciesi` is a list of `[a, b]` pairs as defined above. Treat the union of all these edges as a single directed graph and determine whether this combined graph contains any cycle.

## Best Time to buy Stock

You are given an array `prices` where `prices[i]` is the price of a given stock on day `i` (0-indexed).

You want to maximize your profit by choosing when to buy and sell this stock, subject to the constraints below. You may not hold more than one share at a time (i.e., you must sell before you can buy again).

Assume:

*   `1 <= n <= 10^5` , where `n = prices.length` .
*   `0 <= prices[i] <= 10^4` .

### [](#part-a-at-most-one-transaction)Part A: At most one transaction

You are allowed to complete **at most one** transaction (i.e., buy once and sell once).

*   Return the **maximum profit** you can achieve.
*   If no profit is possible, return `0` .

**Example**

*   Input: `prices = [7, 1, 5, 3, 6, 4]`
*   Output: `5`
*   Explanation: Buy on day 1 (price = 1) and sell on day 4 (price = 6), profit = 6 − 1 = 5.

### [](#part-b-at-most-two-transactions)Part B: At most two transactions

Now you are allowed to complete **at most two transactions**.

*   Each transaction is a buy followed by a sell.
*   You must sell the stock before you buy again.
*   You cannot engage in multiple transactions at the same time.

Return the **maximum total profit** you can achieve with at most two transactions.

**Example**

*   Input: `prices = [3, 3, 5, 0, 0, 3, 1, 4]`
*   Output: `6`
*   Explanation:
    *   Buy on day 3 (price = 0), sell on day 5 (price = 3), profit = 3.
    *   Then buy on day 6 (price = 1), sell on day 7 (price = 4), profit = 3.
    *   Total profit = 3 + 3 = 6.

Design algorithms for both parts that run in **O(n)** time and **O(1)** or **O(n)** extra space.


## Hit Counter and Simple Rate Limiter


### [](#task-3-hit-counter-and-simple-rate-limiter)Task 3: Hit Counter and Simple Rate Limiter

You are asked to design two related components.

### [](#3a-hit-counter-for-last-5-minutes)3a. Hit Counter for Last 5 Minutes

Design an in-memory hit counter with the following API:

*   `hit(timestamp)` : Record a hit at time `timestamp` (in seconds).
*   `getHits(timestamp)` : Return the number of hits received in the **past 300 seconds** (5 minutes), including at `timestamp` .

Assume:

*   `timestamp` is a positive integer representing seconds.
*   Calls to `hit` and `getHits` use **non-decreasing** timestamps (i.e., new timestamps are never in the past).

Design the data structures and algorithms so that both operations are efficient in time and memory, even when the number of hits is large.

### [](#3b-extend-to-a-per-user-request-rate-limiter)3b. Extend to a Per-User Request Rate Limiter

Now extend your design to implement a simple **per-user rate limiter**. Provide an API like:

*   `shouldAllow(userId, timestamp) -> bool`

This should:

*   Return `true` if the given `userId` has made **fewer than `N` requests** in the last `W` seconds (including the current one), and the new request should be allowed.
*   Return `false` if allowing this request would exceed the configured rate limit of `N` requests per rolling window of `W` seconds.

You may assume:

*   `N` and `W` are given configuration parameters (for example, `N = 100` , `W = 60` seconds).
*   `timestamp` is again in seconds and **non-decreasing** per process.

Specify:

*   What data structures you would use.
*   How you would update and clean up data as time moves forward.
*   The expected time and space complexity per call.

## Compute outer boundary of an N-ary tree

[545. 二叉树的边界](https://leetcode.cn/problems/boundary-of-binary-tree/description/)

You are given the root of a rooted **N-ary tree**. Each node contains:

*   An integer value.
*   An ordered list of children from left to right (0 or more children).

Consider all nodes grouped by their depth (root at depth 0, its children at depth 1, etc.). For each depth `d`:

*   Let **L\_d** be the _leftmost_ node at depth `d` (the first node at that depth when scanning children from left to right).
*   Let **R\_d** be the _rightmost_ node at depth `d` (the last node at that depth when scanning children from left to right).

We define the **outer boundary path** of the tree as the sequence of nodes you would "see" when walking around the outside of the tree, starting from the lowest leftmost node, going up to the root, then going down to the lowest rightmost node:

1.  Starting from the **maximum depth D** down to `0` , take `L_D, L_{D-1}, ..., L_0` (bottom-left up to the root).
2.  Then from depth `1` up to `D` , take `R_1, R_2, ..., R_D` (top-right down to bottom-right), **skipping** `R_d` when `R_d` is the same node as `L_d` to avoid duplicates on levels with only one node.

Return the list of node values in this exact order.

Assume:

*   Total number of nodes `n` satisfies `1 <= n <= 10^5` .
*   Tree height `h` can be up to `n` in the worst case.

**Task**: Design an efficient algorithm to compute and return this outer boundary path. Specify the time and space complexity of your approach.



## [](#problem-1-allocate-tasks-between-two-workers-to-maximize-reward)Problem 1: Allocate tasks between two workers to maximize reward

You have **n** independent tasks that must be split between two workers (worker 1 and worker 2). Every task must be done by **exactly one** of the two workers.

For each task iii (0-indexed or 1-indexed, as you prefer), you are given:

*   `reward1[i]` : the reward if **worker 1** does task iii
*   `reward2[i]` : the reward if **worker 2** does task iii

Additionally, worker 1 must do **exactly k** tasks in total (where 0≤k≤n0 \\le k \\le n0≤k≤n). The remaining n−kn - kn−k tasks must be done by worker 2.

**Task**:

*   Assign each of the nnn tasks to one of the two workers such that:
    *   Worker 1 is assigned exactly kkk tasks.
    *   Worker 2 is assigned the remaining tasks.
*   **Maximize** the total reward, defined as the sum over all tasks of the reward from the worker to whom the task is assigned.

**Input format (conceptual)**:

*   Integer `n` — number of tasks.
*   Integer `k` — number of tasks that must be assigned to worker 1.
*   Array `reward1` of length `n` .
*   Array `reward2` of length `n` .

**Output**:

*   A single number: the **maximum possible total reward** .
*   (Optionally, you may also be asked to output which task indices are assigned to worker 1, but the core problem is computing the maximum total reward.)

You should design an algorithm that works efficiently for large nnn (e.g., up to 10510^5105 or 10610^6106).

## [](#problem-2-reorient-edges-in-a-directed-tree-toward-a-root)Problem 2: Reorient edges in a directed tree toward a root

You are given a directed graph with `n` nodes labeled from `1` to `n` and `n - 1` directed edges. It is guaranteed that if you **ignore edge directions**, the underlying undirected graph is a tree (i.e., it is connected and acyclic). However, the current directions of the edges may be arbitrary, so the graph may not currently have a single node reachable from all others.

You are allowed to **reverse the direction** of any subset of edges. Reversing an edge counts as **one operation**.

You want to make the graph such that **there exists some node `r`** (1 ≤ `r` ≤ `n`) with the following property:

*   For **every** node `x` (including `r` ), there is a **directed path** from `x` to `r` following the directions of the (possibly reversed) edges.

In other words, after reorienting some edges, all nodes should be able to "flow" to a single root node `r` along directed paths.

**Task**:

*   Over all possible choices of root node `r` , determine the **minimum number of edge reversals** needed to achieve the property that every node has a directed path to `r` .

**Input format (conceptual)**:

*   Integer `n` — number of nodes.
*   `n - 1` directed edges, each given as an ordered pair `(u, v)` meaning there is an edge from `u` to `v` .

**Output**:

*   A single integer: the **minimum number of edges that must be reversed** so that for some node `r` , every node has a directed path to `r` .
*   (Optionally, you may also be asked to output at least one node `r` that achieves this minimum.)

Your algorithm should run in time close to linear in `n` (e.g., O(n)O(n)O(n) or O(nlog⁡n)O(n \\log n)O(nlogn)).

## [](#problem-3-find-the-leftmost-column-with-a-1-in-a-sorted-binary-matrix)Problem 3: Find the leftmost column with a 1 in a sorted binary matrix

You are given a binary matrix `A` of size `m × n` (mmm rows, nnn columns). Each cell is either 0 or 1. **Each row is sorted in non-decreasing order**, meaning:

*   All the 0s in a row come **before** any 1s in that row.
*   A typical row therefore looks like: `0 0 0 1 1 1` (possibly all 0s or all 1s).

**Task**:

*   Find the **index of the leftmost column** (smallest column index) that contains at least one `1` **in any row** .
*   If the matrix contains **no 1s at all** , return `-1` .

You may assume 0-based column indexing (return an integer in `[0, n-1]`) or 1-based indexing (return in `[1, n]`); just be clear and consistent in your implementation.

**Input format (conceptual)**:

*   Integers `m` , `n` — number of rows and columns.
*   An `m × n` matrix `A` of 0s and 1s, where each row is sorted: all 0s, then all 1s.

**Output**:

*   A single integer: the index of the **leftmost column** that contains at least one `1` , or `-1` if no such column exists.

**Complexity requirement**:

*   Design an algorithm that is **asymptotically faster than** the naive O(m×n)O(m \\times n)O(m×n) scan of the entire matrix in the worst case (for example, O(mlog⁡n)O(m \\log n)O(mlogn) or O(m+n)O(m + n)O(m+n) ).
*   Be prepared to explain your algorithm and analyze its time complexity.


## Design top-K frequency structure

Design an in-memory data structure that supports: add(x) to observe an item, inc(x) to increment its frequency, dec(x) to decrement (deleting when zero), and topK(k) to return the k items with highest frequency, breaking ties by most recent update. Target O(

amortized updates and O(k) retrieval using hash maps plus a doubly linked list of frequency buckets. Describe node layout, how buckets are created/merged, how recency is tracked, tie-breaking, memory bounds under churn, and concurrency considerations.

## [2021. 街上最亮的位置](https://leetcode.cn/problems/brightest-position-on-street)

## Find next and closest palindromes

[Find next and closest palindromes](https://prachub.com/interview-questions/find-next-and-closest-palindromes)

## [Implement 2D word search variants and analyze complexity](https://prachub.com/interview-questions/implement-2d-word-search-variants-and-analyze-complexity)

## [Find earliest common slot](https://prachub.com/interview-questions/find-earliest-common-slot)

## [Solve vault rate and subset-sum](https://prachub.com/interview-questions/solve-vault-rate-and-subset-sum)

## [Sort by squares and find k-th smallest](https://prachub.com/interview-questions/sort-by-squares-and-find-k-th-smallest)

## [Implement binary search variants](https://prachub.com/interview-questions/implement-binary-search-variants)

## [Solve minimum rate and subset sum](https://prachub.com/interview-questions/solve-minimum-rate-and-subset-sum)

## [Simulate fixed-shape tiling on a grid](https://prachub.com/interview-questions/simulate-fixed-shape-tiling-on-a-grid)

## [Simulate one-round color elimination on a grid](https://prachub.com/interview-questions/simulate-one-round-color-elimination-on-a-grid)

## [Count subarrays with at least k fruit pairs](https://prachub.com/interview-questions/count-subarrays-with-at-least-k-fruit-pairs)

## [Produce a valid deployment order](https://prachub.com/interview-questions/produce-a-valid-deployment-order)

## [Count subarrays with ≥k identical pairs](https://prachub.com/interview-questions/count-subarrays-with-k-identical-pairs)

## [Perform matrix Candy Crush elimination](https://prachub.com/interview-questions/perform-matrix-candy-crush-elimination)

## [Find minimal rate k and subset sum](https://prachub.com/interview-questions/find-minimal-rate-k-and-subset-sum)

## [Implement rate limiter, top-K, scheduler algorithms](https://prachub.com/interview-questions/implement-rate-limiter-top-k-scheduler-algorithms)

## [Handle invalid Lisp expression parsing](https://prachub.com/interview-questions/handle-invalid-lisp-expression-parsing)

## [Design top-K tracker with linked lists](https://prachub.com/interview-questions/design-top-k-tracker-with-linked-lists)

## [Enforce ordered execution across threads](https://prachub.com/interview-questions/enforce-ordered-execution-across-threads)

## [Find earliest common meeting slot](https://prachub.com/interview-questions/find-earliest-common-meeting-slot)

## [Implement binary search from scratch](https://prachub.com/interview-questions/implement-binary-search-from-scratch)

## [Design time-versioned KV without timestamp argument](https://prachub.com/interview-questions/design-time-versioned-kv-without-timestamp-argument)

## [Parse and evaluate nested expressions with validation](https://prachub.com/interview-questions/parse-and-evaluate-nested-expressions-with-validation)

## [Simulate robot on grid with obstacles](https://prachub.com/interview-questions/simulate-robot-on-grid-with-obstacles)

## [Implement uniform sampling in squares](https://prachub.com/interview-questions/implement-uniform-sampling-in-squares)