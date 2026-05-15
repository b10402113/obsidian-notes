# Kubernetes 實戰演練（三）測驗

**科目:** Kubernetes 入門課程
**難度:** 混合（基礎、中級、高級）
**題數:** 20 題
**建議時間:** 30 分鐘

---

## 第一部分：選擇題（每題 3 分，共 30 分）

**Q1.** 在 Kubernetes 中，PersistentVolume（PV）的主要功能是什麼？

A) 管理 Pod 的生命週期
B) 抽象持久化存儲設備，如 LocalDisk、NFS、Ceph 等
C) 控制容器的資源配額
D) 管理集群的網絡通信

<details>
<summary>答案</summary>
<b>B) 抽象持久化存儲設備，如 LocalDisk、NFS、Ceph 等</b>

PV 是 Kubernetes 對持久化存儲的抽象，代表了各種存儲設備，和 CPU、內存一樣屬於集群的公共資源。它不負責 Pod 生命週期管理、資源配額控制或網絡通信。
</details>

---

**Q2.** 關於 StorageClass 的作用，以下哪項描述是正確的？

A) 直接創建 PVC 對象
B) 分類存儲設備，讓用戶更容易選擇 PV 對象
C) 替代 PV 的功能
D) 管理容器內部的臨時存儲

<details>
<summary>答案</summary>
<b>B) 分類存儲設備，讓用戶更容易選擇 PV 對象</b>

StorageClass 的作用是分類存儲設備，彌補不同存儲設備之間差異大的問題，幫助用戶更容易選擇合適的 PV 對象。它不直接創建 PVC，也不能替代 PV 的功能。
</details>

---

**Q3.** StatefulSet 與 Deployment 的主要區別是什麼？

A) StatefulSet 不支持滾動更新
B) StatefulSet 會對 Pod 順序編號並保證穩定的網絡標識
C) StatefulSet 只能用於無狀態應用
D) StatefulSet 不需要 Service 對象

<details>
<summary>答案</summary>
<b>B) StatefulSet 會對 Pod 順序編號並保證穩定的網絡標識</b>

StatefulSet 會對 Pod 順序編號、順序創建，保證應用有確定的啟動先後次序，並為每個 Pod 單獨創建順序編號的域名，保證 Pod 有穩定的網絡標識。這使得 StatefulSet 適合管理有狀態應用。
</details>

---

**Q4.** Kubernetes 的滾動更新策略實際上是哪兩個動作的同步進行？

A) 創建和刪除
B) 擴容和縮容
C) 啟動和停止
D) 部署和回退

<details>
<summary>答案</summary>
<b>B) 擴容和縮容</b>

滾動更新策略實際上是兩個同步進行的「擴容」和「縮容」動作，這樣在更新過程中始終會有 Pod 處於可用狀態，能夠平穩地對外提供服務。
</details>

---

**Q5.** Kubernetes 的檢查探針（Probe）不包括以下哪種類型？

A) Startup Probe
B) Liveness Probe
C) Readiness Probe
D) Health Probe

<details>
<summary>答案</summary>
<b>D) Health Probe</b>

Kubernetes 內置的檢查探針有三種：Startup Probe（啟動探測）、Liveness Probe（存活探測）和 Readiness Probe（就緒探測）。沒有 Health Probe 這個類型。
</details>

---

**Q6.** Metrics Server 在 Kubernetes 中的主要功能是什麼？

A) 存儲應用的日誌數據
B) 收集 Kubernetes 核心資源指標
C) 管理集群的網絡策略
D) 提供 Web 管理界面

<details>
<summary>答案</summary>
<b>B) 收集 Kubernetes 核心資源指標</b>

Metrics Server 專門用來收集 Kubernetes 核心資源指標，可以用 kubectl top 來查看集群的狀態，它也是水平自動伸縮對象 HorizontalPodAutoscaler 的前提條件。
</details>

---

**Q7.** 關於 Prometheus，以下哪項描述是錯誤的？

A) 是 CNCF 畢業項目
B) 是雲原生監控領域的「事實標準」
C) 可以與 Grafana 集成進行可視化監控
D) 只能用於監控 Kubernetes 集群

<details>
<summary>答案</summary>
<b>D) 只能用於監控 Kubernetes 集群</b>

Prometheus 是繼 Kubernetes 之後的第二個 CNCF 畢業項目，是雲原生監控領域的「事實標準」，可以與 Grafana 集成監控各種指標，不僅限於 Kubernetes 集群。
</details>

---

**Q8.** 在 Kubernetes 網絡模型中，「IP-per-pod」的含義是什麼？

A) 每個節點有一個 IP 地址
B) 每個 Service 有獨立的 IP 地址
C) 每個 Pod 有獨立的 IP 地址
D) 每個容器共享同一個 IP 地址

<details>
<summary>答案</summary>
<b>C) 每個 Pod 有獨立的 IP 地址</b>

Kubernetes 定義了平坦的網絡模型「IP-per-pod」，意味著每個 Pod 都有自己的獨立 IP 地址，實現它需要符合 CNI 標準。
</details>

---

**Q9.** 在實戰演練中，為什麼要將 MariaDB 從 Deployment 改為 StatefulSet？

A) 為了提高性能
B) 為了實現數據持久化存儲
C) 為了簡化配置
D) 為了減少資源消耗

<details>
<summary>答案</summary>
<b>B) 為了實現數據持久化存儲</b>

MariaDB 作為數據庫需要持久化存儲數據，StatefulSet 可以通過 volumeClaimTemplates 為每個 Pod 生成獨立的 PVC，實現存儲卷與 Pod 的獨立綁定，確保數據不會因為對象的銷毀而丟失。
</details>

---

**Q10.** 在部署 Dashboard 時，為什麼需要使用 Secret 對象存儲證書？

A) 為了提高訪問速度
B) 因為證書屬於機密信息，需要安全存儲
C) 為了減少配置文件大小
D) 為了方便版本控制

<details>
<summary>答案</summary>
<b>B) 因為證書屬於機密信息，需要安全存儲</b>

證書和私鑰文件屬於機密信息，不能明文存儲在配置文件中。使用 Secret 對象（類型為 tls）可以安全地存儲這些敏感信息供 Ingress 使用。
</details>

---

## 第二部分：是非題（每題 2 分，共 10 分）

**Q11.** PV 一般由系統管理員創建，用戶如果要使用 PV 需要通過 PVC 來申請。_(對/錯)_

<details>
<summary>答案</summary>
<b>對</b> - PV 是集群的公共資源，通常由系統管理員創建。用戶通過 PVC（PersistentVolumeClaim）聲明需求的容量、訪問模式等參數，Kubernetes 會查找最合適的 PV 分配給用戶使用。
</details>

---

**Q12.** StatefulSet 創建的 Pod 名稱是隨機生成的，與 Deployment 相同。_(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - Deployment 創建的 Pod 是隨機的名字，而 StatefulSet 會對 Pod 順序編號、順序創建（如 pod-0, pod-1 等），保證 Pod 有穩定的網絡標識。
</details>

---

**Q13.** 使用 kubectl rollout undo 命令可以回退應用的版本更新。_(對/錯)_

<details>
<summary>答案</summary>
<b>對</b> - kubectl rollout undo 是用於回退應用版本的命令。應用的更新歷史可以用 kubectl rollout history 查看，如果更新出現問題，就可以使用 undo 命令回退。
</details>

---

**Q14.** Flannel 網絡插件使用 Route 模式，性能比 Calico 高。_(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - Flannel 使用 Overlay 模式，性能較低；Calico 使用 Route 模式，性能較高。
</details>

---

**Q15.** Dashboard 默認使用 HTTP 協議，可以直接通過瀏覽器訪問。_(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - Dashboard 默認使用的是加密的 HTTPS 協議，拒絕明文 HTTP 訪問。如果要通過 Ingress 訪問，需要配置 TLS 證書。
</details>

---

## 第三部分：填充題（每題 4 分，共 20 分）

**Q16.** StatefulSet 通過 **\_\_\_\_**、**\_\_\_\_** 和 **\_\_\_\_** 三個關鍵能力，可以很好地處理 Redis、MySQL 等有狀態應用。

<details>
<summary>答案</summary>
<b>啟動順序、穩定域名、存儲模板</b>

StatefulSet 的三個關鍵能力是：啟動順序（順序編號創建 Pod）、穩定域名（為每個 Pod 創建穩定的網絡標識）和存儲模板（volumeClaimTemplates 為每個 Pod 生成獨立的 PVC）。
</details>

---

**Q17.** Kubernetes 的檢查探針有三種探測方式，分別是 **\_\_\_\_**、**\_\_\_\_** 和 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>exec、tcpSocket、httpGet</b>

檢查探針的三種探測方式：exec（執行命令）、tcpSocket（TCP 連接檢測）和 httpGet（HTTP 請求檢測）。
</details>

---

**Q18.** 在 WordPress 實戰演練中，MariaDB 使用 StatefulSet 部署後，數據掛載到容器的 **\_\_\_\_** 目錄，WordPress 需要連接的 MariaDB 域名是 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>/var/lib/mysql、maria-sts-0.maria-svc</b>

MariaDB 的數據目錄是 /var/lib/mysql，WordPress 連接的域名是 StatefulSet Pod 的穩定域名格式：{pod-name}.{service-name}，即 maria-sts-0.maria-svc。
</details>

---

**Q19.** Kubernetes 名字空間的資源配額使用 **\_\_\_\_** 對象，除了限制 CPU 和內存，還能限制 **\_\_\_\_** 和 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>ResourceQuota、存儲容量、API 對象數量</b>

ResourceQuota 用於名字空間的資源配額管理，可以限制 CPU、內存、存儲容量以及各種 API 對象的數量，避免多用戶互相擠占。
</details>

---

**Q20.** 在 Dashboard 部署中，需要創建 **\_\_\_\_** 類型的 Secret 對象來存儲 TLS 證書，Ingress 需要在 **\_\_\_\_** 字段中指定後端使用 HTTPS 服務。

<details>
<summary>答案</summary>
<b>kubernetes.io/tls、annotations</b>

證書需要用 kubernetes.io/tls 類型的 Secret 存儲。在 Ingress 的 annotations 字段中通過 nginx.org/ssl-services 指定後端是 HTTPS 服務。
</details>

---

## 第四部分：簡答題（每題 10 分，共 30 分）

**Q21.** 請解釋什麼是動態存儲卷（Dynamic Storage Volume），它如何工作？

<details>
<summary>答案</summary>
動態存儲卷是一種自動化創建 PV 的機制，用於解決手動創建 PV 工作量大且容易出錯的問題。

**工作原理：**
1. 在 StorageClass 中綁定一個 Provisioner 對象
2. 用戶創建 PVC 並指定 StorageClass
3. Provisioner 根據 PVC 的需求自動創建符合要求的 PV
4. Kubernetes 將創建的 PV 綁定到 PVC

**關鍵點：**
- 需要在 StorageClass 中配置 Provisioner
- Provisioner 代替人工自動創建 PV
- 根據 PVC 中聲明的容量、訪問模式等參數創建 PV
</details>

---

**Q22.** 請說明 StatefulSet 的 volumeClaimTemplates 字段的作用和工作方式。

<details>
<summary>答案</summary>
volumeClaimTemplates 是 StatefulSet 中用於定義持久化存儲的字段，其作用和工作方式如下：

**作用：**
- 為每個 Pod 自動創建獨立的 PVC
- 實現存儲卷與 Pod 的獨立綁定
- 確保每個 Pod 有自己專屬的持久化存儲

**工作方式：**
1. 在 StatefulSet 的 YAML 中定義 volumeClaimTemplates，其實就是一個 PVC 模板
2. 當 StatefulSet 創建 Pod 時，會根據模板為每個 Pod 生成一個 PVC
3. 每個 PVC 會申請自己的 PV，實現存儲卷的獨立性
4. 即使 Pod 被刪除重建，PVC 和 PV 依然保留，數據不會丟失

**示例場景：**
MariaDB 使用 StatefulSet 部署時，volumeClaimTemplates 確保每個數據庫實例有獨立的存儲卷，即使 Pod 重啟數據也不會丟失。
</details>

---

**Q23.** 在實戰演練中，為什麼要在 Dashboard 前配置 Ingress 反向代理？請說明配置過程中的關鍵步驟。

<details>
<summary>答案</summary>
在 Dashboard 前配置 Ingress 反向代理是為了實戰練習 Ingress 的用法，並提供更靈活的訪問方式。

**關鍵配置步驟：**

1. **生成 TLS 證書：**
   - Dashboard 默認使用 HTTPS，需要為 Ingress 配置證書
   - 使用 openssl 生成自簽名證書或向 CA 申請證書
   - 生成證書文件和私鑰文件

2. **創建 Secret 存儲證書：**
   - 使用 kubectl create secret tls 命令
   - 類型為 kubernetes.io/tls
   - 指定名字空間為 kubernetes-dashboard

3. **配置 Ingress：**
   - 創建 IngressClass 指定 Controller
   - 在 annotations 中指定後端是 HTTPS 服務（nginx.org/ssl-services）
   - 在 tls 字段中指定域名和 Secret

4. **部署 Ingress Controller：**
   - 修改 Ingress Controller 的 args，指定自定義的 IngressClass
   - 創建 NodePort Service 暴露服務

5. **配置訪問：**
   - 為域名添加解析（修改 /etc/hosts）
   - 使用 HTTPS 協議和指定端口訪問
</details>

---

## 第五部分：配對題（10 分）

**Q24.** 請將以下網絡插件與其特性進行配對：

| 網絡插件 | 特性 |
|---------|------|
| 1. Flannel | A. 使用 Route 模式，性能較高 |
| 2. Calico | B. 使用 Overlay 模式，性能較低 |
| 3. Cilium | C. 新一代網絡插件，支持 eBPF |

<details>
<summary>答案</summary>
1-B, 2-A, 3-C

**解析：**
- Flannel 使用 Overlay 模式，封裝網絡數據包，有一定性能損耗
- Calico 使用 Route 模式，直接路由，性能較高
- Cilium 是新一代網絡插件，基於 eBPF 技術，性能和功能都有優勢
</details>

---

**Q25.** 請將以下 Kubernetes 對象與其主要功能進行配對：

| 對象 | 功能 |
|------|------|
| 1. PersistentVolume | A. 聲明存儲需求，申請 PV |
| 2. PersistentVolumeClaim | B. 抽象存儲設備，代表集群公共資源 |
| 3. StorageClass | C. 分類存儲設備，支持動態供應 |
| 4. StatefulSet | D. 管理有狀態應用，提供穩定標識 |
| 5. ResourceQuota | E. 限制名字空間資源配額 |

<details>
<summary>答案</summary>
1-B, 2-A, 3-C, 4-D, 5-E

**解析：**
- PersistentVolume (PV)：持久化存儲的抽象，代表存儲設備
- PersistentVolumeClaim (PVC)：用戶對存儲的聲明，用於申請 PV
- StorageClass：存儲類別，用於分類存儲和動態供應
- StatefulSet：管理有狀態應用的控制器，提供穩定網絡標識和存儲
- ResourceQuota：資源配額對象，限制名字空間的資源使用
</details>

---

## 答案總覽

| 題號 | 答案 |
|------|------|
| Q1 | B |
| Q2 | B |
| Q3 | B |
| Q4 | B |
| Q5 | D |
| Q6 | B |
| Q7 | D |
| Q8 | C |
| Q9 | B |
| Q10 | B |
| Q11 | 對 |
| Q12 | 錯 |
| Q13 | 對 |
| Q14 | 錯 |
| Q15 | 錯 |
| Q16 | 啟動順序、穩定域名、存儲模板 |
| Q17 | exec、tcpSocket、httpGet |
| Q18 | /var/lib/mysql、maria-sts-0.maria-svc |
| Q19 | ResourceQuota、存儲容量、API 對象數量 |
| Q20 | kubernetes.io/tls、annotations |
| Q24 | 1-B, 2-A, 3-C |
| Q25 | 1-B, 2-A, 3-C, 4-D, 5-E |

---

_生成來源: 32｜实战演练：玩转Kubernetes（3）_