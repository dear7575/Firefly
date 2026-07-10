---
title: ✨Nginx Proxy Manager 相关配置
published: 2024-07-25
pinned: false
description: "记录 Nginx Proxy Manager 的 Docker Compose 部署配置、容器参数和常用反向代理相关设置。"
tags: [Nginx, 配置]
category: 技术分享
draft: false
image: ../uploads/20260110/yy.png
---

# Nginx Proxy Manager 相关配置

> docker-compose.yml 文件配置 

```yaml
services:
  nginx-web:
    image:
        jc21/nginx-proxy-manager:latest
    container_name:
        nginx-web
    restart:
        always
    network_mode:
        bridge
    ports:
        - 80:80
        - 81:81
        - 443:443
    volumes:
        - ./data:/data
        - ./letsencrypt:/etc/letsencrypt
```

> 如果需要设置Nginx Proxy [Manager使用域名访问则需要在阿里云解析中增加xxxx.dear7575.cn](http://xn--Managerxxxx-618qt4bg9fpzn17bg7j57pjjdk3et12ijr2cf27ej6b47s1t3bclx55b35i.dear7575.cn)，且将下图配置IP改成Docker主IP，如:172.17.0.1

---

> ![1.png](../uploads/20240725/20260107_223515_5f14033c.png)



> ![2.png](../uploads/20240725/20260107_223515_f1ace7c2.png)



> ![3.png](../uploads/20240725/20260107_223515_5bf0c057.png)
