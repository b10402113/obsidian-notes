# Kubernetes「棄用 Docker」是怎麼回事？

## 📝 課程概述

本課程深入解析 Kubernetes 棄用 Docker 的來龍去脈，從 CRI 標準的引入到 containerd 的誕生，釐清這項變革對開發者的實際影響，幫助學員理解容器技術的演進趨勢。

## 核心觀念與實作解析

### 歷史背景：從依賴到獨立

- **2014 年**：Kubernetes 誕生時選擇在 Docker 上運行，因為 Docker 當時如日中天
- **2016 年**：Kubernetes 1.0 正式可用，加入 CNCF 成為首個託管專案
- **2016 年底**：Kubernetes 1.5 引入 **CRI**（Container Runtime Interface）

### 什麼是 CRI

**CRI**（Container Runtime Interface）是 Kubernetes 定義的容器運行時介面標準：

- 採用 **ProtoBuffer** 和 **gRPC** 技術
- 規範 kubelet 如何呼叫容器運行時管理容器和映像檔
- 與原本的 Docker 呼叫完全不兼容
- 目的是與 Docker 解耦，允許接入其他容器技術（如 rkt、kata）

### dockershim：過渡方案

為了相容現有 Docker 環境，Kubernetes 在 kubelet 和 Docker 之間加入「適配器」：

```
kubelet → dockershim → Docker Engine → containerd → 容器
```

這個適配器被稱為 **shim**（墊片），負責將 Docker 介面轉換成符合 CRI 標準的介面。

### containerd 的誕生

面對 Kubernetes 的壓力，Docker 採取「斷臂求生」策略：

- 將 Docker Engine 重構，拆分成多個模組
- Docker daemon 部分捐獻給 CNCF，形成 **containerd**
- containerd 作為 CNCF 託管專案，符合 CRI 標準
- Docker Engine 仍調用 containerd，但外部介面保持不變

### 兩種調用鏈的比較

| 方式 | 調用鏈 | 特點 |
|------|--------|------|
| 傳統 Docker | CRI → dockershim → Docker → containerd | 環節多、損耗大 |
| 直接 containerd | CRI → containerd | 簡潔、性能更好 |

根據 Kubernetes 官方測試數據，containerd 1.1 相比 Docker 18.03：
- Pod 啟動延遲降低約 20%
- CPU 使用率降低 68%
- 記憶體使用率降低 12%

### 正式「棄用 Docker」的時間線

- **2020 年**：Kubernetes 1.20 宣布將棄用 dockershim
- **2022 年 5 月**：Kubernetes 1.24 正式移除 dockershim 程式碼

**重要釐清**：所謂「棄用 Docker」實際上只是棄用 **dockershim** 這個適配器，而非 Docker 本身。

### 實際影響

1. **映像檔不受影響**：容器映像檔格式已標準化（OCI 規範），Docker 映像仍可在 Kubernetes 正常使用
2. **工具變化**：使用 `crictl` 替代 `docker ps` 查看容器（若使用 kubectl 則無影響）
3. **環境隔離**：Kubernetes 直接使用 containerd 後，與 Docker 形成獨立環境，彼此無法存取對方管理的容器

### Docker 的未來

Docker 雖在容器編排戰爭落敗，但仍具強韌生命力：

- 映像檔仍可正常使用，開發測試、CI/CD 流程不受影響
- Docker 仍是完整的軟體產品線，包含映像檔構建、分發、測試等服務
- Docker 公司接管 dockershim 程式碼，建立 **cri-dockerd** 專案維持相容性

## 💡 重點摘要

- CRI 是 Kubernetes 定義的容器運行時介面標準，目的是與特定容器技術解耦
- containerd 是 Docker 公司捐獻給 CNCF 的核心元件，符合 CRI 標準
- Kubernetes 1.24 正式移除 dockershim，直接使用 containerd 管理容器
- 「棄用 Docker」實際上是棄用 dockershim 適配器，Docker 映像檔仍可正常使用
- 開發者可繼續使用 Docker 進行本地開發，Kubernetes 底層運行時與開發者無直接關係

## 🔑 關鍵字

CRI, containerd, dockershim, OCI 規範, 容器運行時