# docker-compose 測驗

**科目:** Kubernetes 入門課程
**難度:** 中等
**題數:** 20
**建議時間:** 25 分鐘

---

## 第一部分：選擇題 (每題 5 分)

**Q1.** docker-compose 最初源自哪個項目？

A) Docker Swarm
B) Fig
C) Kubernetes
D) Docker Hub

<details>
<summary>答案</summary>
<b>B) Fig</b> - docker-compose 源自 Fig 項目，該項目為 Docker 引入了「容器編排」的概念，Docker 公司在 2014 年 7 月收購了它並改名為 docker-compose。
</details>

**Q2.** 下列關於 docker-compose 與 Kubernetes 的比較，何者正確？

A) docker-compose 適合管理大規模分散式叢集
B) docker-compose 與 Kubernetes 使用完全相同的 YAML 語法
C) docker-compose 只能在單機環境運行，適合簡單應用場景
D) docker-compose 的運行成本比 Kubernetes 更高

<details>
<summary>答案</summary>
<b>C) docker-compose 只能在單機環境運行，適合簡單應用場景</b> - docker-compose 是單機環境下的輕量級容器編排工具，填補了 Docker 和 Kubernetes 之間的空白，適合快速啟動少量容器進行開發測試，而不需要承擔 Kubernetes 的運行成本。
</details>

**Q3.** 在 docker-compose 的 YAML 文件中，核心概念 "service" 代表什麼？

A) Kubernetes 的 Service API 對象
B) 一個容器化的應用程序
C) Docker 的網絡服務
D) 系統守護進程

<details>
<summary>答案</summary>
<b>B) 一個容器化的應用程序</b> - docker-compose 裡的 "service" 是一個容器化的應用程序，通常是一個後台服務。它與 Kubernetes 的 Service 雖然名字相似，但含義完全不同。
</details>

**Q4.** 在 docker-compose 中，下列哪個字段用於設置容器之間的依賴關係？

A) depends_on
B) requires
C) links
D) dependencies

<details>
<summary>答案</summary>
<b>A) depends_on</b> - depends_on 字段用來設置容器的依賴關係，指定容器啟動的先後順序，這在編排由多個容器組成的應用時非常便利。
</details>

**Q5.** docker-compose 中，容器之間如何進行網絡通信？

A) 必須使用 IP 地址
B) 需要手動配置 DNS
C) 使用 service 名稱作為網絡標識
D) 只能通過環境變量傳遞地址

<details>
<summary>答案</summary>
<b>C) 使用 service 名稱作為網絡標識</b> - docker-compose 會自動將 service 的名字用作網絡標識，類似 Kubernetes 的 Service 域名機制。例如 WordPress 可以直接用 "mariadb" 作為數據庫地址，無需手動指定 IP。
</details>

**Q6.** 啟動 docker-compose 應用的命令是？

A) docker-compose start
B) docker-compose run
C) docker-compose up
D) docker-compose begin

<details>
<summary>答案</summary>
<b>C) docker-compose up</b> - 使用 docker-compose up -d 命令來啟動應用，-d 參數表示後台運行，同時需要用 -f 參數指定 YAML 文件。
</details>

---

## 第二部分：是非題 (每題 5 分)

**Q7.** docker-compose 可以完全替代 Kubernetes 進行大規模生產環境的容器編排。_(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - docker-compose 只能用於單機環境，編排功能比較簡單，缺乏運維監控手段。它適合小型開發測試環境，無法替代 Kubernetes 在大規模生產環境中的叢集管理能力。
</details>

**Q8.** docker-compose 使用 YAML 語法，其設計理念與 Kubernetes 完全相同。_(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - 雖然 docker-compose 和 Kubernetes 都使用 YAML，但 docker-compose 的基因來自 Docker，語法語義更接近 Docker 命令行。Kubernetes 的設計理念則完全不同，使用了更多抽象概念。
</details>

**Q9.** docker-compose 中的 environment 字段可以直接定義環境變量，無需像 Kubernetes 的 ConfigMap 那樣先創建再引用。_(對/錯)_

<details>
<summary>答案</summary>
<b>對</b> - docker-compose 使用 environment 字段直接定義環境變量，比 Kubernetes 的 ConfigMap 更直觀。例如：MARIADB_DATABASE: db 直接在 YAML 中定義，不需要額外創建配置對象。
</details>

**Q10.** 使用 docker-compose 管理的容器無法通過 docker ps 命令查看。_(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - docker-compose 在底層仍然調用 Docker，所以它啟動的容器可以用 docker ps 看到。同時也可以用 docker-compose ps 查看更多信息。
</details>

---

## 第三部分：填充題 (每題 6 分)

**Q11.** Docker 公司在 **____** 年收購了 Fig 項目，並將其改名為 docker-compose。

<details>
<summary>答案</summary>
<b>2014</b> - Docker 公司在 2014 年 7 月收購了 Fig 項目，集成進 Docker 內部後改名為 docker-compose。
</details>

**Q12.** docker-compose 的 YAML 文件中，**____** 字段用於聲明容器端口映射，其語法與 Docker 命令行一致。

<details>
<summary>答案</summary>
<b>ports</b> - ports 字段用於聲明端口映射，例如 "5000:5000"，語法與 Docker 命令行的 -p 參數一致。
</details>

**Q13.** docker-compose 中使用 **____** 字段將宿主機文件掛載到容器中，用於加載配置文件。

<details>
<summary>答案</summary>
<b>volumes</b> - volumes 字段用於掛載文件或目錄，例如 "./wp.conf:/etc/nginx/conf.d/default.conf" 可將本地配置文件掛載到容器內。
</details>

**Q14.** 停止 docker-compose 應用的命令是 **____**。

<details>
<summary>答案</summary>
<b>docker-compose down</b> - 使用 docker-compose down 命令來停止並清理應用，同樣需要 -f 參數指定 YAML 文件。
</details>

---

## 第四部分：簡答題 (每題 10 分)

**Q15.** 請說明 docker-compose 與 Docker 命令行相比的優勢是什麼？

<details>
<summary>答案</summary>
docker-compose 相比 Docker 命令行的優勢：

<b>關鍵要點：</b>
- <b>聲明式配置</b>：使用 YAML 文件定義容器參數，實現「聲明式」操作，避免了 Docker 冗長命令行的煩惱
- <b>自動網絡解析</b>：service 名稱自動作為網絡標識，容器間無需手動指定 IP 地址即可通信
- <b>依賴管理</b>：通過 depends_on 字段自動管理容器啟動順序
- <b>可重用性</b>：YAML 配置文件可以版本控制，方便分享和重現環境
- <b>多容器編排</b>：一次命令即可啟動多個相關聯的容器，比 Shell 腳本更清晰易維護
</details>

**Q16.** 在 docker-compose 中如何使用 Nginx 作為反向代理連接 WordPress？請說明配置要點。

<details>
<summary>答案</summary>
使用 Nginx 作為反向代理的配置要點：

<b>關鍵要點：</b>
- <b>配置文件掛載</b>：使用 volumes 字段將 Nginx 配置文件掛載到容器中，例如 "./wp.conf:/etc/nginx/conf.d/default.conf"
- <b>網絡標識</b>：在 Nginx 配置的 proxy_pass 指令中，直接使用 WordPress 的 service 名稱作為地址，例如 "proxy_pass http://wordpress;"
- <b>依賴關係</b>：在 Nginx service 中使用 depends_on 指定依賴 wordpress，確保啟動順序正確
- <b>端口映射</b>：將 Nginx 的 80 端口映射到宿主機，對外提供訪問入口

docker-compose 雖然沒有 ConfigMap 概念，但通過 volumes 掛載外部文件仍可實現配置管理。
</details>

---

## 第五部分：配對題 (10 分)

**Q17.** 請將下列 docker-compose 命令與其功能進行配對：

| 命令 | 功能 |
|------|------|
| 1. docker-compose up -d | A. 查看容器狀態 |
| 2. docker-compose ps | B. 進入容器內部執行命令 |
| 3. docker-compose down | C. 後台啟動應用 |
| 4. docker-compose exec | D. 停止並清理應用 |

<details>
<summary>答案</summary>
1-C, 2-A, 3-D, 4-B

<b>解析：</b>
- <b>docker-compose up -d</b>：後台啟動應用（-d 表示 detached 模式）
- <b>docker-compose ps</b>：查看容器狀態，比 docker ps 顯示更多信息
- <b>docker-compose down</b>：停止並清理應用，移除容器和網絡
- <b>docker-compose exec</b>：在運行的容器中執行命令，常用於進入容器調試
</details>

---

## 第六部分：綜合應用題 (12 分)

**Q18.** 閱讀以下 docker-compose YAML 配置，回答問題：

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    depends_on:
      - app

  app:
    image: myapp:v1
    environment:
      - DB_HOST=db
      - DB_PORT=3306
    depends_on:
      - db

  db:
    image: mariadb:10
    environment:
      MYSQL_ROOT_PASSWORD: secret
```

請回答：
1. 這個應用包含幾個容器？它們的名稱分別是什麼？
2. 容器啟動的順序會是怎樣的？
3. 如果要從 web 容器訪問 app 容器，應該使用什麼地址？
4. 這個配置有什麼潛在問題？

<details>
<summary>答案</summary>
<b>答案解析：</b>

1. <b>容器數量和名稱：</b>
   - 3 個容器
   - 名稱：web、app、db

2. <b>啟動順序：</b>
   - 首先啟動 db（無依賴）
   - 然後啟動 app（依賴 db）
   - 最後啟動 web（依賴 app）

3. <b>訪問地址：</b>
   - 直接使用 service 名稱 "app" 作為地址
   - 例如在 Nginx 配置中：proxy_pass http://app

4. <b>潛在問題：</b>
   - depends_on 只保證容器啟動順序，不保證服務就緒
   - db 容器啟動後，MariaDB 服務可能還需要幾秒鐘初始化
   - app 可能在數據庫未完全就緒時就嘗試連接，導致失敗
   - 建議添加健康檢查或在應用中實現重試邏輯
</details>

---

## 額外思考題 (不計分)

**Q19.** 在什麼情況下你會選擇使用 docker-compose 而不是 Kubernetes？反之又在什麼情況下選擇 Kubernetes？

<details>
<summary>答案</summary>
<b>選擇 docker-compose 的場景：</b>
- 單機開發測試環境
- 快速原型驗證
- 本地搭建數據庫、消息中間件等開發工具
- 只有幾個容器的小型應用
- 團隊資源有限，無需複雜編排功能
- CI/CD 流水線中的測試環境

<b>選擇 Kubernetes 的場景：</b>
- 大規模生產環境
- 需要高可用和自動故障恢復
- 多節點叢集管理
- 複雜的服務發現和負載均衡需求
- 需要自動擴縮容
- 多團隊協作的大型項目
- 需要完善的監控和運維體系

<b>關鍵原則：</b>「殺雞不用牛刀」 - 根據實際需求選擇合適的工具，避免過度設計。
</details>

**Q20.** docker-compose 與 Kubernetes 在處理配置文件（如 ConfigMap/Secret）方面有何不同？這會如何影響你的設計？

<details>
<summary>答案</summary>
<b>主要差異：</b>

<b>Kubernetes：</b>
- 提供 ConfigMap 和 Secret API 對象
- 配置與應用分離，可以獨立管理和更新
- 支持掛載為文件或環境變量
- 有版本控制和加密機制

<b>docker-compose：</b>
- 沒有 ConfigMap/Secret 概念
- 環境變量直接在 environment 字段定義
- 配置文件需要通過 volumes 從宿主機掛載
- 敏感信息缺乏加密保護

<b>設計影響：</b>
- docker-compose 更適合開發環境，配置簡單直接
- 生產環境需要額外措施保護敏感信息（如使用 .env 文件並加入 .gitignore）
- Kubernetes 更適合管理複雜配置和敏感信息
- 遷移時需要重構配置管理方式
</details>

---

_生成自：PDF/Kubernetes 入門課程/加餐｜docker-compose：单机环境下的容器编排工具.pdf_