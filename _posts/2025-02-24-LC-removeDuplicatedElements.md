---
layout: post
title: "Remove Duplicated elements"
date: 2025-02-24
description: "Remove Duplicated elements"
tag: LeetCode
---

## [27. Remove Element](https://leetcode.cn/problems/remove-element/description/)

```Java
Solution: 快慢指针，慢指针左边是final result [0, slow),
nums[slow - 1] is the last number should keep. 每次遇到符合的，更新nums[slow]就行

class Solution {
    public int removeElement(int[] nums, int val) {
        int i = 0, j = 0;
        while (i < nums.length) {
            if (nums[i] != val) {
                nums[j++] = nums[i];
            }
            i++;
        }
        return j;
    }
}
```

## [26. Remove Duplicates from Sorted Array](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/description/)

```Java
Solution: 快慢指针，慢指针左边是final result [0, slow)。
从第二个开始，因为第一个元素一定会被keep
每次nums[fast]跟nums[slow - 1]对比，如果一样则为duplicated，不一样则更新slow。

class Solution {
    public int removeDuplicates(int[] nums) {
        int i = 1, j = 1;
        while (i < nums.length) {
            if (nums[j - 1] != nums[i]) {
                nums[j++] = nums[i];
            }
            i++;
        }
        return j;
    }
}
```

## [80. Remove Duplicates from Sorted Array II](https://leetcode.cn/problems/remove-duplicates-from-sorted-array-ii/description/)

```Java
Solution: 快慢指针，慢指针左边是final result [0, slow)。
从第三个开始，因为前两个一定会被keep
每次nums[fast]跟nums[slow - 2]对比，如果一样则为duplicated，不一样则更新slow。

class Solution {
    public int removeDuplicates(int[] nums) {
        int n = nums.length;
        if (n <= 2) {
            return n;
        }
        int slow = 2, fast = 2;
        while (fast < n) {
            if (nums[slow - 2] != nums[fast]) {
                nums[slow] = nums[fast];
                ++slow;
            }
            ++fast;
        }
        return slow;
    }
}
```
