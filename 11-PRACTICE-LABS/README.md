
# 11 — 初級實驗清單

所有實驗只在自己的 Windows、Kali 虛擬機或明確授權的練習平台進行。先完成前置課程，不要一次做完全部實驗。

## Lab 01 — 找到電腦規格

**前置課程：** 01 認識電腦

Windows：

```powershell
winver
systeminfo
```

**任務：** 記下 Windows 版本、CPU、記憶體容量。

**驗收：** 能指出 CPU、記憶體和 Windows 版本資訊。

## Lab 02 — 認識 Kali

**前置課程：** 02 Windows、VMware 與 Kali

Kali：

```bash
whoami
pwd
uname -a
cat /etc/os-release
```

**任務：** 記下使用者名稱、目前目錄、核心和發行版資訊。

**驗收：** 能說明每條命令查看什麼。

## Lab 03 — 建立和管理檔案

**前置課程：** 03 Linux 入門

```bash
mkdir -p ~/practice/basic
cd ~/practice/basic
echo "my first lab" > note.txt
cat note.txt
cp note.txt note-copy.txt
ls -l
```

**任務：** 建立資料夾、檔案、複製檔案並查看結果。

**驗收：** 能說明每個命令和檔案位置。

## Lab 04 — 查看網路設定

**前置課程：** 04 網路基礎

Windows：

```powershell
ipconfig /all
nslookup example.com
```

Kali：

```bash
ip addr
ip route
nslookup example.com
```

**任務：** 找到自己的 IP、閘道器和 DNS 查詢結果。

**驗收：** 能區分 IP、閘道器和 DNS。

## Lab 05 — 觀察本機網路

**前置課程：** 04 網路基礎

```bash
ping -c 4 127.0.0.1
ss -tuln
```

**任務：** 觀察本機回送測試和正在監聽的連接埠。

**驗收：** 能說明 127.0.0.1 是什麼，以及監聽連接埠代表什麼。

## Lab 06 — 第一次封包觀察

**前置課程：** 04 網路基礎

1. 開啟 Wireshark。
2. 選擇自己使用的網路介面。
3. 開始擷取。
4. 在自己的電腦執行一次 DNS 查詢。
5. 停止擷取。
6. 找出相關 DNS 封包並記錄查詢名稱和回應。

**驗收：** 能指出封包的時間、協定和查詢內容。不要擷取他人私人流量。

## Lab 07 — 本機連接埠觀察

**前置課程：** 04 網路基礎

```bash
ss -tuln
nmap 127.0.0.1
```

**任務：** 只觀察自己的本機服務。

**驗收：** 能解釋監聽連接埠和掃描結果。不要把目標改成陌生 IP 或網段。

## Lab 08 — 網站請求觀察

**前置課程：** 05 程式入門、08 Web 安全

1. 開啟瀏覽器開發者工具。
2. 選擇 Network 分頁。
3. 重新載入自己正常訪問的網站。
4. 找一筆請求，觀察 Method、Status、Headers。
5. 不要分享 Cookie、Authorization 標頭或個人資料。

**驗收：** 能指出請求方法和回應狀態碼。

## Lab 09 — PortSwigger 初級實驗

**前置課程：** 08 Web 安全

進入 [PortSwigger Web Security Academy](https://portswigger.net/web-security/)，先閱讀入門文章，再選擇初級標示的實驗。按照官方實驗環境完成，不要把測試方法用在其他網站。

**驗收：** 寫出漏洞成因、實驗證據和修復方式。

## Lab 10 — 完成一份安全報告

**前置課程：** 09 滲透測試入門

使用 [實驗紀錄模板](../templates/LAB-REPORT-TEMPLATE.md)，寫出：

- 實驗目標。
- 授權範圍。
- 操作步驟。
- 觀察結果。
- 證據。
- 問題或限制。
- 修復建議。

## 實驗進度

- [ ] Lab 01 電腦規格
- [ ] Lab 02 Kali 系統資訊
- [ ] Lab 03 檔案管理
- [ ] Lab 04 網路設定
- [ ] Lab 05 本機網路
- [ ] Lab 06 封包觀察
- [ ] Lab 07 本機連接埠
- [ ] Lab 08 網站請求
- [ ] Lab 09 初級 Web 實驗
- [ ] Lab 10 安全報告
