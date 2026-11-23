
# 02-03｜Kali 第一次開機檢查清單

## 本課目標
熟悉 Kali 桌面和終端機，能查看系統版本、使用者、網路介面、磁碟和記憶體。

## 1. 先認識桌面
找到 Applications（應用程式）、Terminal（終端機）、File Manager（檔案管理員）、Settings（設定）和 Network（網路）。

Kali 是基於 Debian 的 Linux 發行版，常用於安全測試、數位鑑識和安全研究。

## 2. 查看系統資訊
在 Terminal 依次輸入：
```bash
whoami
pwd
hostname
cat /etc/os-release
uname -r
```
- whoami：目前使用者。
- pwd：目前目錄。
- hostname：電腦名稱。
- cat /etc/os-release：發行版資訊。
- uname -r：核心版本。

## 3. 查看網路
```bash
ip addr
ip route
```
確認是否有網路介面、IP 和預設路由。介面名稱因系統而異。

## 4. 查看磁碟和記憶體
```bash
df -h
free -h
```
df -h 查看檔案系統空間；free -h 查看記憶體。-h 代表使用較容易閱讀的單位。

## 5. 更新套件索引
確認網路可用後：
```bash
sudo apt update
```
這一步更新套件來源索引，不等於立即升級所有已安裝軟體。若出現 DNS、連線或來源錯誤，先記錄完整訊息，不要隨意更換陌生套件來源。

## 6. 本課驗收
- [ ] 能開啟 Kali 終端機
- [ ] 能查看目前使用者
- [ ] 能查看 Kali 版本和核心版本
- [ ] 能查看 IP 和路由
- [ ] 能查看磁碟和記憶體
- [ ] 能解釋 apt update 的用途
