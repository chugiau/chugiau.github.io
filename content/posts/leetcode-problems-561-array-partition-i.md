---
title: 'LeetCode Problems: 561. Array Partition I'
date: 2017-05-07 18:09:27
categories:
  - Technical
keywords:
  - LeetCode
  - Array Partition
  - Array
  - Sort
  - C++
tags:
  - LeetCode
---

# 題目資訊

- 名稱：Array Partition I
- 分類：Algorithms
- 編號：561
- 難度：Easy
- 標籤：Array
- 網址：https://leetcode.com/problems/array-partition-i

<!--more-->

# 題目描述

## 原文

Given an array of **2n** integers, your task is to group these integers into **n** pairs of integer, say (a1, b1), (a2, b2), ..., (an, bn) which makes sum of min(ai, bi) for all i from 1 to n as large as possible.

**Example 1:**

```
Input: [1,4,3,2]

Output: 4
Explanation: n is 2, and the maximum sum of pairs is 4.

```

**Note:**

1. **n** is a positive integer, which is in the range of [1, 10000].
2. All the integers in the array will be in the range of [-10000, 10000].

## 中譯

給定一個有 **2n** 個整數的陣列，你的任務是將這些整數分組成 **n** 組整數對，(a1, b1), (a2, b2), ..., (an, bn)，並且這些整數對 min(ai, bi) 的總和要盡可能地大。

**範例一：**

```
輸入：[1,4,3,2]

輸出：4
解釋：n 是 2，且最大的對總和為 4。
```

**注意：**

1. **n** 為正整數，值域為 [1, 10000]。
2. 所有陣列元素之值的值域為 [-10000, 10000]。

## 補充翻譯

其實這題我看不懂他題目想表達什麼，是透過輸出來觀察。

意思是這樣，舉個例：

[1,4,3,2] 分組成 (1, 2), (3, 4)，這樣 min(1, 2) + min(3, 4) 才會是最大，1 + 3 = 4。

# 參考解法

## 思路

從例子可以看出，由小到大排序後，加總 index 0 及偶數 index 的元素即可。

我偷懶直接用 C++ STL 的排序。

## C++

```cpp
class Solution {
public:
    int arrayPairSum(std::vector<int>& nums) {
        sort(nums.begin(), nums.end());
        
        auto sum = 0;
        for (auto i = 0; i < nums.size(); i += 2) {
            sum += nums[i];
        }

        return sum;
    }
};
```

# 後記

這題雖然名稱是 Array Partition，但實際在解的過程根本就沒用到 partition 的概念，可能為求時間複雜度最快佳的解法得用 array partition。總之，這題做得有點不知所以然，可能是哪邊沒想清楚。
