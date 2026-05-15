# 使用 kubeadm 搭建多節點 Kubernetes 集群

## 📝 課程概述

本課程從 minikube 過渡到更真實的生產環境，使用 kubeadm 工具搭建多節點 Kubernetes 集群。透過 Master-Worker 架構的實際部署，讓學員掌握真實雲原生環境的搭建技能。

## 核心觀念與實作解析

### 為什麼需要 kubeadm

minikube 雖然方便，但本質上是「玩具」等級的單機環境，隱藏了許多細節，無法體驗真實生產環境的需求。**kubeadm** 是 Kubernetes 社區推出的官方部署工具：

- 目標：在集群環境中輕鬆部署接近生產級質量的 Kubernetes
- 原理：用容器和鏡像封裝 Kubernetes 各組件（apiserver、etcd、scheduler 等）
- 優勢：只需少數命令（`init`、`join`、`upgrade`、`reset`）即可完成集群管理

### 集群架構設計

課程採用最小化多節點架構：

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Console   │    │   Master    │    │   Worker    │
│  (控制台)    │───▶│  (管理節點)  │◀───│  (工作節點)  │
│   kubectl   │    │  2核/4GB    │    │   1核/1GB   │
└─────────────┘    └─────────────┘    └─────────────┘
```

- **Master 節點**：運行 apiserver、etcd、scheduler、controller-manager，配置要求較高（至少 2 核 CPU、4GB 記憶體）
- **Worker 節點**：只運行業務應用，配置可較低
- **Console 節點**：邏輯概念，安裝 kubectl 作為管理入口，可復用現有機器

### 安裝前準備工作

Kubernetes 對系統有特殊要求，需完成四項準備：

#### 1. 修改主機名

每個節點的 `hostname` 必須唯一，修改 `/etc/hostname`：

```bash
sudo vi /etc/hostname
```

#### 2. 修改 Docker 配置

將 cgroup 驅動改為 `systemd`：

```bash
cat <<EOF | sudo tee /etc/docker/daemon.json
{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
         "max-size": "100m"
    },
    "storage-driver": "overlay2"
}
EOF

sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### 3. 啟用網路模組

啟用 `br_netfilter` 模組，讓 Kubernetes 能夠檢查和轉發網路流量：

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward=1
EOF

sudo sysctl --system
```

#### 4. 關閉 Swap 分區

提升 Kubernetes 性能：

```bash
sudo swapoff -a
sudo sed -ri '/\sswap\s/s/^#?/#/' /etc/fstab
```

### 安裝 kubeadm

在 Master 和 Worker 節點上都要安裝 kubeadm、kubelet、kubectl：

```bash
# 添加國內鏡像源
sudo apt install -y apt-transport-https ca-certificates curl
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

sudo apt update
sudo apt install -y kubeadm=1.23.3-00 kubelet=1.23.3-00 kubectl=1.23.3-00
sudo apt-mark hold kubeadm kubelet kubectl
```

### 下載 Kubernetes 組件鏡像

Kubernetes 組件鏡像存放在 `gcr.io`，國內訪問困難，需提前下載：

```bash
# 查看所需鏡像
kubeadm config images list --kubernetes-version v1.23.3

# 從國內鏡像站下載並改名
repo=registry.aliyuncs.com/google_containers

for name in `kubeadm config images list --kubernetes-version v1.23.3`; do
    src_name=${name#k8s.gcr.io/}
    src_name=${src_name#coredns/}
    docker pull $repo/$src_name
    docker tag $repo/$src_name $name
    docker rmi $repo/$src_name
done
```

### 安裝 Master 節點

使用 `kubeadm init` 初始化：

```bash
sudo kubeadm init \
    --pod-network-cidr=10.10.0.0/16 \
    --apiserver-advertise-address=192.168.10.210 \
    --kubernetes-version=v1.23.3
```

重要參數說明：
- `--pod-network-cidr`：設置集群內 Pod 的 IP 地址段
- `--apiserver-advertise-address`：指定 apiserver 對外服務的 IP
- `--kubernetes-version`：指定 Kubernetes 版本

安裝完成後需執行：

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**重要**：保存 `kubeadm join` 命令輸出，其他節點加入集群需要 token 和 CA 憑證。

### 安裝 Flannel 網路插件

Master 節點初始狀態為 `NotReady`，需安裝網路插件：

1. 下載 `kube-flannel.yml`
2. 修改 `net-conf.json` 中的 `Network` 為 `--pod-network-cidr` 設置的地址段
3. 執行安裝：

```bash
kubectl apply -f kube-flannel.yml
```

### 安裝 Worker 節點

使用之前保存的 join 命令：

```bash
sudo kubeadm join 192.168.10.210:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

### 驗證集群

```bash
kubectl get node
# 應看到兩個節點都是 Ready 狀態

kubectl run ngx --image=nginx:alpine
kubectl get pod -o wide
# 應看到 Pod 運行在 Worker 節點上
```

## 💡 重點摘要

- **kubeadm** 是 Kubernetes 官方部署工具，能快速搭建生產級別的集群環境
- 安裝前必須完成四項準備：修改主機名、配置 Docker cgroup、啟用網路模組、關閉 Swap
- Kubernetes 組件鏡像在 `gcr.io`，國內需從鏡像站下載後改名
- Master 節點用 `kubeadm init`，Worker 節點用 `kubeadm join`
- 網路插件（如 Flannel）是集群正常運作的必要組件

## 🔑 關鍵字

kubeadm, Master-Worker架構, Flannel, CNI網路插件, kubeadm init/join