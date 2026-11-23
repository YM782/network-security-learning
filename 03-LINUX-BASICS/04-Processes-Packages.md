
# 03-04｜Linux 程序、套件和服務

## 本課目標
學會查看 Linux 正在執行的程序、檢查磁碟和記憶體，並理解 apt 套件管理工具的基本用途。

## 1. 查看程序
```bash
ps
ps aux
top
```
ps 顯示程序；ps aux 顯示較完整清單；top 即時觀察程序和資源，按 q 離開。
不要因為看到不認識的程序就直接結束它，先查明名稱、路徑和用途。

## 2. 查看磁碟和記憶體
```bash
df -h
free -h
```
df -h 查看檔案系統空間；free -h 查看記憶體；-h 使用較容易閱讀的單位。

## 3. apt 是什麼？
APT 是 Debian 系統常用的套件管理工具。Kali 基於 Debian，因此使用 apt 管理套件。
```bash
sudo apt update
apt list --upgradable
```
apt update 更新套件來源索引；apt list --upgradable 列出可升級套件。update 不等於升級所有套件。升級前先閱讀官方文件並確認快照和磁碟空間。

## 4. 服務是什麼？
Service（服務）通常指在背景執行、提供某種功能的程序。Linux 常用 systemd 管理服務。
```bash
systemctl status NetworkManager
```
若服務名稱不存在或不同，不要立即判斷系統損壞；不同版本和安裝方式可能不同。

## 5. 本課驗收
1. ps 和 top 分別適合做什麼？
2. df -h 和 free -h 有何不同？
3. apt update 會直接升級所有軟體嗎？
4. 服務和一般前景命令有什麼不同？
5. 為什麼不應隨意結束陌生程序？
