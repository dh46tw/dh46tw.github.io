---
title: Android 常用 ADB 指令
tags:
  - Android
date: 2021-11-17T16:52:00
description: 介紹 Android 開發中常用的 ADB 指令
---

## 裝置

### 列出裝置清單

```shell=
adb devices
```

### 取得裝置 IP

```shell=
adb shell
$ ip -f inet addr show wlan0
```

### 啟用裝置 port

```shell=
adb -s {deviceId} tcpip {port}
```

### 透過Wi-Fi連結裝置

```shell=
adb -s {deviceId} connect {ip:port}
```

### 斷開連結

```shell=
adb disconnect
```

## 安裝apk

### 全新安裝

在xxxx.apk的根目錄下執行cmd

```shell=
adb -s {deviceId} install {xxxxx.apk}
```

### 覆蓋舊版

```shell=
adb -s {deviceId} install -r {xxxx.apk}
```

## 裝置控制

### 點亮螢幕

```shell=
adb shell input keyevent KEYCODE_POWER
```

## 參考資料

- [Day 8 - 常用 adb 指令及實用小技巧 ](https://ithelp.ithome.com.tw/articles/10241811)
- [如何透過 adb command line 指令啟動 Android App](https://kkboxsqa.wordpress.com/2014/08/20/%E5%A6%82%E4%BD%95%E9%80%8F%E9%81%8E-adb-command-line-%E6%8C%87%E4%BB%A4%E5%95%9F%E5%8B%95-android-app/)
