# Container 本質：是程序而非虛擬機

## 📝 課程概述

本單元將打破「Container 就是輕量級虛擬機」的常見誤解。我們將透過實際操作證明 Container 本質上只是 Host 作業系統上的一個**受限程序**，這個理解對於掌握 Docker 的運作原理至關重要。

## 核心觀念與實作解析

### 核心觀念：Container 就是一個 Process

很多人會將 Container 與 Virtual Machine（VM）做比較，但這個比喻其實會造成誤解。**Container 不是 VM，Container 只是在 Host OS 上運行的一個程序**，只是這個程序受到了額外的限制與隔離。

> Container 是一個受限的程序（Restricted Process），而不是一個完整的作業系統實例。
---
![[Pasted image 20260408192724.png|700]]
### 實驗：觀察 Container 程序

讓我們透過實際操作來驗證這個概念。

**啟動一個 MongoDB Container**：

```bash
docker container run -d --name mongo mongo
```

**使用 Docker 命令查看 Container 內的程序**：

```bash
docker container top mongo
```

輸出會顯示 Container 內運行的程序，包含其 PID（Process ID）。

**關鍵實驗：使用 Host 的程序查看工具**

因為 Container 只是 Host 上的一個程序，我們可以直接在 Host 上看到它：

```bash
ps aux | grep mongo
```

你會在 Host 的程序列表中看到 `mongod` 程序！

> 這證明了 Container 內的程序確實存在於 Host 的程序列表中，並非隱藏在某個虛擬環境裡。

---

### Container 與 VM 的關鍵差異

| 特性 | Container | Virtual Machine |
|------|-----------|-----------------|
| 本質 | Host 上的一個程序 | 完整的作業系統實例 |
| 可見性 | 可用 Host 工具直接觀察 | 需進入 VM 才能觀察 |
| 啟動速度 | 毫秒級 | 分鐘級 |
| 資源佔用 | 共用 Host 核心 | 需要完整 Guest OS |

---

### 停止 Container = 停止程序

當我們停止 Container 時：

```bash
docker container stop mongo
```

再次用 `ps aux | grep mongo` 查看，你會發現程序消失了。

**重新啟動**：

```bash
docker container start mongo
```

程序又會重新出現在 Host 的程序列表中。

> Container 就像一個可以被暫停和恢復的程序，而不是一個獨立的虛擬機器。

---

### 安全隔離層

雖然 Container 是 Host 上的程序，但它受到多層安全機制的限制：

- **Namespace**：隔離程序的可見資源（網路、檔案系統、程序等）
- **Cgroups**：限制資源使用量（CPU、記憶體等）
- **Capabilities**：限制程序的特殊權限

> 這些安全層讓 Container 程序的 PID 與其他 Host 程序不同，但仍可被 Host 工具觀察到。

---

## 💡 重點摘要

- **Container 不是 VM，而是 Host OS 上的一個受限程序。**
- **可用 Host 的 `ps` 等工具直接觀察 Container 內的程序。**
- **停止 Container 等於停止該程序，程序會從程序列表中消失。**
- **Container 的安全隔離透過 Namespace、Cgroups、Capabilities 實現。**

## 🔑 關鍵字

Process, Container, VM, Namespace, Cgroups
