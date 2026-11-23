# Web Application Security（Web 應用程式安全）

本課程從 HTTP 與瀏覽器的工作方式出發，逐步理解常見 Web 弱點的成因、影響、驗證證據與修復方法。重點不是背 Payload，而是能回答：資料從哪裡來、經過哪些元件、在哪裡失去信任邊界，以及應該在哪一層修復。

**實驗範圍：** 只使用本機 OWASP Juice Shop、WebGoat、PortSwigger Web Security Academy，或明確授權的測試環境。不要對真實第三方網站嘗試下列測試。

## 1. Web Request Lifecycle（Web 請求生命週期）

典型流程：

1. 使用者在瀏覽器輸入網址或點擊操作。
2. 瀏覽器解析網域並建立連線。
3. TLS 在 HTTPS 情況下保護傳輸。
4. Web Server 或 Reverse Proxy 接收 HTTP Request。
5. 應用程式執行路由、身分驗證、授權與商業邏輯。
6. 應用程式可能查詢資料庫、快取或其他內部服務。
7. Server 回傳 HTTP Response，瀏覽器解析並呈現內容。

### Request Anatomy（請求結構）

```http
GET /products?id=12 HTTP/1.1
Host: lab.example.test
Accept: text/html
Cookie: session=TEST_SESSION
User-Agent: ExampleBrowser
```

主要部分：
- Method：GET、POST、PUT、PATCH、DELETE 等。
- Path：請求路徑。
- Query String：URL 中的查詢參數。
- Headers：內容型態、Cookie、認證資訊等中繼資料。
- Body：POST、PUT、PATCH 等請求可能包含的資料。

### Response Anatomy（回應結構）

```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{"id":12,"name":"Test product"}
```

檢查時要留意狀態碼、回應標頭、Cookie 屬性、回應本文與伺服器錯誤訊息。

## 2. Trust Boundaries（信任邊界）

瀏覽器送來的所有資料都應視為不可信，包括：
- URL path 與 query parameters。
- HTTP headers 與 cookies。
- Form fields、JSON body 與檔案上傳。
- 來自第三方 API、Webhook 或其他內部服務的資料。

不能只在前端 JavaScript 驗證資料。攻擊者可以繞過瀏覽器介面，直接建立 HTTP 請求。關鍵驗證必須在伺服器端完成。

## 3. SQL Injection（SQL 注入）

### Root Cause（成因）

當應用程式把使用者輸入直接串接進 SQL 指令，輸入內容就可能改變 SQL 的語法結構。

不安全的概念範例：

```python
# 不安全：不要在正式程式中這樣組合 SQL
query = "SELECT * FROM users WHERE name = '" + username + "'"
```

### Defensive Fix（修復方式）

使用 Parameterized Queries（參數化查詢），讓資料與 SQL 指令分開：

```python
cursor.execute(
    "SELECT id, name FROM users WHERE name = ?",
    (username,)
)
```

不同資料庫驅動的參數佔位符可能不同，請依官方文件調整。

### What to Verify（驗證重點）

- 查詢是否使用參數化方式。
- 資料庫帳號是否採最小權限。
- 錯誤訊息是否洩露 SQL、表名或資料結構。
- 是否有安全日誌與異常查詢監控。

不要把單一錯誤訊息或掃描器警告直接當成已確認的 SQL Injection；必須在授權環境中以可重現證據驗證。

## 4. Cross-Site Scripting — XSS（跨站指令碼）

### Root Cause（成因）

應用程式把不可信資料放入 HTML、JavaScript、URL 或其他瀏覽器可執行的上下文，卻沒有進行正確的 Context-aware Output Encoding（依輸出上下文編碼）。

常見類型：
- Reflected XSS（反射型）：惡意輸入在同一次請求的回應中被呈現。
- Stored XSS（儲存型）：輸入被保存，之後由其他使用者瀏覽時觸發。
- DOM-based XSS（DOM 型）：前端 JavaScript 對不可信資料進行不安全的 DOM 操作。

### Defensive Fix（修復方式）

- 依 HTML、屬性、JavaScript、CSS 或 URL 上下文使用正確編碼。
- 優先使用安全的 DOM API，例如 `textContent`，避免將不可信字串直接交給 `innerHTML`。
- 使用成熟的 Template Engine，確認自動跳脫功能已啟用。
- 部署 Content Security Policy（CSP）作為縱深防禦，而不是唯一防線。
- 不要只依靠移除尖括號等簡單字串過濾。

## 5. Cross-Site Request Forgery — CSRF（跨站請求偽造）

### Root Cause（成因）

使用者已登入某網站，瀏覽器會自動附帶 Cookie；若伺服器只依賴 Cookie 判斷請求可信，其他網站可能誘使瀏覽器送出非預期操作。

### Defensive Fix（修復方式）

- 對會改變狀態的請求使用不可預測且經伺服器驗證的 CSRF Token。
- 合理設定 Cookie 的 SameSite、Secure 與 HttpOnly 屬性。
- 驗證 Origin／Referer 作為額外防禦。
- 不要使用 GET 執行轉帳、修改密碼等狀態變更操作。
- 重要操作可要求重新驗證或額外確認。

SameSite Cookie 是重要防禦，但不能在所有情境中取代 CSRF Token 與正確的伺服器端設計。

## 6. Broken Access Control and IDOR（存取控制缺陷與不安全直接物件參照）

### Root Cause（成因）

應用程式只檢查使用者是否登入，卻沒有確認該使用者是否有權存取指定物件。

例如，使用者 A 可以看到自己的訂單；若伺服器接受客戶端傳來的另一個訂單 ID，卻沒有重新檢查擁有權，就可能造成資料越權。

### Defensive Fix（修復方式）

- 每次讀取、更新或刪除物件時，在伺服器端驗證使用者與物件的關係。
- 將授權檢查放在資料存取層或統一的 Policy Layer。
- 不要把隱藏按鈕、難猜的 ID 或前端路由當成授權機制。
- 對水平越權與垂直越權都設計測試案例。

### Safe Test Design（安全測試設計）

在本機靶場建立兩個測試帳號 A、B：
1. A 建立一筆測試資料。
2. B 登入後確認預設只能看到 B 自己的資料。
3. 在授權靶場中檢查伺服器是否正確拒絕 B 存取 A 的資料。
4. 保存請求、回應與預期權限規則。
5. 修復後重複測試，確認未授權請求被拒絕。

只使用虛構資料與測試帳號。

## 7. Server-Side Request Forgery — SSRF（伺服器端請求偽造）

### Root Cause（成因）

伺服器依照使用者提供的 URL 或主機名稱發起請求，卻沒有適當限制目的地。這可能讓伺服器接觸到原本不應由外部使用者存取的內部服務。

### Defensive Fix（修復方式）

- 優先使用目的地 Allowlist（允許清單），不要只依賴黑名單。
- 限制可使用的協定、連接埠與目的網域。
- 檢查 DNS 解析結果與重新導向，避免檢查與連線之間發生差異。
- 對內部網段、Loopback、Link-local 與雲端 Metadata 位址採取明確限制。
- 使用 Egress Filtering（出口流量控管）限制伺服器可連出的目的地。
- 將處理外部 URL 的功能放在受限制的網路環境中。

## 8. File Upload and Path Traversal（檔案上傳與路徑遍歷）

檔案上傳應檢查：
- 檔案大小與配額。
- 副檔名、MIME Type 與實際檔案格式。
- 檔案內容是否符合允許類型。
- 儲存位置是否在 Web Root 之外。
- 檔名是否由伺服器重新產生。
- 上傳檔案是否可能被當成程式執行。

Path Traversal（路徑遍歷）常發生於程式直接把使用者提供的路徑拼接到檔案系統路徑。應使用固定目錄、正規化路徑，並確認最終解析位置仍位於允許目錄之內。

## 9. Authentication and Session Security（身分驗證與工作階段安全）

檢查：
- 密碼是否以專用 Password Hashing Function 儲存。
- 是否有登入失敗節流與異常登入監控。
- 多因素驗證（MFA）是否保護重要帳號。
- Session ID 是否具有足夠隨機性。
- 登入或權限提升後是否更新 Session ID。
- 登出、密碼變更與帳號停用後是否使舊工作階段失效。
- Cookie 是否正確設定 Secure、HttpOnly 與 SameSite。

## 10. API Security（API 安全）

常見問題：
- Broken Object Level Authorization（物件層級授權缺陷）。
- Broken Authentication（身分驗證缺陷）。
- Excessive Data Exposure（回傳過多資料）。
- Unrestricted Resource Consumption（資源消耗未受限制）。
- Security Misconfiguration（安全設定錯誤）。
- Improper Inventory Management（API 資產盤點不足）。

API 防禦：
- 每個 Endpoint 都驗證身分與物件權限。
- 使用 Schema Validation（結構驗證）。
- 限制請求大小、速率與昂貴操作。
- 不回傳不必要的欄位。
- 管理 API 版本與淘汰流程。
- 對日誌中的 Token、密碼與個資進行遮罩。

## 11. Practical Lab — OWASP Juice Shop（實作：OWASP Juice Shop）

原始專案：https://github.com/juice-shop/juice-shop

建議在隔離靶場中完成以下學習：
1. 登入、登出與 Session 行為觀察。
2. 使用瀏覽器 Developer Tools 查看 Request／Response。
3. 使用 Burp Suite 攔截自己測試帳號的請求。
4. 觀察輸入驗證、錯誤回應與資料欄位。
5. 依照課程提示理解 XSS、SQL Injection、存取控制等弱點的成因。
6. 對每個問題寫出防禦方案，並在修復後重新測試。

## 12. Finding Report Template（弱點報告範本）

| 欄位 | 要記錄的內容 |
|---|---|
| Title（標題） | 清楚描述弱點 |
| Asset / Endpoint（資產／端點） | 測試範圍內的應用程式與路徑 |
| Preconditions（前置條件） | 帳號角色、設定與必要狀態 |
| Steps（重現步驟） | 可重現、最小化的測試流程 |
| Evidence（證據） | 請求、回應、截圖與時間 |
| Impact（影響） | 對機密性、完整性、可用性的影響 |
| Severity（嚴重性） | 根據實際影響與環境評估 |
| Root Cause（根本原因） | 程式或設計層面的成因 |
| Remediation（修復） | 具體修正方式 |
| Retest（複測） | 修復後的驗證結果 |

## 13. Knowledge Check（知識測驗）

1. 為什麼前端驗證不能取代伺服器端驗證？
2. SQL 參數化查詢如何避免資料被當成 SQL 語法？
3. XSS 的輸出編碼為什麼必須考慮上下文？
4. 登入成功為什麼不代表可以讀取任何使用者的資料？
5. SSRF 為什麼需要同時考慮 DNS、重新導向與出口流量？
6. SameSite Cookie 是否能完全取代 CSRF Token？為什麼？

## Sources（資料來源）

- OWASP Web Security Testing Guide: https://github.com/OWASP/wstg
- OWASP Cheat Sheet Series: https://github.com/OWASP/CheatSheetSeries
- OWASP Juice Shop: https://github.com/juice-shop/juice-shop
- OWASP WebGoat: https://github.com/WebGoat/WebGoat
- PortSwigger Web Security Academy: https://portswigger.net/web-security

本頁為原創學習筆記。所有實驗只限自有或明確授權的環境。
