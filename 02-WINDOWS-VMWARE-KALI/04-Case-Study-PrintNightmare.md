# 02-04｜真實漏洞研究案例：Windows PrintNightmare（CVE-2021-34527）

> 難度：初級至中級｜建議時間：90–120 分鐘｜先修：本模組 01–03
>
> 本課研究 Windows 服務、權限邊界和更新管理如何影響安全。只做本機唯讀檢查，不執行 PrintNightmare 利用程式，也不修改登錄檔。

## 一、真實案例概覽

PrintNightmare 是 2021 年 Windows Print Spooler（列印多工緩衝處理器）相關安全事件。這裡聚焦 **CVE-2021-34527**，其正式分類是遠端程式碼執行（RCE）；在符合條件的系統上，攻擊可能使程式碼以 SYSTEM 權限執行。

**先分清 CVE-2021-1675 和 CVE-2021-34527：** 兩者都涉及 Print Spooler，時間接近、技術領域相似，但 Microsoft 和 CERT/CC 明確指出它們是相似但不同的漏洞，不能把兩個 CVE 當成同一個問題。當時媒體和公開討論有時會把「PrintNightmare」當作整個事件的統稱，因此閱讀資料時一定要核對 CVE 編號。

先閱讀：
- [Microsoft MSRC：CVE-2021-34527 指引](https://www.microsoft.com/en-us/msrc/blog/2021/07/clarified-guidance-for-cve-2021-34527-windows-print-spooler-vulnerability)
- [NIST NVD：CVE-2021-34527](https://nvd.nist.gov/vuln/detail/cve-2021-34527)
- [CERT/CC：VU#383432，Print Spooler / AddPrinterDriverEx 分析](https://kb.cert.org/vuls/id/383432)
- [CVE.org：CVE-2021-34527](https://www.cve.org/CVERecord?id=2021-34527)

注意：CERT/CC VU#383432 是一份涵蓋 Print Spooler 相關公開利用與變體的綜合說明，正文也明確區分 CVE-2021-1675 和 CVE-2021-34527。不要把其中每個 PoC 細節自動歸到單一 CVE；本課用它來理解 Spooler、RPC 與驅動程式安裝介面的安全邊界，具體 CVE-2021-34527 的受影響性仍以 Microsoft/NVD 對該 CVE 的公告為準。

本案例使用的是官方公告和 CERT/CC 的事後技術分析，不是完整的原始研究日記。以下會把「公開資料確認的技術事實」和「我們用來學習的通用分析流程」分開，不會編造研究員沒有公開的發現時間線。

## 二、先理解 Windows 服務

Windows Service（Windows 服務）是可以在背景執行、通常不需要使用者一直開啟視窗的程式。

例如：
- Windows Update。
- Print Spooler。
- Windows Defender 相關服務。
- 網路和系統管理服務。

服務通常具有服務名稱、執行檔路徑、啟動類型、執行帳戶和權限。

### 為什麼安全研究員會關注服務？

因為服務可能：
- 接收本機或遠端請求。
- 讀取使用者提供的資料。
- 操作檔案和系統資源。
- 以比一般使用者更高的權限執行。

研究問題不是「這個服務有沒有開啟」這麼簡單，而是：

**低權限使用者能不能透過服務提供的功能，讓高權限服務替他完成不應允許的操作？**

這就是信任邊界（Trust Boundary）和權限邊界（Privilege Boundary）。

## 三、PrintNightmare 的安全問題

Print Spooler 不只是接收列印工作。它還提供安裝和管理印表機及驅動程式的介面，因此會碰到高權限檔案操作。

CERT/CC 的技術說明指出，研究這條攻擊面時要特別注意印表機驅動程式安裝介面，例如 `RpcAddPrinterDriverEx()`，以及相關的非同步 RPC 路徑。該介面會接收描述驅動程式的資料；如果服務對呼叫者權限、驅動程式來源和檔案操作的限制不充分，就可能讓由高權限執行的 Print Spooler（`spoolsv.exe`）載入攻擊者控制的 DLL。

把信任邊界畫清楚：

    已驗證的低權限使用者
              |
              v
    印表機驅動程式安裝 RPC 介面
              |
              v
    Print Spooler（高權限服務）
              |
              v
    驅動程式檔案處理 / DLL 載入
              |
              v
        SYSTEM 權限執行

CERT/CC 描述的情境包括：遠端已驗證使用者透過 SMB/RPC 發出相關請求，可能讓服務以 SYSTEM 權限執行任意 DLL；在某些條件下，本機未提權使用者也可能造成權限提升。**不要把它簡化成「任何網路匿名使用者都能直接取得 SYSTEM」**；帳號、服務狀態、系統修補和 Point and Print 政策都會影響實際可利用性。

### Point and Print 設定扮演什麼角色？

Microsoft 在 2021 年 7 月的補充指引中特別提醒：安全更新不會自動改寫既有登錄檔設定。管理員還需要檢查：

- `NoWarningNoElevationOnInstall`
- `UpdatePromptSettings`

Microsoft 指引要求這些值設為 `0` 或未定義（預設安全狀態）。CERT/CC 也指出，將 `NoWarningNoElevationOnInstall` 設成非零值可能使系統處於不安全配置。

這些設定是重要的風險／緩解條件，但不要把「Point and Print 登錄值」直接等同於漏洞的全部根因。要分開分析程式本身的權限檢查，以及部署環境如何配置這些政策。

## 四、研究員的分析思路：從服務到權限邊界

以下是一個可重複使用的研究流程，不是漏洞原作者的發現時間線。

### 第一步：列出服務

先了解系統有哪些背景服務，以及它們各自的用途。

### 第二步：確認執行帳戶

一個服務由一般使用者帳戶執行，與由 LocalSystem 執行，風險完全不同。

### 第三步：找出外部輸入

服務可能接受 RPC、SMB、命名管道、本機 IPC 或其他請求。需要了解哪些資料由使用者控制。

### 第四步：畫出資料流

把流程畫成：

    一般使用者或遠端請求
              |
              v
       Print Spooler 介面
              |
              v
        參數與權限檢查
              |
              v
       高權限檔案操作
              |
              v
          系統結果

研究重點是確認每一層是否有正確驗證，尤其是由低權限輸入進入高權限操作的位置。

### 第五步：對照公告和修補

研究人員要區分：
- 漏洞根因。
- 觸發條件。
- 系統配置對風險的影響。
- 官方修補。
- 緩解措施。
- 修補後仍需檢查的政策設定。

Microsoft 特別提醒，安裝安全更新後仍要檢查文件所列的 Point and Print 政策值。這說明修補程式和安全配置檢查不能互相取代。

## 五、在自己的 Windows 測試主機做唯讀檢查

以下命令只讀取本機資訊，不會嘗試利用漏洞，也不會修改系統。

以 PowerShell 開啟終端機。

### 1. 查看 Print Spooler 服務狀態

    Get-Service -Name Spooler

觀察：
- Status：目前是否正在執行。
- Name：服務名稱。
- DisplayName：顯示名稱。

服務正在執行不代表存在漏洞；服務已停止也不代表整台電腦就安全。

### 2. 查看服務執行帳戶

    Get-CimInstance Win32_Service -Filter "Name='Spooler'" |
      Select-Object Name, State, StartMode, StartName, PathName

重點：
- StartName：服務執行帳戶。
- StartMode：啟動方式。
- PathName：服務執行檔路徑。

### 3. 查看最近安裝的更新

    Get-HotFix |
      Sort-Object InstalledOn -Descending |
      Select-Object -First 15 HotFixID, InstalledOn, Description

這只是初步檢視。Get-HotFix 不一定完整列出所有更新類型，不能單靠它判定某個 CVE 是否已修補。

### 4. 唯讀檢查 Point and Print 政策

    $path = "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Printers\PointAndPrint"

    if (Test-Path $path) {
        Get-ItemProperty -Path $path
    } else {
        Write-Output "Policy key does not exist"
    }

這段程式只讀取登錄檔。不要為了練習而自行建立或修改這些值。

## 六、如何判斷結果？

請記錄：
- Windows 版本和 OS Build。
- Print Spooler 是否啟動。
- 服務執行帳戶。
- 最近更新紀錄。
- Point and Print 政策是否存在。
- 是否已依 Microsoft 公告完成修補。

如果要確認目前系統是否受到保護，必須把 Windows 版本、更新修訂、供應商公告和政策設定一起核對。不能只看一個 KB 編號或一個服務狀態就下結論。

## 七、VMware 安全實驗設計

你可以在自己的 Windows 測試虛擬機中研究服務和權限：

1. 建立 Windows 測試 VM。
2. 建立快照，記錄快照名稱和時間。
3. 使用 NAT 或 Host-only，避免故意有漏洞的環境暴露到公共網路。
4. 練習查看服務、執行帳戶、更新和事件檢視器。
5. 只使用官方修補公告和唯讀檢查，不執行真實漏洞利用。
6. 實驗後關機並記錄結果。

不要在公司電腦或日常使用的主機上測試未修補的 PrintNightmare。

## 八、研究報告應該怎麼寫？

一份好的案例分析至少要回答：

| 欄位 | 本案例應記錄的內容 |
|---|---|
| 產品 | Windows Print Spooler |
| CVE | CVE-2021-34527 |
| 漏洞類型 | 遠端程式碼執行 |
| 信任邊界 | 使用者請求進入高權限服務 |
| 根因摘要 | 特權檔案操作的安全驗證問題 |
| 潛在影響 | 在符合條件時可能達到 SYSTEM 權限 |
| 版本與配置 | 必須依公告核對 |
| 修補 | Microsoft 安全更新及政策檢查 |
| 證據 | 系統版本、更新、服務與政策檢查結果 |

## 九、常見誤區

- 服務啟動不等於系統可利用。
- 有漏洞公告不等於所有 Windows 版本都受影響。
- 安裝更新不代表所有安全政策都已正確。
- Point and Print 配置是重要的風險因素，但不能把每一項緩解設定都誤稱為漏洞根因。
- CVE-2021-1675 和 CVE-2021-34527 不能混為一談；檢查公告、補丁和測試結果時要使用精確 CVE 編號。
- 不要在真實設備上為了驗證而執行公開 PoC。

## 十、本課驗收

- [ ] 能解釋 Windows Service 和服務執行帳戶。
- [ ] 能解釋低權限輸入進入高權限服務為什麼值得檢查。
- [ ] 能在自己的 Windows 10 上唯讀查看 Spooler 狀態。
- [ ] 能查看服務執行帳戶和更新紀錄。
- [ ] 能解釋為什麼必須核對官方公告和實際系統版本。
- [ ] 能寫出一份不包含攻擊利用的漏洞案例摘要。

## 十一、原始資料

1. [Microsoft MSRC：Clarified Guidance for CVE-2021-34527](https://www.microsoft.com/en-us/msrc/blog/2021/07/clarified-guidance-for-cve-2021-34527-windows-print-spooler-vulnerability)
2. [NIST NVD：CVE-2021-34527](https://nvd.nist.gov/vuln/detail/cve-2021-34527)
3. [CERT/CC：VU#383432](https://kb.cert.org/vuls/id/383432)

**本課最重要的一句話：** 研究 Windows 漏洞時，要追蹤「誰能輸入資料、服務以什麼權限執行、資料最後能影響什麼操作」，而不是只背漏洞名稱和 CVE 編號。
