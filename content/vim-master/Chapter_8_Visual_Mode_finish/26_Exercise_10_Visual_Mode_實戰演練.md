# Exercise 10 — Visual Mode 實戰演練

## 📝 課程概述

本單元是 Visual Mode 的手把手實戰演練，透過一個練習檔案（`visual-practice.txt`），逐一演示各種 Visual Mode 操作在實際場景中的應用。從 Character-wise 刪除、Line-wise 合併段落、到 Block-wise 的複雜選取與編輯，本課將所有理論轉化為可操作的肌肉記憶。

---

## 核心觀念與實作解析

### 練習檔案準備

```bash
cd ~/Downloads/vim-class/
vim visual-dash-practice.txt
```

---

### 任務一：Character-wise 刪除特定文字

**目標**：刪除「Delete me, Delete Me.」這段文字

**步驟拆解**：

1. `tD`：跳到「Delete」前一格（Till 到大寫 D）
2. `v`：進入 Character-wise Visual Mode
3. `l` + `w`：用 motion 逐步擴展選取範圍，直到選到 `<` 符號之前
4. 往回退一格（`h`），**不要選到 `<`**
5. `d`：刪除選取內容

> **核心觀念**：Visual Mode 的優點在於你可以一邊擴展選取，一邊確認範圍是否正確。按 `l` 選到一半發現太長，就馬上知道要調整。

---

### 任務二：Yank 整個 Sentence 到 Register

**目標**：將「Delete me...」整句話複製到 unnamed register

**步驟拆解**：

1. 游標移到句子的任意位置
2. `0`：跳到行首（即句子開頭）
3. `yiS`：Yank 整個 Sentence（inner Sentence）

**為什麼用 `yiS` 而不用 `vyS`？**

兩者效果相同，但 `yiS` 少按一個鍵。`y` 是 yank 指令，`iS` 是 inner Sentence（不包含句尾空白），一次完成選取與複製。

**驗證 Register 內容**：

```vim
:reg "
```

---

### 任務三：Line-wise 合併多行為一行

**目標**：將散落在多行的段落文字合併成**同一行**

**步驟拆解**：

1. 游標移到段落任意位置
2. `V`：進入 Line-wise Visual Mode
3. `ip`：Text Object — 選取「inner paragraph」（整個段落）
4. `Shift + J`（即 `J`）：執行 **Join** 指令

> 合併後若想確認行數，輸入 `:set number` 或 `:set nu` 開啟行號顯示。Vim 的自動換行（line wrap）與真正的多行是不同的概念。

---

### 任務四：將多行轉為大寫 + 置中

**目標**：將 `#` 包圍的標題文字全部變成大寫，並置中

**步驟拆解**：

1. `V` → 選取目標行
2. `Shift + U`（即 `U`）：將選取範圍**全部改為大寫**
3. `gv`：重新選取（因為進入 Insert Mode 後會離開 Visual Mode）
4. `:center`：置中對齊

> **`U` 為什麼在大寫模式下不是「復原」？** 在 Vim 的預設設定中，`u` 是 Undo，`U` 是「將整行改為大寫」。這兩者在 Visual Mode 下的行為是不一樣的，要注意區分。

---

### 任務五：Block-wise 刪除多行開頭的數字與引號

**目標**：將 `"001"`, `"002"` ... 等格式轉換成 `1`, `2`, `3` ...

**步驟拆解（兩階段）**：

#### 第一階段：刪除前置零

1. `Ctrl + V`：進入 Block-wise Visual Mode
2. `l` + `l`：選取最左邊兩個字元（`0` 與 `"` 的位置）
3. `j` × 6：往下選取所有目標行
4. `d`：刪除選取的矩形區塊

#### 第二階段：刪除行內引號

1. `l`：游標右移一格（現在在 `"` 的位置）
2. `Ctrl + V` → `j` × 6：重新進入 Block-wise，選取引號欄位
3. `d`：刪除所有引號

---

### 任務六：Block-wise 在多行開頭插入 `#`（Insert）

**目標**：在連續多行的開頭統一加上 `# `

**步驟拆解**：

1. `Ctrl + V`：進入 Block-wise
2. `j`：選取目標行（欄位寬度此時不重要）
3. `Shift + I`：進入 Insert Mode（**大寫 I**，不是 `i`）
4. 輸入 `# `（井號加空格）
5. `Esc`：確認後，所有選取行的**開頭**都會補上 `# `

> **為什麼按一次 `Esc` 就能讓所有行都生效？** 因為 `Shift + I` 在 Block-wise Mode 下的行為是：將游標放到視覺區塊涵蓋的所有行的**同一欄位**，當你輸入文字時，該文字會出現在所有這些行上，類似「平行編輯」的效果。

---

### 任務七：Block-wise 在多行末尾附加內容（Append）

**目標**：在每行末尾加上 ` # EOL`（End of Line 標記）

**步驟拆解**：

1. `Ctrl + V`：進入 Block-wise
2. `j`：選取目標行
3. `$`：讓選取範圍延伸到**每行行尾**（這是關鍵！）
4. `Shift + A`：進入 Append Mode（**大寫 A**）
5. 輸入 ` # EOL`
6. `Esc`：所有行的行尾都會出現相同文字

> **這裡的 `$` 非常重要**：如果不用 `$`，你的 Append 只會作用在選取區塊的最後一格，結果會完全錯誤。`$` 讓 Block-wise 的選取範圍動態延伸到每行的真實結尾。

---

### 放棄練習並重新開始

如果練習到一半想把所有改動都還原，重新來過：

```vim
:q!
```

這會**不放存**離開編輯狀態，檔案恢復到開啟時的原始狀態。

---

## 💡 重點摘要

- **Character-wise Visual Mode 配合 `t`（Till）與 `w`（word motion），可以精準刪除不規則片段。**
- **`yiS` 是複製整句話最有效率的寫法，比 `vyS` 少按一個鍵。**
- **Line-wise Visual Mode + `ip` + `J`（Join）可以將多行段落一次合併成一行。**
- **Block-wise Insert（`Shift + I`）與 Append（`Shift + A`）的關鍵前提：前者用於行首，後者通常需要先按 `$` 延伸到行尾，再 Append。**
- **`$` 在 Block-wise Mode 中用於「動態延伸選取至每行行尾」，是進行 Column 編輯時的核心動作。**

---

## 🔑 關鍵字

Character-wise, Line-wise, Block-wise, Inner Sentence, Inner Paragraph, Join, Block-wise Insert, Block-wise Append, `$` Motion, gv
