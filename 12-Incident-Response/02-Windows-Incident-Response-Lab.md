# Windows Incident Response Lab（Windows 資安事件應變實作）

本實驗在自有 Windows VM 中完成一次可重現的事件調查：收集主機資訊、檢查程序與連線、讀取 Windows Event Log、建立時間線、計算雜湊並撰寫事件報告。

**實驗邊界：** 只使用自有 VM 與測試帳號。不要在公司設備、他人電腦或未授權系統上測試。本實驗不會執行惡意程式，也不會嘗試入侵其他設備。

## 1. Lab Topology（實驗拓樸）

| VM | Purpose | Network |
|---|---|---|
| WIN-IR01 | Windows 10／11 測試端點 | Host-only |
| KALI01（選用） | 封包觀察與 Linux 工具學習 | 同一隔離網段 |
| HOST | VMware 宿主機 | 不要把靶場橋接到外部網路 |

建議先建立 VMware Snapshot。Windows 10 已於 2025 年 10 月 14 日結束一般支援；若使用 Windows 10 作為實驗端點，請保持隔離。

## 2. Prepare Evidence Folder（建立證據資料夾）

以系統管理員 PowerShell 執行：

`powershell
$CaseId = "IR-LAB-001"
$Evidence = "C:\IR-Lab-Evidence\$CaseId"

New-Item -ItemType Directory -Path $Evidence -Force
Get-Date -Format o | Out-File "$Evidence\collection-time.txt"
hostname | Out-File "$Evidence\hostname.txt"
whoami /all | Out-File "$Evidence\identity.txt"
ipconfig /all | Out-File "$Evidence\network-config.txt"
`

不要把真實使用者名稱、內網 IP、公司網域或其他敏感資訊上傳到公開倉庫。

## 3. Collect Host Baseline（收集主機基礎資訊）

`powershell
Get-ComputerInfo |
  Select-Object CsName,WindowsProductName,WindowsVersion,OsBuildNumber |
  Export-Csv "$Evidence\host-info.csv" -NoTypeInformation -Encoding UTF8

Get-Service |
  Where-Object Status -eq "Running" |
  Select-Object Name,DisplayName,Status |
  Export-Csv "$Evidence\running-services.csv" -NoTypeInformation -Encoding UTF8

Get-Process |
  Select-Object Name,Id,CPU,StartTime |
  Export-Csv "$Evidence\processes.csv" -NoTypeInformation -Encoding UTF8
`

觀察主機名稱、作業系統版本、執行中的服務、程序 PID 與啟動時間。不要只依程序名稱判斷惡意性；應搭配執行路徑、數位簽章、父程序與其他日誌。

## 4. Collect Network State（收集網路狀態）

`powershell
Get-NetIPConfiguration |
  Format-List * |
  Out-File "$Evidence\ip-configuration.txt"

Get-NetTCPConnection |
  Select-Object LocalAddress,LocalPort,RemoteAddress,RemotePort,State,OwningProcess |
  Export-Csv "$Evidence\tcp-connections.csv" -NoTypeInformation -Encoding UTF8

Get-DnsClientCache |
  Export-Csv "$Evidence\dns-cache.csv" -NoTypeInformation -Encoding UTF8
`

DNS 快取與網路連線資料是調查線索，不應單獨作為攻擊定論。

## 5. Review Windows Security Events（檢查安全事件）

常見事件：
- 4624：成功登入。
- 4625：登入失敗。
- 4688：程序建立（需要啟用對應稽核）。
- 4720：建立使用者帳號。
- 4728：加入全域安全群組。
- 4740：帳號遭鎖定。
- 1102：Security Audit Log 被清除。

查詢最近 24 小時：

`powershell
$Since = (Get-Date).AddHours(-24)

$Events = Get-WinEvent -FilterHashtable @{
  LogName = "Security"
  Id = 4624,4625,4688,4720,4728,4740,1102
  StartTime = $Since
} -MaxEvents 1000

$Events |
  Select-Object TimeCreated,Id,RecordId,ProviderName,Message |
  Export-Csv "$Evidence\security-events.csv" -NoTypeInformation -Encoding UTF8
`

若沒有 4688，先確認 Process Creation 稽核是否啟用。不要為了產生事件而在正式設備上任意修改稽核政策。

## 6. Optional: Generate a Benign Test Event（選用：產生無害測試事件）

在自有實驗機建立一筆明確標示為測試的 Application Log 事件：

`powershell
eventcreate /T INFORMATION /ID 100 /L APPLICATION /SO IR-Lab /D "Benign incident response training event"
`

查詢該事件：

`powershell
Get-WinEvent -FilterHashtable @{
  LogName = "Application"
  ProviderName = "IR-Lab"
  Id = 100
  StartTime = (Get-Date).AddHours(-1)
} | Select-Object TimeCreated,Id,ProviderName,Message
`

此測試事件不代表發生安全事件，只用來練習日誌讀取、時間線與報告流程。

## 7. Hash and Preserve Evidence（計算雜湊並保存證據）

`powershell
Get-ChildItem $Evidence -File |
  Get-FileHash -Algorithm SHA256 |
  Export-Csv "$Evidence\sha256-manifest.csv" -NoTypeInformation -Encoding UTF8
`

雜湊清單本身也需要保存與保護。正式鑑識中，應將證據副本存放在受限權限、具備存取紀錄的儲存位置。

## 8. Build an Incident Timeline（建立事件時間線）

| Time | Source | Event | Interpretation | Confidence |
|---|---|---|---|---|
| 10:00 | Host info | 主機資訊收集 | 建立調查基線 | High |
| 10:05 | Security Log | 登入事件 | 待確認帳號與來源 | Medium |
| 10:10 | Process list | 程序清單 | 與其他證據交叉比對 | Low/Medium |
| 10:15 | TCP connections | 網路連線 | 檢查目的位址與程序關係 | Medium |

以上時間只是報告格式範例，請填入實際觀察到的時間。

## 9. Report Template（事件報告範本）

### Executive Summary（執行摘要）
- Case ID：
- 發現時間：
- 受影響系統：
- 初步分級：
- 目前狀態：

### Scope（調查範圍）
- 主機：
- 帳號：
- IP：
- 日誌來源：
- 調查時間區間：

### Evidence（證據）
- Evidence ID：
- 檔案名稱：
- SHA-256：
- 收集者：
- 收集時間：
- 保存位置：

### Findings（調查發現）
- 已觀察事實：
- 待驗證假設：
- 已確認發現：
- 未知事項：

### Response Plan（處理計畫）
- 遏止：
- 根除：
- 復原：
- 持續監控：
- 負責人與批准人：

### Lessons Learned（事後改善）
- 日誌缺口：
- 權限缺口：
- 備份與復原問題：
- 偵測規則改善：
- 後續負責人與期限：

## 10. Completion Checklist（完成檢查）

- [ ] 已記錄主機名稱、時間與作業系統版本。
- [ ] 已收集程序、服務與網路狀態。
- [ ] 已查詢 Windows Security Event Log。
- [ ] 已匯出調查資料。
- [ ] 已計算 SHA-256。
- [ ] 已建立事件時間線。
- [ ] 已區分事實、假設與未知事項。
- [ ] 已撰寫事件報告。
- [ ] 未執行未授權的隔離、刪除或停機操作。

## Sources（資料來源）

- NIST SP 800-61 Rev.3: https://csrc.nist.gov/pubs/sp/800/61/r3/final
- Microsoft Learn — Get-WinEvent: https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent
- Microsoft Windows Security Auditing: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/
- Microsoft Sysinternals: https://learn.microsoft.com/en-us/sysinternals/

實驗目標是建立可重現、可驗證、可追溯的調查流程，而不是執行攻擊。
