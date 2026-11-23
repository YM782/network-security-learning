# 04 — 電腦網路基礎：從家用網路到封包分析

> 本模組把網路概念按實際通訊順序排列。先認識設備，再學 IP 和路由，接著理解 DNS、DHCP、TCP/UDP，最後用瀏覽器和 Wireshark 觀察封包。

## 課程順序

1. [01 網路設備與資料傳輸：電腦如何連接 Internet？](01-Network-Devices.md)
2. [02 IPv4、子網路遮罩、CIDR 和預設閘道器](02-IP-Address-and-Gateway.md)
3. [03 DNS、DHCP 和路由：電腦如何找到網路上的服務？](03-DNS-DHCP-Routing.md)
4. [04 TCP、UDP、連接埠和三次握手](04-TCP-UDP-Ports.md)
5. [05 HTTP、HTTPS、TLS 和 Wireshark 封包分析入門](05-HTTP-and-Wireshark.md)

## 本模組完成標準

- [ ] 能畫出家用網路拓撲，說明網卡、交換器、路由器和無線基地台。
- [ ] 能解釋 MAC 位址和 IP 位址的差別。
- [ ] 能讀懂 IPv4、私有 IP、/24 子網路和預設閘道器。
- [ ] 能說明 DHCP 的 DORA 流程。
- [ ] 能解釋 DNS 查詢和基本路由決策。
- [ ] 能說明 TCP 三次握手、UDP 和常見連接埠。
- [ ] 能讀懂基本 HTTP Request 和 Response。
- [ ] 能說明 HTTPS/TLS 的保護作用和限制。
- [ ] 能用 Chrome Network 面板觀察請求。
- [ ] 能用 Wireshark 找到自己產生的 DNS 和 TCP 封包。

## 練習安全規則

只觀察自己的電腦和自己產生的流量。不要擷取公共 Wi-Fi 中其他人的私人通訊；不要把包含 Cookie、Authorization、Token 或內部網路資訊的封包檔公開上傳。Nmap 和其他掃描工具的授權範圍會在後續安全工具模組另外學習。

## 參考資料

- [Cisco Networking Academy](https://www.netacad.com/)
- [IETF RFC Editor](https://www.rfc-editor.org/)
- [Wireshark 官方文件](https://www.wireshark.org/docs/)
- [Microsoft Learn：Windows 網路命令](https://learn.microsoft.com/windows-server/administration/windows-commands/)
