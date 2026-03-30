# Git 內部運作機制導論

## 📝 課程概述

本單元帶領學員「掀開 Git 的引擎蓋」，一窺 `.git` 目錄的內部結構。我們會從設定檔（config）切入，逐步理解 refs、HEAD、Objects 目錄等核心元件的運作原理，並學習 Git 用來保證資料完整性與可追蹤性的底層技術——SHA-1 Hashing。這些知識不會讓你立刻寫出更漂亮的 commit 訊息，但能讓你成為一個對 Git 有「深度理解」的開發者。

## 核心觀念與實作解析

### 從 .git 目錄開始

`.git` 是每個 Git 倉庫的心臟，裡面儲存了 Git 運作所需的一切資料。在這一章節中，我們會逐步探索其中幾個關鍵檔案與目錄。

### 設定檔（config）——最實用的起點

在 `.git/config` 檔案中，存放著**僅作用於當前倉庫的本地設定**。相較於全域設定（`~/.gitconfig`），這裡的設定只影響這個 repo，不會外溢到其他專案。

> **為什麼這很重要？** 因為在團隊合作中，你可能需要針對不同專案使用不同的使用者名稱、編輯器，甚至不同的分支命名規則——本地設定就是為此而生。

你可以用命令列直接修改它，例如：

```bash
git config --local user.name "chicken little"
git config --local user.email "chicken@gmail.com"
```

也可以直接手動編輯這個檔案。格式看起來有點陌生，但 Git 文件說明了一切。

### 顏色設定——讓終端機更易讀

這裡還藏著一個有趣的彩蛋：**Git 的顏色設定**。你可以在 config 中調整 `git diff`、`git branch`、`git status` 等命令的輸出顏色。

```ini
[color]
    ui = true
[color "branch"]
    current = yellow bold
    local = cyan bold
[color "diff"]
    old = magenta bold
    new = green
```

你可以把 `git branch` 的輸出從預設的綠色（目前分支）與黑色（其他分支），改成自己順眼的配色。這純粹是美觀需求，但長期使用終端機的開發者會發現它能有效提升閱讀效率。

### refs 目錄——Branch 與 Tag 的真面目

在 `.git/refs/` 目錄下，**每一個 branch 與 tag 都是一個檔案**，檔案名稱就是分支或標籤的名稱，而檔案內容只有一行：某個 commit 的 SHA-1 hash 值。

也就是說，當你在圖示中看到「分支指標指向一個 commit」時，實際上 Git 只是在 `refs/heads/分支名稱` 這個檔案裡存了一個 hash。

```
.git/refs/heads/
  ├── master      # 包含一個 commit hash
  ├── new-branch  # 包含另一個 commit hash
  └── chicken     # 新建的分支也有自己的檔案
```

這個目錄底下還有：
- **`refs/heads/`**：本地分支
- **`refs/tags/`**：所有標籤
- **`refs/remotes/`**：remote 追蹤分支（如 `origin/master`）

### HEAD 檔案——Git 怎麼知道「現在在哪」？

`.git/HEAD` 是一個純文字檔，**指向你目前所在的分支（或 commit）**。

- **正常狀態**：HEAD 檔案內容是 `ref: refs/heads/分支名稱`，也就是「參考某個 branch 檔案」，再由那個 branch 檔案指向 commit。這是一層間接參照（reference to a reference）。
- **Detached HEAD 狀態**：如果你執行 `git checkout <hash>`，HEAD 檔案會直接寫入一個 commit hash，不再指向 branch。這就是「游離的 HEAD」——離開了安全錨點。

```bash
# 正常狀態的 HEAD 檔案內容
ref: refs/heads/master

# Detached HEAD 時的內容（直接是 hash）
a3f8b2c1d4e5...
```

### Objects 目錄——Git 的核心心臟

這是 Git 儲存**所有資料**的地方，包括：Blob、Tree、Commit、Annotated Tag 四種 Git Object。這些物件全部以 SHA-1 hash 命名，並以「前兩碼」作為子資料夾名稱、「後 38 碼」作為檔案名稱儲存。

```bash
.git/objects/
  ├── ce/        # 以 ce 開頭的 hash
  │   └── 0136...   # 後 38 碼是檔案名稱
  ├── fd/
  │   └── 915...    # fd915... 這個 hash 代表的檔案
  └── info/
  └── pack/
```

> **為什麼要分資料夾？** 作業系統對同一目錄下的檔案數量有限制。將 hash 的前兩碼抽出來當資料夾名稱，可以大幅分散儲存，避免效能問題。

### SHA-1 Hashing——Git 的信任基石

Git 使用 SHA-1 雜湊函數，**對任意大小的輸入產生固定 40 個十六進位字元（160 bits）的輸出**。

```bash
# 不管輸入多短或多長，輸出永遠是 40 個十六進位字元
$ echo "hello" | git hash-object --stdin
ce0136...        # 40 個字元
$ echo "hello world, this is a very long string..." | git hash-object --stdin
a7f3c2...        # 仍然 40 個字元
```

**為什麼 SHA-1 對 Git 至關重要？**

| 特性 | 對 Git 的意義 |
|------|--------------|
| 確定性（Deterministic） | 相同內容一定產生相同 hash → Git 可快速判斷檔案是否變動 |
| 單向性（One-way） | 從 hash 無法反推內容 → 儲存的內容無法被猜測 |
| 雪崩效應（Avalanche） | 輸入一字元之差，輸出天差地遠 → 小改變一定會被偵測到 |
| 低碰撞率 | 碰撞機率極低 → 每個物件幾乎都有唯一識別碼 |

### Git 作為 Key-Value Data Store

這是理解 Git 底層的關鍵：**Git 本質上是一個可以儲存任意資料、並以 SHA-1 hash 作為 key 來取回資料的資料庫。**

```bash
# 儲存資料（不實際寫入 .git/objects）
echo "hello" | git hash-object --stdin
# → 輸出 ce0136... (key)

# 儲存並寫入 objects 目錄
echo "hello" | git hash-object -w --stdin
# → 寫入 .git/objects/ce/0136...

# 用 key 取回資料
git cat-file -p ce0136
# → hello
```

只要持有那個 40 字元的 hash，就能從 Git 的資料庫中取出對應的原始內容。這就是日後 commit、branch、checkout 等操作能夠正常運作的底層基礎。

## 💡 重點摘要

- `.git/config` 是**本地倉庫專屬**的設定檔，優先於全域設定，適合對單一專案做客製化調整。
- **Branch 與 Tag 本質上就是 refs 目錄下的檔案**，內容只有一個 commit hash，沒什麼魔法。
- **HEAD 檔案**用來告訴 Git「你現在在哪個 commit 或分支上」，Detached HEAD 就是繞過 branch 直接 checkout 到 commit。
- Git Objects 目錄儲存了 Git 所有資料，包括 Blobs、Trees、Commits 四種物件，全部以 SHA-1 hash 命名。
- **SHA-1 是 Git 的信任基石**：相同輸入 → 相同輸出，讓 Git 能極快速偵測檔案變動，並以 hash 作為資料庫的 key。

## 關鍵字

config, refs, HEAD, refs/heads, refs/tags, refs/remotes, SHA-1, Hashing, Objects Directory, Detached HEAD, Key-Value Data Store, git hash-object, git cat-file
