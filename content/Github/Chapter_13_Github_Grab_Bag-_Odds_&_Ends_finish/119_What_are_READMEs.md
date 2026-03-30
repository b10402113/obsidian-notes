# README 與 Markdown：專案的門面與寫作語法

## 📝 課程概述

本節介紹 GitHub Repository 中最重要的文件——README.md，以及支撐它運作的 Markdown 語法。README.md 是每個人來到你的專案時第一眼看到的內容，相當於專案的「說明書」；而 Markdown 則是一種輕量級的文字格式化語法，讓你能用極簡的符號寫出結構清晰且美觀的文件，而不需要撰寫繁瑣的 HTML。

## 核心觀念與實作解析

### README.md 的定位與價值

當我們說「README.md 是專案的入口」，這句話背後的意義是：**每一個來到你 Repository 的人，無論是想了解你的專案、做貢獻、還是只是路過，他們第一個接觸的就是 README**。

一個好的 README 應當涵蓋以下核心內容：

- **專案是做什麼的**：一句話說清楚你的專案解決了什麼問題
- **目標讀者是誰**：開發者？一般使用者？資料科學家？
- **如何開始**：如何安裝、需要哪些前置條件、如何運行
- **技術棧（Technologies）**：用了哪些語言、框架、資料庫
- **如何貢獻**：貢獻指南、Code of Conduct、聯絡方式
- **授權條款（License）**：採用什麼開源授權

> GitHub 的特殊之處在於：只要把 `README.md` 放在 Repository 的根目錄，GitHub 就會**自動在 Repository 首頁渲染它的內容**。這就是為什麼幾乎所有知名的 GitHub 專案都有 README——它是 GitHub 提供的一個免費的「專案介紹頁」。

### Markdown 是什麼？為什麼要學？

Markdown 是一種「純文字格式化語法」，它的設計目標是：**讓人用最少的學習成本，寫出結構化的文件**。

傳統上，如果你要讓文字有標題、粗體、連結、程式碼區塊，你需要寫 HTML。但 HTML 語法繁瑣，可讀性差。Markdown 的出發點是：「我想要 **Readable** 的同時又能 **Renderable**」。

```
# 一級標題
## 二級標題
### 三級標題
```

上面這些井字號，Markdown 會自動轉換成對應層級的 HTML `<h1>` 到 `<h6>` 標題標籤。這就是 Markdown 的核心理念：**用符號表達結構，用最直覺的方式寫作**。

### Markdown 語法速查

以下是你在撰寫 README 時最常使用的語法：

#### 標題（Headings）

```markdown
# 一級標題（主要標題，每個 README 通常只有一個）
## 二級標題（章節標題）
### 三級標題（子章節）
```

**常見陷阱**：井字號後面**必須加一個空格**，否則 GitHub 不會將其識別為標題。

#### 文字樣式

```markdown
**粗體**        # 雙 asterisks 包住文字
*斜體*          # 單 asterisk 包住文字
~~刪除線~~      # 雙 tilde 包住文字
`行內程式碼`     # 反引號包住，常用於變數名、函數名、指令
```

#### 程式碼區塊

```markdown
# 單行行內程式碼
`console.log("Hello")`

# 多行程式碼區塊（可指定語言以獲得語法高亮）
```javascript
const greeting = "Hello World";
console.log(greeting);
```
```

GitHub 支援數十種語言的語法高亮，只要在開頭的三個反引號後面加上語言名稱（如 `javascript`、`python`、`bash`）。

#### 引用區塊（Blockquotes）

```markdown
> 這是一段引用文字
> 可以跨越多行
```

> 這就是引用區塊的呈現效果，常用於提示、重要說明或引用文件內容。

#### 清單

```markdown
# 無序清單（可用 -, +, * 任意一種）
- 第一項
- 第二項
  - 子項目（縮排兩個空格）
  - 子項目

# 有序清單
1. 第一步
2. 第二步
3. 第三步
```

#### 連結與圖片

```markdown
[連結文字](https://www.example.com)

![替代文字](https://www.example.com/image.png)
```

連結的語法是 `[文字](URL)`，圖片則是多了一個驚嘆號 `![文字](URL)`。

### Markdown 不等於 GitHub 專屬

值得強調的是：**Markdown 不是 GitHub 發明的，也不是 GitHub 獨有的**。它是一個通用的文字格式標準，被廣泛應用於：

- Reddit 的評論系統（過去常用，現在可能仍部分支援）
- 許多部落格平台（如 Ghost、Jekyll）
- VS Code 的 `.md` 檔案預覽
- 獨立應用如 Notion、Obsidian、Typora
- 很多聊天工具的訊息格式化（如 Slack、Discord）

學會 Markdown 的價值遠超 GitHub 本身，這是一項通用技能。

## 💡 重點摘要

- **README.md 是專案的第一印象**：放在 Repository 根目錄後，GitHub 會自動在首頁渲染顯示。
- **Markdown 的核心價值**在於用極簡語法替代 HTML，讓文件兼具可讀性與結構化。
- 標題語法 `#` 後面**必須有空格**，否則不會被識別為標題——這是新手最常犯的錯誤。
- 程式碼區塊用**三個反引號**包裹，可指定語言獲得語法高亮（如 ````javascript````）。
- Markdown 的應用範圍不限於 GitHub，是一項通用寫作技能。

## 關鍵字

README.md, Markdown, .md, headings, h1, bold, italic, code block, syntax highlighting, blockquote, list, unordered list, ordered list, link, image, backtick, asterisk, horizontal rule, GitHub Flavored Markdown (GFM)
