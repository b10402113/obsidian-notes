# Kubernetes 實戰演練測驗（第15講）

**科目：** Kubernetes 入門課程  
**難度：** 中等  
**題數：** 20題  
**建議時間：** 30-40分鐘  

---

## 第一部分：選擇題（每題5分，共40分）

**Q1.** Kubernetes 集群中的控制面（Control Plane）包含哪些核心組件？

A) kubelet、kube-proxy、container-runtime  
B) apiserver、etcd、scheduler、controller-manager  
C) Docker、Pod、Service、Deployment  
D) Master、Worker、Node、Cluster  

<details>
<summary>答案</summary>
<b>B) apiserver、etcd、scheduler、controller-manager</b>

控制面是 Master 節點的核心組件，負責管理集群和運維監控應用。apiserver 提供 RESTful 接口，etcd 存儲集群狀態，scheduler 負責調度，controller-manager 執行控制器邏輯。選項 A 是數據面（Worker 節點）的組件。
</details>

**Q2.** 在 Kubernetes 中，最小的管理單位是什麼？

A) Container  
B) Node  
C) Pod  
D) Deployment  

<details>
<summary>答案</summary>
<b>C) Pod</b>

Pod 是 Kubernetes 最核心的 API 對象，它捆綁了一組存在密切協作關係的容器，容器之間共享網絡和存儲，在集群裡必須一起調度一起運行。Kubernetes 通過 Pod 簡化了對容器的管理工作，所有其他任務都是對 Pod 的再包裝。
</details>

**Q3.** 在 WordPress 網站搭建過程中，為什麼需要使用 ConfigMap？

A) 存儲敏感的密碼信息  
B) 配置 Pod 的網絡策略  
C) 以環境變量形式注入配置信息給 Pod  
D) 儲存 Pod 的持久化數據  

<details>
<summary>答案</summary>
<b>C) 以環境變量形式注入配置信息給 Pod</b>

ConfigMap 對應配置信息，需要以環境變量或存儲卷的形式注入進 Pod，然後進程才能在運行時使用。在 WordPress 搭建中，ConfigMap 存儲了 MariaDB 和 WordPress 的環境變量（如數據庫名、用戶名、密碼等），統一用「聲明式」來管理。
</details>

**Q4.** 使用 `kubectl port-forward` 命令的主要目的是什麼？

A) 創建新的 Pod  
B) 將本機端口映射到集群內部的 Pod 端口  
C) 刪除集群中的對象  
D) 查看集群的節點狀態  

<details>
<summary>答案</summary>
<b>B) 將本機端口映射到集群內部的 Pod 端口</b>

Pod 都運行在 Kubernetes 內部的私有網段裡，外界無法直接訪問。kubectl port-forward 專門負責把本機的端口映射到目標對象的端口號，類似 Docker 的 -p 參數，經常用於 Kubernetes 的臨時調試和測試。
</details>

**Q5.** 在 MariaDB Pod 的 YAML 配置中，使用 `envFrom` 字段而不是 `env.valueFrom` 的原因是什麼？

A) envFrom 更安全  
B) envFrom 可以一次性導入 ConfigMap 的所有字段並指定前綴  
C) envFrom 只支持 Secret  
D) envFrom 不需要 ConfigMap  

<details>
<summary>答案</summary>
<b>B) envFrom 可以一次性導入 ConfigMap 的所有字段並指定前綴</b>

因為 ConfigMap 里的信息比較多，如果用 env.valueFrom 一個個地寫會非常麻煩，容易出錯。而 envFrom 可以一次性地把 ConfigMap 里的字段全導入進 Pod，並能夠指定變量名的前綴（如 MARIADB_），非常方便。
</details>

**Q6.** Kubernetes 中 Job/CronJob 這兩個 API 對象的主要用途是什麼？

A) 長期運行的線上業務  
B) 配置管理  
C) 離線作業和定時任務  
D) 网络代理  

<details>
<summary>答案</summary>
<b>C) 離線作業和定時任務</b>

Job/CronJob 對應的是離線作業，它們逐層包裝了 Pod，添加了作業控制和定時規則。線上業務應該直接使用 Pod 或 Deployment，而不是 Job/CronJob。
</details>

**Q7.** 在 YAML 文件中描述 API 對象時，必須寫的「頭字段」包括哪些？

A) apiVersion、spec、status  
B) kind、metadata、data  
C) apiVersion、kind、metadata  
D) name、labels、annotations  

<details>
<summary>答案</summary>
<b>C) apiVersion、kind、metadata</b>

使用 YAML 描述 API 對象有固定的格式，必須寫的「頭字段」是 apiVersion、kind、metadata，它們表示對象的版本、種類和名字等元信息。實體對象如 Pod 會再有 spec 字段，非實體對象如 ConfigMap 使用 data 字段。
</details>

**Q8.** 在 WordPress 搭建中，需要在集群外啟動 Nginx 反向代理的原因是什麼？

A) Nginx 可以加速 WordPress  
B) WordPress 使用 URL 重定向，直接使用 8080 端口會導致跳轉故障  
C) Nginx 提供安全加密功能  
D) Kubernetes 不支持直接訪問 Pod  

<details>
<summary>答案</summary>
<b>B) WordPress 使用 URL 重定向，直接使用 8080 端口會導致跳轉故障</b>

WordPress 網站使用了 URL 重定向，直接使用「8080」會導致跳轉故障，所以需要 Nginx 反向代理，保證外界看到的仍然是「80」端口號，讓網站正常工作。
</details>

---

## 第二部分：是非題（每題3分，共15分）

**Q9.** Kubernetes 的 Pod IP 地址是固定的，即使 Pod 重啟也不會變化。 _(對/錯)_  

<details>
<summary>答案</summary>
<b>錯</b> - Pod 的 IP 地址會隨著 Pod 重啟而變化。在課程中提到，這是一個需要解決的問題，後續的「中級篇」會用 Service 對象來解決服務發現問題，實現自動的服務發現機制。
</details>

**Q10.** `kubectl port-forward` 命令適合用於生產環境的服務暴露。 _(對/錯)_  

<details>
<summary>答案</summary>
<b>錯</b> - kubectl port-forward 只能用於測試和臨時調試，不適合生產環境。生產環境應該使用 Service、Ingress 等更高級的 API 對象來暴露服務。port-forward 的功能很弱，且只在本機有效。
</details>

**Q11.** ConfigMap 和 Secret 都可以用來存儲配置信息，主要區別是 Secret 會對數據進行 base64 編碼。 _(對/錯)_  

<details>
<summary>答案</summary>
<b>對</b> - ConfigMap 用於存儲普通的配置信息，Secret 用於存儲敏感信息。Secret 中的數據會進行 base64 編碼，在課後作業中也提到可以將 ConfigMap 改用 Secret 實現，只需將 data 里的值用 base64 編碼即可。
</details>

**Q12.** Kubernetes Dashboard 只能查看集群狀態，不能執行任何管理操作。 _(對/錯)_  

<details>
<summary>答案</summary>
<b>錯</b> - Dashboard 可以執行多種管理操作。在 Pod 管理界面中，右上角有 4 個重要功能：查看日誌、進入 Pod 內部（exec）、編輯 Pod、刪除 Pod，相當於執行 logs、exec、edit、delete 命令，比命令行更直觀友好。
</details>

**Q13.** YAML 是 JSON 的超集，語法更簡潔，使用「聲明式」表述對象狀態。 _(對/錯)_  

<details>
<summary>答案</summary>
<b>對</b> - YAML 是 JSON 的超集，語法更簡潔，表現能力更強。最重要的是它以「聲明式」來表述對象的狀態，不涉及具體的操作細節，這樣 Kubernetes 就能夠依靠存儲在 etcd 里的集群狀態信息，不斷地「調控」對象，直至實際狀態與期望狀態相同。
</details>

---

## 第三部分：填充題（每題5分，共15分）

**Q14.** Kubernetes 源自 Google 的內部系統 **\_\_\_\_\_\_**，它戰勝了競爭對手 Apache Mesos 和 Docker Swarm，成為容器編排領域的事實標準。

<details>
<summary>答案</summary>
<b>Borg</b>

Kubernetes 源自 Borg 系統，凝聚了 Google 的內部經驗和 CNCF 的社區智慧，戰勝了競爭對手成為容器編排領域的事實標準，也成為了雲原生時代的基础操作系統。
</details>

**Q15.** 在 WordPress 搭建步驟中，使用 `kubectl get pod **\_\_\_\_\_\_**` 命令可以查看 Pod 的 IP 地址。

<details>
<summary>答案</summary>
<b>-o wide</b>

使用 kubectl apply 創建對象後，可以用 kubectl get pod 查看狀態，如果想要獲取 IP 地址需要加上參數 -o wide。
</details>

**Q16.** Kubernetes 的數據面（Worker 節點）包含的核心組件有 **\_\_\_\_\_\_**、**\_\_\_\_\_\_** 和 **\_\_\_\_\_\_**。

<details>
<summary>答案</summary>
<b>kubelet、kube-proxy、container-runtime</b>

數據面是 Worker 節點，受 Master 節點管控，核心組件包括：kubelet（負責與 Master 通信）、kube-proxy（網絡代理）、container-runtime（容器運行時）。
</details>

---

## 第四部分：簡答題（每題10分，共20分）

**Q17.** 說明 Kubernetes 的「聲明式」管理方式與傳統運維方式的區別，以及這種方式的優勢。

<details>
<summary>答案</summary>

<b>模型答案：</b>

Kubernetes 使用「聲明式」表述對象的狀態，不涉及具體的操作細節。傳統運維方式需要列出詳細的操作步驟，而聲明式只需描述期望的最終狀態。

<b>關鍵點：</b>

- **區別：** 傳統運維是「命令式」，需要逐步執行操作；Kubernetes 是「聲明式」，只需描述期望狀態
- **優勢 1：** 降低心智負擔，調度、創建、監控等雜事都交給 Kubernetes 處理
- **優勢 2：** 自動化運維，Kubernetes 會不斷「調控」對象，直至實際狀態與期望狀態相同
- **優勢 3：** 配置文件更容易閱讀和版本化管理（相比 Shell 腳本）
- **優勢 4：** YAML 文件可以清晰描述應用狀態和它們之間的關係
</details>

**Q18.** 在 WordPress 網站搭建的四個步驟中，第三步「端口映射」和第四步「反向代理」各自的目的是什麼？為什麼需要兩步而不是一步完成？

<details>
<summary>答案</summary>

<b>模型答案：</b>

<b>第三步（端口映射）：</b>
使用 kubectl port-forward 命令將本機端口（如 8080）映射到 WordPress Pod 的 80 端口。因為 Pod 都運行在 Kubernetes 內部的私有網段裡，外界無法直接訪問，需要通過端口轉發來傳遞數據。

<b>第四步（反向代理）：</b>
在集群外啟動 Nginx 反向代理，監聽 80 端口，將請求轉發到 127.0.0.1:8080（port-forward 創建的本地地址）。

<b>需要兩步的原因：</b>
- WordPress 網站使用了 URL 重定向，直接使用「8080」端口會導致跳轉故障
- 需要保證外界看到的仍然是「80」端口號，讓網站正常工作
- port-forward 創建的地址是 127.0.0.1:8080，只能本機訪問
- Nginx 反向代理將這個內部地址轉換為對外的 80 端口服務

這種方式模擬了真實環境的部署，增加了部署難度，讓學員理解 Kubernetes 內外網絡隔離的特性。
</details>

---

## 第五部分：配對題（10分）

**Q19.** 將以下 Kubernetes API 對象與其主要用途進行配對：

| 對象              | 用途            |
| ----------------- | --------------- |
| 1. Pod            | A. 配置信息存儲  |
| 2. ConfigMap      | B. 最小管理單位  |
| 3. Job            | C. 離線作業      |
| 4. Secret         | D. 定時任務      |
| 5. CronJob        | E. 敏感信息存儲  |

<details>
<summary>答案</summary>
<b>1-B, 2-A, 3-C, 4-E, 5-D</b>

<b>說明：</b>
- Pod：捆綁一組密切協作的容器，是 Kubernetes 最小管理單位
- ConfigMap：存儲配置信息，以環境變量或存儲卷注入 Pod
- Job：離線作業，包裝 Pod 添加作業控制
- Secret：存儲敏感信息，數據進行 base64 編碼
- CronJob：定時任務，包裝 Job 添加定時規則
</details>

---

## 答案總結

本測驗涵蓋 Kubernetes 實戰演練的核心概念：

- **架構理解：** Master/Node 架構、控制面與數據面組件
- **API 對象：** Pod、ConfigMap、Secret、Job/CronJob 的用途與特性
- **實戰操作：** WordPress 搭建的四個步驟、端口映射、反向代理
- **工具使用：** kubectl 命令、Dashboard 管理界面
- **設計理念：** 聲明式管理、自動化運維的優勢

---

<b>生成來源：Kubernetes 入門課程 第15講「實戰演練：玩轉Kubernetes（1）」</b>  
<b>測驗設計：包含選擇題、是非題、填充題、簡答題、配對題五種題型</b>  
<b>難度分布：基礎題 60%，應用題 30%，分析題 10%</b>