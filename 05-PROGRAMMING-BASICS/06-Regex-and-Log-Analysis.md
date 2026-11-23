# 05-06｜正規表示式與離線日誌分析實驗

> 難度：初級至初中級｜建議時間：120 分鐘｜先修：05-01 至 05-05
>
> 這是本模組的整合實驗。你會使用 Python 的 re、collections、pathlib 和 ipaddress 標準函式庫，分析自己建立的虛構日誌。整個實驗只讀取本地測試檔案，不掃描設備，也不連接外部主機。

## 一、本課學習成果

完成後，你應該能夠：
1. 說明正規表示式（Regular Expression，Regex）的用途。
2. 使用 re.search() 和 re.findall() 搜尋文字。
3. 理解常見的正規表示式符號。
4. 使用 collections.Counter() 統計資料。
5. 使用 ipaddress 驗證 IPv4/IPv6 字串。
6. 逐行分析本地日誌並統計事件等級。
7. 說明為什麼日誌分析不能只靠簡單字串匹配判定攻擊。

## 二、正規表示式是什麼？

正規表示式是一種描述文字模式的語法，常用來搜尋、驗證或擷取文字。

例如，日誌可能包含：

    2026-11-23T09:02:45 ERROR permission denied user=bob ip=198.51.100.23

我們可能想找出：
- 事件等級：ERROR。
- 使用者欄位：bob。
- IP 字串：198.51.100.23。

如果資料量很多，逐行人工尋找會很慢。正規表示式可以幫助我們建立可重複使用的匹配規則。

## 三、常見 Regex 符號

| 符號 | 意義 | 簡單例子 |
|---|---|---|
| . | 匹配任意單一字元（預設不含換行） | a.c |
| \d | 數字 | \d+ |
| \w | 字母、數字或底線等文字字元 | \w+ |
| \s | 空白字元 | \s+ |
| + | 前一項至少出現一次 | \d+ |
| * | 前一項出現零次或多次 | ab* |
| ? | 前一項可有可無 | colou?r |
| {n} | 前一項重複 n 次 | \d{4} |
| [] | 字元集合 | [A-Z] |
| () | 群組 | (INFO|ERROR) |
| \b | 單字邊界 | \bERROR\b |
| ^ | 字串或行開頭 | ^2026 |
| $ | 字串或行結尾 | failed$ |

Regex 的實際匹配行為會受旗標和上下文影響。初學時先用小型測試字串驗證模式。

## 四、Python 的 re 模組

re 是 Python 標準函式庫，不需要另外安裝。

### 1. re.search()

    import re

    line = "2026-11-23 ERROR login failed"
    match = re.search(r"ERROR", line)

    if match:
        print("Found an error")

search() 會在字串中尋找第一個匹配位置，找到時回傳 Match 物件，否則回傳 None。

### 2. re.findall()

    import re

    text = "INFO ERROR WARNING ERROR"
    results = re.findall(r"ERROR", text)
    print(results)

findall() 會回傳所有匹配結果組成的清單。

### 3. 為什麼常用 r 前綴？

    pattern = r"\d+"

r 表示 raw string，能讓反斜線在 Python 字串中更容易表達 Regex 語法。這在包含很多反斜線的模式中尤其有用。

## 五、匹配日誌等級

    import re

    line = "2026-11-23T09:02:45 ERROR permission denied"
    match = re.search(r"\b(INFO|WARNING|ERROR)\b", line)

    if match:
        print(match.group(1))

這個模式尋找 INFO、WARNING 或 ERROR，並用括號把匹配到的等級保存成群組。

## 六、擷取 IP 字串和驗證 IP 位址

先用 Regex 找出類似 IPv4 的文字：

    import re

    line = "client ip=198.51.100.23"
    candidates = re.findall(r"\b(?:\d{1,3}\.){3}\d{1,3}\b", line)
    print(candidates)

注意：這個模式只能找出「外觀像四組數字」的字串，不保證它是真正有效的 IPv4 位址。

例如 999.999.999.999 也可能符合這種簡單模式。

因此再使用 ipaddress 驗證：

    import ipaddress

    candidate = "198.51.100.23"

    try:
        address = ipaddress.ip_address(candidate)
        print(address.version)
    except ValueError:
        print("Invalid IP address")

ipaddress 會檢查位址格式是否合法，並可協助判斷 IPv4 或 IPv6。

本課使用的 192.0.2.0/24、198.51.100.0/24 和 203.0.113.0/24 都是文件示例保留網段，不代表真實客戶或伺服器。

## 七、建立虛構測試日誌

在本課程資料夾建立 sample_security.log，內容如下：

    2026-11-23T09:00:00 INFO login success user=alice ip=192.0.2.10
    2026-11-23T09:01:12 WARNING failed login user=bob ip=198.51.100.23
    2026-11-23T09:02:45 ERROR permission denied user=bob ip=198.51.100.23
    2026-11-23T09:03:10 INFO session closed user=alice ip=192.0.2.10
    2026-11-23T09:04:20 WARNING invalid record ip=999.999.999.999

這些資料完全是虛構的，最後一行刻意放入不合法的 IP 字串，供驗證程式測試。

## 八、完整程式：離線日誌統計器

建立 analyze_log.py，輸入以下程式：

    from collections import Counter
    from pathlib import Path
    import ipaddress
    import re

    base_dir = Path(__file__).resolve().parent
    log_file = base_dir / "sample_security.log"

    level_pattern = re.compile(r"\b(INFO|WARNING|ERROR)\b")
    ip_pattern = re.compile(r"\b(?:\d{1,3}\.){3}\d{1,3}\b")

    level_counts = Counter()
    ip_counts = Counter()
    invalid_ips = []

    with log_file.open("r", encoding="utf-8") as file:
        for line_number, line in enumerate(file, start=1):

            level_match = level_pattern.search(line)

            if level_match:
                level = level_match.group(1)
                level_counts[level] += 1

            candidates = ip_pattern.findall(line)

            for candidate in candidates:
                try:
                    address = ipaddress.ip_address(candidate)
                    ip_counts[str(address)] += 1
                except ValueError:
                    invalid_ips.append((line_number, candidate))

    print("=== Event Level Summary ===")

    for level in ["INFO", "WARNING", "ERROR"]:
        print(f"{level}: {level_counts[level]}")

    print("\n=== Valid IP Summary ===")

    for address, count in ip_counts.most_common():
        print(f"{address}: {count}")

    print("\n=== Invalid IP Candidates ===")

    for line_number, candidate in invalid_ips:
        print(f"Line {line_number}: {candidate}")

## 九、逐段理解程式

### 1. 匯入標準函式庫

- Counter：統計每個項目出現的次數。
- Path：處理檔案路徑。
- ipaddress：驗證和操作 IP 位址。
- re：使用正規表示式匹配文字。

這些都是 Python 標準函式庫，不需要額外安裝。

### 2. 建立檔案路徑

    base_dir = Path(__file__).resolve().parent
    log_file = base_dir / "sample_security.log"

程式會以 analyze_log.py 所在資料夾為基準，尋找同一資料夾中的測試日誌。

### 3. 建立 Regex 模式

    level_pattern = re.compile(r"\b(INFO|WARNING|ERROR)\b")
    ip_pattern = re.compile(r"\b(?:\d{1,3}\.){3}\d{1,3}\b")

第一條尋找事件等級，第二條尋找外觀像 IPv4 的文字。

第二條並不負責完整驗證 IP，所以後續仍需要 ipaddress。

### 4. 建立計數器

    level_counts = Counter()
    ip_counts = Counter()

Counter 可以把每種等級或位址的出現次數累加，不需要自己建立複雜的計數邏輯。

### 5. 逐行讀取檔案

    for line_number, line in enumerate(file, start=1):

enumerate() 同時提供行號和行內容。start=1 讓行號從 1 開始，符合一般文字編輯器的閱讀習慣。

### 6. 找出事件等級

    level_match = level_pattern.search(line)

若有匹配，就用 group(1) 取得括號中的等級，並增加計數。

### 7. 驗證 IP

Regex 只負責找出候選字串；ipaddress.ip_address() 再判斷格式是否合法。

遇到 ValueError 時，程式把行號和無效字串記錄到 invalid_ips，而不是直接停止整個分析。

## 十、預期輸出

    === Event Level Summary ===
    INFO: 2
    WARNING: 2
    ERROR: 1

    === Valid IP Summary ===
    192.0.2.10: 2
    198.51.100.23: 2

    === Invalid IP Candidates ===
    Line 5: 999.999.999.999

如果結果不同，先確認測試日誌內容、檔案路徑和程式是否保存完整。

## 十一、這個工具能做什麼，不能做什麼？

能做：
- 統計指定日誌中的 INFO、WARNING 和 ERROR。
- 找出外觀像 IPv4 的字串。
- 驗證候選 IP 的格式。
- 顯示無效 IP 的行號。
- 作為後續日誌分析的基礎程式。

不能做：
- 單憑某個 IP 出現次數就判定攻擊者。
- 單憑 ERROR 就判定系統遭到入侵。
- 證明某個帳號真的被盜。
- 代替 SIEM、EDR 或專業事件調查。
- 判斷日誌內容是否完整、可信或被竄改。

日誌分析必須結合時間、來源、系統背景、使用者行為和其他證據。

## 十二、常見錯誤排查

**FileNotFoundError**

確認 sample_security.log 和 analyze_log.py 是否位於同一資料夾，並檢查檔名是否拼錯。

**Regex 沒有匹配到結果**

先把模式縮小，例如只搜尋 ERROR，再逐步增加群組和邊界。注意大小寫和空白。

**不合法 IP 沒有被排除**

確認程式不只是執行 Regex，還有呼叫 ipaddress.ip_address()，並捕捉 ValueError。

**輸出計數為零**

檢查日誌等級拼寫是否與模式一致，並確認讀取的是正確檔案。

## 十三、進階練習

完成基本程式後，再嘗試：
1. 增加 CRITICAL 等級。
2. 統計每個使用者出現的次數。
3. 只顯示 ERROR 所在的完整行。
4. 把統計結果輸出到 summary.json。
5. 加入日期篩選，但先使用固定格式的虛構日期。
6. 將程式拆成 read_log()、parse_line() 和 print_summary() 三個函式。

## 十四、本課測驗

1. Regex 和一般字串搜尋有什麼不同？
2. re.search() 和 re.findall() 有什麼差別？
3. 為什麼 Regex 找到的 IPv4 字串仍需要 ipaddress 驗證？
4. Counter 的用途是什麼？
5. enumerate() 為什麼適合用來分析日誌？
6. 為什麼不能只憑 ERROR 數量判定系統遭到入侵？
7. 本課的測試資料為什麼使用文件示例保留 IP 網段？

## 十五、參考資料

- [Python 官方文件：re](https://docs.python.org/3/library/re.html)
- [Python 官方文件：collections.Counter](https://docs.python.org/3/library/collections.html#collections.Counter)
- [Python 官方文件：ipaddress](https://docs.python.org/3/library/ipaddress.html)
- [IETF RFC 5737：IPv4 Address Blocks Reserved for Documentation](https://www.rfc-editor.org/rfc/rfc5737)

**完成標準：** 能獨立執行離線日誌統計器，逐段解釋 Regex、Counter 和 ipaddress 的作用，並能說明工具的限制。
