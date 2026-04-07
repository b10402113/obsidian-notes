# Kubernetes Service 四大類型概覽

## 📝 課程概述

本單元介紹 Kubernetes Service 的核心概念與四種類型。Service 是為一組 Pod 提供穩定存取端點的抽象層，讓其他服務能夠可靠地連接到這些 Pod。理解四種 Service 類型的差異，是正確暴露服務的關鍵。

---

## 核心觀念與實作解析

### 什麼是 Service？

當你建立 Pod 後，它們**不會自動獲得 DNS 名稱或穩定的 IP 位址**。Service 的角色就是：

- 提供**一致的存取端點**（DNS 名稱 + IP）
- 讓叢集內部或外部的服務能夠連接到 Pod
- 由 **coreDNS** 負責名稱解析

> Service 與 Swarm 的 service 概念不同。在 Swarm 中，service 是部署單位；在 Kubernetes 中，Service 是「存取端點」的抽象層，部署單位是 Deployment。

---

### 四種 Service 類型

#### 1. ClusterIP（預設類型）

**特性**：
- **僅在叢集內部可用**
- 獲得虛擬 IP 位址與 DNS 名稱
- 適合叢集內部服務對服務的通訊

```
┌─────────────────────────────────────┐
│           Kubernetes Cluster        │
│  ┌─────────┐       ┌─────────────┐  │
│  │ Pod A   │ ───→  │ Service     │  │
│  └─────────┘       │ (ClusterIP) │  │
│                    │     ↓       │  │
│                    │  ┌───────┐  │  │
│                    │  │ Pods  │  │  │
│                    │  └───────┘  │  │
│                    └─────────────┘  │
└─────────────────────────────────────┘
```

> 這是最基本的 Service 類型，所有其他類型都建立在 ClusterIP 之上。

#### 2. NodePort

**特性**：
- 在**每個 Node 上開啟一個高位 Port**
- 允許外部流量透過 Node IP + Port 進入
- Port 範圍預設為 30000-32767

```
外部流量 → Node IP:3xxxx → Service → Pods
```

**適用場景**：
- 資料中心環境，沒有雲端 Load Balancer
- 測試與開發環境
- 需要外部存取但不需要正式域名

> NodePort 是 **ClusterIP + NodePort** 的組合，會自動建立 ClusterIP。

#### 3. LoadBalancer

**特性**：
- 需要**基礎設施提供商**支援（AWS ELB、GCP Load Balancer、DigitalOcean LB 等）
- 自動建立 ClusterIP 與 NodePort
- 透過 API 控制外部 Load Balancer

```
┌──────────────────────────────────────────┐
│         外部 Load Balancer (AWS ELB)      │
│                    ↓                      │
│              NodePort (3xxxx)             │
│                    ↓                      │
│              ClusterIP                    │
│                    ↓                      │
│                 Pods                      │
└──────────────────────────────────────────┘
```

**Docker Desktop 特例**：
- 內建 Load Balancer 支援
- 可直接在 localhost 上使用指定 port

> LoadBalancer 是 **ClusterIP + NodePort + 外部 Load Balancer** 的三層疊加。

#### 4. ExternalName

**特性**：
- **與流量進入無關**
- 用於叢集內部服務存取外部服務
- 在 coreDNS 中建立 CNAME 記錄

**適用場景**：
- 遷移期間，將外部服務 DNS 對應到內部
- 無法控制外部 DNS 時，在叢集內部管理名稱解析

---

### Service 類型的疊加關係

重要觀念：**Service 類型是累加的**

```
ClusterIP (基礎)
    ↓
NodePort = ClusterIP + NodePort
    ↓
LoadBalancer = ClusterIP + NodePort + External LB
```

這表示：
- 建立 NodePort 時，會自動建立 ClusterIP
- 建立 LoadBalancer 時，會自動建立 ClusterIP 與 NodePort

---

### 第五種方式：Ingress

Ingress 是另一種暴露服務的方式，專門用於 **HTTP/HTTPS 流量**。後續課程會詳細介紹。

---

## 💡 重點摘要

- **Service 是為 Pod 提供穩定存取端點的抽象層，由 coreDNS 負責名稱解析。**
- **ClusterIP 是預設類型，僅在叢集內部可用，是所有類型的基礎。**
- **NodePort 在每個 Node 開啟高位 Port，允許外部流量進入。**
- **LoadBalancer 需要雲端提供商支援，自動控制外部負載平衡器。**
- **Service 類型是累加的：LoadBalancer 包含 NodePort，NodePort 包含 ClusterIP。**

---

## 🔑 關鍵字

ClusterIP, NodePort, LoadBalancer, ExternalName, coreDNS, Service
