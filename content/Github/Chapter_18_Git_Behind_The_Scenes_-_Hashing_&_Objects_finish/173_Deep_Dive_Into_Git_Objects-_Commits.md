# Git Commits 物件解析

## 📝 課程概述

本單元是 Git 內部運作機制系列的高潮——深入理解 **Commit 物件**的完整結構。我們將理解 Commit 如何將「快照」（Tree）、歷史（Parent Commit）與元資料（作者、訊息）串聯在一起，並認識 Git 之所以能完整追蹤專案歷史的底層原理。

## 核心觀念與實作解析

### Commit 物件的構成

一個 Commit 物件是 Git 整個版本控制模型的**膠水層**，它將四個關鍵元件組合成一個可追蹤的歷史節點：

1. **Tree**：指向這次提交的完整目錄結構快照
2. **Parent Commit**：指向前一個 commit（第一個 commit 沒有 parent）
3. **Author / Committer**：提交者與提交時間的中繼資料
4. **Message**：這次提交的文字說明

```bash
# 用 pretty-print 查看一個 commit 的內容
$ git cat-file -p 79dd6a...
tree c387a1d4e5f678901234567890123456789abcdef   # 這個 commit 的 Tree
parent 5119b2c3d4e5f678901234567890123456789abcd # 前一個 commit（如果有）
author Colt Steele <colt@mega.co> 1576000000 -0500
committer Colt Steele <colt@mega.co> 1576000000 -0500

Add dogs file
```

### Tree 是 Commit 的核心

**每一個 Commit 都會儲存一個 Tree**——那個 Tree 代表了「此 commit 建立時，整個專案目錄的完整快照」。

> **為什麼 Commit 不直接儲存檔案內容？** 因為 Tree 已經做了這件事。Commit 的職責是「記錄」而非「儲存」——它記錄了快照的根 Tree、來源的 Parent、以及描述性的訊息，實際的檔案內容由 Tree → Blob 處理。這樣的分工讓每個 Commit 物件非常輕量。

### Parent Commit——歷史的鏈條

Commit 之間透過 **Parent Commit Hash** 形成一條**歷史鏈**：

```
Commit C (hash: 4ec3...)
  └── parent: 5119b2... (Commit B)

Commit B (hash: 5119b2...)
  └── parent: 79dd6a... (Commit A)

Commit A (hash: 79dd6a...)  ← 第一個 commit
  └── parent: (不存在的)
```

> **第一個 commit 的特殊性**：因為沒有「之前」的 commit，所以第一個 commit 的 `parent` 欄位是**空白的**。這是終止條件，讓 Git 的歷史遍歷演算法知道「到此為止」。

只要持有任意一個 commit hash，Git 就可以沿著 parent 鏈一路回溯到專案的最開始——**所有歷史都可以從任何一個 commit 起點完整重建**。

### Git 的 Snapshot 模型 vs Delta 模型

這是理解 Commit 最關鍵的概念——**Git 儲存完整快照，而非差異**：

```bash
# 情境：dogs.txt 經過三次 commit

# Commit 1：初始版本（只有一個 blob）
$ git cat-file -p 79dd6a...^{tree}
100644 blob fd9154... dogs.txt

# Commit 2：新增內容（新的 blob），但重複的內容仍指向同一個 blob！
$ git cat-file -p 5119b2...^{tree}
100644 blob fd9154... dogs.txt   # Rusty + Wyatt → hash 相同，未變動
100644 blob a1b2c3... cats.txt   # 新增的 blob

# Commit 3：修改 dogs.txt（新的 blob）
$ git cat-file -p 4ec3a1...^{tree}
100644 blob 04998a... dogs.txt   # 新 blob（新 hash）
100644 blob a1b2c3... cats.txt    # cats.txt hash 完全相同
```

> **驚人的洞察**：在 Commit 2 中，`dogs.txt` 的 blob hash (`fd9154...`) 和 Commit 1 的完全相同。Git 聰明地讓 Commit 2 的 Tree 繼續指向同一個 Blob——**沒有浪費空間重複儲存未變動的檔案內容**。

### git log 的底層原理

當你執行 `git log` 時，Git 實際上在做的事是：

1. 從 `HEAD` 找到目前的 commit hash
2. 讀取該 commit，顯示 author、message、date
3. 取出 commit 的 parent hash
4. 重複直到 parent 為空

```bash
$ git log
commit 4ec3a1b2c3d4e5f678901234567890123456789a
Author: Colt Steele <colt@mega.co>
Date:   Mon Dec 16 12:00:00 2019

    Update dog's file

commit 5119b2c3d4e5f678901234567890123456789ab
Author: Colt Steele <colt@mega.co>
Date:   Mon Dec 16 11:50:00 2019

    Add cats file

commit 79dd6a4e5f678901234567890123456789abcdef
Author: Colt Steele <colt@mega.co>
Date:   Mon Dec 16 11:40:00 2019

    Initial commit
```

`git log` 只是不斷遍歷 parent chain 並格式化輸出的結果。

### Commit Hash 的由來

Commit 的 SHA-1 hash 是一次對整個 Commit 物件內容的 SHA-1 雜湊結果：

```
Commit Hash = SHA-1(
    tree-hash
    + parent-hash(es)
    + author-name + author-email + author-date
    + committer-name + committer-email + committer-date
    + commit-message
)
```

這意味著：

- **改了訊息 → 不同的 hash** → 完全不同的 commit（歷史被改寫了！）
- **改了任何一個檔案 → 對應的 blob hash 變了 → tree hash 變了 → commit hash 變了**

> **為什麼不可逆？** 因為 SHA-1 是單向函數。你知道 commit hash 無法反推出 tree 內容或訊息內容；但如果你知道原始內容，你可以驗證 hash 是否匹配——這就是 Git 歷史不可篡改的密碼學保證。

### 分支分歧的底層機制

當兩個分支指向不同 commit 時，底層發生的事情是：

```bash
# 分歧前的狀態
$ cat .git/refs/heads/master
5119b2c...   # master 和 feature 都指向同一個 commit

$ cat .git/refs/heads/feature
5119b2c...   # 完全相同的 hash

# 在 feature 上建立新 commit
$ git commit -m "Add new feature"
$ cat .git/refs/heads/feature
4ec3a1b...   # feature 現在指向新 commit

$ cat .git/refs/heads/master
5119b2c...   # master 仍然在原地，沒變
```

> **分支分歧的真相**：兩個 branch refs 只是分別指向**不同的 commit**，而每個 commit 都各自有一個指向早期共同祖先的 parent chain。Git 比較分支時，只需要比較兩個 refs 指向的 commit hash 是否相同即可。

### 第四種物件：Annotated Tag

Annotated Tag 是 Git 四種物件中最不常見的一種，但它同樣存放在 Objects 目錄中，有自己的 SHA-1 hash：

```bash
# Annotated Tag 的內容
$ git cat-file -p v1.0^{tag}
object a3f8b2c1...   # 指向的 commit
type commit
tag v1.0
tagger Colt Steele <colt@mega.co> ...

Release version 1.0
```

**Lightweight Tag vs Annotated Tag** 的差異在於：
- **Lightweight Tag**：只是 `refs/tags/` 下的一個檔案，內容只有一個 commit hash（**不是** Git Object）
- **Annotated Tag**：是一個**真正的 Git Object**，儲存在 Objects 目錄中，帶有自己的中繼資料（tagger、日期、message）

### 全部串聯：Git 的完整資料模型

```
refs/heads/master (Branch Pointer)
  → Commit Object (hash: a3f8b2c...)
        ├── tree: c387a1d... (整個專案的快照)
        │     ├── blob: index.html
        │     ├── blob: main.js
        │     └── tree: styles/
        │           ├── blob: app.css
        │           └── blob: nav.css
        ├── parent: 5119b2c... (上一個 commit)
        ├── author: Colt Steele <colt@mega.co>
        ├── committer: Colt Steele <colt@mega.co>
        └── message: "Update styles"
```

**Hash 的意義**：整個 Objects 目錄就像一個巨大的、全域的、去重的內容定址儲存系統。Tree、Blob、Commit、Tag 都以 SHA-1 hash 作為 key，只要持有任意一個 hash，Git 就能取出對應的內容。

## 💡 重點摘要

- **Commit 是 Git 歷史的基本單位**，每個 commit 儲存了：Tree（快照）、Parent（歷史鏈）、Author/Committer（中繼資料）、Message（說明）。
- **Git 儲存完整快照而非差異**：如果一個檔案連續兩次 commit 內容相同，Git 會讓兩個 commit 的 Tree 指向同一個 Blob，不會浪費空間重複儲存。
- **Parent chain 形成完整的歷史**：只要有第一個 commit 的 hash，就能沿著 parent 鏈完整重建整個專案歷史。
- **Commit Hash 是整個 Commit 內容的 SHA-1**，任何微小的變動（修改訊息、修改檔案）都會導致 hash 完全改變——這是 Git 歷史不可篡改的密碼學基礎。
- **Branch 分歧只是兩個 refs 指向不同的 commit**——兩個 commit 各自有通往共同祖先的 parent chain，Git 的分支模型就是建立在這個簡單事實之上。

## 關鍵字

Commit Object, Tree, Parent Commit, Author, Committer, SHA-1 Hash, Snapshot Model, Parent Chain, git log, git cat-file -p, Branch Divergence, Annotated Tag, Lightweight Tag, History Chain, Immutable History
