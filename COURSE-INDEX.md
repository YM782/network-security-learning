# Course Index（網路安全課程索引）

本頁是網路安全倉庫的課程入口。技術名稱保留英文原文，旁邊附繁體中文說明。

## Core Foundations（核心基礎）

- [Computer Fundamentals（計算機底層基礎）](01-Computer-Fundamentals/README.md)：CPU、暫存器、快取、記憶體、作業系統、程序與執行緒，附 Linux／Windows 檢查命令與實驗。
- [Linux and Windows Fundamentals（Linux 與 Windows 基礎）](02-Linux-Windows/README.md)：使用者、群組、權限、程序、服務、網路與日誌。
- [Computer Networking Fundamentals（計算機網路基礎）](03-Computer-Networking/README.md)：OSI、TCP/IP、MAC、IP、ARP、DNS、TCP/UDP、HTTP/HTTPS 與 Wireshark 實驗。

- [Programming for Cybersecurity（網路安全程式設計基礎）](04-Programming/README.md)：Python、Bash、PowerShell、SQL 與 C 語言。
- [Cybersecurity Fundamentals（網路安全基礎原理）](05-Cybersecurity-Fundamentals/README.md)：CIA、身分驗證、授權、密碼學、弱點與風險。
- [Network Analysis（網路流量分析與資產識別）](06-Network-Analysis/README.md)：Wireshark、Nmap、封包過濾、服務辨識與實驗紀錄。
- [Web Application Security（Web 應用程式安全）](07-Web-Security/README.md)：SQL Injection、XSS、CSRF、IDOR、SSRF、檔案上傳與 API 安全。
- [Active Directory Domain Services（Active Directory 網域服務）](09-Active-Directory/README.md)：網域、樹系、DNS、Kerberos、LDAP、SMB、OU、群組與 GPO。
- [Active Directory Lab Setup（AD 隔離實驗室建置）](09-Active-Directory/02-Lab-Setup.md)：VMware Host-only、Windows Server、網域控制站與用戶端加入網域。
- [AD Defensive Audit and Troubleshooting（AD 防禦稽核與故障排查）](09-Active-Directory/03-Defensive-Audit-and-Troubleshooting.md)：帳號、權限、SPN、GPO、事件日誌與網域健康檢查。
- [Advanced Active Directory Architecture（AD 進階架構）](09-Active-Directory/04-Advanced-Architecture.md)：FSMO、目錄複寫、站台、信任、Global Catalog、gMSA 與備份復原。

## Authorized Testing and Practical Labs（授權測試與實作）

- [Authorized Penetration Testing（授權滲透測試方法）](08-Penetration-Testing/README.md)：測試範圍、Rules of Engagement、資產盤點、弱點驗證、風險評估與報告。
- [Burp Suite Community Guide（Burp Suite 實作手冊）](08-Penetration-Testing/Burp-Suite.md)：Proxy、HTTP history、Repeater、Decoder、Scope 與排錯。
- [Metasploit Framework Guide（Metasploit 實驗手冊）](08-Penetration-Testing/Metasploit.md)：模組架構、msfconsole、參數檢查、工作區與安全實驗。
- [Security Labs（網路安全實驗室）](10-Security-Labs/README.md)：VMware Host-only、Docker、WebGoat 與 Juice Shop 隔離靶場。
- [Troubleshooting（故障排查手冊）](11-Troubleshooting/README.md)：系統、網路、Docker、VMware 與 Burp Suite 常見問題。

## Security Operations and Cloud（資安營運與雲端安全）

- [Incident Response（資安事件應變與數位鑑識）](12-Incident-Response/README.md)：NIST CSF 2.0、事件分級、調查、證據保全、遏止、根除與復原。
- [Windows Incident Response Lab（Windows 事件應變實驗）](12-Incident-Response/02-Windows-Incident-Response-Lab.md)：PowerShell、Windows Event Log、時間線與 SHA-256 證據管理。
- [SIEM and Security Monitoring（SIEM 與安全監控）](13-SIEM-and-Detection/README.md)：日誌收集、正規化、偵測工程、Wazuh、Microsoft Sentinel 與 KQL。
- [Wazuh SIEM Lab（Wazuh 安全監控實驗）](13-SIEM-and-Detection/02-Wazuh-Lab.md)：Ubuntu、Windows Agent、Sysmon 與事件調查。
- [Cloud Security（雲端安全）](14-Cloud-Security/README.md)：AWS、Azure、IAM、網路、資料保護、日誌與共同責任模型。
- [Cloud Security Lab（雲端安全實驗）](14-Cloud-Security/02-Cloud-Security-Lab.md)：雲端身分、網路、儲存、稽核與成本檢查。

## Reference（參考）

- [Study Roadmap（12 個月主線與延伸學習路線）](STUDY-ROADMAP.md)
- [Sources（原始資料來源）](SOURCES.md)

## Lab Safety（實驗安全）

所有掃描與測試只限於自己擁有或明確獲得授權的環境。故意存在漏洞的靶場必須隔離，不得暴露到公開網際網路。
