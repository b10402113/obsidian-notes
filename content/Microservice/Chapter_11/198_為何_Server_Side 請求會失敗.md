# 為何 Server-Side 請求會失敗？

## 📝 課程概述

本單元揭示了整個 Chapter 11 最核心、也最容易被忽視的問題：當在 `getInitialProps`（伺服器端）嘗試向 Auth Service 發送請求時，`axios.get('/api/users/currentuser')` 會得到 `Connection refused localhost:80` 錯誤。本節深入剖析**為何**會發生這個錯誤。

## 核心觀念與實作解析

### 客戶端請求為何能成功？

在 Component 內部發送 Axios 請求時：
- 瀏覽器看到 URL `/api/users/currentuser` 沒有指定網域
- 瀏覽器自動補上當前網域 → `ticketing.dev/api/users/currentuser`
- `ticketing.dev` 由 hosts 檔案解析到 `127.0.0.1:80`
- 請求進入 Ingress NGINX，正確路由至 Auth Service
- **成功**

### 伺服器端請求為何失敗？

當在 `getInitialProps` 內使用 Axios 時，**Node.js 的 HTTP 層接手處理網路請求**：

- Axios 嘗試對 `/api/users/currentuser` 發送請求
- Node.js HTTP 層自動補上 `localhost:80`（容器內的 localhost）
- **容器內的 Port 80 根本沒有任何服務在監聽**
- `Connection refused` → **失敗**

> 關鍵洞察：`127.0.0.1` 在**客戶端**指的是你自己的電腦主機；但在**容器內**指的是**那個容器本身**的網路空間，兩個是完全不同的上下文。

### 根本原因：容器網路的隔離性

```
Client Pod（容器內部）
  localhost → 容器自身（Port 80 沒有任何東西）
  └── 請求永遠無法離開容器

真正能轉發請求的，是 Ingress NGINX（運行在另一個 Pod）
```

這就是為什麼本機端開發環境（請求直接發給本機的 Ingress NGINX）能work，但部署到 K8s 之後在 SSR 階段發出的請求會失敗。

---

## 💡 重點摘要

- **在 Node.js（伺服器端）發出 HTTP 請求時，`localhost` 指的是容器本身，而非宿主機或 Ingress NGINX Pod。**
- **瀏覽器發請求時 `localhost` 補的是宿主機，再由 hosts 檔案導向 Ingress NGINX；兩者上下文完全不同。**
- **容器之間的網路是隔離的，容器內的 `localhost` 永遠只指向容器自己。**

## 🔑 關鍵字

Connection refused, localhost, 容器網路隔離, Node.js HTTP, getInitialProps, 跨 Pod 請求
