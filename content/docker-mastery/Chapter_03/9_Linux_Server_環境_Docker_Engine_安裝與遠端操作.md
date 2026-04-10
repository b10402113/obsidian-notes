# Linux Server 環境 Docker Engine 安裝與遠端操作

## 📝 課程概述

本單元針對兩種情境：一是你需要將 Docker 運行在遠端 Linux Server 上（而非本地），二是由於授權、效能或權限等因素無法在本地運行 Docker。我們將學習如何安裝 Docker Engine，以及如何從本地端透過 SSH 遠端控制 Server 上的 Docker，實現「本地編輯、遠端執行」的工作流程。

---

## 核心觀念與實作解析

### Docker Engine vs. Docker Desktop

這是兩個完全不同的產品：

| Docker Desktop   | Docker Engine        |
| ---------------- | -------------------- |
| 本地開發用            | Server 部署用           |
| 包含 GUI Dashboard | 純 CLI                |
| 包含許多額外工具         | 單一 daemon（dockerd）   |
| 授權軟體             | 開源軟體（Apache License） |

**Docker Engine** 是一個單一 binary（`dockerd`），可以在任何 Linux Server 上運行，只要該系統有 Linux Kernel。

> Docker Engine 也有 Windows 版本（`dockerd.exe`），可以運行 Windows Container，但本課程不涵蓋此主題。

---

### 支援的 Linux 發行版

Docker 官方有測試並支援的發行版清單，主要包括：

- Ubuntu
- Debian
- CentOS
- Red Hat Enterprise Linux
- Fedora

**不支援的系統：**

- BSD（不是 Linux）
- 一些較冷門的發行版

請參考 Docker 官方文件確認完整的支援清單。

---

### 安裝方式

#### 方法一：使用官方安裝腳本（推薦給學習者）

最簡單的方式是使用 `get.docker.com` 提供的腳本：

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

**這個腳本會自動：**

1. 偵測你的 Linux 發行版和版本
2. 移除舊版本的 Docker
3. 設定正確的套件庫來源
4. 安裝 Docker Engine 及相關工具

#### 方法二：手動安裝

如果你需要精確控制安裝過程，可以按照官方文件逐步執行，使用對應的套件管理器（apt、yum、apk 等）。

---

### 安裝後驗證

安裝完成後，執行以下指令確認：

```bash
docker version
```

成功時會顯示：

- **Client** 版本（Docker CLI）
- **Server** 版本（Docker Engine/daemon）

這表示你的 CLI 可以透過本地 socket 與 Engine 溝通。

---

### 登入 Docker Hub

與其他平台相同，建議登入以獲得更多 pull 次數：

```bash
docker login
```

輸入你的 Docker ID 和密碼。

**Pull 限制說明：**

| 狀態 | 限制 |
|------|------|
| 未登入 | ~100 pulls / 6 小時 |
| 登入免費帳號 | ~200 pulls / 6 小時 |
| 付費訂閱 | 無限制 |

---

### 遠端操作 Docker Engine 的核心概念

這是本單元最重要的部分。你不需要一直 SSH 進入 Server，而是可以**從本地端控制遠端的 Docker Engine**。

#### 傳統方式的問題

在 SSH tunnel 功能出現之前（約 2018 年），要遠端操作 Docker 只能：

1. **SSH 進入 Server**：所有工作都在 Server 上進行
2. **使用 TCP port**：預設無認證、無加密，安全風險極高

> TCP 方式曾導致嚴重的安全問題——任何人只要能連到該 port，就能獲得 root 權限。

#### 現代方式：SSH Tunnel

Docker 現在內建 SSH tunnel 支援，只要你具備 SSH 連線能力（key 或密碼認證），就可以透明地遠端操作 Docker。

---

### 設定遠端 Docker 操作

#### Step 1：在本地安裝 Docker CLI

你只需要安裝 **Docker CLI**，不需要完整的 Docker Desktop 或 Engine。

**各平台安裝方式：**

| 平台 | 安裝方式 |
|------|----------|
| macOS | `brew install docker`（只會安裝 CLI） |
| Linux | 使用 Docker 套件庫，安裝 `docker-ce-cli` 套件 |
| Windows | 使用 Chocolatey 安裝 |

> 如果你已經安裝了 Docker Desktop，就不需要額外安裝 CLI——Docker Desktop 已經包含了。

#### Step 2：設定 DOCKER_HOST 環境變數

這是關鍵步驟。透過環境變數告訴 Docker CLI 要連接到哪個 Server：

```bash
export DOCKER_HOST=ssh://username@hostname
```

**說明：**

- `username`：你在 Server 上的帳號
- `hostname`：Server 的 IP 位址或網域名稱
- 這個環境變數告訴 Docker CLI：「透過 SSH 連線到指定的 Server」

#### Step 3：測試連線

```bash
docker version
```

如果設定正確，你會看到：

- **Client**：顯示你本地系統的資訊（如 Darwin/macOS）
- **Server**：顯示 Linux（因為 Engine 運行在遠端 Server）

**這代表你的指令已經透過 SSH 傳送到遠端執行了！**

---

### 工作流程示意

```
本地端                                    遠端 Server
┌─────────────────┐                      ┌─────────────────┐
│ 程式碼          │                      │ Docker Engine   │
│ Docker CLI      │── SSH Tunnel ───────▶│ Container       │
│ 瀏覽器          │                      │ Image           │
└─────────────────┘                      └─────────────────┘
```

**你的工作環境：**

- 本地：程式碼、編輯器、Docker CLI、瀏覽器
- Server：Docker Engine、Container

---

### 切換回本地 Docker

如果要取消遠端設定：

```bash
unset DOCKER_HOST
```

這樣 Docker CLI 就會恢復連接本地的 Docker Engine（如果有的話）。

**進階技巧：Docker Context**

Docker 有一個 `docker context` 指令，可以儲存多組連線設定並快速切換，這是更進階的用法，會在課程後段介紹。

---

### 重要提醒：瀏覽器存取

當你在遠端 Docker 上執行 container 並開放 port 時，**瀏覽器不能使用 `localhost`**。

**正確做法：**

```
# 錯誤
http://localhost:8080

# 正確
http://<server-ip-or-hostname>:8080
```

因為 container 運行在遠端 Server 上，你需要連接到 Server 的 IP 位址。

---

### 取得課程 Repository

在**本地端** clone 課程 repository：

```bash
cd ~
git clone <repository URL>
```

所有的程式碼和設定檔都放在本地，只有 Docker 指令會透過 SSH 在遠端執行。

---

## 💡 重點摘要

- **Docker Engine 是開源的 daemon，專門用於 Linux Server 部署，與 Docker Desktop 是不同的產品。**
- **使用 `get.docker.com` 的安裝腳本是最快速簡單的安裝方式。**
- **透過設定 `DOCKER_HOST=ssh://user@host` 環境變數，可以從本地端透明地控制遠端 Docker Engine。**
- **本地端只需安裝 Docker CLI（不需要 Docker Desktop），程式碼也放在本地，只有 Docker 指令會遠端執行。**
- **瀏覽器存取 container 時，要使用 Server 的 IP 位址而非 localhost。**

---

## 🔑 關鍵字

Docker Engine, Docker CLI, SSH Tunnel, DOCKER_HOST, Remote Docker
