# Git Tags 概念總覽：標記歷史中的重要時刻

## 📝 課程概述

本單元帶領學員認識 Git Tags 的核心概念與應用場景。Tags 是用來**永久指向特定 commit 的指標**，與會自動移動的 Branch Reference 不同，Tag 一旦設定就靜態釘在該 commit 上，是軟體發布與版本管理的基礎工具。

## 核心觀念與實作解析

### 什麼是 Git Tag？

在 Git 中，Tag 就是一個**指向特定 commit 的指標**，就像在歷史上的一個 commit 貼上一張便利貼，標記「這裡很重要」。這就是 Git Docs 所說的：

> "Git has the ability to tag specific points in repositories history as being important."

**Tags 與 Branches 的關鍵差異**在於：
- **Branch Reference**：隨新 commit 產生會自動前進，永遠指向該分支的最新 commit
- **Tag**：靜態不動，一旦設定就永遠指向那個 commit（除非你手動移動它）

### 為什麼需要 Tags？

我們之所以需要 Tags，是因為軟體開發有**版本發布**的需求。當專案進行到一定階段，團隊需要對外宣示：「這一刻的程式碼就是 3.1.0 版本。」這個標記必須永久存在，讓所有人日後都能找到對應的 commit。

> 想像一下：如果沒有 Tag，你要怎麼向別人解釋「v3.1.0 的程式碼在哪裡？」有了 Tag，你只需要說「3.1.0 標記的那個 commit」就行了。

### 兩種 Tag 類型

#### Lightweight Tag（輕量標籤）

這是最簡單的形式，**只是一個名稱**，指向一個 commit，沒有任何額外資訊。

就像在便利貼上只寫一個名字，然後貼上去，僅此而已。

#### Annotated Tag（附注標籤）

這種 Tag 包含豐富的 metadata：
- **Tag 訊息**（類似 Commit Message）
- **作者名稱與 Email**
- **建立日期**
- **完整資訊可供驗證**

大型開源專案通常**偏好使用 Annotated Tags**，因為它們可以被 GPG 簽署，具備驗證性。

### Tag 的命名彈性

Tag 的名稱可以是**任何你想要的字串**，最常見的當然是版本號（如 `v3.1.0`），但理論上你可以命名為 `hotdog` 或 `pickle`，完全取決於你的使用需求。

## 💡 重點摘要

- Tag 是**靜態指標**，一經設定不會自動移動；Branch Reference 會隨 commit 自動前進
- Tags 最常見的用途是**標記軟體版本發布點**
- Lightweight Tag 僅包含名稱；Annotated Tag 包含完整 metadata，**大型專案普遍偏好後者**
- `git tag` 是操作 Tags 的核心命令基底

## 關鍵字

Git Tag, Lightweight Tag, Annotated Tag, Branch Reference, Semantic Versioning, 版本發布, metadata, commit pointer
