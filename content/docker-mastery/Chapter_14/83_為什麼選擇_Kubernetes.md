# 為什麼選擇 Kubernetes

## 📝 課程概述

本單元探討何時需要編排系統、如何評估編排系統的效益，以及在選擇 Kubernetes 後該如何決定使用哪個發行版。我們將學習一套簡單的公式來判斷編排系統的投資報酬率。

## 核心觀念與實作解析

### 你真的需要編排系統嗎？

編排系統越來越流行，但**不是運行 Container 的唯一方式**。回顧本課程學過的方式：

- `docker run` — 單一 Container
- Docker Compose — 多 Container 應用
- 雲端平台服務 — AWS、GCP 等
- 編排系統 — Swarm、Kubernetes

許多客戶仍使用單一節點 + AWS Auto Scaling + ELB，完全不用編排系統。

### 編排效益評估公式

講師提出一個簡單公式：

> **伺服器數量 × 變更頻率 = 編排效益**

| 情境 | 伺服器數量 | 變更頻率 | 編排效益 |
|------|-----------|---------|---------|
| 小型團隊 | 1-5 台 | 每月少於一次 | 低 |
| 中型團隊 | 10-50 台 | 每週部署 | 中 |
| 大型團隊 | 100+ 台 | 每天多次 | 高 |

編排系統的價值在於**自動化變更**與**監控狀態**。若你的環境變更頻率低，編排系統的維護成本可能超過其效益。

### 替代方案考量

若評估後發現編排系統效益不高，可考慮：

- **Elastic Beanstalk** (AWS)
- **Heroku**
- 其他 PaaS 服務

這些平台抽象化了基礎設施管理，適合小型團隊或獨立開發者。

### 編排系統選擇：主要平台

截至 2019 年，主要的編排平台：

| 平台 | 特性 |
|------|------|
| **Swarm** | Docker 內建，跨平台 |
| **Kubernetes** | 最廣泛支援，功能最豐富 |
| **ECS** | AWS 專屬 |
| **Cloud Foundry / Mesos** | 傳統方案 |

若需要**混合雲**或**多雲**支援，選擇只剩下 **Swarm** 與 **Kubernetes**。

### 選擇 Kubernetes 後的決策

#### 雲端託管 vs. 自行管理

**雲端託管 (Managed Kubernetes)**：
- AWS EKS、GKE、AKS 等
- 廠商負責 Control Plane 維運
- 你只需專注於工作負載

**自行管理 (Self-Managed)**：
- 使用發行版安裝在自己的伺服器
- 需自行維護 Control Plane
- 適合有特殊需求或地端資料中心

#### 發行版選擇

**CNCF 認證發行版**保證 API 相容性：

- Docker Enterprise
- OpenShift (Red Hat)
- Rancher
- VMware PKS
- Canonical (Ubuntu)

選擇考量：
- 現有供應商合約
- 團隊熟悉度
- 作業系統偏好
- 支援選項

### 為什麼使用發行版？

Kubernetes 開源版本缺少許多生產環境必需的元件：

```
Upstream Kubernetes
├── 核心 API ✓
├── 認證系統 ✗ (需自行整合)
├── Web UI ✗ (需自行安裝)
├── 網路方案 ✗ (需選擇 CNI)
├── 儲存方案 ✗ (需選擇 CSI)
└── 監控系統 ✗ (需自行建置)
```

發行版預先整合這些元件，讓你更快進入生產環境。

### Upstream 版本的定位

> Upstream Kubernetes 適合**學習**，不適合直接用於**生產**。

不同發行版在安全、網路、認證等方面的實作可能差異很大，學習一個不代表能無縫遷移到另一個。

## 💡 重點摘要

- 使用「伺服器數量 × 變更頻率」公式評估編排系統效益
- 混合雲或多雲需求應選擇 Swarm 或 Kubernetes
- 發行版預先整合生產環境必需元件，建議優先使用
- CNCF 認證發行版確保 API 相容性與工作負載可移植性
- Upstream 適合學習，生產環境應使用發行版

## 🔑 關鍵字

Kubernetes, Distribution, CNCF, Managed Service, Hybrid Cloud
