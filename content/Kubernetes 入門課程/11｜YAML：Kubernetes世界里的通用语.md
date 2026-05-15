# YAML：Kubernetes 的通用語

## 📝 課程概述
YAML 是 Kubernetes 世界的工作語言，採用「聲明式」描述 API 对象的期望状态。本課程講解 YAML語法、API對象結構，以及編寫 YAML 的實用技巧。

## 核心觀念與實作解析

### 命令式 vs 聲明式
| 方式 |特點 |比喻 |
|------|------|------|
| **命令式（Imperative）** | 注重順序和过程，告訴計算機每步該做什麼 | Docker命令、Dockerfile |
| **聲明式（Declarative）** | 不關心过程，只給目標状态，讓計算機自己完成 | Kubernetes YAML |

**比喻**：打車去高铁站
- 命令式：告訴司机走哪條路、哪個路口轉向
- 聲明式：只說「我要去高铁站」，司机自己選最优路線

**為何 Kubernetes用聲明式？**
Kubernetes 比我们更了解集群状态，不需要「外行指導內行」，只給目標让它自己處理。

### YAML語法要点
YAML 是 JSON 的超集，语法更简洁：

- 使用**空白与缩进**表示层次（类似Python）
- 使用 `#` 书写注释
- 对象Key**不需要双引号**
- 数组使用 `-` 开头的清单形式（类似MarkDown）
- `:` 和 `-` 后面必须要有空格
- 使用 `---`分隔多个 YAML 对象

**YAML 示例**：
```yaml
# YAML数组
OS:- linux- macOS
  - Windows

# YAML 对象
Kubernetes:
  master: 1
  worker: 3
```

### 什么是 API 对象
Kubernetes 把集群里的一切资源都定义为 **API对象**：
- 通过 RESTful接口管理
- 存储在 etcd 数据库
- 目前有50多种 API 对象

查看所有 API 对象：
```bash
kubectl api-resources
```

**常用简写**：Pod → `po`，Service → `svc`，CronJob → `cj`

### API 对象的 YAML 结构
API 对象描述分为「header」和「body」两部分：

**Header（必备字段）**：
| 字段 |含义 |
|------|------|
| `apiVersion` | API 版本号（如 v1、batch/v1） |
| `kind` | 资源类型（如 Pod、Job、Service） |
| `metadata` | 元信息（name、labels等） |

**Body（spec字段）**：
- 描述对象的「期望状态」（desired status）
- 不同对象有不同的规格定义

**Pod 示例**：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ngx-pod
  labels:
    env: demo
    owner: chrono
spec:
  containers:
  - image: nginx:alpine
    name: ngx
    ports:
    - containerPort: 80
```

### 编写 YAML 的三个技巧

**技巧1：kubectl api-resources**
显示资源对象相应的 API 版本和类型，照着写不会错。

**技巧2：kubectl explain**
Kubernetes 自带的 API 文档：
```bash
kubectl explain pod
kubectl explain pod.spec.containers
```

**技巧3：生成 YAML样板**
使用 `--dry-run=client -o yaml` 参数：
```bash
export out="--dry-run=client -o yaml"
kubectl run ngx --image=nginx:alpine $out > ngx-pod.yml
```

### 操作 API 对象
```bash
kubectl apply -f ngx-pod.yml   # 创建
kubectl delete -f ngx-pod.yml   # 删除
kubectl get pod --v=9           # 查看详细HTTP请求过程
```

## 💡 重點摘要
- 聲明式只給目標状态，讓Kubernetes 自己處理细节
- YAML 是 JSON超集，语法简洁，可读性好
- API对象必备字段：apiVersion、kind、metadata、spec
- `kubectl api-resources` 查看对象列表和版本
- `kubectl explain` 查看字段说明文档
- `--dry-run=client -o yaml` 生成 YAML样板

## 🔑關键字
YAML, Declarative, API Object, apiVersion, kind, metadata, spec