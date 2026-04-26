---
title: ✨Docker 启动异常 containerd did not exit successfully
published: 2024-05-13
pinned: false
description: ""
tags: [Docker]
category: 技术分享
draft: false
image: ./images/firefly1.avif
---

> 原因是容器非正常关闭导致容器文件损坏

```markdown

## 1.查找文件

    find / -name meta.db

## 2.删除meta.db文件

    rm -rf /xxx/xxx/meta.db

```
