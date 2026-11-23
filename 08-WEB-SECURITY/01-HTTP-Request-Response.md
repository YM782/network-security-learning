
# 08-01｜HTTP 請求、回應、Cookie 和 Session

## 本課目標
讀懂基本 HTTP 請求和回應，理解 Cookie 和 Session 的用途，為後面的 Web 安全打底。

## 1. HTTP 請求包含什麼？
- Method：請求方法，例如 GET、POST。
- Path：要存取的路徑。
- Headers：附加資訊，例如內容類型和瀏覽器資訊。
- Body：請求本文，常用於傳送表單或 API 資料。

## 2. 常見 HTTP 方法
| 方法 | 常見用途 |
|---|---|
| GET | 取得資源 |
| POST | 提交資料或觸發處理 |
| PUT | 建立或完整替換資源（依 API 設計） |
| PATCH | 部分更新資源 |
| DELETE | 刪除資源 |

實際行為仍取決於伺服器和 API 設計，不能只看方法名稱推斷權限。

## 3. HTTP 回應
伺服器回應通常包含狀態碼、標頭和本文。
- 200：成功。
- 301 / 302：重新導向。
- 400：請求格式或內容有問題。
- 401：需要驗證或驗證未通過。
- 403：伺服器拒絕授權存取。
- 404：找不到資源。
- 500：伺服器內部錯誤。

## 4. Cookie 和 Session
Cookie 是瀏覽器保存並在符合條件時送回伺服器的一小段資料。
Session 是伺服器維持使用者狀態的常見方式。Cookie 常用來保存 Session ID，但實作方式不只一種。

## 5. 瀏覽器觀察實驗
1. 開啟 Chrome。
2. 按 F12。
3. 點 Network。
4. 重新載入一個正常網站。
5. 點選一筆請求。
6. 找到 Request Method、Status Code 和 Headers。
不要分享 Cookie、Authorization、Session Token 或私人資料。

## 6. 本課驗收
能解釋 GET、POST、200、403、404、Request Header、Response Header、Cookie 和 Session。
