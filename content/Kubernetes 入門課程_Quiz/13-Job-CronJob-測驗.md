# Job/CronJob 測驗

**科目:** Kubernetes 入門課程
**難度:** 混合 (基礎、中級、進階)
**題目數:** 20 題
**建議時間:** 30 分鐘

---

## 第一部分: 選擇題 (每題 5 分，共 50 分)

**Q1.** 為什麼 Kubernetes 不直接在 Pod 層面添加功能來處理所有業務需求？

A) 因為 Pod 功能太過複雜，難以擴展
B) 因為這違反了「單一職責」原則，應該讓 Pod 專注於容器管理
C) 因為 Pod 的效能不足以處理複雜業務
D) 因為 Kubernetes 的 API 限制無法在 Pod 上添加新功能

<details>
<summary>答案</summary>
<b>B) 因為這違反了「單一職責」原則，應該讓 Pod 專注於容器管理</b>

Kubernetes 採用面向對象設計思想，遵循「單一職責」原則。Pod 已經是一個相對完善的對象，專門負責管理容器，不應該再「畫蛇添足」地為它擴充功能，而是要保持它的獨立性，其他功能由其他對象來實現。
</details>

---

**Q2.** 在 Kubernetes 的對象設計中，「組合優於繼承」原則的含義是什麼？

A) 對象應該只專注於做好一件事情
B) 應該讓對象在運行時產生聯繫，保持鬆耦合，不用硬編碼固定對象關係
C) 對象應該通過繼承來擴展功能
D) 對象應該避免與其他對象通信

<details>
<summary>答案</summary>
<b>B) 應該讓對象在運行時產生聯繫，保持鬆耦合，不用硬編碼固定對象關係</b>

「組合優於繼承」強調對象之間應該通過靈活的組合方式建立關係，而不是通過繼承的強耦合方式。這樣可以讓每個對象保持獨立性，同時又能協作完成複雜任務。
</details>

---

**Q3.** 以下哪項屬於「離線業務」的特性？

A) 一旦運行起來基本上不會停止，永遠在線
B) 直接服務於外部用戶
C) 必定會退出，不會無期限運行
D) 運行時間短，不需要狀態檢查

<details>
<summary>答案</summary>
<b>C) 必定會退出，不會無期限運行</b>

離線業務的特點是必定會退出，不會無期限地運行下去，因此需要考慮運行超時、狀態檢查、失敗重試、獲取計算結果等管理事項。這與在線業務（如 Nginx、Node.js 等）形成對比。
</details>

---

**Q4.** 在 Job 的 YAML 定義中，`spec.template` 字段的作用是什麼？

A) 定義 Job 的調度策略
B) 定義用來運行業務的 Pod 模板
C) 定義 Job 的超時時間
D) 定義 Job 的並發數量

<details>
<summary>答案</summary>
<b>B) 定義用來運行業務的 Pod 模板</b>

`spec.template` 字段定義了一個「應用模板」，裡面嵌入了一個 Pod，這樣 Job 就可以從這個模板來創建出 Pod。這是組合模式的應用，Job 對象裡嵌入了 Pod 對象。
</details>

---

**Q5.** 在 Job 中，如果 `restartPolicy` 設置為 `OnFailure`，當 Pod 運行失敗時會發生什麼？

A) 容器不會重啟，Job 會調度生成新的 Pod
B) 容器會在原地重啟
C) Job 會立即終止並報錯
D) 系統會發送警報但不採取任何行動

<details>
<summary>答案</summary>
<b>B) 容器會在原地重啟</b>

`restartPolicy` 設置為 `OnFailure` 時，失敗會原地重啟容器。如果設置為 `Never`，則不重啟容器，讓 Job 去重新調度生成一個新的 Pod。
</details>

---

**Q6.** Job 的 `completions` 字段設置為 4，`parallelism` 字段設置為 2，這意味著什麼？

A) 最多運行 2 個 Pod，總共運行 4 次
B) 總共需要運行完 4 個 Pod，同一時刻最多並發 2 個 Pod
C) 運行 4 次 Job，每次並發 2 個任務
D) 需要 2 個 Pod 完成，每個 Pod 運行 4 次

<details>
<summary>答案</summary>
<b>B) 總共需要運行完 4 個 Pod，同一時刻最多並發 2 個 Pod</b>

`completions` 表示 Job 完成需要運行多少個 Pod（預設是 1 個），`parallelism` 表示允許並發運行的 Pod 數量。這樣可以控制作業的並行度和完成數量。
</details>

---

**Q7.** CronJob 的 YAML 定義中，為什麼會有三層嵌套的 `spec` 字段？

A) 這是 Kubernetes 的設計錯誤
B) 因為每層 spec 分別屬於 CronJob、Job、Pod 對象的規格聲明
C) 為了提高 YAML 文件的可讀性
D) 為了兼容舊版本的 Kubernetes

<details>
<summary>答案</summary>
<b>B) 因為每層 spec 分別屬於 CronJob、Job、Pod 對象的規格聲明</b>

第一個 spec 是 CronJob 自己的對象規格聲明，第二個 spec 從屬於 jobTemplate（定義 Job 對象），第三個 spec 從屬於 template（定義 Job 裡運行的 Pod）。這體現了組合模式，CronJob 組合了 Job，Job 又組合了 Pod。
</details>

---

**Q8.** 以下哪個命令可以用來創建 Job 的 YAML 樣板文件？

A) `kubectl run job --image=busybox`
B) `kubectl create job --image=busybox`
C) `kubectl apply job --image=busybox`
D) `kubectl generate job --image=busybox`

<details>
<summary>答案</summary>
<b>B) `kubectl create job --image=busybox`</b>

要創建 Pod 以外的其他 API 對象，需要使用 `kubectl create` 命令，再加上對象的類型名。`kubectl run` 只能創建 Pod。加上 `--dry-run=client -o yaml` 參數可以生成 YAML 樣板文件。
</details>

---

**Q9.** 在 Kubernetes 的「控制鏈」中，正確的順序是什麼？

A) CronJob → Pod → Job → Container
B) Job → CronJob → Pod → Container
C) CronJob → Job → Pod → Container
D) Pod → Job → CronJob → Container

<details>
<summary>答案</summary>
<b>C) CronJob → Job → Pod → Container</b>

Kubernetes 的控制鏈是：CronJob 使用定時規則控制 Job，Job 使用並發數量控制 Pod，Pod 再定義參數控制容器，容器再隔離控制進程，進程最終實現業務功能。每個環節各司其職。
</details>

---

**Q10.** Job 的 `activeDeadlineSeconds` 字段設置為 15 表示什麼？

A) Job 最多重試 15 次
B) Job 運行 15 秒後開始執行
C) Pod 運行的超時時間為 15 秒
D) Job 的總運行時間為 15 秒

<details>
<summary>答案</summary>
<b>C) Pod 運行的超時時間為 15 秒</b>

`activeDeadlineSeconds` 用來設置 Pod 運行的超時時間。如果 Pod 運行超過這個時間，Kubernetes 會終止該 Pod，防止任務無限期運行。
</details>

---

## 第二部分: 是非題 (每題 3 分，共 15 分)

**Q11.** Pod 是 Kubernetes 調度的最小單位，但為了保持它的獨立性，不應該向它添加多餘的功能。_(是/非)_

<details>
<summary>答案</summary>
<b>是</b>

這正是 Kubernetes 設計理念的核心。Pod 專注於容器管理，保持「單一職責」，其他功能由其他對象（如 Job、CronJob）來實現，通過「組合」的方式協作。
</details>

---

**Q12.** Job 和 CronJob 都屬於「在線業務」類型的應用。_(是/非)_

<details>
<summary>答案</summary>
<b>非</b>

Job 和 CronJob 都屬於「離線業務」，它們必定會退出，不會無期限地運行下去。「在線業務」是指像 Nginx、Node.js、MySQL、Redis 等長時間運行的應用。
</details>

---

**Q13.** 在 Job 的 YAML 定義中，`backoffLimit` 字段位於 `template` 字段下，用來控制 Pod 的重啟次數。_(是/非)_

<details>
<summary>答案</summary>
<b>非</b>

`backoffLimit` 字段不在 `template` 字段下，而是在 Job 的 `spec` 字段下。它設置的是 Pod 的失敗重試次數，屬於 Job 級別的控制字段，用來控制模板裡的 Pod 對象。
</details>

---

**Q14.** CronJob 的 `schedule` 字段使用標準的 Cron 語法，指定分鐘、小時、天、月、週。_(是/非)_

<details>
<summary>答案</summary>
<b>是</b>

CronJob 的 `schedule` 字段使用標準的 Cron 語法，格式與 Linux 上的 crontab 一樣，包含五個字段：分鐘、小時、天、月、週。例如 `'*/1 * * * *'` 表示每分鐘運行一次。
</details>

---

**Q15.** Job 和 CronJob 的用法完全不同，沒有任何相似之處。_(是/非)_

<details>
<summary>答案</summary>
<b>非</b>

Job 和 CronJob 都屬於離線業務，它們的用法幾乎是一樣的。主要區別在於 CronJob 多了一個 `schedule` 字段用於定時規則，以及使用 `jobTemplate` 來定義 Job 模板。它們都使用 `kubectl apply` 創建，都用 `kubectl get` 查看狀態。
</details>

---

## 第三部分: 填空題 (每題 4 分，共 20 分)

**Q16.** Kubernetes 使用 **\_\_\_\_** API，把集群中的各種業務都抽象為 HTTP 資源對象，在這個層次之上，可以使用 **\_\_\_\_** 的方式來考慮問題。

<details>
<summary>答案</summary>
<b>RESTful，面向對象</b>

Kubernetes 使用 RESTful API，把集群中的各種業務都抽象為 HTTP 資源對象，這樣就可以使用面向對象的方式來考慮問題，應用「單一職責」和「組合優於繼承」等設計原則。
</details>

---

**Q17.** 在線業務類型的應用（如 Nginx、MySQL）一旦運行起來基本不會停止，這稱為 **\_\_\_\_**；而離線業務（如日志分析、數據建模）必定會退出，這稱為 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>永遠在線，臨時任務/定時任務</b>

在線業務（Online Business）如 Nginx、MySQL 等長時間運行，稱為「永遠在線」。離線業務（Offline Business）分為兩種：「臨時任務」（對應 Job）和「定時任務」（對應 CronJob），它們必定會退出。
</details>

---

**Q18.** Job 的 YAML 中，**\_\_\_\_** 字段用來設置 Pod 運行的超時時間，**\_\_\_\_** 字段用來設置 Pod 的失敗重試次數。

<details>
<summary>答案</summary>
<b>activeDeadlineSeconds，backoffLimit</b>

`activeDeadlineSeconds` 設置 Pod 運行的超時時間，`backoffLimit` 設置 Pod 的失敗重試次數。這兩個字段都屬於 Job 級別，用來控制離線作業的執行。
</details>

---

**Q19.** CronJob 的 YAML 定義中，**\_\_\_\_** 字段定義了 Job 模板，**\_\_\_\_** 字段定義了定時運行的規則。

<details>
<summary>答案</summary>
<b>jobTemplate，schedule</b>

CronJob 的關鍵字段是 `spec.jobTemplate` 和 `spec.schedule`。`jobTemplate` 定義了 Job 對象的模板，`schedule` 定義了定時運行的規則（使用標準 Cron 語法）。
</details>

---

**Q20.** Kubernetes 的控制鏈是：CronJob 使用 **\_\_\_\_** 控制 Job，Job 使用 **\_\_\_\_** 控制 Pod，Pod 再定義參數控制容器。

<details>
<summary>答案</summary>
<b>定時規則，並發數量</b>

控制鏈體現了層層遞進的關係：CronJob 使用定時規則控制 Job，Job 使用並發數量控制 Pod，Pod 再定義參數控制容器，容器再隔離控制進程，進程最終實現業務功能。
</details>

---

## 第四部分: 簡答題 (每題 7 分，共 21 分)

**Q21.** 請解釋 Kubernetes 為什麼要採用「組合優於繼承」的設計原則，以及這種設計帶來的好處。

<details>
<summary>答案</summary>
Kubernetes 採用「組合優於繼承」的設計原則，是為了讓對象保持鬆耦合和靈活性。

<b>設計理念：</b>
- 對象應該在運行時產生聯繫，而不是通過硬編碼固定對象關係
- 每個對象保持獨立性，專注於自己的核心職責
- 通過「組合」方式將對象嵌套在一起，形成更強大的功能

<b>帶來的好處：</b>
1. <b>職責清晰：</b> 每個對象只關注自己的業務領域，不「缺位」也不「越位」
2. <b>易於維護：</b> 對象之間鬆耦合，修改一個對象不會影響其他對象
3. <b>可重用性高：</b> 對象可以靈活組合，Pod 可以被 Job、CronJob、Deployment 等多種對象使用
4. <b>擴展性強：</b> 新增功能時不需要修改現有對象，只需定義新對象並組合
5. <b>降低複雜度：</b> 避免了深度繼承帶來的複雜性和耦合問題

例如，Job 組合了 Pod，CronJob 又組合了 Job，每層各司其職，既分工又協作。
</details>

---

**Q22.** 請說明 Job 和 CronJob 的具體應用場景，並舉例說明它們能夠解決什麼問題。

<details>
<summary>答案</summary>
<b>Job 的應用場場景（臨時任務）：</b>
- <b>數據處理：</b> 一次性數據導入、數據遷移、歷史全量數據導入
- <b>系統維護：</b> 數據庫備份與恢復、安全檢查、系統巡檢
- <b>計算任務：</b> 模型訓練、數據建模、視頻轉碼
- <b>初始化任務：</b> 服務設置、文件構建、環境配置

<b>CronJob 的應用場景（定時任務）：</b>
- <b>定期備份：</b> 每日/每週數據庫自動備份
- <b>數據同步：</b> 每天定時增量數據同步、定期數據清理
- <b>監控檢查：</b> 定期系統健康檢查、安全掃描、證書更新
- <b>週期性運維：</b> 日誌分析、報表生成、消息推送

<b>能夠解決的問題：</b>
1. <b>任務管理：</b> 自動處理執行超時、失敗重試、並發控制
2. <b>狀態追蹤：</b> 記錄任務執行結果，方便監控與排查
3. <b>資源優化：</b> 控制並發數量，避免過多佔用集群資源
4. <b>自動化運維：</b> 無需人工干預，Kubernetes 自動調度執行
5. <b>聲明式配置：</b> 通過 YAML 描述任務需求，簡單直觀
</details>

---

**Q23.** 請解釋 Job 的 YAML 定義中，為什麼 Pod 模板不需要 `apiVersion`、`kind`、`metadata` 等「頭字段」，以及這種設計的優勢。

<details>
<summary>答案</summary>
<b>不需要「頭字段」的原因：</b>

Pod 模板在 Job 的 `spec.template` 字段中，受 Job 的管理和控制，不直接和 apiserver 打交道。它只是 Job 對象的一部分，用來定義 Pod 的配置信息，而不是一個獨立的 API 對象。

<b>設計優勢：</b>

1. <b>簡化配置：</b> 避免重複定義，減少配置文件的複雜度和出錯機會
2. <b>清晰的層次結構：</b> 體現了對象的組合關係，Job 包含 Pod 模板，職責明確
3. <b>統一管理：</b> Job 統一管理 Pod 的生命周期，包括命名、調度、重試等
4. <b>自動命名：</b> Pod 名字由 Job 自動生成（Job 名字 + 隨機字符串），避免命名衝突
5. <b>靈活控制：</b> Job 級別的字段（如 `activeDeadlineSeconds`、`backoffLimit`）可以統一控制所有從模板生成的 Pod

<b>對比：</b>
- 獨立 Pod：需要完整的 `apiVersion`、`kind`、`metadata`，直接與 apiserver 交互
- Job 中的 Pod 模板：只需要 `spec` 定義容器相關信息，由 Job 代為管理

這種「無頭」設計體現了 Kubernetes 對象組合的精髓，讓配置更簡潔，管理更統一。
</details>

---

## 第五部分: 配對題 (共 14 分)

**Q24.** 請將以下 Job/CronJob 相關字段與其功能進行配對：

| 字段                     | 功能描述                                  |
| ------------------------ | ----------------------------------------- |
| 1. activeDeadlineSeconds | A. 設置 Pod 的失敗重試次數                |
| 2. backoffLimit          | B. 定義任務周期運行的規則（Cron 語法）    |
| 3. completions           | C. 設置 Pod 運行的超時時間                |
| 4. parallelism           | D. 定義 Job 模板                          |
| 5. schedule              | E. Job 完成需要運行的 Pod 數量            |
| 6. jobTemplate           | F. 允許並發運行的 Pod 數量                |

<details>
<summary>答案</summary>
1-C, 2-A, 3-E, 4-F, 5-B, 6-D

<b>詳細說明：</b>
- <b>activeDeadlineSeconds:</b> 設置 Pod 運行的超時時間，防止任務無限期運行
- <b>backoffLimit:</b> 設置 Pod 的失敗重試次數，超過後標記 Job 為失敗
- <b>completions:</b> Job 完成需要運行的 Pod 總數，預設為 1
- <b>parallelism:</b> 允許並發運行的 Pod 數量，控制資源使用
- <b>schedule:</b> CronJob 專用字段，定義定時運行規則（Cron 語法）
- <b>jobTemplate:</b> CronJob 專用字段，定義 Job 對象的模板
</details>

---

**Q25.** 請將以下 Kubernetes 對象與其描述進行配對：

| 對象     | 描述                                                |
| -------- | --------------------------------------------------- |
| 1. Pod   | A. 定時任務對象，按 Cron 規則周期運行               |
| 2. Job   | B. 最小調度單元，管理容器                            |
| 3. CronJob | C. 臨時任務對象，運行完成後退出                    |

<details>
<summary>答案</summary>
1-B, 2-C, 3-A

<b>詳細說明：</b>
- <b>Pod:</b> Kubernetes 的最小調度單元，專注於容器管理，不應添加多餘功能
- <b>Job:</b> 處理臨時任務（一次性任務），確保任務完成後退出，支持失敗重試、超時控制等
- <b>CronJob:</b> 處理定時任務，按照 Cron 語法定時創建 Job，周期性執行任務
</details>

---

## 答案總覽

### 選擇題 (1-10)
1. B
2. B
3. C
4. B
5. B
6. B
7. B
8. B
9. C
10. C

### 是非題 (11-15)
11. 是
12. 非
13. 非
14. 是
15. 非

### 填空題 (16-20)
16. RESTful，面向對象
17. 永遠在線，臨時任務/定時任務
18. activeDeadlineSeconds，backoffLimit
19. jobTemplate，schedule
20. 定時規則，並發數量

### 簡答題 (21-23)
請參考各題詳細解答。

### 配對題 (24-25)
24. 1-C, 2-A, 3-E, 4-F, 5-B, 6-D
25. 1-B, 2-C, 3-A

---

**生成來源:** 13｜Job/CronJob：為什麼不直接用 Pod 來處理業務？
**課程:** Kubernetes 入門實戰課
**講師:** Chrono

---

<b>學習建議：</b>
- 理解 Kubernetes 的面向對象設計思想（單一職責、組合優於繼承）
- 掌握 Job 和 CronJob 的使用場景和區別
- 熟練使用 YAML 定義 Job 和 CronJob 對象
- 理解控制鏈的概念：CronJob → Job → Pod → Container → Process
- 實踐操作：嘗試創建自己的 Job 和 CronJob 對象