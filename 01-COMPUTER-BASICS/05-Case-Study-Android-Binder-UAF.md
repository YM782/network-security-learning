# 01-05｜真實漏洞研究案例：Android Binder 的 Use-After-Free（CVE-2019-2215）

> 難度：初級至中級｜建議時間：90–120 分鐘｜先修：本模組 01–03
>
> 本文不是教你攻擊 Android 手機，而是帶你閱讀一份真實的漏洞研究紀錄，學會研究人員如何從線索縮小範圍、理解記憶體物件生命週期、確認漏洞根因，以及追蹤修補是否真正到達使用者設備。

## 一、先看真實事件

2019 年，Google Project Zero 公開分析了 Android 核心 Binder 驅動程式中的 CVE-2019-2215。它屬於 Use-After-Free（UAF，釋放後使用）漏洞，可造成權限提升。

這個案例特別值得初學者研究，因為它不是單純的「程式寫錯一行」：

- 問題位於 Android Linux Kernel 的 Binder 驅動程式。
- 受影響的核心物件在釋放後，仍有其他元件保留參照。
- Project Zero 不是盲目搜尋所有核心程式碼，而是先從設備版本、核心修補紀錄和已知線索縮小範圍。
- 這個問題早在 2017 年已由 syzkaller 找到並回報，後來雖然有上游修補，部分已上市設備卻沒有及時取得安全更新。

先讀原始文章：[Bad Binder: Android In-The-Wild Exploit — Google Project Zero](https://projectzero.google/2019/11/bad-binder-android-in-wild-exploit.html)

## 二、先補足必要的底層知識

### 1. 程序和核心不是同一層

一般應用程式在使用者空間（User Space）執行。作業系統核心（Kernel）負責管理程序、記憶體、裝置和系統呼叫。

Android 應用程式需要透過 Binder 與其他程序或系統服務溝通。Binder 驅動程式位於核心層，因此它的錯誤可能影響整個系統的安全邊界。

### 2. 什麼是指標 Pointer？

指標可以理解為保存某個記憶體位置的資料。程式透過指標存取物件或結構。

指標本身不代表它指向的物件一定還存在。當物件被釋放後，其他程式碼如果仍使用舊指標，就可能發生嚴重問題。

### 3. 什麼是 Use-After-Free？

假設程式建立一個物件：

    object = allocate()
    use(object)
    free(object)

free(object) 之後，這塊記憶體已經不再屬於原本物件的有效生命週期。

如果其他程式碼仍保留舊參照，並再次讀取或寫入：

    use(old_reference)

就可能形成 Use-After-Free。

這段只是概念示意，不是可執行的漏洞利用程式。

### 4. 為什麼它危險？

被釋放的記憶體可能被其他資料重新使用。舊參照因此可能指向已改作他用的內容。

結果可能包括：
- 程式崩潰。
- 資料結構損壞。
- 非預期的記憶體讀寫。
- 在特定條件下造成權限提升或程式碼執行。

是否能造成更高影響，必須根據實際環境、緩解措施和可控條件判斷，不能只看到 UAF 就直接宣稱可以取得 root。

## 三、Binder 案例的真正根因

Project Zero 的技術分析把問題定位在 Binder 驅動程式中的物件生命週期：

1. `binder_thread` 結構內嵌一個 `wait_queue_head_t wait`。
2. Binder 的 `binder_poll()` 會把這個 wait queue 提供給 `epoll` 監看。
3. 當執行 `BINDER_THREAD_EXIT` ioctl 時，`binder_thread_release()` 會釋放包含 wait queue 的 `binder_thread`。
4. 但 epoll 監看項目仍可能保留指向該 wait queue 的參照。
5. 後續移除 epoll watch 時，清理路徑會呼叫 `remove_wait_queue()`，其參數仍指向已釋放的 wait queue。
6. KASAN 報告的 UAF 發生在 `remove_wait_queue()` 使用已失效的 `q`，例如存取 `q->lock`。

更精確地說，這不是「epoll 一般都會保留失效指標」；問題在於 Binder 提供給 poll/epoll 的 wait queue，其生命週期沒有依照核心 poll wait queue 的預期規則處理。Project Zero 指出，Binder 的這條路徑需要妥善處理 `POLLFREE` 等生命週期通知。

**研究重點：** 不只是「物件被 free 了」，還要找出哪個子系統持有參照、哪個清理路徑會再次使用它，以及物件釋放前是否通知所有持有者。

## 四、Project Zero 如何從線索定位問題？

這裡要區分兩件事：**原始錯誤由 syzkaller 找到；Project Zero 後來根據在野利用線索，重新確認它的安全影響並追查受影響設備。**

### 第一步：整理利用線索

2019 年夏末，Google TAG、Android Security 和 Project Zero 收到有關 Android 零日利用的線索。線索包括：
- 問題是可從 Chrome renderer 沙箱觸及的核心 UAF。
- Pixel 1、Pixel 2 受影響，Pixel 3、Pixel 3a 不受影響。
- 線索指出某個上游核心修補已在 Linux 4.14 出現，但當時沒有 CVE。
- `CONFIG_DEBUG_LIST` 等核心配置會影響利用條件。

Project Zero 當時沒有拿到實際利用樣本；研究員是根據線索、行銷材料、受影響設備清單和程式碼分析，判斷這個漏洞很可能正在被利用。

### 第二步：比較 Pixel 2 和 Pixel 3 的核心程式碼

研究員先嘗試其他候選問題，後來才把注意力轉向 Binder。接著比較 Pixel 2 與 Pixel 3 的 `drivers/android/binder.c` 和變更紀錄。

其中一個提交特別突出：
- 提交說明明確提到修復 use-after-free；
- 修補來自上游；
- 修補只進入特定核心版本分支。

### 第三步：用所有線索交叉驗證

研究員再把候選修補與最初收到的設備、核心配置和利用條件逐項比對，並檢查 Pixel 2 到 Pixel 3 之間其他約 25 項 Binder 變更。這個候選問題與線索吻合。

這是很好的研究方法：**不要只因為提交訊息裡出現 UAF 就認定找到了漏洞；要讓程式碼、版本差異、配置和外部線索彼此吻合。**

### 第四步：追溯原始發現和修補落差

這個 UAF 最早由 syzkaller/syzbot 在 2017 年 11 月回報，2018 年 2 月已有 Linux 及 Android 核心修補。但修補沒有被納入 Android 月度安全公告，因而沒有及時到達許多已上市設備。

這提醒我們：
- 找到程式錯誤，不代表它已被辨識為安全漏洞；
- 上游核心有修補，不代表所有廠商產品都已合併；
- 有修補提交，不代表使用者設備已收到更新。

## 五、你應該從這個案例學到什麼？

| 研究概念 | 你要理解的問題 |
|---|---|
| Object Lifetime | 物件何時建立、何時釋放？ |
| Pointer | 還有哪些地方保存物件參照？ |
| Use-After-Free | 釋放後是否仍有人使用？ |
| Race Condition | 不同執行流程是否可能交錯？ |
| Version Diffing | 哪個版本有問題，哪個版本有修補？ |
| Fuzzing | 能否自動探索異常的操作順序？ |
| Variant Analysis | 相同程式模式中是否還有其他問題？ |

## 六、初學者安全實驗：畫出物件生命週期

這個練習不需要 Android 手機、不需要 root，也不需要執行漏洞 PoC。

請在筆記中畫出：

    建立 binder_thread
            |
            v
    epoll 取得 wait queue 參照
            |
            v
    binder_thread 被釋放
            |
            v
    epoll 是否還保留舊參照？
            |
            v
    後續清理可能使用已失效參照

接著回答：

1. 哪一步開始出現生命週期問題？
2. 釋放物件之前，應該如何處理仍然存在的參照？
3. 為什麼核心驅動程式的 UAF 比一般應用程式崩潰更值得重視？
4. 為什麼同一份修補可能沒有及時進入所有手機？
5. 研究人員如何利用版本差異縮小搜尋範圍？

## 七、延伸研究：Variant Analysis

真正的漏洞研究不一定在第一個 CVE 就結束。

找到一個 UAF 後，可以進一步問：

- 同一驅動程式是否還有類似生命週期管理？
- 其他 poll handler 是否也保存了生命週期不一致的 wait queue？
- 修補是否只修正一條程式路徑？
- 修補是否引入新的競爭條件？
- 上游修補是否已同步到產品分支？

這種從已知漏洞模式尋找同類問題的方法叫 Variant Analysis（變體分析）。

## 八、閱讀順序

1. [Project Zero：Bad Binder](https://projectzero.google/2019/11/bad-binder-android-in-wild-exploit.html) — 先讀研究過程和線索。
2. [Project Zero：CVE-2019-2215 Root Cause Analysis](https://googleprojectzero.github.io/0days-in-the-wild/0day-RCAs/2019/CVE-2019-2215.html) — 再讀 UAF 的技術根因和變體分析。
3. [CVE.org：CVE-2019-2215](https://www.cve.org/CVERecord?id=CVE-2019-2215) — 查正式漏洞紀錄。

## 九、本課驗收

- [ ] 能解釋 Pointer、Object Lifetime 和 Use-After-Free。
- [ ] 能畫出 Binder 案例的物件生命週期。
- [ ] 能說明版本差異分析如何縮小搜尋範圍。
- [ ] 能區分原始漏洞發現、後續漏洞研究和產品修補。
- [ ] 能用自己的話解釋 Variant Analysis。

**本課最重要的一句話：** 漏洞研究不是看到異常就猜攻擊方式，而是從程式的資料、物件生命週期、信任邊界和版本差異中，逐步建立可以驗證的假設。
