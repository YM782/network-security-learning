# Linux and Windows Fundamentals（Linux 與 Windows 基礎）

本課程學習兩種常見作業系統的基本管理方式。目標是能檢查系統狀態、理解權限、管理程序與服務，並讀取日誌。

## 1. Users and Groups（使用者與群組）

Linux：
```bash
whoami
id
getent passwd
getent group
```

Windows PowerShell：
```powershell
whoami
whoami /groups
Get-LocalUser
Get-LocalGroup
```

使用者（User）代表登入身分；群組（Group）用來集中管理一組使用者的權限。

## 2. File Permissions（檔案權限）

Linux 權限通常分為 Owner（擁有者）、Group（群組）與 Others（其他使用者），基本權限為 Read（讀取）、Write（寫入）、Execute（執行）。

```bash
ls -l
stat filename
chmod u+x script.sh
chmod 640 notes.txt
```

- `chmod u+x`：替檔案擁有者增加執行權限。
- `chmod 640`：擁有者讀寫、群組唯讀、其他人無權限。
- 不要對不明原因的系統目錄使用遞迴的寬鬆權限。

Windows ACL（Access Control List，存取控制清單）可用以下命令查看：

```powershell
icacls C:\Lab
Get-Acl C:\Lab | Format-List
```

## 3. Processes（程序）

Linux：
```bash
ps aux
top
pgrep -a ssh
```

Windows：
```powershell
Get-Process
Get-Process -Name notepad
```

程序具有 PID（Process ID，程序識別碼）。結束程序前，先確認程序名稱、擁有者與用途。

## 4. Services（服務）

Linux 使用 systemd 的系統可使用：

```bash
systemctl --type=service --state=running
systemctl status ssh
journalctl -u ssh --since today
```

Windows：
```powershell
Get-Service
Get-Service -Name Spooler
Get-Service -Name Spooler | Select-Object *
```

管理服務前先了解它提供的功能。不要為了測試而停止未知的系統服務。

## 5. Network Inspection（網路檢查）

Linux：
```bash
ip -br addr
ip route
ss -tulpn
```

Windows：
```powershell
Get-NetIPConfiguration
Get-NetAdapter
Get-NetTCPConnection
ipconfig /all
```

## 6. Logs（系統日誌）

Linux：
```bash
journalctl -b
journalctl -p err -b
journalctl -u NetworkManager --since today
```

Windows：
```powershell
Get-WinEvent -LogName System -MaxEvents 30
Get-WinEvent -LogName Application -MaxEvents 30
```

日誌常包含時間、來源、事件識別碼與訊息。排錯時先確認時間範圍，再依事件來源縮小範圍。

## 7. Lab（實驗）

在自己的 Linux 虛擬機：
1. 執行 `whoami` 與 `id`，記錄目前身分。
2. 建立一個練習資料夾與文字檔。
3. 使用 `chmod 640` 調整檔案權限。
4. 使用 `ls -l` 驗證權限。
5. 找出一項正在執行的服務，使用 `systemctl status` 查看狀態。
6. 查看該服務日誌，不要任意停止服務。

在 Windows 測試環境：
1. 執行 `whoami /all`。
2. 執行 `Get-Process`。
3. 執行 `Get-Service`。
4. 使用 `Get-WinEvent` 查看近期 System 事件。

## 8. Troubleshooting（故障排查）

| 問題 | 檢查 |
|---|---|
| Permission denied（權限遭拒） | `id`、`ls -l`、`stat` 或 Windows ACL |
| Service failed（服務啟動失敗） | `systemctl status`、`journalctl -u` 或 Windows 事件記錄 |
| Port already in use（連接埠已被使用） | Linux `ss -tulpn`；Windows `Get-NetTCPConnection` |
| Command not found（找不到命令） | `command -v`、套件管理器與 PATH |
| Access denied in PowerShell（PowerShell 存取遭拒） | 確認目前帳號權限與執行環境，不要盲目提升權限 |

## Sources（資料來源）

- Linux Journey: https://github.com/labex-labs/linuxjourney
- Linux man-pages: https://man7.org/linux/man-pages/
- Microsoft Learn: https://learn.microsoft.com/powershell/
- Microsoft Learn — Windows commands: https://learn.microsoft.com/windows-server/administration/windows-commands/windows-commands

本課程的操作請先在自己的測試帳號或虛擬機中練習。
