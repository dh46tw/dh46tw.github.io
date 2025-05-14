---
title: "比 Android 原生更方便的 Log 工具: Timber"
tags:
  - Published
  - Android
date: 2023-05-27
description: 本文介紹了 Timber 這個 Android Logger Library，它是基於 Android Log 的開發工具，解決了開發過程中需要移除 Log 訊息的問題，並提供簡潔易用的 API 來進行日誌記錄。詳細介紹了如何使用 Timber，包括安裝、初始化與基本操作。
---

> 本文同步發表在 [HackMD](https://hackmd.io/@dh46tw/android-lib-timber) & [Medium](https://dh46tw.medium.com/%E6%AF%94-android-%E5%8E%9F%E7%94%9F%E6%9B%B4%E6%96%B9%E4%BE%BF%E7%9A%84-log-%E5%B7%A5%E5%85%B7-timber-bc7e27e78cb8)

## Timber 是什麼

Timber 是一個以 Android `Log` 為基底所開發的 Logger Library，由 [Jake Wharton](https://github.com/JakeWharton) 大神所開發。

## Timber 為了解決什麼問題

### 1. 開發時可以留著，但發佈版本需要移除 `Log`

```kotlin
// 你可能很常看到類似這樣的寫法...
if (BuildConfig.DEBUG) {
    Log.d(TAG, "Hello World!")
}
```

一般來說在開發上，我們習慣使用 Android 的 `Log` class 來印出所需的資訊。但是當今天開發到一定的階段，程式必須發布上線時，為了資訊安全等需求，需要將這些 Log 給全部註解或移除，又或是加上 buildFlavor 或 buildType 判斷，這一切實在是太麻煩了...

### 2. 每次在新的類別中使用 `Log` 就要建一個該類別的 TAG `String`

```kotlin
val TAG: String = Hello::class.java.simple

if (BuildConfig.DEBUG) {
    Log.d(TAG, "Hello World!")
}
```

> 同步發表在 [HackMD](https://hackmd.io/@dh46tw/android-lib-timber) & [Medium](https://dh46tw.medium.com/%E6%AF%94-android-%E5%8E%9F%E7%94%9F%E6%9B%B4%E6%96%B9%E4%BE%BF%E7%9A%84-log-%E5%B7%A5%E5%85%B7-timber-bc7e27e78cb8)

## Timber 怎麼使用

### 1. Dependency

在 `build.gradle` 中加入以下的 Dependency。

```groovy
repositories {
  mavenCentral()
}

dependencies {
  implementation 'com.jakewharton.timber:timber:5.0.1'
}
```

### 2. 初始化

在專案內繼承的 `Application` class 的 `onCreate()` 中呼叫以下程式碼。這樣一來，`Timber` 的 log 就只會在 `BuildType` 是 `DEBUG` 的時候才會印出。 

```kotlin
override fun onCreate() {
    super.onCreate()

    if (BuildConfig.DEBUG) {
        Timber.plant(Timber.DebugTree())
    }
}
```

如果有想要在正式版時印出特定的除錯資訊，可以參考官方的[範例](https://github.com/JakeWharton/timber/tree/trunk/timber-sample)。

### 3. 使用

```kotlin
Timber.d("initFcmToken: token = $token")
Timber.e(e, "initCertificate: ")
```

基本上 `i`, `w`, `d`, `e` 四種 Log 類型都有。
另外，Library 本身還帶有語法檢查，如果使用 `Timber` 但格式錯誤，或是有用 Android 的 `Log`，都會有 IDE 的 Highlight 提醒你修正。

## 參考資料

- [Github: Jake Wharton-Timber](https://github.com/JakeWharton/timber)
- [Day 30 - Timber與謝幕](https://ithelp.ithome.com.tw/articles/10189008)
- [使用Timber來幫忙打印log](https://jimmy4302001.medium.com/%E4%BD%BF%E7%94%A8timber%E4%BE%86%E5%B9%AB%E5%BF%99%E6%89%93%E5%8D%B0log-afc54aaa76d6)
