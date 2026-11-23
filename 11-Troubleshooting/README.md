# Troubleshooting（故障排查手冊）

網路安全學習不只要會操作，也要能找出失敗原因。每次排錯都應記錄症狀、環境、錯誤訊息、檢查步驟、根本原因與修復方式。

## 1. General Troubleshooting Method（通用排錯流程）

1. Define the symptom（描述現象）：什麼不能用？何時開始？是否每次都發生？
2. Record the environment（記錄環境）：作業系統、版本、虛擬機、網卡模式、工具版本。
3. Reproduce safely（安全重現）：只在自己的測試環境重現。
4. Check one layer at a time（逐層檢查）：硬體／虛擬化 → 作業系統 → 網路 → 服務 → 應用程式。
5. Change one variable（一次只改一項設定）。
6. Verify the result（確認結果）：再次測試並檢查日誌。
7. Record the root cause（記錄根本原因）與預防方式。

## 2. System Checks（系統檢查）

Linux：

```bash
uname -a
uptime
free -h
df -h
lsblk
top
journalctl -b -p err
```

Windows PowerShell：

```powershell
Get-ComputerInfo
Get-Process
Get-Service
Get-Volume
Get-WinEvent -LogName System -MaxEvents 30
```

## 3. Network Checks（網路檢查）

Linux：

```bash
ip -br addr
ip route
ip neigh
ss -tulpn
getent hosts example.com
```

Windows：

```powershell
ipconfig /all
route print
arp -a
Get-NetTCPConnection
Resolve-DnsName example.com
```

建議依序確認：
- 網路介面是否啟用。
- 是否取得正確 IP 與子網路遮罩。
- 是否有預設閘道。
- DNS 是否能解析。
- 目標服務是否監聽。
- 防火牆是否阻擋連線。

## 4. Docker Checks（Docker 檢查）

```bash
docker version
docker info
docker ps -a
docker images
docker logs --tail 100 <CONTAINER_NAME>
```

常見原因：
- Docker daemon 沒有啟動。
- 映像檔下載失敗。
- 容器啟動後立即退出。
- 本機連接埠已被其他程式使用。
- 容器內服務尚未完成初始化。

## 5. Common Problems（常見問題）

| Symptom（現象） | Possible cause（可能原因） | First check（優先檢查） |
|---|---|---|
| Kali 無法上網 | 虛擬網卡、IP、路由或 DNS | `ip -br addr`、`ip route` |
| 找不到靶機 | 虛擬機不在同一實驗網段 | VMware 網卡模式與 IP |
| Connection refused | 服務未監聽或主動拒絕 | `ss -tulpn`、服務日誌 |
| Connection timed out | 路由、防火牆或目標無回應 | 路由、封包與防火牆 |
| Port already in use | 連接埠已被占用 | `ss -tulpn` 或 Windows 連線檢查 |
| Permission denied | 使用者權限或 ACL 不足 | `id`、`ls -l`、`icacls` |
| DNS resolution failed | DNS 設定或解析服務異常 | `dig`、`nslookup`、`Resolve-DnsName` |
| Docker container exits | 設定錯誤或應用程式啟動失敗 | `docker logs` |
| Burp 看不到請求 | 瀏覽器代理、憑證或代理監聽設定 | Proxy listener 與瀏覽器設定 |

## 6. VMware Network Troubleshooting（VMware 網路排錯）

1. 確認虛擬機網卡已勾選 Connected（已連線）。
2. 確認網卡模式是 NAT、Bridged 或 Host-only；不要混淆用途。
3. 執行 `ip -br addr` 確認是否取得 IP。
4. 執行 `ip route` 檢查路由。
5. 先測試本機，再測試實驗網段閘道，最後才測試授權的外部服務。
6. 若修改網卡後仍無法連線，記錄變更並逐項還原，不要同時重設所有網路服務。

## 7. Burp Suite Troubleshooting（Burp Suite 排錯）

- 確認 Proxy listener 正在執行。
- 確認瀏覽器代理指向正確的本機位址與連接埠。
- 若要檢查 HTTPS，需在自己的測試瀏覽器中正確安裝 Burp CA 憑證。
- 不要在日常主要瀏覽器中長期安裝測試代理憑證。
- 先使用本機示範網站確認請求是否出現在 Proxy HTTP history。

## 8. Lab Report Template（故障紀錄範本）

```text
Date / 日期：
System / 系統：
Tool and version / 工具與版本：
Symptom / 故障現象：
Expected behavior / 預期行為：
Actual behavior / 實際行為：
Error message / 錯誤訊息：
Checks performed / 已執行檢查：
Root cause / 根本原因：
Fix / 修復方式：
Verification / 驗證結果：
Prevention / 預防措施：
```

## Sources（資料來源）

- Linux man-pages: https://man7.org/linux/man-pages/
- Microsoft Learn: https://learn.microsoft.com/
- Docker Docs: https://docs.docker.com/
- Wireshark Documentation: https://www.wireshark.org/docs/
- VMware Documentation: https://docs.vmware.com/

不要在公開 Issues 或提交紀錄中貼出密碼、權杖、私人 IP 資產清單、VPN 訂閱連結或其他敏感資料。
