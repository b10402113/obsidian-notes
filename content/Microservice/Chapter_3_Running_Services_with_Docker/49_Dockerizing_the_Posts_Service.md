# Dockerize Posts Service：將 Node.js 服務封裝為 Docker Image

## 📝 課程概述

本單元開始實作層面的工作：為 Posts Service 建立 **Dockerfile** 與 **.dockerignore**，並透過 `docker build` 將其打包成 Image，接著用 `docker run` 啟動 Container 驗證是否正常運作。重點在於理解 Dockerfile 每一行的意義，以及為什麼要用多階段複製（先複製 `package.json` 再複製原始碼）來最佳化 Image 建置流程。

## 核心觀念與實作解析

### 什麼是 Dockerize？

「Dockerize」的意思是：**讓一個應用程式能夠在 Docker Container 內運行**。具體來說，就是建立 Dockerfile 並透過 `docker build` 建出 Image。

### Dockerfile 設計邏輯

以下是我們為 Posts Service 設計的 Dockerfile 結構：

```dockerfile
FROM node:16-alpine        # ① 指定基礎 Image
WORKDIR /app               # ② 設定工作目錄
COPY package.json ./       # ③ 只複製 package.json
RUN npm install            # ④ 安裝依賴
COPY . .                   # ⑤ 複製其餘所有原始碼
CMD ["npm", "start"]       # ⑥ 設定啟動命令
```

#### ① `FROM node:16-alpine`

`alpine` 是一個極度精簡的 Linux 發行版映像檔。用它作為 base image，`node:16-alpine` 的體積遠比完整 Ubuntu/CentOS 小得多，下載與部署都更快。

#### ② `WORKDIR /app`

設定此後所有指令的工作目錄，相當於在 container 內執行 `cd /app`。統一工作目錄能避免路徑混淆。

#### ③ & ④ 先複製 `package.json` 再 `npm install` —— 為什麼？

這是 Docker 建置的最佳實踐，原因是 **Layer Caching（層級快取）**：

- Docker 在建置 Image 時，會將每一個指令視為一個 Layer。
- 當一個 Layer 的內容（指令+上下文）沒有變化時，後續建置可以**直接使用快取**，跳過 `npm install`。
- 如果我們一開始 `COPY . .` 把整個專案（含 `package.json`）一次複製進去，每次改任何一行程式碼，都會讓這整個 Layer 快取失效，導致每次都要重新 `npm install`——在大專案中這可能耗費數十秒到數分鐘。

**正確的做法**是：先單獨複製 `package.json`，单独跑一次 `npm install`。如此一來，只要 `package.json` 沒變，`npm install` 這層就會命中快取；只有 `COPY . .` 這層會在程式碼變動時失效。

```dockerfile
COPY package.json ./   # 這層通常不會經常變
RUN npm install        # 這層依賴上一層，所以通常也會命中快取
COPY . .               # 只有這層會經常因為改 code 而重建
```

#### ⑥ `CMD` 與 `RUN` 的區別

- `RUN`：在 Image 建置階段執行，用於設定 Image 內容（一次性操作，如安裝套件）。結果會寫入 Image 本身。
- `CMD`：在 Container **啟動時**執行，定義容器預設要做的動作。若 `docker run` 時有覆蓋命令，則 `CMD` 會被取代。

### `.dockerignore`：避免把不需要的檔案送進 Image

在專案根目錄建立 `.dockerignore`，內容如下：

```
node_modules
```

**為什麼要這個？** 因為 Docker 建置時，`COPY . .` 會把所有檔案複製進 Image。如果不排除 `node_modules`：

1. 浪費建置時間（要把數千個檔案複製進去）
2. 浪費 Image 體積（`node_modules` 可以在建置時由 `npm install` 重新產生）
3. 可能出現本機與容器內版本不一致的問題

`.dockerignore` 的作用類似 Git 的 `.gitignore`，只不過它作用於 `docker build` 的 context。

### 建置與執行

```bash
# 建置 Image
docker build -t stephengrider/posts .

# 啟動 Container
docker run stephengrider/posts
```

如果看到 Server 正常啟動的輸出，代表 Dockerize 成功。

## 💡 重點摘要

- `node:16-alpine` 是 Node.js + 精簡 Linux 的官方 base image，體積小是它的最大優點。
- **一定要先複製 `package.json` 再 `npm install`，最後才複製其餘程式碼**——這是利用 Docker Layer Caching 加速重建的關鍵技巧。
- `.dockerignore` 可以防止 `node_modules` 等本機產物進入 Image，避免浪費空間並保持 Image 可重現性。
- `RUN` 用於建置階段，`CMD` 用於容器啟動階段——兩者執行時機不同，不可混淆。
- Image 建完後，`docker run` 就是把 Image 實例化為一個正在運行的 Container。

## 關鍵字

Dockerfile, Docker Image, Docker Container, Docker Build, Layer Caching, .dockerignore, node:alpine, FROM, WORKDIR, COPY, RUN, CMD, npm install, Docker Build Context
