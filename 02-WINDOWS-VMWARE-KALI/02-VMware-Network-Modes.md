# 02-02｜VMware 網路模式：NAT、Host-only、Bridged 詳細解析

> 難度：初級至初中級｜建議時間：90 分鐘｜先修：02-01
>
> 本課非常重要。虛擬機網路模式會影響 Kali 能不能上網、能不能與 Windows 通訊，以及隔離靶場是否會接觸到家庭或公司網路。

## 一、本課學習成果

完成後，你應該能夠：

1. 說明 NAT、Host-only 和 Bridged 的差異。
2. 知道 Kali 的 IP 位址由哪個網路環境提供。
3. 在 VMware 查看目前網路模式。
4. 使用 ip addr、ip route 和 Windows ipconfig 查看網路資訊。
5. 為一般 Kali 學習和故意有漏洞的靶場選擇較合適的網路模式。

## 二、先認識虛擬網路

VMware 可以建立虛擬交換器和虛擬網路介面。Kali 的虛擬網卡連接到某個虛擬網路後，才能與其他虛擬設備、宿主機或外部網路通訊。

先記住三個角色：

- Guest：Kali 虛擬機。
- Host：安裝 VMware 的 Windows。
- 外部 LAN：家用路由器或公司網路等實體區域網路。

不同模式的差別，在於 Guest 如何連接這些網路。

## 三、NAT 模式

NAT 是 Network Address Translation（網路位址轉換）。

一般桌面 VMware NAT 模式下，Kali 連接到 VMware 建立的私有虛擬網路，再透過宿主機的網路連線存取外部網路。

簡化流程：

    Kali Guest
        │
    VMware 虛擬 NAT 網路
        │
    Windows Host 的網路連線
        │
    家用路由器／外部網路
        │
    Internet

### NAT 的優點

- Kali 通常可以透過 Windows 的連線存取外部網路。
- Guest 通常不會像 Bridged 一樣直接成為家用 LAN 中的一台獨立設備。
- 適合一般 Kali 更新、下載文件和初級學習。

### NAT 的限制

- NAT 不等於完整安全隔離。
- Guest 仍可能主動連接外部網路。
- 宿主機和 Guest 的可達性取決於虛擬網路和防火牆設定。
- 連接埠轉發可能讓外部設備連到 Guest 服務。

### 什麼時候使用？

一般 Kali 系統更新和需要上網的日常學習，通常可以先考慮 NAT。

## 四、Host-only 模式

Host-only（僅主機）通常建立一個宿主機與虛擬機之間的私有網路。

簡化流程：

    Windows Host
        │
    VMware Host-only 虛擬交換器
        │
    Kali Guest

在典型設定下，Host-only 網路不會自動把 Kali 接到外部 Internet，但實際隔離程度仍取決於 Windows 路由、共享網路、虛擬網路設定和其他軟體。

### Host-only 的用途

- 在 Windows 和 Kali 之間做本機通訊實驗。
- 建立不需要直接連外的本機測試環境。
- 把靶場限制在受控的虛擬網路內。

### 注意

Host-only 不代表「絕對不可能連外」。如果 Windows 啟用了網路共享、路由或其他轉發設定，仍需另外檢查。

## 五、Bridged 模式

Bridged（橋接）模式會讓 Guest 透過實體網卡連接到目前的實體區域網路。

簡化流程：

    Kali Guest ─┐
                ├─ 家用路由器／公司交換器 ─ Internet
    Windows ────┘

在這種模式下，Kali 可能從家用路由器取得另一個 LAN IP，看起來像是家中另一台獨立設備。

### Bridged 的用途

- 某些需要讓 Guest 直接參與實體 LAN 的測試。
- 在有明確授權和隔離控制的環境中模擬獨立設備。

### 風險

- Kali 可能與家庭其他設備位於同一區域網路。
- 如果 Kali 運行有漏洞的服務，其他 LAN 設備可能接觸到它。
- 公共 Wi-Fi 或公司網路可能有使用政策和安全限制。

**初學者不要把故意有漏洞的靶場直接橋接到家庭、公司或公共網路。**

## 六、三種模式比較

| 比較項目 | NAT | Host-only | Bridged |
|---|---|---|---|
| Kali 通常可否連 Internet | 可以 | 通常不直接可以 | 通常可以 |
| Kali 能否與 Windows 通訊 | 視設定，通常可以 | 通常可以 | 視 LAN 和防火牆設定 |
| Kali 是否直接加入實體 LAN | 通常不是 | 否 | 是 |
| 常見用途 | 更新、一般學習 | 本機隔離實驗 | 受控 LAN 測試 |
| 初學者注意事項 | 不是完整隔離 | 檢查 Windows 共享和路由 | 不要把脆弱靶場暴露到實體 LAN |

## 七、實驗 A：只查看 VMware 目前模式

1. 正常關閉 Kali。
2. 在 VMware 選取 Kali 虛擬機。
3. 打開虛擬機設定。
4. 點選 Network Adapter。
5. 查看目前選取的是 NAT、Host-only 還是 Bridged。
6. 查看是否勾選 Connected 和 Connect at power on。
7. 記錄設定，不要急著切換。

不同 VMware 版本的選單位置可能略有不同。

## 八、實驗 B：在 Kali 查看 IP 和路由

啟動 Kali，打開 Terminal。

### 1. 查看網路介面

    ip addr

觀察：
- 介面名稱，例如 ens33、eth0 或其他名稱。
- 狀態是否為 UP。
- 是否取得 IPv4 位址。
- 是否有 loopback 介面 lo。

不要假設每台 Kali 的網卡名稱都一樣。

### 2. 查看路由表

    ip route

常見輸出可能包含 default via 某個 IP dev 某個介面。這表示系統有一條預設路由，將非本地網段的流量交給下一跳。

### 3. 查看 DNS 設定

    cat /etc/resolv.conf

這個檔案可能由 NetworkManager 或其他系統服務管理。只查看，不要直接編輯它來解決所有 DNS 問題。

## 九、實驗 C：在 Windows 查看實體網路資訊

開啟 PowerShell 或命令提示字元：

    ipconfig /all

找到目前使用的 Wi-Fi 或 Ethernet 介面，記錄：
- IPv4 Address
- Subnet Mask
- Default Gateway
- DNS Servers

注意：Windows 實體網卡 IP 和 Kali 虛擬網卡 IP 在 NAT 模式下通常不會相同，這是正常現象。

## 十、如何判斷 Kali 是否能連外？

按順序檢查，不要一開始就修改 DNS 或重裝 VMware。

### 步驟 1：確認網卡存在

    ip addr

如果看不到預期的網卡，先檢查 VMware 的 Network Adapter 是否連接，以及是否在虛擬機設定中啟用。

### 步驟 2：確認是否取得 IP

在 ip addr 中查看網卡是否有 IPv4 位址。如果沒有，可能是 DHCP 或虛擬網路服務問題。

### 步驟 3：確認是否有預設路由

    ip route

如果沒有預設路由，Guest 可能無法把資料送往其他網路。

### 步驟 4：測試 DNS 解析

    getent hosts example.com

如果有回傳 IP，代表系統取得了名稱解析結果。若沒有回傳，可能是 DNS、網路或目標解析服務問題。

### 步驟 5：可選擇測試 IP 層連通性

    ping -c 3 1.1.1.1

如果沒有回應，不能直接判定 Internet 中斷，因為目標或中間網路可能封鎖 ICMP。要結合其他測試判斷。

## 十一、常見故障排查

**問題 1：NAT 模式下 Kali 沒有 IP。**

排查：
1. 關閉 Kali。
2. 查看 VMware Network Adapter 是否選擇 NAT。
3. 確認 Connected 和 Connect at power on 已啟用。
4. 啟動 Kali，再執行 ip addr。
5. 如果仍無 IP，檢查 VMware 虛擬網路服務和 DHCP 設定，不要先重裝 Kali。

**問題 2：Kali 有 IP，但不能解析網站名稱。**

可能是 DNS 設定或 DNS 連線問題。先查看 ip route、/etc/resolv.conf 和 getent hosts 的結果。不要隨意把網路文章中的 DNS 設定整段貼進系統。

**問題 3：Kali 可以上網，但 Windows 找不到 Kali。**

可能是 NAT 網路的隔離、Windows 防火牆、Guest 防火牆或服務監聽介面設定。先確認雙方 IP 和路由，再確認是否有必要讓兩者互通。

**問題 4：Bridged 模式沒有取得 IP。**

可能是實體網卡選擇錯誤、Wi-Fi 網路限制、DHCP 不允許新設備，或公共網路限制虛擬機。先恢復到 NAT 進行一般學習，不要反覆改動未知設定。

## 十二、本課安全選擇原則

- 一般 Kali 上網和更新：通常先用 NAT。
- Windows 與 Kali 本機互通練習：可考慮 Host-only，並檢查路由與共享設定。
- 故意有漏洞的靶場：使用專門隔離的虛擬網路，避免橋接到日常 LAN。
- 需要 Bridged 的專業實驗：先取得網路所有者授權，確認測試範圍和風險。

## 十三、本課測驗

1. NAT、Host-only 和 Bridged 的核心差別是什麼？
2. 為什麼 NAT 不等於完整安全隔離？
3. 為什麼 Bridged 可能讓 Kali 出現在家庭路由器的設備清單？
4. ip addr 和 ip route 分別查看什麼？
5. Kali 有 IP 但不能開網站，可能是哪個環節有問題？
6. 為什麼不能把故意有漏洞的靶場直接橋接到公司網路？

## 十四、參考資料

- [VMware 官方文件入口](https://docs.vmware.com/)
- [Kali Linux：Network Configuration](https://www.kali.org/docs/)
- [Red Hat：Networking Guide](https://docs.redhat.com/)

**完成標準：** 能說明三種模式，能在 VMware 找到目前設定，並能使用 Kali 的 ip addr、ip route 初步判斷網路狀態。