# Network Security Study Roadmap（網路安全系統學習路線）

本路線以每週約 10–12 小時為基準，規劃 12 個月。這是參考時程，不是保證；若尚未達到階段驗收標準，請延長該階段。

## Month 1 — Computer Fundamentals（第 1 個月：計算機底層）

**Theory（理論）**
- CPU, registers, caches, RAM, storage, buses（CPU、暫存器、快取、記憶體、儲存裝置與匯流排）。
- BIOS/UEFI and boot process（BIOS/UEFI 與開機流程）。
- Processes, threads, virtual memory, paging（程序、執行緒、虛擬記憶體與分頁）。
- Kernel mode, user mode, system calls, interrupts（核心模式、使用者模式、系統呼叫與中斷）。
- Filesystems and drivers（檔案系統與驅動程式）。

**Commands（命令）**
```bash
uname -a
lscpu
free -h
lsblk
ps aux
journalctl -b
```

**Lab（實驗）：** 在自己的 Linux 虛擬機中檢查 CPU、記憶體、磁碟、程序與開機日誌。  
**Checkpoint（驗收）：** 能解釋程式如何載入、排程並取得記憶體。

## Month 2 — Linux and Windows Administration（第 2 個月：Linux 與 Windows 管理）

- Users, groups, permissions, ACLs（使用者、群組、權限與 ACL）。
- Processes, services, startup, scheduled tasks（程序、服務、啟動與排程工作）。
- Filesystems, logs, package management（檔案系統、日誌與套件管理）。
- PowerShell and Bash basics（PowerShell 與 Bash 基礎）。

**Linux Commands（Linux 命令）**
```bash
id
find
grep
chmod
chown
systemctl
journalctl
```

**Windows PowerShell**
```powershell
whoami /all
Get-Process
Get-Service
Get-NetTCPConnection
Get-NetIPConfiguration
```

**Lab：** 建立測試使用者、調整檔案權限、檢查服務，並比較 Linux 與 Windows 日誌。  
**Checkpoint：** 能排查簡單的權限或服務啟動問題。

## Month 3 — Computer Networking（第 3 個月：計算機網路）

- OSI and TCP/IP models（OSI 與 TCP/IP 模型）。
- Ethernet, MAC, ARP, IPv4/IPv6, subnetting（乙太網路、MAC、ARP、IP 與子網路切割）。
- DNS, DHCP, NAT, routing, switching（DNS、DHCP、NAT、路由與交換）。
- TCP handshake, UDP, ports（TCP 交握、UDP 與連接埠）。
- HTTP/HTTPS and TLS basics（HTTP/HTTPS 與 TLS 基礎）。

**Commands（命令）**
```bash
ip -br addr
ip route
ip neigh
ss -tulpn
nslookup example.com
curl -I https://example.com
```

**Lab：** 使用 Wireshark 擷取自己測試機產生的 DNS 與 TCP 交握封包。  
**Checkpoint：** 能解釋瀏覽器如何解析網域並建立 HTTPS 連線。

## Month 4 — Programming and Automation（第 4 個月：程式設計與自動化）

- Python basics, functions, modules, exceptions（Python 基礎、函式、模組與例外處理）。
- File handling, regular expressions, JSON, CSV（檔案處理、正規表示式、JSON 與 CSV）。
- Bash and PowerShell automation（Bash 與 PowerShell 自動化）。
- SQL basics and C memory concepts（SQL 基礎與 C 語言記憶體概念）。

**Labs：** 建立日誌分析器、檔案完整性檢查器，以及整理自己實驗掃描結果的腳本。  
**Checkpoint：** 能閱讀、修改並解釋一段小型程式。

## Month 5 — Security Fundamentals（第 5 個月：網路安全基礎）

- Confidentiality, integrity, availability（機密性、完整性與可用性）。
- Authentication, authorization, least privilege（身分驗證、授權與最小權限）。
- Hashing, symmetric and asymmetric cryptography（雜湊、對稱式與非對稱式密碼學）。
- TLS certificates and digital signatures（TLS 憑證與數位簽章）。
- CVE, CVSS, threat, vulnerability, risk（CVE、CVSS、威脅、弱點與風險）。
- Firewalls, IDS/IPS, logging, incident response（防火牆、入侵偵測／防禦、日誌與事件應變）。

**Checkpoint：** 能說明 vulnerability、exploit、threat 與 risk 的差異。

## Month 6 — Network Analysis and Asset Discovery（第 6 個月：網路分析與資產識別）

- Wireshark display filters and protocol analysis（Wireshark 顯示過濾器與協定分析）。
- Nmap host discovery and service identification in an owned lab（在自有靶場中進行主機探索與服務識別）。
- DNS, HTTP, TLS, and packet troubleshooting（DNS、HTTP、TLS 與封包排錯）。
- Read vendor advisories and map versions to known issues（閱讀廠商公告並核對軟體版本風險）。

**Lab Command（實驗命令）**
```bash
nmap -sV -oN lab-scan.txt <LAB_TARGET_IP>
```
只將佔位符替換成自己隔離靶場中的目標 IP。不得將此命令用於未授權的第三方網路。

**Checkpoint：** 能整理附有證據與範圍說明的資產及服務清單。

## Month 7 — Web Foundations（第 7 個月：Web 基礎）

- HTTP methods, headers, status codes（HTTP 方法、標頭與狀態碼）。
- Cookies, sessions, authentication（Cookie、工作階段與身分驗證）。
- HTML, JavaScript, browser security basics（HTML、JavaScript 與瀏覽器安全基礎）。
- REST APIs, JSON, CORS（REST API、JSON 與 CORS）。
- Reverse proxies and server-side request flow（反向代理與伺服器端請求流程）。

**Lab：** 啟動本機 Juice Shop；使用瀏覽器開發者工具與 Burp Suite Community 分析請求、回應、Cookie 與 Session。先練習 Proxy、HTTP history 與 Repeater。  
**Checkpoint：** 能解釋完整的 HTTP Request／Response 流程，並安全地將一筆本機靶場請求送到 Repeater。

## Month 8 — Web Application Security（第 8 個月：Web 應用程式安全）

學習以下弱點的成因、影響、驗證方式與防禦方法：

- SQL injection（SQL 注入）。
- Cross-site scripting / XSS（跨站指令碼）。
- CSRF（跨站請求偽造）。
- SSRF（伺服器端請求偽造）。
- Broken access control / IDOR（存取控制缺陷／不安全直接物件參照）。
- File upload and path traversal（檔案上傳與路徑遍歷）。
- Authentication and session weaknesses（身分驗證與工作階段弱點）。
- API security（API 安全）。

**Labs：** 僅在指定教學環境中使用 WebGoat、OWASP Juice Shop 與 PortSwigger Academy；逐項學習 SQL Injection、XSS、CSRF、IDOR、SSRF、檔案上傳與 API 安全。  
**Checkpoint：** 能解釋弱點根因、防禦方式，並撰寫包含前置條件、重現步驟、影響、證據與修復建議的報告。

## Month 9 — Host Security（第 9 個月：主機安全）

- Linux services, permissions, sudo, logs（Linux 服務、權限、sudo 與日誌）。
- Windows accounts, services, registry, event logs（Windows 帳號、服務、登錄檔與事件日誌）。
- Secure configuration and patch management（安全組態與修補程式管理）。
- Backup, hardening, endpoint monitoring（備份、系統強化與端點監控）。

**Checkpoint：** 強化一台測試虛擬機，並記錄設定變更前後的差異。

## Month 10 — Active Directory and Enterprise Networks（第 10 個月：Active Directory 與企業網路）

- AD DS hierarchy: objects, domains, trees, forests, OUs, sites, trusts（AD DS 階層：物件、網域、樹系、OU、站台與信任）。
- FSMO roles, Global Catalog, replication topology and SYSVOL / DFSR（FSMO 角色、通用類別目錄、複寫拓樸與 SYSVOL／DFSR）。
- Domain Controller, NTDS.dit, SYSVOL and Global Catalog（網域控制站、目錄資料庫、SYSVOL 與通用類別目錄）。
- DNS SRV records and DC locator（DNS SRV 記錄與網域控制站探索）。
- Kerberos tickets, TGT, TGS, SPN and time synchronization（Kerberos 票證、TGT、TGS、SPN 與時間同步）。
- LDAP / LDAPS, SMB, NTFS permissions and Group Policy（LDAP／LDAPS、SMB、NTFS 權限與群組原則）。
- Privileged groups, service accounts, Windows LAPS and security auditing（高權限群組、服務帳號、Windows LAPS 與安全稽核）。

**Reading（閱讀）：** 先讀 [AD DS 核心架構](09-Active-Directory/README.md)，再讀 [AD 進階架構](09-Active-Directory/04-Advanced-Architecture.md)。

**Labs（實驗）：** 依照 [AD Lab Setup](09-Active-Directory/02-Lab-Setup.md) 在 VMware Host-only 網路建立一台 Windows Server DC 與一台 Windows Pro 用戶端；完成網域加入、測試 OU／群組建立、GPO 套用、Kerberos 票證觀察及事件日誌檢查。再依照 [AD Defensive Audit](09-Active-Directory/03-Defensive-Audit-and-Troubleshooting.md) 完成權限與健康狀態稽核。  
**Checkpoint：** 能畫出實驗網域拓樸、解釋 DNS 如何定位 DC、描述 Kerberos 票證流程，並提交包含命令輸出、預期結果、故障排查與修復建議的實驗報告。

## Month 11 — Authorized End-to-end Assessment（第 11 個月：授權綜合安全測試）

- Define scope, rules of engagement, and test accounts（定義測試範圍、測試規則與測試帳號）。
- Inventory assets and services（盤點資產與服務）。
- Validate findings safely with approved tools such as Burp Suite and carefully selected Metasploit auxiliary modules（使用 Burp Suite 與經過審查的 Metasploit 輔助模組，在授權範圍內安全驗證發現）。
- Review module side effects and avoid disruptive tests unless explicitly approved（檢查模組副作用；除非明確核准，否則避免可能造成中斷的測試）。
- Preserve evidence and avoid unnecessary impact（保留證據並避免不必要的影響）。
- Prioritize risk and recommend remediation（排列風險優先順序並提出修復建議）。

**Checkpoint：** 完成一份針對故意存在漏洞的靶場所撰寫的測試報告。

## Month 12 — Reporting and Portfolio（第 12 個月：報告與作品集）

完成以下成果：

1. Lab topology（實驗網路拓樸）。
2. System and network notes（系統與網路筆記）。
3. Reproducible lab instructions（可重現的實驗步驟）。
4. Troubleshooting records（故障排查紀錄）。
5. Security findings and remediation guidance（安全發現與修復建議）。
6. Final assessment report（最終安全測試報告）。

## Extension Tracks（延伸學習階段）

完成第 12 個月的授權測試與作品集後，可繼續以下三個階段。若每週只有 10–12 小時，建議每個階段安排 4–6 週，依實驗完成度調整。

### Month 13 — Incident Response and Digital Forensics（第 13 個月：事件應變與數位鑑識）

- NIST CSF 2.0 and SP 800-61 Rev.3（NIST 資安框架與事件應變指南）。
- Incident triage, severity, evidence preservation and chain of custody（事件分流、分級、證據保全與證據保管鏈）。
- Windows Event Log, PowerShell and Linux journal analysis（Windows 事件日誌、PowerShell 與 Linux 日誌分析）。
- Containment, eradication, recovery and lessons learned（遏止、根除、復原與事後檢討）。

**Lab：** 完成 [Windows Incident Response Lab](12-Incident-Response/02-Windows-Incident-Response-Lab.md)，提交時間線、雜湊清單與事件報告。  
**Checkpoint：** 能區分事實、假設與已確認發現，並解釋為何證據保全與授權決策不可省略。

### Month 14 — SIEM and Detection Engineering（第 14 個月：SIEM 與偵測工程）

- Log sources, collection, parsing, normalization and enrichment（日誌來源、收集、解析、正規化與資料豐富化）。
- Detection rules, correlation, false positives and coverage（偵測規則、事件關聯、誤報與覆蓋率）。
- Wazuh architecture, Windows Agent and Sysmon（Wazuh 架構、Windows Agent 與 Sysmon）。
- Microsoft Sentinel, KQL, analytics rules and incidents（Microsoft Sentinel、KQL、分析規則與事件案件）。
- MTTD, MTTA, MTTR and log source health（偵測、確認、回應時間與日誌來源健康度）。

**Lab：** 完成 [Wazuh SIEM Lab](13-SIEM-and-Detection/02-Wazuh-Lab.md)，收集 Windows Security 與 Sysmon 事件，撰寫一條偵測規則並分析誤報。  
**Checkpoint：** 能說明 SIEM 資料流程，並從原始日誌走到可調查的告警。

### Month 15 — Cloud Security（第 15 個月：雲端安全）

- IaaS, PaaS, SaaS and shared responsibility（雲端服務模型與共同責任）。
- AWS IAM, Azure Entra ID, RBAC, MFA and workload identities（AWS／Azure 身分、權限與工作負載身分）。
- VPC／VNet, network segmentation and private access（雲端網路分段與私人存取）。
- Storage security, encryption, secrets and key management（儲存安全、加密、秘密與金鑰管理）。
- Cloud audit logs, threat detection, incident response and cost controls（雲端稽核日誌、威脅偵測、事件應變與成本控制）。

**Lab：** 完成 [Cloud Security Lab](14-Cloud-Security/02-Cloud-Security-Lab.md)，選擇 AWS 或 Azure，完成身分、網路、儲存與日誌安全檢查。  
**Checkpoint：** 能畫出雲端安全架構、解釋責任邊界，並提交不含任何秘密或真實資產資料的稽核報告。

## Weekly Study Balance（每週時間分配）

| Activity（學習內容） | Hours（小時） |
|---|---:|
| Theory and documentation（理論與文件） | 3 |
| Commands and coding（命令與程式設計） | 2 |
| Hands-on labs（實作實驗） | 4 |
| Troubleshooting and notes（故障排查與筆記） | 2 |

## Lab Report Template（實驗報告範本）

每次實驗都記錄：

- Objective（實驗目的）
- Environment and network scope（實驗環境與網路範圍）
- Theory（理論原理）
- Commands and configuration（命令與設定）
- Expected output（預期輸出）
- Actual output（實際輸出）
- Errors and troubleshooting（錯誤與排查）
- Security impact（安全影響）
- Defensive measures（防禦措施）
- Sources（資料來源）

未經明確授權，不掃描或利用任何第三方系統。故意存在漏洞的靶機必須隔離；不要將密碼、權杖、私鑰或 VPN 訂閱網址提交到 Git。
