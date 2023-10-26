---
layout: post
title: "LeetCode daily problems"
date: 2023-10-21
description: "LeetCode daily problems"
tag: LeetCode
---

## 2520. Count the Digits That Divide a Number 🟢 10-26-2023

[2520. Count the Digits That Divide a Number](https://leetcode.com/problems/count-the-digits-that-divide-a-number/)

**Description**:

<div class="xFUwe" data-track-load="description_content"><p>Given an integer <code>num</code>, return <em>the number of digits in <code>num</code> that divide </em><code>num</code>.</p>

<p>An integer <code>val</code> divides <code>nums</code> if <code>nums % val == 0</code>.</p>

<p>&nbsp;</p>
<p><strong>Example 1:</strong></p>

<pre><strong>Input:</strong> num = 7
<strong>Output:</strong> 1
<strong>Explanation:</strong> 7 divides itself, hence the answer is 1.
</pre>

<p><strong>Example 2:</strong></p>

<pre><strong>Input:</strong> num = 121
<strong>Output:</strong> 2
<strong>Explanation:</strong> 121 is divisible by 1, but not 2. Since 1 occurs twice as a digit, we return 2.
</pre>

<p><strong>Example 3:</strong></p>

<pre><strong>Input:</strong> num = 1248
<strong>Output:</strong> 4
<strong>Explanation:</strong> 1248 is divisible by all of its digits, hence the answer is 4.
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>1 &lt;= num &lt;= 10<sup>9</sup></code></li>
	<li><code>num</code> does not contain <code>0</code> as one of its digits.</li>
</ul>
</div>

**Ideas**:

The best way to do it is using the modulo operation. If convert to String, the space complexity will be O(n) where n is the length of the string. If we use modulo operation, the space complexity will be O(1).

**Solution**:

```Java
class Solution {
    public int countDigits(int num) {
        int t = num;
        int res = 0;
        while (t != 0) {
            if (num % (t % 10) == 0) {
                res++;
            }
            t /= 10;
        }
        return res;
    }
}
```

Time complexity: O(logn) where n is the number.

Space complexity: O(1)

## 2698. Find the Punishment Number of an Integer 🟠 10-25-2023

[2698. Find the Punishment Number of an Integer](https://leetcode.com/problems/find-the-punishment-number-of-an-integer/)

**Description**:

<div class="xFUwe" data-track-load="description_content"><p>Given a positive integer <code>n</code>, return <em>the <strong>punishment number</strong></em> of <code>n</code>.</p>

<p>The <strong>punishment number</strong> of <code>n</code> is defined as the sum of the squares of all integers <code>i</code> such that:</p>

<ul>
	<li><code>1 &lt;= i &lt;= n</code></li>
	<li>The decimal representation of <code>i * i</code> can be partitioned into contiguous substrings such that the sum of the integer values of these substrings equals <code>i</code>.</li>
</ul>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> n = 10
<strong>Output:</strong> 182
<strong>Explanation:</strong> There are exactly 3 integers i that satisfy the conditions in the statement:
- 1 since 1 * 1 = 1
- 9 since 9 * 9 = 81 and 81 can be partitioned into 8 + 1.
- 10 since 10 * 10 = 100 and 100 can be partitioned into 10 + 0.
Hence, the punishment number of 10 is 1 + 81 + 100 = 182
</pre>

<p><strong class="example">Example 2:</strong></p>

<pre><strong>Input:</strong> n = 37
<strong>Output:</strong> 1478
<strong>Explanation:</strong> There are exactly 4 integers i that satisfy the conditions in the statement:
- 1 since 1 * 1 = 1. 
- 9 since 9 * 9 = 81 and 81 can be partitioned into 8 + 1. 
- 10 since 10 * 10 = 100 and 100 can be partitioned into 10 + 0. 
- 36 since 36 * 36 = 1296 and 1296 can be partitioned into 1 + 29 + 6.
Hence, the punishment number of 37 is 1 + 81 + 100 + 1296 = 1478
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>1 &lt;= n &lt;= 1000</code></li>
</ul>
</div>

**Ideas**:

After we analyze the problem, we can find the hard part is to cut the number into contiguous substrings. To get different combination of numbers, we can use DFS to solve this.

**Solution**:

```Java
    public int punishmentNumber(int n) {
        int sum = 0;
        for (int i = 1; i <= n; i++) {
            if (canPartitioned(String.valueOf(i*i), i, 0, 0)) sum += i * i;
        }
        return sum;

    }
    public boolean canPartitioned(String num, int target, int index, int sum) {
        if (index == num.length()) {
            return sum == target;
        }
        // here we do not have the choice to pick nothing, so the index start from index + 1
        // add [index, i) substring to the sum
        for (int i = index + 1; i <= num.length(); i++) {
            // here we can do a early termination if the sum is greater than target
            if (canPartitioned(num, target, i, sum + Integer.parseInt(num.substring(index, i)))) return true;
        }
        return false;
    }
```

Time complexity: non-polynomial time complexity since DFS is used

Space complexity: O(n \* len(i\*i)) where n is the input and len(i*i) is the length of the square of i (the maximum length is 2 * len(n)), so the space complexity is O(n \* len(n))

## 1155. Number of Dice Rolls With Target Sum 🟠 10-24-2023

[1155. Number of Dice Rolls With Target Sum](https://leetcode.com/problems/number-of-dice-rolls-with-target-sum/)

**Description**:

<div class="xFUwe" data-track-load="description_content"><p>You have <code>n</code> dice, and each die has <code>k</code> faces numbered from <code>1</code> to <code>k</code>.</p>

<p>Given three integers <code>n</code>, <code>k</code>, and <code>target</code>, return <em>the number of possible ways (out of the </em><code>k<sup>n</sup></code><em> total ways) </em><em>to roll the dice, so the sum of the face-up numbers equals </em><code>target</code>. Since the answer may be too large, return it <strong>modulo</strong> <code>10<sup>9</sup> + 7</code>.</p>

<p>&nbsp;</p>
<p><strong class="example">Example 1:</strong></p>

<pre><strong>Input:</strong> n = 1, k = 6, target = 3
<strong>Output:</strong> 1
<strong>Explanation:</strong> You throw one die with 6 faces.
There is only one way to get a sum of 3.
</pre>

<p><strong class="example">Example 2:</strong></p>

<pre><strong>Input:</strong> n = 2, k = 6, target = 7
<strong>Output:</strong> 6
<strong>Explanation:</strong> You throw two dice, each with 6 faces.
There are 6 ways to get a sum of 7: 1+6, 2+5, 3+4, 4+3, 5+2, 6+1.
</pre>

<p><strong class="example">Example 3:</strong></p>

<pre><strong>Input:</strong> n = 30, k = 30, target = 500
<strong>Output:</strong> 222616187
<strong>Explanation:</strong> The answer must be returned modulo 10<sup>9</sup> + 7.
</pre>

<p>&nbsp;</p>
<p><strong>Constraints:</strong></p>

<ul>
	<li><code>1 &lt;= n, k &lt;= 30</code></li>
	<li><code>1 &lt;= target &lt;= 1000</code></li>
</ul>
</div>

**Ideas**:
We can use dynamic programming to solve this problem.
DP[n][t] represents the number of possible ways to roll n dices, so the sum of the face-up numbers equals t.

We have the transition function:

> **DP[n][t] = ∑ (DP[n - 1][t - j]) where j is from 1 to k && t - j >= 0**

**Solution**:

```Java
class Solution {
    private static final int MOD = 1000000007;
    public int numRollsToTarget(int n, int k, int target) {
        int[][] DP = new int[n + 1][target + 1];
        // min of k and target (corner case will cause index out of bound)
        int min = k > target ? target : k;
        // initilization the base case
        for (int i = 1; i <= min; i++) {
            DP[1][i] = 1;
        }
        // DP bottom up
        for (int i = 2; i < n + 1; i++) {
            // start with DP[i][i] because when t < i, it is impossible to roll i dices and get the sum of t
            for (int t = i; t < target + 1; t++) {
                for (int j = 1; j <= k; j++) {
                    if (t - j >= 0) DP[i][t] = (DP[i][t] + DP[i - 1][t - j]) % MOD;
                }
            }
        }
        return DP[n][target];
    }
}
```

Time complexity: O(n \* k \* target)

Space complexity: O(n \* target)

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
