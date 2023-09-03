---
title: 'LeetCode Problems: 7. Reverse Integer'
date: 2017-05-01 00:52:17
meta_title: ""
description: ""
image: ""
author: "Kán Chúgiâu"
categories:
  - Technical
keywords:
  - LeetCode
  - Reverse Integer
  - C++
tags:
  - LeetCode
draft: false
---

# 題目資訊

- 名稱：Reverse Integer
- 分類：Algorithms
- 編號：7
- 難度：Easy
- 標籤：Math
- 網址：https://leetcode.com/problems/reverse-integer

# 題目描述

## 原文

Reverse digits of an integer.

**Example1:** x = 123, return 321
**Example2:** x = -123, return -321

[click to show spoilers.](https://leetcode.com/problems/reverse-integer/#)

Have you thought about this?

Here are some good questions to ask before coding. Bonus points for you if you have already thought through this!

If the integer's last digit is 0, what should the output be? ie, cases such as 10, 100.

Did you notice that the reversed integer might overflow? Assume the input is a 32-bit integer, then the reverse of 1000000003 overflows. How should you handle such cases?

For the purpose of this problem, assume that your function returns 0 when the reversed integer overflows.

**Note:**
The input is assumed to be a 32-bit signed integer. Your function should **return 0 when the reversed integer overflows**.

## 中譯

反向排列一個整數的數字。

範例一：x = 123, 回傳 321

範例二：x = -123, 回傳 -321

你有想過這個嗎？

在開始寫 code 前，有一些問題要先問。如果你已經有想到，那恭喜你。

若整數的最後一個數字是 0，那輸出應該是什麼？例如 10, 100。

你有注意到反向排列後的整數可能會溢位嗎？假設輸入的值是一個 32-bit 的整數，那麼 1000000003 顛倒後會發生溢位。你該如何處理這些情況呢？

對於此問題的目的，你的函式在會發生整數溢位時要回傳 0。

注意：

輸入值均假設是一個 32-bit 帶有正負號的整數。當反向排列後的整數會發生溢位，你的函示應該要回傳 0。

# 參考解法

## 思路

直觀上，就是把數字反過來放。暫時先不處理溢位問題。

每次只處理一個數字，運用 mod 把數字取出來（x mod 10）。

乘以 10 將結果往左移，除以 10 將輸入值往右移，直到輸入值歸零。

```
e.g.
input 123

a = 0
x = 123
      ^
a = 3	(a = a*10 + (x mod 10))
x = 12	(x = x/10)
     ^
a = 32	(a = a*10 + (x mod 10))
x = 1	(x = x/10)
    ^
a = 321	(a = a*10 + (x mod 10))
x = 0	(x = x/10)

end
```

至於正負號問題，C++ modulo 負值會回傳負值，與數學定義的模除不同，運用這特性可以省去處理正負號的麻煩。

接著，處理帶有正負號的整數運算溢位問題。

要如何判斷 signed integer overflow 呢？

在這題需要運用的溢位判斷，其實比想像中的簡單，由於運算式是固定的，因此只要判斷運算前與運算後的值是否相等即可，不相等代表發生溢位了。

```
a = b + c
如果 a - c 不等於 b，則代表 b + c 發生溢位，乘法亦同
```

因此，用這方式來補上前述的數字反向排列。

```
e.g.
input 123

a = 0
x = 123
      ^
a' = a*10 + (x mod 10) = 3
若 (a' - (x mod 10))/10 != a, 則 return 0
a = 3	(a = a')
x = 12	(x = x/10)
     ^
a' = a*10 + (x mod 10) = 32
若 (a' - (x mod 10))/10 != a, 則 return 0
a = 32	(a = a')
x = 1	(x = x/10)
    ^
a' = a*10 + (x mod 10) = 321
若 (a' - (x mod 10))/10 != a, 則 return 0
a = 321	(a = a')
x = 0	(x = x/10)

end
```

## C++

```cpp
class Solution {
public:
    int reverse(int x)
    {
        int reversed = 0;
        while (x != 0) {
            int digit = x%10;
            int tempReversed = reversed*10 + digit;
            if ((tempReversed - digit)/10 != reversed) {
                return 0;
            }
            reversed = tempReversed;
            x /= 10;
        }
        return reversed;
    }
};
```

# 參考資料

- pmg (2016-08-02), _How to detect integer overflow in C/C++?_, <http://stackoverflow.com/questions/199333/how-to-detect-integer-overflow-in-c-c>
