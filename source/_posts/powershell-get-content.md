---
title: PowerShell Get-Content 讀檔換行陷阱
date: 2017-11-18 18:12:23
categories: Technical
tags:
    - PowerShell
    - Get-Content
    - EOL
    - .NET
---

最近遇到一個 PowerShell 的陷阱，用 `Get-Content` 讀檔時，不會保留換行符號。因為 `Get-Content` 行為是單行回傳物件，多行回傳陣列，而不是單一字串。
一種簡單的 EOL 保留方法是

```powershell
(Get-Content <path>) -join "`n"

# e.g.
$content = (Get-Content C:\Users\Me\note.txt) -join "`n"
```

把它串回來即可。
另一種作法是改用 .NET Class，但稍嫌麻煩。

```powershell
[System.IO.File]::ReadAllText("file\to\path")

# e.g.
$content = [System.IO.File]::ReadAllText("C:\Users\Me\note.txt")
```

<!--more-->

# References

- Microsoft (2017), Get-Content, https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-content?view=powershell-5.1
