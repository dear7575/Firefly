---
title: ✨Nacos Docker-Compose 配置
published: 2024-10-22
pinned: false
description: "记录 Nacos 使用 Docker Compose 部署的配置示例，包含容器参数、环境变量和启动配置。"
tags: [Nacos, Docker-Compose]
category: 技术分享
draft: false
image: ../uploads/20260110/mt.png
---

# Nacos Docker-Compose 配置

```yaml
services:
    nacos:
        image:
            nacos/nacos-server
        container_name:
            nacos
        restart:
            always
        network_mode:
            bridge
        ports:
            - 8848:8848 # Nacos 控制台端口
            - 9848:9848 # Nacos 服务端口
        environment:
            - MODE=standalone # 单机模式
            - MYSQL_SERVICE_HOST=127.0.0.1
            - MYSQL_SERVICE_PORT=3306
            - MYSQL_SERVICE_DB_NAME=indicator_cloud_nacos
            - MYSQL_SERVICE_USER=root
            - MYSQL_SERVICE_PASSWORD=123456
            - SPRING_DATASOURCE_PLATFORM=mysql
            - NACOS_AUTH_ENABLE=true
            - NACOS_AUTH_TOKEN=VGhpc0lzTXlDdXN0b21TZWNyZXRLZXkwMTIzNDU2Nzg=
            - NACOS_AUTH_IDENTITY_KEY=nacos
            - NACOS_AUTH_IDENTITY_VALUE=nacos
        volumes:
            - ./data/log:/home/nacos/logs  # 持久化日志文件
            - ./data/conf/application.properties:/home/nacos/conf/application.properties

```
