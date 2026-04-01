# Vim 實作練習 06：插入、改寫、搜尋與取代的完整演練

## 📝 課程概述

本單元是 Chapter 6 全部內容的**實作驗證**——透過一個完整的練習檔案（`insert_practice.txt` 與 `search_practice.txt`），帶你將前面三課所學的 `I`/`A`/`o`/`O`、Replace mode、`cw`、`Case` 指令，以及 `/` 搜尋、`:` substitute 全域取代，**全部實際操作過一遍**。動手做是內化這些指令的唯一途徑。

---

## 核心觀念與實作解析

### Part 1：插入、改寫、Case 與合併實作

以下按練習順序逐一說明操作邏輯：

#### `i` vs `I` 的對比練習

- **`i`**：從目前游標位置進入 Insert mode，適合在任意位置插入文字
- **`I`**（`Shift+I`）：一鍵跳至行首第一個非空白字元並進入 Insert mode

```text
原本：What is your favorite editor?
操作：I → 輸入 "Vim " → <Esc>
結果：Vim What is your favorite editor?
```

> **實用情境**：多數時候你想在行首加入內容，`I` 比起 `0i` 或 `^i` 都更直覺。

#### `a` 與 `A`：插入到游標右側 / 行尾

- **`a`**：進入 Insert mode，游標位置**不動**，輸入出現在目前字元右側
- **`A`**：直接跳到行尾並進入 Insert mode

#### `o` 與 `O`：新增行並進入 Insert mode

```text
o  →  在目前行下方新增一行，進入 Insert mode
O  →  在目前行上方新增一行，進入 Insert mode
```

實務上常用 `o` 快速新增一行空白再輸入內容，比 `A<Enter>` 更順暢。

#### Replace mode：`R` 與 `r`

```text
Replace me with a different sentence.
操作：
  1. 移到目標行開頭
  2. R 進入 Replace mode
  3. 輸入 "I love using vim!"
結果：I love using vim!（覆蓋原內容，<Esc> 離開）

bat → cat（單一字元替換）
操作：移到 b，按 r，再按 c → 游標自動回到 Normal mode
```

> **`r` 完成後不需要按 `Esc`**——這是 Vim 對「只做一件事」的直覺設計。

#### `cw` — Change Word

```text
great
操作：/gr → <Enter> → cw → 輸入 brilliant → <Esc>
結果：brilliant
```

`cw` 會刪除從游標到 word 結尾的範圍，並進入 Insert mode。與 `f` 或 `/` 搜尋結合，是快速定位後立刻修改的標準組合。

#### `cc` — Change Line（整行改寫）

```text
某一行內容
操作：cc → 輸入 "The sky is beautiful." → <Esc>
結果：The sky is beautiful.
```

`cc` 保留行首空白，只清除內容。適合用來**完全取代**一行文字。

#### Case 切換：`~` 家族

| 操作 | 指令 | 說明 |
|------|------|------|
| 單一字元大小寫交換 | `~` | 游標在誰身上，交換誰 |
| 單字大小寫交換 | `g~w` | 交換游標後一個 word |
| 整行大小寫交換 | `g~~` | 交換整行 |
| 單字轉大寫 | `gUw` | 強制全大寫 |
| 單字轉小寫 | `guw` | 強制全小寫 |

```text
Monday
操作：/~M → <Enter> → ~ →  M → Monday
shout
操作：/sh → <Enter> → gUw → SHOUT
```

> **`gU` vs `~`**：若單字中已有大寫字母，用 `~`（交換）不會得到全大寫結果。這時要用 `gU` 強制轉大寫。

#### Count + Insert mode：`80I*` 與 `3o# `

```text
80I*<Esc>   →  建立一行 80 個星號
3o# <Esc>  →  建立三行以 "# " 開頭的行（適合 YAML 或 Markdown 列表）
```

這個技巧在建立分隔線、註解區塊、待辦清單時特別實用。

#### Join：`J` 與 `gJ`

```text
The cat chased the mouse.
The dog barked loudly.
操作：將游標放在第一行，按 J
結果：The cat chased the mouse.  The dog barked loudly.
```

`J` 會自動在行尾添加空格（句點後自動加兩個空格）。若不想要額外空格，用 `gJ`。

---

### Part 2：搜尋與取代實作

#### 行內搜尋：`f` / `F` / `t` / `T` 的實戰演練

```text
操作：f f     →  跳到下一個字母 f
      ;      →  重複上一次 f，往右繼續找 f
      F t    →  往回找 t
      t b    →  跳到 b 之前的位置
      T r    →  往回跳到 r 之後的位置
```

#### 全文搜尋：`/` 與 `?`

```text
/and <Enter>   →  搜尋 "and"（注意加空格避免匹配其他單字）
n              →  繼續往同方向找下一個
N              →  往反方向找
```

> **加空格的技巧**：搜尋 `/and ` 只會匹配完整單字 `and`，而不會匹配 `incandescent`、`hand` 等含有 `and` 的單字。

#### 用 `*` / `#` 快速找單字

```text
將游標放在 "it" 上
*             →  直接跳到下一個 "it"
#             →  往回找 "it"
n / N         →  繼續往同方向或反方向
```

#### 全域取代：`:%s/old/new/g`

```text
:%s/sat/lad/g    →  將整份檔案所有 "sat" 替換為 "lad"
```

```vim
:%s/SAT/lad/g    " % = 整個檔案範圍
                 " g = 該行所有匹配都替換
```

> **`g` flag 為什麼幾乎必加？** 因為若某行同時有兩個 `SAT`，不加 `g` 只會替換第一個。若你不確定行內是否會有多個匹配，**一律加 `g`** 是最安全的做法。

#### 放棄修改離開：`q!`

```vim
:q!    " 放棄所有修改，強制退出
```

---

## 💡 重點摘要

- **`I`/`A`/`o`/`O` 讓你在不需要先移動游標的情況下，精准選擇插入點**——牢記這四個指令，減少多餘的移動動作。
- **`r`（單字元）+ `R`（替換模式）**各有適用場景；`r` 替換完後自動回到 Normal mode，是修正錯字的極速解法。
- **`gU` / `gu` vs `~`**：`gU` 強制大寫，`gu` 強制小寫，`~` 只是交換——根據目標需求選對指令。
- **搜尋時加空格（`/word `）可以避免意外匹配包含目標字串的其他單字**，這是新手常忽略的細節。
- **`:%s/old/new/g` 是「全檔案、所有匹配」的快捷鍵**——修改設定檔中的主機名、IP、路徑時，這是最乾淨的做法。

---

## 🔑 關鍵字

I, A, o, O, r, R, cw, cc, ~, g~, gU, gu, J, gJ, f, F, t, T, ;, ,, *, #, :s, :%s, g flag, q!
