# Pod：Kubernetes 最核心的概念

## 📝課程概述
Pod 是 Kubernetes 管理应用的最小单位，是对容器的「打包」，让多个容器保持相对独立又能共享网络、存储等资源。本課程讲解 Pod 的概念、YAML 描述方式及操作命令。

## 核心觀念與實作解析

### 為什麼要有 Pod
**Pod 原意**：豌豆荚、舱室、太空舱

**容器技術的局限**：
- 容器隔离性好，但现实中很多应用需要**多个进程密切协作**
- 如 WordPress 需要 Nginx、WordPress、MariaDB 三个容器协同工作
- 有些应用结合紧密无法拆开（如日志代理需读取另一个应用的本地磁盘文件）

**不應把多應用放在同一容器**：
-违背容器初衷（一个容器一个进程）
- 让容器难以管理

**Pod 的解决方案**：
在容器外建立「收纳舱」，让多个容器：
- 保持相对独立
- 小范围共享网络、存储等资源
- 永远是「绑在一起」的状态

### 為什麼 Pod 是 Kubernetes 的核心对象
Pod 是对容器的「打包」，里面的容器是一个**整体**：
- **一起调度、一起运行**，绝不分离
- 属于 Kubernetes，可在不触碰下层容器的情况下任意定制修改
- 基于Pod 可构建出更多复杂的业务形态

**Pod 是 Kubernetes 世界里的「原子」**，所有资源都直接或间接依附在 Pod 之上。

### 如何使用 YAML 描述 Pod

**必备字段**：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busy-pod        # Pod 必须有名字
  labels:               # 标签用于归类识别
    owner: chrono
    env: demo
    region: north
    tier: back
```

**spec.containers 字段**（数组，每个元素是一个容器）：
```yaml
spec:
  containers:
  - image: busybox:latest
    name: busy
    imagePullPolicy: IfNotPresent    # 拉取策略
    env:                             # 环境变量
      - name: os
        value: "ubuntu"
    command:                         # 启动命令（相当于ENTRYPOINT）
      - /bin/echo
    args:                            # 运行参数（相当于CMD）
      - "$(os), $(debug)"
    ports:                           # 暴露端口
      - containerPort: 80
```

**重要字段说明**：
| 字段 |含义 | Docker 对应 |
|------|------|-------------|
| `imagePullPolicy` | 镜像拉取策略（Always/Never/IfNotPresent） | - |
| `env` | 环境变量，运行时指定 | ENV 指令 |
| `command` | 容器启动命令 | ENTRYPOINT |
| `args` | command 的参数 | CMD |

### 如何使用 kubectl 操作 Pod

**创建和删除**：
```bash
kubectl apply -f busy-pod.yml
kubectl delete -f busy-pod.yml
kubectl delete pod busy-pod      # 直接指定名字删除
```

**查看状态和日志**：
```bash
kubectl get pod                  # 查看Pod列表
kubectl describe pod busy-pod    # 详细状态（调试排错）
kubectl logs busy-pod            # 查看标准输出
```

**进入 Pod 内部**：
```bash
kubectl cp a.txt ngx-pod:/tmp    # 拷贝文件进Pod
kubectl exec -it ngx-pod -- sh   # 进入Pod执行Shell
```

**注意**：kubectl exec 需要在 Pod 后面加 `--`，分隔kubectl 命令与 Shell 命令。

### Pod 的状态
- `Running`：正常运行
- `CrashLoopBackOff`：反复停止-启动的循环错误状态
- `Completed`：任务完成（离线作业）

## 💡 重點摘要
- Pod 解决多进程密切协作问题，「打包」一个或多个容器
- Pod 是 Kubernetes 管理应用的最小单位，是「原子」概念
- spec.containers 是关键字段，定义容器运行状态
- kubectl 操作命令与 Docker 类似，但有些差异（如 exec 需要 `--`）
- 通常不直接创建 Pod，需要 Job、Deployment 等对象增添更多功能

## 🔑 關鍵字
### Pod, containers, labels, imagePullPolicy, command, args, kubectl exec