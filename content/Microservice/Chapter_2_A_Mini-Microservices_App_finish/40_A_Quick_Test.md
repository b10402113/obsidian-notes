# Moderation 功能端對端測試：三种狀態的呈現

## 📝 課程概述

本單元完成 Moderation 功能的端對端測試。我們會驗證三種留言狀態（`approved`、`rejected`、`pending`）在 React 前端的正確呈現，並透過實際「關閉 Moderation Service」來模擬真人審核延遲的場景。這些測試完美驗證了 Option 2 + Option 3 混合模式的運作邏輯。

---

## 核心觀念與實作解析

### 三種留言狀態的 UI 實作

在 CommentList 元件中，根據 `comment.status` 決定顯示內容：

```jsx
let displayContent;
if (comment.status === 'approved') {
  displayContent = comment.content;
} else if (comment.status === 'pending') {
  displayContent = '此留言正在等待審核中';
} else if (comment.status === 'rejected') {
  displayContent = '此留言已被拒絕';
}
```

### 測試一：留言通過審核

提交不含敏感字的留言（如 "This is great!"）→ `status` 為 `approved` → 畫面正常顯示留言內容。

### 測試二：留言被拒絕

提交含敏感字「orange」的留言 → `status` 為 `rejected` → 畫面顯示「此留言已被拒絕」。

### 測試三：模擬 Moderation 延遲（pending 狀態）

**操作步驟**：
1. 在終端機中**停止 Moderation Service**
2. 在瀏覽器中提交留言（此時 Moderation Service 已下線，事件無法被處理）
3. 刷新頁面

**預期結果**：Query Service 已收到 `CommentCreated` 事件（`status: pending`），因此留言正常顯示，但內容為「此留言正在等待審核中」。

> 這正是 Option 2 解決的核心問題：使用者提交留言後，**立馬就能看到自己的留言處於「待審核」狀態**，而不是什麼都看不到。

---

## 💡 重點摘要

- `pending` 狀態的設計讓使用者即使在 Moderation Service 離線時，也能即時看到自己提交的留言處於「等待審核」狀態。
- 三種狀態（`pending` / `approved` / `rejected`）的 UI 邏輯簡單明瞭，但事件驅動的背後流程卻涉及四個 Service 的協調。
- 這個測試也暴露了一個新問題：**當 Moderation Service 恢復上線後，積壓的事件會不會遺失？** 這正是下一個單元要解決的問題。

## 關鍵字

Moderation Test, pending, approved, rejected, CommentList Rendering, Status Display, Offline Moderation, UX Design, Option 2, Event Processing
