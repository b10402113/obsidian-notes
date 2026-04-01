# Exercise：Text Objects 實戰演練

## 📝 課程概述

本單元是 Text Objects 的實戰練習課。我們透過一個練習檔案（`text_objects_practice.txt`），逐步演示各种 Text Objects 的實際操作情境，包括單字、括號、引號、角括號、標籤、大括號、句子與段落等，幫助學員將理論轉化為肌肉記憶。

---

## 核心觀念與實作解析

### 實戰情境一：單字操作——`aw` vs `iw` 的對比

```text
The time traveler arrived early.
```

- **motion `w`**：從游標當下位置前進到下個單字邊界，配合 `d` 就是 `dw`
- **Text Object `aw`**：不論游標在哪，**整個單字（含尾部空白）**都會被刪除

> 💡 練習時游標放在單字中間，分別執行 `dw` 與 `daw`，觀察差異。

### 實戰情境二：改變（Change）操作結合 Text Objects

要將 `traveler` 改為 `tourist`，不需要慢慢刪除再輸入：

```text
# 游標在 "traveler" 任意位置
cwtourist   # cw = change word，會刪除游標到單字結尾並進入插入模式
# 但更好的方式：
ciw         # 改變整個單字（inner word），游標無關位置
```

### 實戰情境三：括號內文字——`ci(` 的魔力

```text
We will call him (John Doe) tomorrow.
```

- 游標放在括號內任意位置（開括號、閉括號、或中間都可以）
- `ci(` 或 `ci)`：Vim 會聰明地幫你把游標放在括號內，準備讓你輸入替換文字

### 實戰情境四：刪除整個括號區塊——`da(`

```text
Send email to (user@example.com) for confirmation.
```

- `da(` 或 `da)`：連同括號**本身及其內容全部刪除**

### 實戰情境五：雙引號與單引號字串

```text
The weather was "It was cold".
```

- `ci"`：改變雙引號內內容，游標可在引號上或內部
- `ci'`：改變單引號內內容，同樣不限位置

### 實戰情境六：角括號與寄存器（Registers）的結合

```text
<Yank me> should be stored in a register.
```

```text
# 將 "<Yank me>" 存入 a register（不含括號）
"ayi>   # "a = 目標寄存器, y = yank, i> = inner angle bracket

# 將 "<Yank me>" 存入 a register（含括號）
"aya>   # a = around，會把 < > 一起複製進去

# 確認寄存器內容
:reg a
```

### 實戰情境七：HTML Tag 操作——`cit` / `cat`

```text
<a href="Linux Training Academy">Visit our site</a>
```

- `cit`：改變 `<a>` 標籤內的所有內容（`Linux Training Academy`）
- `cat`：刪除整個 `<a>` 標籤含標籤本身

> ⚠️ 如果游標誤放在巢狀標籤（如 `<title>`）上執行 `dit`，只會刪除該內層標籤，而不會影響外層的 `<a>`。務必確認游標位置。

### 實戰情境八：大括號（Curly Braces）

```text
musicians: {
  name: "Charlie Parker",
  instrument: "Alto Saxophone"
}
```

- `di{`：刪除大括號內所有內容（不留 `{}`）
- `da{`：刪除整個區塊含 `{}` 本身

> 💡 大括號操作在 JavaScript、JSON、CSS 等語言中極為常見。

### 實戰情境九：刪除空白行 + 合併行

刪除一整個包含內容的區塊後，有時會剩下孤立的空行。這時可以用：

- `J`（小寫）：將下一行合併到當前行尾
- `gJ`：合併但不插入空白

### 實戰情境十：句子與段落的 yank / delete

```text
# Yank 一個句子到 s register
"syas   # "s = s寄存器, y = yank, as = a sentence

# 確認內容
:reg s

# 刪除整個段落
dap     # delete a paragraph
```

> 💡 Vim 的「句子」判斷邏輯：**句號、驚嘆號、問號後接空白或行尾。**即使句子實際上跨了多行，Vim 仍會聰明地視為同一個句子。

---

## 💡 重點摘要

- **`ci"`、`ci'`、`ci(`、`ci[` 等 `ci` 組合是實戰中最高頻的 Text Objects 操作。**
- **寄存器（Registers）可以配合 Text Objects 使用，`"ayiw` 能把一個單字存入 a register，`"ayas` 能把整句話存入。**
- **執行破壞性操作前（如 `dat`）要確認游標位置，否則可能只作用在巢狀標籤上。**
- **`dap` 一個指令刪除整段，是整理長文件時的利器。**
- **寄存器內容可以透過 `:reg [name]` 隨時確認，是 Debug 你的 Vim 操作的好習慣。**

---

## 🔑 關鍵字

Registers, Tag, Angle Bracket, Curly Brace, Yank, Paragraph, cit, da
