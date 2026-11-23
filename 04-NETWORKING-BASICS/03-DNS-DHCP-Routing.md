# 04-03｜DNS、DHCP 和路由：電腦如何找到網路上的服務？

> 難度：初級至初中級｜建議時間：90–120 分鐘｜先修：04-01、04-02
>
> 你輸入網站名稱後，電腦如何知道要連接哪個 IP？新設備接上 Wi-Fi 後，為什麼通常不用手動設定 IP？資料離開家中網路後，又如何找到下一個方向？本課會回答這三個問題。

## 一、本課學習成果

完成後，你應該能夠：
1. 說明 DHCP 如何提供網路設定。
2. 說明 DNS 如何把網域名稱解析成 IP 位址。
3. 理解路由表和預設路由。
4. 使用 nslookup、getent、ip route 和 route print 做基本觀察。
5. 分辨 DNS 故障、DHCP 故障和路由故障。
6. 按照合理順序排查「網站打不開」的問題。

## 二、DHCP：自動取得網路設定

DHCP（Dynamic Host Configuration Protocol，動態主機設定協定）讓設備自動取得網路設定，避免使用者每次加入網路都手動填寫 IP。

DHCP 可能提供：
- IP 位址。
- 子網路遮罩或前綴長度。
- 預設閘道器。
- DNS 伺服器。
- 租約時間等資訊。

### DHCP 的基本流程：DORA

常見 DHCPv4 初始分配流程可以用 DORA 記憶：

1. Discover：客戶端尋找 DHCP 伺服器。
2. Offer：伺服器提供可用設定。
3. Request：客戶端請求使用其中一個配置。
4. Acknowledge：伺服器確認租約。

這是簡化流程，實際網路可能有中繼代理、既有租約續約或其他情況。

### DHCP 為什麼需要租約？

DHCP 分配的 IP 通常有租約期限。設備會在適當時間嘗試續約，伺服器也能回收長期未使用的位址。

## 三、DNS：網域名稱解析

DNS（Domain Name System，網域名稱系統）負責把網域名稱解析成 IP 位址，讓使用者不必記住每個網站的數字位址。

例如，你輸入：

    example.com

系統需要透過 DNS 查詢取得對應的記錄，再使用 IP 與目標服務通訊。

### 常見 DNS 記錄

| 記錄類型 | 用途 |
|---|---|
| A | 網域名稱對應 IPv4 位址 |
| AAAA | 網域名稱對應 IPv6 位址 |
| CNAME | 別名，指向另一個網域名稱 |
| MX | 郵件交換伺服器 |
| NS | 網域的權威名稱伺服器 |
| TXT | 文字資料，常用於驗證和郵件安全設定 |

DNS 不只是「網址轉 IP」，還能保存多種服務和網域資訊。

### DNS 查詢大致經過什麼？

簡化流程：
1. 應用程式要求解析網域名稱。
2. 系統可能先檢查本機快取或 hosts 設定。
3. 若需要，再向設定的遞迴解析器查詢。
4. 遞迴解析器可能向根、頂級網域和權威 DNS 伺服器取得答案。
5. 回傳結果並依 TTL 等規則快取。

不同作業系統、瀏覽器和網路環境可能有不同快取與加密 DNS 行為。

## 四、DNS 實驗：nslookup

在 Windows PowerShell 或 Kali 終端機執行：

    nslookup example.com

觀察：
- Server：目前查詢使用的 DNS 伺服器。
- Name：查詢的網域名稱。
- Address：回傳的 IP 位址。

也可以查詢指定記錄類型：

    nslookup -type=MX example.com

這會查詢 MX 郵件交換記錄。不同 nslookup 版本的選項細節可能略有差異。

在 Kali 還可以使用：

    getent hosts example.com

getent 會透過系統設定的名稱服務機制查詢主機資訊。它和直接向指定 DNS 伺服器發送查詢的工具不完全相同。

## 五、Routing：路由是什麼？

路由（Routing）是根據目的 IP 網路，決定封包下一步應該送往哪裡的過程。

路由器會使用路由表（Routing Table）做決策。路由表常包含：
- 目的網段。
- 下一跳（Next Hop）。
- 出口介面。
- 路由來源和優先順序等資訊。

### 預設路由 Default Route

如果路由表中沒有更具體的路由符合目的地，通常會使用預設路由。

在 IPv4 中，常見表示方式是：

    0.0.0.0/0

它代表所有 IPv4 目的地的預設匹配範圍。實際選路時會考慮最長前綴匹配等規則。

## 六、查看 Linux 路由表

    ip route

可能看到類似：

    default via 192.168.1.1 dev eth0
    192.168.1.0/24 dev eth0 proto kernel scope link

這只是示例，實際輸出會依你的網路而不同。

第一行可讀作：預設流量交給 192.168.1.1，從 eth0 介面送出。

查看某個目的 IP 會選擇哪條路由：

    ip route get 1.1.1.1

它可以顯示系統對該目的位址的路由選擇。

## 七、查看 Windows 路由表

在 PowerShell 或命令提示字元執行：

    route print

可查看 IPv4 和 IPv6 路由表，以及介面資訊。

Windows 也可以使用：

    Get-NetRoute

這是 PowerShell 網路命令，可列出路由資訊。輸出可能較多，先觀察 IPv4 的活動路由即可。

## 八、三者如何合作？

當你打開網站時，簡化流程如下：

1. DHCP 先提供電腦的 IP、遮罩、閘道器和 DNS 等設定。
2. 瀏覽器或作業系統需要解析網站網域名稱。
3. DNS 回傳可用的 IP 位址。
4. 作業系統查路由表，判斷目的地是否在本地網段。
5. 若不在本地網段，通常交給預設閘道器。
6. 路由器再依自己的路由表把封包往下一跳傳送。

DHCP、DNS 和路由是不同功能，但它們會一起影響你能否正常訪問網站。

## 九、完整排查：網站打不開

請依序檢查，不要一次改很多設定。

### 第一步：確認介面是否有 IP

Windows：

    ipconfig

Kali：

    ip addr

若沒有正常 IP，先檢查網路連線和 DHCP。

### 第二步：確認預設路由

Kali：

    ip route

Windows：

    route print

如果沒有預期的預設路由，檢查 DHCP 或靜態網路設定。

### 第三步：確認 DNS 解析

    nslookup example.com

若 IP 和路由正常，但 DNS 查詢失敗，可能是 DNS 伺服器、網路政策或解析服務問題。

### 第四步：檢查目標服務

DNS 成功不代表網站一定正常。目標伺服器可能離線、拒絕連線、服務異常或受到防火牆限制。

## 十、常見故障情境

**情境 1：有 IP，但沒有閘道器。**

可能是 DHCP 配置不完整或靜態設定錯誤。先查看目前使用的網路介面和 DHCP 狀態。

**情境 2：可以連 IP，但網域名稱解析失敗。**

可能是 DNS 設定或解析器故障。先用 nslookup 和 getent 比較結果，不要直接判定整個 Internet 中斷。

**情境 3：DNS 有結果，但網站仍打不開。**

可能是 TCP 連線、TLS、HTTP、代理伺服器或網站服務本身的問題。DNS 只負責名稱解析，不負責保證網站可用。

**情境 4：Windows 能上網，Kali 不能。**

兩者可能使用不同虛擬網路、DNS 或路由設定。回到 VMware NAT／Host-only 設定，再檢查 Kali 的 ip addr、ip route 和 DNS。

## 十一、本課測驗

1. DHCP 的 DORA 四個階段分別是什麼？
2. DHCP 通常提供哪些網路設定？
3. DNS 的 A 和 AAAA 記錄有什麼不同？
4. 遞迴解析器和權威 DNS 伺服器有什麼差別？
5. 0.0.0.0/0 代表什麼？
6. ip route 和 route print 分別用於什麼系統？
7. DNS 成功解析，是否代表網站一定能打開？為什麼？
8. 網站無法訪問時，為什麼要依序檢查 IP、路由、DNS 和服務？

## 十二、參考資料

- [IETF RFC 2131：DHCP](https://www.rfc-editor.org/rfc/rfc2131)
- [IETF RFC 1034：DNS Concepts and Facilities](https://www.rfc-editor.org/rfc/rfc1034)
- [IETF RFC 1035：DNS Implementation and Specification](https://www.rfc-editor.org/rfc/rfc1035)
- [IETF RFC 1812：IPv4 Router Requirements](https://www.rfc-editor.org/rfc/rfc1812)
- [Microsoft Learn：route](https://learn.microsoft.com/windows-server/administration/windows-commands/route_ws2008)

**完成標準：** 能使用 nslookup、getent、ip route 和 route print 查看 DNS 與路由資訊，並能區分 DHCP、DNS 和路由故障。