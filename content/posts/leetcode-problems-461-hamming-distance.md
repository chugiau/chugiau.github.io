---
title: 'LeetCode Problems: 461. Hamming Distance'
date: 2017-05-09 00:44:05
categories:
  - Technical
keywords:
  - LeetCode
  - Hamming Distance
  - C++
tags:
  - LeetCode
---

# 題目資訊

- 名稱：Hamming Distance
- 分類：Algorithms
- 編號：461
- 難度：Easy
- 標籤：Bit Manipulation
- 網址：https://leetcode.com/problems/hamming-distance

<!--more-->

# 題目描述

## 原文

The [Hamming distance](https://en.wikipedia.org/wiki/Hamming_distance) between two integers is the number of positions at which the corresponding bits are different.

Given two integers `x` and `y`, calculate the Hamming distance.

**Note:**
0 ≤ `x`, `y` < 231.

**Example:**

```
Input: x = 1, y = 4

Output: 2

Explanation:
1   (0 0 0 1)
4   (0 1 0 0)
       ↑   ↑

The above arrows point to positions where the corresponding bits are different.
```

## 中譯

漢明距離是計算兩個整數之間，相對應位置的位元有幾個不同。

給定兩個整個 `x` 及 `y`，計算漢明距離。

**注意：**

0 ≤ `x`, `y` < 231.

範例：

```
輸入：x = 1, y = 4

輸出：2

解釋：
1	(0 0 0 1)
4	(0 1 0 0)
       ↑   ↑
       
上方的箭頭所指的位置就是位元不同。
```

#  參考解法

## 思路

這題是大專院校幾乎都教過的東西。

先 x XOR y，篩出不同的 bits，然後計算有多少個 bit 不同。

## C++

```cpp
class Solution {
public:
    int hammingDistance(int x, int y) {
        auto distance = 0;
        auto diff = x ^ y;
        
        while (diff != 0) {
            ++distance;
            diff &= diff - 1;
        }

        return distance;
    }
};
```

# 後記

有更快速的 bit manipulation 實作方法，可以參考 LeetCode 上其他人的 Solutions。
