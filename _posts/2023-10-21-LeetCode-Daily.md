---
layout: post
title: "LeetCode daily problems"
date: 2023-10-21
description: "LeetCode daily problems"
tag: LeetCode
---

## 2678. Number of Senior Citizens 🟢 10-23-2023

[2678. Number of Senior Citizens](https://leetcode.com/problems/number-of-senior-citizens/)

**Description**:

<div class="xFUwe" data-track-load="description_content"><p>You are given a <strong>0-indexed</strong> array of strings <code>details</code>. Each element of <code>details</code> provides information about a given passenger compressed into a string of length <code>15</code>. The system is such that:</p>

<ul>
	<li>The first ten characters consist of the phone number of passengers.</li>
	<li>The next character denotes the gender of the person.</li>
	<li>The following two characters are used to indicate the age of the person.</li>
	<li>The last two characters determine the seat allotted to that person.</li>
</ul>

<p>Return <em>the number of passengers who are <strong>strictly </strong><strong>more than 60 years old</strong>.</em></p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> details = ["7868190130M7522","5303914400F9211","9273338290F4010"]
<strong>Output:</strong> 2
<strong>Explanation:</strong> The passengers at indices 0, 1, and 2 have ages 75, 92, and 40. Thus, there are 2 people who are over 60 years old.
</pre>

<p><strong class="example">Example 2:</strong></p>

<pre><strong>Input:</strong> details = ["1313579440F2036","2921522980M5644"]
<strong>Output:</strong> 0
<strong>Explanation:</strong> None of the passengers are older than 60.
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>1 &lt;= details.length &lt;= 100</code></li>
	<li><code>details[i].length == 15</code></li>
	<li><code>details[i] consists of digits from '0' to '9'.</code></li>
	<li><code>details[i][10] is either 'M' or 'F' or 'O'.</code></li>
	<li>The phone numbers and seat numbers of the passengers are distinct.</li>
</ul>
</div>

**Ideas**:

This is a simple problem, the only thing we need to do is to extract the age from the string and compare it with 60.

**Solution**:

```Java
class Solution {
    public int numOlderThan60(String[] details) {
        int res = 0;
        for (String detail : details) {
        // extract the age from the string using substring, instead of using character and ascii code to manually calculate the age like this : Integer.valueOf(str.charAt(11) - '0') * 10 + Integer.valueOf(str.charAt(12) - '0');
            int age = Integer.parseInt(detail.substring(10, 12));
            if (age > 60) res++;
            // or we can only compare the 11th character with '6' to check if the age is greater than 60
        }
        return res;
    }
}
```

```Java
Class Solution {
    // using Stream API only for Java 8 and above
    public int numOlderThan60(String[] details) {
        return (int)Arrays.stream(details).filter(string -> Integer.valueOf(string.substring(11,13)) > 60).count();
    }
}
```

Time complexity: O(n)

Space complexity: O(1)

## 1402. Reducing Dishes 🔴 10-22-2023

**[1402. Reducing Dishes](https://leetcode.com/problems/reducing-dishes/)**

**Description**:

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

**Ideas**:

The intuition behind this approach is that we always try to maximize the like-time coefficient by prioritizing dishes with higher satisfaction.

To get the maximum like-time coefficient, apparently all non-negative satisfaction dishes should be included. The rest is how to `deal with negative satisfaction dishes`.

First we sort the array in ascending order and iterate from back to front. `[-9, -8, -1, 0, 5]` when it comes to `-1`, now the dishes we pick is `[0, 5]` and the sum is 5, each time we add a new dishes, the satisfaction level will increase by sum of all previous selected dishes and the current dishes (here is - 1 + 0 + 5 = 4) which is `sum + satisfaction[i]`. if `sum + satisfaction[i]` is less than 0, we can stop the iteration because the rest dishes will be less than 0. If `sum + satisfaction[i]` is greater than 0, we can add it to the result.

To record the result, we can use a variable `res` to record the result, each time we pick a new dishes, the result will increase `sum + satisfaction[i]`. We don't need to record the list of the dishes and calculate the sum by `times[i] * satisfaction[i]`.

**Solution**:

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
