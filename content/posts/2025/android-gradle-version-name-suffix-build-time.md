---
title: Android 開發 | 讓 Gradle 自動在版本名稱後加上 Build 時間
tags:
  - Android/Build
  - Gradle
date: 2025-04-28
description: 教你如何用 Gradle 的 versionNameSuffix 搭配打包時間，自動為 APK 加上唯一版本識別，提升 Android App 測試流程的辨識效率與方便性。
---

在開發 App 的過程中，常常需要產出不同版本的 APK，給團隊成員或測試人員驗證功能。  這時候，「快速辨識版本」就變成一件非常重要的事了。

最常見、也最直接的方法，就是利用 Gradle 的 `versionNameSuffix`，在 App 的版本名稱後面自動加上一些額外資訊。  以我們團隊的習慣來說，我們會直接加上**Build 當下的時間**作為版本流水號，這樣每個 APK 都能有獨一無二的識別。但如果每次出新版本，都還要**手動去改 Gradle 設定**，實在是有點麻煩。

下面就分享一個簡單的做法，讓 Gradle 在每次 Build 的時候，自動把「打包時間」加到版本名稱後面。

## 程式碼範例

### Groovy 寫法

```groovy
// 產生日期時間字串
def getDate() {
    def date = new Date()
    def formattedDate = date.format('yyyyMMddHHmmss')
    return formattedDate
}

// 在buildType中就可使用
debug {
   applicationIdSuffix '.debug'
   // 設定版本名稱後綴
   versionNameSuffix '-dev' + ' (' + getDate() + ')' 
   // 其他設定...
}
```

### Kotlin DSL (KTS) 寫法

```kotlin
android {  
    // 其他設定...
    buildTypes {
	    // 略...
        debug {
            isMinifyEnabled = false  
            applicationIdSuffix = ".dev"  
            // 設定版本名稱後綴
            versionNameSuffix = "-dev (${getDate()})"  
        }    
    } 
}

/**  
 * 取得目前的日期時間字串
 *  
 * @return  
 */  
fun getDate(): String {  
    val date = Calendar.getInstance().time
    val formatter = SimpleDateFormat("yyyyMMdd-HHmm", Locale.getDefault())  
    return formatter.format(date)  
}
```

這樣設好之後，每次打包 Debug 版 APK，都會自動帶上像 `-dev(20250428-1430)` 這種字串， 讓相關人員清楚知道這份檔案是什麼時候打包的。

## 參考來源

- [how append date build to versionNameSuffix on gradle](https://stackoverflow.com/a/19184323/9982091)
- [如何在Android裝置上部署多版本App](https://medium.com/@wsrew2000/%E5%A6%82%E4%BD%95%E5%9C%A8android%E8%A3%9D%E7%BD%AE%E4%B8%8A%E9%83%A8%E7%BD%B2%E5%A4%9A%E7%89%88%E6%9C%ACapp-bb44004e9c25)
