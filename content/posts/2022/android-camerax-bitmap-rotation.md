---
title: Android CameraX Bitmap 繪製後照片方向錯誤
tags:
  - Android/Camera
date: 2022-03-25
---

## 問題說明

在使用 CameraX API 時，如果有在相機初始時實作下列程式碼，原則上API 會根據目前手機的方向來自動轉正相片。

```kotlin
// 控制照片轉正
val orientationEventListener = object : OrientationEventListener(this) {
    override fun onOrientationChanged(orientation: Int) {
        val rotation: Int = when (orientation) {
            in 45..134 -> Surface.ROTATION_270 // 頭朝右
            in 135..224 -> Surface.ROTATION_180 // 頭朝下
            in 225..314 -> Surface.ROTATION_90 // 頭朝左
            else -> Surface.ROTATION_0 // 頭朝上
        }
        this@MainActivity.rotation = rotation
        imageCapture!!.targetRotation = rotation
    }
}
orientationEventListener.enable()
```

但是，當我們需要在輸出的照片上繪製文字或圖案時，就會遇到讀入的Bitmap 都是預設的橫向，而當我們需要繪製的時候，就會有長寬座標錯置的問題。

## 解決方式

**必須把照片轉兩次。**

需要轉兩次的原因在於，第一次的轉正是為了讓我們能夠根據使用者的角度，去繪製我們需要繪製的圖案或文字，但當繪製完成後，在輸出之前必須再將照片轉回原始讀入時的橫向，讓電腦端能夠用EXIF來自動轉正相片。

> 轉正跟轉回，可以試著拿張名片或紙自己轉轉看就會懂了。

實作的程式碼如下

```kotlin
companion object {
   private const val ROTATION_HEAD_UP = 0
   private const val ROTATION_HEAD_LEFT = 1
   private const val ROTATION_HEAD_DOWN = 2
   private const val ROTATION_HEAD_RIGHT = 3
}


private fun processPictureFile(photoFile: File) {
    
    // 建立 Bitmap的設定
    val bitmapOptions = BitmapFactory.Options()
    bitmapOptions.inPurgeable = true
    bitmapOptions.inPreferredConfig = Bitmap.Config.RGB_565
    bitmapOptions.inDither = true
    bitmapOptions.inMutable = true // 避免Canvas物件建立時picBitmap為immutable
    
    // 1. 旋轉照片
    val mat = Matrix()
    when (rotation) {
        ROTATION_HEAD_UP -> mat.postRotate(90f)
        // 手機上端朝右時，照片會上下顛倒，需特別處理轉正。
        ROTATION_HEAD_RIGHT -> mat.postRotate(180f)
        ROTATION_HEAD_DOWN -> mat.postRotate(270f)
        ROTATION_HEAD_LEFT -> mat.postRotate(0f)
    }
    try {
        // 讀入原始相片
        val inputStream = ByteArrayInputStream(photoFile.readBytes())
        // 原始相片轉成 Bitmap
        val bitmap = BitmapFactory.decodeStream(inputStream, null, bitmapOptions) ?: return
        
        Log.d(TAG, "processPictureFile: bitmap.width ${bitmap.width}")
        Log.d(TAG, "processPictureFile: bitmap.height ${bitmap.height}")
        
        // 複製一份要轉方向的 Bitmap
        val picBitmap =
            Bitmap.createBitmap(bitmap, 0, 0, bitmap.width, bitmap.height, mat, true)
        // 建立畫布
        val canvas = Canvas(picBitmap)
        // 繪製在畫布上
        canvas.drawBitmap(bitmap, mat, null)
        
        // TODO: 執行你需要做的繪製行為...
        
        // 設定轉回去的矩陣
        val restoreMat = Matrix()
        when (rotation) {
            ROTATION_HEAD_UP -> restoreMat.postRotate(270f)
            ROTATION_HEAD_RIGHT -> restoreMat.postRotate(180f)
            ROTATION_HEAD_DOWN -> restoreMat.postRotate(90f)
            ROTATION_HEAD_LEFT -> restoreMat.postRotate(0f)
        }
        
        // 執行轉回去
        val restoreBitmap = Bitmap.createBitmap(picBitmap, 0, 0, picBitmap.width, picBitmap.height, restoreMat, true)
        val restoreCanvas = Canvas(restoreBitmap)
        restoreCanvas.drawBitmap(restoreBitmap, mat, null)
        
        // TODO: 輸出...
    } catch (e: java.lang.Exception) {
        Log.e(TAG, "processPictureFile: ", e)
    }
}
```

## 補充: 相片轉正的邏輯

對 Android 手機而言，頭朝左的水平方向(landscape)，才是正常的方向，所以預設讀取的圖片都會是橫向。
因此直向拍照等方式都需要轉向。

- ROTATION_HEAD_UP -> 90 CW: 裝置頭朝上，橫向圖片順時鐘轉 90
- ROTATION_HEAD_RIGHT -> 180 CW: 裝置頭朝右，橫向圖片順時鐘轉 180
- ROTATION_HEAD_DOWN -> 270 CW: 裝置頭朝下，橫向圖片順時鐘轉 270
- ROTATION_HEAD_LEFT -> 0: 裝置正常位置，橫向圖片不用轉
