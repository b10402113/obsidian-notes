# StatefulSet：管理有狀態應用的利器

## 📝 課程概述

本課程探討 Kubernetes 中專門用於管理「有狀態應用」的 API 對象 StatefulSet。課程深入剖析有狀態應用的特性與需求，並詳細說明如何透過 StatefulSet 解決啟動順序、依賴關係、網絡標識等問題，結合 PersistentVolume 完整實現數據持久化與狀態恢復。

## 核心觀念與實作解析

### 理解「有狀態應用」vs「無狀態應用」

#### PersistentVolume 帶來的思考

PersistentVolume 為 Kubernetes 帶來了持久化存儲功能，讓應用能將關鍵數據落盤保存。當 Pod 發生意外崩潰時，只需重啟並掛載 Volume，再加載原數據就能**恢復之前的「狀態」繼續運行**。

從這個角度來看，**理論上任何應用都是有狀態的**，因為應用保存的數據就是它某個時刻的「運行狀態」。

#### 兩種應用類型的區別

**無狀態應用**：
- 狀態信息不重要，即使不恢復狀態也能正常運行
- 典型例子：Nginx Web 服務器
- 只處理 HTTP 請求，本身不生產數據（日志除外）
- 無論以什麼狀態重啟都能正常服務

**有狀態應用**：
- 运行狀態信息非常重要，如果因重啟丢失狀態是灾难性的
- 典型例子：Redis、MySQL 等数据库
- 「狀態」是内存或磁盘上产生的数据，是应用的核心价值
- 必須能夠將數據及時保存並恢復

#### Deployment + PersistentVolume 的局限性

雖然用 Deployment 保證高可用，用 PersistentVolume 存儲數據，可以部分達到管理「有狀態應用」的目的，但在集群化、分布式場景裡仍存在問題：

1. **多實例依賴關係**：master/slave、active/passive 等關係
2. **啟動順序**：需要依次啟動才能保證正常運行
3. **網絡標識**：外界客戶端需要使用固定的網絡標識訪問實例
4. **Pod 重啟後信息不變**：名字、IP 地址、域名必须保持稳定

使用 Deployment 時，多個實例之間是**無關的**：
- 啟動順序不固定
- Pod 名字隨機
- IP 地址隨機
- 域名隨機

这正是「無狀態應用」的特點，而「有狀態應用」需要更穩定的管理方式。

### StatefulSet 的設計與特性

Kubernetes 在 Deployment 的基礎上定義了新的 API對象 **StatefulSet**，專門用來管理有狀態的應用。

#### StatefulSet YAML 基本結構

StatefulSet 的 YAML 描述和 Deployment幾乎完全相同，但多了一個關鍵字段 `serviceName`：

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-sts

spec:
  serviceName: redis-svc    # 关键字段
  replicas: 2
  selector:
    matchLabels:
      app: redis-sts
  template:
    metadata:
      labels:
        app: redis-sts
    spec:
      containers:
      - image: redis:5-alpine
        name: redis
        ports:
        - containerPort: 6379
```

#### StatefulSet 的三個關鍵特性

**1. 固定的 Pod 名字（解決啟動順序）**

StatefulSet 管理的 Pod 不再是隨機名字，而是有**順序編號**，從 0開始：
- `redis-sts-0`
- `redis-sts-1`

Kubernetes 會按照這個順序**依次创建**（0号比1号的 AGE 長），解決了啟動顺序問題。

應用可以通過 `hostname` 或环境变量 `$HOSTNAME` 得到 Pod 名字，自行决定依赖关系：
- 先启动的 0号 Pod → 主实例
- 后启动的 1号 Pod → 从实例

**2. 稳定的网络标识（需要 Service 配合）**

为 StatefulSet 创建 Service 对象：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-svc    # 必须与 StatefulSet 的 serviceName 相同

spec:
  selector:
    app: redis-sts    # 标签必须与 StatefulSet 一致

  ports:
  - port: 6379
    protocol: TCP
    targetPort: 6379
```

Service 会为 StatefulSet 的 Pod 创建**稳定的域名**：
- 完整格式：`Pod名.服务名.名字空间.svc.cluster.local`
- 简写格式：`Pod名.服务名`

例如：
- `redis-sts-0.redis-svc`
- `redis-sts-1.redis-svc`

這些域名由 Service 维护，即使 Pod 的 IP 地址变化，域名也**稳定不变**。

**3. Headless Service 的使用**

对于 StatefulSet，Service 的负载均衡功能反而是**不必要的**，因为 Pod 已经有稳定的域名，外界访问不应通过 Service 这一层。

可以在 Service 中添加字段：

```yaml
spec:
  clusterIP: None    # 不分配 IP 地址
```

这样的 Service 称为 **Headless Service**，从安全和节约系统资源的角度考虑是更好的选择。

### StatefulSet 的数据持久化

#### volumeClaimTemplates 字段

为了强调持久化存储与 StatefulSet 的**一对一绑定关系**，Kubernetes 为 StatefulSet 定义了字段 `volumeClaimTemplates`，直接把 PVC 定义嵌入 StatefulSet 的 YAML 文件里。

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-pv-sts

spec:
  serviceName: redis-pv-svc

  volumeClaimTemplates:
  - metadata:
      name: redis-100m-pvc
    spec:
      storageClassName: nfs-client
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 100Mi

  replicas: 2
  selector:
    matchLabels:
      app: redis-pv-sts

  template:
    metadata:
      labels:
        app: redis-pv-sts
    spec:
      containers:
      - image: redis:5-alpine
        name: redis
        ports:
        - containerPort: 6379

        volumeMounts:
        - name: redis-100m-pvc
          mountPath: /data
```

#### volumeClaimTemplates 的作用

- 创建 StatefulSet 时，会为**每个 Pod 自动创建 PVC**
- PVC 的命名有规律：`PVC名字-StatefulSet名字-序号`
  - 例如：`redis-100m-pvc-redis-pv-sts-0`、`redis-100m-pvc-redis-pv-sts-1`
- 即使 Pod 被销毁，因为名字不变，能找到对应的 PVC，再次绑定使用之前存储的数据

#### 状态恢复验证

创建带持久化功能的 StatefulSet 后：

```bash
kubectl exec -it redis-pv-sts-0 -- redis-cli
# 设置数据：SET a 111, SET b 222

# 删除 Pod 模拟意外
kubectl delete pod redis-pv-sts-0
```

StatefulSet 会很快创建新的 Pod，名字、网络标识都一模一样。由于 NFS 网络存储挂载到 `/data` 目录，Redis 会定期把数据落盘保存，新 Pod 再次挂载时会从备份文件恢复数据，**内存里的数据完全恢复原状**。

## 💡 重點摘要

- **有狀態應用的核心特徵**：运行状态信息至关重要，丢失状态是灾难性的，如 Redis、MySQL 等数据库
- **Deployment 的不足**：Pod 名字、IP、域名随机，启动顺序不固定，无法满足有狀態应用需求
- **StatefulSet 的三大特性**：固定名字（有序编号）、稳定域名（Service 配合）、启动顺序可控
- **Headless Service**：设置 `clusterIP: None`，不分配 IP，Pod 直接通过域名访问
- **volumeClaimTemplates**：内嵌定义 PVC，自动为每个 Pod 创建存储卷，实现数据持久化与状态恢复

## 🔑 關鍵字

StatefulSet, PersistentVolume, Service, volumeClaimTemplates, Headless Service