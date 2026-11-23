# 網路安全學習路線｜零基礎到綜合網路安全

> **目標：建立一套真正可跟著操作的繁體中文教材。** 不以 Markdown 數量或字數充當品質；每個主題要能讓初學者理解原理、完成實驗、看懂輸出並排查常見問題。

## 最終學習目標

從電腦底層開始，逐步掌握 Windows、Linux、網路、程式設計和資訊安全，之後再進入 Web 安全、授權滲透測試、內網安全、防禦監控、事件應變和漏洞研究。

## 從這裡開始

1. [00 — 如何使用課程](00-START-HERE/README.md)
2. [01 — 認識電腦](01-COMPUTER-BASICS/README.md)
3. [02 — Windows、VMware 與 Kali](02-WINDOWS-VMWARE-KALI/README.md)
4. [03 — Linux 命令列入門](03-LINUX-BASICS/README.md)
5. [04 — 電腦網路基礎](04-NETWORKING-BASICS/README.md)
6. [24 週詳細學習路線](STUDY-ROADMAP.md)
7. [資料來源和網站使用順序](SOURCES.md)

## 課程目錄

| 模組 | 內容 | 建議階段 |
|---|---|---|
| [01 電腦基礎](01-COMPUTER-BASICS/README.md) | CPU、RAM、SSD、作業系統、程序和工作管理員實驗 | 現在 |
| [02 Windows / VMware / Kali](02-WINDOWS-VMWARE-KALI/README.md) | 宿主機、虛擬機、快照、網路模式、Kali 檢查 | 現在 |
| [03 Linux 入門](03-LINUX-BASICS/README.md) | 終端機、路徑、檔案、權限、程序和套件管理 | 基礎 |
| [04 網路基礎](04-NETWORKING-BASICS/README.md) | 網路設備、IP、DNS、DHCP、路由、TCP/UDP、HTTP、Wireshark | 基礎 |
| [05 程式設計](05-PROGRAMMING-BASICS/README.md) | Python、檔案、JSON、日誌 | 初級 |
| [06 安全概念](06-CYBERSECURITY-FUNDAMENTALS/README.md) | CIA、威脅、漏洞、風險、權限 | 初級 |
| [07 安全工具](07-SECURITY-TOOLS/README.md) | Wireshark、Nmap、Burp、Goby 等 | 初級至中級 |
| [08 Web 安全](08-WEB-SECURITY/README.md) | HTTP、Cookie、Session、常見漏洞 | 中級 |
| [09 滲透測試](09-PENTESTING-BASICS/README.md) | 授權、範圍、驗證、證據、報告 | 中級 |
| [10 防禦監控](10-DEFENSIVE-SECURITY/README.md) | Windows 日誌、防火牆、告警 | 初級至中級 |
| [11 實作實驗](11-PRACTICE-LABS/README.md) | 按順序完成的練習 | 全程 |
| [12 故障排查](12-TROUBLESHOOTING/README.md) | Linux、網路、VMware 和工具錯誤 | 全程 |
| [13 進階資料庫](13-ADVANCED-LATER/README.md) | 作業系統底層、內網和漏洞研究 | 以後 |

## 教材重寫進度

目前已完成前四個基礎模組的第一輪重寫。其他模組仍需逐篇擴充；未標記完成的內容仍屬初版筆記，不代表已達到完整教材標準。

| 模組 | 狀態 |
|---|---|
| 01 電腦基礎 | 已重寫 4 篇完整課程 |
| 02 Windows / VMware / Kali | 已重寫 3 篇完整課程 |
| 03 Linux 入門 | 已重寫 4 篇完整課程 |
| 04 網路基礎 | 已重寫 5 篇完整課程 |
| 05–10 | 待按順序深化 |
| 11–13 | 待按先修能力深化 |

## 每篇完整教材的最低標準

1. 學習目標、先修知識和難度。
2. 專有名詞中英對照與白話解釋。
3. 技術原理、架構和工作流程。
4. 命令逐條解析：用途、參數、執行位置、權限要求和風險。
5. 預期輸出及如何閱讀結果。
6. 從準備環境到完成驗證的完整實驗。
7. 常見錯誤、可能原因和安全排查步驟。
8. 實際應用場景及安全意義。
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
