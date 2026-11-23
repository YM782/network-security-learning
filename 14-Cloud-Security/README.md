# Cloud Security（雲端安全）

Cloud Security（雲端安全）涵蓋雲端身分、權限、網路、運算、儲存、資料保護、日誌、威脅偵測、應用程式與供應鏈安全。

本章以 AWS 與 Microsoft Azure 為主要案例，同時建立跨雲端通用安全思維。重點不是背誦服務名稱，而是理解雲端責任分工、身分控制平面、網路邊界、資料保護與可觀測性。

## 1. Cloud Service Models（雲端服務模型）

| Model | Full Name | 中文 | 客戶主要管理內容 |
|---|---|---|---|
| IaaS | Infrastructure as a Service | 基礎設施即服務 | VM、OS、應用程式、資料、網路規則與身分 |
| PaaS | Platform as a Service | 平台即服務 | 應用程式、資料、身分與平台設定 |
| SaaS | Software as a Service | 軟體即服務 | 使用者、資料、存取政策與租戶設定 |

不同雲端產品的責任邊界可能不同，應查看服務文件與合約。

## 2. Shared Responsibility Model（共同責任模型）

雲端供應商負責其管理範圍內的底層基礎設施；客戶仍需保護自己管理的身分、資料、組態與工作負載。

| Security Area | IaaS | PaaS | SaaS |
|---|---|---|---|
| Physical Data Center | Provider | Provider | Provider |
| Physical Host | Provider | Provider | Provider |
| Operating System | Customer | Usually Provider | Provider |
| Application Configuration | Customer | Customer / Shared | Customer / Shared |
| Customer Data | Customer | Customer | Customer |
| User Identity and Access | Customer | Customer | Customer |
| Endpoint Devices | Customer | Customer | Shared / Customer |

這是概念性摘要。實際責任需依服務、部署模式與合約確認。雲端不代表供應商會自動替客戶設定正確 IAM、資料權限或公開存取限制。

## 3. Cloud Security Architecture（雲端安全架構）

- Control Plane（控制平面）：建立、修改與刪除雲端資源的管理 API 與操作介面。
- Data Plane（資料平面）：工作負載實際讀取、寫入或處理資料的服務介面。
- Management / Identity Plane（管理／身分平面）：帳號、角色、權限、Token、MFA 與管理工作階段。

若攻擊者取得高權限身分，可能不需要直接攻破 VM，就能透過管理 API 修改資源、權限、網路或資料設定。因此，雲端安全必須優先保護身分與管理入口。

## 4. Identity and Access Management（身分與存取管理）

| Term | 中文 | 說明 |
|---|---|---|
| Identity | 身分 | 人員、服務或工作負載的識別 |
| Authentication | 身分驗證 | 確認使用者或服務是誰 |
| Authorization | 授權 | 決定身分可以執行哪些操作 |
| IAM Policy | IAM 政策 | 定義允許或拒絕的資源操作 |
| Role | 角色 | 可被特定身分或工作負載取得的權限集合 |
| MFA | 多因素驗證 | 使用多種因素驗證身分 |
| RBAC | 角色型存取控制 | 依角色授予權限 |
| ABAC | 屬性型存取控制 | 依使用者、資源或環境屬性授權 |
| Conditional Access | 條件式存取 | 依風險、裝置、位置與條件控制存取 |

安全最佳實務：
- 啟用管理員與高風險帳號的 MFA。
- 使用最小權限原則。
- 優先使用 Federation、SSO 與短期憑證。
- 避免長期存取金鑰與硬編碼密碼。
- 將人員帳號與工作負載身分分開。
- 定期檢查閒置帳號、金鑰、角色與信任關係。
- 對高風險管理操作啟用審核與告警。
- 將緊急存取帳號納入特殊管理、監控與演練。

### AWS IAM（AWS 身分管理）
- Root User：帳號根使用者，應受到最高程度保護，不用於日常工作。
- IAM User：代表人員或應用程式的身分；避免不必要的長期 Access Key。
- IAM Role：透過信任政策與權限政策授予短期存取。
- Policy：以 JSON 定義 Effect、Action、Resource 與 Condition。
- AWS Organizations：管理多個 AWS 帳號與組織層級政策。

### Azure Identity（Azure 身分管理）
- Microsoft Entra ID：雲端身分與存取管理服務。
- Tenant：身分租戶。
- Subscription：Azure 資源與計費管理邊界。
- Azure RBAC：控制對 Azure 資源的管理操作。
- Conditional Access：依使用者、裝置、位置與風險控制存取。
- Managed Identity：讓 Azure 工作負載取得受管理身分，減少自行保管秘密。

## 5. Network Security（雲端網路安全）

AWS 常見元件：
- VPC：虛擬私有雲。
- Subnet：子網路。
- Route Table：路由表。
- Security Group：資源層級的狀態式流量控制。
- Network ACL：子網路層級的無狀態流量控制。
- VPC Flow Logs：網路流量中繼資料紀錄。

Azure 常見元件：
- Virtual Network（VNet）：虛擬網路。
- Subnet：子網路。
- Network Security Group（NSG）：網路安全性群組。
- Azure Firewall：雲端網路防火牆。
- Private Endpoint：透過私人網路存取支援的 PaaS 服務。
- Network Watcher：網路診斷與監控工具。

安全原則：
- 依工作負載與信任邊界分段。
- 管理介面不要直接暴露給整個網際網路。
- 優先使用 VPN、Zero Trust 存取或受控 Bastion。
- 只開放必要來源、目的地、協定與連接埠。
- 對流量日誌設定合理保存與存取控制。
- 定期檢查公網 IP、路由與防火牆規則。

## 6. Compute Security（雲端運算資源安全）

VM 與容器安全：
- 使用受信任且持續更新的基礎映像。
- 停用不必要服務與管理介面。
- 透過集中化工具管理修補。
- 將管理權限與應用程式執行身分分離。
- 限制 VM 對 Metadata Service 的存取方式。
- 監控啟動、映像、擴充功能與遠端管理設定。
- 不在 VM Image、User Data 或 Container Image 中存放秘密。
- 使用最小權限的 Workload Identity。

Container / Kubernetes：
- 使用可信任映像來源。
- 掃描映像中的漏洞與秘密。
- 不以 root 執行不必要的容器程序。
- 限制 Kubernetes RBAC。
- 保護叢集 API Server。
- 使用 Network Policy 分隔工作負載。
- 保護 Secrets 與叢集管理憑證。

## 7. Data Security（雲端資料安全）

資料保護涵蓋整個生命週期：
1. Data Discovery：了解資料位置與擁有人。
2. Classification：公開、內部、機密、受監管資料分類。
3. Access Control：依身分與業務需求授權。
4. Encryption at Rest：保護儲存中的資料。
5. Encryption in Transit：使用 TLS 等安全傳輸方式。
6. Key Management：保護金鑰、輪替與存取紀錄。
7. Backup and Recovery：保留可驗證的復原副本。
8. Secure Deletion：依服務能力與合約處理資料生命週期。

常見服務：
- AWS KMS、Secrets Manager、S3 Block Public Access。
- Azure Key Vault、Managed HSM、Storage Encryption 與 Private Endpoint。

不要把 Access Key、Client Secret、Token、私鑰或資料庫密碼提交到 GitHub。使用 Secret Manager、Managed Identity 或短期憑證。

## 8. Cloud Logging and Detection（雲端日誌與偵測）

### AWS
- CloudTrail：記錄帳號活動與 API 操作。
- AWS Config：記錄資源組態與變更。
- GuardDuty：威脅偵測。
- Security Hub：集中安全發現與安全態勢管理。
- VPC Flow Logs：網路流量中繼資料。
- CloudWatch：監控、指標與日誌。

### Azure
- Azure Activity Log：訂閱層級管理活動。
- Microsoft Entra Sign-in Logs：登入活動。
- Microsoft Entra Audit Logs：目錄與身分變更。
- Microsoft Defender for Cloud：雲端安全態勢與威脅保護。
- Microsoft Sentinel：SIEM／SOAR。
- Azure Monitor：平台與工作負載監控。

### What to Monitor（監控項目）
- Root／Global Administrator／高權限角色登入。
- 新增或修改 IAM Policy、Role、Trust Policy。
- 建立、刪除或停用 API Key。
- 儲存桶或儲存帳戶變成公開。
- 防火牆規則開放到 0.0.0.0/0。
- 新增公網 IP、網路閘道或遠端管理入口。
- 關閉日誌、刪除稽核設定或改變保留政策。
- 大量資料下載、複製或異常存取。
- 未預期的區域、裝置或服務身分活動。

## 9. Infrastructure as Code and DevSecOps（基礎設施即程式碼）

常見工具：
- Terraform：跨雲端 IaC 工具。
- AWS CloudFormation：AWS 原生資源範本。
- Azure Bicep：Azure 宣告式資源部署語言。
- GitHub Actions：CI/CD 工作流程。

安全檢查：
- 將 IaC 放入版本控制與 Pull Request 審查。
- 部署前檢查公開儲存、寬鬆 IAM 與危險網路規則。
- 使用 Secret Scanning 與依賴漏洞掃描。
- 將秘密放在 Secret Manager，不寫入程式碼。
- 對生產環境使用審核、分離權限與部署保護。
- 記錄每次部署的操作者、版本與變更內容。

## 10. Cloud Incident Response（雲端事件應變）

雲端事件調查需要保留：
- 身分登入與 Token 活動。
- IAM Role／Policy 變更。
- API 呼叫與資源組態變更。
- VM、Container、Function 與應用程式日誌。
- 儲存存取與資料傳輸紀錄。
- 網路流量與 DNS 資料。
- 雲端服務商提供的安全告警。

處理順序：
1. 確認受影響 Tenant、Account、Subscription 與 Resource。
2. 保存 Cloud Audit Logs、登入紀錄與告警。
3. 判斷受影響身分、憑證與工作負載。
4. 依核准程序撤銷工作階段、輪替秘密或限制角色。
5. 隔離受影響資源，避免擴大影響。
6. 檢查其他資源是否使用相同身分或金鑰。
7. 從可信任組態與備份復原。
8. 建立後續監控與改善項目。

## 11. Cost and Security Controls（成本與安全控制）

雲端實驗可能產生費用，尤其是 VM、磁碟、日誌攝取、資料傳輸、威脅偵測與儲存服務。

開始前：
- 設定預算與費用告警。
- 了解免費方案與服務計價條件。
- 不要假設所有安全服務都有免費額度。
- 完成實驗後停止或刪除不再使用的資源。
- 檢查快照、磁碟、公網 IP、Log Analytics Workspace 與資料保留費用。
- 不要為了練習而建立公開可存取的資料庫或儲存桶。

## 12. Practical Cloud Security Lab（雲端安全實驗）

完整步驟見 [02-Cloud-Security-Lab.md](02-Cloud-Security-Lab.md)。

任務：
1. 建立獨立雲端測試帳號或訂閱。
2. 啟用 MFA 與預算告警。
3. 建立最低權限測試角色。
4. 建立私人網路與受控工作負載。
5. 確認儲存服務沒有不必要的公開存取。
6. 啟用管理活動日誌。
7. 查詢登入、資源與權限變更。
8. 撰寫雲端安全稽核報告。
9. 清理資源並確認不再持續計費。

## 13. Cloud Security Checklist（雲端安全檢查清單）

- [ ] 管理員帳號已啟用 MFA。
- [ ] Root／緊急管理帳號受到嚴格保護。
- [ ] 使用最小權限與短期憑證。
- [ ] 不在程式碼、映像或公開倉庫存放秘密。
- [ ] VPC／VNet 與子網路分段清楚。
- [ ] 遠端管理入口受到限制。
- [ ] 儲存服務沒有不必要的公開存取。
- [ ] 靜態與傳輸資料有適當保護。
- [ ] 金鑰與秘密有生命週期管理。
- [ ] Cloud Audit Logs 已啟用並受到保護。
- [ ] 高權限與資源變更有告警。
- [ ] VM、容器與應用程式持續更新。
- [ ] 備份與復原流程經過測試。
- [ ] 雲端費用與資料保留成本有監控。

## 14. Knowledge Check（測驗）

1. IaaS、PaaS 與 SaaS 的主要差異是什麼？
2. 什麼是 Shared Responsibility Model？
3. 為什麼雲端身分是重要安全邊界？
4. Control Plane 與 Data Plane 有何不同？
5. IAM Role 與長期 Access Key 有何差異？
6. AWS Security Group 與 Network ACL 有何不同？
7. Azure Activity Log 與 Entra Sign-in Logs 分別記錄什麼？
8. 為什麼不能把秘密寫進 IaC 或 GitHub？
9. 雲端事件調查需要保留哪些資料？
10. 為什麼必須設定預算與費用告警？

## Sources（資料來源）

- Microsoft Learn — Shared Responsibility: https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility
- Microsoft Learn — Azure Security: https://learn.microsoft.com/en-us/azure/security/
- Microsoft Learn — Azure Security Best Practices: https://learn.microsoft.com/en-us/azure/security/fundamentals/best-practices-and-patterns
- AWS Well-Architected Security Pillar: https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/
- AWS Shared Responsibility Model: https://aws.amazon.com/compliance/shared-responsibility-model/
- AWS IAM Best Practices: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html
- NIST SP 800-210: https://csrc.nist.gov/pubs/sp/800/210/final

雲端產品功能、服務名稱與計價可能變動。實作前應閱讀對應服務的官方文件與費用說明。
