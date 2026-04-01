# Vim Text Objects：邏輯區塊編輯術

## 📝 課程概述

本單元介紹 Vim 中最强大的編輯概念之一：**Text Objects（文字物件）**。不同於 `w`（word）這類純移動指令，Text Objects 讓你能以「語意區塊」為單位進行操作，無論你的游標在區塊的哪個位置，都能精準地對整個句子、段落或引號範圍執行刪除、變更、複製等動作，大幅提升編輯效率。

---

## 核心觀念與實作解析

### 為什麼需要 Text Objects？

想像你要刪除一個 `word`，如果游標不在單字開頭，用 `dw` 只能刪除「從游標位置到單字結尾」的範圍，而不是整個單字。這是因為 `w` 是 **motion（移動指令）**，它的行為取決於游標當下位置。

Text Objects 的設計正是為了解決這個痛點：**不管游標在物件的哪個位置，指令都會作用在整個物件上。**

### Text Objects 的基本語法

所有 Text Objects 的使用方式都遵循同一個模式：

```
[operator] + [a 或 i] + [object]
```

- **`a`（around）**：包含物件的**邊界符號**（delimiter）一起操作
- **`i`（inner）**：僅操作物件**內部內容**，不包含邊界符號

> 💡 老師提供了一個很好記的比喻：`a` 就像「一顆蘋果（a fruit）」，包含了果皮果肉整個果實；`i` 是 inner，只取果肉。

### 單字（Word）Text Objects

- `aw`（a word）：刪除整個單字**含空白**
- `iw`（inner word）：刪除單字**不含空白**

```text
# 示範：cursor 在 "compromise" 中間
compromise  # 游標在此單字內任意位置

# daw → 刪除 "compromise "（含尾部空白）
# diw → 刪除 "compromise"（不含空白）
```

> ⚠️ 小寫 `w` 以**空白與標點**作為單字邊界；大寫 `W` 只以**空白**作為邊界，遇到標點不會截斷。

### 句子（Sentence）Text Objects

- `as`（a sentence）：刪除整個句子**含尾部空白**
- `is`（inner sentence）：刪除句子**不含空白**

```text
Hello world. Next sentence continues here.

# 游標在任何位置，das → 刪除整個 "Hello world." 含空白
# 游標在任何位置，dis → 刪除 "Hello world." 不含空白
```

> 💡 Vim 判斷句子結尾的規則是：**句號（`.`）、驚嘆號（`!`）、問號（`?`）後接空白、Tab 或行尾。**

### 段落（Paragraph）Text Objects

- `ap`（a paragraph）：刪除整個段落**含段落間的空白行**
- `ip`（inner paragraph）：刪除段落內容**不含空白行**

> 💡 段落的邊界就是**空白行**，這個概念在編輯配置文件或 Markdown 文件時特別實用。

### 程式語言適用的 Text Objects

Vim 內建了專門針對程式碼與標記語言的 Text Objects，分為以下幾類：

#### 1. 括號類（Parentheses / Brackets）

```text
[green, blue, red]   # 游標在括號內任意位置

ci]  →  刪除括號內內容並進入插入模式（可替換成 ci( 兩者等效）
ca]  →  刪除含括號的整個區塊並進入插入模式
```

- 小括號 `()`：`a)` / `i)` 或 `ab` / `ib`（Vim 將其稱為 **block**）
- 中括號 `[]`：`a[` / `i[`
- 大括號 `{}`：`a{` / `i{` 或 `aB` / `iB`

#### 2. 角括號（Angle Brackets）

```text
<html>some text</html>

yi>   →  複製角括號內的內容（不含 < >）
ya>   →  複製角括號及其內容
```

#### 3. 標籤（Tags，適用於 HTML / XML）

```text
<p><strong>important</strong></p>

cit  →  刪除 <p> 標籤內所有內容並進入插入模式
cat  →  刪除整個 <p> 標籤及其內容
dit  →  刪除標籤內內容但保留標籤本身
```

> 💡 Vim 的 tag object 對**自訂標籤**同樣有效，不僅限於標準 HTML 標籤。

#### 4. 引號類（Quotes）

```text
backup_server = "Deepfreeze01"

ci"   →  修改雙引號內的內容
ci'   →  修改單引號內的內容
ci`   →  修改反引號內的內容
```

#### 5. 大括號類（Curly Braces，適用於 JS/JSON/程式碼區塊）

```text
{ type: "cat", score: 7 }

ci{   →  僅修改大括號內的內容（不含 { }）
ca{   →  修改含大括號的整個區塊
```

---

## 💡 重點摘要

- **Text Objects 的核心價值在於「游標位置無關性」：無論游標在物件的哪個位置，操作都作用於整個物件。**
- **`a` 包含邊界（delimiter），`i` 不含邊界——牢記這個區分就能舉一反三。**
- **Text Objects 讓你以「語意單位」而非「字元位置」思考，大幅減少計算要刪除幾個字元的認知負擔。**
- **HTML/XML 編輯時 `cit` / `cat` 是最高效的標籤編輯組合， 실무 中使用頻率極高。**

---

## 🔑 關鍵字

Text Objects, Motion, Delimiter, Sentence, Paragraph, Tag, Macro
