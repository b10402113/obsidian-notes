# 兩種解決方案與 Ingress NGINX 路由

## 📝 課程概述

本單元提出**兩種**在 `getInitialProps` 中解決「如何對 Ingress NGINX 發送請求」的方案：

- **方案一（推薦）**：讓 Next.js Pod 向 Ingress NGINX Pod 發送請求（透過 `http://ingress-nginx.ingress-nginx.svc.cluster.local`）
- **方案二**：讓 Next.js 直接呼叫 Service（如 `http://auth-svc:3000`）

老師詳細分析了兩種方案的優缺點，並說明**跨 Namespace 呼叫**的完整語法。

## 核心觀念與實作解析

### 為何方案二不推薦？

直接呼叫 Service（如 `http://auth-svc:3000`）看似簡單，但有以下問題：

1. **React Client Code 必須知道每個 Service 的精確名稱**
2. **還要另外知道每個 URL 對應哪個 Service**（例如 `/api/users/*` 對應 auth-svc、`/api/tickets/*` 對應 ticketing-svc）
3. 未來增加新 Service 時，前端程式碼必須跟著修改

> 所有的路由邏輯**已經寫在 Ingress NGINX 設定檔裡**，直接呼叫 Service 等於**重複造輪子**。

### 方案一：向 Ingress NGINX 發送請求

好處是：**前端程式碼永遠只寫路徑**（如 `/api/users/currentuser`）， Ingress NGINX 會根據既有規則自動轉送。

但挑戰在於：**如何從 Client Pod 呼叫到位於 `ingress-nginx` Namespace 的 Ingress NGINX？**

### 跨 Namespace 呼叫的語法

在同一 Namespace 內呼叫 Service：`http://auth-svc`（縮寫）

跨 Namespace 呼叫時，必須寫完整格式：

```
http://<service-name>.<namespace>.svc.cluster.local
```

實際例子：
```
http://ingress-nginx.ingress-nginx.svc.cluster.local
```

執行 `kubectl get svc -n ingress-nginx` 可確認 Service 名稱。

### Cookie 轉發的另一個挑戰

即使能成功呼叫 Ingress NGINX，還有一個問題：**原始請求的 Cookie Header 必須轉發下去**，否則 Auth Service 收到的請求是匿名請求（`currentUser` 會是 `null`）。

---

## 💡 重點摘要

- **直接呼叫 Service（方案二）會讓前端綁定特定 Service 名稱，失去靈活性；方案一復用 Ingress NGINX 的路由規則是更好的設計。**
- **跨 Namespace 呼叫 Service 的完整格式為：`http://<svc>.<namespace>.svc.cluster.local`。**
- **`/api/users/currentuser` 這類路徑只是告訴 Ingress NGINX「要往哪裡路由」，並非直接指定目的地。**

## 🔑 關鍵字

Ingress NGINX, 跨 Namespace, svc.cluster.local, 路由集中化, Cookie 轉發, 方案比較
