# Vimrc 進階：配色、Map 指令與 Leader Key

## 📝 課程概述

本單元接續前一堂課，進一步介紹 vimrc 的進階功能：如何更換 Vim 的色彩主題、如何使用 `colorscheme`；以及如何透過 `map` 指令將一系列 Vim 操作綁定到單一按鍵，大幅客製化自己的編輯體驗。最後介紹 `mkvimrc` 指令，讓你不必手動撰寫 vimrc，而是從當下的工作狀態一鍵匯出設定檔。

---

## 核心觀念與實作解析

### 設定色彩主題：colorscheme

Vim 預設內建多種色彩主題。要查看可用主題的清單，輸入：

```vim
:colorscheme <Ctrl-d>
```

> `:color` 是 `colorscheme` 的縮寫，兩者皆可使用。

不滿意預設配色的原因通常是 **`background` 設定與實際終端機背景不符**——上一堂課提到的 `set background=dark` 或 `set background=light` 往往就是解法。選對 `background` 後，再挑一個順眼的 `colorscheme`，文字就會清楚可讀。

在 vimrc 中指定主題：

```vim
colorscheme slate
```

#### 安裝自訂主題

如果你想要更多選擇，可以上網搜尋「Vim colorschemes」，找到心儀的主題後：

1. 在家目錄建立 `~/.vim/colors/` 目錄（Linux/macOS）
2. 將主題檔案（副檔名 `.vim`）放進該目錄
3. 重啟 Vim，新的 `colorscheme` 就出現在候選清單中了

---

### map 指令：將一連串操作綁定到一個按鍵

`map` 是 Vim 高度客製化的核心機制。**Mapping（映射）**允許你把一組 Vim 命令綁定到一個按鍵上，按下該鍵等於自動執行一整串操作。

#### 基本語法

```vim
map <key> <keystrokes>
```

- `<key>`：你想按的鍵（例如 `F2`）
- `<keystrokes>`：按下該鍵後要自動輸入的字元序列

#### 特殊按鍵的表示法

在 mapping 中，某些按鍵需要用**中括號包住**的表示法：

| 實際按鍵 | 表示法 |
|---|---|
| `Enter`（Carriage Return） | `<CR>` |
| `Escape` | `<Esc>` |
| `Backspace` | `<BS>` |
| `Tab` | `<Tab>` |
| `Ctrl+D` | `<C-d>` |

#### 實作範例

**範例 1：用 F2 快速插入姓名地址**

```vim
map <F2> iJohn Smith<CR>123 Main Street<CR>Anytown, NY<CR><Esc>
```

這條 mapping 做的事情：
1. `i` → 進入插入模式
2. 輸入姓名與地址（每行用 `<CR>` 換行）
3. `<Esc>` → 回到一般模式

**範例 2：用 F3 快速建立 HTML 無序列表**

```vim
map <F3> i<ul><CR><li></li><CR><Esc>0i</ul><Esc>kcc<li>
```

這個 mapping 的邏輯稍微複雜，它利用一連串的移動與變更命令，精準地在游標處產生一個完整的 `<ul>/<li>` 結構。

**範例 3：用 F4 快速新增列表項目**

```vim
map <F4> o<li></li><Esc>T<lic
```

---

### 立即載入 map：不重啟 Vim

定義完 mapping 後，通常需要重啟 Vim 才生效。但如果想要**立即測試**，可以用 `:source`（縮寫 `:so`）重新讀取 vimrc：

```vim
:source ~/.vimrc
```

如此便能在不離開 Vim 的情況下，立即使用新定義的按鍵。

---

### Leader Key：避免按鍵衝突

Vim 原生的按鍵幾乎都已被賦予功能，很難找到一個完全空閒的單鍵。`Leader Key` 就是 Vim 提供的解決方案——它預設是 **反斜線 `\`，**作為你專屬的命名空間_prefix。__

**為什麼要加 Leader？**

假設你直接映射 `w` 鍵：

```vim
map w :w!<CR>
```

代價是**原本的 `w`（跳到下一個單字）功能就消失了**。用 Leader 包住就不會衝突：

```vim
" 設定 Leader 為逗號（可自訂）
let mapleader = ","

" 用 Leader + w 來存檔，不影響原本的 w 功能
map <Leader>w :w!<CR>
```

#### 自訂 Leader

```vim
let mapleader = ","
```

> **注意**：`let mapleader = ...` 必須**放在所有用到 `<Leader>` 的 mapping 之前**，否則那些 mapping 會用到預設的 Leader（`\`）。

---

### 用 `:map` 查看所有現有映射

輸入 `:map`（不加參數）可以檢視目前所有已定義的映射，這是確認自己的設定是否正確的好方法。

---

### 一鍵匯出設定：`mkvimrc`

如果你已經在 Vim 內把環境調到滿意的狀態，不需要手動寫 vimrc，可以直接用 `mkvimrc` 指令**從當前狀態匯出所有設定**：

```vim
:mkvimrc ~/.vimrc
```

- 如果檔案已存在，Vim 會發出警告。
- 要強制覆蓋，加上 `!`：`mkvimrc! ~/.vimrc`
- 也可以先指定另一個檔名，確認匯出內容正確後再覆蓋。

老師現場演示了這個指令產出的檔案，其中有一些值得認識的觀念：

#### mode 命名

| 指令 | 適用模式 |
|---|---|
| `map` | 所有模式（通用） |
| `nmap` | **Normal mode 專用** |
| `vmap` | **Visual mode 專用** |

#### `noremap` 與防範遞迴

預設的 `map` 是**遞迴的**：如果你在 mapping 中用到另一個已被映射的鍵，那個鍵會在 mapping 過程中被再次展開。`noremap` 會**關閉遞迴行為**，避免非預期的展開。

| 指令 | 說明 |
|---|---|
| `noremap` | 通用版本，禁用遞迴 |
| `nnoremap` | Normal mode 版本 |
| `vnoremap` | Visual mode 版本 |

#### Mode Line：將設定嵌入檔案本身

Mode Line 是 Vim 一項特殊功能，允許你在**檔案的最頂端或最底端**（以註解包住）嵌入 Vim 設定，適用於「這個檔案需要特別處理」的場景。

語法格式（在註解中）：
```vim
" vim:ft=vim:  ← 告訴 Vim 把這個檔案視為 vimrc 類型處理
```

`ft` 是 `filetype` 的縮寫。除了 vimrc 外，你也可以用 `:set ft=c` 強制把某個沒有副檔名的檔案當成 C 語言原始碼處理。

> **Mode Line 的語法細節**：`vim:` 後面必須先有**空白**，再接 `:set`，結尾還要再加一個 `:`。若寫錯，Mode Line 不會生效。

---

### 老師的個人 vimrc哲學

mkvimrc 匯出的檔案往往包含大量你看不懂的系統設定，這是正常的。老師的建議是：

> **vimrc 保持極簡，只放你真正用得到的設定。**

他本人的 vimrc 只有四行：

```vim
set background=dark
colorscheme slate
set wildmenu
set ruler
```

這個哲學非常值得參考——與其追求「把所有功能都開起來」，不如只保留那些真的會提升你工作效率的設定。

---

## 💡 重點摘要

- **colorscheme 指令讓你更換 Vim 的配色主題；記得先確認 `background` 設定與終端機一致，否則 highlighting 會看起來很糟糕。**
- **map 指令可以將一組命令序列綁定到一個按鍵（如 F2、F3），大幅提升常用操作的效率。**
- **使用 Leader Key（前綴按鍵，預設 `\`）來包住你的映射，可以避免覆蓋 Vim 原生按鍵的功能。**
- **用 `let mapleader = ","` 自訂 Leader，並且這個設定必須放在所有用到 `<Leader>` 的 mapping 之前。**
- **`noremap` 系列指令禁用遞迴映射，防止 mapping 內使用的按鍵被二次展開。**
- **`mkvimrc` 可以從當前狀態一鍵匯出設定，但匯出的檔案不需要全部保留，只留下自己真正用得到的設定即可。**

---

## 🔑 關鍵字

colorscheme, map, noremap, nnoremap, vnoremap, Leader, mapleader, mkvimrc, Mode Line, filetype, wildmenu, source
