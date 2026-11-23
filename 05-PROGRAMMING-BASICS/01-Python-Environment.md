# 05-01｜Python 環境、直譯器與第一個程式

> 難度：零基礎｜建議時間：60–90 分鐘｜先修：能使用 Windows PowerShell 或 Kali 終端機

## 一、本課學習成果

完成後，你應該能夠：
1. 說明 Python 語言、直譯器和 .py 原始碼檔案的關係。
2. 在 Windows 和 Kali 查看 Python 版本。
3. 建立並執行第一個 Python 程式。
4. 理解 print、註解、縮排和字串。
5. 知道 Python 虛擬環境的用途。
6. 排查常見的命令找不到、路徑錯誤和縮排錯誤。

## 二、Python 是什麼？

Python 是一種通用程式語言，常見用途包括：
- 自動化重複工作。
- 讀取和整理文字檔。
- 處理 JSON、CSV 等資料。
- 網頁和 API 開發。
- 網路管理與安全分析。
- 資料科學和人工智慧。

在安全工作中，Python 常用來整理掃描結果、解析日誌、處理測試資料和自動化流程。本課先學程式基礎，不會直接執行對外掃描或攻擊。

## 三、原始碼、直譯器和執行結果

你寫下的 Python 程式通常保存為 .py 檔案，這是原始碼（Source Code）。

Python 直譯器（Interpreter）負責讀取程式、執行程式語句，並在執行期間處理錯誤。

簡化流程：

    你編寫 hello.py
          |
          v
    Python 直譯器執行
          |
          v
    程式輸出或產生結果

Python 的實際執行過程還涉及編譯成位元碼和虛擬機執行等細節；現在先掌握「原始碼由 Python 執行環境執行」即可。

## 四、檢查 Windows 的 Python

打開 PowerShell，逐條輸入：

    py --version
    python --version

如果其中一條顯示 Python 3.x.x，代表對應命令可以找到 Python。

Windows 的 py 是 Python Launcher，python 則通常直接啟動 Python 執行檔。不同安裝方式可能只支援其中一個命令。

如果出現找不到命令：
1. 檢查是否已安裝 Python 3。
2. 可從 Python 官方網站下載安裝程式。
3. 安裝時留意 PATH 選項，並閱讀安裝器提示。
4. 關閉後重新開啟 PowerShell，再測試命令。

不要從不明網站下載安裝包。

## 五、檢查 Kali Linux 的 Python

打開 Kali 終端機：

    python3 --version
    which python3

Kali 通常提供 Python 3。第一條查看版本，第二條顯示目前找到的執行檔路徑。

如果沒有找到：
1. 檢查系統是否已完成基本安裝。
2. 檢查套件來源和網路狀態。
3. 參考 Kali 官方文件確認安裝方式。

不要為了讓 python 命令存在而隨意建立系統連結；在 Kali 中明確使用 python3 更清楚。

## 六、建立第一個程式

先建立一個專用資料夾：

Windows PowerShell：

    mkdir $HOME\cyber-practice\python-lab
    cd $HOME\cyber-practice\python-lab

Kali：

    mkdir -p ~/cyber-practice/python-lab
    cd ~/cyber-practice/python-lab

使用文字編輯器建立檔案 hello.py，輸入：

    print("Hello, cybersecurity!")
    print("This is my first Python program.")

儲存檔案。注意副檔名必須是 .py，不要不小心存成 hello.py.txt。

## 七、執行第一個程式

Windows PowerShell：

    py hello.py

如果系統沒有 py，但 python 命令有效，可以使用：

    python hello.py

Kali：

    python3 hello.py

預期輸出：

    Hello, cybersecurity!
    This is my first Python program.

這表示 Python 已成功執行這個檔案。

## 八、理解 print()

print() 是 Python 內建函式，用來把資料輸出到標準輸出，通常就是終端機畫面。

例如：

    print("Hello")
    print(2026)
    print(3.14)

輸出可以是文字、數字或其他物件。Python 會把它們轉成適合顯示的文字形式。

## 九、註解和縮排

### 1. 註解 Comment

    # This is a comment
    print("Python is running")

以 # 開始的註解不會當成一般 Python 語句執行。註解適合解釋程式目的，不要把密碼或 Token 寫在註解裡。

### 2. 縮排 Indentation

Python 使用縮排表示程式區塊。示例：

    if True:
        print("Inside the block")
    print("Outside the block")

if 下面的 print 有四個空格縮排，代表它屬於 if 區塊。縮排錯誤可能造成 IndentationError。

建議統一使用四個空格，不要混用 Tab 和空格。

## 十、建立並使用虛擬環境 venv

虛擬環境（Virtual Environment）讓不同 Python 專案可以使用各自的第三方套件，避免把套件全部安裝到系統 Python。

本模組前幾課只使用標準函式庫，不需要額外套件；先認識建立方式即可。

Windows PowerShell：

    py -m venv .venv
    .\.venv\Scripts\Activate.ps1

如果 PowerShell 阻止啟動腳本，不要急著修改整台電腦的執行政策。可以先使用未啟動環境的直譯器路徑執行程式，或參照 Python 官方 venv 文件了解符合自己政策的啟動方式。

Kali：

    python3 -m venv .venv
    source .venv/bin/activate

成功啟動後，終端機提示符通常會顯示環境名稱。離開虛擬環境：

    deactivate

Kali 若提示缺少 venv 支援套件，先查閱 Kali/Debian 套件文件，再依系統版本安裝相應套件。不要使用 sudo pip 把套件直接裝進系統 Python。

## 十一、常見錯誤排查

**錯誤：python is not recognized 或 command not found**

表示目前 Shell 找不到對應命令。確認 Python 是否安裝、命令名稱是否正確，以及 PATH 是否已載入。Windows 可以試 py；Kali 優先試 python3。

**錯誤：can't open file**

通常是檔案路徑或檔名不正確。先輸入：

    pwd
    ls

Windows PowerShell 可使用：

    Get-Location
    Get-ChildItem

確認目前目錄中是否真的有 hello.py。

**錯誤：IndentationError**

檢查區塊內是否統一使用四個空格，並確認 if、for、def 等語句後面有冒號。

**程式沒有任何輸出**

確認是否真的執行了正確檔案，是否保存了修改，以及 print() 是否位於會被執行的程式區塊。

## 十二、本課實作驗收

1. 在 Windows 或 Kali 建立 python-lab 資料夾。
2. 建立 hello.py。
3. 執行程式並看到兩行輸出。
4. 修改第二行文字，再執行一次。
5. 加入一行註解，確認程式仍能執行。
6. 用自己的話解釋 Python 直譯器是什麼。
7. 說明為什麼專案需要虛擬環境。

## 十三、本課測驗

1. .py 檔案通常保存什麼？
2. Windows 的 py 和 Kali 的 python3 分別有什麼用途？
3. print() 做什麼？
4. Python 為什麼重視縮排？
5. 註解會不會像一般程式語句一樣執行？
6. venv 的主要用途是什麼？
7. 為什麼不建議隨意用 sudo pip 安裝套件到系統 Python？

## 十四、參考資料

- [Python 官方下載頁](https://www.python.org/downloads/)
- [Python 官方教學：An Informal Introduction](https://docs.python.org/3/tutorial/introduction.html)
- [Python 官方文件：venv](https://docs.python.org/3/library/venv.html)

**完成標準：** 能自行建立和執行 hello.py，並能在 Windows 或 Kali 說明 Python 命令、檔案路徑和縮排的作用。
