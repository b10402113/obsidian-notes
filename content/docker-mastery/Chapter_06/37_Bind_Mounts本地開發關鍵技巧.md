# Bind Mounts：本地開發的關鍵技巧

## 📝 課程概述

本單元介紹 **Bind Mounts** 的運作原理與實際應用。與 Data Volumes 不同，Bind Mounts 讓我們可以直接將 Host 的檔案或目錄映射進 Container，這對於本地開發環境來說是革命性的功能——你可以在 Host 編輯程式碼，Container 內立即反映變更，無需重建 Image。

---

## 核心觀念與實作解析

### 什麼是 Bind Mount？

**Bind Mount 的本質很簡單：將 Host 的檔案或目錄，映射到 Container 內的檔案或目錄。**

背後的運作機制是：兩個位置指向同一個實體磁碟位置。

> 它同樣會跳過 UFS（Union File System），所以當你刪除 Container 時，Host 上的檔案不會受到影響。

---

### Host 檔案優先原則

當 Bind Mount 的目標路徑在 Container 內已有檔案時，**Host 的檔案會優先**。

> 但這不是真的「覆蓋」Container 內的檔案，而是在 Bind Mount 存在期間，Container 會看到 Host 的檔案。一旦移除 Bind Mount 重新執行 Container，原本的檔案就會重新出現。

---

### Bind Mount 與 Named Volume 的語法差異

兩者都使用 `-v` 參數，但格式不同：

| 類型         | 語法格式                          | Docker 如何識別           |
| ------------ | --------------------------------- | ------------------------- |
| Named Volume | `-v volume_name:/container/path`  | 左邊是名稱（非路徑）      |
| Bind Mount   | `-v /host/path:/container/path`   | 左邊以 `/` 開頭（是路徑） |

**Windows 使用者注意**：路徑格式為 `//C/path/to/file`

---

### 為什麼 Bind Mount 無法寫在 Dockerfile 中？

因為 Bind Mount 需要 Host 上有具體的檔案或目錄才能運作，這與 Image 的可攜性本質衝突。

> Image 應該能在任何機器上運行，不應綁定特定 Host 的檔案路徑。因此 Bind Mount **只能在 runtime** 使用 `docker container run` 指定。

---

### 實作範例：使用 Nginx 即時預覽檔案變更

讓我們用一個實際案例來理解 Bind Mount 的威力。

**情境**：我們有一個簡單的 Nginx 設定，希望能在 Host 編輯 `index.html`，Container 內的 Nginx 立即看到變更。

**步驟 1：進入課程 Repo 的範例目錄**

```bash
cd dockerfile-sample-2
```

目錄內有：
- `Dockerfile`：簡單的 Nginx 設定
- `index.html`：自訂的首頁

**步驟 2：啟動 Container 並掛載目錄**

```bash
docker container run -d --name nginx -p 8080:80 -v $(pwd):/usr/share/nginx/html nginx
```

這裡的 `-v $(pwd):/usr/share/nginx/html` 做了什麼？

- `$(pwd)`：Shell 指令，會被替換成當前目錄的完整路徑
- `/usr/share/nginx/html`：Nginx 預設存放靜態檔案的路徑
- 整個效果：將當前目錄掛載進 Container

**步驟 3：驗證結果**

開啟瀏覽器前往 `localhost:8080`，你會看到自訂的 `index.html` 內容。

> 對比實驗：如果執行相同指令但移除 `-v` 參數，會看到 Nginx 預設的歡迎頁面。

---

### 進一步驗證：雙向同步

讓我們進入 Container 內部觀察：

```bash
docker container exec -it nginx bash
cd /usr/share/nginx/html
ls -al
```

你會發現：
1. 看得到 `Dockerfile`——因為我們掛載的是整個目錄
2. 在 Container 內建立的新檔案，在 Host 也看得到
3. 刪除 Container 內的檔案，Host 也會同步刪除

> **這就是 Bind Mount 的本質：兩邊是同一個實體位置。**

---

### 本地開發的革命性改變

老師分享了他的觀察：當開發者理解 Bind Mount 後，往往會重新思考整個開發環境的建置方式。

**傳統做法的痛點**：
- 使用 Vagrant 或手動設定開發環境
- 需要在本機安裝特定版本的語言、工具
- 不同專案可能有版本衝突

**Docker + Bind Mount 的優勢**：
- 只需執行 Container，不需要在本機安裝複雜工具鏈
- 用 `-v` 掛載程式碼目錄
- 在 Host 用熟悉的編輯器修改程式碼
- Container 內即時反映變更
- 通常甚至不需要進入 Container shell，看 logs 就能除錯

---

### 設定 Read-Only 掛載

如果你想要 Container 只能讀取、不能寫入掛載的目錄：

```bash
docker container run -v /host/path:/container/path:ro nginx
```

在路徑後加上 `:ro` 即可。

---

## 💡 重點摘要

- **Bind Mount 將 Host 檔案/目錄映射進 Container，兩者指向同一實體位置。**
- **Host 檔案優先原則：Bind Mount 存在時，Container 會看到 Host 的檔案版本。**
- **Bind Mount 只能在 runtime 指定，不能寫在 Dockerfile 中——因為它依賴特定 Host 的路徑。**
- **這是本地開發的關鍵技巧：在 Host 編輯程式碼，Container 內即時反映，無需重建 Image。**
- **使用 `$(pwd)` 可以快速指定當前目錄，省去輸入完整路徑的麻煩。**

---

## 🔑 關鍵字

Bind Mount, Volume, Nginx, UFS, Read-Only
