# Burp Suite Community — Practical Guide（Burp Suite Community 實作手冊）

Burp Suite 是 Web 安全測試工具。它可以讓測試人員觀察、攔截、修改與重送 HTTP／HTTPS 請求。本手冊以 Community Edition 的基本手動測試功能為主。

**測試範圍：** 本機 OWASP Juice Shop／WebGoat，或明確授權的測試網站。不要將日常瀏覽器、私人帳號或未授權網站流量導入測試代理。

## 1. Main Components（主要功能）

| Component（功能） | 用途 |
|---|---|
| Proxy | 攔截與檢視瀏覽器和伺服器之間的 HTTP 流量 |
| HTTP history | 保存本次代理工作階段的請求與回應 |
| Repeater | 手動修改並重送單一請求 |
| Intruder | 在授權測試中對參數進行受控測試；Community 版功能與速度有限 |
| Decoder | 編碼與解碼資料 |
| Comparer | 比較兩段請求或回應 |
| Sequencer | 分析工作階段 Token 的隨機性；結果需謹慎解讀 |
| Logger | 檢視工具產生的流量與事件 |

Professional 版的功能與 Community 版不同。自動漏洞掃描等功能可能需要付費版本；不要把 Community 版描述成具有完整自動掃描能力。

## 2. Safe Lab Setup（安全實驗環境）

建議：
- 使用 Kali Linux 或官方支援的桌面環境。
- 啟動本機 Juice Shop 或 WebGoat。
- 使用獨立的測試瀏覽器設定檔，或 Burp 內建瀏覽器。
- 只代理靶場流量。
- 不匯入個人日常瀏覽器的 Cookie、密碼或憑證。

Juice Shop 本機範例請參考倉庫的 [Security Labs](../10-Security-Labs/README.md)。

## 3. Start Burp（啟動 Burp）

1. 從 PortSwigger 官方網站下載 Burp Suite Community Edition。
2. 依照官方安裝說明安裝。
3. 啟動 Burp，選擇 Temporary project（臨時專案）即可開始練習。
4. 開啟 Proxy 工具，確認 Listener 正在運作。
5. 初始常見代理位址為 `127.0.0.1:8080`，實際值以你目前的 Listener 設定為準。
6. 使用 Burp 內建瀏覽器開啟本機靶場。

官方下載與文件：
- https://portswigger.net/burp/communitydownload
- https://portswigger.net/burp/documentation

## 4. Proxy — Intercept a Request（攔截請求）

1. 在瀏覽器開啟本機靶場。
2. 在 Burp 的 Proxy → Intercept 檢查攔截狀態。
3. 在靶場中點擊一個普通頁面或測試商品。
4. 檢視請求方法、路徑、標頭、Cookie 與本文。
5. 使用 Forward（轉送）讓請求繼續。
6. 到 HTTP history 檢查完整請求與回應。

若瀏覽器無法載入頁面：
- 確認 Burp Listener 正在運作。
- 確認瀏覽器使用相同代理位址與連接埠。
- 確認目標靶場正在執行。
- 確認沒有把 Proxy 設定套用到不相關的日常瀏覽器。

## 5. HTTPS and Burp CA（HTTPS 與 Burp CA 憑證）

Burp 要檢查 HTTPS 流量，需要在專用測試瀏覽器中信任 Burp 的測試 CA 憑證。

安全做法：
- 優先使用 Burp 內建瀏覽器。
- 若使用獨立測試瀏覽器，嚴格按照官方文件安裝該測試 CA。
- 不要把測試 CA 安裝到日常主要瀏覽器或其他不需要代理的裝置。
- 實驗結束後，若曾在獨立瀏覽器安裝測試 CA，請移除或還原該測試設定。

如果出現憑證錯誤，先檢查瀏覽器代理與 CA 信任設定，不要直接關閉正式網站的憑證驗證。

## 6. Repeater — Manual Request Testing（手動請求測試）

Repeater 是最重要的手動分析功能之一。

練習流程：
1. 在 HTTP history 選擇一筆自己產生的靶場請求。
2. 送到 Repeater。
3. 先保留原始請求，記錄正常回應。
4. 一次只修改一個無害測試欄位。
5. 點擊 Send。
6. 比較 Status Code、Response Headers、Response Body 與回應時間。
7. 記錄修改前後差異。

分析問題：
- 伺服器是否重新驗證使用者輸入？
- 權限是否在伺服器端檢查？
- 錯誤回應是否洩露內部資訊？
- 回應是否包含不必要的敏感欄位？

不要對真實使用者資料進行修改或刪除測試。

## 7. Decoder and Comparer（編解碼與比較）

Decoder 可用於理解 URL Encoding、Base64、HTML Encoding 等資料表示方式。

重要觀念：
- Base64 是編碼，不是加密。
- URL Encoding 不會讓輸入變安全。
- 解碼結果仍應視為不可信資料。

Comparer 可比較兩份請求或回應，適合觀察不同角色、不同輸入或修復前後的差異。

## 8. Scope and Target Control（範圍與目標控制）

- 在 Burp Target 中只保留授權靶場。
- 使用 Scope 限制可被納入測試的主機與路徑。
- 不要因為 Proxy 正在運作，就認為所有經過的網站都可以測試。
- 測試過程定期檢查 HTTP history，確認沒有混入私人或非測試流量。
- 若不慎攔截到非測試流量，停止分析並避免保存或分享其中的敏感資料。

## 9. Troubleshooting（故障排查）

| 問題 | 可能原因 | 檢查方法 |
|---|---|---|
| 瀏覽器無法連線 | Proxy 位址或連接埠錯誤 | 比對瀏覽器與 Burp Listener 設定 |
| HTTPS 憑證錯誤 | 測試 CA 未正確安裝或不受信任 | 使用專用測試瀏覽器並核對官方步驟 |
| HTTP history 沒有內容 | 流量未經代理或攔截設定不符 | 確認瀏覽器代理與靶場 URL |
| Intercept 一直停住 | 請求等待手動轉送 | Forward 或在測試結束後關閉攔截 |
| 連接埠已被使用 | 其他程式佔用 Listener Port | 更換為可用連接埠並同步修改瀏覽器 |
| 只看到部分流量 | 應用程式使用其他連線方式或流量未走該代理 | 檢查應用程式網路設定與工具文件 |

## 10. Practice Checklist（練習清單）

- [ ] 啟動 Burp Community。
- [ ] 建立獨立測試瀏覽器。
- [ ] 連接本機 Juice Shop。
- [ ] 攔截一筆 GET 請求。
- [ ] 檢查 HTTP history。
- [ ] 將一筆請求送到 Repeater。
- [ ] 比較原始與修改後回應。
- [ ] 使用 Decoder 解讀一段測試字串。
- [ ] 設定 Target Scope。
- [ ] 寫出一份含證據與修復建議的測試紀錄。

## 11. Sources（資料來源）

- Burp Suite Community Download: https://portswigger.net/burp/communitydownload
- Burp Suite Documentation: https://portswigger.net/burp/documentation
- PortSwigger Web Security Academy: https://portswigger.net/web-security
- OWASP Web Security Testing Guide: https://github.com/OWASP/wstg

本手冊只用於自有靶場或明確授權的 Web 安全測試。
