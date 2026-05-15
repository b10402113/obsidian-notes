# PersistentVolume + NFS 網絡共享存儲測驗

**科目：** Kubernetes 入門課程
**難度：** 混合（簡單 30%、中等 50%、困難 20%）
**題數：** 25 題
**建議時間：** 40 分鐘

---

## 第一部分：選擇題（每題 4 分，共 40 分）

**Q1.** 在 Kubernetes 中，為什麼使用 HostPath 作為持久化存儲不是特別實用？

A) HostPath 會導致數據丟失
B) HostPath 存儲卷只能在本機使用，而 Pod 經常會在集群裡「漂移」
C) HostPath 不支持讀寫操作
D) HostPath 需要額外的付費授權

<details>
<summary>答案</summary>
<b>B</b> - HostPath 存儲卷只能在本機使用，而 Kubernetes 裡的 Pod 經常會在集群裡「漂移」，所以這種方式不是特別實用。要讓存儲卷真正能被 Pod 任意掛載，需要使用網絡存儲。
</details>

---

**Q2.** NFS（Network File System）採用的是什麼架構？

A) Peer-to-Peer 架構
B) Client/Server 架構
C) Master-Slave 架構
D) Microservices 架構

<details>
<summary>答案</summary>
<b>B</b> - NFS 採用的是 Client/Server 架構，需要選定一台主機作為 Server，安裝 NFS 服務端；其他要使用存儲的主機作為 Client，安裝 NFS 客戶端工具。
</details>

---

**Q3.** 在 Ubuntu 系統中，安裝 NFS 服務端應該使用哪個命令？

A) `sudo apt -y install nfs-common`
B) `sudo apt -y install nfs-server`
C) `sudo apt -y install nfs-kernel-server`
D) `sudo apt -y install nfs-client`

<details>
<summary>答案</summary>
<b>C</b> - 在 Ubuntu 系統裡安裝 NFS 服務端使用命令：`sudo apt -y install nfs-kernel-server`。選項 A 是安裝 NFS 客戶端工具的命令。
</details>

---

**Q4.** 配置 NFS 訪問共享目錄時，需要修改哪個配置文件？

A) `/etc/nfs.conf`
B) `/etc/exports`
C) `/etc/fstab`
D) `/etc/network/interfaces`

<details>
<summary>答案</summary>
<b>B</b> - 配置 NFS 訪問共享目錄需要修改 `/etc/exports` 文件，指定目錄名、允許訪問的網段，還有權限等參數。
</details>

---

**Q5.** 在 NFS 的 PV 定義中，為什麼 accessModes 可以設置成 ReadWriteMany？

A) 這是 Kubernetes 的默認設置
B) 因為 NFS 存儲空間足夠大
C) 這是由 NFS 的特性決定的，它支持多個節點同時訪問一個共享目錄
D) 為了提高存儲性能

<details>
<summary>答案</summary>
<b>C</b> - accessModes 可以設置成 ReadWriteMany，這是由 NFS 的特性決定的，它支持多個節點同時訪問一個共享目錄。
</details>

---

**Q6.** 在 NFS PV 的 YAML 定義中，哪個字段必須指定 NFS 服務器的 IP 地址和共享目錄名？

A) `spec.nfs`
B) `spec.hostPath`
C) `spec.persistentVolume`
D) `spec.storage`

<details>
<summary>答案</summary>
<b>A</b> - 因為這個存儲卷是 NFS 系統，所以需要在 YAML 裡添加 `spec.nfs` 字段，指定 NFS 服務器的 IP 地址和共享目錄名。
</details>

---

**Q7.** 如果 PV 處於 "pending" 狀態無法使用，最可能的原因是什麼？

A) PV 的名稱不符合規範
B) Kubernetes 集群資源不足
C) spec.nfs 裡的 IP 地址不正確或路徑不存在
D) PVC 沒有正確配置

<details>
<summary>答案</summary>
<b>C</b> - 如果 spec.nfs 裡的 IP 地址不正確，路徑不存在（事先沒有創建好），Kubernetes 按照 PV 的描述會無法掛載 NFS 共享目錄，PV 就會處於 "pending" 狀態無法使用。
</details>

---

**Q8.** 關於動態存儲卷，以下哪個說法是正確的？

A) 動態存儲卷需要管理員手動創建每個 PV
B) 動態存儲卷通過 Provisioner 自動創建 PV
C) 動態存儲卷不支持 NFS 存儲
D) 動態存儲卷比靜態存儲卷更難管理

<details>
<summary>答案</summary>
<b>B</b> - 動態存儲卷使用 StorageClass 綁定一個 Provisioner 對象，Provisioner 能夠自動管理存儲、創建 PV，代替了原來系統管理員的手工勞動。
</details>

---

**Q9.** NFS Provisioner 在 Kubernetes 中以什麼形式運行？

A) 以 DaemonSet 形式運行
B) 以 Service 形式運行
C) 以 Pod 形式運行
D) 以 ConfigMap 形式運行

<details>
<summary>答案</summary>
<b>C</b> - NFS Provisioner 也是以 Pod 的形式運行在 Kubernetes 裡。
</details>

---

**Q10.** 在 StorageClass 的 YAML 定義中，哪個字段指定了應該使用哪個 Provisioner？

A) `metadata.name`
B) `provisioner`
C) `parameters`
D) `storageClassName`

<details>
<summary>答案</summary>
<b>B</b> - YAML 裡的關鍵字段是 `provisioner`，它指定了應該使用哪個 Provisioner。
</details>

---

## 第二部分：是非題（每題 3 分，共 15 分）

**Q11.** NFS 是一個相對較新的網絡存儲系統，在 Linux 系統中還未成為標準配置。_(是/否)_

<details>
<summary>答案</summary>
<b>否</b> - NFS 有著近 40 年的發展歷史，基本上已經成為了各種 UNIX 系統的標準配置，Linux 自然也提供對它的支持。
</details>

---

**Q12.** 在 Kubernetes 集群中，只需要在 Master 節點上安裝 NFS 客戶端即可。_(是/否)_

<details>
<summary>答案</summary>
<b>否</b> - 為了讓 Kubernetes 集群能夠訪問 NFS 存儲服務，需要在每個節點上都安裝 NFS 客戶端。
</details>

---

**Q13.** 使用 NFS 網絡存儲時，數據的持久化不受 Pod 調度位置的影響。_(是/否)_

<details>
<summary>答案</summary>
<b>是</b> - 因為 NFS 是一個網絡服務，不會受 Pod 調度位置的影響，所以只要網絡通暢，這個 PV 對象就會一直可用，數據也就實現了真正的持久化存儲。
</details>

---

**Q14.** 靜態存儲卷和動態存儲卷可以同時在 Kubernetes 集群中使用。_(是/否)_

<details>
<summary>答案</summary>
<b>是</b> - 有了「動態存儲卷」的概念，前面講的手工創建的 PV 就可以稱為「靜態存儲卷」，兩者可以在同一集群中共存使用。
</details>

---

**Q15.** 使用動態存儲卷時，需要手動定義 PV 對象。_(是/否)_

<details>
<summary>答案</summary>
<b>否</b> - 動態存儲卷不需要手工定義 PV 對象，而是要定義 StorageClass，由關聯的 Provisioner 自動創建 PV 完成綁定。
</details>

---

## 第三部分：填充題（每題 4 分，共 20 分）

**Q16.** NFS 採用的是 **\_\_\_\_** 架構，需要選定一台主機作為 Server，安裝 NFS 服務端；其他要使用存儲的主機作為 Client，安裝 NFS 客戶端工具。

<details>
<summary>答案</summary>
<b>Client/Server</b>
</details>

---

**Q17.** 在 NFS 服務器配置中，修改 `/etc/exports` 後，需要使用 **\_\_\_\_** 命令通知 NFS 讓配置生效。

<details>
<summary>答案</summary>
<b>exportfs -ra</b>
</details>

---

**Q18.** 在 Kubernetes 中，**\_\_\_\_** 存儲系統更適合數據持久化，因為它不會受 Pod 調度位置的影響。

<details>
<summary>答案</summary>
<b>網絡</b> - 網絡存儲系統（如 NFS）更適合數據持久化，因為只要網絡通暢，數據就不受 Pod 調度位置影響。
</details>

---

**Q19.** 在動態存儲卷中，**\_\_\_\_** 對象能夠自動管理存儲、創建 PV，代替了系統管理員的手工勞動。

<details>
<summary>答案</summary>
<b>Provisioner</b>
</details>

---

**Q20.** 部署 NFS Provisioner 時需要修改三個 YAML 文件，分別是 rbac.yaml、class.yaml 和 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>deployment.yaml</b>
</details>

---

## 第四部分：簡答題（每題 8 分，共 16 分）

**Q21.** 請說明動態存儲卷相比靜態存儲卷有什麼優勢？是否存在缺點？

<details>
<summary>答案</summary>

**動態存儲卷的優勢：**

- **自動化管理**：不需要管理員手動創建每個 PV，Provisioner 可以自動根據 PVC 需求創建合適的 PV
- **按需分配**：存儲空間根據實際需求動態分配，避免預先分配造成的浪費
- **減少人為錯誤**：自動化過程減少了人工操作可能帶來的配置錯誤
- **適合大規模環境**：在需要大量 PV 的場景下，大大降低了管理員的工作負擔
- **解耦存儲實現**：通過 StorageClass 統一接口，讓開發者專注於需求，存儲細節由各類 Provisioner 處理

**動態存儲卷的缺點：**

- **資源管控挑戰**：申請的空間可能大於實際需求，造成資源浪費
- **調試複雜性**：自動化過程可能增加故障排查的難度
- **依賴性**：需要額外部署和維護 Provisioner 組件
- **權限控制**：需要合理設置 StorageClass 的使用權限，避免資源濫用

</details>

---

**Q22.** 請解釋 StorageClass 在動態存儲卷分配過程中的作用。

<details>
<summary>答案</summary>

**StorageClass 的核心作用：**

1. **關聯 Provisioner**：StorageClass 通過 `provisioner` 字段指定使用哪個 Provisioner，決定了底層存儲的類型（如 NFS、AWS EBS、Ceph 等）

2. **定義存儲特性**：通過 `parameters` 字段配置存儲的具體參數，例如 NFS 的 `archiveOnDelete`、`onDelete` 等回收策略

3. **篩選和綁定控制**：限制了 PV 和 PVC 的綁定關係，只有屬於同一 StorageClass 的 PV 和 PVC 才能進行綁定

4. **抽象存儲層**：為開發者提供統一的存儲請求接口，屏蔽了底層存儲實現的複雜性

5. **支持多種存儲類型**：可以創建多個不同的 StorageClass 來滿足不同的性能、可用性和成本需求

**配置示例：**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client
provisioner: k8s-sigs.io/nfs-subdir-external-provisioner
parameters:
  archiveOnDelete: "false"
```

</details>

---

## 第五部分：配對題（共 9 分）

**Q23.** 請將以下組件與其對應的功能進行配對：

| 組件 | 功能 |
| ---- | ---- |
| 1. PV | A. 存儲資源的申請對象，描述需要多大的存儲空間 |
| 2. PVC | B. 動態存儲卷中自動創建 PV 的應用 |
| 3. StorageClass | C. 物理存儲資源的抽象表示，定義存儲容量和訪問模式 |
| 4. Provisioner | D. 定義存儲類型和參數，關聯 Provisioner |

<details>
<summary>答案</summary>
1-C, 2-A, 3-D, 4-B

**解釋：**
- **PV (PersistentVolume)**：是物理存儲資源的抽象表示，定義了存儲容量、訪問模式等屬性
- **PVC (PersistentVolumeClaim)**：是存儲資源的申請對象，用戶通過 PVC 來請求特定大小的存儲空間
- **StorageClass**：定義存儲類型和參數，關聯到特定的 Provisioner，用於動態存儲卷分配
- **Provisioner**：是能夠自動管理存儲、創建 PV 的應用，根據 StorageClass 的定義自動創建合適的 PV

</details>

---

**Q24.** 請將以下 NFS 相關命令與其功能進行配對：

| 命令 | 功能 |
| ---- | ---- |
| 1. `exportfs -ra` | A. 查看 NFS 網絡掛載情況 |
| 2. `showmount -e` | B. 啟動 NFS 服務器服務 |
| 3. `systemctl start nfs-server` | C. 通知 NFS 重新讀取配置文件 |
| 4. `mount -t nfs` | D. 掛載 NFS 共享目錄到本地 |

<details>
<summary>答案</summary>
1-C, 2-A, 3-B, 4-D

**解釋：**
- **exportfs -ra**：重新讀取 /etc/exports 配置文件，讓 NFS 配置生效
- **showmount -e**：檢查 NFS 的網絡掛載情況，顯示可用的共享目錄
- **systemctl start nfs-server**：使用 systemd 啟動 NFS 服務器服務
- **mount -t nfs**：將 NFS 服務器的共享目錄掛載到本地的指定目錄

</details>

---

## 總結

本測驗涵蓋了 Kubernetes 中使用 NFS 網絡共享存儲的核心概念，包括：

- NFS 的基本架構和安裝配置
- 靜態存儲卷（PV/PVC）的使用
- 動態存儲卷和 Provisioner 的部署
- StorageClass 的作用和配置
- 兩種存儲卷類型的優缺點對比

---

**參考資料：** 《Kubernetes 入門實戰課》第 25 講：PersistentVolume + NFS：怎麼使用網絡共享存儲？