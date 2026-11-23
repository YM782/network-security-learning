
# 05 — 程式設計入門：先學看懂，不急著成為程式設計師

## 為什麼網路安全需要程式？

安全人員會用程式整理日誌、處理文字、分析檔案、呼叫 API 和自動化重複工作。你不需要一開始就寫大型軟體，但要逐步看懂簡單程式。

## 1. 先認識 Python

Python 是一種常用的程式語言，語法相對容易閱讀，適合初學者和安全自動化。

第一個例子：

```python
print("Hello, cybersecurity!")
```

`print` 是輸出文字的函式。括號內的文字會顯示在畫面上。

## 2. 變數（Variable）

變數可以理解為有名稱的資料盒子：

```python
name = "Michael"
age = 20

print(name)
print(age)
```

- name 儲存文字。
- age 儲存數字。
- = 在這裡代表把右邊的值指定給左邊變數。

## 3. 條件判斷（if）

```python
status_code = 200

if status_code == 200:
    print("Request succeeded")
else:
    print("Check the response")
```

`==` 是比較是否相等；`=` 是指定值。這兩個符號用途不同。

## 4. 迴圈（Loop）

```python
for number in range(1, 4):
    print(number)
```

這段程式會輸出 1、2、3。迴圈可以讓電腦重複處理資料。

## 5. 函式（Function）

```python
def greet(user):
    print("Hello", user)

greet("Michael")
```

函式把一段工作包裝起來，之後可以重複使用。

## 6. 清單和字典

```python
ports = [80, 443, 22]

device = {
    "name": "lab-pc",
    "ip": "127.0.0.1"
}

print(ports[0])
print(device["name"])
```

- List（清單）：按順序保存多筆資料。
- Dictionary（字典）：使用鍵和值保存資料。

## 7. 網路安全中會怎麼用？

初期只做安全、簡單的任務：

- 整理自己產生的文字日誌。
- 統計一份測試資料中出現的狀態碼。
- 讀取 JSON 格式的設定檔。
- 把實驗結果整理成 CSV。
- 自動化重複的本機資料整理。

不要下載來路不明的「駭客腳本」直接執行。先閱讀程式，理解它會讀取、修改或傳送哪些資料。

## 8. Web 最低限度要認識的內容

網站通常由 HTML、CSS、JavaScript 和伺服器端程式共同組成：

- HTML：網頁結構。
- CSS：網頁外觀。
- JavaScript：瀏覽器中的互動邏輯。
- Backend：伺服器端處理程式。
- Database：保存資料的資料庫。

先不要求你自己寫完整網站。只要知道瀏覽器如何把請求送到伺服器，就能為後面的 Web 安全打底。

## 9. 本課練習

1. 把第一個程式的文字改成自己的句子。
2. 建立一個變數保存自己的練習代號。
3. 用 if 判斷一個數字是否大於 10。
4. 用 for 輸出 1 到 5。
5. 用字典保存一台虛擬機的名稱和練習用 IP。

**完成標準：** 能讀懂上面的程式，知道每一行大致在做什麼，不要求一次背熟 Python。
