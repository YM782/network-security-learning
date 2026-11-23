
# 03 — 作業系統核心、特權模式與系統呼叫（Kernel, Privilege Modes and System Calls）

## 1. 本章學習成果

完成本章後，你應能：

- 說明作業系統核心與一般應用程式的責任邊界。
- 區分使用者模式、核心模式與 root 管理員身分。
- 理解系統呼叫、中斷、例外與一般函式呼叫的差異。
- 使用 strace 觀察 Linux 程式的系統呼叫。
- 理解 Windows Win32 API、核心服務與系統呼叫不是同一層介面。

## 2. Kernel 是什麼？

Kernel（核心）是作業系統中負責管理硬體資源與提供受控服務的核心部分。常見職責包括：

- Process / Thread Management：程序與執行緒管理。
- Memory Management：虛擬記憶體、頁面配置與回收。
- File Systems：檔案系統與檔案操作。
- Networking：網路協定堆疊與 Socket。
- Device Drivers：透過驅動程式管理硬體。
- Security / Access Control：權限檢查與安全邊界。
- Interrupt Handling：處理硬體中斷與部分例外。

一般應用程式不應直接任意操作硬體暫存器或修改核心資料結構。它通常透過系統呼叫或受控裝置介面請求核心提供服務。

## 3. User Mode 與 Kernel Mode

CPU 架構提供不同的特權層級。以 x86 為例，硬體定義 Ring 0 到 Ring 3 等特權級；一般作業系統通常讓核心程式碼在最高特權級執行，而一般應用程式在較低特權級執行。

| 項目 | User Mode（使用者模式） | Kernel Mode（核心模式） |
|---|---|---|
| 執行對象 | 一般應用程式與使用者空間程式碼 | 核心與核心驅動程式碼 |
| 特權 | 受限制 | 可執行受架構允許的特權操作 |
| 記憶體存取 | 受程序位址空間與權限限制 | 可管理核心映射及受控資源 |
| 錯誤影響 | 通常主要影響該程序 | 核心錯誤可能影響整個系統 |
| 典型入口 | 函式呼叫、系統呼叫、例外 | 系統呼叫入口、中斷／例外處理程序 |

**重要：root 不等於 Kernel Mode。** Linux 的 root 是使用者空間中的高權限帳號；root 程序仍然在 User Mode 執行。它可以透過核心提供的介面請求更多權限，但不會因此自動變成核心程式碼。

## 4. 一般函式呼叫與系統呼叫

### 4.1 一般函式呼叫

例如 C 程式呼叫自己的 calculate 函式：

- CPU 依照呼叫慣例傳遞參數。
- 控制流程跳到函式程式碼。
- 函式完成後返回呼叫位置。
- 整個過程通常仍在同一特權模式中執行。

### 4.2 系統呼叫（System Call）

例如程式需要開啟檔案、建立程序或讀取 Socket：

1. 應用程式呼叫 C 函式庫包裝函式，例如 open 或 read。
2. 函式庫準備系統呼叫編號與參數。
3. CPU 執行架構定義的系統呼叫指令或受控入口。
4. CPU 切換到核心特權狀態，進入核心系統呼叫處理路徑。
5. 核心檢查參數、權限與資源狀態，執行對應操作。
6. 核心回傳結果或錯誤。
7. CPU 返回使用者模式，程式繼續執行。

在 Linux x86-64 上，常見系統呼叫入口指令是 syscall；其他架構可能使用不同指令與 ABI。實際參數暫存器、系統呼叫編號與錯誤回傳規則應查對應架構文件。

### 4.3 Library Call 不等於 System Call

例如 printf 是 C 標準函式庫介面。它可能先在使用者空間格式化和緩衝文字，之後才透過 write 等系統呼叫把資料送往檔案描述符。

因此：

- 一次 printf 不一定對應一次 write。
- 一次 open 函式庫呼叫，在現代 Linux 上可能使用 openat 系統呼叫。
- 程式呼叫的函式名稱，不一定就是核心接收的系統呼叫名稱。

這也是為什麼要用 strace 觀察實際系統呼叫，而不能只從原始碼函式名稱推測。

## 5. 中斷、例外與系統呼叫

| 機制 | 來源 | 例子 |
|---|---|---|
| Interrupt（中斷） | 通常由外部硬體或中斷控制器通知 | 網路卡收到封包、計時器中斷 |
| Exception（例外） | CPU 執行指令時偵測到特定條件 | 除零、非法指令、頁面錯誤 |
| System Call（系統呼叫） | 程式主動請求核心服務 | read、write、mmap |

系統呼叫在硬體層面可能透過特殊指令與受控入口完成；它與硬體中斷、CPU 例外的來源和語意不同。不同架構的具體流程也不同。

## 6. Linux 實驗：用 strace 觀察系統呼叫

先確認工具：

```bash
command -v strace
```

若沒有安裝：

```bash
sudo apt update
sudo apt install strace
```

追蹤一個簡單命令：

```bash
strace -o ls.trace ls
less ls.trace
```

只觀察常見檔案操作：

```bash
strace -e trace=openat,read,write,close -o file.trace cat /etc/hostname
```

觀察重點：

- openat：嘗試開啟檔案或目錄。
- read：從檔案描述符讀取資料。
- write：向檔案描述符寫入資料。
- close：關閉檔案描述符。
- 回傳值：成功時常見非負值；失敗時可能顯示 -1 與錯誤名稱。

追蹤程序建立與執行：

```bash
strace -f -e trace=process -o process.trace sh -c 'echo hello'
```

-f 會跟隨子程序。不同 Shell、系統版本與程式實作會產生不同系統呼叫序列，實驗時應以自己的輸出為準。

### 6.1 如何閱讀一行 strace

範例：

```text
openat(AT_FDCWD, "/etc/hostname", O_RDONLY) = 3
```

拆解：

- openat：系統呼叫名稱。
- AT_FDCWD：以目前工作目錄作為相對路徑解析的參考位置。
- /etc/hostname：路徑參數。
- O_RDONLY：唯讀開啟旗標。
- = 3：成功回傳檔案描述符 3。

若看到：

```text
openat(...) = -1 ENOENT (No such file or directory)
```

表示核心回報路徑不存在或無法依該路徑找到目標。仍需結合工作目錄、掛載命名空間與實際路徑判斷原因。

## 7. Windows：API 層次的基本認識

Windows 應用程式常使用 Win32 API，例如檔案、程序、執行緒與記憶體管理介面。這些 API 可能經過系統函式庫與內部服務入口，最終由核心執行受控操作。

不要把下列概念混為一談：

- Win32 API：應用程式常用的程式設計介面。
- Native API：Windows 內部較低層的使用者模式介面；不是所有部分都屬於穩定公開合約。
- System Call：從使用者模式進入核心服務的機制。
- Kernel Routine：核心內部執行的函式。

建議初學者先使用文件化 Win32 API 與官方診斷工具，不要依賴未文件化的系統呼叫編號。

觀察工具：

- [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon)：觀察檔案、登錄、程序與執行緒活動。
- [Process Explorer](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer)：觀察程序、執行緒、DLL 與控制代碼。
- [Windows API documentation](https://learn.microsoft.com/en-us/windows/win32/api/)：查閱公開 API。

## 8. 常見誤區與故障排查

| 誤區／現象 | 正確理解或檢查方式 |
|---|---|
| root 就是在核心模式 | root 仍是使用者模式程序；核心模式由 CPU 特權狀態決定 |
| 每個函式呼叫都會進入核心 | 大量函式只在使用者空間執行 |
| printf 一定只產生一次 write | 緩衝與函式庫實作會影響系統呼叫次數 |
| strace 沒有輸出 | 檢查命令是否安裝、是否使用正確參數、是否追蹤到子程序 |
| Operation not permitted | 可能受 ptrace 權限、容器限制、安全政策或目標程序身分影響 |
| 看不到某些程序細節 | 權限、核心設定、容器命名空間或工具版本可能限制可見資訊 |

排錯順序：

1. 使用 strace --version 確認工具存在。
2. 先追蹤自己啟動的簡單命令。
3. 查看完整錯誤訊息與退出碼。
4. 再判斷是否涉及權限或系統安全政策。
5. 不要為了追蹤而隨意關閉系統安全機制。

## 9. 本章驗收題

1. 核心的主要責任有哪些？
2. 為什麼 root 程序仍然在使用者模式執行？
3. 一般函式呼叫與系統呼叫的控制流程差異是什麼？
4. 中斷、例外與系統呼叫分別由什麼事件觸發？
5. 為什麼 printf 不一定對應一次 write？
6. strace -f 的用途是什麼？
7. openat 回傳 -1 ENOENT 時，應該如何定位問題？

## 10. 原始資料

- [Linux man-pages — syscalls(2)](https://man7.org/linux/man-pages/man2/syscalls.2.html)。
- [strace 官方文件與專案](https://strace.io/)。
- [Linux Kernel Documentation](https://docs.kernel.org/)。
- [Intel SDM](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)。
- [MIT xv6 book — Traps and System Calls](https://mit-pdos.github.io/xv6-riscv-book/)。
- [Microsoft Win32 API](https://learn.microsoft.com/en-us/windows/win32/api/)。

本章只在自己的 Linux 虛擬機與 Windows 電腦上進行觀察。系統呼叫名稱與參數會依架構、核心版本和函式庫實作改變。
