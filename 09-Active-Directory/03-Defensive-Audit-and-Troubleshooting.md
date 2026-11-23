# Active Directory Defensive Audit and Troubleshooting（AD 防禦稽核與故障排查）

本手冊教你以管理員或安全稽核人員的角度，檢查 AD 網域健康狀態、權限、帳號、群組原則、檔案分享與事件日誌。

**安全範圍：** 命令以讀取與稽核為主，請先在自己的實驗網域練習。正式環境中的任何修正、停用帳號、重設密碼或政策變更，都必須經過變更管理與系統負責人核准。

## 1. Domain Health Checks（網域健康檢查）

在網域控制站以系統管理員 PowerShell 或命令提示字元執行：

```powershell
Get-ADDomain
Get-ADForest
Get-ADDomainController -Filter *
Get-Service DNS,NTDS,Netlogon,KDC
dcdiag /v
dcdiag /test:dns /v
```

重點：
- DC 是否正常回應。
- DNS、NTDS、Netlogon、KDC 服務是否運作。
- DNS SRV 記錄是否正確。
- Directory Service 與 System 日誌是否出現持續性錯誤。

多 DC 環境可檢查複寫：

```powershell
repadmin /replsummary
repadmin /showrepl
```

單一 DC 的實驗網域沒有第二台 DC 可供複寫，因此不要把沒有複寫夥伴誤判成故障。

## 2. User and Computer Account Review（使用者與電腦帳號檢查）

列出部分使用者：

```powershell
Get-ADUser -Filter * -ResultSetSize 30 -Properties Enabled,LastLogonDate |
  Select-Object Name,SamAccountName,Enabled,LastLogonDate
```

列出停用帳號：

```powershell
Search-ADAccount -AccountDisabled -UsersOnly |
  Select-Object Name,SamAccountName
```

找出長時間未登入的帳號（先將結果視為審查清單，不要自動刪除）：

```powershell
Search-ADAccount -AccountInactive -UsersOnly -TimeSpan 90.00:00:00 |
  Select-Object Name,SamAccountName,LastLogonDate
```

列出密碼永不過期的帳號：

```powershell
Get-ADUser -Filter * -Properties PasswordNeverExpires,Enabled |
  Where-Object { $_.Enabled -and $_.PasswordNeverExpires } |
  Select-Object Name,SamAccountName,PasswordNeverExpires
```

注意：服務帳號、緊急帳號與特定系統整合可能有例外，但必須有負責人、用途、風險接受紀錄與替代控制。不要只因為查詢結果出現帳號，就直接停用它。

## 3. Privileged Group Review（高權限群組稽核）

檢查網域管理員群組：

```powershell
Get-ADGroupMember -Identity "Domain Admins" -Recursive |
  Select-Object Name,ObjectClass,SamAccountName
```

檢查樹系管理員群組：

```powershell
Get-ADGroupMember -Identity "Enterprise Admins" -Recursive |
  Select-Object Name,ObjectClass,SamAccountName
```

稽核問題：
- 每個成員是否仍有業務需要？
- 是否有一般日常帳號被加入高權限群組？
- 是否有離職、停用或未知帳號仍保有權限？
- 是否有巢狀群組導致權限擴大？
- 是否有不必要的 OU 委派或 GPO 編輯權限？

應遵循最小權限、獨立管理帳號、定期審查與可追蹤原則。

## 4. Service Account and SPN Inventory（服務帳號與 SPN 清單）

SPN 用於將服務與網域帳號關聯。盤點 SPN 是了解服務依賴與帳號風險的一部分：

```powershell
Get-ADUser -LDAPFilter "(servicePrincipalName=*)" `
  -Properties ServicePrincipalName,PasswordLastSet,Enabled |
  Select-Object Name,SamAccountName,Enabled,PasswordLastSet,ServicePrincipalName
```

審查重點：
- 帳號是否仍被服務使用？
- 密碼與金鑰是否有受控輪替機制？
- 是否能改用 Group Managed Service Account（gMSA）？
- 是否有不必要的網域高權限？
- 服務是否使用正確 SPN 與最低必要權限？

本手冊只做服務帳號盤點與防禦審查，不進行票證擷取、密碼破解或憑證竊取。

## 5. Group Policy Review（群組原則稽核）

列出網域 GPO：

```powershell
Get-GPO -All |
  Select-Object DisplayName,Id,GpoStatus,CreationTime,ModificationTime
```

檢查用戶端實際套用結果：

```powershell
gpresult /r
gpresult /h "$env:TEMP\gpresult.html"
```

審查：
- 是否有未使用或重複的 GPO。
- 是否有過度寬鬆的安全性篩選。
- 是否將管理員權限委派給不適當的群組。
- 是否在 GPO 或 SYSVOL 指令稿中留下明文密碼、API Key 或其他秘密。
- 是否有未經審查的登入指令稿或軟體部署設定。

不要直接刪除看似未使用的 GPO；先確認連結、繼承、篩選與實際套用狀況。

## 6. SMB Share and NTFS Permission Review（共享與檔案權限稽核）

列出 SMB Shares：

```powershell
Get-SmbShare |
  Select-Object Name,Path,Description,EncryptData
```

檢查指定共享權限：

```powershell
Get-SmbShareAccess -Name "LabShare"
```

檢查 NTFS ACL：

```powershell
Get-Acl "C:\LabShare" | Format-List
icacls C:\LabShare
```

審查重點：
- 是否有不必要的 Everyone／廣泛群組寫入權限。
- 是否把敏感資料放在所有員工都能讀取的共享目錄。
- Share 與 NTFS 權限是否一致且符合職務需求。
- 是否有不必要的繼承例外或孤兒 SID。
- SMBv1 是否仍被啟用。

## 7. Security Event Log（安全事件日誌）

常見 Windows Security Event ID：

| Event ID | 意義 |
|---:|---|
| 4624 | 成功登入 |
| 4625 | 登入失敗 |
| 4672 | 指派特殊權限的登入 |
| 4720 | 建立使用者帳號 |
| 4728 | 使用者加入全域安全群組 |
| 4732 | 使用者加入網域本機安全群組 |
| 4756 | 使用者加入通用安全群組 |
| 4740 | 帳號遭鎖定 |
| 4768 | Kerberos TGT 要求 |
| 4769 | Kerberos Service Ticket 要求 |
| 4771 | Kerberos 預先驗證失敗 |
| 4776 | NTLM 認證 |
| 5136 | Active Directory 物件遭修改 |

事件是否產生，取決於 Advanced Audit Policy、事件記錄設定、系統版本與保留期限。事件 ID 本身不能證明攻擊；必須結合帳號、來源主機、時間、登入類型與其他紀錄分析。

查詢最近七天的帳號鎖定事件：

```powershell
Get-WinEvent -FilterHashtable @{
  LogName = 'Security'
  Id = 4740
  StartTime = (Get-Date).AddDays(-7)
} -MaxEvents 50
```

查詢最近七天的帳號建立事件：

```powershell
Get-WinEvent -FilterHashtable @{
  LogName = 'Security'
  Id = 4720
  StartTime = (Get-Date).AddDays(-7)
} -MaxEvents 50
```

若查不到事件，先檢查是否啟用了對應稽核子類別、是否有足夠權限，以及日誌是否已被覆寫。

## 8. Network and Protocol Checks（網路與協定檢查）

在用戶端測試必要服務連通性：

```powershell
Test-NetConnection dc01.adlab.test -Port 53
Test-NetConnection dc01.adlab.test -Port 88
Test-NetConnection dc01.adlab.test -Port 389
Test-NetConnection dc01.adlab.test -Port 445
Test-NetConnection dc01.adlab.test -Port 9389
```

以上主機名稱與網域名稱請替換為你的實驗環境。測試連接埠可連線，不代表整個 AD 已經健康；還需要檢查 DNS SRV、服務狀態、時間同步與驗證結果。

## 9. Troubleshooting Matrix（故障排查表）

| Symptom（現象） | Likely cause（可能原因） | First checks（優先檢查） |
|---|---|---|
| 網域登入失敗 | DNS、時間、DC 不可用或帳號狀態 | ipconfig /all、nltest /dsgetdc、事件日誌 |
| 找不到 DC | DNS SRV 記錄或網路路由問題 | Resolve-DnsName、nltest |
| Kerberos 錯誤 | 時間差、DNS、SPN 或票證問題 | w32tm /query /status、klist |
| GPO 沒有套用 | OU、連結、篩選或繼承設定 | gpresult /h、GPMC |
| 安全通道失效 | 電腦帳號密碼不同步或網路問題 | Test-ComputerSecureChannel -Verbose |
| SMB 存取被拒 | Share／NTFS 權限或身分驗證 | Get-SmbShareAccess、Get-Acl |
| LDAP 查詢失敗 | DNS、連接埠、TLS 或權限問題 | Resolve-DnsName、連線測試、DC 日誌 |
| AD 複寫錯誤 | DNS、時間、網路或複寫拓樸 | repadmin /replsummary、repadmin /showrepl |

### Safe Troubleshooting Sequence（安全排錯順序）

1. 記錄故障時間與受影響的用戶端。
2. 檢查用戶端 IP、DNS 與預設閘道。
3. 檢查 DC 名稱解析與 SRV 記錄。
4. 檢查時間同步與 Kerberos 票證。
5. 檢查必要服務與防火牆。
6. 檢查群組原則、帳號狀態與存取權限。
7. 檢查 DC 與用戶端事件日誌。
8. 一次只調整一個設定，驗證後記錄結果。

不要一遇到網域登入問題就重設電腦帳號、刪除使用者或重建網域。

## 10. Security Audit Checklist（安全稽核清單）

- [ ] Domain Admins／Enterprise Admins 成員經過審查。
- [ ] 一般帳號與管理帳號分離。
- [ ] 停用及長期未使用帳號已由負責人確認。
- [ ] 服務帳號有明確擁有人與用途。
- [ ] SPN 與委派設定經過審查。
- [ ] OU 與 GPO 委派權限符合最小權限。
- [ ] SYSVOL 與登入指令稿沒有明文秘密。
- [ ] SMB Shares 與 NTFS ACL 經過檢查。
- [ ] LDAP Signing／Channel Binding 設定經過相容性評估。
- [ ] SMBv1 已停用或有正式例外紀錄。
- [ ] DC 與用戶端安全更新狀態已檢查。
- [ ] Security Event Log 已啟用適當稽核政策。
- [ ] System State 備份存在且復原程序經過演練。
- [ ] 實驗網域與正式網路隔離。

## 11. Sources（資料來源）

- Microsoft Learn — Active Directory Domain Services: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/
- Microsoft Learn — Windows Security Auditing: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/
- Microsoft Learn — DCDiag: https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/dcdiag
- Microsoft Learn — Repadmin: https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/repadmin
- Microsoft Learn — Windows LAPS: https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview
- Microsoft Learn — SMB Security: https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-security

本手冊以防禦性管理與稽核為主。任何正式環境變更都應先取得核准、備份並安排回復方案。
