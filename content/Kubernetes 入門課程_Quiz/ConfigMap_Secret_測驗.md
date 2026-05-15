# Kubernetes ConfigMap/Secret 測驗

**主題:** Kubernetes 配置管理
**難度:** 中等（混合難度）
**題數:** 25題
**建議時間:** 30分鐘

---

## Section A: 多項選擇題（每題4分，共40分）

**Q1.** Kubernetes 中用於管理配置信息的兩個主要 API 尒象是什麼？

A) Pod 和 Service
B) ConfigMap 和 Secret
C) Volume 和 PersistentVolume
D) Deployment 和 StatefulSet

<details>
<summary>答案</summary>
<b>B) ConfigMap 和 Secret</b> - ConfigMap 用於保存明文配置，Secret 用於保存機密配置。這兩個尒象專門設計用來管理應用的配置信息，實現配置與應用的解耦。
</details>

**Q2.** 關於 ConfigMap 和 Secret 的數據存儲限制，以下哪個敘述是正確的？

A) ConfigMap 和 Secret 沒有大小限制
B) ConfigMap 限制為 10MB，Secret 限制為 1MB
C) ConfigMap 和 Secret 都限制為 1MiB
D) ConfigMap 限制為 100KB，Secret 沒有限制

<details>
<summary>答案</summary>
<b>C) ConfigMap 和 Secret 都限制為 1MiB</b> - 根據 Kubernetes 官方文檔，每個 ConfigMap 和 Secret 最多支持存儲 1MiB 的數據，因為過大的配置數據會對内存產生消耗。
</details>

**Q3.** 在 ConfigMap 和 Secret 的 YAML 定義中，用於存儲數據的字段是什麼？

A) spec
B) data
C) config
D) storage

<details>
<summary>答案</summary>
<b>B) data</b> - ConfigMap 和 Secret 存儲的是靜態字符串數據，不是容器，所以不需要用「spec」字段來說明運行時的規格，而是使用「data」字段來存儲 Key-Value 格式的配置信息。
</details>

**Q4.** Secret 尒象中的數據使用什麼編碼方式？

A) AES 加密
B) RSA 加密
C) Base64 編碼
D) MD5 哈希

<details>
<summary>答案</summary>
<b>C) Base64 編碼</b> - Secret 使用 Base64 編碼對數據進行處理，起到一定的保密作用。但要注意 Base64 只是編碼方式，不是真正的加密，仍然可以被解碼。
</details>

**Q5.** 命令 `kubectl create cm info --from-literal=k=v` 中的 `k=v` 代表什麼？

A) Kubernetes 版本
B) ConfigMap 的名稱和值
C) Key-Value 對的配置數據
D) 尒象的 API 版本

<details>
<summary>答案</summary>
<b>C) Key-Value 對的配置數據</b> - ConfigMap 里的數據都是 Key-Value 結構，`--from-literal` 参数需要使用 `k=v` 的形式，其中 k 是鍵名，v 是鍵值。
</details>

**Q6.** 在 Pod 中以環境變量方式使用 ConfigMap 時，需要使用哪個字段來引用 ConfigMap？

A) configRef
B) configMapKeyRef
C) mapRef
D) configValue

<details>
<summary>答案</summary>
<b>B) configMapKeyRef</b> - 在 Pod 的「env.valueFrom」字段中，使用「configMapKeyRef」來引用 ConfigMap 尒象，需要指定 ConfigMap 的「name」和它里面的「key」。
</details>

**Q7.** 關於 ConfigMap 和 Secret 的更新機制，以下哪個敘述正確？

A) 更新 ConfigMap/Secret後，Pod 中的環境變量會立即同步更新
B) 更新 ConfigMap/Secret後，以 Volume 方式掛載的數據會立即同步更新
C) 更新 ConfigMap/Secret後，Pod 中的環境變量不會自動更新
D) 更新 ConfigMap/Secret後，所有 Pod 都會自動重启

<details>
<summary>答案</summary>
<b>C) 更新 ConfigMap/Secret後，Pod 中的環境變量不會自動更新</b> - 環境變量在 Pod 启动時一次性注入，修改 ConfigMap/Secret 后不会自动更新 Pod 中的環境變量值。需要重启 Pod 才能獲取新的配置值。
</details>

**Q8.** Kubernetes Volume 屬於哪個層級？

A) 屬於容器
B) 屬於 Pod
C) 屬於 ConfigMap
D) 屬於節點

<details>
<summary>答案</summary>
<b>B) 屬於 Pod</b> - Volume 屬於 Pod，不属于容器，所以它和字段「containers」是同级的，都属于「spec」字段。这体现了 Pod 作为邏辑主机的概念。
</details>

**Q9.** 使用命令 `echo -n "123456" | base64` 時，為什麼必須加上 `-n` 参数？

A) 為了加快編碼速度
B) 為了去掉字符串中隐含的换行符
C) 為了提高安全性
D) 為了支持 UTF-8 編碼

<details>
<summary>答案</summary>
<b>B) 為了去掉字符串中隐含的换行符</b> - echo 命令会默認加上一個换行符，虽然看不到但是確實存在。如果不加 `-n` 参数，Base64 編碼出来的字符串就是錯誤的。
</details>

**Q10.** ConfigMap 和 Secret 的數據最終存儲在哪裡？

A) Pod 的内存中
B) Docker 镜像中
C) etcd 數據庫中
D) 節點的磁盘中

<details>
<summary>答案</summary>
<b>C) etcd 數據庫中</b> - ConfigMap 和 Secret 作为 Kubernetes 的 API 尒象，其數據存儲在 etcd 數據庫中，後續可以被其他 API 尒象使用。
</details>

---

## Section B: 判断題（每題4分，共20分）

**Q11.** ConfigMap 只能存儲明文配置數據，不能存儲任何敏感信息。 _(True/False)_

<details>
<summary>答案</summary>
<b>True</b> - ConfigMap 专门用於保存明文配置，也就是不保密、可以任意查询修改的配置信息，如服務端口、运行参数、文件路徑等。敏感信息應該使用 Secret 存儲。
</details>

**Q12.** Secret 使用 Base64 編碼是一種真正的加密方式，可以有效防止數據被破解。 _(True/False)_

<details>
<summary>答案</summary>
<b>False</b> - Base64 只是一種編碼方式，不是真正的加密。任何人都可以使用 Base64 解碼工具輕易地獲取原始數據。Secret 的 Base64 編碼只是起到一定的保密作用，讓用戶不能直接看到原始數據。
</details>

**Q13.** 使用 kubectl describe secret 命令可以直接查看 Secret 中存儲的實際數據內容。 _(True/False)_

<details>
<summary>答案</summary>
<b>False</b> - Secret 是保密的，使用 kubectl describe 不能直接看到内容，只能看到數據的大小。這與 ConfigMap 不同，kubectl describe cm 可以直接显示 Key-Value 内容。
</details>

**Q14.** 在 Pod 中以 Volume 方式掛載 ConfigMap/Secret 時，每個 Key-Value 會變成一個文件，文件名就是 Key。 _(True/False)_

<details>
<summary>答案</summary>
<b>True</b> - 以 Volume 方式掛載 ConfigMap 和 Secret 时，它们会变成目录的形式，Key-Value 会变成一个个文件，文件名就是 Key。這種形式适合存放大数据量的配置文件。
</details>

**Q15.** ConfigMap 和 Secret 必須與 Pod 在同一個 namespace 中才能被引用。 _(True/False)_

<details>
<summary>答案</summary>
<b>True</b> - Kubernetes 的 API 尒象通常只能在同一個 namespace 中引用。ConfigMap 和 Secret 在被 Pod 引用時，必須與 Pod 位於同一個 namespace，否則无法找到該尒象。
</details>

---

## Section C: 填空題（每題4分，共20分）

**Q16.** 在 Kubernetes 中，ConfigMap 用於保存 **\_\_\_\_** 配置，而 Secret 用於保存 **\_\_\_\_** 配置。

<details>
<summary>答案</summary>
<b>明文，機密</b> - ConfigMap 保存明文配置（不保密、可任意查询修改），Secret 保存機密配置（涉及敏感信息需要保密，如密码、密钥、证书等）。
</details>

**Q17.** 创建 Secret 尒象時，使用命令 `kubectl create secret **\_\_\_\_**` 來創建一般機密信息的 Secret。

<details>
<summary>答案</summary>
<b>generic</b> - Kubernetes 里的 Secret 尒象细分为很多类，对于一般機密信息，使用 `kubectl create secret generic` 命令创建。
</details>

**Q18.** 在 Pod 的 YAML 中，環境變量引用 Secret 時需要使用字段 **\_\_\_\_**，而引用 ConfigMap時需要使用字段 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>secretKeyRef，configMapKeyRef</b> - 在 Pod 的「env.valueFrom」字段中，引用 Secret 使用「secretKeyRef」，引用 ConfigMap 使用「configMapKeyRef」。
</details>

**Q19.** 在 Pod 的 Volume 定義中，引用 ConfigMap 使用字段 **\_\_\_\_**，引用 Secret 使用字段 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>configMap，secret</b> - 在「spec.volumes」字段中，引用 ConfigMap 使用「configMap」字段（需指定「name」），引用 Secret 使用「secret」字段（需指定「secretName」）。
</details>

**Q20.** ConfigMap 和 Secret 都使用 **\_\_\_\_** 格式來存儲配置數據，這種結構中的「Key」在 Volume 掛載時會變成 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>Key-Value，文件名</b> - ConfigMap 和 Secret 都以 Key-Value 格式存儲數據。以 Volume 方式掛載時，每個 Key-Value變成一個文件，Key 就是文件名。
</details>

---

## Section D: 简答题（每題8分，共24分）

**Q21.** 說明 ConfigMap 和 Secret 這兩個尒象的相同點和不同點。

<details>
<summary>答案</summary>
<b>相同點：</b>
- 都用來把配置數據和服務程序分离
- 都是一种用於存儲的 API 尒象
- 都以 Key-Value 方式存儲數據
- 都可以作为数据卷挂载在其他 API 尒象上使用
- 都存儲在 etcd 數據庫中
- 都有 1MiB 的大小限制
- 都可以通过環境變量或 Volume 方式注入 Pod

<b>不同點：</b>
- ConfigMap 存儲非機密信息（明文），Secret 存儲機密信息
- Secret 使用 Base64 編碼，ConfigMap 使用明文
- Secret 數據需要保密，kubectl describe 不能直接看到内容，只能看到大小
- ConfigMap 數據可以直接查看
- Secret 的安全性要求更高，需要通過 RBAC 等方式控制訪問权限
</details>

**Q22.** 解釋在 Pod 中使用 ConfigMap/Secret 的兩種方式（環境變量和 Volume），並說明它們各自的適用場景。

<details>
<summary>答案</summary>
<b>1. 環境變量方式：</b>
- 使用 Pod 的「env.valueFrom」字段引用 ConfigMap/Secret
- 通過「configMapKeyRef」或「secretKeyRef」指定尒象名和 Key
- 配置信息在 Pod 启动時一次性注入為環境變量
- <b>適用場景：</b> 存放簡短的字符串配置，如服務端口、簡單参数、密码等
- <b>特點：</b> 用法簡單，但不支持自動更新（需要重启 Pod）

<b>2. Volume 方式：</b>
- 在 Pod 的「spec.volumes」定義 Volume，引用 ConfigMap/Secret
- 在容器中使用「volumeMounts」將 Volume 挂载到指定路徑
- ConfigMap/Secret 变成目录，Key-Value 变成文件
- <b>適用場景：</b> 存放大数据量的配置文件，如 nginx.conf、redis.conf 等
- <b>特點：</b> 功能強大，扩展性好，支持多種存储形式，可能有更新延迟
</details>

**Q23.** 在實際工作中，如果需要更新 Pod 中使用的 ConfigMap 配置，應該如何操作？不同方式（環境變量和 Volume）的更新行為有何差異？

<details>
<summary>答案</summary>
<b>環境變量方式的更新行為：</b>
- <b>不會自動更新</b> - 環境變量在 Pod 启动時一次性注入
- 修改 ConfigMap/Secret 后，Pod 中的環境變量值保持不變
- <b>需要重启 Pod</b> 才能獲取新的配置值
- 可以刪除 Pod 並重新創建，或使用 rollout restart

<b>Volume 方式的更新行為：</b>
- <b>可能會有更新</b> - 以 Volume 挂载的配置文件可能会更新
- <b>存在延迟</b> - 更新不是立即同步，可能有后台定時任務處理
- <b>不保證实时性</b> - 更新机制依赖 Kubernetes 的内部實現
- 仍然建議重启 Pod 以確保配置完全更新

<b>最佳實踐：</b>
- 對於關鍵配置更新，建議刪除 Pod 重新創建或使用 Deployment 的 rollout restart
- 可以使用配置管理工具或程序實現配置熱更新
- 設計應用時應考慮配置更新的策略和機制
</details>

---

## Section E: 配對題（每題8分，共16分）

**Q24.** 將以下 Kubernetes YAML 字段與其所屬的 API 尒象或位置進行配對：

| 字段          | 所屬位置      |
| ------------- | ------------- |
| 1. data       | A. Pod 的容器 |
| 2. env        | B. ConfigMap/Secret |
| 3. volumes    | C. Pod 的 spec |
| 4. volumeMounts | D. Pod 的容器 |

<details>
<summary>答案</summary>
1-B, 2-A, 3-C, 4-D

<b>說明：</b>
- <b>data</b> 屬於 ConfigMap 和 Secret，用於存儲 Key-Value 配置數據
- <b>env</b> 屬於 Pod 的容器，用於定義環境變量
- <b>volumes</b> 屬於 Pod 的 spec 層級，定義存儲卷
- <b>volumeMounts</b> 屬於 Pod 的容器，用於將 Volume 挂载到容器內的路徑
</details>

**Q25.** 將以下配置信息類型與其應使用的 Kubernetes 尒象進行配對：

| 配置信息類型     | Kubernetes 尒象 |
| ---------------- | ---------------- |
| 1. 服務端口號    | A. Secret        |
| 2. 數據庫密码    | B. ConfigMap     |
| 3. Nginx 配置文件 | B. ConfigMap     |
| 4. SSL 私钥      | A. Secret        |

<details>
<summary>答案</summary>
1-B, 2-A, 3-B, 4-A

<b>說明：</b>
- <b>服務端口號</b> 是明文配置，使用 ConfigMap
- <b>數據庫密码</b> 是敏感信息，使用 Secret
- <b>Nginx 配置文件</b> 是非機密配置文件，使用 ConfigMap
- <b>SSL 私钥</b> 是敏感的機密信息，使用 Secret
</details>

---

## 測驗總結

**總分:** 120分

**難度分布:**
- Easy (60%): 基本概念記憶，如 Q1, Q3, Q11, Q16, Q24
- Medium (50%): 概念理解和應用，如 Q2, Q5, Q6, Q12, Q17, Q18, Q21, Q22
- Hard (20%): 深入分析和實際場景，如 Q7, Q15, Q23

**學習重點:**
1. ConfigMap 和 Secret 的基本概念和用途
2. ConfigMap 和 Secret 的 YAML 定義和數據結構
3. Base64 編碼在 Secret 中的作用和限制
4. 在 Pod 中使用 ConfigMap/Secret 的兩種方式
5. 環境變量和 Volume 方式的差異和適用場景
6. ConfigMap/Secret 的更新機制和實際應用策略

---

**生成來源:** 《Kubernetes 入門實戰課》第14課 - ConfigMap/Secret：怎样配置、定制我的应用