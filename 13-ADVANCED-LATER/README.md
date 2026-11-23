
# 13 — 進階資料庫：現在先收藏，不要急著學

本資料夾是完成初級路線後的延伸方向。這些專案品質很好，但不少內容假設讀者已經熟悉網路、Linux、Windows、程式和安全基礎。

## 1. 作業系統與計算機底層

- [OSTEP — Operating Systems: Three Easy Pieces](https://pages.cs.wisc.edu/~remzi/OSTEP/)：作業系統概念教材。等你已理解程序、記憶體和檔案後，再深入閱讀。
- [MIT xv6 RISC-V Book](https://mit-pdos.github.io/xv6-riscv-book/)：教學作業系統，適合未來研究核心、系統呼叫和記憶體。
- [Intel Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)：處理器指令集和系統程式設計規格，屬於專業參考手冊。

## 2. 更完整的電腦科學

- [OSSU Computer Science](https://github.com/ossu/computer-science)：完整自學電腦科學課程，內容很多，部分課程有數學和程式先修要求。請挑選需要的章節，不要現在就從頭開始整套攻讀。

## 3. Web 安全專業資料

- [OWASP WSTG](https://github.com/OWASP/wstg)：網站安全測試方法。
- [OWASP Cheat Sheet Series](https://github.com/OWASP/CheatSheetSeries)：安全開發和修補速查。
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)：測試案例和參考字串。只用於自己或明確授權的實驗環境。

## 4. 企業內網和 Active Directory

- [InternalAllTheThings](https://github.com/swisskyrepo/InternalAllTheThings)：Active Directory 和內網安全測試速查表。
- [GOAD](https://github.com/Orange-Cyberdefense/GOAD)：故意脆弱的 Active Directory 練習環境。

開始這一階段前，至少應熟悉 Windows 使用者、群組、網域、DNS、SMB、LDAP 和基本網路。GOAD 需要多台虛擬機，對電腦資源也有要求；請依官方文件建立完全隔離的環境，絕對不要暴露到公網。

## 5. 進階程式和漏洞研究

未來可以逐步學習：

- C 語言與指標。
- 記憶體配置和程式除錯。
- 組合語言與反組譯。
- ELF / PE 檔案格式。
- 安全編碼、記憶體安全和漏洞成因。
- 逆向工程與數位鑑識。

這些不是現在的入門門檻。等你能看懂一般 Linux 命令、網路封包和簡單 Python 程式，再進入會容易很多。

## 6. 綜合網路安全的長期方向

完整的安全能力不只包含滲透測試，也包含：

- 網路與端點防禦。
- Windows / Linux 系統管理。
- Web 與 API 安全。
- 企業內網與身分管理。
- 雲端與容器安全。
- 日誌分析、SIEM 和事件應變。
- 漏洞管理、風險評估和安全報告。

請先完成主路線，再依照自己的興趣和實際工作目標決定進階順序。
