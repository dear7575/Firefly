---
title: ✨Docker 启动异常 containerd did not exit successfully
published: 2024-05-13
pinned: false
description: "记录 Docker 因异常关闭导致 containerd 启动失败时，定位并清理损坏 meta.db 文件的处理方法。"
tags: [Docker]
category: 技术分享
draft: false
image: ../uploads/20260110/mt.png
---

> 原因是容器非正常关闭导致容器文件损坏

```markdown

## 1.查找文件

    find / -name meta.db

## 2.删除meta.db文件

    rm -rf /xxx/xxx/meta.db

```
