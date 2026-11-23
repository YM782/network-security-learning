
# 03 — Linux 命令列入門

## 本課目標

先學會在 Kali 裡移動、查看和管理檔案。初期不需要背幾十個命令。

## 1. 終端機是什麼？

Terminal（終端機）是輸入命令和閱讀結果的介面。Shell（命令解譯器）負責理解你輸入的命令，再請系統執行。

命令通常由三部分組成：

```text
命令名稱  選項  參數
```

例如：

```bash
ls -l /home
```

- ls：列出檔案。
- -l：使用較詳細的清單格式。
- /home：要查看的目錄。

## 2. 第一組命令：認識目前位置

```bash
pwd
whoami
hostname
```

- pwd：顯示目前所在目錄。
- whoami：顯示目前使用者。
- hostname：顯示電腦名稱。

## 3. 第二組命令：查看和切換目錄

```bash
ls
ls -la
cd /home
pwd
cd ..
cd ~
```

- ls：列出目前目錄內容。
- ls -la：包含隱藏項目和詳細資訊。
- cd /home：進入 /home。
- cd ..：回到上一層目錄。
- cd ~：回到目前使用者的家目錄。

Linux 的路徑使用正斜線 /。大小寫有差別，`Downloads` 和 `downloads` 可能是兩個不同目錄。

## 4. 第三組命令：建立檔案和目錄

```bash
mkdir practice
cd practice
touch note.txt
ls -l
```

- mkdir：建立目錄。
- touch：建立空檔案；若檔案已存在，通常更新時間資訊。
- ls -l：確認結果。

## 5. 第四組命令：讀取檔案

```bash
echo "Hello Kali" > note.txt
cat note.txt
less note.txt
```

- echo：輸出文字。
- >：把輸出寫入檔案，會覆蓋原內容。
- cat：直接顯示檔案內容。
- less：分頁查看較長檔案，按 q 離開。

小心：`>` 會覆蓋檔案；`>>` 才是附加到檔案末尾。操作重要檔案前先確認路徑。

## 6. 第五組命令：複製、移動和刪除

```bash
cp note.txt note-copy.txt
mv note-copy.txt note-backup.txt
ls -l
```

刪除練習檔：

```bash
rm note-backup.txt
```

**安全提醒：** rm 刪除檔案通常不會送進圖形介面的垃圾桶。初學時只刪除自己剛建立的練習檔。不要照抄含有 `sudo rm -rf` 的網路指令。

## 7. 搜尋和閱讀說明

```bash
find . -name "note.txt"
grep "Hello" note.txt
man ls
```

- find：尋找檔案。
- grep：在文字中搜尋指定內容。
- man：查看命令手冊，按 q 離開。

## 8. 管理員權限與 sudo

sudo 讓符合權限的使用者以更高權限執行特定命令。它不是「讓所有命令都更正確」的按鈕。

例如，安裝套件需要管理員權限：

```bash
sudo apt update
```

執行前先理解命令的作用。不要把陌生網站給你的整串 sudo 命令直接貼進終端機。

## 9. 本課練習

在家目錄建立 `cyber-practice` 目錄，裡面建立 `hello.txt`，寫入一行文字，顯示內容，複製一份，再刪除複製檔。

請把你執行的每一條命令寫在筆記中，並說明它的用途。

## 10. 下一步：OverTheWire Bandit

完成本課後，再到 [Bandit](https://overthewire.org/wargames/bandit/) 從 Level 0 開始。它是為絕對初學者設計的 Linux 命令列闖關。先做前幾關即可，不用急著一次完成所有關卡。
