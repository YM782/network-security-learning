
# 02 — 記憶體階層與虛擬記憶體（Memory Hierarchy and Virtual Memory）

## 1. 本章學習成果

完成本章後，你應能：

- 區分暫存器、Cache、RAM、SSD/HDD 的用途與特性。
- 解釋虛擬位址、實體位址、頁面、頁表與 TLB。
- 說明 Page Fault 不一定代表程式崩潰。
- 理解程序隔離、記憶體權限、ASLR、NX 與 Copy-on-Write。
- 使用 Linux /proc、pmap、free 等介面觀察記憶體。

## 2. 記憶體階層

| 層級 | 位置／用途 | 速度與容量的概念 |
|---|---|---|
| CPU Registers | CPU 內部，保存目前運算狀態 | 極快、數量有限 |
| L1/L2/L3 Cache | CPU 快取，保存常用指令與資料 | 通常比 RAM 低延遲、容量較小 |
| RAM | 主記憶體，保存執行中的程式與資料 | 容量較大、斷電後通常失去內容 |
| SSD/HDD | 長期儲存程式、檔案與資料 | 容量大、延遲通常遠高於 RAM |

這是概念層級。不同 CPU、記憶體、儲存裝置和工作負載的實際延遲差異很大，不要將某個網路圖表中的固定數字套用到所有電腦。

## 3. 為什麼需要虛擬記憶體？

假設兩個程序都使用位址 0x400000。這不代表它們正在存取同一個實體 RAM 位置。

作業系統為每個程序建立自己的虛擬位址空間。CPU 的 MMU（Memory Management Unit）依照目前的頁表，把虛擬位址轉換成實體位址，並檢查存取權限。

虛擬記憶體主要帶來：

1. **隔離**：一般程序不能任意讀寫其他程序的私有位址空間。
2. **抽象**：程式不必自行管理實體 RAM 的連續配置。
3. **保護**：頁面可設定可讀、可寫、可執行等權限。
4. **共享**：不同程序可映射同一份唯讀程式碼或共享記憶體。
5. **需求載入**：部分頁面可在真正需要時才載入。
6. **記憶體超額配置**：作業系統可結合分頁、檔案映射與交換空間管理工作集。

虛擬記憶體不是「把 SSD 當成 RAM」這麼簡單。它是位址轉換、保護、映射、需求分頁與記憶體回收等機制的集合。

## 4. Page、Page Table 與 TLB

### 4.1 Page（頁面）

虛擬位址空間通常以固定大小的頁面管理。常見系統的基本頁面大小是 4 KiB，但支援 Huge Pages 等其他頁面大小。

查詢目前 Linux 系統的基本頁面大小：

```bash
getconf PAGESIZE
```

輸出通常以位元組表示。不要假設所有架構或所有映射都只使用一種頁面大小。

### 4.2 Page Table（頁表）

頁表記錄虛擬頁面如何映射到實體頁框，以及相關權限與狀態。64 位元架構常使用多層頁表，避免為整個巨大虛擬位址空間建立單一平面表格。

概念流程：

1. 程式產生虛擬位址。
2. CPU 依照目前頁表根與位址索引逐層查詢頁表。
3. 找到有效映射後，取得實體頁框與頁內偏移。
4. 檢查讀寫執行權限。
5. 若轉換快取未命中，可能需要走訪頁表；若映射不存在或權限不允許，則觸發 Page Fault。

### 4.3 TLB（Translation Lookaside Buffer）

TLB 是 CPU 中用來快取近期虛擬位址轉換結果的硬體結構。它不是頁表本身。

- TLB Hit：找到可用轉換，通常可避免完整頁表走訪。
- TLB Miss：需要進一步查詢頁表；這不必然是程式錯誤。
- Page Fault：CPU 發現映射不存在或存取違規時，轉交核心處理。

### 4.4 Page Fault 的兩大情況

**可恢復的缺頁：**

- 頁面尚未載入，需要從檔案或交換空間讀入。
- Copy-on-Write 頁面第一次被寫入，需要複製頁面。
- 匿名記憶體首次被實際觸碰，需要配置實體頁面。

**無法正常恢復的存取錯誤：**

- 程式存取未映射的位址。
- 程式違反頁面權限，例如寫入唯讀頁面或執行不可執行頁面。
- 核心無法提供合法映射，最後可能向程序傳送訊號或讓程序終止。

因此，發生 Page Fault 不等於程式一定崩潰。缺頁是虛擬記憶體正常運作的一部分。

## 5. 程序虛擬位址空間

一個典型程序可能包含：

| 區域 | 常見用途 |
|---|---|
| 程式碼／Text | 可執行指令，通常可讀與執行，不應任意寫入 |
| Read-only Data | 常數等唯讀資料 |
| Data | 已初始化的全域與靜態資料 |
| BSS | 未初始化或零初始化的全域與靜態資料 |
| Heap | 動態配置記憶體；實際配置可能透過 brk 或 mmap |
| Memory Mappings | 共享函式庫、檔案映射與匿名映射 |
| Stack | 函式呼叫框架、自動變數與返回資訊等 |

這只是常見的概念模型。實際位址布局會受 ABI、作業系統、編譯器、載入器、ASLR、執行緒數量與程式配置影響。

## 6. 記憶體保護與安全機制

- **Read / Write / Execute 權限**：限制頁面可讀、可寫、可執行的方式。
- **NX / XD**：標示某些頁面不可執行，降低把資料區直接當程式碼執行的風險。
- **ASLR**：隨機化部分位址空間布局，提高攻擊者預測位址的難度。
- **DEP（Windows）**：資料執行防護，與不可執行頁面的概念相關。
- **Copy-on-Write（COW）**：多個程序可暫時共享頁面；其中一方要寫入時，再建立私有副本。
- **KASLR**：核心位址空間布局隨機化，屬於核心層級的保護機制。

這些防護能增加攻擊難度，但不是單獨就能修補記憶體安全漏洞。程式仍需正確處理邊界、生命週期、權限與輸入驗證。

## 7. Linux 觀察命令

### 7.1 系統記憶體概況

```bash
free -h
cat /proc/meminfo
vmstat 1 5
```

觀察：

- total：系統可管理的記憶體總量。
- available：核心估算可供新工作負載使用的記憶體，不等於單純的 free。
- buff/cache：核心用於快取與緩衝的記憶體；其中部分可在需要時回收。
- si / so：vmstat 中的 swap-in / swap-out 活動；單次非零不一定代表故障，要看持續趨勢與系統反應。

### 7.2 程序記憶體

先取得程序 PID：

```bash
pgrep -a bash
ps -eo pid,comm,rss,vsz --sort=-rss | head
```

選擇自己的測試 PID 後：

```bash
cat /proc/PID/status
cat /proc/PID/maps
pmap -x PID
```

將 PID 換成實際數字，不要直接照抄字面上的 PID。

重要欄位：

- VmSize：程序虛擬位址空間大小。
- VmRSS：目前駐留在實體記憶體中的頁面估計值。
- VmData：資料區相關記憶體統計。
- VmStk：堆疊相關統計。
- /proc/PID/maps：映射區域、權限、偏移、裝置與路徑等資訊。

RSS、VSZ、PSS、Private Dirty 等指標定義不同，不可直接互換。共享映射也會讓簡單的 RSS 加總高估整體實體記憶體使用量。

### 7.3 查看核心是否記錄 OOM

```bash
journalctl -k -b | grep -i -E 'out of memory|oom|killed process'
```

若沒有權限或系統使用不同日誌設定，可能看不到結果。不要因為沒有輸出就斷言從未發生 OOM。

## 8. Windows 觀察方式

PowerShell：

```powershell
Get-CimInstance Win32_OperatingSystem |
  Select-Object TotalVisibleMemorySize, FreePhysicalMemory

Get-Process |
  Sort-Object WorkingSet64 -Descending |
  Select-Object -First 10 ProcessName, Id, WorkingSet64, PrivateMemorySize64
```

- Working Set 是程序目前駐留在實體記憶體中的頁面集合。
- Private Memory 與 Working Set 的定義不同。
- 工作管理員的「記憶體」欄位不應直接等同於程序完整虛擬位址空間。

進一步工具：

- [Microsoft Sysinternals VMMap](https://learn.microsoft.com/en-us/sysinternals/downloads/vmmap)：分析程序虛擬與實體記憶體分類。
- [Microsoft Sysinternals Process Explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer)：查看程序、執行緒、DLL 與資源。

## 9. 常見故障與排查思路

| 現象 | 可能原因 | 先收集的證據 |
|---|---|---|
| 系統變慢、磁碟燈持續忙碌 | 記憶體壓力、交換活動或其他 I/O | free -h、vmstat 1 10、top、磁碟使用狀況 |
| 程式突然被終止 | OOM Killer、使用者終止、服務管理器或程式自身退出 | journalctl -k、程序退出碼、服務日誌 |
| Segmentation fault | 非法位址、越界、已釋放記憶體使用等 | 核心日誌、GDB、AddressSanitizer |
| Cannot allocate memory | 系統或程序資源限制、位址空間或記憶體壓力 | free -h、ulimit -a、/proc/PID/limits |
| VSZ 很大但 RSS 不高 | 大型保留映射、尚未觸碰的頁面或共享映射 | /proc/PID/maps、pmap -x |

排查時先保存時間、PID、完整錯誤訊息與系統狀態，再逐項驗證；不要一開始就清除快取或任意調整核心參數。

## 10. 本章驗收題

1. 虛擬位址為什麼不能直接當成實體 RAM 位址？
2. TLB 與 Page Table 的差別是什麼？
3. TLB Miss 與 Page Fault 是否相同？
4. 為什麼一次 Page Fault 不一定代表程式出錯？
5. ASLR 與 NX 分別想降低哪一類風險？
6. 為什麼 free 很低不一定代表系統記憶體不足？
7. VSZ、RSS 與 PSS 有什麼差別？

## 11. 原始資料

- [OSTEP — Address Spaces / Memory API](https://pages.cs.wisc.edu/~remzi/OSTEP/)。
- [Linux Kernel Documentation — Memory Management](https://docs.kernel.org/admin-guide/mm/index.html)。
- [Linux proc filesystem manual](https://man7.org/linux/man-pages/man5/proc.5.html)。
- [Linux free manual](https://man7.org/linux/man-pages/man1/free.1.html)。
- [MIT xv6 book — Page Tables](https://mit-pdos.github.io/xv6-riscv-book/)。
- [Microsoft VMMap](https://learn.microsoft.com/en-us/sysinternals/downloads/vmmap)。

本章為原創教學整理。頁面大小、核心介面與記憶體統計會隨架構及作業系統版本改變，應以實際環境和官方文件為準。
