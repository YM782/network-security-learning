
# 06 — 實驗手冊與故障排查（Labs and Troubleshooting）

> 本章將前五章的知識變成可重現的操作。請在自己的 VMware Kali Linux 虛擬機中完成。命令輸出因 CPU、核心版本、虛擬機配置與套件版本而異；請保存自己的實際輸出，不要把範例輸出當成必須完全一致的標準答案。

## 0. 實驗規則與準備

- 只在自己的電腦與隔離虛擬機中操作。
- 不執行來源不明的二進位檔或腳本。
- 不將私人資料、密碼、Token、VPN 訂閱連結或真實資產資訊提交到 GitHub。
- 每個實驗先建立目錄與紀錄檔。
- 遇到錯誤先保存完整訊息。

建立實驗目錄：

```bash
mkdir -p ~/security-labs/computer-fundamentals
cd ~/security-labs/computer-fundamentals
date -Is | tee session-start.txt
uname -a | tee system-info.txt
```

## Lab 1 — 系統與 CPU 盤點

### 目的

確認虛擬機看到的 CPU、核心、邏輯處理器、記憶體與核心版本。

### 操作

```bash
cat /etc/os-release
uname -a
lscpu
free -h
getconf PAGESIZE
```

### 紀錄

| 項目 | 你的實際結果 |
|---|---|
| Linux 發行版與版本 | 自行填寫 |
| Kernel Release | 自行填寫 |
| Architecture | 自行填寫 |
| CPU(s) | 自行填寫 |
| Core(s) per socket | 自行填寫 |
| Thread(s) per core | 自行填寫 |
| Total RAM | 自行填寫 |
| Page Size | 自行填寫 |

### 問題

1. 虛擬機看到的 CPU 數量是否等於宿主機實體核心數？
2. 為什麼 CPU(s) 和 Core(s) 可能不同？
3. 頁面大小是多少？它是否等於 Cache Line 大小？

**驗收：** 能解釋每個欄位，並說明 VMware 虛擬 CPU 與宿主機 CPU 的關係。

## Lab 2 — 觀察程序、父子關係與執行緒

### 目的

建立一個自己控制的測試程序，觀察 PID、PPID、狀態與執行緒。

終端 A：

```bash
sleep 300
```

保持它執行。另開終端 B：

```bash
pgrep -a sleep
ps -eo pid,ppid,stat,comm | grep sleep
pstree -ap
```

將查到的 PID 代入：

```bash
cat /proc/PID/status
ps -L -p PID
top -H -p PID
```

完成後回到終端 A，按 Ctrl+C 結束測試程序。

### 問題

1. sleep 的 PID 是多少？PPID 是多少？
2. STAT 顯示什麼？為什麼它不是持續佔用 CPU？
3. /proc/PID/status 中 Threads 是多少？
4. ps -L 顯示的是程序還是執行緒清單？

**驗收：** 能指出 PID、PPID、狀態和執行緒數，並說明等待中的程序為何不必持續消耗 CPU。

## Lab 3 — 觀察程序的虛擬記憶體

### 目的

建立一個無害 C 程式，觀察全域變數、Stack、Heap 與程序映射。

建立 memory_demo.c：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int global_init = 7;
int global_zero;

int main(void) {
    int local_value = 42;
    int *heap_value = malloc(sizeof(*heap_value));

    if (heap_value == NULL) {
        perror("malloc");
        return 1;
    }

    *heap_value = 99;

    printf("PID: %ld\n", (long)getpid());
    printf("global_init: %p\n", (void *)&global_init);
    printf("global_zero: %p\n", (void *)&global_zero);
    printf("local_value: %p\n", (void *)&local_value);
    printf("heap_value: %p\n", (void *)heap_value);
    printf("Press Enter to exit...\n");
    fflush(stdout);

    (void)getchar();
    free(heap_value);
    return 0;
}
```

編譯與執行：

```bash
gcc -Wall -Wextra -O0 -g memory_demo.c -o memory_demo
./memory_demo
```

記下程式印出的 PID。保持程式等待時，在另一個終端執行：

```bash
cat /proc/PID/maps
pmap -x PID
cat /proc/PID/status
```

將 PID 換成實際數字。完成觀察後回到程式終端按 Enter。

### 問題

1. global_init 與 global_zero 分別屬於哪類資料？
2. local_value 和 heap_value 指向的資料通常位於哪類記憶體區域？
3. 程式印出的指標是虛擬位址還是實體位址？
4. 為什麼每次執行時位址可能不同？
5. /proc/PID/maps 中 r-xp、rw-p 等權限欄位代表什麼？

**驗收：** 能將程式變數與記憶體映射對應起來，並說明 ASLR 對位址的影響。不要只憑位址大小猜測區段；要結合映射路徑與權限判讀。

## Lab 4 — 觀察系統呼叫

### 目的

觀察程式如何透過系統呼叫讀取檔案。

確認 strace 已安裝：

```bash
command -v strace
```

執行：

```bash
strace -e trace=openat,read,write,close -o cat.trace cat /etc/hostname
less cat.trace
```

### 問題

1. 哪個系統呼叫嘗試開啟 /etc/hostname？
2. 成功開啟時回傳值代表什麼？
3. read 的回傳值代表什麼？
4. write 寫入的目的地可能是什麼？
5. 為什麼追蹤檔中的系統呼叫數量可能因環境而不同？

**驗收：** 至少找出一個 openat、read、write 或 close 呼叫，逐欄解釋參數和回傳值。

## Lab 5 — 編譯、檢查與反組譯

### 目的

觀察同一份 C 程式在不同建置階段的輸出。

建立 hello.c：

```c
#include <stdio.h>

int main(void) {
    puts("Hello from my lab.");
    return 0;
}
```

逐步建置：

```bash
gcc -E hello.c -o hello.i
gcc -S hello.c -o hello.s
gcc -c hello.c -o hello.o
gcc hello.o -o hello
```

檢查：

```bash
file hello.i hello.s hello.o hello
readelf -h hello
readelf -l hello
readelf -S hello
objdump -d -Mintel hello | less
```

### 問題

1. hello.o 為什麼通常不能直接當成完整程式執行？
2. readelf -h 顯示的 Machine 是什麼？
3. readelf -l 中有哪些 LOAD 段？
4. objdump 中如何找到 main？
5. -O0 與較高最佳化等級可能如何影響反組譯結果？

**驗收：** 能說明每個建置產物的用途，並從 ELF Header 和反組譯中找到至少兩項有意義的資訊。

## Lab 6 — 綜合故障排查演練

### 情境 A：找不到 gcc

依序檢查：

```bash
command -v gcc
gcc --version
cat /etc/os-release
```

若未安裝，確認套件來源與網路正常後，使用發行版官方套件管理器安裝 build-essential。不要從不明網站下載編譯器安裝檔。

### 情境 B：程序記憶體很高

```bash
ps -eo pid,ppid,stat,rss,vsz,comm --sort=-rss | head
free -h
vmstat 1 5
```

選擇自己有權限觀察的 PID：

```bash
pmap -x PID
cat /proc/PID/status
```

先分辨 RSS、VSZ、共享映射與實際記憶體壓力，不要看到 VSZ 很大就直接判定記憶體洩漏。

### 情境 C：程式出現 Segmentation Fault

若是自己的測試程式，可用 GDB：

```bash
gcc -g -O0 -Wall -Wextra test.c -o test
gdb ./test
```

在 GDB 中：

```text
run
bt
```

bt 可查看呼叫堆疊。也可在編譯器支援時使用 AddressSanitizer：

```bash
gcc -g -O1 -fsanitize=address -fno-omit-frame-pointer test.c -o test-asan
./test-asan
```

只對自己撰寫或可信任的測試程式使用這些除錯工具。

### 情境 D：strace 顯示 Permission denied

1. 確認追蹤的是自己啟動的程序。
2. 確認沒有容器或安全政策限制 ptrace。
3. 保存完整錯誤訊息。
4. 不要為了排錯而關閉 AppArmor、SELinux 或其他安全機制。

## 實驗紀錄模板

每次實驗都要保留：

| 欄位 | 內容 |
|---|---|
| 日期與時間 | 使用本機時間並註明時區 |
| 系統環境 | 發行版、版本、Kernel、CPU、VMware 配置 |
| 實驗目的 | 本次要驗證的概念 |
| 操作步驟 | 依順序記錄命令 |
| 預期結果 | 根據教材或官方文件提出 |
| 實際結果 | 貼上自己的輸出或截圖 |
| 差異分析 | 哪些地方與預期不同、可能原因 |
| 結論 | 用自己的話解釋實驗證明了什麼 |
| 來源 | 原始文件、手冊或專案連結 |

## 本模組最終驗收

- [ ] Lab 1：完成系統與 CPU 盤點。
- [ ] Lab 2：完成程序與執行緒觀察。
- [ ] Lab 3：完成虛擬記憶體映射分析。
- [ ] Lab 4：完成系統呼叫追蹤。
- [ ] Lab 5：完成編譯、ELF 檢查與反組譯。
- [ ] Lab 6：完成至少兩個故障情境分析。
- [ ] 每個實驗都有自己的命令、輸出、解釋與結論。

## 參考文件

- [OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/)。
- [MIT xv6 book and labs](https://mit-pdos.github.io/xv6-riscv-book/)。
- [Linux man-pages](https://man7.org/linux/man-pages/)。
- [GNU Binutils](https://sourceware.org/binutils/docs/)。
- [GDB documentation](https://sourceware.org/gdb/documentation/)。
- [AddressSanitizer documentation](https://clang.llvm.org/docs/AddressSanitizer.html)。

本實驗手冊只針對自有裝置與隔離的測試環境。若輸出與教材不同，請記錄版本和環境，不要為了追求相同輸出而修改不理解的系統設定。
