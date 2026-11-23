# Incident Response（資安事件應變與數位鑑識）

本章建立企業資安事件應變知識體系，從通報、分級、調查、證據保全、遏止、根除、復原，到事後檢討。重點不只是「發現病毒後刪除病毒」，而是保護證據、控制損害、恢復可信任服務，並降低再次發生的機率。

**主要參考框架：** NIST SP 800-61 Rev.3（2025）與 NIST Cybersecurity Framework 2.0。Rev.3 已取代 2012 年的 Rev.2，並將事件應變融入組織整體資安風險管理。

**安全範圍：** 命令只用於自有設備或明確授權的調查。正式環境的隔離、停機、停用帳號或資料處理，必須依組織授權與業務連續性程序執行。

## 1. Event, Alert, Incident（事件、告警與資安事件）

| Term | 中文 | 例子 |
|---|---|---|
| Event | 系統或使用者活動紀錄，不一定代表威脅 | 使用者登入、服務啟動 |
| Alert | 工具依規則或模型產生的可疑訊號 | 同一帳號短時間大量登入失敗 |
| Incident | 已確認或合理懷疑造成安全政策、資料或服務受到影響的情況 | 帳號遭未授權使用、資料外洩、勒索軟體感染 |

一個告警不一定是資安事件；一個資安事件也可能由多個告警、使用者通報與系統紀錄共同確認。

常見事件類型：
- Phishing（釣魚郵件）。
- Account Compromise（帳號遭入侵）。
- Malware Infection（惡意程式感染）。
- Ransomware（勒索軟體）。
- Data Exfiltration（資料外洩）。
- Web Application Compromise（網站遭入侵）。
- Cloud Identity Compromise（雲端身分遭入侵）。
- Denial of Service（阻斷服務）。

## 2. NIST CSF 2.0（資安框架）

NIST CSF 2.0 包含六個核心功能：

| Function | 中文 | 與事件應變的關係 |
|---|---|---|
| Govern | 治理 | 定義責任、政策、風險容忍度與決策權限 |
| Identify | 識別 | 了解資產、資料、業務流程與風險 |
| Protect | 保護 | 建立預防控制、存取控制、備份與教育訓練 |
| Detect | 偵測 | 透過日誌、告警、監控與通報發現異常 |
| Respond | 回應 | 分析、遏止、溝通與處理事件 |
| Recover | 復原 | 恢復可信任服務、監控復原結果並改善韌性 |

事件應變不是只在 Detect 之後才開始。資產盤點、日誌保留、備份、聯絡名單、權限管理與演練，都會影響事故發生時的處理能力。

## 3. Incident Response Team（事件應變團隊）

| Role | Responsibility |
|---|---|
| Incident Commander（事件指揮官） | 決定優先順序、分配任務、批准重要行動 |
| SOC Analyst（SOC 分析師） | 接收告警、初步分流、關聯日誌 |
| Incident Responder（事件應變人員） | 調查端點、帳號、網路與服務，執行核准措施 |
| Forensic Analyst（鑑識分析師） | 擷取、驗證、保存與分析數位證據 |
| IT / System Owner（系統負責人） | 說明系統依賴、執行核准操作與復原 |
| Legal / Privacy（法務／隱私） | 評估通報義務、資料保護與法律風險 |
| Communications（溝通負責人） | 統一對內、對外與客戶溝通 |
| Business Owner（業務負責人） | 評估停機影響與復原優先順序 |

事前必須定義誰有權批准隔離伺服器、停用帳號、暫停服務、聯絡供應商、啟動外部鑑識與從備份復原。不要等事故發生後才臨時尋找決策者。

## 4. Severity and Prioritization（事件分級）

評估事件時至少考慮：
1. Impact：是否影響核心業務、客戶或公共服務？
2. Scope：多少使用者、端點、伺服器、帳號或資料受到影響？
3. Data Sensitivity：是否涉及個人資料、認證資訊、財務或智慧財產？
4. Privilege：是否涉及管理員、網域控制站或雲端高權限角色？
5. Persistence：未授權者是否仍能存取系統？
6. Spread：是否可能橫向影響其他主機或網域？
7. Availability：是否有服務中斷、資料加密或破壞？

| Severity | 說明 | 典型例子 |
|---|---|---|
| SEV-1 Critical | 核心服務大範圍中斷、重大資料外洩或高權限控制平面遭入侵 | Domain Admin／Cloud Global Admin 遭濫用 |
| SEV-2 High | 多個系統或重要資料受影響，威脅可能持續 | 多台端點出現勒索活動 |
| SEV-3 Medium | 範圍有限，尚未發現重大業務影響 | 單一工作站出現可疑程式 |
| SEV-4 Low | 可疑活動尚未證實造成安全影響 | 少量異常登入失敗 |

這是教學分級範例，應依組織政策調整。高權限帳號遭入侵、勒索活動或疑似資料外洩，應依升級程序快速處理。

## 5. Incident Response Workflow（事件應變流程）

### A. Preparation（準備）
- 資產清單、網路拓樸、資料分類與系統負責人。
- 端點、伺服器、網域、雲端與應用程式日誌。
- 正確時間同步（NTP）與統一時區紀錄。
- 備份、復原程序與復原演練。
- EDR／防毒、SIEM、網路監控與存取控制。
- 通報管道、值班表與外部供應商聯絡方式。
- 證據儲存位置、存取權限與保存期限。
- 事件演練與桌上推演（Tabletop Exercise）。

### B. Detection and Triage（偵測與分流）
1. 記錄告警來源、規則名稱與時間。
2. 確認受影響使用者、主機、IP、雲端資源或應用程式。
3. 查詢首次與最近一次活動時間。
4. 使用多個資料來源交叉驗證。
5. 排除已知維護、測試或合法管理活動。
6. 確認是否有持續活動或擴散跡象。

保留原始告警、查詢條件與時間範圍。不要只保留儀表板截圖。

### C. Investigation（調查）
建立事件時間線：
- 使用者何時登入？
- 哪個程序何時啟動？
- 是否建立新帳號或變更群組？
- 是否出現新的排程工作、服務或啟動項目？
- 哪些網路連線與 DNS 查詢值得調查？
- 是否有檔案被建立、修改、刪除或大量讀取？
- 是否有雲端角色、權限或金鑰被修改？

每項結論都應區分 Observed Fact（已觀察事實）、Hypothesis（待驗證假設）、Confirmed Finding（已確認發現）與 Unknown（目前未知）。沒有日誌不等於沒有發生。

### D. Evidence Preservation（證據保全）
證據可能包括 Windows EVTX、Linux Journal、EDR 告警、防火牆、DNS、Proxy、VPN、郵件、記憶體、磁碟映像、檔案雜湊與雲端 Audit Log。

原則：
- 記錄時間、操作者、來源設備與擷取方式。
- 保留原始資料，使用副本分析。
- 計算 SHA-256 雜湊並記錄。
- 限制證據存取權限。
- 每次移交記錄交付人、接收人、時間與用途。
- 依法律、合約與組織政策決定保存期限。

活體系統的每個操作都可能改變狀態。記憶體擷取、程序終止、關機與隔離的先後順序，應由受訓人員依風險與授權決定。

### E. Containment（遏止）
目標是降低損害與阻止擴散，而不是立即刪除所有可疑檔案。

可能措施：
- 使用 EDR 或網路控制隔離受影響端點。
- 暫停遭入侵帳號或撤銷受影響工作階段。
- 對外部連線進行有範圍的封鎖。
- 暫停受影響服務或工作負載。
- 限制受影響雲端角色或憑證。
- 建立替代服務與業務連續性方案。

不要在沒有評估下關閉整台伺服器、重灌或封鎖整個大型網段。

### F. Eradication（根除）
- 修補已確認的漏洞。
- 移除未授權帳號、排程工作、服務或啟動項目。
- 撤銷暴露的 API Key、Token、憑證與工作階段。
- 重設受影響密碼與金鑰。
- 重新建立受信任的系統映像。
- 檢查網域、雲端與其他系統是否存在相同弱點。

只刪除惡意檔案而不修補初始入侵途徑，可能讓威脅再次進入。

### G. Recovery（復原）
- 使用可信任映像或經驗證的乾淨備份。
- 先恢復最重要且依賴關係清楚的服務。
- 驗證系統完整性、帳號、網路與應用程式功能。
- 監控復原後是否再次出現相同告警。
- 記錄復原時間與未解決風險。

### H. Lessons Learned（事後檢討）
整理初始入侵途徑、偵測時間、控制失效、日誌缺口、決策延誤、偵測規則改善、備份問題與後續負責人。檢討目標是改善系統，而不是只歸咎第一線人員。

## 6. Windows Incident Triage（Windows 初步調查）

以下命令主要讀取系統狀態，只能在自有或授權端點使用。

### 6.1 Host Information（主機資訊）

`powershell
Get-Date -Format o
hostname
whoami
Get-ComputerInfo |
  Select-Object CsName,WindowsProductName,WindowsVersion,OsBuildNumber
Get-NetIPConfiguration
`

### 6.2 Process Review（程序檢查）

`powershell
Get-Process |
  Sort-Object CPU -Descending |
  Select-Object -First 20 Name,Id,CPU,StartTime
`

程序名稱本身不足以判定惡意，應搭配路徑、簽章、父程序、命令列、使用者與時間線分析。

### 6.3 Network Review（網路連線）

`powershell
Get-NetTCPConnection |
  Select-Object LocalAddress,LocalPort,RemoteAddress,RemotePort,State,OwningProcess
`

將 PID 對應程序：

`powershell
Get-NetTCPConnection -State Established |
  ForEach-Object {
    $p = Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue
    [PSCustomObject]@{
      RemoteAddress = $_.RemoteAddress
      RemotePort = $_.RemotePort
      PID = $_.OwningProcess
      ProcessName = $p.ProcessName
    }
  }
`

### 6.4 Security Events（安全事件）

`powershell
$Since = (Get-Date).AddHours(-24)
Get-WinEvent -FilterHashtable @{
  LogName = 'Security'
  Id = 4624,4625
  StartTime = $Since
} -MaxEvents 100 |
  Select-Object TimeCreated,Id,ProviderName,Message
`

查詢程序建立事件（需事先啟用對應稽核政策）：

`powershell
Get-WinEvent -FilterHashtable @{
  LogName = 'Security'
  Id = 4688
  StartTime = (Get-Date).AddHours(-24)
} -MaxEvents 50 |
  Select-Object TimeCreated,Id,Message
`

### 6.5 Export and Hash（匯出與雜湊）

`powershell
$Evidence = "C:\IR-Lab-Evidence"
New-Item -ItemType Directory -Path $Evidence -Force

Get-WinEvent -FilterHashtable @{
  LogName = 'Security'
  Id = 4624,4625,4688
  StartTime = (Get-Date).AddHours(-24)
} -MaxEvents 500 |
  Export-Csv "$Evidence\SecurityEvents.csv" -NoTypeInformation -Encoding UTF8

Get-FileHash "$Evidence\SecurityEvents.csv" -Algorithm SHA256
`

CSV 是調查工作副本，不等於完整原始證據。正式鑑識應保留原始 EVTX，並記錄擷取時間、主機與操作者。

## 7. Linux Incident Triage（Linux 初步調查）

`bash
date -Is
hostnamectl
who
last -n 20
ps auxww --sort=-%cpu | head -n 20
ss -plant
ip address
ip route
journalctl --since "24 hours ago" --no-pager
`

檢查指定檔案：

`bash
sha256sum /path/to/suspicious-file
stat /path/to/suspicious-file
file /path/to/suspicious-file
`

不要只因檔名或程序名稱陌生就刪除檔案。先確認雜湊、簽章、套件來源、執行上下文與其他證據。

## 8. Chain of Custody（證據保管鏈）

| Field | Example |
|---|---|
| Evidence ID | IR-2026-001 |
| Source | CLIENT01 / Security.evtx |
| Collected By | Analyst A |
| Collection Time | ISO 8601 timestamp |
| Collection Method | Approved procedure |
| SHA-256 | 64-character hash |
| Storage Location | Restricted evidence repository |
| Transfer Record | Person, time, purpose |
| Notes | Scope, limitations, tool version |

範例中的主機與 ID 是虛構資料。不要把真實客戶名稱、個人資料、認證資訊或未公開事件細節放進公開 GitHub。

## 9. Common Scenarios（常見事件情境）

### Phishing（釣魚郵件）
保存原始郵件與標頭；確認收件人、寄件人、時間、附件雜湊；檢查點擊與認證輸入；關聯郵件閘道、DNS、Proxy、瀏覽器與登入紀錄；若認證暴露，依程序撤銷工作階段並重設憑證。

### Account Compromise（帳號遭入侵）
確認帳號、登入時間、來源 IP、裝置與登入類型；檢查 MFA、新工作階段、群組變更與應用程式授權；依程序撤銷工作階段與憑證；檢查其他系統是否使用相同身分。

### Ransomware（勒索軟體）
啟動重大事件流程；評估感染範圍與備份風險；依授權隔離端點；保留日誌與證據；保護備份系統；從可信任來源重建；評估資料外洩與通報義務。不要在沒有指揮官與系統負責人評估下隨意關閉所有設備或清除磁碟。

## 10. Practical Lab（實驗任務）

完整操作步驟請參閱 [02-Windows-Incident-Response-Lab.md](02-Windows-Incident-Response-Lab.md)。

在自有 Windows VM：
1. 記錄主機名稱、時間、使用者與 IP。
2. 檢查程序與 TCP 連線。
3. 查詢最近 24 小時的 4624、4625 事件。
4. 匯出相關事件並計算 SHA-256。
5. 建立事件時間線。
6. 將發現分為事實、假設、已確認事項與未知事項。
7. 寫出遏止、根除、復原與改善建議；不要在真實設備執行未核准操作。

**Deliverables（成果）：** 調查範圍、工具版本、時間線、證據清單、雜湊、初步分級、調查限制與處理建議。

## 11. Knowledge Check（測驗）

1. Event、Alert 與 Incident 有何不同？
2. NIST CSF 2.0 的六個功能是什麼？
3. 為什麼不能在證據保全前重灌設備？
4. Containment 與 Eradication 有何不同？
5. SHA-256 在證據管理中有何用途？
6. 為什麼必須建立 Chain of Custody？
7. 勒索事件中為什麼要保護備份？
8. 為什麼沒有日誌不等於沒有發生？

## Sources（資料來源）

- NIST SP 800-61 Rev.3: https://csrc.nist.gov/pubs/sp/800/61/r3/final
- NIST Cybersecurity Framework 2.0: https://www.nist.gov/cyberframework
- Microsoft Learn — Get-WinEvent: https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent
- Microsoft Windows Security Auditing: https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/
- Microsoft Sysinternals: https://learn.microsoft.com/en-us/sysinternals/

教育用途教材。正式事件應遵守組織政策、法律、合約與主管機關要求。
