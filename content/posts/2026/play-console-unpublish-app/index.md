---
title: 我以為是權限問題，結果是 Google Play 的控管型發布讓我不能下架 App
tags:
  - Android
  - Google Play
date: 2026-01-06
description: "在 Google Play Console 中嘗試取消發布 App，卻發現「未發布」選項無法選取？本文說明造成無法解除發布的真正原因，並一步步教你如何關閉控管型發布功能，順利將 App 從 Google Play 下架。"
keywords: ["android", "Google Play", "Unpublish App", "Google Play 取消發布", "Google Play 未發布 無法選取", "Google Play 無法解除發布", "Google Play 下架 App", "Android App 取消上架"]
image: /attachments/play-console-unpublish-app.png
summary: "記錄在 Google Play 取消發布 App 時，因未關閉「控管型發布」導致「未發布」選項無法選取的問題。說明官方限制條件，並整理關閉控管型發布後即可順利解除發布的完整操作流程，作為備忘錄。"
cover:
  image: "attachments/play-console-unpublish-app.png"
  relative: true # 圖片在文章相對路徑
  hidden: false           # 不隱藏 cover
  hiddenInList: false     # 在列表頁顯示 cover
  hiddenInSingle: true   # 在單篇頁面顯示 cover
---

前幾天被 PM 要求將某個 App 從 Google Play 上取消發布。  
憑著肌肉記憶找到了取消發布的設定，卻發現 **未發布** 的選項無法選取。

![Play Console 中的未發布選項無法選取](/attachments/play-console-unpublish-app.png)

```
💡 取消發布的設定路徑  
 
1. 點選左側選單的 測試及發布/進階設定
2. 應用程式適用國家/地區  
3. 勾選未發布  
4. 點選右下角 解除發布  
```

---  

一開始以為是帳號權限的問題，結果翻了[官方文件](https://support.google.com/googleplay/android-developer/answer/9859350?hl=zh-Hant#)才發現，要取消發布 App 必須滿足以下條件：  

1. 您已接受最新版《開發人員發布協議》。
2. 您的應用程式已經沒有需要解決的錯誤，例如尚未填寫內容分級問卷，或尚未提供有關應用程式目標對象和內容的詳細資料。  
3. **需要取消發布的應用程式並未開啟控管型發布功能。**

而我遇到的狀況就是，**沒有關閉控管型發布功能**。

```
💡 如何關閉控管型發布功能？  

1. 點選左側選單上的 發布總覽 
2. 點選畫面上的 關閉控管型發布功能
```

關閉控管型發布後，未發布的按鈕就可以選取，並完成解除發布。

這個設定不常用到，每次要用的時候都會卡一下。  
簡單筆記一下整個流程，加強記憶也提醒未來的自己~  
