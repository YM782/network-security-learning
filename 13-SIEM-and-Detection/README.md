# SIEM and Security Monitoring（安全資訊與事件管理）

SIEM（Security Information and Event Management，安全資訊與事件管理）是企業 SOC（Security Operations Center，資安營運中心）的重要技術平台。它將不同設備與服務的日誌集中收集、解析、正規化、關聯與分析，協助分析師從大量資料中找出值得調查的安全訊號。

本章以 Wazuh 作為 VMware 可練習的開源平台，並介紹 Microsoft Sentinel 與 KQL。目標是理解 SIEM 資料流程、建立偵測規則、調整誤報、調查事件與撰寫偵測報告。

## 1. What Is SIEM?（什麼是 SIEM？）

SIEM 不等於單純的日誌收集器，通常整合：
- Log Collection（日誌收集）。
- Parsing（資料解析）。
- Normalization（資料正規化）。
- Enrichment（資料豐富化）。
- Correlation（事件關聯）。
- Detection Engineering（偵測工程）。
- Alert Management（告警管理）。
- Investigation（調查）。
- Case Management（案件管理）。
- Reporting and Retention（報告與資料保留）。

### SIEM vs. SOAR vs. XDR

| Technology | 中文 | 主要用途 |
|---|---|---|
| SIEM | 安全資訊與事件管理 | 集中日誌、關聯事件、偵測與調查 |
| SOAR | 安全協作、自動化與應變 | 自動化告警、工單、通知與核准流程 |
| XDR | 延伸式偵測與回應 | 跨端點、身分、郵件、網路與雲端偵測 |
| EDR | 端點偵測與回應 | 監控端點程序、檔案、連線與行為 |

產品功能可能重疊。選型應考慮資料來源、偵測能力、調查流程、整合能力、成本與營運需求。

## 2. SIEM Architecture（SIEM 架構）

典型資料流程：
1. Data Sources：Windows、Linux、AD、Firewall、DNS、VPN、Proxy、EDR、Cloud Audit Log、資料庫與應用程式。
2. Collectors / Agents：Wazuh Agent、Windows Event Forwarding、Syslog、雲端 Connector 或 API。
3. Transport：透過驗證與加密通道傳送資料。
4. Parsing：拆解時間、帳號、主機、事件類型、來源 IP 等欄位。
5. Normalization：把不同產品欄位轉成可比較的共同模型。
6. Enrichment：加入資產重要度、使用者部門、地理資訊與威脅情報。
7. Storage and Indexing：支援查詢、保留、存取控制與資料生命週期。
8. Detection：使用規則、統計、關聯與行為分析尋找異常。
9. Alert and Case：產生告警、分派分析師、記錄調查與處理結果。
10. Response：依核准程序通知、隔離、撤銷權限或建立工單。

## 3. Log Sources（重要日誌來源）

| Source | Useful Records | Security Value |
|---|---|---|
| Windows Security | 登入、帳號、群組與權限事件 | 身分與權限異常 |
| Sysmon | 程序、網路、檔案與 DNS 活動 | 端點行為關聯 |
| PowerShell | Script Block、Module、執行活動 | 管理工具與腳本稽核 |
| Active Directory | 帳號、群組、GPO 與目錄變更 | 網域身分安全 |
| Linux auth / journal | SSH、sudo、服務與系統事件 | Linux 登入與系統異常 |
| Firewall | 允許、拒絕與連線流量 | 網路邊界活動 |
| DNS | 查詢名稱、回應與來源 | 可疑網域與通訊線索 |
| VPN | 使用者、來源 IP、工作階段 | 遠端存取異常 |
| Cloud Audit Logs | API 呼叫、角色與資源變更 | 雲端控制平面稽核 |
| Web / API Logs | HTTP 狀態、使用者、路徑與延遲 | 應用程式攻擊與濫用 |

### Windows Event IDs（Windows 常見事件）

| ID | 意義 | 調查重點 |
|---:|---|---|
| 4624 | 成功登入 | 帳號、來源、登入類型 |
| 4625 | 登入失敗 | 失敗原因、來源與頻率 |
| 4688 | 程序建立 | 程序、父程序與命令列（依稽核設定） |
| 4720 | 建立帳號 | 建立者與新帳號 |
| 4728 | 加入全域安全群組 | 目標群組與執行者 |
| 4732 | 加入網域本機安全群組 | 目標群組與執行者 |
| 4740 | 帳號鎖定 | 帳號與來源電腦 |
| 4768 | Kerberos TGT 要求 | 帳號與驗證結果 |
| 4769 | Kerberos Service Ticket 要求 | 服務與帳號 |
| 1102 | Security Log 被清除 | 執行者與事件時間 |

事件是否產生，取決於稽核政策、Windows 版本、權限與日誌保留設定。不能只憑單一 Event ID 判定攻擊成立。

## 4. Detection Engineering（偵測工程）

一條可用的偵測規則應包含：
- Detection Objective（偵測目標）。
- Data Source（資料來源）。
- Required Fields（必要欄位）。
- Logic（判斷邏輯）。
- Threshold（門檻）。
- Time Window（時間視窗）。
- Suppression / Deduplication（抑制與去重）。
- Severity（嚴重程度）。
- False Positive Handling（誤報處理）。
- Investigation Steps（調查步驟）。
- Response Guidance（回應建議）。
- Owner and Review Date（負責人與檢查日期）。

### Example A — Repeated Failed Logons（短時間大量登入失敗）
同一帳號在 10 分鐘內出現多次失敗登入；同時分析來源 IP、裝置與登入類型；排除已知測試與合法管理活動；若同時出現成功登入或權限變更，提升調查優先順序。

### Example B — Privileged Group Change（高權限群組變更）
監控 Domain Admins、Enterprise Admins 等敏感群組成員變更；關聯操作者、目標帳號、來源主機與時間；核准變更應有維護視窗與變更單參照。

### Example C — Security Log Cleared（日誌遭清除）
監控 Event ID 1102，關聯執行者、主機、前後事件與是否有已核准維護活動。

## 5. Detection Quality（偵測品質）

| Metric | 中文 | 說明 |
|---|---|---|
| MTTD | 平均偵測時間 | 從事件開始到被發現 |
| MTTA | 平均確認時間 | 從告警到分析師確認 |
| MTTR | 平均回應／復原時間 | 依組織定義，從確認到控制或恢復 |
| False Positive Rate | 誤報率 | 被判定為非威脅的告警比例 |
| Coverage | 覆蓋率 | 重要資產與技術是否有資料及偵測 |
| Data Latency | 資料延遲 | 事件發生到 SIEM 可查詢 |
| Log Source Health | 日誌來源健康度 | 是否有中斷、解析失敗或時間偏差 |

沒有日誌不代表沒有攻擊；有大量日誌不代表偵測能力良好。規則必須持續測試、調整與版本管理。

## 6. Wazuh Architecture（Wazuh 架構）

Wazuh 是免費開源的安全平台，可用於端點監控、日誌分析、檔案完整性監控、安全組態評估與威脅偵測。

主要元件：
- Wazuh Agent：安裝在 Windows、Linux 等受監控端點。
- Wazuh Server：接收與分析 Agent 資料。
- Wazuh Indexer：儲存與索引資料。
- Wazuh Dashboard：查詢、視覺化與管理介面。

小型實驗可採 all-in-one 部署；正式或大型環境應評估分散式架構、高可用性、容量、網路分區與備份。

Wazuh 官方 Quickstart 對 1–25 個 Agent 的 all-in-one 小型部署，建議約 4 vCPU、8 GiB RAM、50 GB 儲存空間。實際需求會依事件量、索引保留與查詢頻率增加。

## 7. Microsoft Sentinel（Microsoft Sentinel）

Microsoft Sentinel 是雲端 SIEM／SOAR 平台，支援：
- Data Connectors（資料連接器）。
- Analytics Rules（分析規則）。
- Incidents（事件案件）。
- Hunting（威脅搜尋）。
- Workbooks（工作簿與儀表板）。
- Playbooks（使用 Logic Apps 自動化回應）。
- KQL（Kusto Query Language，Kusto 查詢語言）。

功能與介面可能依 Azure／Microsoft Defender 入口網站及服務更新而改變，操作前請以官方文件為準。

## 8. KQL Basics（KQL 基礎）

常用運算子：where（篩選）、project（選取欄位）、summarize（聚合）、count（計數）、bin（時間分桶）、sort by（排序）、extend（建立衍生欄位）、join（關聯資料表）。

### Query A — Failed Sign-ins（失敗登入）

假設工作區已收集 Microsoft Entra ID Sign-in Logs：

`kusto
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType != "0"
| summarize FailedAttempts = count() by UserPrincipalName, IPAddress
| sort by FailedAttempts desc
`

### Query B — Azure Activity Changes（Azure 資源變更）

假設 AzureActivity 表已接收資料：

`kusto
AzureActivity
| where TimeGenerated > ago(24h)
| where ActivityStatusValue =~ "Success"
| project TimeGenerated, Caller, OperationNameValue,
          ResourceGroup, ResourceId, CallerIpAddress
| sort by TimeGenerated desc
`

### Query C — Windows Failed Logons（Windows 失敗登入）

假設 SecurityEvent 資料表與 Windows Security Events 已連接：

`kusto
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID == 4625
| summarize FailedLogons = count() by Account, Computer, IpAddress
| sort by FailedLogons desc
`

若工作區使用不同資料表或欄位名稱，先查詢 Schema 再調整 KQL。空結果不一定代表沒有事件，也可能是 Connector、稽核政策或資料表設定問題。

## 9. Practical Wazuh Lab（Wazuh 實作）

完整步驟見 [02-Wazuh-Lab.md](02-Wazuh-Lab.md)。

架構：
- WAZUH01：Ubuntu Server，執行 Wazuh all-in-one。
- WIN-CLIENT01：Windows 測試端點，安裝 Wazuh Agent。
- DC01（選用）：隔離 AD 網域控制站。
- 全部 VM 使用 Host-only 網路。

任務：
1. 安裝 Wazuh 中央元件。
2. 註冊 Windows Agent。
3. 收集 Windows Security Event Log。
4. 安裝 Sysmon 並收集 Sysmon Operational Log。
5. 查詢登入與程序事件。
6. 建立或調整測試偵測規則。
7. 調查告警並撰寫事件報告。
8. 檢查資料延遲、Agent 狀態與磁碟使用量。

## 10. Detection Report Template（偵測規則報告）

- Rule Name：
- Detection Objective：
- Data Sources：
- Required Fields：
- Query / Rule Logic：
- Time Window：
- Threshold：
- Severity：
- Expected True Positive：
- Known False Positives：
- Investigation Steps：
- Response Guidance：
- Test Result：
- Rule Owner：
- Last Review：

## 11. Knowledge Check（測驗）

1. SIEM 與單純日誌伺服器有何不同？
2. Parsing 與 Normalization 有何不同？
3. 為什麼日誌需要時間同步？
4. 4624、4625、4688、1102 分別代表什麼？
5. 偵測規則應包含哪些資訊？
6. False Positive 如何影響 SOC？
7. Wazuh 的四個核心元件是什麼？
8. Sentinel 中 KQL 有什麼用途？
9. 為什麼資料收集成功不等於偵測完整？
10. 如何衡量 SIEM 的實際成效？

## Sources（資料來源）

- Wazuh Documentation: https://documentation.wazuh.com/current/
- Wazuh Quickstart: https://documentation.wazuh.com/current/quickstart.html
- Wazuh Windows Agent: https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html
- Microsoft Sentinel Overview: https://learn.microsoft.com/en-us/azure/sentinel/overview
- Microsoft Sentinel Solutions: https://learn.microsoft.com/en-us/azure/sentinel/solution-setup-essentials
- Microsoft Learn — Kusto Query Language: https://learn.microsoft.com/en-us/kusto/query/
- NIST SP 800-92: https://csrc.nist.gov/pubs/sp/800/92/final

本章以監控、偵測、調查與授權應變為目的。自動封鎖、停用帳號或隔離設備等動作，應先設計核准、例外與回復機制。
