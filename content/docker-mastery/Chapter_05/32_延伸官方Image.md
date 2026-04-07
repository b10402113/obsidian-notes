# 延伸官方 Image

## 📝 課程概述

本單元展示如何以官方 Image 為基礎，建立自訂 Image。這是實務上最常見的做法：從 Docker Hub 的官方 Image 出發，加入自己的設定或原始碼，快速完成 Dockerize。

---

## 核心觀念與實作解析

### 為什麼要延伸官方 Image？

實務上，建立自訂 Image 的流程通常是：

1. **先嘗試官方 Image** — 最省力、維護成本最低
2. **遇到限制時** — 才考慮客製化或尋找其他 Image
3. **最後才自行建立** — 工作量最大、維護成本最高

> **最佳實踐：能用官方 Image 就用官方 Image。**

---

### Dockerfile 範例：自訂 Nginx

```dockerfile
FROM nginx:latest

WORKDIR /usr/share/nginx/html

COPY index.html index.html
```

這個簡單的 Dockerfile 做了三件事：
1. 以官方 `nginx` Image 為基礎
2. 切換到 Nginx 的 HTML 目錄
3. 複製自訂的 `index.html`

---

### 新指令：WORKDIR 與 COPY

#### WORKDIR — 切換工作目錄

```dockerfile
WORKDIR /usr/share/nginx/html
```

`WORKDIR` 類似 `cd` 指令，用來切換 Container 內的工作目錄。

**為什麼不用 `RUN cd`？**

| 方式 | 問題 |
|------|------|
| `RUN cd /app` | 每個 RUN 是獨立 Shell，cd 效果不會延續 |
| `WORKDIR /app` | Dockerfile 專屬指令，效果延續到後續指令 |

> **最佳實踐：** 使用 `WORKDIR` 而非 `RUN cd`，可讀性更好。

---

#### COPY — 複製檔案

```dockerfile
COPY index.html index.html
# 或
COPY . .
```

`COPY` 將本地檔案複製到 Image 中。格式為：

```
COPY <本地路徑> <Image 內路徑>
```

**常見用法：**
- `COPY . .` — 複製當前目錄所有檔案
- `COPY package.json .` — 只複製特定檔案

---

### 繼承機制

當我們使用 `FROM nginx` 時：

```
┌─────────────────────────────┐
│  我們的 Dockerfile           │
│  - WORKDIR                  │
│  - COPY                     │
├─────────────────────────────┤
│  nginx Image（來自 Docker Hub）│
│  - 已有的 ENV               │
│  - 已有的 EXPOSE            │
│  - 已有的 CMD               │
└─────────────────────────────┘
```

**我們繼承了 `nginx` Image 的所有設定**，包括：
- 環境變數
- Exposed Ports
- 預設 CMD

> 這就是為什麼我們的 Dockerfile 不需要 `CMD` — 已經從 `nginx` 繼承了。

---

### 實作流程

#### 1. 建立自訂 Image

```bash
docker build -t nginx-html .
```

#### 2. 執行 Container

```bash
docker container run -p 8080:80 --rm nginx-html
```

#### 3. 驗證結果

瀏覽器開啟 `localhost:8080`，應該會看到自訂的 `index.html`。

---

### 上傳到 Docker Hub

#### 重新 Tag

```bash
docker image tag nginx-html bretfisher/nginx-html
```

#### Push

```bash
docker image push bretfisher/nginx-html
```

---

## 💡 重點摘要

- **延伸官方 Image 是最實務的做法：維護成本低、文件完善、安全性有保障。**
- **`WORKDIR` 用於切換工作目錄，比 `RUN cd` 更清晰且效果延續。**
- **`COPY` 將本地檔案複製到 Image，是原始碼進入 Container 的主要方式。**
- **使用 `FROM` 繼承的 Image 會保留所有設定（ENV、EXPOSE、CMD 等）。**
- **Dockerize 流程：先試官方 Image → 再找社群 Image → 最後才自行建立。**

---

## 🔑 關鍵字

FROM, WORKDIR, COPY, Official Image, Dockerize, Inheritance
