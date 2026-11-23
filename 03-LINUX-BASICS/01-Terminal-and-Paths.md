# 03-01｜Linux 終端機、Shell 和檔案路徑

> 難度：初級｜建議時間：60–90 分鐘｜先修：完成 Kali 第一次開機檢查
>
> 本課從最常用的 Linux 命令開始。目標不是背命令，而是理解命令在哪裡執行、Shell 如何解讀文字，以及路徑如何定位檔案。

## 一、本課學習成果

完成後，你應該能夠：
1. 分辨 Terminal（終端機）和 Shell（命令解譯器）。
2. 說明命令、參數和選項的基本結構。
3. 使用 pwd、ls、cd 查看和切換目錄。
4. 分辨絕對路徑和相對路徑。
5. 理解空格、引號和大小寫為什麼重要。
6. 遇到常見命令錯誤時，知道先檢查什麼。

## 二、Terminal 和 Shell 是什麼？

### 1. Terminal（終端機）

終端機是一個文字輸入和輸出介面。你在終端機中輸入命令，看到 Shell 回傳的結果。

它本身不是 Linux 核心，也不是命令解譯器。

### 2. Shell（命令解譯器）

Shell 接收你輸入的命令，解析參數，再啟動對應程式或執行內建命令。

常見 Shell：
- Bash：許多 Linux 系統預設使用的 Shell。
- Zsh：另一種常見互動式 Shell。
- Fish：強調互動體驗的 Shell。

你可以輸入：

    echo "$SHELL"

查看目前使用者預設 Shell 的相關路徑。注意，這不一定能代表目前終端機程序實際使用的 Shell，因為啟動方式可能不同。

### 3. 命令的基本結構

    command [options] [arguments]

- command：命令名稱。
- options：選項，用來改變命令行為。
- arguments：命令要處理的對象。

例如：

    ls -l /home

ls 是命令；-l 是選項；/home 是參數。

不同命令的參數規則不同，不能假設所有命令都使用相同語法。

## 三、先熟悉四條基本命令

### 1. whoami

    whoami

顯示目前有效使用者名稱。它能幫你確認目前是一般帳號還是其他帳號。

### 2. pwd

    pwd

Print Working Directory，顯示目前工作目錄。Shell 執行相對路徑時，通常會以目前工作目錄作為起點。

### 3. hostname

    hostname

顯示目前主機名稱。它不是 IP 位址，也不保證在網路上唯一。

### 4. date

    date

顯示系統日期和時間。若系統時間錯誤，日誌時間、憑證驗證和排查事件都可能受到影響。

## 四、Linux 路徑系統

Linux 使用目錄樹結構。最上層稱為根目錄，以斜線 / 表示。

常見目錄：

| 路徑 | 常見用途 |
|---|---|
| / | 根目錄，整個檔案系統樹的起點 |
| /home | 一般使用者的家目錄通常位於此處 |
| /root | root 管理員帳號的家目錄 |
| /etc | 系統和服務設定檔 |
| /var | 經常變動的資料，例如日誌和快取 |
| /tmp | 暫存資料 |
| /usr | 大量使用者空間程式和資源 |
| /bin | 基本命令程式（在現代系統中可能與 /usr/bin 合併） |
| /dev | 設備檔案介面 |
| /proc | 核心和程序資訊的虛擬檔案系統 |

不要因為看到目錄名稱就隨意刪除內容，尤其 /etc、/usr、/var 和 /dev。

## 五、絕對路徑和相對路徑

### 1. 絕對路徑 Absolute Path

從根目錄 / 開始的完整路徑，例如：

    /home/kali/Documents

無論目前在哪個目錄，這個路徑都指向相同位置（前提是檔案系統結構未改變）。

### 2. 相對路徑 Relative Path

從目前工作目錄開始的路徑。例如目前位於 /home/kali：

    Documents
    ../Downloads

第一個指向目前目錄下的 Documents；第二個表示先回到上一層，再進入 Downloads。

### 3. 四個特殊符號

- /：根目錄。
- ~：目前使用者的家目錄。
- .：目前目錄。
- ..：上一層目錄。

例如：

    cd ~

會切換到目前使用者的家目錄。

## 六、查看目錄內容：ls

### 基本用法

    ls

列出目前目錄中的項目。

### 常見選項

    ls -l
    ls -a
    ls -la
    ls -lh

- -l：長格式，顯示權限、擁有者、大小和時間等資訊。
- -a：包含以 . 開頭的隱藏項目。
- -la：同時使用長格式和顯示隱藏項目。
- -h：在長格式中以較容易閱讀的單位顯示大小。

注意：Linux 的檔案名稱區分大小寫。File.txt 和 file.txt 可能是兩個不同檔案。

## 七、切換目錄：cd

    cd /home
    pwd
    cd ..
    pwd
    cd ~
    pwd

逐條理解：
1. cd /home：切換到 /home。
2. pwd：確認目前位置。
3. cd ..：回到上一層。
4. 再用 pwd 確認。
5. cd ~：回到目前使用者家目錄。

如果路徑含有空格，需要使用引號：

    cd "My Documents"

也可以使用反斜線跳脫空格：

    cd My\ Documents

初學時建議使用引號，較容易閱讀。

## 八、Tab 自動補全和命令歷史

- Tab：補全檔案或命令名稱；如果有多個相符項目，可能需要再按一次 Tab。
- 上方向鍵：查看先前輸入的命令。
- Ctrl + C：中斷目前前景命令（不是關閉整個 Linux）。
- Ctrl + L：清除終端機畫面顯示，並不會刪除命令歷史或檔案。
- clear：清除畫面顯示，不會停止背景程序。

## 九、動手實驗：建立自己的練習目錄

依次輸入：

    whoami
    pwd
    mkdir -p ~/cyber-practice
    cd ~/cyber-practice
    pwd
    mkdir linux-basics
    cd linux-basics
    pwd
    ls -la

命令解釋：
- mkdir：建立目錄。
- -p：必要時建立不存在的上層目錄；目錄已存在時也不會因而報錯。
- ~/cyber-practice：在自己的家目錄下建立練習資料夾。
- ls -la：檢查目錄內容，包括隱藏項目。

## 十、常見錯誤排查

**錯誤：No such file or directory**

可能原因：
- 路徑拼寫錯誤。
- 大小寫不一致。
- 目錄不存在。
- 目前目錄不是你以為的位置。

排查：先執行 pwd，再執行 ls，逐層確認目錄。

**錯誤：Permission denied**

代表目前使用者沒有足夠權限完成操作。先確認檔案擁有者和權限，不要第一時間加 sudo。

**錯誤：command not found**

可能是命令拼錯、工具未安裝，或程式路徑不在 PATH 中。先檢查拼寫，再用 man 或官方文件查命令。

## 十一、本課測驗

1. Terminal 和 Shell 有什麼不同？
2. pwd 和 ls 分別做什麼？
3. /、~、. 和 .. 有什麼差別？
4. 絕對路徑和相對路徑有什麼差別？
5. 為什麼 Linux 檔名要注意大小寫？
6. Ctrl + L 和 Ctrl + C 有什麼不同？
7. Permission denied 時，為什麼不應立即加 sudo？

## 十二、參考資料

- [Linux man-pages：bash](https://man7.org/linux/man-pages/man1/bash.1.html)
- [Linux man-pages：pwd](https://man7.org/linux/man-pages/man1/pwd.1.html)
- [Linux man-pages：ls](https://man7.org/linux/man-pages/man1/ls.1.html)
- [Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)

**完成標準：** 能在自己的家目錄建立練習資料夾，並能不看教材使用 pwd、ls 和 cd 在目錄間移動。