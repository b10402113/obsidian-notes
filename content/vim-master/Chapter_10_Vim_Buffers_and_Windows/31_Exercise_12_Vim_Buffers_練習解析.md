# Exercise 12：Vim Buffers 練習解析

## 📝 課程概述

本單元是 Chapter 10 Buffer 概念的**實作練習 walkthrough**，帶你從開啟多個檔案開始，逐步演練所有 Buffer 相關指令：透過編號、檔名、Tab 補全切換 Buffer，用 `Ctrl+^` 快速返回上一個編輯位置，練習 `hidden` 選項的效果，並以 `:Explore` 開啟檔案總管，最終執行跨 Buffer 的全域替換。這些操作加起來，構成了一套完整的 Buffer 工作流。

## 核心觀念與實作解析

### Step 1：開啟多個 Buffer

首先，用 Shell expansion 一次開啟所有以 `Buf` 開頭的練習檔案：

```bash
cd ~/Downloads/vim-class    # 視你的下載位置而定
vim buf*
```

開啟後，用 `:ls` 查看 Buffer 列表，確認四個檔案都已載入記憶體。

### Step 2：用不同方式切換 Buffer

**用編號切換：**
```
:ls        " 先確認目標編號，例如 buf-bed.txt 是 Buffer 2
:b 2       " 切到 Buffer 2
```

**用檔名切換：**
```
:b buf-cat.txt   " 直接輸入完整檔名
```

**用 Tab 補全選擇：**
```
:b <Tab>    " 每按一次 Tab 循環一個 Buffer，游標停在目標時按 Enter
```

### Step 3：`Ctrl+^` 在兩個 Buffer 之間來回切換

`Ctrl+^` 是 Vim 中最被低估的快捷鍵之一——它讓你在**當前 Buffer** 與**上一次編輯的 Buffer（alternate buffer）** 之間瞬間切換。

用 `:ls` 觀察切換時指標的變化：
- `%a`（percent + A）移到新的 Buffer
- `#` 指標跟隨過來，標示新的 alternate buffer

不斷來回按 `Ctrl+^`，相當於在兩個最近使用的 Buffer 之間不斷「開關」。

### Step 4：依序遍歷所有 Buffer

```
:bp    " previous，往回走一個
:bn    " next，往前進一個
:bf    " first，回到第一個
:bl    " last，去最後一個
```

 `:bp` / `:bn` 支援繞回（wrap around），從第一個再 `:bp` 會跳到最後一個，反之亦然。

### Step 5：`hidden` 選項的效果實作

**沒有 `hidden` 時的行為：**
對某個 Buffer 做了修改，然後嘗試 `:b 2` 切換—— Vim 會顯示 `E37: No write since last change` 並**拒絕切換**。

這時 `:b 2!`（加 `!`）可以強制切換，Buffer 會帶著 `H` + `+` 兩個指標，意思是「它現在是 hidden 且有未儲存的修改」。

**開啟 `hidden` 之後：**
```
:set hidden
```
再做任何修改，然後正常 `:b 1` 切換——**不會再出現錯誤訊息**。`:ls` 檢視，會看到剛才編輯過的 Buffer 同時帶有 `H` 和 `+` 指標，代表它已進入 hidden 狀態。

> 這裡的 **Why** 很重要：`:b 2!` 之所以要加 `!`，是因為 Vim 預設不允許你在有未儲存修改的 Buffer 尚未寫入的情況下離開它——這是 Vim 的一種保護機制。`hidden` 改變了遊戲規則：Buffer 離開視窗時不會被卸載，所以 Vim 允許你自由切換，待會再回來儲存。

### Step 6：用 `:Explore` 開啟新檔案

`:Explore`（`:E`）是一個特殊的 Buffer，會以檔案總管的形式呈現當前目錄。在總管中：

- 用 `j`/`k` 或搜尋命令（`/pattern`）移動游標
- 移到目標檔案上方，按 `Enter` 開啟該檔案進 Buffer
- 若決定不打開任何檔案，用 `:bd` 刪除總管 Buffer 即可返回原 Buffer

### Step 7：跨 Buffer 的全域替換

使用 `:bufdo` 對所有 Buffer 執行同一命令：
```
:bufdo %s/#/@/g
```

若 `hidden` 未開啟，遇到第一個未儲存的 Buffer 就會失敗並停止。成功執行後，所有 Buffer 會出現 `+` 指標（已修改但未儲存）。最後用 `:wa` 一次儲存全部。

若要**放棄所有修改**重來練習，使用：
```
:qa!
```

## 💡 重點摘要

- **`:ls` 是你的雷達**：每次切換前後都查看一下，就能清楚掌握所有 Buffer 的狀態。
- **`Ctrl+^` 是最被低估的快捷鍵**，學會後可以完全不用記住 alternate buffer 的編號。
- **`hidden` + `:wa` 是最乾淨的多檔編輯流程**：隨意切換，最後一次儲存全部。
- **`:bd` 是解除 Buffet 列表雜訊的工具**，用完的 Buffer 果斷刪除。
- `:bufdo` + `|` 串接 `:w` 是避免「中途失敗」的竅門。

## 🔑 關鍵字

:ls, :buffer, :bufdo, :Explore, :bdelete, hidden, alternate buffer, :wa, :bfirst, :blast
