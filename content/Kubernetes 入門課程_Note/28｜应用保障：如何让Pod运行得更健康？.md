# 應用保障：Pod運行健康檢查機制

## 📝 課程概述

本課程回到 Pod 本身，探討如何為 Pod 配置運行保障機制。課程詳細說明兩種關鍵技術：資源配額（Resources）通過 cgroup 限制 CPU 和内存使用；狀態探针（Probe）提供 Startup、Liveness、Readiness 三種健康检查，讓 Kubernetes 能實時监控应用运行状态，确保应用健康稳定運行。

## 核心觀念與實作解析

### 容器資源配額（Resources）

#### cgroup 的作用

在第 2 課中提到，創建容器有三大隔离技术：namespace、cgroup、chroot。

- **namespace**：实现独立的进程空间
- **chroot**：实现独立的文件系统
- **cgroup**：管控 CPU、内存，防止容器无节制地占用资源

cgroup 保证容器不会影响系统里的其他应用，是资源管控的核心技术。

#### Resources 字段的申请机制

Kubernetes 的做法类似 PersistentVolumeClaim，容器需要先提出「书面申请」，Kubernetes 再决定资源是否分配和如何分配。

在 Pod 容器的描述部分添加 `resources` 字段：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ngx-pod-resources

spec:
  containers:
  - image: nginx:alpine
    name: ngx

    resources:
      requests:
        cpu: 10m
        memory: 100Mi
      limits:
        cpu: 20m
        memory: 200Mi
```

#### requests vs limits

**requests（申请的资源）**：
- 要求 Kubernetes 在创建 Pod 时必须分配这里列出的资源
- 否则容器就无法运行
- 是资源的「下限」

**limits（使用资源的上限）**：
- 容器使用资源不能超过设定值
- 否则可能被强制停止运行
- 是资源的「上限」

#### CPU 和 Memory 的单位表示

**内存（Memory）**：
- 使用 Ki、Mi、Gi 表示 KB、MB、GB
- 例如：512Ki、100Mi、0.5Gi

**CPU**：
- 可以完整使用 CPU（1、2）或部分使用（0.1、0.2）
- 效仿 UNIX「时间片」用法
- 最小单位是 0.001，用特殊单位 **m**（milli，毫）表示
- 例如：500m = 0.5 CPU

示例 YAML 含义：
- 申请：1% CPU 时间（10m）和 100MB 内存
- 上限：2% CPU 时间（20m）和 200MB 内存

#### 资源申请与调度

Kubernetes 根据每个 Pod 声明的需求，像搭积木一样把节点尽量「塞满」，充分利用每个节点的资源，让集群效益最大化。

**不写 resources 字段**：
- Pod 对运行资源「既没有下限，也没有上限」
- Kubernetes 不用管 CPU 和内存是否足够
- 可以调度到任意节点
- 运行时可以无限制使用 CPU 和内存
- 生产环境很危险，可能影响其他应用

#### 资源申请过大的后果

将 requests.cpu 改成极端的「10」（要求10个CPU）：

```yaml
resources:
  requests:
    cpu: 10
```

Pod 能创建成功，但处于 **Pending 状态**，实际上并没有真正被调度运行。

使用 `kubectl describe` 查看原因：
```
0/3 nodes are available: 3 Insufficient cpu.
```

Kubernetes 调度失败，当前集群里的所有节点都无法运行这个 Pod。

### 容器狀態探针（Probe）

#### 为什么需要探针？

即使程序正常启动了，也可能因为某些原因无法对外提供服务：
- 运行时发生「死锁」或「死循环」
- 从外部来看进程一切正常
- 但内部已经一团糟

Kubernetes 需要更细致地监控 Pod 的状态，定时给应用做「体检」，这项功能被命名为**探针（Probe）**。

#### 三种探针类型

**1. StartupProbe（启动探针）**
- 检查应用是否已经启动成功
- 适合有大量初始化工作、启动很慢的应用

**2. LivenessProbe（存活探针）**
- 检查应用是否正常运行
- 检测是否存在死锁、死循环

**3. ReadinessProbe（就绪探针）**
- 检查应用是否可以接收流量
- 检测是否能够对外提供服务

三种探针是**递进的关系**：
- 应用程序先启动 → Startup 状态
- 没有异常 → Liveness 存活状态
- 准备工作完成 → Readiness 状态（最健康可用的状态）

#### 探针失败的处理机制

**Startup 探针失败**：
- Kubernetes 认为容器没有正常启动
- 会尝试反复重启
- Liveness 和 Readiness 探针不会启动

**Liveness 探针失败**：
- Kubernetes 认为容器发生异常
- 会重启容器

**Readiness 探针失败**：
- Kubernetes 认为容器虽然运行，但内部有错误
- 不能正常提供服务
- 会把容器从 Service 的负载均衡集合中排除
- 不会给它分配流量

#### 探针的关键配置参数

所有探针的配置方式一样：

- **periodSeconds**：执行探测的时间间隔，默认 10 秒
- **timeoutSeconds**：探测的超时时间，默认 1 秒
- **successThreshold**：连续几次探测成功才认为正常
  - 对于 startupProbe 和 livenessProbe 只能是 1
- **failureThreshold**：连续探测失败几次才认为异常，默认 3 次

#### 三种探测方式

**1. exec（Shell 方式）**
- 执行一个 Linux 命令
- 例如：ps、cat 等
- 和 container 的 command 字段类似

**2. tcpSocket（TCP Socket 方式）**
- 使用 TCP 协议尝试连接容器的指定端口

**3. httpGet（HTTP GET 方式）**
- 连接端口并发送 HTTP GET 请求

### 探针的實战配置

#### 应用预留「检查口」

使用 Nginx 作为示例，通过 ConfigMap 编写配置文件：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ngx-conf

data:
  default.conf: |
    server {
        listen 80;
        location = /ready {
            return 200 'I am ready';
        }
    }
```

定义 HTTP 路径 `/ready` 作为对外暴露的「检查口」，返回 200 状态码表示工作正常。

#### Pod 探针配置示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ngx-pod-probe

spec:
  volumes:
  - name: ngx-conf-vol
    configMap:
      name: ngx-conf

  containers:
  - image: nginx:alpine
    name: ngx
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /etc/nginx/conf.d
      name: ngx-conf-vol

    startupProbe:
      periodSeconds: 1
      exec:
        command: ["cat", "/var/run/nginx.pid"]

    livenessProbe:
      periodSeconds: 10
      tcpSocket:
        port: 80

    readinessProbe:
      periodSeconds: 5
      httpGet:
        path: /ready
        port: 80
```

**StartupProbe**：
- Shell 方式，使用 cat 检查进程号文件 `/var/run/nginx.pid`
- 存在就认为启动成功
- 每秒探测一次

**LivenessProbe**：
- TCP Socket 方式，尝试连接 80 端口
- 每 10 秒探测一次

**ReadinessProbe**：
- HTTP GET 方式，访问 `/ready` 路径
- 每 5 秒发一次请求

#### 探针执行验证

创建 Pod 后，使用 `kubectl logs` 查看 Nginx 的访问日志：

Kubernetes 以大约 5 秒一次的频率，向 URI `/ready` 发送 HTTP 请求，不断检查容器是否处于就绪状态。

#### 探针失败的演示

修改探针检查错误的文件、错误的端口号：

```yaml
startupProbe:
  exec:
    command: ["cat", "nginx.pid"]    # 错误的文件

livenessProbe:
  tcpSocket:
    port: 8080                      # 错误的端口号
```

重新创建 Pod 后观察：
- **StartupProbe 失败**：Kubernetes 不停重启容器，RESTARTS 次数不断增加
- Pod 是 Running 状态，但永远不会 READY
- **LivenessProbe 失败**：连续执行三次 TCP Socket 探测（每次间隔 10秒，共30秒）都失败才重启容器

#### 探针的执行顺序

- **Startup 成功之后**才能执行后两个探针
- **Liveness 和 Readiness 是并行的**

## 💡 重點摘要

- **资源配额使用 cgroup 技术**：通过 requests 和 limits 限制 CPU 和内存，让 Pod 合理利用系统资源
- **CPU 单位 m**：最小单位 0.001，用 m 表示，如 500m = 0.5 CPU
- **三种探针递进关系**：Startup（启动）→ Liveness（存活）→ Readiness（就绪）
- **探针失败处理**：Startup/Liveness 失败重启容器，Readiness 失败从 Service 排除
- **三种探测方式**：exec（Shell）、tcpSocket（TCP）、httpGet（HTTP），需在应用中预留检查口

## 🔑 關鍵字

Resources, Probe, cgroup, LivenessProbe, ReadinessProbe