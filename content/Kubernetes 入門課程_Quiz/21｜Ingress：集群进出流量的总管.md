# Ingress：集群进出流量的总管 測驗

**主題:** Kubernetes Ingress、Ingress Controller、Ingress Class
**難度:** 中等
**題數:** 20題
**時間:** 建議30-40分頁

---

## 第一部分：選擇題 (每題5分，共40分)

**Q1.** 在 Kubernetes 中，Service 是哪一層的負載均衡？

A) 二層（數據鏈路層）
B) 三層（網絡層）
C) 四層（傳輸層）
D) 七層（應用層）

<details>
<summary>答案</summary>
<b>C)</b> Service 是四層負載均衡，在 TCP/IP 协议栈上转发流量，只能依据 IP 地址和端口号做一些简单的判断和组合。

<b>其他选项解析：</b>
- A) 二層主要處理 MAC 地址
- B) 三層主要處理 IP 路由
- D) 七層是 HTTP/HTTPS 協議，這是 Ingress 的工作層級
</details>

**Q2.** Ingress 相比 Service 的主要優勢是什麼？

A) 性能更高
B) 支持 HTTP/HTTPS 協議的高级路由功能
C) 部署更簡單
D) 不需要配置文件

<details>
<summary>答案</summary>
<b>B)</b> Ingress 基於 HTTP/HTTPS 协议，支持更多高级路由条件，比如主机名、URI、请求头、证书等，而這些在 TCP/IP 网络栈里是根本看不见的。

<b>其他选项解析：</b>
- A) 四層負載均衡性能通常更高
- C) Ingress 配置更複雜
- D) Ingress 需要 YAML 配置文件
</details>

**Q3.** Ingress 本質上是什麼？

A) 一個運行中的服務程序
B) HTTP 路由规则的集合
C) 一個獨立的節點
D) kube-proxy 的替代品

<details>
<summary>答案</summary>
<b>B)</b> Ingress 只是一些 HTTP 路由规则的集合，相当于一份静态的描述文件，真正要把这些规则在集群里实施运行，还需要 Ingress Controller。

<b>其他选项解析：</b>
- A) Ingress Controller 才是運行中的服務程序
- C) Ingress 是 API 對象，不是節點
- D) kube-proxy 對應的是 Service，不是 Ingress
</details>

**Q4.** Ingress Controller 的作用類似於什麼？

A) Deployment
B) Pod
C) kube-proxy
D) Namespace

<details>
<summary>答案</summary>
<b>C)</b> Ingress Controller 的作用就相当于 Service 的 kube-proxy，能够读取、应用 Ingress 规则，处理、调度流量。

<b>其他选项解析：</b>
- A) Deployment 用於管理應用副本
- B) Pod 是容器運行的基本單位
- D) Namespace 用於資源隔离
</details>

**Q5.** Kubernetes 為什麼沒有内置 Ingress Controller？

A) 技術能力不足
B) Ingress Controller 与上层业务联系太密切，交给社区实现更好
C) Ingress Controller 不重要
D) 開發成本太高

<details>
<summary>答案</summary>
<b>B)</b> Ingress Controller 要做的事情太多，与上层业务联系太密切，所以 Kubernetes 把 Ingress Controller 的实现交给了社区，任何人都可以开发 Ingress Controller。

<b>其他选项解析：</b>
- A) Kubernetes 技術能力完全足夠
- C) Ingress Controller 很重要，是集群流量入口
- D) 不是成本問題，是架構設計考量
</details>

**Q6.** 最流行的 Ingress Controller 是基於什麼軟體？

A) Apache
B) HAProxy
C) Nginx
D) Traefik

<details>
<summary>答案</summary>
<b>C)</b> 最著名的就是老牌的反向代理和负载均衡软件 Nginx，它稳定性最好、性能最高，成为了 Kubernetes 里应用得最广泛的 Ingress Controller。

<b>其他选项解析：</b>
- A) Apache 主要用於 Web 服務器
- B) HAProxy 也是負載均衡器但不如 Nginx 流行
- D) Traefik 也是 Ingress Controller 但不如 Nginx 流行
</details>

**Q7.** Ingress Class 的主要作用是什麼？

A) 定义 Ingress Controller 的实现细节
B) 解耦 Ingress 和 Ingress Controller
C) 替代 Service
D) 管理节点资源

<details>
<summary>答案</summary>
<b>B)</b> Ingress Class 插在 Ingress 和 Ingress Controller 中间，作为流量规则和控制器的协调人，解除了 Ingress 和 Ingress Controller 的强绑定关系。

<b>其他选项解析：</b>
- A) Ingress Class 不定義實現細節
- C) Ingress Class 不替代 Service
- D) Ingress Class 不管理節點資源
</details>

**Q8.** 在 Ingress 的 YAML 中，哪個字段用於指定路由規則？

A) ingressClassName
B) rules
C) backend
D) host

<details>
<summary>答案</summary>
<b>B)</b> 在 Ingress YAML 中有两个关键字段："ingressClassName" 和 "rules"，其中 "rules" 用于指定路由规则，包含 host、http、paths 等信息。

<b>其他选项解析：</b>
- A) ingressClassName 指定 Ingress Class
- C) backend 是 rules 的一部分，指定後端服務
- D) host 是 rules 的子字段
</details>

---

## 第二部分：是非題 (每題3分，共15分)

**Q9.** Service 可以完美管理集群的进出流量，不需要 Ingress。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - Service 是四层负载均衡，功能有限，只能依据 IP 地址和端口号做简单判断。现代应用需要 HTTP/HTTPS 层面的高级路由功能，如主机名、URI、请求头等，這些都需要 Ingress。
</details>

**Q10.** Ingress Controller 必须作为内置组件部署在 Kubernetes 集群中。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - Ingress Controller 不是 Kubernetes 内置组件，而是由社区开发的第三方实现。它以 Pod 形式运行在集群里，支持 Deployment 和 DaemonSet 两种部署方式。
</details>

**Q11.** 一个集群只能有一个 Ingress Controller。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - Kubernetes 允許多個 Ingress Controller，通過 Ingress Class 來協調管理。不同的 Ingress Class 可以對應不同的 Ingress Controller，處理不同的業務流量。
</details>

**Q12.** Ingress 支持 TCP 和 UDP 协议的长连接应用。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - Ingress 基本設計只支持 HTTP/HTTPS 协议。对于 TCP/UDP 等其他协议，需要使用 Ingress Controller 的 CRD（如 Nginx 的 Transport Server）來定義。
</details>

**Q13.** Ingress Class 是 Kubernetes v1.18+ 才引入的概念。 _(對/錯)_

<details>
<summary>答案</summary>
<b>對</b> - Ingress Class 是為了解決早期 Ingress 和 Ingress Controller 强绑定带来的问题而引入的，允許多個 Ingress Controller 和 Ingress 资源邏輯分组管理。
</details>

---

## 第三部分：填充題 (每題5分，共20分)

**Q14.** Service 是 **\_\_\_\_\_** 層負載均衡，而 Ingress 是 **\_\_\_\_\_** 層負載均衡。

<details>
<summary>答案</summary>
<b>四层（傳輸層），七层（應用層）</b>
</details>

**Q15.** Ingress 只是 **\_\_\_\_\_** 的集合，真正實施這些規則需要 **\_\_\_\_\_**。

<details>
<summary>答案</summary>
<b>HTTP 路由规则，Ingress Controller</b>
</details>

**Q16.** 在 Ingress YAML 的 rules 字段中，pathType 可以設為 **\_\_\_\_\_**（精确匹配）或 **\_\_\_\_\_**（前缀匹配）。

<details>
<summary>答案</summary>
<b>Exact，Prefix</b>
</details>

**Q17.** Nginx Ingress Controller 需要在 args 中添加 **\_\_\_\_\_** 参数來關聯 Ingress Class。

<details>
<summary>答案</summary>
<b>-ingress-class=ngx-ink</b>（或對應的 Ingress Class 名稱）
</details>

---

## 第四部分：簡答題 (每題10分，共20分)

**Q18.** 請說明四層負載均衡（Service）與七層負載均衡（Ingress）的主要異同點。

<details>
<summary>答案</summary>
<b>相同點：</b>
- 都是負載均衡機制
- 都代理後端的 Pod
- 都有路由規則定義流量分配

<b>不同點：</b>
- Service 工作在四层（TCP/IP），只能基于 IP 地址和端口号转发
- Ingress 工作在七层（HTTP/HTTPS），可以基于主机名、URI、请求头、证书等高级路由
- Service 由 kube-proxy 实现，Ingress 由 Ingress Controller 实现
- Service 配置简单，Ingress 配置复杂但功能强大
- Service 性能通常更高（少一層协议处理），Ingress 更灵活

<b>關鍵要點：</b>
- 协议层級差異
- 路由能力的不同
- 實現方式的差異
- 性能與靈活性的权衡
</details>

**Q19.** 為什麼需要 Ingress Class？它能解決哪些問題？

<details>
<summary>答案</summary>
<b>Ingress Class 的作用：</b>
- 作为 Ingress 和 Ingress Controller 的协调人
- 解除 Ingress 和 Ingress Controller 的强绑定关系
- 定义不同的业务逻辑分组

<b>解决的问题：</b>
- 允许一个集群引入不同的 Ingress Controller（解决单一控制器的限制）
- 防止 Ingress 规则太多导致单个 Ingress Controller 不堪重负
- 为多个 Ingress 对象提供逻辑分组方式，降低管理和维护成本
- 支持不同租户的不同需求，避免冲突

<b>关键要點：</b>
- 解耦架构
- 多租户支持
- 逻辑分组管理
- 灵活性和可扩展性
</details>

---

## 第五部分：配對題 (每題5分，共5分)

**Q20.** 请将以下组件与其对应的描述进行配对：

| 组件                | 描述                                        |
| ------------------- | ------------------------------------------- |
| 1. Ingress          | A. 真正实施路由规则的程序，相当于 kube-proxy |
| 2. Ingress Class    | B. HTTP 路由规则的静态描述文件              |
| 3. Ingress Controller | C. 协调 Ingress 和 Controller 的中间层    |

<details>
<summary>答案</summary>
<b>1-B, 2-C, 3-A</b>
</details>

---

## 學習重點总结

### 核心概念
- **Service**：四層負載均衡，基於 IP/Port 轉發，由 kube-proxy 實現
- **Ingress**：七層負載均衡，基於 HTTP/HTTPS 协议路由
- **Ingress Controller**：實施 Ingress 规则的程序（如 Nginx）
- **Ingress Class**：解耦 Ingress 和 Ingress Controller

### 實際應用
- 使用 kubectl create ing 生成 Ingress YAML
- Nginx Ingress Controller 是最流行的实现
- Ingress Controller 需要通过 Service (NodePort/LoadBalancer) 暴露
- 测试时可用 kubectl port-forward 和 curl --resolve

### 架構關係
```
Ingress Class → Ingress → Ingress Controller → Service → Pod
```

---

**生成來源:** 21｜Ingress：集群进出流量的总管.pdf
**測驗類型:** 練習測驗
**適用對象:** Kubernetes 初學者到中級學習者