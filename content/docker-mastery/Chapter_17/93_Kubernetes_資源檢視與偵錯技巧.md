# Kubernetes 資源檢視與偵錯技巧

## 📝 課程概述

本單元學習如何檢視與診斷 Kubernetes 資源。我們將掌握 `kubectl get` 的各種輸出格式、`kubectl describe` 的詳細報告功能，以及如何即時監控資源變化。這些是日常操作 Kubernetes 必備的核心技能。

---

## 核心觀念與實作解析

### 準備工作環境

在開始前，確保有一個運行中的 Deployment：

```bash
kubectl create deployment my-apache --image httpd
kubectl scale deploy/my-apache --replicas 2
```

---

### kubectl get 的輸出格式

`kubectl get` 是最常用的查詢指令，支援多種輸出格式。

#### 基本查詢

```bash
# 查看常用資源
kubectl get all

# 查看特定資源
kubectl get deploy/my-apache
kubectl get pods
```

#### -o wide：擴充資訊

```bash
kubectl get deploy/my-apache -o wide
```

額外顯示：
- 容器名稱
- Image 來源
- Selector（標籤選擇器）

> **Labels 與 Selectors** 是 Kubernetes 資源關聯的關鍵機制。Deployment 透過 selector 找到對應的 ReplicaSet，ReplicaSet 再找到對應的 Pod。

#### -o yaml：完整資源定義

```bash
kubectl get deploy/my-apache -o yaml
```

這會輸出 Kubernetes **對該資源的所有認知**，即存在 etcd 中的完整記錄。

**YAML 結構解析**：

| 區塊 | 內容 |
| --- | --- |
| **kind** | 資源類型 |
| **apiVersion** | API 版本 |
| **metadata** | 名稱、namespace、labels、uid 等 |
| **spec** | 期望狀態（使用者定義） |
| **status** | 實際狀態（系統維護） |

> 這是後續學習宣告式 YAML 配置的基礎。所有的 Dashboard、GUI 工具本質上都是在呈現這份 YAML 的資訊。

#### 其他輸出格式

```bash
# JSON 格式（適合機器處理）
kubectl get deploy/my-apache -o json

# Go template（進階自動化）
kubectl get deploy/my-apache -o go-template='...'

# JSONPath（進階查詢）
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
```

---

### kubectl describe：人類友善的報告

`kubectl describe` 提供**摘要式的人類可讀報告**，並包含事件歷程。

#### Deployment Describe

```bash
kubectl describe deploy/my-apache
```

輸出重點：
- **Pod Template**：Pod 的配置模板
- **Rolling Update Strategy**：更新策略
- **Replicas**：期望/實際/可用數量
- **Events**：該資源的事件歷程

#### Pod Describe

```bash
# 先取得 Pod 名稱
kubectl get pods

# Describe 特定 Pod
kubectl describe pod/my-apache-{hash}-{id}
```

輸出重點：
- **IP**：Pod 的叢集內部 IP
- **Controlled By**：上層管理者（ReplicaSet）
- **Containers**：容器詳細配置
- **Conditions**：Pod 狀態條件（應全部為 True）
- **Events**：排程、映像檔拉取、容器啟動等事件

> 當 Pod 無法啟動時，Events 區塊是首要診斷點——可以看到映像檔拉取失敗、資源不足等錯誤訊息。

#### Node Describe

```bash
kubectl get nodes
kubectl describe node/docker-desktop
```

輸出重點：
- **Labels**：節點標籤（用於 Pod 排程）
- **Conditions**：節點健康狀態
- **Capacity**：CPU、記憶體、儲存容量
- **Allocatable**：可分配資源
- **Pods**：節點上運行的 Pod 清單與資源佔用

> 在大型叢集中，如果 Pod 無法被排程，通常是因為沒有節點滿足資源需求（requests/limits）。

---

### Describe vs Get -o yaml

| 面向 | kubectl describe | kubectl get -o yaml |
| --- | --- | --- |
| **目的** | 人類閱讀的摘要報告 | 完整的資源定義 |
| **資訊量** | 精選重要資訊 | 全部資訊 |
| **Events** | ✅ 包含 | ❌ 不包含（events 是獨立資源） |
| **適用場景** | 快速診斷、狀態檢查 | 匯出配置、除錯 spec 問題 |

---

## 💡 重點摘要

- **`kubectl get -o yaml` 輸出資源的完整定義，包含 spec 與 status。**
- **`kubectl describe` 提供人類友善的摘要報告，並包含 events 歷程。**
- **Labels 與 Selectors 是資源關聯的核心機制，Deployment 用它找到 ReplicaSet 與 Pod。**
- **Pod 的 Events 區塊是診斷啟動失敗的首要線索。**
- **Node describe 可查看資源容量與已分配資源，幫助診斷排程失敗問題。**

---

## 🔑 關鍵字

kubectl get, kubectl describe, Labels, Selectors, Events, YAML, Status, Spec
