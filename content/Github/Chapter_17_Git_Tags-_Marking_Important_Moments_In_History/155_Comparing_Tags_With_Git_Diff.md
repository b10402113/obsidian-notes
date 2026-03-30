# 檢視與比對 Tags

## 📝 課程概述

本單元說明如何在本機端檢視所有 Tags 以及使用 `git diff` 比較不同 Tag 之間的差異。這些操作讓你能夠快速回顧某個版本發布時的程式碼狀態，並理解不同版本之間究竟發生了哪些變更。

## 核心觀念與實作解析

### 檢視所有 Tags

使用 `git tag`（不帶任何參數）即可列出目前 repository 中所有的 Tags：

```bash
git tag
```

在擁有大量 Tags 的專案（如 React 超過 140 個 Tags）中，這個命令會輸出長長的列表。**離開列表的方式與 `git log` 相同：按 `Q` 鍵。**

```
v3.1.0
v3.1.1
v17.0.0
v17.0.1
...
```

> 我們這裡之所以用 React 專案來練習，正是因為它有足夠多的 Tags，能讓你感受到實際開發中管理大量 Tags 的情境。

### 使用 `-l` 篩選與搜尋 Tags

當 Tags 數量龐大時，可以加上 `-l`（list 的意思）搭配萬用字元（wildcard）進行模式匹配：

```bash
# 列出所有包含 "beta" 的 Tags
git tag -l "*beta*"

# 列出所有以 "v17" 開頭的 Tags
git tag -l "v17*"
```

**注意**：`-l` 參數在搭配 pattern 使用時是必要的，否則 Git 會誤以為你要建立一個新 Tag。如果只是 `git tag` 不帶任何 pattern，`-l` 可以省略，因為這是預設行為。

### 檢出（Checkout）Tag 與 Detached HEAD

就像 Branches 一樣，你可以直接 `git checkout` 到某個 Tag：

```bash
git checkout v15.3.1
```

**但這裡有一個重要的陷阱**：

- Branch 是指向 branch tip 的指標，checkout branch 不會進入 Detached HEAD
- Tag 本身是指向**特定 commit** 的指標，所以 checkout Tag 就等於 checkout 那個 commit

因此，**checkout Tag 會讓你進入 Detached HEAD 狀態**。Git 會顯示警告訊息，告訴你目前不在任何 branch 上。

> 這不是錯誤，而是正常行為。如果你想在舊版本的基礎上繼續工作，只需要從那個 Tag 建立一個新 branch 即可：
> ```bash
> git checkout -b my-branch-from-tag
> ```

### 使用 `git diff` 比較兩個 Tags

比較兩個 Tag 之間的差異，是理解某次發布到底改了什麼的最佳方式：

```bash
# 比較兩個版本的差異
git diff v17.0.0..v17.0.1
```

**實例**：`v17.0.0` 到 `v17.0.1`（一個 Patch 版本），差異極小——只有一個檔案、幾行變更。

**對比實例**：`v16.14.0` 到 `v17.0.0`（一個 Major 版本），則有數百個檔案的大幅變動，這正是 Semantic Versioning 中 Major 版本代表「破壞性變更」的具體呈現。

> 這就是為什麼 Major 版本的發布通常需要團隊撰寫遷移文件——差異真的非常大。

## 💡 重點摘要

- `git tag` 列出所有 Tags；`git tag -l "pattern*"` 進行萬用字元篩選
- **Checkout Tag 會進入 Detached HEAD 狀態**，因為 Tag 等於直接 checkout 某個 commit
- `git diff tag1..tag2` 可以精確看出兩個版本之間的差異，是分析發布內容的利器
- Patch 版本的差異通常極小；Major 版本的差異則可能是翻天覆地的重構

## 關鍵字

git tag, git checkout, Detached HEAD, git diff, wildcard, pattern matching, -l flag, tag filtering, 版本比對
