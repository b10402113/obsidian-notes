# Docker 的三項核心創新

## 📝 課程概述

本單元是整個 Docker 學習路徑的起點，老師帶領我們認識 Docker 自 2013 年誕生以來最重要的三項創新：**Container Image**、**Registry** 與 **Container**。這三者構成了著名的 "Build, Ship, Run" 工作流程，也是雲原生生態系統的基石。

---

## 核心觀念與實作解析

### 三項核心創新概覽

Docker 的出現並非憑空而來，而是基於三個緊密相扣的創新：

1. **Container Image（容器映像檔）** — 通用應用程式打包器
2. **Registry（映像檔倉庫）** — 應用程式分發機制
3. **Container（容器）** — 隔離的執行環境

這三者的關係密不可分，業界稱之為 **"Build, Ship, Run"**：
- **Build**：將軟體打包成 Image
- **Ship**：將 Image 傳輸到 Registry
- **Run**：從 Image 建立 Container 並執行

> CNCF（Cloud Native Computing Foundation）旗下的絕大多數工具，都預設你已經採用這種 container workflow。

---

### Container Image：通用應用程式打包器

在 Docker 出現之前，我們有無數的 Package Manager，但從來沒有一個能夠**跨作業系統、跨平台、且與程式語言無關**的通用打包方案。

**Container Image 的核心特性：**
- 只要目標環境執行 Linux 或 Windows，Image 就能運作
- 不包含 Host 的驅動程式或 Linux Kernel，只包含應用程式所需的內容

#### Dockerfile：Image 的配方

老師透過一個簡單的 Dockerfile 範例說明 Image 的組成：

```dockerfile
FROM python
RUN pip install flask
WORKDIR /app
COPY . .
```

每個 Dockerfile 都必須以 `FROM` 開頭，指定基礎 Image。後續的每個指令（如 `RUN`、`COPY`）都會建立一個 **Layer（層）**。

**Layer 的運作原理：**
- 每個 Layer 是獨立的 tarball，包含檔案、目錄、權限與 metadata
- Image 建立時，會逐一執行 Dockerfile 中的指令
- 背景實際上是為每個 Layer 建立臨時 Container、執行指令、再儲存結果

> Dockerfile 中的每個指令都會建立新的 Layer，這些 Layer 最終堆疊成完整的 Image。

**OCI Image Standard：** Docker Image 規格如此成功，現在已成為業界標準，稱為 OCI（Open Container Initiative）Image Standard。

---

### Registry：應用程式分發機制

每個應用程式打包系統都需要一套分發機制，Docker Registry 就是這個角色。

**Registry 的特性：**
- 現在稱為 **OCI Distribution Spec** 或 **OCI Registry Specification**
- 幾乎所有雲端平台都有提供：Docker Hub（最普及）、GitHub、GitLab、Bitbucket
- 也可以自建 Private Registry（有數十種開源與商業方案可選）

#### Image 傳輸流程

每個 Image 都有獨特的 **SHA Hash**，能確保在不同系統上驗證其完整性與一致性。

```
開發機器                Registry                 目標伺服器
   │                      │                         │
   │   docker push        │                         │
   │ ──────────────────>  │                         │
   │                      │      docker pull        │
   │                      │  <──────────────────────│
   │                      │                         │
```

**核心保證：** 無論你在哪個 Linux Distribution 上建立 Image，只要目標環境是 Linux，就能以完全相同的方式執行。例如：在 CentOS 建立的 Image，在 Ubuntu 上執行時，應用程式與依賴項目完全一致。

---

### Container：隔離的執行環境

完成 Build 與 Ship 後，終於來到 Run 階段。

#### Docker Engine 架構

在 Linux Server 上，Docker Engine 會在背景執行，負責：
- 下載 Image
- 驗證 Image 完整性
- 建立與管理 Container

#### Container 的隔離特性

當執行 `docker run` 時，Container 會獲得：

| 隔離資源 | 說明 |
|---------|------|
| **Namespace** | Linux 特性，防止 Container 看見 Host OS 的其他部分 |
| **File System** | 空白檔案系統，只包含 Image 中的檔案 |
| **IP Address** | 獨立的虛擬網卡與 IP |
| **Process List** | 獨立的程序列表 |

**重點：** 每個 Container 都是獨立的 Namespace，Container 1 與 Container 2 互不干擾。即使執行相同的 Image，它們也是完全隔離的執行實體。

#### 高可用性場景

若在多台 Server 上執行相同的 Container，就能建立 High Availability 架構：
- 多個 Container 可同時執行，提供備援或負載平衡
- 檔案系統、權限、metadata 在每個 Container 中都一致

---

### 為何 "Build, Ship, Run" 如此重要？

這三個步驟是 Container 與 Kubernetes、Helm 等雲原生工具的核心工作流程：

1. 開發者將應用程式打包成 Image
2. Image 傳輸到 Registry
3. 測試環境與生產環境從 Registry 取得 Image 並執行

> 無論是開發、測試還是部署，這三個步驟確保了應用程式在不同環境間的一致性。

---

## 💡 重點摘要

- **Docker 的三項核心創新：Container Image（打包）、Registry（分發）、Container（執行），構成 "Build, Ship, Run" 工作流程。**
- **Container Image 是跨平台、跨語言的通用打包格式，現已標準化為 OCI Image Standard。**
- **Registry 提供 Image 的儲存與分發，Docker Hub 是最普及的選擇，各大雲平台也都有提供。**
- **Container 透過 Linux Namespace 實現隔離，每個 Container 有獨立的檔案系統、IP 與程序空間。**
- **"Build, Ship, Run" 是雲原生生態系統的基石，Kubernetes、Helm 等工具都預設採用此工作流程。**

---

## 🔑 關鍵字

Docker, Container Image, Registry, Container, Dockerfile, Layer, OCI, Namespace
