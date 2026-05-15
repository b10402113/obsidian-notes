# YAML：Kubernetes 世界里的通用語 測驗

**科目:** Kubernetes 入門課程
**難度:** 混合 (基礎/中等/進階)
**題數:** 20題
**建議時間:** 25-30分鐘

---

## 第一部分：選擇題 (每題 5分，共 25分)

**Q1.** 在 Kubernetes 中，YAML 語言的核心特性是什麼？

A) 命令式 (Imperative) - 強調順序和過程
B) 聲明式 (Declarative) - 注重結果而非過程
C) 過程式 (Procedural) - 需要詳細執行步驟
D) 交互式 (Interactive) - 即時響應用戶操作

<details>
<summary>答案</summary>
<b>B) 聲明式 (Declarative) - 注重結果而非過程</b>
YAML 在 Kubernetes 中的核心特性是「聲明式」，與「命令式」完全相反。聲明式不關心具體的過程，更注重結果。我們只需要告訴 Kubernetes 一個目標狀態，它自己就會想辦法去完成任務，自動化、智能化程度更高。
選項 A、C 都屬於命令式特點；選項 D 不符合 Kubernetes YAML 的核心特性。
</details>

---

**Q2.** YAML 與 JSON 的關係是什麼？

A) YAML 和 JSON 完全不同，沒有任何關聯
B) YAML 是 JSON 的子集，功能更簡化
C) YAML 是 JSON 的超集，任何合法的 JSON 都是 YAML
D) YAML 和 JSON 是同一種格式的不同名稱

<details>
<summary>答案</summary>
<b>C) YAML 是 JSON 的超集，任何合法的 JSON 都是 YAML</b>

YAML 是 JSON 的超集，支持整數、浮點數、布林、字符串、數組和對象等數據類型。任何合法的 JSON 文檔也都是 YAML 文檔，如果你了解 JSON，學習 YAML 會容易很多。

選項 A 錯誤，兩者有密切關係；選項 B 說反了；選項 D 錯誤，兩者是不同的格式。
</details>

---

**Q3.** 在描述 Kubernetes API 對象時，哪些字段是必須的？

A) name, labels, containers
B) apiVersion, kind, metadata
C) spec, status, containers
D) image, ports, name

<details>
<summary>答案</summary>
<b>B) apiVersion, kind, metadata</b>

描述 API 象時，「header」部分包含 API 對象的基本信息，有三個必須字段：apiVersion、kind、metadata。apiVersion 表示操作這種資源的 API 版本號；kind 表示資源對象的類型；metadata 表示資源的「元信息」。

選項 A 的 labels 不是必須字段；選項 C 的 status 是對象創建後自動生成的；選項 D 都是 spec 里的字段，不是 header 必須字段。
</details>

---

**Q4.** 以下哪個命令可以查看 Kubernetes API 對象的版本和類型信息？

A) kubectl get pods
B) kubectl describe pod
C) kubectl api-resources
D) kubectl apply -f

<details>
<summary>答案</summary>
<b>C) kubectl api-resources</b>kubectl api-resources 命令會顯示當前 Kubernetes 版本支持的所有對象，包括對象的名字、簡寫、API 版本等信息。在「NAME」欄可以看到對象名字，第二欄「SHORTNAMES」是資源的簡寫。選項 A 是查看 Pod 狀態；選項 B 是查看 Pod 詳細信息；選項 D 是應用 YAML 文件。
</details>
---

**Q5.** 如何使用 kubectl 生成一個 Pod 的 YAML 模板？

A) kubectl create pod --template
B) kubectl run ngx --image=nginx:alpine --dry-run=client -o yaml
C) kubectl generate yaml ngx
D) kubectl template ngx --image=nginx:alpine

<details>
<summary>答案</summary>
<b>B) kubectl run ngx --image=nginx:alpine --dry-run=client -o yaml</b>

使用 --dry-run=client 和 -o yaml 這兩個特殊參數，前者是空運行，後者是生成 YAML 格式，結合起來使用會讓 kubectl 不會有實際的創建動作，而只生成 YAML 文件。這個技巧可以生成絕對正確的 YAML 文件模板。

選項 A、C、D 都不是有效的 kubectl 命令格式。
</details>

---

## 第二部分：是非題 (每題 3分，共 15分)

**Q6.** YAML 使用花括號和方括號來表示層次結構，就像 JSON 一樣。 _(是非題)_

<details>
<summary>答案</summary>
<b>錯誤</b> - YAML 使用空白與縮進表示層次（類似 Python），可以不使用花括號和方括號。這是 YAML 相比 JSON 更簡潔清晰的重要特點，避免了閉合標記的麻煩。
</details>

---

**Q7.** 在 YAML 中，表示對象的冒號（:）和表示數組的橫線（-）後面必須要有空格。 _(是非題)_

<details>
<summary>答案</summary>
<b>正確</b> - YAML 語法規定，表示對象的 : 和表示數組的 - 後面都必須要有空格，這是 YAML 格式的強制要求，否則會導致解析錯誤。
</details>

---

**Q8.** Kubernetes 的 apiserver 是集群的唯一入口，所有操作都通過它來進行。 _(是非題)_

<details>
<summary>答案</summary>
<b>正確</b> - apiserver 是 Kubernetes 系統的唯一入口，外部用戶和內部組件都必須和它通信。它採用了 HTTP 協議的 URL 資源理念，API 風格用 RESTful 的 GET/POST/DELETE 等，所以 Kubernetes 的概念被稱為「API 象」。
</details>

---

**Q9.** 在 Kubernetes 中，image（鏡像）是一種獨立的 API 對象類型。 _(是非題)_

<details>
<summary>答案</summary>
<b>錯誤</b> - 在 Kubernetes 的 API 對象中，image 不是獨立的資源類型。鏡像信息是作為 Pod 的 spec.containers 字段的一部分存在的，並沒有獨立的 images API 對象。
</details>

---

**Q10.** 使用 kubectl apply 命令創建對象時，如果對象已存在，會直接報錯終止。 _(是非題)_

<details>
<summary>答案</summary>
<b>錯誤</b> - kubectl apply 是聲明式操作，具有「補丁」特性。如果對象不存在則創建；如果對象已存在，則比較 spec 進行相應更新變更，而不是報錯終止。這正是聲明式與命令式的區別之一。
</details>

---

## 第三部分：填充題 (每題 4分，共 20分)

**Q11.** YAML 使用 **\_\_\_\_\_\_\_\_** 符號來書寫註釋，比起 JSON 是很大的改進。

<details>
<summary>答案</summary>
<b># (井號)</b>

YAML 可以使用 #書寫註釋，這是相比 JSON 的重要優勢。JSON 不支持註釋，這在配置文件中很不方便。
</details>

---

**Q12.** 可以使用 **\_\_\_\_\_\_\_\_** 在一個 YAML 文件里分隔多個 YAML 象。

<details>
<summary>答案</summary>
<b>---</b>

YAML 支持 使用 --- 在一個文件里分隔多個 YAML 象，這樣可以在一個文件中定義多個 Kubernetes 資源對象，方便管理。
</details>

---

**Q13.** Kubernetes API 象的 YAML 描述可以分成「header」和「body」兩部分，「header」包含的三個必須字段是：apiVersion、kind 和 **\_\_\_\_\_\_\_\_**。

<details>
<summary>答案</summary>
<b>metadata</b>

描述 API 對象時，header 包含三個必須字段：apiVersion（API 版本）、kind（資源類型）、metadata（元信息）。metadata 包含 name、labels 等信息，用來標記對象。
</details>

---

**Q14.** Kubernetes API 象的「body」部分表現為 **\_\_\_\_\_\_\_\_** 字段，表示我們對對象的「期望狀態」(desired status)。

<details>
<summary>答案</summary>
<b>spec (specification)</b>

spec 字段是 API 對象的「body」部分，表示 specification（規格說明），記錄了對象的「期望狀態」。不同的 API 象會有不同的 spec 定義，比如 Pod 的 spec 包含 containers 數組。
</details>

---

**Q15.** 使用 kubectl explain 命令可以查看 API 對象字段的詳細說明，相當於 Kubernetes 自帶的 **\_\_\_\_\_\_\_\_**。

<details>
<summary>答案</summary>
<b>API 文檔</b>

kubectl explain 命令相當於 Kubernetes 自帶的 API 文檔，會給出對象字段的詳細說明。例如 kubectl explain pod、kubectl explain pod.spec.containers 等，這樣就不必去網上查找官方文檔了。
</details>

---

## 第四部分：簡答題 (每題 10分，共 30分)

**Q16.** 請解釋「命令式」(Imperative) 和「聲明式」(Declarative) 的主要區別，並說明為什麼 Kubernetes 選擇使用聲明式。

<details>
<summary>答案</summary>

**主要區別：**

- **命令式 (Imperative):** 注重順序和過程，必須告訴計算機每步該做什麼，所有步驟都列清楚，程序才能一步步執行完成任務。交互性強，但自動化程度低。

- **聲明式 (Declarative):** 不關心具體過程，注重結果。只需要告訴系統目標狀態，它自己會想辦法完成任務，自動化、智能化程度更高。

**Kubernetes 選擇聲明式的原因：**

1. Kubernetes 採用 Master/Node 架構，對整個集群狀態了如指掌
2. 內部有眾多組件和插件自動監控管理應用
3. Kubernetes 知道的信息比用戶更多更全面（如節點狀態、資源分配）
4. 用戶是「外行」，不應去指導 Kubernetes「內行」
5. 聲明式能避免引入繁瑣操作步驟干擾系統，與高度自動化的內部結構相得益彰
6. 純文本 YAML 易於版本化，適合 CI/CD 流程

**關鍵點要包括：**
- 過程 vs 結果的區別
- Kubernetes 的自動化架構特性
- 信息掌握程度的對比（內行 vs 外行）
- 與 DevOps/CI/CD 的契合性
</details>

---

**Q17.** 說明 YAML 相比 JSON 的優勢，以及 YAML 在描述 Kubernetes API 對象時的語法特點。

<details>
<summary>答案</summary>

**YAML 相比 JSON 的優勢：**

1. **語法更簡潔：** 使用空白與縮進表示層次，不需要花括號和方括號
2. **可讀性更好：** 格式清晰緊湊，更適合人類閱讀
3. **支持註釋：** 可以使用 #書寫註釋，JSON 不支持註釋
4. **Key 不需要引號：** 對象的 Key 不需要雙引號，更清爽
5. **元素分隔簡單：** 每個元素後不需要逗號
6. **多對象支持：** 可以用 --- 在一個文件分隔多個對象

**YAML 描述 Kubernetes API 對象的語法特點：**

1. **層次結構：** 使用縮進表示嵌套關係（metadata、spec 等）
2. **對象格式：** Key: Value（冒號後必須有空格）
3. **數組格式：** 使用橫線 - 開頭的清單形式（如 containers 列表）
4. **組合能力：** 可以組合數組和對象描述任意複雜的 API 象
5. **必須字段：** apiVersion、kind、metadata 是必須的 header 字段
6. **spec 字段：** 描述期望狀態的 body 部分，每種對象有不同定義

**關鍵點要包括：**
- 可讀性和註釋支持
- 縮進和分隔符的語法規則
- Key 不需要引號的便利性
- 與 Kubernetes API 對象結構的對應關係
</details>

---

**Q18.** 說明如何使用 kubectl 命令來編寫 YAML 文件的三個技巧，並解釋每個技巧的作用。

<details>
<summary>答案</summary>

**技巧一：kubectl api-resources**

**作用：** 查看資源對象相應的 API 版本和類型信息，包括：
- NAME：對象名稱（如 Pod、ConfigMap、Service）
- SHORTNAMES：簡寫（如 po、svc）
- APIVERSION：API 版本（如 v1、networking.k8s.io/v1）

**使用方式：** 照著顯示的版本和類型寫 YAML，確保 apiVersion 和 kind 字段絕對正確。

---

**技巧二：kubectl explain**

**作用：** 查看 API 對象字段的詳細說明文檔，相當於內置 API 參考手冊。

**使用方式：**
```
kubectl explain pod              # 查看 Pod 整體結構
kubectl explain pod.metadata     # 查看 metadata 字段
kubectl explain pod.spec         # 查看 spec 字段
kubectl explain pod.spec.containers  # 查看 containers 子字段
```

**優勢：** 不必去網上查官方文檔，直接在本地獲取字段說明。

---

**技巧三：--dry-run=client -o yaml**

**作用：**讓 kubectl 生成 YAML 模板文件，免去打字和格式對齊工作。

**使用方式：**
```bash
kubectl run ngx --image=nginx:alpine --dry-run=client -o yaml
```

**結果：** 生成絕對正確的 YAML 文件模板，包含所有必要字段。

**進階技巧：** 定義 Shell 變量簡化操作：
```bash
export out="--dry-run=client -o yaml"
kubectl run ngx --image=nginx:alpine $out
```

---

**關鍵點要包括：**
- 三個技巧的具體命令
- 每個技巧解決的具體問題
- 如何組合使用提高效率
- Shell 變量的進階用法
</details>

---

## 第五部分：配對題 (10分)

**Q19.** 請將下列 Kubernetes API 象的必須字段與其功能說明進行配對：

| 字段 | 功能說明 |
| ---- | -------- |
| 1. apiVersion | A. 表示資源的一些「元信息」，用來標記對象，如 name、labels |
| 2. kind | B. 表示操作這種資源的 API 版本號，如 v1、v1beta1 |
| 3. metadata | C. 表示資源對象的類型，如 Pod、Node、Service |
| 4. spec | D. 表示對象的「期望狀態」，記錄具體的配置要求 |

<details>
<summary>答案</summary>
1-B, 2-C, 3-A, 4-D

**詳細說明：**

- **apiVersion (B):** 表示 API 版本號，用於區分不同版本的對象定義，如 v1、v1alpha1、v1beta1 等。

- **kind (C):** 表示資源對象類型，如 Pod、Node、Job、Service、Deployment 等。

- **metadata (A):** 表示元信息，包含 name（對象名稱）、labels（標籤）、namespace 等標記信息。

- **spec (D):** 表示期望狀態 (desired status)，每種對象有不同的 spec 定義，如 Pod 的 spec 包含 containers 數組、鏡像、端口等信息。
</details>

---

## 第六部分：實作應用題 (10分)

**Q20.** 以下是一個描述 Kubernetes Pod 的 YAML 文件，請分析其結構並說明每個部分的作用：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ngx-pod
  labels:
    env: demo
    owner: chrono
spec:
  containers:
  - image: nginx:alpine
    name: ngx
    ports:
    - containerPort: 80
```

<details>
<summary>答案</summary>

**YAML 文件結構分析：**

## Header 部分（必須字段）

**apiVersion: v1**
- 作用：指定使用 v1 版本的 API 接口
- 這是 Pod 對象的標準 API 版本

**kind: Pod**
- 作用：聲明資源類型為 Pod
- 表示要創建一個 Pod 對象

**metadata（元信息）**
- **name: ngx-pod**
  - 作用：給 Pod 唯一的名稱標識
  - 用於查找、引用和管理 Pod

- **labels（標籤）**
  - **env: demo** - 環境標籤，標記為演示環境
  - **owner: chrono** - 擁有者標籤，標記負責人
  - 作用：便於篩選、查找和管理對象（如 kubectl get pods -l env=demo）

## Body 部分（期望狀態）

**spec（規格說明）**
- 作用：描述 Pod 的期望狀態，告 Kubernetes 想要什樣的 Pod

**containers（容器列表）**
- 這是一個數組，每個元素描述一個容器

**第一個容器配置：**
- **image: nginx:alpine**
  - 作用：指定使用 nginx:alpine 鏡像創建容器
  - alpine 版本更輕量

- **name: ngx**
  - 作用：給容器命名為 ngx，Pod 內唯一標識

- **ports**
  - **containerPort: 80**
  - 作用：聲明容器暴露的端口為 80
  - 說明容器內應用監聽的端口

## 整體聲明式特點

這個 YAML 文件完整描述了「目標狀態」：
- 使用 nginx:alpine 鏡像
- 創建名為 ngx-pod 的 Pod
- 運行名為 ngx 的容器
- 暴露 80 端口
- 添加環境和擁有者標籤

Kubernetes 收到這個聲明後，會自動決定：
- Pod 在哪個節點運行
- 如何拉取鏡像
- 如何創建容器
- 如何配置網絡
- 如何維護運行狀態

**關鍵分析點：**
- Header vs Body 的結構
- metadata 的元信息作用
- spec 的期望狀態定義
- containers 數組的配置
- 聲明式的「目標狀態」理念
</details>

---

## 測驗結束

**總分：** 100分

**評分標準建議：**
- 90-100分：優秀，完全掌握 YAML 與 Kubernetes API 對象
- 70-89分：良好，基本理解核心概念，部分細節需要加強
- 60-69分：及格，需要更多實作練習
- 低於 60分：需要重新學習本課程內容

---

_Generated from: 11｜YAML：Kubernetes世界里的通用語.pdf_
_Kubernetes 入門課程 | 時長 18:09 | 作者 Chrono_