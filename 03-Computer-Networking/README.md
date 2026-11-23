# Computer Networking Fundamentals（計算機網路基礎）

網路安全人員必須理解資料如何從來源主機到達目的主機。本課程從分層模型、IP、DNS、TCP/UDP 與 HTTP 開始。

## 1. OSI and TCP/IP Models（OSI 與 TCP/IP 模型）

| OSI Layer（OSI 層） | English | Examples（例子） |
|---|---|---|
| 7 應用層 | Application | HTTP、DNS、SMTP |
| 6 表達層 | Presentation | 資料格式、加密表示 |
| 5 會議層 | Session | 工作階段管理概念 |
| 4 傳輸層 | Transport | TCP、UDP |
| 3 網路層 | Network | IPv4、IPv6、ICMP |
| 2 資料連結層 | Data Link | Ethernet、MAC、ARP（ARP 常被放在網路介面與網路層交界討論） |
| 1 實體層 | Physical | 電纜、無線電訊號、光纖 |

TCP/IP 常以四層模型說明：Application、Transport、Internet、Link。OSI 是教學與分析模型，實際協定不一定能一對一對應。

## 2. MAC and IP Addresses（MAC 與 IP 位址）

- MAC Address（媒體存取控制位址）：在區域網路鏈路中識別網路介面。
- IP Address（網際網路協定位址）：用於跨網路尋址與路由。
- Subnet Mask（子網路遮罩）：區分 IP 位址中的網路部分與主機部分。
- Default Gateway（預設閘道）：主機將非本地網段流量交給的下一跳路由器。

查看 Linux 網路資訊：

```bash
ip -br addr
ip route
ip neigh
```

Windows：

```powershell
ipconfig /all
route print
arp -a
```

## 3. ARP（位址解析協定）

在 IPv4 區域網路中，主機需要知道同一鏈路上下一跳的 MAC 位址時，可能使用 ARP 查詢。ARP 快取可用 `ip neigh` 或 `arp -a` 查看。

注意：ARP 本身不提供身分驗證，因此區域網路可能面臨 ARP Spoofing（ARP 欺騙）等風險。學習時先理解正常解析流程與防禦方法。

## 4. DNS（網域名稱系統）

DNS 將網域名稱解析成 IP 位址。常見記錄類型：
- A：IPv4 位址。
- AAAA：IPv6 位址。
- CNAME：別名。
- MX：郵件交換伺服器。
- TXT：文字資料，常用於網域驗證與郵件安全設定。
- NS：權威名稱伺服器。

查詢範例：

```bash
nslookup example.com
dig example.com A
dig example.com AAAA
```

## 5. TCP and UDP（TCP 與 UDP）

TCP（Transmission Control Protocol，傳輸控制協定）提供可靠、有序的位元組串流。UDP（User Datagram Protocol，使用者資料包協定）則提供較精簡的資料報傳輸，不保證交付與順序。

TCP 三向交握概念：
1. Client → Server：SYN
2. Server → Client：SYN-ACK
3. Client → Server：ACK

常見連接埠只是慣例，不代表服務一定使用該連接埠：
- 22：SSH
- 53：DNS
- 80：HTTP
- 443：HTTPS
- 3389：RDP

## 6. HTTP and HTTPS（HTTP 與 HTTPS）

HTTP Request（請求）通常包含方法、路徑、標頭與選用的本文；Response（回應）包含狀態碼、標頭與選用的本文。

常見方法：
- GET：取得資源。
- POST：提交資料或觸發處理。
- PUT：建立或完整替換資源（依 API 設計）。
- PATCH：部分更新資源。
- DELETE：刪除資源（依權限與 API 設計）。

常見狀態碼：
- 200 OK：成功。
- 301／302：重新導向。
- 400：請求格式或內容有問題。
- 401：尚未通過身分驗證。
- 403：伺服器拒絕授權存取。
- 404：找不到資源。
- 500：伺服器內部錯誤。

HTTPS 是 HTTP over TLS。TLS 用於提供傳輸加密、完整性保護與伺服器身分驗證（依憑證驗證流程）。

## 7. Practical Commands（實用命令）

```bash
# 顯示網路介面
ip -br addr

# 顯示路由表
ip route

# 檢查 DNS
dig example.com

# 顯示監聽中的 TCP/UDP Socket
ss -tulpn

# 查看 HTTP 回應標頭
curl -I https://example.com

# 追蹤網路路徑（不同系統可能使用 traceroute 或 tracepath）
tracepath example.com
```

## 8. Lab — Capture Your Own Traffic（實驗：擷取自己的網路流量）

**範圍：** 只分析自己的電腦與自己發起的測試連線。

1. 開啟 Wireshark，選擇正在使用的網路介面。
2. 開始擷取。
3. 在終端執行 `nslookup example.com`。
4. 停止擷取並使用顯示過濾器 `dns`。
5. 再執行 `curl -I https://example.com`。
6. 使用 `tcp` 或 `tls` 過濾器查看相關封包。
7. 找出 DNS 查詢、TCP 交握與 TLS 握手的封包。
8. 記錄封包時間、來源、目的地與協定。

注意：HTTPS 的應用層內容通常受到 TLS 保護；未解密時不應期待直接看到完整 HTTP 內容。

## 9. Troubleshooting（故障排查）

| 現象 | 檢查方向 |
|---|---|
| 有 IP 但無法開啟網域 | DNS 設定、解析結果與 DNS 連線 |
| 能連本地網段但不能連外 | Default Gateway、路由、NAT 與防火牆 |
| TCP 連線逾時 | 目標是否可達、路由、防火牆、服務狀態 |
| Connection refused | 目標主機可達，但該連接埠可能沒有服務監聽或主動拒絕 |
| TLS 憑證錯誤 | 系統時間、憑證鏈、網域名稱與信任根 |
| Wireshark 看不到預期封包 | 選錯介面、擷取過濾器、流量未經該介面 |

## Sources（資料來源）

- Wireshark Documentation: https://www.wireshark.org/docs/
- Cloudflare Learning Center: https://www.cloudflare.com/learning/
- RFC Editor: https://www.rfc-editor.org/
- Nmap Reference Guide: https://nmap.org/book/man.html

本課程中的網路測試僅限自有設備與明確授權的實驗網路。
