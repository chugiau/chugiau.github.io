---
title: "在 Windows 上安裝並使用炫炮的終端機環境"
date: 2022-07-17T20:38:45+08:00
draft: false
categories:
  - Technical
keywords:
  - Windows
  - Terminal
  - PowerShell
  - PowerShell 7
  - PowerShell Core
  - pwsh
  - oh-my-posh
  - Command Prompt
  - scoop
  - theme
  - productivity
tags:
  - PowerShell
  - Windows
  - Productivity
---

繼[在 Windows 上安裝 Oh My Posh 替換掉 PowerShell prompt 的樣式](/posts/install-oh-my-posh-on-windows)這篇文章之後，我想提供我個人在 Windows 的 Terminal 開發環境，也順便用來記錄終端機環境如何安裝。

以下安裝說明僅支援 Windows 10 以上的版本，包括 Windows 11。

# 安裝 Windows Terminal

首先安裝終端機來取代系統內建的 `cmd` 小黑窗或 PowerShell 5.x。

如果是使用 Windows 11 或有在使用 [`winget`][winget] 的人，可以打開終端機輸入下列命令安裝 [Windows Terminal][]。

```powershell
winget install Microsoft.WindowsTerminal
```

或是到 [Microsoft Store][] 上搜尋 `Windows Terminal`，然後安裝：

![Windows Terminal app on Microsoft Store](/img/search-windows-terminal-in-microsoft-store.png)

如果你是使用 Windows 11，可以確認預設終端機應用程式是否有改成 [Windows Terminal][]：

![Default terminal app settings on Windows](/img/windows-default-terminal-app.png)

# 安裝套件管理系統 (Scoop/Chocolatey/winget)

Windows 套件管理系統我到寫這篇文章時，個人偏好使用 [Scoop][]，簡單來說，scoop 比較貼近 UNIX-like 系統的套件管理系統的使用體驗。例如 Debian-based 的 APT，macOS 的 [Homebrew][]。

你可以選擇 [Chocolatey][] 或 [`winget`][winget]。順帶一提，Windows 11 已經內建 `winget`。

這裡以安裝 [Scoop][] 為例，按 Scoop 官方提供的安裝方式，輸入下列命令來安裝 Scoop：

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex
```

接下來的套件若未特別說明，均以 [Scoop][] 安裝。

# 安裝 PowerShell 7

[PowerShell 7][PowerShell] 有別於 Windows 內建的 PowerShell 5.x (習慣使用 `powershell.exe`) 以下的版本，PowerShell 6.x 以上的版本為 PowerShell Core (習慣上使用 `pwsh.exe` 與 5.x 以下版本做區分)，是新一代 PowerShell 且跨平台。但自第 7 版開始統一改叫 PowerShell 7，不再特別標示 Core 版。

透過套件管理安裝：

```powershell
scoop install pwsh
```

# Windows Terminal 新增 PowerShell 7 設定檔

如果 [Windows Terminal][] 沒有新增 PowerShell 7 (`pwsh.exe`) 設定檔，則可按下列步驟新增。

1. 開啟 Windows Terminal
2. 按下 `+` 圖示右邊的下拉式選單按鈕。
   ![Add pwsh profile from Windows Terminal settings](/img/windows-terminal-settings-add-profile.png)
3. 調整設定，將命令列欄位改成 `pwsh`, `pwsh.exe` 或 PowerShell 7 的絕對路徑。
4. 所有選項設定完後按下 `[儲存]` 按鈕。
5. 再次按下 `+` 按鈕旁的下拉式選單查看有沒有剛剛新增的 PowerShell 7 設定。

關於直接修改 Windows Terminal 的 `settings.json`，可參考下列 JSON 設定片段新增 profile：

```json
{
  "profiles":
  {
    "list":
    [
      {
        "background": "#2A2A2A",
        "closeOnExit": "graceful",
        "commandline": "pwsh",
        "cursorColor": "#FFFFFF",
        "guid": "{574e775e-4f2a-5b96-ac1e-a2962a402336}",
        "hidden": false,
        "historySize": 9001,
        "icon": "ms-appx:///ProfileIcons/{574e775e-4f2a-5b96-ac1e-a2962a402336}.png",
        "name": "PowerShell 7",
        "snapOnInput": true,
        "source": "Windows.Terminal.PowershellCore",
        "startingDirectory": "%USERPROFILE%"
      }
    ]
  }
}
```

修改 Windows Terminal 的預設啟動 profile：

1. 打開 Windows Terminal 設定介面，左邊清單選擇`[啟動]`。
2. 將`預設設定檔`欄位改成剛才新增的 `PowerShell 7`。
3. 儲存。
4. 按下 `+` 按鈕或是 Windows Terminal 關掉重開看是不是啟動剛才設定的 profile。

![Change the default launching profile](/img/windows-terminal-settings-launch-default-profile.png)

# 修改 Windows Terminal 配色

接著調整配色，Windows Terminal 已經內建幾款常見配色，如果覺得不夠，可以到 [Windows Terminal Themes][] 找自己喜歡的配色，或是自己調。

例如挑 `Ayu Migrate` 這款配色，該 scheme 如下：

```json
{
  "name": "Ayu Mirage",
  "black": "#191e2a",
  "red": "#ed8274",
  "green": "#a6cc70",
  "yellow": "#fad07b",
  "blue": "#6dcbfa",
  "purple": "#cfbafa",
  "cyan": "#90e1c6",
  "white": "#c7c7c7",
  "brightBlack": "#686868",
  "brightRed": "#f28779",
  "brightGreen": "#bae67e",
  "brightYellow": "#ffd580",
  "brightBlue": "#73d0ff",
  "brightPurple": "#d4bfff",
  "brightCyan": "#95e6cb",
  "brightWhite": "#ffffff",
  "background": "#1f2430",
  "foreground": "#cbccc6",
  "selectionBackground": "#33415e",
  "cursorColor": "#ffcc66"
}
```

把配色設定新增到下列設定檔 (`settings.json`) 區段：

```json
{
  "schemes":
  [
    {
      "name": "Ayu Mirage",
      "black": "#191e2a",
      "red": "#ed8274",
      "green": "#a6cc70",
      "yellow": "#fad07b",
      "blue": "#6dcbfa",
      "purple": "#cfbafa",
      "cyan": "#90e1c6",
      "white": "#c7c7c7",
      "brightBlack": "#686868",
      "brightRed": "#f28779",
      "brightGreen": "#bae67e",
      "brightYellow": "#ffd580",
      "brightBlue": "#73d0ff",
      "brightPurple": "#d4bfff",
      "brightCyan": "#95e6cb",
      "brightWhite": "#ffffff",
      "background": "#1f2430",
      "foreground": "#cbccc6",
      "selectionBackground": "#33415e",
      "cursorColor": "#ffcc66"
    }
  ]
}
```

新增完後，可以選擇修改預設色彩配置，或是只修改某個 profile 的色彩配置。這裡以修改預設配置為例：

![Change default colro scheme](/img/windows-terminal-settings-global-color-scheme.png)

# 安裝字體

接下來要安裝 patch 過的字體，提供更美觀的字體及豐富的圖示。

到 [Nerd Fonts][] 選擇一個你喜歡的字體，然後下載下來並安裝。例如我選擇的是 patch 過的 `JetBrains Mono`。建議選擇有完整 monospace 的字體，因為後續終端機圖案顯示會有影響。

接著修改 Windows Terminal 字體設定。例如我採用 `JetBrainsMono Nerd Font Mono`：

![Change the default font](/img/windows-terminal-settings-default-font.png)

你也可以順便把外觀其他選項調一調，例如字體大小。

# 安裝 Oh My Posh

參考 <https://ohmyposh.dev/docs/installation/windows>

安裝 [Oh My Posh][] 本體：

```powershell
scoop install https://github.com/JanDeDobbeleer/oh-my-posh/releases/latest/download/oh-my-posh.json
```

# 安裝 PowerShell Modules

說明安裝哪些 PowerShell 模組，你可以到 [PowerShell Gallery][] 尋找更多 modules。

其他注意事項：

- 下列執行範例會裝給所有人使用，如果要限定只安裝給當前使用者使用，增加參數 `-Scope CurrentUser`。
- 如果安裝時不想再三確認，則新增參數 `-Force`。
- 如果想要使用 prerelease 版，新增參數 `-AllowPrerelease`。

## 安裝 `oh-my-posh` module

```powershell
Install-Module -Name oh-my-posh
```

若要更新 Oh My Posh module，執行下列命令：

```powershell
Update-Module oh-my-posh
```

## 安裝 `Terminal-Icons` module

```powershell
Install-Module -Name Terminal-Icons
```

## 安裝 `PSReadLine` module

```powershell
Install-Module -Name PSReadLine
```

## 安裝 `posh-git` module

```powershell
Install-Module -Name posh-git
```

## 調整 PowerShell profile

輸入下列命令修改你的 PowerShell profile，避免每次啟動都要重新設定，使用任何一款你順手的文字編輯工具修改 profile。例如使用 Visual Studio Code (有安裝命令列工具整合)：

```powershell
code $PROFILE
```

下列是我個人使用的 profile，供參考：

```powershell
Import-Module oh-my-posh
Import-Module posh-git
Import-Module PSReadLine
Import-Module Terminal-Icons

Set-PoshPrompt -Theme slimfat

Set-PSReadlineKeyHandler -Key Tab -Function Complete
Set-PSReadlineKeyHandler -Key Tab -Function MenuComplete
Set-PSReadLineKeyHandler -Key "Ctrl+z" -Function Undo

Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle ListView
```

這個 profile 作用如下：

1. 每次啟動 pwsh 時，會自動載入 `Import-Module` 那些 module，接著用 `Set-PoshPrompt` 修改 PowerShell 佈景主題為 `slimfat`，佈景主題可到 [Oh My Posh Themes][] 搜尋你偏好的樣式。
2. 透過 `PSReadLine` 提供的功能去修改熱鍵。
   1. 自動完成用 `Tab` 鍵。
   2. `Ctrl+Z` 按下去是執行復原 (Undo)。
3. 命令預測改成條列檢視。

設定完後，可把 Terminal 關掉重開，或直接輸入 `. $PROFILE` 重新載入 profile，參考截圖：

![Windows Termainl with Oh My Posh](/img/windows-terminal-oh-my-posh-1.png)

# 參考

- Will 保哥 (2021), _如何打造一個華麗又實用的 PowerShell 命令輸入環境_, <https://blog.miniasp.com/post/2021/11/24/PowerShell-prompt-with-Oh-My-Posh-and-Windows-Terminal>

[Chocolatey]: https://chocolatey.org/
[Homebrew]: https://brew.sh/
[Microsoft Store]: https://apps.microsoft.com/store/apps
[Nerd Fonts]: https://www.nerdfonts.com/
[Oh My Posh]: https://ohmyposh.dev/
[Oh My Posh Themes]: https://ohmyposh.dev/docs/themes/
[PowerShell]: https://docs.microsoft.com/en-us/powershell/scripting/overview/
[PowerShell Gallery]: https://www.powershellgallery.com/
[Scoop]: https://scoop.sh/
[Windows Terminal]: https://docs.microsoft.com/en-us/windows/terminal/
[Windows Terminal Themes]: https://windowsterminalthemes.dev/
[winget]: https://docs.microsoft.com/en-us/windows/package-manager/winget/
