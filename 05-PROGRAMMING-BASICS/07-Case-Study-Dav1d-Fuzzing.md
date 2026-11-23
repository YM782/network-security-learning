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

dav1d 是高效能 AV1 解碼器，已經被 OSS-Fuzz 等自動化測試環境測試。

研究員檢查現有 Fuzzer 的設定後，發現了一個值得注意的限制：

- 測試器限制了可處理的最大畫面尺寸。
- 某些正式使用者可以設定的參數沒有被充分測試。
- 多執行緒配置和輸入尺寸的組合會影響程式路徑。
- 測試環境的記憶體限制，也會縮小能探索的輸入範圍。

研究員調整 Fuzzer 配置，移除不適用於所有使用情境的尺寸限制，並擴大設定參數的測試範圍。

在多執行緒測試及 UBSan 的協助下，整數溢位問題被觸發。

## 四、漏洞根因：整數溢位如何變成越界寫入？

這個案例中的核心問題是：

1. 解碼器根據影像分割資訊計算 tile 起始偏移。
2. 某個計算使用有號 32 位元整數。
3. 特定尺寸和設定組合讓計算結果超出可表示範圍。
4. 整數溢位後，錯誤偏移被傳入後續程式。
5. 後續程式根據錯誤偏移計算指標。
6. 這些指標被用來寫入資料，形成越界寫入。

概念示意：

    輸入尺寸與解碼設定
             |
             v
       計算資料偏移
             |
             v
       整數超出範圍
             |
             v
       得到錯誤偏移
             |
             v
       計算錯誤指標
             |
             v
       越界記憶體寫入

不要把這個流程簡化成「數字太大就一定能攻擊」。實際影響需要確認資料型別、編譯器行為、記憶體配置、執行路徑和程式緩解措施。

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
2. [Project Zero：This Shouldn't Have Happened — Vulnerability Postmortem](https://projectzero.google/2021/12/this-shouldnt-have-happened.html)
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
