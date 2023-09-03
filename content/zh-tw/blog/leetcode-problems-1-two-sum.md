---
title: 'LeetCode Problems: 1. Two Sum'
date: 2017-04-30 12:57:42
meta_title: ""
description: ""
image: ""
author: "Kán Chúgiâu"
categories:
  - Technical
keywords:
  - LeetCode
  - Two Sum
  - C++
tags:
  - LeetCode
draft: false
---

# 題目資訊

- 名稱：Two Sum
- 分類：Algorithms
- 編號：1
- 難度：Easy
- 標籤：Array, Hash Table
- 網址：https://leetcode.com/problems/two-sum

# 題目描述

## 原文

Given an array of integers, return **indices** of the two numbers such that they add up to a specific target.

You may assume that each input would have **_exactly_** one solution, and you may not use the *same* element twice.

**Example:**

```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

## 中譯

給定一個整數陣列，回傳兩個索引，這兩個元素加起來必須等於指定目標。

你得假設每一組輸入只會有一組解，且陣列元素不可重複使用。

範例：

```
給定 nums = [2, 7, 11, 15], target = 9,

因為 nums[0] + nums[1]  = 2 + 7 = 9
回傳 [0, 1].
```

# 參考解法

## 方法一

### 思路

直觀來說，兩數相加等於目標值，因此簡單遍歷兩次陣列就可以得出結果。

由於不能重複使用元素的關係。因此，

第一次遍歷的最後一個元素可以跳過，因為第二次遍歷會用到，所以第一個迴圈只需要跑 n - 1 次。

第二次遍歷時，由於第一次遍歷已跑了 i 個，因此第二次遍歷從第 i + 1 個開始跑即可。

此解法的時間複雜度為 O(n^2)。

### C++

```cpp
class Solution {
public:
    std::vector<int> twoSum(std::vector<int>& nums, int target)
    {
        auto size = nums.size();
        auto firstSize = size - 1;
        for (auto i = 0; i != firstSize; ++i) {
            for (auto j = i + 1; j != size; ++j) {
                if (nums[i] + nums[j] == target) {
                    return std::vector<int> {i, j};
                }
            }
        }
        return std::vector<int>();
    }
};
```

## 方法二

### 思路

為了減少時間複雜度，有些運算其實可以在一邊遍歷一邊運算時就保存起來。

已知 `target = nums[i] + x`， `target - nums[i] = x` ，

此時可以用關聯式容器（map, table, dictionary 等）將 `nums[i]` 及 `i` 存起來，`nums[i]` 為 key，`i` 為 value。

如果 `x` 存在於 map，代表 `x` 曾被遍歷過 ，此時的 `x` 及現在的 `nums[i]` 加起來等於 `target`，回傳 `x` 的 index（從 map 取出）及 `i` 即為答案。

此解法的時間複雜度為 O(n)。

### C++

```cpp
class Solution {
public:
    std::vector<int> twoSum(std::vector<int>& nums, int target)
    {
        auto size = nums.size();
        auto map = std::map<int, int>(); // <num, index>
        for (auto i = 0; i != size; ++i) {
            auto num = nums[i];
            auto diff = target - num;
            if (map.find(diff) != map.end()) {
                return std::vector<int> {map[diff], i};
            }
            map[num] = i;
        }
        return std::vector<int>();
    }
};
```

