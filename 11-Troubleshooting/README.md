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

## 8. Active Directory Troubleshooting（Active Directory 故障排查）

若實驗網域出現登入、DNS、Kerberos 或 GPO 問題，請依序檢查用戶端 DNS、SRV 記錄、時間同步、DC 服務與事件日誌。不要一開始就重設電腦帳號或重建網域。

用戶端常用檢查：

```powershell
ipconfig /all
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.adlab.test
nltest /dsgetdc:adlab.test
w32tm /query /status
klist
gpresult /r
Test-ComputerSecureChannel -Verbose
```

網域控制站常用檢查：

```powershell
Get-Service DNS,NTDS,Netlogon,KDC
dcdiag /test:dns /v
repadmin /replsummary
```

單一 DC 實驗室沒有複寫夥伴；只有多 DC 環境才可完整判讀複寫狀態。詳細命令、帳號與權限稽核請參閱 [AD Defensive Audit and Troubleshooting](../09-Active-Directory/03-Defensive-Audit-and-Troubleshooting.md)。

## 9. Incident Response and SIEM Troubleshooting（事件應變與 SIEM 排錯）

| Symptom（現象） | First checks（優先檢查） |
|---|---|
| Windows 查不到 Security Event | 確認事件記錄是否存在、稽核政策是否啟用、權限是否足夠 |
| PowerShell Get-WinEvent 回傳空結果 | 檢查 LogName、Event ID、時間範圍與時區 |
| Wazuh Agent Disconnected | 檢查 Agent 服務、管理端位址、網路與 Agent Log |
| Wazuh 沒有收到 Sysmon | 檢查 Sysmon/Operational 是否有事件、ossec.conf Event Channel 設定與 Agent 服務 |
| SIEM 告警過多 | 檢查資料來源、規則門檻、去重、維護視窗與誤報 |
| KQL 查詢沒有結果 | 確認 Connector、資料表、欄位名稱、時間範圍與資料延遲 |
| 雲端登入日誌缺失 | 檢查租戶稽核設定、資料連接器、權限與保留期限 |

詳細實驗請參閱 [Windows Incident Response Lab](../12-Incident-Response/02-Windows-Incident-Response-Lab.md) 與 [Wazuh SIEM Lab](../13-SIEM-and-Detection/02-Wazuh-Lab.md)。

## 10. Cloud Security Troubleshooting（雲端安全排錯）

| Symptom（現象） | First checks（優先檢查） |
|---|---|
| 雲端使用者無法存取資源 | 身分驗證、MFA、RBAC／IAM Policy、條件式存取與拒絕政策 |
| VM 無法連線 | VPC／VNet、路由、Security Group／NSG、DNS 與服務監聽 |
| 儲存服務存取被拒 | 資源政策、角色、公開存取設定、金鑰與網路限制 |
| 雲端日誌沒有資料 | Audit Log 啟用狀態、Connector、權限、目的地與時間範圍 |
| 帳單持續增加 | VM、磁碟、快照、公網 IP、日誌攝取與資料保留費用 |

不要為了排錯而把資源改成公開或授予廣泛管理權限。每次調整都應記錄變更前後設定與回復方式。

## 11. Lab Report Template（故障紀錄範本）

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
