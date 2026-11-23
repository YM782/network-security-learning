# 04-01｜網路設備與資料傳輸：電腦如何連接 Internet？

> 難度：初級｜建議時間：90 分鐘｜先修：完成電腦基礎和 Kali 基礎
>
> 本課從家用網路開始，理解網卡、交換器、路由器、無線基地台和數據機的分工，再建立 OSI 七層模型的基本概念。你不需要一次背熟所有協定，但要能說明資料大致如何從電腦送到網站。

## 一、本課學習成果

完成後，你應該能夠：
1. 分辨 LAN、WAN 和 Internet。
2. 說明網卡、交換器、路由器、無線基地台和數據機的用途。
3. 畫出簡單的家用網路拓撲。
4. 理解 MAC 位址和 IP 位址的用途不同。
5. 說明 OSI 七層模型的基本分工。
6. 在 Windows 查看自己的網路介面資訊。

## 二、網路是什麼？

Network（網路）是讓兩台或多台設備交換資料的連線系統。設備可以透過網路線、Wi-Fi、光纖或其他通訊媒介連接。

常見網路範圍：
- LAN（Local Area Network，區域網路）：例如家裡、學校或辦公室的區域網路。
- WAN（Wide Area Network，廣域網路）：連接較遠地點的網路。
- Internet（網際網路）：由全球大量互相連接的網路組成。

Wi-Fi 是無線區域網路技術，不等於 Internet。即使連上 Wi-Fi，也可能因路由器、電信業者或 DNS 問題而無法上網。

## 三、五種常見網路設備

### 1. Network Adapter（網路介面卡／網卡）

網卡讓電腦接入有線或無線網路。常見類型：
- Ethernet Adapter：乙太網路介面。
- Wi-Fi Adapter：無線網路介面。
- Virtual Network Adapter：虛擬網路介面，例如 VMware 提供給 Kali 的虛擬網卡。

網卡有自己的 MAC 位址，用於區域網路中的資料鏈結層通訊。

### 2. Switch（交換器）

交換器通常用於連接同一個區域網路中的多台設備。它會學習 MAC 位址與交換器埠之間的對應關係，再根據目的 MAC 位址轉送乙太網路訊框。

簡單理解：交換器主要處理同一個區域網路內設備之間的資料轉送。

### 3. Router（路由器）

路由器負責在不同 IP 網路之間轉送封包。它會參考路由表，決定資料下一步應該送往哪個網路或下一跳。

家用路由器通常還整合 NAT、防火牆、DHCP 和 Wi-Fi 等功能，但這些功能在概念上是不同的。

### 4. Access Point（無線基地台）

無線基地台提供 Wi-Fi 無線接入，讓手機、平板和筆記型電腦加入區域網路。

有些家用路由器內建無線基地台，因此使用者會把整台設備統稱為「Wi-Fi 路由器」。

### 5. Modem / ONU（數據機／光網路終端）

Modem 或 ONU/ONT 依接入技術不同，負責把家中設備連接到電信業者的接入網路。光纖環境常見 ONU/ONT，纜線或 DSL 環境則可能使用不同類型的數據機。

實際家用設備可能把路由器、交換器、Wi-Fi 和光纖終端整合在一起。

## 四、家用網路拓撲圖

簡化示意：

    筆記型電腦 ── Wi-Fi ──┐
                           │
    手機 ─────── Wi-Fi ────┤
                           ▼
                    家用路由器
                           │
                    光纖終端設備
                           │
                      電信業者網路
                           │
                        Internet
                           │
                       網站伺服器

如果使用 VMware NAT，Kali 會先連接 VMware 的虛擬 NAT 網路，再透過 Windows 的實體網路連線存取外部網路。這是虛擬網路，不要和家中實體路由器混為一談。

## 五、MAC 位址和 IP 位址有什麼不同？

### 1. MAC Address（媒體存取控制位址）

MAC 位址通常用於資料鏈結層，識別同一鏈路中的網路介面。常見表示形式是六組十六進位數字，例如：

    00:1A:2B:3C:4D:5E

這只是格式示例，不是任何特定設備的真實位址。

### 2. IP Address（網際網路協定位址）

IP 位址用於網路層，讓路由器能依照目的網路轉送封包。IPv4 例子：

    192.168.1.20

這也是示例，不代表你的電腦一定使用這個位址。

### 3. 核心差別

- MAC：主要用於本地鏈路中的訊框傳送。
- IP：用於跨網路尋址和路由。
- 路由器轉送 IP 封包時，下一段鏈路上的資料鏈結層封裝會依鏈路重新建立。

因此，MAC 和 IP 不是同一種位址，也不能互相替代。

## 六、OSI 七層模型：先建立概念

OSI（Open Systems Interconnection，開放式系統互連）模型把網路通訊分成七層，方便學習和排查。

| 層數 | 英文 | 中文 | 常見例子 |
|---:|---|---|---|
| 7 | Application | 應用層 | HTTP、DNS、SMTP |
| 6 | Presentation | 表示層 | 資料格式、編碼、加密概念 |
| 5 | Session | 會議層 | 會話建立與管理概念 |
| 4 | Transport | 傳輸層 | TCP、UDP |
| 3 | Network | 網路層 | IP、路由 |
| 2 | Data Link | 資料鏈結層 | Ethernet、MAC、交換 |
| 1 | Physical | 實體層 | 電纜、光纖、無線電訊號 |

現代 TCP/IP 網路常以四層或五層模型描述。OSI 是學習和分析工具，不代表每個真實協定都能完美對應單獨一層。

## 七、資料傳輸時會發生什麼？

當你在 Chrome 開啟網站時，簡化流程如下：

1. 應用程式產生 HTTP 或 HTTPS 請求。
2. 傳輸層使用 TCP 或 UDP 等協定承載資料。
3. 網路層使用 IP 位址決定來源和目的地。
4. 資料鏈結層在本地鏈路上封裝訊框。
5. 實體層透過電纜、光纖或無線訊號傳送位元。
6. 路由器依 IP 路由資訊把封包轉往下一個網路。
7. 目的端逐層處理資料，交給對應的應用程式。

這個過程稱為封裝（Encapsulation）。每一層會加入自己需要的控制資訊。

## 八、Windows 實驗：查看自己的網路介面

打開 PowerShell 或命令提示字元，輸入：

    ipconfig

它會顯示網路介面的基本 IP 設定。

再輸入：

    ipconfig /all

可以查看更完整的資訊，包括：
- 介面描述。
- Physical Address（MAC 位址）。
- IPv4 Address。
- Subnet Mask。
- Default Gateway。
- DNS Servers。
- DHCP 是否啟用。

不要把截圖公開分享，因為可能包含裝置名稱、MAC 位址、內部 IP 和其他網路資訊。

## 九、Kali 實驗：查看虛擬網卡

在 Kali 終端機輸入：

    ip link
    ip addr

ip link 查看網路介面及鏈路狀態；ip addr 查看介面位址。

如果看到 ens33、eth0 或其他名稱，這是介面名稱，不代表每台電腦都一樣。

## 十、常見故障排查

**問題 1：連上 Wi-Fi，但不能上網。**

Wi-Fi 只代表無線連線建立，不保證路由器能連 Internet。接著檢查 IP、預設閘道器、DNS 和電信業者連線。

**問題 2：Windows 有網路，Kali 沒有網路。**

Windows 的實體網卡正常，不代表 VMware 虛擬網卡一定正常。檢查 VMware Network Adapter、Kali 的 ip addr 和 ip route。

**問題 3：不知道哪個網卡正在使用。**

在 Windows 執行 ipconfig /all，查看有 IP、閘道器和 DNS 的介面；在 Kali 使用 ip addr 和 ip route 對照。

## 十一、本課測驗

1. Wi-Fi 和 Internet 有什麼不同？
2. 交換器和路由器的主要差別是什麼？
3. MAC 位址和 IP 位址各自主要用在哪裡？
4. OSI 第 2、3、4、7 層分別處理什麼？
5. 為什麼路由器需要路由表？
6. 為什麼 Windows 有網路不代表 Kali 一定有網路？
7. 請畫出你家中設備連接 Internet 的簡化拓撲。

## 十二、參考資料

- [Cisco：Networking Basics](https://www.netacad.com/courses/networking-basics)
- [Cisco：Introduction to Networks](https://www.netacad.com/)
- [Wireshark 官方文件](https://www.wireshark.org/docs/)
- [IETF RFC Editor](https://www.rfc-editor.org/)

**完成標準：** 能畫出家用網路拓撲，分辨網卡、交換器、路由器和基地台，並能說明 OSI 模型的基本分工。