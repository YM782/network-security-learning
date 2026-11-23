# Security Labs（網路安全實驗室）

本頁記錄如何建立隔離、可還原的學習環境。靶場可能故意包含漏洞，絕對不要直接暴露到公開網際網路。

## 1. Recommended Lab Topology（建議實驗拓樸）

- Host（實體主機）：Windows 10／11，負責執行 VMware。
- Attacker VM（測試虛擬機）：Kali Linux。
- Target VM（靶場虛擬機）：只安裝教學靶場或測試服務。
- Network（網路）：優先使用 VMware Host-only，讓虛擬機彼此通訊但不直接接入外部區域網路。

初學階段可以將 WebGoat 或 Juice Shop 綁定在 Kali 本機的 127.0.0.1，先從單機實驗開始。

## 2. Before Starting（開始前檢查）

1. 更新並備份宿主機。
2. 為 Kali 建立 VMware Snapshot（快照）。
3. 確認虛擬網卡模式與 IP 網段。
4. 不使用真實帳號、真實個資或正式環境資料。
5. 不把靶場服務映射到路由器或公開 IP。
6. 記錄實驗目的與允許測試的範圍。

## 3. Docker Basics（Docker 基礎）

檢查 Docker：

```bash
docker --version
docker info
systemctl status docker
```

查看容器：

```bash
docker ps
docker ps -a
docker images
```

查看日誌：

```bash
docker logs --tail 50 <CONTAINER_NAME>
```

停止容器：

```bash
docker stop <CONTAINER_NAME>
```

## 4. OWASP WebGoat（OWASP WebGoat 教學靶場）

Original source（原始來源）：https://github.com/WebGoat/WebGoat

WebGoat 是故意包含弱點的教學應用程式，透過課程引導學習 Web 安全概念。

開始前請先查看官方 README，確認目前映像檔、埠號與啟動方式。若使用官方常見 Docker 映像，可參考以下本機綁定範例：

```bash
docker pull webgoat/webgoat

docker run --rm -it \
  -p 127.0.0.1:8080:8080 \
  -p 127.0.0.1:9090:9090 \
  webgoat/webgoat
```

若目前版本的映像或連接埠有變更，請以官方 README 為準。服務啟動後，依官方文件提供的本機網址開啟課程。

## 5. OWASP Juice Shop（OWASP Juice Shop 綜合靶場）

Original source（原始來源）：https://github.com/juice-shop/juice-shop

```bash
docker pull bkimminich/juice-shop

docker run --rm \
  -p 127.0.0.1:3000:3000 \
  bkimminich/juice-shop
```

在同一台主機的瀏覽器開啟：

`http://127.0.0.1:3000`

此設定將服務綁定到本機回環位址。不要將連接埠改成對外公開的綁定方式，除非你已建立隔離網路並清楚了解風險。

## 6. VMware Host-only（VMware 僅主機網路）

Host-only 網路通常讓宿主機與指定虛擬機互通，並將實驗網路與外部網路隔離。不同 VMware 版本的選單名稱可能不同：

1. 關閉虛擬機。
2. 開啟 VM Settings（虛擬機設定）。
3. 選擇 Network Adapter（網路介面卡）。
4. 選擇 Host-only。
5. 啟動虛擬機。
6. 在 Linux 執行 `ip -br addr`，確認取得實驗網段 IP。
7. 只在該實驗網段內進行測試。

若靶場需要下載更新，請先停止靶場服務，再以受控方式暫時使用 NAT 取得更新；完成後恢復隔離設定。

## 7. Active Directory Mini Lab（Active Directory 迷你實驗）

若要練習 Windows 企業網域，請使用獨立的 VMware Host-only 網路，建立專用 Windows Server DC 與 Windows Pro 用戶端。完整步驟請見 [Active Directory Lab Setup](../09-Active-Directory/02-Lab-Setup.md)。

- 不要將 DC 或故意存在弱點的 VM 設為 Bridged。
- 不要將實驗網域接入公司網路或公開 Wi-Fi。
- 不要在實驗網域使用真實帳號、密碼或資料。
- 需要更新時先關閉靶場服務，再使用受控方式處理；更新完成後恢復隔離。

## 8. Incident Response and SIEM Labs（事件應變與 SIEM 實驗）

- [Windows Incident Response Lab](../12-Incident-Response/02-Windows-Incident-Response-Lab.md)：在自有 Windows VM 中收集主機、程序、網路與事件日誌，建立時間線與 SHA-256 證據清單。
- [Wazuh SIEM Lab](../13-SIEM-and-Detection/02-Wazuh-Lab.md)：在隔離 Host-only 網路部署 Wazuh，收集 Windows Security 與 Sysmon 事件。

請勿將 SIEM 管理介面或測試端點暴露至公共網際網路；不要把密碼、Token、Agent Key 或真實事件資料提交至 Git。

## 9. Troubleshooting（故障排查）

| 問題 | 檢查方式 |
|---|---|
| Docker daemon 未啟動 | `systemctl status docker`、`journalctl -u docker` |
| 容器立即退出 | `docker ps -a`、`docker logs <name>` |
| Port already allocated（連接埠已被占用） | `ss -tulpn`，再選擇未使用的本機連接埠 |
| 瀏覽器無法開啟 | 確認容器狀態、埠號映射與 127.0.0.1 |
| Kali 與靶機無法互通 | 檢查 VMware 網卡模式、IP、子網路與防火牆 |
| 宿主機網路受到影響 | 停止實驗、檢查虛擬網卡設定，必要時還原快照 |

## Sources（資料來源）

- WebGoat: https://github.com/WebGoat/WebGoat
- OWASP Juice Shop: https://github.com/juice-shop/juice-shop
- Docker Docs: https://docs.docker.com/
- VMware Workstation Documentation: https://docs.vmware.com/

本頁只描述自有、隔離環境中的學習實驗。不要對未授權系統進行掃描或漏洞測試。
