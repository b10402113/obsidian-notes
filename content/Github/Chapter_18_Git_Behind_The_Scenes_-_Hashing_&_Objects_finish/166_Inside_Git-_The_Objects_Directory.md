# Git Objects 目錄與 SHA-1 演算法

## 📝 課程概述

本單元揭示 Git 儲存資料的核心——Objects 目錄的運作方式，以及 SHA-1 雜湊演算法在 Git 中的關鍵角色。我們將理解 Git 並非儲存「差異（Diff）」，而是儲存**完整快照（Snapshot）**，並認識 Blobs、Trees、Commits、Annotated Tags 四種 Git Object 的基本概念。

## 核心觀念與實作解析

### Objects 目錄——Git 的心臟

在 `.git/objects/` 目錄下，Git 儲存了**倉庫中的所有資料**：每一次 commit、每一個檔案的內容快照、每一個目錄結構的描述，全部都存在這裡。這些都是不可變的、經過壓縮與序列化的二進位檔案。

```bash
.git/objects/
  ├── 1f/        # 以 "1f" 開頭的 hash 的資料夾
  │   └── 7a8... # 實際的 object 檔案（壓縮過的）
  ├── ce/
  ├── fd/
  └── pack/      # 包裝過的 object（節省空間）
```

> **檔案名稱的由來**：SHA-1 hash 有 40 個十六進位字元。Git 取**前兩個字元**當作資料夾名稱，**後 38 個字元**當作檔案名稱。這是因為作業系統對單一目錄下的檔案數量有限制，這樣的設計可以將檔案分散到 256 個可能的子目錄中。

### Git 儲存的是「快照」，不是「差異」

這是理解 Git 最重要的事情之一。與許多版本控制系統不同，**Git 儲存的是專案在每次 commit 時的完整快照，而不是相鄰版本之間的差異（Delta）。**

```bash
# 假設你有一個 1000 行的 JavaScript 檔案
# 版本 1 → 版本 2，只改了一行

# 其他 VCS 的做法：儲存「第 47 行有變動」這個差異
# Git 的做法：直接儲存版本 2 的完整檔案內容快照
```

> **為什麼 Git 選擇快照而不是差異？** 因為 SHA-1 hash 讓 Git 可以瞬間知道「這個檔案內容是否改變」——只要比較 hash 是否相同。如果 hash 相同，Git 完全不需要重新儲存任何東西。這讓很多操作（如分支切換、歷史查詢）的實作大幅簡化，而且現代磁碟空間充足，這樣的設計反而更可靠。

### 四種 Git Object

Git Objects 目錄中只有**四種基本類型**的物件：

| 類型 | 用途 | 儲存什麼 |
|------|------|---------|
| **Blob** | 儲存檔案內容 | 純文字或二進位資料，**不含檔名** |
| **Tree** | 儲存目錄結構 | 指向 blobs 或其他 trees，**含檔名** |
| **Commit** | 儲存一次提交 | 指向一個 tree + parent commit + 作者/訊息 |
| **Annotated Tag** | 標記重要版本 | 指向一個 commit + 額外的中繼資料 |

### SHA-1：Git 的信任基石

**SHA-1（Secure Hash Algorithm-1）** 是一種密碼學雜湊函數，Git 用它對所有物件產生固定長度的唯一識別碼。

```bash
# SHA-1 的輸出特性：固定 40 個十六進位字元
$ echo "hello" | git hash-object --stdin
ce0136dd36c19a...   # 40 個字元

$ echo "hello world a very long file content here..." | git hash-object --stdin
a7f3c2b1d4e5f6...   # 無論輸入多長，輸出仍是 40 個字元
```

**SHA-1 滿足 Git 需要的四個關鍵特性：**

1. **確定性（Deterministic）**：`"hello"` 每次雜湊一定得到相同的 40 字元輸出。這讓 Git 可以立即判斷「這個檔案是否已經變動」。

2. **單向性（One-way）**：從輸出無法反推輸入。如果你將密碼經過 SHA-1 處理後儲存，即使資料庫被竊取，攻擊者也無法反推出原始密碼。

3. **雪崩效應（Avalanche Effect）**：輸入一字之差，輸出天差地遠：
   ```bash
   "I love chickens"  → ce0136...
   "I love chicken"   → 27d547...   # 完全不同！
   ```
   這確保了任何微小的改動都會被偵測到。

4. **低碰撞率**：SHA-1 有 2^160 種可能的輸出，碰撞機率極低，Git 基本上可以為每個內容產生唯一的識別碼。

### SHA-1 的應用場景

在 Git 中，**幾乎所有東西都會被 SHA-1 處理**：

- **Blob hash**：每個檔案內容都被 hash，結果用來命名 .git/objects 下的檔案
- **Tree hash**：目錄結構被 hash，用來唯一識別某個目錄的狀態
- **Commit hash**：包含 tree、parent commit、作者、訊息在內的全部內容被 hash
- **Annotated Tag hash**：標籤本身的內容被 hash

> **值得注意**：即使未來 SHA-1 被正式棄用（Git 開發團隊已有計畫更換），Git 的架構設計讓這種遷移可以在幕後進行，不會影響使用者體驗。

### 用視覺化理解 Git 的資料模型

```
Commit Hash (e.g., a3f8b2c...)
  └── Tree Hash (e.g., c387a1d...)  ← 代表整個專案在某時間點的快照
        ├── Blob: index.html  (hash: 1f7a8c...)
        ├── Blob: main.js     (hash: be3210...)
        ├── Tree: styles/
        │     ├── Blob: app.css  (hash: 10abb9...)
        │     └── Blob: nav.css  (hash: ffe771...)
        └── Tree: images/
              └── Blob: logo.png
```

每一個 commit、tree、blob 都有一個 SHA-1 hash 作為其在 Objects 目錄中的存取 key。這就是 Git 能夠在任何時間點精確還原「哪個檔案的哪個版本」的底層基礎。

## 💡 重點摘要

- `.git/objects/` 是 Git 儲存**所有資料**的地方，包含 Blobs、Trees、Commits 四種 Git Object，全部以 SHA-1 hash 命名。
- **Git 儲存完整快照（Snapshot）**，而非相鄰版本的差異（Delta）——這是理解 Git 與其他 VCS 最大的不同之處。
- SHA-1 讓 Git 具備**確定性與雪崩效應**，能極快速偵測檔案變動，並以 hash 作為取回資料的 key。
- 四種 Git Object 的關係是：**Commit → Tree → Blob/Tree**，形成一個完整的樹狀結構，完整描述了每次提交時的專案狀態。
- SHA-1 hash 的**前兩個字元當資料夾名稱、後 38 個字元當檔案名稱**，這是 Objects 目錄結構的由來。

## 關鍵字

Objects Directory, SHA-1, Blob, Tree, Commit, Annotated Tag, Snapshot vs Delta, Hashing Function, Deterministic, Avalanche Effect, SHA-1 Hash, 40 Hexadecimal Characters, git hash-object, Cryptographic Hash Function
