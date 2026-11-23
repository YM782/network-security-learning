# Network Analysis（網路流量分析與資產識別）

本課程介紹如何在自己擁有或已獲明確授權的實驗網路中，觀察封包、辨識服務與記錄資產。分析結果必須結合環境與證據，不能只憑一個工具輸出就判定存在漏洞。

## 1. Wireshark（封包分析工具）

Wireshark 可擷取並分析網路封包。建議先從自己電腦發出的 DNS、TCP 與 HTTPS 流量開始。

### Common Display Filters（常用顯示過濾器）

| Filter（過濾器） | 用途 |
|---|---|
| `dns` | 顯示 DNS 封包 |
| `tcp` | 顯示 TCP 封包 |
| `udp` | 顯示 UDP 封包 |
| `http` | 顯示可辨識的 HTTP 封包 |
| `tls` | 顯示 TLS 封包 |
| `ip.addr == 192.0.2.10` | 顯示與指定示範 IP 有關的封包 |
| `tcp.port == 443` | 顯示 TCP 連接埠 443 的封包 |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | 顯示初始 SYN 封包 |

注意：Display Filter（顯示過濾器）只改變畫面顯示，不會改變已擷取的封包。Capture Filter（擷取過濾器）則在擷取階段限制封包種類，兩者語法不同。

### Lab 1 — DNS and TCP（實驗一：DNS 與 TCP）

1. 開啟 Wireshark，選擇自己的網路介面。
2. 開始擷取。
3. 在終端執行 `nslookup example.com`。
4. 使用 `dns` 顯示過濾器。
5. 執行 `curl -I https://example.com`。
6. 使用 `tcp` 與 `tls` 檢查連線。
7. 記錄來源、目的地、時間與協定。

HTTPS 的應用層內容通常受到 TLS 保護；沒有正確解密條件時，不應期待直接看到完整 HTTP 內容。

## 2. Nmap（網路探索與服務識別）

Nmap 用於網路探索與服務識別。只能對自己擁有或已取得明確書面授權的目標執行。

先檢查本機回環介面：

```bash
nmap 127.0.0.1
```

在自有隔離靶場中，可進一步檢查服務版本並保存結果：

```bash
nmap -sV -oN lab-scan.txt <LAB_TARGET_IP>
```

參數說明：
- `-sV`：嘗試辨識開放連接埠上的服務與版本。
- `-oN`：以一般文字格式儲存結果。
- `<LAB_TARGET_IP>`：替換為自己隔離靶場中的目標 IP，不可填入未授權的第三方目標。

常見輸出欄位：
- PORT：連接埠與傳輸協定。
- STATE：狀態，例如 open、closed、filtered。
- SERVICE：推測的服務名稱。
- VERSION：工具辨識到的版本資訊。

服務版本辨識可能不完整或不準確，必須再以服務設定、官方公告與人工驗證確認。

## 3. Asset Inventory（資產清單）

在授權範圍內，建立以下紀錄：

| Field（欄位） | 說明 |
|---|---|
| Asset ID（資產編號） | 自訂唯一識別碼 |
| Hostname（主機名稱） | 主機名稱 |
| IP Address（IP 位址） | 實驗或授權範圍內的 IP |
| Operating System（作業系統） | 已確認或推測的系統 |
| Open Ports（開放連接埠） | 觀察到的連接埠 |
| Service（服務） | 服務名稱與版本 |
| Evidence（證據） | 掃描輸出、封包或設定截圖 |
| Scope（測試範圍） | 授權文件所列範圍 |
| Date（日期） | 檢查日期 |

不要把內部資產清單、真實 IP、主機名稱或網路拓樸上傳到公開倉庫。

## 4. Troubleshooting（故障排查）

| 現象 | 常見原因 | 檢查方法 |
|---|---|---|
| Wireshark 沒有封包 | 選錯介面、介面閒置或權限不足 | 確認介面與擷取狀態 |
| DNS 過濾器沒有結果 | 擷取時沒有 DNS 流量 | 重新產生一次 DNS 查詢 |
| Nmap 顯示 filtered | 防火牆或封包過濾 | 檢查實驗防火牆與路由 |
| Nmap 顯示 closed | 主機可回應，但該連接埠沒有服務監聽 | 在目標主機檢查服務 |
| 服務版本不明 | 服務不回應探測或版本資訊被隱藏 | 使用服務端管理資訊核對 |
| 封包時間不正確 | 系統時間或時區設定不正確 | 檢查系統時間與時區 |

## 5. Lab Report（實驗紀錄）

每次分析請記錄：
- 授權範圍與目標。
- 擷取介面與時間。
- 使用的顯示過濾器。
- 觀察到的封包與協定。
- Nmap 命令與輸出檔案。
- 判斷依據與不確定性。
- 排錯過程。
- 修復或防禦建議。

## Sources（資料來源）

- Wireshark Documentation: https://www.wireshark.org/docs/
- Nmap Reference Guide: https://nmap.org/book/man.html
- RFC Editor: https://www.rfc-editor.org/
- NIST Cybersecurity Framework: https://www.nist.gov/cyberframework

所有網路探索與測試都必須遵守明確授權及實驗範圍。
