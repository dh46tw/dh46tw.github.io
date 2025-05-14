---
title: Android 開發 | 讓 Gradle 自動為 AAR、AAB、APK 設定客製化檔名
tags:
  - Android/Build
  - Gradle
date: 2025-04-30
description: 教你如何透過 Gradle 客製 Android 專案輸出檔案名稱，為 APK、AAB、AAR 自動加上版本號與 Build Type，提升測試流程與團隊協作效率。
---

每次打包 AAB 或 APK，Android Studio 預設都會產出像 `app-release.aab` 或 `app-debug.apk` 這類的檔案名稱。這種預設命名在使用 Firebase App Distribution 或 Google Play 內部測試流程時，雖然不太會造成困擾，但如果你需要將 APK 或 AAB 檔案直接提供給 PM、QA 或外部合作夥伴安裝測試，那麼能夠一眼看出檔案版本與類型，會讓流程更順暢也更不容易搞混。

本文將示範如何透過 Gradle 設定，讓你在打包時自動加上版本號、Build Type 等資訊，生成便於辨識的輸出檔名。

## 設定方式

你可以根據不同的產出檔案類型（AAB、APK、AAR），在 app module 或 library module 的 `build.gradle` 中加入以下設定。

### AAB：變更 App Bundle 檔名

在 `build.gradle`（app module）中 `android` 區塊內加入以下設定：

```groovy
android {
	// 其他設定省略...
	setProperty("archivesBaseName", "taiwanNo1")
}
```

🚨 **注意：** 這個設定只會影響輸出檔案名稱的前綴，並不會完整覆蓋檔名結構。例如：

```
原始檔名：app-release.aab
變更後：taiwanNo1-release.aab
```

> 💡  為什麼使用 `archivesBaseName` ？
> 因為目前 Gradle 尚未提供官方 API 可以直接命名 AAB 檔。
> `archivesBaseName` 是 Gradle 用來設定各類 archives 類檔案輸出名稱的共通屬性。
> 所謂的 archives 檔案包含： APK、AAR、JAR、ZIP 等。

參考資料：[StackOverflow: How to change the generated filename for App Bundles with Gradle?](https://stackoverflow.com/a/59221190/9982091)

### APK：自訂 APK 輸出檔案名稱

同樣在 `build.gradle (app)` 的 `android` 區塊中，加入以下設定，即可針對每個 Variant 輸出不同名稱的 APK：

```groovy
// 檔案中最底部
android {
    // 略...

    applicationVariants.configureEach { variant ->
        variant.outputs.configureEach {output ->
            def variantName = variant.name
            def versionName = variant.versionName
            def versionCode = variant.versionCode
		    // 這裡的 archivesBaseName 可以透過前述的 setProperty() override
            outputFileName = "${archivesBaseName}-${versionName}.apk"
        }
    }
}
```

這樣打包出來的 APK，會長這樣的格式：

```
taiwanNo1-1.0.3-release.apk
taiwanNo1-1.0.3-debug.apk
```

清楚標示版本與 build type，對團隊來說超級友善。

### AAR：變更 Library 模組輸出檔名

如果你是在開發 Android Library（非應用程式），也可以使用相似的方式客製 AAR 檔名。差別只在於 Variant 類型需改用 `libraryVariants`：

```groovy
android {  
    // 其他設定
  
    libraryVariants.configureEach { variants ->  
        variants.outputs.all { output ->  
	        // 原本 Library 名稱-BuildType 名稱-版本名
            outputFileName = "${archivesBaseName}-${variants.name}-${defaultConfig.versionName}.aar"  
        }  
    }}
```

設定完成後，打包出來的檔案會像這樣：

```
taiwanNo1-release-1.0.0.aar
```

## 小結

這樣的檔名客製化看似只是小細節，但對於多人協作、測試流程管理、甚至上架備份流程來說，都是一種品質與效率的提升。如果你常需要手動改檔名、加版本號，那真的建議一次設定好 Gradle，未來就可以安心自動化產出好讀、好用的檔案名稱。
