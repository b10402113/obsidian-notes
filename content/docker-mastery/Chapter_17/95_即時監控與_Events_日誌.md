# 即時監控與 Events 日誌

## 📝 課程概述

本單元學習如何即時監控 Kubernetes 資源變化。我們將使用 `-w`（watch）選項與 `kubectl get events` 來觀察叢集中的即時事件，這對於診斷問題與理解 Kubernetes 的運作機制非常有幫助。

---

## 核心觀念與實作解析

### -w（watch）選項

`kubectl get` 支援 `-w` 或 `--watch` 選項，持續監控資源變化：

```bash
kubectl get pods -w
```

這個指令會：
1. 先輸出當前所有 Pod 的狀態
2. 然後持續監控，任何變化都會輸出新的一行

#### 與 Linux watch 指令的差異

| 指令 | 行為 |
| --- | --- |
| **Linux watch** | 每 N 秒重新執行指令，畫面刷新為當前狀態 |
| **kubectl -w** | 持續輸出變化事件，保留歷史記錄 |

> `kubectl -w` 輸出的是**事件流**而非當前狀態快照，適合觀察變化過程。

---

### 實驗：刪除 Pod 並觀察

開啟兩個終端機視窗：

**終端機 1（監控）**：
```bash
kubectl get pods -w
```

**終端機 2（刪除 Pod）**：
```bash
kubectl delete pod/my-apache-{hash}-{id}
```

觀察終端機 1 的輸出：
```
NAME                        READY   STATUS    RESTARTS   AGE
my-apache-xxx-aaa           1/1     Running   0          5m
my-apache-xxx-aaa           1/1     Terminating   0       5m
my-apache-xxx-bbb           0/1     Pending       0       0s
my-apache-xxx-bbb           0/1     ContainerCreating   0   0s
my-apache-xxx-bbb           1/1     Running   0          2s
my-apache-xxx-aaa           0/1     Completed   0        5m
```

> 因為我們使用 Deployment 管理 Pod，當 Pod 被刪除時，ReplicaSet Controller 會自動建立新 Pod 維持期望的 replica 數量。

---

### kubectl get events

Events 是 Kubernetes 的**獨立資源類型**，記錄叢集中發生的所有事件：

```bash
kubectl get events
```

輸出欄位：
- **LAST SEEN**：最後發生時間
- **TYPE**：Normal 或 Warning
- **REASON**：事件原因（Scheduled、Pulled、Started、Killing 等）
- **OBJECT**：相關資源
- **MESSAGE**：詳細訊息

#### --watch-only 選項

```bash
kubectl get events --watch-only
```

只顯示**新發生的事件**，不輸出歷史記錄。這對於即時診斷非常有用。

#### 實驗：觀察事件

**終端機 1**：
```bash
kubectl get events --watch-only
```

**終端機 2**：
```bash
kubectl delete pod/my-apache-{hash}-{id}
```

終端機 1 會顯示：
```
LAST SEEN   TYPE      REASON      OBJECT         MESSAGE
2s          Normal    Killing     pod/xxx        Stopping container httpd
1s          Normal    Successful  replicaset     Created pod: xxx
0s          Normal    Scheduled   pod/xxx        Successfully assigned default/xxx to node
0s          Normal    Pulled      pod/xxx        Container image "httpd:latest" already present
0s          Normal    Created     pod/xxx        Created container httpd
0s          Normal    Started     pod/xxx        Started container httpd
```

---

### Events 的特性與限制

**重要觀念**：

1. **Events 不等於 Container Logs** — Events 是 Kubernetes API 層級的事件，Container Logs 是應用程式輸出
2. **儲存位置**：Events 存在 etcd，有保留期限（通常 1 小時）
3. **非長期儲存**：不適合作為稽核或歷史查詢用途
4. **僅 API 可見**：只記錄 Kubernetes API 可感知的事件

> 生產環境需要集中式日誌解決方案（ELK、Loki 等）來處理 Container Logs。

---

### 診斷流程建議

當遇到問題時，建議的診斷順序：

```
1. kubectl get pods
   → 確認 Pod 狀態（Running? CrashLoopBackOff? Pending?）

2. kubectl describe pod/{name}
   → 查看 Events 區塊，了解排程、映像檔拉取、容器啟動情況

3. kubectl logs pod/{name}
   → 查看容器應用程式輸出

4. kubectl get events
   → 查看叢集層級的事件
```

---

## 💡 重點摘要

- **`kubectl get pods -w` 持續輸出變化事件流，而非當前狀態快照。**
- **Deployment 管理的 Pod 被刪除後，Controller 會自動重建以維持期望狀態。**
- **Events 是 Kubernetes API 層級的事件記錄，與 Container Logs 不同。**
- **`--watch-only` 只顯示新事件，適合即時診斷。**
- **Events 有保留期限，不適合作為長期稽核用途。**

---

## 🔑 關鍵字

watch, Events, --watch-only, kubectl get, Diagnostics, Normal, Warning
