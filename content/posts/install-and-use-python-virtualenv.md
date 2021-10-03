---
title: 安裝並使用 Python virtualenv
date: 2018-11-03 22:05:01
categories:
  - Technical
keywords:
  - Python
  - virtualenv
  - pyenv
  - brew
  - Homebrew
  - Chocolatey
  - choco
  - macOS
  - Windows
tags:
  - Python
  - virtualenv
---

# 簡介

[virtualenv][] 是一套 [Python][] 常用的虛擬環境工具，可以用來隔離環境所需的套件。

**案例一，避免套件版本衝突**
你的專案 Foo 是使用 Flask，另一項專案 Bar 是使用 Django，而這兩個專案很不巧使用了一樣的套件 LibBaz，但 LibBaz 使用的版本卻不一樣。
這時候 virtualenv 就十分有用。
<!-- more -->
**案例二，不想汙染全域空間**
你的某個 C++ 專案使用了 conan 這個用 Python 寫的套件管理工具，用 pip 安裝。
你不想要安裝 conan 在全域，因為沒有必要。這時 virtualenv 就能幫助開發者隔離這樣的環境。

**案例三，避免 Python 版本衝突**
倘若不使用 pyenv 這類 Python 版本管理工具，就得使用 virtualenv 或類似的工具去隔離執行環境。
例如專案 A 使用 Python 3.5.4，而專案 B 使用 Python 3.7.1，都是 Python 3，但你又不想打 python35 或 python37 這樣的別名。
這時候 virtualenv 能幫助開發者隔離 Python 執行版本了。

# 前提

這篇假設讀者已經對純文字命令列操作有一定程度的熟悉度。

# macOS

這章將說明如何在 macOS 安裝 [Homebrew][]、[Python][] 及 [virtualenv][]。

## 安裝 Homebrew

_如果已經安裝過 Homebrew，可以直接跳過_

[Homebrew][] (以下簡稱 brew，同時命令名稱預設也是叫 `brew`) 是一套方便好用的套件管理工具，用來補足 macOS 沒有像 Linux 那樣有 apt, apt-get 或 yum 等官方套件管理工具。
接下來安裝 python 或 pip 時會用到 brew。

到官網，執行官方提供的 shell script。
http://brew.sh

官方提供的安裝腳本，請以 brew 官方提供的為準：

```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## 安裝 Python

接著說明如何在 macOS 安裝 Python。

### pyenv

個人建議使用 [pyenv][] 安裝並切換 Python 版本。
https://github.com/yyuu/pyenv

如果要直接用 brew 安裝 Python 也是可以，但若有不同專案使用了不同版本的 Python，還是建議用 pyenv 安裝 Python。

### 安裝 pyenv

```shell
brew install pyenv
```

### 安裝指定版本的 Python

**安裝 Python 3.7.1**

```shell
pyenv install 3.7.1
```

**安裝 Python 2.7.15**

```shell
pyenv install 2.7.15
```

### (Optional) 獨立安裝 Python 2

Mac OS X 內建了 Python 2，預設也是 Python 2，除非你要另外裝，但不太建議。

```shell
brew install python2
```

接著請調整全域變數 PATH 的路徑排序，如果要優先使用 brew 安裝的 Python 2，則應改成類似下列設定：

```shell
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:$PATH
```

讓系統先從 `/usr/local/bin` 開始找 `python` 在哪。
極度不建議將 `python` 連結至 Python 3，除非你用的東西通通都不是依賴 Python 2。

### (Optional) 獨立安裝 Python 3

```shell
brew install python3
```

## 安裝 pip

pip 已內建在 Python 2 (>= 2.7.9)，或 Python 3 (>= 3.4)，但仍然需要自行升級 pip 至最新版。

### 用 Homebrew 安裝 pip

```shell
brew install pip
```

### 升級 pip

```shell
pip install --upgrade pip
```

## virtualenv

此章將說明如何安裝並使用 virtualenv。

### 安裝 virtualenv

```shell
pip install virtualenv
```

#### 建立 virtualenv

```shell
# 假設當前路徑在 /Users/me/Workspace/myproject/
virtualenv .venv
```

`.venv` 是虛擬環境的根目錄名稱，可以隨你的意命名。但按慣例，Python 虛擬環境目錄名稱通常是 `.venv`, `env`, `venv`。

#### 啟用 virtualenv

```shell
source .venv/bin/activate
```

#### 退出 virtualenv

```shell
deactivate
```

### 應用

```shell
#!/bin/sh

# 此 shell script 是用來安裝專案開發要用的 Python 環境

VENV_DIR=".venv"

# 檢查 .venv 目錄是否存在
if [ -s "$VENV_DIR" ]; then
  echo "Virtual environment '$VENV_DIR' already exists."
  exit
fi

# 安裝 Python 虛擬環境到 .venv
pip install $VENV_DIR

# 啟用虛擬環境
source $VENV_DIR/bin/active

# 升級虛擬環境裡的 pip
pip install --upgrade pip

# 安裝專案有用到的 packages
pip install Flask

# 退出虛擬環境

deactivate
```

# Windows

這章說明如何在 Windows 安裝 [Chocolatey][]、[Python][] 及 [virtualenv][]。

## 安裝 Chocolatey

[Chocolatey][] 是一套在 Windows 的套件管理工具，用來補足 Windows 沒有像 macOS 的 Homebrew，或 Linux apt, yum 等套件管理工具的麻煩。

到 [https://chocolatey.org/install](https://chocolatey.org/install) 根據指示安裝 Chocolatey。

## 安裝 Python 3

```shell
choco install -y python3
```

## 安裝 pip

同 macOS，Python 2.7 以上，或 3.4 以上的版本已自帶 pip，但仍需自行更新 pip 至最新版。

### 更新 pip

```shell
pip install --upgrade pip
```

如果無法直接用 pip 升級，請改用下列指令：

```shell
python -m pip install --upgrade pip
```

## 安裝 virtualenv

```shell
pip install virtualenv
```

### 建立 virtualenv

```shell
# 假設當前路徑在 C:\Users\me\Workspace\myproject\
virtualenv .venv
```

`.venv` 是虛擬環境的根目錄名稱，可以隨你的意命名。但按慣例，Python 虛擬環境目錄名稱通常是 `.venv`, `env`, `venv`。

### 啟用 virtualenv

```shell
.venv\Scripts\activate
```

如果你遇到無法 activate 的問題，可以參考 [https://github.com/cmderdev/cmder/issues/1207](https://github.com/cmderdev/cmder/issues/1207)。

### 退出 virtualenv

```shell
deactivate
```



[Python]: https://www.python.org
[virtualenv]: https://virtualenv.pypa.io
[Homebrew]: https://brew.sh
[Chocolatey]: https://chocolatey.org
[pyenv]: https://github.com/yyuu/pyenv
