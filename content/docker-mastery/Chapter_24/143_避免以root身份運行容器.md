# 避免以 Root 身份運行容器應用程式

## 📝 課程概述

本單元說明如何在 Dockerfile 中使用 `USER` 指令,避免應用程式以 root 身份在容器內運行。這項安全實踐能降低容器被入侵後的影響範圍,並進一步透過 User Namespaces 機制強化隔離效果。

## 核心觀念與實作解析

### 為何要避免以 Root 身份運行?

#### 兩層次的安全考量

1. **容器內的安全性**:若攻擊者突破應用程式,root 身份可在容器內為所欲為
2. **逃逸至宿主機的風險**:若存在核心漏洞,root 身份更容易逃離容器

**關鍵認知**:容器內的 root 雖然無法直接存取宿主機,但若發生 Docker 或 Linux 核心漏洞,root 身份將大幅增加逃逸成功的機率。

### 預設行為分析:官方映像檔的現況

#### Daemon 類程式的處理方式

許多 Daemon 類程式(如 Nginx、MySQL、PostgreSQL)會自動處理權限問題:

**Nginx 的實際案例**:
```bash
docker run nginx
docker top [container_id]
```

輸出會顯示:
- 一個 master process 以 root 身份運行
- 多個 worker process 以一般使用者(如 101)身份運行

**設計邏輯**:master process 僅負責管理,實際處理連線的是 worker process,這樣即使 HTTP 協定出現漏洞,攻擊者也被限制在一般使用者權限內。

#### 程式語言類映像檔的預設值

對於 Node.js、Python、Ruby 等程式語言的官方映像檔:

- **預設行為**:以 root 身份運行
- **有建立使用者**:大多數官方映像檔會建立 `node`、`python` 等使用者
- **但不切換**:需要開發者自行在 Dockerfile 中切換使用者

**為什麼不預設切換?** 因為 Docker 無法預測你的應用程式需求,貿然切換可能導致:
- 應用程式無法寫入檔案
- 無法建立目錄
- 權限錯誤導致程式崩潰

### 實作步驟:在 Dockerfile 中切換使用者

#### 基本語法

```dockerfile
# 使用官方映像檔已建立的使用者
USER node

# 或自行建立使用者
RUN groupadd -r myuser && useradd -r -g myuser myuser
USER myuser
```

#### 權限設定的關鍵細節

切換使用者後,必須確保檔案權限正確:

```dockerfile
# 方法 1:在建立目錄時設定權限
RUN mkdir -p /app && chown -R node:node /app

# 方法 2:在 COPY 指令中使用 --chown
COPY --chown=node:node . /app

# 方法 3:在 RUN 指令前處理權限
RUN chown -R node:node /app
USER node
RUN npm install  # 此時以 node 身份執行
```

**常見錯誤**:忘記設定權限,導致應用程式無法讀取檔案或寫入日誌。

### 需要特別注意的應用類型

#### 會寫入檔案的應用程式

特別是允許使用者上傳檔案的應用(如 WordPress):
- 使用者上傳圖片
- 日誌檔案寫入
- 暫存檔建立

**解決方案**:在容器啟動前,確認所有寫入目錄的權限都已正確設定。

### User Namespaces:進階隔離機制

#### 運作原理

User Namespaces 是 Docker Engine 的宿主機層級設定:

- **設定位置**:`/etc/docker/daemon.json`
- **效果**:所有容器的程序在宿主機上以非 root 身份運行
- **對應關係**:容器內的 root 對應到宿主機的高編號使用者(如 100000)

**技術細節**:Docker 使用 `containerd` 與 `runC` 來啟動容器,啟用 User Namespaces 後,這些程序不再以 root 身份運行。

#### 限制與注意事項

1. **不支援自訂網路**:建立 bridge 網路需要 root 權限
2. **Volume 權限複雜性**:容器寫入的檔案會以高編號使用者身份存檔,需處理權限對應問題
3. **相容性問題**:並非所有應用程式都支援 User Namespaces

#### 部署策略

在叢集環境中,建議採用分段部署:
- **特定 Worker Node 啟用**:僅在特定節點啟用 User Namespaces
- **應用程式分類**:將支援的應用程式排程到這些節點
- **DMZ 環境優先**:在對外暴露的 DMZ 區域優先啟用

### 實作建議

#### 測試流程

1. **先在開發環境測試**:確認應用程式功能正常
2. **檢查寫入需求**:列出所有需要寫入檔案的路徑
3. **調整權限**:使用 `chown` 確保使用者有寫入權限
4. **完整 CI/CD 測試**:在自動化流程中驗證

#### 參考資源

- Docker 官方文件關於 User Namespaces 的設定說明
- DockerCon 演講影片:實際案例展示

## 💡 重點摘要

- 程式語言類官方映像檔預設以 root 運行,需開發者自行使用 `USER` 指令切換
- 切換使用者後,必須使用 `--chown` 或 `chown` 確保檔案權限正確
- User Namespaces 讓容器程序在宿主機上以非 root 身份運行,提供額外防護層
- 啟用 User Namespaces 需考慮網路與 Volume 權限的相容性問題

## 🔑 關鍵字

USER, User Namespaces, Container Isolation, File Permissions, runC
