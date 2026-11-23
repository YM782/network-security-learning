
# 04 — 程序、執行緒與 CPU 排程（Processes, Threads and Scheduling）

## 1. 本章學習成果

完成本章後，你應能：

- 區分程式（Program）、程序（Process）與執行緒（Thread）。
- 說明程序資源、虛擬位址空間與執行緒上下文。
- 理解程序狀態、排程、優先權與上下文切換。
- 使用 Linux 命令查看程序樹、執行緒與狀態。
- 使用 Windows 工作管理員與 PowerShell 觀察程序。

## 2. Program、Process、Thread 有什麼差別？

| 概念 | 說明 | 例子 |
|---|---|---|
| Program（程式） | 儲存裝置上的可執行檔或程式碼 | /usr/bin/python3 |
| Process（程序） | 程式被載入並執行後形成的資源與執行環境 | 某個 PID 為 1234 的 Python 程序 |
| Thread（執行緒） | 程序內可被排程執行的控制流程 | Python 程序中的主執行緒 |

程序通常具有：

- 虛擬位址空間。
- 至少一個執行緒。
- 開啟的檔案描述符或控制代碼。
- 使用者與群組身分、安全憑證或安全上下文。
- 工作目錄、環境變數與資源限制。
- 程序識別碼 PID，以及父程序等關係資訊。

執行緒通常共享同一程序的位址空間與許多程序資源，但每個執行緒有自己的執行上下文，例如暫存器狀態、堆疊、排程狀態與執行緒識別碼。

## 3. 為什麼需要執行緒？

假設瀏覽器同時需要更新畫面、下載資料、執行 JavaScript、播放音訊及處理使用者輸入。若全部工作都由單一執行緒依序完成，某個長時間操作可能讓其他工作無法及時執行。多執行緒可讓工作分開排程，但也引入共享資料同步問題。

### 3.1 共享與獨立

同一程序內的執行緒通常共享：

- 程序虛擬位址空間。
- Heap 與全域資料。
- 開啟的檔案描述符等程序資源（具體語意依作業系統）。

每個執行緒通常有自己的：

- Stack。
- 暫存器上下文。
- Thread-local storage。
- 排程狀態與執行緒識別碼。

因為 Heap 常由多個執行緒共享，若兩個執行緒同時修改同一份資料，就可能出現 Race Condition（競爭條件）。後續學習鎖、原子操作與同步原語時，必須理解這個問題。

## 4. 程序與執行緒狀態

教學模型常用以下狀態：

- New：正在建立。
- Ready：可以執行，等待 CPU。
- Running：正在 CPU 上執行。
- Blocked / Waiting：等待 I/O、鎖、事件或其他條件。
- Terminated：已結束。

實際作業系統的狀態名稱與細分方式不同。Linux ps 的狀態字母不會完全等同於上述教學模型。

### 4.1 Linux 常見 ps 狀態字母

| 字母 | 大意 |
|---|---|
| R | Running 或 Runnable |
| S | Interruptible Sleep |
| D | Uninterruptible Sleep，常見於等待某些 I/O |
| T | Stopped 或被追蹤停止 |
| Z | Zombie，程序已結束但尚待父程序回收退出狀態 |

Zombie 不是「還在正常執行的程序」。它已經結束，保留少量程序表資訊，等待父程序透過 wait 系列介面取得退出狀態。

## 5. CPU Scheduler（排程器）

當多個執行緒都想使用 CPU 時，作業系統需要決定誰先執行、執行多久，以及何時切換。

排程考量可能包括：

- 響應時間：互動操作是否及時。
- 吞吐量：單位時間完成多少工作。
- 公平性：避免某些工作長期得不到 CPU。
- 優先權：重要工作是否能較快獲得執行機會。
- 即時性：特定工作是否必須在期限內完成。

現代 Linux 使用公平排程與即時排程等不同類型的策略；Linux 核心版本更新也可能改變預設排程器的實作。不要把舊教材中的排程器名稱當成所有新版本的固定現況。

## 6. Context Switch（上下文切換）

當 CPU 從一個執行緒切換到另一個執行緒時，作業系統需要保存目前執行緒的必要狀態，並恢復下一個執行緒的狀態。

概念上可能涉及：

1. 觸發排程事件，例如時鐘中斷、阻塞或主動讓出 CPU。
2. 保存目前執行緒的暫存器與必要執行狀態。
3. 更新排程資料結構。
4. 選擇下一個可執行執行緒。
5. 切換位址空間或相關記憶體管理狀態（若需要）。
6. 恢復下一個執行緒的上下文。
7. 返回適當的執行位置。

切換執行緒可能有成本，例如保存／恢復狀態、快取與 TLB 影響、排程器工作等。但上下文切換的成本取決於硬體、核心、工作負載與切換類型。

## 7. Linux 程序觀察

### 7.1 列出程序

```bash
ps -ef
ps aux
```

常見欄位：

- PID：程序識別碼。
- PPID：父程序識別碼。
- USER：程序使用者。
- STAT：程序狀態。
- %CPU：工具依時間區間估算的 CPU 使用比例。
- %MEM：記憶體使用比例估算。
- COMMAND：命令列或程序名稱。

### 7.2 查看程序樹

```bash
pstree -ap
```

程序樹可以幫助理解某個服務由誰啟動，以及它建立了哪些子程序。程序父子關係會隨服務管理器、登入方式與系統版本而變化。

### 7.3 觀察某個程序的執行緒

先找 PID：

```bash
pgrep -a firefox
```

選擇一個自己有權觀察的 PID：

```bash
ps -L -p PID
top -H -p PID
```

ps -L 列出該程序的執行緒；top -H 以執行緒為單位顯示。請把 PID 換成實際數字。

### 7.4 查看程序詳細狀態

```bash
cat /proc/PID/status
cat /proc/PID/limits
ls -l /proc/PID/fd
```

/proc/PID/fd 內的項目是檔案描述符連結。部分內容可能因權限或程序結束而無法讀取。

### 7.5 觀察程序建立與結束

```bash
strace -f -e trace=process -o proc.trace sh -c 'sleep 2'
```

這個實驗用來觀察自己啟動的測試程序。不同 Shell 可能使用不同建立程序方式，例如 fork、vfork 或 clone，不應期待每個系統輸出完全相同。

## 8. Windows 程序觀察

PowerShell：

```powershell
Get-Process |
  Sort-Object CPU -Descending |
  Select-Object -First 10 ProcessName, Id, CPU, Handles, Threads

Get-CimInstance Win32_Process |
  Select-Object ProcessId, ParentProcessId, Name, ExecutablePath
```

工作管理員：

1. 開啟「工作管理員」。
2. 進入「詳細資料」頁籤。
3. 顯示 PID、CPU、記憶體等欄位。
4. 進入「效能」頁籤查看 CPU 與記憶體概況。
5. 使用 Microsoft Process Explorer 深入查看執行緒與模組。

Windows 與 Linux 的程序模型有共通概念，但內部資料結構、API 和排程細節不同，不要直接把 Linux 的 task_struct 等同於 Windows 的內部結構。

## 9. 常見問題與排查

| 現象 | 可能原因 | 檢查方向 |
|---|---|---|
| CPU 使用率很高 | 程式忙迴圈、工作量大、執行緒競爭 | top -H、ps -L、效能分析工具 |
| 程序長時間為 D | 等待不可中斷 I/O 或核心路徑 | ps -o pid,stat,wchan,comm -p PID、核心日誌 |
| 出現 Zombie | 父程序尚未回收退出狀態 | ps -ef、查看 PPID 與父程序日誌 |
| 程序無法結束 | 等待核心操作、訊號處理或權限問題 | 先確認狀態與日誌；不要一開始就使用 kill -9 |
| 執行緒數量很多 | 程式設計、工作池配置或資源洩漏 | ps -L、top -H、程序資源限制 |
| 程序記憶體突然升高 | 配置增加、快取、映射或記憶體洩漏 | pmap -x、/proc/PID/status、時間序列觀察 |

排查時先記錄 PID、程序啟動時間、父程序、狀態與資源變化。PID 可能被重複使用，因此長時間分析時也要記錄程序啟動時間或其他識別資訊。

## 10. 本章驗收題

1. 程式、程序與執行緒的差別是什麼？
2. 同一程序中的兩個執行緒，哪些資料通常共享？哪些狀態通常獨立？
3. 為什麼多執行緒程式可能出現 Race Condition？
4. Linux R、S、D、T、Z 分別代表什麼？
5. Zombie 程序是否仍在執行指令？
6. Context Switch 需要保存與恢復哪些類型的狀態？
7. 為什麼不能單憑一瞬間的 CPU 百分比判斷程序是否異常？

## 11. 原始資料

- [OSTEP — Processes and Threads](https://pages.cs.wisc.edu/~remzi/OSTEP/)。
- [Linux proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html)。
- [Linux ps(1)](https://man7.org/linux/man-pages/man1/ps.1.html)。
- [Linux top(1)](https://man7.org/linux/man-pages/man1/top.1.html)。
- [Microsoft Learn — About Processes and Threads](https://learn.microsoft.com/en-us/windows/win32/procthread/about-processes-and-threads)。
- [MIT xv6 book](https://mit-pdos.github.io/xv6-riscv-book/)。

本章為原創教學整理。排程器實作與程序狀態會隨作業系統版本改變；請以實際環境輸出及對應版本文件為準。
