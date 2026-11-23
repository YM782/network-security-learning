# 03-02｜Linux 檔案操作：建立、讀取、搜尋、複製和刪除

> 難度：初級｜建議時間：90 分鐘｜先修：03-01
>
> 檔案操作是 Linux 最重要的基本能力之一。安全人員需要閱讀設定檔、分析日誌、搜尋字串、整理證據和保存實驗結果。本課所有操作都限制在自己的練習目錄。

## 一、本課學習成果

完成後，你應該能夠：
1. 使用 mkdir、touch、echo 建立目錄和檔案。
2. 使用 cat、less、head、tail 閱讀文字。
3. 使用 cp 和 mv 複製、移動和重新命名檔案。
4. 使用 find 搜尋檔案，使用 grep 搜尋文字。
5. 理解 >、>> 和管線 | 的基本作用。
6. 安全地刪除自己建立的練習檔。

## 二、建立專用練習環境

先建立目錄：

    mkdir -p ~/cyber-practice/files
    cd ~/cyber-practice/files
    pwd

確認輸出路徑位於自己的家目錄下。後續所有操作都在這個練習目錄進行。

## 三、建立檔案和目錄

### 1. mkdir

    mkdir reports

建立名為 reports 的目錄。

如果上層目錄不存在，可以使用：

    mkdir -p ~/cyber-practice/files/lab/day1

-p 會建立所需的上層目錄。

### 2. touch

    touch note.txt

建立空檔案。如果檔案已存在，touch 通常會更新時間戳記，不會清空原內容。

### 3. echo

    echo "Hello Kali"

把文字輸出到終端機。

要把輸出保存到檔案：

    echo "Hello Kali" > note.txt

這裡的 > 是 Shell 的輸出重新導向。它會建立檔案，或覆蓋既有檔案內容。使用前一定要確認目標檔案，避免覆蓋重要資料。

若要附加內容：

    echo "Second line" >> note.txt

>> 會把內容加到檔案末尾，而不是覆蓋原有內容。

## 四、閱讀檔案

### 1. cat

    cat note.txt

顯示整個檔案內容，適合小型文字檔。不要直接 cat 非文字大型檔案，可能產生大量無法閱讀的輸出。

### 2. less

    less note.txt

以分頁方式閱讀較長的檔案：
- 空白鍵：向下翻頁。
- b：向上翻頁。
- /word：搜尋 word。
- n：下一個搜尋結果。
- q：離開。

### 3. head 和 tail

    head -n 5 note.txt
    tail -n 5 note.txt

head 查看前幾行；tail 查看最後幾行；-n 5 指定五行。

查看日誌最新內容時，常會用：

    tail -n 20 application.log

這只是讀取檔案，不會修改日誌。

## 五、複製、移動和重新命名

### 1. cp：複製

    cp note.txt note-copy.txt

把 note.txt 複製成 note-copy.txt。

複製到另一個目錄：

    cp note.txt reports/

### 2. mv：移動或重新命名

    mv note-copy.txt backup.txt

把檔案重新命名為 backup.txt。

移動到另一個目錄：

    mv backup.txt reports/

mv 也可能覆蓋目的地既有檔案，執行前先確認目標名稱是否已存在。初學時不要對重要目錄使用不熟悉的覆蓋選項。

## 六、搜尋檔案：find

在目前目錄尋找 note.txt：

    find . -name "note.txt"

參數：
- find：搜尋檔案系統中的項目。
- .：從目前目錄開始。
- -name：依名稱匹配。
- "note.txt"：要搜尋的名稱模式。

搜尋所有 .log 檔案：

    find . -type f -name "*.log"

- -type f：只找一般檔案。
- *.log：符合以 .log 結尾的名稱。
- 引號可以避免 Shell 先把萬用字元展開成目前目錄的檔案清單。

不要一開始就從整個 / 根目錄搜尋，可能產生大量結果並遇到權限錯誤。先從自己的練習目錄開始。

## 七、搜尋文字：grep

先建立測試檔：

    printf "INFO service started\nWARN disk space low\nERROR test failed\n" > sample.log

搜尋 ERROR：

    grep "ERROR" sample.log

常見選項：

    grep -n "ERROR" sample.log
    grep -i "error" sample.log
    grep -n -i "error" sample.log

- -n：顯示行號。
- -i：忽略英文字母大小寫。
- 多個選項可以一起使用。

在目前練習目錄搜尋所有 .log 檔：

    grep -n "ERROR" ./*.log

如果目錄中沒有符合的檔案，Shell 可能會把萬用字元原樣交給 grep，並顯示找不到檔案。先用 ls 確認目標存在。

## 八、管線 Pipe：把一個命令的輸出交給另一個命令

例如：

    ps aux | less

左側 ps aux 產生程序清單；右側 less 讓你分頁閱讀。管線符號 | 把左側標準輸出送到右側標準輸入。

另一個例子：

    ls -l | grep "note"

先列出檔案，再把輸出交給 grep 搜尋包含 note 的行。

管線不會自動保存結果。如果要保存，可以使用重新導向：

    ps aux > process-list.txt

這會把程序清單寫入檔案並覆蓋既有內容。若要附加，使用 >>。

## 九、標準輸入、輸出和錯誤輸出

Linux 程式通常有三種標準資料流：

| 名稱 | 檔案描述符 | 用途 |
|---|---:|---|
| stdin | 0 | 標準輸入 |
| stdout | 1 | 一般輸出 |
| stderr | 2 | 錯誤訊息 |

例如：

    ls /not-a-real-folder

錯誤訊息通常會送到 stderr。你可以把一般輸出和錯誤輸出分別保存：

    command > output.txt 2> error.txt

這裡的 command 只是語法示意，不要把字面上的 command 當成需要執行的具體工具。

## 十、刪除練習檔

先確認所在位置：

    pwd
    ls -la

只刪除自己建立的測試檔：

    rm note-copy.txt

rm 通常不會把檔案送到圖形介面的垃圾桶。刪除前一定要確認路徑和檔名。

本課不要求使用 rm -rf。遞迴強制刪除可能造成大量資料無法恢復，不要照抄網路上不理解的刪除命令。

## 十一、完整實驗：整理一份簡單日誌

### 實驗步驟

    cd ~/cyber-practice/files
    mkdir -p log-lab
    cd log-lab
    printf "INFO app started\nWARN disk space low\nERROR login test failed\nINFO app stopped\n" > app.log
    cat app.log
    grep -n "ERROR" app.log
    cp app.log app-copy.log
    mv app-copy.log archived.log
    ls -lh
    find . -type f -name "*.log"

### 預期結果

- 目錄中有 app.log 和 archived.log。
- grep 只顯示包含 ERROR 的行及行號。
- find 找到兩個 .log 檔案。

### 完成後檢查

    pwd
    ls -la

確認檔案都在自己的 log-lab 目錄內。

## 十二、常見錯誤與排查

**問題 1：cat: file: No such file or directory**

先執行 pwd 和 ls，確認目前位置和檔名。注意大小寫。

**問題 2：Permission denied**

查看檔案權限：

    ls -l filename

不要為了讀取一個檔案就隨意修改整個目錄權限。先確認檔案是否屬於自己，以及是否有合理的存取方式。

**問題 3：echo > file 後原內容不見了**

因為 > 會覆蓋檔案。要保留原內容並追加，應使用 >>。如果已覆蓋，先停止寫入，不要再做更多修改；能否恢復取決於檔案系統、備份和後續寫入情況。

**問題 4：grep 沒有輸出**

可能是檔案中沒有匹配文字、大小寫不同或指定了錯誤檔案。使用 cat 或 less 確認內容，再考慮 -i。

## 十三、本課測驗

1. touch 和 mkdir 分別做什麼？
2. > 和 >> 有什麼不同？
3. cat 和 less 適合什麼情況？
4. cp 和 mv 有什麼不同？
5. find 和 grep 搜尋的對象有何不同？
6. 管線 | 的作用是什麼？
7. 為什麼不應在不確認路徑時使用遞迴刪除？

## 十四、參考資料

- [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/)
- [GNU grep Manual](https://www.gnu.org/software/grep/manual/)
- [Linux man-pages：find](https://man7.org/linux/man-pages/man1/find.1.html)
- [Linux man-pages：less](https://man7.org/linux/man-pages/man1/less.1.html)

**完成標準：** 能在自己的練習目錄建立日誌、搜尋指定文字、複製和重新命名檔案，並清楚解釋每一條命令及重新導向的風險。