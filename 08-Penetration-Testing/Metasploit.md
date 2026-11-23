# Metasploit Framework — Safe Lab Guide（Metasploit Framework 安全實驗手冊）

Metasploit Framework 是一套模組化的安全測試平台，包含輔助模組、漏洞驗證模組、Exploit 模組、Payload 與 Post 模組等。本手冊先從介面、模組辨識、設定檢查與低風險實驗開始。

**重要：** Metasploit 的部分模組可能造成服務中斷、資料變更或取得系統控制權。以下練習只限隔離靶場與明確授權環境；初學階段先熟悉模組資訊與低風險輔助檢查，不要在正式系統或第三方網路執行。

## 1. Framework Architecture（框架架構）

| Term（術語） | 中文說明 |
|---|---|
| Module（模組） | 執行特定任務的元件 |
| Auxiliary（輔助模組） | 資訊蒐集、協定檢查或其他輔助工作 |
| Exploit（漏洞利用模組） | 利用特定弱點觸發預期之外的行為 |
| Payload（載荷） | 漏洞觸發後執行的功能內容 |
| Post（後續模組） | 在已取得授權測試環境存取後執行的檢查工作 |
| Encoder（編碼器） | 對特定資料格式進行編碼轉換，不等於可靠的防禦或隱匿保證 |
| Session（工作階段） | 工具與測試目標之間建立的互動工作階段 |

模組類型與功能會隨 Framework 版本改變，應以本機 `info` 與官方文件為準。

## 2. Launch msfconsole（啟動主控台）

在 Kali Linux 中，若已安裝 Metasploit Framework：

```bash
msfconsole
```

進入後可使用：

```text
help
version
banner
exit
```

若命令找不到：
- 確認套件是否安裝。
- 檢查 Kali 軟體來源與套件管理狀態。
- 參考官方安裝文件，不要使用來路不明的安裝腳本。

## 3. Search and Read Modules（搜尋與閱讀模組）

常用命令：

```text
search type:auxiliary
search name:http
search platform:linux
info <MODULE_PATH>
```

先閱讀模組說明，再決定是否適合實驗。檢查：
- 模組用途與類型。
- 支援的平台與服務。
- 所需參數。
- 可能的副作用。
- 是否支援安全的 Check 功能。
- 參考資料與模組限制。

不要因為搜尋結果出現某個模組，就直接執行它。

## 4. Module Options（模組參數）

選定模組後，可使用以下命令檢查：

```text
use <MODULE_PATH>
info
show options
show advanced
show targets
```

不同模組可用命令與參數不同。先閱讀 `info` 與 `show options`，不要猜測參數名稱。

常見參數概念：
- RHOSTS：遠端測試目標。
- RPORT：遠端服務連接埠。
- THREADS：部分模組使用的執行緒數量。
- TIMEOUT：部分模組的逾時設定。

只將 RHOSTS 指向自己隔離靶場中的目標。不要填入不屬於授權範圍的網域或 IP。

## 5. Safe Local Lab（安全本機實驗）

先在本機確認 Metasploit 可以啟動並查詢模組，不連接任何遠端目標：

```text
msfconsole
version
search type:auxiliary
info auxiliary/scanner/portscan/tcp
```

以上步驟只查看本機模組資訊，不會對遠端系統執行掃描。

若要進行網路實驗：
1. 先在 VMware 建立 Host-only 隔離網路。
2. 確認 Kali 與靶機都在同一個實驗網段。
3. 記錄靶機 IP 與允許測試的服務。
4. 選擇經過理解且低影響的 Auxiliary Scanner。
5. 先檢查模組的 `info`、`show options` 與副作用。
6. 僅對實驗目標執行，並使用保守的測試設定。
7. 保存輸出，關閉模組並檢查靶機狀態。

本手冊不提供對公開目標的掃描範圍或批量測試設定。

## 6. Check Before Run（執行前檢查）

每次執行前確認：

- 目標是否明確列在授權範圍。
- 模組是否為 Auxiliary、Exploit 或 Post。
- 是否可能造成服務中斷、資料修改或建立 Session。
- 是否需要先備份或建立 VM Snapshot。
- 是否設定正確的目標與連接埠。
- 是否有停止條件與回復方式。

若不確定模組副作用，先不要執行。

## 7. Database and Workspace（資料庫與工作區）

Metasploit 可搭配資料庫管理測試工作區與部分測試資料。資料庫功能會依安裝方式與版本而不同。

可先檢查：

```text
db_status
workspace
workspace -a lab-notes
workspace
```

如果資料庫未連線，先依 Kali 與 Metasploit 官方文件確認服務與設定。不要把真實客戶資產或敏感測試資料放進個人公開環境。

## 8. Troubleshooting（故障排查）

| 問題 | 可能原因 | 檢查方向 |
|---|---|---|
| `msfconsole: command not found` | Framework 未安裝或 PATH 不正確 | 套件狀態與官方安裝文件 |
| 模組搜尋不到 | 模組路徑或本機版本不同 | `search`、`version` |
| 缺少必要參數 | 尚未設定模組必填選項 | `info`、`show options` |
| Target unreachable | 實驗網路、路由或靶機狀態錯誤 | VMware 網卡、IP 與路由 |
| Connection refused | 目標服務沒有監聽或主動拒絕 | 在靶機檢查服務狀態 |
| Database not connected | 資料庫服務或設定問題 | `db_status` 與官方文件 |
| Module failed | 版本不相容、目標不符合條件或參數錯誤 | 閱讀完整輸出與模組說明 |

## 9. Lab Notes Template（實驗紀錄範本）

```text
Date / 日期：
Metasploit version / 版本：
Lab topology / 實驗拓樸：
Authorized target / 授權目標：
Module path / 模組路徑：
Module type / 模組類型：
Purpose / 實驗目的：
Options reviewed / 已檢查參數：
Potential side effects / 潛在副作用：
Expected result / 預期結果：
Actual result / 實際結果：
Troubleshooting / 故障排查：
Cleanup / 清理與還原：
Conclusion / 結論：
```

## 10. Knowledge Check（知識測驗）

1. Auxiliary、Exploit、Payload 與 Post 模組的差異是什麼？
2. 為什麼應先閱讀 `info` 與 `show options`？
3. 為什麼模組搜尋結果不能直接視為漏洞存在的證據？
4. 執行可能造成服務中斷的模組前，應完成哪些準備？
5. 為什麼測試資料庫與工作區也需要保護？

## 11. Sources（資料來源）

- Metasploit Framework: https://github.com/rapid7/metasploit-framework
- Rapid7 Metasploit Documentation: https://docs.metasploit.com/
- Kali Metasploit Framework: https://www.kali.org/tools/metasploit-framework/
- VMware Documentation: https://docs.vmware.com/

本手冊是安全學習入門材料。只在自有隔離靶場或明確授權環境中使用 Metasploit。
