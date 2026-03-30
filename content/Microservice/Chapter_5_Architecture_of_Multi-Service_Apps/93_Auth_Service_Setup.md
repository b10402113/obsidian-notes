# Auth Service 初始建置：專案架構與 Kubernetes 部署設定

## 📝 課程概述

本單元正式啟動票券系統的實作，從頭建立第一個 microservice——**Auth Service**。我們將完成從 npm 初始化、TypeScript 設定、基本 Express 應用程式骨架，到撰寫 Dockerfile 並成功建構 Docker Image，最後再設定對應的 Kubernetes Deployment 與 ClusterIP Service。這是「邊開發邊建立基礎設施」策略的起點，讓每一個微服務在上線之初就具備可部署至 Kubernetes 的完整配置。

## 核心觀念與實作解析

### Auth Service 的職責與路由設計

Auth Service 負責整個應用程式的使用者身份驗證，未來會實作四條主要路由：

| 方法 | 路由 | 說明 |
|------|------|------|
| POST | `/auth/signup` | 使用者註冊 |
| POST | `/auth/signin` | 使用者登入 |
| POST | `/auth/signout` | 使用者登出 |
| GET | `/auth/currentuser` | 取得目前登入者資訊 |

這四條路由覆蓋了身份驗證的完整生命週期，後續課程會陸續實作它們。

### 初始專案建立流程

#### 1. 目錄結構與 npm 初始化

```
workspace/
└── ticketing/         ← 整個專案的根目錄
    └── auth/          ← Auth Service 所在目錄
        ├── src/
        │   └── index.ts
        └── package.json
```

在 `ticketing/auth` 目錄中執行 `npm init -y` 來建立 `package.json`，作為這個 Service 的起點。

#### 2. 安裝初始依賴

```bash
npm install typescript @types/node @types/express ts-node-dev express
```

- **TypeScript**：全專案採用強型別，必須第一個安裝。
- **ts-node-dev**：開發環境專用的熱重載執行工具，修改程式碼後會自動重啟服務。
- **express** + **@types/express**：Web 框架與其 TypeScript 型別定義。

#### 3. TypeScript 初始化

在 `auth` 目錄執行 `tsc --init`，產生 `tsconfig.json`。這讓 TypeScript 編譯器知道如何處理 `.ts` 檔案、輸出目標等設定。

#### 4. 基本 Express 骨架

`src/index.ts` 的最簡起點：

```typescript
import express from 'express';

const app = express();

app.use(express.json());

app.listen(3000, () => {
  console.log('Listening on port 3000');
});
```

> **注意**：這裡使用 `port 3000` 只是暫定值。一旦進入 Kubernetes 環境，真正的連接埠由 **Kubernetes Service** 管理，Service 的 `selector` 會將流量導向正確的 Pod，所以我們不需要過度在意應用程式層級的連接埠號。

#### 5. 設定 npm start 腳本

在 `package.json` 中新增：

```json
"scripts": {
  "start": "ts-node-dev --rs --src/index.ts"
}
```

- `--rs`：全名 `--respawn`，表示熱重載模式，程式碼變更後自動重啟。

---

### Dockerfile 的撰寫

在 `auth/` 目錄中新增 `Dockerfile`，將 Service 包裝成 Docker Image：

```dockerfile
FROM node:alpine

WORKDIR /app

COPY package.json ./
RUN npm install

COPY . .

CMD ["npm", "start"]
```

同時建立 `.dockerignore` 檔案，避免 `node_modules` 被複製進 Image：

```
node_modules
```

> **為什麼要先複製 `package.json` 再安裝依賴，最後才複製其余檔案？**
> 這是 Docker 層級的最佳實踐（Multi-stage 的精神）。這樣做可以確保當只有原始碼改變（而 `package.json` 不變）時，Docker 可以直接使用已快取的 `npm install` 層，大幅加速後續建構速度。

---

### Kubernetes Deployment 與 Service 設定

#### 為什麼現在就設定 Kubernetes？

講師採用「**邊開發邊建立基礎設施**」的策略，而不是先把所有 Service 實作完再來設定 Kubernetes。這樣做的好處是：
- 每一個 Service 從第一天起就具備可部署的狀態
- 可以在本地使用 `skaffold` 即時同步程式碼變更到遠端叢集
- 開發流程與部署流程無縫接軌

#### Deployment 設定（`infra/k8s/auth-depl.yaml`）

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-depl
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth
  template:
    metadata:
      labels:
        app: auth
    spec:
      containers:
        - name: auth
          image: your-docker-id/auth
```

關鍵點：
- `replicas: 1`：目前只運行一個 Pod，production 環境可按需擴展。
- `selector.matchLabels` 告訴 Deployment **如何找到它所管理的 Pod**。
- `template.metadata.labels` 是每個新 Pod 會帶上的標籤。

#### Service 設定（寫在同一個檔案中，用 `---` 分隔）

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: auth-srv
spec:
  selector:
    app: auth
  ports:
    - name: auth
      protocol: TCP
      port: 3000
      targetPort: 3000
```

- **ClusterIP Service（預設類型）**：這個 Service 只能在 Kubernetes 叢集**內部**被存取，外部（瀏覽器）無法直接訪問。我們稍後會透過 Ingress Nginx 來暴露外部流量。
- `selector: app: auth`：找到所有標籤為 `app: auth` 的 Pod。
- `port: 3000`：Service 對外監聽的連接埠。
- `targetPort: 3000`：轉發到 Pod 內部容器的連接埠。

---

## 💡 重點摘要

- Auth Service 的四條核心路由（Sign Up / Sign In / Sign Out / Current User）構成了整個系統的身份驗證基礎。
- Docker 建構時**先複製 `package.json` 再 `npm install`**，是讓 Image 層級快取發揮效益的標準做法。
- Kubernetes Deployment 負責管理 Pod 的生命週期，Kubernetes Service（ClusterIP）則負責讓叢集內的其他服務能夠找到並存取這個 Deployment。
- 採用「邊開發邊設定 Kubernetes」的策略，確保每個 Service 從一開始就是可部署的狀態。

## 關鍵字

Auth Service、TypeScript、ts-node-dev、Dockerfile、node:alpine、.dockerignore、Kubernetes Deployment、ClusterIP Service、matchLabels、selector、targetPort、skaffold、Infrastructure as Code
