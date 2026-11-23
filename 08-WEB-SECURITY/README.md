
# 08 — Web 網站安全入門

## 本課目標

理解網站如何工作，再學網站常見漏洞。不要在還不知道 HTTP 是什麼時就直接背 SQL Injection 或 XSS 的測試字串。

## 1. 網站如何工作？

當你在瀏覽器輸入網址：

1. 瀏覽器解析網址。
2. 透過 DNS 找到伺服器位址。
3. 與伺服器建立網路連線。
4. 傳送 HTTP 請求。
5. 伺服器處理請求並回傳回應。
6. 瀏覽器解析 HTML、CSS、JavaScript 並顯示頁面。

## 2. HTTP 請求包含什麼？

常見元素：

- Method（方法）：例如 GET、POST。
- Path（路徑）：要存取的資源。
- Headers（標頭）：瀏覽器和伺服器交換的附加資訊。
- Body（本文）：常用於傳送表單或 API 資料。

伺服器回應常包含：

- Status Code（狀態碼）：例如 200、301、403、404、500。
- Headers（標頭）。
- Response Body（回應本文）。

## 3. Cookie 和 Session

- Cookie：瀏覽器保存並在符合條件時送回伺服器的一小段資料。
- Session：伺服器用來維持使用者狀態的一種常見機制。
- Authentication：確認使用者身分。
- Authorization：判斷使用者能存取什麼。

若 Session 管理或權限檢查設計錯誤，可能造成帳號或資料安全問題。

## 4. 初學者先認識的漏洞

| 漏洞類型 | 白話解釋 | 防護方向 |
|---|---|---|
| SQL Injection | 把不可信輸入錯誤地當成資料庫指令 | 參數化查詢、權限控制 |
| XSS | 網站把不可信內容當成可執行的瀏覽器程式碼 | 輸出編碼、輸入處理、CSP 等 |
| Broken Access Control | 使用者能做超出授權範圍的事 | 伺服器端逐次檢查權限 |
| Authentication Failure | 登入、驗證或工作階段管理不安全 | 安全驗證流程、多因素驗證 |
| Path Traversal | 使用者輸入影響檔案路徑，可能讀取不該讀的檔案 | 路徑正規化、允許清單和權限限制 |

現在只理解成因和防護，不需要對真實網站試驗。

## 5. 正確的實作順序

1. 先學 HTTP 請求和回應。
2. 學會用瀏覽器開發者工具查看 Network。
3. 學 Cookie、Session、登入與權限。
4. 安裝 Burp Suite Community，先只觀察自己網站的請求。
5. 到 PortSwigger Web Security Academy 完成初級教學。
6. 再使用 OWASP WebGoat 或 Juice Shop 的本機隔離環境。

## 6. 推薦練習平台

- [PortSwigger Web Security Academy](https://portswigger.net/web-security/)：免費文章和互動實驗。
- [OWASP WebGoat](https://github.com/WebGoat/WebGoat)：有引導課程的故意脆弱網站。
- [OWASP Juice Shop](https://github.com/juice-shop/juice-shop)：有教學模式和不同難度的挑戰。

WebGoat 和 Juice Shop 都是故意有漏洞的軟體。請只在隔離的本機環境使用，遵守官方安全說明，不要公開暴露服務。

## 7. 本課驗收

1. GET 和 POST 大致用來做什麼？
2. HTTP 回應中的狀態碼代表什麼？
3. Cookie 和 Session 有什麼不同？
4. SQL Injection 和 XSS 的根本問題有什麼不同？
5. 為什麼要在伺服器端檢查權限？
6. 為什麼只能在授權靶場練習漏洞？
