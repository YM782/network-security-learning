# 03-03｜Linux 使用者、群組、檔案權限和 sudo

> 難度：初級至初中級｜建議時間：90 分鐘｜先修：03-01、03-02
>
> 權限是 Linux 安全的基礎。你必須理解「誰在執行命令」「誰擁有檔案」和「目前帳號能做什麼」，才能安全管理系統。

## 一、本課學習成果

完成後，你應該能夠：
1. 使用 whoami 和 id 查看目前使用者及群組。
2. 看懂 ls -l 的基本權限字串。
3. 解釋讀取、寫入和執行權限。
4. 使用 chmod 修改自己建立的測試檔權限。
5. 理解 sudo、root 和最小權限原則。
6. 知道為什麼 777 權限通常不是安全的解決方法。

## 二、Linux 為什麼需要使用者和群組？

Linux 是多使用者作業系統。不同使用者可以登入同一台設備，並依照權限存取資源。

常見身分：
- 一般使用者：執行日常工作，權限受到限制。
- root：傳統 Unix/Linux 超級使用者，擁有非常高的系統管理權限。
- 系統服務帳號：供特定服務執行，通常不應用來互動登入。

群組（Group）可以把多個使用者歸在一起，方便管理共享檔案和資源權限。

## 三、查看目前使用者

### 1. whoami

    whoami

顯示目前有效使用者名稱。

### 2. id

    id

可能看到 UID、GID 和群組清單。

- UID：User ID，使用者識別碼。
- GID：Group ID，主要群組識別碼。
- Groups：目前使用者所屬的群組。

不要只憑帳號名稱判斷權限，還要看 UID、群組、檔案權限和其他存取控制設定。

## 四、看懂 ls -l 的權限資訊

先建立測試檔：

    cd ~/cyber-practice/files
    touch permission-demo.txt
    ls -l permission-demo.txt

可能看到類似：

    -rw-r--r-- 1 kali kali 0 Nov 23 10:00 permission-demo.txt

最前面的字元表示檔案類型，後面九個字元分成三組：

    rw-  r--  r--

- 第一組：檔案擁有者（Owner）。
- 第二組：檔案所屬群組（Group）。
- 第三組：其他使用者（Others）。

三種權限：
- r：read，讀取。
- w：write，寫入。
- x：execute，執行。

對目錄而言，x 通常代表能進入或搜尋該目錄；目錄的 r 和 w 與列出、建立或刪除目錄項目有關。檔案和目錄的權限效果不完全相同。

## 五、權限數字表示法

Linux 常用八進位數字表示 rwx：

| 權限 | 數字 | 說明 |
|---|---:|---|
| --- | 0 | 沒有權限 |
| --x | 1 | 執行 |
| -w- | 2 | 寫入 |
| -wx | 3 | 寫入和執行 |
| r-- | 4 | 讀取 |
| r-x | 5 | 讀取和執行 |
| rw- | 6 | 讀取和寫入 |
| rwx | 7 | 讀取、寫入和執行 |

三位數字依序代表 Owner、Group、Others。

例如：
- 644：擁有者讀寫；群組和其他人只讀。
- 600：只有擁有者讀寫。
- 755：擁有者可讀寫執行；群組和其他人可讀取和執行。
- 700：只有擁有者可讀寫執行。

這些只是常見配置，不代表所有檔案都應使用同一組權限。

## 六、chmod：修改權限

### 1. 數字方式

對自己建立的測試檔執行：

    chmod 600 permission-demo.txt
    ls -l permission-demo.txt

預期權限會變成：

    -rw-------

這代表只有檔案擁有者可以讀取和寫入。

### 2. 符號方式

    chmod u+x script.sh

- u：user，擁有者。
- +：增加權限。
- x：執行權限。

其他符號：
- g：group。
- o：others。
- a：all，全部身分類別。
- -：移除權限。
- =：設定成指定權限。

例如：

    chmod go-rwx private.txt

移除群組和其他使用者對 private.txt 的讀、寫、執行權限。只對自己的測試檔操作。

## 七、chown 和 chgrp 是什麼？

- chown：修改檔案擁有者，也可以修改群組。
- chgrp：修改檔案所屬群組。

查看方式：

    ls -l permission-demo.txt

初學階段先學會辨識擁有者和群組，不要為了練習而修改系統目錄的擁有者。

## 八、sudo 和 root

### 1. root 是什麼？

root 是 Linux 的超級使用者帳號，具有很高的系統管理權限。錯誤的 root 操作可能刪除系統檔案、修改網路設定或破壞套件管理狀態。

### 2. sudo 是什麼？

sudo 讓獲得授權的使用者以較高權限執行指定命令。

例如：

    sudo apt update

這裡 sudo 只提升這條命令的執行權限，不代表之後所有命令都自動以 root 執行。

### 3. 為什麼不要習慣性加 sudo？

如果一條命令只需要讀取自己家目錄裡的檔案，通常不需要 sudo。使用過高權限可能讓錯誤命令造成更大損害。

最小權限原則（Least Privilege）：使用者和程式只取得完成工作所需的最低權限。

## 九、為什麼 chmod 777 不應成為萬用解法？

777 代表 Owner、Group 和 Others 都有讀、寫、執行權限。

可能風險：
- 其他本機使用者可能修改檔案。
- 可執行檔可能被未授權者替換。
- 目錄中的檔案可能被刪除或新增。
- 錯誤權限可能讓敏感資料暴露。

如果遇到 Permission denied，正確做法是查明誰需要什麼權限，而不是把所有權限開到最大。

## 十、動手實驗：觀察並修改自己的測試檔

逐條執行：

    mkdir -p ~/cyber-practice/permission-lab
    cd ~/cyber-practice/permission-lab
    touch private.txt
    ls -l private.txt
    chmod 600 private.txt
    ls -l private.txt
    chmod 644 private.txt
    ls -l private.txt

觀察權限字串如何變化。

再建立一個測試腳本：

    printf '#!/bin/sh\necho "permission lab"\n' > demo.sh
    ls -l demo.sh
    chmod u+x demo.sh
    ls -l demo.sh

只有在你確認檔案內容是自己建立的測試程式後，才執行：

    ./demo.sh

## 十一、常見錯誤與排查

**問題 1：Permission denied**

先檢查：

    whoami
    id
    ls -l filename

確認目前使用者、群組、檔案擁有者和權限。不要先使用 chmod 777。

**問題 2：chmod: Operation not permitted**

可能是你不是檔案擁有者，或檔案系統有額外限制。先確認檔案位置和擁有者，不要直接對整個目錄使用 sudo chmod。

**問題 3：腳本存在但無法執行**

檢查：
1. 檔案內容是否有正確的 shebang。
2. 是否具有執行權限。
3. 目前目錄是否正確。
4. 檔案是否位於不允許執行的掛載位置。

## 十二、本課測驗

1. UID 和 GID 是什麼？
2. ls -l 中三組 rwx 分別代表誰？
3. 644 和 600 有什麼不同？
4. 目錄的 x 權限通常有什麼用途？
5. sudo 和 root 有什麼關係？
6. 為什麼 chmod 777 不是萬用解法？
7. Permission denied 時，應該先檢查哪三項資訊？

## 十三、參考資料

- [Linux man-pages：chmod](https://man7.org/linux/man-pages/man1/chmod.1.html)
- [Linux man-pages：chown](https://man7.org/linux/man-pages/man1/chown.1.html)
- [Linux man-pages：sudo](https://man7.org/linux/man-pages/man8/sudo.8.html)
- [Debian Reference](https://www.debian.org/doc/manuals/debian-reference/)

**完成標準：** 能看懂 rwx 權限，能對自己建立的檔案設定 600 和 644，並能解釋為什麼不應隨意使用 sudo 或 chmod 777。