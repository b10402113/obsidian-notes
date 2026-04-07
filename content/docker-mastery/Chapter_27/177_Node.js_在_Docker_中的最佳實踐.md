# Node.js 在 Docker 中的最佳實踐

## 📝 課程概述

本單元是 DockerCon 2022 的演講整理，由 Bret Fisher 主講，深入探討 Node.js 應用程式在 Docker 中的最佳實踐。內容涵蓋 Base Image 的選擇策略、Dockerfile 的安全優化、PID 1 問題的解決方案，以及 Multi-stage Builds 的進階應用，幫助開發者打造更安全、更精簡的 Production-grade Node.js Container。

---

## 核心觀念與實作解析

### Base Image 的選擇困境

選擇一個「最安全、最小、且有官方支援」的 Node.js Image，其實比我們想像的複雜許多。讓我們來看看各個選項的優缺點：

#### 不要使用 `node:latest` 或 `node:16`（無 tag）

這兩個是最危險的選擇：

- **`node:latest`**：包含大量不必要的 packages（如 ImageMagick、Subversion、Mercurial），擁有超過 **800 個 CVEs**
- **`node:16`**（無 slim tag）：擁有近 **2000 個 CVEs**，完全不適合 Production 使用

> 老師強調：除了第一天學習 Docker 之外，**永遠不要使用 `node:latest`**。

#### 為什麼不推薦 Alpine？

雖然 Alpine 以輕量聞名，但對於 Node.js 有幾個問題：

1. **Node.js 團隊只將 Alpine 視為實驗性支援**，並非 Tier 1 支援
2. **大小優勢不再明顯**：Debian slim 版本的大小與 Alpine 相當，Distroless 甚至更小
3. **APK 套件無法正確 pin 版本**：當 Alpine 版本更新時，舊版本可能從 package manager 消失，導致 image build 失敗
4. **Production 故事多**：許多開發者回報 Alpine-specific 的問題，往往需要切換回 Debian/Ubuntu 才能解決

#### 推薦的 Base Image 選項

| 選項 | CVE 數量 | 優點 | 缺點 |
|------|----------|------|------|
| `node:16-bullseye-slim` | ~131 | 官方支援、容易取得 | CVE 較多 |
| `node:16-bullseye-slim` (Debian 11) | 較少 | 使用最新 Debian | 需要手動指定 tag |
| Ubuntu 20.04 + 自建 | ~15 (0 high/critical) | 最少 CVE、Stack Overflow 資源多 | 需額外步驟 |
| Google Distroless | ~13 | 最小、最安全 | 無 shell、只能作為最後 stage |

#### 如何自建 Ubuntu Base Image

老師推薦的方式是「從官方 Node image 複製 binary 到 Ubuntu」：

```dockerfile
# 定義版本一次，後續重複使用
FROM node:16-bullseye-slim AS node

FROM ubuntu:20.04
# 指定日期版本確保 reproducible builds

# 複製 Node.js 相關檔案
COPY --from=node /usr/local /usr/local

# 設定 corepack
RUN corepack disable && corepack enable
```

這個方式可以達到 **0 high/critical CVE，只有 15 個 medium/low**，遠優於官方 slim image 的 12 high + 74 medium/low。

---

### Dockerfile 最佳實踐

#### 1. 使用 .dockerignore

**一定要**將 `.dockerignore` 設定好，至少排除：

```
.git
node_modules
```

#### 2. Production 只安裝必要依賴

```dockerfile
# Production stage 只安裝 production dependencies
RUN npm ci --only=production
```

`npm ci` 會嚴格按照 `package-lock.json` 安裝，確保版本與測試環境一致。

#### 3. 不要以 root 執行

```dockerfile
# 官方 node image 已內建 node user
USER node

# 若使用 Ubuntu 或 Distroless，需手動建立
RUN groupadd -r node && useradd -r -g node node
```

#### 4. COPY 時設定正確擁有者

```dockerfile
COPY --chown=node:node package*.json ./
RUN npm ci --only=production
COPY --chown=node:node . .
```

---

### PID 1 問題與 Tini

#### 為什麼 Node.js 不應該作為 PID 1？

在 Linux Container 中，第一個啟動的 process 會成為 **PID 1**。但 Node.js 並非設計來處理 PID 1 的職責：

- 無法正確處理 Linux Kernel 的 **Signal**（如 SIGTERM）
- 無法回收 **Zombie Processes**（失去 parent 的 process）
- 可能導致 Health Check 建立大量 zombie processes

#### 解決方案：使用 Tini

**Tini** 是一個專門的 init process，可以：

1. 正確處理來自 Kernel 的 signals
2. 防止 zombie processes 累積
3. 支援 Docker health check 與 Kubernetes probes

```dockerfile
# 安裝 Tini
RUN apt-get install -y tini

# 設定為 entrypoint
ENTRYPOINT ["/usr/bin/tini", "--"]

# CMD 直接執行 node（不要透過 npm）
CMD ["node", "index.js"]
```

> **不要用 `npm start` 作為 CMD**，因為 npm 無法正確處理 signals，且是一個不必要的額外 process。

---

### Multi-stage Builds 進階應用

Multi-stage 的核心目標是：**一個 Dockerfile 支援 dev、test、production 三種環境**。

#### 基本結構

```dockerfile
# === Base Stage ===
FROM node:16-bullseye-slim AS base
WORKDIR /app
# 只複製 lock files
COPY package*.json ./
RUN npm ci --only=production
# 安裝 Tini、設定 user...

# === Dev Stage ===
FROM base AS dev
RUN npm ci  # 安裝所有 devDependencies
CMD ["nodemon", "index.js"]

# === Source Stage ===
FROM base AS source
COPY --chown=node:node . .

# === Test Stage ===
FROM source AS test
COPY --from=dev /app/node_modules /app/node_modules
RUN npm test

# === Production Stage ===
FROM source AS production
# 只添加 metadata，不重新複製 source
ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["node", "index.js"]
```

#### 為什麼要分離 Source Stage？

關鍵在於**測試與部署的一致性**：

- Test stage 從 `source` stage 複製程式碼並執行測試
- 測試通過後，Production stage **從同一個 `source` stage** 取得程式碼
- 這樣可以確保「測試什麼，就部署什麼」

> **重要觀念**：Production stage 只添加 metadata（ENTRYPOINT、CMD），不重新複製 source，確保測試與部署使用完全相同的 layers。

---

## 💡 重點摘要

- **選擇 Base Image 時，避免 `node:latest` 與無 tag 版本，優先考慮 `slim` 變體或自建 Ubuntu image。**
- **Alpine 對 Node.js 而言有潛在風險：官方支援等級低、APK 版本 pin 問題、production 故事多。**
- **Node.js 不應作為 PID 1，必須搭配 Tini 等 init process 處理 signals 與 zombie processes。**
- **Multi-stage Builds 可以在單一 Dockerfile 中實現 dev/test/prod 的分離，確保測試與部署的一致性。**
- **Production stage 不應重新複製 source，而是從已測試的 stage 繼承，只修改 metadata。**

---

## 🔑 關鍵字

Base Image, CVE, Alpine, Distroless, Tini, PID 1, Multi-stage, npm ci
