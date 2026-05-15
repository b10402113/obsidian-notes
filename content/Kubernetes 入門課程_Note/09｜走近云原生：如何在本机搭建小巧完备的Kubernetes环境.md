# 在本機搭建 Kubernetes 環境

## 📝 課程概述
容器技術解決了應用打包分發問題，但面對複雜生產環境仍需容器編排技術。本課程介紹 Kubernetes 的起源、概念，以及如何使用 minikube 在本機搭建完整的 Kubernetes 環境。

## 核心觀念與實作解析

### 為什麼需要容器編排（Container Orchestration）
容器技術三大核心要素：**容器、鏡像、倉庫**，實現「一次開發，到處運行」。但生產環境需求複雜：服務發現、負載均衡、狀態監控、健康檢查、擴容縮容、應用遷移、高可用等。面對數百台伺服器、上千容器，**容器之上的管理調度工作就是容器編排**。

### Kubernetes 的起源
- **源自 Google 內部的 Borg 系統**：2014 年 Google 將 Borg 用 Go語言重寫並開源
- **CNCF（雲原生基金會）**：Google 聯合 Linux 基金會成立，Kubernetes 成為種子項目
- **戰勝競爭對手**：Apache Mesos、Docker Swarm，成為容器編排領域的唯一霸主

### Kubernetes 能為我們做什麼
Kubernetes 是**生產級別的容器編排平台和集群管理系統**：
- 創建、調度容器
- 监控、管理伺服器
- 讓中小型公司具備輕鬆運維海量計算節點的能力

### minikube 環境搭建
Kubernetes 官網推薦的本地環境工具：**kind** 和 **minikube**

| 工具 | 特點 | 建議用途 |
|------|------|----------|
| kind | 功能少、速度快 | 有經驗用戶快速開發測試 |
| minikube | 小而美、功能完善 | **學習研究** |

**minikube 特點**：
- 執行文件不到 100MB
- 運行鏡像約 1GB
- 集成 Kubernetes 大多數功能特性
- 支援 Dashboard、GPU、Ingress、Istio 等插件

### 實作：搭建 minikube 環境

**安裝 minikube**：
```bash
# Intel x86_64
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Apple arm64 (M1)
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-arm64

sudo install minikube /usr/local/bin/
```

**安裝 kubectl**（操作 Kubernetes 的客戶端工具）：
```bash
minikube kubectl  # minikube 提供簡化安裝方式
alias kubectl="minikube kubectl --"  # 建議建立別名
source <(kubectl completion bash)    # 啟用命令自動補全
```

**啟動 Kubernetes 集群**：
```bash
minikube start --kubernetes-version=v1.23.3
minikube status
minikube node list
```

**運行第一個 Pod**：
```bash
kubectl run ngx --image=nginx:alpine
kubectl get pod
```

## 💡 重點摘要
- 容器編排解決容器之上管理調度問題，Kubernetes 是此領域事實標準
- Kubernetes 源自 Google Borg 系統，CNCF 雲原生基金會保駕護航
- minikube 是學習 Kubernetes 的最佳工具，小而美、功能完善
- kubectl 是操作 Kubernetes 的客戶端工具，用法與 docker類似
- **雲原生**：應用開發、部署、运维都向 Kubernetes 看齊，使用容器、微服務、聲明式 API

## 🔑 關鍵字
Kubernetes, minikube, kubectl, Container Orchestration, CNCF