# 啟動與管理 Nginx Container
![[Pasted image 20260408191651.png|700]]

## 📝 課程概述

本單元是 Container 實作的第一步，我們將學習如何啟動、停止、移除 Container，並查看其日誌與運行程序。同時也會釐清 Image 與 Container 的根本差異，為後續課程奠定基礎。

## 核心觀念與實作解析

### Image 與 Container 的差異
![[Pasted image 20260408191714.png|700]]

在開始操作之前，我們必須理解這兩個核心概念：

| 概念 | 說明 | 類比 |
|------|------|------|
| **Image** | 應用程式的二進位檔、函式庫、原始碼集合 | 軟體安裝光碟 |
| **Container** | Image 的運行實例 | 安裝後正在執行的程式 |

> 一個 Image 可以產生多個 Container，就像一張安裝光碟可以安裝到多台電腦。

**Image 的來源**：Registry（映像檔倉庫），類似 GitHub 之於原始碼。Docker 的預設 Registry 是 **Docker Hub**（hub.docker.com）。

---

### 啟動第一個 Container

```bash
docker container run -p 80:80 nginx
```

這個命令觸發了一連串的背景作業：

**Docker 在背景做了什麼？**
![[Pasted image 20260408191942.png|700]]

1. **尋找 Image**：在本機 Image Cache 中尋找 `nginx`
2. **下載 Image**：若本機沒有，從 Docker Hub 下載最新版本（`latest` tag）
3. **建立 Container**：基於該 Image 啟動新 Container
4. **設定網路**：分配虛擬 IP、設定虛擬網路
5. **Port 映射**：開啟 Host Port 80，轉送到 Container Port 80
6. **執行啟動命令**：執行 Image 中指定的預設命令

> 若要指定版本，可用 `nginx:1.19` 格式，不指定則預設為 `latest`。

---

### 背景執行與命名

**背景執行（Detached Mode）**：

```bash
docker container run -d -p 80:80 --name webhost nginx
```

- `-d`（detach）：讓 Container 在背景執行
- `--name webhost`：指定 Container 名稱（方便管理）

> 若不指定名稱，Docker 會自動生成一個隨機名稱，由「形容詞 + 著名駭客/科學家姓氏」組成。

---

### 查看 Container 狀態

**列出運行中的 Container**：

```bash
docker container ls
```

**列出所有 Container（包含已停止）**：

```bash
docker container ls -a
```

輸出包含：
- Container ID（唯一識別碼）
- Image 名稱
- 執行狀態
- Port 映射
- 名稱
![[Pasted image 20260408192105.png|700]]
---

### 查看 Container 日誌與程序

**查看日誌**：

```bash
docker container logs webhost
```

常用選項：
- `-f`（follow）：持續追蹤日誌輸出
- `--tail N`：只顯示最後 N 行

**查看運行中的程序**：

```bash
docker container top webhost
```

這會顯示 Container 內運行的程序列表，類似 Linux 的 `ps` 命令。

---

### 停止與移除 Container

**停止 Container**：

```bash
docker container stop <container_id_or_name>
```

> 只需輸入 ID 的前幾個字元，只要能唯一識別即可。

**移除 Container**：

```bash
docker container rm <container_id_or_name>
```

**一次移除多個 Container**：

```bash
docker container rm 63f 690 0de
```

**強制移除運行中的 Container**：

```bash
docker container rm -f <container_id>
```

> 預設無法移除運行中的 Container，這是安全機制。`-f`（force）會先停止再移除。

---

## 💡 重點摘要

- **Image 是靜態的應用程式檔案集合；Container 是 Image 的運行實例。**
- **`docker container run` 會自動從 Docker Hub 下載 Image 並啟動 Container。**
- **`-d` 讓 Container 背景執行，`--name` 指定名稱便於管理。**
- **`ls`、`logs`、`top` 用於查看狀態；`stop`、`rm` 用於停止與移除。**

## 🔑 關鍵字

Image, Container, Nginx, Registry, Docker Hub
