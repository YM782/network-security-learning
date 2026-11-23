
# 04-03｜DNS、DHCP 和路由

## 本課目標
理解設備如何自動取得網路設定、網域名稱如何轉換成 IP，以及資料如何選擇下一個傳送方向。

## 1. DHCP：自動分配網路設定
DHCP（Dynamic Host Configuration Protocol，動態主機設定協定）可以自動向設備提供 IP、子網路遮罩、預設閘道器和 DNS 等設定。

## 2. DNS：網域名稱解析
DNS（Domain Name System，網域名稱系統）把網域名稱解析成 IP 位址。輸入網站名稱後，設備需要取得可用 IP，才能進一步與伺服器通訊。

## 3. DNS 查詢實驗
Windows 或 Kali：
```text
nslookup example.com
```
觀察使用哪個 DNS Server、查詢的 Name 和回傳的 Address。不同地區、DNS 服務和時間可能得到不同結果。

## 4. 路由是什麼？
Routing（路由）是根據路由表決定資料應該往哪個網路方向傳送。路由表通常包含目的網段、下一跳和網路介面等資訊。

Kali：
```bash
ip route
```
Windows：
```powershell
route print
```

## 5. 初步連線排查
網頁打不開時，按順序檢查：
1. 網路介面是否啟用？
2. 有沒有取得 IP？
3. 有沒有預設閘道器？
4. DNS 能否解析網域？
5. 目標網站或服務本身是否可用？

## 6. 本課驗收
1. DHCP 主要負責什麼？
2. DNS 解決什麼問題？
3. 路由表有什麼用途？
4. DNS 解析失敗和沒有 IP 是同一種故障嗎？
5. 為什麼排查網路要一步一步檢查？
