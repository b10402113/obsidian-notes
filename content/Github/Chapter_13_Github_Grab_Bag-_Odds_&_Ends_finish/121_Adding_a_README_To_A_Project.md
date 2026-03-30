# 實作 README.md 與 GitHub Gists 介紹

## 📝 課程概述

本節分為兩個部分：首先，我們實際在本地建立一個 README.md 檔案，並將其推送至 GitHub Repository，親眼見證 GitHub 自動渲染 README 的效果。其次，我們介紹 GitHub Gists——一個輕量級的程式碼片段分享工具，適用於不想建立完整 Repository 的簡單分享場景。

## 核心觀念與實作解析

### 在本地建立 README 的完整流程

雖然 GitHub 提供了在網頁上直接建立檔案的功能，但講師選擇在本地建立 README，這是更推薦的做法。以下是完整的操作流程：

**第一步：確保本地處於最新狀態**

在任何新的變更之前，先執行一次 pull 是好習慣，可以避免之後產生合併衝突：

```bash
git pull origin main
```

**第二步：建立 README.md 檔案**

```bash
touch README.md
```

接著在編輯器中加入內容。例如：

```markdown
# Bookish Disco

**Bookish Disco** 是一個由 GitHub 隨機生成的 Repository 名稱。本專案不做任何實際用途，純粹作為 Git/GitHub 課程的示範專案。
```

**第三步：提交並推送**

```bash
git add README.md
git commit -m "add README file"
git push origin main
```

完成後，刷新 GitHub Repository 頁面，你會看到兩件事同時發生：

1. **Repository 檔案列表中出現了 `README.md`**（這是正常的檔案）
2. **Repository 首頁自動渲染了 README 的內容**（GitHub 額外為這個檔名做的特殊處理）

這就是為什麼 README.md 是特殊的——GitHub 會「認得」這個檔名並將其內容直接顯示在首頁，而不需要點進去查看。

### README 不只是給技術人員看的

講師特別提到一個被很多人忽略的事實：** recruiter（招募人員）這類非技術背景的人，通常第一眼看到的就是 README**。他們不會去讀你的程式碼，但 README 提供了一個「理解的入口」，讓他們至少能理解專案的目的與價值。這使得 README 成為一種**自我展示的工具**，不僅是技術文件。

### GitHub Gists：輕量級程式碼分享

當你需要分享一小段程式碼，但不想建立完整的 Repository 時，GitHub Gists 就是最佳選擇。它的使用場景包括：

- 在論壇或社群中分享程式碼片段
- 儲存自己常用的設定檔或腳本
- 教學時分享練習範例
- 讓他人 Fork 並討論你的程式碼

#### Gists 的核心特性

1. **支援多種檔案類型**：Gists 可以是任何副檔名的檔案，GitHub 會根據副檔名自動套用語法高亮。例如 `.md` 會渲染成 Markdown，`.js` 會有 JavaScript 高亮。
2. **支援多檔案**：一個 Gist 可以包含多個檔案。
3. **兩種隱私設定**：
   - **Secret Gist**：不會出現在 Discovery 頁面，只有知道 URL 的人能存取，適合私人備忘錄或設定檔。
   - **Public Gist**：會出現在 GitHub 的 Discovery 頁面，任何人都能搜尋和發現。
4. **具備基本 Git 功能**：Gists 底層其實是 Git Repository，因此支援 Fork、Star、Revision 歷史、Comment 等功能，但介面被大幅精簡。
5. **可用於組織內部共享**：即使不是公開的 Gist，也可以在公司內部與同事分享、討論。

#### 為什麼選擇 Gist 而不是 Repository？

| 情境 | 選擇 |
|------|------|
| 想分享一個設定檔或一小段腳本 | Gist |
| 想做一個完整的專案 | Repository |
| 想記錄自己的筆記或教學範例 | Gist |
| 需要多人長期共同維護一個專案 | Repository |
| 想快速在 Reddit 或論壇分享程式碼 | Gist |

> 建立一個完整的 Repository 需要初始化、設定 remote、處理分支——有時候你只是想分享兩行指令，這時候 Gist 就是最輕巧的選擇。

## 💡 重點摘要

- **建立 README.md 的正確流程**：本地建立 → `git add` → `git commit` → `git push origin main` → GitHub 自動渲染顯示在首頁。
- **提交前先 `git pull`** 可以避免不必要的合併衝突，這是任何多人協作專案中的好習慣。
- **README 是專案對外的門面**，即便是非技術背景的人（recruiter、老闆）也會透過 README 來評估你的專案。
- **GitHub Gists** 是分享程式碼片段的輕量工具，適用於不想建立完整 Repository 的場景，支援 Secret 與 Public 兩種模式。
- Gists 底層是 Git Repository，因此具備 Revision 歷史和 Fork 等功能，但介面比標準 Repository 精簡得多。

## 關鍵字

README.md, git add, git commit, git push origin main, git pull origin main, GitHub auto-render, Gist, Secret Gist, Public Gist, code snippet, syntax highlighting, Fork, Star, Revision history, discovery page
