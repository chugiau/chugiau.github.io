---
title: C++ 通用 value 轉 bytes，bytes 轉 value
date: 2018-09-10 22:19:22
meta_title: ""
description: ""
image: ""
author: "Kán Chúgiâu"
categories:
  - Technical
keywords:
  - C++
  - C++11
  - C++14
  - C++17
  - bytes conversion
tags:
  - C++
  - C++11
  - C++14
  - C++17
draft: true
---

記錄一下 C++ 如何方便快速將一個簡單數值轉成 bytes。

```cpp
#include <cstddef>
#include <type_traits>

#include <gsl/span>

template<typename T>
gsl::span<std::byte> to_bytes(T value)
{
	auto begin = reinterpret_cast<const std::byte*>(std::addressof(value));
	auto end = begin + sizeof(T);
	return gsl::make_span(begin, end);
}

template<typename T>
T from_bytes(
	gsl::span<std::byte>::const_iterator begin,
	gsl::span<std::byte>::const_iterator end)
{
	T value;
	auto begin_value = reinterpret_cast<std::byte*>(std::addressof(value));
	std::copy(begin, end, begin_value);
	return value;
}
```
