# 05-02｜變數、資料型別、運算子和輸入輸出

> 難度：初級｜建議時間：75–100 分鐘｜先修：05-01

## 一、本課學習成果

完成後，你應該能夠：
1. 建立變數並理解賦值。
2. 分辨 str、int、float、bool 和 None。
3. 使用 type() 查看資料型別。
4. 理解數字、比較和邏輯運算子。
5. 使用 input() 接收文字輸入。
6. 使用 int()、float() 和 f-string 處理資料。
7. 避免把不可信輸入直接當成程式碼執行。

## 二、什麼是變數 Variable？

變數可以理解為一個有名稱的資料參照，讓程式能在後續使用某個值。

    user_name = "Michael"
    login_count = 3
    is_active = True

左側是變數名稱，右側是要賦給變數的值。單一等號 = 在這裡表示賦值，不是數學上的相等比較。

Python 變數名稱通常使用英文字母、數字和底線，但不能以數字開頭，也不能使用 Python 保留關鍵字。

建議使用能表達用途的名稱，例如 login_count，而不是 x1。

## 三、五種常見資料型別

### 1. str：字串 String

    user_name = "Michael"
    message = "Hello, Python"

字串用來保存文字。單引號和雙引號都可以建立字串。

### 2. int：整數 Integer

    port = 443
    retry_count = 3

整數可用來保存數量、連接埠或計數值。

### 3. float：浮點數 Floating Point

    cpu_usage = 25.5
    response_time = 0.032

浮點數適合表示帶小數的數值。由於二進位浮點表示的限制，有些十進位小數無法精確表示。

### 4. bool：布林值 Boolean

    is_connected = True
    has_error = False

布林值只有 True 和 False，常用於條件判斷。

### 5. None：空值

    result = None

None 表示目前沒有值或沒有結果。它和字串 "None" 不同，也和數字 0 不同。

## 四、查看資料型別：type()

    user_name = "Michael"
    port = 443
    cpu_usage = 25.5
    is_connected = True
    result = None

    print(type(user_name))
    print(type(port))
    print(type(cpu_usage))
    print(type(is_connected))
    print(type(result))

你會看到對應的 str、int、float、bool 和 NoneType。

## 五、型別轉換 Type Conversion

input() 接收的資料預設是字串，即使使用者輸入 25，也不會自動變成整數。

    age_text = input("Enter a number: ")
    age = int(age_text)
    print(age + 1)

int() 將可解析的數字文字轉成整數；float() 將可解析的數字文字轉成浮點數；str() 可以把許多值轉成文字。

如果使用者輸入 abc，int("abc") 會產生 ValueError。下一課會進一步練習錯誤處理。

## 六、基本算術運算子

| 運算子 | 意義 | 範例 |
|---|---|---|
| + | 加法 | 7 + 3 = 10 |
| - | 減法 | 7 - 3 = 4 |
| * | 乘法 | 7 * 3 = 21 |
| / | 除法 | 7 / 2 = 3.5 |
| // | 整除 | 7 // 2 = 3 |
| % | 取餘數 | 7 % 2 = 1 |
| ** | 次方 | 2 ** 3 = 8 |

注意：除以零會產生錯誤。不要把來自使用者或外部檔案的數字視為一定有效。

## 七、比較運算子

| 運算子 | 意義 |
|---|---|
| == | 相等 |
| != | 不相等 |
| > | 大於 |
| < | 小於 |
| >= | 大於或等於 |
| <= | 小於或等於 |

例如：

    status_code = 200
    print(status_code == 200)
    print(status_code >= 400)

比較運算的結果通常是 True 或 False。

特別注意：
- = 是賦值。
- == 是比較是否相等。

## 八、邏輯運算子

- and：兩邊條件都成立才為 True。
- or：至少一邊條件成立就可能為 True。
- not：反轉布林結果。

例如：

    is_authenticated = True
    has_permission = False

    can_access = is_authenticated and has_permission
    print(can_access)

這個例子中，雖然已驗證身分，但沒有權限，因此 can_access 為 False。真實系統的授權判斷會更複雜，不能只靠前端變數控制。

## 九、input() 接收使用者輸入

    user_name = input("Your name: ")
    print("Hello", user_name)

input() 會暫停等待輸入，並回傳字串。

接收數字時要明確轉換：

    port_text = input("Enter a port: ")
    port = int(port_text)
    print(port)

這個簡化例子假設輸入一定是整數。實際程式應檢查輸入是否有效，並檢查連接埠是否在 1 到 65535 的有效範圍內。

## 十、f-string：把變數放進文字

    user_name = "Michael"
    port = 443

    print(f"User: {user_name}")
    print(f"Service port: {port}")

f-string 以 f 開頭，花括號內可以放變數或表達式。

例如：

    response_time = 0.032
    print(f"Response time: {response_time:.3f} seconds")

:.3f 表示以小數點後三位顯示。

## 十一、完整小練習：簡單連接埠資料卡

    service_name = "HTTPS"
    port = 443
    is_enabled = True

    print("Service Information")
    print(f"Name: {service_name}")
    print(f"Port: {port}")
    print(f"Enabled: {is_enabled}")
    print(f"Port type: {type(port)}")

預期能看到服務名稱、連接埠、啟用狀態和資料型別。

這只是本地資料展示，不會開啟網路連線或掃描任何設備。

## 十二、安全提醒：不要使用 eval()

不要用 eval() 把使用者輸入的文字直接當成 Python 程式碼執行。

使用者輸入、檔案內容、網路回應和日誌都應視為不可信資料。需要數字時使用 int() 或 float() 並處理錯誤；需要解析 JSON 時使用 json 模組。

## 十三、常見錯誤排查

**TypeError**

常見原因是把不相容的資料型別直接放在一起運算，例如字串加整數。先用 type() 查看型別，再決定是否轉換。

**ValueError**

常見於 int("abc") 這類無法轉換的資料。檢查輸入內容，並在後續課程學習 try/except。

**NameError**

可能是變數名稱拼錯，或在建立變數之前就使用它。Python 變數名稱區分大小寫。

## 十四、本課實作驗收

1. 建立 user_name、port、is_enabled 三個變數。
2. 使用 type() 查看它們的型別。
3. 用 input() 接收一個數字文字。
4. 把輸入轉換成 int。
5. 判斷輸入是否大於 100。
6. 使用 f-string 輸出完整結果。
7. 故意輸入 abc，觀察錯誤並說明原因。

## 十五、本課測驗

1. = 和 == 有什麼不同？
2. input() 回傳什麼資料型別？
3. int、float 和 str 分別保存什麼？
4. 7 // 2 和 7 % 2 的結果是什麼？
5. and 和 or 有什麼不同？
6. None 和 "None" 有什麼不同？
7. 為什麼不應使用 eval() 執行外部輸入？

## 十六、參考資料

- [Python 官方教學：Numbers、Strings 和其他型別](https://docs.python.org/3/tutorial/introduction.html)
- [Python 官方文件：Built-in Types](https://docs.python.org/3/library/stdtypes.html)
- [Python 官方文件：Built-in Functions](https://docs.python.org/3/library/functions.html)

**完成標準：** 能建立不同型別的變數，完成基本運算、輸入轉換和條件比較，並能解釋常見型別錯誤。
