+++
date = '2023-04-27T15:16:00+08:00'
draft = false
title = 'Git 更新版本'
tags = ['Git']
description = '如何查詢 Git 目前版本並更新至新版'
+++

## 1. 檢查目前 Git 版本

打開命令提示字元，輸入以下指令檢查目前 Git 版本

```git=
git version
```

## 2. 根據版本選擇不同更新方式

### 2-1 早於 2.14.1 版

請到 Git 官網下載新版，解除安裝本機的 Git 後手動安裝新版。

### 2-2 版本介於 2.14.2 和 2.16.1 之間

打開命令提示字元，輸入以下指令更新。

```git=
git update
```

### 2-3 版本 2.16.1 以上

打開命令提示字元，輸入以下指令更新。

```git=
git update-git-for-windows
```

執行後會於 CMD 自動下載，不過安裝過程還是會跳 GUI 要你選擇不同安裝選項。


## 參考資料

1. [Poy Chang 更新本機 Git 到最新版](https://blog.poychang.net/git-update-to-latest-version/)

> 文章同步發表在
> [HackMD](https://hackmd.io/@dh46tw/update-git)
