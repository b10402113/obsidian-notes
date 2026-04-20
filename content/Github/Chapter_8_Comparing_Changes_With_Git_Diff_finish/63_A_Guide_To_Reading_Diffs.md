# A Guide to Reading Diffs — 如何解讀 Git Diff 輸出格式

## 📝 課程概述

本節是整個 Chapter 8 的核心。我們會深入剖析 `git diff` 輸出的每一個組成部分——從最上方的 `diff --git` 標頭，到 `@@` Chunk Header，再到 `-` 與 `+` 開頭的實際內容行。理解這些符號的意義，才能真正把 Diff 變成你除錯與 Code Review 的利器，而不是一團乱碼。

## 核心觀念與實作解析

### Diff 輸出的基本結構

當你執行 `git diff` 時，Git 產生的輸出總是遵循相同結構。以一個名為 `rainbow.txt` 的檔案為例，假設我們將「purple」改成了「indigo」和「violet」，輸出會長這樣：

```
diff --git a/rainbow.txt b/rainbow.txt
--- a/rainbow.txt
+++ b/rainbow.txt
@@ -3,4 +3,5 @@
 yellow
 green
-blue
+indigo
+violet
```

讓我們逐行拆解。

### 第一行：`diff --git` 檔案標頭

```
diff --git a/rainbow.txt b/rainbow.txt
```

這一行說明 Git 正在比較 `rainbow.txt` 的兩個版本。`a/` 代表**舊版本**（File A），`b/` 代表**新版本**（File B）。在大多數使用情境下，這兩者是同一個檔案在不同時間點的狀態——例如「上次 Commit 的版本」vs.「目前 Working Directory 中的版本」。

### 第二、三行：檔案元資料行

```
--- a/rainbow.txt
+++ b/rainbow.txt
```

- `---` 代表 File A（通常是較舊的版本）
- `+++` 代表 File B（通常是較新的版本）

> **教學提示**：這兩行的格式源自 Unix 的 `diff` 工具，Git 繼承了這套表示法。`---` 不是「減去」的意思，而是標記 A 檔案；`+++` 不是「加總」，而是標記 B 檔案。

### 第四、五行：符號對照行（通常可忽略）

```
@@ -3,4 +3,5 @@
```

這兩行（`-3,4` 與 `+3,5`）是 Chunk Header，見下節詳細說明。

### `@@` Chunk Header：Git 只顯示「有變化」的範圍

Git 的聰明之處在於：**它不會把整個檔案全部顯示出來**。如果你的檔案有 10,000 行，而你只改了 1 行，Git 只會顯示那一行附近的一小段上下文（Context）。

Chunk Header 的格式是 `@@ -start,length +start,length @@`：

- `@@ -3,4` → 從 File A 的第 3 行開始，擷取 **4 行**
- `+3,5` → 從 File B 的第 3 行開始，擷取 **5 行**
![[Pasted image 20260413195553.png|700]]

**為什麼長度不同？** 因為我們在 File A 中刪了 1 行（`-`），又在 File B 新增了 2 行（`+`），所以兩個版本的行數自然不同。
### 實際內容行：`-` 與 `+` 的意義

```
 yellow        ← 上下文，兩者都有，無標記
 green         ← 上下文，兩者都有，無標記
-blue         ← `-` 開頭：這行來自 File A，在 File B 中不存在（被刪除）
+indigo       ← `+` 開頭：這行來自 File B，在 File A 中不存在（被新增）
+violet       ← `+` 開頭：同上
```

> **重要澄清**：`+` 和 `-` **並不總是代表「新增」和「刪除」**。它們代表的是「這行內容來自哪個版本」。在比較兩個分支時，如果把順序顛倒，`+` 和 `-` 的含義就會反過來——所以理解 **順序** 非常重要。

### Git 選擇性地顯示上下文

Git 不會顯示整個檔案，而是聰明地只顯示**變化周圍的上下文**。這就是為什麼你會看到一些看起來「沒什麼變化」的行（例如 `yellow`、`green`）出現在差異區塊中——它們只是用來告訴你「這段變化發生在檔案的哪個位置」。

如果你想看到更多或更少的上下文，可以用 `--unified=<n>` 參數控制。

### 一次比較多個檔案時

當一次 `git diff` 比較多個檔案時，Git 會把每個檔案的差異**分組輸出**，每組結構相同。例如：

```
diff --git a/index.html b/index.html
...（index.html 的差異）

diff --git a/main.css b/main.css
...（main.css 的差異）
```

每次看到新的 `diff --git` 開頭，就代表進入了一個新檔案的差異區塊。

## 💡 重點摘要

- `diff --git` 標頭說明比較的兩個檔案，`a/` 是舊版、`b/` 是新版
- `-` 開頭的行來自 File A，`+` 開頭的行來自 File B，**順序決定符號的實際意義**
- `@@ -3,4 +3,5 @@` 告訴你：File A 從第 3 行取 4 行，File B 從第 3 行取 5 行
- Git 會自動只顯示變化周圍的**上下文**，而不會顯示整個檔案
- 比較分支或 Commit 時，**順序會翻轉 `+` 和 `-` 的意義**，務必留意

## 關鍵字

diff --git, File A, File B, Chunk Header, @@, context, [[git diff]] output, unified diff, minus, plus, staging area
