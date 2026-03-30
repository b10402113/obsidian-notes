# 本章節的學習地圖：GitHub 必備周邊知識

## 📝 課程概述

本章節是 GitHub 的綜合實戰應用章節，涵蓋了「公開與私有 Repository 的差異」、「Collaborators 權限管理」、「README.md 的使用方式」、「Markdown 語法」以及「GitHub Pages 靜態網站托管」等重要但非核心的功能。這些工具雖非 Git 操作本身，卻是日常協作中不可或缺的周邊能力。

## 核心觀念與實作解析

### Public vs. Private Repository：visibility 不是 permission

很多初學者容易混淆「誰能看見」與「誰能修改」這兩件事。我們在這裡要釐清一個核心觀念：**Public Repository 和 Private Repository 決定的是「可見性（visibility）」，而不是「可操作性（permission）」**。

- **Public Repository**：網路上的任何人都能夠「看見、搜尋、clone」這個 Repository，但他們**無法推送任何修改**。沒有 Collaborator 權限的人，連一個 commit 都 push 不進去。
- **Private Repository**：只有「擁有者本人」以及「被明確授予 Collaborator 權限的人」才能看見這個 Repository。

> 這就是為什麼即便把 Repository 設為 public，也不等於任何人都能幫你改 code。這是兩層獨立的權限模型。

#### 如何變更隱私設定

變更 Repository 的可見性（public ↔ private）需要具備以下條件之一：

1. 你是這個 Repository 的擁有者（owner）
2. 在 Organization 或 Enterprise 等級的帳戶下，你被授予了變更可見性的權限

操作路徑是：**Settings → Options → 滾動到最底部的 Danger Zone → Change repository visibility**。GitHub 會要求你輸入 Repository 的完整名稱（如 `username/repo-name`）來確認身份，並輸入密碼才能完成變更。

**重要提醒**：將一個已經有 Collaborator、Watchers、Stars 的公開 Repository 轉為私有，這些關注和星星都會消失，而且所有曾經「看見」這個 Repo 的人都會瞬間失去存取權。這在大型專案中是非常危險的操作，原則上我們推薦的順序是：**從 Private 開始，確認沒問題後再轉為 Public**。

### 雙帳號實驗：親眼驗證權限差異

講師在影片中使用了兩個帳號（Colt 與 Stevie Chicks）來示範 Public 和 Private 的差異，這個設計非常值得學習。當用 Stevie 的帳號登入並訪問 Colt 的公開 Repo（如 `bookish-disco`）時，Stevie 只會看到一個普通的公開頁面，沒有任何 Settings 選項可操作。而當 Colt 將 Repo 改為 Private 後，Stevie 訪問同一個 URL 直接得到 404 錯誤——因為他根本沒有權限知道這個 Repository 存在。

這個實驗讓我們理解到：**Private Repository 的不僅是「看不見內容」，而是「連是否存在都不知道」**。

## 💡 重點摘要

- **Repository 的可見性（public/private）**與**程式碼修改權限（collaborator）**是兩個獨立的維度，必須分開理解。
- 任何人可以在網路上找到並 Clone 公開的 Repository，但**沒有 Collaborator 權限就無法推送任何變更**。
- 將公開 Repo 轉為私有會導致所有 Watchers、Stars、Wiki 等公開屬性全部消失，操作需謹慎。
- 建議的開發順序是：**先建立 Private Repository 確保安全，確認完成後再對外公開**。
- 變更 Repository 可見性的操作位於 Settings → Options 的最底部「Danger Zone」區塊，需要密碼驗證。

## 關鍵字

Public Repository, Private Repository, visibility, permission, Collaborator, Danger Zone, Change repository visibility, Stars, Watchers, clone
