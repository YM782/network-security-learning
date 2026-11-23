
# 03-02｜Linux 檔案和文字操作

## 本課目標
在自己的練習目錄建立、讀取、複製、移動、搜尋和刪除檔案。

## 1. 建立練習目錄
```bash
mkdir -p ~/cyber-practice/files
cd ~/cyber-practice/files
pwd
```
mkdir 建立目錄；-p 必要時建立上層目錄；~ 代表目前使用者家目錄。

## 2. 建立檔案並寫入文字
```bash
touch note.txt
echo "Hello Kali" > note.txt
cat note.txt
```
touch 建立空檔案；echo 輸出文字；> 將輸出寫入檔案並覆蓋原內容；cat 顯示內容。

若要附加文字而不覆蓋：
```bash
echo "Second line" >> note.txt
cat note.txt
```
>> 會把文字附加到檔案末尾。

## 3. 複製和移動
```bash
cp note.txt note-copy.txt
mv note-copy.txt backup.txt
ls -l
```
cp 複製檔案；mv 移動或重新命名；ls -l 確認結果。

## 4. 閱讀較長檔案
```bash
less note.txt
head -n 5 note.txt
tail -n 5 note.txt
```
less 分頁閱讀，按 q 離開；head 查看前幾行；tail 查看最後幾行；-n 5 指定五行。

## 5. 搜尋檔案和文字
```bash
find . -name "note.txt"
grep "Hello" note.txt
```
find 從目前目錄尋找檔案；. 代表目前目錄；grep 搜尋文字。

## 6. 刪除練習檔
```bash
rm backup.txt
ls -l
```
rm 通常不會把檔案送進圖形介面的垃圾桶。只刪除自己建立的測試檔；刪除前用 pwd 和 ls 確認位置。不要照抄不明來源的遞迴刪除命令。

## 7. 本課驗收
在 ~/cyber-practice/files 建立 note.txt，寫入兩行，複製、重新命名，再刪除副本；能解釋每條命令和 >、>> 的差別。
