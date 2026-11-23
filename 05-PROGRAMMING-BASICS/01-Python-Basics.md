# 05-01｜Python 基本語法（快速複習）

> 這是原本的入門速覽，保留作為快速複習；完整課程請從 [05 Python 程式設計基礎](README.md) 開始。

## 本課目標
能讀懂簡單 Python 程式，理解輸出、變數、資料型別、條件、迴圈和函式。

## 1. Python 是什麼？
Python 是常用於資料處理、自動化、Web 開發和安全工具的程式語言。現在的目標是先看懂簡單程式，不是立即成為專業程式設計師。

## 2. 第一個程式
```python
print("Hello, cybersecurity!")
```
`print()` 會把括號中的內容輸出到畫面。

## 3. 變數和資料型別
```python
name = "Kali"
count = 3
is_ready = True

print(name)
print(count)
print(is_ready)
print(type(name))
```
`name` 是字串（str）；`count` 是整數（int）；`is_ready` 是布林值（bool）；`type()` 查看資料型別。

## 4. 使用者輸入
```python
user_name = input("Enter your name: ")
print("Hello", user_name)
```
`input()` 等待使用者輸入文字。

## 5. 條件判斷
```python
status_code = 200

if status_code == 200:
    print("Request succeeded")
else:
    print("Check the response")
```
`=` 是指定值；`==` 是比較是否相等。

## 6. 迴圈
```python
for number in range(1, 6):
    print(number)
```
`range(1, 6)` 會產生 1 到 5，不包含 6。

## 7. 函式
```python
def greet(user):
    print("Hello", user)

greet("Michael")
```
`def` 定義函式；呼叫 `greet()` 時執行函式內容。

## 8. 本課練習
1. 修改第一個程式輸出的文字。
2. 建立一個變數保存練習代號。
3. 用 if 判斷數字是否大於 10。
4. 用 for 輸出 1 到 5。
5. 建立一個函式，接收名稱並輸出問候。

**完成標準：** 能逐行說明程式用途，不要求背熟全部語法。
