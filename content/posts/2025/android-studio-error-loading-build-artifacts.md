---
title: Android Studio 編譯失敗 - Error loading build artifacts
tags:
  - Android/Build
date: 2025-09-12
description: Android Studio 出現「Error loading build artifacts」導致無法編譯？本文提供詳細錯誤訊息、排查步驟與最終修復方式，協助開發者快速解決問題。
---

前幾天在公司專案遇到一個奇怪的 Bug，這裡筆記一下解決過程。

---

## 問題描述

在 Android Studio 點選 `Run 'app'` 時，專案完全沒編譯就直接報錯，錯誤訊息如下：

```
Error loading build artifacts from: D:\Daniel\Projects\cropssurvey-mk2-android\app\build\intermediates\apk_ide_redirect_file\debug\createDebugApkListingFileRedirect\redirect.txt
```

## 環境資訊

我的 Android Studio 設定參數如下：

```
Android Studio Meerkat Feature Drop | 2024.3.2 Patch 1
Build #AI-243.26053.27.2432.13536105, built on May 22, 2025
Runtime version: 21.0.6+-13368085-b895.109 amd64
VM: OpenJDK 64-Bit Server VM by JetBrains s.r.o.
Toolkit: sun.awt.windows.WToolkit
Windows 11.0
GC: G1 Young Generation, G1 Concurrent GC, G1 Old Generation
Memory: 4096M
Cores: 16
Registry:
  debugger.new.tool.window.layout=true
  ide.experimental.ui=true
Non-Bundled Plugins:
  one.util.ideaplugin.screenshoter (1.8.1)
  com.wzc.sw.plugin (1.3.3)
  Dart (243.26753.1)
  com.duke.screenmatch (3.2)
  idea.plugin.protoeditor (243.22562.13)
  com.developerphil.adbidea (1.6.19)
  com.godwin.kdocer (1.6)
  org.sonarlint.idea (10.27.0.81781)
  io.flutter (86.0.1)
```

## 嘗試過的解法（失敗）

- Sync Project with Gradle Files
- Rebuild Project
- Invalid Cache and Restart
- 重開 Android Studio
- 刪除 `/.gradle` 與 `/.idea` 兩個資料夾。[參考來源](https://blog.csdn.net/zhangphil/article/details/142108246)

**結果，以上方法，都 沒 有 用 ...**

一度想說乾脆重新從遠端 Clone 一份新的 Code 下來，再把目前的變更用 Patch 還原回去，但光想到就覺得麻煩。

## 真正的解法

後來在 StackOverflow 上找到這篇討論 [IDEA does not always deploy latest changes](https://stackoverflow.com/questions/75360810/idea-does-not-always-deploy-latest-changes)。

雖然是 IntelliJ IDEA 的案例，但畢竟 Android Studio 是 IntelliJ 的分支，看起來滿值得一試的。

### 解決步驟

1. 開啟 Run/Debug Configurations
2. 點選視窗左上角的 + ，新增一個新的 Android App Configuration
3. 填寫 Configuration Name 與選擇要執行的 Module
4. 按下 Apply 儲存
5. 執行剛才建立的 Configuration

**以上步驟做完，就可以成功 build 了！🎉**

## 結語

沒想到錯誤會是在 Run Configuration，記下來給未來的自己，避免下次又掉進同一個坑。

**除蟲完畢，下次見！**