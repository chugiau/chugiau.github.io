---
title: Unity Resources 系統的最佳實務做法
date: 2017-06-18 05:47:33
categories:
  - Technical
tags:
  - Unity
  - Resources
  - AssetBundle
  - Best Practices
---


# Resources 的最佳做法

**別用 Resources**

**別用 Resources**

**別用 Resources**

因為很重要，所以要講三次。為什麼呢？理由有三。

1. 細粒度的記憶體控管會更困難。
2. 不當使用 Resources 資料夾會導致 app 啟動時間、建置時間更久。這問題在 Resources 檔案愈多，愈是明顯，尤其是規格比較差的行動裝置。app 在 splash screen 前黑畫面太久的其中一個原因正是這一點。
3. Resources 內的 assets 無法動態下載更新，也沒辦法處理 asset 變體。例如貼圖有分 sd、hd 版。

Unity 的程式開發人員，從 Unity 5 開始，應該要發展一套能夠方便使用 AssetBundle 的系統，如此便能不再倚賴 Resources。

<!--more-->

## 如何正確地使用 Resources

但我還是想用 Resources，那該怎麼做？

下列情況下仍可以考慮使用 Resources。

1. 存放在 Resources 下的 assets 影響記憶體控管的幅度較小。
2. 此 asset 從頭到尾都在 app 裡被使用。
3. 鮮少需要更新，甚至不需要更新。
4. Asset 不相依於任何平台或裝置。

例如一些設定檔、文字檔、全域 prefab 之類的就可以放在 Resources。

另外，如果開發者沒有一套基於 AssetBundle 的資源管理系統，那麼 prototyping 還是可以考慮先使用 Resources，但還是建議之後別用 Resources，趕快實作一套基於 AssetBundle 的資源管理系統來取代 Resources。

# 參考資料

- Unity, The Resources folder, https://unity3d.com/learn/tutorials/temas/best-practices/resources-folder
