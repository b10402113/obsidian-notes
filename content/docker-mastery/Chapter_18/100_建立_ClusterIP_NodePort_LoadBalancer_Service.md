# 使用 kubectl expose 建立 Service

## 📝 課程概述

本單元實作如何使用 `kubectl expose` 指令建立不同類型的 Service。我們將建立 ClusterIP、NodePort 與 LoadBalancer 三種類型，並驗證它們的運作方式。

---

## 核心觀念與實作解析

### 準備工作：建立 Deployment

首先建立一個用於測試的 Deployment：

```bash
# 建立 Deployment
kubectl create deployment httpenv --image bretfisher/httpenv

# 擴展為 5 個副本
kubectl scale deploy/httpenv --replicas 5
```

> `httpenv` 是一個簡單的 HTTP server，會回傳容器的環境變數，方便我們驗證請求被分配到哪個 Pod。

---

### 建立 ClusterIP Service

```bash
kubectl expose deployment httpenv --port 8888
```

參數說明：
- `deployment httpenv`：要暴露的資源類型與名稱
- `--port 8888`：Service 監聽的 port

#### 驗證 Service

```bash
kubectl get services
```

輸出會顯示：
- **NAME**：httpenv
- **TYPE**：ClusterIP
- **CLUSTER-IP**：虛擬 IP（如 10.96.0.1）
- **PORT**：8888/TCP

> ClusterIP 類型**僅在叢集內部可用**，外部無法直接存取。

#### 從叢集內部測試

使用臨時 Pod 進行測試：

```bash
kubectl run tmp-shell --rm -it --image bretfisher/netshoot -- bash
```

參數說明：
- `--rm`：Pod 結束後自動刪除
- `-it`：互動模式，取得 shell
- `--image`：使用的映像檔（netshoot 包含網路診斷工具）
- `--`：之後的參數為容器指令

在 Pod 內部執行：

```bash
curl httpenv:8888
```

> Service 名稱會自動成為 DNS 名稱，coreDNS 負責解析。

---

### 建立 NodePort Service

```bash
kubectl expose deployment httpenv --port 8888 \
  --name httpenv-np \
  --type NodePort
```

參數說明：
- `--name`：自訂 Service 名稱（避免與現有 Service 衝突）
- `--type NodePort`：指定類型為 NodePort

#### 查看 NodePort

```bash
kubectl get services
```

輸出範例：
```
NAME         TYPE        CLUSTER-IP      PORT(S)          
httpenv      ClusterIP   10.96.100.1     8888/TCP         
httpenv-np   NodePort    10.96.200.2     8888:32334/TCP   
```

**Port 格式說明**：

```
8888:32334/TCP
│    │
│    └── Node 上暴露的高位 Port（外部存取用）
└── 容器內部 Port
```

> 與 Docker/Swarm 的 port 對應格式**相反**！Kubernetes 是 `容器Port:NodePort`。

#### 從外部測試

**Linux**：可直接使用 `localhost:32334`

**Docker Desktop（Mac/Windows）**：
- 內建 vpnkit 轉發機制
- 可使用 `localhost:32334` 存取

```bash
curl localhost:32334
```

---

### 建立 LoadBalancer Service

LoadBalancer 需要**基礎設施提供商**支援。雲端環境會自動建立 ELB/ALB，Docker Desktop 則提供內建支援。

```bash
kubectl expose deployment httpenv --port 8888 \
  --name httpenv-lb \
  --type LoadBalancer
```

#### Docker Desktop 的 LoadBalancer

在 Docker Desktop 環境中：

```bash
kubectl get services
```

輸出範例：
```
NAME         TYPE           CLUSTER-IP      PORT(S)          
httpenv-lb   LoadBalancer   10.96.300.3     8888:31527/TCP
```

Docker Desktop 會：
- 在 `localhost:8888` 上提供服務
- 不需要使用高位 Port

```bash
curl localhost:8888
```

> 這是 Docker Desktop 特有的便利功能。正式環境需依賴雲端 Load Balancer。

---

### Service 的累加特性

當你建立 NodePort 或 LoadBalancer 時：

| 建立的類型 | 實際建立的層級 |
| --- | --- |
| **ClusterIP** | ClusterIP |
| **NodePort** | ClusterIP + NodePort |
| **LoadBalancer** | ClusterIP + NodePort + LoadBalancer |

這解釋了為什麼 LoadBalancer 的 PORT 欄位仍顯示 NodePort：

```
8888:31527/TCP
      ↑
      即使是 LoadBalancer，仍會建立 NodePort 作為內部層級
```

---

### 清理資源

```bash
kubectl delete deployment httpenv
kubectl delete service httpenv httpenv-np httpenv-lb
```

> 可以在單一指令中刪除多個資源，以空白分隔即可。

---

## 💡 重點摘要

- **`kubectl expose` 是建立 Service 的指令式方法，需指定資源類型、名稱與 port。**
- **ClusterIP 僅在叢集內部可用，需透過臨時 Pod 測試連線。**
- **NodePort 會在每個 Node 上開啟高位 Port（30000-32767），允許外部存取。**
- **LoadBalancer 需要雲端提供商支援，Docker Desktop 提供內建模擬。**
- **Port 格式為 `容器Port:NodePort`，與 Docker/Swarm 相反。**

---

## 🔑 關鍵字

kubectl expose, ClusterIP, NodePort, LoadBalancer, Service, port mapping
