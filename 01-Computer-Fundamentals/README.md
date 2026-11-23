
# 01 — Computer Fundamentals（計算機底層基礎）

> 本模組是一套從硬體執行、記憶體管理、作業系統核心，到程式載入與除錯的技術課程。每章包含原理、可觀察證據、命令、實驗、常見誤區與驗收題，不以名詞清單代替教學。

## 學習目標

完成本模組後，你應能：

1. 解釋 CPU 如何透過暫存器、指令集與執行單元執行程式。
2. 區分實體記憶體、虛擬位址、頁表、TLB、快取與磁碟。
3. 解釋使用者模式、核心模式、中斷、例外與系統呼叫的關係。
4. 區分程式、程序、執行緒、排程與上下文切換。
5. 追蹤 C 程式從原始碼、編譯、連結、載入到執行的基本流程。
6. 使用 Linux 與 Windows 工具觀察系統狀態，並根據證據排查問題。

## 課程地圖

| 章節 | 主題 | 必須掌握的成果 |
|---|---|---|
| [01 CPU 架構與指令執行](01-CPU-Architecture.md) | ISA、微架構、暫存器、指令週期、快取與管線 | 能解釋指令處理流程，讀懂基本 CPU 資訊 |
| [02 記憶體階層與虛擬記憶體](02-Memory-and-Virtual-Memory.md) | RAM、Cache、MMU、頁表、TLB、Page Fault、保護機制 | 能區分虛擬位址與實體位址，檢查程序映射 |
| [03 核心、使用者模式與系統呼叫](03-Kernel-and-System-Calls.md) | Kernel、特權級、System Call、中斷、例外 | 能用 strace 觀察程式與核心互動 |
| [04 程序、執行緒與排程](04-Processes-Threads-and-Scheduling.md) | Process、Thread、狀態、排程、Context Switch | 能追蹤程序樹、執行緒與資源使用 |
| [05 程式編譯、載入與 ELF](05-Program-Lifecycle-and-ELF.md) | 編譯、組譯、連結、ELF、動態載入 | 能檢查測試程式的檔案格式與載入資訊 |
| [06 動手實驗與故障排查](06-Labs-and-Troubleshooting.md) | 系統盤點、記憶體、程序、系統呼叫與除錯 | 完成可重現、有紀錄的實驗 |

## 建議環境

- 宿主機：Windows 10/11。
- 虛擬機：VMware Workstation 中的 Kali Linux 或其他 Debian 系 Linux。
- 權限：一般實驗先使用普通使用者；只有安裝套件或讀取受限資訊時才使用 sudo。
- 工具：GCC、binutils、procps、psmisc、strace、GDB；Windows 端可使用 PowerShell、工作管理員與 Microsoft Sysinternals。
- 實驗原則：只操作自己的電腦與隔離虛擬機；不要為了學習而修改宿主機核心、停用安全機制或執行來源不明的二進位檔。

### Kali 安裝實驗工具

```bash
cat /etc/os-release
uname -a
command -v gcc readelf objdump strace gdb
```

若缺少工具，可在 Kali/Debian 中安裝：

```bash
sudo apt update
sudo apt install build-essential binutils procps psmisc strace gdb
```

安裝前確認虛擬機有足夠磁碟空間，並在重要變更前建立 VMware Snapshot。

## 每章固定學習方法

1. **先理解原理**：先讀懂資料流與元件關係，不要先背命令。
2. **再觀察系統**：執行命令，記錄自己的實際輸出；不同 CPU、核心與發行版的結果可能不同。
3. **完成實驗**：每次只改一個變數，保留命令、輸出與觀察。
4. **解釋結果**：不只貼截圖，還要說明每個欄位代表什麼。
5. **故障排查**：先確認現象、收集證據，再提出假設與驗證。
6. **完成驗收題**：不看筆記也能說明原理，才算掌握。

## 重要概念邊界

- **CPU 架構（ISA）不等於 CPU 微架構**：ISA 定義軟體可見的指令與狀態；微架構是處理器實現 ISA 的內部設計。
- **程序不等於執行緒**：程序提供資源與虛擬位址空間；執行緒是排程器可派發的執行單位。
- **虛擬位址不等於實體位址**：一般程式看到的是虛擬位址，硬體 MMU 與作業系統共同完成轉換。
- **系統呼叫不等於一般函式呼叫**：系統呼叫透過受控入口切換到核心執行特權操作。
- **核心模式不等於 root 使用者**：root 是高權限使用者身分；核心模式是 CPU 的執行特權狀態，兩者不是同一概念。

## 主要原始資料與對照方式

| 原始資料 | 本課程使用方式 |
|---|---|
| [OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/) | 對照程序、位址空間、記憶體、執行緒與持久化章節 |
| [MIT xv6 教材](https://mit-pdos.github.io/xv6-riscv-book/) 與 [原始碼](https://github.com/mit-pdos/xv6-riscv) | 用小型教學作業系統理解核心、系統呼叫、記憶體與排程 |
| [Intel 64 and IA-32 Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html) | 查閱 x86-64 架構、暫存器、指令與系統程式設計 |
| [Linux man-pages](https://man7.org/linux/man-pages/) | 核對 Linux 系統呼叫、函式與命令介面 |
| [Linux Kernel Documentation](https://docs.kernel.org/) | 對照 Linux 核心子系統與實作文件 |
| [Microsoft Learn — Processes and Threads](https://learn.microsoft.com/en-us/windows/win32/procthread/about-processes-and-threads) | 對照 Windows 程序、執行緒與虛擬位址空間 |
| [Microsoft Sysinternals](https://learn.microsoft.com/en-us/sysinternals/) | 使用 Process Explorer、VMMap 等工具觀察 Windows |

**使用原則：** 本倉庫提供原創中文講解與實驗設計；原始教材、程式碼、圖表與商標仍歸原作者或權利人所有。引用或重新散布內容前，必須遵守原專案授權。外部文件可能更新，學習時以原始來源當前版本為準。

## 模組驗收清單

- [ ] 能畫出 CPU、RAM、儲存裝置與作業系統之間的基本關係。
- [ ] 能說明 ISA 與微架構的差別。
- [ ] 能解釋虛擬位址、頁表、TLB 與 Page Fault。
- [ ] 能解釋使用者模式與核心模式的差別。
- [ ] 能用 strace 找出測試程式發出的部分系統呼叫。
- [ ] 能用 ps、pstree、top -H 觀察程序與執行緒。
- [ ] 能使用 readelf 與 objdump 檢查自己的測試程式。
- [ ] 完成第 06 章所有實驗，並保存實驗紀錄。

> 完成標準不是「看過」，而是能在自己的虛擬機重現結果、解釋結果，並在結果不同時找到原因。
