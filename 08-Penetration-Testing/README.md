# Authorized Penetration Testing（授權滲透測試方法）

滲透測試不是「看到 IP 就開始掃描」，而是一個有範圍、有規則、有證據、有風險控制並以修復建議為成果的專業流程。本頁提供從準備到報告的標準工作方法。

**重要原則：** 只有在系統擁有人明確授權、測試範圍清楚且遵守 Rules of Engagement（測試規則）時，才可執行測試。沒有授權時，不要掃描、登入、猜測密碼或驗證漏洞。

## 1. Rules of Engagement（測試規則）

測試前必須書面確認：

- 測試目的與委託方。
- 允許的網域、IP、應用程式與 API。
- 禁止測試的資產與第三方服務。
- 測試時間、時區與維護窗口。
- 允許與禁止的測試類型。
- 是否允許帳號建立、資料修改或服務壓力測試。
- 緊急聯絡人與停止測試條件。
- 證據保存、資料保護與報告交付方式。

不要自行把「同一家公司」或「同一個 IP 段」理解成已獲授權。

## 2. Assessment Workflow（測試流程）

### Phase 1 — Planning（規劃）

輸出：
- Scope（範圍）。
- Rules of Engagement（測試規則）。
- Test Accounts（測試帳號）。
- Lab / Production Separation（測試與正式環境區分）。
- Communication Plan（溝通與緊急聯絡方式）。

### Phase 2 — Asset Inventory（資產盤點）

在核准範圍內記錄：
- 主機與網域。
- 作業系統與服務。
- Web 應用程式與 API。
- 使用者角色與測試帳號。
- 外部依賴與信任邊界。

每項資產都要能對應到授權範圍。

### Phase 3 — Enumeration（資訊整理）

整理可公開或由授權環境提供的技術資訊，例如：
- HTTP Response Headers。
- TLS 憑證資訊。
- 應用程式版本與功能。
- 登入、角色與權限模型。
- API 文件與資料流。

避免大量或高頻請求；遵守速率限制與測試窗口。

### Phase 4 — Vulnerability Validation（弱點驗證）

每個發現都應：
1. 明確指出前置條件。
2. 使用最小化、低風險的驗證步驟。
3. 保存請求與回應證據。
4. 評估實際影響。
5. 避免讀取、修改或下載不必要的真實資料。
6. 在無法安全驗證時標記為待確認，而不是宣稱已證實。

### Phase 5 — Risk Assessment（風險評估）

考慮：
- 技術嚴重性。
- 是否需要登入或特殊權限。
- 可接觸的資料與功能。
- 對業務的影響。
- 是否存在緩解措施。
- 資產對組織的重要性。

CVSS 可作為技術嚴重性參考，但不能取代組織自己的風險判斷。

### Phase 6 — Reporting and Retest（報告與複測）

交付：
- Executive Summary（管理摘要）。
- Scope and Methodology（範圍與方法）。
- Findings（弱點清單）。
- Evidence（證據）。
- Impact and Severity（影響與嚴重性）。
- Remediation（修復建議）。
- Retest Results（複測結果）。

## 3. Tool Selection（工具選擇）

| Tool（工具） | Purpose（用途） | Learning boundary（學習範圍） |
|---|---|---|
| Nmap | 資產與服務識別 | 僅限授權網段 |
| Wireshark | 封包分析 | 自有或授權流量 |
| Burp Suite Community | Web 請求分析與手動測試 | 本機靶場與授權 Web |
| Metasploit Framework | 安全驗證與實驗模組 | 隔離靶場、先理解模組副作用 |
| Goby | 資產盤點與漏洞管理 | 授權資產 |
| OWASP ZAP | Web 安全測試 | 本機靶場與授權 Web |

工具輸出不是最終結論。要檢查誤報、版本差異、設定差異與實際可利用條件。

## 4. Evidence Handling（證據管理）

- 只收集證明弱點所需的最少資料。
- 使用虛構帳號與測試資料。
- 遮蔽 Cookie、Authorization Header、Token、密碼與個資。
- 記錄時間、時區、工具版本與測試人員。
- 將證據存放在受控位置，限制存取權。
- 不要把敏感證據直接提交到公開 GitHub。

## 5. Stop Conditions（停止測試條件）

出現以下情況時立即停止相關測試並聯絡委託方：
- 服務不穩定或出現非預期中斷。
- 觸及未列入範圍的第三方資產。
- 意外看到真實使用者的敏感資料。
- 發現可能造成資料破壞或大規模影響的弱點。
- 測試授權過期或範圍有爭議。

## 6. Sample Finding（弱點報告範例）

**標題：** 測試環境中的物件層級授權缺陷（IDOR）

- 資產：本機教學應用程式。
- 前置條件：存在兩個虛構測試帳號 A 與 B。
- 預期結果：B 只能存取 B 自己的測試物件。
- 實際結果：在教學環境中，伺服器未正確驗證物件擁有權。
- 影響：若同樣問題出現在正式環境，可能造成跨使用者資料未授權存取。
- 修復：在伺服器端對每個物件執行角色與擁有權檢查。
- 複測：以 A、B 兩種角色重新驗證允許與拒絕的情況。

這是示範報告結構，不代表任何真實網站存在此弱點。

## 7. Practical Deliverables（實作成果）

完成一個隔離靶場評估後，應保存：
1. 授權範圍說明。
2. 網路拓樸圖。
3. 資產清單。
4. 測試方法與工具版本。
5. 弱點證據與風險判斷。
6. 修復建議。
7. 複測紀錄。
8. 最終報告。

## 8. Sources（資料來源）

- OWASP Web Security Testing Guide: https://github.com/OWASP/wstg
- NIST SP 800-115: https://csrc.nist.gov/pubs/sp/800/115/final
- PTES: http://www.pentest-standard.org/
- FIRST CVSS: https://www.first.org/cvss/
- PortSwigger Web Security Academy: https://portswigger.net/web-security

本頁描述的是授權安全測試流程，不構成對任何未授權目標進行測試的許可。
