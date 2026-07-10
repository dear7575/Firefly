---
title: ✨Docker 和 Docker-Compose离线安装教程
published: 2024-04-22
pinned: false
description: "记录 Linux 环境下 Docker 与 Docker Compose 的离线安装流程、防火墙处理和常用配置步骤。"
tags: [Docker, Docker-Compose]
category: 技术分享
draft: false
image: ../uploads/20260110/mt.png
---

# Docker 和 Docker-Compose离线安装教程

> 全程需要使用Linux命令和Docker命令进行操作

```markdown
# 防火墙

## 1.查看防火墙状态

    systemctl status firewalld

## 2.关闭防火墙状态

    systemctl stop firewalld

## 3.禁止防火墙状态

    systemctl disable firewalld

# 下载镜像替换

## 1.阿里镜像地址

    <https://developer.aliyun.com/mirror/>

## 2.替换下载镜像 X86

    wget -O /etc/yum.repos.d/CentOS-Base.repo <https://mirrors.aliyun.com/repo/Centos-7.repo>

## 3.替换下载镜像 ARM

    wget -O /etc/yum.repos.d/CentOS-Base.repo <http://mirrors.aliyun.com/repo/Centos-altarch-7.repo> 

## 4.清除缓存

    yum clean all

## 5.生成缓存

    yum makecache

## 6.重新加载

	yum repolist

## 7.更新

    yum -y update

# Docker X86 安装

## 1.安装基础组件

    yum install -y yum-utils device-mapper-persistent-data lvm2

## 2.配置yum源

    yum-config-manager --add-repo <https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo>

## 3.安装 docker

    yum -y install docker-ce

## 4.编辑 docker配置文件

    vim /etc/docker/daemon.json

## 5.创建 docker 目录

    mkdir -p /home/docker

## 6.快速创建命令 加载镜像加速站点 配置文件内容：graph代表docker指定的安装目录

	mkdir -p /etc/docker
	tee /etc/docker/daemon.json <<-'EOF'
	{
		"registry-mirrors": ["<https://tzc8ht4r.mirror.aliyuncs.com>"]
	}
	EOF

## 7.加载镜像

    systemctl daemon-reload

## 8.启动docker 并且设置开机启动

    systemctl enable docker && systemctl restart docker

## 9.进入容器

    docker exec -it 容器名 /bin/bash

## 10.退出容器

    exit;

## 11.查看所有容器

    docker ps -a

## 12.查看所有正在运行的容器

    docker ps 

## 13.暂停容器

    docker stop 容器ID

## 14.删除容器

    docker rm 容器ID 

## 15.停止所有容器

    docker stop $(docker ps -a -q)

## 16.删除所有容器

    docker rm $(docker ps -a -q)

## 17.删除镜像

    docker rmi 镜像ID

## 18.重启Docker

    systemctl restart docker

## 19.停止Docker

    systemctl stop docker

## 20.命令用于使用 Dockerfile 创建镜像

	docker build -t 镜像名称 .

## 21.export 导出镜像(导入导出镜像名称最好保持一致)

	docker export 镜像ID > 镜像名称.tar

## 22.import 导入镜像

	docker import - 镜像名称 < 镜像名称.tar

## 23. save 导出镜像 

	docker save -o 镜像.tar 镜像:版本  
	docker save -o service.tar service:latest

## 24. load 导入镜像

	docker load < service.tar

## 25. 删除None 镜像

	docker image prune -f

## 26. 离线安装一键安装详见工作微信收藏

# Docker ARM 安装

## 卸载

	apt remove docker
	sudo apt-get autoremove docker docker-ce docker-engine <http://docker.io> containerd runc

## 下载安装

	1.wget <https://download.docker.com/linux/static/stable/aarch64/docker-20.10.9.tgz>
	2.tar -zxvf docker-20.10.9.tgz
	3.mv docker/* /usr/bin/
	4.dockerd

## 添加 systemd

```shell
vim /usr/lib/systemd/system/docker.service
```

```ruby
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
[Service]
Type=notify
ExecStart=/usr/bin/dockerd --default-ulimit nofile=65536:65536
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
[Install]
WantedBy=multi-user.target
```

## 为docker.service添加执行权限

```
1.chmod +x /usr/lib/systemd/system/docker.service
2.systemctl daemon-reload
```

## 编辑daemon.json

```shell
vi /etc/docker/daemon.json

{
  "registry-mirrors": ["<http://hub-mirror.c.163.com>"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}

systemctl daemon-reload

systemctl start docker

systemctl enable docker
```

# docker-compose 安装

## 1.下载docker-compose包

```
curl -L <https://github.com/docker/compose/releases/download/v2.15.1/docker-compose-`uname> -s`-`uname -m` > /usr/local/bin/docker-compose
```

## 2.设置执行权限

```
chmod +x /usr/local/bin/docker-compose
```

# 查看服务器信息

```
uname -a
uname -m
cat /etc/issue
cat /proc/cpuinfo
cat /etc/kylin-release
lscpu
```

## **清理docker磁盘**

```shell
#类似于Linux上的df命令，用于查看Docker的磁盘使用情况:
docker system df

#可以用于清理磁盘，删除关闭的容器、无用的数据卷和网络，以及dangling镜像(即无tag的镜像)。 
docker system prune

#清理得更加彻底，可以将没有容器使用Docker镜像都删掉。注意，这两个命令会把你暂时关闭的容器，以及暂时没有用到的Docker镜像都删掉了…
#所以使用之前一定要想清楚.。我没用过，因为会清理 没有开启的  Docker 镜像。
docker system prune -a

#docker system df -v 命令可以进一步查看空间占用细节，以确定是哪个镜像、容器或本地卷占用过高空间
docker system df -v 

#自动清理命令
docker system prune -a -f
#删除无用的卷
docker volume prune
#删除无用的网络
docker network prune
#查看占用运行内存
docker stats --no-stream --format "{{.Container}}: {{.MemUsage}}" | sort -k2 -h
#查看容器IP
docker ps -q | xargs -n 1 docker inspect --format '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' | sed 's/^\\///'
```

## **启动异常处理**

```
1.删除/etc/docker/daemon.json
2.启动docker
3.再配置/etc/docker/daemon.json
```

## 常用命令

### 查看端口

```
netstat -luntp
```

### 快速创建多个文件夹

```
mkdir -p ./xxx/{xxx,xxx,xxx}
```

### 快速创建文件

> touch


### 固定MAC地址

```
    networks:
      parsing_network:
        mac_address: "fa:c7:17:f0:06:74"
```
