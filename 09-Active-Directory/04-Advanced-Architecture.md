# Advanced Active Directory Architecture（Active Directory 進階架構）

本章延伸 AD DS 核心課程，深入理解 FSMO、目錄複寫、站台、信任、Global Catalog、服務帳號與備份復原。這些概念能幫助你理解大型企業網域為何需要多台 DC，以及錯誤設定如何影響整個企業網路。

**範圍：** 以下命令以讀取、盤點與健康檢查為主。FSMO 轉移、信任建立、複寫修復與復原操作都可能影響正式網域，必須先取得核准並準備復原方案。

## 1. FSMO Roles（彈性單一主機作業角色）

AD DS 大部分目錄變更可透過多主機複寫，但部分操作需要由特定 DC 負責，這些角色稱為 FSMO（Flexible Single Master Operations）。

| Role（角色） | Scope（範圍） | Responsibility（主要職責） |
|---|---|---|
| Schema Master（結構描述主機） | Forest（樹系） | 管理 Schema 變更，例如新增物件類別或屬性 |
| Domain Naming Master（網域命名主機） | Forest（樹系） | 管理樹系內網域新增與移除 |
| RID Master（RID 主機） | Domain（網域） | 分配 RID Pool，讓 DC 能產生唯一 SID |
| PDC Emulator（PDC 模擬器） | Domain（網域） | 密碼變更優先處理、帳號鎖定協調、時間階層及相容性工作 |
| Infrastructure Master（基礎架構主機） | Domain（網域） | 更新跨網域物件參照 |

查詢 FSMO 角色：

`cmd
netdom query fsmo
`

PowerShell 查詢：

`powershell
Get-ADForest | Select-Object SchemaMaster,DomainNamingMaster
Get-ADDomain | Select-Object PDCEmulator,RIDMaster,InfrastructureMaster
`

理解重點：
- 一個樹系只有一個 Schema Master 與 Domain Naming Master。
- 每個網域各自有 RID Master、PDC Emulator 與 Infrastructure Master。
- 多台 DC 可以分散提供驗證與目錄查詢，但 FSMO 角色仍有指定持有者。
- FSMO 角色持有者失效時，不要立即強制接管；先確認故障範圍、復原可能性與官方程序。

## 2. AD Replication（Active Directory 目錄複寫）

多台 DC 之間會複寫目錄變更，讓使用者、群組與電腦物件在不同 DC 間保持一致。

主要概念：
- Multi-master replication（多主機複寫）：一般目錄更新可在不同 DC 接受，再透過複寫同步。
- Replication Partner（複寫夥伴）：與某台 DC 交換目錄資料的其他 DC。
- Naming Context / Directory Partition（命名內容／目錄分割區）：例如 Schema、Configuration、Domain 與 Application partitions。
- Replication Topology（複寫拓樸）：決定 DC 之間如何交換資料。
- SYSVOL Replication：網域原則與登入指令稿的複寫；現代網域通常使用 DFS Replication（DFSR）。

檢查複寫摘要：

`cmd
repadmin /replsummary
repadmin /showrepl
`

PowerShell 檢查複寫夥伴：

`powershell
Get-ADReplicationPartnerMetadata -Target * -Scope Forest |
  Select-Object Server,Partner,LastReplicationSuccess,LastReplicationResult
`

常見複寫問題：
- DNS 無法解析其他 DC。
- DC 之間的必要連接埠遭防火牆阻擋。
- 系統時間不同步。
- DC 長期離線。
- 目錄物件或複寫拓樸設定異常。

不要把重複執行強制同步當成萬用修復方式。先讀取錯誤碼、事件日誌與複寫夥伴狀態。

## 3. Sites and Subnets（AD 站台與子網路）

AD Sites 用於描述企業網路的實體或邏輯網路位置，幫助用戶端選擇較合適的 DC，並控制跨站台複寫成本與排程。

例子：
- Tokyo-Site：東京辦公室子網路。
- Osaka-Site：大阪辦公室子網路。
- Site Link：描述站台之間的複寫連結、成本與排程。

Subnet 必須正確對應到 AD Site。若子網路沒有正確建立或對應，用戶端可能找到不理想的 DC，造成登入延遲或跨 WAN 流量增加。

檢查命令：

`powershell
Get-ADReplicationSite -Filter *
Get-ADReplicationSubnet -Filter *
Get-ADReplicationSiteLink -Filter *
`

實務建議：
- 依真實網路拓樸規劃 Site，而不是單純依部門名稱建立。
- 確認每個企業 IP Subnet 都對應到正確 Site。
- 站台連結成本與排程應反映網路頻寬與營運需求。
- 不要在沒有網路設計資訊時隨意修改 Site Link。

## 4. Trust Relationships（網域信任關係）

Trust 是網域或樹系之間的驗證信任機制。它不等於自動授予所有資源權限；使用者仍須通過目標資源的授權檢查。

常見類型：

| Trust Type（信任類型） | 中文說明 |
|---|---|
| Parent-child trust（父子網域信任） | 同一樹系中父網域與子網域之間的信任 |
| Tree-root trust（樹系根信任） | 同一樹系不同網域樹之間的信任 |
| External trust（外部信任） | 與不同樹系或舊式網域建立的特定信任 |
| Forest trust（樹系信任） | 不同 AD 樹系之間的信任 |
| Shortcut trust（捷徑信任） | 為常用跨網域驗證路徑建立的捷徑 |

信任還有方向與傳遞性：
- One-way（單向）：一方信任另一方。
- Two-way（雙向）：雙方互相信任。
- Transitive（可傳遞）：信任可沿著信任路徑延伸。
- Non-transitive（不可傳遞）：信任僅限指定關係。

稽核重點：
- 是否仍需要該信任？
- 信任方向與範圍是否過寬？
- 外部樹系是否有足夠的安全控制？
- 是否定期審查跨網域群組與資源 ACL？
- 是否有未使用的舊信任關係？

## 5. Global Catalog（通用類別目錄）

Global Catalog（GC）保存樹系中各網域物件的部分屬性，讓用戶端能跨網域搜尋物件，並支援 Universal Group Membership 等功能。

常見連接埠：
- TCP 3268：Global Catalog。
- TCP 3269：Global Catalog over TLS。

GC 不代表整個樹系的所有屬性都完整複製到每台 GC；它主要保存完整 Schema 與物件的部分屬性集合。

## 6. Service Accounts and gMSA（服務帳號與群組受管理服務帳號）

服務帳號用於讓 Windows 服務、排程工作或應用程式以特定身分執行。

風險：
- 密碼長期不變。
- 權限高於服務實際需求。
- 多個系統共用同一組認證。
- 帳號負責人與用途不清楚。
- 密碼被寫入腳本、設定檔或 GPO。

Group Managed Service Account（gMSA）可由 AD 管理密碼，並支援符合條件的服務與主機使用。它能降低人工管理服務密碼的負擔，但仍需限制允許擷取受管理密碼的主機範圍。

稽核與改善方向：
1. 建立服務帳號清冊與負責人。
2. 盤點服務所需權限。
3. 優先評估 gMSA 是否適用。
4. 移除不必要的互動式登入權限。
5. 定期檢查 SPN、群組成員與服務設定。
6. 不在公開倉庫保存密碼或認證材料。

## 7. AD Backup and Recovery（AD 備份與復原）

AD DS 是企業核心身分基礎設施，備份必須包含可支援目錄復原的資料與程序。

重點：
- System State Backup（系統狀態備份）。
- 備份媒體與備份帳號的存取控制。
- 離線或不可變備份副本。
- DSRM（目錄服務還原模式）程序。
- AD Recycle Bin（Active Directory 資源回收筒）設定與限制。
- 定期復原演練與復原時間目標。

重要提醒：
- AD Recycle Bin 不能取代完整的系統狀態備份。
- 一般 VM Snapshot 不能直接當作正式 AD 備份策略。
- 虛擬化平台、Windows Server 版本與 VM-Generation ID 支援會影響 DC 虛擬機復原方式。
- 不要任意複製正在運作的 DC 虛擬機並同時啟動多份副本。
- 正式復原應遵循 Microsoft 支援的 AD DS 備份與還原程序。

## 8. Enterprise Security Design（企業網域安全設計）

### Privileged Access（特權存取）

- 日常帳號與管理帳號分開。
- 將 Domain Admin 權限限制在必要的管理任務。
- 對高權限操作使用受控管理工作站與 MFA 支援的管理入口。
- 避免管理員帳號登入一般工作站或不受信任的伺服器。
- 使用 Windows LAPS 管理本機管理員密碼。

### Tiered Administration（分層管理）

常見概念：
- Tier 0：AD、身分驗證系統、PKI 與最高權限控制平面。
- Tier 1：伺服器與企業應用程式。
- Tier 2：一般使用者工作站與終端設備。

實際分層模型需依組織架構調整。核心目的是避免低信任層級的設備或帳號接觸更高權限的管理憑證。

### Monitoring（監控）

持續關注：
- 高權限群組成員變更。
- 非預期的新建帳號與電腦物件。
- 異常登入失敗與帳號鎖定。
- Kerberos 與 NTLM 驗證模式變化。
- GPO 與 SYSVOL 內容修改。
- 非預期的 DC 服務、DNS 或複寫異常。

## 9. Practical Review Lab（實作稽核）

在自己的單一網域實驗室完成：

1. 執行 netdom query fsmo，記錄五種 FSMO 角色持有者。
2. 執行 dcdiag /test:dns，記錄 DNS 檢查結果。
3. 執行 Get-ADReplicationSite -Filter *，查看目前站台。
4. 執行 Get-ADDomainController -Filter *，記錄 DC。
5. 檢查 Domain Admins 成員。
6. 檢查服務帳號與 SPN 清單。
7. 檢查 GPO 清單與 CLIENT01 的 gpresult。
8. 檢查 SMB 分享與 NTFS ACL。
9. 設計一份備份與復原演練計畫，但不要在正式環境執行復原操作。

單 DC 實驗室無法完整演示多 DC 複寫、跨站台與跨樹系信任；這些內容先以架構與讀取檢查為主，日後再建立多 VM 隔離實驗。

## 10. Knowledge Check（知識測驗）

1. 五種 FSMO 角色中，哪兩種是 Forest-wide？
2. 為什麼多 DC 環境需要目錄複寫？
3. AD Site 與 IP Subnet 如何配合？
4. Trust 是否等於自動取得對方所有資源權限？
5. Global Catalog 保存的資料有何特點？
6. gMSA 如何改善服務帳號密碼管理？
7. 為什麼 VM Snapshot 不能取代 AD 正式備份？
8. Tier 0、Tier 1、Tier 2 的主要安全目的為何？

## Sources（資料來源）

- Microsoft Learn — Active Directory Domain Services: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/
- Microsoft Learn — Active Directory Replication: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/replication/active-directory-replication
- Microsoft Learn — Group Managed Service Accounts: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-managed-service-accounts/group-managed-service-accounts-overview
- Microsoft Learn — Windows LAPS: https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview
- Microsoft Learn — AD DS Backup and Recovery: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/backup-restore/active-directory-domain-services-backup-and-restore
- Microsoft Learn — Kerberos Authentication: https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview

本章為架構學習與防禦性管理材料。正式環境的角色轉移、信任修改與目錄復原，必須遵循官方程序及變更管理。
