# Wazuh SIEM Lab（Wazuh 安全監控實驗室）

本實驗使用 VMware 建立小型隔離 SIEM 環境，學習部署 Wazuh、註冊 Windows Agent、收集 Windows Security 與 Sysmon 日誌，並進行基本事件調查。

**實驗拓樸：**
- WAZUH01：Ubuntu Server 22.04／24.04，Wazuh all-in-one。
- WIN-CLIENT01：Windows 10／11 測試 VM，安裝 Wazuh Agent。
- DC01（選用）：Windows Server AD DS 實驗網域控制站。
- 全部連接 VMware Host-only 網路。

不要將故意存在弱點的靶場設為 Bridged，也不要把 SIEM 管理介面暴露在公共網際網路。

## 1. Resource Planning（資源規劃）

Wazuh 官方 Quickstart 對 1–25 個 Agent 的 all-in-one 部署，建議約 4 vCPU、8 GiB RAM 與 50 GB 儲存空間。這是小型起步參考，不是所有環境的固定需求。

| VM | vCPU | RAM | Disk |
|---|---:|---:|---:|
| WAZUH01 | 4 | 8 GB | 50–80 GB |
| WIN-CLIENT01 | 2 | 4 GB | 50 GB |
| DC01（選用） | 2 | 4–6 GB | 60 GB |

實際儲存需求取決於日誌量、索引、副本與保留期限。不要把 50 GB 視為任何規模都足夠。

## 2. Network Plan（網路規劃）

以下為範例位址，先確認不與宿主機、VPN 或其他 VM 網段衝突：

| Host | Example IP |
|---|---|
| WAZUH01 | 192.168.77.40 |
| WIN-CLIENT01 | 192.168.77.20 |
| DC01 | 192.168.77.10 |

使用 Host-only 網路。實驗環境可不設定預設閘道；若需要下載更新，先規劃受控的暫時網路方式，下載完成後恢復隔離。不要把管理介面對外公開。

## 3. Prepare Ubuntu Server（準備 Ubuntu Server）

安裝受支援的 Ubuntu Server，設定主機名稱與靜態 IP。完成後檢查：

```bash
hostnamectl
ip address
ip route
resolvectl status
timedatectl
df -h
free -h
```

先建立 VMware Snapshot。確認磁碟空間與時間同步正常，再進行安裝。

## 4. Install Wazuh All-in-one（安裝 Wazuh）

**重要：** 版本與安裝命令可能更新。以下命令格式來自 Wazuh 官方 Quickstart；執行前先開啟官方文件，確認版本、支援作業系統與下載來源。不要從不明網站下載安裝腳本。

官方文件：https://documentation.wazuh.com/current/quickstart.html

官方 Quickstart 範例：

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

這會安裝 Wazuh Indexer、Wazuh Server 與 Wazuh Dashboard。安裝完成後，終端機會顯示 Dashboard 位址與登入資訊。

**認證安全：**
- 不要把管理員密碼寫入 GitHub。
- 不要將安裝產生的密碼檔上傳到公開倉庫。
- 將密碼保存在本機密碼管理器。
- 初次登入後檢查憑證與管理帳號。
- 只允許實驗網段存取 Dashboard。

## 5. Verify Wazuh Services（檢查服務）

在 WAZUH01 執行：

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
sudo systemctl --failed
df -h
free -h
```

服務名稱或管理方式可能依 Wazuh 版本與部署方式不同。若服務不存在，先檢查安裝摘要與該版本官方文件，不要直接重複執行安裝腳本。

## 6. Install Windows Agent（安裝 Windows Agent）

在 Wazuh Dashboard 中開啟 Agents Management → Deploy new agent，選擇 Windows，輸入 WAZUH01 的實驗網路 IP，依畫面提供的命令安裝。

官方 Windows Agent 文件：
https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html

常見服務檢查：

```powershell
Get-Service WazuhSvc
Start-Service WazuhSvc
Get-Service WazuhSvc
```

確認 Agent 已成功註冊、狀態顯示 Active，且 WAZUH01 與 WIN-CLIENT01 在隔離網路中可正常通訊。不要把註冊 Token、密碼或 Agent Key 提交到 GitHub。

## 7. Collect Windows Security Events（收集 Windows 安全事件）

Wazuh Agent 可收集 Windows Event Channel。若需要明確指定 Security 日誌，依目前版本的 Agent 設定文件確認 ossec.conf。

設定概念範例（放在 ossec_config 區塊內，先備份原檔）：

```xml
<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
</localfile>
```

完成後重新啟動 Agent：

```powershell
Restart-Service WazuhSvc
Get-Service WazuhSvc
```

不要覆蓋整份原始設定檔。只新增必要設定，並確認 XML 結構正確。

## 8. Add Sysmon Telemetry（加入 Sysmon 端點遙測）

Sysmon 是 Microsoft Sysinternals 工具，可記錄程序建立、網路連線、檔案建立、DNS 查詢等端點活動。事件量與欄位取決於 Sysmon 版本與設定。

官方文件：https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

在自己的 Windows 測試 VM 中，下載 Sysmon，閱讀授權與說明，使用可信任設定檔。安裝範例：

```powershell
.\Sysmon64.exe -accepteula -i .\sysmonconfig.xml
```

在 Wazuh Agent 的 ossec.conf 中，備份後於 ossec_config 區塊內加入：

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

重新啟動 Agent：

```powershell
Restart-Service WazuhSvc
```

注意：Sysmon Event ID 3（Network Connection）與 Event ID 7（Image Loaded）可能需要在設定檔中明確啟用，且會增加資料量。先從必要事件開始，再逐步擴充。

## 9. Understand Key Sysmon Events（理解常見 Sysmon 事件）

| Event ID | Meaning | Investigation Use |
|---:|---|---|
| 1 | Process Create | 程序、命令列、父程序 |
| 3 | Network Connect | 程序與網路連線關聯 |
| 7 | Image Loaded | DLL／映像載入分析，可能產生大量事件 |
| 11 | File Create | 檔案建立活動 |
| 12–14 | Registry Events | 登錄機碼建立、刪除與值修改 |
| 22 | DNS Query | 程序與 DNS 查詢關聯 |

實際記錄內容依 Sysmon 設定而異。事件 ID 不是惡意判定；應與使用者、主機、程序路徑、簽章、網路與時間線交叉比對。

## 10. Explore Wazuh Dashboard（探索 Dashboard）

依序檢查：
1. Agent Summary：Agent 是否在線。
2. Security Events：事件是否進入平台。
3. Integrity Monitoring：檔案完整性監控結果。
4. Security Configuration Assessment：安全組態評估。
5. Vulnerability Detection：漏洞資料與端點盤點。
6. MITRE ATT&CK：告警與技術分類。
7. Agent Logs：Agent 連線與設定問題。

先確認資料來源、時間與欄位正確，再建立偵測規則。

## 11. Troubleshooting（故障排查）

| Symptom | Likely Cause | First Check |
|---|---|---|
| Agent 顯示 Disconnected | 網路、服務或註冊問題 | Get-Service WazuhSvc、Agent Log |
| Dashboard 無法開啟 | Dashboard 服務或網路 | systemctl status wazuh-dashboard |
| 有 Agent 但沒有事件 | Log Collector 設定或稽核未啟用 | Agent 設定、Windows Event Log |
| Sysmon 沒有資料 | Sysmon 未安裝或 Event Channel 設定錯誤 | Event Viewer → Sysmon/Operational |
| 磁碟快速增加 | 事件量、索引或保留策略 | df -h、索引與資料保留設定 |
| 告警過多 | 規則門檻或事件來源過廣 | Rule ID、頻率與誤報分析 |

Windows Agent 常見日誌位置：
```text
C:\Program Files (x86)\ossec-agent\ossec.log
```

## 12. Lab Deliverables（實驗成果）

- WAZUH01 與 WIN-CLIENT01 網路拓樸。
- Wazuh 版本與安裝方式。
- Agent 註冊狀態截圖（遮蔽敏感資訊）。
- Windows Security Log 收集證據。
- Sysmon Operational Log 收集證據。
- 至少一個可解釋的安全事件調查。
- 一份偵測規則說明。
- 一份誤報分析與改善建議。
- 一份資源使用量與保留策略說明。

## Sources（資料來源）

- Wazuh Quickstart: https://documentation.wazuh.com/current/quickstart.html
- Wazuh Installation Guide: https://documentation.wazuh.com/current/installation-guide/
- Wazuh Windows Agent: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html
- Wazuh Agent Configuration: https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/localfile.html
- Microsoft Sysmon: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

本實驗只監控自己的測試端點。不要將 SIEM 管理介面、Agent 註冊資訊或內部日誌公開。
