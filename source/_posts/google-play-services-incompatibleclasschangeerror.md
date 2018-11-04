---
title: Google Play Services IncompatibleClassChangeError
date: 2018-04-04 12:51:06
categories:
  - Technical
tags:
  - Android
  - Java
  - Google Play Services
  - GoogleApiClient
  - IncompatibleClassChangeError
  - Unity
---

之前工作在接第三方登入 SDK 時，遇到 `GoogleApiClient.connect()' was expected to be of type interface but was found to be virtual` 這個錯誤。

# 原因

查一查發現是第三方登入 SDK 使用的 Google Play Services 是 7.8.* 以下的版本，而我是引用 8.* 以上的版本。

Google Play Services Android SDK 的 `GoogleApiClient` 自 8.1 版（2015/08）開始是 abstract class，在此之前是 interface。

# 解決方法

兩種解決方法。
一是請對方升級 Google Play Services 程式庫，
二是專案的 `com.google.android.gms:play-services` 只能用 7.8.0。

後來我們是採用方法一，而對方開發者也知道引用的 Google Play Services SDK 太舊要處理，所以都順便解決了，至少升到 8.4.0。

# References

- https://stackoverflow.com/questions/33073779/googleapiclient-connect-was-expected-to-be-of-type-interface-but-was-found-to
