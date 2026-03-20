---
title: Android 14 AlarmManager SCHEDULE_EXACT_ALARM 權限閃退
tags:
  - Android
  - Android/AlarmManager
date: 2024-03-27
description: 解決 Android 14 精確鬧鐘設定閃退問題，介紹 SCHEDULE_EXACT_ALARM 和 USE_EXACT_ALARM 權限設定，提供最佳實踐，幫助開發者應對精確鬧鐘功能在新系統中的變動。
keywords:
  - "Android"
  - "Android/AlarmManager"
  - "AlarmManager"
  - "Android 14 AlarmManager SCHEDULE_EXACT_ALARM 權限閃退"
summary: 解決 Android 14 精確鬧鐘設定閃退問題，介紹 SCHEDULE_EXACT_ALARM 和 USE_EXACT_ALARM 權限設定，提供最佳實踐，幫助開發者應對精確鬧鐘功能在新系統中的變動。
---
## 一、問題描述

Android 14 的使用者反映應用程式閃退，經查測後發現是設定指定時間提醒功能時出現閃退。

```
Fatal Exception: java.lang.SecurityException: Caller com.tms.qpass needs to hold android.permission.SCHEDULE_EXACT_ALARM or android.permission.USE_EXACT_ALARM to set exact alarms.
```

從錯誤訊息可以清楚看到，問題發生當下是拋出 `SecurityException` 。

## 二、錯誤原因說明

其實從錯誤訊息中就可略知一二，一切都跟 `SCHEDULE_EXACT_ALARM` 還有 `USE_EXACT_ALARM` 有關。

> ✍ 如果是日曆或鬧鐘應用程式，應該要宣告的是 [USE_EXACT_ALARM](https://developer.android.com/reference/android/Manifest.permission#USE_EXACT_ALARM) 精確鬧鐘權限。
> 
> USE_EXACT_ALARM 是一項即將推出的新權限，作用是針對以 Android 13 (API 級別 33) 以上版本為目標的應用程式，授予[精確鬧鐘功能](https://developer.android.com/about/versions/13/features#use-exact-alarm-permission)的存取權。
> 
> USE_EXACT_ALARM 是一項受限制權限，只有當應用程式的核心功能支援精確鬧鐘需求，應用程式才能宣告這項權限。要求這項受限制權限的應用程式必須接受審查，若是應用程式不符合使用限制條件，就禁止在 Google Play 發布
> 核心功能: 
> 	- 應用程式是鬧鐘或計時器應用程式。
> 	- 應用程式是會顯示活動通知的日曆應用程式。
> 上述以外的用途，則應評估能否選擇使用 SCHEDULE_EXACT_ALARM 做為替代方案。
> 
> 以上整理自 [Google Play Policy](https://support.google.com/googleplay/android-developer/answer/9888170?hl=zh-Hant)。

### SCHEDULE_EXACT_ALARM 權限

> 💡 適用的精準鬧鐘 API 
> 
> 1. `setExact()` 
> 2. `setExactAndAllowWhileIdle()`
> 3. `setAlarmClock()`

#### Android 12 以前

可直接使用 API。

#### Android 12 後

從 Android 12 開始，所有 Target SDK 為 Android 12 的 App ，如果要使用 `AlarmManager` 的精準鬧鐘 API，都必須要在 `AndroidManifest.xml`  中宣告`SCHEDULE_EXACT_ALARM` 權限。

因為此權限系統預設允許，所以不用特別引導使用者開啟設定。

#### Android 14 後

在以下幾種情況時，`SCHEDULE_EXACT_ALARM` 權限預設關閉。

1. App Target SDK 為 Android 13 或以上。
	1. 包含使用者使用備份軟體將 App 還原到較新版本的裝置上。
	2. 原本就有安裝且允許權限，然後系統自動更新到新系統者不在此限。
2. 在 `AndroidManifest.xml` 中有宣告 `SCHEDULE_EXACT_ALARM`。
3. App 不在豁免條件或預先允許的情境內。
4. 非日曆或鬧鐘應用程式。

## 三、最佳實踐

### 1. 在 `AndroidManifest.xml` 宣告使用的權限

```xml
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />
```

### 2. 設定 Alarm 前動態檢查，否則會出 `SecurityExeption`

使用 `AlarmManager.canScheduleExactAlarm()` 檢查是否取得權限。

```kotlin
/**
 * [ALARMMANAGER] Android 12 以上 精確鬧鐘權限
 *
 * @return
 */
private fun isSetExactAlarmAllowed(): Boolean =
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
        val alarmManager =
            requireContext().getSystemService(Context.ALARM_SERVICE) as AlarmManager
        alarmManager.canScheduleExactAlarms()
    } else {
        true
    }
```

### 3. 若無法取得權限，可以跟使用者說明後，使用 `Intent` 帶往系統設定。

```kotlin
if (Build.VERSION.SDK_INT < Build.VERSION_CODES.S) return

val intent = Intent(Settings.ACTION_REQUEST_SCHEDULE_EXACT_ALARM)
intent.data = Uri.parse("package:${context?.packageName}")
startActivity(intent)
```

### 4. 官方建議在 `onResume()` 再次檢查是否有取得權限。

```kotlin
override fun onResume() {   
	if (alarmManager.canScheduleExactAlarms()) {
		// Set exact alarms.
		alarmManager.setExact(...)
	} else { 
		// Permission not yet approved. Display user notice and revert to a fallback approach.
		alarmManager.setWindow(...)
	}
}
```

> 🚨 監聽權限允許結果
> 
> 當 `SCHEDULE_EXACT_ALARMS` 的權限允許時，系統會發送帶有 [`ACTION_SCHEDULE_EXACT_ALARM_PERMISSION_STATE_CHANGED`](https://developer.android.com/reference/android/app/AlarmManager#ACTION_SCHEDULE_EXACT_ALARM_PERMISSION_STATE_CHANGED)的 Broadcast，App 可以實作 `BroadcastReceiver`來監聽此事件。
> 在監聽事件的回呼中，官方也建議再用 `canSetExactAlarm()` 檢查是否真的有拿到權限。
> 
> 我個人的作法，會另外將是否曾請求過權限記錄在本地端，避免因為單一功能重複向使用者請求權限。

## 四、參考資料

- [Android Doc: AlarmManager.setExactAndAllowWhileIdle](https://developer.android.com/reference/android/app/AlarmManager#setExactAndAllowWhileIdle(int,%20long,%20android.app.PendingIntent))
	- 基本上跟 `setExact()` 一樣的方法，可以設定指定時間執行的鬧鐘。差別在於當系統處於低電量模式時，鬧鐘仍會執行。
- [SCHEDULE_EXACT_ALARM](https://developer.android.com/reference/android/Manifest.permission#SCHEDULE_EXACT_ALARM)
	- 權限文件
- [Android 14 Behaviour changes--Schedule exact alarms are denied by default](https://developer.android.com/about/versions/14/changes/schedule-exact-alarms#migration)
	- 基本上可以單看這篇就好，把整個權限的前因後果，以及最佳實踐寫得很清楚。
- [App Targets Android 12 Behaviour changes--Exact Alarm Permission](https://developer.android.com/about/versions/12/behavior-changes-12#exact-alarm-permission)
- [Medium--Android 14 Behavior Changes：精確鬧鐘行為變更](https://carterchen247.medium.com/android-14-behavior-changes-%E7%B2%BE%E7%A2%BA%E9%AC%A7%E9%90%98%E8%A1%8C%E7%82%BA%E8%AE%8A%E6%9B%B4-87b432f979e0)
	- 中文版的說明，寫得很清楚，文末也有提供一個開源的 Library 可以使用。