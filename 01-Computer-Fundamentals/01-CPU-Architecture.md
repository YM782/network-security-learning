
# 01 — CPU 架構與指令執行（CPU Architecture and Instruction Execution）

## 1. 本章學習成果

完成本章後，你應能：

- 說明 ISA、微架構、核心、硬體執行緒之間的差別。
- 認識通用暫存器、RIP、RSP、RFLAGS 等 x86-64 可見狀態。
- 解釋取指、解碼、執行、寫回的教學模型，以及現代 CPU 為何不一定按這種簡單順序完成所有工作。
- 理解快取、管線、分支預測與亂序執行的目的。
- 在 Linux 中查看 CPU 型號、架構與旗標。

## 2. 先區分四個容易混淆的概念

| 名稱 | 意思 | 例子 |
|---|---|---|
| ISA（指令集架構） | 軟體可見的指令、暫存器、資料型態與例外行為規格 | x86-64、AArch64、RISC-V |
| Microarchitecture（微架構） | CPU 內部如何實作 ISA | 解碼器、執行單元、快取、分支預測器 |
| Core（核心） | 可獨立執行指令流的處理器核心 | 多核心 CPU 中的某一核心 |
| Hardware Thread（硬體執行緒） | 核心可維持的架構執行狀態；部分 CPU 以 SMT 同時支援多個 | Intel Hyper-Threading 是 SMT 的一種實作 |

同一個 ISA 可以由不同微架構實作。程式通常依賴 ISA 的規則，而不是依賴某一顆 CPU 內部的管線長度或快取容量。

### 為什麼這對資安重要？

- 惡意程式分析需要理解指令、暫存器與呼叫慣例。
- 漏洞研究會遇到堆疊、記憶體位址、指令邊界與控制流程。
- 側通道研究可能利用快取或分支預測等微架構行為；這屬於進階主題，不能把它簡化成一般程式可直接讀取快取內容。
- 核心漏洞與驅動程式問題常涉及特權級、記憶體保護與硬體介面。

## 3. CPU 的主要組成

教學上可將 CPU 分成下列部分：

1. **Control Unit（控制單元）**：協調指令解碼、執行與資料流。
2. **Execution Units（執行單元）**：執行整數、浮點、向量、位址計算等操作。
3. **Registers（暫存器）**：保存目前指令執行所需的少量高速狀態。
4. **Cache（快取）**：保存可能再次使用的指令與資料副本。
5. **Branch Predictor（分支預測器）**：預測條件分支方向，減少管線等待。
6. **Memory Interface（記憶體介面）**：與記憶體子系統交換資料。

實際 CPU 可能有多個解碼器、執行埠、重排序緩衝區與多層快取；不同廠商和世代的設計並不相同。

## 4. 暫存器：CPU 目前正在處理什麼

以 x86-64 為例，常見的 64 位元通用暫存器包括：

| 暫存器 | 常見用途（概念層面） |
|---|---|
| RAX | 常用於算術運算、函式回傳值等 |
| RBX | 通用暫存器；保存規則依呼叫慣例 |
| RCX | 通用暫存器；在部分指令與呼叫慣例中有特殊用途 |
| RDX | 通用暫存器；在部分運算與呼叫慣例中有特殊用途 |
| RSI、RDI | 通用暫存器；System V AMD64 呼叫慣例中常用於參數傳遞 |
| RBP | 常被編譯器用作 frame pointer，但不保證每個函式都使用 |
| RSP | Stack Pointer，指向目前堆疊頂端附近 |
| RIP | Instruction Pointer，表示下一步執行位置 |
| RFLAGS | 保存條件碼與控制狀態旗標 |

暫存器用途會受到指令集規格、作業系統 ABI、編譯器與呼叫慣例影響。不要把「常見用途」理解為所有程式都必須如此使用。

### 4.1 旗標暫存器

比較兩個整數後，CPU 會更新部分狀態旗標。常見旗標：

- ZF（Zero Flag）：結果是否為零。
- CF（Carry Flag）：無號運算是否產生進位／借位相關條件。
- SF（Sign Flag）：結果最高位所反映的符號狀態。
- OF（Overflow Flag）：有號運算是否溢位。

條件跳轉指令會依旗標決定是否改變控制流程。這是理解反組譯程式中 cmp、test、jz、jnz、jg、jl 等指令的基礎。

## 5. 指令週期：一個有用但簡化的模型

初學時常用以下流程理解指令：

1. **Fetch（取指）**：取得指令位元組。
2. **Decode（解碼）**：判斷指令種類、運算元與操作。
3. **Execute（執行）**：由相應執行單元完成運算或位址計算。
4. **Memory Access（記憶體存取）**：必要時讀取或寫入記憶體。
5. **Write Back（寫回）**：將結果寫入目的暫存器或記憶體。

這是教學模型，不表示現代 CPU 每次只處理一條指令，也不表示每條指令都必須經過完全相同的階段。

### 5.1 現代 CPU 為什麼更複雜？

- **Pipelining（管線）**：讓不同指令處於不同處理階段，以重疊工作。
- **Superscalar（超純量）**：在條件允許時，一個時脈週期可派發多條指令。
- **Out-of-Order Execution（亂序執行）**：在不改變架構可見結果的前提下，讓彼此獨立的操作先執行。
- **Register Renaming（暫存器重新命名）**：減少部分假相依。
- **Branch Prediction（分支預測）**：猜測條件分支方向，提前準備可能路徑。

CPU 必須維持程式規定的架構可見行為。內部可能亂序執行，但退休（retirement）機制會讓可見結果符合架構規則；例外與中斷也必須以規定方式呈現。

## 6. Cache：為什麼 CPU 不直接每次都讀 RAM？

CPU 核心與主記憶體之間存在速度與容量差距。快取保存近期或可能再次使用的資料，減少等待主記憶體的次數。

常見層級：

- L1：通常容量較小、延遲較低，常分成指令快取與資料快取。
- L2：通常比 L1 大。
- L3 / Last-Level Cache：常由多個核心共享，但實際設計依 CPU 而異。
- RAM：容量大得多，但存取延遲通常高於 CPU 快取。

快取利用兩種常見區域性：

- Temporal Locality（時間區域性）：剛用過的資料可能很快再用。
- Spatial Locality（空間區域性）：某個位址附近的資料可能也會被使用。

Cache、RAM 與磁碟不是同一種東西：它們在硬體位置、用途、容量、延遲與持久性上都不同。

## 7. 在 Kali Linux 觀察 CPU

### 7.1 架構與核心版本

```bash
uname -m
uname -r
lscpu
```

重點欄位：

- Architecture：目前系統架構，例如 x86_64。
- CPU(s)：作業系統可見的邏輯 CPU 數量，不一定等於實體核心數。
- Core(s) per socket：每個 CPU 插槽的核心數。
- Thread(s) per core：每個核心可見的硬體執行緒數。
- Model name：處理器型號。
- Flags：核心向作業系統呈現的 CPU 功能旗標。

虛擬機可能只看到 VMware 配置給它的虛擬 CPU，而不是宿主機的完整硬體拓撲。

### 7.2 CPU 資訊檔

```bash
less /proc/cpuinfo
grep -m 1 -E 'vendor_id|model name|cpu MHz' /proc/cpuinfo
```

cpu MHz 可能隨省電策略、負載與虛擬化環境變化，不應將單次讀值當成固定時脈。

### 7.3 即時負載

```bash
uptime
top
```

load average 是一段時間內可執行或處於不可中斷等待等狀態的工作負載平均值；它不是 CPU 使用率百分比。判讀時要同時考慮可見 CPU 數量、I/O 等待與程序狀態。

## 8. Windows 觀察方式

PowerShell：

```powershell
Get-CimInstance Win32_Processor |
  Select-Object Name, NumberOfCores, NumberOfLogicalProcessors, MaxClockSpeed

Get-CimInstance Win32_OperatingSystem |
  Select-Object Caption, Version, OSArchitecture
```

工作管理員的「效能 → CPU」可查看邏輯處理器與即時負載。若要更細緻觀察程序、執行緒與模組，可使用 Microsoft Sysinternals Process Explorer。

## 9. 常見誤區

| 誤區 | 正確理解 |
|---|---|
| 8 個邏輯處理器就是 8 個實體核心 | 邏輯 CPU 可能包含 SMT 執行緒 |
| CPU 主頻越高，所有程式一定越快 | 效能還受 IPC、快取、記憶體、分支與功耗限制 |
| 每條指令都只需一個時脈週期 | 指令延遲、吞吐量與微架構差異很大 |
| 程式直接讀取實體 RAM 位址 | 一般程序使用虛擬位址，由 MMU 與作業系統管理映射 |
| Cache 是 RAM 的一部分 | Cache 是處理器記憶體階層中的獨立硬體結構 |

## 10. 本章驗收題

1. ISA 與微架構有什麼差別？同一 ISA 能否有不同微架構？
2. 為什麼 RSP 與 RIP 的用途不同？
3. CPU(s) 為什麼不一定等於實體核心數？
4. 分支預測的目的為何？預測錯誤可能帶來什麼成本？
5. 時間區域性與空間區域性分別是什麼？
6. 為什麼不能只用 load average 判斷 CPU 使用率？

## 11. 原始資料

- [Intel 64 and IA-32 Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html) — ISA、暫存器、指令與系統程式設計規格。
- [OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/) — 虛擬化、並行與持久化的作業系統概念。
- [Linux lscpu manual](https://man7.org/linux/man-pages/man1/lscpu.1.html)。
- [Linux proc_cpuinfo manual](https://man7.org/linux/man-pages/man5/proc_cpuinfo.5.html)。
- [MIT xv6 book](https://mit-pdos.github.io/xv6-riscv-book/) — 以教學核心理解硬體與作業系統的交界。

本章內容為原創教學整理；具體指令行為與 CPU 功能應以處理器廠商手冊及目前系統文件為準。
