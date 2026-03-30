# Git Restore — 還原檔案與取消 Staging 的現代化指令

## 📝 課程概述

本節介紹 Git 中一個相對較新的專用指令 `git restore`。它的出現是為了分擔 `git checkout` 過於龐大的職責——`git restore` 只專注於兩件事：**還原 Working Directory 中的檔案**以及**取消 Staging Area 中的檔案**。這個指令語義清晰，比 `git checkout` 更不容易造成混淆，是現代 Git 使用的推薦方式。

## 核心觀念與實作解析

### 為什麼需要 `git restore`？

`git checkout` 這個指令肩負了太多職責：
- 切換分支
- 進入 Detached HEAD
- 還原檔案至 HEAD 狀態

這讓 `checkout` 變得非常「多才多藝」，但也因此容易混淆。Git 新增 `git restore` 和 `git switch` 這兩個專用指令，來讓每個指令**只做一件事，並把那件事做好**。

### `git restore` 的兩大功能

#### 功能一：還原 Working Directory 的檔案

```bash
git restore <file>
```

這個指令等價於 `git checkout HEAD -- <file>`，但語義更加直觀——「把這個檔案還原回來」。

**情境**：
- 你修改了 `dog.txt`，但這些修改全部是錯誤的，想全部放棄
- `git restore dog.txt` → 檔案立刻恢復到 HEAD 的狀態，所有未 Commit 的修改消失

**預設還原目標**：`HEAD`（最新 Commit）。如果你想還原到更早的 Commit，需要使用 `--source` 選項。

#### 功能二：取消 Staging（將檔案移出 Staging Area）

```bash
git restore --staged <file>
```

這個指令的用途是：**把已 `git add` 的檔案取消 Staging**，讓它回到「未 Staged」的狀態，但**不刪除 Working Directory 中的任何變更**。

**情境**：
- 你意外 `git add secrets.txt`（包含 API Key 和信用卡號的檔案）
- 想把這個檔案從 Staging Area 移除，但不希望刪除 Working Directory 中的內容
- `git restore --staged secrets.txt` → 檔案從 Staging Area 移出，但 `secrets.txt` 仍然存在於 Working Directory

> **關鍵理解**：`git restore --staged` 不會刪除你的檔案，也不會刪除你的修改。它只是讓 Git「不再追蹤」這個檔案的這次變更，讓它不會被包含在下一個 Commit 中。

#### 指定還原到特定 Commit：`--source` 選項

```bash
git restore --source=HEAD~2 dog.txt
```

這個語法讓你可以把檔案還原到**任意歷史 Commit** 的狀態，而不僅限於 HEAD。

**情境**：
- 你在三個 Commit 前刪除了某個功能，現在想把它們找回來
- `git restore --source=HEAD~3 cat.txt` → 把 `cat.txt` 還原到 HEAD 往前第三個 Commit 的狀態

> **警告**：這個操作會**覆蓋**目前 Working Directory 中的所有未 Commit 變更，且無法恢復。如果不確定，先用 `git stash` 暫存當前工作。

### `--source` 預設值：HEAD

如果省略 `--source`，Git 預設使用 `HEAD` 作為來源：

```bash
git restore dog.txt
# 等價於
git restore --source=HEAD dog.txt
```

### 透過 `git status` 獲取提示

講師特別強調的一個技巧：`git status` 在你需要時會**自動提示**你該用哪個指令。

```
# 當你有未 Staged 的修改時：
Changes not staged for commit:
  (use "git restore <file>..." to discard changes in working directory)

# 當你有已 Staged 的修改時：
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
```

也就是說，你不需要把這些語法全部背起來——只要 `git status`，Git 就會告訴你下一步該怎麼做。

## 💡 重點摘要

- `git restore <file>`：將 Working Directory 中的檔案還原至 HEAD 狀態（等價於 `git checkout HEAD -- <file>`）
- `git restore --staged <file>`：將檔案從 Staging Area 移出，**不刪除 Working Directory 中的變更**
- `git restore --source=<commit> <file>`：將檔案還原到指定 Commit 的狀態，預設來源是 `HEAD`
- `--source` 預設值是 `HEAD`，所以最常見的用法只需要 `git restore <file>`
- `git status` 會自動提示你何時該用 `git restore`，不需要死記硬背
- 這個指令的設計目的就是讓還原操作**語義更清晰、不容易誤用**

## 關鍵字

git restore, git restore --staged, git restore --source, working directory, staging area, discard changes, unstage, HEAD, git status
