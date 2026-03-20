---
title: Android 動態更改應用程式圖標 App Icon
tags: 
  - Android
date: 2023-11-23T16:38:00
description: 本文探討 Android 上如何動態更改應用程式圖標 App Icon 的方法，並提供範例程式碼。
keywords:
  - "Android"
  - "Android 動態更改應用程式圖標 App Icon"
cover:
  image: "attachments/Pasted image 20241227105916.png"
  relative: true
  alt: "Android 動態更改應用程式圖標 App Icon"
  hidden: false
  hiddenInList: false
  hiddenInSingle: true
summary: 本文探討 Android 上如何動態更改應用程式圖標 App Icon 的方法，並提供範例程式碼。
---
## 一、問題描述

最近使用 [Todoist](https://play.google.com/store/apps/details?id=com.todoist&pcampaignid=web_share) 做任務管理時發現，App 內有提供自選圖標的功能，也就是動態更換 App 的 Launcher Icon。

| Step 0                                               | Step 1                                               | Step 2                                               | Step 3                                               |
| ---------------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| ![Android 動態更改應用程式圖標 App Icon操作畫面1](attachments/Pasted%20image%2020241227105916.png) | ![Android 動態更改應用程式圖標 App Icon操作畫面2](attachments/Pasted%20image%2020241227105926.png) | ![Android 動態更改應用程式圖標 App Icon操作畫面3](attachments/Pasted%20image%2020241227105934.png) | ![Android 動態更改應用程式圖標 App Icon操作畫面4](attachments/Pasted%20image%2020241227105943.png) |
| 按下按鈕啟用自選圖標功能。                                        | 原本 App 預設是紅色的 App Icon。                              | 在 App 內設定指定的 Icon 後。                                 | Launcher 上的 Todoist Icon 就真的換剛才所選的 App Icon 了！       |

另外一個 App [DuckDuckGo](https://play.google.com/store/apps/details?id=com.duckduckgo.mobile.android&pcampaignid=web_share) 也有一樣的功能。不過操作流程上不太一樣。DuckDuckGo 不需要特別啟用自選圖標，但在每次更換 Icon 時，App 都會在設定完成後自動關閉，使用者必須自動重啟 App 才能繼續使用。 

🚩 兩者使用體驗差異整理:

1. Todoist: 
    - 首次使用必須啟用功能。
    - 啟用功能後 App 會自行關閉，需要使用者自行重開 App。
    - 重開後就可以使用自選圖標的功能，而且往後的更換行為 App 都不會再自動關閉。
2. DuckDuckGo:
    - 不須特別啟用功能。
    - 每次更換圖標都會自動關閉 App，使用者要自己重開。


==如果想直接看怎麼做，可以跳到 [三、解決方案](#三、解決方案)。==

## 二、研究歷程

### 2-1 問題釐清

搜尋幾個關鍵字其實滿快就找到相關的解法了，基本上分成使用 `AndroidManifest.xml` 的 `activity-alias` 更換 App 進入點的參數，或是使用 [App Shortcuts](https://developer.android.com/guide/topics/ui/shortcuts/creating-shortcuts?hl=zh-tw)。

Shortcuts 比較代表性的 App 是 Instagram。但仔細觀察兩個 App 所呈顯出來的效果，可以確定不是透過建立 Shortcut 的方式更換 Icon，比較接近第一種方式。

### 2-2 初探 `activity-alias`

有關`activity-alias` 的操作方式 StackOverflow 都有，但我最後選擇跟著 [Dynamic App Icon In Android](https://oguzhanaslann.medium.com/dynamic-app-icon-in-android-a61f8570ab9f) 這篇文章一起實作，發現確實可以更換 App Icon。

這個方法是透過，在 `AndroidManifest.xml` 中建立 `MainActivity` 的 `activity-alias`，每一個版本的 Icon 都會需要一個對應的 `activity-alias`。

```xml
<activity
    android:name=".MainActivity"
    android:exported="true"
    android:label="@string/app_name"
    android:theme="@style/Theme.DynamicAppIcon">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>

<activity-alias
    android:name="Option1"
    android:enabled="false"
    android:exported="true"
    android:icon="@mipmap/ic_launcher_option_1"
    android:roundIcon="@mipmap/ic_launcher_option_1_round"
    android:targetActivity=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity-alias>
```
 
💡 什麼是 `activity-alias`?

顧名思義，就是 `Activity` 的別名(alias)。

- `activity-alias` 在 `AndroidManifest` 中的順序，必須在它的目標 `Activity` 之後。
- 一般 `activity` tag 有的屬性欄位它都有。
- `activity-alias` 裡的屬性可以視為是 `activity` tag 的子集(subset)。

[官方文件連結](https://developer.android.com/guide/topics/manifest/activity-alias-element)  


設定完成後，在程式中動態的啟用與關閉 `activity-alias` 來達到更換 Icon 的效果。

```kotlin
/**
 * 更換目前啟用的 activity-alias
 *
 * @param enabled
 * @param disabled
 */
private fun changeEnabledComponent(enabled: String, disabled: String) {
    // Enable
    packageManager.setComponentEnabledSetting(
        ComponentName(this@MainActivity, "$packageName.$enabled"),
        PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
        PackageManager.DONT_KILL_APP
    )
    
    // Disable
    packageManager.setComponentEnabledSetting(
        ComponentName(this@MainActivity, "$packageName.$disabled"),
        PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
        PackageManager.DONT_KILL_APP
    )
}
```

🚨 前面所完成的版本，雖然可以動態改 Icon，但是會有以下幾個問題:

1. 每次更換 Icon 都會關閉 App，沒辦法像 Todoist 一樣，只要第一次啟用功能時關閉就好。
2. Debug install 錯誤。在換成 `activitiy-alias` 的狀態下，透過 IDE 是沒辦法重新 Install App 的。會出現 `Error running 'app': Activity class {tw.dh46.dynamicicon/tw.dh46.dynamicicon.MainActivity} does not exist` 的錯誤。如果要透過 IDE Reinstall，就必須換回原本的 `activity` tag。  

## 三、解決方案

細讀 [StackOverflow: Change Android Launcher Icon like Instagram/Todoist](https://stackoverflow.com/q/68576022/9982091) 裡的討論，與 [Github oguzhanaslann/DynamicIcon](https://github.com/oguzhanaslann/DynamicIcon/) 專案，並查看反編譯 Todoist 後得到的 `AndroidManifest.xml`。

![Android 動態更改應用程式圖標 App Icon操作畫面5](attachments/Pasted%20image%2020241227110003.png)

整理出最終的設計流程大概是下方這樣。

### 3-1 設定 `AndroidManifest.xml`

#### 3-1-1 將 App 進入點的 Activity 設定如下

```xml
<activity
    android:name=".MainActivity"
    android:exported="true">
    <intent-filter>
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

#### 3-1-2 設定安裝當下預設與啟用功能後的預設 `activity-alias`

`AndroidManifest` 中的 `activitiy-alias` 除了要更換的種類外，要再建立兩個特殊的 `activitiy-alias`。

1. 安裝當下的預設 `activity-alias`。(後面用 `BuildIn` 代稱)
2. 啟用更換 Icon 功能後的預設 `activity-alias`。(後面用 `Default` 代稱)

```xml
<!--  安裝當下的  -->
<activity-alias
    android:name=".MainActivityBuiltIn"
    android:exported="true"
    android:icon="@mipmap/ic_launcher"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:targetActivity=".MainActivity">

    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity-alias>

<!--  啟用後的 Default -->
<activity-alias
    android:name=".MainActivityDefault"
    android:enabled="false"
    android:exported="true"
    android:icon="@mipmap/ic_launcher"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:targetActivity=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity-alias>
```

#### 3-1-3 設定其他 Icon 對應的 `activity-alias`

```xml
<!--  Blue  -->
<activity-alias
    android:name=".MainActivityBlue"
    android:enabled="false"
    android:exported="true"
    android:icon="@mipmap/ic_launcher_blue"
    android:roundIcon="@mipmap/ic_launcher_blue_round"
    android:targetActivity=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity-alias>

<!--  Red  -->
<activity-alias
    android:name=".MainActivityRed"
    android:enabled="false"
    android:exported="true"
    android:icon="@mipmap/ic_launcher_red"
    android:roundIcon="@mipmap/ic_launcher_red_round"
    android:targetActivity=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity-alias>

<!--  Yellow  -->
<activity-alias
    android:name=".MainActivityYellow"
    android:enabled="false"
    android:exported="true"
    android:icon="@mipmap/ic_launcher_yellow"
    android:roundIcon="@mipmap/ic_launcher_yellow_round"
    android:targetActivity=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity-alias>
```

### 3-2 建立 `AppIconManager` 封裝更換的邏輯

```kotlin
package tw.dh46.android.dynamic_app_icon

import android.content.ComponentName
import android.content.Context
import android.content.pm.ActivityInfo
import android.content.pm.PackageManager

/**
 *  Created by DanielHuang on 2023/11/21
 */
sealed class AppIcon(val alias: String, val iconResId: Int, var isEnable: Boolean) {
    data object BuiltIn : AppIcon("MainActivityBuiltIn", R.mipmap.ic_launcher, true)
    data object Default : AppIcon("MainActivityDefault", R.mipmap.ic_launcher, false)
    data object Blue : AppIcon("MainActivityBlue", R.mipmap.ic_launcher_blue, false)
    data object Red : AppIcon("MainActivityRed", R.mipmap.ic_launcher_red, false)
    data object Yellow : AppIcon("MainActivityYellow", R.mipmap.ic_launcher_yellow, false)
}

private val appIconOptions = listOf(AppIcon.Default, AppIcon.Blue, AppIcon.Red, AppIcon.Yellow)

/**
 * App icon manager
 *
 * @property context
 * @property packageManager
 * @property buildInAppIcon 安裝當下預設的 Icon
 * @property defaultAppIcon 啟用更換 Icon 後的預設 Icon
 * @property targetActivity App 進入點的 Activity 名稱，也是 activity-alias 的 targetActivity
 * @constructor Create empty App icon manager
 */
class AppIconManager(
    private val context: Context,
    private val packageManager: PackageManager = context.packageManager,
    private val buildInAppIcon: AppIcon = AppIcon.BuiltIn,
    private val defaultAppIcon: AppIcon = AppIcon.Default,
    private val targetActivity: String = "MainActivity"
) {
    /**
     * 啟用更換 Icon 功能
     */
    fun activateFeature() {
        setAliasComponentState(defaultAppIcon.alias, true)
        setAliasComponentState(buildInAppIcon.alias, false)
    }

    /**
     * 停用更換 Icon 功能
     */
    fun deactivateFeature() {
        setActiveAppIcon(buildInAppIcon)
    }

    /**
     * 設定目前啟用的 AppIcon
     *
     * @param appIcon
     */
    fun setActiveAppIcon(appIcon: AppIcon) {
        // 功能啟用後才能變換 Icon
        checkFeatureActivated()

        // 停用其他選項
        disableOtherIconOptions(appIcon)

        // 啟用選中的選項
        enableIcon(appIcon)
    }

    private fun enableIcon(appIcon: AppIcon) {
        appIcon.isEnable = true
        setAliasComponentState(appIcon.alias, true)
    }

    /**
     * 停用其他選項
     *
     * @param appIcon
     */
    private fun disableOtherIconOptions(appIcon: AppIcon) {
        getLatestIconOptions().filterNot {
            it == appIcon
        }.forEach {
            it.isEnable = false
            setAliasComponentState(it.alias, false)
        }
    }

    /**
     * 檢查是否啟用
     */
    private fun checkFeatureActivated() {
        if (!isFeatureActivated()) throw IllegalStateException("Feature is not activated!")
    }

    /**
     * 檢查 activity-alias: MainActivityDefault 是否被停用 (Disabled)
     * 來辨識是否已啟用更換圖示的功能
     *
     * @return
     */
    fun isFeatureActivated(): Boolean {
        return packageManager.getComponentEnabledSetting(
            createComponentName(buildInAppIcon.alias),
        ) == PackageManager.COMPONENT_ENABLED_STATE_DISABLED
    }

    /**
     * 取得最新狀態的 Icon 選項
     *
     * @return
     */
    fun getLatestIconOptions(): List<AppIcon> {
        getLauncherActivityInfoList().forEach { activityInfo ->
            val isEnabled = isComponentEnabled(activityInfo)

            if (isEnabled) {
                appIconOptions.forEach {
                    if (activityInfo.name == "${context.packageName}.${it.alias}") {
                        it.isEnable = true
                    }
                }
            }
        }

        return appIconOptions
    }

    // ----------------------------------------------------

    /**
     * 設定 Component 狀態 
     *
     */
    private fun setAliasComponentState(alias: String, enable: Boolean) {
        // 檢查是否有非法 alias
        require(buildInAppIcon.alias == alias || appIconOptions.any { it.alias == alias }) { "Invalid alias: $alias" }

        val newState = if (enable) {
            PackageManager.COMPONENT_ENABLED_STATE_ENABLED
        } else {
            if (alias == buildInAppIcon.alias) {
                // 如果是安裝預設選項 STATE 要設為 DISABLED 停用
                PackageManager.COMPONENT_ENABLED_STATE_DISABLED
            } else {
                // 樣式 Icon 切換，關閉是設為 DEFAULT (避免切換 Icon 後 App 必須關閉)
                PackageManager.COMPONENT_ENABLED_STATE_DEFAULT
            }
        }

        packageManager.setComponentEnabledSetting(
            createComponentName(alias),
            newState,
            PackageManager.DONT_KILL_APP
        )
    }

    private fun createComponentName(alias: String) =
        ComponentName(context, "${context.packageName}.$alias")

    /**
     * 檢查該 Component 是否啟用
     * 
     */
    private fun isComponentEnabled(activityInfo: ActivityInfo): Boolean {
        val state =
            packageManager.getComponentEnabledSetting(ComponentName(context, activityInfo.name))

        val isEnabled = if (state == PackageManager.COMPONENT_ENABLED_STATE_DEFAULT) {
            activityInfo.enabled
        } else {
            state == PackageManager.COMPONENT_ENABLED_STATE_ENABLED
        }
        return isEnabled
    }

    /**
     * 取得目前 targetActivity 相關的 activity, activity-alias 
     * 兩種 tag 的 ActivityInfo
     */
    private fun getLauncherActivityInfoList(targetActivityClassName: String = targetActivity): List<ActivityInfo> {
        return packageManager.getPackageInfo(
            context.packageName,
            PackageManager.GET_ACTIVITIES or PackageManager.MATCH_DISABLED_COMPONENTS
        ).activities.filter {
            it.name.contains(targetActivityClassName)
        }
    }
}
```

#### 3-2-1 `activateFeature()`

首次啟用更換 Icon 功能時執行，App 會自動關閉，需要使用者自行重開 App。
    
- 將 `BuildIn` 設為 `COMPONENT_ENABLED_STATE_DISABLED`
- 將 `Default` 設為 `COMPONENT_ENABLED_STATE_ENABLED`

#### 3-2-2 `setActiveAppIcon()`

啟用後更換 Icon 時執行，App 不會自動關閉，且 Icon 已換成設定的樣式。

- 將 `Default` 設為 `COMPONENT_ENABLED_STATE_DEFAULT`
- 將 **要更換的 `activity-alias`** 設為 `COMPONENT_ENABLED_STATE_ENABLED`

#### 3-2-3 `deactivateFeature()`

要停用切換功能時執行。
    
- 將 `BuildIn` 設為 `COMPONENT_ENABLED_STATE_ENABLED`
- 將 `Default` 設為 `COMPONENT_ENABLED_STATE_DEFAULT`

💡 以上完成的版本，就可以達到與 Todoist 相同的使用體驗。

![Android 動態更改應用程式圖標 App Icon示範動畫6](attachments/android-dynamic-app-icon.gif)

如果有需要完整的範例專案，請參考 [Github: dh46tw/android-dynamic-app-icon-demo](https://github.com/dh46tw/android-dynamic-app-icon-demo)。

## 四、已知問題

### 4-1 Debug build install error

改成其他 Alias 後，透過 IDE 重 build App 會出現 

```
Error running 'app': Activity class {tw.dh46.dynamicicon/tw.dh46.dynamicicon.MainActivity} does not exist
```

這部分是因為 ADB 執行的指令會是:

```log
2023-11-23 16:25:58: Launching app on 'Pixel 7 Pro API 33.
$ adb shell am start -n "tw.dh46.android.dynamic_app_icon/tw.dh46.android.dynamic_app_icon.MainActivityBuiltIn" -a android.intent.action.MAIN -c android.intent.category.LAUNCHER --splashscreen-show-icon 

Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=tw.dh46.android.dynamic_app_icon/.MainActivityBuiltIn }
```

但是更換 Icon 後，`activity-alias` 已經不再是 `MainActivityBuiltIn` ，所以才會無法透過 IDE 啟動 App。

### 4-2 發布後不能隨便刪除 `activity-alias`

> _新图标的启动在代码中控制，之后发版不要轻易删除别名列表，否则覆盖安装时，当前展示的别名图标被删除了会引发找不到activity崩溃_
> 
> 來源: [activity-alias 的使用及若干问题](https://blog.csdn.net/JiaoJunfeng/article/details/129360815)

## 五、參考資料

### StackOverflow

- [StackOverflow: How to change an application icon programmatically in Android?](https://stackoverflow.com/questions/1103027/how-to-change-an-application-icon-programmatically-in-android)
- [StackOverflow: Is there any way to dynamically change an app icon like Calendar app does?](https://stackoverflow.com/a/34617982/9982091)
- [StackOverflow: Change Android Launcher Icon like Instagram/Todoist](https://stackoverflow.com/q/68576022/9982091)

### Medium

- [Dynamic App Icon In Android](https://oguzhanaslann.medium.com/dynamic-app-icon-in-android-a61f8570ab9f)
- [Dynamic Launcher Icon and Name for Android](https://medium.com/@simonmisles/dynamic-launcher-icon-and-name-for-android-e40bf0561715)

### Github

- [DuckDuckGo-AppIconModifier.kt](https://github.com/duckduckgo/Android/blob/develop/app/src/main/java/com/duckduckgo/app/icon/api/AppIconModifier.kt)

---

📢 同步發表在 [iT 邦幫忙](https://ithelp.ithome.com.tw/articles/10312323)、[HackMD](https://hackmd.io/@dh46tw/android-dynamic-app-icon)。