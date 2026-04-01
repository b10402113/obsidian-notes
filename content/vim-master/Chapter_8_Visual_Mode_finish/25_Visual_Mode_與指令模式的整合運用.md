# Visual Mode 與指令模式的整合運用

## 📝 課程概述

本單元進一步延伸 Visual Mode 的應用範疇，探討它與 Vim 指令模式（Command Mode）的協作。我們將學習如何用 Visual Mode 選取的範圍作為 `:command` 的作用範圍，以及 `Shiftwidth`、`Tabstop`、`Expandtab` 等縮排設定背後的邏輯——這些都是提升程式碼整潔度的關鍵設定。

---

## 核心觀念與實作解析

### Visual Mode 結合 Text Object 處理縮排

假設你打開一個 Shell Script 函式，發現整個程式碼區塊都沒有縮排，閱讀起來非常吃力。這個時候 Visual Mode 配合 Text Object 就能優雅地一次搞定：

1. 將游標移到**任意一對大括號內**的位置
2. `V` 進入 Line-wise Visual Mode
3. `i}`（inner closing curly brace）：選取**大括號內所有行**
4. `>`：整個 block 向右縮排

> 你也可以用 `>i{` 來達成相同效果——`i{` 與 `i}` 的差異只在於選取時是否包含大括號本身。

#### 不進入 Visual Mode 的替代寫法

事實上，純粹用 `>i}` 也能達到同樣效果，完全不需要進入 Visual Mode。但 Visual Mode 的價值在於**先看再動**——你能確認選取的範圍確實正確，再按 `>` 下手。

#### 快速恢復上一次選取：`gv`

`gv` 會自動重新選取**上一次 Visual Mode 選取的範圍與模式**，非常適合在執行 `>` 縮排後，想快速回到同樣範圍做進一步調整的場景。

---

### 理解縮排設定：Shiftwidth、Tabstop、Expandtab

這三個設定彼此連動，理解它們之間的關係是正確配置 Vim 的第一步。

#### `shiftwidth`（縮排寬度）

每次執行 `>`、`<` 這類 shift 指令時，文字移動的幅度。預設值是 **8**。

```
:set shiftwidth?
```

#### `tabstop`（Tab 寬度）

一個 Tab 字元在畫面上呈現的寬度。預設同樣是 **8**。

```
:set tabstop?
```

#### `expandtab`（是否展開 Tab）

- 關閉（預設）：`>` 指令會插入**真正的 Tab 字元**
- 開啟：所有 Tab 都会被**空格**取代

```
:set expandtab   " 開啟
:set noexpandtab " 關閉
```

#### 三者連動的實際影響

當 `shiftwidth=8`、`tabstop=8`、`expandtab` 關閉時，每次 `>` 會插入一個 Tab。如果你之後再按一次 `>`，文字會移到下一個 8 的倍數位置，Vim 會再插入一個 Tab——於是你的程式碼開始出現**Tab 與空格混雜**的亂象。

> **建議**：開發時統一開啟 `expandtab`，並將 `shiftwidth` 設為 4（多數程式碼風格的慣例）。這樣每次 shift 都是空格，無論何時縮排都一致。

#### 如何看到隱藏的 Tab 字元？

開啟 `list` 模式：

```
:set list
```

- Tab 字元顯示為 `^I`（Caret + 大寫 I）
- 行尾多餘的空白顯示為 `$`

```
:set list!  " 關閉（驚嘆號錨定切換）
```

---

### 將 Visual 範圍餽入 Command Mode

這是本單元最實用的高級技巧：**用 Visual Mode 選取的範圍，作為 Ex 命令的作用範圍**。

#### 機制說明

1. 在 Normal Mode 下，用 `V` 選取目標行
2. 按 `:` 進入 Command Mode
3. 你會發現命令列自動出現：**`<,'>`**
   - `'<` = 選取範圍的**起始行**
   - `'>` = 選取範圍的**結束行**
4. 繼續輸入你要執行的命令，按 `Enter`

#### 實際範例：對特定行進行 Substitution

想把「United States of America」改成「USA」，但**只改那些包含 State Capital 的行**：

1. `V` 選取目標行範圍（如用 `/G` 搜尋跳到最後一行）
2. `:` → 命令列自動出現 `<,'>`
3. `:'<,'>s/United States of America/USA/g`
4. `Enter`

> 重要提醒：Command Mode 的範圍是**以「行」為單位**，即使是 Character-wise 選取，Command Mode 的效果仍然會影響**整行**。這個行為在 Vim 文件中有特別說明，未來版本可能會有變化。

---

### 對齊文字的三個命令：center、left、right

Visual Mode 選取的範圍，除了能配合 `s` 進行替換，還能用於三個強大的對齊命令。

#### `:'<,'>center [width]`

將選取範圍的每一行**置中**對齊。預設寬度為 **80 個字元**（當 `textwidth` 未設定時）。

```vim
:'<,'>center    " 預設 80 欄
:'<,'>center 40 " 指定 40 欄
```

#### `:'<,'>left [indent]`

將選取範圍**靠左對齊**，`indent` 參數可以設定從左邊留多少空格。

```vim
:'<,'>left   " 靠左（不留縮排）
:'<,'>left 5 " 靠左，但左側留 5 格空白
```

#### `:'<,'>right [column]`

將選取範圍**靠右對齊**，預設以第 80 欄為右界。

```vim
:'<,'>right   " 預設靠右至 80 欄
:'<,'>right 60 " 靠右至第 60 欄
```

#### 實用技巧：同時置中並加上 `#` 標頭

在程式碼中加入 comment header 是很常見的需求。操作步驟：

1. `:'<,'>center`：先將文字置中
2. `0`：游標移到行首
3. `Ctrl + V` → `j` → 選取同樣列數
4. `R#`：將選取範圍**替換**成 `#` 字元

這樣文字會保持置中，而每行開頭會自動加上 `#`，非常適合用來製作醒目的程式碼標頭。

---

### gv：重取上次選取範圍

無論你執行了什麼操作，只要想回到上一次 Visual Mode 選取的範圍，輸入 `gv` 即可。這在反覆對同一區塊做調整時特別有用。

```vim
gv  " 重新選取上一個 Visual 範圍
```

---

## 💡 重點摘要

- **Visual Mode 結合 Text Object（如 `i}`）可以一次對多行區塊進行縮排，且先選取再操作能確保範圍正確。**
- **`shiftwidth`、`tabstop`、`expandtab` 三個設定連動：建議平時開啟 `expandtab` 並設 `shiftwidth=4`，避免 Tab/空格混雜。**
- **在 Visual Mode 中按 `:`，Vim 會自動將選取範圍轉為 `'<,'>` 格式，讓你對特定行執行 Ex 命令。**
- **Command Mode 中的視覺範圍（`'<,'>`）**始终以「行」為單位**，即使你是用 Character-wise 選取的。**
- **center、left、right 三個對齊命令可以與 Visual Mode 選取範圍結合，一次對多行進行格式化。**

---

## 🔑 關鍵字

Shiftwidth, Tabstop, Expandtab, Visual Range, `:'<,'>`, center, left, right, Text Object, gv
