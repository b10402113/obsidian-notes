# Dockerize 其他服務：將標準化 Dockerfile 應用於全專案

## 📝 課程概述

本單元將 Posts Service 的 Dockerfile 與 `.dockerignore` 複製到所有其他服務（Query、Moderation、Event Bus、Comments、Client），並驗證 Event Bus 可以成功 Dockerize。同時說明了一個重要的設計原則：**如果所有服務共享相同的技術棧（Tech Stack）與啟動方式，理論上可以共享同一套 Dockerfile——但這並非常態，在真實專案中通常需要針對不同語言或版本進行客製化。**

## 核心觀念與實作解析

### 複製 Dockerfile 到所有服務

由於本專案中所有後端服務都滿足以下條件：
- 使用 **Node.js** 作為 runtime
- 啟動命令統一為 **`npm start`**
- 依賴管理都是 `package.json`

因此，Posts Service 的 `Dockerfile` 與 `.dockerignore` **幾乎可以原封不動地複製到每一個服務**：

| 服務 | Dockerfile | .dockerignore |
|------|-----------|--------------|
| query | ✅ 直接複製 | ✅ 直接複製 |
| moderation | ✅ 直接複製 | ✅ 直接複製 |
| event-bus | ✅ 直接複製 | ✅ 直接複製 |
| comments | ✅ 直接複製 | ✅ 直接複製 |
| client（React） | ✅ 直接複製 | ✅ 直接複製 |

這裡值得強調一點：**Client（React 前端）雖然是 Node 生態系的專案，但在實際部署時通常不適合用 Node Alpine 這種方式在 Server 端 Dockerize，而是需要不同的建置策略（打包成靜態檔案 + Nginx serving）。** 本章課程只做概念性展示。

### 驗證 Event Bus：一次乾淨的 Docker Build

在終端機中執行：

```bash
cd event-bus
docker build .
```

輸出會依序經過 Dockerfile 的每一個 Layer：

```
Step 1/7 : FROM node:16-alpine
 ---> <image_id>
Step 2/7 : WORKDIR /app
 ---> Using cache
Step 3/7 : COPY package.json ./
 ---> Using cache
Step 4/7 : RUN npm install
 ---> <install_output>
Step 5/7 : COPY . .
 ---> <file_copy_output>
Step 6/7 : CMD ["npm", "start"]
 ---> <execution_info>
Successfully built <image_id>
```

注意這裡用到了 **Layer Cache**：如果 `package.json` 沒變，`Step 3` 與 `Step 4` 會直接取用快取，`Step 5` 的 `COPY . .` 是唯一需要重新執行的 Layer。這正是上一章我們強調的最佳實踐。

啟動 Container：

```bash
docker run <image_id>
```

### 什麼時候需要客製化 Dockerfile？

雖然本專案的服務高度一致，但老師特別提醒：**這不是常態**。真實專案中，不同服務可能：

- 使用不同語言（Node.js vs. Go vs. Python）
- 使用同一語言但不同版本（例如 `node:14` vs. `node:18`）
- 有不同的啟動命令或環境變數需求
- 需要預先編譯（編譯型語言如 Java、Go）
- 需要不同的 base image（例如 `python:3.11-slim`、`maven:3.9-eclipse-temurin`）

**因此，你通常會需要針對每個服務維護一個專屬的 Dockerfile。** 本專案的「直接複製」策略，是因為一開始就刻意將所有服務設計為相同的技術棧，這是一種預先規劃的結果，而非意外。

### 下一步預告：Kubernetes

當所有服務都 Dockerize 完成後，下一步就是：**用 Kubernetes 來管理這些 Containers**。具體來說，我們將學習：

- 如何撰寫 Kubernetes YAML 設定檔
- 如何透過 `kubectl` 指令把這些 Containers 部署到 Cluster
- 如何利用 Kubernetes 的 Service Discovery 讓 Event Bus 不再需要知道每個服務的具體位址

## 💡 重點摘要

- 當多個服務共享相同的 runtime（Node.js）、相同的套件管理器（npm）與相同的啟動命令（`npm start`）時，可以共用同一套 Dockerfile 與 `.dockerignore`。
- `.dockerignore` 和 `Dockerfile` 兩者都必須複製到每個服務的目錄中，不能只複製其中一個。
- **真實專案中，不同語言或不同版本的服務通常需要各自客製化 Dockerfile**；本專案之所以能這樣做，是因為前期刻意統一的設計選擇。
- 驗證 Event Bus 的 `docker build` 成功，代表所有後端服務的 Dockerize 流程都已暢通，可以進入下一階段的 Kubernetes 編排。

## 關鍵字

Dockerfile, .dockerignore, Docker Build, Service Consistency, Multi-Service Architecture, node:16-alpine, npm start, Layer Cache, Microservices Standardization, Kubernetes Readiness
