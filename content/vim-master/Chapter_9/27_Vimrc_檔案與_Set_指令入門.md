# Vimrc 檔案與 Set 指令入門

## 📝 課程概述

本單元是 Chapter 9 的第一堂課，目的是讓學員學會如何將個人偏好的 Vim 設定永久保存下來，不必每次啟動 Vim 時重新手動配置。課程從 `vimrc` 檔案的基本概念切入，逐步介紹 `set` 家族指令的各種用法，為建立個人專屬的 Vim 環境打下基礎。

---

## 核心觀念與實作解析

### 什麼是 vimrc 檔案？

RC 結尾的檔案是 Unix / Linux 的傳統命名慣例，**RC = Run Commands**。每次啟動 Vim 時，它會自動執行 vimrc 檔案中的所有指令，等同於你逐一在 Vim 內輸入那些命令。

Vim 有兩層級的 vimrc：

- **系統層級（System-wide）**：所有人共用，由 Vim 安裝時自動建立，預設值放在這裡。不同版本的 Vim 預設值略有差異，這也解釋了「為什麼你看到的結果和老師不同」。
- **使用者層級（User-specific）**：這裡才是你應該放入個人化設定的地方。

| 作業系統 | 檔案路徑 |
|---|---|
| Linux / macOS | `~/.vimrc`（`~` = 家目錄） |
| Windows | `~/_vimrc`（底線開頭） |

> 如果不確定自己的系統 vimrc 放在哪裡，可以在 Vim 內輸入 `:version` 並往下滑，會顯示系統層級與使用者層級的 vimrc 路徑。

---

### vimrc 的寫作語法

vimrc 裡的每一行都是一個 Ex 命令，與你在 Vim 內以 `:` 開頭輸入的命令語法相同，只是**不需要前綴冒號**。例如：

```vim
set ruler
```

在 vimrc 裡等於你在 Vim 中輸入 `:set ruler` 再按 Enter。

在 vimrc 中還可以使用雙引號 `"` 寫註解，方便日後回顧「當初為什麼這樣設定」：

```vim
" 顯示游標位置
set ruler
```

---

### set 指令家族

#### 查詢所有已變更的設定

輸入 `:set`（不加任何參數）會顯示**所有與預設值不同的選項**，這是確認目前變更狀態最快的方式。

#### 查詢特定選項的目前值

```vim
:set optionname?
```

範例：查詢 `incsearch`（增量搜尋）是否啟用：

- 啟用中 → 顯示 `incsearch`
- 關閉中 → 顯示 `noincsearch`

#### 布林選項（Boolean Options）

布林選項的特點是：只有「開」與「關」兩種狀態。

| 操作 | 指令 |
|---|---|
| 開啟 | `:set optionname` |
| 關閉 | `:set nooptionname` |
| 切換（Toggle） | `:set optionname!` |

**重要觀念**：Vim 有大量選項是布林類型（例如 `hlsearch`、`incsearch`、`ruler`）。Toggle 的 `!` 語法非常實用，可以快速反轉狀態。

#### 數值或字串選項

有些選項需要賦予一個具體的數值或字串：

```vim
:set history=1000
```

此處 `history` 控制 Vim 記住多少條命令歷史，預設是 50，老師個人偏好設為 1000。

#### 將選項還原為預設值

在選項名稱後面加上 `&` 即可還原：

```vim
:set history&
```

#### 查看選項說明文件

```vim
:help 'optionname'
```

> 使用**單引號**包住選項名稱，會直接跳到該選項的專屬文件，否則 `:help` 可能會帶你到其他相關主題。

Vim 共有約 379 種選項，可用 `:options` 指令開啟一個視窗，集中瀏覽所有選項及其說明。

---

### 老師推薦的實用設定

以下是他建議放入 vimrc 的核心選項，每個都有其明確的使用理由：

#### `set history=1000`

Vim 預設只保存最近 50 條命令歷史。如果你是重度指令使用者，大幅提高這個數值可以讓 `↑` / `↓` 鍵回溯更多過往命令，特別是在結合部分命令輸入 + `↑` 鍵做「模糊過濾」時非常有用——假設你打了 `:set h` 再按 `↑`，只會顯示以 `:set h` 開頭的歷史記錄。

#### `set ruler`

在狀態列顯示目前游標的**行號與列號**。老師在之前的課程已提過，對於隨時需要知道位置的使用者非常實用。

#### `set showcmd`

在視窗**右下角**顯示正在輸入中的不完全命令。例如輸入 `2y` 時，右下角會浮現 `2y`；當命令執行完畢後隨即消失。這能幫助你確認自己打的計數值是否正確。

#### `set wildmenu`

使用 Tab 補全時，在狀態列顯示可用選項的**視覺化選單**，而非只有純文字替換。這對 `help` 指令的參數補全特別有幫助。

#### `set scrolloff=5`

控制游標上下至少保留多少行「上下文」。預設為 0，意即 `z Enter` 會把文字推到螢幕最頂端。設為 5 之後，游標上下始終各保留 5 行畫面內容，閱讀體驗更順暢。

#### `set hlsearch` + `set incsearch`

- `hlsearch`：對搜尋結果進行**反白**。
- `incsearch`：在**輸入階段**就即時顯示目前符合的位置，而非等按 Enter 後才一次顯示。

兩者搭配使用，大幅提升搜尋效率。

#### `set ignorecase` + `set smartcase`

- `ignorecase`：忽略大小寫。
- `smartcase`：當搜尋Pattern中出現大寫字母時，**自動覆寫** `ignorecase`，恢復大小寫敏感。這是一個非常聰明的預設邏輯——小寫搜尋時不分大小寫，一旦你明確打了大寫，就尊重你的意圖。

#### `set number`（或 `set nu`）

顯示行號。適合需要快速定位、跳轉，或是需要對照文件說明的場景。

#### `set backup`

在每次存檔前自動產生**備份檔**，副檔名預設會加上 `~`（例如 `example.txt~`）。老師特別指出這個 `~` 結尾的命名是 Vim 的刻意設計，用意是避免覆蓋你真正在乎的檔案。如果不滿意預設的備份副檔名，可用 `set bex=` 自行指定。

#### `set linebreak`（`set lbr`）

Vim 預設會在**任意字元**換行，可能把一個單字從中間截斷。開啟 `lbr` 後，Vim 會在空白處才換行，讓長文字閱讀更美觀。

#### `set autoindent`（`set ai`） + `set smartindent`（`set si`）

- `ai`：新行自動继承上一行的縮排。
- `si`：在 C 語言系語法中自動做「聰明」縮排——遇到 `{` 增加縮排，遇到 `}` 自動退回。

兩者建議一起開，會讓你在寫程式或配置文件時大幅減少手動調整縮排的時間。

#### `set background=dark` 或 `set background=light`

明確告知 Vim 你的終端機背景顏色，Vim 會據此調整 syntax highlighting 的配色。如果開了 syntax highlighting 但文字顏色看起來很糟糕，首先要檢查的就是這個設定。

---

### 讓設定生效

vimrc 的內容是在 **Vim 啟動時**才讀取執行，所以在編輯完 vimrc 後，必須**關閉並重新啟動 Vim**，設定才會生效。

另外也可以用 **`:source`** 指令（縮寫 `:so`）在不退出 Vim 的情況下立即載入 vimrc 的變更：

```vim
:source ~/.vimrc
```

---

## 💡 重點摘要

- **vimrc = Run Commands 檔案**：每次啟動 Vim 都會自動執行其中的設定，不需要每次手動配置。**
- **使用者層級的 `~/.vimrc`（Linux/macOS）或 `~/_vimrc`（Windows）是你應該編輯的地方，系統層級的預設檔案不要動。**
- **布林選項可用 `set option!` 做 Toggle，數值選項用 `set option=value`，還原預設用 `set option&`。**
- **查詢選項值用 `?`，查說明文件用 `:help 'optionname'`（單引號）。**
- **所有設定在編輯完 vimrc 後需要重啟 Vim（或 `:source ~/.vimrc`）才會生效。**

---

## 🔑 關鍵字

vimrc, set, history, ruler, hlsearch, incsearch, smartcase, scrolloff, autoindent, smartindent, wildmenu, linebreak, background, source
