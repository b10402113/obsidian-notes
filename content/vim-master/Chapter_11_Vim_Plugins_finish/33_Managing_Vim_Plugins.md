# Vim 插件管理

## 📝 課程概述

本單元介紹 Vim 插件（Plugin）的管理方式，從 Vim 8 內建的插件系統說起，帶學員建立對 `pack path`、`start` 目錄與 `opt` 目錄的核心概念。課程透過四個實用插件的安裝與使用演示——NERD Tree、CtrlP、Tabular、easy-motion——讓學員親身體驗插件如何擴展 Vim 的功能，最終能獨立完成插件的安裝、載入與移除。

---

## 核心觀念與實作解析

### 插件管理的演進：從三方工具到 Vim 8 內建機制

在 Vim 8 之前，管理插件必須依靠第三方插件管理器，例如 Vim Plug、Vundle、NeoBundle 或 Pathogen。這些工具各有優點，但都不是 Vim 本身提供的功能。

**從 Vim 8 開始，Vim 內建了官方的插件管理機制。** 老師的建議是：如果你是插件的新手，直接使用 Vim 8 內建的系統即可；若你已習慣使用第三方工具，升級到 Vim 8 後也完全可以繼續沿用。

> 本課程的焦點在於 Vim 8 內建的插件管理機制，不依賴任何第三方工具。

---

### 尋找插件的資源

插件的取得管道主要有兩個：

1. **搜尋引擎（Google 等）** — 直接以功能關鍵字搜尋
2. **Vim Awesome（vimawesome.com）** — 專門收錄 Vim 插件的聚合網站，可依類別瀏覽數千個插件

目前絕大多數插件都托管在 GitHub 上，因此 `git clone` 是最常見的安裝方式。

---

### 前置需求：系統必須安裝 Git

安裝插件的本質，就是將插件的 Git  repository 複製到本地。因此你的系統必須先裝好 Git：

| 作業系統 | 安裝指令 |
|---|---|
| Red Hat 系（Fedora、CentOS） | `sudo dnf install -y git` |
| Debian 系（Debian、Ubuntu） | `sudo apt install -y git` |
| Windows | 至 Git 官網下載 Git for Windows |

若執行 `git clone` 時出現 `command not found`，就代表需要先安裝 Git。

---

### 理解 Vim 的目錄結構：`pack` 與 `path`

這是插件管理最核心的概念。我們一層層來拆解：

#### 1. 查看 `packpath`

在 Vim 中執行以下命令，可以查看 Vim 搜尋插件的路徑：

```
:echo &packpath
```

在 Unix/Linux/macOS 系統上，預設的 `packpath` 為 `~/.vim`（即家目錄下的 `.vim` 資料夾）。在 Windows 上則是 `~/vimfiles`（家目錄下的 `vimfiles` 資料夾）。

#### 2. 目錄層級結構

`packpath` 是最上層的根目錄，在其內部你需要依序建立以下結構：

```
~/.vim/                    ← packpath（根目錄）
 └── pack/                 ← 固定名稱，不可改
      └── [你自己命名的套件資料夾]/    ← Package 目錄（名稱自訂，例如 plugins、vendor、git-stuff）
           ├── start/      ← 固定名稱，放進此目錄的插件會自動載入
           └── opt/        ← 固定名稱，放進此目錄的插件需手動載入
                └── [插件目錄]/
```

> **為什麼要有 Package 的概念？** Package 就像一個分類收納盒。你可以將同一類型的插件放在同一個 Package 下——例如所有 Git 相關插件放進一個名為 `git-stuff` 的 Package，所有 Python 相關插件放進 `python-plugins`。當然，你也可以把所有插件都放在一個 Package 中（如 `plugins`），兩種做法都完全有效。

---

### `start` 與 `opt`：自動載入 vs 手動載入

這是理解 Vim 插件系統的關鍵：

- **`start` 目錄**：放進此目錄的插件，**Vim 啟動時會自動載入**，適合每天都會用到的核心插件。
- **`opt` 目錄**：放進此目錄的插件，**預設不會載入**，適合那些「偶爾才用一次」的插件，節省 Vim 的啟動時間。

#### 手動載入 `opt` 目錄中的插件

當你想使用一個放在 `opt` 目錄中的插件時，必須先用 `:packadd` 命令告知 Vim 載入它：

```
:packadd <插件目錄名稱>
```

> `:packadd` 的縮寫就是 `:pa`。兩個命令完全等價：`packadd tabular` 等同於 `pa tabular`。

---

### 實作演示一：NERD Tree — 自動載入插件

**NERD Tree** 是一個檔案系統瀏覽器插件，可讓你以視覺化方式瀏覽目錄結構並快速開啟檔案。

安裝步驟：

```bash
# 建立目錄結構
mkdir -p ~/.vim/pack/plugins/start
cd ~/.vim/pack/plugins/start

# 透過 git clone 安裝插件
git clone https://github.com/preservim/nerdtree.git
```

在 Vim 中開啟 NERD Tree：

```
:NERDTree
```

不帶參數執行，會開啟目前工作目錄的視圖。也可以指定路徑，例如 `:NERDTree ~/var`。

常用操作：
- `?` — 開啟/關閉說明面板
- `o` — 開啟檔案或展開目錄
- `O` — 遞迴展開目錄（含所有子目錄）
- `X` — 關閉所有展開的子目錄
- `/` — 搜尋檔案名稱

---

### 實作演示二：CtrlP — 模糊檔案搜尋

**CtrlP** 是一個模糊搜尋插件，適用於專案中有大量檔案和子目錄的場景，可以快速定位任意檔案。

安裝方式同樣是 clone 到 `start` 目錄：

```bash
git clone https://github.com/ctrlpvim/ctrlp.vim.git
```

啟動：直接按 `Ctrl+P`，畫面底部會彈出一個搜尋視窗。開始輸入檔名或路徑中的字元，CtrlP 會即時過濾候選結果。

- `↑` / `↓` 鍵 — 在結果間移動
- `Enter` — 以正常方式開啟檔案
- `Ctrl+X` — 在**水平分割**視窗中開啟
- `Ctrl+V` — 在**垂直分割**視窗中開啟

> CtrlP 的搜尋範圍是從當前工作目錄往下遞迴查找所有子目錄，因此最適合在專案根目錄使用。

---

### 實作演示三：Tabular — 手動載入插件

**Tabular** 是一個文字對齊插件，可將含有特定分隔符號的文字對齊成整齊的欄位格式。這個演示的重點在於展示 `opt` 目錄與 `:packadd` 的使用方式。

安裝到 `opt` 目錄：

```bash
cd ~/.vim/pack/plugins/opt
git clone https://github.com/godlygeek/tabular.git
```

此時直接執行 `:Tab` 會得到 `E492: Not an editor command`——因為插件尚未載入。正確做法是先執行：

```
:packadd tabular
:Tab
```

Tabular 的基本用法：

```
:Tab /
```

這會根據 `/`（斜線）將文字對齊。若要進一步格式化（如靠右對齊、欄位間保留至少三個空格）：

```
:Tab / r3
```

> `:packadd` 的 `:pa` 縮寫讓這一步驟更簡短：`packadd tabular` → `pa tabular`。

---

### 實作演示四：easy-motion — 快速跳轉

**easy-motion** 簡化了 Vim 中的跳轉操作，傳統上需要多次按鍵才能移動到遠處，而 easy-motion 透過在畫面上標示可跳轉的目標位置，讓你只按一兩個鍵就能抵達。

預設的 Leader key 為 `\`（反斜線）。常用操作：

| 操作 | 按鍵 | 說明 |
|---|---|---|
| 向下跳轉 | `\\j` | 在每行開頭顯示跳轉標記，輸入目標字母即可 |
| 向上跳轉 | `\\k` | 同上，方向向上 |
| 跳到單字開頭 | `\\w` | 搜尋並跳到某個單字開頭 |
| 單字擊鍵搜尋 | `\\s` | 提示輸入一個字元，然後跳轉到下一個含該字元的位置 |

當候選目標太多時（例如 `\\w` 在有許多單字的行），easy-motion 會進入「二階段擊鍵」模式——先顯示所有以相同字母開頭的候選，輸入第二個字母完成跳轉。

---

### 多 Package 的組織方式

你可以在 `pack` 目錄下建立多個 Package，彼此獨立存放不同用途的插件。例如：

```
~/.vim/pack/
 ├── plugins/start/   ← 通用插件（NERD Tree、CtrlP 等）
 └── git-stuff/start/ ← Git 相關插件（如 vim-fugitive）
```

新增一個 Package 時，只要重複建立目錄結構並 clone 插件即可。**所有 Package 中的 `start` 目錄都會在 Vim 啟動時被自動掃描**，不同 Package 不會互相影響。

---

### 移除插件

Vim 插件的移除方式非常直覺——**只需要刪除插件的目錄即可**：

```bash
rm -rf ~/.vim/pack/plugins/start/vim-fugitive
```

不需要任何命令或設定檔。刪除後重開 Vim，該插件就不會再出現。

---

## 💡 重點摘要

- **Vim 8 內建了官方插件管理機制，新使用者應直接使用內建方式，不再需要額外安裝第三方插件管理器。**
- **`packpath` 是 Vim 搜尋插件的根目錄；`pack/` 是其下固定名稱的子目錄；`start/` 與 `opt/` 決定插件是自動還是手動載入。**
- **Plugin 的本質是 Git repository，安裝過程就是 `git clone` 插件的 GitHub 仓库到正確的目錄位置。**
- **`opt` 目錄中的插件預設不載入，節省 Vim 啟動時間，需要時用 `:packadd <插件名>` 手工載入。**
- **移除插件無需任何命令，直接刪除插件目錄即可。**

---

## 🔑 關鍵字

packpath, start, opt, :packadd, Package, NERD Tree, CtrlP, Tabular, easy-motion, vim-fugitive
