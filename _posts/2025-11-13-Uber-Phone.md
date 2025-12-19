---
layout: post
title: "UB Phone"
date: 2025-06-24
description: "interview problems"
tag: Algorithms
---


## Graph 成环问题（Service Dependency）

### 基本解法
- 用 DFS + 三色标记，判断有向图是否存在环
- 或使用 拓扑排序（Topological Sort），若无法完成所有节点的排序则说明存在环

### Follow-up 1：多个 dependency array 如何处理？
- 可以先对每个 dependency array 单独建图并分别判断是否有环
- 若每个 array 均无环，再将它们合并成一个整体图进行最终判断（Batch Processing）
- 也可以直接将所有 dependency array 合并成一个图，一次性判断是否存在环
- 取决于需求是更关注整体正确性，还是需要定位具体哪一批依赖出问题

### Follow-up 2：现实工作中的优化（增量式检查）
- 在真实系统中，依赖关系往往是动态新增的
- 对于新增的一条依赖边 A -> B：
- 是否形成环 ⇔ 当前图中是否已经存在从 B 到 A 的路径

#### 判定方式
- 对新增边执行一次 reachability check
- 从 B 出发，使用 DFS / BFS 判断是否能够到达 A

#### 结果处理
- 能到达 A：
- 加这条边会形成环
- reject / 报错
- 不能到达 A：
- 安全
- 将该边加入图中

### 缓存优化（Reachability Cache）
- 可以缓存节点之间的可达性关系，例如 canReach
𝑎
a
𝑏
b
- 新增依赖时优先从缓存中判断是否可达，避免重复 DFS / BFS
- 缓存策略可以包括：
- 只缓存高访问频率的节点
- 使用 LRU 缓存最近 N 次查询结果
- 在依赖结构发生变化时，可选择精确更新相关缓存或直接失效缓存


## 汇率转换

[汇率转换](https://www.hack2hire.com/companies/uber/coding-questions/67895195402dc47b67a14950/practice?questionId=678951e8402dc47b67a14951)

```java
class Edge {
    String to;
    double rate;

    Edge (String to, double rate) {
        this.to = to;
        this.rate = rate;
    }
}
class CurrencyConverter {
    Map<String, List<Edge>> graph = new HashMap<>();
    public CurrencyConverter(String[] fromArr, String[] toArr, double[] rateArr) {
        // TODO: Initialize CurrencyConverter
        for (int i = 0; i < fromArr.length; i++) {
            Edge A = new Edge(toArr[i], rateArr[i]);
            Edge B = new Edge(fromArr[i], 1 / rateArr[i]);
            graph.computeIfAbsent(fromArr[i], a -> new ArrayList<>()).add(A);
            graph.computeIfAbsent(toArr[i], a -> new ArrayList<>()).add(B);
        }
    }

    public double getBestRate(String from, String to) {
        // TODO: Implement getBestRate logic
        if (!graph.containsKey(from) || !graph.containsKey(to)) return -1;
        return dfs(from, to, 1, new HashSet<>());

    }
    public double dfs (String cur, String to, double rate, Set<String> visit) {
        if (cur.equals(to)) {
            return rate;
        }
        visit.add(cur);
        double ans = -1;
        for (Edge e : graph.get(cur)) {
            if (!visit.contains(e.to)) {
                double res = dfs(e.to, to, rate * e.rate, visit);
                if (res > ans) {
                    ans = res;
                }
            }
        }
        visit.remove(cur);
        return ans;
    }
}
```

## Leftmost Column with at Least a One (Row-Sorted Binary Matrix)

Problem: Leftmost Column with at Least a One (Row-Sorted Binary Matrix)

You are given a binary matrix of size m × n containing only 0 and 1.

Each row is sorted in non-decreasing order, meaning all 0s appear before any 1s in that row.

Return the index (0-based) of the leftmost column that contains at least one 1.
If no such column exists (i.e., the matrix contains only 0s), return -1.

```
Example 1:
Input:
matrix =
[
  [0, 0, 0, 1],
  [0, 0, 1, 1],
  [0, 1, 1, 1]
]
Output: 1
Explanation: Column 1 is the first column that has a 1 (at row 2).

Example 2:
Input:
matrix =
[
  [0, 0],
  [0, 0]
]
Output: -1
Explanation: There is no 1 in the matrix.
```

Constraints:
- m == number of rows, n == number of columns
- 1 <= m, n
- matrix[i][j] is either 0 or 1
- Each row is sorted: matrix[i][0] <= matrix[i][1] <= ... <= matrix[i][n-1]

二分法：O(m * logn)

走楼梯从右上角，如果是1则往左，遇到0则往下 O(m + n)


## 564. Find the Closest Palindrome



---------------------------------------------------------------------------------------------

phone screen: [1438](https://leetcode.cn/problems/longest-continuous-subarray-with-absolute-diff-less-than-or-equal-to-limit/)

单调队列维护min和max

```java
class Solution {
    public int longestSubarray(int[] nums, int limit) {
        int l = 0;
        int ans = 0;
        Deque<Integer> max = new ArrayDeque<>();
        Deque<Integer> min = new ArrayDeque<>();
        for (int r = 0; r < nums.length; r++) {
            while (!max.isEmpty() && nums[max.peekLast()] <= nums[r] ) {
                max.pollLast();
            }
            max.offerLast(r);
            while (!min.isEmpty() && nums[min.peekLast()] >= nums[r] ) {
                min.pollLast();
            }
            min.offerLast(r);
            while (nums[max.peekFirst()] - nums[min.peekFirst()] > limit) {
                l++;
                if (max.peekFirst() < l) {
                    max.pollFirst();
                }
                if (min.peekFirst() < l) {
                    min.pollFirst();
                }
            }
            ans = Math.max(ans, r - l + 1);
        }
        return ans;
    }
}
```

[305 number of islands 1 & 2](https://leetcode.cn/problems/number-of-islands-ii/description/)

第三问return currentMaxIslandSize

coding 三流而 然后rate limiter扩展

SD 是Uber eats 的search function

BQ 就是通常的BQ


---------------------------------------------------------------------------------------------

Phone Screen Problem: Determine the minimum number of characters required to append to the end of a string to convert it into a palindrome

[214](https://leetcode.cn/problems/shortest-palindrome/description/)

Onsite Round 1: Data Structures & Algorithms Problem: Given an integer $x$ and an array, determine if it is possible to make the entire array equal to a target number $y$. The operation allowed is adding or subtracting a value between $0$ and $x$ to each element in the array.

[2779](https://leetcode.cn/problems/maximum-beauty-of-an-array-after-applying-operation/description/)

Initial Case: $x = 1$.
Follow-up: $x \ge 1$. Result: Provided multiple solutions.

---------------------------------------------------------------------------------------------

* Given two inputs,
* First input is the location map, a 2D array
* | O | E | E | E | X |
* | E | O | X | X | X |
* | E | E | E | E | E |
* | X | E | O | E | E |
* | X | E | X | E | X |
* O = Robot, E = Empty, X = blocker
* {{'O','E','E','E','X'},{'E','O','X','X','X'},{'E','E','E','E','E'},{'X','E','O','E','E'},{'X','E','X','E','X'}}
* Second input is the query. It’s a 1D array consisting of distance to the closest blocker in the order from left, top, bottom and right
* [2, 2, 4, 1] -> This means distance of 2 to the left blocker, 2 to the top blocker, 4 to the bottom blocker and 1 to the right blocker
* The location map boundary is also considered blocker, meaning if the robot hits the boundary it also means it’s hitting the blocker.
说了思路，然后结束了。整个一轮都没有写代码

```
Left：每行从左到右，last = -1（左边界），遇到 X 更新 last，否则距离 L = c - last

Right：每行从右到左，last = n（右边界），遇到 X 更新 last，否则距离 R = last - c

Up：每列从上到下，last = -1（上边界），遇到 X 更新 last，否则距离 U = r - last

Down：每列从下到上，last = m（下边界），遇到 X 更新 last，否则距离 D = last - r

最后只遍历 O：
if (L==qL && U==qU && D==qD && R==qR) -> add (r,c)。
```

75min project & bq，像其他面经一样，准备一个slides会很有用

sd：design instagram

uber面完hr直接失踪，发邮件也不回，不知道什么时候才能给结果。。。其实都是面经题，但是uber电面和onsite直接隔了太久，楼主之前准备的都忘了。。。。

---------------------------------------------------------------------------------------------

写一个Counter class。每个被放进Counter的元素会有一个过期时间。这个过期时间会在Counter初始化的时候传入。

要求实现三个function:
- put(element): 将一个element放入Counter
- get_count(element): 返回当前该element在Counter中的有效数量
- get_total_count(): 返回当前Counter中所有有效element的数量

例如:
counter(window=10)
time 1: counter.put(‘a’)
time 3: counter.put(‘a’)
time 5: counter.put(‘b’)
time 6: counter.get_count(‘a’) -> 2
time 6: counter.get_total_count() -> 3
time 12: counter.get_count(‘a’) -> 1
time 12: counter.get_total_count() -> 2

### 解法 A（推荐）：全局队列 + HashMap 计数

- Deque<Event> events  
  按时间递增存储 `(timestamp, element)`

- Map<Element, Integer> freq  
  记录当前有效窗口内每个 element 的数量

- int total  
  当前窗口内所有有效 element 的总数量  
  （也可以用 events.size()，但单独维护 total 更直观）

### 核心思想：过期清理 evict(now)

- 窗口大小：`window`
- 有效条件：  
  `ts > now - window`

示例说明（window = 10）：
- time = 12 时  
  - ts = 1 → 1 <= 2 → 过期  
  - ts = 3 → 3 > 2 → 仍然有效

因此 **过期条件** 为：



---------------------------------------------------------------------------------------------

 骑👂🍷


Given a sorted array and an elephant's path length, use binary search to find the value in the path that is closest to the target path length.

Implement the function:

def find_elephant_path(paths: List[int], target_length: int) -> int:
Function parameters:

paths: An integer list representing sorted path lengths.
target_length: An integer representing elephant's target path length.
Return:

The path length closest to the target path length.
Requirement:

Implement using binary search algorithm with a time complexity of O(log n).
Example:

Input: paths = [1, 3, 7, 10, 14, 20], target_length = 8
Output: 7
Input: paths = [2, 5, 12, 17, 27], target_length = 20
Output: 17

### 就是找离target最近的点

---------------------------------------------------------------------------------------------

Given a string s, find the longest odd-length palindromic substring within it. The substring's length must be odd and among all such substrings, it should have the maximum length. You may assume the maximum length of s is 1000.

Input:

A string s of length [1, 1000] composed of lowercase English letters.
Output:

The longest odd-length palindromic substring.
Test cases:

Input: "babad"
Output: "bab"

Input: "cbbd"
Output: "b"

Input: "racecar"
Output: "racecar"

Input: ""
Output: ""

Input: "a"
Output: "a"

---------------------------------------------------------------------------------------------

Alien Dictionary (LeetCode Premium)

---------------------------------------------------------------------------------------------

coding one：给一个送货服务的couriers的时间表，couriers[i] = [start, end]是第i个courier的工作时间段，start是inclusive，end时间是exclusive的。每个courier的interval可能有重叠。问题是要你返回一个list of sublists，条件是只有当同时处在工作中的courier的人数改变的时候（无论是增加或减少），你才需要输出某个sublist。举个例子，如果couriers = [[1, 5], [3, 7]]的话，那么输出应该为 [[1, 3, 1], [3, 5, 2], [5 ,7, 1]]，其中每个sublist的第三个element表示同时处于工作中的人数。
system design：设计一个类似于facebook messenger的聊天系统。
previous project + BQ。previous project要求画系统设计图。问我当时项目如何规划的（时间，人员方面），如果再做一次这个项目，会有什么样的改进。BQ都是比较常见的问题，比如发生conflict时如何处理的，如何帮助junior组员进步等等。
coding two：一上来问我一道和phone screen一样的题目，我说做过了，然后就临时换了一道：给一个BST，要求返回第K个最大的元素。要求自己去生成BST，我就写了一个简单的node class来生成BST。

---------------------------------------------------------------------------------------------


店面：
外星人字典

VO：
code1： QuadTree，套了个图像压缩的壳子
很快写完了，需要自己写 test case，我的方法是 dfs leaf 然后 print leaf val
follow up1：如何优化，没想出来。答案是用 CPU 的多线程
follow up2：怎么decode，如果decode需要怎么改。这个不用写代码，讲思路就行。
考完后想了想，可能面试官期待的test方法是 decode quadtree 然后比较 input array 和 decoded array，这种方法应该比较优雅


Implement a Quadtree algorithm for image compression and optimize it for performance. The initial implementation should include a function that takes a 2D integer array image representing the image pixels and returns the root node of the compressed Quadtree.

Test Case
Input
image = [[...], [...], ...]

Output
root = QuadTreeNode

Requirements
Write unit tests to verify the compression correctness.
Provide an optimization method, such as utilizing CPU multi-threading.
Discuss decoding the Quadtree to reconstruct the original image, no code needed.
Constraints
Image size is $n \times n$, where $n \leq 1024$.


code2：上来就说要做俩道题
第一道：酒齐齐
已排序的数组按绝对值重新排序，有 TIE 的时候负数排在前面
[-5, -4, -2, 0, 1, 2, 3] => [0, 1, -2, 2, 3, -4, -5]
双指针 O(n) 秒了
第二道：也是第一道的 follow up
还是这道题找第 k 大的 直接说了 O(log n) 的 bianry search 方法，用的思路就是 find k-th in two sorted array lc 第四题的思路。给面试官讲思路就用了将近 20 多分钟，最后 20 分钟写完。 这个题面经出现过，我也拿 GPT 和 lc4 练过，没什么问题。

Re-sort a sorted array by absolute values, with negative numbers taking precedence in ties. Write a function that takes an integer array nums as input and outputs the array sorted by absolute values.

Test Case
Input
nums = [-5, -4, -2, 0, 1, 2, 3]

Output
[0, 1, -2, 2, 3, -4, -5]

Requirements
Time complexity: O(n)
Space complexity: O(1)
Constraints
$-10^9 \leq \text{nums}[i] \leq 10^9$
$1 \leq n \leq 10^5$


---------------------------------------------------------------------------------------------

第一轮是利口 巴耳漆 我phone screen也考的这个，第二轮是meeting room变种 第三轮是design Uber第四轮是bq 和hm
最近在准备面试求米！

人工岛，把一个变成1，问连接的岛最大值

```java
class Solution {
    int[] xes = new int[] {0, 0, 1, -1};
    int[] yes = new int[] {1, -1, 0, 0};
    public int largestIsland(int[][] grid) {
        int m = grid.length;
        int n = grid[0].length;
        int idx = 2;
        int ans = 0;
        List<Integer> area = new ArrayList<>();
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == 1) {
                    int res = dfs(grid, i, j, idx++);
                    area.add(res);
                    ans = Math.max(res, ans);
                }
            }
        }
        Set<Integer> around = new HashSet<>();
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] != 0) continue;
                
                int s = 1;
                for (int c = 0; c < 4; c++) {
                    int nx = i + xes[c];
                    int ny = j + yes[c];
                    if (nx >= 0 && nx < m && ny >= 0 && ny < n) {
                        around.add(grid[nx][ny]);
                    } else {
                        around.add(0);
                    }
                }

                for (int index : around) {
                    s += index == 0 ? 0 : area.get(index - 2);
                }
                ans = Math.max(ans, s);
                around.clear();
            }
        }
        return ans;
    }
    public int dfs(int[][] grid, int x, int y, int index) {
        int m = grid.length;
        int n = grid[0].length;
        grid[x][y] = index;
        int ans = 1;

        for (int i = 0; i < xes.length; i++) {
            int nx = x + xes[i];
            int ny = y + yes[i];
            if (nx >= 0 && nx < m && ny >= 0 && ny < n && grid[nx][ny] == 1) {
                ans += dfs(grid, nx, ny, index);
            }
        }
        return ans;
    }
}
```


SD: 购物网站首页展示 popular 商品
也算高频题，我是按 click 商品的次数定义popularity，总体用 ads click aggregation + top k 的思路。 因为前面面试官自我介绍和问项目用了20多分钟，我怕时间不够刚开始答的有些着急，miss 了一些 requirement 和 API。 面试官提醒后才加上。图画的应该没什么问题，核心的 kafka + flink + redis 都讨论了存什么数据以及scalability，kafka 的 partition key， 还写了 flink 的 pseudo code。还有就是展示商品的popular要最近多长时间的，flink 时间窗口设多大等等。

---------------------------------------------------------------------------------------------


时间线：
7月份：Uber Recruiter Call
8月份：Phone Call for Coding
9月份：Onsite
10月份：出结果
phone:
利口 121, Follow Up 还是比较丰富的，包括不同方法求解这个问题
VO:
第一轮算法, Convex Function 求最小值 （Binary Search的思想）
第二轮算法，Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit
第三轮behavioral, 主要介绍之前的相关工作经历，提前准备PPT很有帮助
第四轮设计，推荐系统。
过程虽有波折，总体感觉不错，面试官人都挺友善。求加米！

---------------------------------------------------------------------------------------------

面了 schedule meeting
首先叫我写了linear 的算法
后面improve成 logn 用了treemap

Map<Id, Meeting>
TreeMap<StartTime, Meeting>
ListNode head;
ListNode tail; --> double linked list store last N;

followup 有三个
1.加上一个方法返回last n meetings
2.加上一个方法取消meeting last n meetings也要更新
3.如果是concurrent request 怎么改code 我加了一个global lock 然后follow up问我怎么改进我说可以room level lock但没时间写了
面完以后也不知道算不算过了 题不难但是感觉考察得很杂

---------------------------------------------------------------------------------------------
