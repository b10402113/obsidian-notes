# 解決 CORS 錯誤：為 Express Service 加入 cors 中間件

## 📝 課程概述

本單元處理一個幾乎每個 microservices 開發者都會碰到的經典問題：**CORS 錯誤（CORS Request Error）**。當 React App（`localhost:3000`）嘗試向 Post Service（`localhost:4000`）發送 HTTP 請求時，瀏覽器因為 origin 不同（port 不同）而阻止了這個請求。我們會在 Express Service 端安裝並設定 `cors` 中間件來解決這個問題。

---

## 核心觀念與實作解析

### 為什麼會發生 CORS 錯誤？

瀏覽器的 CORS（Cross-Origin Resource Sharing）政策是瀏覽器的安全機制。當下列條件**任一**成立時，就會觸發 CORS 檢查：

- 請求的 **origin（來源網域）** 不同
- 請求的 **port** 不同
- 請求使用的 **subdomain** 不同

在我們的架構中：
- React App 運行在 `http://localhost:3000`
- Post Service 運行在 `http://localhost:4000`

Port 3000 ≠ Port 4000，因此瀏覽器會阻止從 `:3000` 發向 `:4000` 的請求——除非 `:4000` 的伺服器明確允許這個跨來源請求。

> 瀏覽器是唯一會執行 CORS 檢查的環境。在 server-to-server 的通訊中（例如 Service 之間的 HTTP 呼叫），**不會有 CORS 問題**，因為沒有瀏覽器介入。

---

### 解決方案：在 Server 端設定 CORS 中間件

我們**無法**在瀏覽器端或 React App 端解決這個問題。必須在 Express Service 端進行設定。

#### 安裝 `cors` 套件

```bash
npm install cors
```

#### 在 Express Service 中啟用 cors 中間件

```javascript
const cors = require('cors');
const app = express();

app.use(cors());
```

`cors()` 這個函式會在每個 response 的 header 中自動加入 `Access-Control-Allow-Origin: *`，告知瀏覽器「允許來自任何 origin 的請求」。這對於開發環境來說完全可接受。

---

### ⚠️ 重要：Post Service 與 Comment Service 都必須設定

由於 React App 同時需要呼叫 Post Service 和 Comment Service，**兩個 Service 都必須安裝並啟用 `cors` 中間件**。只要漏掉任何一個，該 Service 的端點就仍然會被瀏覽器阻擋。

---

### 為什麼在 production 環境我們可以避免 CORS？

在後續的大型專案中，我們會採用不同的前端架構（例如：前端靜態檔案與 API Server 部署在同一個 origin 下），所以不需要每個 Service 都各自處理 CORS。這是小型學習專案與 production 架構之間的差異之一。

---

## 💡 重點摘要

- **CORS 是一種瀏覽器安全機制**，server-to-server 通訊不受影響。
- CORS 錯誤只能在 **Server 端** 解決，瀏覽器端和 React App 無能為力。
- `cors()` 中間件透過在 response header 加入 `Access-Control-Allow-Origin` 來解除跨來源限制。
- 在 microservices 架構中，只要有 Service 需要被瀏覽器直接呼叫，**每一個 Service 都必須設定 cors**。
- Production 環境有更好的架構方式來避免這個問題，詳見後續章節。

## 關鍵字

CORS, Cross-Origin Resource Sharing, cors 中間件, Access-Control-Allow-Origin, Browser Security Policy, Express Middleware, localhost:3000 vs localhost:4000, CORS Error Fix
