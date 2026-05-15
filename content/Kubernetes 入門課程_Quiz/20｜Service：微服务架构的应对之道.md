# Service：微服務架構的應對之道 - 測驗

**課程:** Kubernetes 入門課程
**章節:** 第 20 謲
**難度:** 中等（複習測驗）
**題數:** 20題
**建議時間:** 25-30分鐘

---

## 第一部分：選擇題（每題5分，共50分）

**Q1.** 在 Kubernetes 中，Service 的主要用途是什麼？

A) 管理 Pod 的生命週期
B) 實現負載均衡和服務發現
C) 存儲應用配置信息
D) 監控容器運行狀態

<details>
<summary>答案</summary>
<b>B) 實現負載均衡和服務發現</b>

解釋：Service 是 Kubernetes 中的負載均衡機制，主要解決服務發現的關鍵問題。它為動態變化的 Pod 集合提供一個靜態的 IP 地址，實現流量轉發和負載均衡功能。
</details>

**Q2.** Service 使用哪種技術實現負載均衡？（選擇最主要的實現方式）

A) userspace
B) iptables
C) ipvs
D) nginx

<details>
<summary>答案</summary>
<b>B) iptables</b>

解釋：Service 主要使用 iptables 技術實現負載均衡，由每個節點上的 kube-proxy 組件自動維護 iptables 規則。雖然還有 userspace 和 ipvs 兩種實現方式，但 iptables 是最常用的。
</details>

**Q3.** 在 Service 的 YAML 定義中，哪個字段用於選擇要代理的 Pod？

A) ports
B) type
C) selector
D) clusterIP

<details>
<summary>答案</summary>
<b>C) selector</b>

解釋：selector 字段與 Deployment/DaemonSet 中的作用相同，用於通過標籤匹配來選擇要代理的 Pod。這是 Kubernetes 標籤機制的核心應用。
</details>

**Q4.** 以下哪個命令可以為 Deployment 創建 Service？

A) kubectl create service
B) kubectl expose deploy
C) kubectl apply service
D) kubectl generate service

<details>
<summary>答案</summary>
<b>B) kubectl expose deploy</b>

解釋：Kubernetes 使用 kubectl expose 命令來創建 Service，而不是 kubectl create。這個命令可以從 Pod、Deployment、DaemonSet 等多種對象創建服務。
</details>

**Q5.** Service 的默認類型是什麼？

A) NodePort
B) LoadBalancer
C) ExternalName
D) ClusterIP

<details>
<summary>答案</summary>
<b>D) ClusterIP</b>

解釋：Service 的默認類型是 ClusterIP，這種類型的 Service 只能在集群內部訪問，會分配一個虛擬的集群內部 IP 地址。
</details>

**Q6.** 如果要讓 Service 對外暴露服務，應該使用哪種類型？

A) ClusterIP
B) NodePort
C) InternalIP
D) ExternalDNS

<details>
<summary>答案</summary>
<b>B) NodePort</b>

解釋：NodePort 類型的 Service 會在集群每個節點上開啟一個端口（默認範圍 30000-32767），允許外部通過節點 IP 和端口訪問服務。
</details>

**Q7.** Service 的 IP 地址具有什麼特性？

A) 動態分配，會隨 Pod 變化而變化
B) 靜態分配，但可以 ping 通
C) 靜態分配的虛擬地址，不存在實體
D) 從 Pod IP 地址池中分配

<details>
<summary>答案</summary>
<b>C) 靜態分配的虛擬地址，不存在實體</b>

解釋：Service 的 IP 地址是靜態且虛擬的，只用於轉發流量，不存在實體網絡設備。因此無法 ping 通這個 IP 地址。
</details>

**Q8.** Kubernetes 中名字空間（namespace）的作用是什麼？

A) 現現操作系統級別的資源隔離
B) 為 API 對象提供邏輯上的隔離和分組
C) 管理網絡通信安全
D) 控制資源配額限制

<details>
<summary>答案</summary>
<b>B) 為 API 對象提供邏輯上的隔離和分組</b>

解釋：Kubernetes 的 namespace（名字空間）與 Linux 的 namespace 技術不同，它用於對 API 象實現邏輯上的分組和隔離，避免名稱衝突。
</details>

**Q9.** Service 的完整域名格式是什麼？

A) 象名.namespace.cluster
B) 對象名.namespace.svc.cluster.local
C) 對象名.svc.namespace.cluster
D) 對象名.cluster.local.svc.namespace

<details>
<summary>答案</summary>
<b>B) 對象名.namespace.svc.cluster.local</b>

解釋：Service 的完整域名格式是「對象名.名字空間.svc.cluster.local」，但很多情況下可以省略後面的部分，簡寫為「對象名.名字空間」甚至僅「對象名」。
</details>

**Q10.** NodePort 類型 Service 的默認端口範圍是？

A) 1-1023
B) 1024-49151
C) 30000-32767
D) 49152-65535

<details>
<summary>答案</summary>
<b>C) 30000-32767</b>

解釋：為避免端口衝突，Kubernetes 默認只在 30000-32767 茺圍內為 NodePort 類型的 Service 分配端口，約有 2000 多個可用端口。
</details>

---

## 第二部分：是非題（每題3分，共15分）

**Q11.** Service 可以直接選擇 Deployment 對象作為後端服務。_(True/False)_

<details>
<summary>答案</summary>
<b>False</b>

解釋：Service 雖然可以從 Deployment 創建，但它實際上是通過 selector 字段選擇 Pod，而不是 Deployment。只有 Pod 才有 IP 地址，Service 代理的是 Pod，不是 Deployment。
</details>

**Q12.** Service 的 IP 地址可以被 ping 工具測試連通性。_(True/False)_

<details>
<summary>答案</summary>
<b>False</b>

解釋：Service 的 IP 地址是虛擬地址，不存在實體，只用於轉發流量。因此 ping 工具無法得到回應數據包，測試會失敗。
</details>

**Q13.** Kubernetes 的名字空間與 Linux namespace 技術是完全相同的概念。_(True/False)_

<details>
<summary>答案</summary>
<b>False</b>

解釋：Kubernetes 的名字空間只是借用了術語，用於 API 對象的邏輯分組和隔離，與 Linux namespace 的資源隔離技術完全不同，不能混淆。
</details>

**Q14.** NodePort 類型的 Service 會在所有節點上都開啟相同的端口。_(True/False)_

<details>
<summary>答案</summary>
<b>True</b>

解釋：NodePort 類型的 Service 會在集群內的每個節點上都開啟同一個專用映射端口，這正是「NodePort」名稱的由來。外部可以通過任意節點的 IP 加上這個端口訪問服務。
</details>

**Q15.** Service 支持多種負載均衡算法，如加權輪詢、一致性哈希等。_(True/False)_

<details>
<summary>答案</summary>
<b>False</b>

解釋：Service 的負載均衡能力比較弱，只支持最簡單的 round-robin（輪詢）算法，不支持加權輪詢、一致性哈希等複雜算法。
</details>

---

## 第三部分：填空題（每題4分，共12分）

**Q16.** Service 使用 **\_\_\_\_** 字段來匹配後端的 Pod，使用 **\_\_\_\_** 字段定義端口映射規則。

<details>
<summary>答案</summary>
<b>selector, ports</b>

解釋：Service 的 YAML 定義中，selector 字段用於通過標籤選擇 Pod，ports 字段定義外部端口（port）、容器端口（targetPort）和協議（protocol）。
</details>

**Q17.** 使用 kubectl expose 命令創建 Service 時，需要指定 **\_\_\_\_** 參數定義 Service 的端口，以及 **\_\_\_\_** 參數定義 Pod 的容器端口。

<details>
<summary>答案</summary>
<b>--port, --target-port</b>

解釋：kubectl expose 命令使用 --port 指定 Service 的外部端口，--target-port 指定 Pod 的容器端口，用法類似 Docker 的 -p 參數。
</details>

**Q18.** Kubernetes 的默認名字空間是 **\_\_\_\_**，核心系統組件（如 apiserver、etcd）所在的namespace 是 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>default, kube-system</b>

解釋：Kubernetes 有默認名字空間 default，如果不顯式指定，API 對象都會在 default 中。kube-system 名字空間包含核心組件的 Pod。
</details>

---

## 第四部分：簡答題（每題8分，共16分）

**Q19.** 請解釋為什麼 Kubernetes 需要 Service 對象？Pod 的哪些特性使得服務發現成為問題？

<details>
<summary>答案</summary>

**核心原因：Pod 的生命週期短暫且動態變化**

Pod 的特性導致服務發現問題：
1. **生命周期短暂** - Pod 會不停創建和銷毀，IP 地址變化
2. **动态稳定** - Deployment/DaemonSet 保持數量穩定，但具體 Pod 會變
3. **IP 地址不固定** - Pod 銷毀重建後 IP 地址會改變

Service 的作用：
1. 提供靜態 IP 地址，屏蔽後端 Pod 的變化
2. 實現負載均衡，轉發流量到多個 Pod
3. 解決微服務架構中的服務發現關鍵問題
4. 維護動態變化的 Pod 集合，為客戶端提供穩定服務

**關鍵點：**
- Pod IP 會變 → Service IP 固定
- Pod 動態變化 → Service 自動更新
- 這對微服務架構非常重要
</details>

**Q20.** NodePort 類型的 Service 有哪些優點和缺點？在什麼場景下適合使用？

<details>
<summary>答案</summary>

**優點：**
1. 簡單易行，能快速對外暴露服務
2. 不需要額外的負載均衡器
3. 可以在集群外直接訪問服務
4. 在每個節點上開端口，提供多個訪問入口

**缺點：**
1. **端口數量有限** - 只有 30000-32767 茺圍，約 2000 多個端口
2. **非標準端口** - 不是標準 HTTP/HTTPS 端口（80/443）
3. **網絡成本** - 在所有節點開端口，kube-proxy 路由增加通信成本
4. **安全問題** - 需暴露節點 IP 地址，可能需要額外反向代理
5. **複雜度增加** - 大集群時不經濟，增加方案複雜度

**適用場景：**
- 小型集群或測試環境
- 临时暴露服務
- 內部服務對外快速驗證
- 在更好的方案（如 Ingress）出現前的过渡方案

**不適用場景：**
- 大規模生產環境
- 需要標準端口（80/443）的場景
- 有大量服務需要對外暴露的情況
</details>

---

## 第五部分：匹配題（7分）

**Q21.** 將以下 Service 相關概念與其描述進行匹配：

| 概念 | 描述 |
| ----- | --- |
| 1. ClusterIP | A. 在每個節點開啟端口，允許外部訪問 |
| 2. NodePort | B. 邏輯上隔離 API 對象的機制 |
| 3. Selector | C. 默認類型，僅集群內部可訪問 |
| 4. Namespace | D. 用於選擇要代理的 Pod |
| 5. kube-proxy | E. 維護 iptables 规則的組件 |

<details>
<summary>答案</summary>
1-C, 2-A, 3-D, 4-B, 5-E

解釋：
- ClusterIP 是 Service 的默認類型，分配虛擬 IP，僅集群內可訪問
- NodePort 在每個節點開端口，讓外部能訪問服務
- Selector 用標籤匹配選擇後端 Pod
- Namespace 對 API 對象邏輯分組和隔離
- kube-proxy 組件自動維護 iptables 規則實現負載均衡
</details>

---

## 評分標準

| 分數範圍 | 等級 | 評語 |
| ------- | --- | --- |
| 85-100分 | 優秀 | 已全面掌握 Service 的概念與實踐應用 |
| 70-84分 | 良好 | 理解 Service 的基本原理和使用方法 |
| 60-69分 | 及格 | 掌握了 Service 的核心概念 |
| 60分以下 | 需加強 | 建議重新學習本章內容 |

---

## 重點概念總結

**本測驗涵蓋的關鍵知識點：**

1. **Service 的核心作用** - 負載均衡與服務發現
2. **Service 工作原理** - iptables + kube-proxy
3. **Service YAML 定義** - selector、ports 字段
4. **Service 常用命令** - kubectl expose
5. **Service 類型** - ClusterIP（默認）、NodePort（對外暴露）
6. **名字空間** - API 對象邏輯分組
7. **DNS 城名** - Service 的域名訪問方式
8. **虛擬 IP** - Service IP 的特性（靜態、虛擬、不可 ping）
9. **Service-Pod 关係** - 通過 selector 關聯
10. **NodePort 優缺點** - 簡單但有局限性

---

_生成來源：Kubernetes 入門課程 - 第 20 講：Service：微服務架構的應對之道_