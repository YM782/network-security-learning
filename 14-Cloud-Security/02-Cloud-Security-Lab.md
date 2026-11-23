# Cloud Security Lab（AWS／Azure 雲端安全實作）

本實驗以安全稽核與組態檢查為主，帶你理解雲端帳號、身分、網路、儲存、日誌與成本管理。你可以先閱讀與畫圖，再決定是否建立真實雲端測試帳號。

**重要成本提醒：** 雲端服務可能產生費用。開始前先設定預算告警，確認服務價格與免費額度，實驗完成後刪除不再使用的資源。不要建立公開資料庫、公開儲存桶或不受控的管理入口。

## 1. Lab Goals（實驗目標）

- 理解 AWS Account 與 Azure Tenant／Subscription 的管理邊界。
- 建立最低權限的測試身分。
- 檢查 MFA 與管理員權限。
- 理解 VPC／VNet、Subnet、Security Group／NSG。
- 檢查儲存服務的公開存取與加密設定。
- 查詢雲端 Audit Logs。
- 建立雲端安全稽核報告。

## 2. Before You Start（開始前準備）

1. 使用個人專用實驗帳號，不使用公司或他人的雲端帳號。
2. 啟用 MFA。
3. 設定帳單預算與通知。
4. 閱讀服務計價與免費額度條件。
5. 不要把 Root／Global Administrator 帳號用於日常練習。
6. 不要把 Access Key、Client Secret、Token 或私鑰存入 GitHub。
7. 使用獨立測試資源群組／專案，避免與正式資源混在一起。

若暫時不想註冊雲端帳號，可以先用官方架構圖與本章政策範例完成紙上稽核。

## 3. AWS Security Review（AWS 安全檢查）

### 3.1 Account and Identity（帳號與身分）
- Root User 是否啟用 MFA。
- 是否使用 IAM Identity Center 或受控 Federation。
- 管理員是否使用個人化身分，而非共用帳號。
- 是否存在長期未使用的 Access Key。
- IAM Role 的 Trust Policy 是否只允許預期身分。
- 權限政策是否使用最小權限。

### 3.2 Read-only IAM Policy Example（唯讀 IAM 政策範例）

以下範例只允許列出指定測試 S3 Bucket 的內容。請替換成自己的測試 Bucket 名稱，並閱讀 AWS IAM Policy 文件。

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListOnlyOneLabBucket",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::YOUR-LAB-BUCKET"
    }
  ]
}
```

這只是教學政策片段，不是完整帳號安全設定。還需要考慮信任政策、其他允許／拒絕政策、SCP、Permission Boundary、Session Policy 與資源政策。

### 3.3 Network Review（網路檢查）
- VPC CIDR 是否與既有網路重疊。
- Subnet 是否按工作負載分段。
- Security Group 是否只開放必要連線。
- SSH／RDP 是否暴露給整個網際網路。
- 是否使用受控 VPN 或 Bastion。
- VPC Flow Logs 是否啟用並有保存策略。

### 3.4 Storage Review（儲存檢查）
檢查 S3 的 Block Public Access、Bucket Policy、ACL、預設加密、KMS 金鑰、Versioning、備份與存取日誌。不要上傳真實個人資料或公司資料。

### 3.5 Audit and Detection（稽核與偵測）
檢查 CloudTrail 是否記錄管理活動、日誌是否存放於受保護位置、是否監控高權限 IAM 變更、公開儲存設定與網路規則變更，以及 GuardDuty／Security Hub 的評估與成本。

## 4. Azure Security Review（Azure 安全檢查）

### 4.1 Tenant and Identity（租戶與身分）
檢查 Microsoft Entra ID 管理員角色、MFA、Conditional Access、緊急存取帳號、服務主體與憑證生命週期、Managed Identity，以及 Azure RBAC 最小權限。

### 4.2 Subscription and Resource Groups（訂閱與資源群組）
檢查 Subscription 擁有人、Resource Group 分隔、Management Group、Azure Policy、資源標籤與允許部署區域。

### 4.3 Network Review（網路檢查）
檢查 VNet／Subnet 分段、NSG 規則、公網 IP、Private Endpoint、VPN／Bastion 與 Network Watcher 流量紀錄。

### 4.4 Storage and Key Vault（儲存與金鑰保護）
檢查 Storage Account Public Network Access、Blob Container 匿名存取、加密、Key Vault 權限模型、網路限制、金鑰到期日與備份。

### 4.5 Logging and Defender（日誌與防護）
檢查 Azure Activity Log、Microsoft Entra Sign-in Logs、Audit Logs、Log Analytics Workspace、Defender for Cloud 與 Microsoft Sentinel，並評估資料保留與查詢費用。

## 5. Compare AWS and Azure（AWS 與 Azure 比較）

| Security Area | AWS | Azure |
|---|---|---|
| Identity | IAM、IAM Identity Center | Microsoft Entra ID |
| Resource Boundary | Account、Organization | Tenant、Subscription、Management Group |
| Network | VPC、Subnet、Security Group、NACL | VNet、Subnet、NSG、Azure Firewall |
| Audit | CloudTrail | Azure Activity Log、Entra Logs |
| Posture / Threat | Security Hub、GuardDuty、Config | Defender for Cloud、Azure Policy |
| Secrets | Secrets Manager、KMS | Key Vault、Managed HSM |
| SIEM | 可整合第三方 SIEM | Microsoft Sentinel |

產品名稱相似不代表功能完全相同。比較時應閱讀各服務功能範圍、計價與責任說明。

## 6. Safe Audit Worksheet（安全稽核工作表）

| Check | Expected State | Finding | Risk | Recommendation |
|---|---|---|---|---|
| MFA | 高權限身分啟用 MFA | 待填寫 | 待評估 | 啟用並測試復原流程 |
| IAM | 最小權限 | 待填寫 | 待評估 | 收斂權限與角色 |
| Public Storage | 無不必要公開存取 | 待填寫 | 待評估 | 限制公開存取 |
| Network | 只開放必要流量 | 待填寫 | 待評估 | 收斂 NSG／SG |
| Audit Logs | 已啟用並受保護 | 待填寫 | 待評估 | 集中收集與保留 |
| Secrets | 不在程式碼中 | 待填寫 | 待評估 | 使用 Secret Manager |
| Backup | 可復原且經測試 | 待填寫 | 待評估 | 定期演練復原 |
| Cost | 已設定預算告警 | 待填寫 | 待評估 | 設定預算與通知 |

## 7. Cleanup（清理實驗資源）

1. 停止不再使用的 VM、資料庫與運算資源。
2. 刪除測試用公網 IP、磁碟、快照與閒置資源。
3. 檢查 Log Analytics、CloudWatch 與其他日誌儲存是否持續計費。
4. 撤銷不再使用的測試角色、Token 與憑證。
5. 確認沒有公開儲存桶或公開管理入口。
6. 檢查帳單與預算告警。
7. 保存非敏感實驗報告，不保存秘密或真實資產清單。

## 8. Completion Checklist（完成檢查）

- [ ] 已理解 Shared Responsibility Model。
- [ ] 已完成 AWS 或 Azure 身分安全檢查。
- [ ] 已檢查 MFA 與高權限帳號。
- [ ] 已檢查網路分段與公開管理入口。
- [ ] 已檢查儲存公開存取與加密。
- [ ] 已檢查雲端 Audit Logs。
- [ ] 已評估資料保留與服務費用。
- [ ] 已完成資源清理。
- [ ] 已撰寫安全稽核報告。

## Sources（資料來源）

- AWS Well-Architected Security Pillar: https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/
- AWS Shared Responsibility Model: https://aws.amazon.com/compliance/shared-responsibility-model/
- AWS IAM Best Practices: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html
- Microsoft Learn — Shared Responsibility: https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility
- Microsoft Learn — Azure Security: https://learn.microsoft.com/en-us/azure/security/
- Microsoft Learn — Azure Security Best Practices: https://learn.microsoft.com/en-us/azure/security/fundamentals/best-practices-and-patterns

本實驗以授權稽核與安全組態檢查為主。不要在未授權的雲端帳號、租戶或資源上執行測試。
