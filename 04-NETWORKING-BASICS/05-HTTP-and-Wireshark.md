# 04-05｜HTTP、HTTPS、TLS 和 Wireshark 封包分析入門

> 難度：初級至初中級｜建議時間：120 分鐘｜先修：04-01 至 04-04
>
> 本課把網路基礎連接到 Web 安全。你將學習瀏覽器如何向伺服器發出 HTTP 請求、HTTPS 如何保護通訊，以及如何使用 Wireshark 觀察自己產生的 DNS 和 TCP 流量。

## 一、本課學習成果

完成後，你應該能夠：
1. 說明 HTTP Request 和 Response 的基本結構。
2. 認識常見 HTTP 方法和狀態碼。
3. 理解 HTTPS 和 TLS 的基本作用。
4. 使用 Chrome Developer Tools 查看自己的請求。
5. 使用 Wireshark 擷取自己產生的網路流量。
6. 分辨 Capture Filter 和 Display Filter。
7. 理解為什麼 HTTPS 封包通常看不到明文網頁內容。

## 二、HTTP 是什麼？

HTTP（Hypertext Transfer Protocol，超文字傳輸協定）是瀏覽器和 Web 伺服器常用的應用層協定。

當你打開網站時，瀏覽器會向伺服器發出請求，伺服器處理後回傳回應。瀏覽器再根據回應內容顯示頁面。

## 三、HTTP Request 請求的基本結構

一個典型 HTTP 請求可能包含：

1. Request Method：請求方法。
2. Request Target：請求的路徑或目標。
3. HTTP Version：協定版本。
4. Headers：標頭。
5. Body：本文（不一定存在）。

示意：

    GET /index.html HTTP/1.1
    Host: example.com
    Accept: text/html

這是教學用簡化示例。HTTP/2 和 HTTP/3 的線上傳輸表示方式與 HTTP/1.1 不同。

### 常見 HTTP 方法

| 方法 | 常見用途 |
|---|---|
| GET | 取得資源 |
| POST | 提交資料或要求伺服器處理 |
| PUT | 建立或完整替換資源（依 API 設計） |
| PATCH | 部分更新資源 |
| DELETE | 刪除資源 |
| HEAD | 取得回應標頭，不要求回傳一般本文 |
| OPTIONS | 查詢目標資源支援的通訊選項 |

方法名稱不等於權限檢查。伺服器仍必須驗證使用者身分和授權。

## 四、HTTP Response 回應的基本結構

伺服器回應通常包含：
- Status Code：狀態碼。
- Headers：回應標頭。
- Body：回應本文。

常見狀態碼：

| 狀態碼 | 意義 |
|---:|---|
| 200 | OK，請求成功 |
| 201 | Created，資源已建立 |
| 301 | Moved Permanently，永久重新導向 |
| 302 | Found，暫時重新導向 |
| 400 | Bad Request，請求格式或內容有問題 |
| 401 | Unauthorized，需要驗證或驗證未通過 |
| 403 | Forbidden，伺服器拒絕授權存取 |
| 404 | Not Found，找不到資源 |
| 429 | Too Many Requests，請求過於頻繁 |
| 500 | Internal Server Error，伺服器內部錯誤 |
| 503 | Service Unavailable，服務暫時無法處理請求 |

狀態碼要結合回應本文和應用程式行為理解。401 和 403 在實務上常被誤用，不能只憑狀態碼判定完整原因。

## 五、HTTP Headers、Cookie 和 Session

### 1. Headers（標頭）

標頭提供請求或回應的附加資訊，例如：
- Host：目標主機名稱。
- User-Agent：客戶端識別資訊。
- Content-Type：本文資料類型。
- Authorization：驗證相關資訊。
- Set-Cookie：伺服器要求瀏覽器保存 Cookie。

### 2. Cookie

Cookie 是瀏覽器保存的小型資料，之後在符合網域、路徑、安全和其他條件時，可能隨請求送回伺服器。

常見安全屬性：
- Secure：只在安全傳輸條件下傳送。
- HttpOnly：一般頁面 JavaScript 不能直接讀取該 Cookie。
- SameSite：控制跨站請求時 Cookie 的傳送方式。

Cookie 不一定都是登入資訊，也可能用於偏好設定和追蹤。

### 3. Session

Session（工作階段）是伺服器維持使用者狀態的常見方式。Cookie 可能保存 Session ID，伺服器再依這個識別碼查找對應狀態。

Session ID 若外洩，可能造成帳號被冒用。因此不能把 Cookie 或 Authorization 標頭貼到公開論壇或 GitHub。

## 六、HTTPS 和 TLS 是什麼？

HTTPS 是使用 TLS 保護的 HTTP 通訊。

TLS（Transport Layer Security，傳輸層安全協定）主要提供：
- 機密性：防止旁觀者直接讀取受保護的通訊內容。
- 完整性：偵測傳輸過程中資料是否被未授權修改。
- 身分驗證：透過憑證等機制驗證伺服器身分。

HTTPS 不代表網站一定合法、可信或沒有漏洞。它主要保護通訊通道，不能保證網站程式本身安全。

## 七、瀏覽器實驗：Chrome Developer Tools

### 步驟

1. 開啟 Chrome。
2. 訪問一個你有權使用的正常網站。
3. 按 F12 或 Ctrl + Shift + I。
4. 選擇 Network（網路）分頁。
5. 重新載入頁面。
6. 點選一筆請求。
7. 查看 Headers、Status Code、Request Method 和 Timing。

### 觀察重點

- General：請求 URL、方法、狀態碼。
- Request Headers：瀏覽器送出的標頭。
- Response Headers：伺服器回傳的標頭。
- Payload：請求本文（若有）。
- Timing：請求不同階段花費的時間。

**隱私提醒：** 不要把 Cookie、Authorization、Session Token、真實帳號資料或私人 URL 分享給他人。

## 八、Wireshark 是什麼？

Wireshark 是網路封包分析工具。它可以從你有權限監控的網路介面擷取封包，並將協定欄位解析成容易閱讀的格式。

Wireshark 常用於：
- 分析 DNS 查詢。
- 觀察 TCP 三次握手。
- 查看封包來源和目的地。
- 排查網路連線問題。
- 學習 HTTP、TLS 和其他協定。

Wireshark 不是破解 HTTPS 的工具。若沒有合法的解密條件，TLS 保護的應用資料通常不會以明文呈現。

## 九、Capture Filter 和 Display Filter

兩者很容易混淆。

### Capture Filter（擷取篩選器）

在開始擷取前，決定哪些封包要被收集。設定錯誤可能導致封包根本沒有被保存。

### Display Filter（顯示篩選器）

封包擷取後，決定畫面顯示哪些封包。它不會刪除原始擷取資料。

初學者建議先不設定 Capture Filter，先擷取自己的測試流量，再使用 Display Filter 篩選。

## 十、Wireshark 實驗 A：觀察 DNS 查詢

### 實驗安全要求

只擷取自己電腦產生的流量。不要擷取公共 Wi-Fi 中他人的私人通訊，也不要分享包含敏感資料的 PCAP 檔案。

### 步驟

1. 開啟 Wireshark。
2. 找到目前使用中的 Wi-Fi 或 Ethernet 介面。
3. 雙擊該介面開始擷取。
4. 開啟自己的終端機。
5. 執行：

    nslookup example.com

6. 回到 Wireshark，停止擷取。
7. 在 Display Filter 輸入：

    dns

8. 點選一筆 DNS 查詢封包。
9. 展開 Domain Name System 欄位。

### 觀察

- 查詢的網域名稱。
- 查詢類型，例如 A 或 AAAA。
- DNS 伺服器位址。
- 回應是否成功。
- 是否有對應回應封包。

如果沒有看到 DNS，可能是 DNS 快取、加密 DNS、選錯介面或測試流量未在擷取期間產生。

## 十一、Wireshark 實驗 B：觀察 TCP 三次握手

### 步驟

1. 開始在自己的網路介面擷取。
2. 在瀏覽器訪問一個你有權使用的網站。
3. 停止擷取。
4. 使用 Display Filter：

    tcp

5. 找到建立連線時的封包。
6. 查看 TCP Flags。

常見旗標：
- SYN：建立連線的同步請求。
- ACK：確認收到資料或序號。
- FIN：正常關閉方向上的傳送。
- RST：重設連線。

如果封包很多，可以使用：

    tcp.flags.syn == 1

這個篩選器會顯示 SYN 旗標為 1 的 TCP 封包。要區分初始 SYN 和 SYN-ACK，還需要查看 ACK 旗標及封包方向。

## 十二、Wireshark 實驗 C：觀察 TLS

使用瀏覽器訪問 HTTPS 網站，再於 Wireshark 查看相關流量。

可先嘗試：

    tls

如果 Wireshark 已解析到 TLS，通常能看到握手相關資訊，例如 Client Hello、Server Hello 和憑證相關資料（依 TLS 版本和加密方式而異）。

你通常看不到受 TLS 保護的網頁本文明文。這是加密正常運作的結果，不代表 Wireshark 故障。

## 十三、常見故障排查

**問題 1：Wireshark 完全沒有封包。**

可能原因：選錯介面、介面未啟用、沒有擷取權限，或流量沒有經過該介面。先確認正在使用哪張網卡。

**問題 2：DNS 篩選器沒有結果。**

可能原因：DNS 快取、加密 DNS、使用其他解析器或測試時機不對。重新擷取並在擷取期間執行 nslookup。

**問題 3：HTTPS 只有 TLS 握手，沒有網頁內容。**

這通常是正常的。TLS 會加密應用資料；沒有合法的解密條件時，不能期待看到明文。

**問題 4：Display Filter 顯示紅色或語法錯誤。**

確認篩選器名稱和語法。先使用簡單的 dns、tcp 或 tls，再逐步增加條件。

## 十四、本課測驗

1. HTTP Request 和 Response 各包含哪些主要部分？
2. GET 和 POST 常見用途有什麼不同？
3. 401 和 403 大致代表什麼？
4. Cookie 和 Session 有什麼關係？
5. HTTPS 中 TLS 提供哪些主要安全能力？
6. Capture Filter 和 Display Filter 有什麼差別？
7. 為什麼 HTTPS 封包通常看不到網頁本文？
8. tcp.flags.syn == 1 主要篩選什麼封包？
9. 為什麼不應擷取或公開他人的私人網路通訊？

## 十五、參考資料

- [IETF RFC 9110：HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [IETF RFC 9112：HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9112)
- [IETF RFC 8446：TLS 1.3](https://www.rfc-editor.org/rfc/rfc8446)
- [Wireshark User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/)
- [Wireshark Display Filter Reference](https://www.wireshark.org/docs/dfref/)

**完成標準：** 能讀懂基本 HTTP 請求和回應，能使用 Chrome Network 面板觀察自己的請求，並能用 Wireshark 找到自己產生的 DNS 和 TCP 封包。