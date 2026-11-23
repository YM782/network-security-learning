# 05-03｜條件判斷、迴圈和程式流程

> 難度：初級｜建議時間：90 分鐘｜先修：05-01、05-02

## 一、本課學習成果

完成後，你應該能夠：
1. 使用 if、elif 和 else 做條件判斷。
2. 使用 for 逐項處理資料。
3. 使用 while 重複執行條件成立的工作。
4. 理解 range() 的起點、終點和步長。
5. 使用 break 和 continue 控制迴圈。
6. 理解無限迴圈和輸入驗證的重要性。

## 二、程式流程是什麼？

程式流程（Control Flow）是程式語句實際執行的順序。

最基本的流程有三種：
- 順序：由上而下執行。
- 選擇：根據條件決定走哪個分支。
- 重複：在條件成立時重複執行一段程式。

安全分析腳本經常需要根據日誌等級選擇處理方式，並逐行讀取大量資料，因此條件和迴圈非常重要。

## 三、if 條件判斷

    status_code = 200

    if status_code == 200:
        print("Request succeeded")

if 後面放條件，條件成立時執行縮排區塊。

注意冒號和縮排，兩者都不能省略。

## 四、if、elif 和 else

    status_code = 404

    if status_code == 200:
        print("Success")
    elif status_code == 404:
        print("Not found")
    else:
        print("Other response")

- if：第一個判斷。
- elif：前面條件不成立時，繼續檢查其他條件。
- else：前面的條件都不成立時執行。

一組 if/elif/else 只會選擇符合流程的分支，不會把所有分支都執行一遍。

## 五、條件可以組合

    is_authenticated = True
    has_permission = True
    account_locked = False

    if is_authenticated and has_permission and not account_locked:
        print("Access allowed")
    else:
        print("Access denied")

這個例子示範多個條件組合。真實授權必須在可信任的伺服器端完成，不能只相信客戶端送來的布林值。

## 六、for 迴圈

for 適合逐項處理一個序列或可迭代物件。

    for number in [1, 2, 3, 4, 5]:
        print(number)

每次迴圈會把下一個元素放進 number，再執行區塊。

### range()

    for number in range(1, 6):
        print(number)

range(1, 6) 產生 1、2、3、4、5，不包含 6。

也可以設定步長：

    for number in range(0, 10, 2):
        print(number)

這會輸出 0、2、4、6、8。

## 七、while 迴圈

while 在條件成立時重複執行：

    count = 1

    while count <= 5:
        print(count)
        count = count + 1

每次執行都會增加 count，當 count 大於 5 時，條件不成立，迴圈結束。

如果忘記改變條件，可能形成無限迴圈，讓程式一直運行。

## 八、break 和 continue

### break：離開整個迴圈

    for number in range(1, 10):
        if number == 5:
            break
        print(number)

當 number 等於 5 時，break 終止迴圈。

### continue：跳過這一輪

    for number in range(1, 6):
        if number == 3:
            continue
        print(number)

當 number 等於 3 時，跳過本輪剩餘語句，繼續下一輪。

## 九、enumerate()：同時取得索引和值

    events = ["INFO", "WARNING", "ERROR"]

    for index, event in enumerate(events, start=1):
        print(index, event)

enumerate() 可以同時提供序號和元素，適合逐行分析檔案或整理事件清單。

## 十、完整練習：統計測試狀態

    statuses = [200, 404, 200, 500, 403, 200]

    success_count = 0
    error_count = 0

    for status in statuses:
        if status == 200:
            success_count += 1
        elif status >= 400:
            error_count += 1

    print(f"Success: {success_count}")
    print(f"Error: {error_count}")

預期輸出：

    Success: 3
    Error: 3

這裡的資料是程式內的虛構測試值，沒有向任何網站發出請求。

## 十一、常見錯誤排查

**程式一直不停止**

檢查 while 條件是否會改變，是否有正確更新計數器，是否有合理的結束條件。必要時在自己的終端機使用 Ctrl+C 中斷目前前景程式。

**輸出比預期少**

檢查 break 是否過早執行，以及 if 條件是否把某些資料排除。

**輸出比預期多**

檢查是否把 print() 放在錯誤的縮排層級，或是否重複處理了同一筆資料。

**SyntaxError**

檢查冒號、括號、引號和運算子拼寫。

## 十二、本課實作驗收

1. 使用 if 判斷一個數字是正數、負數或零。
2. 使用 for 輸出 1 到 10。
3. 使用 range() 輸出 2、4、6、8、10。
4. 使用 while 從 5 倒數到 1。
5. 使用 continue 跳過列表中的一個指定值。
6. 使用 break 在遇到指定值時結束迴圈。
7. 用自己的話解釋無限迴圈為什麼會發生。

## 十三、本課測驗

1. if、elif 和 else 分別做什麼？
2. range(1, 6) 會產生哪些數字？
3. for 和 while 的典型使用場景有什麼不同？
4. break 和 continue 有什麼差別？
5. 為什麼 while 迴圈需要可達到的結束條件？
6. enumerate() 可以幫助我們取得什麼？
7. 為什麼不能只依靠客戶端的 if 判斷保護真實系統權限？

## 十四、參考資料

- [Python 官方教學：if Statements](https://docs.python.org/3/tutorial/controlflow.html#if-statements)
- [Python 官方教學：for Statements](https://docs.python.org/3/tutorial/controlflow.html#for-statements)
- [Python 官方文件：range()](https://docs.python.org/3/library/stdtypes.html#range)

**完成標準：** 能使用條件、for 和 while 處理一組資料，並能解釋 break、continue 和無限迴圈。
