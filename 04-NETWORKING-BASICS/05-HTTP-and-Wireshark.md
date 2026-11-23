
# 04-05｜HTTP、HTTPS 和封包觀察

## 本課目標
理解瀏覽器和網站伺服器如何交換資料，並初步認識 Wireshark。

## 1. HTTP 是什麼？
HTTP（Hypertext Transfer Protocol，超文字傳輸協定）是瀏覽器和 Web 伺服器常用的應用層協定。
基本互動：瀏覽器準備請求 → 伺服器接收和處理 → 伺服器回應 → 瀏覽器顯示內容。

## 2. HTTPS 是什麼？
HTTPS 是透過 TLS 保護的 HTTP 連線。TLS 提供加密、完整性保護和伺服器身分驗證。HTTPS 不代表網站一定可信，也不代表網站沒有漏洞。

## 3. 瀏覽器觀察實驗
1. 開啟 Chrome。
2. 按 F12 或 Ctrl + Shift + I。
3. 選 Network（網路）。
4. 重新載入一個正常網站。
5. 點選一筆請求，觀察 Method、Status、Headers 和 Timing。
不要分享 Cookie、Authorization 標頭或私人資料。

## 4. Wireshark 是什麼？
Wireshark 是網路封包分析工具，可顯示網路介面擷取到的封包，協助理解 DNS、TCP 和 TLS。它不是破解 HTTPS 的工具；正常 HTTPS 應用資料通常會加密。

## 5. 第一次封包觀察
1. 開啟 Wireshark。
2. 選擇自己的網路介面。
3. 開始擷取。
4. 在自己的終端機執行 nslookup example.com。
5. 停止擷取。
6. 使用顯示篩選器 dns。
7. 找到查詢名稱和回應。

只擷取自己有權限監控的流量。

## 6. 本課驗收
1. HTTP 請求和回應是什麼？
2. TLS 有什麼作用？
3. Wireshark 用來做什麼？
4. 為什麼 HTTPS 封包通常看不到明文網頁內容？
5. 為什麼不能擷取他人的私人通訊？
