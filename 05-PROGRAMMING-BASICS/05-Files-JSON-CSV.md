# 05-05｜檔案、JSON、CSV 和例外處理

> 難度：初級至初中級｜建議時間：100–120 分鐘｜先修：05-01 至 05-04

## 一、本課學習成果

完成後，你應該能夠：
1. 使用 pathlib 定位檔案。
2. 使用 with 和 open() 安全讀取文字。
3. 寫入和追加測試檔案。
4. 使用 json 模組讀取和寫入 JSON。
5. 使用 csv 模組讀取 CSV。
6. 使用 try/except 處理常見錯誤。
7. 理解編碼、檔案路徑和敏感資料保護。

## 二、為什麼安全人員需要讀取檔案？

日常安全工作經常遇到：
- 應用程式日誌。
- 防火牆事件紀錄。
- 匯出的掃描結果。
- JSON 格式的 API 回應。
- CSV 格式的資產清單。
- 設定檔和實驗報告。

程式可以幫助我們重複處理資料，但前提是先理解資料格式和來源。真實日誌可能含有個人資料、內部 IP、帳號和 Token，本課只使用自己建立的虛構檔案。

## 三、使用 pathlib 處理路徑

pathlib 是 Python 標準函式庫，提供跨平台的檔案路徑操作方式。

    from pathlib import Path

    lab_folder = Path("python-lab")
    print(lab_folder)

建立一個 Path 物件並不代表資料夾已經存在。

取得目前工作目錄：

    from pathlib import Path

    print(Path.cwd())

檢查檔案是否存在：

    file_path = Path("sample.log")
    print(file_path.exists())

相對路徑通常以程式執行時的目前工作目錄為基準，不一定是 .py 檔案所在目錄。若要以程式檔所在位置為基準，可以使用：

    from pathlib import Path

    BASE_DIR = Path(__file__).resolve().parent
    file_path = BASE_DIR / "sample.log"

這種方式在不同工作目錄執行同一個腳本時更穩定。

## 四、讀取文字檔

先建立 sample.log，內容如下：

    2026-11-23 INFO application started
    2026-11-23 WARNING disk space is low
    2026-11-23 ERROR test connection failed

建立 read_log.py：

    from pathlib import Path

    file_path = Path(__file__).resolve().parent / "sample.log"

    with file_path.open("r", encoding="utf-8") as file:
        for line in file:
            print(line.strip())

解釋：
- Path(__file__)：目前 Python 程式檔案的路徑。
- resolve()：取得解析後的路徑。
- parent：取得程式檔所在資料夾。
- open("r")：以讀取模式開啟。
- encoding="utf-8"：明確指定文字編碼。
- with：結束區塊後自動關閉檔案。
- strip()：移除字串前後空白，包括換行符號。

## 五、寫入和追加文字檔

寫入模式：

    from pathlib import Path

    output_file = Path("result.txt")

    with output_file.open("w", encoding="utf-8") as file:
        file.write("First line\n")
        file.write("Second line\n")

w 模式會建立檔案，若檔案已存在則會覆蓋內容。

追加模式：

    with output_file.open("a", encoding="utf-8") as file:
        file.write("Third line\n")

a 模式會把內容加到檔案末尾。

執行前要確認檔案位置和模式。若目標檔案重要，不要用 w 模式進行不必要的測試。

## 六、JSON 是什麼？

JSON（JavaScript Object Notation）是常見的資料交換格式。

示例：

    {
      "device": "lab-kali",
      "ip": "192.0.2.10",
      "port": 443,
      "enabled": true,
      "tags": ["practice", "local"]
    }

JSON 支援字串、數字、布林值、null、陣列和物件等資料型別。JSON 的 true、false、null 和 Python 的 True、False、None 寫法不同。

## 七、使用 json.load() 讀取 JSON

建立 device.json，內容使用上一節的 JSON 範例。

Python 程式：

    import json
    from pathlib import Path

    file_path = Path(__file__).resolve().parent / "device.json"

    with file_path.open("r", encoding="utf-8") as file:
        data = json.load(file)

    print(data["device"])
    print(data["ip"])
    print(data["tags"])

json.load() 從檔案讀取 JSON 並轉換成 Python 資料結構。JSON 物件通常轉成 dict，陣列通常轉成 list。

## 八、使用 json.dump() 寫入 JSON

    import json
    from pathlib import Path

    data = {
        "device": "lab-kali",
        "purpose": "local practice",
        "enabled": True
    }

    output_file = Path("device-output.json")

    with output_file.open("w", encoding="utf-8") as file:
        json.dump(data, file, ensure_ascii=False, indent=2)

- dump()：把 Python 資料寫入檔案。
- ensure_ascii=False：讓非 ASCII 字元可直接以文字形式保存。
- indent=2：使用兩格縮排，方便閱讀。

寫入前請確認輸出路徑，避免覆蓋重要檔案。

## 九、CSV 是什麼？

CSV（Comma-Separated Values，逗號分隔值）常用來保存表格資料。

示例：

    name,port,protocol
    HTTP,80,TCP
    HTTPS,443,TCP
    DNS,53,UDP

使用 csv 模組讀取：

    import csv
    from pathlib import Path

    csv_file = Path("services.csv")

    with csv_file.open("r", encoding="utf-8", newline="") as file:
        reader = csv.DictReader(file)

        for row in reader:
            print(row["name"], row["port"], row["protocol"])

DictReader 會把標題列當作欄位名稱，每一列讀成字典。

CSV 欄位不一定都是數字；如果要做數學運算，需要自行驗證並轉換。

## 十、例外處理 try/except

程式可能遇到檔案不存在、權限不足、編碼錯誤或 JSON 格式錯誤。

例如：

    from pathlib import Path

    file_path = Path("missing.txt")

    try:
        text = file_path.read_text(encoding="utf-8")
        print(text)
    except FileNotFoundError:
        print("File does not exist")
    except PermissionError:
        print("Permission denied")

try 包含可能發生錯誤的程式；except 定義遇到特定錯誤時的處理方式。

不要使用沒有必要的空白 except: 把所有錯誤吞掉，否則程式可能看似正常，實際上卻漏掉重要問題。

## 十一、處理 JSON 格式錯誤

    import json

    raw_text = '{"device": "lab-kali"'

    try:
        data = json.loads(raw_text)
        print(data)
    except json.JSONDecodeError as error:
        print("Invalid JSON:", error)

loads() 從字串解析 JSON；load() 從檔案物件讀取 JSON。

當資料來源不可信時，必須假設格式可能錯誤或內容不完整。

## 十二、完整小實驗：讀取服務清單並輸出 JSON

建立 services.csv：

    name,port,protocol
    HTTP,80,TCP
    HTTPS,443,TCP
    DNS,53,UDP

建立 convert_services.py：

    import csv
    import json
    from pathlib import Path

    base_dir = Path(__file__).resolve().parent
    input_file = base_dir / "services.csv"
    output_file = base_dir / "services.json"

    services = []

    with input_file.open("r", encoding="utf-8", newline="") as file:
        reader = csv.DictReader(file)

        for row in reader:
            services.append({
                "name": row["name"],
                "port": int(row["port"]),
                "protocol": row["protocol"]
            })

    with output_file.open("w", encoding="utf-8") as file:
        json.dump(services, file, ensure_ascii=False, indent=2)

    print(f"Converted {len(services)} services")

執行後應該會產生 services.json，裡面包含三筆服務資料。

這個實驗只轉換自己建立的本地測試資料，不會連接網路。

## 十三、常見錯誤排查

**FileNotFoundError**

先確認目前工作目錄、檔名、大小寫和副檔名。使用 Path(__file__).resolve().parent 可以避免部分相對路徑問題。

**PermissionError**

確認目前使用者是否有讀取或寫入該路徑的權限。不要第一時間使用管理員權限執行整個程式。

**UnicodeDecodeError**

檔案實際編碼可能不是 UTF-8。先確認資料來源和編碼，不要盲目使用錯誤的編碼讀取。

**JSONDecodeError**

檢查 JSON 的引號、逗號、括號和 true/false/null 寫法。JSON 不允許註解，也不能在物件最後一項後面留下多餘逗號。

**KeyError**

JSON 或 CSV 中不存在程式要求的欄位。先查看實際欄位名稱，再使用 get() 或適當的欄位檢查。

## 十四、資料保護注意事項

- 不要把真實登入日誌、使用者個資和公司內部資料上傳公開倉庫。
- 不要把密碼、API Token 或 Cookie 硬編碼進程式。
- 不要對來源不明的資料直接使用 pickle 反序列化。
- 不要把讀取檔案的路徑直接交由不可信使用者控制，否則可能產生路徑穿越或未授權檔案存取風險。
- 練習時只使用自己的虛構檔案。

## 十五、本課實作驗收

1. 建立 sample.log 並用 Python 逐行讀取。
2. 建立 device.json 並使用 json.load() 讀取。
3. 使用 json.dump() 輸出另一個 JSON 檔案。
4. 建立 services.csv 並用 DictReader 讀取。
5. 故意指定不存在的檔案，使用 FileNotFoundError 處理。
6. 故意讀取格式錯誤的 JSON，使用 JSONDecodeError 處理。
7. 解釋 w 和 a 模式的不同。

## 十六、本課測驗

1. pathlib 的用途是什麼？
2. with 有什麼好處？
3. r、w 和 a 分別代表什麼？
4. json.load() 和 json.loads() 有什麼不同？
5. csv.DictReader() 會產生什麼資料？
6. FileNotFoundError 和 JSONDecodeError 分別代表什麼？
7. 為什麼不應把真實 Token 寫進程式碼或公開日誌？

## 十七、參考資料

- [Python 官方文件：pathlib](https://docs.python.org/3/library/pathlib.html)
- [Python 官方文件：open()](https://docs.python.org/3/library/functions.html#open)
- [Python 官方文件：json](https://docs.python.org/3/library/json.html)
- [Python 官方文件：csv](https://docs.python.org/3/library/csv.html)
- [Python 官方文件：Exceptions](https://docs.python.org/3/tutorial/errors.html)

**完成標準：** 能讀取文字檔、JSON 和 CSV，輸出整理後的 JSON，並能處理檔案不存在和資料格式錯誤。
