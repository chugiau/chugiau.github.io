---
title: Hexo 插件載入地雷
date: 2016-11-21 15:27:59
categories: Technical
tags:
  - Hexo
  - plugins
  - hexo-deployer-git
  - hexo-generator-cname
  - package.json
  - config
---

# 摘要

剛開始學習如何使用 hexo 弄個靜態部落格，也弄了個網域。而在使用插件後，就發生災難了，無法 deploy。此文是記錄為什麼會遇到此問題，以及記錄如何處理。

<!--more-->
# 事由

Hexo 版本為 `3.2.2`。

起初只有使用 `hexo-deployer-git`，沒有使用 `hexo-generator-cname`，這情況下可以正常地 deploy，也不需要特地修改 `_config.yml` 的 `plugins:`。

後來部落格使用了 `hexo-generator-cname`，就無法 deploy，即便 `_config.yml` 的 `plugins:` 添加 `hexo-generator-cname`，`package.json` 也有 `hexo-deployer-git` 及 `hexo-generator-cname`。

# 原因

發現是 hexo 的插件載入實作特性的關係，只要有在 `_config.yml` 的 `plugins:` 手動添加任何插件，則 hexo 不會去抓 `package.json`。

# 解法

解決方法有兩種。

1. 將所有有使用到的插件都手動放進 `plugins:`。
2. 不要用 `plugins:`。

# 感想

我認為 hexo 官方有必要將插件載入行為寫在文件上，已經有人開了個 issue 了，但還是晾在那邊沒動。到了 2017-05-04，issue 還是沒人處理。

# 參考資料

- https://github.com/hexojs/hexo/issues/1848
- https://github.com/hexojs/site/issues/262
