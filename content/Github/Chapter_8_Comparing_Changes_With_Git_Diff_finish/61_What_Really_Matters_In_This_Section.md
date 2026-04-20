# What Really Matters in This Section — Git Diff 總覽

## 📝 課程概述

本節課程聚焦於 Git 中一個重要的「唯讀」指令：`git diff`。與 `add`、`commit`、`status` 這些日常高頻指令不同，`git diff` 並非每天都會用到，但它能幫助我們在複雜的開發情境中精確掌握「到底改了什麼」。本節的核心目標是讓你學會閱讀 `git diff` 的輸出格式，並理解不同比較場景背後的邏輯。

## 核心觀念與實作解析

### 為什麼需要 `git diff`？

在真實開發中，我們常常會在多個檔案之間來回修改，過了一段時間後，很難憑記憶回想「這個檔案到底改了什麼」。`git diff` 就是為了解決這個問題而生的——它讓我們能清楚看見 Working Directory、Staging Area、Commit 和 Remote 四個區域之間的差異。

> **關鍵點**：`git diff` 不會修改任何東西，也不會影響 Repository 的狀態。它是純資訊性的指令，就像 `git log` 和 `git status` 一樣，純粹只是讓你「窺視」變化。

### `git diff` 能比較什麼？

`git diff` 這個單一指令可以應對多種比較場景：
![[Pasted image 20260413195201.png|700]]
- **Working Directory vs. Staging Area**：哪些修改還沒 staging？
- **Staging Area vs. Last Commit**：即將被 commit 的內容是什麼？
- **兩個 Branch 之間**：這個 Feature Branch 和主線有什麼差異？
- **兩個 Commit 之間**：某次重構究竟改動了哪些檔案？
- **本地 vs. Remote**：還沒 pull 的變更有哪些？
![[Pasted image 20260413195028.png|700]]
### 本節學習重點

在這整個章節中，**最關鍵的是理解如何閱讀 `git diff` 的輸出格式**。講師特別強調：

- `git diff` 的輸出看起來一開始會有些嚇人（充滿 `@@`、`+`、`-` 符號），但只要熟悉它的結構，其實並不複雜。
- `git diff` 的**基本用法**（不帶任何參數）是「Critical」等級，必須熟練。
- 其餘各種參數變化（比較分支、指定檔案、指定 Commit）是「Important」等級，用到時查手冊即可。

### 為什麼 `git diff` 輸出看起來很複雜？

當你運行 `git diff` 時，Git 會產生一段類似這樣的輸出：
![[Pasted image 20260413195237.png|700]]

```
diff --git a/index.html b/index.html
--- a/index.html
+++ b/index.html
@@ -25,7 +25,9 @@
 red
 yellow
-blue
+indigo
+violet
```

- `diff --git` 開頭說明比較的兩個檔案（通常是同一個檔案的兩個版本）
- `---` 代表**舊版本**（File A），`+++` 代表**新版本**（File B）
- `-` 符號開頭的行 = 來自舊版（被移除的內容）
- `+` 符號開頭的行 = 來自新版（被新增的內容）
- `@@` 包圍的數字是**Chunk Header**，告訴你這個差異區塊涵蓋檔案的哪些行

這段輸出格式的詳細解析，會是下一支影片的重點。

## 💡 重點摘要

- `git diff` 是一個**純資訊性**的指令，不會對 Repository 造成任何改變
- **閱讀輸出格式**是本節最重要的技能，勝過記憶所有參數變化
- `git diff` 能比較 Working Directory、Staging Area、Branch、Commit 等多種組合
- 理解 `+`（新增）與 `-`（移除）的意義，是解讀任何 Diff 的基礎
- 基本 `git diff` 的使用頻率較低，但當你需要時，它會是無可取代的工具

## 關鍵字

[[git diff]], Working Directory, Staging Area, commit, branch, diff output format, chunk header, git log, git status
