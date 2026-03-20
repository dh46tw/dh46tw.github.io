---
title: 如何檢查與移除 macOS 上的 JDK
tags:
  - macOS
  - JDK
  - Java
date: 2026-03-20
description: "整理 macOS 上檢查預設 JDK、列出所有已安裝版本，以及安全移除指定 JDK 的完整指令。"
keywords: ["macOS", "JDK", "java -version", "java_home -V", "移除 JDK", "刪除 Java 版本", "檢查 Java 版本"]
image: "assets/unsplash-macbook-with-coffee.jpg"
summary: "本文示範在 macOS 檢查 JDK 版本、查看已安裝的版本清單，並移除指定的 JDK。"
cover:
  image: "assets/unsplash-macbook-with-coffee.jpg"
  relative: true # 圖片在文章相對路徑
  hidden: false           # 不隱藏 cover
  hiddenInList: false     # 在列表頁顯示 cover
  hiddenInSingle: true   # 在單篇頁面顯示 cover
---

![MacBook with coffee](assets/unsplash-macbook-with-coffee.jpg)

> Photo by <a href="https://unsplash.com/@asci_en?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Artem R</a> on <a href="https://unsplash.com/photos/black-laptop-computer-beside-clear-glass-cup-arF0vKB0VPI?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

---

最近把用了 4 年的 MacBook Air M1 換掉了。  

在 2026 年來看，M1 作為一般日常使用依然是綽綽有餘，所以就想整理一下作為備用機跟家人的文書機。  

這時候才發現，原來我之前裝了這麼多版本的 JDK 啊...  

然後就有了這篇筆記，記錄怎麼用指令：

確認目前預設的 JDK、列出所有已安裝的版本，以及安全移除不再需要的 JDK。

## 檢查預設 JDK 版本

```bash
java -version
```

指令成功執行後，你應該會看到目前電腦預設的 JDK 版本。

類似這樣：

```bash                                                        
openjdk version "17.0.14" 2025-01-21
OpenJDK Runtime Environment JBR-17.0.14+1-1367.22-nomod (build 17.0.14+1-b1367.22)
OpenJDK 64-Bit Server VM JBR-17.0.14+1-1367.22-nomod (build 17.0.14+1-b1367.22, mixed mode, sharing)
```

## 檢查目前已安裝的 JDK 清單

```bash
/usr/libexec/java_home -V
```

指令成功執行後，你會看到目前電腦上安裝的所有 JDK 版本與路徑。

```
Matching Java Virtual Machines (1):
    17.0.14 (arm64) "JetBrains s.r.o." - "JBR-17.0.14+1-1367.22-nomod 17.0.14" {檔案路徑}
```

## 移除指定的 JDK

其實沒什麼特別的，就是用指令刪除指定路徑下的資料。
執行後，如果沒有跳出任何錯誤訊息，就代表刪除成功了。

**執行前務必確認路徑是否正確，避免誤刪。**

> 刪錯了是救不回來的喔...

由於以下範例位於使用者家目錄，通常不需要 `sudo`，也不需要輸入密碼。

```bash
rm -rf /Users/danielhuang/Library/Java/JavaVirtualMachines/azul-13.0.14
```

刪除完後可以再用上面提過的指令，檢查目前已安裝與預設的 JDK 是否正確。
