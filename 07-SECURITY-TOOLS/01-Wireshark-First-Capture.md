
# 07-01｜Wireshark 第一次封包觀察

## 本課目標
啟動 Wireshark、選擇自己的網路介面、擷取自己產生的 DNS 查詢，並找到對應封包。

## 1. Wireshark 是什麼？
Wireshark 是網路協定分析工具。它把擷取到的封包解析成較容易閱讀的欄位，協助理解 DNS、TCP、TLS 等通訊。

## 2. 開始前
只在自己的電腦操作；不要擷取他人的私人通訊；選擇目前使用的網路介面；不要把包含私人資訊的封包檔公開上傳。

## 3. 第一次擷取
1. 開啟 Wireshark。
2. 在介面清單找到正在使用的 Wi-Fi 或 Ethernet。
3. 雙擊介面開始擷取。
4. 在終端機執行：
```bash
nslookup example.com
```
5. 回到 Wireshark，停止擷取。
6. 在 Display Filter（顯示篩選器）輸入 dns。

## 4. 觀察封包欄位
選取 DNS 封包，觀察 Time（時間）、Source（來源）、Destination（目的地）、Protocol（協定）和 Info（摘要）。展開 Domain Name System，尋找查詢名稱和回應資料。

## 5. Capture Filter 和 Display Filter
Capture Filter 在擷取前決定哪些封包要收集；Display Filter 在擷取後決定畫面顯示哪些封包。初學者先使用 Display Filter，避免因擷取篩選器設定錯誤而漏掉資料。

## 6. 常見問題
- 看不到封包：可能選錯介面，或測試流量沒有在擷取期間產生。
- 沒有 DNS：可能使用快取、加密 DNS 或其他解析方式。
- 看不到 HTTPS 明文：TLS 會保護應用資料，這是正常現象。

## 7. 本課驗收
能找到自己產生的 DNS 查詢，指出來源、目的地、協定和查詢名稱，並解釋 Wireshark 不等於破解加密。
