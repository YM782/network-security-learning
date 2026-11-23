# 03 — Linux 命令列入門：從零開始管理系統

> 本模組以 Kali Linux 為主要練習環境。所有命令先在自己的練習目錄中操作，理解每個參數後再擴展到其他目錄。

## 課程順序

1. [01 Linux 終端機、Shell 和檔案路徑](01-Terminal-and-Paths.md)
2. [02 Linux 檔案操作：建立、讀取、搜尋、複製和刪除](02-File-Operations.md)
3. [03 Linux 使用者、群組、檔案權限和 sudo](03-Users-Permissions.md)
4. [04 Linux 程序、套件管理、服務和系統資源](04-Processes-Packages.md)

## 本模組完成標準

- [ ] 能分辨 Terminal 和 Shell。
- [ ] 能使用 pwd、ls、cd 和絕對／相對路徑。
- [ ] 能在自己的目錄建立、讀取、搜尋、複製和移動檔案。
- [ ] 理解 >、>>、管線和標準錯誤輸出。
- [ ] 能看懂 rwx 權限和常見 chmod 數字。
- [ ] 能解釋 sudo、root 和最小權限原則。
- [ ] 能使用 ps、top、free -h、df -h 查看系統。
- [ ] 能分辨 apt update、apt upgrade 和 apt install。
- [ ] 能初步查看 systemd 服務和日誌。

## 練習安全規則

所有建立、修改和刪除檔案的練習，都先放在 ~/cyber-practice/ 之下。不要把練習命令直接套用到 /etc、/usr、/var 等系統目錄；不要因為 Permission denied 就習慣性加 sudo；不要在不理解時使用遞迴強制刪除。

## 建議延伸練習

完成本模組後，再開始 [OverTheWire Bandit](https://overthewire.org/wargames/bandit/)，把命令列知識用闖關方式練熟。
