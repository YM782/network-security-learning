# Cybersecurity Fundamentals（網路安全基礎原理）

本課程建立安全思維。工具可以協助檢查與驗證，但若不理解機密性、完整性、可用性、身分驗證、授權與風險，就無法正確判斷一個問題是否真的構成安全弱點。

## 1. CIA Triad（CIA 資訊安全三要素）

| Principle（原則） | English meaning（英文解釋） | 中文說明 |
|---|---|---|
| Confidentiality（機密性） | Only authorized parties can access information. | 只有獲授權的人員或系統可以存取資料。 |
| Integrity（完整性） | Data remains accurate and is not altered improperly. | 資料不應遭到未授權或不當修改。 |
| Availability（可用性） | Systems and data are accessible when needed. | 系統與資料在需要時能正常使用。 |

例子：
- 機密性問題：私人檔案被未授權的人讀取。
- 完整性問題：資料庫紀錄遭到未授權修改。
- 可用性問題：服務因故障或流量攻擊而無法提供服務。

## 2. Authentication vs. Authorization（身分驗證與授權）

- Authentication（身分驗證）：確認使用者或系統「是誰」。
- Authorization（授權）：決定該身分「可以做什麼」。
- Accounting / Auditing（記錄與稽核）：保留操作紀錄，以便追蹤與調查。

例子：登入網站成功代表通過身分驗證，不代表使用者可以讀取其他人的私人訂單。伺服器仍必須逐項檢查授權。

## 3. Least Privilege（最小權限原則）

使用者、服務與程式只應取得完成工作所需的最低權限。

實踐方式：
- 日常工作不要一直使用 Administrator 或 root。
- 服務帳號只授予必要目錄與網路權限。
- 定期檢查不再需要的帳號與權限。
- 管理操作使用獨立帳號並保留稽核紀錄。

## 4. Encryption, Hashing, and Encoding（加密、雜湊與編碼）

| Concept（概念） | Purpose（目的） | Reversible?（可逆？） |
|---|---|---|
| Encryption（加密） | 使用金鑰保護資料機密性 | 有正確金鑰時可解密 |
| Hashing（雜湊） | 產生固定長度摘要，用於完整性檢查等 | 設計上不可直接反推原文 |
| Encoding（編碼） | 轉換資料表示方式，便於傳輸或儲存 | 通常可逆，不等於安全保護 |

不要把 Base64 說成加密。Base64 只是編碼，任何人都可以解碼。

### Hash Lab（雜湊實驗）

建立測試檔案：

```bash
printf 'security lab\n' > sample.txt
sha256sum sample.txt
```

修改檔案內容後再執行一次，觀察 SHA-256 摘要是否改變。

Windows PowerShell：

```powershell
"security lab" | Set-Content sample.txt
Get-FileHash .\sample.txt -Algorithm SHA256
```

## 5. Password Storage（密碼儲存）

安全系統不應以明文保存密碼。密碼儲存通常需要專門的 Password Hashing Function（密碼雜湊函式），例如 Argon2id、bcrypt 或 scrypt，並使用適當參數與唯一 Salt（鹽值）。

一般 SHA-256 雖然是安全雜湊演算法，但不適合單獨用來儲存使用者密碼，因為它速度太快，容易遭受大量猜測。

## 6. TLS and Certificates（TLS 與憑證）

TLS（Transport Layer Security，傳輸層安全協定）用來保護網路連線，提供加密、完整性與伺服器身分驗證。

瀏覽器連線出現憑證錯誤時，應檢查：
- 網域名稱是否符合憑證。
- 憑證是否過期。
- 憑證鏈是否完整。
- 系統時間是否正確。
- 信任根憑證是否正常。

不要為了方便而忽略正式服務的憑證警告。

## 7. Vulnerability, Threat, Risk（弱點、威脅與風險）

- Vulnerability（弱點）：系統或流程中的缺陷。
- Threat（威脅）：可能利用弱點造成損害的情況或行為者。
- Exploit（利用方式）：觸發弱點的技術或程式。
- Impact（影響）：成功利用後造成的後果。
- Risk（風險）：綜合可能性與影響後的風險程度。

CVE 是公開弱點識別編號；CVSS 是弱點嚴重性評分框架。CVSS 分數不能單獨代表某個組織的實際風險，還要考慮資產重要性、暴露範圍與緩解措施。

## 8. Threat Modeling（威脅建模）

可使用以下問題開始分析：
1. What are we building?（我們正在保護什麼系統？）
2. What can go wrong?（可能出現什麼問題？）
3. What are we doing about it?（有哪些防禦措施？）
4. Did we do enough?（措施是否足夠？）

先畫出資料流圖，標示使用者、服務、資料庫、信任邊界與敏感資料，再分析可能的威脅。

## 9. Knowledge Check（知識測驗）

1. 身分驗證成功是否代表使用者能讀取所有資料？為什麼？
2. 為什麼 Base64 不等於加密？
3. 為什麼密碼不應只使用一般 SHA-256 雜湊儲存？
4. CVSS 分數是否等於組織的實際風險？
5. CIA 三要素分別代表什麼？

## Sources（資料來源）

- OWASP Cheat Sheet Series: https://github.com/OWASP/CheatSheetSeries
- NIST Cybersecurity Framework: https://www.nist.gov/cyberframework
- NIST Digital Identity Guidelines: https://pages.nist.gov/800-63-3/
- CVE Program: https://www.cve.org/
- FIRST CVSS: https://www.first.org/cvss/

本頁為教育用途的原創摘要。實際安全控制應依系統風險與官方標準設計。
