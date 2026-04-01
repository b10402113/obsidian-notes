# Vim Macros：自動化重複性編輯任務

## 📝 課程概述

本單元介紹 Vim 中另一個强大的自動化工具：**Macros（巨集）**。不同於 `.`（dot command）只能重複上一個單一命令，Macros 可以錄製並回放一連串跨多模式的複雜操作，並儲存在寄存器（Registers）中反覆使用。我們將學習如何錄製、播放、編輯以及持久化儲存 Macros。

---

## 核心觀念與實作解析

### 什麼是 Macro？為什麼需要它？

在日常編輯中，我們常遇到需要對**多行或複雜格式**執行相同的一系列操作：

- 將一組 CSV 資料中的欄位重新排列
- 標準化一批伺服器的設定格式
- 在每行程式碼前加上統一的標記

`.` 只能重複**一個**上一個動作，而 Macros 能將**任意多個步驟**錄製成組，之後像調用函式一樣反覆呼叫。

### Macro 的核心概念：儲存在 Registers 中的按鍵序列

```text
# 錄製 Macro 到 a register
q a          # q = 開始錄製，a = 目標寄存器
[一系列操作]
q            # 結束錄製
```

> 💡 Macros 本質上就是**按鍵序列（keystrokes）**，只不過被儲存在寄存器裡。

### 錄製與播放實戰

```text
# 目標：在每行開頭插入 "note: "
q a          # 開始錄製到 a register
I            # 進入插入模式並移到行首
note:        # 輸入文字
<Esc>        # 返回 Normal mode
q            # 停止錄製

# 播放 Macro
@a           # 播放 a register 中的 Macro
@@           # 快速重播上一個執行的 Macro
```

### 狀態列的錄製提示

開始錄製後，Vim 狀態列會顯示 `recording @a`，讓你隨時知道目前正在錄製中。

### 讓 Macro 自動移至下一行

一開始的 Macro 執行完後，游標會停留在當前行。如果要一次處理多行，手動跳行很麻煩。**在 Macro 最後加入 `j`（下一行）**就能實現連續自動處理：

```text
q b          # 錄製到 b register
0            # 移到行首（標準化游標位置）
Itip: <Esc> # 插入 "tip: " 並返回 Normal mode
j            # 移至下一行 ← 關鍵！
q            # 停止錄製

# 現在可以這樣使用：
5 @b         # 對下麵 5 行執行 Macro
```

### Macro 錄製的最佳實踐

#### 1. 標準化游標起始位置（`0`）

**在 Macro 開頭使用 `0`**：確保每次執行時游標都從行首開始，消除「上次執行後游標位置不確定」的問題。

```text
q c
0            # 移到行首 — 每次執行都從這裡開始
[操作...]
j
q
```

#### 2. 在 Macro 結尾加入 `j`

如上所示，讓游標自動移到下一行，方便串聯執行。

#### 3. 對寄存器內容進行追加（Append）

如果想給已錄好的 Macro 追加步驟，使用**大寫寄存器字母**：

```text
# 假設 c register 的 Macro 忘了加 j
q C          # 大寫 C = 追加到 c register
j            # 新增 "移至下一行"
q            # 完成追加
```

> 💡 這和 `ya` / `yA` 的追加行為一致——大寫寄存器字母代表「追加」。

### 透過 `:normal` 命令對範圍執行 Macro

當你需要在大量行上執行 Macro 時，可以使用範圍語法：

```text
:27,35 normal @d
# 等於對第 27 到 35 行，分別執行一次 @d Macro
```

`normal` 命令的意義：**對指定範圍內的每一行，模擬在該行執行 `:normal` 後的命令**。

```text
# 對整個檔案執行 Macro
:1,$ normal @d    # 第一行到最後一行
# 簡寫：
:% normal @d     # 全部行
```

### 用計數（Count）控制 Macro 重複次數

```text
5 @b              # 執行 5 次 Macro b
```

### 編輯（Edit）已錄好的 Macro

Macro 存在寄存器裡，你可以像編輯普通文字一樣修改它：

```text
# 假設要修改 a register 的 Macro
"ap           # 把 a register 的內容貼到文件中
[修改內容]     # 編輯這些按鍵序列
0"ay$         # 從頭 yank 回 a register
```

### 持久化儲存 Macro：寫入 vimrc

Vim 關閉後，寄存器的內容預設會因為 `viminfo` 檔案而被保留，但若覆寫了同名寄存器，Macro 就會丢失。

**最可靠的方式是將 Macro 寫入 `.vimrc`（或 `_vimrc`）檔案：**

```text
let @d = 'Iprefix: <Esc>'
# 語法：let @[register] = '[按鍵序列字串]'
```

#### 手動輸入特殊字元（如 `<Esc>`）

```text
# 如果要在 vimrc 中輸入 Escape 字元
# 不能直接按 Esc，否則會退出當前模式
# 正確做法：Ctrl+v 再按 Esc
# Ctrl+v 在 Vim 中表示「插入原始字元」，讓你能輸入控制字元
```

> ⚠️ 如果 Macro 內容包含單引號 `'`，要用雙引號包住整個 `let` 語句：`let @d = "Itext: '<Esc>"`

### 儲存階段的最佳做法

```text
# 推薦：先錄製好 Macro，再貼進 vimrc
# 1. 先在 Vim 中正常錄製 Macro
qa
...
q

# 2. 確認內容無誤
:reg a

# 3. 開啟 vimrc 並貼上
:edit ~/.vimrc
O                       # 在上方插入新行
let @a = '              # 開始輸入
[貼上寄存器內容]
'                        # 結束
```

---

## 💡 重點摘要

- **Macros 是儲存在寄存器中的按鍵序列，支援 Normal / Insert / Visual / Command 所有模式，幾乎沒有功能限制。**
- **錄製 Macro 時以 `0` 開頭（標準化行首）、以 `j` 结尾（自動下移）是兩個最重要的最佳實踐。**
- **`:[range] normal @[register]` 讓你能優雅地對大量行應用 Macro，比手動數行數更可靠。**
- **透過 `let @[register] = '...'` 寫入 `.vimrc` 是持久化 Macro 的標準做法，能跨 session 保留。**
- **大寫寄存器字母（如 `qC`）用於追加內容，小寫（如 `qc`）用於覆寫。**

---

## 🔑 關鍵字

Register, Recording, Playback, Append, vimrc, normal, @, Dot Command
