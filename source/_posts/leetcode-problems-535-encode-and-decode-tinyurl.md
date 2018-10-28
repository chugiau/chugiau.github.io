---
title: 'LeetCode Problems: 535. Encode and Decode TinyURL'
date: 2017-05-07 12:26:31
categories: Technical
tags:
    - LeetCode
    - Algorithms
    - Encode and Decode TinyURL
    - TinyURL
    - C++
---

# 摘要

這篇是筆者解短網址編碼、解碼的過程。這題是之前 LeetCode 的 System Design 題目（[編號 534](https://leetcode.com/problems/design-tinyurl)），也就是設計將一般網址轉成短網址的服務。但這題的寫 code 題型其實設計上有瑕疵，內文將會說明。

# 題目資訊

- 名稱：Encode and Decode TinyURL
- 分類：Algorithms
- 編號：535
- 難度：Medium
- 標籤：Hash Table, Math
- 網址：https://leetcode.com/problems/encode-and-decode-tinyurl

<!--more-->

# 題目描述

## 原文

TinyURL is a URL shortening service where you enter a URL such as `https://leetcode.com/problems/design-tinyurl` and it returns a short URL such as `http://tinyurl.com/4e9iAk`.

Design the `encode` and `decode` methods for the TinyURL service. There is no restriction on how your encode/decode algorithm should work. You just need to ensure that a URL can be encoded to a tiny URL and the tiny URL can be decoded to the original URL.

##  中譯

TinyURL 是一個縮短 URL  的網址，例如 `https://leetcode.com/problems/design-tinyurl` 縮短後會變成 `http://tinyurl.com/4e9iAk`。

為 TinyURL 服務設計 `編碼` 及 `解碼` 方法。此題沒有限制編碼/解碼演算法。你只需要確保 URL 邊碼成短 URL 後，這個短 URL 可以正確解碼為原本的 URL 即可。

# 參考解法

## 方法一

此方法是偷吃步，請勿模仿唷。

### 思路

題目說明並不設限 encode/decode 的方法，並且題目的 code 直接告訴你系統會如何呼叫你寫的 code。如下。

```cpp
// Your Solution object will be instantiated and called as such:
// Solution solution;
// solution.decode(solution.encode(url));
```

因此，直接回傳參數就好了，真的，沒騙人。

### C++

```cpp
class Solution {
public:

    // Encodes a URL to a shortened URL.
    string encode(string longUrl) {
        return longUrl;
    }

    // Decodes a shortened URL to its original URL.
    string decode(string shortUrl) {
        return shortUrl;
    }
};

// Your Solution object will be instantiated and called as such:
// Solution solution;
// solution.decode(solution.encode(url));
```

## 方法二

這是比較正式的作法，不偷機。

### 思路

#### Encode

首先，此題在 System Design 的題型裡，有要求無論要 encode 的 long URL 是什麼，回傳的結果都必須是 unique 的。並考慮盡可能縮短編碼後的長度，這時我採用的概念是建一個 DB table（只是用關聯式容器來模擬），並用 incremental index  key，接著用這 key 當作編碼依據。

接著，用 base 62 壓縮 key，把 10 進制轉 62 進制，這 62 進制是什麼？數字 + 英文字母大小寫。由 0 排至 61，對應的「數字」為 `0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ`。用這 key 將 long URL 存到表裡，到時候查詢會用。

在實作 base 62 時，先做張數字轉數值的反查表會比較方便。

最後，回傳 base URL + key。e.g. `http://tinyurl.com/1aZ726`

#### Decode

首先，將 key 從 URL 取出。e.g. `http://tinyurl.com/1aZ726` 取出 `1aZ726`。

用 base 62 轉 base 10 把 key 還原成 10 進制，接著用還原後的數值去把 long URL 查出來回傳就可以了。

### C++

```cpp
class Solution {
public:    
    Solution() {
        for (string::size_type i = 0; i < chars.size(); ++i) {
            charToValueMap[chars[i]] = i;
        }
    }

    // Encodes a URL to a shortened URL.
    string encode(string longUrl) {
        auto id = urls.size() + 1;
        urls[id] = longUrl;
        
        std::vector<char> digits;
        while (id != 0) {
            digits.emplace_back(chars[id%base]);
            id /= base;
        }
        return baseUrl + string(digits.rbegin(), digits.rend());
    }

    // Decodes a shortened URL to its original URL.
    string decode(string shortUrl) {
        auto idString = shortUrl.substr(baseUrlSize, shortUrl.size() - baseUrlSize);
        auto id = 0ULL;
        auto idStringSize = idString.size();
        for (string::size_type i = 0; i < idStringSize; ++i) {
            id += charToValueMap[idString[i]]*static_cast<uint64_t>(std::pow(base, idStringSize - i - 1));
        }
        return urls[id];
    }
    
private:
    const string chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
    const int base = chars.size();
    const string baseUrl = "http://tinyurl.com/";
    const int baseUrlSize = baseUrl.size();

    std::unordered_map<char, int> charToValueMap;
    std::unordered_map<uint64_t, string> urls;
};

// Your Solution object will be instantiated and called as such:
// Solution solution;
// solution.decode(solution.encode(url));
```

※注意！部分寫法是配合 LeeCode 的限制，一般來說不會這麼寫。

# 後記

嚴格來說，這題是系統設計題，方法二的瓶頸很明顯。有考慮過用 UUID 取 6 碼當 key 來處理，但可能還是會面臨大量 UUID 也有些效能問題，且取 6 碼會增加碰撞機會。這些問題就留在第 534，系統設計題想吧。