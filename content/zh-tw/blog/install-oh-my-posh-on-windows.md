---
title: "在 Windows 上安裝 Oh My Posh 替換掉 PowerShell prompt 的樣式"
date: 2021-11-21T11:46:53+08:00
meta_title: ""
description: ""
image: ""
author: "Kán Chúgiâu"
categories:
  - Technical
keywords:
  - PowerShell
  - PowerShell Core
  - pwsh
  - Oh My Posh
  - oh-my-posh
  - Terminal
  - Command Prompt
  - Productivity
  - Windows
  - Theme
tags:
  - PowerShell
  - Windows
  - Productivity
draft: false
---

記錄使用 [Oh My Posh][] 這套在 Windows 上常用的命令列佈景主題引擎，取代掉 Windows 系統內建醜醜的外觀。

# 簡介

Oh My Posh 是一套在 Windows 平台上常用的命令列佈景引擎，可以改顏色、自定義命令列各項區塊，通常用別人已經弄好的佈景主題 (theme)。

這篇只會記錄在 Windows 上安裝使用 Oh My Posh，Linux 及 macOS 我使用的是 [Oh My Zsh][]。

# 安裝 Windows Terminal

說明如何安裝 Windows Terminal，**強烈建議使用 Windows Terminal**，除非使用的 Windows 版本不是 10 18362.0 以上的版本。
如果是 Windows 10 18362.0 以下的版本，可以考慮用 [ConEmu][] 取代 Windows 內建的 Console 視窗。

## 步驟一，開啟 Microsoft Store

通常在 Windows 工具列上，沒有的話就開 Windows Search (熱鍵: `Windows Key`+`S`) 搜尋 Microsoft Store。

![Microsoft Store 工具列](/img/microsoft-store-taskbar.png)

開啟後應該會看到 Microsoft Store 的視窗。

![Microsoft Store](/img/microsoft-store.png)

## 步驟二，搜尋 Windows Terminal

在 Microsoft Store 的搜尋列上搜尋 `Windows Terminal`。

![在 Microsoft Store 搜尋 Windows Terminal](/img/search-windows-terminal-in-microsoft-store.png)

接著按下**安裝**等它安裝完即可。

## (選擇性) 步驟三，設定 Windows Terminal 為 Windows 預設的終端機應用程式

這個功能設定在 Windows 11 才有。

可以直接用 Windows Search 搜尋`終端機設定`找出設定預設 terminal app 的系統設定視窗。
如果找不到，位置在`設定` > `隱私權與安全性` > `開發人員專用` > `終端機`。

![設定 Windows 預設終端機應用程式為 Windows Terminal](/img/windows-default-terminal-app.png)

# 安裝 PowerShell 7

**極度建議安裝 PowerShell 7 取代系統內建的 PowerShell**

用微軟官方提供的 MSI 安裝檔安裝最省事: <https://docs.microsoft.com/zh-tw/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.2#msi>

或是直接按 <https://github.com/PowerShell/PowerShell/releases/download/v7.2.0/PowerShell-7.2.0-win-x64.msi> 下載並安裝。

備註：

* 寫這篇時 PowerShell 的最新版本是 7.2.0。
* PowerShell 之前發布過 PowerShell Core 6，但從第 7 版開始與 .NET 同步命名，名稱把 Core 去掉。 

# 安裝 Oh My Posh

說明如何安裝 Oh My Posh。

Oh My Posh 官方寫得很完整，覺得看英文也 OK 的話可以直接看官方文件: <https://ohmyposh.dev/docs>

## 安裝方法一，直接用 PowerShell

參考 <https://ohmyposh.dev/docs/pwsh>

1. 打開 Windows Terminal，頁籤記得選 PowerShell 7 或 PowerShell Core，不要選 Windows 內建的 PowerShell。
2. 輸入下列指令以安裝 `oh-my-posh`。

   ```powershell
   Install-Module oh-my-posh -Scope CurrentUser
   ```

## 安裝方法二，透過套件管理工具 [Chocolatey][]

參考 <https://ohmyposh.dev/docs/windows>

此方法可讓 Oh My Posh 透過 `oh-my-posh.exe` 啟動指定 shell，例如 powershell，可在 Windows Terminal 設定直接開啟一個頁籤。

輸入下列指令以安裝 `oh-my-posh`

``` powershell
choco install -y oh-my-posh
```

# 安裝 Oh My Posh 佈景主題

Oh My Posh 安裝完成後，接著說明如何安裝佈景主題。

## 簡介

以 PowerShell 為例，每位使用者在登入 shell 時會載入各自的 profile，我們將使用 profile 讓 PowerSell 自動在使用者登入後啟用指定的佈景主題，這些 profile 本身也是 shell script。

## 安裝 [Nerd Fonts][]

不管是 Oh My Posh 還是 Oh My Zsh 的 demo，我們會看到畫面上出現了許多特殊 icon 或其他圖案，那些是用這套 patch 過的字體畫出來的。
如果想要顯示那些炫炮的字體，要安裝 Nerd Font。

1. 到 <https://www.nerdfonts.com/font-downloads> 挑一個你喜歡的字體，我是用 `JetBrainsMono Nerd Font`。
2. 下載後解壓縮。
3. 選取所有字型檔，接著按右鍵，按安裝。
4. 等系統把字型檔裝完後即可。

![安裝 Nerd Fonts](/img/install-nerd-fonts.png)

## 設定 Windows Terminal 外觀字型及配色

字型裝完後，要把終端機的字型換掉。

開啟 Windows Terminal 後按下列視窗順序開啟外觀設定

打開 Windows Terminal 的設定頁面：

![打開 Windows Terminal 的設定頁面](/img/windows-terminal-settings.png)

打開頁籤的外觀設定：

![打開頁籤的外觀設定](/img/windows-terminal-settings-appearance.png)

1. 接著把字型換成你剛裝好的 Nerd Font，範例為 `JetBrainsMono Nerd Font Mono`。
2. 然後挑選你要的色彩配置，系統預設為 `Campbell`，範例選了 `One Half Dark`。
3. 最後按右下角的`儲存`鈕即可。

## 挑佈景主題

字型和配色換好後。

可以到 <https://ohmyposh.dev/docs/themes> 看要選用什麼佈景主題，選好之後接下來要設定 profile。

或是開啟 PowerShell 後，輸入下列指令直接在 terminal 下預覽佈景主題：

``` powershell
Import-Module oh-my-posh
Get-PoshThemes
```

## 設定 PowerShell profile

輸入下列命令來新增或修改 profile

**Notepad++**

``` powershell
notepad++.exe $PROFILE
```

**Visual Studio Code**

``` powershell
code $PROFILE
```

**Windows 內建記事本**

``` powershell
notepad.exe $PROFILE
```

接著添加下列內容

``` powershell
Import-Module oh-my-posh
Set-PoshPrompt -Theme <你想要的佈景主題名稱>
```

其中 `Import-Module oh-my-posh` 的用途是讓使用者登入 shell 時自動載入 oh-my-posh，省去每次啟動都要打這行指令。

例如

``` powershell
Import-Module oh-my-posh
Set-PoshPrompt -Theme night-owl
```

這樣每次開啟 PowerShell 7 都會自動啟用 oh-my-posh 並將佈景主題切換成 night-owl，效果如下：

![Oh My Posh Theme night-owl](/img/oh-my-posh-theme-night-owl.png)

# 參考

- Jan De Dobbeleer and contributors (2021), _Oh My Posh Docs_, <https://ohmyposh.dev/docs>
- Kayla Cinnamon, Matt Wojciakowski (2021), _Oh-My-Posh in PowerShell theme for Windows Terminal_, <https://docs.microsoft.com/en-us/windows/terminal/custom-terminal-gallery/powerline-in-powershell>

[Oh My Posh]: https://ohmyposh.dev/
[Oh My Zsh]: https://ohmyz.sh/
[ConEmu]: https://conemu.github.io/
[Nerd Fonts]: https://www.nerdfonts.com/
[Chocolatey]: https://chocolatey.org/
