---
title: leetcode88
date: 2025-05-18T17:46:11+08:00
tags:
  - 每日一题
  - easy
author: liuzifeng
---
## 题目

给你两个按 **非递减顺序** 排列的整数数组 `nums1` 和 `nums2`，另有两个整数 `m` 和 `n` ，分别表示 `nums1` 和 `nums2` 中的元素数目。

请你 **合并** `nums2` 到 `nums1` 中，使合并后的数组同样按 **非递减顺序** 排列。

**注意：**最终，合并后数组不应由函数返回，而是存储在数组 `nums1` 中。为了应对这种情况，`nums1` 的初始长度为 `m + n`，其中前 `m` 个元素表示应合并的元素，后 `n` 个元素为 `0` ，应忽略。`nums2` 的长度为 `n` 。

## 示例

**输入：nums1 = [1,2,3,0,0,0], m = 3, nums2 = [2,5,6], n = 3
输出：**[1,2,2,3,5,6]

## 思路

从后往前合并：

- 从 `nums1` 的第 `m-1` 个和 `nums2` 的第 `n-1` 个元素开始比较

- 每次取较大的元素放到 `nums1` 的末尾（即 `nums1[m + n - 1]` 开始往前填）

- 这样就能避免在合并过程中覆盖掉 `nums1` 的原始数据

## 代码

```
class Solution {

public:

    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {

         int i = nums1.size();

         while (n > 0) {

            if (m > 0 && nums1[m - 1] > nums2[n -1]) {

                nums1[--i] = nums1[--m];

            } else {

                nums1[--i] = nums2[--n];

            }

         }

    }

};
```
