# Kubernetes 工作機制的奧秘

## 📝 課程概述
Kubernetes 是雲原生時代的操作系統，本課程深入剖析其 Master/Node 架構、核心組件與插件，揭示其強大自動化运维能力的秘密。

## 核心觀念與實作解析

### Kubernetes：雲原生時代的操作系統
從集群級別來看，Kubernetes 就是一個**操作系統**：
- **Linux**：管理單機的 CPU、内存、硬盘、网卡，調度單機进程
- **Kubernetes**：管理多台伺服器的計算資源，調度成千上萬的进程

**DevOps 的融合**：
- 在 Kubernetes裡只有一類人：**DevOps**
- 開發人員需考慮部署运维，运维人員需早期介入開發

### Kubernetes 基本架構
Kubernetes採用 **控制面/數據面（Control Plane/Data Plane）**架構：

| 節點類型 | 角色 | 功能 |
|----------|------|------|
| Master Node | 控制面 | 管理集群、运维監控應用 |
| Worker Node | 數據面 | 运行具體業務應用 |

**重要特性**：Master 和 Node 划分不是绝对的。小集群時 Master也可承擔 Node 工作（如minikube）。

### Master 節點組件（4個）

| 組件 | 功能 |比喻 |
|------|------|------|
| **apiserver** | 系統唯一入口，RESTful API + 验证授权 | 聯絡員 |
| **etcd** | 高可用分布式 Key-Value 数据库，持久化存儲 | 配置管理员 |
| **scheduler** | 容器编排，檢查节点资源状态，调度 Pod | 部署人员 |
| **controller-manager** | 维护容器和节点状态，故障检测、服务迁移、应用伸缩 | 监控运维人员 |

查看組件状态：
```bash
kubectl get pod -n kube-system
```

### Worker Node 組件（3個）

| 組件 | 功能 |比喻 |
|------|------|------|
| **kubelet** | Node 代理，状态报告、命令下发、启停容器 | 小管家 |
| **kube-proxy** | 网络代理，管理容器网络通信，转发 TCP/UDP | 小邮差 |
| **container-runtime** | 容器和镜像的实际使用者，创建容器 | 苦力 |

**重要**：Kubernetes 不限定 container-runtime 必须是 Docker，可替換成 containerd、CRI-O等。

查看 kube-proxy 和 kubelet：
```bash
minikube ssh
docker ps | grep kube-proxy
ps -ef | grep kubelet
```

### Kubernetes 工作流程
1. **kubelet** 定期向 **apiserver** 上报节点状态，存入 **etcd**
2. **kube-proxy** 实现TCP/UDP反向代理，让容器对外提供稳定服务
3. **scheduler** 通过apiserver得到节点状态，调度 Pod，kubelet启动容器
4. **controller-manager** 通过apiserver监控异常，调节恢复

### 插件（Addons）
Kubernetes 可安装附加功能扩展管理能力：
```bash
minikube addons list
```

**重要插件**：
- **DNS**：域名解析服务，服务发现和负载均衡的基础（必备）
- **Dashboard**：图形化操作界面，支持中文
```bash
minikube dashboard
```

## 💡 重點摘要
- Kubernetes 可視為集群級操作系統，管理應用和伺服器資源
- Master/Node 架構：Master 管理，Worker 跑業務
- Master 組件：apiserver（入口）、etcd（存儲）、scheduler（调度）、controller-manager（监控）
- Node 件：kubelet（代理）、kube-proxy（网络）、container-runtime（容器运行时）
- 必备插件：DNS（域名解析）、Dashboard（图形界面）

## 🔑 關鍵字
Master Node, Worker Node, apiserver, etcd, scheduler, controller-manager, kubelet, kube-proxy