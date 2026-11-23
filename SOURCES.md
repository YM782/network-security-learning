# 網路安全學習資料來源與課程對照

本頁說明資料何時使用、先看什麼，以及如何與本倉庫配合。不要一次打開所有資源。真實漏洞文章優先閱讀原作者、官方安全團隊和可信研究機構的原始分析。

## 一、目前最適合的三個主平台

| 平台 | 用途 | 建議時機 | 官方連結 |
|---|---|---|---|
| Cisco Networking Academy | 網路和網路安全概念入門 | 第 1–4 週 | [Introduction to Cybersecurity](https://www.netacad.com/courses/introduction-to-cybersecurity) |
| TryHackMe Pre Security | 電腦、系統、網路和 Web 互動入門 | 第 1–12 週 | [Pre Security](https://tryhackme.com/path/outline/presecurity) |
| OverTheWire Bandit | Linux 命令列闖關 | 第 5–8 週 | [Bandit](https://overthewire.org/wargames/bandit/) |

先用這三個建立穩定節奏，不要同時註冊很多平台。

## 二、其他 GitHub 以外的資源

| 資源 | 內容 | 建議階段 | 使用方式 |
|---|---|---|---|
| [Cisco Networking Basics](https://www.netacad.com/courses/networking-basics) | 網路設備、協定、IP 和連線 | 第 9–12 週 | 配合模組 04 |
| [TryHackMe Cyber Security 101](https://tryhackme.com/path/outline/cybersecurity101) | 初級攻防和防禦知識 | 基礎完成後 | Pre Security 後再開始 |
| [PortSwigger Web Security Academy](https://portswigger.net/web-security/) | Web 安全文章和互動實驗 | 第 19–20 週以後 | 先學 HTTP，再做初級實驗 |
| [HTB Academy](https://academy.hackthebox.com/) | 網路、Linux、Windows 和安全模組 | 初級基礎完成後 | 注意部分內容需付費或學習點數 |
| [picoCTF](https://picoctf.org/) | CTF 題目和安全思維 | 第 13 週以後 | 先做初級題目 |
| [Microsoft Learn](https://learn.microsoft.com/en-us/windows/) | Windows 系統和安全官方文件 | 模組 01、02、10 | 查 Windows 專有名詞 |
| [Linux man-pages](https://man7.org/linux/man-pages/) | Linux 命令和系統呼叫參考 | 模組 03 | 查命令正式用法 |
| [Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/) | Wireshark 官方使用手冊 | 模組 04、07 | 配合封包觀察 |

## 三、真實漏洞研究文章（建議按順序閱讀）

| 文章 | 真實案例 | 你要學會什麼 | 對應模組 |
|---|---|---|---|
| [Bad Binder — Project Zero](https://projectzero.google/2019/11/bad-binder-android-in-wild-exploit.html) | CVE-2019-2215，Android Binder UAF | 從線索、版本差異和核心程式碼縮小範圍 | 01 |
| [PrintNightmare — Microsoft MSRC](https://www.microsoft.com/en-us/msrc/blog/2021/07/clarified-guidance-for-cve-2021-34527-windows-print-spooler-vulnerability) | CVE-2021-34527，Windows Print Spooler | 服務權限、信任邊界、修補和配置 | 02 |
| [Dirty Pipe — Max Kellermann](https://dirtypipe.cm4all.com/) | CVE-2022-0847，Linux Kernel | 從檔案異常、最小重現到 git bisect | 03 |
| [HTTP Desync Attacks — PortSwigger](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn) | HTTP Request Smuggling（漏洞類別研究，並非單一 CVE） | 多層代理、協議解析差異和安全驗證 | 04 |
| [Effective Fuzzing — Project Zero](https://projectzero.google/2024/10/effective-fuzzing-dav1d-case-study.html) | CVE-2024-1580，dav1d 整數溢位 | Fuzzer 配置、輸入邊界、覆蓋率和 Sanitizer | 05 |
| [Baron Samedit — Qualys](https://blog.qualys.com/vulnerabilities-threat-research/2021/01/26/cve-2021-3156-heap-based-buffer-overflow-in-sudo-baron-samedit) | CVE-2021-3156，sudo Heap Overflow | 命令列參數解析和記憶體邊界 | 03 |
| [Practical Web Cache Poisoning — PortSwigger](https://portswigger.net/research/practical-web-cache-poisoning) | Web Cache Poisoning | 找出未納入 Cache Key 的輸入及其影響 | 04 |
| [This Shouldn't Have Happened — Project Zero](https://projectzero.google/2021/12/this-shouldnt-have-happened.html) | NSS BigSig（CVE-2021-43527） | 端到端測試、輸入大小上限和覆蓋率指標如何影響 Fuzzing | 05 |

閱讀時先看研究員如何提出假設、取得證據和縮小範圍；不要只複製文章中的 PoC。

## 四、GitHub 優質專案

| 專案 | 中文說明 | 難度 | 使用方式 |
|---|---|---|---|
| [OSSU Computer Science](https://github.com/ossu/computer-science) | 完整自學電腦科學課程 | 中高 | 未來按需選課 |
| [h4cker](https://github.com/The-Art-of-Hacking/h4cker) | 綜合網路安全資源索引 | 中級至進階 | 查找主題，不當線性教材 |
| [OWASP WSTG](https://github.com/OWASP/wstg) | Web 安全測試指南 | 中級 | 學 Web 後閱讀 |
| [OWASP Cheat Sheet Series](https://github.com/OWASP/CheatSheetSeries) | 安全開發與防護速查 | 中級 | 查修復方式 |
| [OWASP WebGoat](https://github.com/WebGoat/WebGoat) | 故意有漏洞的 Web 教學靶場 | 初中級 | 只在本機隔離環境 |
| [OWASP Juice Shop](https://github.com/juice-shop/juice-shop) | 故意有漏洞的網站訓練平台 | 初中級 | 從教學模式開始 |
| [Google OSS-Fuzz](https://github.com/google/oss-fuzz) | 開源軟體持續 Fuzzing 基礎設施 | 中高 | 先理解 Fuzzing 後再研究 |
| [LLVM LibFuzzer](https://llvm.org/docs/LibFuzzer.html) | 覆蓋率導向的 Fuzzing 引擎 | 中高 | 先看基本概念 |
| [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) | Web 測試案例參考 | 中高 | 僅在授權靶場使用 |
| [InternalAllTheThings](https://github.com/swisskyrepo/InternalAllTheThings) | AD 和內網安全速查 | 進階 | 先學 Windows 網域 |
| [GOAD](https://github.com/Orange-Cyberdefense/GOAD) | 故意有弱點的 AD 實驗環境 | 進階 | 多台 VM、隔離網路，現在不要安裝 |

## 五、本倉庫課程對應來源

| 本倉庫課程 | 優先來源 |
|---|---|
| 01 電腦基礎 | TryHackMe Pre Security、Microsoft Learn、Project Zero |
| 02 Windows / VMware / Kali | Microsoft Learn、VMware 與 Kali 官方文件、Microsoft MSRC |
| 03 Linux | OverTheWire Bandit、Linux man-pages、Dirty Pipe 原始文章 |
| 04 網路 | Cisco Networking Basics、Wireshark 文件、PortSwigger Research |
| 05 Python | Python 官方教學、Google OSS-Fuzz、Project Zero Fuzzing 文章 |
| 06 安全概念 | Cisco Introduction to Cybersecurity |
| 07 工具 | Wireshark、Nmap 官方文件 |
| 08 Web 安全 | PortSwigger、OWASP WSTG、Cheat Sheets |
| 09 滲透測試 | OWASP 測試指南和授權實驗平台 |
| 10 防禦監控 | Microsoft Learn、Wazuh 官方文件 |
| 13 進階底層 | OSTEP、MIT xv6、Intel 官方手冊 |

## 六、資料品質檢查

優先選官方、大學、專業安全組織和原作者資料；確認更新、版本、授權、先修要求和安全實驗環境。GitHub 星星數量不能代表教學品質。原始教材的內容和授權歸原作者，本倉庫提供繁體中文導覽與自製練習。
