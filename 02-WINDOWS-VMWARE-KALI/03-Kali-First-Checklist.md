# 02-03｜Kali Linux 第一次開機：系統檢查、網路確認和安全更新

> 難度：初級｜建議時間：90 分鐘｜先修：02-01、02-02
>
> 本課不要求你立即安裝大量安全工具。先把 Kali 的版本、使用者、磁碟、記憶體和網路狀態查清楚，這樣以後出現問題時才有依據。

## 一、本課學習成果

完成後，你應該能夠：

1. 認識 Kali 桌面、終端機和檔案管理員。
2. 查看 Kali 版本、核心版本、主機名稱和目前使用者。
3. 查看 IP、路由、磁碟和記憶體。
4. 理解 apt update、apt upgrade 的差異。
5. 按照順序排查 Kali 無法上網或套件更新失敗的問題。
6. 知道如何安全關閉虛擬機。

## 二、Kali Linux 是什麼？

Kali Linux 是以 Debian 為基礎的 Linux 發行版，提供許多安全測試、數位鑑識和研究工具。

要注意：
- Kali 不是所有 Linux 知識的替代品。
- 安裝 Kali 不代表已經掌握網路安全。
- 工具的使用必須符合授權範圍。
- 不要因為 Kali 預裝某個工具，就在陌生設備上執行它。

## 三、先認識 Kali 桌面

初次進入桌面，先找到：

- Applications：應用程式選單。
- Terminal：終端機，用來輸入命令。
- File Manager：檔案管理員。
- Settings：系統設定。
- Network：網路狀態和連線設定。

本課以圖形桌面和 Terminal 交替操作。命令請逐條輸入，不要把多條未知命令一次貼上。

## 四、查看系統版本和基本資訊

打開 Terminal。

### 1. 查看目前使用者

    whoami

用途：顯示目前登入的使用者名稱。

### 2. 查看目前目錄

    pwd

用途：顯示目前工作目錄（Working Directory）。

### 3. 查看主機名稱

    hostname

用途：顯示目前系統的主機名稱。

### 4. 查看 Kali 發行版資訊

    cat /etc/os-release

用途：顯示作業系統名稱、版本和識別資訊。這比單憑桌面外觀判斷版本可靠。

### 5. 查看 Linux 核心版本

    uname -r

用途：顯示目前正在運行的核心版本。核心版本和 Kali 發行版版本不是同一個概念。

## 五、查看 CPU、記憶體和磁碟

### 1. CPU

    lscpu

查看 CPU 架構、處理器數量和核心資訊。虛擬機裡顯示的是 VMware 提供的虛擬 CPU 資訊。

### 2. RAM 和 Swap

    free -h

常見欄位：
- total：總量。
- used：使用量。
- free：完全未使用的部分。
- shared：共享記憶體。
- buff/cache：緩衝和快取。
- available：估算可供新程式使用的記憶體。
- Swap：交換空間使用狀態。

### 3. 區塊裝置

    lsblk

查看系統識別到的磁碟和分割區裝置。

### 4. 檔案系統空間

    df -h

查看已掛載檔案系統的總容量、已使用和可用空間。

lsblk 和 df -h 不一樣：前者看裝置和分割區結構，後者看已掛載檔案系統的容量使用情況。

## 六、查看網路狀態

### 1. 查看網路介面和 IP

    ip addr

查看網卡名稱、狀態、MAC 位址和 IP 位址。介面名稱可能是 eth0、ens33 或其他名稱。

### 2. 查看路由表

    ip route

查看預設路由和其他網段的路由。若沒有 default 路由，可能無法正常連接外部網路。

### 3. 查看 DNS 設定

    cat /etc/resolv.conf

查看系統目前使用的 DNS 設定。這個檔案可能由網路管理服務自動管理，因此先查看，不要直接編輯。

### 4. 查看網路管理服務

    systemctl status NetworkManager

如果 Kali 使用 NetworkManager，這條命令可以查看其狀態。若顯示服務不存在，先確認系統使用的網路管理方式，不要直接判斷系統損壞。

## 七、第一次網路連線檢查流程

當 Kali 無法打開網站時，依照以下順序排查：

1. VMware Network Adapter 是否連接？
2. Kali 是否看得到網卡？
3. 網卡是否取得 IP？
4. 是否有預設路由？
5. DNS 是否能解析網域？
6. 目標網站或套件來源是否可用？

### 測試 DNS

    getent hosts kali.org

如果回傳 IP，代表系統成功取得名稱解析結果。沒有回傳時，可能是 DNS 或網路問題。

### 可選：測試 ICMP

    ping -c 3 1.1.1.1

如果沒有回應，不一定代表網路中斷。部分網路會封鎖 ICMP，因此必須結合其他測試。

## 八、Kali 套件管理：apt update 和 apt upgrade

Kali 使用 APT 管理套件。

### 1. apt update

    sudo apt update

用途：從已設定的軟體來源更新套件索引，讓系統知道有哪些版本可以安裝或升級。

這條命令本身不會把所有已安裝軟體直接升級到新版本。

### 2. 查看可升級套件

    apt list --upgradable

用途：列出目前可升級的套件。

### 3. apt upgrade

    sudo apt upgrade

用途：依套件管理器規則升級已安裝套件。它可能下載和安裝大量資料，也可能需要較長時間。

執行前應確認：
- 網路連線正常。
- 虛擬機有足夠磁碟空間。
- 有重要資料的話先備份。
- 已建立快照，但不要把快照當成唯一備份。
- 閱讀套件管理器顯示的變更內容。

初學者不要在不了解差異時隨意執行完整系統升級或刪除大量套件。

## 九、常見錯誤和排查方法

### 問題 1：Temporary failure resolving

通常代表名稱解析失敗，但原因可能是 DNS 設定、網路路由或上游連線問題。

排查順序：
1. ip addr：確認是否取得 IP。
2. ip route：確認是否有預設路由。
3. cat /etc/resolv.conf：查看 DNS 設定。
4. getent hosts kali.org：測試解析。
5. 回到 VMware 查看 NAT 或其他網路模式。

不要一看到錯誤就隨意更換套件來源或貼上陌生 DNS 設定。

### 問題 2：Could not get lock

通常代表另一個套件管理程序正在使用 APT，或上次操作沒有正常結束。

排查：
1. 等待正在執行的更新或安裝完成。
2. 確認是否有其他終端機正在執行 apt。
3. 查看程序狀態。
4. 不要直接刪除 lock 檔案，否則可能破壞套件管理狀態。

### 問題 3：No space left on device

代表目標檔案系統沒有足夠空間，不一定是整顆磁碟都滿了。

檢查：

    df -h

再用：

    lsblk

確認分割區和掛載點。不要在不了解目錄用途時大量刪除 /var、/usr 或系統檔案。

### 問題 4：套件來源無法連線

先確認 Kali 網路本身是否正常，再查看套件來源設定。只使用 Kali 官方文件推薦的來源，不要把不明網站提供的來源加入系統。

## 十、如何正常關閉 Kali？

在 Kali 桌面選單選擇 Log Out / Shut Down / Power Off（名稱依桌面環境而異），等待虛擬機完全關閉。

也可以在終端機使用：

    sudo poweroff

這會要求系統關機。執行前先保存文件並退出正在執行的工作。

不要在 Kali 正在更新、寫入檔案或安裝套件時，直接從 VMware 強制關閉電源。

## 十一、第一次開機檢查表

- [ ] 能開啟 Kali 桌面和 Terminal。
- [ ] whoami 顯示目前使用者。
- [ ] cat /etc/os-release 顯示發行版資訊。
- [ ] uname -r 顯示核心版本。
- [ ] lscpu 顯示 CPU 資訊。
- [ ] free -h 顯示記憶體。
- [ ] df -h 顯示磁碟空間。
- [ ] ip addr 顯示網路介面。
- [ ] ip route 顯示路由。
- [ ] getent hosts kali.org 能測試名稱解析。
- [ ] 理解 apt update 和 apt upgrade 的差別。

## 十二、本課測驗

1. Kali Linux 基於哪個 Linux 發行版？
2. cat /etc/os-release 和 uname -r 有什麼不同？
3. lscpu、free -h、lsblk、df -h 分別查看什麼？
4. ip addr 和 ip route 有什麼差別？
5. apt update 是否等於升級所有套件？
6. Temporary failure resolving 可能是什麼問題？
7. 為什麼不應直接刪除 APT lock 檔案？
8. 為什麼系統更新期間不應強制關閉虛擬機？

## 十三、參考資料

- [Kali Linux 官方文件](https://www.kali.org/docs/)
- [Kali：常見問題](https://www.kali.org/docs/)
- [Debian Reference](https://www.debian.org/doc/manuals/debian-reference/)
- [Linux man-pages](https://man7.org/linux/man-pages/)

**完成標準：** 能獨立完成第一次開機檢查表，並在 Kali 無法連網或更新失敗時，按照網卡、IP、路由、DNS、套件來源的順序排查。