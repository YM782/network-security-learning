
# 10-01｜Windows 安全功能和事件檢視器

## 本課目標
認識 Windows 內建防護功能，能打開事件檢視器並閱讀基本事件欄位。

## 1. Windows Security
Windows 安全性整合病毒與威脅防護、防火牆和裝置安全等功能。不要為了學習而隨意關閉防毒或防火牆；若程式被阻擋，先確認程式來源和警告內容。

## 2. Windows Defender Firewall
防火牆依照規則控制網路連線。它可以限制哪些應用程式或連接埠接受或發起網路流量，但不能取代更新、備份和權限管理。

## 3. Event Viewer（事件檢視器）
開啟方式：
1. 按 Win + R。
2. 輸入 eventvwr.msc。
3. 按 Enter。
4. 展開 Windows Logs。
5. 查看 System 和 Application。

## 4. 閱讀一筆事件
- Logged：記錄時間。
- Level：事件等級。
- Source：事件來源。
- Event ID：事件識別碼。
- General：一般說明。
- Details：詳細資料。

Event ID 必須結合來源和事件內容解讀，不能只憑一個數字判斷入侵。

## 5. 本課練習
打開 Windows Logs → System，選一筆普通事件，記錄時間、Level、Source 和 General 內容。不要清除日誌。

## 6. 本課驗收
能開啟事件檢視器，找到一筆事件，並說明時間、來源、等級和事件 ID。
