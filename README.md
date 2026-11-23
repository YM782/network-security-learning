# 網路安全學習路線｜零基礎到綜合網路安全

> **目標：建立一套真正可跟著操作的繁體中文教材。** 不以 Markdown 數量或字數充當品質；每個主題要能讓初學者理解原理、完成實驗、看懂輸出、排查常見問題，並逐步學會閱讀真實漏洞研究文章。

## 最終學習目標

從電腦底層開始，逐步掌握 Windows、Linux、網路、程式設計和資訊安全，之後再進入 Web 安全、授權滲透測試、內網安全、防禦監控、事件應變和漏洞研究。

## 從這裡開始

1. [00 — 如何使用課程](00-START-HERE/README.md)
2. [01 — 認識電腦](01-COMPUTER-BASICS/README.md)
3. [02 — Windows、VMware 與 Kali](02-WINDOWS-VMWARE-KALI/README.md)
4. [03 — Linux 命令列入門](03-LINUX-BASICS/README.md)
5. [04 — 電腦網路基礎](04-NETWORKING-BASICS/README.md)
6. [05 — Python 程式設計基礎](05-PROGRAMMING-BASICS/README.md)
7. [真實漏洞研究方法](VULNERABILITY-RESEARCH-PLAYBOOK.md)
8. [漏洞研究報告模板](VULNERABILITY-REPORT-TEMPLATE.md)
9. [24 週詳細學習路線](STUDY-ROADMAP.md)
10. [資料來源和網站使用順序](SOURCES.md)

## 真實漏洞研究入口

前五個模組已加入五篇真實漏洞研究案例。你將從記憶體生命週期、Windows 服務權限、Linux 核心資料管理、HTTP 協議解析和 Fuzzing 五個角度，學習研究人員如何提出假設、收集證據、建立最小重現、定位根因和理解修補。

| 模組 | 真實案例 | 核心研究能力 |
|---|---|---|
| 01 電腦基礎 | Android Binder UAF（CVE-2019-2215） | 記憶體生命週期、版本差異、變體分析 |
| 02 Windows / VMware / Kali | PrintNightmare（CVE-2021-34527） | 服務、權限邊界、修補與配置 |
| 03 Linux | Dirty Pipe（CVE-2022-0847） | Page Cache、最小重現、git bisect |
| 04 網路 | HTTP Request Smuggling | 代理鏈、協議解析差異、低風險驗證 |
| 05 Python | dav1d Fuzzing（CVE-2024-1580） | 測試空間、輸入邊界、覆蓋率、Sanitizer |

**建議先讀原理，再讀案例；先理解根因，再看 PoC。** 不要把能執行別人的 PoC 當成掌握漏洞。

## 課程目錄

| 模組 | 內容 | 建議階段 |
|---|---|---|
| [01 電腦基礎](01-COMPUTER-BASICS/README.md) | CPU、RAM、SSD、作業系統、程序、記憶體生命週期 | 現在 |
| [02 Windows / VMware / Kali](02-WINDOWS-VMWARE-KALI/README.md) | 宿主機、虛擬機、快照、網路模式、Windows 服務和權限 | 現在 |
| [03 Linux 入門](03-LINUX-BASICS/README.md) | 終端機、檔案、權限、程序、核心資料管理 | 基礎 |
| [04 網路基礎](04-NETWORKING-BASICS/README.md) | 網路設備、IP、DNS、DHCP、路由、TCP/UDP、HTTP、Wireshark | 基礎 |
| [05 Python 程式設計](05-PROGRAMMING-BASICS/README.md) | Python 3、資料型別、控制流程、函式、檔案、JSON、CSV、Regex、Fuzzing | 初級 |
| [06 安全概念](06-CYBERSECURITY-FUNDAMENTALS/README.md) | CIA、威脅、漏洞、風險、權限 | 初級 |
| [07 安全工具](07-SECURITY-TOOLS/README.md) | Wireshark、Nmap、Burp、Goby 等 | 初級至中級 |
| [08 Web 安全](08-WEB-SECURITY/README.md) | HTTP、Cookie、Session、常見漏洞 | 中級 |
| [09 滲透測試](09-PENTESTING-BASICS/README.md) | 授權、範圍、驗證、證據、報告 | 中級 |
| [10 防禦監控](10-DEFENSIVE-SECURITY/README.md) | Windows 日誌、防火牆、告警 | 初級至中級 |
| [11 實作實驗](11-PRACTICE-LABS/README.md) | 按順序完成的練習 | 全程 |
| [12 故障排查](12-TROUBLESHOOTING/README.md) | Linux、網路、VMware 和工具錯誤 | 全程 |
| [13 進階資料庫](13-ADVANCED-LATER/README.md) | 作業系統底層、內網和漏洞研究 | 以後 |

## 教材重寫進度

前五個基礎模組目前共有 27 篇課程與真實案例文件：22 篇基礎課程，加上 5 篇真實案例分析。其他模組仍需逐篇擴充；未標記完成的內容仍屬初版筆記，不代表已達到完整教材標準。

| 模組 | 狀態 |
|---|---|
| 01 電腦基礎 | 4 篇基礎課程 + 1 篇真實案例 |
| 02 Windows / VMware / Kali | 3 篇基礎課程 + 1 篇真實案例 |
| 03 Linux 入門 | 4 篇基礎課程 + 1 篇真實案例 |
| 04 網路基礎 | 5 篇基礎課程 + 1 篇真實案例 |
| 05 Python 程式設計 | 6 篇基礎課程 + 1 篇真實案例 |
| 06–10 | 待按順序深化 |
| 11–13 | 待按先修能力深化 |

## 每篇完整教材的最低標準

1. 學習目標、先修知識和難度。
2. 專有名詞中英對照與白話解釋。
3. 技術原理、架構和工作流程。
4. 命令或程式逐段解析：用途、參數、執行位置、權限要求和風險。
5. 預期輸出及如何閱讀結果。
6. 從準備環境到完成驗證的完整實驗。
7. 常見錯誤、可能原因和安全排查步驟。
8. 真實案例的線索、假設、證據、根因和修補。
9. 基礎、理解和實操測驗。
10. 官方文件和可信的延伸資源。

篇幅依主題複雜度決定，不為湊字數重複內容；一篇課程如果無法支援讀者理解和實作，就不應標記為完成。

## 建議學習節奏

每次 30–45 分鐘，每週 3–5 次。先理解，再跟著操作，然後不看答案重做，最後完成驗收題。遇到不懂的術語或錯誤訊息，可以先停下來，不需要趕進度。

## 外部學習平台

- [Cisco Networking Academy](https://www.netacad.com/)：網路和網路安全入門。
- [TryHackMe Pre Security](https://tryhackme.com/path/outline/presecurity)：互動式零基礎課程。
- [OverTheWire Bandit](https://overthewire.org/wargames/bandit/)：Linux 命令列闖關。
- [PortSwigger Web Security Academy](https://portswigger.net/web-security/)：Web 安全免費實驗。
- [HTB Academy](https://academy.hackthebox.com/)：基礎完成後的專業訓練。

## 實驗安全規則

只測試自己的設備、自己建立的虛擬機或明確授權的靶場。故意有漏洞的靶場必須隔離，不要暴露到公網。不要把密碼、Token、VPN 訂閱連結、私人日誌或敏感設定檔上傳 GitHub。

本倉庫提供繁體中文導覽、自製筆記和練習；外部教材仍以原作者的內容、授權和版本為準。
