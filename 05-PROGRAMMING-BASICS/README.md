# 05 — Python 程式設計基礎：從零開始寫安全自動化和測試工具

> 本模組以 Python 3 為主。你不需要有程式設計經驗；先理解每一行程式，再逐步寫出能讀取檔案、整理 JSON、分析日誌和測試輸入邊界的小工具。

## 課程順序

1. [01 Python 環境、直譯器與第一個程式](01-Python-Environment.md)
2. [02 變數、資料型別、運算子和輸入輸出](02-Variables-and-Types.md)
3. [03 條件判斷、迴圈和程式流程](03-Control-Flow.md)
4. [04 清單、元組、字典、集合和函式](04-Collections-and-Functions.md)
5. [05 檔案、JSON、CSV 和例外處理](05-Files-JSON-CSV.md)
6. [06 正規表示式與離線日誌分析實驗](06-Regex-and-Log-Analysis.md)
7. [07 真實案例：Project Zero 如何透過 Fuzzing 找到 dav1d 漏洞（CVE-2024-1580）](07-Case-Study-Dav1d-Fuzzing.md)

## 本模組完成標準

- [ ] 能在 Windows 和 Kali 確認 Python 3 是否安裝。
- [ ] 能建立並執行 .py 檔案。
- [ ] 能理解字串、整數、浮點數、布林值和 None。
- [ ] 能使用 if、for、while 控制程式流程。
- [ ] 能使用 list、tuple、dict 和 set 儲存資料。
- [ ] 能定義函式、傳入參數並回傳結果。
- [ ] 能讀取文字檔、JSON 和 CSV。
- [ ] 能處理常見檔案和資料格式錯誤。
- [ ] 能用正規表示式找出日誌欄位。
- [ ] 能使用 ipaddress 驗證日誌中的 IP 字串。
- [ ] 能解釋 Fuzzing、種子資料、覆蓋率和 Sanitizer。
- [ ] 能完成離線日誌分析和輸入邊界檢查練習。

## 快速複習文件

- [Python 基本語法速覽](01-Python-Basics.md)
- [讀取文字檔、JSON 和日誌速覽](02-Read-Logs-and-JSON.md)

## Python 與真實漏洞研究

Python 常用於自動化資料處理、產生測試輸入、分析結果和建立研究輔助工具。但許多底層記憶體漏洞發生在 C/C++、核心或原生函式庫中，Python 並不會自動替代 C 語言、除錯器或 Sanitizer。

本模組的目標是先建立程式閱讀和測試思維，再逐步學習 Fuzzing 的實際工具鏈。

## 建議環境

- Windows：使用 PowerShell，透過 Python Launcher 的 py 命令或 python 命令執行。
- Kali Linux：使用終端機，透過 python3 命令執行。
- 編輯器：先使用已安裝的文字編輯器即可；熟悉後可選擇 VS Code。
- 套件：前幾課只使用 Python 標準函式庫，不需要額外安裝第三方套件。

## 實驗安全規則

1. 初學時使用自己建立的虛構資料，不要先讀取公司、他人或真實使用者的私人日誌。
2. 不要把密碼、Token、Cookie、VPN 訂閱連結或真實個資寫進程式碼。
3. 不要使用 eval() 執行使用者輸入的文字。
4. 不要為了安裝套件而隨意使用管理員權限；需要第三方套件時，先學會建立虛擬環境。
5. Fuzzing 只針對自己的程式、開源專案的授權測試環境或明確允許的目標。
6. 上傳 GitHub 前，檢查檔案是否含有秘密或私人資料。

## 官方參考資料

- [Python 官方下載頁](https://www.python.org/downloads/)
- [Python 3 官方教學](https://docs.python.org/3/tutorial/)
- [Python 標準函式庫](https://docs.python.org/3/library/)
- [Python 虛擬環境 venv](https://docs.python.org/3/library/venv.html)
- [Google OSS-Fuzz](https://github.com/google/oss-fuzz)
- [LLVM LibFuzzer](https://llvm.org/docs/LibFuzzer.html)

**完成標準：** 能獨立寫出一個讀取自己建立的測試日誌、統計 INFO/WARNING/ERROR 數量並驗證 IP 位址的小程式，並能解釋真實 Fuzzing 案例中的測試盲區。
