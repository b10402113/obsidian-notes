# 在 Kubernetes 中執行 Next.js

## 📝 課程概述

本單元將上一堂課在本機端建立的 Next.js 專案完整部署至 Kubernetes 叢集。我們會建立 **Deployment**、**ClusterIP Service**，並更新 **Skaffold** 與 **Ingress NGINX** 設定，讓外部流量能正確路由到 Next.js Pod。

## 核心觀念與實作解析

### 三個必要的設定步驟

將 Next.js 部署到叢集並非只是 `kubectl apply`，你需要完成以下三件事：

1. **建立 `client-deployment.yaml`**：Deployment + ClusterIP Service 的組合，幾乎與 Auth Service 的設定相同，只把所有 `auth` 置換為 `client`
2. **更新 `skaffold.yaml`**：在 `build.artifacts` 中新增 `client` 的建構設定，並將 watch 目標設為 `**/*.js`（而非 `src/**/*`）
3. **更新 `ingress-srv.yaml`**：新增一條 catch-all 路徑，將所有未匹配的路徑導向 `client-svc:3000`

### 為何 client 路徑要放在 Ingress 的最底部？

```
paths:
  - /api/auth/...     → auth-svc
  - /api/ticketing/... → ticketing-svc
  - path: /?           → client-svc  ← 必須在最下面！
```

Ingress NGINX 依序比對路徑規則。如果把 `/?` 放在最上面，幾乎所有請求都會被它攔截，導致 API 請求永遠無法送達後端 Service。**所以 Generic Path（萬用路徑）一定要放在最後。**

### Next.js 容器內的檔案監聽問題

Next.js 運行在 Docker 容器內時，**Webpack 的檔案監聽（watch）機制可能失效**，導致儲存程式碼後頁面未自動更新。解決方案是在 `next.config.js` 中強制設定輪詢間隔：

```javascript
// next.config.js
module.exports = {
  webpackDevMiddleware: (config) => {
    config.watchOptions.poll = 300;
    return config;
  }
};
```

這告訴 Webpack 每 300ms 主動輪詢一次檔案系統，而非依賴 OS 的 inotify/ FSEvents 事件。

> 老師特別提醒：即便做了這個設定，仍然不保證 100% 有效。如果發現程式碼變更未反映，執行 `kubectl delete pod <client-pod-name>` 讓 Deployment 重新建立 Pod 即可。

### Deployment 中的 image 格式（Google Cloud GKE 須知）

若使用 Docker Hub，推送 image 後在 Deployment 中直接寫：
```
image: your-docker-id/client
```

若使用 GKE，須改為 GCR 的格式：
```
image: gcr.io/<your-project-id>/client
```

---

## 💡 重點摘要

- **部署 Next.js 到 K8s 需要三個步驟：Deployment + Service、Skaffold 更新、Ingress 新增路徑。**
- **Ingress 的 catch-all 萬用路徑必須放在所有具體 API 路徑的下面，否則會攔截所有流量。**
- **Next.js 在容器內的熱更新不穩定，可透過 `poll: 300` 緩解；仍無效時手動砍掉 Pod 重建。**
- **不同 Kubernetes 供應商（Docker Hub vs GKE）使用的 image 格式不同。**

## 🔑 關鍵字

client-deployment, ClusterIP Service, Ingress NGINX, Skaffold, next.config.js, poll
