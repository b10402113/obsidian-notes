# 网络通信：CNI 网络接口标准

## 📝 課程概述

本課程深入探討 Kubernetes 的網絡通信基礎。課程詳細說明 Kubernetes 的「IP-per-pod」網絡模型、CNI（Container Networking Interface）標準接口，以及 Flannel、Calico、Cilium 等常见网络插件的工作原理与实现方式，讓我們理解 Pod 如何在集群中实现跨主机通信。

## 核心觀念與實作解析

### Kubernetes 的網絡模型

#### Docker 网络的回顾

Docker 有三种网络模式：null、host、bridge。

最常用的 **bridge 网络模式**：
- Docker 创建网桥 docker0
- 默认私有网段 172.17.0.0/16
- 每个容器创建虚拟网卡对（veth pair）
- 两个虚拟网卡分别「插」在容器和网桥上
- 容器之间可以互联互通

Docker 的网络方案简单有效，但**只局限在单机环境**，跨主机通信非常困难（需要端口映射和网络地址转换）。

#### Kubernetes 的「IP-per-pod」网络模型

针对 Docker 的网络缺陷，Kubernetes 提出自己的网络模型，有 4 点基本假设：

1. **集群里的每个 Pod 都会有唯一的 IP 地址**
2. **Pod 里的所有容器共享这个 IP 地址**
3. **集群里的所有 Pod 都属于同一个网段**
4. **Pod 可以基于 IP 地址直接访问另一个 Pod，不需要 NAT（网络地址转换）**

这是一个「平坦」的网络模型，让 Pod 摆脱主机的硬限制：
- Pod 相当于一台虚拟机，直连互通
- 容易实施域名解析、负载均衡、服务发现
- 以前的运维经验能够直接使用
- 对应用的管理和迁移非常友好

### CNI 标准：网络插件接口

#### CNI 的定义

Kubernetes 定义的网络模型很完美，但落地实现不容易。所以 Kubernetes 制定了标准：**CNI（Container Networking Interface）**。

CNI 为网络插件定义了一系列通用接口，开发者遵循规范就可以接入 Kubernetes：
- 为 Pod 创建虚拟网卡
- 分配 IP 地址
- 设置路由规则
- 实现「IP-per-pod」网络模型

#### CNI 插件的实现技术分类

依据实现技术的不同，CNI 插件大致分成三种：

**1. Overlay 模式**
- 原意是「覆盖」，构建工作在真实底层网络之上的「逻辑网络」
- 把原始的 Pod 网络数据封包，通过下层网络发送
- 到目的地再拆包
- **特点**：对底层网络要求低，适应性强
- **缺点**：有额外的传输成本，性能较低

**2. Route 模式**
- 在底层网络之上工作，但没有封包和拆包
- 使用系统内置的路由功能实现 Pod 跨主机通信
- **特点**：性能高
- **缺点**：对底层网络的依赖性比较强，底层不支持就无法工作

**3. Underlay 模式**
- 直接用底层网络实现 CNI
- Pod 和宿主机在一个网络里，Pod 和宿主机是平等的
- **特点**：性能最高
- **缺点**：对底层的硬件和网络的依赖性最强，不够灵活

#### 常见的 CNI 插件

**Flannel**
- 由 CoreOS 公司（已被 Redhat 收购）开发
- 最早是 Overlay 模式（使用 UDP 和 VXLAN 技术）
- 后来用 Host-Gateway 技术支持了 Route 模式
- **特点**：简单易用，最流行的 CNI 插件
- **缺点**：性能方面表现不太好，一般不建议在生产环境使用

**Calico**
- Route 模式的网络插件
- 使用 **BGP 协议（Border Gateway Protocol）** 维护路由信息
- **特点**：性能比 Flannel 好，支持多种网络策略
- **功能**：数据加密、安全隔离、流量整形

**Cilium**
- 较新的网络插件，同时支持 Overlay 和 Route 模式
- 深度使用 Linux **eBPF 技术**，在内核层次操作网络数据
- **特点**：性能很高，灵活实现各种功能
- 2021 年加入 CNCF，成为孵化项目
- 非常有前途的 CNI 插件

### Flannel 的 Overlay 工作原理

#### Flannel 的网络结构

创建 3 个 Nginx Pod：

```bash
kubectl create deploy ngx-dep --image=nginx:alpine --replicas=3
```

Flannel 默认使用基于 **VXLAN 的 Overlay 模式**，网络结构与 Docker 相似：
- 网桥换成 **cni0**（而不是 docker0）
- 每个 Pod 创建虚拟网卡对（veth pair）

#### 单机网络的实现

**Pod 内的虚拟网卡**：

在 Pod 里执行 `ip addr`：

```
3: eth0@if45: ...
```

- 第一个数字「3」是序号，第 3 号设备
- @if45 是另一端连接的虚拟网卡，序号 45

**宿主机上的虚拟网卡**：

登录 master 节点，执行 `ip addr`：

```
45: veth41586979@if3: ...
```

- veth 表示虚拟网卡
- @if3 是 Pod 里对应的 3 号设备（eth0）

**cni0 网桥的信息**：

在宿主机上执行 `brctl show`：

```
cni0: ... veth41586979 ...
```

网卡被「插」在 cni0 网桥上，Pod 连上 cni0 网桥。

借助网桥，本机的 Pod 可以直接通信。

#### 跨主机网络的实现

查看节点的路由表（`route`）：

```
10.10.0.0/24    0.0.0.0    cni0
10.10.1.0/24    0.0.0.0    flannel.1
192.168.10.0/24 0.0.0.0    ens160
```

含义：
- 10.10.0.0/24 网段的数据，走 cni0 设备（网桥）
- 10.10.1.0/24 网段的数据，走 flannel.1 设备（Flannel）
- 192.168.10.0/24 网段的数据，走 ens160 设备（宿主机网卡）

**跨主机通信流程**：

假设 master 节点的 Pod（10.10.0.3）访问 worker 节点的 Pod（10.10.1.77）：

1. cni0 网桥管理的是 10.10.0.0/24 网段
2. 按路由表，10.10.1.0/24 让 flannel.1 处理
3. 进入 Flannel 插件的工作流程
4. Flannel 决定数据发到 192.168.10.220（worker 节点）
5. 封装成 VXLAN 报文，用 ens160 网卡发出
6. worker 节点收到后拆包，反向处理，交给目标 Pod

Overlay 模式需要**封包和拆包**，有性能损失。

### Calico 的 Route 工作原理

#### Calico 的安装

下载 YAML 文件：

```bash
wget https://projectcalico.docs.tigera.io/manifests/calico.yaml
```

预先拉取镜像（镜像较大）：

```bash
docker pull calico/cni:v3.23.1
docker pull calico/node:v3.23.1
docker pull calico/kube-controllers:v3.23.1
```

安装（记得先删除 Flannel）：

```bash
kubectl apply -f calico.yaml
```

Calico 运行在 kube-system 名字空间。

#### Calico 的网络特点

创建 3 个 Nginx Pod，观察：
- IP 地址与 Flannel 不同：10.10.219.* 和 10.10.171.*
- Calico 的 IP 地址分配策略不同

查看 Pod 的网卡，发现：
- 虚拟网卡名字是 **calica17a7ab6ab@if4**
- **没有连接到 cni0 网桥**

这是 Calico 的 **Route 模式**导致的正常现象：
- 不使用网桥
- 在宿主机上创建路由规则
- 数据包不经过网桥，直接「跳」到目标网卡

#### Calico 的路由规则

查看节点路由表：

```
10.10.219.68 dev cali051dd144e34
```

假设 Pod A（10.10.219.67）访问 Pod B（10.10.219.68）：
- 查路由表，知道要走 cali051dd144e34 设备
- 它恰好就在 Pod B 里
- 数据直接进 Pod B 的网卡
- 省去网桥的中间步骤

**Route 模式的优势**：
- 数据包直接发送到目标网卡
- 不经过网桥
- **性能高**

跨主机通信对照路由表，一步步「跳」到目标 Pod（通过 tunl0 设备）。

## 💡 重點摘要

- **IP-per-pod 网络模型**：每个 Pod 有唯一 IP，Pod 直连互通，不需要 NAT，平坦网络易于管理
- **CNI 是网络插件标准接口**：定义通用接口让开发者接入 Kubernetes，实现网络模型
- **三种实现模式**：Overlay（封包拆包，适应性強但性能低）、Route（路由规则，性能高）、Underlay（底层网络，性能最高但不够灵活）
- **Flannel 使用 Overlay 模式**：cni0 网桥、flannel.1 设备、VXLAN 封包，本机通信走网桥，跨主机封包发送
- **Calico 使用 Route 模式**：不使用网桥，创建路由规则，数据包直接跳到目标网卡，性能高

## 🔑 關鍵字

CNI, Flannel, Calico, VXLAN, BGP