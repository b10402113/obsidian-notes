# NPM Organizations 與套件安全設定

## 📝 課程概述

將 Common Module 發布為 NPM Package 是解決方案，但在那之前，我們需要先確認「誰能夠看到這個套件」。本單元介紹 NPM Organizations 的三種發布方式（公開 Registry、組織、私人 Registry），並實作建立一個 Organization 來管理我們的共用套件。

---

## 核心觀念與實作解析

### 三種 NPM 發布方式

| 發布方式 | 可見範圍 | 費用 | 適用情境 |
|---|---|---|---|
| 直接發布到公開 Registry | **所有人**可見 | 免費 | 開源專用的 library |
| 發布到 Organization（可設定 public/private） | 組織成員（私人模式）或不特定人士（公開模式） | 私人組織需付費 | 商業團隊協作 |
| 發布到私人 Registry | 只有被授權的人可以存取 | 私人 Registry 需自架或付費 | 高度敏感的商業程式碼 |

### 為什麼不直接發布到公開 Registry？

如果我們將 `@sg-tickets/common` 直接 publish 到 NPM 公開 Registry，**任何人**都可以查看、安裝，甚至看到我們內部的 business logic 與實作細節。對於含有商業機密或內部演算法的專案來說，這是不可接受的風險。

### Organization 的 public vs. private

- **Public Organization**：任何人都能看到套件內容，但只有組織成員可以發布新版本。適合開源但希望管制發布權限的專案。
- **Private Organization**：只有組織成員能夠看到與安裝套件。但需要付費訂閱 NPM。

### 建立 Organization 的實作流程

1. 登入 [npmjs.com](https://npmjs.com)
2. 點擊右上角帳號 → **Add organization**
3. 選擇 **Public Organization**（我們選擇這個，因為 Common Module 沒有太多敏感程式碼，不需要付費）
4. 輸入組織名稱（**必須唯一**）。講師使用 `sg-tickets`，你需要自己想一個獨特的名字
5. 點擊 **Create**，跳過邀請成員的步驟

> **名稱的命名慣例**：講師使用 `sg-tickets`（SG = Steven Grider 的縮寫）。建議取一個容易記憶、符合個人或團隊風格的名字。

---

### Organization 建立後的套件發布

建立 Organization 後，我們就可以將 Common Module 發布到這個組織底下。

發布時套件的名稱格式為：

```json
{
  "name": "@<org-name>/common"
}
```

例如：`@sg-tickets/common`

這個前綴 `@org-name/` 是 Organization 套件的標準命名空間，NPM 會自動將其與 Organization 關聯起來。

> **注意**：如果要在 Organization 內發布 public package，發布時必須加上 `--access public` 旗標。否則 NPM 預設會將套件設為 private，而 private package 在付費帳號之外無法發布。

---

## 💡 重點摘要

- **直接發布到 NPM 公開 Registry 會讓所有程式碼暴露在網路上，存在安全風險。**
- **Organization 讓團隊可以管理誰有權限發布與查看套件。**
- **Private Organization 需要付費；Public Organization 免費但內容公開。**
- **組織名稱必須全球唯一，且日後很難更名，選擇時要謹慎。**
- **發布 public package 到 Organization 時，必須加上 `--access public`，否則會因為權限不足而發布失敗。**

---

## 🔑 關鍵字

NPM Organization, NPM Registry, `@org-name/common`, `--access public`, Private Package, Public Package
