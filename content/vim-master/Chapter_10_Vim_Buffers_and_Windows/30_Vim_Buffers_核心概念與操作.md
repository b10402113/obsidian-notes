# Vim Buffers 核心概念與操作

## 📝 課程概述

Buffer 是 Vim 管理檔案的底層機制，理解它等於掌握了 Vim「多檔編輯」的精髓。本單元從「什麼是 Buffer」出發，逐步帶你學會一次開啟多個檔案、在多個 Buffer 之間自由切換，並探討 `hidden` 選項如何讓編輯流程更加順暢。這些都是进阶使用 Vim 的基礎功。

## 核心觀念與實作解析

### 什麼是 Buffer？

每次你用 Vim 開啟一個檔案，Vim 並不是直接編輯磁碟上的檔案，而是先**把檔案內容讀進記憶體**，這份存在記憶體中的資料就叫做 Buffer。你可以把它想成「檔案的工作副本」—— 所有編輯都在 Buffer 裡進行，只有在你明確下達 `:w`（write）指令時，變更才會寫回磁碟上的原檔案。

> 為什麼要這樣設計？因為這讓 Vim 具備「後悔藥」的能力—— 你可以任意修改、實驗，若不滿意直接捨棄 Buffer，原始檔案絲毫不受影響。

Buffer 不一定與檔案綁定。啟動 Vim 時不帶任何檔案參數，Vim 會建立一個**空白的未命名 Buffer**（unnamed buffer），你可以在裡面任意打字，要儲存時再指定檔案路徑。

### 一次開啟多個檔案

有幾種方式可以同時開啟多個檔案：

**直接以命令列參數傳入：**
```bash
vim buf-a.txt buf-bed.txt buf-cat.txt
```
Vim 會讀入所有檔案，但畫面只顯示第一個（Buffer 1）。其餘檔案已在記憶體中，隨時可以切換過去。

**在 Vim 內部用 `:edit`（簡寫 `:e`）新增檔案：**
```
:e buf-cat.txt
:e buf-dad.txt
```
每次執行都會把新檔案加入 Buffer 列表。

**使用 Shell 的萬用字元 expansion：**
```bash
vim buf*
```
Shell 會先將 `buf*` 展開成所有匹配的檔案，再傳給 Vim。這是 Shell 本身的功能，與 Vim 無關—— Vim 只是收到了展開後的檔案列表。

### 檢視 Buffer 列表

```
:buffers   " 完整命令
:ls         " 簡寫，Unix 使用者最熟悉的形式
:files      " 另一個等價的簡寫
```

輸出範例：
```
  1 %a   "buf-a.txt"              line 1
  2  h   "buf-bed.txt"            line 1
  3 #    "buf-cat.txt"            line 1
  4  h+  "buf-dad.txt"            line 1
```

各欄位的意義：
- **最左側數字**：該 Buffer 的**唯一編號**，在同一個 Vim Session 中不會改變（Buffer 1 永遠是 Buffer 1）。
- **%a**：出現在當前視窗的 Buffer，`a` 表示 active（正在顯示中）。
- **#**：alternate buffer，即你上一次編輯的 Buffer。
- **`+`**：Buffer 有未儲存的修改。
- **`h`**：hidden buffer，已載入記憶體但目前沒有顯示在任何視窗中。
- **無任何標記**：inactive buffer，尚未載入記憶體。

### 切換 Buffer 的各種方法

#### 用 `:buffer`（簡寫 `:b`）指定目標
```
:b 2              " 用編號切換
:b buf-cat.txt    " 用檔名切換
```
支援 **Tab 自動補全**，也可以用 `:b ` + `Tab` 循環選擇，或 `:b Ctrl+d` 列出所有候選。

#### 依序遍歷
```
:bn    " next，下一個 Buffer
:bp    " previous，上一個 Buffer
:bfirst " 第一個 Buffer（簡寫 :bf）
:blast  " 最後一個 Buffer（簡寫 :bl）
```
這些命令都會**繞回到開頭**（wrap around）—— 從最後一個再 `:bn` 會回到第一個，同樣 `:bp` 從第一個會跳到最後一個。

#### 快速切回上一個編輯的 Buffer
```
Ctrl+^   " 英文鍵盤，control + shift + 6
```
等同於 `:b#`，相當於「開關」兩個 Buffer。熟練使用這個快捷鍵可以大幅減少 `:bn` / `:bp` 的輸入。

### 修改未儲存的處理邏輯

當 Buffer 有 `+` 標記（即有未儲存修改）時，嘗試切換到別的 Buffer 會得到 `E37: No write since last change` 錯誤。

有幾種應對方式：

1. **先儲存再切換**：`:w` 儲存，然後正常切換。
2. **強致切換**（不建議用於重要檔案）：在命令後加 `!`，如 `:bn!`。
3. **最推薦的方式：開啟 `hidden` 選項**（見下一節）。

### `hidden` 選項：讓 Buffer 留在記憶體中

預設情況下，當你離開一個 Buffer，Vim 會將它**從記憶體中卸載**（unload）。若已開啟 `hidden`，Buffer 離開視窗後會轉為 hidden 狀態，仍然留在記憶體裡，隨時可以切回來。

```
:set hidden
```

這個設定開啟之後，切換 Buffer 時再也不會看到 `E37` 錯誤—— Vim 會默默把未儲存的 Buffer 藏起來，等你決定何時儲存。

**使用 `hidden` 的風險管控**：若你忘記儲存任何修改就直接 `:q` 或 `:q!`，Vim 會**攔截並拒絕退出**，然後自動跳回第一個有未儲存修改的 Buffer**，讓你決定要 `:w` 還是 `:q!` 放棄。這點很重要——千萬別以為 `hidden` 等於「不用儲存就沒事」。

兩個實用的批量命令：
```
:wa   " write all — 儲存所有 Buffer
:qa   " quit all — 嘗試關閉所有 Buffer（有未儲存會失敗）
:qa!  " 強制關閉所有 Buffer（不放過任何一個）
```

### 進階 Buffer 管理

**`:badd`**：將檔案加入 Buffer 列表，但**不切換過去**。
```
:bad modes.txt
```
適合先預載入一些參考檔案，之後隨時可用 `:b` 快速切換。

**`:bdelete`**（簡寫 `:bd`）：從 Buffer 列表中移除並卸載記憶體。
```
:bd 3        " 刪除編號 3 的 Buffer
:1,3 bd     " 刪除範圍內的所有 Buffer
:% bd        " 刪除所有 Buffer（相當於重置工作區）
```

**`:bufdo`**：對**每一個 Buffer** 執行同一條命令。
```
:bufdo %s/#/@/g   " 在所有 Buffer 中全域替換
```

但 `:bufdo` 也會遇到 `E37` 錯誤——有兩種解法：
1. **串接 `:w`**：用 `|` 將替換與儲存綁在一起執行：
   ```
   :bufdo %s/#/@/g | w
   ```
2. **開啟 `hidden`**：讓每個 Buffer 切換時自動進入 hidden 狀態，最後 `:wa` 一次儲存全部。

### 內建檔案總管：`:Explore`

```
:Explore    " 簡寫 :E
```
開啟一個以當前目錄為根的檔案總管 Buffer。你可以用所有 Vim 的導航命令（`j`/`k`、搜尋等）移動游標，移到目標檔案上方按 `Enter` 即可開啟。若決定不打開，直接 `:bd` 刪除這個總管 Buffer 即可。

## 💡 重點摘要

- **Buffer 是檔案的記憶體副本**，編輯都在記憶體中進行，只有 `:w` 才寫回磁碟。
- `:ls` 是檢視 Buffer 列表最快的命令，**% = 當前**，**# = 上一個（alternate）**。
- `Ctrl+^` 是切回 alternate buffer 的最快方式，熟練後極為高效。
- **`set hidden`** 大幅提升多檔編輯體驗，但退出前務必確認所有修改已儲存或確認要放棄。
- `:bufdo` + `hidden` 組合是批次處理多個檔案替換的最佳實踐。

## 🔑 關鍵字

Buffer, :buffers, :ls, :buffer, :badd, :bdelete, :bufdo, :Explore, hidden, alternate buffer
