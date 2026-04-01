# MongoDB URI 環境變數化 — 避免連線到錯誤資料庫

## 📝 課程概述

本節解決了一個非常實際的問題：在複製 Auth Service 程式碼時，`index.ts` 中 MongoDB 連線 URL 是硬編碼的，導致 Tickets Service 啟動後連到了 Auth 的 MongoDB 而非自己的 MongoDB。解決方案是將連線字串提取到 `MONGO_URI` 環境變數，並在 Deployment YAML 中定義它，同時在程式碼中加入強制檢查，確保未定義時直接拋錯。

## 核心觀念與實作解析

### 問題的根本原因

從 Auth Service 複製程式碼時，`index.ts` 中的 MongoDB URL 是這樣的：

```typescript
mongoose.connect('mongodb://auth-mongo-srv:27017/auth', { ... })
```

這個字串被原封不動帶到 Tickets Service，但 `auth-mongo-srv` 是 Auth 的 MongoDB，Tickets 根本沒有自己的 Service 叫這個名字。更危險的是，如果名稱恰好不存在但仍能連線（也許 DNS 解析到了別的資料庫），開發者可能完全沒發現自己寫入的是錯誤的資料庫。

### 解決方案：環境變數

**Deployment YAML 中的環境變數設定：**

```yaml
env:
  - name: MONGO_URI
    value: "mongodb://tickets-mongo-srv:27017/tickets"
```

**`index.ts` 中的程式碼：**

```typescript
if (!process.env.MONGO_URI) {
  throw new Error('MONGO_URI must be defined');
}
mongoose.connect(process.env.MONGO_URI!);
```

### 為何可以直接把 URL 寫在 Deployment YAML 中？

這個 URL 只在 Kubernetes 叢集內部有意義，叢集外部的人無法解析 `tickets-mongo-srv` 這個 DNS 名稱。更關鍵的是，這個 MongoDB **沒有任何認證**（沒有 username / password），因此即使有人看到這個 URL，也無法從叢集外部連線。如果 MongoDB 需要帳密，就應該使用 Kubernetes Secret 來存放（就像 `JWT_KEY` 一樣）。

### 加入啟動檢查的好處

`if (!process.env.MONGO_URI)` 這個檢查聽起來多餘，但非常關鍵：

- **部署前就能發現問題**：如果忘記設定環境變數，Pod 啟動時會立即 crash，不會進入「看起來正常但連錯資料庫」的隱性錯誤狀態。
- **及時暴露複製錯誤**：一旦從別的 service 複製程式碼後忘記更新，馬上就會被這行檢測到。

## 💡 重點摘要

- 複製其他 service 的程式碼後，務必檢查並更新所有硬編碼的連線字串。
- 將 MongoDB URI 放到環境變數中是標準做法，搭配 Deployment YAML 的 `env` 區塊即可。
- 沒有認證的 MongoDB URL 寫在 YAML 中是安全的（只要叢集外部無法解析內部 DNS）。
- 在 `mongoose.connect()` 前加入環境變數存在性檢查，能及時發現部署錯誤而非隱藏在執行期。

## 🔑 關鍵字

MONGO_URI, Kubernetes Secret, mongoose.connect, Environment Variable, Deployment YAML
