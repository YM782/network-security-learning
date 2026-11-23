# Active Directory Lab Setup（Active Directory 隔離實驗室建置）

本實驗建立一個小型 Windows 網域，讓你親自觀察 Domain Controller、DNS、Kerberos、LDAP、群組與群組原則如何協同運作。

**實驗目標：** 建立一台 Windows Server 網域控制站，以及一台 Windows 用戶端；整個網域使用 VMware Host-only 隔離網路。

**禁止事項：** 不要將此實驗網域連接到公司網路、公共 Wi-Fi 或公開 IP。不要使用真實密碼、真實帳號或真實資料。

## 1. Lab Topology（實驗拓樸）

| VM | Role（角色） | Example address（範例 IP） |
|---|---|---|
| DC01 | Windows Server、AD DS、DNS | 192.168.77.10 |
| CLIENT01 | Windows 10／11 Pro 測試用戶端 | 192.168.77.20 |
| KALI01（選用） | Linux 管理與封包觀察 | 192.168.77.30 |

以上 IP 僅為範例。先確認 192.168.77.0/24 沒有與宿主機、VPN 或其他 VMware 網路重疊，再使用這組網段。若有衝突，選擇另一個不重疊的私人網段並同步修改所有設定。

### Recommended VM Resources（建議虛擬機資源）

| VM | vCPU | RAM | Disk |
|---|---:|---:|---:|
| DC01 | 2 | 4–6 GB | 60 GB |
| CLIENT01 | 2 | 4 GB | 50 GB |
| KALI01（選用） | 2 | 4 GB | 40 GB |

資源不足時可先只建立 DC01，再建立 CLIENT01。請依實體電腦可用記憶體調整，不要讓宿主機因 RAM 不足而大量交換到磁碟。

## 2. Requirements（準備項目）

- VMware Workstation。
- Windows Server Evaluation 或合法授權的 Windows Server 安裝媒體。
- Windows 10／11 Pro、Enterprise 或 Education 用戶端。Windows Home 不支援加入傳統 AD DS 網域。
- 至少一個 VMware Host-only 虛擬網路。
- 每台 VM 的快照（Snapshot）。

Windows 10 已於 2025 年 10 月 14 日結束一般支援。若使用 Windows 10 作為用戶端，只應放在隔離實驗環境中；日常連網與正式工作請使用仍受支援的作業系統。

官方 Windows Server 評估中心：https://www.microsoft.com/evalcenter/

## 3. Create Host-only Network（建立 Host-only 網路）

1. 關閉所有實驗 VM。
2. 開啟 VMware Virtual Network Editor。
3. 選擇或建立 Host-only 網路。
4. 記錄該網路的子網路與遮罩。
5. 確認沒有與宿主機目前使用的網路、VPN 網段或其他 VM 網路重疊。
6. 將 DC01、CLIENT01 與選用的 KALI01 都連接到同一個 Host-only 網路。
7. 不要使用 Bridged（橋接）模式連接故意存在弱點的靶場。

若使用手動靜態 IP，確認 DHCP 不會分配相同位址。隔離實驗網路可以不設定預設閘道；不要為了方便而把它接到真實公司網路。

## 4. Configure DC01 Network（設定網域控制站網路）

在 DC01 設定：
- IP：192.168.77.10（僅為本手冊範例）。
- Subnet Mask：255.255.255.0。
- Default Gateway：隔離網路通常留空。
- Preferred DNS：先設定為 DC01 自己的靜態 IP；升級為 DC 後由本機 DNS 服務解析網域。

PowerShell 檢查：

```powershell
Get-NetIPConfiguration
ipconfig /all
```

不要直接複製範例 IP，先確認你的 VMware Host-only 網段確實使用相同子網路。

在升級為網域控制站之前，先把伺服器電腦名稱設定為 DC01，並重新啟動：

```powershell
Rename-Computer -NewName "DC01" -Restart
```

重新啟動後，以系統管理員登入，再確認名稱與網路設定：

```powershell
hostname
Get-NetIPConfiguration
```

## 5. Install AD DS Role（安裝 AD DS 角色）

在全新 Windows Server VM 中，以系統管理員身分開啟 PowerShell：

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

檢查角色安裝結果：

```powershell
Get-WindowsFeature AD-Domain-Services
```

接著在這台專用實驗伺服器上建立新的樹系：

```powershell
Install-ADDSForest `
  -DomainName "adlab.test" `
  -DomainNetbiosName "ADLAB" `
  -InstallDNS
```

系統會要求設定 DSRM（Directory Services Restore Mode，目錄服務還原模式）密碼。請使用獨立、強度足夠的實驗密碼，安全保存，不要寫入 GitHub。

**重要：** Install-ADDSForest 會把這台伺服器升級為新的網域控制站，並重新啟動。只在全新、專供實驗的 VM 上執行，不要在日常電腦或現有企業伺服器執行。

## 6. Verify the Domain（驗證網域）

DC01 重新啟動後，以網域管理員登入，執行：

```powershell
Get-ADDomain
Get-ADForest
Get-ADDomainController -Filter *
Get-Service DNS,NTDS,Netlogon
dcdiag /test:dns
```

檢查 DNS SRV 記錄：

```powershell
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.adlab.test
Resolve-DnsName -Type SRV _kerberos._tcp.adlab.test
```

**預期結果：** 網域名稱應為 adlab.test，DC 名稱應指向你建立的伺服器，DNS、NTDS 與 Netlogon 等服務應正常運作。

## 7. Create Test OUs and Users（建立測試 OU 與使用者）

先建立兩個 OU：

```powershell
New-ADOrganizationalUnit -Name "LabUsers" -Path "DC=adlab,DC=test"
New-ADOrganizationalUnit -Name "LabComputers" -Path "DC=adlab,DC=test"
```

建立一個虛構測試帳號：

```powershell
$Password = Read-Host "Enter a temporary lab password" -AsSecureString

New-ADUser `
  -Name "Lab User 01" `
  -SamAccountName "lab.user01" `
  -UserPrincipalName "lab.user01@adlab.test" `
  -Path "OU=LabUsers,DC=adlab,DC=test" `
  -AccountPassword $Password `
  -Enabled $true
```

建立一般測試安全群組：

```powershell
New-ADGroup `
  -Name "Lab-Helpdesk" `
  -SamAccountName "Lab-Helpdesk" `
  -GroupCategory Security `
  -GroupScope Global `
  -Path "OU=LabUsers,DC=adlab,DC=test"

Add-ADGroupMember -Identity "Lab-Helpdesk" -Members "lab.user01"
Get-ADGroupMember -Identity "Lab-Helpdesk"
```

以上命令只建立實驗帳號與一般測試群組。不要把一般測試帳號加入 Domain Admins。

## 8. Configure CLIENT01 DNS（設定用戶端 DNS）

將 CLIENT01 的網路介面連接到相同 Host-only 網路，並設定：
- IP：192.168.77.20（範例）。
- Subnet Mask：255.255.255.0。
- DNS Server：DC01 的 IP，例如 192.168.77.10。
- Default Gateway：隔離實驗網路通常留空。

先測試名稱解析：

```powershell
ipconfig /all
Resolve-DnsName dc01.adlab.test
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.adlab.test
nltest /dsgetdc:adlab.test
```

如果 DNS 查詢失敗，先不要急著加入網域；先修正 DNS、IP 與網路介面設定。

## 9. Join CLIENT01 to the Domain（將用戶端加入網域）

先在用戶端設定電腦名稱為 CLIENT01（若尚未設定），重新啟動後再加入網域：

```powershell
Rename-Computer -NewName "CLIENT01" -Restart
```

確認用戶端版本支援加入傳統 AD 網域，且 DNS 已指向 DC01。以系統管理員身分執行：

```powershell
Add-Computer `
  -DomainName "adlab.test" `
  -Credential (Get-Credential) `
  -Restart
```

在認證視窗輸入實驗網域管理員帳號，例如 ADLAB\Administrator。不要把密碼直接寫進命令列或腳本。

重新啟動後，可使用實驗網域帳號登入。完成後檢查：

```powershell
whoami
whoami /all
nltest /dsgetdc:adlab.test
Test-ComputerSecureChannel -Verbose
```

網域加入成功後，CLIENT01 的電腦帳號預設可能位於內建 Computers 容器。若要讓 GPO 套用到 LabComputers OU，請在 DC01 上以管理員身分將電腦物件移入該 OU：

```powershell
Move-ADObject -Identity (Get-ADComputer -Identity "CLIENT01").DistinguishedName `
  -TargetPath "OU=LabComputers,DC=adlab,DC=test"
```

先確認電腦物件名稱與目標 OU 正確；不要對正式環境的電腦物件執行此範例。

## 10. Group Policy Practice（群組原則練習）

在 DC01 開啟 Group Policy Management（群組原則管理）：
1. 建立名為 Lab-Workstation-Baseline 的測試 GPO。
2. 將測試用戶端放入 LabComputers OU。
3. 將 GPO 連結到該 OU。
4. 先設定一項低風險、可回復的測試原則。
5. 在 CLIENT01 執行 gpupdate /force。
6. 使用 gpresult /h "$env:TEMP\gpresult.html" 查看套用結果。

不要一開始就修改整個網域的密碼政策、防火牆或登入限制。先在專用 OU 與測試用戶端驗證。

## 11. Observe Authentication and Logs（觀察驗證與日誌）

在 CLIENT01：
- 登入實驗網域帳號。
- 執行 klist 查看 Kerberos 票證。
- 執行 whoami /all 查看目前身分與群組。
- 開啟 Event Viewer，查看 Windows Logs → Security。

在 DC01：
- 查看 Security 日誌中的登入與帳號管理事件。
- 查看 DNS Server 與 Directory Service 相關日誌。
- 記錄事件時間、帳號、來源電腦與結果。

只有在對應稽核政策已啟用且事件未被覆寫時，才會看到完整的相關事件。

## 12. Snapshot and Cleanup（快照與清理）

1. 關閉 VM 或確認服務狀態正常。
2. 建立 VMware Snapshot。
3. 將實驗筆記與非敏感輸出保存到私人位置。
4. 不要提交密碼、票證、認證資料、私鑰或真實資產清單。
5. 結束實驗時關閉靶場 VM，確認沒有對外埠號轉發。
6. 不再需要時，還原快照或刪除專用實驗 VM。

## 13. Troubleshooting（故障排查）

| 現象 | 優先檢查 |
|---|---|
| 找不到網域控制站 | 用戶端 DNS 是否指向 DC、SRV 記錄是否存在 |
| 加入網域失敗 | DNS、時間、用戶端版本、帳號權限與網路連通 |
| Kerberos 驗證失敗 | DC／用戶端時間、DNS、SPN 與票證狀態 |
| GPO 沒有套用 | OU 位置、GPO Link、安全性篩選與繼承 |
| 用戶端無法登入 | DC 狀態、DNS、帳號狀態與網路 |
| AD PowerShell 命令不存在 | 是否安裝 RSAT／ActiveDirectory 模組 |

## Sources（資料來源）

- Microsoft Learn — Active Directory Domain Services: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/
- Microsoft Evaluation Center: https://www.microsoft.com/evalcenter/
- Microsoft Learn — Install Active Directory Domain Services: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services--level-100-
- VMware Workstation Documentation: https://docs.vmware.com/

本實驗只適用於自有、隔離的虛擬網路。
