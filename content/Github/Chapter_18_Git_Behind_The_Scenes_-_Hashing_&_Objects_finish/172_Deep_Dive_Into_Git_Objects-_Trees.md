# Git Trees 物件解析

## 📝 課程概述

本單元深入探討 Git 的第二種核心 Object——**Tree 物件**。Tree 是 Git 用來描述「目錄結構」的物件，連接了 Blob（檔案內容）與 Commit（提交），形成完整可追蹤的專案狀態快照。我們將理解 Tree 如何在 Blob 與目錄結構之間建立對應關係。

## 核心觀念與實作解析

### Blob 只儲存內容，Tree 才儲存「檔案名稱」

在上一個單元中，我們已經知道 **Blob 只儲存檔案的內容，不包含檔案名稱**。那麼 Git 是如何知道「`index.html` 這個檔名對應到哪個 blob」的呢？

答案就是 **Tree 物件**。

> **這是理解 Git 資料模型的關鍵跳板**：Blob = 內容，Tree = 目錄結構（含檔名與子目錄的對應關係）。

### Tree 的結構

每一個 Tree 物件就像一份「資料夾清單」，裡面包含多個 entry，每個 entry 記錄了：

- **類型**：這個 entry 指向的是 Blob（檔案）還是 Tree（子目錄）
- **名稱**：檔案名稱或目錄名稱
- **Hash**：對應 Blob 或 Tree 的 SHA-1 hash

用 `git cat-file -p` 檢視一個 Tree 時，你會看到這樣的輸出：

```bash
$ git cat-file -p c387a1d4
100644 blob a1b2c3d4e5f6... index.html
100644 blob d4e5f6a7b8c9... main.js
040000 tree e7f8a9b1c2d3... styles
040000 tree f1e2d3c4b5a6... images
```

| 欄位 | 意義 |
|------|------|
| `100644` | 檔案權限（`100` =  regular file，`644` = 權限） |
| `blob` / `tree` | 這個 entry 的類型 |
| hash | 對應物件的 SHA-1 hash |
| 名稱 | 檔案名稱或目錄名稱 |

### Tree 與 Blob 的關係——完整範例

假設我們有以下目錄結構：

```
project/
├── index.html       (blob: 1f7a8c...)
├── main.js         (blob: be3210...)
├── styles/
│   ├── app.css     (blob: 10abb9...)
│   └── nav.css     (blob: ffe771...)
└── images/
    └── logo.png    (blob: 3ab4c5...)
```

Git 會為這個結構創建多個 Tree 物件：

```bash
# 根目錄 Tree（包含所有一級 entry）
$ git cat-file -p root-tree-hash
100644 blob 1f7a8c... index.html
100644 blob be3210... main.js
040000 tree styles-hash styles   # 子目錄 → 另一個 Tree
040000 tree images-hash images  # 子目錄 → 另一個 Tree

# styles/ 目錄的 Tree
$ git cat-file -p styles-hash
100644 blob 10abb9... app.css
100644 blob ffe771... nav.css

# images/ 目錄的 Tree
$ git cat-file -p images-hash
100644 blob 3ab4c5... logo.png
```

### Tree 的遞迴結構

Tree 可以指向其他 Tree（子目錄），形成一個**樹狀結構**。這就是為什麼 Git 能夠處理任意深度的目錄巢狀：

```bash
# 子目錄下又有子目錄也完全沒問題
$ git cat-file -p deep-tree-hash
040000 tree nested-subtree-hash nested
$ git cat-file -p nested-subtree-hash
100644 blob xyz789... readme.md
```

> **為什麼 Tree 與 Blob 要分開儲存？** 因為 Git 的 Snapshot 是「完整快照」而非差異。如果把整個專案的所有檔案內容都塞進一個 Tree 裡，那麼每次只改一個檔案，整個 Tree 的 hash 都會改變。將 Blob 獨立出來，讓子 Tree 可以共享未被修改的 Blob，節省空間並加速比對。

### 用 git cat-file 查看任意 Tree

使用 `git cat-file -p` 加上特定語法，可以查看任意 commit 對應的 Tree：

```bash
# 查看 master 分支最新 commit 的 Tree
$ git cat-file -p master^{tree}
100644 blob a1b2c3... AUTHORS
040000 tree e7f8a9... fixtures
040000 tree c3d4e5... packages
040000 tree b1a2c3... scripts
```

> **`master^{tree}` 語法**：`^{tree}` 是一個特別的語法，表示「取出這個 commit 物件所指向的 Tree」，而不需要手動複製 commit hash 再查詢。

### Tree 與 Branch 的互動

當你執行 `git checkout` 切換分支時，Git 的底層流程是：

1. 讀取目標 branch 的 commit hash
2. 取出該 commit 指向的 Tree
3. 遞迴遍歷 Tree，根據每個 entry 取回對應的 Blob 內容
4. 在工作目錄重建完整的檔案結構

```
Branch Pointer (e.g., refs/heads/master)
  → Commit Object
    → Tree Object（整個專案結構的根節點）
      → Blob: index.html
      → Blob: main.js
      → Tree: styles/
        → Blob: app.css
```

**這就是 `git checkout <branch>` 能夠瞬間切換整個專案狀態的原理**——Git 只需要依序讀取並重建一組 Tree + Blob 的組合，而不需要真正「複製」任何東西（因為所有 Blob 和 Tree 都已經是不可變地存在 Objects 目錄中了）。

### 檢視 React 這種大型專案的 Tree

以 React 為例，隨便一個 commit 的 Tree 結構就非常龐大：

```bash
$ git cat-file -p master^{tree}
...（非常多的 entry）
040000 tree fixtures-hash  fixtures
040000 tree packages-hash packages
040000 tree scripts-hash scripts
...（更多檔案與目錄）
```

每一個目錄都是一個 Tree，而每個 Tree 都包含指向 Blob（檔案）或 Tree（子目錄）的 entry。這種設計讓 Git 可以用相同的底層機制處理任意規模的專案。

## 💡 重點摘要

- **Tree = Git 描述目錄結構的物件**，每一個 entry 包含：類型（blob/tree）、名稱（檔名或目錄名）、hash（指向實際內容的 Blob 或子 Tree）。
- **Blob 不儲存檔名**，檔名存在 Tree 的 entry 中，這是理解 Git 資料模型的核心概念。
- Tree 的巢狀結構讓 Git 能處理**任意深度的目錄結構**——子目錄也是 Tree，一個 Tree 可以指向另一個 Tree。
- **`git cat-file -p <tree-hash>`** 可以直接查看任意 Tree 的內容，了解專案在某個時間點的完整目錄結構。
- **`master^{tree}`** 語法是查看分支最新 commit 所對應 Tree 的捷徑，不需要手動複製 commit hash。

## 關鍵字

Tree Object, Blob, Tree Entry, SHA-1 Hash, git cat-file -p, master^{tree}, Directory Structure, Recursive Tree, Blob vs Tree, Snapshot, Tree Hash, Object Types
