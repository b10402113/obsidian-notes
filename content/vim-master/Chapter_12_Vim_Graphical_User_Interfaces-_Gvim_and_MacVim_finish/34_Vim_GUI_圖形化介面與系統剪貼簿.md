# Vim GUI 圖形化介面與系統剪貼簿

## 📝 課程概述

本單元帶領學員從純文字的命令列 Vim，過渡到具有圖形化介面的 Vim（gVim / MacVim）。我們將探討切換到 GUI 版本的動機、各作業系統的啟動方式，以及 GUI 模式下專有的功能——特別是與系統剪貼簿的無縫整合。最後介紹專屬於 GUI 版本的設定檔 `gVimRC`。

---

## 核心觀念與實作解析

### 為什麼要使用 GUI 版本的 Vim？

並不是所有人都願意（或需要）全程在終端機環境下使用 Vim。以下是三個常見的驅動因素：

1. **跨環境一致性**：身為 Linux 系統管理者，日常在伺服器上以 SSH 終端操作 Vim，但有時也會使用圖形化桌面環境。此時若能在 GUI 中繼續使用同一套編輯器，就不必在不同環境切換操作習慣。

2. **原本就不是終端機使用者**：在 macOS 或 Windows 這類以圖形介面為主的系統上，多數人平時不會開著命令列。此時將 Vim 做為獨立的圖形應用程式執行，比在終端機視窗中執行更自然。

3. **使用 GUI 專屬功能**：這是最核心的理由。圖形版本解鎖了許多終端機模式無法做到的功能——
   - 存取系統剪貼簿
   - 滑鼠滾輪滾動文件
   - 檔案總管式地開啟檔案
   - 系統原生字型與工具列

> **重點觀念：你現有所有關於 Vim 的知識（命令、巨集、文字物件等）在 GUI 版本中完全適用，兩者是同一套編輯器，只是呈現介面不同。**

---

### 各作業系統啟動 GUI Vim 的方式

| 作業系統 | 啟動方式 |
|----------|----------|
| Windows  | 開始選單 → Vim → 選擇 **GVim**；或使用搜尋功能輸入 `gvim` |
| macOS    | 應用程式資料夾找到 **MacVim**；或使用 Spotlight 搜尋 |
| Linux    | 從命令列輸入 `gvim`；或使用該發行版的套件管理員安裝 `vim-gtk`、`vim-gnome` 等套件 |

> **注意**：終端機版本的 Vim 安裝完成，並不代表 GUI 版本已同步安裝。若在 macOS 找不到 MacVim，或在 Linux 輸入 `gvim` 收到 `command not found`，就必須另外安裝 GUI 版本。可至 [vim.org](https://www.vim.org) 取得官方版本，或透過 Linux 套件管理員安裝。

---

### 選單系統：滑鼠點選其實就是在執行 Vim 命令

GUI Vim 會在視窗頂端顯示與一般圖形應用程式相同的選單（File、Edit、Global Settings…）。這裡有一個重要的認知顛覆：

> **點選選單項目，Vim 只是在「幫你打字」——背後執行的是你已經學過的 Vim 命令。**

以 Save（儲存）為例：
- 滑鼠游標移到 **File → Save** 上，Vim 會在狀態列顯示 ` :w`
- 點下去之後，等同於你在一般 Vim 中輸入 `:w` 並按下 Enter

其他選單項目也是同樣原理：
- **Edit → Undo**：執行 `u`（復原）
- **Edit → Redo**：執行 `Ctrl+r`（重做）
- **Global Settings → Toggle Pattern Highlight**：執行 `:set hlsearch!`
- **Global Settings → Context Lines (5)**：執行 `:set scrolloff=5`

這意味著：**選單只是捷徑，學會 Vim 命令永遠比依賴滑鼠更快、更有效率。** 老師在影片中特別以 Redo 選單為例說明——用 `.` 句點重複比移動滑鼠導航到選項再點擊，效率高出太多。

如果想深入了解每個選單項目背後實際執行的函數，可以查閱 `menu.vim` 檔案（`:version` 可顯示其路徑）。

---

### 工具列、捲軸與滑鼠滾輪

GUI Vim 預設不一定會顯示工具列（Toolbar），你可以透過 **Edit → Global Settings → Toggle Toolbar** 開啟它。工具列提供快速按鈕：開啟檔案、儲存、復原、重做等。

此外，GUI 模式還原生支援：
- **捲軸（Scroll Bar）**：拖曳即可上下瀏覽檔案
- **滑鼠滾輪**：直接滾動文件，無需任何額外設定

這些都是純終端機 Vim 無法做到的功能。

---

### 系統剪貼簿整合：`*` 與 `+`  registers

這是 GUI Vim 最實用的功能之一。圖形模式 Vim 有兩個額外的特殊 registers，用來與作業系統的系統剪貼簿互動：

#### `+` register（剪貼簿 register）

- 儲存系統剪貼簿的內容
- 在 macOS/Windows 上，**`+` 與 `*` register 等價**
- 操作方式：`"+y` 從 Vim 複製到系統剪貼簿，`"+p` 從系統剪貼簿貼上

#### `*` register（選取 register / PRIMARY 剪貼簿）

- **macOS / Windows**：`*` 與 `+` 完全相同（兩者指向同一個系統剪貼簿）
- **Linux / Unix（X Window）**：兩者不同——
  - `*` register：滑鼠選取反白的文字
  - `+` register：`Ctrl+C` 複製的文字（傳統剪貼簿）
  - 在 X Window 中，你可以用滑鼠中鍵（中鍵點擊）貼上 `*` register 的內容

> **驗證方式**：在 Vim 中輸入 `:registers` 檢視目前所有 registers 的狀態。

#### 實戰操作範例

假設你在某個 macOS 應用程式中用 `Command+C` 複製了一段文字，回到 MacVim：

```vim
"+p       " 使用 + register 貼上（游標停在貼上文字的開頭）
"+gp      " 同上，但游標停在貼上文字的**結尾**（gP = 往後游標）
```

`"p` 與 `"gp` 的差異：
- `"p`：游標停留在貼上文字的**最後一個字元**
- `"gp`：游標停留在貼上文字的**之後一個位置**（`g` 前綴改變游標放置邏輯）

---

### 讓 Vim 預設使用系統剪貼簿：`clipboard` 選項

每次操作都要手動指定 `"+` 很繁瑣。Vim 提供了一個設定，可以把預設的 unnamed register 置換成系統剪貼簿：

#### macOS / Windows

```vim
:set clipboard=unnamed
```

執行後，所有 **yank、delete、change、put** 這些本來寫入 unnamed register `"` 的操作，都會自動改為寫入 `+` register（系統剪貼簿）。從此：
- `yw` 不需要前綴，就能直接 `Command+V` 貼到其他應用程式
- 在其他應用程式 `Command+C` 複製後，直接 `p` 就能貼進 Vim

#### Linux / Unix

在 Linux 上，需要使用 `unnamedplus` 而非 `unnamed`：

```vim
:set clipboard=unnamedplus
```

原因是 Linux 的 X Window 系統有兩套獨立的剪貼簿機制，`unnamed` 對應的是 `*` register（滑鼠選取），而 `unnamedplus` 對應的才是 `+` register（傳統 `Ctrl+C` 剪貼簿）。

> 若你喜歡這個設定，建議將它寫入設定檔（見下一節），省去每次啟動 Vim 都要重新設定的麻煩。

---

### GUI 專屬設定檔：gVimRC

你已經知道 `~/.vimrc` 是 Vim 的全域設定檔。**gVimRC**（GVim Resource Configuration）是 GUI 版本專用的設定檔，只在圖形模式 Vim 啟動時讀取。

#### 檔案命名

| 作業系統 | 檔案名 |
|----------|--------|
| macOS / Linux / Unix | `~/.gvimrc` |
| Windows | `_gvimrc` |

#### 讀取順序

```
~/.vimrc  →  ~/.gvimrc  →  /usr/share/vim/...（系統預設）
```

也就是說，**gVimRC 在 vimrc 之後才讀取**，因此 GUI 的設定不會覆蓋掉 vimrc 中的全域設定，兩者各自獨立。

#### 查看 gVimRC 的位置

```vim
:version
```

這會列出 Vim 正在查找的所有設定檔路徑，包括 gVimRC 與前面提到的 `menu.vim` 位置。

---

### 字型設定：`gfn` 選項

GUI Vim 允許你使用作業系統的任何原生字型，透過 `gfn`（GUI FoNt）選項控制。

```vim
" 打開系統字型選擇器（輸入 * 代表開啟對話方塊）
:set gfn=*

" 直接指定字型（注意：含有空格的字型名稱需要用反斜線轉義）
:set gfn=Courier\ New\ 14

" 查詢目前使用的字型
:set gfn?
```

> **關於引號包圍值**：部分 Vim 選項可以用引號包圍帶有空白的值，但 `gfn` 不支援這種寫法。遇到有空格的字型名稱，正確做法是用 `\` 逐一轉義每個空格。

若希望每次啟動 gVim 都使用同一字型，將設定寫入 `~/.gvimrc`：

```vim
:set gfn=Courier\ New\ 14
```

---

## 💡 重點摘要

- **GUI Vim 與終端機 Vim 是同一套編輯器，所有既有知識（命令、巨集、文字物件）完全通用，差異只在呈現介面。**
- **選單項目本質上是 Vim 命令的滑鼠捷徑，學會底層命令永遠比使用滑鼠點選更有效率。**
- **`+` register 對應系統剪貼簿，`*` register 在 macOS/Windows 等於 `+`；在 Linux X Window 環境兩者不同。**
- **`set clipboard=unnamed`（macOS/Windows）或 `set clipboard=unnamedplus`（Linux）可讓 Vim 預設使用系統剪貼簿，大幅簡化跨程式複製貼上的流程。**
- **gVimRC 是 GUI 專屬設定檔，命名與讀取時機皆與 vimrc 平行，GUI 特有設定（如字型）應放在這裡。**

---

## 🔑 關鍵字

gVim, MacVim, clipboard, unnamed, unnamedplus, gVimRC, gfn, register, hlsearch, scrolloff
