# Bind Mounts 實作：Jekyll 靜態網站開發

## 📝 課程概述

本單元透過 **Jekyll** 靜態網站產生器，實際演練 Bind Mounts 在本地開發的應用。我們將體驗「在 Host 編輯 Markdown 檔案，Container 內的 Web Server 即時反映變更」的開發流程，這正是 Bind Mounts 最強大的使用場景。

---

## 核心觀念與實作解析

### 為什麼選擇 Jekyll 作為範例？

**Jekyll** 是一個 Static Site Generator (SSG)，它的特色是：

- 用 **Markdown** 寫內容（基本上就是純文字）
- Jekyll 自動將 Markdown 轉換成 HTML 網站
- 適合部落格、簡單網站，不需要成為 HTML 專家
- **GitHub Pages** 背後使用的就是 Jekyll

> 這個範例要示範的不是網頁開發，而是 Bind Mount 的核心概念：**Host 與 Container 的檔案即時同步**。

---

### 傳統開發環境的痛點

如果要在本機使用 Jekyll，開發者需要：

1. 安裝特定版本的 **Ruby**
2. 安裝 **Jekyll** gem
3. 安裝相關 dependencies
4. 設定 file watcher（偵測檔案變更並重新編譯）
5. 設定 local web server

> 不同專案可能需要不同版本的 Ruby，版本衝突是常見問題。這就是「環境設定地獄」。

---

### Docker + Bind Mount 的解決方案

**關鍵優勢**：所有工具鏈都打包在 Container 內，開發者不需要在本機安裝任何東西。

- Container 內已有：Ruby、Jekyll、file watcher、web server
- 開發者只需要：Docker + 文字編輯器
- Host 的檔案變更會即時同步到 Container
- Container 內的 file watcher 偵測變更，自動重新編譯並刷新網站

---

### 實作步驟

#### 步驟 1：進入範例目錄

```bash
cd bindmount-sample-1
```

目錄內有 Jekyll 標準的 template 檔案：
- Markdown 檔案（`_posts/` 目錄）
- 設定檔（`_config.yml`）
- 其他靜態資源

---

#### 步驟 2：啟動 Jekyll Container

```bash
docker container run -d -p 80:4000 -v $(pwd):/site bretfisher/jekyll-serve
```

指令解析：
- `-p 80:4000`：將 Container 的 4000 port 映射到 Host 的 80 port
- `-v $(pwd):/site`：將當前目錄掛載到 Container 的 `/site`
- `bretfisher/jekyll-serve`：老師準備的自訂 Image，包含所有必要工具

---

#### 步驟 3：觀察啟動 Log

Container 啟動時，你會看到：
- Jekyll 正在從 source files 建置網站
- 最後出現 `Server running...` 類似訊息

> 這些 Log 代表 Jekyll 已經從你的 Markdown 檔案生成了 HTML 網站。

---

#### 步驟 4：開啟瀏覽器驗證

前往 `http://localhost`，你會看到 Jekyll 的預設模板網站，包含一篇文章。

---

#### 步驟 5：編輯檔案並觀察即時更新

使用你喜歡的編輯器（VS Code、Atom、Vim 等）開啟 `_posts/` 目錄下的 Markdown 檔案：

```markdown
---
title: "Welcome to Jekyll"
---

# Change this title!

This is my first post.
```

存檔後：
1. 回到 Terminal，你會看到 Container 偵測到檔案變更的 Log
2. 重新整理瀏覽器，變更已經反映

---

### 這證明了什麼？

1. **Host 的檔案變更即時同步到 Container**
2. **Container 內的工具能偵測變更並重新處理**
3. **整個過程不需要進入 Container shell**
4. **開發者不需要在本機安裝任何工具**

---

### 延伸學習：Jekyll 相關資源

如果你對 Jekyll 有興趣：
- **GitHub Pages** 免費提供 Jekyll hosting
- 可在 Docker Hub 或 GitHub 找到老師的 `bretfisher/jekyll-serve` Image 文件
- 你也可以自己建立客製化的 Jekyll Container

---

## 💡 重點摘要

- **Bind Mount 讓開發者能在 Host 用熟悉的編輯器修改檔案，Container 內即時反映變更。**
- **Jekyll 範例展示了「不需要在本機安裝工具鏈」的開發模式——所有 dependency 都在 Container 內。**
- **File watcher + web server 的組合，讓「存檔即預覽」的開發體驗成為可能。**
- **這種模式大幅簡化了開發環境設定，特別適合需要複雜工具鏈的專案。**
- **查看 Container logs 是了解內部運作的好方法，能清楚看到檔案變更偵測的過程。**

---

## 🔑 關鍵字

Bind Mount, Jekyll, SSG, Markdown, File Watcher
