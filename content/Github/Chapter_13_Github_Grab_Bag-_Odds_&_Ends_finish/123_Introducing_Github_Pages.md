# GitHub Pages：免費靜態網站托管服務

## 📝 課程概述

GitHub Pages 是 GitHub 提供的免費靜態網站托管功能，讓你可以將任何 Repository 的程式碼即時部署成一個可以訪問的網站。本節介紹 GitHub Pages 的兩種形態（User Site 與 Project Site）、使用限制，以及完整的部署流程。我們會看到如何用一個 `gh-pages` 分支來存放網站內容，並透過 GitHub 的 Settings 設定將其上線。

## 核心觀念與實作解析

### GitHub Pages 的兩種形態

GitHub Pages 分為兩種使用模式，了解它們的差異有助於選擇正確的部署策略：

1. **User Site（用戶站點）**
   - 每個 GitHub 帳號**僅限一個**
   - 部署後的 URL 格式：`username.github.io`
   - 適用於個人網站、作品集（Portfolio）
   - 設定方式與 Project Site 略有不同

2. **Project Site（專案站點）**
   - **每個 Repository 都可以有一個**
   - 部署後的 URL 格式：`username.github.io/repo-name`
   - 適用於：專案文件網站、JavaScript 遊戲展示、API 文件、Open Source 專案的文件站
   - 這是大多數人最常使用的模式

> GitHub Pages 特別適合用來托管 **Portfolio 網站**（個人作品集）或 **Open Source 專案的文件網站**。例如 Faker.js 這個開源函式庫，在 GitHub Repository 下方就有一個對應的文件網站，正是透過 GitHub Pages 托管的。

### 使用限制：靜態網站，不支援後端

GitHub Pages 的核心限制是：**它只能托管靜態檔案**。這意味著你只能使用 HTML、CSS 和 JavaScript，**無法執行任何伺服器端語言**（Python、Ruby、Node.js、PHP 等）。

這不是缺點，而是設計上的取捨。靜態網站的好處是：
- 速度極快，無需資料庫查詢
- 部署簡單，只需上傳檔案
- 成本低，適合個人專案和開源專案

但如果你需要：
- 使用者在後端有帳號登入
- 從資料庫讀取資料動態生成頁面
- 有 API 接口處理請求

那你就需要真正的網頁托管服務（如 Vercel、Netlify、Heroku、AWS 等）。

### 完整部署流程：使用專用分支

以下是在現有專案 Repository 中建立 GitHub Pages 網站的完整步驟：

#### 情境設定

假設你有一個名為 `chickens` 的 Repository，目前只有一個 `chickens.txt` 檔案，你想要在同一個 Repository 中另外建立一個網站，但又不想影響原來的專案程式碼。**最佳做法是建立一個專門的分支來存放網站檔案**。

#### 步驟一：在本地建立新分支

```bash
git switch -c gh-pages
```

> 過去 GitHub Pages 會自動認 `gh-pages` 這個分支名稱，現在可以自己指定，不過這個名稱仍然是業界慣例。

#### 步驟二：建立 `index.html`

```bash
# 刪除或保留原有的 chickens.txt 都可以
# 建立網站的首頁檔案（必須命名為 index.html）
touch index.html
```

在 `index.html` 中填入基本的 HTML 內容：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Chickens!</title>
    <link rel="stylesheet" href="app.css">
</head>
<body>
    <h1>Chickens!</h1>
    <p>Welcome to my chicken website.</p>
</body>
</html>
```

#### 步驟三：提交並推送

```bash
git add index.html
git commit -m "add basic HTML for GitHub Pages"
git push origin gh-pages
```

#### 步驟四：在 GitHub 上啟用 GitHub Pages

1. 進入 Repository 的 **Settings**
2. 滾動到最底部，找到 **GitHub Pages** 區塊
3. 在 **Source** 下拉選單中，選擇 `gh-pages` 分支
4. 點擊 **Save**

GitHub 會告訴你網站即將發布的位置：`username.github.io/repo-name`。首次部署可能需要數分鐘（偶爾可能更久，講師提到曾經遇到過 20 分鐘的情況）。

### 部署完成後的更新流程

部署完成後，如果想要更新網站內容，只需在 `gh-pages` 分支上修改檔案，然後 push 即可：

```bash
git add .
git commit -m "update website"
git push origin gh-pages
```

等待幾分鐘後刷新網站，就能看到更新後的內容。

### 關於 main 分支

值得注意的是，你**可以選擇將 main 分支設定為 GitHub Pages 的來源分支**，而不一定要用 `gh-pages`。但前提是 main 分支的根目錄中必須有 `index.html`。如果你的網站剛好就是整個專案的核心呈現（例如一個純 HTML/CSS 的前端作品），直接用 main 分支托管是完全可以的。

另外，你也可以指定 `docs/` 資料夾作為網站內容的來源——如果你想讓網站檔案與主要程式碼分開管理，這是個不錯的選擇。

## 💡 重點摘要

- **GitHub Pages 只支援靜態網站**（HTML/CSS/JavaScript），無法執行伺服器端語言——這是一個功能限制，不是 bug。
- **每個 Repository 都可以有一個 Project Site**，URL 格式為 `username.github.io/repo-name`；每個帳號只有一個 User Site（`username.github.io`）。
- 部署網站的標準做法是**建立一個專用分支（如 `gh-pages`）存放網站檔案**，這樣不會與專案的程式碼分支互相干擾。
- 網站的首頁檔案必須命名為 `index.html`，這是 GitHub Pages 的預設入口。
- 首次部署後，**更新內容只需在對應分支上 push**，GitHub 會自動重新构建網站。

## 關鍵字

GitHub Pages, static website, User Site, Project Site, gh-pages branch, index.html, username.github.io, deploy, hosting, HTML, CSS, JavaScript, documentation site, portfolio website, custom domain, docs folder
