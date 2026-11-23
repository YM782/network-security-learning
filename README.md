# Network Security Learning（網路安全學習）

> A bilingual, source-linked curriculum from computer fundamentals to authorized security testing.
> 從計算機底層到授權安全測試的中英雙語系統課程，保留每份資料的原始來源。

## Learning Objective（學習目標）

建立完整能力：理解電腦與網路的運作原理、分析系統與流量、理解漏洞成因、在授權靶場驗證問題、排查故障，並撰寫專業安全報告。

本倉庫用於防禦性學習與安全教育。所有掃描與漏洞驗證都必須限定在自己擁有或已取得明確授權的系統。

## Repository Map（資料夾導覽）

| Folder / file（英文名稱） | 中文說明 |
|---|---|
| `01-Computer-Fundamentals/` | 計算機組成、CPU、記憶體、程序、執行緒與作業系統原理 |
| `02-Linux-Windows/` | Linux、Windows、Shell、權限、服務與日誌 |
| `03-Computer-Networking/` | OSI、TCP/IP、DNS、ARP、路由、交換與 TLS |
| `04-Programming/` | Python、Bash、PowerShell、SQL 與 C 語言基礎 |
| `05-Cybersecurity-Fundamentals/` | 安全原則、密碼學、認證、風險與漏洞管理 |
| `06-Network-Analysis/` | Wireshark、Nmap、流量分析與資產識別 |
| `07-Web-Security/` | HTTP、API、身分認證、SQL Injection、XSS、SSRF 等 |
| `08-Penetration-Testing/` | 授權測試流程、範圍管理、驗證與報告；另含 Burp Suite 與 Metasploit 詳細手冊 |
| `09-Active-Directory/` | AD DS 架構、DNS、Kerberos、LDAP、SMB、GPO、FSMO、複寫、隔離實驗與防禦稽核 |
| `10-Security-Labs/` | WebGoat、Juice Shop、GOAD 等隔離靶場 |
| `11-Troubleshooting/` | 系統、網路、虛擬機、Docker 與工具故障排查 |
| `SOURCES.md` | 原始 GitHub 專案、官方文件與課程來源 |
| `STUDY-ROADMAP.md` | 12 個月學習路線與階段驗收 |

以上為規劃中的資料夾；隨著學習進度逐步加入原創筆記、實驗報告與故障紀錄。

## Learning Principles（學習原則）

1. Understand the mechanism before memorizing commands（先理解原理，再記憶命令）。
2. Pair each theory topic with a reproducible lab（每個理論主題都搭配可重現的實驗）。
3. Record expected output, actual output, and troubleshooting steps（記錄預期輸出、實際輸出與排錯步驟）。
4. Prefer official documentation and original repositories（優先使用官方文件與原始專案）。
5. Never test third-party systems without explicit authorization（未經明確授權，不測試第三方系統）。

## Lab Safety（實驗安全）

- Use VMware host-only or another isolated virtual network for vulnerable machines（脆弱靶機使用 VMware Host-only 或其他隔離網路）。
- Take snapshots before risky changes（進行高風險變更前建立快照）。
- Do not expose WebGoat, Juice Shop, GOAD, or other intentionally vulnerable labs to the public internet（不要將故意存在漏洞的靶場暴露到公開網際網路）。
- Use only test accounts and synthetic data（只使用測試帳號與虛構資料）。
- Keep credentials, tokens, subscription links, and private keys out of Git（不要將密碼、權杖、訂閱連結與私鑰提交到 Git）。

## Start Here（從這裡開始）

1. Open [COURSE-INDEX.md](COURSE-INDEX.md)（課程索引）查看已建立的課程。
2. Read [STUDY-ROADMAP.md](STUDY-ROADMAP.md)（12 個月學習路線）。
3. Review [SOURCES.md](SOURCES.md)（原始資料來源）。
4. Begin with computer fundamentals and Linux, not exploitation tools（先從計算機基礎與 Linux 開始，不要一開始就學漏洞利用工具）。
