# Visual Mode 三種模式詳解

## 📝 課程概述

本單元是 Vim Visual Mode 的核心教學，帶領學員認識三種截然不同的選取模式——Character-wise、Line-wise 與 Block-wise，並實際演練各種常用操作。我們將發現，Visual Mode 不只是滑鼠選取的鍵盤替代品，在處理**多行欄位編輯**時，它的威力遠遠超過多數圖形化編輯器。

---

## 核心觀念與實作解析

### 為什麼需要 Visual Mode？

在圖形化編輯器中，我們用滑鼠拖曳就能選取文字；但在純鍵盤的 Vim 環境中，Visual Mode 提供了完全相同的選取能力，而且**更強**。其中最關鍵的一點是：Visual Mode 讓你在正式下達指令前，能先**預覽**這次改動會影響到哪些範圍——這在編輯複雜文件時，是避免失手的絕佳保障。

---

### 三種 Visual Mode 一覽

| 模式 | 進入指令 | 選取邏輯 | 常用時機 |
| ---- | ------- | -------- | -------- |
| Character-wise | `v`（小寫） | 以字元為單位，可選取不規則範圍 | 選取片段文字、關鍵詞 |
| Line-wise | `V`（大寫） | 最小單位是**一整行** | 選取多行、段落 |
| Block-wise | `Ctrl + V` | 以**垂直矩形區塊**選取 | 同時編輯多行的同一欄位 |

> **Block-wise Visual Mode 是 Vim 最具代表性的特色功能之一**——多數圖形化編輯器根本做不到這種「豎向選取」，而這在面對格式化資料時非常實用。

---

### Character-wise Visual Mode（`v`）

#### 基本操作流程

1. 將游標移到選取範圍的**起點**
2. 輸入 `v` 進入 Character-wise Visual Mode
3. 用 **motion**（如 `l`、`w`、`/` 等）擴展選取範圍——選取的範圍會即時 highlight
4. 下達操作指令，如 `d`（刪除）、`y`（複製）等

#### 選取範圍的兩端切換：`o`

當你向某個方向擴展選取，卻突然想往反方向延伸時，不需要取消重来——按 **`o`**（小寫 o），游標會跳到選取範圍的另一端，現在往哪個方向擴展都由你決定。

#### 結合 Text Object

Visual Mode 也能與 Text Object 搭配使用，選取範圍更精準：

- `v` + `iw`：選取游標所在的**內部單字**（inner word）
- `v` + `ap`：選取**整個段落**（around paragraph）

> 這裡有一個節省按鍵的小技巧：如果你已經熟悉 `yaw`（yank around word），直接用 `yaw` 比 `v` + `iw` + `y` 少按一個鍵。但如果想先確認選取範圍再執行，Visual Mode 就是更好的選擇。

---

### Line-wise Visual Mode（`V`）

#### 核心特性：最小單位是「一行」

即使你的游標在行的**中間**，一進入 Line-wise Visual Mode，**整行都會被選取**。這個設計直覺很明確：Line-wise 模式下，motion 作用的對象是「行」而非字元，所以 `j`、`k` 每次增減一行，`/` 搜尋也會讓**包含搜尋結果的整行**進入選取狀態。

#### 常用操作

- `V` + `j` / `k`：向下或向上擴展選取
- `V` + `/`：用搜尋結果擴展選取
- `o`：切換游標到選取範圍的對向端（與 Character-wise 相同）
- `~`：將選取範圍內的文字**全部改為大寫**（或 `gUU` / `guu`）
- `gv`：重新選取**上一次 Visual Mode 選取的相同範圍**，類型也會一致

---

### Block-wise Visual Mode（`Ctrl + V`）

這是 Vim 最有特色的模式，也是很多人「從此愛上 Vim」的原因。

#### 與 Character-wise 的根本差異

假設你在第一行選到第 5 個字元，然後 `j` 往下移動：

- **Character-wise**：第一行從頭選到第 5 格，第二行只選到**游標所在位置**（階梯狀）
- **Block-wise**：兩行**同樣寬度**的矩形區塊被選取（方正的矩形）

#### `O` 與 `o` 的差異（Block-wise 限定）

- `o`（小寫）：游標跳到選取區塊的**對向垂直端**（上 vs 下）
- `O`（大寫）：游標在**同一行**的兩個水平端之間來回跳動

#### 實戰情境一：一次改變多行的大小寫

想把連續多行的 `usa` 變成 `USA`？Block-wise Mode 讓這件事輕而易舉：

1. `Ctrl + V` 進入 Block-wise
2. 用 `j` 選取所有目標行
3. `~`：所選範圍內所有字元**切換大小寫**

#### 實戰情境二：一次在多行末尾附加文字（`A`）

想在每行末尾加上 `.`？

1. `Ctrl + V` → 選取目標行
2. `$`（`Shift + 4`）：讓選取延伸到**每行行尾**
3. `Shift + A`：進入 Insert Mode 並將游標移到每行**行尾**
4. 輸入你要加的文字，按 `Esc`

> 注意：此處**必須用 `A`（大寫）**，`a`（小寫）在 Visual Mode 中沒有這個功能。

#### 實戰情境三：一次在多行中間插入文字（`I`）

想在每行的**開頭**（或中間某位置）統一加上 `#`？

1. `Ctrl + V` → 選取目標行與目標欄位
2. `Shift + I`：進入 Insert Mode
3. 輸入文字，按 `Esc`

> 同樣的道理：**必須用 `I`（大寫）**。

#### 實戰情境四：一次替換多行的同一個單字（`c`）

想把連續多行的 `John` 統一改成 `Billy`？

1. `Ctrl + V` → 用 `j` 選取所有目標行
2. `c`：刪除選取範圍並進入 Insert Mode
3. 輸入新文字（`Billy`），按 `Esc`

---

## 💡 重點摘要

- **Visual Mode 讓你在執行操作前能先預覽選取範圍，大幅降低失手風險。**
- **Character-wise（`v`）、Line-wise（`V`）、Block-wise（`Ctrl + V`）三種模式各有擅場：片段選取、整行選取、垂直矩形選取。**
- **Block-wise Visual Mode 是 Vim 最具差異化的功能——多數編輯器根本無法做到同樣的事情。**
- **`o` 在 Character-wise / Line-wise 中切換兩端；Block-wise 下 `o` 跳垂直端、`O` 跳同列另一端。**
- **對多行同時 Insert / Append 時，必須用大寫 `I` 與 `A`，小寫 `i` / `a` 在 Visual Mode 中不起作用。**

---

## 🔑 關鍵字

Character-wise, Line-wise, Block-wise, Visual Mode, Text Object, Block-wise Insert, Block-wise Append
