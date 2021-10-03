---
title: C++ Boost Asio 基本介紹
date: 2018-08-04 11:16:22
categories:
  - Technical
keywords:
  - C++
  - C++11
  - C++14
  - C++17
  - C++20
  - Boost
  - Asio
  - Network
  - TCP
  - Socket
  - Async
  - Proactor
  - Design Pattern
tags:
  - C++
  - C++11
  - C++14
  - C++17
  - C++20
  - Boost
  - Asio
  - Network
  - TCP
  - Socket
  - Async
  - Proactor
  - Design Pattern
draft: true
---

最近我在用 Boost.Asio 來開發伺服器，趁著還有在用的時候趕快寫下來，好方便之後複習。

先講一下 Boost，Boost 是一套知名的 C++ 程式庫，後來的標準程式庫大都是從 Boost 而來，所以 Boost 又有 C++「準」標準程式庫的別稱。 

以下說明以 Boost 1.67 版為主。

# Asio 簡介

Asio (發音：A-si-o) 是一套提供網路及 low-level IO 且跨平台的 C++ 程式庫。後來加入 Boost，成為 Boost 的其中一個模塊，不過仍有提供獨立於 Boost 的版本。

# Asio 架構

Asio 整個設計架構環繞在它的名字－－Asynchronous IO，也就是處理非同步 IO，。

非同步模型為 [Proactor Pattern][]，但遺憾的是只有 Windows [IOCP][] 是 OS 原生的 proactor，Linux 的 [epoll][] 及 BSD 的 [kqueue][] 都是 [Reactor Pattern][]。

# Proactor Pattern





```cpp
#include <boost/>
```



# References

- https://think-async.com/
- https://www.boost.org/doc/libs/1_67_0/doc/html/boost_asio



[Proactor Pattern]: https://en.wikipedia.org/wiki/Proactor_pattern
[Reactor Pattern]: https://en.wikipedia.org/wiki/Reactor_pattern
[IOCP]: https://zh.wikipedia.org/zh-tw/IOCP
[epoll]: https://zh.wikipedia.org/zh-tw/Epoll
[kqueue]: https://en.wikipedia.org/wiki/Kqueue
[Winsock]: https://zh.wikipedia.org/zh-tw/Winsock
