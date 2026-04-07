# Ingress 與 HTTP 路由

## 📝 課程概述

本單元介紹 Kubernetes 中的 **Ingress** 機制，解決「如何在同一個叢集上運行多個網站、共享相同 Port（80/443）」的問題。Ingress 讓你能根據 HTTP Host 名稱或路徑，將流量路由到不同的 Container，這是在生產環境中部署多網站服務的關鍵技術。

---

## 核心觀念與實作解析

### 為什麼需要 Ingress？

在 Kubernetes 中，當你有多個網站（例如 `a.com` 和 `b.com`）需要運行在同一個叢集上時，會遇到一個問題：所有網站都需要使用標準的 HTTP/HTTPS Port（80 和 443），但同一個 Port 無法直接綁定到多個 Service。

這就是 **Ingress** 登場的時機。Ingress 是一個 Kubernetes 資源，專門解決 **OSI Layer 7（HTTP）路由** 的問題：

- 根據 **Host 名稱**（如 `a.com` vs `b.com`）
- 或 **URL 路徑**（如 `/api` vs `/web`）

將流量導向正確的 Container。

> **與 Swarm 的差異**：Docker Swarm 原生沒有 Ingress 概念，你需要手動部署一個 Proxy Container 並自行管理路由邏輯。Kubernetes 則內建了 Ingress 機制，讓這件事更標準化。

---

### Ingress Controller 的運作方式

Kubernetes **沒有重新發明 HTTP Proxy**，而是提供了一套機制，讓第三方 Proxy 能夠整合進 Kubernetes：

1. 你在叢集中安裝一個 **Ingress Controller**
2. Ingress Controller 監聽 Kubernetes API 中的 Ingress 資源
3. 根據 Ingress 規則，Controller 自動配置 Proxy 的路由

#### 常見的 Ingress Controller 選項

| Ingress Controller | 特點 |
| --- | --- |
| **Nginx** | 最常見、文件最豐富，建議初學者優先嘗試 |
| **Traefik** | 專為 Container 環境設計，原生支援動態服務發現與 Let's Encrypt |
| **HAProxy** | 效能優異，適合高流量場景 |
| **F5** | 企業級硬體 Load Balancer 的 Controller |
| **Envoy / Istio** | 新一代 Proxy，與 Service Mesh 整合 |

> **講師推薦**：如果你沒有特殊需求，可以從 Nginx 開始。如果想要更現代化的體驗（特別是自動 SSL 憑證管理），Traefik 是很好的選擇。

---

### 如何使用 Ingress？

在應用開發者的 YAML 檔案中，你需要加入 **annotations** 來告訴 Ingress Controller 如何處理你的服務：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: a.com
    http:
      paths:
      - path: /
        backend:
          service:
            name: service-a
            port:
              number: 80
```

Ingress Controller 會根據這些設定：
- 設定路由規則
- 處理 SSL/TLS 憑證（可自動從 Let's Encrypt 取得）
- 將流量轉發到對應的 Service

---

### 重要提醒

- **Ingress 功能在 Kubernetes 1.15 仍為 Beta 狀態**，不同 Controller 的實作可能略有差異
- Ingress **不是** Load Balancer 或 Service，它是獨立的資源類型
- 選擇 Ingress Controller 時，應考慮團隊的技術背景與既有基礎設施

---

## 💡 重點摘要

- **Ingress 解決多網站共享同一組 Port（80/443）的路由問題，根據 HTTP Host 或路徑將流量導向不同 Container。**
- **Kubernetes 透過 Ingress Controller 整合第三方 Proxy（如 Nginx、Traefik），而非重新發明 HTTP Proxy。**
- **Traefik 專為 Container 環境設計，原生支援動態服務發現與 Let's Encrypt 自動憑證管理。**
- **使用 Ingress 時，需在 YAML 中加入 annotations 告訴 Controller 如何路由你的服務。**

---

## 🔑 關鍵字

Ingress, Ingress Controller, Nginx, Traefik, Layer 7
