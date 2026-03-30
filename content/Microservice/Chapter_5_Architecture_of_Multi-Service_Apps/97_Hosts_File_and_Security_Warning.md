# Hosts 檔案設定與 Self-Signed 憑證安全警告處理

## 📝 課程概述

本單元完成了本地開發環境的最後一個設定環節：將 `ticketing.dev` 網域對應到本機 IP，並解決 Chrome 瀏覽器對於 Ingress Nginx 使用 **Self-Signed 憑證**時顯示的安全警告。透過修改 `hosts` 檔案與繞過瀏覽器的 SSL 警告，我們成功建立了從瀏覽器一路直達 Kubernetes 叢集內 Auth Service 的完整網路路徑。

## 核心觀念與實作解析

### 為什麼需要修改 hosts 檔案？

在本地 Kubernetes 開發環境中，Ingress Nginx Controller 運行在我們的機器上（透過 minikube 或 Docker Desktop 內建叢集）。但 `ticketing.dev` 是一個**不存在的虛構網域**——沒有任何 DNS 伺服器知道它的 IP 位址。

`hosts` 檔案的作用是：**在作業系統層級覆寫 DNS 查詢結果**，讓瀏覽器對 `ticketing.dev` 的請求被直接導向本機（`127.0.0.1`），而不是真的去網路上查找。

> **這是本地開發多服務應用程式的標準技巧**。在正式 production 環境中，會由真實的 DNS 服務（如 Route 53、CloudFlare 等）處理網域到 IP 的映射，而不需要也不應該修改 hosts 檔案。

#### macOS / Linux 的 hosts 檔案位置

```
/etc/hosts
```

#### Windows 的 hosts 檔案位置

```
C:\Windows\System32\drivers\etc\hosts
```

> **注意**：編輯 `hosts` 檔案需要管理員（Administrator）權限。macOS/Linux 使用 `sudo`，Windows 以系統管理員身份開啟編輯器。

#### 新增的對應規則

在 `hosts` 檔案中新增一行：

```
127.0.0.1 ticketing.dev
```

如此，所有發往 `ticketing.dev` 的流量都會被導向本機，再由本機上的 Ingress Nginx Controller 截獲並路由。

### Chrome 安全警告的由來

當我們第一次訪問 `http://ticketing.dev/api/users/currentuser` 時，Chrome 顯示了「Your connection is not private」錯誤訊息。這是因為：

1. Ingress Nginx Controller **預設啟用了 HTTPS**（基於安全最佳實踐）
2. 但它使用的 SSL 憑證是 **Kubernetes Ingress Controller 自動產生的 Fake Certificate**（並非由受信任的 CA 簽發）
3. Chrome（以及所有主流瀏覽器）只信任由受信任憑證機構（CA）簽發的憑證，不信任 Self-Signed 憑證

> **這是一個完全預期的開發環境現象**，正式 production 環境會使用由 Let's Encrypt 或商業 CA 簽發的真實 SSL 憑證，不會有這個問題。

### 繞過安全警告的正確方法

影片中介紹了一個 Chrome 的「隱藏技巧」：

1. 進入顯示警告的頁面
2. 在鍵盤上直接輸入：`thisisunsafe`（不需要點擊任何輸入框，也不需要按 Enter）
3. 頁面會自動解除警告，成功顯示內容

> 這個技巧僅用於**本地開發環境**。在正式網站上看到這個警告，正確的做法是**立即離開**，不要繼續操作。

### 完整網路路徑總結

從瀏覽器到 Auth Service 的完整網路路徑如下：

```
瀏覽器（發送請求到 ticketing.dev/api/users/currentuser）
    ↓（作業系統查詢 hosts 檔案，發現 ticketing.dev → 127.0.0.1）
本機 Nginx Ingress Controller（監聽本機 Port 80/443）
    ↓（根據 Ingress 規則：host=ticketing.dev, path=/api/users/?(.*)）
auth-srv:3000（Kubernetes ClusterIP Service）
    ↓（selector: app: auth）
auth Pod（運行我們的 Express 應用程式）
    ↓（回應 HTTP 200）
原路徑返回，最終顯示在瀏覽器上
```

### 重要提醒：什麼時候需要重新安裝 Ingress Nginx？

並**不是**每次切換專案都要重新安裝 Ingress Nginx Controller。只有在以下情況才需要重新安裝：

- **完全重置了 Kubernetes 叢集**（例如刪除並重建 minikube）
- **手動刪除了 Ingress Nginx Controller 的 Deployment**

如果在兩個專案之間沒有重置叢集，Ingress Nginx Controller 會持續運行，我們只需要寫入新的 Ingress 路由設定（`ingress-srv.yaml`），Skaffold 會自動將其更新到現有的 Ingress Controller。

---

## 💡 重點摘要

- **`hosts` 檔案**是在本地環境中模擬 DNS 解析的標準做法，將虛構網域對應到本機 IP，讓 Ingress Nginx Controller 能夠截獲瀏覽器的請求。
- **Self-Signed 憑證**導致的安全警告是**開發環境專屬的問題**，在正式 production 環境中不存在（會使用真實 CA 簽發的 SSL 憑證）。
- **`thisisunsafe` 技巧**是在 Chrome 中繞過 Self-Signed 憑證警告的捷徑，**絕對不要**在非本地開發環境的網站上使用。
- Ingress Nginx Controller 在叢集中**持續運行**，不需要每次建新專案都重新安裝——只需要更新它的路由設定即可。
- 完整的網路路徑是：`hosts` 檔案 → Nginx Ingress Controller → Kubernetes Service → Pod → Express App。

## 關鍵字

hosts file, /etc/hosts, ticketing.dev, 127.0.0.1, Self-Signed Certificate, SSL Certificate, Chrome Security Warning, thisisunsafe, Ingress Nginx Controller, Kubernetes ClusterIP Service, DNS Override, Local Development, Production vs Development, Ingress Controller, kubectl apply, Skaffold, nginx ingress class
