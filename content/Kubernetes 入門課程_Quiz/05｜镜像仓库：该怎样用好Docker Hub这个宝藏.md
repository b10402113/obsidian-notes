# 鏡像倉庫測驗：如何用好 Docker Hub

**科目：** Kubernetes 入門課程
**難度：** 混合（基礎、中等、困難）
**題目數量：** 15 題
**建議時間：** 20-25 分鐘

---

## 第一部分：選擇題（共 5 題，每題 4 分）

**Q1.** 在 Docker 的官方架構圖中，Registry 位於哪個位置？它的主要功能是什麼？

A) 位於左側，主要功能是構建鏡像
B) 位於右側，主要功能是存儲、分發和管理鏡像
C) 位於中間，主要功能是協調容器運行
D) 位於頂部，主要功能是監控容器狀態

<details>
<summary>答案</summary>
<b>B) 位於右側，主要功能是存儲、分發和管理鏡像</b>

Registry（鏡像倉庫）在架構圖中位於右側區域，是一個綜合的鏡像管理服務站點。它不僅提供基本的鏡像上傳和下載功能，還包括查詢、刪除等完整的鏡像管理服務。
</details>

---

**Q2.** Docker Hub 上有三種類型的鏡像，以下哪一種不屬於這三種類型？

A) 官方鏡像（Official image）
B) 認證鏡像（Verified publisher）
C) 社區鏡像（Community image）
D) 非官方鏡像（Unofficial image）

<details>
<summary>答案</summary>
<b>C) 社區鏡像（Community image）</b>

Docker Hub 上的鏡像主要分為三種類型：
1. 官方鏡像：Docker 公司官方提供的高質量鏡像
2. 認證鏡像：由知名公司發布，帶有 "Verified publisher" 標記
3. 非官方鏡像：其他個人或公司上傳的鏡像

"社區鏡像"不是 Docker Hub 的官方分類。
</details>

---

**Q3.** 以下關於 Docker 官方鏡像的描述，哪一項是錯誤的？

A) 都經過嚴格的漏洞掃描和安全檢測
B) 支持 x86_64、arm64 等多種硬件架構
C) 目前約有 100 多個官方鏡像
D) 所有官方鏡像都必須付費才能使用

<details>
<summary>答案</summary>
<b>D) 所有官方鏡像都必須付費才能使用</b>

Docker 官方鏡像是免費提供給公眾使用的，不需要付費。官方鏡像具有高質量、嚴格的安全檢測、多架構支持等特點，是構建鏡像的首選。
</details>

---

**Q4.** 在選擇 Docker Hub 上的鏡像時，以下哪項不是重要的參考依據？

A) 下載量（Download count）
B) 星數（Stars）
C) 更新歷史
D) 鏡像名稱的長度

<details>
<summary>答案</summary>
<b>D) 鏡像名稱的長度</b>

選擇鏡像時應該綜合評估：是否為官方認證、下載量（通常百萬級別較好）、星數、以及更新歷史。鏡像名稱的長度不是質量判斷的標準。下載量是最重要的參考依據。
</details>

---

**Q5.** 在離線環境中，如果無法連接外網，以下哪種方法無法解決鏡像管理問題？

A) 使用 docker save 和 docker load 命令導出導入鏡像
B) 搭建私有 Registry 服務
C) 使用 Docker Hub 的離線模式功能
D) 使用壓縮包在內網傳輸鏡像

<details>
<summary>答案</summary>
<b>C) 使用 Docker Hub 的離線模式功能</b>

Docker Hub 本身是在線服務，不支持離線模式。在離線環境中，可以：
1. 搭建私有 Registry 服務（如 Docker Registry、CNCF Harbor）
2. 使用 docker save 將鏡像導出為壓縮包，通過 U 盤或 FTP 傳輸
3. 使用 docker load 從壓縮包恢復鏡像
</details>

---

## 第二部分：是非題（共 3 題，每題 3 分）

**Q6.** 在生產環境中，直接使用鏡像的 "latest" 標籤是一種最佳實踐。_(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b>

在生產環境中使用 "latest" 標籤是不負責任的做法。雖然簡單方便，但會導致版本不可控。應該明確指定具體的版本號和操作系統標籤，以確保環境的一致性和可重現性。
</details>

---

**Q7.** Docker Hub 是 Docker 公司搭建的官方 Registry 服務，成立於 2014 年 6 月。_(對/錯)_

<details>
<summary>答案</summary>
<b>對</b>

Docker Hub 成立於 2014 年 6 月，與 Docker 1.0 同時發布。它是世界上最大的鏡像倉庫，和 GitHub 一樣，幾乎成為了容器世界的基礎設施。
</details>

---

**Q8.** 所有在 Docker Hub 上的 "Verified publisher" 認證鏡像都是免費獲得該標記的。_(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b>

成為 "Verified publisher" 需要向 Docker 公司付費。這也是為什麼有些公司雖然有官方發布的鏡像，但並沒有認證標記，因為不想花這筆費用。這類鏡像被稱為"半官方"鏡像。
</details>

---

## 第三部分：填充題（共 3 題，每題 5 分）

**Q9.** Docker Hub 上的鏡像命名規則為 **\_\_\_\_** 的形式，例如 bitnami/nginx、ubuntu/nginx。下載非官方鏡像時必須包含此部分，否則會默認使用官方鏡像。

<details>
<summary>答案</summary>
<b>用戶名/應用名</b>
</details>

---

**Q10.** 鏡像標籤的格式通常由 **\_\_\_\_** 和 **\_\_\_\_** 組成。例如 nginx:1.21.6-alpine 中，1.21.6 表示版本號，alpine 表示操作系統。

<details>
<summary>答案</summary>
<b>應用版本號、操作系統</b>
</details>

---

**Q11.** 將自己的鏡像上傳到 Docker Hub 需要 4 個步驟：註冊賬號、使用 **\_\_\_\_** 命令登錄、使用 **\_\_\_\_** 命令為鏡像改名、使用 **\_\_\_\_** 命令推送鏡像。

<details>
<summary>答案</summary>
<b>docker login、docker tag、docker push</b>
</details>

---

## 第四部分：簡答題（共 3 題，每題 10 分）

**Q12.** 解釋 Docker Hub 上鏡像標籤中 "slim" 和 "fat" 的含義及其適用場景。

<details>
<summary>答案</summary>
<b>參考答案：</b>

**Slim 鏡像：**
- 經過精簡的鏡像，體積較小
- 運行效率高，啟動速度快
- 適合生產環境部署，節省存儲和網絡傳輸成本

**Fat 鏡像：**
- 包含較多輔助工具的鏡像，體積較大
- 功能完整，便於調試和開發
- 適合開發和測試環境使用

選擇建議：生產環境優先選擇 slim 鏡像以提高效率，開發環境可選擇 fat 鏡像以獲得更多工具支持。
</details>

---

**Q13.** 為什麼許多公司（如 Bitnami、Rancher）在 Docker 官方已經提供 Nginx、Redis 等應用鏡像的情況下，還要發布自己打包的鏡像？請列出至少三個原因。

<details>
<summary>答案</summary>
<b>參考答案：</b>

**主要原因包括：**

1. **定制化需求**：公司在官方鏡像基礎上添加自己需要的工具、配置或環境，滿足特定業務需求

2. **標準化環境**：公司可以建立統一的基礎鏡像標準（如 Bitnami 使用 minideb），確保所有應用運行在一致的環境中

3. **品牌推廣**：通過發布鏡像提高公司在開源社區的知名度和影響力

4. **技術棧整合**：將公司自己的產品或工具預先打包進鏡像，方便用戶直接使用

5. **安全與合規**：企業可能有自己的安全標準和合規要求，需要定制鏡像以滿足內部規範

6. **版本控制**：企業可能需要對特定版本進行長期維護和支持，而不依賴官方更新節奏
</details>

---

**Q14.** 請解釋什麼是"半官方"鏡像，以及在使用這類鏡像時需要注意什麼。

<details>
<summary>答案</summary>
<b>參考答案：</b>

**半官方鏡像定義：**
由知名公司或組織在其 Docker Hub 官方賬號發布，但未經 Docker 公司正式認證（沒有 "Verified publisher" 標記）的鏡像。

**產生原因：**
成為認證發布商需要向 Docker 公司付費，許多公司為了節省成本，選擇只開設公司賬號而不申請認證。

**使用注意事項：**
1. **驗證來源**：確認鏡像確實來自官方賬號，避免冒名頂替
2. **檢查更新頻率**：查看鏡像的更新歷史，確保維護及時
3. **查看下載量和星數**：下載量大、星數多的鏡像通常更可靠
4. **安全掃描**：使用前進行安全掃描，確保沒有漏洞
5. **官方文檔**：查看是否有完善的文檔和使用說明

總體來說，半官方鏡像質量通常較可靠，但相比官方認證鏡像，使用時需要更謹慎地評估。
</details>

---

## 第五部分：配對題（共 1 題，10 分）

**Q15.** 請將以下鏡像標籤與其含義進行配對：

| 標籤 | 含義 |
| ---- | ---- |
| 1. redis:7.0-rc-bullseye | A. 版本號 17，基於精簡的 Debian 10 |
| 2. node:17-buster-slim | B. 版本號 1.21.6，基於最新的 Alpine |
| 3. nginx:1.21.6-alpine | C. 版本號 7.0 候選版，基於 Debian 11 |

<details>
<summary>答案</summary>
<b>1-C, 2-A, 3-B</b>

**解析：**
1. redis:7.0-rc-bullseye - "rc" 表示候選版本（release candidate），"bullseye" 是 Debian 11 的代號
2. node:17-buster-slim - "buster" 是 Debian 10 的代號，"slim" 表示精簡版
3. nginx:1.21.6-alpine - "alpine" 表示基於 Alpine Linux，版本號為 1.21.6
</details>

---

## 答案總覽

**選擇題：**
1. B
2. C
3. D
4. D
5. C

**是非題：**
6. 錯
7. 對
8. 錯

**填充題：**
9. 用戶名/應用名
10. 應用版本號、操作系統
11. docker login、docker tag、docker push

**簡答題：**
12-14. 見各題詳細答案

**配對題：**
15. 1-C, 2-A, 3-B

---

**生成來源：** 《Kubernetes 入門實戰課》第 05 課 - 鏡像倉庫：該怎樣用好 Docker Hub 這個寶藏