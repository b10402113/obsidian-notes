# Event Bus 設計：手工打造第一個事件轉發中心

## 📝 課程概述

本單元從理論進入實作，帶你用 Express + Axios 手工實作一個最簡化的 Event Bus。我們會了解 Event Bus 的核心職責——**接收來自任一 Service 的事件，並將其原封不動地廣播給所有其他已連接的 Service**。雖然這只是個極度簡化的練習版本，但足以讓你透徹理解事件驅動架構背後的資料流動方式。

---

## 核心觀念與實作解析

### 業界常見的 Event Bus 實作

常見的 Event Bus / Message Broker 選項包括：

| 名稱 | 特色 |
|------|------|
| **RabbitMQ** | 功能豐富，支援複雜的 routing 規則 |
| **Apache Kafka** | 高吞吐量，適合大規模資料流處理 |
| **NATS** | 極簡、高效能、容易上手（這門課的最終選擇） |

> 這門課的迷你專案我們自己刻 Event Bus，是為了**透徹理解背後的運作原理**。後續的大型專案，會使用 NATS Streaming Server（production-grade 解決方案）。

### Event 的結構：Type + Data

我們約定所有事件的格式為：

```javascript
{
  type: 'PostCreated',       // 描述發生了什麼事件
  data: { id, title }        // 與事件相關的資料
}
```

`type` 是事件的類型名稱；`data` 是携带的具體資料。事件結構完全由你自己定義，沒有強制規範。

### Event Bus 的工作流程

```
Post Service ──POST /events──→ Event Bus
                                   │
                 ┌─────────────────┼─────────────────┐
                 ↓                 ↓                 ↓
            [Post Service]   [Comment Service]  [Query Service]
              (自己發的不理)    (收到後處理)       (收到後處理)
```

Event Bus 收到事件後，會將**同樣的資料**分別 POST 給所有已連接的 Service——**包括原本發送事件的 Service 自己**。

### 手工實作 Event Bus

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const axios = require('axios');

const app = express();
app.use(bodyParser.json());

app.post('/events', (req, res) => {
  const event = req.body;

  // 廣播給所有 Service（包括自己）
  axios.post('http://localhost:4000/events', event);  // Post Service
  axios.post('http://localhost:4001/events', event);  // Comment Service
  axios.post('http://localhost:4002/events', event);  // Query Service

  res.send({ status: 'OK' });
});

app.listen(4005, () => {
  console.log('Listening on 4005');
});
```

**設計細節說明**：
- Event Bus 本身也是一個 Express 應用程式，Listen on Port 4005
- 它提供一個 `POST /events` endpoint 接收事件
- 收到事件後，立即向所有已知的 Service 發送相同的 POST 請求（使用 Axios）
- 所有 Service 都需要實作對應的 `POST /events` 端點，否則會收到 404 錯誤

### ⚠️ 這個實作的局限性

1. **沒有錯誤處理**：如果某個 Service 沒有正常運行，Event Bus 發出的 POST 會失敗，但 Event Bus 目前假設所有請求都會成功
2. **需要手動維護 Service 清單**：每次新增 Service，都要修改 Event Bus 的程式碼
3. **沒有重試機制**：如果事件發送失敗，事件就永遠遺失了

這些問題在 production 環境中是無法接受的，所以後續的大型專案會使用 NATS 這類更成熟的解決方案。

---

## 💡 重點摘要

- Event Bus 的核心職責只有一個：**接收事件，原封不動地廣播給所有已連接的 Service**。
- 事件的結構完全由你自己定義，業界沒有強制標準——但保持 `type + data` 的一致性格式是好習慣。
- 我們的 Event Bus 目前沒有錯誤處理、重試機制或可靠的訊息傳遞保證——這是簡化版的限制。
- Event Bus 在概念上非常簡單，但在 production 環境中，要做到**可靠的事件傳遞**其實是相當複雜的工程問題。

## 關鍵字

Event Bus, Message Broker, RabbitMQ, Kafka, NATS, Event Broadcasting, POST /events, Express Event Bus, Service Discovery, Event Structure, Type + Data
