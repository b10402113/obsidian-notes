# Container Logs 查詢技巧

## 📝 課程概述

本單元學習如何使用 `kubectl logs` 查詢容器的應用程式日誌。我們將理解 Kubernetes 日誌的儲存機制、各種查詢選項，以及何時需要更強大的日誌解決方案。

---

## 核心觀念與實作解析

### Kubernetes 日誌架構

**重要觀念**：Kubernetes **不儲存容器日誌**。

日誌實際儲存位置：
- 由 **Container Runtime** 管理（Docker、containerd、cri-o）
- 預設儲存在 **各 Node 的本地檔案系統**
- Kubernetes API 只是**代理轉送**日誌給使用者

```
kubectl logs → API Server → Kubelet → Container Runtime → 讀取日誌檔案
```

> 這是為什麼 `kubectl logs` 只適合小型叢集與開發環境。生產環境需要集中式日誌解決方案。

---

### 基本日誌查詢

#### 從 Deployment 查詢

```bash
kubectl logs deploy/my-apache
```

Kubernetes 會自動：
1. 找到該 Deployment 下的 ReplicaSet
2. 找到 ReplicaSet 下的 Pod
3. 選擇其中一個 Pod
4. 如果只有一個容器，顯示該容器日誌

> 當有多個 Pod 時，**無法保證取得哪一個 Pod 的日誌**。這適合快速檢查，但不適合精確診斷。

#### 從特定 Pod 查詢

```bash
# 先取得 Pod 名稱
kubectl get pods

# 查詢特定 Pod
kubectl logs pod/my-apache-{hash}-{id}
```

#### 指定容器

如果 Pod 內有多個容器：

```bash
kubectl logs pod/{name} -c {container-name}
```

容器名稱可透過以下方式取得：
- `kubectl describe pod/{name}`
- `kubectl get pod/{name} -o yaml`

---

### 常用選項

#### --follow（即時追蹤）

```bash
kubectl logs deploy/my-apache --follow
# 或簡寫
kubectl logs deploy/my-apache -f
```

持續輸出新日誌，類似 `tail -f`。

#### --tail（限制行數）

```bash
# 只顯示最後 1 行
kubectl logs deploy/my-apache --tail 1

# 顯示最後 100 行
kubectl logs deploy/my-apache --tail 100
```

#### 組合使用

```bash
# 即時追蹤最後 1 行之後的新日誌
kubectl logs deploy/my-apache --follow --tail 1
```

這是**最常用的診斷組合**，適合快速檢查是否有異常。

---

### 查詢多個 Pod 的日誌

#### 使用 Label Selector

```bash
kubectl logs -l app=my-apache
```

這會查詢所有帶有 `app=my-apache` 標籤的 Pod，並輸出所有日誌。

> 輸出是**各 Pod 日誌的原始拼接**，不會按時間順序交錯排列。這是為什麼大型系統需要集中式日誌。

#### --all-containers

```bash
kubectl logs pod/{name} --all-containers=true
```

查詢 Pod 內所有容器的日誌（合併輸出）。

---

### 日誌查詢的限制

`kubectl logs` 的不足：

| 限制 | 說明 |
| --- | --- |
| **無時間戳** | 預設不顯示日誌時間（需應用程式自行輸出） |
| **無順序保證** | 多 Pod/容器時，日誌可能未按時間排序 |
| **無搜尋功能** | 無法搜尋特定關鍵字 |
| **無持久化** | Pod 刪除後日誌消失 |
| **無跨 Node 彙整** | 需逐一查詢各 Node |

---

### Stern：進階日誌工具

當 `kubectl logs` 不夠用時，推薦使用 **stern**：

- GitHub 開源工具
- 自動收集多 Pod、多容器日誌
- 不同 Pod 以**顏色區分**
- 支援正則表達式過濾

```bash
# 安裝後
stern my-apache
```

---

### 清理資源

```bash
kubectl delete deploy/my-apache
```

---

## 💡 重點摘要

- **Kubernetes 不儲存容器日誌，日誌由 Container Runtime 存在各 Node 本地。**
- **`kubectl logs` 只是代理轉送機制，適合開發與小型叢集。**
- **`--follow --tail 1` 是最常用的診斷組合，即時查看新日誌。**
- **使用 Label Selector（`-l`）可查詢多 Pod 日誌，但輸出不保證時間順序。**
- **生產環境需搭配集中式日誌解決方案（ELK、Loki 等）。**

---

## 🔑 關鍵字

kubectl logs, Container Runtime, --follow, --tail, Label Selector, stern, Centralized Logging
