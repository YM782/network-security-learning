# Programming for Cybersecurity（網路安全程式設計基礎）

網路安全工程師不一定每天都要開發大型軟體，但必須能閱讀程式、理解資料流、修改範例並撰寫小型自動化工具。本課程先掌握 Python，再學習 Bash、PowerShell、SQL 與 C 語言基礎。

## 1. Python（Python 程式語言）

### Core Topics（核心主題）

1. Variables and data types（變數與資料型態）：int、float、str、bool。
2. Conditions and loops（條件與迴圈）：if、for、while。
3. Functions（函式）：參數、回傳值與作用範圍。
4. Data structures（資料結構）：list、tuple、dict、set。
5. Files（檔案處理）：讀取文字、JSON、CSV。
6. Exceptions（例外處理）：try、except、finally。
7. Modules and packages（模組與套件）：import、pip、虛擬環境。
8. Regular expressions（正規表示式）：搜尋與擷取文字。
9. Networking basics（網路程式設計基礎）：socket、HTTP client。

### First Program（第一個程式）

```python
name = "student"
hours = 2

if hours >= 1:
    print(f"Hello, {name}. You studied for {hours} hour(s).")
else:
    print("Start with a short study session.")
```

### Lab 1 — Read a Local Log File（實驗一：讀取本機日誌）

建立一個名為 `sample.log` 的文字檔：

```text
2026-11-23 INFO service started
2026-11-23 ERROR connection failed
2026-11-23 WARNING retry scheduled
2026-11-23 ERROR timeout
```

建立 `count_errors.py`：

```python
from pathlib import Path

log_path = Path("sample.log")

if not log_path.exists():
    raise SystemExit("Log file not found.")

error_count = 0

for line in log_path.read_text(encoding="utf-8").splitlines():
    if "ERROR" in line:
        error_count += 1
        print(line)

print(f"Total ERROR lines: {error_count}")
```

執行：

```bash
python3 count_errors.py
```

預期結果：顯示兩筆 ERROR 日誌，最後輸出 `Total ERROR lines: 2`。

**學習重點：** 檔案路徑、文字編碼、迴圈、條件判斷與計數器。

## 2. Bash（Linux Shell 指令稿）

先掌握：
- 變數：`name="lab"`
- 輸出：`echo "$name"`
- 條件：`if ...; then ...; fi`
- 迴圈：`for item in ...; do ...; done`
- 管線：`command1 | command2`
- 重新導向：`>`、`>>`、`2>`

範例：

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Current user: $(whoami)"
echo "Current directory: $(pwd)"
echo "Disk usage:"
df -h
```

使用 `bash script.sh` 執行。對不熟悉的腳本，先閱讀內容，不要直接使用 root 權限執行。

## 3. PowerShell（Windows 自動化）

常用概念：
- Cmdlet（命令指令）：Verb-Noun，例如 Get-Process。
- Pipeline（管線）：將物件傳給下一個命令。
- Object（物件）：PowerShell 傳遞的資料通常是物件，不只是純文字。
- Properties（屬性）：例如 Name、Id、CPU。

範例：

```powershell
Get-Process |
  Sort-Object CPU -Descending |
  Select-Object -First 10 Name, Id, CPU
```

這會列出 CPU 累積時間較高的前 10 個程序。CPU 欄位不是即時百分比，解讀時要注意欄位意義。

## 4. SQL（資料庫查詢語言）

先學習：
- SELECT：查詢資料。
- WHERE：篩選資料。
- ORDER BY：排序。
- GROUP BY：分組。
- JOIN：關聯資料表。
- INSERT／UPDATE／DELETE：資料變更操作。

請先在自己的 SQLite 測試資料庫中練習，不要對正式資料庫執行不熟悉的修改語句。

範例：

```sql
SELECT username, role
FROM users
WHERE enabled = 1
ORDER BY username;
```

## 5. C Language（C 語言與記憶體概念）

網路安全學習 C 語言的重點：
- Data types（資料型態）。
- Pointers（指標）。
- Arrays（陣列）。
- Stack and heap（堆疊與堆積）。
- Memory allocation（記憶體配置）。
- Buffer boundaries（緩衝區邊界）。
- Undefined behavior（未定義行為）。

先在本機編譯並執行簡單程式，理解指標與記憶體位址，不要直接把不安全範例用於真實服務。

## 6. Troubleshooting（故障排查）

| 問題 | 檢查方向 |
|---|---|
| `python3: command not found` | 檢查 Python 是否安裝及 PATH |
| `FileNotFoundError` | 確認目前工作目錄與檔案路徑 |
| Python indentation error | 檢查縮排是否一致 |
| Permission denied | 檢查檔案權限與目前使用者 |
| PowerShell command not found | 確認 Cmdlet 名稱與模組是否存在 |
| SQL no such table | 確認目前連線的資料庫與資料表名稱 |

## Sources（資料來源）

- Python Tutorial: https://docs.python.org/3/tutorial/
- Python Library Reference: https://docs.python.org/3/library/
- Bash Reference Manual: https://www.gnu.org/software/bash/manual/
- Microsoft PowerShell Documentation: https://learn.microsoft.com/powershell/
- SQLite Documentation: https://www.sqlite.org/docs.html
- Learn C: https://www.learn-c.org/

本課程以自有電腦與本機測試資料為練習範圍。
