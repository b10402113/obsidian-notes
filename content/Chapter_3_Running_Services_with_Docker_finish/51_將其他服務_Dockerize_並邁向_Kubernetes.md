# 將其他服務 Dockerize 並邁向 Kubernetes

## 📝 課程概述

本單元將上一堂課為 Posts Service 建立的 `Dockerfile` 與 `.dockerignore` 複製到所有其他微服務（Query、Moderation、Event Bus、Comments、Client），並驗證 Event Bus 的 Image 能否成功構建與運行。最後預告下一階段：正式進入 Kubernetes 的世界。

---

## 核心觀念與實作解析

### 為什麼所有服務可以共用同一份 Dockerfile？

這並非巧合，而是**在專案初期就刻意設計好的架構決策**。讓我們檢視每個服務的共同點：

| 服務 | 執行環境 | 啟動命令 |
|---|---|---|
| Posts | Node.js | `npm start` |
| Comments | Node.js | `npm start` |
| Query | Node.js | `npm start` |
| Moderation | Node.js | `npm start` |
| Event Bus | Node.js | `npm start` |
| Client（React） | Node.js | `npm start` |

> **關鍵 insight**：所有服務都是 Node.js runtime，且啟動方式完全一致。這使得一個 `Dockerfile` 就能滿足幾乎所有服務的需求——這是「統一技術棧」在 DevOps 層面帶來的巨大红利。

如果你的專案混用了 Python、Go、Ruby 等不同語言的服務，就不可能這樣複製粘貼，而必須為每種語言設計各自的 Base Image。

### 複製的具體操作

直接將 `posts/` 目錄下的 `Dockerfile` 與 `.dockerignore` 複製到：
- `query/`
- `moderation/`
- `event-bus/`
- `comments/`
- `client/`

這個操作不涉及任何程式碼改寫，因為兩個檔案的內容對所有服務都是通用的。

### 驗證 Event Bus Image 的建構

```bash
cd event-bus
docker build .
```

建構過程與 Posts Service 完全一致：
1. 從 `node:alpine` 拉取 Base Image
2. 設定 `WORKDIR /app`
3. 複製 `package.json` 並執行 `npm install`
4. 複製 `index.js` 等原始碼
5. 設定 `CMD ["npm", "start"]`

建構完成後，拿 Image ID 或 Tag 執行 `docker run <ID>`，若成功啟動，Event Bus 的 HTTP server 就會開始監聽。

### 這一步的戰略意義

此時此刻，我們的專案已經完成了一個關鍵里程碑：**所有微服務都具備了被封裝為 Container 的能力**。這意味著：

- **環境一致性**：無論在本機、VM、還是 Kubernetes Cluster，每個服務的運行環境完全相同
- **可移植性**：只要目標機器有 Docker Engine，就能原封不動地運行這些 Image
- **邁向 Kubernetes 的前提**：Kubernetes 的最小調度單位正是 Pod（封裝了 Container），所有服務都 Dockerize 之後，下一步就是讓 Kubernetes 來接管它們

> 我們現在「準備好了」，但真正的威力要等到 Kubernetes 上線才會釋放——Service 層的自動負載平衡與服務發現，會徹底取代 Event Bus 中那些硬編碼的 URL。

---

## 💡 重點摘要

- **統一技術棧（Node.js + `npm start`）讓一個 `Dockerfile` 就能滿足幾乎所有服務，大幅降低 DevOps 複雜度。**
- 將 `Dockerfile` 與 `.dockerignore` 直接複製到其餘服務，是 CI/CD 流水線中常見的「相同模板批量應用」模式。
- 透過 `docker build .` + `docker run <ID>` 快速驗證 Image 是否能正確啟動，是本地端 Debug Image 的標準流程。
- **所有服務 Dockerize 完成，是進入 Kubernetes 前的必備前置條件——這是從「手動管理」到「自動協調」的關鍵轉折點。**

---

## 🔑 關鍵字

Dockerfile, .dockerignore, Event Bus, 統一技術棧, Image 建構, Container, Kubernetes 準備
