# GitHub Collaborators：多人協作的核心權限設定

## 📝 課程概述

本節介紹如何將其他 GitHub 使用者設定為 Repository 的 Collaborator，讓他們具備 push 權限來共同維護專案。我們會看到完整的邀請流程、Collaborator 與 Owner 的權限差異，以及兩個人如何在實際專案中協作——一人 clone、push，一人 pull、下載對方的變更。

## 核心觀念與實作解析

### Public Repository 不等於可以修改

這是一個常見的誤解。當一個 Repository 設定為 public 時，網路上的任何人確實可以：
- 搜尋並找到這個 Repository
- Clone 到自己的機器上
- 觀看所有程式碼

但他們**絕對無法推送任何變更**。如果任何人都有權限直接修改 Repository，那麼任何人也可以刪除整個專案、推上一個空版本——這顯然是不可接受的安全性漏洞。

因此，「讓別人真正參與協作」與「把 Repository 設為公開」是兩件完全獨立的事。你需要明確地將某人加入為 Collaborator。

### 邀請 Collaborator 的完整流程

身為 Owner（擁有者），在 GitHub 上邀請 Collaborator 的步驟如下：

1. 進入目標 Repository 的 **Settings → Manage access**
2. 點擊 **Invite a collaborator**
3. 輸入對方的 GitHub 使用者名稱或註冊信箱
4. 系統會發送邀請給對方

**關鍵點在於這是一個「雙向同意」的過程**。邀請方（Owner）發出邀請之後，被邀請方必須親自到自己的 email 或 GitHub 通知中點擊「Accept」來接受邀請。這是 GitHub 設計上的安全機制——不能強行把別人加進專案。

接受邀請後，Owner 的 Settings → Manage access 頁面中就會看到該 Collaborator 的名稱出現在列表中。如果需要移除某個 Collaborator，只要勾選該成員的核取方塊，點擊刪除即可。

### Collaborator 與 Owner 的權限邊界

成為 Collaborator 之後，你獲得了**閱讀（read）**與**寫入（write）**的權限。具體來說：

- ✅ 可以 clone 和 pull 這個 Repository
- ✅ 可以 push 新的 commit 到這個 Repository
- ✅ 可以建立、修改、刪除分支
- ❌ **無法**進入 Settings 頁面
- ❌ **無法**刪除整個 Repository
- ❌ **無法**邀請其他人成為 Collaborator
- ❌ **無法**修改其他成員的權限

> 簡單來說，Collaborator 負責「寫 code」，Owner 負責「管專案」。這種分工在大型多人專案中非常重要。

如果你需要更細緻的權限分層（例如讓某人可以管理其他成員、修改專案設定），那就需要使用 **Organization** 或 **Enterprise** 等級的帳戶——在這些方案中可以設定多個 Owner 以及不同層級的團隊權限。

### 實際協作 Workflow：雙人 Clone → Commit → Push → Pull

以下是用兩個帳號模擬實際協作的完整流程。我們把這個過程想像成「兩台隔著地球的電腦在共同維護一個專案」：

#### 角色一（Colt）：建立 Repository 並推送第一個 commit

```bash
mkdir bookish-disco
cd bookish-disco
git init
# 建立一個 artists.txt 檔案並寫入內容
git add artists.txt
git commit -m "initial commit"
git branch -M main                        # 將分支命名為 main
git remote add origin https://github.com/colt/bookish-disco.git
git push origin main
```

#### 角色二（Stevie）：接受邀請後 Clone 並推送變更

```bash
# 先接受 GitHub 發送的 Collaborator 邀請
git clone https://github.com/colt/bookish-disco.git
cd bookish-disco
git log  # 確認可以看到 Colt 的 commit
# 修改 artists.txt 加入更多內容
git add artists.txt
git commit -m "add two artists"
git push origin main  # 將自己的 commit 推送上去
```

#### 角色一（Colt）：Pull 對方的新 commit

```bash
git pull origin main
# Git 回應: Fast-forward ... 表示成功合併了 Stevie 的 commit
git log  # 現在可以看到兩個人的 commit 了
```

### Git Pull 的時機

在協作過程中，**何時該 pull** 是需要建立的直覺。當以下情況發生時，你就需要 pull：

- 你的隊友 push 了新的 commit
- 你準備開始新的工作之前
- 你從別的分支切回來繼續開發時

使用 `git pull origin main` 會將遠端 `origin` 的 `main` 分支上的新 commit 下載並合併到本地的 `main` 分支。當本地落後於遠端時，Git 會執行 **fast-forward** 合併——這是最理想的情況，雙方的工作沒有衝突，Git 只需要把指標往前移。

## 💡 重點摘要

- **Public Repository 與 Collaborator 權限是完全獨立的兩件事**。Public 只解決「誰能看見」的問題，「誰能修改」需要另外設定 Collaborator。
- **邀請 Collaborator 需要雙方同意**：Owner 發送邀請後，對方必須親自到 GitHub 上點擊「Accept」。
- **Collaborator 可以 push 但不能修改專案設定**，也無法刪除 Repository 或管理其他成員——這些權限屬於 Owner。
- 多人協作的基本循環是：各自的機器上 **clone → commit → push/pull**。這是所有協作工作流程的基礎原型。
- Pull 前確保本地沒有尚未推送的變更，否則可能產生合併衝突。

## 關鍵字

Collaborator, Owner, Manage access, Invite collaborator, Accept invitation, push, pull, clone, fast-forward, git branch -M main, git remote add origin, Repository visibility, Permission levels, Organization, Enterprise
