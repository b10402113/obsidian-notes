# Dockerfile 實作練習解答

## 📝 課程概述

本單元是 Dockerfile 實作練習的解答影片。老師帶領我們從頭建立一個 Node.js 應用程式的 Dockerfile，完整示範 Dockerize 的迭代過程：建立 → Build → 測試 → 修正 → 重新 Build。

---

## 核心觀念與實作解析

### 實作情境

任務是為一個 Node.js 應用程式建立 Dockerfile。作為 Docker Admin，你可能不熟悉 Node.js，但需要完成 Dockerize。

**開發者提供的資訊：**
- 使用 Node.js 6.x 版本
- Base Image 需使用 Alpine
- 應用程式監聽 Port 3000
- 需要安裝 `tini` 套件

---

### Dockerfile 逐步解析

#### Step 1: FROM — 指定 Base Image

```dockerfile
FROM node:6-alpine
```

根據 Docker Hub 的 Node 官方 Image，`6-alpine` 表示 Node 6 版本的 Alpine 變體。

---

#### Step 2: EXPOSE — 宣告 Port

```dockerfile
EXPOSE 3000
```

應用程式監聽 3000 Port，後續 `docker run` 時需使用 `-p 80:3000`。

---

#### Step 3: RUN — 安裝套件與建立目錄

```dockerfile
RUN apk add --no-cache tini
RUN mkdir -p /usr/src/app
```

**Alpine 的 Package Manager 是 `apk`**，不同於 Debian/Ubuntu 的 `apt`。

> 最佳實踐：`--no-cache` 避免快取索引，減少 Image 大小。

---

#### Step 4: WORKDIR — 設定工作目錄

```dockerfile
WORKDIR /usr/src/app
```

後續指令會在此目錄下執行。

---

#### Step 5: COPY + RUN — 安裝依賴

```dockerfile
COPY package.json package.json
RUN npm install && npm cache clean --force
```

**為什麼先 COPY package.json 再 npm install？**

- 若先複製所有檔案，任何原始碼變更都會導致 `npm install` 重新執行
- 先只複製 `package.json`，依賴變更時才重新安裝
- 這是善用 Build Cache 的技巧

**清理 npm cache 與 `&&` 串接：**
- `npm cache clean` 清理暫存檔
- 用 `&&` 串接確保清理與安裝在同一個 Layer

---

#### Step 6: COPY — 複製原始碼

```dockerfile
COPY . .
```

複製所有檔案到工作目錄。

---

#### Step 7: CMD — 啟動指令

```dockerfile
CMD ["tini", "--", "node", "bin/www"]
```

這裡使用 **JSON Array 格式**（exec form）：

```dockerfile
CMD ["executable", "param1", "param2"]
```

**為什麼用 `tini`？**

`tini` 是一個輕量級 init process，能正確處理：
- Signal forwarding（如 SIGTERM）
- Zombie process 回收

> 在 Alpine 容器中使用 `tini` 是常見做法，確保程序能正確接收停止訊號。

---

### 完整 Dockerfile

```dockerfile
FROM node:6-alpine

EXPOSE 3000

RUN apk add --no-cache tini

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

COPY package.json package.json
RUN npm install && npm cache clean --force

COPY . .

CMD ["tini", "--", "node", "bin/www"]
```

---

### Build 與測試流程

#### 1. Build Image

```bash
docker build -t testnode .
```

#### 2. 測試執行

```bash
docker run --rm -p 80:3000 testnode
```

瀏覽器開啟 `localhost` 確認應用程式正常運作。

---

### 上傳 Docker Hub 流程

#### 1. 重新 Tag

```bash
docker tag testnode bretfisher/testing-node
```

#### 2. Push

```bash
docker push bretfisher/testing-node
```

#### 3. 清除本地 Image 並重新 Pull 測試

```bash
docker image rm bretfisher/testing-node
docker run --rm -p 80:3000 bretfisher/testing-node
```

這確保 Docker Hub 上的 Image 正確可用。

---

### Dockerfile 開發的迭代本質

老師強調一個重要觀念：

> **很少人能一次寫出正確的 Dockerfile。**

典型流程：
1. 寫 Dockerfile
2. Build → 失敗 → 修正 → 重 Build
3. Run Container → 失敗 → 修正 → 重 Build
4. 測試成功 → 完成

這是正常的學習曲線，透過迭代累積經驗。

---

## 💡 重點摘要

- **Dockerize 是迭代過程：Build → 測試 → 修正 → 重 Build 是常態。**
- **Alpine 使用 `apk` 作為 Package Manager，`--no-cache` 可減少 Image 大小。**
- **先 COPY package.json 再 npm install，可善用 Build Cache 加速建置。**
- **`&&` 串接指令確保清理動作與安裝在同一 Layer，避免暫存檔殘留。**
- **CMD 使用 JSON Array 格式（exec form）是 Dockerfile 的標準寫法。**

---

## 🔑 關鍵字

Dockerfile, Alpine, apk, npm, tini, CMD, Build Cache, Dockerize
