# StatefulSet：怎麼管理有狀態的應用？測驗

**課程:** Kubernetes 入門課程
**主題:** StatefulSet與有狀態應用管理
**難度:** 中等
**題數:** 20題
**時間:** 30分鐘

---

## 第一部分：選擇題 (每題5分，共30分)

**Q1.** 在 Kubernetes 中,StatefulSet 主要用來管理哪種類型的應用？

A) 無狀態應用（如 Nginx）
B) 有狀態應用（如 Redis、MySQL）
C) 批次處理任務
D) Daemon 類型的服務

<details>
<summary>答案</summary>
<b>B)</b> StatefulSet專門用來管理有狀態應用,如Redis、MySQL等需要保存運行狀態的應用。無狀態應用通常使用Deployment管理,批次處理任務使用Job/CronJob,而Daemon服務使用DaemonSet。
</details>

---

**Q2.** 以下哪項不是有狀態應用的特點？

A) 需要保存運行狀態數據
B) Pod 名稱需要固定且有順序
C) 需要穩定的網絡標識
D) 重啟後狀態可以完全丟失

<details>
<summary>答案</summary>
<b>D)</b> 有狀態應用的核心特點是必須保存和恢復運行狀態,狀態丟失會導致嚴重問題。其他選項都是正確的描述:需要持久化數據(A)、固定名稱和啟動順序(B)、穩定的網絡標識(C)。
</details>

---

**Q3.** StatefulSet 的 YAML 配置中,哪個字段是 Deployment 所沒有的關鍵字段？

A) replicas
B) selector
C) serviceName
D) template

<details>
<summary>答案</summary>
<b>C)</b> serviceName是StatefulSet特有的字段,用來指定與之關聯的Service名稱。其他字段(replicas、selector、template)在Deployment中也存在。
</details>

---

**Q4.** StatefulSet 管理的 Pod 的命名規則是什麼？

A) 完全隨機的哈希值
B) Pod名-隨機字符串
C) StatefulSet名-序號(從0開始)
D) 使用者自定義名稱

<details>
<summary>答案</summary>
<b>C)</b> StatefulSet管理的Pod會按順序編號,命名格式為"StatefulSet名-序號",例如redis-sts-0、redis-sts-1,序號從0開始依次遞增。這保證了固定的啟動順序和穩定的身份標識。
</details>

---

**Q5.** 當為 StatefulSet 配置 Service 時,應該如何設置才能避免分配不必要的 ClusterIP？

A) 不創建 Service
B) 設置 clusterIP: "None"
C) 不配置 selector
D) 只使用 NodePort

<details>
<summary>答案</summary>
<b>B)</b> 設置clusterIP: None可以創建Headless Service,不分配ClusterIP。因為StatefulSet的Pod已有穩定域名,外界可直接訪問特定Pod,不需要Service的負載均衡功能。這既安全又節省資源。
</details>

---

**Q6.** StatefulSet 中用來直接定義 PVC 的字段名稱是什麼？

A) persistentVolumeClaim
B) volumeClaimTemplates
C) storageClaims
D) pvcTemplates

<details>
<summary>答案</summary>
<b>B)</b> volumeClaimTemplates字段允許在StatefulSet YAML中直接嵌入PVC定義,會為每個Pod自動創建PVC,確保持久化存儲與Pod的一對一绑定關係。
</details>

---

## 第二部分：是非題 (每題4分，共20分)

**Q7.** 理論上所有應用都是有狀態的,只是有些應用的狀態信息不重要,可以稱為"無狀態應用"。 _(是/否)_

<details>
<summary>答案</summary>
<b>是</b> - 從持久化存儲的角度看,任何應用都可以保存運行狀態。但如Nginx等Web服務器的狀態信息(除日志外)不重要,重啟後不需恢復狀態也能正常運行,因此被歸類為無狀態應用。
</details>

---

**Q8.** StatefulSet 可以直接使用 kubectl create 命令生成 YAML 樣板文件。 _(是/否)_

<details>
<summary>答案</summary>
<b>否</b> - StatefulSet不能直接用kubectl create創建樣板文件,需要手動編寫YAML或修改Deployment的YAML配置。這與DaemonSet類似,都是Deployment的特例。
</details>

---

**Q9.** StatefulSet 管理的 Pod 啟動順序是完全隨機的,沒有固定次序。 _(是/否)_

<details>
<summary>答案</summary>
<b>否</b> - StatefulSet管理的Pod會按照序號依次啟動(0號先於1號),這正是它解決有狀態應用啟動順序問題的關鍵特性。Deployment的Pod才是隨機啟動。
</details>

---

**Q10.** StatefulSet 中 Pod 的完整域名格式為"Pod名.服務名.名字空間.svc.cluster.local"。 _(是/否)_

<details>
<summary>答案</summary>
<b>是</b> - Service會為StatefulSet的Pod創建穩定域名,完整格式確實是"Pod名.服務名.名字空間.svc.cluster.local",也可以簡寫為"Pod名.服務名"。
</details>

---

**Q11.** StatefulSet 關聯的 PVC 名稱是隨機生成的,無法預測。 _(是/否)_

<details>
<summary>答案</summary>
<b>否</b> - PVC名稱有固定規律,格式為"PVC名-StatefulSet名-序號",例如redis-100m-pvc-redis-pv-sts-0。這樣即使Pod重建,因名稱相同還能找到之前的PVC恢復數據。
</details>

---

## 第三部分：填空題 (每題6分，共24分)

**Q12.** StatefulSet 解決了有狀態應用的三個關鍵問題:______、______和______。

<details>
<summary>答案</summary>
<b>啟動順序、依賴關係、網絡標識</b>
</details>

---

**Q13.** 在 Pod 部可以使用環境變量 ______ 或執行命令 ______ 來查看 Pod 的名稱。

<details>
<summary>答案</summary>
<b>$HOSTNAME, hostname</b>
</details>

---

**Q14.** 為 StatefulSet 配置 Service 時,Service 的 ______ 必須與 StatefulSet 的 serviceName 字段一致。

<details>
<summary>答案</summary>
<b>metadata.name</b>
</details>

---

**Q15.** 使用 volumeClaimTemplates 嵌入的 PVC 定義中,需要指定 storageClassName、accessModes 和 ______ 等關鍵字段。

<details>
<summary>答案</summary>
<b>resources.requests.storage</b>
</details>

---

## 第四部分：簡答題 (每題10分，共20分)

**Q16.** 說明為什麼訪問 StatefulSet 的 Pod 時應該使用每個 Pod 的專屬域名(如 redis-sts-0.redis-svc),而不應該通過 Service 的負載均衡功能？

<details>
<summary>答案</summary>

**主要原因:**

- **穩定性:** Pod的IP地址可能變化,但域名由Service維護是穩定不變的
- **精確訪問:** 有狀態應用通常有主從/主備關係,需要精確訪問特定Pod(如訪問master節點)
- **避免混淆:** Service的負載均衡會隨機分配到不同Pod,可能導致狀態不一致
- **資源節約:** Headless Service(clusterIP: None)不分配IP,節省系統資源
- **安全考量:** 直接訪問特定Pod更符合有狀態應用的業務邏輯

**關鍵點:**
- Service域名格式: Pod名.服務名 或 Pod名.服務名.名字空間.svc.cluster.local
- 外界客戶端可使用固定編號訪問特定實例

</details>

---

**Q17.** 說明 volumeClaimTemplates 字段的作用,以及它如何確保 Pod 重啟後能恢復數據？

<details>
<summary>答案</summary>

**volumeClaimTemplates的作用:**

- **自動創建PVC:** 創建StatefulSet時自動為每個Pod生成PVC
- **一對一绑定:** 每個Pod都有專屬的PVC,不會混淆
- **嵌入式定義:** 直接在StatefulSet YAML中定義,提高可用性

**數據恢復機制:**

- **固定命名:** PVC名稱為"PVC名-StatefulSet名-序號",即使Pod重建名稱相同
- **持久绑定:** Pod重建後仍能找到之前創建的PVC
- **數據持久化:** PVC將數據存儲在持久化存儲(如NFS),Pod刪除不會刪除數據
- **自動掛載:** Pod重啟後自動掛載相同的PVC到指定目錄
- **狀態恢復:** 應用從備份文件讀取數據恢復到之前的狀態

**示例流程:**
1. redis-pv-sts-0創建,生成PVC redis-100m-pvc-redis-pv-sts-0
2. Pod寫入數據到/data目錄(掛載的NFS存儲)
3. Pod被刪除重建,新Pod仍命名為redis-pv-sts-0
4. 新Pod自動绑定相同的PVC
5. 恢復/data目錄中的數據

</details>

---

## 第五部分：配對題 (每題6分，共6分)

**Q18.** 將以下概念與其對應的描述進行配對:

| 概念                    | 描述                                          |
| ----------------------- | --------------------------------------------- |
| 1. Deployment           | A. 專門管理有狀態應用,Pod有固定名稱和順序      |
| 2. StatefulSet          | B. 管理無狀態應用,Pod名稱隨機                 |
| 3. Headless Service     | C. 為StatefulSet提供穩定域名,不分配ClusterIP  |

<details>
<summary>答案</summary>
1-B, 2-A, 3-C

**解析:**
- Deployment:適合無狀態應用,Pod完全隨機命名
- StatefulSet:適合有狀態應用,Pod按序號命名(redis-sts-0, redis-sts-1)
- Headless Service:設置clusterIP: None,為StatefulSet Pod提供穩定域名但無負載均衡

</details>

---

---

## 總結

本測驗涵蓋了 StatefulSet 的核心概念:

- **有狀態應用特點:** 需要持久化狀態、固定啟動順序、穩定網絡標識
- **StatefulSet配置:** serviceName字段、Pod命名規則、volumeClaimTemplates
- **Service配合:** Headless Service、域名格式、避免負載均衡
- **數據持久化:** PVC自動創建、命名規律、數據恢復機制

**學習重點:**
1. 理解無狀態與有狀態應用的區別
2. 掌握 StatefulSet 的 YAML 編寫
3. 了解 Pod 命名和啟動順序的重要性
4. 學會配置 Headless Service 和域名訪問
5. 熟悉 volumeClaimTemplates 的使用

---

_Generated from: Kubernetes 入門課程 - 26｜StatefulSet：怎么管理有状态的应用？_