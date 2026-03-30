# Skaffold 開發流程與 Ingress 路由設定

## 📝 課程概述

本單元將 Skaffold 整合進票券系統的開發流程，讓每次對程式碼或 Kubernetes 設定檔的修改，都能自動同步到叢集中的 Pod。同時，我們設定了 Ingress Nginx 路由規則，讓外部流量（瀏覽器）可以透過一個模擬的本地網域（`tickets.dev`）訪問叢集中的 Auth Service。這套工作流程讓開發體驗與正式部署環境高度一致，奠定了後續所有服務開發的基礎。

## 核心觀念與實作解析

### Skaffold 的核心價值

Skaffold 是一個專為 Kubernetes 開發設計的工具，它解決了「開發階段如何高效地與叢集互動」的問題。

傳統開發 vs. Skaffold 開發：
- **傳統**：修改程式碼 → 手動重新 build Docker Image → 推送至 registry → 重新部署到 Kubernetes → 才能測試。
- **Skaffold**：修改程式碼 → Skaffold 自動偵測變更 → 直接同步檔案到運行中的 Container → 即時看到結果。

> Skaffold 的 watch 機制讓我們在**本地修改 TypeScript 檔案**，就可以在遠端 Kubernetes Pod 中看到即時更新，大幅縮短開發反饋週期。

### `skaffold.yaml` 完整設定解析

```yaml
apiVersion: skaffold/v2alpha3
kind: Config

deploy:
  kubectl:
    manifests:
      - ./infra/k8s/*

build:
  local:
    push: false
  artifacts:
    - image: your-docker-id/auth
      context: auth
      dockerfile: Dockerfile
      sync:
        manual:
          - src: "src/**/*.ts"
            dest: "."
```

#### `deploy.manifests`

指定 Skaffold 要監控並部署到叢集的 Kubernetes 設定檔位置。這裡使用 `infra/k8s/*`，所以 `infra/k8s/` 目錄下所有 `.yaml` 檔案（Deployment、Service、Ingress 等）一旦變更，Skaffold 都會自動重新套用到叢集。

#### `build.local.push: false`

明確告訴 Skaffold：**不要把建好的 Image 推送到 Docker Hub 或任何 remote registry**，因為我們是在本地叢集（minikube 或 Docker Desktop 內建叢集）上開發，Image 只需要存在本地就夠了。

#### `build.artifacts[].sync`

`sync.manual` 設定了**檔案同步規則**：
- `src: "src/**/*.ts"`：監控 `auth/src/` 目錄下所有 TypeScript 檔案的變更。
- `dest: "."`：將變更後的檔案同步到 Container 內的 `/app` 目錄（因為 `WORKDIR` 設為 `/app`）。

> 這裡的同步是**直接覆蓋**，而非重新編譯。若同步後 `ts-node-dev` 沒有偵測到重啟訊號，是因為 sync 跳過了 Dockerfile 的 rebuild 流程——這在多數情況下是預期行為。

---

### 測試路由：搶先驗證 Auth Service 可達

在設定 Ingress 之前，講師先建立一個**搶先版**的 GET 路由，確保能從瀏覽器發送請求到 Auth Service：

```typescript
app.get('/api/users/currentuser', (req, res) => {
  res.send('Hi there');
});
```

這個路由的存在不僅是測試用途，更是確保整條網路路徑暢通無阻的驗證點。

---

### Ingress Nginx 路由設定（`infra/k8s/ingress-srv.yaml`）

#### 為什麼是 Ingress？

在 Kubernetes 中，**ClusterIP Service 只能被叢集內部的其他 Service 存取**。瀏覽器在叢集外部，必須透過 Ingress（本質上是 Nginx）才能將流量路由進來。

#### 設定檔重點說明

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-service
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  rules:
    - host: ticketing.dev
      http:
        paths:
          - path: /api/users/?(.*)
            backend:
              serviceName: auth-srv
              servicePort: 3000
```

**Annotations 的意義：**
- `kubernetes.io/ingress.class: nginx`：指定這個 Ingress 由 Nginx Ingress Controller 處理（而非其他如 Traefik、GCE 等）。
- `nginx.ingress.kubernetes.io/use-regex: "true"`：允許路徑中使用正則表達式（如 `/?(.*)`），讓 `/api/users`、`/api/users/`、或 `/api/users/anything` 都能匹配。

**路徑的正則表達式：**
`/api/users/?(.*)` 中：
- `/?` 可選的斜線
- `(.*)` 捕獲剩餘所有路徑

**Host 的意義：**
`ticketing.dev` 是一個**虛構的本地網域**，必須配合本機的 `/etc/hosts` 檔案，將 `tickets.dev` 指向 `127.0.0.1`（或叢集所在的 IP）。這讓瀏覽器對 `tickets.dev` 的請求被本機的 Nginx Ingress Controller 截獲，再路由到叢集內部。

#### 更新本機 Host 檔案

最後一步是在 `/etc/hosts` 新增一行：

```
127.0.0.1 ticketing.dev
```

這樣瀏覽器對 `http://ticketing.dev/api/users/currentuser` 的請求，會被導向本機的 Ingress Controller，Ingress 再根據規則轉發到 `auth-srv:3000`。

---

### 完整的本地開發流程

```
修改 src/index.ts
    ↓
Skaffold 偵測到檔案變更
    ↓
同步 .ts 檔案到 Pod 的 /app 目錄
    ↓
ts-node-dev 熱重載，自動重啟 Node 程式
    ↓
終端機顯示更新後的 console.log 輸出
```

```
瀏覽器 → http://ticketing.dev/api/users/currentuser
    ↓
Ingress Nginx Controller（截獲）
    ↓
路由到 auth-srv:3000
    ↓
Auth Pod（Express）回應 "Hi there"
```

---

## 💡 重點摘要

- `skaffold.yaml` 中的 `push: false` 告訴 Skaffold 不要推送 Image，因為本地開發無需 remote registry。
- `sync.manual` 讓 Skaffold 直接同步 TypeScript 原始碼到運行中的 Container，跳過 Image rebuild，大幅加速開發迭代。
- Ingress Nginx 是外部流量進入 Kubernetes 叢集的唯一入口，必須為每個 Service 定義對應的路由規則。
- `use-regex: "true"` 註解讓路徑可以使用正則表達式，適合 RESTful 風格的 API 路由設計。
- `/etc/hosts` 的 `ticketing.dev` 設定，讓本機瀏覽器可以透過自訂網域對 Ingress Controller 發出請求。

## 關鍵字

Skaffold、skaffold.yaml、Hot Reload、Sync Manual、Local Build、Ingress Nginx、ClusterIP Service、Ingress Controller、use-regex、Host File、ticketing.dev、kubernetes.io/ingress.class: nginx、Kubernetes Development Workflow、Hot Code Reload
