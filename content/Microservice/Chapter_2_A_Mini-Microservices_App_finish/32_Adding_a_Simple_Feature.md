# Moderation 功能需求分析：Option 1 方案及其缺陷

## 📝 課程概述

本單元引入一個新功能需求——**Content Moderation（內容審核）**。留言需要經過審核，系統會根據留言內容是否包含特定敏感字（教學中以「orange」為例）來將留言狀態設為 `approved`、`rejected` 或 `pending`。我們分析了第一種實作方案（Moderation Service 直接控制 Query Service 的資料），並發現它存在一個根本性的 UX 問題。

---

## 核心觀念與實作解析

### Moderation 功能的业务规则

- 留言有三種狀態：`pending`（待審核）、`approved`（通過）、`rejected`（拒絕）
- 如果留言內容包含敏感字，回傳 `rejected`
- 如果留言內容不含敏感字，回傳 `approved`
- `pending` 表示審核尚未完成

### 為什麼不把 Moderation Logic 寫在 React App？

兩大原因：
1. **篩選規則可能頻繁變動**：如果每次修改關鍵字都要重新部署整個 React App，勢必造成巨大的運維負擔
2. **可能由真人審核**：Moderation 不一定只是簡單的字串比對，可能需要人工介入，這就無法在 client 端完成

### Option 1：Moderation Service 直接 emit 事件更新 Query Service

```
User Submit → Comment Service → emit CommentCreated
                                    ↓
                              Moderation Service
                                    ↓
                              emit CommentModerated（攜帶 status）
                                    ↓
                              Query Service 更新留言狀態
```

**Option 1 的致命問題**：Moderation 可能需要很長時間（真人審核可能需要數小時甚至數天）。在 Moderation 完成之前，**Query Service 根本不知道有這筆留言存在**。

後果：使用者提交留言後，立即整理頁面，**什麼都看不到**——因為 Query Service 尚未收到 `CommentModerated` 事件。這是災難性的使用者體驗。

### 留言結構的演進

原本的留言結構：
```javascript
{ id: "comment-id-1", content: "Great post!" }
```

加入 Moderation 之後：
```javascript
{ id: "comment-id-1", content: "Great post!", status: "pending" }
```

`status` 可以是 `"pending"`、`"approved"` 或 `"rejected"`。

### 三種留言狀態的 UI 呈現

| 狀態 | 顯示 |
|------|------|
| `approved` | 正常顯示留言內容 |
| `pending` | 顯示「此留言正在等待審核中」 |
| `rejected` | 顯示「此留言已被拒絕」 |

---

## 💡 重點摘要

- Option 1 的核心問題是：**Query Service 無法立即知道有新留言存在**，導致使用者體驗極差。
- 如果 Moderation 是全自動的字串比對，延遲很小；但如果 Moderation 需要人工介入，延遲可能是數小時到數天。
- 這正是事件驅動架構中常見的問題：**事件的發射與消費之間存在時間差**。

## 關鍵字

Content Moderation, Option 1, Comment Status, pending, approved, rejected, UX Problem, Event-Driven, Moderation Delay, Query Service, CommentModerated Event
