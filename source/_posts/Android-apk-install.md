---
title: Android开发-Apk安装步骤
date: 2021-08-19 17:10:29
tags: Android
categories: Android
---

Apk 安装的主要步骤：

1. 将 apk 文件复制到 `data/app` 目录
2. 解析 apk 信息
3. dexopt 操作
4. 更新权限信息
5. 完成安装，发送 `Intent.Action_PACKAGE_ADDED` 广播
