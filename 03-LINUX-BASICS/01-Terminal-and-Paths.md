
# 03-01｜終端機、Shell 和 Linux 路徑

## 本課目標
理解終端機是什麼，知道命令在哪裡輸入，並能在 Linux 目錄之間移動。

## 1. Terminal 和 Shell
- Terminal（終端機）：輸入命令、閱讀結果的視窗。
- Shell（命令解譯器）：接收命令、解析後請系統執行。
- Command（命令）：要求系統完成某項工作的文字。

終端機不是 Linux 本身，而是與 Shell 互動的一種方式。

## 2. 第一組命令
```bash
whoami
pwd
hostname
date
```
whoami 顯示目前使用者；pwd 顯示目前目錄；hostname 顯示電腦名稱；date 顯示系統時間。

## 3. 絕對路徑和相對路徑
絕對路徑（Absolute Path）從根目錄 / 開始，例如 /home/kali/Documents。
相對路徑（Relative Path）從目前目錄開始，例如 Documents、../Downloads。

- /：根目錄。
- ~：目前使用者的家目錄。
- .：目前目錄。
- ..：上一層目錄。

## 4. 查看和切換目錄
```bash
pwd
ls
ls -la
cd /home
pwd
cd ..
cd ~
pwd
```
ls 列出檔案；-l 顯示詳細資訊；-a 包含隱藏項目；cd 切換目錄。

## 5. 練習
1. 輸入 pwd，記下目前位置。
2. 輸入 ls，查看內容。
3. 輸入 cd /home，再輸入 pwd。
4. 輸入 cd ~，回到家目錄。
5. 輸入 ls -la，找出隱藏項目。

## 6. 常見錯誤
No such file or directory：路徑不存在或拼寫錯誤。Linux 區分大小寫。
Permission denied：目前使用者沒有足夠權限。先檢查路徑和權限，不要直接使用 sudo。

## 7. 本課驗收
能解釋 pwd、ls、cd、~、/、. 和 ..，並能從家目錄切換到指定資料夾。
