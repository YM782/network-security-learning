# 03-04｜Linux 程序、套件管理、服務和系統資源

> 難度：初級至初中級｜建議時間：90–120 分鐘｜先修：03-01 至 03-03
>
> 本課把 Linux 的程序、服務和套件管理串起來。你將學會查看系統正在做什麼、哪些資源正在被使用，以及如何安全更新 Kali。

## 一、本課學習成果

完成後，你應該能夠：
1. 使用 ps 和 top 查看程序。
2. 理解 PID、程序狀態和 CPU／記憶體欄位。
3. 使用 df -h 和 free -h 查看磁碟及記憶體。
4. 認識 APT 套件管理工具。
5. 分辨 apt update、apt upgrade 和 apt install。
6. 初步查看 systemd 服務狀態。
7. 遇到 APT 鎖定、空間不足或更新失敗時，知道安全的排查順序。

## 二、什麼是程序？

程序（Process）是正在執行的程式實例。作業系統會為程序分配識別碼、記憶體空間和其他資源。

每個程序可能有多個執行緒。程序可能由使用者啟動，也可能由系統服務或其他程序建立。

## 三、查看程序：ps

### 1. 基本命令

    ps

顯示目前終端機工作階段相關的程序。

### 2. 完整程序清單

    ps aux

常見欄位：
- USER：程序所屬使用者。
- PID：程序識別碼。
- %CPU：CPU 使用比例的估算。
- %MEM：記憶體使用比例。
- STAT：程序狀態。
- COMMAND：啟動命令或程序名稱。

### 3. 只查看自己的程序

    ps -u "$USER"

這會列出目前使用者相關的程序。不同 ps 選項組合會影響顯示範圍和格式。

### 4. 搜尋特定程序

    ps aux | grep -i firefox

這個範例搜尋包含 firefox 的輸出行。grep 本身也可能出現在結果中，因為它也是正在執行的程序。更精確的程序查詢可依系統工具使用 pgrep 或其他方式。

## 四、即時查看：top

    top

top 會持續更新系統負載和程序清單。

常見資訊：
- 系統運行時間。
- 使用者數量。
- Load Average（平均負載）。
- CPU 使用狀態。
- 記憶體和 Swap。
- 程序清單。

按 q 離開 top。

Load Average 不等於 CPU 使用率。它反映一段時間內可執行或處於不可中斷等待狀態的工作平均數，解讀時要結合 CPU 核心數和系統狀態。

## 五、查看記憶體和磁碟

### 1. 記憶體

    free -h

查看 total、used、free、shared、buff/cache、available 和 Swap。

available 通常比單純 free 更適合初步估計還能提供多少記憶體給新工作。

### 2. 檔案系統空間

    df -h

查看各掛載檔案系統的容量和使用率。

### 3. 區塊裝置

    lsblk

查看磁碟、分割區和裝置關係。

如果 APT 更新失敗並顯示 No space left on device，先用 df -h 找出哪個掛載點空間不足，不要直接刪除系統目錄。

## 六、APT 套件管理工具

Kali 基於 Debian，使用 APT 管理軟體套件。

APT 可以處理：
- 從套件來源取得軟體索引。
- 安裝套件。
- 升級套件。
- 移除套件。
- 處理相依套件關係。

### 1. apt update：更新套件索引

    sudo apt update

它會讀取已設定的套件來源，下載最新的套件索引。這一步不會直接升級所有已安裝套件。

### 2. 查看可升級套件

    apt list --upgradable

列出目前有新版本可升級的套件。

### 3. apt upgrade：升級已安裝套件

    sudo apt upgrade

它會根據套件管理器的規則升級已安裝套件。過程可能下載大量資料，並需要較長時間。

執行前：
1. 確認網路連線正常。
2. 確認虛擬機有足夠磁碟空間。
3. 保存正在編輯的檔案。
4. 重要實驗資料另外備份。
5. 閱讀 APT 顯示的套件變更內容。

### 4. apt install：安裝套件

    sudo apt install tree

這是一個示例，會嘗試安裝名為 tree 的套件。執行前先確認套件名稱和來源。若不需要安裝，不必為了練習而真的安裝。

### 5. 查詢套件資訊

    apt show tree

顯示套件的描述、版本和其他資訊。若套件不存在或索引未更新，結果可能不同。

## 七、服務 Service 和 systemd

服務通常是背景運行、提供某種系統功能的程序。Kali 常見的服務管理方式是 systemd。

### 1. 查看服務狀態

    systemctl status NetworkManager

這個命令查看 NetworkManager 服務狀態。若服務不存在，先確認系統是否使用該服務，不要直接判定系統損壞。

### 2. 查看失敗服務

    systemctl --failed

列出被 systemd 標記為失敗的服務。出現失敗項目不代表整台系統一定無法使用，要閱讀服務名稱和錯誤原因。

### 3. 查看系統日誌

    journalctl -b -p warning

查看本次開機以來 warning 等級及更嚴重的日誌。日誌可能包含主機名稱、使用者名稱、IP 或其他資訊，分享前要檢查並遮蔽敏感內容。

## 八、動手實驗：觀察系統資源

逐條執行：

    whoami
    ps aux
    top
    free -h
    df -h
    lsblk
    systemctl --failed

請記錄：
- 目前使用者。
- CPU 使用率較高的程序。
- 記憶體 available。
- 根檔案系統 / 的使用率。
- 是否有 systemd failed 服務。

不要因為看到某個程序或服務不熟悉就結束它或刪除它。

## 九、常見錯誤與排查

### 問題 1：Could not get lock

可能有另一個 APT 程序正在工作。

排查：
1. 等待正在執行的安裝或更新完成。
2. 檢查是否開啟其他套件管理視窗或終端機。
3. 查看相關程序。
4. 不要直接刪除 /var/lib/dpkg/lock 或其他 lock 檔案。

### 問題 2：Temporary failure resolving

這通常表示 DNS 名稱解析失敗。先確認 ip addr、ip route 和 DNS 設定，再檢查 VMware 網路模式。

### 問題 3：No space left on device

執行：

    df -h

確認哪個檔案系統空間不足。不要在不清楚用途時刪除 /var/lib/dpkg、/usr 或其他系統目錄。

### 問題 4：dpkg was interrupted

可能是套件安裝或升級中途被中斷。先保存錯誤訊息，確認沒有其他 apt/dpkg 程序正在運行，再參照 Debian/Kali 官方文件修復套件狀態。不要同時啟動多個套件管理程序。

### 問題 5：服務顯示 failed

查看：

    systemctl status 服務名稱

再查看相關日誌：

    journalctl -u 服務名稱 -b

把「服務名稱」替換成實際服務名稱。不要照抄佔位文字。

## 十、本課安全原則

- 不要在不理解時使用高風險刪除命令。
- 不要直接刪除 APT lock 檔案。
- 不要在更新期間強制關閉虛擬機。
- 不要把陌生套件來源加入系統。
- 不要因為服務名稱陌生就判定它是惡意程式。
- 重要虛擬機資料要另外備份。

## 十一、本課測驗

1. ps 和 top 有什麼差別？
2. PID 的用途是什麼？
3. free -h 和 df -h 分別查看什麼？
4. apt update 和 apt upgrade 有什麼不同？
5. systemctl status 和 systemctl --failed 分別做什麼？
6. APT 顯示 Could not get lock 時，為什麼不能直接刪除 lock 檔？
7. 如何排查 No space left on device？
8. 為什麼不應在更新期間強制關閉 Kali？

## 十二、參考資料

- [Debian Reference：APT 套件管理](https://www.debian.org/doc/manuals/debian-reference/)
- [Kali Linux 官方文件](https://www.kali.org/docs/)
- [Linux man-pages：ps](https://man7.org/linux/man-pages/man1/ps.1.html)
- [systemd：systemctl 文件](https://www.freedesktop.org/software/systemd/man/latest/systemctl.html)
- [systemd：journalctl 文件](https://www.freedesktop.org/software/systemd/man/latest/journalctl.html)

**完成標準：** 能查看 Kali 程序、記憶體、磁碟和服務狀態，並能解釋 APT 的基本工作流程和常見更新錯誤的安全排查方法。