---
title: ✨SpringBoot 网卡获取忽略代理配置
published: 2024-04-21
pinned: false
description: ""
tags: [SpringBoot, ignoredInterfaces]
category: 技术分享
draft: false
image: ./images/firefly1.avif
---

> 修改 application.yml 配置文件

```yaml
spring:
    cloud:
        inetutils:
		        # 正则数组：忽略指定名称的网卡(或者可以添加preferredNetworks属性指定IP前缀)
            ignoredInterfaces: ['(.*?)Virtual(.*?)', 'Virtual(.*?)', 'Sangfor(.*?)', 'Clash(.*?)'] 
```
