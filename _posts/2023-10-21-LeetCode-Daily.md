---
layout: post
title: "LeetCode daily problems"
date: 2023-10-21
description: "LeetCode daily problems"
tag: LeetCode
---

## [1402. Reducing Dishes](https://leetcode.com/problems/reducing-dishes/)

`time : 10-21-2023`

### **Description**:

<div class="xFUwe" data-track-load="description_content"><p>A chef has collected data on the <code>satisfaction</code> level of his <code>n</code> dishes. Chef can cook any dish in 1 unit of time.</p>

<p><strong>Like-time coefficient</strong> of a dish is defined as the time taken to cook that dish including previous dishes multiplied by its satisfaction level i.e. <code>time[i] * satisfaction[i]</code>.</p>

<p>Return the maximum sum of <strong>like-time coefficient </strong>that the chef can obtain after preparing some amount of dishes.</p>

<p>Dishes can be prepared in <strong>any </strong>order and the chef can discard some dishes to get this maximum value.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> satisfaction = [-1,-8,0,5,-9]
<strong>Output:</strong> 14
<strong>Explanation:</strong> After Removing the second and last dish, the maximum total <strong>like-time coefficient</strong> will be equal to (-1*1 + 0*2 + 5*3 = 14).
Each dish is prepared in one unit of time.</pre>

<p><strong class="example">Example 2:</strong></p>

<pre><strong>Input:</strong> satisfaction = [4,3,2]
<strong>Output:</strong> 20
<strong>Explanation:</strong> Dishes can be prepared in any order, (2*1 + 3*2 + 4*3 = 20)
</pre>

<p><strong class="example">Example 3:</strong></p>

<pre><strong>Input:</strong> satisfaction = [-1,-4,-5]
<strong>Output:</strong> 0
<strong>Explanation:</strong> People do not like the dishes. No dish is prepared.
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>n == satisfaction.length</code></li>
	<li><code>1 &lt;= n &lt;= 500</code></li>
	<li><code>-1000 &lt;= satisfaction[i] &lt;= 1000</code></li>
</ul>
</div>

### **Ideas**:

The intuition behind this approach is that we always try to maximize the like-time coefficient by prioritizing dishes with higher satisfaction.

To get the maximum like-time coefficient, apparently all non-negative satisfaction dishes should be included. The rest is how to `deal with negative satisfaction dishes`.

First we sort the array in ascending order and iterate from back to front. `[-9, -8, -1, 0, 5]` when it comes to `-1`, now the dishes we pick is `[0, 5]` and the sum is 5, each time we add a new dishes, the satisfaction level will increase by sum of all previous selected dishes and the current dishes (here is - 1 + 0 + 5 = 4) which is `sum + satisfaction[i]`. if `sum + satisfaction[i]` is less than 0, we can stop the iteration because the rest dishes will be less than 0. If `sum + satisfaction[i]` is greater than 0, we can add it to the result.

To record the result, we can use a variable `res` to record the result, each time we pick a new dishes, the result will increase `sum + satisfaction[i]`. We don't need to record the list of the dishes and calculate the sum by `times[i] * satisfaction[i]`.

### **Solution**:

We can use a greedy approach. We start by sorting the satisfaction array in descending order. Then, we iterate over the array and keep accumulating the satisfaction until the accumulated value is greater than zero. At each step, we update our result.

```Java
class Solution {
    public int maxSatisfaction(int[] satisfaction) {
        int sum = 0;
        int res = 0;
        Arrays.sort(satisfaction);
        for (int i = satisfaction.length - 1; i >= 0; i--) {
            if (satisfaction[i] < 0 && sum + satisfaction[i] < 0) break;
            sum += satisfaction[i--];
            res += sum;
        }
        return res;
    }
}
```

Time complexity: O(nlogn)

Space complexity: O(1)
