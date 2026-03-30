# Reflog 的限制與 git reflog 命令實戰

## 📝 課程概述

本單元先強調兩個關於 Reflog 的**前提限制**（本地性、過期性），再正式介紹 `git reflog` 命令本身。你會學會用這個命令讀取 reflog 資料，並理解它與 `git log` 在記錄範圍上的根本差異——特別是那些被 `reset` 或 `rebase` 抹去的 commit，Reflog 仍有可能為你找回來。

## 核心觀念與實作解析

### 兩大前提限制：Local 與 Ephemeral

在深入使用 Reflog 之前，必須先搞清楚它**不是什麼**，否則你可能會在最關鍵的時刻發現它根本幫不上忙。

**限制一：Reflog 是 Local 的**

假設你在一個有上千位貢獻者的專案（如 React）工作，你擁有的 reflog **只屬於你自己**。Git 不會、也不可能把每個貢獻者每分鐘的 HEAD 移動都同步給你——那樣的資料量會讓人崩潰。

> **所以：團隊合作時，別人的失誤（誤刪 commit、錯誤 rebase）是無法靠你的 Reflog 救回來的。** 每個人的 Reflog 只記錄自己機器上的操作。

**限制二：Reflog 會過期**

根據 Git 官方文件，預設情況下舊的 reflog  entry 大約在 **90 天後會被清除**。Git 會自動執行垃圾回收，把那些「太舊而且已經沒人指向」的 reflog 記錄刪掉。

> **千萬不要把 Reflog 當成永久的 commit backup。** 如果一個月前你誤刪了 commit、卻一直沒處理，到時候連 reflog 都救不回來了。真正重要的 commit，請靠 `git tag` 或推送到遠端來保護。

---

### git reflog 命令結構

`git reflog` 這個命令有四個 subcommand：

| Subcommand | 用途 | 一般使用者需要嗎？ |
|---|---|---|
| `show` | 顯示 reflog 內容 | ✅ 主要工具 |
| `expire` | 手動清除舊 entry | ❌ 不需要 |
| `delete` | 刪除特定 entry | ❌ 不需要 |
| `exists` | 檢查某 reference 是否有 reflog | ❌ 不需要 |

講師特別說明，他不是偷懶不教後三個——而是官方文件也明確指出它們「typically not used directly by end users」。所以我們只需要專注在 `git reflog show`。

---

### git reflog show 的基本用法

```bash
git reflog show HEAD
```

執行後，你會看到一個與 `git log` 外觀相似、但內容截然不同的輸出。每一行代表 HEAD 的一次移動，最上方是最新的 entry，向下依序是較舊的記錄。

**一個重要的示範**：講師在 React 專案中曾經做過 `git reset`，把一個 commit 刪掉了。執行 `git log` 完全看不到那個 commit 的蹤影，但 `git reflog show HEAD` 中仍然保留了那個 commit 的 hash。

> **這個對比就是 Reflog 最核心的價值：`git log` 只顯示「仍然活著」的 commit graph，而 reflog 顯示「你曾經走過的所有足跡」。**

---

### Reflog Entry 的識別方式

Reflog 中的每一筆 entry 都有自己專屬的識別名稱，語法是：

```
HEAD@{number}
```

- `HEAD@{0}` = HEAD 最近一次移動
- `HEAD@{1}` = 再上一次移動
- `HEAD@{56}` = 從現在往前數第 56 步（最老的 entry）

> **這些數字是動態的，不是固定不變的。** 每當你做一個新的操作（如切換分支），原本的 `HEAD@{0}` 就變成 `HEAD@{1}`，`HEAD@{1}` 變成 `HEAD@{2}`，以此類推。所以千萬不要把這些數字當成 commit ID 來記憶——它們只是 reflog 中的位置索引。

---

### 查看任意 Reference 的 Reflog

不只是 HEAD，任何 branch reference 都有屬於自己的 reflog：

```bash
git reflog show <branch-name>
# 例如：
git reflog show donkey
```

**Branch tip reflog 與 HEAD reflog 的差異**：

- **HEAD reflog**：每次切分支、做 commit、reset、rebase 都會更新——只要 HEAD 動了，就有一筆記錄
- **Branch tip reflog**：只記錄「該分支的 tip（最新 commit）移動」的時刻。**切換分支本身不會產生分支 reflog 的新 entry**（因為切分支並不影響那個分支的 tip）

講師演示：當他在 `donkey` 分支做 `echo "hahah" > haha.txt` → `git add .` → `git commit -m "add haha"` 後，`git reflog show donkey` 從兩筆記錄變成三筆——多出來的那筆就是剛才 commit 造成的 tip 前進。

## 💡 重點摘要

- **Reflog 只記錄「你自己」在「本地機器」的操作，團隊夥伴的失誤無法靠你的 Reflog 補救。**
- **Reflog 會在約 90 天後自動過期，千萬別把它當成永久的 commit 備份。**
- **`git reflog show <reference>` 是我們唯一需要學習的 subcommand，可以查看任意 reference 的 reflog 內容。**
- **`HEAD@{n}` 語法用於引用 reflog 中第 n 步的 entry，且這些數字會隨新的操作而遞增（動態的）。**
- **被 `reset` 或 `rebase` 抹去的 commit 會從 `git log` 消失，但仍然留在 reflog 中——這就是搶救的關鍵。**

## 關鍵字

`git reflog show`、local only、90 days expiration、`HEAD@{n}`、branch tip reflog、reset recovery、rebase undo、`git log` vs `git reflog`、ephemeral logs
