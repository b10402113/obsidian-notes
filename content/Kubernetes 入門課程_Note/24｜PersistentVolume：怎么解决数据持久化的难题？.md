# PersistentVolume：解決數據持久化的難題

## 📝 課程概述
本節課進入 Kubernetes「高級篇」，探討如何解決 Pod 銷毀後數據丟失的問題。透過 PersistentVolume (PV)、PersistentVolumeClaim (PVC) 和 StorageClass 三個 API 對象，實現真正的數據持久化存儲，並以 HostPath 類型進行實作演示。

## 核心觀念與實作解析

### 為什麼需要持久化存儲？

Pod 裡的容器由鏡像產生，鏡像本身是只讀的，進程讀寫磁盤只能使用**臨時存儲空間**。一旦 Pod 銷毀，臨時存儲立即回收，數據也隨之丟失。為了保證 Pod 銷毀後重建數據依然存在，需要讓 Pod 使用真正的「虛擬盤」。

### PV/PVC/StorageClass 三者關係

**PersistentVolume (PV)** 是 Kubernetes 對存儲設備的抽象，隱藏底層實現（Ceph、GlusterFS、NFS、本地磁盤等），屬於集群系統資源，與 Node 平級，Pod 只有使用權。

**PersistentVolumeClaim (PVC)** 是 Pod 向系統申請存儲資源的代理，說明需求容量、訪問模式等參數，Kubernetes 會查找最合適的 PV 並「綁定」。

**StorageClass** 抽象特定類型的存儲系統，在 PVC 和 PV 之間充當「協調人」，幫助 PVC 找到合適的 PV。

### 生活類比理解

| 概念 | 類比 |
|------|------|
| PVC | 打電話申請 10 張紙 |
| StorageClass | 前台裡各種品牌規格的辦公用紙 |
| PV 綁定 | 前台挑選一包 A4 紙並登記 |
| PV | 最終到手的 A4 紙包 |

### PV 的 YAML 定義

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: host-10m-pv

spec:
  storageClassName: host-test    # 存儲類型名稱，可自定義
  accessModes:
  - ReadWriteOnce                # 訪問模式
  capacity:
    storage: 10Mi                # 容量，注意使用 Ki/Mi/Gi
  hostPath:
    path: /tmp/host-10m-pv/      # 本地路徑
```

**accessModes 三種訪問模式**：
- `ReadWriteOnce`：可讀可寫，只能被一個節點上的 Pod 挂載
- `ReadOnlyMany`：只讀，可被任意節點上的 Pod 多次挂載
- `ReadWriteMany`：可讀可寫，可被任意節點上的 Pod 多次挂載

**注意**：訪問模式限制的對象是**節點**而不是 Pod。

### PVC 的 YAML 定義

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: host-5m-pvc

spec:
  storageClassName: host-test    # 與 PV 匹配
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Mi               # 申请容量
```

PVC 表示「申請」，用 `resources.requests` 表示希望申請的容量。若申請 5MB 但系統只有 10MB 的 PV，Kubernetes 會把 10MB 的 PV 分配出去；若找不到符合要求的 PV，PVC 會處於 `Pending` 狀態。

### Pod 中挂載 PVC

```yaml
spec:
  volumes:
  - name: host-pvc-vol
    persistentVolumeClaim:
      claimName: host-5m-pvc     # 指定 PVC 名稱

  containers:
  - volumeMounts:
    - name: host-pvc-vol
      mountPath: /tmp            # 挂載進容器的路徑
```

### HostPath 的限制

HostPath 類型 PV 數據存儲在節點本地，速度快，但**不能跟隨 Pod 遷移**。若 Pod 重建時被調度到其他節點，持久化功能失效。適用於測試或 DaemonSet 等與節點關係密切的應用。

## 💡 重點摘要

- **PV** 是存儲設備的抽象，由系統管理員維護，Pod 只有使用權
- **PVC** 是 Pod 的代理，向系統申請存儲資源，Kubernetes 會查找最合適的 PV 並綁定
- **StorageClass** 抽象存儲類型，簡化 PV/PVC 的綁定過程
- **capacity 使用 Ki/Mi/Gi**（基數 1024），不要寫成 KB/MB/GB 否則容量會對不上
- **HostPath** 數據存儲在節點本地，Pod 調度到其他節點會失效

## 🔑 關鍵字

PersistentVolume, PersistentVolumeClaim, StorageClass, HostPath, accessModes