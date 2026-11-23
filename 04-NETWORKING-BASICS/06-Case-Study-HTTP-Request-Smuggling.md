# 04-06｜真實漏洞研究案例：HTTP Request Smuggling（HTTP 請求走私）

> 難度：初級至中級｜建議時間：120–150 分鐘｜先修：IP、TCP、HTTP Request/Response、反向代理
>
> 本課研究網站前端與後端對 HTTP 訊息邊界的理解不一致，如何形成安全漏洞。所有動手實驗只使用 PortSwigger Academy 或自己搭建的隔離環境。

## 一、先讀真實研究文章

[HTTP Desync Attacks: Request Smuggling Reborn — PortSwigger Research](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn)

研究員 James Kettle 以真實案例說明 HTTP Request Smuggling 的研究方法、檢測策略、風險控制和防禦方式。文章不是只列 Payload，而是解釋如何理解多層 HTTP 基礎設施，以及如何避免在測試時影響其他使用者。

## 二、先理解現代網站的架構

很多網站不是由一台伺服器直接處理所有請求，而是由多個元件串接：

    瀏覽器
       |
       v
    CDN / WAF
       |
       v
    Reverse Proxy
       |
       v
    Load Balancer
       |
       v
    Backend Server
       |
       v
    Web Application

這些元件可能由不同軟體、不同版本或不同設定組成。

每一層都需要解析 HTTP 請求，理解請求從哪裡開始、在哪裡結束，以及下一個請求何時開始。

## 三、HTTP Request Smuggling 是什麼？

HTTP Request Smuggling（HTTP 請求走私）是一類請求解析差異問題。

假設前端代理和後端伺服器對同一段 HTTP 資料的結束位置有不同理解：

- 前端認為第一個請求已經結束。
- 後端卻認為後面還有一段資料屬於同一個請求。
- 下一段資料因此可能被重新解讀。

問題核心是：

**同一條連線上的 HTTP 訊息，前端和後端必須對邊界有一致的理解。**

如果不一致，就可能形成 HTTP Desynchronization（HTTP 去同步化）。

## 四、為什麼 Content-Length 和 Transfer-Encoding 值得研究？

HTTP/1.1 使用不同方式描述訊息長度，例如：
- Content-Length：指出訊息主體的長度。
- Transfer-Encoding：指定訊息傳輸編碼方式，例如 chunked。

如果代理和後端對這些標頭的處理方式不一致，就可能形成解析差異。

常見研究分類包括：
- CL.TE：前端與後端對 Content-Length、Transfer-Encoding 的優先理解不同。
- TE.CL：兩層系統的理解方向相反。
- TE.TE：兩層系統對 Transfer-Encoding 標頭的處理方式不一致。

這些名稱描述的是不同元件對請求邊界的解析差異，不是只要看到某個標頭就能判定漏洞。

## 五、研究員如何思考和發現？

### 第一步：先畫出請求路徑

不要一開始就發送大量特殊請求。

先了解：
- 是否存在 CDN？
- 是否存在 WAF？
- 是否使用反向代理？
- 是否存在多個後端？
- 是否重用後端連線？
- HTTP/1.1 和 HTTP/2 是否在不同層之間轉換？

### 第二步：確認解析邊界

研究員要問：

1. 前端如何判斷請求結束？
2. 後端如何判斷請求結束？
3. 兩者是否使用相同的解析規則？
4. 是否拒絕含糊或互相衝突的訊息？
5. 代理是否會修改、刪除或重新產生 HTTP 標頭？

### 第三步：建立安全的檢測假設

不要先追求影響最大的利用方式。

先在自己控制的環境中確認：
- 兩層系統是否出現解析差異？
- 是否能穩定重現？
- 是否能排除網路延遲和應用程式本身的異常？
- 是否會影響其他連線？

### 第四步：確認影響

只有在授權環境中，才進一步評估：
- 是否能造成請求錯配？
- 是否可能影響其他使用者請求？
- 是否可能繞過前端安全控制？
- 是否可能影響快取或後端路由？

如果測試可能影響其他使用者，就應停止擴大測試，改用隔離環境或供應商提供的測試方式。

## 六、為什麼這類漏洞容易被忽略？

因為每一層系統單獨看起來可能都正常。

例如：
- WAF 正常接受請求。
- 反向代理正常轉發資料。
- 後端伺服器正常解析資料。
- 但三者對訊息結束位置的理解不一致。

這種問題不是簡單的「某個輸入欄位沒有過濾」，而是多個元件之間的協議語義不一致。

這也是為什麼網路安全研究需要理解協定和架構，而不只是記住工具。

## 七、在 PortSwigger Academy 做合法實驗

官方免費實驗：

[PortSwigger Web Security Academy — HTTP Request Smuggling](https://portswigger.net/web-security/request-smuggling)

建議順序：
1. 先閱讀 HTTP Request Smuggling 的概念介紹。
2. 完成最初級的 CL.TE 或 TE.CL 實驗。
3. 觀察 Burp Repeater 中請求和回應的差異。
4. 記錄每次修改了哪個變數。
5. 完成後閱讀解答，理解為什麼原本的假設成立或不成立。
6. 再學習 HTTP/2 和前後端協議轉換相關案例。

只在 Academy 的指定實驗環境中操作。不要把練習中的請求直接傳送到不屬於你的網站。

## 八、實驗紀錄模板

| 項目 | 記錄內容 |
|---|---|
| 實驗環境 | PortSwigger 指定 Lab |
| 前端元件 | 依 Lab 描述記錄 |
| 後端元件 | 依 Lab 描述記錄 |
| HTTP 版本 | HTTP/1.1 或 HTTP/2 |
| 觀察到的差異 | 請求邊界或回應行為 |
| 重現次數 | 記錄成功和失敗次數 |
| 排除因素 | 網路延遲、連線重用、應用程式錯誤 |
| 影響 | 只記錄 Lab 中實際證明的結果 |
| 修復方向 | 統一解析規則、拒絕歧義請求、更新代理 |

## 九、修復與防禦

開發和維運人員可以從以下方向降低風險：

1. 前端和後端使用一致且受支援的 HTTP 解析實作。
2. 拒絕有歧義或互相衝突的訊息長度標示。
3. 代理在轉發前正規化請求，避免不同元件重新解讀不一致的資料。
4. 定期更新 CDN、WAF、反向代理和後端伺服器。
5. 減少不必要的後端連線重用，並依供應商建議配置協議轉換。
6. 在部署前以整條代理鏈進行端到端測試。

不能只更新其中一台伺服器，就假設整條鏈路已經安全。

## 十、延伸案例：Web Cache Poisoning

另一篇非常值得閱讀的文章：

[Practical Web Cache Poisoning — PortSwigger Research](https://portswigger.net/research/practical-web-cache-poisoning)

研究員先找出會影響回應、但沒有被納入快取鍵（Cache Key）的輸入，再確認回應是否會被快取，最後評估影響。

可以從中學習另一種研究思路：

- 找出輸入。
- 觀察輸入是否影響輸出。
- 理解中間層如何保存結果。
- 判斷其他使用者是否可能收到錯誤的快取內容。
- 在不影響真實使用者的前提下證明問題。

## 十一、本課驗收

- [ ] 能畫出 CDN、WAF、反向代理和後端的請求路徑。
- [ ] 能解釋 HTTP Request Smuggling 的核心原因。
- [ ] 能說明 Content-Length 和 Transfer-Encoding 的用途。
- [ ] 能解釋 CL.TE、TE.CL 和 TE.TE 是解析差異分類。
- [ ] 能在 PortSwigger Lab 中記錄一個實驗的假設、輸入和結果。
- [ ] 能說明為什麼真實網站測試必須避免影響其他使用者。
- [ ] 能提出至少三個防禦方向。

**本課最重要的一句話：** 真正的網路漏洞研究，常常不是找一個錯誤的封包，而是找出兩個正常工作的系統對同一份資料有不同理解。
