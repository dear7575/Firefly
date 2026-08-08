---
title: ✨Google 同步操作
published: 2024-04-25
pinned: false
description: "记录 Chrome/Google 同步异常时的排查步骤，包含代理模式、同步页面和 sync-internals 日志检查。"
tags: [Google, Sync]
category: 技术分享
draft: false
image: ../uploads/20260110/yy.png
---

> 操作步骤如下

1. 开启代理软件，如Clash，并切换到全局模式。
1. 打开网址 [https://chrome.google.com/sync?hl=zh-CN，](https://chrome.google.com/sync?hl=zh-CN%EF%BC%8C) 查看底部的同步时间。

1. 或者可以打开chrome://sync-internals/，查看 Sync Protocol Log 是否有日志输出。
