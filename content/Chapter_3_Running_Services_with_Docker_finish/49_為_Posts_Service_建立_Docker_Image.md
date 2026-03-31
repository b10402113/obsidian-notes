# 為 Posts Service 建立 Docker Image

## 📝 課程概述

本單元從實作層面出發：如何將 Posts Service 包裝成一個 Docker Image 並啟動為 Container。我們會建立 `Dockerfile` 與 `.dockerignore`，逐步拆解 `docker build` 的過程，最後熟練幾個最基礎的 Docker CLI 指令。

---

## 核心觀念與實作解析

### 為什麼使用 `node:alpine` 作為 Base Image？

Alpine 是專為容器優化的輕量級 Linux 發行版，Image 體積極小（通常不到 150MB）。Node.js 官方有提供 Alpine 變體（`node:alpine`），這意味著我們不需要手動安裝 Node 環境，**Base Image 已經包含了我們需要的一切**。

### Dockerfile 的結構邏輯

```dockerfile
FROM node:alpine
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
```

每一行的設計背後都有其意義：

| 指令 | 目的 |
|---|---|
| `WORKDIR /app` | 設定後續所有指令的工作目錄，避免路徑混亂 |
| `COPY package.json ./` | **先複製 `package.json`，再執行 `npm install`** |
| `RUN npm install` | 在 Image 構建階段就完成依賴安裝，讓 Image 層緩存發揮最大效益 |
| `COPY . .` | 把，其餘程式碼複製進去（此時才包含 `index.js`）|

> **為什麼要先複製 `package.json` 再安裝依賴，最後才複製原始碼？** 這是 Docker 層緩存的關鍵策略。如果你的程式碼經常變動但 `package.json` 不常變動，Docker 會直接使用快取中的 `npm install` 結果，不需要每次都重新安裝相依套件，大幅縮短構建時間。

### `.dockerignore` 的必要性

```dockerignore
node_modules
```

如果不排除 `node_modules`，`COPY . .` 會把本機的 `node_modules/` 整個複製進 Image，覆蓋掉剛才 `npm install` 安裝的依賴——而且本機的版本可能與 Alpine 架構不相容，導致運行失敗。**`.dockerignore` 是防止錯誤資料被帶入 Image 的安全閥。**

### Docker 指令實戰

#### 1. 建構 Image（無標籤）
```bash
docker build .
```
成功後輸出結尾會顯示新 Image 的 ID（48 位十六進位字串）。

#### 2. 建構 Image（帶標籤，方便後續引用）
```bash
docker build -t <DockerHub_ID>/posts .
```
- `-t` 即 `--tag`，為 Image 取一個易讀名稱
- 格式為 `username/project-name`，上傳至 Docker Hub 後全球皆可取用

#### 3. 啟動 Container
```bash
docker run <IMAGE_ID>
# 或使用標籤
docker run stephengrider/posts
```

#### 4. 覆蓋預設命令：進入 Container 內部 Shell
```bash
docker run -it stephengrider/posts sh
```
- `-it`：互動式終端（interactive + pseudo-TTY）
- `sh`：啟動一個 Shell 而非執行 `npm start`

進入後可以執行 `ls`、`cd /app` 等命令，觀察 Container 內部的檔案結構。

#### 5. 查看運行中的 Container
```bash
docker ps
```
顯示 Container ID、使用的 Image、啟動命令、建立時間等資訊。

#### 6. 在運行中的 Container 內執行命令
```bash
docker exec -it <CONTAINER_ID> sh
```
**不重啟 Container，直接在內部開一個新的 shell 進去**，常用於 Debug。

#### 7. 查看 Container 日誌
```bash
docker logs <CONTAINER_ID>
```
`npm start` 輸出到標準輸出的所有內容都會被捕獲，可以用來排查啟動失敗的問題。

---

## 💡 重點摘要

- **Dockerfile 的分層策略：`package.json` 先行複製、依賴安裝先行、原始碼最後複製——這個順序是最佳實踐，能最大化 Docker 的層級快取效率。**
- **`.dockerignore` 務必排除 `node_modules`，否則本機編譯依賴會覆蓋 Image 內的依賴，甚至引入架構不相容的原生模組。**
- `docker run -it <image> sh` 讓你進入 Container 內部，是排查「Image 內環境是否符合預期」的最佳工具。
- `docker logs <container_id>` 是定位啟動錯誤的第一線手段。
- Image 是靜態模板，Container 是動態實例；同一個 Image 可以同時運行多個 Container。

---

## 🔑 關鍵字

Dockerfile, docker build, docker run, node:alpine, .dockerignore, Image, Container, docker exec, docker logs
