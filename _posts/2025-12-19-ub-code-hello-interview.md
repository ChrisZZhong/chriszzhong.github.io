---
layout: post
title: "UB coding hello interview"
date: 2025-12-19
description: "UB coding hello interview"
tag: Algorithms
prime: false
---


# Problem

## - [‼️] [Leetcode 269. Alien Dictionary](https://leetcode.cn/problems/alien-dictionary)

```java
class Solution {
    public String alienOrder(String[] words) {
        Set<Integer>[] graph = new HashSet[26];
        Arrays.setAll(graph, _ -> new HashSet<>());

        int[] indegree = new int[26];
        boolean[] present = new boolean[26];

        // 1) 统计出现过的字符
        for (String w : words) {
            for (int i = 0; i < w.length(); i++) {
                present[w.charAt(i) - 'a'] = true;
            }
        }

        // 2) 只比较相邻单词，构建有向边
        for (int i = 0; i < words.length - 1; i++) {
            String l = words[i];
            String r = words[i + 1];

            int minLen = Math.min(l.length(), r.length());
            boolean foundDiff = false;

            for (int k = 0; k < minLen; k++) {
                if (l.charAt(k) != r.charAt(k)) {
                    int x = l.charAt(k) - 'a';
                    int y = r.charAt(k) - 'a';

                    // 只在第一次加这条边时才增加入度（避免重复计数）
                    if (graph[x].add(y)) {
                        indegree[y]++;
                    }

                    foundDiff = true;
                    break;
                }
            }

            // 前缀非法：比如 "abc" 在 "ab" 前面
            if (!foundDiff && l.length() > r.length()) return "";
        }

        // 3) 拓扑排序（只处理出现过的字符）
        Deque<Integer> queue = new ArrayDeque<>();
        int total = 0;
        for (int i = 0; i < 26; i++) {
            if (present[i]) {
                total++;
                if (indegree[i] == 0) queue.offerLast(i);
            }
        }

        StringBuilder sb = new StringBuilder();
        while (!queue.isEmpty()) {
            int cur = queue.pollFirst();
            sb.append((char) (cur + 'a'));

            for (int nxt : graph[cur]) {
                if (--indegree[nxt] == 0) {
                    queue.offerLast(nxt);
                }
            }
        }

        // 没有输出完所有出现过的字符 => 有环/不合法
        return sb.length() == total ? sb.toString() : "";
    }
}
```

## - [‼️] [Leetcode 247. Strobogrammatic Number II](https://leetcode.cn/problems/strobogrammatic-number-ii)

```java
class Solution {
    private static final char[][] PAIRS = {
        {'0','0'}, {'1','1'}, {'6','9'}, {'8','8'}, {'9','6'}
    };

    public List<String> findStrobogrammatic(int n) {
        return build(n, n);
    }

    private List<String> build(int n, int totalLen) {
        if (n == 0) return List.of("");
        if (n == 1) return List.of("0", "1", "8");

        List<String> res = new ArrayList<>();
        for (String mid : build(n - 2, totalLen)) {
            for (char[] p : PAIRS) {
                // 最外层不能用 0
                if (n == totalLen && p[0] == '0') continue;
                res.add(p[0] + mid + p[1]);
            }
        }
        return res;
    }
}
```

## - [‼️] [Leetcode 384. Shuffle an Array](https://leetcode.cn/problems/shuffle-an-array) 

```java
public int[] shuffle() {
    int[] res = nums.clone();
    Random rd = new Random();
    for (int i = res.length - 1; i > 0; i--) {
        int j = rd.nextInt(i + 1);
        int tmp = res[i];
        res[i] = res[j];
        res[j] = tmp;
    }
    return res;
}
```

## - [‼️] [Leetcode 305. Number of Islands II](https://leetcode.cn/problems/number-of-islands-ii)

```java
class Solution {
    int[] father;
    int size;
    
    public void build(int m, int n) {
        father = new int[m * n];
        for (int i = 0; i < m * n; i++) {
            father[i] = i;
        }
        size = 0;
    }

    public int find(int x) {
        int root = x;
        while (root != father[root]) {
            root = father[root];
        }
        while (x != father[x]) {
            int tmp = father[x];
            father[x] = root;
            x = tmp;
        }
        return root;
    }

    public void union(int x, int y) {
        int fx = find(x);
        int fy = find(y);
        if (fx != fy) {
            father[fx] = father[fy];
            size--;
        }
    }

    public List<Integer> numIslands2(int m, int n, int[][] positions) {
        int[][] map = new int[m][n];
        build(m, n);
        int[] xes = new int[] {0, 0, -1, 1};
        int[] yes = new int[] {-1, 1, 0, 0};
        List<Integer> ans = new ArrayList<>();
        for (int[] point : positions) {
            int x = point[0];
            int y = point[1];
            if (map[x][y] == 1) {
                ans.add(size);
                continue;
            }
            map[x][y] = 1;
            size++;
            for (int i = 0; i < 4; i++) {
                int nx = x + xes[i];
                int ny = y + yes[i];
                if (nx >= 0 && nx < m && ny >= 0 && ny < n && map[nx][ny] == 1) {
                    int a = x * n + y;
                    int b = nx * n + ny;
                    union(a, b);
                }
            }
            ans.add(size);
        }
        return ans;
    }
}
```

## - [‼️] [Leetcode 2402. Meeting Rooms III](https://leetcode.cn/problems/meeting-rooms-iii) 用两个heap存free和busy

```java
class Solution {
    public int mostBooked(int n, int[][] meetings) {
        int[] cnt = new int[n];
        PriorityQueue<Integer> freeRoom = new PriorityQueue<>();
        for (int i = 0; i < n; i++) {
            freeRoom.offer(i);
        }
        PriorityQueue<long[]> busyRoom = new PriorityQueue<>((long[] o1, long[] o2) -> {
            if (o1[1] == o2[1]) {
                return Long.compare(o1[0], o2[0]);
            }
            return Long.compare(o1[1], o2[1]);
        });
        Arrays.sort(meetings, (o1, o2) -> o1[0] - o2[0]);
        for (int i = 0; i < meetings.length; i++) {
            int[] cur = meetings[i];
            int start = cur[0];
            int end = cur[1];
            while (!busyRoom.isEmpty() && (int) busyRoom.peek()[1] <= start) {
                int room = (int) busyRoom.poll()[0];
                freeRoom.offer(room);
            }
            if (!freeRoom.isEmpty()) {
                int room = freeRoom.poll();
                busyRoom.offer(new long[] {room, end});
                cnt[room]++;
            } else {
                long[] top = busyRoom.poll(); // 最早结束
                int room = (int) top[0];
                long newEnd = top[1] + (end - start);
                busyRoom.offer(new long[]{room, newEnd});
                cnt[room]++;
            }
        }
        // int ans = 0;
        int mx = 0;
        for (int i = 0; i < n; i++) {
            mx = Math.max(mx, cnt[i]);
        }
        for (int i = 0; i < n; i++) {
            if (cnt[i] == mx) return i;
        }
        return -1;
    }
}
```

## - [‼️] [Leetcode 815. Bus Routes](https://leetcode.cn/problems/bus-routes) 对bus和route同时记录visit

```java
class Solution {
    public int numBusesToDestination(int[][] routes, int source, int target) {
        List<Integer>[] stopToRoutes = new ArrayList[1000000];
        Arrays.setAll(stopToRoutes, _ -> new ArrayList<>());
        for (int i = 0; i < routes.length; i++) {
            for (int j = 0; j < routes[i].length; j++) {
                stopToRoutes[routes[i][j]].add(i);
            }
        }
        Deque<int[]> queue = new ArrayDeque<>();
        boolean[] visit = new boolean[1000000];
        queue.offerLast(new int[] {source, 0});
        visit[source] = true;
        while (!queue.isEmpty()) {
            int[] cur = queue.pollFirst();
            if (cur[0] == target) {
                return cur[1];
            }
            for (int route : stopToRoutes[cur[0]]) {
                if (routes[route] == null) continue;
                for (int stop : routes[route]) {
                    if (visit[stop]) continue;
                    visit[stop] = true;
                    queue.offerLast(new int[] {stop, cur[1] + 1});
                }
                routes[route] = null;
            }
        }
        return -1;
    }
}
```

## - [‼️] [Leetcode 427. Construct Quad Tree](https://leetcode.cn/problems/construct-quad-tree) 二维前缀和求面积

```java
class Solution {
    int[][] pre;
    public void build(int[][] grid) {
        int n = grid.length;
        pre = new int[n + 1][n + 1];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                pre[i + 1][j + 1] = pre[i][j + 1] + pre[i + 1][j] - pre[i][j] + grid[i][j];
            }
        }
    }
    public Node construct(int[][] grid) {
        build(grid);
        int n = grid.length;
        return buildTree(0, 0, n - 1, n - 1);
    }
    public Node buildTree(int a, int b, int c, int d) {
        // 暴力搜索
        // if (rowStart > rowEnd || colStart > colEnd) {
        //     return null;
        // }
        // boolean isLeaf = true;
        // int val = grid[rowStart][colStart];
        // for (int i = rowStart; i <= rowEnd; i++) {
        //     for (int j = colStart; j <= colEnd; j++) {
        //         if (grid[i][j] != val) {
        //             isLeaf = false;
        //             break;
        //         }
        //     }
        // }
        // if (isLeaf) {
        //     return new Node(val == 1, true, null, null, null, null);
        // }
        int area = pre[c + 1][d + 1] - pre[a][d + 1] - pre[c + 1][b] + pre[a][b];
        int len = c - a + 1;

        if (area == 0 || area == len * len) {
            return new Node(area == len * len, true);
        }

        Node root = new Node();
        root.isLeaf = false;

        int midRow = (a + c) / 2;
        int midCol = (b + d) / 2;

        root.topLeft = buildTree(a, b, midRow, midCol);
        root.topRight = buildTree(a, midCol + 1, midRow, d);
        root.bottomLeft = buildTree(midRow + 1, b, c, midCol);
        root.bottomRight = buildTree(midRow + 1, midCol + 1, c, d);

        return root;
    }
}
```

## - [‼️] [Leetcode 564. Find the Closest Palindrome](https://leetcode.cn/problems/find-the-closest-palindrome)

```java
class Solution {
    private long minD = Long.MAX_VALUE;
    private long ans;

    public String nearestPalindromic(String n) {
        long num = Long.parseLong(n);
        int m = n.length(); // num 的十进制长度
        update((long) Math.pow(10, m - 1) - 1, num); // 十进制长为 m-1 的最大回文数
        update((long) Math.pow(10, m) + 1, num);     // 十进制长为 m+1 的最小回文数

        int left = Integer.parseInt(n.substring(0, (m + 1) / 2));
        // 枚举十进制长为 m 的邻近回文数
        for (int l = left - 1; l <= left + 1; l++) {
            long pal = l;
            for (int x = m % 2 > 0 ? l / 10 : l; x > 0; x /= 10) {
                pal = pal * 10 + (x % 10);
            }
            update(pal, num);
        }

        return Long.toString(ans);
    }

    private void update(long pal, long num) {
        long d = Math.abs(pal - num);
        if (d > 0 && (d < minD || d == minD && pal < ans)) {
            minD = d;
            ans = pal;
        }
    }
}

作者：灵茶山艾府
链接：https://leetcode.cn/problems/find-the-closest-palindrome/solutions/3855597/zhi-xu-kao-lu-5-ge-shu-zi-pythonjavacgo-3td25/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```


## - [⚠️] [Leetcode 5. Longest Palindromic Substring](https://leetcode.cn/problems/longest-palindromic-substring)

## - [⚠️] [Leetcode 1244. Design A Leaderboard](https://leetcode.cn/problems/design-a-leaderboard) TreeMap store score cnt

## - [⚠️ 复习] [Leetcode 648. Replace Words](https://leetcode.cn/problems/replace-words) Trie Tree

## - [⚠️] [Leetcode 79. Word Search](https://leetcode.cn/problems/word-search)

## - [✅] [Design an in-memory database to handle transactions](https://www.hellointerview.com/community/questions/memory-database-transactions/cm6uaoy1y00003b6la4jojd7s?company=Uber&level=STAFF)

## - [ ] [Package Installation Order](https://www.hellointerview.com/community/questions/package-installation-order/cm6jw34rg0018dar9w2f337wf?company=Uber&level=MID_LEVEL)

## - [ ] [Leetcode 127. Word Ladder](https://leetcode.cn/problems/word-ladder)

## - [ ] [Leetcode 749. Contain Virus](https://leetcode.cn/problems/contain-virus)

## - [✅] [Leetcode 588. Design In-Memory File System](https://leetcode.cn/problems/design-in-memory-file-system) Trie Tree

```java
class TrieNode {
    Map<String, String> files;
    Map<String, TrieNode> directory;

    TrieNode() {
        files = new HashMap<>();
        directory = new HashMap<>();
    }
}

class FileSystem {

    TrieNode root;

    public FileSystem() {
        root = new TrieNode();
    }
    
    public List<String> ls(String path) {
        TrieNode cur = root;
        String[] paths = path.split("/");
        for (int i = 1; i < paths.length - 1; i++) {
            if (!cur.directory.containsKey(paths[i])) {
                cur.directory.put(paths[i], new TrieNode());
            }
            cur = cur.directory.get(paths[i]);
        }
        List<String> ans = new ArrayList<>();
        if (paths.length != 0) {
            String last = paths[paths.length - 1];
            if (cur.files.containsKey(last)) {
                ans.add(last);
                return ans;
            } else {
                cur = cur.directory.get(paths[paths.length - 1]);
            }
        }
        for (String str : cur.files.keySet()) {
            ans.add(str);
        }
        for (String str : cur.directory.keySet()) {
            ans.add(str);
        }
        Collections.sort(ans);
        return ans;
    }
    
    public void mkdir(String path) {
        TrieNode cur = root;
        String[] paths = path.split("/");
        for (int i = 1; i < paths.length; i++) {
            if (!cur.directory.containsKey(paths[i])) {
                cur.directory.put(paths[i], new TrieNode());
            }
            cur = cur.directory.get(paths[i]);
        }
    }
    
    public void addContentToFile(String filePath, String content) {
        TrieNode cur = root;
        String[] paths = filePath.split("/");
        for (int i = 1; i < paths.length - 1; i++) {
            if (!cur.directory.containsKey(paths[i])) {
                cur.directory.put(paths[i], new TrieNode());
            }
            cur = cur.directory.get(paths[i]);
        }
        String fileName = paths[paths.length - 1];
        if (!cur.files.containsKey(fileName)) {
            cur.files.put(fileName, content);
        } else {
            String curContent = cur.files.get(fileName);
            cur.files.put(fileName, curContent + content);
        }
    }
    
    public String readContentFromFile(String filePath) {
        TrieNode cur = root;
        String[] paths = filePath.split("/");
        for (int i = 1; i < paths.length - 1; i++) {
            if (!cur.directory.containsKey(paths[i])) {
                cur.directory.put(paths[i], new TrieNode());
            }
            cur = cur.directory.get(paths[i]);
        }
        String fileName = paths[paths.length - 1];
        if (!cur.files.containsKey(fileName)) {
            return "";
        } else {
            return cur.files.get(fileName);
        }
    }
}
```

## - [✅] [Leetcode 1166. Design File System](https://leetcode.cn/problems/design-file-system) Trie Tree

## - [✅] [Leetcode 1438. Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit](https://leetcode.cn/problems/longest-continuous-subarray-with-absolute-diff-less-than-or-equal-to-limit)

## - [✅] [Leetcode 207. Course Schedule](https://leetcode.cn/problems/course-schedule)

## - [✅] [Leetcode 638. Shopping Offers](https://leetcode.cn/problems/shopping-offers) 

## - [✅] [Leetcode 146. LRU Cache](https://leetcode.cn/problems/lru-cache) double linked list + MAP

## - [✅] [Leetcode 415. Add Strings](https://leetcode.cn/problems/add-strings)

## - [✅] [Leetcode 53. Maximum Subarray](https://leetcode.cn/problems/maximum-subarray) DP

## - [✅] [Leetcode 200. Number of Islands](https://leetcode.cn/problems/number-of-islands)

## - [✅] [Leetcode 314. Binary Tree Vertical Order Traversal](https://leetcode.cn/problems/binary-tree-vertical-order-traversal) BFS or DFS + TreeMap

## - [✅] [Leetcode 347. Top K Frequent Elements](https://leetcode.cn/problems/top-k-frequent-elements) PQ
