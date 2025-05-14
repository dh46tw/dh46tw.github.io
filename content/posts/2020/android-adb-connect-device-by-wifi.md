---
title: Android 使用 ADB 指令以 Wi-Fi 連接手機
tags:
  - Android
date: 2020-07-21
description: 介紹在 Android 開發中，如何使用 ADB 指令透過 Wi-Fi 連接手機
---

## 前言

以往要連接測試裝置都一定要使用實體傳輸線。
但有時候因為 App 的功能或需求不同，會需要以無線方式連接進行除錯。
爬文後發現方法不困難，只要透過 adb 指令即可。

## 步驟一、前置作業

請先準備一條手機傳輸線與你要連接的手機。

## 步驟二、接上線後開啟 CMD 視窗

列出目前連線的裝置，並記下要連線的裝置 ID。

```cmd
$ adb devices

List of devices attached
98181FFAZ00814  device
emulator-5554   device
```

## 步驟三、設定裝置監聽的 Port 號

```cmd
$ adb -s 98181FFAZ00814 tcpip 5555

restarting in TCP mode port: 5555
```

## 步驟四、取得裝置目前的 IP 位址

這裡有兩種方式，一種是直接操作手機，進到設定中的狀態查看IP位址。
另一種方式則是使用 adb 命令直接看。

```cmd
// 進入裝置
$ adb -s 98181FFAZ00814 shell

// 查詢 IP 位址
$ ip -f inet addr show wlan0

30: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 3000
    inet 192.168.8.47/24 brd 192.168.8.255 scope global wlan0
       valid_lft forever preferred_lft forever

// 下 exit 離開裝置
$ exit
```

## 步驟五、斷開 USB 連接，以 IP 連線

```cmd
$ adb connect 192.168.8.47:5555

connected to 192.168.8.47:5555
```

這樣子就完成囉！

## 參考來源

- [adb 透過 Wi-Fi 連線裝置](https://blog.tonycube.com/2017/02/adb-wi-fi.html)
