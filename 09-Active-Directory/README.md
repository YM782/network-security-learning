# Active Directory Domain Services（Active Directory 網域服務）

本章從 Windows 企業網域的架構開始，逐步理解身分驗證、名稱解析、目錄查詢、檔案分享、群組原則與網域安全。目標是能看懂企業內網如何運作、辨識常見設定錯誤，並在自己的隔離實驗室中進行安全稽核。

**學習邊界：** 所有命令與檢查只用於自有 Windows Server／用戶端，或已取得明確授權的環境。不要將實驗網域連接到公司網路或公開網際網路。

## 1. What Is Active Directory?（什麼是 Active Directory？）

Active Directory Domain Services（AD DS，Active Directory 網域服務）是 Windows Server 提供的目錄服務。企業可用它集中管理使用者與電腦帳號、安全群組、網域登入與身分驗證、Group Policy（群組原則）、伺服器、工作站與其他目錄物件。

AD 不只是「一個登入帳號資料庫」。它也是一套階層式目錄、驗證與管理基礎架構。

### Important Terms（重要名詞）

| Term（術語） | 中文解釋 |
|---|---|
| Object（物件） | 使用者、電腦、群組、印表機等目錄項目 |
| Attribute（屬性） | 物件欄位，例如名稱、SID、電子郵件或群組成員 |
| Schema（結構描述） | 定義目錄中可建立哪些物件類別與屬性 |
| Domain（網域） | 共同目錄資料與管理邊界 |
| Domain Controller / DC（網域控制站） | 執行 AD DS、提供驗證與目錄服務的伺服器 |
| Forest（樹系） | 一個或多個網域組成的 AD 安全與管理架構 |
| Tree（網域樹） | 具有連續 DNS 命名空間的一組網域 |
| OU（組織單位） | 用於組織物件、委派管理與套用 GPO 的容器 |
| Global Catalog / GC（通用類別目錄） | 提供樹系中物件部分屬性的查詢服務 |
| Trust（信任） | 不同網域或樹系之間的驗證信任關係 |
| SID（安全性識別碼） | Windows 用來識別安全主體的識別碼 |
| SYSVOL | 儲存並複寫網域登入指令稿與群組原則檔案的共享資料夾 |

### Domain, Tree, and Forest（網域、網域樹與樹系）

- Domain：例如 `adlab.test`。
- Child Domain：例如 `sales.adlab.test`。
- Forest：可以包含多個網域，並共用 Schema、Configuration 與 Global Catalog 架構。

實務上，不應為了每個部門都建立一個網域。OU、群組與委派管理通常已足以處理許多組織需求。網域與樹系的設計需要考慮管理邊界、信任、複寫、復原與安全風險。

## 2. Domain Controller and AD Database（網域控制站與目錄資料庫）

Domain Controller（DC）通常負責：
- 驗證網域使用者與電腦。
- 回應 LDAP 目錄查詢。
- 提供 Kerberos KDC（Key Distribution Center，金鑰發佈中心）。
- 提供 AD 整合 DNS（依部署設定）。
- 複寫目錄資料與 SYSVOL。
- 提供網域原則資訊。

重要元件：
- **NTDS.dit**：AD DS 目錄資料庫。
- **SYSVOL**：網域原則與登入指令稿的複寫共享資料。
- **NTDS service**：提供目錄資料庫服務。
- **KDC**：負責 Kerberos 票證服務。
- **DNS Server**：協助用戶端尋找網域控制站及服務。

不要直接複製、修改或刪除正在運作的 NTDS.dit。AD 備份與復原應使用支援的 System State／伺服器備份流程。

## 3. DNS in Active Directory（Active Directory 中的 DNS）

DNS 對 AD 非常重要。用戶端不只是查詢 DC 的主機名稱，也需要透過 SRV（Service）記錄尋找 LDAP、Kerberos 等服務。

常見記錄：
- A／AAAA：主機名稱對應 IPv4／IPv6。
- CNAME：別名。
- SRV：服務位置與連接埠。
- PTR：反向查詢。

常見 AD SRV 記錄範例：

`text
_ldap._tcp.dc._msdcs.adlab.test
_kerberos._tcp.adlab.test
`

在網域用戶端執行：

`powershell
ipconfig /all
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.adlab.test
Resolve-DnsName -Type SRV _kerberos._tcp.adlab.test
nltest /dsgetdc:adlab.test
`

**預期觀察：** DNS SRV 查詢應回傳實驗網域控制站的主機名稱與服務連接埠；nltest 應能找到網域控制站。

**常見錯誤：** 網域用戶端把公共 DNS 當作唯一 DNS。公共 DNS 不知道你的私人 AD 網域與內部 SRV 記錄。實驗環境中，用戶端 DNS 應指向實驗 DC 的 DNS 服務。

## 4. Kerberos Authentication（Kerberos 身分驗證）

Kerberos 是現代 Active Directory 網域常用的驗證協定。它使用票證（Ticket）與金鑰，而不是在每次存取服務時直接傳送使用者密碼。

### Main Components（主要元件）

- Client（用戶端）：提出登入與服務存取要求。
- KDC（Key Distribution Center）：通常在 DC 上運作。
- AS（Authentication Service）：處理初始驗證並簽發 TGT。
- TGS（Ticket-Granting Service）：依 TGT 簽發特定服務票證。
- Service Principal Name / SPN（服務主體名稱）：識別網域服務執行個體。

### Simplified Flow（簡化流程）

1. **AS-REQ / AS-REP**：用戶端向 KDC 進行初始驗證，取得 TGT（Ticket-Granting Ticket）。
2. **TGS-REQ / TGS-REP**：用戶端使用 TGT，向 KDC 要求存取某個服務的 Service Ticket。
3. **AP-REQ**：用戶端向目標服務出示服務票證。
4. 服務驗證票證後，依 Windows 權限模型檢查使用者是否有權執行操作。

簡化記憶：先取得 TGT，再取得目標服務票證，最後向服務出示票證。

### SPN（服務主體名稱）

SPN 用來識別服務執行個體，例如某個主機上的 HTTP 或 MSSQL 服務。SPN 設定錯誤可能造成 Kerberos 無法正常工作，並可能讓服務帳號管理產生安全風險。

### Time Synchronization（時間同步）

Kerberos 對時間差有容忍範圍。DC、用戶端與服務時間差異過大，可能導致票證驗證失敗。請檢查時間來源與時區，不要只靠反覆重設密碼排錯。

檢查命令：

`powershell
w32tm /query /status
klist
`

klist 可檢視目前登入工作階段的 Kerberos 票證資訊。不要將票證或認證材料上傳到公開倉庫。

## 5. LDAP（輕量型目錄存取協定）

LDAP（Lightweight Directory Access Protocol）用來查詢與管理目錄資料。AD DS 使用 LDAP 資料模型呈現目錄物件與屬性。

常見連接埠：
- TCP 389：LDAP。
- TCP 636：LDAPS（透過 TLS 保護的 LDAP）。
- TCP 3268：Global Catalog。
- TCP 3269：Global Catalog over TLS。

常見識別名稱（Distinguished Name，DN）：

`text
CN=Alex Chen,OU=LabUsers,DC=adlab,DC=test
`

拆解：
- CN：Common Name，物件名稱。
- OU：Organizational Unit，組織單位。
- DC：Domain Component，網域名稱片段。

LDAP Filter 範例（查詢啟用中的使用者）：

`text
(&(objectCategory=person)(objectClass=user)(!(userAccountControl:1.2.840.113556.1.4.803:=2)))
`

LDAP 查詢只應讀取完成管理或稽核工作所需的屬性。正式環境應依組織安全政策使用適當的 LDAP 簽署、通道繫結與 TLS 設定。

## 6. SMB and File Sharing（SMB 與檔案分享）

SMB（Server Message Block）常用於 Windows 檔案分享、印表機分享及其他網路服務。常見連接埠為 TCP 445。

網路分享的有效存取權限通常同時受到 Share Permissions（共享權限）與 NTFS Permissions（檔案系統權限）影響。兩層權限必須一起檢查。不要只看共享權限，就判定使用者一定可以讀取或修改檔案。

檢查命令（在自有 Windows Server 上）：

`powershell
Get-SmbShare
Get-SmbShareAccess -Name "LabShare"
Get-Acl "C:\LabShare"
`

安全建議：
- 移除 Everyone 等不必要的廣泛授權。
- 依職務使用安全群組授權，不要逐一對使用者設定。
- 不要使用 SMBv1。
- 依環境需求評估 SMB Signing 與 SMB Encryption。
- 定期檢查共享目錄中的敏感資料與繼承權限。

## 7. Groups, OUs, and Permissions（群組、OU 與權限）

### Security Groups（安全群組）

| Scope（範圍） | 主要用途 |
|---|---|
| Global（全域） | 組織同一網域中的使用者與群組 |
| Domain Local（網域本機） | 將權限授予網域中的資源 |
| Universal（通用） | 在多網域樹系中組織成員與權限 |

常見設計思路：將使用者加入職務群組，再將職務群組加入資源權限群組。不要把一般員工直接加入 Domain Admins。

### OU and Delegation（OU 與委派管理）

OU 用來組織物件、套用 GPO 與委派有限管理工作。OU 不是安全邊界；真正的存取控制仍需依賴權限與管理設計。

### Built-in Privileged Groups（內建高權限群組）

例如 Domain Admins、Enterprise Admins、Schema Admins、Administrators、Account Operators。高權限群組成員應極少、可追蹤、定期審查。日常使用帳號與管理帳號應分開。

## 8. Group Policy — GPO（群組原則）

Group Policy Object（GPO）可集中設定 Windows 電腦與使用者環境，例如密碼與帳戶鎖定政策、Windows Defender 與防火牆設定、Windows Update 設定、稽核政策、登入指令稿與使用者環境限制。

常見套用順序為 LSDOU：
1. Local（本機）。
2. Site（站台）。
3. Domain（網域）。
4. OU（組織單位，從上層到下層）。

強制套用（Enforced）、阻止繼承（Block Inheritance）及安全性篩選可能影響最終結果。

檢查命令：

`powershell
gpresult /r
gpresult /h "$env:TEMP\gpresult.html"
gpupdate /force
`

注意：gpupdate /force 會重新套用原則；在正式環境執行前應確認變更影響與維護窗口。

## 9. Important AD Network Ports（AD 常見網路連接埠）

| Port | Protocol / Service（協定／服務） | 用途 |
|---:|---|---|
| 53 | TCP／UDP DNS | 網域名稱與服務探索 |
| 88 | TCP／UDP Kerberos | 網域驗證 |
| 123 | UDP NTP | 時間同步 |
| 135 | TCP RPC Endpoint Mapper | RPC 端點對應 |
| 389 | TCP／UDP LDAP | 目錄查詢 |
| 445 | TCP SMB | 檔案分享與部分網域作業 |
| 464 | TCP／UDP Kerberos Password Change | Kerberos 密碼變更 |
| 636 | TCP LDAPS | TLS 保護的 LDAP |
| 3268 | TCP Global Catalog | 通用類別目錄查詢 |
| 3269 | TCP Global Catalog over TLS | TLS 保護的通用類別目錄 |
| 9389 | TCP AD Web Services | Active Directory Web Services |
| 49152–65535 | TCP Dynamic RPC（常見預設範圍） | Windows 動態 RPC |

實際需要開放的連接埠取決於角色、版本、網路拓樸與防火牆設定。不要為了「讓 AD 能用」就對所有網段開放整段連接埠；應依官方文件與最小權限原則設定來源、目的地與服務。

## 10. Defensive Baseline（防禦基準）

- 使用獨立的日常帳號與管理帳號。
- 限制 Domain Admins、Enterprise Admins 等高權限群組成員。
- 為本機管理員密碼使用 Windows LAPS 等受控機制。
- 使用 MFA 保護支援 MFA 的管理入口與遠端存取。
- 保持 DC 與用戶端更新，限制 DC 的一般瀏覽與日常工作。
- 檢查 LDAP Signing、LDAP Channel Binding 與 LDAPS 相容性後再逐步強化。
- 停用 SMBv1，並依相容性評估 SMB Signing／Encryption。
- 使用 Windows Defender、Firewall 與集中式事件日誌。
- 定期備份 System State，並實際演練復原。
- 定期審查舊帳號、服務帳號、群組成員、GPO 委派與共享資料夾權限。
- 保護 SYSVOL、登入指令稿與 GPO，避免存放明文密碼或秘密。

## 11. First PowerShell Commands（第一批 PowerShell 命令）

以下命令主要用於讀取自己實驗網域的狀態。請在已安裝 RSAT Active Directory 模組的管理工作站，或網域控制站上執行。

`powershell
Import-Module ActiveDirectory
Get-ADDomain
Get-ADForest
Get-ADDomainController -Filter *
Get-ADOrganizationalUnit -Filter *
Get-ADUser -Filter * -ResultSetSize 10 |
  Select-Object Name, SamAccountName, Enabled
Get-ADGroup -Filter * -ResultSetSize 10 |
  Select-Object Name, GroupScope, GroupCategory
Get-ADDefaultDomainPasswordPolicy
`

Windows 用戶端檢查：

`powershell
whoami /all
nltest /dsgetdc:adlab.test
klist
gpresult /r
ipconfig /all
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.adlab.test
Test-ComputerSecureChannel -Verbose
`

將 adlab.test 替換成你自己的實驗網域名稱。

## 12. Knowledge Check（知識測驗）

1. Domain、Tree 與 Forest 有什麼差異？
2. 為什麼 AD 用戶端 DNS 應指向內部 AD DNS？
3. Kerberos 中 TGT 與 Service Ticket 分別做什麼？
4. LDAP 與 Kerberos 的用途有何不同？
5. 為什麼 OU 不是安全邊界？
6. GPO 的 LSDOU 套用順序是什麼？
7. 為什麼共享權限與 NTFS 權限必須一起檢查？
8. 為什麼一般員工不應加入 Domain Admins？

## Sources（資料來源）

- Microsoft Learn — Active Directory Domain Services: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/
- Microsoft Learn — Kerberos Authentication Overview: https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview
- Microsoft Learn — Windows LAPS: https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview
- Microsoft Learn — Group Policy: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-policy/group-policy-overview
- Microsoft Learn — SMB Security: https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-security
- Microsoft Learn — Active Directory PowerShell: https://learn.microsoft.com/en-us/powershell/module/activedirectory/

本頁是原創學習筆記。請以 Microsoft 官方文件及你實際部署的 Windows Server 版本為準。
