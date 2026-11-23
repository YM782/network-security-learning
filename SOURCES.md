
# 網路安全學習資料來源（繁體中文導覽）

本頁把資料分成兩類：**GitHub 專案**與**GitHub 以外的學習網站**。不是每個資源都要從頭讀到尾；請依照建議階段使用。

## A. GitHub 優質專案

| 專案 | 中文說明 | 適合階段 | 正確用法 |
|---|---|---|---|
| [OSSU Computer Science](https://github.com/ossu/computer-science) | 開放式自學電腦科學課程，包含程式、系統、網路和資訊安全 | 中後期選讀 | 非初學者第一本教材。只挑與目前課程對應的章節，不必完成整個大學級課程 |
| [The-Art-of-Hacking/h4cker](https://github.com/The-Art-of-Hacking/h4cker) | Omar Santos 維護的網路安全資源庫，涵蓋攻防、鑑識、雲端、應用程式安全、工具和實驗 | 全程查詢；中後期較有用 | 當作大型索引和參考書，不要從第一個資料夾一路讀到最後 |
| [OWASP Web Security Testing Guide](https://github.com/OWASP/wstg) | OWASP 網站安全測試指南，介紹網站安全測試方法與檢查項目 | Web 安全階段 | 先學 HTTP，再依主題閱讀。穩定版 4.2 可作為固定參考；5.0 仍在開發 |
| [OWASP Cheat Sheet Series](https://github.com/OWASP/CheatSheetSeries) | OWASP 安全速查表，說明常見應用程式安全問題和防護建議 | Web 安全與防禦階段 | 用來查「如何安全實作／修補」，不要當成零基礎主教材 |
| [OWASP WebGoat](https://github.com/WebGoat/WebGoat) | OWASP 故意設計有漏洞的教學網站，透過課程學習網站安全 | Web 安全初中階 | 必須在本機隔離環境使用；預設限制在 localhost。依官方警告，不要讓脆弱靶場暴露到網際網路 |
| [OWASP Juice Shop](https://github.com/juice-shop/juice-shop) | 故意有漏洞的網站安全訓練平台，含循序教學和挑戰 | Web 安全初中階 | 先用官方教學模式和初級題目。不要直接挑戰高難度題目 |
| [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) | Web 安全與滲透測試常見測試字串、案例和參考資料 | 中後期 | 是參考手冊，不是初學課程。只在自己建立或明確授權的靶場閱讀和驗證 |
| [InternalAllTheThings](https://github.com/swisskyrepo/InternalAllTheThings) | Active Directory 與企業內網安全測試速查資料 | 內網安全階段 | 先學 Windows、網路、網域、帳號與權限，再使用 |
| [Orange-Cyberdefense/GOAD](https://github.com/Orange-Cyberdefense/GOAD) | Game of Active Directory：可練習企業網域安全的實驗室 | 進階內網階段 | 不適合現在直接安裝。需要多台虛擬機與資源；環境故意脆弱，必須完全隔離，絕不可暴露到公網 |

### GitHub 專案的使用優先順序

1. 先從本倉庫的 00–06 課程建立基礎。
2. 到 h4cker 查找目前章節的延伸資源，但先不要下載大量工具。
3. 到 OWASP WebGoat / Juice Shop 做受控網站練習。
4. 學會 HTTP 與 Web 基礎後，再查 WSTG 和 Cheat Sheet Series。
5. 學會 Windows 網域與基本網路後，才使用 InternalAllTheThings 和 GOAD。
6. PayloadsAllTheThings 和 OSSU 都是後續參考資料，不要求初學者一次看完。

## B. GitHub 以外的優質學習網站

| 網站／課程 | 中文說明 | 費用與特點 | 建議順序 |
|---|---|---|---|
| [Cisco Networking Academy — Introduction to Cybersecurity](https://www.netacad.com/courses/introduction-to-cybersecurity) | 思科網路安全入門，介紹威脅、漏洞、隱私、防護與職涯 | 官方頁面標示免費、自學、初級，約 6 小時；提供繁體中文等多種語言 | 最先修讀 |
| [Cisco Networking Academy — Networking Basics](https://www.netacad.com/courses/networking-basics) | 網路基礎，學習設備、媒介、協定、IP 和基本網路連線 | 自學課程；含練習與測驗，實際語言和開課形式以網站顯示為準 | 電腦基礎後 |
| [TryHackMe — Pre Security](https://tryhackme.com/path/outline/presecurity) | 從零開始的互動式課程，包含電腦、作業系統、命令列、網路、Web 和安全入門 | 有引導式實驗；平台部分內容或功能可能受方案限制 | 最適合當作主線實作課 |
| [TryHackMe — Cyber Security 101](https://tryhackme.com/path/outline/cybersecurity101) | 初級綜合安全課程，包含 Linux、Windows、網路、密碼學、攻防和防禦 | 比 Pre Security 更廣、更深入 | Pre Security 完成後 |
| [OverTheWire — Bandit](https://overthewire.org/wargames/bandit/) | 透過 SSH 闖關練習 Linux 命令列 | 適合絕對初學者；每一關都會用到檔案和命令列基礎 | Linux 第 5–8 週 |
| [PortSwigger Web Security Academy](https://portswigger.net/web-security/) | 專業 Web 安全學院，提供文章、互動實驗和進度追蹤 | 官方標示 100% 免費，適合循序練習網站漏洞 | HTTP 與 Burp 基礎後 |
| [HTB Academy — Information Security Foundations](https://academy.hackthebox.com/path/preview/information-security-foundations) | Hack The Box 的資訊安全基礎課程，涵蓋網路、Linux、Windows、Web 等 | 有免費模組，也有使用學習點數或付費方案的內容 | 基礎穩定後作為第二套教材 |
| [CyLab Security Academy](https://www.picoctf.org/) | 卡內基美隆大學的網路安全學習平台，承接 picoCTF 的教育資源 | 免費教育平台，透過 CTF 題目練習 | 完成基本命令列和安全概念後 |

### 重要的網站更新提醒

- TryHackMe 的舊版 Complete Beginner、Introduction to Cyber Security 等路徑已被新路徑取代；請優先使用目前的 Pre Security 和 Cyber Security 101，不要照著多年以前的影片找舊課程。
- picoCTF 在 2026 年轉向 CyLab Security Academy 品牌。舊帳號和進度的遷移方式，請以平台目前公告為準。
- HTB Academy 有免費和付費內容，不能把整個平台都當成免費課程。
- PortSwigger Web Security Academy 的學習內容和互動實驗免費，但仍應只在其授權實驗環境內操作。

## C. 我建議你實際只先用這 3 個網站

不要一次註冊八個平台，先減少資訊量：

1. **Cisco Introduction to Cybersecurity**：先建立網路安全的整體概念。
2. **TryHackMe Pre Security**：跟著互動課程學電腦、作業系統、網路和 Web。
3. **OverTheWire Bandit**：配合 Kali 練習 Linux 命令列。

完成這三項的基礎內容後，再加入 PortSwigger Web Security Academy。

## D. 如何判斷資料是否值得學

- 優先選官方文件、知名大學、專業安全組織和原作者維護的專案。
- 查看最後更新時間、版本、授權、Issue 和官方文件。
- 不把星星數量當成教學品質的唯一指標。
- 不把工具清單當成學習路線。
- 不直接執行陌生 GitHub 專案中的安裝腳本或二進位檔。
- 本倉庫以連結和中文導覽介紹原始資料，不複製整份受版權保護的教材。
