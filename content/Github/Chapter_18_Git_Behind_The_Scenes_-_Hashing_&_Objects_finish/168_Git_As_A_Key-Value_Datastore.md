# Git 作為 Key-Value Data Store

## 📝 課程概述

本單元以最底層的視角重新認識 Git——它不只是一個版本控制工具，本質上是一個**可以儲存任意資料的 Key-Value Data Store**。我們將手動操作 `git hash-object` 與 `git cat-file` 這兩個「管線命令（Plumbing Commands）」，體會 Git 如何將任何資料轉換成 SHA-1 key 並存入 Objects 目錄。

## 核心觀念與實作解析

### Git 的雙重身份

大多數人認識 Git，是因為它能追蹤程式碼的歷史、支援多人協作。但從底層來看，Git 的核心運作其實非常簡單：**把資料存進去，之後用一個 key 把資料取出來。**

> **為什麼這麼重要？** 因為當你理解 Git 是一個 Key-Value Data Store 之後，所有高層操作（commit、branch、checkout、rebase）的原理都會變得非常清晰——它們只不過是在操作 key、用 key 取資料、用 key 建立關聯。

### git hash-object——存入資料

`git hash-object` 接收任意資料（字串或檔案），使用 SHA-1 雜湊，產生一個 40 字元的 key。

```bash
# 單純計算 hash（不寫入 objects 目錄）
$ echo "hello" | git hash-object --stdin
ce0136dd36c19a646a63c2b2a0d56d0a6e4b7c1

# 加上 -w（write），真正寫入 objects 目錄
$ echo "hello" | git hash-object -w --stdin
ce0136dd36c19a646a63c2b2a0d56d0a6e4b7c1
```

`-w` 參數告訴 Git：「**真的**把這個資料存進去，不要只是算個 hash 給我看。」執行後你會在 `.git/objects/` 下看到新生成的檔案：

```bash
$ ls .git/objects/
ce/                     # 前兩個字元 → 資料夾
ce0136dd36c...          # 後 38 個字元 → 檔案名稱
```

> **為什麼 SHA-1 輸出要被拆成資料夾 + 檔案名？** 作業系統對單一目錄下的檔案數量有上限（約數萬個，視檔案系統而定）。Git 用 hash 的前兩個字元建立資料夾，可以把檔案分散到最多 256 個資料夾中，避免達到目錄上限。

### 同一內容，永遠同一 key

這是 SHA-1 的確定性保證：

```bash
# 執行一千次，永遠得到相同的 hash
$ echo "hello" | git hash-object --stdin
ce0136dd...  (每次相同)

$ echo "goodbye" | git hash-object --stdin
dd7e1b2c...  (不同於 hello 的 hash)
```

這意味著：**只要內容沒變，Git 就不需要儲存重複的資料。** 如果同一個檔案出現在兩個 commit 中，Git 只會儲存一個 blob，兩個 commit 的 tree 都會指向這個 blob。

### 資料大小無關輸出長度

無論你輸入多短的字串還是多巨大的檔案，SHA-1 永遠只輸出 40 個十六進位字元：

```bash
# 短字串
$ echo "hi" | git hash-object --stdin
8fec5a0b2f1d3e...  (40 字元)

# 整個 React 原始碼庫（數百 MB）
# 仍然只輸出 40 個字元的 hash
$ git hash-object react-repo/main.js
3f7c2a9b1d4e5f6...  (40 字元)
```

> **實務上的意義**：Git 不會因為檔案變大而需要更長的識別碼，也不會因為檔案變小而需要更短的。固定長度的 key 讓 Git 的內部實作極度簡潔。

### git cat-file——取回資料

有存就要能取。`git cat-file` 讓你用一個 hash（完整或部分）從 Git 的資料庫中取回對應的原始內容：

```bash
# 用完整的 key 取出資料
$ git cat-file -p ce0136dd36c19a646a63c2b2a0d56d0a6e4b7c1
hello

# 用前幾個字元就足夠（Git 會自動搜尋匹配）
$ git cat-file -p ce01
hello
```

`-p` 代表「pretty-print」，Git 會根據物件類型格式化輸出。

你也可以查詢一個物件的類型：

```bash
$ git cat-file -t ce0136dd36c19a646a63c2b2a0d56d0a6e4b7c1
blob
```

### 手動演練：儲存與取回檔案的多個版本

這個流程完整展示了 Git 的核心運作：

**步驟 1：儲存檔案的第一個版本**

```bash
$ cat dogs.txt
Rusty
Wyatt

$ git hash-object dogs.txt -w
39e275b2...   # Git 產生 blob 並寫入 objects 目錄
```

**步驟 2：修改檔案後儲存第二個版本**

```bash
$ cat dogs.txt
Rusty
Wyatt
Cheyenne
Sirius Black

$ git hash-object dogs.txt -w
fd9154c2...   # 完全不同的 hash（新內容 → 新 hash）
```

**步驟 3：取回任意版本**

```bash
# 取回第一個版本
$ git cat-file -p 39e27
Rusty
Wyatt

# 取回第二個版本
$ git cat-file -p fd91
Rusty
Wyatt
Cheyenne
Sirius Black
```

> **重點來了**：即使你刪除了 `dogs.txt` 檔案，只要 `.git` 目錄還在，Git 就能用 hash 取回任意一個版本。這就是 Git commit、branch、checkout 功能的底層原理——只不過這些高層命令幫你自動完成了整個流程。

### Blob——Git 第一種 Object

我們剛才用 `git hash-object` 建立的，就是 Git 的第一種 Object：**Blob（Binary Large Object）**。

Blob 的定義非常簡單：**只儲存檔案的內容，不包含檔案名稱。**

```bash
# 假設一個 blob 的 hash 是 "39e275..."
# 這個 blob 儲存的是 "Rusty\nWyatt\n" 這個純文字內容
# 它**不**儲存 "dogs.txt" 這個檔名
```

檔案名稱存在 Tree 物件中（我們下一個單元會深入討論），Blob 只負責儲存「內容」。

### 為什麼 Git 選擇將內容與檔名分開儲存？

這個設計讓 Git 極度高效：相同內容的檔案，即使出現在不同資料夾、不同專案中，都只會有一個 blob。例如：
- `src/utils/helpers.js`（內容 hash: abc123...）
- `lib/shared/utils.js`（內容 hash: abc123...）

Git 只需要儲存**一個 blob**，兩個 tree 分別指向同一個 blob 即可。這是 Git 能快速處理大型專案的關鍵優化之一。

## 💡 重點摘要

- **Git = Key-Value Data Store**：你可以存入任何資料，Git 以 SHA-1 hash 作為 key 並回傳給你，之後用同一個 key 就能取回資料。
- `git hash-object -w` 是**寫入**資料的命令，`git cat-file -p` 是**讀取**資料的命令，兩者構成 Git 底層儲存系統的核心 API。
- SHA-1 的**確定性**（相同輸入 → 相同輸出）讓 Git 可以瞬間偵測檔案是否變動，並自動避免儲存重複內容。
- **Blob 只儲存檔案內容，不含檔名**，檔名由 Tree 物件管理，這是 Git 高效利用空間的關鍵設計。
- 即使完全沒有 commit，只要 `.git` 目錄存在，你仍然可以用 `git cat-file` 取回手動存入的任意資料。

## 關鍵字

Key-Value Data Store, git hash-object, git cat-file, SHA-1 Hash, Blob, -w flag, --stdin, Pretty Print, Plumbing Commands, Deterministic, Object Storage, .git/objects, Hash Collision
