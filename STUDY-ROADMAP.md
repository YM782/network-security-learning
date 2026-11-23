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

- Domain controller, users, groups, Group Policy（網域控制站、使用者、群組與群組原則）。
- LDAP, Kerberos, DNS, SMB（LDAP、Kerberos、DNS 與 SMB）。
- Domain trust and access control（網域信任與存取控制）。
- Network segmentation and enterprise logging（網路分段與企業日誌）。

**Lab：** 在隔離虛擬網路中建立或使用 GOAD-Light／MINILAB。  
**Checkpoint：** 能繪製網域拓樸，並解釋驗證與授權流程。

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
