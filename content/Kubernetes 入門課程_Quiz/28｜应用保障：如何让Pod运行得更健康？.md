# Pod 應用保障測驗

**科目:** Kubernetes 入門課程
**難度:** 中等
**題數:** 15 題
**建議時間:** 20 分鐘

---

## 第一部分：選擇題（每題 5 分，共 25 分）

**Q1.** Kubernetes 中 resources 欄位下的 requests 和 limits 有什麼區別？

A) requests 和 limits 完全相同，可以互換使用
B) requests 是容器運行時資源下限，limits 是容器運行時資源上限
C) requests 是容器運行時資源上限，limits 是容器運行時資源下限
D) requests 用於存儲資源，limits 用於 CPU 資源

<details>
<summary>答案</summary>
<b>B) requests 是容器運行時資源下限，limits 是容器運行時資源上限</b>

requests 表示容器申請的資源，Kubernetes 在創建 Pod 時必須分配這些資源，否則容器無法運行。limits 表示容器使用資源的上限，不能超過設定值，否則可能被強制停止運行。
</details>

---

**Q2.** 在 Kubernetes 中，CPU 資源單位 "m" 代表什麼意思？

A) 兆（million）
B) 毫（milli），表示千分之一
C) 分鐘（minute）
D) 記憶體（memory）

<details>
<summary>答案</summary>
<b>B) 毫（milli），表示千分之一</b>

Kubernetes 允許容器精細分割 CPU，CPU 的最小使用單位是 0.001，用 m 表示「毫」，例如 500m 就相當於 0.5 CPU。
</details>

---

**Q3.** 以下關於 Kubernetes 三種健康探針的執行順序，哪個描述是正確的？

A) Startup、Liveness、Readiness 三種探針同時並行執行
B) Startup 探針成功後，Liveness 和 Readiness 探針才會開始執行
C) Readiness 探針最先執行，成功後才執行 Startup 探針
D) Liveness 探針失敗後，才會啟動 Startup 探針

<details>
<summary>答案</summary>
<b>B) Startup 探針成功後，Liveness 和 Readiness 探針才會開始執行</b>

這三種探針是遞進關係：應用程序先啟動，進入 Startup 狀態，之後如果沒有異常就是 Liveness 存活狀態，最後到達 Readiness 狀態。Startup 探針失敗時，後面的 Liveness 和 Readiness 探針不會啟動。
</details>

---

**Q4.** 當 Readiness 探針檢測失敗時，Kubernetes 會採取什麼行動？

A) 重啟容器
B) 刪除 Pod
C) 將容器從 Service 的負載均衡集合中排除
D) 增加資源配額

<details>
<summary>答案</summary>
<b>C) 將容器從 Service 的負載均衡集合中排除</b>

Readiness 探針失敗時，Kubernetes 認為容器雖然在運行，但內部有錯誤，不能正常提供服務，會把容器從 Service 對象的負載均衡集合中排除，不會給它分配流量。
</details>

---

**Q5.** 以下哪種探針檢測方式最適合檢查 Web 應用是否就緒？

A) exec 執行 shell 命令
B) tcpSocket 連接端口
C) httpGet 發送 HTTP 請求
D) 直接檢查進程狀態

<details>
<summary>答案</summary>
<b>C) httpGet 發送 HTTP 請求</b>

對於 Web 應用，使用 httpGet 發送 HTTP 請求是最佳選擇，因為它可以檢查應用是否能正常響應 HTTP 請求，更準確地判斷應用是否就緒。課程範例中使用 httpGet 訪問 /ready 路徑來檢測 Nginx 的就緒狀態。
</details>

---

## 第二部分：是非題（每題 5 分，共 15 分）

**Q6.** 如果 Pod 不寫 resources 欄位，Kubernetes 會拒絕創建該 Pod。_(O/X)_

<details>
<summary>答案</summary>
<b>X (錯誤)</b>

如果 Pod 不寫 resources 欄位，Kubernetes 會認為 Pod 對運行資源「既沒有下限，也沒有上限」，可以把 Pod 調度到任意節點上，後續運行時也可以無限制使用 CPU 和記憶體。但在生產環境中這樣做很危險。
</details>

---

**Q7.** Liveness 探針失敗時，Kubernetes 會重啟容器。_(O/X)_

<details>
<summary>答案</summary>
<b>O (正確)</b>

當 Liveness 探針失敗時，Kubernetes 會認為容器發生了異常（如死鎖、死循環），會重啟容器。
</details>

---

**Q8.** Startup 探針適合用於檢查那些啟動很快的應用。_(O/X)_

<details>
<summary>答案</summary>
<b>X (錯誤)</b>

Startup 探針專門用來檢查應用是否已經啟動成功，適合那些有大量初始化工作要做、啟動很慢的應用。對於啟動很快的應用，不需要特別配置 Startup 探針。
</details>

---

## 第三部分：填充題（每題 8 分，共 16 分）

**Q9.** Kubernetes 中記憶體資源的表示方法使用 **\_\_\_\_**、**\_\_\_\_**、**\_\_\_\_** 來表示 KB、MB、GB。

<details>
<summary>答案</summary>
<b>Ki、Mi、Gi</b>

記憶體的寫法和磁碟容量一樣，使用 Ki、Mi、Gi 來表示 KB、MB、GB，例如 512Ki、100Mi、0.5Gi 等。
</details>

---

**Q10.** Kubernetes 定義了三種健康探針：**\_\_\_\_** 探針用於檢查應用是否已啟動，**\_\_\_\_** 探針用於檢查應用是否正常運行，**\_\_\_\_** 探針用於檢查應用是否可以接收流量。

<details>
<summary>答案</summary>
<b>Startup、Liveness、Readiness</b>

Startup 探針用來檢查應用是否已經啟動成功；Liveness 探針用來檢查應用是否正常運行，是否存在死鎖、死循環；Readiness 探針用來檢查應用是否可以接收流量，是否能夠對外提供服務。
</details>

---

## 第四部分：簡答題（每題 12 分，共 24 分）

**Q11.** 請說明 Liveness 探針和 Readiness 探針的區別，以及它們失敗時 Kubernetes 的不同處理方式。

<details>
<summary>答案</summary>

<b>Liveness 探針和 Readiness 探針的區別：</b>

**用途不同：**
- Liveness 探針：檢查應用是否正常運行，檢測是否存在死鎖、死循環等異常
- Readiness 探針：檢查應用是否可以接收流量，是否能夠對外提供服務

**失敗時的處理方式不同：**
- Liveness 探針失敗：Kubernetes 會認為容器發生了異常，會重啟容器
- Readiness 探針失敗：Kubernetes 會認為容器雖然在運行，但內部有錯誤，不能正常提供服務，會把容器從 Service 對象的負載均衡集合中排除，不會給它分配流量

**關鍵差異：**
Liveness 關注的是容器內部程序的健康狀態，失敗會導致容器重啟；而 Readiness 關注的是容器是否能對外提供服務，失敗只會暫時停止流量分配，不會重啟容器。
</details>

---

**Q12.** 請解釋 Kubernetes 中三種探針檢測方式（Shell、TCP Socket、HTTP GET）的優缺點及適用場景。

<details>
<summary>答案</summary>

<b>三種探針檢測方式的比較：</b>

**1. Shell (exec)**
- 優點：靈活性高，可以執行任意 Linux 命令進行複雜檢查
- 缺點：需要容器內有相應命令工具，輸出解析相對複雜
- 適用場景：檢查文件是否存在、進程狀態等需要執行命令的場景
- 範例：`exec: command: ["cat", "/var/run/nginx.pid"]`

**2. TCP Socket**
- 優點：簡單直接，只需檢查端口是否可連接
- 缺點：只能判斷端口是否開放，無法知道應用是否真正正常工作
- 適用場景：非 HTTP 應用，如數據庫服務、緩存服務等
- 範例：`tcpSocket: port: 80`

**3. HTTP GET**
- 優點：最貼近實際使用場景，能真實反映應用健康狀態
- 缺點：需要應用提供 HTTP 端點，配置相對複雜
- 適用場景：Web 應用、REST API 服務等 HTTP 應用
- 範例：`httpGet: path: /ready, port: 80`

**選擇建議：**
- Web 應用優先使用 HTTP GET，能更準確檢測應用狀態
- 非 HTTP 服務可使用 TCP Socket
- 需要複雜檢查邏輯時使用 Shell
</details>

---

## 第五部分：配對題（共 20 分）

**Q13.** 請將以下探針配置參數與其功能進行配對：

| 參數               | 功能說明                            |
| ------------------ | ----------------------------------- |
| 1. periodSeconds   | A. 探測動作的超時時間                |
| 2. timeoutSeconds  | B. 連續探測失敗幾次才認為真正異常     |
| 3. successThreshold| C. 執行探測動作的時間間隔            |
| 4. failureThreshold| D. 連續幾次探測成功才認為是正常       |

<details>
<summary>答案</summary>
<b>1-C, 2-A, 3-D, 4-B</b>

- periodSeconds：執行探測動作的時間間隔，默認是 10 秒探測一次
- timeoutSeconds：探測動作的超時時間，如果超時就認為探測失敗，默認是 1 秒
- successThreshold：連續幾次探測成功才認為是正常，對於 startupProbe 和 livenessProbe 來說它只能是 1
- failureThreshold：連續探測失敗幾次才認為是真正發生了異常，默認是 3 次
</details>

---

**Q14.** 請將以下資源配額數值與其實際含義進行配對：

| 資源配額    | 實際含義                  |
| ----------- | ------------------------- |
| 1. cpu: 10m | A. 要求 10 個 CPU         |
| 2. cpu: 10  | B. 要求 1% CPU 時間       |
| 3. cpu: 500m| C. 要求 0.5 CPU 時間      |
| 4. cpu: 1000m| D. 要求 1 個完整 CPU     |

<details>
<summary>答案</summary>
<b>1-B, 2-A, 3-C, 4-D</b>

- cpu: 10m - 10 毫核，相當於 0.01 CPU，即 1% CPU 時間
- cpu: 10 - 10 個完整 CPU
- cpu: 500m - 500 毫核，相當於 0.5 CPU
- cpu: 1000m - 1000 毫核，相當於 1 個完整 CPU
</details>

---

**Q15.** 請將探針類型與其對應的 Kubernetes 處理方式進行配對：

| 探針類型        | Kubernetes 處理方式                          |
| --------------- | --------------------------------------------- |
| 1. Startup 探針 | A. 從 Service 負載均衡中排除容器             |
| 2. Liveness 探針| B. 反復重啟容器，不執行後續探針              |
| 3. Readiness 探針| C. 重啟容器                                  |

<details>
<summary>答案</summary>
<b>1-B, 2-C, 3-A</b>

- Startup 探針失敗：Kubernetes 認為容器沒有正常啟動，會嘗試反覆重啟，Liveness 和 Readiness 探針不會啟動
- Liveness 探針失敗：Kubernetes 認為容器發生異常，會重啟容器
- Readiness 探針失敗：Kubernetes 認為容器雖然運行但不能正常提供服務，會從 Service 的負載均衡集合中排除
</details>

---

## 參考來源

本測驗基於課程材料：《Kubernetes 入門實戰課》第 28 講「應用保障：如何讓 Pod 運行得更健康？」