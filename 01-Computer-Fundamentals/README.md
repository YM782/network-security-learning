# Computer Fundamentals（計算機底層基礎）

本課程從硬體、作業系統與程式執行流程開始。學習目標不是背誦名詞，而是能解釋一個程式如何從儲存裝置載入、進入記憶體、由 CPU 執行，並透過作業系統使用硬體資源。

## 1. Computer Architecture（計算機架構）

一台一般電腦可先理解為幾個主要部分：

| Component（元件） | English term | Function（功能） |
|---|---|---|
| 中央處理器 | CPU | 執行指令、進行運算與控制 |
| 暫存器 | Registers | CPU 內部極快速的小型儲存位置 |
| 快取記憶體 | Cache | 暫存近期或常用資料，減少存取 RAM 的延遲 |
| 主記憶體 | RAM | 暫時保存正在執行的程式與資料 |
| 儲存裝置 | SSD / HDD | 長期保存作業系統、程式與檔案 |
| 主機板 | Motherboard | 連接 CPU、記憶體、儲存裝置與周邊 |
| 匯流排／互連 | Bus / Interconnect | 傳輸資料、位址與控制訊號 |

### CPU（中央處理器）

CPU 會依照程式指令執行運算。初學時可先理解 Instruction Cycle（指令週期）：

1. Fetch（擷取）：取得下一條指令。
2. Decode（解碼）：判斷指令要做什麼。
3. Execute（執行）：進行運算、資料搬移或控制流程。
4. Write Back（寫回）：必要時將結果寫回暫存器或記憶體。

現代 CPU 會使用 Pipeline（管線）、Branch Prediction（分支預測）、Out-of-Order Execution（亂序執行）等技術，因此實際執行並非簡單地一次只做一條指令。

### Registers（暫存器）

暫存器位於 CPU 內部，容量小但速度非常快。不同架構有不同暫存器名稱與用途，例如一般用途暫存器、指令指標／程式計數器、堆疊指標與狀態暫存器。

### Cache（快取）

CPU 通常具有多層快取，例如 L1、L2、L3。一般而言，越靠近核心的快取越快、容量越小。快取利用 Locality（區域性）原理：
- Temporal locality（時間區域性）：剛使用過的資料可能很快再次使用。
- Spatial locality（空間區域性）：某個位置附近的資料可能也會被使用。

## 2. Memory Hierarchy（記憶體階層）

可粗略理解為：暫存器 → L1/L2/L3 Cache → RAM → SSD/HDD。越靠近 CPU 通常速度越快、容量越小、成本越高。

### RAM vs. Storage（記憶體與儲存裝置）

- RAM（Random Access Memory，隨機存取記憶體）：程式執行時使用的主要工作空間，斷電後一般不保留資料。
- SSD/HDD：長期保存檔案與程式，關機後資料仍存在。
- Virtual Memory（虛擬記憶體）：作業系統提供給程式的位址空間抽象；不等於「把硬碟當成 RAM」。系統可能透過 Paging（分頁）及 Swap／Pagefile（交換空間／分頁檔）管理記憶體壓力。

## 3. Operating System（作業系統）

作業系統負責管理 CPU、記憶體、檔案、裝置與程序，並提供程式使用硬體的共同介面。

### Kernel and User Mode（核心模式與使用者模式）

- User Mode（使用者模式）：一般應用程式執行的受限制環境。
- Kernel Mode（核心模式）：作業系統核心執行具較高權限的程式碼。
- System Call（系統呼叫）：應用程式請求核心提供服務的介面，例如開啟檔案、建立程序或進行網路 I/O。

應用程式通常不能任意直接操作硬體，而是透過系統 API 與系統呼叫請求作業系統代為處理。

## 4. Process and Thread（程序與執行緒）

- Process（程序）：正在執行的程式實例，具有自己的虛擬位址空間與系統資源。
- Thread（執行緒）：程序中的執行流程。同一程序內的多個執行緒通常共享該程序的位址空間，但各自有執行堆疊與暫存器狀態。
- Context Switch（情境切換）：CPU 從一個執行緒切換到另一個執行緒時，作業系統需要保存與還原執行狀態。

注意：多執行緒不代表一定更快；共享資料可能產生 Race Condition（競爭條件）與 Deadlock（死結）。

## 5. Boot Process（開機流程）

以一般 UEFI 電腦為例，概念流程如下：

1. 電源啟動，硬體初始化。
2. UEFI 韌體執行硬體檢查與啟動項目選擇。
3. 載入 Bootloader（開機載入程式）。
4. Bootloader 載入作業系統核心。
5. 核心初始化記憶體、驅動程式與裝置。
6. 啟動第一個使用者空間程序與系統服務。
7. 顯示登入畫面或桌面。

實際細節會依 Windows、Linux、UEFI 設定與發行版而不同。

## 6. Linux Inspection Commands（Linux 檢查命令）

在自己的 Kali 或其他 Linux 虛擬機中執行：

```bash
# 核心與作業系統資訊
uname -a
cat /etc/os-release

# CPU 資訊
lscpu

# 記憶體概況
free -h

# 磁碟與分割區
lsblk
df -h

# 程序清單
ps aux

# 即時程序與資源使用
top

# 本次開機日誌
journalctl -b
```

### What to Observe（觀察重點）

- `lscpu`：CPU 架構、核心與執行緒數量。
- `free -h`：總記憶體、已使用與可用記憶體；Linux 的 buff/cache 不應直接視為浪費。
- `lsblk`：磁碟、分割區與掛載關係。
- `ps aux`：程序擁有者、PID、CPU 與記憶體使用情況。
- `journalctl -b`：本次開機的系統日誌。

## 7. Windows Inspection Commands（Windows 檢查命令）

在自己的 Windows 電腦或測試虛擬機中，使用 PowerShell：

```powershell
# 系統資訊
Get-ComputerInfo

# CPU
Get-CimInstance Win32_Processor |
  Select-Object Name, NumberOfCores, NumberOfLogicalProcessors

# 記憶體模組
Get-CimInstance Win32_PhysicalMemory |
  Select-Object Capacity, Speed, Manufacturer

# 程序
Get-Process

# 服務
Get-Service

# 系統事件記錄（最近 20 筆 System 錯誤）
Get-WinEvent -LogName System -MaxEvents 20 |
  Select-Object TimeCreated, Id, LevelDisplayName, ProviderName
```

## 8. Lab 1 — Observe Your Own VM（實驗一：觀察自己的虛擬機）

**目的：** 將硬體、記憶體與程序概念對應到真實系統。

**步驟：**
1. 在 Kali 執行 `lscpu`，記錄 CPU 架構、核心與執行緒數。
2. 執行 `free -h`，記錄總記憶體與可用記憶體。
3. 執行 `lsblk`，記錄磁碟與分割區。
4. 執行 `ps aux --sort=-%mem | head`，查看記憶體使用量較高的程序。
5. 開啟一個應用程式，再執行 `ps aux`，觀察程序變化。
6. 將結果寫入實驗紀錄，不要公開貼上包含個人識別資訊的完整輸出。

**預期成果：** 能指出 CPU、RAM、磁碟與程序資訊，並解釋各欄位的意義。

## 9. Troubleshooting（故障排查）

| 現象 | 可能原因 | 檢查方式 |
|---|---|---|
| 虛擬機很慢 | 配置資源不足、宿主機負載高、磁碟 I/O 忙碌 | 檢查 Task Manager、`top`、`free -h` |
| 可用記憶體偏低 | 程序佔用、快取、記憶體洩漏 | 比較 `free -h`、`ps aux` 與長時間變化 |
| 磁碟空間不足 | 日誌、快取或映像檔累積 | `df -h`、`du -h -d 1` |
| 找不到命令 | 套件未安裝或 PATH 不正確 | `command -v 命令名稱` |
| 權限不足 | 目前使用者沒有相應權限 | `whoami`、`id`、檢查檔案權限 |

排錯原則：先觀察、記錄錯誤訊息，再一次只改一項設定；不要在不理解原因時直接刪除系統檔案。

## 10. Knowledge Check（知識測驗）

1. CPU Cache 與 RAM 有什麼差異？
2. 為什麼程式需要透過作業系統核心請求部分硬體服務？
3. Process 與 Thread 的主要差異是什麼？
4. Virtual Memory 是否等於 RAM？
5. `ps aux` 與 `free -h` 分別能觀察什麼？

## Sources（資料來源）

- OSSU Computer Science: https://github.com/ossu/computer-science
- Operating Systems: Three Easy Pieces: https://pages.cs.wisc.edu/~remzi/OSTEP/
- Linux man-pages: https://man7.org/linux/man-pages/
- Microsoft Learn — Windows architecture: https://learn.microsoft.com/windows/

本頁為原創教學整理。架構細節請以處理器、作業系統與官方技術文件為準。
