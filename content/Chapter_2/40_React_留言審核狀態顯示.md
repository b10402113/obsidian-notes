# React 留言審核狀態顯示

## 📝 課程概述

本單元將 React 前端的 `CommentList` Component 升級，根據留言的 `status` 欄位（`approved`、`rejected`、`pending`）呈現不同的 UI：通過的留言正常顯示內容，被拒絕的留言顯示「此留言已被拒絕」，待審核的留言顯示「此留言正在審核中...」。這是整個 Moderation 功能的「最後一哩路」——讓使用者真正感受到審核結果。

## 核心觀念與實作解析

### CommentList 根據 status 渲染不同內容

```jsx
const CommentList = ({ comments }) => {
  const renderedComments = comments.map(comment => {
    let content;

    if (comment.status === 'approved') {
      content = comment.content;
    } else if (comment.status === 'pending') {
      content = '此留言正在審核中...';
    } else if (comment.status === 'rejected') {
      content = '此留言已被拒絕';
    }

    return <li key={comment.id}>{content}</li>;
  });

  return <ul>{renderedComments}</ul>;
};
```

### 「pending」狀態的意義

`pending` 是一個重要的 UX 設計選擇。當使用者提交留言後馬上重新整理頁面，他**應該馬上看到**自己的留言處於「等待審核」的狀態，而不是頁面空白或404。這意味著：
- Query Service 必須在 Moderation 完成**之前**就收到並儲存這個留言（`pending` 狀態）
- 這正是為什麼 `CommentCreated` 事件由 Event Bus 同時發給 Query Service **和** Moderation Service

### 測試：關閉 Moderation Service 模擬「人工審核延遲」

最真實的測試：直接 kill 掉 Moderation Service（Ctrl+C），然後提交新留言：
1. 留言成功建立（Comments Service 不受 Moderation 影響）
2. Event Bus 將 `CommentCreated` 發給 Query Service → 留言被儲存為 `pending`
3. Moderation Service 掛掉，`CommentModerated` 永遠不會發出
4. 使用者重新整理頁面 → 看到「此留言正在審核中...」

這完美模擬了**人工審核需要數小時**的真實場景——使用者的留言不會消失，只是暫時看不見內容。

### React 元件無需大幅改動

值得注意的是，這個 UI 變更幾乎沒有觸碰後端程式碼——Moderation Service 獨立運作，Comments Service 獨立運作，Query Service 獨立運作，React 只是根據資料的 `status` 欄位呈現不同內容。**新功能的增加幾乎不需要修改現有程式碼**，這正是 Event-Driven 架構的魅力。

## 💡 重點摘要

- **`pending` 狀態**是 UX 設計的關鍵——讓使用者在 Moderation 完成前就知道自己的留言已被系統接收，只是需要時間審核。
- **關閉 Moderation Service 測試**：完美模擬「AI/人工審核延遲」的真實場景，驗證系統的韌性。
- `CommentList` 只根據 `status` 欄位做條件渲染，不涉及任何 API 呼叫——純粹的 Presentation Logic。
- Event-Driven 架構讓 Moderation 新功能的加入，幾乎不需修改任何現有 Service 的程式碼。

## 🔑 關鍵字

`Content Moderation`, `Status Field`, `Pending State`, `UX Design`, `Presentation Logic`
