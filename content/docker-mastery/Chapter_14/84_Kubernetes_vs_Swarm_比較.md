# Kubernetes vs Swarm：如何選擇

## 📝 課程概述

本單元比較兩大主流容器編排平台：Docker Swarm 與 Kubernetes。我們將從技術特性、學習曲線、生態系支援等面向分析兩者的優勢，幫助你根據實際需求做出選擇。

## 核心觀念與實作解析

### 一句話總結

| 平台 | 核心特色 |
|------|---------|
| **Swarm** | Easy（簡單易用） |
| **Kubernetes** | Powerful（功能強大） |

> Swarm 容易建置、容易加入節點、容易上手、容易管理。Kubernetes 則有更多功能、彈性與自訂選項。

### 兩者的共同點

- 都是 Container Orchestrator
- 都運行在 Container Runtime 之上（預設 Docker）
- 都有廠商與開發團隊支援
- 都能在地端與雲端運行

### Docker Swarm 的優勢

#### 1. 內建於 Docker

```
Docker Engine = Container Runtime + Swarm Orchestrator
```

單一廠商支援，Runtime 與 Orchestrator 整合度高。

#### 2. 資源佔用低

Swarm 在 Docker 之上的額外資源消耗非常少。

#### 3. 開發者友善

設計時考慮開發者工作流程，Ops 與 Dev 都容易使用。

#### 4. 簡單易管理

可輕鬆擴展到數百甚至數千節點，不需要大型團隊維護。

#### 5. 80/20 法則

> Swarm 大約擁有 Kubernetes 20% 的功能，但能解決 80% 的使用情境。

開箱即用的功能：
- Overlay Networking
- Ingress Routing Mesh
- 加密通訊
- Secrets 管理
- Service 探索

#### 6. 跨平台支援最廣

Docker Engine 支援的環境比任何其他工具都多：

| 平台 | 支援狀態 |
|------|---------|
| Linux | ✓ |
| Windows Server | ✓ (2016+) |
| Mainframe | ✓ |
| ARM (32/64 bit) | ✓ |
| IoT / Raspberry Pi | ✓ |

#### 7. 安全性開箱即用

建立 Swarm 時自動：
- Mutual TLS 認證
- Control Plane 加密
- 資料庫加密（保護 Secrets）

#### 8. 除錯容易

使用與 Docker 相同的工具與日誌：
- `docker logs`
- `docker events`
- daemon logs

### Swarm 適用情境

- 剛開始學習容器編排
- 個人或小型團隊
- 需要快速驗證概念
- 不確定是否需要 Kubernetes 的全部功能

> 講師建議：除非你確定必須使用 Kubernetes，否則先從 Swarm 開始。當你發現 Swarm 無法滿足需求時，再考慮 Kubernetes。

### Kubernetes 的優勢

#### 1. 最廣泛的雲端與廠商支援

每個雲端供應商都提供 Kubernetes 服務：

- AWS EKS
- Google GKE
- Azure AKS
- 其他數十家

#### 2. 最廣泛的生態系支援

許多廠商採取 **Kubernetes First** 策略：

> 廠商會優先確保產品支援 Kubernetes，再考慮其他平台。

例如 Jenkins Enterprise 提供 Kubernetes 部署範例與工具，Swarm 雖然也能運行 Jenkins，但獲得的支援較少。

#### 3. 更多功能與使用案例

更多可調整的參數、更多功能、更多第三方整合，能覆蓋更多邊緣案例。

#### 4. 「選擇 IBM 不會被解僱」效應

> "No one ever got fired for buying IBM"

這是過去 IT 產業的說法：當無法決定時，選擇最知名的品牌最安全。

Kubernetes 已成為 CIO/CTO 的「勾選項目」，即使沒有技術需求，也可能因為管理層要求而必須使用。

### 決策因素：不只是技術

現實中的決策往往不只是技術考量：

| 決策因素 | 說明 |
|---------|------|
| 政治決策 | 「管理層讀了雜誌，決定要用 Kubernetes」 |
| 廠商鎖定 | 已有廠商合約，直接用他們的方案 |
| 團隊經驗 | 團隊成員有過往經驗 |
| 工作機會 | Kubernetes 技能需求較高 |

> 講師在實體工作坊最常聽到的答案是：「因為老闆叫我用這個。」

### 建議：兩者都學

身為 DevOps 從業人員，建議：

1. 能夠安裝兩者
2. 能夠在兩者上部署應用
3. 了解基本的應用部署、Ingress 設定、應用更新流程

這樣你才能在團隊中提供最好的建議。

## 💡 重點摘要

- Swarm 強調簡單易用，適合入門與小型團隊
- Kubernetes 生態系最完整，適合複雜需求與企業環境
- Swarm 解決 80% 的使用案例，剩餘 20% 需要 Kubernetes
- 決策常受政治、廠商關係、團隊經驗影響，非純技術因素
- 建議兩者都學，才能做出最佳判斷

## 🔑 關鍵字

Swarm, Kubernetes, Orchestrator, Ecosystem, Kubernetes First
