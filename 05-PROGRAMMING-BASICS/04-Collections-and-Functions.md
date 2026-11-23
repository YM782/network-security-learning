# 05-04｜清單、元組、字典、集合和函式

> 難度：初級｜建議時間：100–120 分鐘｜先修：05-01 至 05-03

## 一、本課學習成果

完成後，你應該能夠：
1. 使用 list 保存可變動的多筆資料。
2. 使用 tuple 保存固定結構的資料。
3. 使用 dict 以鍵和值組織資料。
4. 使用 set 去除重複項目。
5. 理解索引、切片和常見集合方法。
6. 定義函式、傳入參數和回傳結果。
7. 把重複工作拆成可重用的函式。

## 二、為什麼需要資料結構？

如果程式只有一個變數，只能保存一筆資料。安全工作經常需要處理大量事件、IP、連接埠、使用者或日誌紀錄，因此需要資料結構（Data Structures）。

Python 常見內建資料結構：
- list：清單。
- tuple：元組。
- dict：字典。
- set：集合。

## 三、list 清單

list 可以保存多筆資料，並且允許修改。

    services = ["HTTP", "HTTPS", "SSH"]

    print(services)
    print(services[0])
    print(services[1])

Python 索引從 0 開始，因此 services[0] 是第一個元素。

常見操作：

    services.append("DNS")
    services.remove("SSH")
    print(len(services))

- append()：在尾端新增元素。
- remove()：移除第一個符合的值。
- len()：取得元素數量。

索引超出範圍時會產生 IndexError。使用前要確認清單長度。

### 切片 Slicing

    numbers = [10, 20, 30, 40, 50]
    print(numbers[1:4])

結果是 20、30、40。切片的結束索引不包含在結果中。

## 四、tuple 元組

tuple 和 list 類似，但建立後不能直接修改其中元素。

    device = ("lab-kali", "192.0.2.10", "Linux")
    print(device[0])

tuple 適合保存固定欄位組合。若要變更資料，通常需要建立新的 tuple。

## 五、dict 字典

dict 使用鍵和值組織資料，非常適合表示一筆有欄位名稱的紀錄。

    device = {
        "name": "lab-kali",
        "ip": "192.0.2.10",
        "system": "Linux"
    }

    print(device["name"])
    print(device["ip"])

讀取不存在的鍵會產生 KeyError。若不確定鍵是否存在，可以使用 get()：

    print(device.get("owner", "unknown"))

這表示如果沒有 owner 欄位，就回傳 unknown。

新增或修改：

    device["owner"] = "student"
    device["ip"] = "192.0.2.20"

取得所有鍵值：

    for key, value in device.items():
        print(key, value)

## 六、set 集合

set 保存不重複的元素：

    ports = {80, 443, 22, 443, 80}
    print(ports)

重複值只會保留一份。set 不保證按照你輸入的順序顯示。

常見操作：

    ports.add(53)
    ports.discard(22)

- add()：加入元素。
- discard()：若元素存在就移除，不存在也不會因而報錯。

set 適合去重和集合運算，不適合依固定位置取值。

## 七、四種資料結構比較

| 型別 | 是否可修改 | 是否允許重複 | 典型用途 |
|---|---|---|---|
| list | 是 | 是 | 有順序的事件清單 |
| tuple | 否 | 是 | 固定欄位組合 |
| dict | 是 | 鍵唯一 | 以欄位名稱保存資料 |
| set | 是 | 否 | 去重和集合運算 |

## 八、函式 Function

函式可以把一段工作包裝起來，之後重複呼叫。

    def greet(user_name):
        print(f"Hello, {user_name}")

    greet("Michael")

- def：定義函式。
- greet：函式名稱。
- user_name：參數。
- greet("Michael")：呼叫函式並傳入引數。

## 九、return 回傳結果

print() 是顯示資料；return 是把結果交回呼叫函式的地方。兩者不同。

    def add_numbers(a, b):
        result = a + b
        return result

    total = add_numbers(10, 20)
    print(total)

函式回傳 30，呼叫端再把結果保存到 total。

## 十、參數和預設值

    def describe_service(name, port=443):
        return f"{name} uses port {port}"

    print(describe_service("HTTPS"))
    print(describe_service("HTTP", 80))

port 有預設值 443，因此第一次呼叫時可以省略第二個引數。

## 十一、把資料整理工作寫成函式

    def count_errors(events):
        total = 0

        for event in events:
            if event == "ERROR":
                total += 1

        return total

    sample_events = ["INFO", "ERROR", "WARNING", "ERROR"]
    result = count_errors(sample_events)
    print(result)

預期結果是 2。

把工作放進函式後，可以用不同的測試資料重複呼叫，減少複製貼上。

## 十二、變數作用範圍 Scope

在函式內建立的區域變數，通常只能在該函式內直接使用。

    def make_message():
        message = "Local message"
        return message

    print(make_message())

不要在沒有必要時大量使用 global 修改全域變數。把需要的資料透過參數傳入，再用 return 回傳結果，通常更容易測試和維護。

## 十三、完整練習：統計事件類型

    events = [
        {"level": "INFO", "message": "service started"},
        {"level": "ERROR", "message": "connection failed"},
        {"level": "WARNING", "message": "disk space low"},
        {"level": "ERROR", "message": "permission denied"}
    ]

    def count_by_level(records, target_level):
        count = 0

        for record in records:
            if record.get("level") == target_level:
                count += 1

        return count

    print("ERROR:", count_by_level(events, "ERROR"))
    print("INFO:", count_by_level(events, "INFO"))

預期輸出：

    ERROR: 2
    INFO: 1

這裡的 events 是程式內建立的虛構資料，不是真實系統日誌。

## 十四、常見錯誤排查

**IndexError**

清單索引不存在。先用 len() 查看長度，注意索引從 0 開始。

**KeyError**

字典中沒有指定鍵。可先用 key in dictionary 檢查，或使用 get()。

**TypeError: object is not callable**

可能把變數取名為 list、dict 或其他內建函式名稱，覆蓋了原本的內建名稱。避免用內建型別名稱當作變數名。

**函式回傳 None**

可能函式沒有執行 return。只有 print() 不代表函式已回傳可供後續運算的結果。

## 十五、本課實作驗收

1. 建立包含五個服務名稱的 list。
2. 使用 append() 新增一項。
3. 建立保存設備資訊的 dict。
4. 使用 get() 讀取不存在的欄位並提供預設值。
5. 建立 set，觀察重複元素如何處理。
6. 定義一個接收事件清單的函式。
7. 函式回傳 ERROR 數量，並由呼叫端印出結果。

## 十六、本課測驗

1. list 和 tuple 有什麼不同？
2. dict 為什麼適合保存設備欄位？
3. set 有什麼特點？
4. Python 清單索引從幾開始？
5. append() 和 remove() 分別做什麼？
6. print() 和 return 有什麼不同？
7. 為什麼函式參數和回傳值有助於程式維護？

## 十七、參考資料

- [Python 官方教學：Data Structures](https://docs.python.org/3/tutorial/datastructures.html)
- [Python 官方文件：Built-in Types](https://docs.python.org/3/library/stdtypes.html)
- [Python 官方教學：Defining Functions](https://docs.python.org/3/tutorial/controlflow.html#defining-functions)

**完成標準：** 能使用四種基本資料結構保存資料，並能寫出接收資料、處理資料和回傳結果的函式。
