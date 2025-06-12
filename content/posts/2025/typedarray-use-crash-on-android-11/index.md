---
title: TypedArray.use{} 在 Android 11 以下出錯？ Crash 原因與正確寫法一次看懂
tags:
  - Android
  - TypedArray
  - Kotlin
  - AutoCloseable
date: 2025-06-12
description: "在 Android 開發中使用 Kotlin 的 use {} 是常見的資源釋放寫法，但你知道在 Android 11 (API 30) 以前對某些類別其實會造成 Crash 嗎？這篇文章帶你深入解析 TypedArray 為什麼不能直接用 use {}，並提供正確的修正方式與建議，避免掉不必要的陷阱！"
keywords: ["android", "kotlin", "typedarray", "autocloseable","use function", "crash"]
image: /attachments/kotlin-use-typedarray-error.png
cover:
#   image: "attachments/kotlin-use-typedarray-error.png"
  relative: true # 圖片在文章相對路徑
  hidden: false           # 不隱藏 cover
  hiddenInList: false     # 在列表頁顯示 cover
  hiddenInSingle: false   # 在單篇頁面顯示 cover
---

在 Android 開發中，我們常使用 Kotlin 的 `use {}` 語法來自動管理資源，例如關閉檔案、關閉資料庫 Cursor。但這個便利的語法在 Android 11 以下，對某些類別其實會出錯造成應用程式閃退。如果一時不查，小心這坑就這樣踩了下去...

## 錯誤說明

```kotlin
context
    .obtainStyledAttributes(attrs, R.styleable.ActionFooterView, 0, 0)
    .use {
        // ...
    }
```

這段程式碼在 Android 12（API 31）以上沒問題，但在 Android 11 (API 30) 以下執行時，會拋出以下錯誤：

```
java.lang.IncompatibleClassChangeError
Class 'android.content.res.TypedArray' does not implement interface 'java.lang.AutoCloseable' in call to 'void java.lang.AutoCloseable.close()'
```

白話來說，上面這個錯誤告訴我們，`TypedArray` 並沒有實作 `AutoCloseable` 介面，所以在試圖呼叫 `AutoCloseable.close()` 時拋出 `IncompatibleClassChangeError` 錯誤。

## 問題釐清：`use {}` 的背後原理

Kotlin `use {}` 是一個 extension function，會在 Block 結束後自動呼叫 `AutoCloseable.close()` 方法來釋放資源。

```kotlin
inline fun <T : AutoCloseable?, R> T.use(block: (T) -> R): R {
    try {
        return block(this)
    } finally {
        this?.close()  // ← 關鍵問題點
    }
}
```

但在 Android 11 (API 30) 以下的版本，`TypedArray` 並沒有實作 `AutoCloseable`，也就是說它根本不支援 `close()` 方法，導致執行期間會嘗試呼叫一個不存在的方法，最後拋出錯誤導致 App 閃退。

## 正確做法：使用 AndroidX 的 `use {}`

改用 `androidx.core.content.res.use` ，而它的實作方式如下：

```kotlin
inline fun <T> TypedArray.use(block: (TypedArray) -> T): T {
    try {
        return block(this)
    } finally {
        recycle()  // ← 這才是正確釋放資源的方式
    }
}
```

跟 Kotlin 版相比， `androidx.core.content.res.use` 是用 `recycle()` 來釋放資源，不會去呼叫那個不存在的 `AutoCloseable.close()`。

結論，如果你遇到這個問題，請記得將 `import androidx.core.content.res.use` 補上，就沒問題了。

## 延伸補充與建議

Kotlin 的 `use` 有兩種實作來源：

1. `kotlin.io.CloseableKt` 的 `Closeable.use {}` [source](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/jvm/src/kotlin/io/Closeable.kt)
2. `kotlin.AutoCloseable` 的 `AutoCloseable.use {}` [source](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/AutoCloseable.kt)

在開發時建議檢查類別是否有實作 `AutoCloseable` 或 `Closeable`。
如果有實作就可以安心使用 Kotlin 的 `use {}` 。
否則請改用 AndroidX 提供的 `use {}` 方法，或者手動呼叫 `recycle()` 或 `close()` 釋放資源。

舉例：`android.database.Cursor` 就有實作 `Closeable`，所以可以使用 Kotlin 的 `use {}`。

---

以上就是本次的踩坑筆記分享，希望能幫到你的忙，或是至少能幫到未來的我自己！？🤯  
下個坑見 🤪
