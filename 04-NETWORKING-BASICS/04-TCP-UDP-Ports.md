# 04-04｜TCP、UDP、連接埠和三次握手

> 難度：初級至初中級｜建議時間：90–120 分鐘｜先修：04-01 至 04-03
>
> IP 主要負責把封包送往目的網路；TCP 和 UDP 則提供不同的傳輸方式。連接埠讓同一台電腦上的不同服務能夠區分網路資料。本課會用日常例子和本機觀察理解這些概念。

## 一、本課學習成果

完成後，你應該能夠：
1. 說明 IP、傳輸層協定和連接埠之間的關係。
2. 分辨 TCP 和 UDP 的主要特性。
3. 解釋 TCP 三次握手的基本流程。
4. 理解來源連接埠和目的連接埠。
5. 認識常見服務連接埠。
6. 使用 Windows netstat 和 Linux ss 查看本機連線及監聽狀態。

## 二、為什麼有了 IP 還需要連接埠？

想像一台伺服器同時提供網站、遠端登入和 DNS 服務。IP 位址可以找到那台設備，但作業系統還需要知道資料應該交給哪個網路服務。

連接埠（Port）就是傳輸層用來區分應用程式通訊端點的數字。

TCP 和 UDP 的連接埠空間分別管理。相同數字可以同時出現在不同傳輸協定中，例如 TCP 53 和 UDP 53 並不是同一個連線。

## 三、TCP 是什麼？

TCP（Transmission Control Protocol，傳輸控制協定）是一種連線導向的傳輸層協定。

TCP 提供的主要能力：
- 連線建立和關閉。
- 序號與確認機制。
- 需要時重傳遺失資料。
- 接收端資料排序和重組。
- 流量控制。
- 壅塞控制。

這些機制讓應用程式可以使用可靠、有序的位元組串流服務。

TCP 不保證應用程式一定成功完成工作，也不代表網路永遠不會中斷。它只是提供傳輸層的可靠性機制。

## 四、TCP 三次握手 Three-way Handshake

TCP 建立連線時，常見的初始流程：

1. SYN：客戶端送出同步請求，提出初始序號。
2. SYN-ACK：伺服器確認客戶端的請求，並提出自己的初始序號。
3. ACK：客戶端確認伺服器的序號。

完成後，雙方可以開始傳送應用資料。

簡化示意：

    Client                  Server
       | ------ SYN ------> |
       | <--- SYN + ACK --- |
       | ------ ACK ------> |
       |                    |
       |    Data Transfer   |

三次握手是建立 TCP 連線的典型過程。重傳、逾時、同時開啟和其他情況會讓實際封包序列更複雜。

## 五、UDP 是什麼？

UDP（User Datagram Protocol，使用者資料包協定）提供較簡單的資料報傳送服務。

UDP 不像 TCP 一樣提供連線建立、交付確認和排序重傳等完整機制。應用程式若需要可靠性，可以自行設計確認和重試。

常見使用情境：
- DNS 查詢（常見情況）。
- 即時語音和影音。
- 線上遊戲。
- 某些 VPN 和隧道協定。
- QUIC 等現代傳輸技術的底層承載（QUIC 使用 UDP）。

UDP 不一定永遠比 TCP 快。實際速度和延遲取決於應用、網路環境和協定設計。

## 六、TCP 和 UDP 比較

| 比較 | TCP | UDP |
|---|---|---|
| 連線模型 | 連線導向 | 無連線導向 |
| 交付確認 | 有相關機制 | 協定本身不提供 |
| 資料排序 | 提供有序位元組串流 | 不保證順序 |
| 重傳 | 協定層支援 | 應用程式自行決定 |
| 資料單位 | 位元組串流 | 資料報 |
| 常見用途 | Web、SSH、檔案傳輸等 | DNS、即時影音、遊戲等 |

## 七、連接埠號碼範圍

TCP 和 UDP 使用 16 位元連接埠號碼，範圍為 0 到 65535。

IANA 常見分類：
- System Ports：0–1023。
- User Ports：1024–49151。
- Dynamic/Private Ports：49152–65535。

這是常見註冊分類。作業系統的臨時連接埠範圍可能依設定不同。

## 八、常見連接埠

| Port | 常見協定／服務 | 說明 |
|---:|---|---|
| 20/21 | FTP | 傳統檔案傳輸控制和資料通道 |
| 22 | SSH | 安全遠端登入 |
| 25 | SMTP | 郵件傳輸 |
| 53 | DNS | 網域名稱服務，常用 UDP，也可能用 TCP |
| 67/68 | DHCPv4 | 伺服器和客戶端 |
| 80 | HTTP | 未加密的 Web 通訊 |
| 110 | POP3 | 郵件接收 |
| 143 | IMAP | 郵件存取 |
| 443 | HTTPS | 透過 TLS 保護的 HTTP |
| 3389 | RDP | Windows 遠端桌面常見連接埠 |

連接埠號碼只是慣例。服務可以配置在其他連接埠；看到 443 也不代表服務一定安全。

## 九、Socket（通訊端）是什麼？

Socket 是應用程式使用的網路通訊端點抽象。對 TCP 連線而言，常見識別資訊包括：
- 協定。
- 本機 IP 和本機連接埠。
- 遠端 IP 和遠端連接埠。

一條 TCP 連線通常可由來源 IP、來源 Port、目的 IP、目的 Port 和傳輸協定等資訊區分。

## 十、Windows 實驗：netstat

在 PowerShell 或命令提示字元輸入：

    netstat -ano

常見參數：
- -a：顯示所有連線和監聽連接埠。
- -n：以數字顯示位址和連接埠，不嘗試解析名稱。
- -o：顯示擁有該連線的程序 PID。

觀察：
- Local Address：本機位址和連接埠。
- Foreign Address：遠端位址和連接埠。
- State：TCP 連線狀態。
- PID：相關程序識別碼。

如果看到 LISTENING，通常代表本機有服務在等待連線；不代表它已經遭到入侵。

## 十一、Kali 實驗：ss

在 Kali 終端機輸入：

    ss -tuln

參數：
- -t：TCP。
- -u：UDP。
- -l：只顯示監聽中的 Socket。
- -n：使用數字顯示，不解析服務名稱。

如果要查看更詳細的 TCP 連線狀態，可以使用：

    ss -tan

不要因為看到陌生連接埠就直接關閉服務。先查明服務用途、執行程序和系統角色。

## 十二、如何理解常見 TCP 狀態？

| 狀態 | 基本意思 |
|---|---|
| LISTEN | 服務正在等待連線 |
| ESTABLISHED | TCP 連線已建立 |
| TIME_WAIT | 主動關閉的一方等待一段時間，處理延遲封包等情況 |
| CLOSE_WAIT | 本機收到對方關閉通知，應用程式尚未完成關閉 |
| SYN_SENT | 已送出 SYN，等待對方回應 |
| SYN_RECEIVED | 收到 SYN 並回覆，等待最後確認 |

不要只憑某個狀態判斷故障。要結合持續時間、程序、服務和應用程式行為。

## 十三、常見故障排查

**問題 1：網站 IP 可以解析，但 TCP 連線失敗。**

可能是目標服務沒有監聽、網路路由錯誤、防火牆阻擋、代理設定或伺服器故障。DNS 成功不代表 TCP 一定能建立。

**問題 2：本機有很多 LISTENING。**

作業系統和應用程式可能需要多種本機服務。先確認程序 PID 和服務用途，不要一看到監聽就認為是漏洞。

**問題 3：UDP 沒有顯示 ESTABLISHED。**

UDP 是無連線協定，工具顯示方式和 TCP 不同。不能用 TCP 的連線狀態直接解讀 UDP。

## 十四、本課測驗

1. 為什麼有 IP 還需要 Port？
2. TCP 三次握手的 SYN、SYN-ACK、ACK 分別表示什麼？
3. TCP 和 UDP 在交付、排序和重傳方面有何不同？
4. DNS 為什麼不只使用 UDP？
5. 443 連接埠一定代表 HTTPS 嗎？
6. netstat -ano 中 -a、-n、-o 分別代表什麼？
7. ss -tuln 中 -t、-u、-l、-n 分別代表什麼？
8. 為什麼 LISTENING 不等於已遭入侵？

## 十五、參考資料

- [IETF RFC 9293：Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293)
- [IETF RFC 768：User Datagram Protocol](https://www.rfc-editor.org/rfc/rfc768)
- [IANA：Service Name and Transport Protocol Port Number Registry](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml)
- [Microsoft Learn：netstat](https://learn.microsoft.com/windows-server/administration/windows-commands/netstat)
- [Linux man-pages：ss](https://man7.org/linux/man-pages/man8/ss.8.html)

**完成標準：** 能說明 TCP 和 UDP 的差異，理解三次握手，並能在 Windows 和 Kali 查看本機連線與監聽連接埠。