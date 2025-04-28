---
title: Android 開發｜一鍵清除 App 資料，還原初始狀態的實作方法
tags:
  - Android
date: 2023-12-05
---

## 問題說明

前陣子剛好海巡到 [一則 iT 邦幫忙上的技術提問](https://ithelp.ithome.com.tw/questions/10214760)，大略找了一下資料，發現實作細節不難就順手回了一下。對方最後成功地完成他預期的需求，很高興能夠幫到對方的忙~

提問問題如下:

> 📢 App 需要新增一個刪除帳號的功能，需求方期望按下刪除鈕後，App 能夠回到初始安裝的狀態。等同於用戶直接到手機系統設定的應用程式清單中，按下清除資料。

## 解決方案

### Android 4.4 以後請用: ActivityManager.clearApplicationUserData()

根據[文件](https://developer.android.com/reference/android/app/ActivityManager#clearApplicationUserData())的說明，呼叫 `ActivityManager.clearApplicationUserData()` 等同使用者於設定中按下清除資料。

- Android API 19 以上可用。
- 內部 (Internal) 與外部 (External) 的應用程式私有資料都會被清除。
- 所有已取得的動態請求權限都會被撤銷。

```kotlin
val activityManager = application.getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager
activityManager.clearApplicationUserData()
```

 **📢 注意: 呼叫此方法後 App 會強制退出，使用者需要自行重新開啟 App。**

### Android 4.4 以前請用: Runtime.exec()

透過 `Runtime` 執行 ADB 指令 `pm clear packageName`。

```kotlin
Runtime.getRuntime().exec("pm clear ${applicationContext.packageName}")
```

Reference: [StackOverflow: Clear Application's Data Programmatically](https://stackoverflow.com/a/65976509/9982091)

#### 💡 延伸討論: 開發過程中完整清除 App 資料

有時候在開發或測試 App 的時候，也會需要還原 App 的初始狀態。
這時候可以改在 ADB 環境下，執行前述 `Runtime` 執行的清除指令。

```adb
adb shell pm clear packageName
```

Reference: [StackOverflow: Clear android application user data](https://stackoverflow.com/a/26400275/9982091)

## 延伸討論

### 單獨清除快取?

```kotlin
// 內部儲存空間
cacheDir.deleteRecursively()
// 外部儲存空間
externalCacheDir?.deleteRecursively()
```

Reference: [StackOverflow: Clear Cache in Android Application programmatically](https://stackoverflow.com/a/53492299/9982091)