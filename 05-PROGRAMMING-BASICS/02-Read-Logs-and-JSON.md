# 05-02｜讀取文字檔、JSON 和日誌（快速複習）

> 這是原本的檔案與日誌練習，保留作為快速複習；完整課程請接著閱讀 [05 檔案、JSON、CSV 和例外處理](05-Files-JSON-CSV.md) 與 [06 正規表示式與離線日誌分析實驗](06-Regex-and-Log-Analysis.md)。

## 本課目標
理解安全人員為什麼需要整理文字、設定檔和日誌，並用 Python 讀取自己建立的資料。

## 1. 建立測試日誌
在練習目錄建立 sample.log，內容：

```text
2026-11-23 INFO application started
2026-11-23 WARN disk space is low
2026-11-23 ERROR test connection failed
```

這是虛構測試資料，不是真實系統日誌。

## 2. Python 讀取文字檔
```python
with open("sample.log", "r", encoding="utf-8") as file:
    for line in file:
        print(line.strip())
```
`open()` 開啟檔案；`r` 是讀取模式；`encoding` 指定文字編碼；`with` 協助妥善關閉檔案；`strip()` 移除行首和行尾空白。先確認程式與 sample.log 位於相同目錄。

## 3. 只搜尋 ERROR
```python
with open("sample.log", "r", encoding="utf-8") as file:
    for line in file:
        if "ERROR" in line:
            print(line.strip())
```
這段程式只搜尋自己建立的測試日誌，不會修改檔案。

## 4. JSON 是什麼？
JSON 是常見資料交換格式，使用鍵和值保存資料。

```json
{
  "device": "lab-kali",
  "ip": "127.0.0.1",
  "purpose": "local practice"
}
```

## 5. 使用 Python 讀取 JSON
```python
import json

with open("device.json", "r", encoding="utf-8") as file:
    data = json.load(file)

print(data["device"])
print(data["ip"])
```
`json.load()` 把 JSON 檔案讀成 Python 資料結構。

## 6. 為什麼安全人員需要這些技能？
可以用來找日誌錯誤、整理安全告警、比較設定檔、統計測試結果和整理實驗報告。不要把真實密碼、Token 或私人日誌上傳到公開 GitHub。

## 7. 本課驗收
1. open() 的 r 模式代表什麼？
2. with 有什麼好處？
3. JSON 如何保存資料？
4. 程式如何只輸出包含 ERROR 的行？
5. 為什麼先使用虛構測試資料？
