---
title: Android 取得即時定位 LocationManager
tags:
  - Android
  - Android/Location
  - GPS
date: 2023-05-24
---

> 📢 除非有特殊需求，官方建議使用 Google Fused Location Provider API。

## 權限請求

### AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <!--  宣告定位權限  -->
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

    <application>
        <!-- 略 -->
    </application>

</manifest>
```

### 請求動態權限

因為定位屬於危險權限需要動態向使用者請求。

```kotlin
import android.Manifest
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts

class MainActivity : AppCompatActivity() {

    // -----------------------Request Location Permission------------------------------
    private val resultLauncherPermission =
        registerForActivityResult(ActivityResultContracts.RequestPermission()) {
            if (it) {
                initLocationManager()
            } else {
                Toast.makeText(this, "請允許權限以開啟定位", Toast.LENGTH_SHORT).show()
                finish()
            }
        }

    // -----------------------------LifeCycle Event-------------------------------------

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        // 啟動時請求權限
        resultLauncherPermission.launch(Manifest.permission.ACCESS_FINE_LOCATION)
    }

    // --------------------------------------------------------------------------------

    private fun initLocationManager() {
        Toast.makeText(this, "定位權限請求成功，準備初始化 LocationManager", Toast.LENGTH_SHORT).show()
    }
}
```

## LocationManager 實例化

```kotlin
import android.location.LocationManager

val locationManager = getSystemService(Context.LOCATION_SERVICE) as LocationManager
```

## 請求定位

Android S 以上支援 `FUSED_PROVIDER`，不過建議直接用 [Android Fused Location Provider API](/Adhk0WeUSkSzV_Sw4MsS5Q)，支援的系統版本比較廣。

```kotlin
@SuppressLint("MissingPermission")
private fun requestLocationUpdates() {
    val minTimeMs = 500L
    val minDistanceM = 0.0f
    val provider = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
        LocationManager.FUSED_PROVIDER
    } else {
        LocationManager.GPS_PROVIDER
    }
    locationManager?.requestLocationUpdates(
        provider,
        minTimeMs,
        minDistanceM,
        locationListener
    )
}
```

## 輸出 `LocationListener.onLocationChanged()`

> 📢 注意 [除了 `LocationManager.GPS_PROVIDER` 外，其餘在 `Location.getTime()` 拿到的都不是 GPS 時間]()

```kotlin
private val locationListener = object : LocationListener {
    override fun onLocationChanged(location: Location) {
        val output = "${location.provider}: ${location.longitude}, ${location.latitude}, ${location.altitude}"
        Log.d(TAG, "onLocationChanged: $output")
    }
}
```

## 停止定位更新

```kotlin
private fun stopLocationUpdates(){
    locationManager?.removeUpdates(locationListener)
}
```

## 參考資料

- [Android Doc: LocationManager](https://developer.android.com/reference/android/location/LocationManager)
