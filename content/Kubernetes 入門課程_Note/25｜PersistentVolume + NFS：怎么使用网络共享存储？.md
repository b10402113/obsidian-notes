# PersistentVolume + NFS：網絡共享存儲實戰

## 📝 課程概述

本課程承接上節課的 PersistentVolume 基礎概念，進一步探討如何在 Kubernetes 中使用網絡存儲系統 NFS。課程詳細說明了靜態存儲卷與動態存儲卷的配置方式，並透過 NFS Provisioner 實現存儲資源自動化管理，解決大規模集群中存儲分配的難題。

## 核心觀念與實作解析

### 為何需要網絡存儲？

在上節課中我們使用了 HostPath 作為存儲卷，但這種方式存在一個致命缺陷：**存儲卷只能在本機使用**。由於 Kubernetes 中的 Pod 經常會在集群裡「漂移」，當 Pod 被調度到其他節點時，原先節點上的本地存儲就無法訪問了。

要解決這個問題，我們需要使用**網絡存儲系統**，讓 Pod 無論在哪個節點運行，都能通過網絡訪問同一個存儲設備。網絡存儲讓數據真正實現了持久化，不再受 Pod 調度位置的影響。

### NFS 系統架構與安裝

**NFS（Network File System）** 是一個經典的網絡存儲系統，擁有近 40 年的發展歷史，基本上已成為各種 UNIX 系統的標準配置。

NFS 採用 **Client/Server 架構**：
- **Server 端**：提供存儲服務的主機，安裝 NFS 服務端
- **Client 端**：需要使用存儲的主機，安裝 NFS 客戶端工具

#### 安裝 NFS Server

在 Ubuntu 系統上安裝 NFS 服務端：

```bash
sudo apt -y install nfs-kernel-server
```

安裝完成後，需要指定一個**存儲目錄**作為網絡共享目錄。課程範例使用 `/tmp/nfs`：

```bash
mkdir -p /tmp/nfs
```

接著配置 NFS 訪問權限，修改 `/etc/exports` 文件：

```bash
/tmp/nfs 192.168.10.0/24(rw,sync,no_subtree_check,no_root_squash,insecure)
```

這行配置指定了：
- 共享目錄路徑
- 允許訪問的網段
- 權限參數（讀寫、同步、不壓制 root 權限等）

配置完成後，執行以下命令：

```bash
sudo exportfs -ra              # 讓配置生效
sudo exportfs -v               # 驗證配置
sudo systemctl start nfs-server    # 啟動服務
sudo systemctl enable nfs-server   # 設置開機自啟
```

使用 `showmount -e 127.0.0.1` 檢查 NFS 的網絡掛載情況。

#### 安裝 NFS Client

在 Kubernetes 集群的**每個節點**上都需要安裝 NFS 客戶端：

```bash
sudo apt -y install nfs-common
```

安裝後可用 `showmount -e <NFS_SERVER_IP>` 測試連接。手動掛載測試：

```bash
mkdir -p /tmp/test
sudo mount -t nfs 192.168.10.208:/tmp/nfs /tmp/test
touch /tmp/test/x.yml    # 在客戶端創建文件
```

到 NFS Server 上檢查 `/tmp/nfs` 目錄，應該能看到同樣的文件，說明 NFS 安裝成功。

### 靜態存儲卷配置

有了 NFS 存儲系統後，我們可以手動創建 PersistentVolume（靜態存儲卷）。

#### 創建 PV 對象

PV 的 YAML 描述文件關鍵字段：

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-1g-pv

spec:
  storageClassName: nfs
  accessModes:
    - ReadWriteMany    # NFS 支持多節點同時讀寫
  capacity:
    storage: 1Gi

  nfs:
    path: /tmp/nfs/1g-pv
    server: 192.168.10.208
```

**重點說明**：
- `storageClassName: nfs` - 標識這是 NFS 類型的存儲
- `accessModes: ReadWriteMany` - NFS 特性決定支持多節點同時訪問
- `nfs.path` 和 `nfs.server` - **必須正確配置**，否則 PV 會處於 pending 狀態

#### 創建 PVC 對象

PVC 向系統申請存儲資源：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-static-pvc

spec:
  storageClassName: nfs
  accessModes:
    - ReadWriteMany

  resources:
    requests:
      storage: 1Gi
```

創建 PVC 後，Kubernetes 會根據描述找到最合適的 PV 並進行**綁定**。

#### 在 Pod 中使用 PVC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-static-pod

spec:
  volumes:
  - name: nfs-pvc-vol
    persistentVolumeClaim:
      claimName: nfs-static-pvc

  containers:
    - name: nfs-pvc-test
      image: nginx:alpine
      ports:
        - containerPort: 80

      volumeMounts:
        - name: nfs-pvc-vol
          mountPath: /tmp
```

Kubernetes 會自動執行 NFS 掛載，將 NFS 共享目錄掛載到 Pod 內的 `/tmp`。

### 動態存儲卷與 Provisioner

#### 靜態存儲卷的局限性

雖然網絡存儲解決了 Pod 漂移的問題，但**手動管理 PV 仍存在諸多問題**：

1. **運維負擔重**：在大集群中，每天可能有數百上千個 PVC 需求，管理員需要手動創建對應的 PV
2. **容量難以精確控制**：容易出現空間不足或浪費的情況
3. **響應不及時**：存儲分配工作可能大量積壓

#### 動態存儲卷的概念

**動態存儲卷**通過 StorageClass 綁定一個 **Provisioner 對象**，讓計算機代替人工自動管理存儲、創建 PV。

對應地，前面手動創建的 PV 稱為**靜態存儲卷**。

#### 部署 NFS Provisioner

NFS 的 Provisioner 是 **NFS subdir external provisioner**，以 Pod 形式運行在 Kubernetes 中。需要部署三個 YAML 文件：

1. **rbac.yaml** - 權限控制配置，需將名字空間改為 `kube-system`
2. **class.yaml** - StorageClass 定義
3. **deployment.yaml** - Provisioner 的 Pod 部署配置

**deployment.yaml 的關鍵修改**：

```yaml
spec:
  template:
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        ...
        env:
          - name: PROVISIONER_NAME
            value: k8s-sigs.io/nfs-subdir-external-provisioner
          - name: NFS_SERVER
            value: 192.168.10.208      # NFS Server IP
          - name: NFS_PATH
            value: /tmp/nfs            # NFS 共享目錄
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.10.208      # NFS Server IP
            path: /tmp/nfs              # NFS 共享目錄
```

由於原鏡像 `k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner:v4.0.2` 拉取困難，可改用 Docker Hub 鏡像 `chronolaw/nfs-subdir-external-provisioner:v4.0.2`。

部署命令：

```bash
kubectl apply -f rbac.yaml
kubectl apply -f class.yaml
kubectl apply -f deployment.yaml
```

### 使用 NFS 動態存儲卷

#### StorageClass 定義

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client

provisioner: k8s-sigs.io/nfs-subdir-external-provisioner
parameters:
  archiveOnDelete: "false"    # 自動回收存儲空間
```

關鍵字段說明：
- `provisioner` - 指定使用的 Provisioner
- `parameters` - 調節 Provisioner 運行參數

也可以自定義 StorageClass，如添加 `onDelete: "retain"` 暫時保留分配的存儲。

#### 創建 PVC（無需手動創建 PV）

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-dyn-10m-pvc

spec:
  storageClassName: nfs-client
  accessModes:
    - ReadWriteMany

  resources:
    requests:
      storage: 10Mi    # 申請 10MB 存儲空間
```

#### Pod 掛載動態存儲卷

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-dyn-pod

spec:
  volumes:
  - name: nfs-dyn-10m-vol
    persistentVolumeClaim:
      claimName: nfs-dyn-10m-pvc

  containers:
    - name: nfs-dyn-test
      image: nginx:alpine
      ports:
        - containerPort: 80

      volumeMounts:
        - name: nfs-dyn-10m-vol
          mountPath: /tmp
```

創建 PVC 和 Pod 後，**NFS Provisioner 會自動創建一個 PV**，大小剛好是 PVC 申請的 10MB。NFS 服務器的共享目錄下也會自動生成對應的子目錄。

### 靜態存儲卷 vs 動態存儲卷對比

| 特性 | 靜態存儲卷 | 動態存儲卷 |
|------|-----------|-----------|
| PV 創建方式 | 手動創建 | Provisioner 自動創建 |
| 運維負擔 | 高，需管理員維護 | 低，自動化管理 |
| 存儲分配 | 預先分配，可能浪費 | 按需分配 |
| 適用場景 | 小規模集群 | 大規模集群 |

## 💡 重點摘要

- **網絡存儲解決 Pod 漂移問題**：相比 HostPath，網絡存儲讓數據不再受 Pod 調度位置限制，實現真正的持久化
- **NFS 採用 Client/Server 架構**：需要在 Server 端安裝 nfs-kernel-server，在每個集群節點安裝 nfs-common 客戶端
- **靜態存儲卷需手動創建 PV**：在 PV YAML 中正確配置 NFS Server IP 和共享目錄路徑是關鍵
- **動態存儲卷通過 Provisioner 實現自動化**：StorageClass 綁定 Provisioner，自動創建 PV 並完成綁定
- **動態存儲卷按需分配存儲**：無需管理員手動維護，適合大規模集群場景

## 🔑 關鍵字

NFS, PersistentVolume, StorageClass, Provisioner, PVC