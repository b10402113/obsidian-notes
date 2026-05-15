# Job/CronJob：離線業務處理

## 📝課程概述
Kubernetes基於「單一職責」和「組合優於繼承」的設計原則，為離線業務提供了 Job 和 CronJob 兩種 API 对象。本課程讲解如何用 YAML描述和操作这两种对象。

## 核心觀念與實作解析

### 為什麼不直接使用 Pod
Kubernetes 采用**面向对象设计**思想：
- **单一职责**：对象应只专注于做好一件事
- **组合优于继承**：让对象在运行时产生联系，保持松耦合

Pod 已经是相对完善的对象，专门负责管理容器，不应「画蛇添足」扩充功能。容器之外的功能应定义其他对象，把 Pod 作为成员「组合」进去。

###業務類型分類
| 类型 | 特點 | 代表应用 |
|------|------|----------|
| **在线业务** |长时间运行，一旦启动基本不停 | Nginx、Node.js、MySQL、Redis |
| **离线业务** | 短时间运行，必定会退出 | 日志分析、数据建模、视频转码 |

**离线业务**需考虑：运行超时、状态检查、失败重试、获取计算结果等，这些与容器管理无关，应由专门对象处理。

### Job：临时任务

**Job 的 YAML 结构**：
```yaml
apiVersion: batch/v1          # 注意不是 v1
kind: Job
metadata:
  name: echo-job

spec:
  template:                   # Pod 模板
    spec:
      restartPolicy: OnFailure    # 失败时原地重启
      containers:
      - image: busybox
        name: echo-job
        imagePullPolicy: IfNotPresent
        command: ["/bin/echo"]
        args: ["hello", "world"]
```

**关键理解**：
- Job 对象里应用了**组合模式**
- `template` 字段定义一个「应用模板」，嵌入一个 Pod
- 这个 Pod 受 Job 管理，不直接和 apiserver 打交道

**Job 级别的控制字段**（在 spec 下，不是 template 下）：
| 字段 |含义 |
|------|------|
| `activeDeadlineSeconds` | Pod 运行的超时时间 |
| `backoffLimit` | Pod 的失败重试次数 |
| `completions` | Job 完成需要运行多少个 Pod（默认1） |
| `parallelism` | 允许并发运行的 Pod 数量 |

**复杂Job 示例**（sleep-job）：
```yaml
spec:
  activeDeadlineSeconds: 15     # 15秒超时
  backoffLimit: 2               # 最多重试2次
  completions: 4                # 需运行4个Pod
  parallelism: 2                # 同时最多2个并发
```

### CronJob：定时任务

**CronJob 的 YAML 结构**：
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: echo-cj

spec:
  schedule: '*/1 * * * *'       # Cron 语法：每分钟运行
  jobTemplate:                  # Job 模板
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - image: busybox
            name: echo-cj
            command: ["/bin/echo"]
            args: ["hello", "world"]
```

**关键理解**：
- CronJob 连续有**三个 spec 嵌套层次**
- CronJob 组合了 Job，Job 又组合了 Pod
- `schedule` 字段使用标准 Cron语法（分钟、小时、天、月、周）

### 操作命令
```bash
# 创建
kubectl apply -f job.yml
kubectl apply -f cronjob.yml

# 查看
kubectl get job
kubectl get cj              # cj 是 CronJob 简写
kubectl get pod

# 查看结果
kubectl logs echo-job-xxxxx
```

###生成 YAML样板
```bash
export out="--dry-run=client -o yaml"
kubectl create job echo-job --image=busybox $out
kubectl create cj echo-cj --image=busybox --schedule="" $out
```

## 💡 重點摘要
- Kubernetes 设计遵循「单一职责」和「组合优于继承」原则
- Job 处理临时任务，CronJob 处理定时任务
- Job 关键字段：template（Pod模板）、completions、parallelism
- CronJob 关键字段：jobTemplate（Job模板）、schedule（定时规则）
- 形成「控制链」：CronJob → Job → Pod → 容器 → 进程

## 🔑 關鍵字
Job, CronJob, template, jobTemplate, schedule, completions, parallelism, batch/v1