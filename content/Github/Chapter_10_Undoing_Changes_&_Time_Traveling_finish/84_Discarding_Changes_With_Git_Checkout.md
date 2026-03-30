# Discarding Changes With Git Checkout — 還原檔案至 HEAD 狀態

## 📝 課程概述

本節說明如何使用 `git checkout HEAD -- <file>`（以及簡寫 `git checkout -- <file>`）來將 Working Directory 中的檔案**直接還原到上一次 Commit（HEAD）的狀態**，Discarding（拋棄）所有後續的修改。這是一個**危險但有時必要**的操作——它會永久刪除未 Commit 的變更，無法透過 Git 恢復。

## 核心觀念與實作解析

### 語法詳解

```bash
# 完整寫法
git checkout HEAD -- dog.txt

# 簡寫（最常見的用法）
git checkout -- dog.txt

# 同時還原多個檔案
git checkout -- dog.txt cat.txt
```

- `HEAD`：告訴 Git「我要取回 HEAD 指標所指向的那個 Commit 的檔案版本」
- `--`：分隔符號，確保後面的內容被當作檔案路徑而非分支名稱
- `<file>`：你要還原的檔案

### `checkout -- <file>` 做了什麼？

這個指令告訴 Git：**「用 HEAD 上的版本，完全覆蓋指定的檔案」**。

也就是說：
- 如果檔案有未 Commit 的修改，這些修改會**被 HEAD 的版本覆蓋，永久消失**
- 如果檔案在 Staging Area 中，這個指令不會影響 Staging Area 的狀態（只影響 Working Directory）

> **警告**：這是一個**不可逆**的操作。所有未 Commit 的變更都會消失。Git 不會把你丟掉的內容存入 Stash——它們直接被覆蓋掉了。

### 為什麼有這個需求？

在真實開發中，你可能會遇到這樣的情境：

- 不小心在某個檔案裡輸入了一堆亂碼
- 錯誤地刪除了某個重要檔案的內容
- 進行了實驗性修改，後來決定全部不要了

這時，`git checkout -- <file>` 是一個乾淨俐落的方式，讓檔案一鍵回到「上次 Commit 時的樣子」。

### 實作演示

**情境設定**：
- 一個 Repo 有 `cat.txt` 和 `dog.txt` 兩個檔案
- 每個 Commit 都只是在檔案裡追加一行文字（方便對照歷史）
- 目前 HEAD 指向「third commit」（兩個檔案都有 third commit 這行文字）

**誤操作**：
- 打開 `dog.txt`，錯誤地刪除了原有的內容，寫入了一堆乱码
- 執行 `git status`，Git 顯示 `dog.txt` 有 Modification

**還原**：
```bash
git checkout HEAD -- dog.txt
```

執行後：
- `dog.txt` 恢復到 HEAD 的狀態（包含 third commit 的內容）
- 誤操作的內容完全消失，無法恢復
- `cat.txt` 不受影響

### 新語法：`git restore` 的出現

講師預告，下一支影片會介紹一個新的專用指令 `git restore`，它**就是為了取代這個 `git checkout -- <file>` 的用法**而出現的。

```bash
# git restore 是更清晰的語法，用法更直觀
git restore dog.txt
# 等價於
git checkout HEAD -- dog.txt
```

`git restore` 的設計初衷正是因為 `git checkout` 做的事情太多了——它既用來切換分支，也用來還原檔案，也用來進入 Detached HEAD。Git 社群後來認為這些職責應該分開，因此新增了專門負責還原的 `git restore`。

## 💡 重點摘要

- `git checkout HEAD -- <file>`：用 HEAD 的版本**完全覆蓋**指定的檔案，**不可逆**
- 簡寫 `git checkout -- <file>` 中，`HEAD` 是隱含的預設值，所以可以省略
- 這個操作只影響 **Working Directory**，不會改變 Staging Area
- 這個指令**無法恢復**：一旦執行，未 Commit 的變更就消失了
- `git restore <file>` 是較新的語法，功能與 `git checkout HEAD -- <file>` 相同，但語義更清晰
- `--` 是一個分隔符，確保後面的名稱被當作檔案路徑，而非分支名稱

## 關鍵字

git checkout, git checkout HEAD, git checkout --, discard changes, restore file, working directory, HEAD, git status, unstaged changes
