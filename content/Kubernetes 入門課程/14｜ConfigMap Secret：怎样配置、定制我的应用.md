# ConfigMap/Secret：應用配置管理

## 📝課程概述
Kubernetes 提供 ConfigMap 和 Secret 两种 API 对象管理配置信息，分别存储明文配置和机密配置。本課程讲解如何创建这两种对象，以及通过环境变量和存储卷两种方式注入Pod。

## 核心觀念與實作解析

### 配置信息的两种类型
| 类型 | 特點 | 示例 |
|------|------|------|
| **明文配置** | 不保密，可任意查询修改 | 服务端口、运行参数、文件路径 |
| **机密配置** | 涉及敏感信息需保密 | 密码、密钥、证书 |

### ConfigMap：明文配置

**创建 ConfigMap**：
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: info

data:                    # 注意：用 data 字段，不是 spec
  count: '10'
  debug: 'on'
  path: '/etc/systemd'
  greeting: |
    say hello to kubernetes.
```

**关键点**：
- ConfigMap 存储**静态字符串数据**，不是容器，所以没有 `spec` 字段
- 使用 `data` 字段，Key-Value 结构
- 值最好是字符串（用引号），避免解释成数字

**生成样板命令**：
```bash
kubectl create cm info --from-literal=k=v --dry-run=client -o yaml
```

### Secret：机密配置

**创建 Secret**：
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: user

data:
  name: cm9vdA==     # root（Base64 编码）
  pwd: MTIzNDU2      # 123456
  db: bXlzcWw=       # mysql
```

**关键点**：
- Secret 数据必须是 **Base64 编码**
- Base64 只是编码，不是真正加密
- `kubectl describe secret` 不能直接看到内容

**手动 Base64 编码**：
```bash
echo -n "123456" | base64      # 编码
echo "MTIzNDU2" | base64 -d    # 解码
```

**生成样板命令**：
```bash
kubectl create secret generic user --from-literal=name=root --dry-run=client -o yaml
```

### 使用方式一：环境变量

在 Pod 的 `spec.containers.env` 中使用 `valueFrom` 引用：

```yaml
spec:
  containers:
  - env:
      - name: COUNT
        valueFrom:
          configMapKeyRef:      # 引用 ConfigMap
            name: info          # ConfigMap 对象名
            key: count          # Key 名
      - name: PASSWORD
        valueFrom:
          secretKeyRef:         # 引用 Secret
            name: user          # Secret 对象名
            key: pwd            # Key 名
```

**特点**：适合存放简短字符串，Pod启动时注入，不会自动更新。

### 使用方式二：存储卷（Volume）

**定义Volume**（在 spec.volumes 下）：
```yaml
spec:
  volumes:
  - name: cm-vol
    configMap:
      name: info
  - name: sec-vol
    secret:
      secretName: user
```

**挂载Volume**（在 containers.volumeMounts 下）：
```yaml
containers:
- volumeMounts:
    - mountPath: /tmp/cm-items
      name: cm-vol
    - mountPath: /tmp/sec-items
      name: sec-vol
```

**特点**：
- ConfigMap/Secret 变成目录，Key-Value 变成文件
- 文件名就是 Key
- 适合大数据量的配置文件
- Volume方式会同步更新（环境变量方式不会）

### 操作命令
```bash
# 创建
kubectl apply -f cm.yml
kubectl apply -f secret.yml

# 查看
kubectl get cm
kubectl get secret
kubectl describe cm info
kubectl describe secret user
```

## 💡 重點摘要
- ConfigMap 存储明文配置，Secret 存储机密配置（Base64 编码）
- 两者都用 `data` 字段，Key-Value 结构，没有 `spec` 字段
- 环境变量方式：使用 `valueFrom.configMapKeyRef` / `secretKeyRef`
- 存储卷方式：定义 `volumes`，再在 `volumeMounts` 挂载
- 小数据用环境变量，大数据用存储卷

## 🔑 關鍵字
ConfigMap, Secret, data, valueFrom, configMapKeyRef, secretKeyRef, volumes, volumeMounts