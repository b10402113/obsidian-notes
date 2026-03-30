# 使用 git cat-file 取回 Git 資料

## 📝 課程概述

本單元延續 Git 作為 Key-Value Data Store 的主題，詳細介紹 `git cat-file` 命令的各種用法。我們將學會如何用 SHA-1 hash 查詢 Git Object 的類型、格式化輸出內容，並透過實際演練理解 Git 如何在底層管理檔案的多個版本快照。

## 核心觀念與實作解析

### git cat-file 的由來

`git cat-file` 是 Git 提供的「plumbing command」之一，專門用來**查詢與讀取 Git Objects 目錄中的物件**。我們已經知道用 `git hash-object` 可以存入資料，而 `cat-file` 就是對應的「取出」操作。

```bash
# 基本語法
git cat-file [-t] [-p] <object>
# -t : 查詢物件類型（type）
# -p : 格式化輸出內容（pretty-print）
```

### -t：查詢物件類型

```bash
# 查詢一個 blob（由 hash-object 建立）
$ git cat-file -t ce0136dd36c19a646a63c2b2a0d56d0a6e4b7c1
blob

# 查詢一個 commit（稍後會詳細討論）
$ git cat-file -t a3f8b2c1d4e5f678901234567890123456789abcd
commit

# 查詢一個 tree（稍後會詳細討論）
$ git cat-file -t c387a1d4e5f678901234567890123456789abcdef
tree
```

`-t` 的輸出非常簡短，只告訴你這是哪一種 Git Object。**這在你不確定某個 hash 代表什麼類型的物件時非常有用。**

### -p：Pretty-Print 格式化輸出

`-p` 會根據物件類型自動格式化輸出，**讓人類可讀**（原本這些檔案都是壓縮過的二進位格式）：

```bash
# 對 blob 使用 -p
$ git cat-file -p ce0136dd36c19a646a63c2b2a0d56d0a6e4b7c1
hello

# 對 tree 使用 -p（會顯示 tree 中的所有 entry）
$ git cat-file -p c387a1d...
100644 blob a1b2c3... index.html
100644 blob d4e5f6... main.js
040000 tree e7f8a9... styles
```

### 不需要完整 hash

Git 足夠聰明，只要給予**足夠 uniquely identify 該物件的字元數**，就能正確取回資料：

```bash
# 給完整 40 字元
$ git cat-file -p ce0136dd36c19a646a63c2b2a0d56d0a6e4b7c1

# 給部分字元（只要能唯一識別就夠）
$ git cat-file -p ce01
hello
```

> **實務上的提示**：一般建議使用 6~8 個字元，在正常的專案歷史中衝突機率極低。但若在大型多人專案或特別密集的 commit 歷史中，可能需要更多字元才能確保唯一性。

### 實務演練：管理檔案的多個版本

這段實作完整展示了 Git 如何在底層管理同一檔案的多個版本快照：

**情境**：我們有一個 `dogs.txt` 檔案，經歷了三個不同版本

```bash
# 版本 1：初始內容
Rusty
Wyatt
$ git hash-object dogs.txt -w
39e275b2c3...   # Blob #1 建立

# 版本 2：新增內容
Rusty
Wyatt
Cheyenne
Sirius Black
$ git hash-object dogs.txt -w
fd9154c2d4...   # Blob #2 建立（不同 hash！）

# 版本 3：修改既有內容
Rusty（已過世）
Wyatt（已過世）
Cheyenne
Sirius Black
$ git hash-object dogs.txt -w
04998a7b1c...   # Blob #3 建立
```

**重點洞察**：版本 2 和版本 3 都包含 `Cheyenne` 和 `Sirius Black` 這兩行，但 Git 並不會聰明地「只存差異」然後重建——實際上每個 blob 都是完整內容。

> **等等，這樣不會浪費空間嗎？** 表面上確實如此。但現代磁碟空間便宜，而 Git 的設計哲學是「以空間換取速度與可靠性」：每次查詢歷史都不需要重建差異，直接取出對應快照即可。而且如果內容真的完全相同（如某一行文字沒變），Git 會聰明地讓兩個 tree 指向同一個 blob，不會重複儲存。

### 刪除工作區檔案，但 Objects 仍然完整

這是理解 Git 持久性的關鍵實驗：

```bash
# 假設 dogs.txt 目前是「版本 3」的內容
$ cat dogs.txt
Rusty（已過世）
Wyatt（已過世）
Cheyenne
Sirius Black

# 我們可以刪除整個檔案
$ rm dogs.txt

# 用版本 1 的 hash 還原檔案
$ git cat-file -p 39e27 > dogs.txt

$ cat dogs.txt
Rusty
Wyatt

# 用版本 2 的 hash 還原檔案
$ git cat-file -p fd91 > dogs.txt

$ cat dogs.txt
Rusty
Wyatt
Cheyenne
Sirius Black
```

> **這就是 `git checkout` 和 `git restore` 的底層原理**——這些高層命令會根據指定的 commit（或 ref）找到對應的 tree，再從 tree 中取出每個 blob 的內容，最終在工作目錄重建完整的檔案狀態。整個過程從未「修改」任何既有的 Git Object，所有快照都是不可變的。

### Objects 的不可變性

一旦資料寫入 `.git/objects`，它就**永久存在且不可改變**（Git 幾乎沒有命令可以修改既有的 object）。這帶來了幾個重要的優點：

| 特性 | 帶來的好處 |
|------|-----------|
| 不可變（Immutable） | 歷史永遠可靠，沒有任何操作可以破壞既有的 commit |
| 可尋址（Addressable） | 每個 commit 都有穩定的 hash，遠端同步不會有版本衝突 |
| 可分享（Shareable） | 同一個 blob 可以被多個 tree 參照，完全不浪費空間 |

### 組合使用：hash-object + cat-file

這兩個命令是 Git 底層儲存系統的完整 API：

```bash
# 一行存入
echo "some content" | git hash-object -w --stdin

# 一行取回
git cat-file -p <hash>
```

只要你不刪除 `.git` 目錄，**這些資料會永久保存**。這也是為什麼 `git reflog`（記錄所有 HEAD 移動軌跡）可以讓你恢復「已經刪除」的 commit——那些 commit 的物件從未真正消失，只是失去了 branch 的參照而已。

## 💡 重點摘要

- `git cat-file -t <hash>` 查詢物件類型（blob / tree / commit / tag），`git cat-file -p <hash>` 格式化輸出物件內容。
- **Git 只需要 hash 的前幾個字元就能唯一識別物件**，在正常專案中使用 6~8 個字元就足夠。
- 每一次檔案變動都會產生一個**新的 blob**，但 Git 不會重複儲存相同內容的 blob——完全相同的內容共享同一個 blob hash。
- 工作目錄的檔案可以隨意刪除，只要 `.git` 目錄還在，就能用 `git cat-file` + hash 還原任意歷史版本。
- Git Objects 的**不可變性**是整個版本控制系統可靠性與安全性的基石——過去的 commit 永遠不可能被修改。

## 關鍵字

git cat-file, -t flag, -p flag, Pretty Print, Blob, SHA-1 Hash, git hash-object, Immutable Objects, Snapshot Restoration, Plumbing Commands, git checkout, Partial Hash Resolution
