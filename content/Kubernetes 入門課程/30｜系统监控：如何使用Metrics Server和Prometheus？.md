# 系统监控：Metrics Server 与 Prometheus

## 📝 課程概述

本課程探討 Kubernetes 集群的可观测性解决方案。課程詳細說明 Metrics Server 和 Prometheus 两種系統級監控工具的安裝與使用，並介紹 HorizontalPodAutoscaler 实现的应用自动水平伸缩功能，讓集群的整体运行状况透明可見，更准确方便地做好集群运维工作。

## 核心觀念與實作解析

### Metrics Server：核心资源指标收集

#### Metrics Server 的作用

Metrics Server 是专门用来收集 Kubernetes **核心资源指标（metrics）** 的工具：
- 定时从所有节点的 kubelet 里采集信息
- 对集群的整体性能影响极小
- 每个节点只大约占用 **1m CPU** 和 **2MB 内存**
- 性价比非常高

Metrics Server 调用 kubelet 的 API 拿到节点和 Pod 的指标，再把这些信息交给 apiserver，这样 kubectl、HPA 就可以利用 apiserver 来读取指标。

#### Metrics Server 的安装

**下载 YAML 文件**：

```bash
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

**准备工作 1：修改 YAML 文件**

需要在 Deployment 对象里添加参数 `--kubelet-insecure-tls`：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
spec:
  ...
  template:
    spec:
      containers:
      - args:
        - --kubelet-insecure-tls
        ...
```

原因：Metrics Server 默认使用 TLS 协议验证证书，实验环境不需要，加上参数让部署简单（生产环境慎用）。

**准备工作 2：预先下载镜像**

Metrics Server 的镜像仓库用的是 gcr.io，下载困难。使用国内镜像网站：

```bash
repo=registry.aliyuncs.com/google_containers

name=k8s.gcr.io/metrics-server/metrics-server:v0.6.1
src_name=metrics-server:v0.6.1

docker pull $repo/$src_name

docker tag $repo/$src_name $name
docker rmi $repo/$src_name
```

**部署 Metrics Server**：

```bash
kubectl apply -f components.yaml
```

Metrics Server 属于名字空间 kube-system：

```bash
kubectl get pod -n kube-system
```

#### 使用 kubectl top 命令

Metrics Server 安装后，可以使用 `kubectl top` 查看资源状态：

```bash
kubectl top node        # 查看节点的资源使用率
kubectl top pod -n kube-system    # 查看 Pod 的资源使用率
```

由于 Metrics Server 收集信息需要时间，必须等一小会儿才能执行命令。

**示例输出**：
- 集群里两个节点 CPU 使用率：8% 和 4%
- 内存使用：master 48%，worker 89%
- apiserver 最消耗资源：75m CPU 和 363MB 内存

### HorizontalPodAutoscaler：自动水平伸缩

#### HPA 的作用

在第 18 讲中使用 `kubectl scale` 手动调整 Deployment 的 Pod 数量（水平方向的「扩容」和「缩容」）。但手动调整：
- 需要人工参与
- 很难准确把握时机
- 难以及时应对生产环境中突发的大流量

**HorizontalPodAutoscaler（简称 hpa）** 实现自动化：
- 适用于 Deployment 和 StatefulSet
- 不能用于 DaemonSet（因为每个节点固定一个 Pod）
- 完全基于 Metrics Server
- 从 Metrics Server 获取运行指标（主要是 CPU 使用率）
- 依据预定策略增加或减少 Pod 数量

#### 创建 Deployment 和 Service

先定义 Deployment 和 Service 作为自动伸缩的目标对象：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ngx-hpa-dep

spec:
  replicas: 1
  selector:
    matchLabels:
      app: ngx-hpa-dep

  template:
    metadata:
      labels:
        app: ngx-hpa-dep
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
        ports:
        - containerPort: 80

        resources:
          requests:
            cpu: 50m
            memory: 10Mi
          limits:
            cpu: 100m
            memory: 20Mi
---

apiVersion: v1
kind: Service
metadata:
  name: ngx-hpa-svc
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: ngx-hpa-dep
```

**重要提醒**：必须用 `resources` 字段写清楚资源配额，否则 HPA 无法获取 Pod 的指标，无法实现自动化扩缩容。

#### 创建 HorizontalPodAutoscaler

使用 `kubectl autoscale` 创建样板 YAML，有三个参数：
- **--min**：Pod 数量的最小值（缩容下限）
- **--max**：Pod 数量的最大值（扩容上限）
- **--cpu-percent**：CPU 使用率指标（大于此值扩容，小于此值缩容）

```bash
export out="--dry-run=client -o yaml"
kubectl autoscale deploy ngx-hpa-dep --min=2 --max=10 --cpu-percent=5 $out
```

生成的 YAML：

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: ngx-hpa

spec:
  maxReplicas: 10
  minReplicas: 2
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ngx-hpa-dep
  targetCPUUtilizationPercentage: 5
```

创建 HPA：

```bash
kubectl apply -f ngx-hpa.yml
```

HPA 发现 Deployment 实例只有 1 个，不符合 min 定义的下限要求，先扩容到 2 个。

#### HPA 的自动伸缩测试

运行测试 Pod，使用 httpd:alpine（包含 ab 性能测试工具）：

```bash
kubectl run test -it --image=httpd:alpine -- sh
```

向 Nginx 发送一百万个请求，持续 1 分钟：

```bash
ab -c 10 -t 60 -n 1000000 'http://ngx-hpa-svc/'
```

观察 HPA：

```bash
kubectl get hpa
```

**Metrics Server 大约每 15 秒采集一次数据**，HPA 按这个时间点逐步处理：
- 发现 CPU 使用率超过预定 5% 后
- 以 2 的倍数开始扩容，一直到数量上限
- 持续监控一段时间
- 如果 CPU 使用率回落，缩容到最小值

### Prometheus：云原生监控标准

#### Prometheus 的历史与地位

**Prometheus** 的历史比 Kubernetes 还要早：
- 2012 年由 Google 离职员工创建
- 灵感来源于 Borg 配套的 BorgMon 监控系统
- 2016 年作为第二个项目加入 CNCF
- 2018 年继 Kubernetes 之后顺利毕业
- 成为 CNCF 的「二当家」
- 云原生监控领域的「事实标准」

#### Prometheus 的架构

**核心组件 Prometheus Server**：
- **TSDB**：时序数据库，存储监控数据
- **Retrieval**：使用拉取（Pull）方式从各个目标收集数据
- **HTTP Server**：把数据交给外界使用

**三个重要组件**：
- **Push Gateway**：适配特殊监控目标，把默认 Pull 模式转变为 Push 模式
- **Alert Manager**：告警中心，预先设定规则，通过邮件等方式告警
- **Grafana**：图形化界面，定制直观的监控仪表盘

#### Prometheus 的安装（kube-prometheus）

下载 kube-prometheus 源码包（版本 0.11）：

```bash
wget https://github.com/prometheus-operator/kube-prometheus/archive/refs/tags/v0.11.tar.gz
```

解压缩后，YAML 文件在 manifests 目录里，近 100 个。

**准备工作 1：修改 Service 对象**

修改 prometheus-service.yaml、grafana-service.yaml，添加 type: NodePort：

```yaml
spec:
  type: NodePort
```

这样可以直接通过节点的 IP 地址访问。

**准备工作 2：修改镜像**

修改 kubeStateMetrics-deployment.yaml、prometheusAdapter-deployment.yaml，因为镜像在 gcr.io：

```yaml
image: k8s.gcr.io/kube-state-metrics/kube-state-metrics:v2.5.0
image: k8s.gcr.io/prometheus-adapter/prometheus-adapter:v0.9.1

# 改为
image: chronolaw/kube-state-metrics:v2.5.0
image: chronolaw/prometheus-adapter:v0.9.1
```

**部署 Prometheus**：

```bash
kubectl create -f manifests/setup    # 创建名字空间等基本对象
kubectl create -f manifests          # 创建 Prometheus 对象
```

Prometheus 的对象都在名字空间 monitoring 里：

```bash
kubectl get pod -n monitoring
```

#### Prometheus 和 Grafana 的使用

查看服务端口：

```bash
kubectl get svc -n monitoring
```

- Grafana：端口 30358
- Prometheus Web：端口 30827（9090 对应）

**Prometheus Web 界面**：
- 查询框使用 **PromQL** 查询指标
- 生成可视化图表
- 例如：node_memory_Active_bytes（当前正在使用的内存容量）

**Grafana 界面**：
- 访问节点端口 30358
- 默认用户名和密码都是 admin
- 预置了很多强大易用的仪表盘
- Dashboards - Browse 里任意挑选
- 例如：Kubernetes / Compute Resources / Namespace (Pods)
- 比 Metrics Server 的 kubectl top 命令好看得多

## 💡 重點摘要

- **Metrics Server 是 Kubernetes 插件**：收集核心资源指标，kubectl top 命令依赖它，性能影响极小（1m CPU、2MB 内存）
- **HorizontalPodAutoscaler 自动水平伸缩**：基于 Metrics Server，根据 CPU 使用率自动调整 Pod 数量，应对突发流量
- **Prometheus 是云原生监控标准**：CNCF 毕业项目，使用 PromQL 查询数据，配合 Grafana 显示图形界面
- **Metrics Server 采集周期**：约每 15 秒采集一次，HPA 按这个时间点逐步处理扩缩容
- **HPA 的三个参数**：min（最小 Pod 数）、max（最大 Pod 数）、cpu-percent（CPU 使用率阈值）

## 🔑 關鍵字

Metrics Server, Prometheus, HorizontalPodAutoscaler, Grafana, PromQL