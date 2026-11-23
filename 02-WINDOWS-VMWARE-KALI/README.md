# 02 — Windows、VMware 與 Kali：虛擬化、服務與安全邊界

> 本模組教你理解並管理自己的 Windows + VMware + Kali 學習環境。先看懂資源分配和網路隔離，再研究 Windows 服務、執行帳戶和權限邊界。

## 課程順序

1. [01 Windows、VMware 和 Kali：宿主機與虛擬機完整入門](01-Host-Guest-and-VMware.md)
2. [02 VMware 網路模式：NAT、Host-only、Bridged 詳細解析](02-VMware-Network-Modes.md)
3. [03 Kali Linux 第一次開機：系統檢查、網路確認和安全更新](03-Kali-First-Checklist.md)
4. [04 真實案例：Windows PrintNightmare（CVE-2021-34527）](04-Case-Study-PrintNightmare.md)

## 本模組完成標準

- [ ] 能解釋 Host、Guest 和 Hypervisor。
- [ ] 能查看虛擬 CPU、RAM、硬碟和網卡設定。
- [ ] 能分辨快照、複製和備份。
- [ ] 能說明 NAT、Host-only 和 Bridged 的用途及風險。
- [ ] 能在 Kali 查看系統版本、核心、使用者、CPU、記憶體和磁碟。
- [ ] 能用 ip addr、ip route 和 DNS 查詢逐步排查網路問題。
- [ ] 能區分 apt update 和 apt upgrade。
- [ ] 能安全關閉 Kali 虛擬機。
- [ ] 能查看 Windows 服務狀態、執行帳戶和更新紀錄。
- [ ] 能解釋低權限輸入進入高權限服務為什麼是重要的研究方向。

## 漏洞研究視角

研究 Windows 漏洞時，不能只看某個服務是否開啟。還要問：
- 服務由哪個帳戶執行？
- 它接受什麼輸入？
- 請求從哪個介面進入？
- 一般使用者能否影響高權限操作？
- 修補和安全政策是否都已正確套用？

## 安全原則

一般 Kali 學習和更新通常先使用 NAT；本機隔離靶場可以考慮 Host-only，並確認 Windows 沒有啟用額外轉發；故意有漏洞的靶場不要直接橋接到家庭、公司或公共網路。重要實驗資料另行備份，不要只依賴 VMware 快照。

## 參考資料

- [VMware 官方文件](https://docs.vmware.com/)
- [Kali Linux 官方文件](https://www.kali.org/docs/)
- [Debian 官方文件](https://www.debian.org/doc/)
- [Microsoft MSRC 安全公告](https://msrc.microsoft.com/)
