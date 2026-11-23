# 02-04｜真實漏洞研究案例：Windows PrintNightmare（CVE-2021-34527）

> 難度：初級至中級｜建議時間：90–120 分鐘｜先修：本模組 01–03
>
> 本課研究 Windows 服務、權限邊界和更新管理如何影響安全。只做本機唯讀檢查，不執行 PrintNightmare 利用程式，也不修改登錄檔。

## 一、真實案例概覽

PrintNightmare 是 2021 年 Windows Print Spooler（列印多工緩衝處理器）相關安全事件。CVE-2021-34527 被列為遠端程式碼執行漏洞。NVD 說明指出，漏洞與 Print Spooler 服務不正確地執行高權限檔案操作有關，成功利用可能取得 SYSTEM 權限。

先閱讀：
- [Microsoft MSRC：CVE-2021-34527 指引](https://www.microsoft.com/en-us/msrc/blog/2021/07/clarified-guidance-for-cve-2021-34527-windows-print-spooler-vulnerability)
- [NIST NVD：CVE-2021-34527](https://nvd.nist.gov/vuln/detail/cve-2021-34527)
- [CERT/CC：VU#383432](https://kb.cert.org/vuls/id/383432)

注意：這些是漏洞公告和事後技術分析，不是完整的原始發現日記。我們不會把公開資料沒有說明的研究過程編造成故事。

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

Print Spooler 負責處理列印相關工作。列印系統涉及列印工作、驅動程式、遠端管理介面和檔案操作。

研究這類系統時，需要追問：

1. 哪些使用者可以向服務提交請求？
2. 請求是否來自遠端？
3. 服務用哪個帳戶執行？
4. 使用者是否能控制檔案路徑、驅動程式或其他輸入？
5. 服務是否在執行高權限檔案操作之前完成驗證？
6. 低權限輸入是否能影響高權限操作？

CVE-2021-34527 的公開描述指出，Print Spooler 不正確地執行特權檔案操作，可能造成 SYSTEM 層級的程式碼執行。

不要把「列印服務存在」直接等同於「系統一定可被攻擊」。漏洞是否適用，還取決於系統版本、修補狀態、服務配置、政策和實際攻擊條件。

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

## 五、在自己的 Windows 10 電腦做唯讀檢查

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
