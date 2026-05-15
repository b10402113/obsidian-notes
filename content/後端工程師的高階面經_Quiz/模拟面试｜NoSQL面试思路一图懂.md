# 模擬面試｜NoSQL面試思路一圖懂 Quiz

**Subject:** 後端工程師的高階面經 - NoSQL面試模擬
**Difficulty:** Mixed (Easy/Medium/Hard)
**Questions:** 15
**Time:** 25-30 minutes

---

## Section A: Multiple Choice (24 points)

**Q1.** [Easy] Elasticsearch 的節點可以扮演多個角色嗎？

A)不可以，一個節點只能扮演一個角色
B) 可以，一個節點可以同時扮演多個角色
C) 只能在配置文件中預設一個角色
D) 角色是由集群自動分配的，無法手動設置

<details>
<summary>Answer</summary>
<b>B) 可以，一個節點可以同時扮演多個角色</b>

面試題明確問到「一個節點可以扮演多個角色嗎？」Elasticsearch 的節點可以同時扮演多種角色，如數據節點、主節點、協調節點等，但在實踐中需要根據業務需求合理安排不同節點扮演的角色。
</details>

**Q2.** [Easy] MongoDB 用於記錄操作日誌以實現複製機制的是什麼？

A) Transaction log
B) Oplog
C) Redo log
D) Binary log

<details>
<summary>Answer</summary>
<b>B) Oplog</b>

MongoDB 使用 oplog (operations log) 來記錄主節點上的所有寫入操作，從節點通過讀取 oplog 來同步數據。面試題中問到「為什麼 MongoDB 的 oplog 總是很多？」這是理解 MongoDB 複製機制的關鍵概念。
</details>

**Q3.** [Medium] Elasticsearch 引入分片的主要目的是什麼？

A) 提高查詢的準確性
B) 解決單機存儲容量限制和提高查詢性能
C) 簡化索引結構
D) 減少網絡傳輸開銷

<details>
<summary>Answer</summary>
<b>B) 解決單機存儲容量限制和提高查詢性能</b>

面試題問到「Elasticsearch 為什麼引入分片？為了解決什麼問題？」分片可以將大索引拆分到多個節點上，解決單機存儲容量限制，同時支持並行查詢以提高性能，是 Elasticsearch 高可用和高性能的基礎設計。
</details>

**Q4.** [Medium] MongoDB 的 ESR 規則是指什麼？

A) Equal, Sort, Range - 索引字段排序規則
B) Exact, Sort, Range - 索引設計優化規則
C) Equal, Search, Read - 查詢優化規則
D) Efficient, Speed, Reliable - 性能優化規則

<details>
<summary>Answer</summary>
<b>B) Exact, Sort, Range - 索引設計優化規則</b>

ESR 規則是 MongoDB 索引設計的重要原則：Exact（精確匹配字段優先）、Sort（排序字段其次）、Range（範圍查詢字段最後）。遵守此規則可以優化查詢性能，面試題問到「為什麼要遵守 ESR 視則？不遵守行不行？」
</details>

**Q5.** [Medium] 在 Elasticsearch 中，投票節點可以被選為主節點嗎？

A) 可以，投票節點就是候選主節點
B) 不可以，投票節點只參與投票不參與選舉
C) 只在緊急情況下可以
D) 投票節點的概念不存在

<details>
<summary>Answer</summary>
<b>B) 不可以，投票節點只參與投票不參與選舉</b>

面試題明確問到「投票節點可以被選為主節點嗎？為什麼要引入投票節點？」投票節點（voting node）只參與投票過程，本身不會被選為主節點。引入投票節點是為了在候選主節點數量不足時增加投票權重，保證選舉過程的可靠性。
</details>

**Q6.** [Hard] Elasticsearch 的 Translog 可以保證數據一定不丢失嗎？

A) 可以，Translog 確保數據絕對不丢失
B) 不可以，Translog 只能減少數據丢失風險，不能完全保證
C) Translog 不涉及數據持久化問題
D) 只有在同步模式下才能保證不丢失

<details>
<summary>Answer</summary>
<b>B) 不可以，Translog 只能減少數據丢失風險，不能完全保證</b>

面試題問到「Elasticsearch 的 Translog 是拿來干什麼的？它可以保證數據一定不丢失嗎？」Translog 記錄未持久化的操作，類似於數據庫的 redo log，但根據配置的同步策略（sync/async），在極端情況下（如硬件故障）仍可能丢失數據，不能完全保證數據不丢失。
</details>

**Q7.** [Hard] MongoDB 在主從選舉時，如何讓系統優先選擇同機房的從節點？

A) 通過配置優先級（priority）參數
B) 自動根據網絡延遲判斷
C) 使用標籤（tags）和優先級組合配置
D) MongoDB 不支持此功能

<details>
<summary>Answer</summary>
<b>C) 使用標籤（tags）和優先級組合配置</b>

面試題問到「怎麼樣可以讓 MongoDB 在主從選舉的時候優先選擇同機房的從節點？」可以通過為節點設置標籤（如機房位置）並配合優先級配置，讓同機房的節點在選舉時具有更高的優先級，這是在多機房部署中的重要策略。
</details>

---

## Section B: True/False (12 points)

**Q8.** [Easy] Elasticsearch 是實時搜索引擎，所有寫入數據可以立即被查詢到。_(True/False)_

<details>
<summary>Answer</summary>
<b>False</b>

面試題明確問到「Elasticsearch 是實時的嗎？」實際上 Elasticsearch 是近實時（near real-time）搜索引擎，寫入的數據需要經過 refresh 間隔（默1秒）才能被搜索到，不是完全實時的。
</details>

**Q9.** [Medium] MongoDB 的塊（chunk）遷移是自動觸發的，不需要人工干预。_(True/False)_

<details>
<summary>Answer</summary>
<b>True</b>

MongoDB 的分片集群會自動監測塊的大小和分布，當某些塊超過限制或分布不均衡時會自動觸發塊遷移，實現負載均衡（再平衡）。面試題問到「什麼情況下會觸發塊遷移？怎麼遷移？」
</details>

**Q10.** [Medium] Elasticsearch 在合并段（segment merge）時會影響已有的查詢。_(True/False)_

<details>
<summary>Answer</summary>
<b>False</b>

面試題問到「Elasticsearch 在合并段的時候，會影響到已有的查詢嗎？」段合并過程中，舊段仍然可以查詢，新段準備好後才會替換舊段。查詢會根據 commit point 來確定使用哪些段，不會受到合并過程的影響。
</details>

**Q11.** [Hard] MongoDB 的配置服務器崩潰後，集群仍然可以正常進行數據讀寫操作。_(True/False)_

<details>
<summary>Answer</summary>
<b>False</b>

面試題問到「有沒有遇過配置服務器崩潰的問題？怎麼提高配置服務器的可用性？」配置服務器存儲集群元數據，如果崩潰會影響分片路由和集群管理，嚴重情況下會導致集群無法正常工作。因此需要部署多個配置服務器組成副本集來保證高可用。
</details>

---

## Section C: Fill-in-the-Blank (12 points)

**Q12.** [Easy] Elasticsearch 的寫入流程中，數據首先寫入 **\_\_\_\_\_\_**，然後才寫入磁盤段。

<details>
<summary>Answer</summary>
<b>Translog</b>

面試題問到「當一個寫入請求發送到 Elasticsearch 之後，發生了什麼？」數據首先寫入 Translog 確保數據安全，然後寫入內存緩衝區，定期刷新到磁盤段。Translog 是類似 redo log 的預寫日誌。
</details>

**Q13.** [Medium] MongoDB 的分片鍵（shard key）選擇需要考慮數據分布的**\_\_\_\_\_\_**和查詢模式。

<details>
<summary>Answer</summary>
<b>均衡性</b>

分片鍵決定數據如何分布在不同分片上，選擇需要考慮數據分布的均衡性以避免熱點分片，還要考慮查詢模式以優化查詢效率。這是 MongoDB 分片設計的核心問題。
</details>

**Q14.** [Medium] Elasticsearch 的協調節點主要負接收請求、**\_\_\_\_\_\_**和返回結果。

<details>
<summary>Answer</summary>
<b>路由請求/分發請求</b>

面試題問到「你知道什麼是協調節點嗎？它的作用是什麼？」協調節點負責接收客戶端請求，將請求路由到相應的數據節點，聚合結果並返回給客戶端。它不存儲數據，只負協調查詢。
</details>

**Q15.** [Hard] MongoDB 控制寫入語義時，可以選擇的確認級別包括 w:1、**\_\_\_\_\_\_** 和 w:majority 等。

<details>
<summary>Answer</summary>
<b>w:0</b>

MongoDB 的寫入語義（write concern）包括：w:0（不等待確認）、w:1（等待主節點確認）、w:majority（等待大多數節點確認）等。面試題問到「怎麼控制 MongoDB 的寫入語義？你用的是什麼語義？」需要根據業務需求選擇。
</details>

---

## Section D: Short Answer (24 points)

**Q16.** [Medium] 說明 Elasticsearch 如何保證高可用，需要從節點角色設計、分片策略和故障恢復等方面回答。

<details>
<summary>Answer</summary>
<b>Elasticsearch 高可用保證策略：</b>

<b>1. 瀑點角色設計：</b>
- 主節點（Master）：負責集群管理，可設置多個候選主節點
- 數據節點（Data）：存儲數據和執行查詢，可部署多個實例
- 協調節點（Coordinating）：路由請求，可獨立部署減輕數據節點負擔
- 投票節點：參與選舉但不會被選為主節點

<b>2. 分片策略：</b>
- 主分片和副本分片：每個主分片至少配置1個副本
- 分片分布：自動分布在不同節點上
- 故障時副本分片可升級為主分片

<b>3. 故障恢復：</b>
- 點故障：副本分片自動升級為主分片
- 主節點選舉：候選主節點通過投票選出新主節點
- 數據恢復：從副本重新複製數據到新節點
- Translog 保證：減少數據丢失風險

<b>4. 部署建議：</b>
- 至3個候選主節點避免腦裂
- 數據節點根據數據量和查詢壓力部署
- 跨機房部署考慮網絡延遲
</details>

**Q17.** [Hard] MongoDB 的分片集群包含哪些組件？請說明每個組件的作用，以及配置服務器崩潰時會發生什麼問題。

<details>
<summary>Answer</summary>
<b>MongoDB 分片集群組件：</b>

<b>1. mongos（路由服務器）：</b>
- 接收客戶端請求
- 根據分片鍵路由到對應分片
- 聚合多個分片的查詢結果
- 不存儲數據，可部署多個實例

<b>2. shard（分片服務器）：</b>
- 存儲實際數據
- 每個分片是獨立的副本集
- 負責處理路由過來的請求

<b>3. config server（配置服務器）：</b>
- 存集群元數據（分片映射、塊範圍等）
- 必須是副本集保證高可用
- mongos 通過配置服務器獲取路由信息

<b>配置服務器崩潰的影響：</b>
- 集群元數據無法更新
- 新的塊遷移無法進行
- mongos 無法獲取分片路由信息
- 可能導致寫入操作失敗（無法確定目標分片）
- 查詢可能無法正確路由
- 嚴重時整個集群可能不可用

<b>解決方案：</b>
- 配置服務器必須部署為副本集（至少3節點）
- 定期備份配置數據
- 監控配置服務器健康狀態
</details>

---

## Answer Key Summary

| Question | Type | Difficulty | Answer |
| -------- | ---- | ---------- | ------ |
| Q1 | MCQ | Easy | B) 可以，一個節點可以同時扮演多個角色 |
| Q2 | MCQ | Easy | B) Oplog |
| Q3 | MCQ | Medium | B) 解決單機存儲容量限制和提高查詢性能 |
| Q4 | MCQ | Medium | B) Exact, Sort, Range - 紹引設計優化規則 |
| Q5 | MCQ | Medium | B) 不可以，投票節點只參與投票不參與選舉 |
| Q6 | MCQ | Hard | B) 不可以，Translog 只能減少數據丢失風險，不能完全保證 |
| Q7 | MCQ | Hard | C) 使用標籤（tags）和優先級組合配置 |
| Q8 | T/F | Easy | False |
| Q9 | T/F | Medium | True |
| Q10 | T/F | Medium | False |
| Q11 | T/F | Hard | False |
| Q12 | Fill-in | Easy | Translog |
| Q13 | Fill-in | Medium | 均衡性 |
| Q14 | Fill-in | Medium | 路由請求/分發請求 |
| Q15 | Fill-in | Hard | w:0 |
| Q16 | Short | Medium | 需要從節點角色、分片策略、故障恢復等方面說明 |
| Q17 | Short | Hard | 包含mongos、shard、config server三個組件，配置服務器崩潰會嚴重影響集群 |

---

_Generated from: 模拟面试｜NoSQL面试思路一图懂.md_