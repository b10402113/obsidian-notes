# Creating Our First GitHub Repo & Remotes — 建立 GitHub 倉庫與連接本地 Repo

## 📝 課程概述

本節介紹兩件事：**如何從 GitHub 建立新的空倉庫**，以及**如何將本地已有的 Repo 連接到 GitHub**。這是使用 GitHub 的第一步——建立連接。本節的重點是理解 `git remote` 的概念：它只是給 URL 起一個方便記憶的名字（通常是 `origin`），讓後續的 Push、Fetch、Pull 操作更簡潔。

## 核心觀念與實作解析

### 建立空 GitHub 倉庫

在 GitHub 網站上建立新 Repo 的步驟：
1. 點擊右上角 **+** 號 → **New repository**
2. 填寫 Repository 名稱（建議與本地專案名稱一致，但不強制）
3. 選擇 **Public**（公開）或 **Private**（私人）
4. **不要**勾選「Initialize this repository with a README」（因為你已有本地 Repo）
5. 點擊 **Create repository**

> **重要**：如果你的本地 Repo 已經有一些 Commit 和歷史，直接 Clone 從 GitHub 建立 Repo 的選項**不要勾選**，否則會產生衝突。

### `git remote`：將 URL 變成方便記憶的名字

**Remote 是什麼？** Remote 就是一個 URL 的「標籤」——讓你不必每次都輸入完整的 URL。

```bash
# 查看目前已設定的 Remote
git remote -v

# 新增 Remote（origin 只是慣例名稱，可以改成任何你喜歡的）
git remote add origin https://github.com/username/repo-name.git
```

**`origin` 只是慣例，不是特殊關鍵字**：
- `origin` 就像 `master` 分支名稱——只是預設值，不具任何神奇力量
- 你可以叫它 `github`、`my-server` 或任何名稱
- 教學與業界慣例使用 `origin`，建議你也這樣做

### `git remote -v`：查看 Remote 設定

```bash
git remote -v
# 輸出：
# origin  https://github.com/username/repo-name.git (fetch)
# origin  https://github.com/username/repo-name.git (push)
```

`-v` 是 `--verbose` 的縮寫，會顯示：
- Remote 的名稱（`origin`）
- 對應的 URL（fetch 與 push 的 URL，通常相同）

### 為什麼 Clone 出來的 Repo 已經有 Remote？

當你執行 `git clone <url>` 時，Git 會**自動**幫你設定好 Remote：
- Remote 名稱設為 `origin`
- URL 設為你 Clone 的那個 URL

但如果是**自己用 `git init` 建立的 Repo**，就沒有任何 Remote，需要手動用 `git remote add` 設定。

### 其他 Remote 管理指令

```bash
# 重新命名 Remote
git remote rename old-name new-name

# 刪除 Remote
git remote remove origin
```

### 兩種連接本地與 GitHub 的工作流

**方式一（適合已有本地 Repo）**：
1. 在 GitHub 建立空的 Repo
2. 本地執行 `git remote add origin <GitHub-URL>`
3. 執行 `git push origin master` 推送

**方式二（適合從零開始）**：
1. 在 GitHub 建立空 Repo
2. Clone 到本機（自動設定 Remote）
3. 開始在本機工作
4. `git push origin main` 推送

> **教學提示**：講師在影片中特別強調，方式一適合「本地已有程式碼」的情境（正是這個課程中的大多數 Repo）；方式二適合「從零開始新專案」的情境。兩者各有用途，不必選邊站。

## 💡 重點摘要

- `git remote add origin <url>`：將 GitHub 倉庫的 URL 與「origin」這個名字關聯起來
- `origin` 只是慣例名稱，和 `master` 分支名稱一樣，不具特殊意義
- 用 `git init` 建立的 Repo 沒有 Remote，需要手動設定；用 `git clone` 的 Repo 自動有 Remote
- `git remote -v` 顯示所有 Remote 的名稱與 URL
- 設定 Remote 只是「記住 URL」，真正傳輸資料的是 `push`、`fetch`、`pull` 指令

## 關鍵字

git remote, git remote add, origin, git remote -v, git remote rename, git remote remove, remote URL, local repo, GitHub repository
