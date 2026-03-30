# Git refs 目錄解析

## 📝 課程概述

本單元深入探討 `.git/refs` 目錄的內部結構，解開 Branch 和 Tag 在 Git 眼中「到底是什麼」的謎底。我們將理解 refs（references）如何讓 Git 維持一個龐大專案的歷史與分支結構，並認識 remote 追蹤分支背後的運作機制。

## 核心觀念與實作解析

### refs 是什麼？

**refs = references（參照）**，是 Git 用來「指向 commit」的所有機制的總稱。當我們說「某個分支指向 commit A」時，Git 實際上是在 `refs` 目錄下創建了一個檔案，內容是 commit A 的 SHA-1 hash。

這就是整個 refs 目錄的本質：**一個儲存 commit hash 的地方。**

### refs 目錄的三大子目錄

```bash
.git/refs/
  ├── heads/    # 本地分支（每一個 branch 都是一個檔案）
  ├── tags/     # 所有標籤
  └── remotes/  # Remote 追蹤分支（如 origin/master）
```

### refs/heads——本地分支的真面目

在 `refs/heads/` 中，**每一個本地分支都是一個獨立的檔案**。檔案名稱就是分支名稱，檔案內容只有一個 commit hash。

```bash
# 假設有一個分支叫 "dark-mode"
$ cat .git/refs/heads/dark-mode
796a3b2c1d4e5f6...   # 這就是該分支目前指向的 commit hash
```

> **為什麼 Git 要這樣設計？** 答案在於簡單性。一個分支只需要知道「我指向哪個 commit」，而 hash 可以完美地作為那個 commit 的唯一識別碼。Branch Pointer（分支指標）並不是什麼神秘的指標，而就是一個檔案。

當你新建一個分支時，Git 只是複製當前 HEAD 指向的那個 commit hash 到新檔案裡：

```bash
$ git switch -c chicken
# Git 在 refs/heads/ 下創建了一個名為 "chicken" 的檔案
# 內容寫入與 new-branch 相同的 commit hash
```

此時 `new-branch` 和 `chicken` 指向同一個 commit，這就是「分歧前的同一個節點」。

### refs/tags——標籤也是參照

標籤（Tag）和分支一樣，都是 refs，只是存在 `refs/tags/` 目錄底下。每一個 Tag 都是一個檔案，內容同樣是一個 commit hash（lightweight tag）或更複雜的物件（annotated tag）。

```bash
$ cat .git/refs/tags/v1.0
a1b2c3d4e5f6...   # 標籤指向的 commit
```

> **重要區分**：Tag 和 Branch 最大的差異在於語義——Branch 會隨著新 commit 而「移動」，Tag 則是靜止的命名快照，通常用來標記發行版本。

### refs/remotes——Remote 追蹤分支的運作方式

Remote 追蹤分支存在 `refs/remotes/` 目錄下，結構稍有不同：

```bash
.git/refs/remotes/
  └── origin/
      ├── master   # 代表 origin 這個 remote 上的 master 分支
      └── dev
```

當你執行 `git fetch` 時，Git 會連線到 remote，並將 remote 上各分支的 **最新 commit hash** 更新到 `refs/remotes/origin/` 下的對應檔案中。

這就是 Git 能判斷「本地分支落後於遠端幾個 commit」的原理——它只是比較 `refs/heads/master`（本地分支）和 `refs/remotes/origin/master`（remote 追蹤分支）的 hash 值是否相同。

```bash
$ git fetch origin
# Git 連線 origin，取回 remote 各分支的最新 commit hash
# 將這些 hash 寫入 refs/remotes/origin/ 下的對應檔案
```

### HEAD：指向 refs 的指標

HEAD 檔案（`.git/HEAD`）的內容通常不是直接寫入 commit hash，而是寫成 `ref: refs/heads/分支名稱`。這是一層**間接參照**：

```
HEAD（指標）
  → refs/heads/master（分支檔案）
    → commit hash（實際的 commit 物件）
```

在 Detached HEAD 狀態下，HEAD 檔案則直接寫入 commit hash，跳過了分支這一層。

## 💡 重點摘要

- **refs 是 Git 用來指向 commit 的機制**，無論是 branch、tag 還是 remote 追蹤分支，都屬於 refs。
- `.git/refs/heads/` 中的**每一個分支就是一個檔案**，內容只有一個 commit hash，沒有任何魔法。
- Remote 追蹤分支（`refs/remotes/`）儲存的是 **remote 上各分支的最新 commit hash**，用來支援分支落後與否的比較。
- **HEAD 通常指向一個 branch 檔案**（間接參照），而非直接指向 commit，這是 Git 分支模型的基礎設計。
- 新建分支只是複製一個 hash 到新檔案裡——Git 不會複製任何歷史或檔案內容。

## 關鍵字

refs, refs/heads, refs/tags, refs/remotes, Branch Pointer, Remote Tracking Branch, Lightweight Tag, HEAD, Detached HEAD, SHA-1 Hash, git fetch, git switch -c
