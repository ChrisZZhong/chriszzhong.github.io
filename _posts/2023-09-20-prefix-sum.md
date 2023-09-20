---
layout: post
title: "Prefix Sum problems"
date: 2023-08-07
description: "Prefix Sum problems"
tag: LeetCode
---

# Prefix Sum problems

| LeetCode                                                                           | 难度 |
| ---------------------------------------------------------------------------------- | ---- |
| [724. Find Pivot Index](https://leetcode.com/problems/find-pivot-index)            | 🟢   |
| [560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) | 🟡   |

## 724. Find Pivot Index

### Description

Given an array of integers `nums`, calculate the **pivot index** of this array.

The **pivot index** is the index where the sum of all the numbers **strictly** to the left of the index is equal to the sum of all the numbers **strictly** to the index's right.

If the index is on the left edge of the array, then the left sum is 0 because there are no elements to the left. This also applies to the right edge of the array.

Return the **leftmost pivot index**. If no such index exists, return -1.

```
Input: nums = [1,7,3,6,5,6]
Output: 3
Explanation:
The pivot index is 3.
Left sum = nums[0] + nums[1] + nums[2] = 1 + 7 + 3 = 11
Right sum = nums[4] + nums[5] = 5 + 6 = 11
```

```
Input: nums = [1,2,3]
Output: -1
Explanation:
There is no index that satisfies the conditions in the problem statement.
```

```
Input: nums = [2,1,-1]
Output: 0
Explanation:
The pivot index is 0.
Left sum = 0 (no elements to the left of index 0)
Right sum = nums[1] + nums[2] = 1 + -1 = 0
```

### Solution

第一眼看上去是 prefix sum，但是其实不需要

我的 Prefix sum 的解法如下：维护一个 leftsum，然后遍历数组，每次加上当前元素，然后判断是否 leftsum == presum - nums[i] - leftsum，相等则返回当前下标

```java
class Solution {
    public int pivotIndex(int[] nums) {
        int presum = 0;
        //数组的和
        for (int x : nums) {
           presum += x;
        }
        int leftsum = 0;
        for (int i = 0; i < nums.length; ++i) {
            //发现相同情况
            if (leftsum == presum - nums[i] - leftsum) {
                return i;
            }
            leftsum += nums[i];
        }
        return -1;
    }
}
```

**Better solution:**

设 `total` 为数组所有元素的和，左侧元素和为`sum` 则右侧为`total - sum - nums[i]`

则 **sum = total - sum - nums[i]** 即满足 **2 \* sum + nums[i] = total** 即可，不必用 prefix sum 去判断左右

```java
class Solution {
    public int pivotIndex(int[] nums) {
        int total = Arrays.stream(nums).sum();
        int sum = 0;
        for (int i = 0; i < nums.length; ++i) {
            if (2 * sum + nums[i] == total) {
                return i;
            }
            sum += nums[i];
        }
        return -1;
    }
}
```

## 560. Subarray Sum Equals K

### Description

Given an array of integers `nums` and an integer `k`, return _the total number of continuous subarrays whose sum equals to `k`_.

```
Input: nums = [1,1,1], k = 2
Output: 2
-1000 <= nums[i] <= 1000
```

这里注意，题目中的数组元素有负数，所以不能用滑动窗口，因为滑动窗口只能用于正数。

若数组元素都是正数，则可以用滑动窗口，因为窗口内的和是递增的，所以可以用双指针来滑动窗口

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int res = 0;
        int preLeft = 0;
        int preRight = 0;
        int l =  0;
        for (int r = 0; r < nums.length; r++) {
            preRight += nums[r];
            while (l <= r && preRight - preLeft >= k) {
                if (preRight - preLeft == k) res++;
                preLeft += nums[l++];
            }
        }
        return res;
    }
}
```

但是这里有负数，所以不能用滑动窗口，而是用 prefix sum + hashmap， 详解见 [solution](https://leetcode.cn/problems/subarray-sum-equals-k/solutions/238572/he-wei-kde-zi-shu-zu-by-leetcode-solution/)

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int res = 0;
        int presum = 0;
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0, 1);
        for (int i = 0; i < nums.length; ++i) {
            presum += nums[i];
            if (map.containsKey(presum - k)) {
                res += map.get(presum - k);
            }
            map.put(presum, map.getOrDefault(presum, 0) + 1);
        }
        return res;
    }
}
```
