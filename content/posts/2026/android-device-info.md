---
title: Android 取得裝置資訊
tags:
  - Android
date: 2026-06-24
description: "如何在 runtime 取得 Android 裝置的廠牌、製造商、型號與系統版本？本文整理 Build.MANUFACTURER、Build.BRAND、Build.MODEL、Build.VERSION 等 API 用法，並說明這些資訊在除錯特定機型 bug 時的實用價值。"
keywords: ["Android 裝置資訊", "Build.MODEL", "Build.MANUFACTURER", "Build.BRAND", "Build.VERSION", "Android 型號", "Android API Level", "取得手機型號", "特定機型除錯"]
summary: "解析如何用 Android Build 類別在 runtime 取得裝置廠牌、製造商、型號與系統版本，全程免特殊權限。說明 BRAND 與 MANUFACTURER 在紅米等子品牌的差異、MODEL 因發行地區而不同的特性，以及 RELEASE 與 SDK_INT 的對應關係。並點出為何 Crashlytics 與 Play Console 的事後聚合資料不足以應付非 crash 的特定機型怪 bug，runtime 自取資訊才能精準輔助除錯。"
---

## 前言

被公司同仁問到，就順手整理一下舊筆記...

以下裝置資訊的取得都不需要特殊權限，取得方式也很單純。但在除錯的時候，如果有這類型的資訊輔助，有時候可以抓到一些特定型號才會出現的 bug。

像是之前曾經遇過 [Samsung S10e 的 GNSS 時間錯誤問題](../1%20-%20Android%20Dev/DateTime%20日期時間/Android%20取得準確的時間%20(GPS_NTP).md)，經查才發現是 GPS 週數翻轉(GPS Week Number Rollover)導致算出來的時間都是錯的。

也許有人會想，Crashlytics 或 Play Console 不是都有這些裝置資訊嗎？但那些是事後聚合的資料，有些情境還是得自己在 runtime 讀——像上面那種非 crash 的怪 bug 它根本不會幫我們抓到，或是想針對特定機型走不同處理邏輯的時候，這些都得在程式裡自己拿到型號才行。

## 廠牌

品牌 `Build.BRAND` 印出的會是我們比較熟悉的品牌名稱，例如 `Google` 、`Samsung` 等。  
製造商 `Build.MANUFACTURER` 則會印出實際製造的廠商名稱。

以 Google、Samsung 來說，這兩個值都會是一樣的。

但像是中國手機，就比較容易出現兩個值不同的情況。  

例如：小米的子品牌紅米。

```
MANUFACTURER: Xiaomi
BRAND: Redmi
```

### 製造商

```kotlin
android.os.Build.MANUFACTURER

Log.d("製造商 - ${android.os.Build.MANUFACTURER}")
```

### 品牌

```kotlin
android.os.Build.BRAND

Log.d("製造商 - ${android.os.Build.BRAND}")
```

## 型號

`Build.MODEL` 印出的型號並非我們熟悉的產品名稱，而是品牌或製造商根據不同規格與發行地區所設定的型號。

舉例來說，Samsung S25 台灣版本印出的型號會是 `SM-S9310`。  
韓國版本則是 `SM-S931N`。

```kotlin
android.os.Build.MODEL

Log.d("型號 - ${android.os.Build.MODEL}")
```

## 系統版本

### 版本名稱

一般使用者所熟知的 Android 版本名稱，例如 `Android 15` 、 `Android 16`。

```kotlin
android.os.Build.VERSION.RELEASE

Log.d("Android Version - ${android.os.Build.VERSION.RELEASE}")
```

### 版本號 API Level

通常是開發者才會知道的 Android 版本號，例如 `36` ( Android 16 ) 、 `35` ( Android 15)。

```kotlin
android.os.Build.VERSION.SDK_INT

Log.d("API Level - ${android.os.Build.VERSION.SDK_INT}")
```

## 參考資料

- [StackOverflow: Retrieving Android API version programmatically](https://stackoverflow.com/questions/3423754/retrieving-android-api-version-programmatically)
- [Android Doc: Build](https://developer.android.com/reference/android/os/Build)
