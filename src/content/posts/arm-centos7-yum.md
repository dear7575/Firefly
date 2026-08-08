---
title: ✨Dockerfile 编译CentOS7基础镜像替换YUM源
published: 2024-07-11
pinned: false
description: "记录基于 Dockerfile 构建 CentOS 7 基础镜像时替换 YUM 源、安装 EPEL 源和更新缓存的配置。"
tags: [Docker, Dockerfile]
category: 技术分享
draft: false
image: ../uploads/20260110/mt.png
---


# Dockerfile 编译CentOS7基础镜像替换YUM源

> Dockerfile 文件编辑

```dockerfile
FROM centos:7

# 设置镜像源并更新yum源，同时安装 epel-release 并替换 EPEL 源
RUN curl -o /etc/yum.repos.d/CentOS-Base.repo <http://mirrors.aliyun.com/repo/Centos-altarch-7.repo> && \\
    yum clean all && \\
    yum makecache && \\
    yum install -y epel-release && \\
    curl -o /etc/yum.repos.d/epel.repo <http://mirrors.aliyun.com/repo/epel-7.repo> && \\
    yum clean all && \\
    yum makecache
```
