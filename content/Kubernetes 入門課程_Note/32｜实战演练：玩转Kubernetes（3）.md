# 實戰演練：玩轉 Kubernetes（3）

## 📝 課程概述
本節課是「高級篇」的總結與實戰演練，回顧 API 對象、應用管理、集群管理三大核心主題，並透過兩個實戰專案：優化 WordPress 網站的 MariaDB 持久化存儲，以及部署 Kubernetes Dashboard 並配置 Ingress HTTPS 訪問，綜合應用所學知識。

## 核心觀念與實作解析

### 高級篇知識要點回顧

#### API 對象

**PersistentVolume 系列**：PV 是持久化存儲的抽象，StorageClass 分類存儲設備，PVC 向系統申請存儲資源。動態存儲卷透過 StorageClass 綁定 Provisioner，根據 PVC 自動創建 PV。

**StatefulSet**：管理有狀態應用的 API 對象，三大關鍵能力：Pod 順序編號、穩定域名（如 `maria-sts-0.maria-svc`）、`volumeClaimTemplates` 存儲模板。

####應用管理

**滾動更新**：`kubectl apply` 觸發更新，`kubectl rollout history` 查看歷史，`kubectl rollout undo` 回退版本。

**資源配額與檢查探針**：資源配額限制 CPU/內存，三種探針 Startup/Liveness/Readiness 監控容器狀態。

#### 集群管理

**名字空間**：`ResourceQuota` 為集群切分資源池，限制 CPU、內存、存儲容量和 API 對象數量。

**系統監控**：Metrics Server 收集核心資源指標，Prometheus 是雲原生監控事實標準。

**網絡通信**：Kubernetes 定義「IP-per-pod」平坦網絡模型，Flannel 使用 Overlay 模式，Calico 使用 Route 模式。

### 實戰一：WordPress 網站優化

將 MariaDB 從 Deployment 改為 **StatefulSet**，掛載 NFS 動態存儲卷：

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: maria-sts
spec:
  serviceName: maria-svc           # Headless Service
  volumeClaimTemplates:            # PVC 模板
  - metadata:
      name: maria-100m-pvc
    spec:
      storageClassName: nfs-client
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 100Mi
```

WordPress ConfigMap 需修改資料庫連接地址為 `maria-sts-0.maria-svc`。

### 實戰二：部署 Dashboard

**部署步驟**：

1. 下載並應用 Dashboard YAML：`kubectl apply -f dashboard.yaml`

2. 生成自簽名證書並創建 Secret：
   ```bash
   openssl req -x509 -days 365 -out k8s.test.crt -keyout k8s.test.key \
     -newkey rsa:2048 -nodes -sha256 -subj '/CN=k8s.test'

   kubectl create secret tls dash-tls -n kubernetes-dashboard \
     --cert=k8s.test.crt --key=k8s.test.key
   ```

3. 配置 Ingress（HTTPS）：
   ```yaml
   metadata:
     annotations:
       nginx.org/ssl-services: "kubernetes-dashboard"
   spec:
     tls:
       - hosts: [k8s.test]
         secretName: dash-tls
   ```

4. 創建管理員賬號並獲取 Token：
   ```bash
   kubectl describe secrets -n kubernetes-dashboard admin-user-token-xxxx
   ```

5. 訪問 `https://k8s.test:30443` 用 Token 登錄

## 💡 重點摘要

- **StatefulSet** 適用於有狀態應用，提供順序編號、穩定域名、存儲模板三大關鍵能力
- **滾動更新** 通過擴容和縮容同步進行實現零停機，支持版本歷史查看和回退
- **Dashboard HTTPS** 需配置 TLS 證書、Ingress annotations 指定後端 HTTPS 服務
- **NFS 動態存儲** 透過 StorageClass 綁定 Provisioner 實現 PV 自動創建
- 進階改進方向：添加健康檢查、資源配額、自動水平伸縮

## 🔑 關鍵字

StatefulSet, PersistentVolumeClaim, Dashboard, Ingress, NFS