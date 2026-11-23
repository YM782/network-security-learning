
# 05 — 程式編譯、載入與 ELF（Program Lifecycle and ELF）

## 1. 本章學習成果

完成本章後，你應能：

- 說明 C 原始碼如何經過預處理、編譯、組譯與連結。
- 區分可執行檔、目標檔、靜態函式庫與動態函式庫。
- 認識 Linux ELF 檔案的 Header、Program Header、Section Header。
- 使用 file、readelf、objdump 檢查自己的測試程式。
- 理解程式啟動後，載入器如何建立程序的初始執行環境。

## 2. 從原始碼到可執行檔

常見 C 程式建置流程：

1. Preprocessing（預處理）：處理 include、define 與條件編譯。
2. Compilation（編譯）：將 C 語言轉成組合語言或中間表示。
3. Assembly（組譯）：將組合語言轉成目標檔中的機器碼。
4. Linking（連結）：解析符號參照，組合目標檔與函式庫。
5. Loading（載入）：作業系統與載入器建立程序執行環境。

GCC 可分階段觀察：

```bash
gcc -E hello.c -o hello.i
gcc -S hello.c -o hello.s
gcc -c hello.c -o hello.o
gcc hello.o -o hello
```

各階段輸出：

| 檔案 | 意義 |
|---|---|
| hello.c | C 原始碼 |
| hello.i | 預處理後的 C 程式 |
| hello.s | 組合語言 |
| hello.o | 可重定位目標檔 |
| hello | 連結後的可執行檔 |

以上命令假設已安裝 GCC。若使用其他編譯器或不同目標平台，輸出格式與選項可能不同。

## 3. Linker 與 Loader 的差別

**Linker（連結器）** 在建置階段處理符號與目標檔：

- 將不同目標檔的程式碼與資料組合。
- 解析函式與變數符號。
- 處理重定位。
- 決定靜態連結或動態連結的部分安排。

**Loader（載入器）** 在執行階段建立程式映像：

- 檢查可執行檔格式。
- 建立程序位址空間。
- 將需要的區段映射進位址空間。
- 設定初始堆疊、參數與環境資訊。
- 若是動態連結程式，啟動動態載入器解析共享函式庫。
- 將控制權交給程式入口流程。

Linux 的 execve() 會讓目前程序映像被新程式映像取代；它本身不是「建立另一個 PID」的同義詞。常見 Shell 會先建立子程序，再由子程序呼叫 execve()，但具體流程依 Shell 實作與情境而異。

## 4. ELF 是什麼？

ELF（Executable and Linkable Format）是 Linux 與許多 Unix-like 系統常用的二進位檔案格式。它可用於可執行檔、目標檔、共享函式庫與核心模組等。

### 4.1 ELF Header

ELF Header 說明檔案基本屬性，例如：

- Magic：識別 ELF 格式。
- Class：32-bit 或 64-bit。
- Data：位元組序。
- Type：可重定位檔、可執行檔、共享物件等。
- Machine：目標架構。
- Entry Point：程式入口位址（對應檔案類型與載入方式解讀）。

### 4.2 Sections 與 Segments

這兩個概念非常重要：

- **Section（區段）**：主要供連結器與工具理解程式碼、資料、符號表與重定位等內容。
- **Segment（段）**：主要描述載入時需要映射到記憶體的內容與權限。

例如 .text、.data、.bss 是常見 Section 名稱；PT_LOAD 是常見 Program Header 類型。不能把 Section 和 Segment 當成完全相同的東西。

### 4.3 動態連結

動態連結程式可能依賴共享函式庫，例如 libc。ELF 的動態資訊會指出載入器與相依性等資料。

常見概念：

- Shared Object：共享函式庫，Linux 常見副檔名為 .so。
- Dynamic Linker / Loader：負責載入共享函式庫並完成必要符號解析。
- PLT / GOT：常見於動態連結與函式呼叫解析機制。

不要對來源不明的二進位檔直接執行 ldd。某些環境與特殊 ELF 狀況下，檢查相依性可能帶來執行風險。對不可信檔案，優先使用 readelf -d 靜態檢查動態區段。

## 5. 建立自己的測試程式

建立 hello.c：

```c
#include <stdio.h>

int main(void) {
    puts("Hello from my lab.");
    return 0;
}
```

編譯並執行：

```bash
gcc -Wall -Wextra -O0 -g hello.c -o hello
./hello
```

檢查檔案：

```bash
file hello
readelf -h hello
readelf -l hello
readelf -S hello
readelf -d hello
```

重點觀察：

- file：檔案格式、架構與是否為動態連結等概要。
- readelf -h：ELF Header。
- readelf -l：Program Header 與載入段。
- readelf -S：Section Header。
- readelf -d：Dynamic Section。

查看反組譯：

```bash
objdump -d -Mintel hello | less
```

-Mintel 選擇 Intel 風格的 x86 組語輸出；若不是 x86 架構，該選項可能不適用。

## 6. 觀察程式啟動時的記憶體映射

執行測試程式時，可以在另一個終端查看它的映射：

```bash
cat /proc/PID/maps
pmap -x PID
```

將 PID 替換為實際程序識別碼。映射可能包括：

- 可執行檔的程式碼與資料。
- 動態載入的共享函式庫。
- Heap。
- Stack。
- 匿名映射與記憶體映射檔案。

位址會受到 ASLR 影響，因此每次啟動看到的位址可能不同。這是預期的安全設計，不代表程式載入錯誤。

## 7. Windows 的對應概念：PE

Windows 常見可執行檔格式為 PE（Portable Executable），常見副檔名包括 .exe、.dll、.sys。

概念上可對照：

| Linux ELF | Windows PE |
|---|---|
| ELF Header | DOS Header、PE Signature 與 COFF Header 等 |
| Program Header | Section Table 與載入相關結構 |
| Dynamic Linker | Windows Loader 與 DLL 載入機制 |
| Shared Object .so | Dynamic-link Library .dll |

這不是一對一欄位對應，只是幫助理解兩種平台都需要描述架構、區段、匯入／匯出與載入資訊。

## 8. 常見錯誤與排查

| 錯誤 | 常見原因 | 檢查方式 |
|---|---|---|
| gcc: command not found | 未安裝編譯器或 PATH 不正確 | command -v gcc、套件管理器 |
| undefined reference | 連結階段找不到符號定義或函式庫 | 檢查原始碼、目標檔與連結參數 |
| No such file or directory，但檔案明明存在 | 可能缺少 ELF 指定的動態載入器或相依環境 | file、readelf -l、檢查 interpreter |
| Exec format error | 架構或檔案格式不相容 | file、readelf -h、CPU 架構 |
| Permission denied | 檔案權限或掛載選項限制 | ls -l、findmnt、檔案系統掛載參數 |
| readelf 找不到 | 未安裝 binutils | command -v readelf、安裝 binutils |

不要為了讓未知程式執行而盲目使用 chmod 777 或停用安全控制。先判斷檔案來源、架構、權限與依賴關係。

## 9. 本章驗收題

1. 預處理、編譯、組譯、連結與載入的順序和工作分別是什麼？
2. Object File 與可執行檔有何差別？
3. Linker 和 Loader 分別在哪個階段工作？
4. ELF Section 與 Segment 有何差異？
5. readelf -l 與 readelf -S 分別查看什麼？
6. 為什麼不建議對不可信二進位檔直接執行 ldd？
7. ASLR 為什麼會讓同一程式每次啟動的位址不同？

## 10. 原始資料

- [System V ABI — ELF specification](https://refspecs.linuxfoundation.org/elf/elf.pdf)。
- [Linux execve(2) manual](https://man7.org/linux/man-pages/man2/execve.2.html)。
- [GNU Binutils documentation](https://sourceware.org/binutils/docs/)。
- [GCC documentation](https://gcc.gnu.org/onlinedocs/)。
- [Microsoft PE Format](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format)。
- [MIT xv6 book](https://mit-pdos.github.io/xv6-riscv-book/)。

本章以自建、無害的 Hello World 程式為實驗對象。不要執行來源不明的二進位檔。
