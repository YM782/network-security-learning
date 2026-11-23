# 05-07｜真實漏洞研究案例：Project Zero 如何透過 Fuzzing 找到 dav1d 漏洞

> 難度：初級至中級｜建議時間：120–150 分鐘｜先修：Python 基礎、整數、函式、檔案和錯誤處理
>
> 這個案例可以讓你理解一個重要事實：一個專案即使已經有自動化 Fuzzing，也不代表所有重要程式路徑都被測試到了。

## 一、真實案例

2024 年 10 月，Google Project Zero 公開 Nick Galloway 對 dav1d AV1 解碼器的研究。

研究發現一個整數溢位（Integer Overflow），可能導致越界寫入（Out-of-Bounds Write），被編號為 CVE-2024-1580。

原始文章：

[Effective Fuzzing: A Dav1d Case Study — Project Zero](https://projectzero.google/2024/10/effective-fuzzing-dav1d-case-study.html)

## 二、先理解 Fuzzing 是什麼？

Fuzzing（模糊測試）是一種自動化測試方法。

它會不斷產生或修改輸入資料，交給目標程式處理，觀察是否出現：
- 程式崩潰。
- AddressSanitizer 等工具回報記憶體錯誤。
- UndefinedBehaviorSanitizer 回報未定義行為。
- 非預期輸出。
- 超時或資源異常。

簡化流程：

    種子測試資料
          |
          v
      修改輸入
          |
          v
       目標程式
          |
          v
      觀察執行結果
          |
          v
    崩潰或異常？
       /      \
      是       否
      |        |
   保存樣本   繼續探索

Fuzzer 不是隨機按鍵工具。好的 Fuzzer 會結合程式覆蓋率、輸入格式、種子資料和執行配置。

## 三、為什麼 dav1d 已經有 Fuzzer，仍然會漏掉漏洞？

這篇研究最有價值的地方，不只是「Fuzzer 找到一個整數溢位」，而是研究員先檢查了既有 Fuzzer 的配置，發現它沒有覆蓋所有正式使用者可能採用的設定組合。

Project Zero 原文指出幾個具體限制：

1. **執行緒數量有限。** 當時 Fuzzer 的配置主要使用單執行緒或四個執行緒；如果程式問題只在其他執行緒配置下出現，原本配置就可能漏掉它。
2. **影格大小被人為限制。** OSS-Fuzz 的 dav1d Fuzzer 使用 `DAV1D_FUZZ_MAX_SIZE = 4096 * 4096`，並把這個值設為 `frame_size_limit`。
3. **消費者配置比 Fuzzer 更廣。** 這個尺寸上限並非 dav1d 所有正式使用情境都會套用；原文指出 64 位元平台沒有相同的預設上限。
4. **設定值本身沒有充分被 fuzz。** 例如 `output_invisible_frames` 等使用者可設定選項，在原本 Fuzzer 中並未涵蓋所有可能值。

研究員移除 Fuzzer 中不適用於所有正式使用情境的影格尺寸限制，使用多執行緒配置並啟用 UBSan，便觸發了整數溢位。之後又擴大設定值 fuzzing，讓測試器探索更多正式使用者可能採用的組合。

**這裡的教訓不是「把所有限制都刪掉」。** 測試環境可能因記憶體預算而設限制；研究員要做的是辨認哪些限制會把真實產品的有效輸入排除在測試空間之外，並設計額外測試去覆蓋它們。

## 四、漏洞根因：整數溢位如何變成越界寫入？

Project Zero 的技術分析指出，當 dav1d 使用多個解碼執行緒時，計算 tile 起始偏移（tile start offset）陣列的運算可能發生**有號 32 位元整數溢位**。

簡化資料流如下：

1. 解碼器根據影格分割資訊計算 `tile_start_off`。
2. 其中一段乘加運算超出 signed 32-bit integer 可表示範圍。
3. 溢位後的錯誤偏移被傳入 `setup_tile()`。
4. `setup_tile()` 再用這個偏移計算 `pal_idx`、`cbi` 和 `cf` 等指標位置。
5. 後續程式透過這些指標寫入資料，形成越界寫入（Out-of-Bounds Write）。

原始文章列出了具體的 `tile_start_off` 計算式和 `setup_tile()` 中的指標運算。閱讀原文時，建議先找出「整數溢位發生在哪個運算」，再沿著變數傳遞追到「錯誤偏移在哪裡被轉成指標」，最後確認「指標在哪個程式路徑被寫入」。

這比只記住「整數溢位會導致越界寫入」更重要，因為漏洞研究需要把資料流完整串起來。

CVE-2024-1580 已在 dav1d 1.4.0 修補。實際使用的 Linux 發行版可能 backport 修補，因此不能只靠套件版本字串判斷；要核對發行版安全公告。

## 五、研究員真正值得學習的思路

### 1. 不要只問 Fuzzer 跑了多少小時

應該問：
- 它使用了哪些種子資料？
- 它能走到哪些程式碼路徑？
- 它測試了哪些參數？
- 它有沒有測試多執行緒？
- 它是否限制了輸入尺寸？
- 它是否只在一種 CPU 架構執行？
- 它是否啟用了 Sanitizer？

### 2. 測試配置本身也可能形成盲區

一個 Fuzzer 的測試空間由多個部分共同決定：

    測試引擎
       +
    種子資料
       +
    輸入格式
       +
    執行配置
       +
    架構和編譯選項
       +
    資源限制
       =
    實際被探索的測試空間

如果某個正式使用情境不在測試配置中，Fuzzer 就可能永遠到不了那條路徑。

### 3. 覆蓋率不是安全保證

Code Coverage（程式碼覆蓋率）表示測試執行到哪些程式碼，不等於證明程式沒有漏洞。

覆蓋率高，也可能因為：
- 輸入範圍太小。
- 測試參數固定。
- 特定架構沒有執行。
- 多執行緒路徑沒有被觸發。
- 測試器只建立物件，沒有真正使用物件。
- 某些極端尺寸被限制。

## 六、初學者實驗：用 Python 檢查整數邊界

Python 的整數可以自動擴大精度，不會像一般固定寬度的 C 整數一樣直接發生相同的溢位。因此這個練習是學習輸入邊界檢查，不是重現 dav1d 漏洞。

建立 boundary_check.py：

    INT32_MAX = 2 ** 31 - 1

    def checked_buffer_size(width, height, bytes_per_pixel):
        if width <= 0 or height <= 0 or bytes_per_pixel <= 0:
            raise ValueError("Dimensions must be positive")

        size = width * height * bytes_per_pixel

        if size > INT32_MAX:
            raise OverflowError("Size exceeds signed 32-bit range")

        return size

    test_cases = [
        (1920, 1080, 4),
        (8192, 8192, 4),
        (50000, 50000, 4)
    ]

    for width, height, bpp in test_cases:
        try:
            size = checked_buffer_size(width, height, bpp)
            print(width, height, "=>", size)
        except (ValueError, OverflowError) as error:
            print(width, height, "=> rejected:", error)

執行：

    python3 boundary_check.py

在 Windows PowerShell 可以使用：

    py boundary_check.py

你應該觀察到：
- 一般尺寸可以通過。
- 超出設定的 32 位元上限時會被拒絕。
- 不合法的零或負數尺寸也會被拒絕。

這只是輸入驗證示範。真實 C/C++ 程式需要在正確的型別和運算位置進行溢位檢查，不能依賴呼叫端的 Python 檢查。

## 七、如何開始閱讀一個真實 Fuzzer？

讀原始專案時，按以下順序：

1. 找到 Fuzzer 的入口函式。
2. 看它如何取得輸入資料。
3. 看它如何建立目標程式的設定。
4. 看它如何呼叫解析器或解碼器。
5. 看它是否有設定輸入大小上限。
6. 看它是否啟用多執行緒。
7. 看它是否使用 ASan、UBSan 等工具。
8. 看它保存崩潰樣本和縮減測試案例的方式。

不要一開始就讀整個大型 C 專案。先找測試入口和設定檔。

## 八、Sanitizer 是什麼？

Sanitizer 是編譯器提供的一組動態檢查工具。

- AddressSanitizer（ASan）：協助發現越界存取和部分記憶體生命週期錯誤。
- UndefinedBehaviorSanitizer（UBSan）：協助發現部分未定義行為，例如某些整數溢位情況。
- ThreadSanitizer（TSan）：協助發現資料競爭。

不同 Sanitizer 有不同用途和成本。它們不是萬能的漏洞掃描器，也不會證明沒有漏洞。

## 九、從這個案例建立自己的研究清單

研究任何解析器、影像處理器、壓縮程式或檔案格式時，記錄：

| 問題 | 你要檢查的內容 |
|---|---|
| 輸入格式 | 程式接受什麼資料？ |
| 尺寸限制 | 是否有人為上限？ |
| 數字型別 | 長度和偏移使用什麼型別？ |
| 邊界條件 | 0、1、最大值、超大值如何處理？ |
| 多執行緒 | 不同執行緒配置是否走不同路徑？ |
| 架構 | x86-64、ARM 等路徑是否不同？ |
| 測試覆蓋 | 哪些設定沒有被測？ |
| Sanitizer | 是否能發現記憶體或未定義行為？ |

## 十、本課延伸閱讀

1. [Project Zero：Effective Fuzzing — Dav1d Case Study](https://projectzero.google/2024/10/effective-fuzzing-dav1d-case-study.html)
2. [Project Zero：This Shouldn't Have Happened — NSS BigSig（CVE-2021-43527）](https://projectzero.google/2021/12/this-shouldnt-have-happened.html) — 另一個獨立案例，研究既有 Fuzzer 為何漏掉缺少長度檢查的記憶體錯誤，重點是端到端測試、輸入尺寸限制和覆蓋率指標。
3. [Google OSS-Fuzz](https://github.com/google/oss-fuzz)
4. [LLVM LibFuzzer](https://llvm.org/docs/LibFuzzer.html)
5. [AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html)
6. [UndefinedBehaviorSanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html)

## 十一、本課驗收

- [ ] 能解釋 Fuzzing 的基本流程。
- [ ] 能說明種子資料、覆蓋率和測試配置。
- [ ] 能解釋為什麼 Fuzzer 已存在仍可能漏掉漏洞。
- [ ] 能說明整數溢位如何影響偏移和指標計算。
- [ ] 能執行 Python 邊界檢查練習。
- [ ] 能說明 ASan 和 UBSan 的差別。
- [ ] 能閱讀一個 Fuzzer 的入口函式和設定。

**本課最重要的一句話：** 漏洞研究不只是讓程式跑更多次，而是設計測試，讓程式走進以前沒有被測試過的路徑。
