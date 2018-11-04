---
title: Unity 5 AssetBundle 基礎
date: 2017-06-21 08:25:32
categories:
  - Technical
tags:
  - Unity
  - AssetBundle
  - AssetBundleManifest
  - WWW
  - UnityWebRequest
---


# 前言

一直以來，Unity 的 `AssetBundle` 機制與不透明的資訊一直為人所詬病（社群論壇有些批評，例如 asset 序列化管理架構、機制本身就不良，來源就不附上了），以往大部分的資訊都只是瞎子摸象試出來的結果，大都未經 Unity 官方證實。
直到 Unity 4、5 開始，特別是從 Unity 5.x 開始，官方才比較願意公開詳細地說明 Unity `AssetBundle`、檔案資源系統及 `Resources` 機制。

包括我在內，相信有不少 Unity 開發人員踩了不少 `AssetBundle` 相關的地雷，因此寫了這篇來記錄 `AssetBundle` 相關的基礎知識與細節。這篇主要是以 Unity 官方的 AssetBundle fundamentals 主題文章為主，加上一些自身開發經歷與參考。

# `AssetBundle` 概觀

`AssetBundle` 是一款 Unity 的檔案資源系統，目的是為了讓資料交付可以相容於 Unity 自身的序列化系統。因為 Unity 本身的檔案資源處理機制，導致沒辦法直接使用檔案，原始 asset 檔與 Unity 最終在使用的 asset 是不同的。
次要目的是縮減最終發布的 asset 大小，縮減執行期記憶體用量，可根據目標裝置給予最適合的內容（例如根據不同裝置選擇貼圖壓縮格式）等。

# `AssetBundle` 內部儲存結構

一包 `AssetBundle` 由兩個部分組成。

1. 檔頭 (header)
2. 資料段 (data segment)


## `AssetBundle` Header

檔頭由 Unity 在建置 `AssetBundle` 時產生，記錄了這包 `AssetBundle` 的 ID、壓縮格式、manifest 等資訊。

### Header Manifest

這裡的 manifest 是指檔頭的 manifest，不是指 `AssetBundle` 建置出來的 manifest。

Manifest 是一張查找表，以 `UnityEngine.Object` 名稱為主鍵，值為此 `UnityEngine.Object` 指向資料段的索引。官方指出大部分的平台都是用 C++ STL `std::multimap` 來實作，因此一個 `UnityEngine.Object` 可能會有很多個資料段。官方也說明演算法會根據不同平台來選擇以利最佳化，大都是使用各種平衡搜尋樹 (balanced search tree)。例如在 Windows、OSX 及 iOS 會採用紅黑樹 (red-black tree)。因此在建構 manifest 時，消耗的時間不是線性，會比線性還久。

## `AssetBundle` Data Segment

`AssetBundle` 資料段保存了 Unity 序列化 asset 後的原始資料。若資料段是被壓縮的，則會根據壓縮演算法來選擇要如何解壓縮這些資料段。例如 LZMA 壓縮會蒐集所有的資料段後才解壓縮，這代表了 LZMA 得解壓縮一整包 `AssetBundle` 才能取出裡面的 `UnityEngine.Object`，即便你只是想要取出某一個 `UnityEngine.Object`。而 Unity 5.3 之後新增的 LZ4 則是個別壓縮 `UnityEngine.Object`，允許獨立解壓縮取出一個，不用解壓縮一整包才能取出 `UnityEngine.Object`。
<!--more-->
# 載入 `AssetBundle`s

Unity 5 提供了 4 種可載入 `AssetBundle` 的 APIs，這些 APIs 的載入行為會根據下列兩種準則而有所不同。

1. `AssetBundle` 壓縮格式，LZMA、LZ4 或者是不壓縮。
2. 正在載入 `AssetBundle` 的目標平台。

這 4 個 APIs 分別為：

1. [AssetBundle.LoadFromMemoryAsync](http://docs.unity3d.com/ScriptReference/`AssetBundle`.LoadFromMemoryAsync.html) (`LoadFromMemory`)，5.3.2 之前版本名為 `CreateFromMemory`
2. [AssetBundle.LoadFromFile](http://docs.unity3d.com/ScriptReference/`AssetBundle`.LoadFromFile.html) (`LoadFromFileAsync`)，5.3.2 之前版本名為 `CreateFromFile`
3. [WWW.LoadFromCacheOrDownload](http://docs.unity3d.com/ScriptReference/WWW.LoadFromCacheOrDownload.html)
4. [UnityWebRequest](http://docs.unity3d.com/ScriptReference/Networking.UnityWebRequest.html)'s [DownloadHandler`AssetBundle`](http://docs.unity3d.com/ScriptReference/Networking.DownloadHandler`AssetBundle`.html) (on Unity 5.3 or newer)

## `AssetBundle.LoadFromMemoryAsync`

**Unity 官方不建議使用此 API。**

為什麼不建議使用呢？因為這方法十分低效，整個流程如下。

1. Client code 配置一組 managed-code byte array (C# `byte[]`)。
2. `AssetBundle.LoadFromMemoryAsync` 又再複製一份從 client code 來的 managed-code byte array 到 native memory。
3. 如果 `AssetBundle` 是 LZMA 壓縮，則會在解壓縮的同時複製。LZ4 壓縮或不壓縮的 `AssetBundle`s 則是逐 byte 複製。

這代表了這個 API 的**記憶體最大消耗量至少是 `AssetBundle` 本身大小的兩倍**，一份在 managed-code，另一份在 native memory 複本。
當用此 API 產生的 `AssetBundle` 來載入 assets 時，**記憶體消耗量將會是三倍**。一份在 managed-code，一份在 native memory 複本，另一份則是 asset 本身，這個 asset 的記憶體位置可能是在 GPU 或主記憶體。

這個 API 在 Unity 5.3.2 之前的版本名稱為 `AssetBundle.CreateFromMemoryAsync`，功能一樣，只是換個名字。

## `AssetBundle.LoadFromFile`

這個 API 是比較高效的作法，Unity 官方建議用此 API，而不是 `AssetBundle.LoadFromMemory`。其中，若壓縮格式為 LZ4，則這個 API 會有特定的行為：

- **行動裝置**
  API 只會載入 `AssetBundle` 的檔頭，其餘的資料仍留存在硬碟上。`AssetBundle` 的物件將會基於「按需求 (on-demand)」的模式來載入，此情況下不會消耗額外記憶體空間。如果你的資源管理系統在第一次載入特別地慢，就得注意一下這點。
- **Unity Editor**
  API 將會把整包 `AssetBundle` 載入到記憶體，行為很像 `AssetBundle.LoadFromMemoryAsync`。因此，在 Editor 下測試記憶體用量是不準確的。

### 補充說明

Note: 這個 API 在 Unity 5.2 之前的版本名稱為 `AssetBundle.CreateFromFile`，功能一樣，只是換個名字。

Note: 在 Unity 5.3 版之前，這個 API 在 Android 平台上若試著從 StreamingAssets 路徑載入 `AssetBundle`s，則會發生錯誤。這是因為內容物在壓縮過的 .jar 檔裡。在 Unity 5.4 以上的版本修正了這個 bug。

Note: Unity 在 5.3.7f1 及 5.4.3f1 之後的版本修正了`AssetBundle.LoadFromFile` 在載入 LZMA 壓縮的 `AssetBundle`s 時總是會失敗的 bug。

## `WWW.LoadFromCacheOrDownload`

這個 API 可用來從遠端或本地端下載，或從快取載入 `AssetBundle`。如果快取不存在，則會從來源下載 `AssetBundle` 來並快取。如果快取存在，則以類似 `AssetBundle.LoadFromFile` 的方式載入 `AssetBundle`。

- **未快取**
  若未快取，則 `WWW.LoadFromCacheOrDownload` 會從來源端讀取 `AssetBundle`。接著，若 `AssetBundle` 是有壓縮的，則將會在一條工作執行緒來解壓縮，並寫到快取。反之若沒壓縮，則會直接用工作執行緒把 `AssetBundle` 寫到快取。
- **已快取**
  若已快取，則 `WWW.LoadFromCacheOrDownload` 將會從快取載入檔頭、解壓縮後的 `AssetBundle`。其中載入行為與 `AssetBundle.LoadFromFile` 一樣。

使用 `WWW` 時要注意 `WWW.bytes`，為了支援此屬性存取，Unity 會在 native memory 保留一份完整的 `AssetBundle` bytes。所以，若是用此方法載入 `AssetBundle`，則務必要控制每包 `AssetBundle` 的大小，例如 8 MiB，特別是在行動裝置上或記憶體十分有限的硬體上。另外，`WWW` 取出 `AssetBundle` 後，`WWW` 也得清除乾淨。

另外，Unity 官方指出建議同時執行的 `WWW` 最好控制在一定數量之內，例如 2 個。因為每次呼叫一次 `WWW.LoadFromCacheOrDownload` 都會產生一條工作執行緒來執行此作業。

# `UnityWebRequest` (`DownloadHandlerAssetBundle`)

Unity 從 5.3 版開始，推出了 `UnityWebRequest` API 來作為 `WWW` 的替代品，我個人也建議在後續新版的 Unity 改用 `UnityWebRequest`，盡量不要再用 `WWW`。`UnityWebRequest` 更有彈性，允許開發者決定要如何處理下載下來的資料，更能避免額外的記憶體開銷。`UnityWebRequest` 內建了 `UnityWebRequest.GetAssetBundle` 方法來取得 `AssetBundle`。

`DownloadHandlerAssetBundle` 的行為類似 `WWW.LoadFromCacheOrDownload`，開一條工作執行緒，串流下載資料到固定大小的緩衝區，根據 `DownloadHandler` 的設定，將會決定要把緩衝區的資料串起來放到另一個暫存空間，還是放到 `AssetBundle` 快取。LZMA 壓縮的 `AssetBundle` 將會一邊下載一邊解壓縮到快取。

為了解決記憶體開銷及 managed heap 問題，`DownloadHandler` 操作都會在 native-code 完成，並且不會在 native-code 保留或複製下載下來的 bytes，從而解決已往下載 `AssetBundle` 導致記憶體開銷過大的問題。

`UnityWebRequest` API 也以 `WWW.LoadFromCacheOrDownload` 相同的方式支援快取。也就是若 `UnityWebRequest` 存在某 `AssetBundle` 的快取，則載入此 `AssetBundle` 時將會以與 `AssetBundle.LoadFromFile` 相同的方式載入 `AssetBundle`。要注意的是，`UnityWebRequest` 與 `WWW` 的快取是共享的。另外 `UnityWebRequest` 不像 `WWW`，`UnityWebRequest` 擁有自己內部的執行緒池，這直接限制了同一時間可執行的 `UnityWebRequest` 數量。目前 Unity 不提供任何方式來設定執行緒池大小。

## 建議作法

一般來說，盡可能採用 `AssetBundle.LoadFromFile`，不要用 `AssetBundle.LoadFromMemoryAsync`。

Unity 官方也極度建議 Unity 5.3 之後的版本改用 `UnityWebRequest`，而不是 `WWW`。

若使用 `WWW.LoadFromCacheOrDownload` 時，官方強烈建議專案的 `AssetBundle`s 的最好小於最大記憶體預算的 2-3%，避免記憶體用量爆衝導致閃退。

`AssetBundle` 的大小建議限制在 5 MiB 以內，同一時間可執行下載作業的數量為 1-2。

`WWW` 及 `UnityWebRequest`，當不再需要時，應該要呼叫 `Dispose` 來卸載，請善用 C# 的 `using` 述句。

對於大型工程團隊，而且有特殊快取、下載需求時，可能有必要自行實作一個下載器。自行實作一個下載器頗麻煩，而且要與 `AssetBundle.LoadFromFile` 相容。詳細內容可以參考下一章的 [Distribution](https://unity3d.com/learn/tutorials/temas/best-practices/assetbundle-usage-patterns#Distribution)。

# 從 `AssetBundle`s 載入 Assets

`AssetBundle` 主要有 3 種 APIs 可供載入 assets，並且這 3 種 APIs 都有同步與非同步版：

| 同步 (Sync)                | 非同步 (Async)                   | 用途                       |
| ------------------------ | ----------------------------- | ------------------------ |
| `LoadAsset`              | `LoadAssetAsync`              | 載入單一 asset。              |
| `LoadAllAssets`          | `LoadAllAssetsAsync`          | 載入所有 assets。             |
| `LoadAssetWithSubAssets` | `LoadAssetWithSubAssetsAsync` | 載入指定 asset 及其相依的 assets。 |

同步版的 API 總是會比非同步快，至少快 1 frame。

官方特別提到，Unity 5.1 版之前的非同步 APIs 每一個 frame 只會載入一個 `UnityEngine.Object`，因此 `LoadAllAssetsAsync` 及 `LoadAssetWithSubAssetsAsync` 會比對應的同步版 APIs 還要慢很多。這個問題在 Unity 5.2 之後的版本已被修正。

### 用途解說

#### `LoadAllAssets`, `LoadAllAssetsAsync`

`LoadAllAssets` 應該用來載入多個單獨 `UnityEngine.Object`（彼此之間無關聯），並且只在用到該包的主要 `UnityEngine.Object` 或其所有 `UnityEngine.Object`s 才會使用此 API。效能表現上，這個 API 會比呼叫數次 `LoadAsset` 來得快一點。因此，若一次載入的 assets 數量很大，但載入的數量卻又不到 2/3，那就該考慮把這包 `AssetBundle` 拆得更細。

個人經驗行動裝置上的遊戲比較少到用此 API，主要是為了解決記憶體消耗量，特別是峰值，因此 `AssetBundle` 拆分粒度要夠細。再來是資源重複打包的問題，在舊版 Unity 經過這問題的洗禮，所以本來就會拆解資源相依問題。慶幸的是新版 Unity 在處理相依性變得非常方便了。

#### `LoadAssetWithSubAssets`, `LoadAssetWithSubAssetsAsync`

這個 API 用來載入複合的 asset，意即內嵌了 `UnityEngine.Object`s。例如 FBX 模型內嵌了動畫、sprite 等。另一個適用情況是需要載入的 `UnityEngine.Object`s 都來自同一個 asset，同時這個 `AssetBundle` 裡有許多其他不相干的 `UnityEngine.Object`s，那就可以使用這個 API。

#### `LoadAsset`, `LoadAssetAsync`

其餘狀況就使用此 API。

### 底層載入細節

`UnityEngine.Object` 載入作業不是在主執行緒執行，`UnityEngine.Object` 的資料是在工作執行緒上從儲存體 (storage) 讀取。任何不觸及 Unity 系統中執行緒敏感的部分 (腳本、圖形) 的東西都會放在工作執行緒上。例如從 meshes 建立 VBOs，解壓縮 textures 等。

在 Unity 5.3 版之前，`UnityEngine.Object` 是依序載入，並且 `UnityEngine.Object` 的某些部分是只能在主執行緒上載入。這步驟稱為「整合」(integration)。當工作執行緒載完 `UnityEngine.Object` 的資料，則這個工作執行緒會暫停，並且把這些載完的資料整合到主執行緒裡。在主執行緒完成整合之前，工作執行緒會暫停，不會繼續載入下一個 `UnityEngine.Object`。

Unity 從 5.3 版開始，`UnityEngine.Object` 的載入作業是平行的 (parallelized)。多個 `UnityEngine.Object`s 可以在工作執行緒上被反序列化 (deserialized)、處理及整合。接著 `Awake` callback 將會被呼叫，並且在下一個 frame 時，在 Unity 引擎中變成可用狀態。

同步版的 `AssetBundle.Load` 系列方法會暫停主執行緒，直到 `UnityEngine.Object` 載入完畢。而在 Unity 5.3 版之前的非同步版 `AssetBundle.LoadAsync` 系列方法，則在進行整合作業前，都不會暫停主執行緒。`UnityEngine.Object` 加載會分配時間切片 (time-slice) 以確保載入時不會占用每一個 frame 太多時間（單位為毫秒）。這個值可以在 `Application.backgroundLoadingPriority`設定：

- `ThreadPriority.High`: 每一個 frame 的上限為 50 毫秒。
- `ThreadPriority.Normal`: 每一個 frame 的上限為 10 毫秒。
- `ThreadPriority.BelowNormal`: 每一個 frame 的上限為 4 毫秒。
- `ThreadPriority.Low`: 每一個 frame 的上限為 2 毫秒。

在 Unity 5.1 或更舊的版本中，非同步版的 APIs 每一個 frame 只會整合一個 `UnityEngine.Object`。其實這是一個 bug，在 Unity 5.2 之後的版本已經被修正了。正確的行為是在每一個 frame 可以執行多個 `UnityEngine.Object`s 的整合作業，直到 frame-time 達上限。`AssetBundle.LoadAsync` 系列方法耗用時間總是會比同步版的 APIs 來得久，至少慢一個 frame，因為在 `AssetBundle.LoadAsync` 系列方法呼叫後，要讓 `UnityEngine.Object` 在 Unity 引擎裡變成可用狀態至少要等到下一個 frame。

官方提供的實測數據：

Unity 5.2 前，在某個低端設備上載入大量 textures，同步版需要 7 毫秒，非同步版需要 70 毫秒。而在 Unity 5.2 之後，同步與非同步幾乎沒有差別，差距接近 0。

### `AssetBundle` 相依性

在 Unity 5 的 `AssetBundle` 系統，`AssetBundle`s 之間的相依性會透過兩種不同的 APIs 追蹤。實際上要用哪一種 API 則是由執行期環境決定。
在 Unity Editor 時，可以用 `AssetDatabase` 的 API 來查詢。`AssetBundle` 的設定及相依性可以用 `AssetImporter` API 來存取或修改。
在執行期時，Unity 提供了一個可選用的 API 來載入相依資訊，這個相依資訊是 `AssetBundleManifest`，一個基於 `ScriptableObject` 的 API，`AssetBundleManifest` 在建置 `AssetBundle`s 時會產生。

當一個或多個 parent `AssetBundle` 的 `UnityEngine.Object`s 參照一個或多個其他 `AssetBundle` 的 `UnityEngine.Object`s 時，那麼就存在了間接參照的關係（A 用了 B，B 用了 C，C 和 A 之間有間接參照關係）。物件互相引用的關係之詳細說明，可以去看 [Assets, Objects and Serialization](https://unity3d.com/learn/tutorials/temas/best-practices/assets-objects-and-serialization) 的 [Inter-Object references](https://unity3d.com/learn/tutorials/temas/best-practices/assets-objects-and-serialization#InterObject_References) 這節。

如同 [Serialization and instances](https://unity3d.com/learn/tutorials/temas/best-practices/assets-objects-and-serialization#Serialization_and_Instances) 裡所描述的一樣，`AssetBundle`s 裡儲存了 `UnityEngine.Object`s，並用 FileGUID 及 LocalID 來識別這些物件。

由於 `UnityEngine.Object` 在它的 `AssetBundle` 載入時會給定一個有效的 Instance ID。又，因為一個 `UnityEngine.Object` 在它的 Instance ID 第一次解參照時（意思就是有東西使用這個物件），會載入此 `UnityEngine.Object`。因此 `AssetBundle`s 載入順序並不是很重要，重要的是 `AssetBundle`s 之間裡的物件有沒有相依性。例如 `AssetBundle` A 的 a 物件相依於 B、C、D `AssetBundle`s 裡的物件，那麼 A、B、C、D 這幾個 `AssetBundle`s 之間的載入順序並不重要，只要在提取 A 裡的 a 之前有載入 B、C、D 就可以了。相依資訊可以從 `AssetBundleManifest` 得知。

**官方提供的例子：**

假設 material A 參照了 texture B。Material A 包在 AssetBundle 1，texture B 包在 AssetBundle 2。

![Example](https://unity3d.com/sites/default/files/learn/ab1.jpg)

在這個例子中，AssetBundle 2 必須要先載入，接著才可以載入 AssetBundle 1 裡的 material A。

在這裡必須要再次強調，這不表示一定要先載 AssetBundle 2 才能載 AssetBundle 1。你可以先載 AssetBundle 1 才載 AssetBundle 2，只要保證在取出 material A 之前有載入 AssetBundle 2 就可以了。

由於 Unity 不會自動載入相依關係，開發者得透過腳本來手動載入這些相依關係。用 `AssetBundleManifest` 取得相依資訊，具體範例可以參考 Unity 官方的 [AssetBundle Demo](https://bitbucket.org/Unity-Technologies/assetbundledemo)。

### `AssetBundle` manifests

執行 `BuildPipeline.BuildAssetBundles` API 時，Unity 會把 `AssetBundle` 相依資訊序列化到一個 `AssetBundleManifest` 物件上。

`AssetBundleManifest` 這個 asset 會儲存與 AssetBundle 輸出目錄名稱一樣的 `AssetBundleManifest` 與 \*.manifest (這個是給人看的，不會真的用到) 檔案。例如建置 `AssetBundle`s，輸出到 `<project_root>/build/Client/`，則會產生一個 `<project_root>/build/Client/Client` 的 `AssetBundleManifest` 檔（注意！沒有副檔名），及給人看的 `<project_root>/build/Client/Client.manifest` 檔，它是一個 YAML 格式的檔案。

`AssetBundleManifest` 提供了下列三個 API 供查詢 `AssetBundle`s 名稱及其相依性。

- `AssetBundleManifest.GetAllAssetBundles`
  取得所有的 `AssetBundle`s 名稱清單。
- `AssetBundleManifest.GetAllDependencies`
  取得特定的 `AssetBundle` 之所有相依 `AssetBundle`s 的名稱。例如 A 參照 B，B 參照 C，則用此 API 查詢 A 會回傳 B 和 C。
- `AssetBundleManifest.GetDirectDependencies`
  取得特定的 `AssetBundle` 之直接相依 `AssetBundle`s 的名稱。例如 A 參照 B，B 參照 C，則用此 API 查詢 A 只會回傳 B。

要注意 `AssetBundleManifest.GetAllDependencies` 及 `AssetBundleManifest.GetDirectDependencies` 會配置字串陣列，慎用之，特別是在效能斤斤計較的時候。

### 建議作法

在玩家進入效能關鍵的部分前，例如戰鬥、遊戲主場景、世界等，盡量把需要的 `UnityEngine.Object`s 都載入。尤其是在行動裝置上，因為行動裝置存取本地端儲存體通常很慢，而且在載入、卸載時會變動記憶體，容易觸發 .NET 的垃圾回收 (GC, Garbage Collection)，進而導致遊戲卡頓。

對於那些需要動態載入、卸載的情況，可以參考 [AssetBundle usage patterns](https://unity3d.com/learn/tutorials/topics/best-practices/asset-bundle-usage-patterns) 中的 [Managing loaded assets](https://unity3d.com/learn/tutorials/topics/best-practices/asset-bundle-usage-patterns#Managing_Loaded_Assets)。

# 參考資料

- Unity, AssetBundle fundamentals, https://unity3d.com/learn/tutorials/topics/best-practices/assetbundle-fundamentals
- Unity, AssetBundle Demo, https://bitbucket.org/Unity-Technologies/assetbundledemo
