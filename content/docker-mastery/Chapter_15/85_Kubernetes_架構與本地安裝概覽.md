# Kubernetes 架構與本地安裝

## 📝 課程概述

本單元是進入 Kubernetes 實作前的關鍵準備。我們將深入理解 Kubernetes 的核心架構元件、術語定義，以及如何在本地環境建立學習用的 Kubernetes 叢集。這些基礎觀念是後續所有實作的基石。

---

## 核心觀念與實作解析

### Kubernetes 術語定義

在深入架構之前，我們先釐清幾個核心術語：

| 術語 | 定義 |
| --- | --- |
| **Kubernetes / K8s** | 整個容器編排系統的統稱，"K8s" 中的 "8" 代表 K 與 S 之間的 8 個字母 |
| **kubectl** | 官方命令列工具，用來與 Kubernetes API 溝通（可讀作 kube-control、kube-cuddle 或 kubectl，官方文件目前統一稱 kubectl） |
| **Node** | 叢集中的伺服器，與 Swarm 的 node 概念相同 |
| **Kubelet** | 運行在每個 node 上的代理程式（agent），負責與 Kubernetes master 溝通並管理本地容器執行期 |

> 與 Docker Swarm 不同，Kubernetes 需要獨立的 Kubelet agent，因為 Docker Engine 本身不內建 Kubernetes 的管理功能。

---

### Kubernetes 架構：Control Plane 與 Node

Kubernetes 的架構設計與 Swarm 有許多相似之處，但元件更加細緻。核心精神是 **「每個元件只做一件事，並且做好」**。

#### Control Plane（Master Nodes）

Control Plane 是叢集的大腦，負責所有決策與狀態管理。包含以下核心元件：

| 元件 | 功能 |
| --- | --- |
| **etcd** | 分散式 key-value 儲存系統，存放叢集所有狀態資料。使用 RAFT 協定，與 Swarm 一樣需要奇數節點以確保共識 |
| **API Server** | 叢集的統一入口，所有指令都透過 API 進入系統 |
| **Scheduler** | 決定 Pod 應該被調度到哪個 node 上執行 |
| **Controller Manager** | 持續監控叢集狀態，比對「期望狀態」與「實際狀態」的差異，並採取行動達成期望 |
| **coreDNS** | 提供叢集內部的 DNS 解析服務 |

#### Worker Nodes

每個 worker node 需要運行：

- **Kubelet**：與 Control Plane 溝通的代理程式
- **kube-proxy**：管理 node 的網路規則

> 所有節點都運行在 Docker 或其他容器執行期（containerd、cri-o）之上。

---

### 單節點 vs 多節點架構

在學習環境中，我們通常使用**單一節點**，所有 Control Plane 元件與 worker 元件都在同一台機器上運行。

在生產環境中，則會將角色分離：

- **Master 節點**：運行 Control Plane 元件，建議 3 個（奇數）以確保容錯能力
- **Worker 節點**：運行應用程式容器

> 與 Swarm 相同，Master 節點也可以同時作為 Worker 使用，但生產環境通常會將角色分離。

---

### 本地安裝選項比較

Kubernetes 的安裝方式多元，以下針對不同學習情境提供最佳建議：

| 環境 | 推薦工具 | 優點 |
| --- | --- | --- |
| **Docker Desktop 使用者** | Docker Desktop 內建 Kubernetes | 一鍵啟用、自動安裝 kubectl、可快速開關 Kubernetes |
| **Docker Toolbox 使用者** | minikube | 與 Docker Toolbox 體驗相似，使用 VirtualBox 作為虛擬化後端 |
| **Linux 原生環境** | Microk8s | 使用 Snap 安裝，自動設定 kubectl |
| **無法安裝軟體** | Katacoda | 瀏覽器中直接學習，已預先建好環境 |

#### Docker Desktop（最佳選擇）

如果你已經有 Docker Desktop，這是最簡單的方式：

1. 在偏好設定中勾選 Enable Kubernetes
2. 自動安裝與 VM 版本匹配的 kubectl
3. 提供快速開關功能，節省系統資源

#### minikube

適合 Docker Toolbox 使用者：

```bash
# 下載後執行
minikube start

# 常用指令與 Docker Machine 類似
minikube stop
minikube delete
```

> 注意：minikube **不會自動安裝 kubectl**，需要另外下載。

#### Microk8s（Linux）

使用 Snap 安裝：

```bash
sudo snap install microk8s --classic

# 使用 microk8s 專屬指令
microk8s.kubectl get nodes

# 建議設定 alias
alias kubectl='microk8s.kubectl'
# 或更簡短
alias k='microk8s.kubectl'
```

---

### Kubernetes 核心物件

在開始實作前，必須理解幾個核心物件：

#### Pod

**Pod 是 Kubernetes 最小的部署單位。**

- 我們不直接部署容器，而是部署 Pod
- 一個 Pod 可以包含一個或多個容器
- 幾乎不會單獨建立 Pod，而是透過 **Controller** 來管理

#### Controller

Controller 是一個持續運行的控制迴圈，確保「實際狀態」符合「期望狀態」：

| Controller 類型 | 用途 |
| --- | --- |
| **Deployment** | 最常用的控制器，管理無狀態應用 |
| **ReplicaSet** | 確保指定數量的 Pod 複本正在運行 |
| **StatefulSet** | 管理有狀態應用 |
| **DaemonSet** | 確保每個 node 都運行一個 Pod 複本 |
| **Job / CronJob** | 執行一次性或定時任務 |

> Deployment 與 Swarm 的 service 概念相似，但它管理的是 ReplicaSet，ReplicaSet 再管理 Pod。

#### Service

Kubernetes 的 Service 與 Swarm 的 service **意義不同**：

- **Service** 是為一組 Pod 提供穩定的存取端點（DNS 名稱與 port）
- 當使用 Deployment 部署多個 Pod 複本時，Service 提供統一的入口

#### Namespace

Namespace 是一種**視圖過濾機制**，並非安全功能：

- 用來組織與篩選命令列看到的資源
- Docker Desktop 預設使用 `default` namespace，隱藏系統容器
- 可以切換 namespace 來查看不同群組的資源

---

## 💡 重點摘要

- **Kubernetes 架構採用「每個元件只做一件事」的設計哲學，Control Plane 包含 API、Scheduler、Controller Manager、etcd、coreDNS 等多個獨立元件。**
- **kubectl 是與 Kubernetes API 溝通的官方命令列工具，是所有操作的核心。**
- **Docker Desktop 內建 Kubernetes 是最方便的學習環境，可一鍵啟用並快速開關。**
- **Pod 是 Kubernetes 最小部署單位，實務上透過 Controller（如 Deployment）來管理。**
- **Kubernetes 的 Service 是為 Pod 提供穩定存取端點，與 Swarm 的 service 概念不同。**

---

## 🔑 關鍵字

Kubernetes, kubectl, Control Plane, etcd, Pod, Deployment, Service, Namespace, Kubelet
