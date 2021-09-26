---
title: 'LeetCode Problems: 9. Palindrome Number'
date: 2017-05-07 00:34:07
categories:
  - Technical
tags:
  - LeetCode
  - Algorithms
  - Palindrome Number
  - C++
---

# 摘要

這篇是筆者解數字回文的過程，說明如何判斷給定的一個數是否回文。這題用字串、拆解數字的方法也可以，但比較慢。其實提示有講，標籤也寫 Math，因此有更好的方式來解。

# 題目資訊

- 名稱：Palindrome Number
- 分類：Algorithms
- 編號：9
- 難度：Easy
- 標籤：Math
- 網址：https://leetcode.com/problems/palindrome-number

<!--more-->

# 題目描述

## 原文

Determine whether an integer is a palindrome. Do this without extra space.

[click to show spoilers.](https://leetcode.com/problems/palindrome-number/#)

Some hints:

Could negative integers be palindromes? (ie, -1)

If you are thinking of converting the integer to string, note the restriction of using extra space.

You could also try reversing an integer. However, if you have solved the problem "Reverse Integer", you know that the reversed integer might overflow. How would you handle such case?

There is a more generic way of solving this problem.

## 中譯

判斷給定的一個數是否為回文，不消耗額外空間。

一些提示：

負數可以是回文嗎？例如 -1

如果你想用整數轉字串，記得使用額外空間的限制。

你可能也想反轉整數。然而，你若解過「反轉整數」的問題，你知道反轉後的整數是有可能溢位的。那你該如何處理這個情況？

其實有一個更通用的作法來解這問題。

# 參考解法

## 方法一

### 思路

用第一個提示的解法，消耗額外空間來解，也就是直接把數值拆成一個一個的數字，然後判斷是否為回文。

負數、模除後等於零的不可能是回文，因此可以先判斷處理掉。

接著處理拆數字，模除 10 取數字，除以 10 移到下一個數字。

最後，判斷拆解後的數字，判斷左、右邊是否一致，因此迴圈只需要跑到中間即可。

### C++

```cpp
class Solution {
public:
    bool isPalindrome(int x)
    {
        if (x < 0 || x != 0 && x%10 == 0) {
            return false;
        }

        std::vector<int> digits;
        while (x != 0) {
            digits.emplace_back(x%10);
            x /= 10;
        }
        
        auto size = digits.size();
        auto mid = size/2;
        for (auto i = 0; i < mid; ++i) {
            if (digits[i] != digits[size - 1 - i]) {
                return false;
            }
        }

        return true;
    }
};
```

## 方法二

### 思路

根據標籤及後半段提示，將某半段的數字段反過來跟另一半比較是否相等就可以了。而且截半後數字最多只有 5 個，不可能 integer overflow。

一樣事先判斷負數、模除後等於零。

另外，要注意中段數字有 0 的情況，例如 101 這種。

### C++

```cpp
class Solution {
public:
    bool isPalindrome(int x)
    {
        if (x < 0 || x != 0 && x%10 == 0) {
            return false;
        }

        auto sum = 0;
        while (x > sum) {
            sum = sum*10 + x%10;
            x /= 10;
        }
        
        return x == sum || x == sum/10;
    }
};
```

